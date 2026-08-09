Yes. This is your **Day-3 running notes**. I’ll preserve the trainer's flow, but organize it so the **“what → why → how → internal flow → real-time problem”** is easy to revise.

I’ll also clearly mark a few places where the spoken notes are simplified so you don't memorize an inaccurate Kubernetes internal flow.

# DAY-3 — AWS EKS Cluster, kubectl, Pods, ReplicaSets & Deployments

---

# 1. Kubernetes Cluster — Basic Understanding

A Kubernetes cluster is broadly made up of:

```text
Kubernetes Cluster
│
├── Control Plane
│
└── Worker Nodes
```

### Control Plane

The Control Plane manages the Kubernetes cluster.

It contains components such as:

```text
API Server
Scheduler
Controller Manager
etcd
```

### Worker Nodes

Worker Nodes are where application workloads actually run.

```text
Worker Node
│
├── Kubelet
├── Container Runtime
├── Networking components
└── Pods
```

---

# 2. How Do We Connect to a Kubernetes Cluster?

Suppose an EKS cluster is already running in AWS.

We need a client from which we can execute Kubernetes commands.

There are two common approaches:

### Option 1 — Local Laptop

```text
Your Laptop
    │
    │ kubectl
    ▼
AWS EKS Cluster
```

### Option 2 — Dedicated EC2 Client

Create a dedicated EC2 instance and use it as the Kubernetes administration/client machine.

```text
Dedicated EC2
     │
     │ kubectl / eksctl
     ▼
AWS EKS Cluster
```

For AWS authentication, the client needs appropriate AWS credentials.

For an EC2-based client, an **IAM Role can be attached to the EC2 instance** instead of configuring long-lived access keys.

---

# 3. Why Use a Dedicated EC2 Client?

For learning/lab purposes, a dedicated EC2 instance can be useful because:

```text
EC2 Client
│
├── kubectl
├── eksctl
├── AWS CLI
└── Terraform
```

can all be installed in one place.

The EC2 instance becomes the administration machine from which we interact with AWS and Kubernetes.

### Important

For production, do not automatically assume that a publicly accessible EC2 instance with broad Administrator permissions is the correct architecture.

The training setup may use:

```text
AdministratorAccess
```

for simplicity during learning.

In real environments, follow **least privilege** and secure access patterns.

---

# 4. AWS EKS

When Kubernetes is provided as a managed AWS service, we use:

**Amazon EKS — Elastic Kubernetes Service**

Conceptually:

```text
AWS
│
└── EKS Cluster
    │
    ├── AWS-managed Control Plane
    │
    └── Worker Nodes
```

One of the biggest advantages of EKS is that AWS manages the Kubernetes Control Plane for us.

Therefore, when we look at the EC2 console, we normally see our Worker Nodes, but we don't manage the EKS Control Plane EC2 instances ourselves.

---

# 5. Methods to Create an EKS Cluster

There are three common approaches discussed in the training:

```text
1. Manual / AWS Console
2. CLI
3. Terraform
```

## Manual

Useful for understanding the AWS components, but:

* More steps
* More manual configuration
* More difficult to reproduce
* More difficult to clean up consistently

## CLI

Useful for quick testing and learning.

For example, `eksctl` can create an EKS cluster with a command.

## Terraform

Preferred for production-style Infrastructure as Code because infrastructure becomes:

* Reproducible
* Version controlled
* Reviewable
* Automated
* Easier to modify consistently

---

# 6. Why Terraform for Production?

Suppose we manually create:

```text
VPC
Subnets
IAM
EKS
Node Groups
Security Groups
Autoscaling
```

Later someone asks:

> "What exactly did we create?"

or:

> "Create the same environment in another region/account."

Manual configuration becomes difficult.

With Terraform:

```text
Terraform Code
      ↓
AWS Infrastructure
```

The infrastructure configuration is stored as code.

This makes changes easier to review and reproduce.

---

# 7. `kubectl` and `eksctl` — Different Purposes

Two important commands/tools:

## kubectl

`kubectl` is the Kubernetes command-line client.

It is used to interact with the Kubernetes cluster.

Examples:

```bash
kubectl get nodes
kubectl get pods
kubectl apply -f pod.yml
kubectl delete pod nginx
```

Mental model:

```text
kubectl
   ↓
Kubernetes API Server
   ↓
Kubernetes Cluster
```

---

## eksctl

`eksctl` is used primarily to create and manage EKS clusters.

Example:

```bash
eksctl create cluster ...
```

Mental model:

```text
eksctl
   ↓
AWS EKS
   ↓
Create/manage EKS infrastructure
```

### Important

Do not confuse:

```text
kubectl → operate Kubernetes resources

eksctl  → create/manage EKS clusters
```

---

# 8. Installing the Required Tools

If the client machine gives:

```text
kubectl: command not found
```

then `kubectl` needs to be installed.

If:

```text
eksctl: command not found
```

then `eksctl` needs to be installed.

For the EKS learning environment, commonly required tools are:

```text
AWS CLI
kubectl
eksctl
```

Terraform is additionally required when we move to Infrastructure as Code.

---

# 9. Creating an EKS Cluster Using `eksctl`

Basic syntax:

```bash
eksctl create cluster \
  --name <cluster-name> \
  --region <region> \
  --node-type <instance-type>
```

Example structure:

```text
eksctl create cluster
       │
       ├── cluster name
       ├── AWS region
       └── worker node instance type
```

The exact defaults can depend on the `eksctl` version and configuration, so don't blindly memorize a default worker-node count as a universal rule.

---

# 10. EKS Control Plane — Why Can't We See It?

In EKS:

```text
EKS Cluster
│
├── AWS-managed Control Plane
│
└── Customer-managed Worker Nodes / Node Groups
```

AWS manages the EKS Control Plane infrastructure.

Therefore, from your EC2 console you generally see the Worker Node instances that belong to your node group, not the EKS-managed control-plane machines.

Your Kubernetes commands still interact with the Control Plane:

```text
kubectl
   │
   ▼
EKS API Server
   │
   ▼
Control Plane
   │
   ▼
Worker Nodes
```

---

# 11. AWS EKS Control Plane High Availability

The EKS Control Plane is a managed AWS service designed for high availability.

The exact internal AWS implementation is abstracted from the customer.

Therefore, don't memorize a simplified statement such as:

> "There is one primary control plane and AWS immediately promotes a backup."

Instead remember:

> **AWS manages the highly available EKS Control Plane for us; customers don't manage its underlying control-plane instances.**

---

# 12. CloudFormation and `eksctl`

When `eksctl` creates EKS resources, AWS infrastructure orchestration can involve **AWS CloudFormation**.

This explains why the IAM permissions required during the training include permissions related to services such as:

```text
EKS
EC2
VPC
CloudFormation
IAM
```

The exact resources created depend on the `eksctl` configuration.

---

# 13. EKS Cluster Creation Takes Time

Creating an EKS cluster is not instantaneous.

It can take significant time because AWS needs to provision and configure multiple resources.

Conceptually:

```text
eksctl create cluster
        ↓
AWS infrastructure provisioning
        ↓
EKS Control Plane
        ↓
Networking
        ↓
Node Group
        ↓
Worker Nodes
        ↓
Cluster ready
```

Therefore, don't assume a command returning immediately means the entire cluster is already ready.

---

# 14. Node Group = Worker Nodes

In EKS terminology:

```text
EKS Cluster
│
├── Control Plane
│
└── Node Group
       │
       ├── Worker Node
       ├── Worker Node
       └── Worker Node
```

A Node Group is a group of Worker Nodes managed together.

---

# 15. Why AWS Knowledge Is Important for Kubernetes

Kubernetes can be learned independently, but when using **EKS**, AWS knowledge becomes extremely important.

You need to understand concepts such as:

```text
VPC
Subnets
Availability Zones
Security Groups
IAM
EC2
Load Balancers
ECR
Auto Scaling
Networking
```

because Kubernetes workloads are ultimately running inside an AWS infrastructure environment.

---

# 16. ECR → Kubernetes → Pod

In a real application environment, developers provide application source code.

A CI/CD pipeline typically:

```text
Source Code
    ↓
Build
    ↓
Docker Image
    ↓
Push Image
    ↓
Amazon ECR
```

Then Kubernetes references the image:

```text
ECR
 │
 │ container image
 ▼
Kubernetes
 │
 ▼
Pod
 │
 ▼
Container
 │
 ▼
Application
```

Typical production flow:

```text
Developer
   ↓
Git
   ↓
CI/CD Pipeline
   ↓
Docker Build
   ↓
Amazon ECR
   ↓
Kubernetes Deployment
   ↓
Pod
   ↓
Container
   ↓
Application
```

This is why understanding both **Docker and Kubernetes** is important.

---

# 17. Connecting to an Existing EKS Cluster

Once the EKS cluster is ready, configure the Kubernetes client so `kubectl` knows which cluster/API Server to communicate with.

A commonly used command is:

```bash
aws eks update-kubeconfig \
  --region <region> \
  --name <cluster-name>
```

This updates the kubeconfig configuration used by `kubectl`.

After that:

```bash
kubectl get nodes
```

can communicate with the EKS cluster.

---

# 18. What Happens Internally When We Run `kubectl get nodes`?

When we execute:

```bash
kubectl get nodes
```

the request goes to the Kubernetes API Server.

Simplified flow:

```text
kubectl
   │
   │ GET Nodes
   ▼
API Server
   │
   ▼
Cluster State
   │
   ▼
API Server
   │
   ▼
kubectl
   │
   ▼
Display Nodes
```

The API Server obtains the current cluster state from its backing data store/state mechanisms and returns the result.

For learning purposes, remember:

```text
kubectl
   ↓
API Server
   ↓
Cluster state
   ↓
Response
   ↓
kubectl
```

---

# 19. `kubectl get nodes -o wide`

Normal:

```bash
kubectl get nodes
```

gives basic node information.

Using:

```bash
kubectl get nodes -o wide
```

provides additional information.

For example, it can show information such as:

```text
NAME
STATUS
ROLES
AGE
VERSION
INTERNAL-IP
EXTERNAL-IP
OS-IMAGE
KERNEL-VERSION
CONTAINER-RUNTIME
```

The exact columns can vary by Kubernetes version/configuration.

---

# 20. Worker Node Failure and Auto Scaling

Suppose:

```text
Node-1
Node-2
```

are running.

If a Worker Node is manually terminated, the behavior depends on how the node group/infrastructure is configured.

When the nodes are managed by an Auto Scaling mechanism with desired capacity maintained, the underlying AWS autoscaling system can launch a replacement instance.

Conceptually:

```text
Desired Nodes = 2

Node-1
Node-2
```

Node-2 is terminated:

```text
Node-1
Node-2 ❌
```

The infrastructure autoscaling mechanism can launch:

```text
Node-1
Node-3
```

### Important distinction

This is **node-level infrastructure availability**.

It is different from Kubernetes deciding:

> "A Pod is pending, therefore create another Worker Node."

A normal Auto Scaling Group does not automatically understand Kubernetes Pod scheduling requirements by itself.

---

# 21. Why Doesn't the Node Automatically Increase When Pods Become Pending?

Suppose:

```text
Worker Node-1 → 90% utilized
Worker Node-2 → 90% utilized
```

A new Pod requires resources but cannot fit.

The Pod becomes:

```text
Pending
```

A basic/static ASG does not automatically interpret:

```text
Pending Pod
    ↓
Need another Kubernetes node
```

Therefore:

```text
Pods increase
     ↓
Resources become insufficient
     ↓
Pod Pending
```

does **not by itself guarantee**:

```text
Pending Pod
     ↓
New Node
```

---

# 22. Cluster Autoscaling

To automatically add Worker Nodes based on Kubernetes scheduling demand, we need a Kubernetes-aware node autoscaling mechanism.

The concept is:

```text
New Pods
    ↓
Insufficient Node Resources
    ↓
Pod Pending
    ↓
Cluster-level Node Autoscaler
    ↓
Increase Worker Nodes
    ↓
New Node joins cluster
    ↓
Scheduler can place Pod
```

This is different from merely having an AWS ASG with a fixed desired capacity.

The exact autoscaling solution will be studied separately.

---

# 23. Important Difference — ASG vs Kubernetes-Aware Node Autoscaling

### ASG

Primarily maintains infrastructure capacity according to its configured scaling behavior.

Example:

```text
Desired = 2
Min = 2
Max = 4
```

If one instance disappears:

```text
2 → 1
```

ASG can replace it to maintain desired capacity.

But it doesn't inherently mean:

```text
Pod Pending
   ↓
ASG automatically creates Node
```

### Kubernetes-aware autoscaling

Understands Kubernetes scheduling demand and can increase node capacity when Pods cannot be scheduled.

Conceptually:

```text
Pod Pending
    ↓
Need more node capacity
    ↓
Autoscaler
    ↓
Node group capacity increases
```

---

# 24. Why Do We Need Pods?

Kubernetes is a container orchestration platform.

Applications ultimately run inside containers.

However, Kubernetes doesn't normally manage containers directly as its primary workload abstraction.

The Kubernetes object used to run containers is the:

```text
Pod
```

Therefore:

```text
Kubernetes
    ↓
Pod
    ↓
Container
    ↓
Application
```

---

# 25. Pod Is the Smallest Deployable Kubernetes Unit

A Pod can contain one or more containers.

For basic applications:

```text
Pod
 └── Container
      └── Application
```

It is possible to have:

```text
Pod
 ├── Container-1
 └── Container-2
```

The containers in the same Pod share the Pod's networking namespace and can share storage volumes when configured.

---

# 26. Kubernetes YAML Files

Kubernetes resources are commonly declared using YAML manifests.

Examples:

```text
pod.yml
replicaset.yml
deployment.yml
```

For this training:

> **Focus more on understanding the Kubernetes concepts and internal behavior than on memorizing YAML syntax.**

The YAML is the desired-state declaration.

---

# 27. Creating the First Pod

Example:

```text
pod.yml
```

The training uses the public `nginx` image for learning.

Why?

Because the image is already available.

In a real application:

```text
Application Source
       ↓
Dockerfile
       ↓
Docker Image
       ↓
Container Registry
       ↓
Kubernetes Pod
```

For learning Kubernetes behavior, using a public image avoids unnecessary image-building work.

---

# 28. Create a Pod

Example:

```bash
kubectl apply -f pod.yml
```

Then:

```bash
kubectl get pods
```

This shows the Pods currently known to Kubernetes.

For example:

```text
NAME
nginx
```

---

# 29. `kubectl get pods -o wide`

Use:

```bash
kubectl get pods -o wide
```

to obtain additional Pod information.

It can show information such as:

```text
Pod Name
Ready
Status
Restarts
Age
IP
Node
```

This is particularly useful because it allows us to see:

```text
Pod
  ↓
Scheduled Node
```

For example:

```text
nginx
  ↓
worker-node-1
```

---

# 30. Why Can't One Pod Be Split Across Two Worker Nodes?

A Pod is scheduled as a unit.

Suppose:

```text
Pod
 └── Container
```

The Pod cannot be partially placed on:

```text
Node-1
```

and partially placed on:

```text
Node-2
```

The Scheduler chooses **one Worker Node for the Pod**.

For example:

```text
             Scheduler
                 │
                 ▼
              Pod-1
                 │
                 ▼
            Worker Node-1
```

If the application needs more capacity, Kubernetes can run additional Pods:

```text
Pod-1 → Node-1
Pod-2 → Node-2
Pod-3 → Node-1
```

This is one reason replication and higher-level workload controllers are important.

---

# 31. What Happens When `kubectl apply -f pod.yml` Is Executed?

Simplified mental flow:

```text
kubectl
   ↓
API Server
   ↓
Authentication
   ↓
Authorization
   ↓
Admission
   ↓
Validation
   ↓
Pod object created
   ↓
Persisted in etcd
   ↓
Scheduler sees unscheduled Pod
   ↓
Selects suitable Worker Node
   ↓
Scheduling decision sent through API Server
   ↓
Pod assigned to Worker Node
   ↓
Kubelet on selected Node
   ↓
Container Runtime
   ↓
Pull image
   ↓
Create container
   ↓
Start application
```

### Important correction to the simplified training explanation

Do not memorize this as:

```text
API Server
   ↓
"asks etcd"
   ↓
etcd decides
   ↓
Scheduler
```

Instead:

> The API Server handles the Kubernetes API request and persists the object through etcd. The Scheduler watches the API for Pods that need scheduling.

Also, the Scheduler does not normally directly tell the Kubelet to start the Pod. The scheduling decision is recorded in the Pod object through the API Server.

---

# 32. Is Controller Manager Involved in a Directly Created Pod?

For the basic **bare Pod** scenario:

```yaml
kind: Pod
```

we are not using a Deployment or ReplicaSet to manage it.

Therefore, the main flow being studied is:

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

There is no ReplicaSet/Deployment controller responsible for maintaining that Pod.

The detailed Controller Manager behavior will be introduced when we study ReplicaSets and Deployments.

---

# 33. Applying the Same YAML Again

Suppose:

```bash
kubectl apply -f pod.yml
```

has already created:

```text
Pod name = nginx
```

If we run the same command again:

```bash
kubectl apply -f pod.yml
```

Kubernetes does not create another Pod with the same identity.

The existing Kubernetes object is reconciled with the requested configuration.

Conceptually:

```text
Requested Object
       │
       ▼
API Server
       │
       ▼
Existing Pod
       │
       ▼
Is there a meaningful change?
      / \
    No   Yes
    │     │
    ▼     ▼
 No new  Update
 Pod     object
```

---

# 34. YAML Filename Is NOT the Kubernetes Resource Identity

Suppose:

```text
pod.yml
```

contains:

```yaml
metadata:
  name: nginx
```

You rename the file:

```text
pod.yml → pod1.yml
```

and run:

```bash
kubectl apply -f pod1.yml
```

Kubernetes does not care that the Linux filename changed.

The Kubernetes object is still:

```text
Pod
name = nginx
```

Therefore, it refers to the same Kubernetes resource identity.

### Important

```text
pod.yml
pod1.yml
nginx.yaml
myfile.yaml
```

are just local filenames.

Kubernetes identifies the resource using its API identity, including things such as:

```text
Kind
Namespace
Name
```

---

# 35. Changing the Pod Name Creates a New Pod

If you change:

```yaml
metadata:
  name: nginx
```

to:

```yaml
metadata:
  name: nginx-2
```

and apply:

```bash
kubectl apply -f pod1.yml
```

then Kubernetes sees a different object identity:

```text
nginx
```

versus:

```text
nginx-2
```

Therefore a new Pod object is created.

Conceptually:

```text
etcd:
  nginx exists
  nginx-2 does not exist

             ↓

Create nginx-2
```

---

# 36. Deleting a Bare Pod

Command:

```bash
kubectl delete pod <pod-name>
```

The Pod object is removed through the Kubernetes API.

Conceptually:

```text
kubectl
   ↓
API Server
   ↓
Pod deletion
   ↓
etcd
   ↓
Kubelet
   ↓
Container termination
```

Because this is a bare Pod:

```text
No Deployment
No ReplicaSet
No controller maintaining replicas
```

there is no controller whose job is to create a replacement Pod.

Therefore:

```text
Pod deleted
    ↓
No replacement Pod
```

---

# 37. Why Doesn't the Bare Pod Automatically Come Back?

Because there is no desired replica count being maintained by a higher-level controller.

For a directly created Pod:

```text
Desired:
"This particular Pod exists."

Current:
"Pod exists."
```

If the Pod is deleted, the Pod object itself is gone.

There is no ReplicaSet saying:

```text
Desired = 3
Current = 2
```

and therefore:

```text
Create replacement
```

---

# 38. Why Bare Pods Are Not Normally Used for Application HA

Suppose:

```text
Application
    ↓
One Pod
```

If the Pod dies:

```text
Application
    ↓
Pod ❌
```

The application is unavailable.

Therefore, production applications normally use higher-level controllers such as:

```text
Deployment
StatefulSet
DaemonSet
Job
```

depending on the workload.

For learning/troubleshooting, however, a temporary bare Pod can be very useful.

Example:

```text
Create temporary Pod
      ↓
Test DNS
      ↓
Test service connectivity
      ↓
Test network
      ↓
Delete Pod
```

---

# 39. Delete Using YAML

Deleting the Linux file:

```bash
rm -rf pod.yml
```

does **not** delete the Kubernetes Pod.

It only deletes the local YAML file.

To delete the Kubernetes resource using its manifest:

```bash
kubectl delete -f pod.yml
```

This sends a Kubernetes deletion request through the API.

Conceptually:

```text
pod.yml
   ↓
kubectl delete -f
   ↓
API Server
   ↓
Delete Pod object
   ↓
etcd
```

---

# 40. Terraform and Kubernetes Pods

A key distinction:

```text
AWS EC2
```

is AWS infrastructure.

Terraform can create/manage it using the AWS provider.

A Kubernetes Pod is a Kubernetes workload object, not an AWS infrastructure service.

Therefore, in this learning context:

```text
Terraform
   ↓
AWS infrastructure
   ↓
EKS cluster
```

while:

```text
kubectl
   ↓
Kubernetes resources
   ↓
Pods / Deployments / Services / etc.
```

Terraform can also interact with Kubernetes resources through Kubernetes-related providers, but that is a separate design choice and should not be confused with using Terraform to provision the underlying AWS infrastructure.

---

# 41. ReplicaSet — Why Do We Need It?

A bare Pod does not provide self-healing through a replica controller.

ReplicaSet introduces the concept of:

```text
Desired number of Pods
```

Example:

```yaml
replicas: 3
```

This means:

```text
Desired State = 3 Pods
```

The ReplicaSet Controller continuously tries to maintain:

```text
Current State = Desired State
```

---

# 42. ReplicaSet Self-Healing

Suppose:

```text
Desired = 3
Current = 3
```

Everything is healthy:

```text
Pod-1
Pod-2
Pod-3
```

Now Pod-2 is deleted:

```text
Pod-1
Pod-2 ❌
Pod-3
```

Current state becomes:

```text
Current = 2
```

But:

```text
Desired = 3
Current = 2
```

The ReplicaSet Controller detects the difference.

```text
ReplicaSet Controller
        │
        │ Desired ≠ Current
        ▼
Create replacement Pod
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
New Pod
```

Therefore:

> **ReplicaSet provides self-healing by maintaining the desired number of Pod replicas.**

---

# 43. ReplicaSet Selector and Pod Labels

The ReplicaSet needs to know:

> "Which Pods belong to me?"

This is established using labels and selectors.

Example concept:

```text
ReplicaSet selector:
app = nginx
```

Pod template:

```text
labels:
  app = nginx
```

Conceptually:

```text
ReplicaSet
   │
   │ selector: app=nginx
   │
   ▼
Pods with label:
app=nginx
```

This connection is fundamental to understanding ReplicaSets and Deployments.

---

# 44. Create a ReplicaSet

Typical workflow:

```bash
kubectl apply -f rs.yml
```

Then inspect:

```bash
kubectl get pods
```

and:

```bash
kubectl get pods -o wide
```

To inspect ReplicaSets:

```bash
kubectl get rs
```

This allows you to see the relationship:

```text
ReplicaSet
    ↓
Pods
```

---

# 45. Deleting a Pod Managed by ReplicaSet

Suppose:

```text
ReplicaSet
Desired = 3

Pod-1
Pod-2
Pod-3
```

Delete:

```bash
kubectl delete pod <pod-name>
```

Then:

```text
Pod deleted
    ↓
Cluster state changes
    ↓
ReplicaSet Controller observes
    ↓
Current = 2
Desired = 3
    ↓
Create replacement Pod
```

The important idea is:

> **The ReplicaSet Controller does not directly "magically restart" the deleted Pod. It notices that the desired state and current state differ and creates a replacement Pod.**

---

# 46. ReplicaSet vs AWS Auto Scaling Group

A useful analogy:

### AWS ASG

```text
Desired EC2 instances = 2
Current = 1

ASG
 ↓
Create replacement EC2
```

### Kubernetes ReplicaSet

```text
Desired Pods = 3
Current = 2

ReplicaSet Controller
 ↓
Create replacement Pod
```

The concepts are similar:

```text
Desired State
      ↓
Compare Current State
      ↓
Take corrective action
```

But they operate at different layers:

```text
ASG
 ↓
Infrastructure / EC2

ReplicaSet
 ↓
Kubernetes Pods
```

---

# 47. Why ReplicaSet Alone Is Not Normally Used for Application Deployments

ReplicaSet provides replication and self-healing.

However, application releases frequently require:

```text
New image
New application version
Rolling update
Rollback
Version history
```

A ReplicaSet alone is not the higher-level deployment mechanism we normally use for these application release workflows.

Example:

```text
Version 1
nginx:v1
```

Later:

```text
Version 2
nginx:v2
```

We want:

```text
Old Pods
   ↓
Gradually replace
   ↓
New Pods
```

without unnecessarily taking the application completely offline.

This is where **Deployment** becomes important.

---

# 48. Deployment — Production-Level Workload Management

A Deployment provides a higher-level abstraction for managing application Pods and ReplicaSets.

Conceptually:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

Therefore:

```text
Deployment Controller
        ↓
ReplicaSet
        ↓
Pods
```

This provides capabilities such as:

* Desired replica management
* Self-healing through ReplicaSet
* Rolling updates
* Rollbacks
* Versioned rollout management

---

# 49. Deployment Rolling Update

Suppose the current application is:

```text
Version 1
nginx:v1
```

and there are:

```text
Pod-1
Pod-2
Pod-3
```

Developer releases:

```text
Version 2
nginx:v2
```

The Deployment configuration is updated.

The Deployment manages the rollout so that the application can transition from:

```text
v1
```

to:

```text
v2
```

without unnecessarily deleting all old Pods at once.

Simplified:

```text
v1 Pods
v1 Pods
v1 Pods
    │
    │ Rolling Update
    ▼
v2 Pod created
    ↓
old v1 Pod removed
    ↓
v2 Pod created
    ↓
old v1 Pod removed
    ↓
...
    ▼
v2 Pods
v2 Pods
v2 Pods
```

The exact rollout behavior depends on the Deployment strategy and its configuration.

---

# 50. Observing a Deployment Rollout

A useful learning technique is to watch Pods in another terminal:

```bash
kubectl get pods --watch
```

Then update the Deployment image and apply:

```bash
kubectl apply -f deploy.yml
```

Observe:

```text
New Pods
   ↓
Old Pods terminating
   ↓
New Pods becoming Ready
   ↓
Old Pods removed
```

This helps visualize a rolling update rather than just reading about it.

---

# 51. Why Deployment Is Preferred in Real-Time Application Environments

For a typical stateless application:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

is the common production pattern.

The Deployment provides a controlled mechanism for:

```text
Application version management
        ↓
Rolling updates
        ↓
Rollback capability
        ↓
Replica management
        ↓
Self-healing
```

Therefore, for a normal stateless microservice, we generally don't deploy the application directly as:

```text
kind: Pod
```

Instead:

```text
kind: Deployment
```

is commonly used.

---

# 52. Final Comparison — Pod vs ReplicaSet vs Deployment

| Resource   | Main Purpose                                | Self-Healing |                        Rolling Updates | Typical Application Use           |
| ---------- | ------------------------------------------- | -----------: | -------------------------------------: | --------------------------------- |
| Pod        | Run workload directly                       |            ❌ |                                      ❌ | Testing/troubleshooting           |
| ReplicaSet | Maintain number of Pods                     |            ✅ | Not the higher-level rollout mechanism | Usually managed by Deployment     |
| Deployment | Manage application releases and ReplicaSets |            ✅ |                                      ✅ | Common for stateless applications |

### Mental Model

```text
Pod
 │
 └── "Run this workload."

ReplicaSet
 │
 └── "Keep N Pods running."

Deployment
 │
 └── "Manage the application,
      replicas, updates and rollouts."
```

---

# 53. Complete Day-3 Kubernetes Mental Model

Keep this architecture in your mind:

```text
                         EKS CLUSTER
                              │
              ┌───────────────┴───────────────┐
              │                               │
        CONTROL PLANE                    WORKER NODES
              │                               │
       ┌──────┼───────┐                ┌──────┴──────┐
       │      │       │                │             │
    API     Scheduler Controller      Kubelet    Runtime
   Server            Manager            │
       │                               │
      etcd                             Pods
```

For a **bare Pod**:

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

For a **ReplicaSet**:

```text
kubectl
   ↓
API Server
   ↓
ReplicaSet Controller
   ↓
Pods
   ↓
Scheduler
   ↓
Kubelets
```

For a **Deployment**:

```text
kubectl
   ↓
API Server
   ↓
Deployment Controller
   ↓
ReplicaSet
   ↓
Pods
   ↓
Scheduler
   ↓
Kubelets
   ↓
Containers
```

---

# 54. Production Problem → Kubernetes Solution

## Problem 1 — Pod dies

```text
Bare Pod
   ↓
Pod dies
   ↓
No replacement
```

### Solution

Use a controller such as ReplicaSet/Deployment.

---

## Problem 2 — Need multiple replicas

```text
One Pod
```

is not enough for availability.

### Solution

```text
replicas: 3
```

through a ReplicaSet/Deployment.

---

## Problem 3 — Pod deleted accidentally

### Bare Pod

```text
Pod deleted
   ↓
Application unavailable
```

### Deployment/ReplicaSet

```text
Pod deleted
   ↓
Controller detects difference
   ↓
Replacement Pod
```

---

## Problem 4 — New application version

```text
v1 → v2
```

We don't want:

```text
Delete all Pods
   ↓
Application completely unavailable
   ↓
Create all new Pods
```

### Solution

Use a Deployment rolling update.

```text
v1 Pods
   ↓
Gradual replacement
   ↓
v2 Pods
```

---

## Problem 5 — Pods cannot be scheduled because nodes lack resources

```text
Pods
 ↓
Insufficient resources
 ↓
Pending Pods
```

A basic ASG does not automatically solve the Kubernetes scheduling problem merely because a Pod is pending.

### Solution

Integrate a Kubernetes-aware node autoscaling mechanism.

```text
Pending Pod
    ↓
Autoscaling mechanism
    ↓
Increase node capacity
    ↓
New Worker Node
    ↓
Scheduler
    ↓
Pod scheduled
```

---

# 55. Day-3 Key Takeaways

Remember these points rather than memorizing commands:

```text
1. EKS = AWS managed Kubernetes service.

2. Control Plane is managed by AWS in EKS.

3. Worker Nodes run our Pods.

4. kubectl = interact with Kubernetes.

5. eksctl = create/manage EKS clusters.

6. Terraform = preferred Infrastructure-as-Code approach
   for reproducible production infrastructure.

7. ECR stores container images.

8. Kubernetes pulls the image and runs it inside Pods.

9. Pod = smallest deployable Kubernetes workload unit.

10. Bare Pod does not provide controller-based self-healing.

11. ReplicaSet maintains the desired number of Pods.

12. ReplicaSet uses selectors and labels to identify Pods.

13. Deployment manages ReplicaSets and application rollouts.

14. Deployment supports controlled rolling updates and rollbacks.

15. ASG maintains infrastructure capacity,
    but basic ASG behavior is different from
    Kubernetes-aware node autoscaling.

16. Kubernetes state is managed through the API Server,
    with etcd serving as the persistent backing store.

17. A Pod is scheduled as a unit onto one Worker Node.

18. YAML filename is not the Kubernetes resource identity.
    Resource identity comes from Kubernetes object metadata
    such as kind/namespace/name.
```

## The most important progression from Day-3

```text
Bare Pod
   │
   │ Problem: No self-healing
   ▼
ReplicaSet
   │
   │ Problem: Not the complete application
   │          rollout/release management solution
   ▼
Deployment
   │
   ├── Replica management
   ├── Self-healing
   ├── Rolling updates
   └── Rollbacks
   │
   ▼
Production Application Workload
```

This progression is the key story of today's session:

> **We don't move from Pod → ReplicaSet → Deployment just because Kubernetes has three YAML files. We move upward because each level solves a real operational problem.**
