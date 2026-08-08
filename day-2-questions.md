Absolutely. These are the **notes from your questions and the clarifications we discussed**, written so you can add them directly to your Day-2 running notes.

# Day-2 — Important Questions & Kubernetes Internal Flow

## 1. Kubernetes Pod States

A Kubernetes Pod has **five official Pod phases**:

| Pod Phase   | Meaning                                                                                                     |
| ----------- | ----------------------------------------------------------------------------------------------------------- |
| `Pending`   | Pod has been accepted by the cluster, but one or more containers are not yet running                        |
| `Running`   | Pod has been assigned to a node and at least one container is running or starting                           |
| `Succeeded` | All containers terminated successfully                                                                      |
| `Failed`    | All containers terminated and at least one container failed                                                 |
| `Unknown`   | Kubernetes cannot determine the Pod's current state, usually because communication with the node has failed |

### Important

Do not confuse **Pod phases** with the statuses/reasons displayed by `kubectl get pods`.

For example:

```text
Pending
Running
Succeeded
Failed
Unknown
```

are Pod phases.

Whereas:

```text
CrashLoopBackOff
ImagePullBackOff
ContainerCreating
ErrImagePull
```

are not Pod phases. They indicate particular conditions/reasons associated with the workload.

---

# 2. What Happens When We Apply a Pod YAML for the First Time?

Suppose we have:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

We execute:

```bash
kubectl apply -f pod.yml
```

## Internal Flow

```text
User
  │
  │ kubectl apply
  ▼
API Server
  │
  ├── Authentication
  ├── Authorization
  ├── Admission
  └── Validation
  │
  ▼
etcd
  │
  │ Pod object is persisted
  ▼
Scheduler
  │
  │ Selects suitable Worker Node
  ▼
API Server
  │
  │ Pod gets node assignment
  ▼
Kubelet
  │
  ▼
Container Runtime
  │
  ├── Pull image if required
  ├── Create container
  └── Start container
  │
  ▼
Pod Running
```

### Important Mental Model

When the Pod is first created, the API Server creates/persists the **Pod object**.

At that point, the Pod may exist as an object but still not have a Worker Node assigned.

Conceptually:

```text
Pod Object
    │
    ├── Exists
    │
    └── nodeName = empty
```

The Scheduler later selects a suitable Worker Node.

After scheduling:

```text
Pod Object
    │
    └── nodeName = worker-node-1
```

The kubelet on that Worker Node then takes responsibility for getting the Pod running.

---

# 3. What Happens If I Apply the Exact Same YAML Again?

Suppose the Pod already exists:

```text
Pod
name = nginx
```

Now we execute again:

```bash
kubectl apply -f pod.yml
```

The request **again goes to the API Server first**.

It does NOT go directly to the Scheduler.

```text
kubectl
   │
   ▼
API Server
   │
   ▼
Existing Pod object
   │
   ▼
Compare requested configuration
with existing object
```

If there is no meaningful change:

```text
Existing configuration
        =
Requested configuration
        │
        ▼
No new Pod is created
```

Therefore:

```text
First apply
    ↓
Create Pod

Second identical apply
    ↓
No duplicate Pod
```

### Important Mental Model

Do not think:

```text
API Server
   ↓
Ask etcd: "Does it exist?"
   ↓
etcd decides
```

Instead think:

> The API Server handles the API request and uses etcd as the persistent data store for Kubernetes objects.

**etcd stores state; it is not the component that makes Kubernetes decisions.**

---

# 4. Does Every New Pod Request Go to the Scheduler First?

## No.

This is an important correction.

The Scheduler is **not the first entry point**.

Incorrect:

```text
User
  ↓
Scheduler
  ↓
API Server
```

Correct:

```text
User
  ↓
API Server
  ↓
Pod Object
  ↓
Scheduler notices an unscheduled Pod
  ↓
Scheduler selects Worker Node
  ↓
API Server
  ↓
Kubelet
  ↓
Container Runtime
  ↓
Pod
```

### Golden Rule

> **API Server is the front door of Kubernetes.**

The Scheduler is one of the Control Plane components that works after the Pod object has been accepted into the Kubernetes API.

---

# 5. How Does the Scheduler Know About Worker Node Resources?

The Scheduler needs information about Worker Nodes before deciding where a Pod should run.

Conceptually:

```text
Worker Node
    │
    │ Node status/resource information
    ▼
Kubelet
    │
    ▼
API Server
    │
    ▼
Scheduler
```

However, do NOT imagine that every time a Pod arrives:

```text
Scheduler
   ↓
Ask Node-1 kubelet
   ↓
Ask Node-2 kubelet
   ↓
Ask Node-3 kubelet
```

That is not the normal scheduling model.

The Scheduler receives Kubernetes object/state information through the API Server and maintains an **internal scheduling cache** containing information needed for scheduling decisions.

---

# 6. What Information Does Scheduler Use?

The Scheduler considers information such as:

```text
Worker Node
 ├── CPU capacity
 ├── Memory capacity
 ├── Allocatable resources
 ├── Existing Pod resource requests
 ├── Node labels
 ├── Taints
 ├── Node conditions
 ├── Affinity/anti-affinity
 └── Other scheduling constraints
```

Example:

```text
Node-1

Allocatable CPU     = 4 CPU
Pod requests        = 3 CPU

Remaining/requestable capacity
                      = approximately 1 CPU
```

If a new Pod requests:

```yaml
resources:
  requests:
    cpu: "2"
```

Node-1 may not be suitable.

The Scheduler considers other nodes.

---

# 7. Metrics vs Scheduling Information

This distinction is extremely important.

Do not assume:

> Scheduler gets CPU utilization metrics from Metrics Server and then decides where to put the Pod.

Normal scheduling primarily works with **resource requests and node allocatable capacity**, along with scheduling constraints.

For example:

```text
Node capacity
      ↓
Node allocatable
      ↓
Existing Pod requests
      ↓
New Pod requests
      ↓
Scheduler decision
```

This is different from monitoring metrics such as:

```text
Current CPU utilization = 82%
Current memory utilization = 71%
```

Those metrics are commonly used by monitoring/autoscaling systems and are not simply the input used for every normal scheduling decision.

---

# 8. How Does Scheduler Select the Node?

Suppose:

```text
New Pod

CPU request = 2 CPU
Memory request = 2Gi
```

There are three Worker Nodes:

```text
             Scheduler
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
    Node-1     Node-2    Node-3
      ✗           ✓         ✓
```

The Scheduler first eliminates nodes that cannot satisfy the Pod's requirements.

Then it evaluates suitable candidates and selects an appropriate node.

Simplified:

```text
Pending Pod
    ↓
Find eligible nodes
    ↓
Filter unsuitable nodes
    ↓
Score/select suitable nodes
    ↓
Choose Worker Node
```

We will learn the actual **filtering and scoring process** later when we reach Scheduler in depth.

---

# 9. After Scheduler Selects the Node, Who Does It Inform?

The Scheduler does not normally directly call the kubelet and say:

> "Hey Node-2, run this Pod."

Instead, the scheduling decision is recorded through the Kubernetes API.

```text
Scheduler
    │
    │ selects Node-2
    ▼
API Server
    │
    ▼
Pod object updated
nodeName = Node-2
    │
    ▼
Kubelet on Node-2
    │
    ▼
Container Runtime
    │
    ▼
Container
```

The kubelet observes the Kubernetes state and sees that a Pod has been assigned to its node.

It then works to make that Pod actually run.

---

# 10. Does Controller Manager Come Between Scheduler and Kubelet?

For a **directly created bare Pod**, don't put Controller Manager into the main Pod scheduling flow.

Example:

```yaml
kind: Pod
```

The simplified flow is:

```text
kubectl
   ↓
API Server
   ↓
etcd
   ↓
Scheduler
   ↓
API Server
   ↓
Kubelet
   ↓
Container Runtime
   ↓
Pod
```

There is no Deployment or ReplicaSet managing that Pod.

---

# 11. When Does Controller Manager Become Important?

Controller Manager becomes important when we use Kubernetes resources that are managed by controllers.

For example:

```text
Deployment
ReplicaSet
Job
DaemonSet
StatefulSet
Node Controller
```

At a high level:

```text
Deployment
     ↓
Deployment Controller
     ↓
ReplicaSet
     ↓
Pod
     ↓
Scheduler
     ↓
Kubelet
```

The controller's job is to continuously work toward the desired state.

---

# 12. Desired State vs Current State

Suppose a Deployment says:

```yaml
replicas: 3
```

Kubernetes wants:

```text
Desired State = 3 Pods
```

Suppose only two Pods are currently running:

```text
Current State = 2 Pods
```

The controller notices:

```text
Desired = 3
Current = 2
```

Difference:

```text
1 Pod missing
```

The controller takes corrective action.

Simplified flow:

```text
Deployment
    │
    ▼
Controller
    │
    ├── Desired = 3
    └── Current = 2
            │
            ▼
       Create another Pod
            │
            ▼
         Scheduler
            │
            ▼
       Select Worker Node
            │
            ▼
          Kubelet
            │
            ▼
       Container Runtime
            │
            ▼
           Pod
```

---

# 13. Important Difference Between Controller and Scheduler

Remember these three statements:

### Controller

> **"Do I have the state I am supposed to have?"**

### Scheduler

> **"Which Worker Node should this unscheduled Pod run on?"**

### Kubelet

> **"This Pod has been assigned to my node. I need to make it run."**

So:

```text
Controller
   │
   │ creates/reconciles Pod requirement
   ▼
Pod
   │
   ▼
Scheduler
   │
   │ selects node
   ▼
Kubelet
   │
   │ runs Pod
   ▼
Container
```

These are three different responsibilities.

---

# 14. Production Example — Pod Failure

Suppose a Deployment wants:

```text
Desired = 3
```

Initially:

```text
Pod-1
Pod-2
Pod-3
```

Then Pod-2 disappears:

```text
Pod-1
Pod-3
```

Now:

```text
Desired = 3
Current = 2
```

The controller detects the difference:

```text
Controller
    ↓
Need another Pod
    ↓
Create replacement Pod
    ↓
Scheduler
    ↓
Select suitable Node
    ↓
Kubelet
    ↓
Container Runtime
    ↓
Replacement Pod
```

This is the beginning of the **reconciliation** concept.

We will study reconciliation deeply later.

---

# 15. Kubernetes Brain — Current Level

For now, don't try to memorize every internal detail.

Think of Kubernetes as:

```text
                         USER
                           │
                           │ kubectl
                           ▼
                    ┌─────────────┐
                    │ API SERVER  │
                    └──────┬──────┘
                           │
                           ▼
                         etcd
                           ▲
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
        Controller                  Scheduler
             │                           │
             │                           │
             │ creates/reconciles        │ selects node
             │                           │
             └────────────┐  ┌───────────┘
                          ▼  ▼
                       API Server
                          │
                          ▼
                       Kubelet
                          │
                          ▼
                  Container Runtime
                          │
                          ▼
                         Pod
```

### Your current mental model should be:

```text
API Server
    ↓
Central communication point

etcd
    ↓
Persistent cluster state

Controller
    ↓
Maintains desired state

Scheduler
    ↓
Selects Worker Node

Kubelet
    ↓
Makes assigned Pods run

Container Runtime
    ↓
Actually creates/runs containers
```

We **will not go deeper into Controllers, Scheduler internals, Informers, Work Queues, Watches, ReplicaSet reconciliation, etc. yet**.

When we reach each concept in the course, we'll take it apart **pin-to-pin** and connect it back to this architecture.
