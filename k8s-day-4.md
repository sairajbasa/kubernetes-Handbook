**EKS → YAML → ReplicaSet → Deployment → Pending Pods → Cluster Autoscaler → IAM → Services → HPA**

Your notes are quite detailed and, importantly, you are focusing on the **interview/debugging side**, not just definitions. 

### Your Day 4 mental flow

```text
Create EKS Cluster
       ↓
kubectl client machine
       ↓
kubectl + kubeconfig
       ↓
Apply YAML
       ↓
Pod
       ↓
ReplicaSet
       ↓
Deployment
       ↓
replicas: 3
       ↓
Need more Pods
       ↓
replicas: 35 / 70 / 100
       ↓
Node resources insufficient
       ↓
Pods → Pending
       ↓
kube-scheduler cannot find suitable Node
       ↓
Cluster Autoscaler
       ↓
ASG / EKS Node Group
       ↓
New EC2 Node
       ↓
Pods get scheduled
```

And then your next concept is:

```text
                    TRAFFIC
                       ↓
                 Pod count needs
                  to increase
                       ↓
                    HPA
                       ↓
              More Pods required
                       ↓
          Cluster Autoscaler checks
             node capacity
                       ↓
              More Nodes required
                       ↓
                    ASG
                       ↓
                  EC2 Nodes
```

### The most important interview chain from Day 4

You should be able to explain this **without looking at notes**:

> **Pod Pending → Why? → `kubectl describe pod` → scheduler cannot place it → insufficient node resources → Cluster Autoscaler detects unschedulable pods → communicates with AWS Auto Scaling/EKS node group → desired node count increases → new EC2 node joins cluster → scheduler schedules pending pods onto the new node.**

That is much more valuable than memorizing installation commands. Your notes explicitly emphasize this interview-oriented approach. 

### One important distinction to keep very clear

You have **two different scaling layers**:

| Layer        | What increases?        | Responsible component          |
| ------------ | ---------------------- | ------------------------------ |
| Pod scaling  | Number of Pods         | HPA                            |
| Node scaling | Number of worker Nodes | Cluster Autoscaler / Karpenter |
| EC2 capacity | EC2 instances          | AWS ASG / EKS Node Group       |

So tomorrow's concept naturally continues from today's:

```text
HPA
 ↓
Pod count increases
 ↓
Node capacity becomes insufficient
 ↓
Pod → Pending
 ↓
Cluster Autoscaler
 ↓
Node count increases
 ↓
Pending Pods get scheduled
```

And your **Services** section starts a completely different but equally important flow:

```text
User
 ↓
Service
 ↓
Pod
```

with:

```text
ClusterIP   → internal communication
NodePort    → external/dev-test access
LoadBalancer → external production access
Headless    → direct Pod discovery / stateful workloads
```

Overall, **Day 4 is a strong day** because you're moving from *"what is Kubernetes?"* toward *"what happens inside Kubernetes when something goes wrong?"* — exactly the direction you want for interview preparation.

---
Absolutely. Below are your **Day 4 Kubernetes/EKS running notes**, organized for revision while keeping the concepts and terminology from your original notes. I have also incorporated the IAM clarification we just discussed. 

# DAY 4 — EKS, YAML, Deployments, Autoscaling & Services

## 1. Create / Connect to EKS Cluster

Immediately set up the EKS cluster.

For the client machine, use a dedicated EC2 instance and install:

```text
kubectl
eksctl
```

Attach an IAM role to this client EC2.

For learning/demo purposes, `AdministratorAccess` was used initially.

```text
Dedicated Client EC2
        │
        ├── kubectl
        ├── eksctl
        ├── IAM Role
        └── ~/.kube/config
                 │
                 ↓
              EKS API
                 │
                 ↓
             EKS Cluster
```

**Important interview point:**

> In interviews, installation commands are generally less important than understanding how Kubernetes components work and how to troubleshoot them.

---

# 2. Kubernetes YAML

Example Pod:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod
  labels:
    env: dev

spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

### Important YAML sections

```text
apiVersion
    ↓
Which Kubernetes API version

kind
    ↓
What Kubernetes object you are creating

metadata
    ↓
Name + labels + other metadata

spec
    ↓
Desired configuration
```

### Labels

Labels are simply:

```text
key: value
```

Example:

```yaml
env: dev
```

---

# 3. ReplicaSet

ReplicaSet is used to maintain the required number of Pods.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
```

Under `spec`:

```text
replicas
selector
template
```

### ReplicaSet structure

```text
ReplicaSet
    │
    ├── replicas
    ├── selector
    └── template
          │
          └── Pod details
```

### Important selector rule

The labels under:

```yaml
spec:
  selector:
    matchLabels:
```

must match the corresponding labels under:

```yaml
spec:
  template:
    metadata:
      labels:
```

Example:

```yaml
selector:
  matchLabels:
    app: nginx

template:
  metadata:
    labels:
      app: nginx
```

`template.metadata.labels` can contain additional labels, but every label specified in `matchLabels` must be present and match.

---

# 4. ReplicaSet Pod Names

When ReplicaSet creates Pods, you don't normally specify a Pod name inside the template.

If:

```yaml
metadata:
  name: my-replicaset

spec:
  replicas: 3
```

Pods can look like:

```text
my-replicaset-abc12
my-replicaset-x7k9p
my-replicaset-m4n8q
```

Pattern:

```text
ReplicaSet name + generated suffix
```

The generated suffix is handled by Kubernetes.

---

# 5. ReplicaSet Controller

ReplicaSet provides:

```text
Self-healing
+
Maintaining desired replica count
```

Example:

```text
Desired = 3 Pods
       ↓
3 Pods running
       ↓
One Pod deleted
       ↓
ReplicaSet detects difference
       ↓
Creates replacement Pod
       ↓
3 Pods again
```

The general Kubernetes controller principle:

```text
Desired State
      ↕
Controller
      ↕
Current State
```

---

# 6. Deployment

Deployment provides additional functionality over ReplicaSet.

When you create:

```yaml
kind: Deployment
```

Kubernetes internally creates/manages a ReplicaSet.

```text
Deployment
    │
    ↓
ReplicaSet
    │
    ↓
Pods
```

### Deployment provides

```text
ReplicaSet
+
Rollouts
+
Rolling Updates
+
Rollbacks
```

### Memory trick

```text
Rollout
→ Move to a new version

Rolling Update
→ Move to the new version gradually

Rollback
→ Go back to the previous version
```

Deployment's default update strategy is `RollingUpdate`, so the application can be updated gradually rather than deleting everything at once. 

---

# 7. Static Pod Scaling / Replicas

Suppose:

```yaml
spec:
  replicas: 3
```

Kubernetes maintains:

```text
3 Pods
```

Even if:

```text
Traffic = 0
```

you still have:

```text
3 Pods
```

This is **static scaling**.

If you change:

```yaml
replicas: 35
```

Kubernetes tries to create:

```text
35 Pods
```

---

# 8. Pod Hardware Comes From the Node

Pods consume resources from the worker node.

```text
Worker Node
    │
    ├── Pod
    ├── Pod
    ├── Pod
    └── Pod
```

The hardware/resources ultimately come from the node.

If there isn't enough available capacity:

```text
Pod
 ↓
Pending
```

---

# 9. How to Troubleshoot a Pending Pod

The important command from your class:

```bash
kubectl describe pod <pod-name>
```

The basic troubleshooting flow:

```text
Pod Pending
    ↓
kubectl describe pod <pod-name>
    ↓
Check Events
    ↓
Why couldn't scheduler place the Pod?
    ↓
Often insufficient node resources
```

### Important concept

If no suitable node is available:

```text
Pod
 ↓
Pending
```

The kube-scheduler has not successfully assigned the Pod to a node.

Until a Pod is scheduled to a node, it doesn't make sense to expect the kubelet on some node to start that Pod.

---

# 10. Increasing Node Capacity

Suppose:

```text
2 worker nodes
```

and you need more capacity.

You cannot simply expect changing an individual EC2 instance to permanently change an EKS managed node group's capacity.

Your notes emphasize the **Launch Template / Auto Scaling Group** relationship.

Conceptually:

```text
EKS Node Group
      ↓
Auto Scaling Group
      ↓
EC2 Instances
```

If you want more nodes, the node group's scaling configuration needs to permit additional nodes.

Example:

```text
Desired = 2
Min = 2
Max = 4
```

Then the node group is allowed to scale up to 4 nodes.

---

# 11. ASG Alone Doesn't Know When Kubernetes Needs Nodes

This is a very important Day 4 concept.

Suppose:

```text
ASG:
Desired = 2
Min = 2
Max = 4
```

You create 35 Pods.

Some Pods become:

```text
Pending
```

The ASG does **not automatically understand**:

> "Kubernetes has Pending Pods, therefore I should create another node."

You need a Kubernetes-to-AWS scaling integration.

That is where:

# Cluster Autoscaler

comes in.

---

# 12. Cluster Autoscaler

Cluster Autoscaler monitors the cluster for unschedulable Pods.

Basic flow:

```text
Deployment
    ↓
More Pods
    ↓
Node capacity insufficient
    ↓
Pods → Pending
    ↓
Cluster Autoscaler detects
unschedulable Pods
    ↓
AWS Auto Scaling / Node Group
    ↓
Increase node count
    ↓
New EC2 Node
    ↓
Node joins EKS
    ↓
Scheduler schedules Pods
```

### Simple definition

> **Cluster Autoscaler automatically increases or decreases the number of worker nodes based on the need for schedulable capacity.**

Your notes also mention **Karpenter** as another node-scaling solution. 

---

# 13. Static Pod Scaling vs Dynamic Pod Scaling

### Static

```yaml
replicas: 3
```

Regardless of traffic:

```text
Low traffic → 3 Pods
High traffic → 3 Pods
```

### Dynamic

With Pod Autoscaling:

```text
Low traffic
    ↓
Fewer Pods

High traffic
    ↓
More Pods
```

Your next topic after Cluster Autoscaler is therefore:

```text
Pod Autoscaler / HPA
```

---

# 14. Why You Reduced Replicas Before Installing Integrations

Your notes reduced:

```yaml
replicas: 35
```

to:

```yaml
replicas: 2
```

The reason was to free resources.

Because Kubernetes integrations/add-ons themselves can run as workloads/Pods.

If your existing application Pods consume all node capacity:

```text
Existing Pods
     ↓
Node capacity full
     ↓
New integration Pod
     ↓
Pending
```

So you first create available capacity before installing the integration.

Your notes summarize the practical rule as:

> If you deploy an application as a Kubernetes workload, it runs inside a Pod. If you install something directly on the node, it does not automatically become a Pod. 

---

# 15. kube-system Namespace

Kubernetes has namespaces.

Example:

```text
EKS Cluster
│
├── kube-system
│     ├── CoreDNS
│     ├── kube-proxy
│     ├── AWS-related add-ons
│     └── cluster-autoscaler
│
├── default
│     └── Application Pods
│
└── other namespaces
```

Check:

```bash
kubectl get pods -n kube-system
```

Filter Cluster Autoscaler:

```bash
kubectl get pods -n kube-system -l app=cluster-autoscaler
```

Here:

```text
-n kube-system
    ↓
Select namespace

-l app=cluster-autoscaler
    ↓
Select Pods having this label
```

**Important:**

```text
kube-system = namespace
NOT a node
```

The Cluster Autoscaler Pod runs on a worker/data-plane node while belonging to the `kube-system` namespace. 

---

# 16. IAM — Very Important Day 4 Concept

This was one of the most important parts of your class.

There are different IAM contexts.

## A. Client EC2 IAM Role

Your dedicated client EC2 has an IAM role.

```text
Client EC2
    │
    ├── kubectl
    ├── eksctl
    └── IAM Role
```

This allows the machine/user process to interact with AWS/EKS according to the permissions assigned.

---

## B. EKS Node Group IAM Role

When the EKS node group was created, an IAM role already existed for the worker nodes.

```text
EKS Node Group
      │
      ↓
IAM Role
      │
      ↓
Worker Nodes
```

Your sir used this **existing node-group IAM role** and added the required permissions for the Cluster Autoscaler demonstration rather than creating a separate role.

Your notes explicitly mention checking:

```text
EKS
 ↓
Compute
 ↓
Node Group
 ↓
Node Group IAM Role ARN
```



---

# 17. IAM Permissions Used for Cluster Autoscaler

Your notes list permissions such as:

```text
autoscaling:DescribeAutoScalingGroups
autoscaling:DescribeAutoScalingInstances
autoscaling:DescribeLaunchConfigurations
autoscaling:DescribeScalingActivities
autoscaling:SetDesiredCapacity
autoscaling:TerminateInstanceInAutoScalingGroup
eks:DescribeNodegroup
```

These permissions allow the relevant identity to interact with AWS Auto Scaling/EKS APIs. 

### Your class's mental model

```text
Worker Node
    │
    │ IAM Role
    ↓
Node has AWS permissions
    │
    ↓
Cluster Autoscaler Pod
    │
    ↓
AWS Auto Scaling APIs
    │
    ↓
Create / remove nodes
```

### Important wording

Don't memorize this as:

> "Every Pod automatically gets the node's IAM permissions."

For **your class's setup**, remember:

> **The Cluster Autoscaler is running on a worker node whose IAM role has been given the required AWS permissions, so the autoscaler can use that AWS access in this setup.**

Your notes also mention that more specific Pod-level identity mechanisms such as Service Accounts/OIDC exist. 

---

# 18. Cluster Autoscaler YAML

Your notes use the Kubernetes Autoscaler example YAML.

Conceptually:

```bash
kubectl apply -f <autoscaler-yaml>
```

The YAML contains multiple Kubernetes objects/resources required for the integration.

You can apply YAML from a local machine:

```bash
kubectl apply -f deployment.yml
```

Your notes also show the idea of applying a YAML hosted in GitHub.

---

# 19. Editing an Existing Deployment

Your sir used:

```bash
kubectl edit deployment cluster-autoscaler -n kube-system
```

Important distinction:

### `kubectl edit`

```bash
kubectl edit deployment my-app -n default
```

You **do not provide `.yml` or `.yaml`**.

It directly opens the existing Kubernetes resource for editing.

### `kubectl apply`

```bash
kubectl apply -f deployment.yml
```

Here you **do provide the YAML file** because Kubernetes reads the desired configuration from the file.

### Memory trick

```text
kubectl edit
    ↓
Edit existing resource

kubectl apply -f
    ↓
Apply configuration from YAML
```

---

# 20. Cluster Name in Cluster Autoscaler

The Cluster Autoscaler configuration needs information identifying the cluster/node-group it should work with.

Your sir edited the Cluster Autoscaler deployment and supplied the cluster information.

Conceptually:

```text
Cluster Autoscaler
       ↓
Knows which cluster/node group
it should manage
       ↓
AWS Auto Scaling
```

---

# 21. Node Group Scaling Configuration

Your notes use:

```bash
aws eks update-nodegroup-config \
  --cluster-name <your-cluster-name> \
  --nodegroup-name <your-node-group-name> \
  --scaling-config minSize=2,maxSize=6,desiredSize=2
```

This means:

```text
minSize     = 2
desiredSize = 2
maxSize     = 6
```

### Why is `maxSize` important?

If:

```text
min = 2
desired = 2
max = 2
```

the node group cannot scale above 2 nodes.

If:

```text
min = 2
desired = 2
max = 6
```

the autoscaler has room to increase the node count up to 6.

---

# 22. Test Cluster Autoscaler

Suppose:

```yaml
replicas: 35
```

Apply:

```bash
kubectl apply -f deploy.yml
```

Watch nodes:

```bash
kubectl get nodes --watch
```

Check Pods:

```bash
kubectl get pods
```

Expected flow:

```text
35 Pods requested
       ↓
Existing nodes insufficient
       ↓
Some Pods Pending
       ↓
Cluster Autoscaler detects them
       ↓
Node count increases
       ↓
New node joins cluster
       ↓
Pending Pods scheduled
       ↓
Pods become Running
```

---

# 23. Scaling to 70 / 100 Pods

Your sir then tested larger replica counts.

Example:

```text
35 → 70 → 100
```

As demand for node capacity increases:

```text
More Pods
    ↓
More resources required
    ↓
Pending Pods
    ↓
Cluster Autoscaler
    ↓
More Nodes
```

If:

```text
maxSize = 6
```

then the node group can scale only up to 6 nodes.

If Pods remain Pending after reaching 6 nodes, increase the permitted maximum if appropriate.

Troubleshoot with:

```bash
kubectl get nodes
```

and:

```bash
kubectl get pods
```

For a Pending Pod:

```bash
kubectl describe pod <pending-pod-name>
```

---

# 24. What Happens If Nodes Are Deleted?

Interview question:

> What happens if 3 worker nodes suddenly disappear?

Conceptually:

```text
3 Nodes disappear
       ↓
Pods on those nodes are affected
       ↓
Kubernetes detects node failure
       ↓
Healthy nodes may receive replacement workloads
       ↓
If insufficient capacity
       ↓
Some Pods remain Pending
       ↓
Cluster Autoscaler can create replacement nodes
       ↓
New nodes join cluster
       ↓
Pods can be scheduled
```

The exact behavior depends on available capacity and the workload/controller involved.

---

# 25. Scale Down

After testing:

```yaml
replicas: 100
```

your sir reduced it to:

```yaml
replicas: 3
```

Then:

```bash
kubectl apply -f deploy.yml
```

Kubernetes removes the excess Pods until the desired state becomes:

```text
3 Pods
```

Node count can subsequently decrease as capacity becomes unnecessary, subject to the autoscaler's configuration and behavior.

---

# 26. Pod Autoscaler — Next Concept

At the end of Day 4, the next problem is identified.

Static:

```yaml
replicas: 3
```

means:

```text
Traffic = 0
    ↓
3 Pods

Traffic = High
    ↓
Still 3 Pods
```

You want:

```text
Traffic increases
       ↓
Pod count increases
       ↓
Node capacity becomes insufficient
       ↓
Cluster Autoscaler
       ↓
Node count increases
```

And when traffic decreases:

```text
Traffic decreases
       ↓
Pod count decreases
       ↓
Node capacity becomes unnecessary
       ↓
Node count can decrease
```

So the two autoscaling layers work together:

```text
             Traffic
                ↓
              HPA
                ↓
          Pod count changes
                ↓
        Cluster Autoscaler
                ↓
          Node count changes
                ↓
               ASG
                ↓
            EC2 Nodes
```

---

# 27. Kubernetes Services

Now comes another major Day 4 topic.

You have Pods running inside the cluster.

Question:

> **How do users access the application?**

In Docker, you might use:

```text
-p <hostPort>:<containerPort>
```

In Kubernetes, you generally use a **Service**.

```text
User
 ↓
Service
 ↓
Pod
```

A Service provides stable communication to Pods.

---

# 28. Service Types

Your notes focus on four types:

```text
1. NodePort
2. LoadBalancer
3. ClusterIP
4. Headless Service
```

---

## ClusterIP

Used for internal communication.

Example:

```text
Frontend Pod
     ↓
ClusterIP Service
     ↓
Backend Pod
```

It is not intended as the normal external entry point for users.

---

## NodePort

Conceptually:

```text
External User
      ↓
Node IP + NodePort
      ↓
Service
      ↓
Pod
```

Your class notes emphasize that EKS worker nodes commonly run in private subnets and don't normally have public IPs, so NodePort isn't the preferred production external-access approach in that architecture.

Useful primarily for learning/dev/testing in your class context.

---

## LoadBalancer

Used for external communication.

Conceptually:

```text
Internet
   ↓
AWS Load Balancer
   ↓
Kubernetes Service
   ↓
Pods
```

Your notes identify LoadBalancer as the recommended external communication approach in this AWS context.

---

## Headless Service

Your notes identify Headless Services as useful for stateful applications such as databases.

Conceptually:

```text
Service
  ↓
Individual Pod discovery
```

This will be covered later in more detail.

---

# 29. Service Communication Summary

Keep this table in your mind:

| Service          | Main purpose                                        |
| ---------------- | --------------------------------------------------- |
| **ClusterIP**    | Internal communication                              |
| **NodePort**     | External/dev-test access through node port          |
| **LoadBalancer** | External access using cloud load balancer           |
| **Headless**     | Direct Pod discovery, useful for stateful workloads |

---

# 30. Monitoring Commands

Your notes mention:

```bash
kubectl top nodes
```

Shows resource usage of nodes.

```bash
kubectl top pods
```

Shows resource usage of Pods.

These are useful when investigating resource consumption and autoscaling behavior.

---

# ⭐ Day 4 Interview Questions to Master

Don't concentrate only on:

> "What is a Pod?"

Instead, practice questions like:

### 1. Pod is Pending. How do you troubleshoot?

```text
kubectl get pods
        ↓
kubectl describe pod <pod-name>
        ↓
Check Events
        ↓
Identify scheduling/resource problem
```

### 2. You increased replicas from 3 to 30. Why are some Pods Pending?

Because the available worker-node resources may not be sufficient to schedule all requested Pods.

### 3. Why didn't ASG automatically create another node?

Because ASG by itself doesn't understand Kubernetes Pod scheduling requirements. A Kubernetes-aware node autoscaling mechanism such as Cluster Autoscaler is needed.

### 4. What does Cluster Autoscaler do?

```text
Pending/unschedulable Pods
        ↓
Cluster Autoscaler
        ↓
Increase node capacity
        ↓
New worker node
```

### 5. Why do you need IAM permissions?

The autoscaling component needs AWS API permissions to inspect and modify the relevant AWS Auto Scaling/EKS resources.

### 6. Which IAM role did your sir modify?

**The existing EKS node-group IAM role**, adding the required autoscaling/EKS permissions for the demonstration. 

### 7. Where does Cluster Autoscaler run?

As a **Pod**, commonly in:

```text
kube-system
```

and that Pod runs on a worker/data-plane node in this EKS architecture.

### 8. Is `kube-system` a node?

**No.**

```text
kube-system = Namespace
```

### 9. What is the difference between `kubectl edit` and `kubectl apply -f`?

```text
kubectl edit
→ directly edits existing Kubernetes resource

kubectl apply -f
→ applies configuration from YAML
```

### 10. What is the difference between HPA and Cluster Autoscaler?

```text
HPA
→ scales Pods

Cluster Autoscaler
→ scales Nodes
```

### 11. What happens when traffic increases?

The eventual desired architecture is:

```text
Traffic ↑
   ↓
HPA
   ↓
Pods ↑
   ↓
Node capacity insufficient
   ↓
Cluster Autoscaler
   ↓
Nodes ↑
```

---

## 🔥 The single Day 4 flow you should remember

```text
                         EKS CLUSTER
                              │
                ┌─────────────┴─────────────┐
                │                           │
          Control Plane                Worker Nodes
                                            │
                              ┌─────────────┼─────────────┐
                              │             │             │
                            Pod           Pod      Cluster Autoscaler
                                                          │
                                                          │
                                                          ↓
                                                   AWS IAM permissions
                                                          │
                                                          ↓
                                                   AWS Auto Scaling
                                                          │
                                                          ↓
                                                   New EC2 Node
                                                          │
                                                          ↓
                                                   More capacity
                                                          │
                                                          ↓
                                                   Pending Pods
                                                     → Running


YAML
 ↓
Deployment
 ↓
ReplicaSet
 ↓
Pods
 ↓
Insufficient resources
 ↓
Pending
 ↓
Cluster Autoscaler
 ↓
More Nodes
 ↓
Pods Running
 ↓
Service
 ↓
Users access application
```

And then **Day 5 naturally continues from here**:

```text
Traffic
   ↓
HPA / Pod Autoscaler
   ↓
Pod count changes
   ↓
Cluster Autoscaler
   ↓
Node count changes
   ↓
Services
   ↓
Application access
```

This is the core of what your **Day 4 running notes are teaching you**. 
---
``` Question
i.e. we attached the permissions to node group so pod is running as process inside nodes so they have the permission to AWS Services if we provided
```
Yes — **that is the mental model your sir is using in this example.** Your notes say the Cluster Autoscaler runs as a Pod on the worker nodes, and the node-group role was given the required AWS permissions. 

Think of it like this:

```text
EKS Worker Node
      │
      │ IAM Role attached to Node
      │
      ↓
  ┌──────────────────────────────┐
  │          Node                │
  │                              │
  │  Pod 1                       │
  │  Pod 2                       │
  │  Cluster Autoscaler Pod      │
  │         │                    │
  │         ↓                    │
  │    AWS API request           │
  └─────────┼────────────────────┘
            ↓
       AWS Services
            │
            ↓
     IAM permissions
```

So your statement can be remembered as:

> **The IAM role is attached to the worker node. Since the Cluster Autoscaler Pod is running on that worker node, the Pod can use the AWS permissions available through that node's IAM role in this setup.**

For your specific example:

```text
Cluster Autoscaler Pod
        ↓
Running on Worker Node
        ↓
Worker Node has IAM Role
        ↓
IAM Role has Auto Scaling permissions
        ↓
Cluster Autoscaler can call AWS Auto Scaling APIs
        ↓
Increase Desired Capacity
        ↓
New EC2 Node
```

### One small wording correction

Don't think:

> "Every Pod automatically gets the node's IAM permissions."

A better understanding is:

> **In this setup, the Pod's AWS access is being obtained through the node's IAM identity/credentials.**

Modern EKS can instead give individual Pods their own AWS identity using mechanisms such as **EKS Pod Identity or IRSA**, which avoids giving every workload on the node the same AWS permissions.

But **for your Day 4 class example**, keep the simple flow:

**Node Group IAM Role → required permissions → Cluster Autoscaler Pod → AWS Auto Scaling.** 

