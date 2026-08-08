Got it. I found your **Day-2 running notes**. 

This day is much more important technically because you started **Kubernetes Architecture and the internal request flow**. So I’ll preserve your trainer's points, but make the notes work the way we discussed: **concept → why → internal flow → production scenario → troubleshooting → Kubernetes brain**.

# Kubernetes — Day 2 Running Notes

## Kubernetes Architecture & Internal Working

---

# 1. Kubernetes Architecture

Kubernetes has two major types of nodes:

```text
                    Kubernetes Cluster
                           │
             ┌─────────────┴─────────────┐
             │                           │
        Control Plane                Worker Nodes
        (Control Node)              (Data Plane)
             │                           │
       Controls cluster          Runs applications
```

### Control Plane

The Control Plane is responsible for **managing and controlling the Kubernetes cluster**.

It decides things such as:

* What should run?
* How many Pods should run?
* Which Worker Node should run a Pod?
* What is the current state?
* Is the desired state being maintained?

### Worker Node

Worker Nodes are where the **actual application workloads run**.

For example:

```text
Worker Node 1
   ├── Pod
   ├── Pod
   └── Pod

Worker Node 2
   ├── Pod
   ├── Pod
   └── Pod
```

Your trainer emphasized an important distinction:

> **Control Plane controls the cluster; Worker Nodes run the applications.** 

---

# 2. Cloud-Managed Kubernetes vs Self-Managed Kubernetes

There are different ways to operate Kubernetes.

### Cloud-Managed Kubernetes

Examples:

* AWS → EKS
* Azure → AKS
* Google Cloud → GKE

The cloud provider manages much of the Control Plane infrastructure.

### Self-Managed Kubernetes

One example discussed in class:

**KOps — Kubernetes Operations**

In this model, you are responsible for more of the Kubernetes infrastructure, including the Control Plane and Worker Nodes.

---

# 3. EKS is Still Kubernetes

This is extremely important.

Don't think:

```text
Kubernetes ≠ EKS
```

Instead think:

```text
Kubernetes
     +
AWS Cloud Integration
     =
Amazon EKS
```

Similarly:

```text
Kubernetes + Azure = AKS

Kubernetes + GCP   = GKE
```

The Kubernetes concepts remain largely the same.

The cloud provider adds integrations for things such as:

* Networking
* Storage
* Load Balancers
* IAM
* Compute
* Cloud-native services

Your trainer's key point was that EKS is **Kubernetes running with AWS-provided cloud infrastructure/integrations**, rather than a completely different technology. 

---

# 4. Why Learn EKS First?

If you understand Kubernetes concepts properly on EKS, a large portion of the Kubernetes concepts transfer to AKS and GKE.

Think:

```text
                 Kubernetes Concepts
                        │
        ┌───────────────┼───────────────┐
        │               │               │
       EKS             AKS             GKE
       AWS             Azure           GCP
```

The Kubernetes control-plane concepts remain common.

The major differences come from the **cloud integrations**.

---

# 5. Managed Control Plane — Important Mental Model

Suppose you create an EKS cluster.

You can see/manage:

```text
AWS Account
    │
    └── EKS Cluster
          │
          ├── Worker Node
          ├── Worker Node
          └── Worker Node
```

But you don't normally manage the underlying AWS-managed Control Plane EC2 instances directly.

Your trainer compared this concept to RDS:

> You use RDS, but you don't manage the underlying database server as if it were your own EC2 instance.

Similarly, with managed Kubernetes:

> You interact with the Control Plane, while AWS manages the underlying Control Plane infrastructure. 

---

# 6. Control Plane Components

The major Control Plane components introduced today are:

```text
                 CONTROL PLANE
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ▼               ▼                ▼
  API Server          etcd          Scheduler
       │
       ▼
Controller Manager
```

We will understand each component individually.

---

# 7. API Server — The Main Entry Point

The **API Server** is the main communication point of Kubernetes.

Whenever you run:

```bash
kubectl apply -f pod.yaml
```

your request goes to the API Server.

Think:

```text
Developer
   │
   │ kubectl apply
   ▼
API Server
```

The API Server receives the request and coordinates communication with other Kubernetes components.

Your trainer described the API Server as the "brain" or central component through which Kubernetes instructions pass. 

### Important interview point

If asked:

> **What is the API Server?**

Answer:

> The Kubernetes API Server is the central entry point to the Kubernetes control plane. It exposes the Kubernetes API, validates and processes requests, and acts as the communication hub between users and Kubernetes components.

---

# 8. Why Is API Server So Important?

Your trainer emphasized:

> Components communicate through the API Server.

Conceptually, think:

```text
             API SERVER
            /     |      \
           /      |       \
        etcd  Scheduler  Controllers
```

The important mental model is:

**The API Server is the central communication hub.**

Don't imagine:

```text
Scheduler ───────────► etcd

Controller ──────────► Scheduler
```

as the normal communication path.

Instead:

```text
Scheduler ──► API Server
Controller ─► API Server
API Server ─► etcd
API Server ─► Scheduler
API Server ─► Controller
```

This API-centric architecture is critical to understanding Kubernetes internals. 

---

# 9. etcd

Now we come to one of the most important components.

## What is etcd?

**etcd is the distributed key-value data store used by Kubernetes to persist cluster state.**

Your trainer compared it with a Terraform state file.

That is a useful beginner analogy:

```text
Terraform

Terraform Code
      │
      ▼
State File
```

Kubernetes:

```text
Kubernetes Objects / Cluster State
            │
            ▼
           etcd
```

But remember:

> **etcd is not literally a Terraform state file.**

It is a distributed database designed for Kubernetes' cluster state.

---

# 10. What Does etcd Store?

Conceptually, etcd stores Kubernetes state such as:

* Pod information
* Deployment information
* ReplicaSet information
* Node information
* Services
* Configurations
* Metadata
* Desired state

Your trainer specifically highlighted information such as which workload is running on which node and how many workloads exist. 

---

# 11. Why Does Kubernetes Need etcd?

This is where you should start thinking like Kubernetes.

Suppose you run:

```bash
kubectl apply -f pod.yaml
```

You request:

```text
Pod = nginx
```

Kubernetes needs a reliable source of truth.

It needs to know:

> "Does this object already exist?"

This is where the API Server and persistent cluster state become important.

---

# 12. Terraform Analogy

Suppose Terraform has:

```hcl
resource "aws_instance" "web" {
  ...
}
```

First execution:

```text
Terraform
   ↓
State doesn't contain resource
   ↓
Create EC2
```

Run again:

```text
Terraform
   ↓
State says resource exists
   ↓
No duplicate resource
```

Kubernetes similarly maintains object state so that repeated declarations don't simply mean "create another identical object."

Your trainer used this Terraform analogy to explain why Kubernetes needs persistent state. 

---

# 13. Scheduler

Next component:

**kube-scheduler**

Its main responsibility is:

> **Select an appropriate Worker Node for an unscheduled Pod.**

Suppose:

```text
Pod-1
Status = Pending
```

There are:

```text
Worker Node-1
Worker Node-2
Worker Node-3
```

The Scheduler evaluates the available nodes and selects a suitable one.

Simplified mental flow:

```text
Pending Pod
     │
     ▼
Scheduler
     │
     ├── Check available resources
     ├── Check constraints
     ├── Filter unsuitable Nodes
     └── Select suitable Node
             │
             ▼
        Node selected
```

Your trainer emphasized that the Scheduler's job is **node selection**. 

---

# 14. Very Important: Scheduler Does NOT Start Containers

This distinction is important for interviews.

Scheduler:

> **Which node should run this Pod?**

Kubelet:

> **Make sure the Pod actually runs on my node.**

Container runtime:

> **Actually create/start the container.**

So:

```text
Scheduler
   │
   │ selects Node
   ▼
Kubelet
   │
   │ instructs runtime
   ▼
Container Runtime
   │
   ▼
Container
```

This separation is extremely important.

---

# 15. Controller Manager

Another major Control Plane component:

**kube-controller-manager**

Its fundamental job is to run controllers that continuously work toward the desired state.

Think:

```text
Desired State = 6 Pods

Current State = 5 Pods

        ↓

Controller detects difference

        ↓

Take corrective action

        ↓

Create another Pod
```

Your trainer repeatedly emphasized:

> **Controller maintains Desired State = Current State.** 

---

# 16. The Most Important Kubernetes Concept: Desired State

Suppose YAML says:

```yaml
replicas: 6
```

That means:

```text
DESIRED STATE = 6 Pods
```

Initially:

```text
CURRENT STATE = 0
```

Controller sees:

```text
Desired = 6
Current = 0
```

Difference:

```text
6
```

So Kubernetes works toward:

```text
Current = 6
```

---

# 17. Production Failure Example

Suppose six Pods are running:

```text
Pod-1
Pod-2
Pod-3
Pod-4
Pod-5
Pod-6
```

Suddenly:

```text
Pod-4 crashes
```

Now:

```text
Desired = 6

Current = 5
```

Controller detects:

```text
6 != 5
```

Therefore:

```text
Create replacement Pod
```

Eventually:

```text
Desired = 6

Current = 6
```

Problem solved.

This is the **reconciliation loop** mindset you should start developing.

---

# 18. Kubernetes Brain

Whenever you see:

```text
replicas: 6
```

train your brain to think:

```text
                 Desired = 6
                     │
                     ▼
             Controller watches
                     │
                     ▼
             Current = 5?
                /         \
              YES          NO
               │            │
               ▼            ▼
        Take action      Do nothing
               │
               ▼
         Create Pod
```

This is how Kubernetes continuously thinks.

---

# 19. Worker Node Components

Now we move from the Control Plane to the Worker Node.

A simplified Worker Node:

```text
                 WORKER NODE
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       Kubelet    Container    kube-proxy
                    Runtime
                       │
                       ▼
                    Pods
```

Your trainer identified **kubelet**, **container runtime**, and **kube-proxy** as important Worker Node components. 

---

# 20. Kubelet

Kubelet is the main agent running on every Worker Node.

Think:

> **Kubelet is responsible for making sure the Pods assigned to its node are actually running and healthy.**

Simplified:

```text
API Server
    │
    ▼
Kubelet
    │
    ▼
Container Runtime
    │
    ▼
Container
```

Every Worker Node has a kubelet.

---

# 21. Container Runtime

Who actually creates the container?

**Container Runtime.**

This is an extremely important distinction.

```text
Scheduler
    ↓
Selects Node

Kubelet
    ↓
Requests container runtime

Container Runtime
    ↓
Creates/starts container
```

Kubernetes communicates with the runtime through the **Container Runtime Interface (CRI)**.

Your trainer emphasized that Kubernetes does not require the Docker Engine itself to create containers in modern Kubernetes architectures. 

Examples of container runtimes include:

* containerd
* CRI-O

---

# 22. Important Correction for Your Notes

Your trainer said:

> "K8s own container runtime."

For interview-quality notes, write it more accurately as:

> Kubernetes defines the **Container Runtime Interface (CRI)**. The cluster can use a CRI-compatible runtime such as containerd or CRI-O. Kubernetes itself is not the container runtime.

This distinction is important.

---

# 23. kube-proxy

Another Worker Node component:

**kube-proxy**

It participates in implementing Kubernetes Service networking.

Simplified:

```text
User
 │
 ▼
Service
 │
 ▼
kube-proxy / node networking rules
 │
 ▼
Pod
```

Your trainer introduced kube-proxy as the component involved in providing access to Pods through Services. 

Internally, kube-proxy can program networking rules such as iptables or IPVS depending on configuration.

---

# 24. Pod vs Container

This is another important Day-2 concept.

### Container

A container is the execution environment containing your application and its dependencies.

### Pod

A Pod is the **smallest deployable unit in Kubernetes**.

A Pod can contain one or more containers.

For the common single-container case:

```text
Pod
 │
 └── Application Container
```

Your trainer described the Pod as an abstraction layer around the container that helps Kubernetes manage and communicate with workloads. 

---

# 25. Docker vs Kubernetes Mental Model

Remember:

```text
Docker

Image
  ↓
Container
```

Kubernetes:

```text
Container Image
      ↓
     Pod
      ↓
Container
```

Do not say:

> Kubernetes creates containers directly.

Better:

> Kubernetes schedules Pods, and the kubelet instructs the container runtime to create the containers inside those Pods.

That's the **experienced-engineer answer**.

---

# 26. Complete Internal Flow

Now combine everything.

Suppose the developer executes:

```bash
kubectl apply -f pod.yaml
```

Think like this:

```text
Developer
    │
    │ kubectl apply
    ▼
API Server
    │
    ▼
Authenticate / Authorize / Validate
    │
    ▼
Persist object state
    │
    ▼
etcd
```

Now Kubernetes needs to get the Pod running.

```text
API Server
    │
    ▼
Scheduler
    │
    │ Select Worker Node
    ▼
API Server records scheduling decision
    │
    ▼
Kubelet on selected Worker Node
    │
    ▼
Container Runtime
    │
    ▼
Container
    │
    ▼
Pod Running
```

Meanwhile:

```text
Controller
    │
    ▼
Continuously watches desired vs current state
    │
    ▼
Reconciles differences
```

And:

```text
kube-proxy
    │
    ▼
Helps implement Service networking
```

---

# 27. The Kubernetes Engine — Mental Visualization

This is the mental model I want you to develop.

```text
                       USER
                        │
                        │ kubectl
                        ▼
                 ┌─────────────┐
                 │ API SERVER  │
                 └──────┬──────┘
                        │
             ┌──────────┼──────────┐
             │          │          │
             ▼          ▼          ▼
           etcd    Scheduler   Controllers
             ▲          │          │
             │          │          │
             └──────────┼──────────┘
                        │
                        ▼
                 Selected Node
                        │
                        ▼
                    Kubelet
                        │
                        ▼
              Container Runtime
                        │
                        ▼
                     Pod
                        │
                        ▼
                  Application
```

**The key idea:**

Kubernetes isn't a script that runs once.

It is a **continuous control loop**.

---

# 28. Production Troubleshooting — Pod Pending

Now let's start thinking like an engineer.

Suppose:

```bash
kubectl get pods
```

shows:

```text
NAME       STATUS
nginx      Pending
```

Don't immediately recreate the Pod.

Ask:

### Question 1

Has the Scheduler assigned a Node?

```bash
kubectl describe pod nginx
```

Look at:

```text
Node:
```

If no Node is assigned, investigate scheduling.

Possible reasons:

```text
Insufficient CPU
Insufficient Memory
Taints
Node affinity
Node selector
Resource constraints
No available nodes
```

---

# 29. Production Troubleshooting — Pod Assigned but Not Running

Suppose:

```text
Node: worker-node-1

Status: ContainerCreating
```

Now the Scheduler already did its job.

Don't troubleshoot Scheduler.

Move down the flow:

```text
Scheduler
   ✓

Kubelet
   ↓

Container Runtime
   ↓

Image Pull
   ↓

Container Creation
```

Investigate:

* Image pull
* Registry authentication
* Volume mount
* CNI/network setup
* Runtime errors
* Secrets/configuration

This is how you isolate problems instead of randomly running commands.

---

# 30. Production Troubleshooting — ImagePullBackOff

Suppose:

```text
STATUS

ImagePullBackOff
```

Think:

```text
Pod scheduled?
       │
       ▼
YES

Kubelet trying to pull image
       │
       ▼
Registry
       │
       ├── Wrong image?
       ├── Wrong tag?
       ├── Private registry?
       ├── Authentication?
       ├── Image doesn't exist?
       └── Network/DNS problem?
```

Then inspect:

```bash
kubectl describe pod <pod-name>
```

Look at the **Events** section.

---

# 31. Production Troubleshooting — Pod Suddenly Deleted

Suppose:

```text
Desired = 6
Current = 5
```

Don't manually create a Pod immediately.

Ask:

> Who owns this Pod?

Maybe:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

The controller should reconcile it.

Your job is to determine **why the Pod disappeared** and whether Kubernetes is already creating its replacement.

---

# 32. Cluster

A Kubernetes Cluster is the overall environment consisting of the Control Plane and Worker Nodes.

```text
                 KUBERNETES CLUSTER
                        │
          ┌─────────────┴─────────────┐
          │                           │
     Control Plane               Worker Nodes
          │                    ┌──────┴──────┐
          │                    │             │
       API Server            Node-1        Node-2
       etcd                    │             │
       Scheduler             Pods          Pods
       Controllers
```

Your trainer summarized a cluster as the combination of Control Plane and Worker Nodes. 

---

# 33. Important Practical Point — Cluster Creation

Creating an EKS cluster is generally a **one-time infrastructure activity** for a given environment.

Once the cluster exists, you don't recreate it every time you deploy an application.

Instead:

```text
Create Cluster
      │
      ▼
Cluster Ready
      │
      ├── Deploy App-1
      ├── Deploy App-2
      ├── Deploy App-3
      └── Deploy App-4
```

You may create separate clusters for environments such as:

```text
Development
Testing
Staging
Production
```

depending on the organization's architecture.

---

# 34. Your First Practical Commands

Your trainer introduced Killercoda for practice.

Basic flow:

```bash
vi pod.yml
```

Create the YAML.

Then:

```bash
kubectl apply -f pod.yml
```

Check:

```bash
kubectl get pods
```

Run the same command again:

```bash
kubectl apply -f pod.yml
```

You may see:

```text
unchanged
```

This is a very useful demonstration of Kubernetes' declarative model.

Your trainer specifically used this example to demonstrate that applying the same configuration repeatedly doesn't mean creating duplicate objects. 

---

# 🧠 Day-2 Kubernetes Brain

Don't memorize this as a diagram.

**Visualize it.**

Imagine you run:

```bash
kubectl apply -f pod.yaml
```

Your brain should automatically see:

```text
1. User sends request
       ↓
2. API Server receives request
       ↓
3. API Server validates/processes it
       ↓
4. Kubernetes persists desired object state
       ↓
5. Scheduler finds unscheduled Pod
       ↓
6. Scheduler selects Worker Node
       ↓
7. Kubelet on that node notices assignment
       ↓
8. Kubelet asks container runtime
       ↓
9. Runtime pulls image if needed
       ↓
10. Runtime creates container
       ↓
11. Pod becomes Running
       ↓
12. Controllers keep watching
       ↓
13. If state changes, reconciliation begins
```

And the most important loop:

```text
       DESIRED STATE
             │
             ▼
        Controller
             │
             ▼
       CURRENT STATE
             │
        ┌────┴────┐
        │         │
     Same?      Different?
        │         │
        ▼         ▼
     Nothing    Reconcile
                 │
                 ▼
           Desired = Current
```

**This is the mindset I want you to develop.**

Don't think:

> "Scheduler is one component, kubelet is another component."

Think:

> **"A Pod has to go from desired state to actual running state. Which component is responsible for each step?"**

That question will make Kubernetes architecture much easier.

---

## Day-2 Key Interview Questions

1. What are the major components of Kubernetes architecture?
2. What is the difference between Control Plane and Worker Node?
3. What is the role of API Server?
4. Why does Kubernetes need etcd?
5. What is the role of Scheduler?
6. Does Scheduler create containers?
7. What does Controller Manager do?
8. What is desired state vs current state?
9. What is kubelet?
10. Who actually creates the container?
11. What is CRI?
12. What is the difference between a Pod and a Container?
13. What is kube-proxy?
14. What happens internally when `kubectl apply -f pod.yaml` is executed?
15. Why is the Kubernetes API Server so important?
16. Why does a Pod remain Pending?
17. How would you troubleshoot `ImagePullBackOff`?
18. What is the difference between EKS and Kubernetes?
19. What is the difference between managed and self-managed Kubernetes?
20. What is KOps?

Your Day-2 notes are now centered on **architecture and internal behavior**, rather than just definitions, so when we move into **Pods, YAML, ReplicaSets, Deployments, Services, Scheduler, Controllers, etc.**, we can keep building on this mental model. 
