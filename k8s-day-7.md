# =========================================

# DAY - 7

# Kubernetes Private Image & Argo CD

# =========================================

## 1. KUBERNETES — CONCEPTS COVERED SO FAR

=========================================

So far we discussed:

* Kubernetes Introduction
* Kubernetes Architecture
* Pod
* ReplicaSet
* Deployment
* Deployment YAML
* Services

  * ClusterIP
  * NodePort
  * LoadBalancer
* Node Autoscaling
* kube-proxy
* Kubernetes Endpoints
* Horizontal Pod Autoscaling (HPA)

### High Availability in Kubernetes

Q: How do you achieve High Availability in Kubernetes?

Mainly through:

1. Node Autoscaling
2. Pod Autoscaling

## 2. KUBERNETES SERVICE TYPES

=========================================

### ClusterIP

Used mainly for:

* Internal communication
* Pod-to-pod/application communication inside the cluster

Example:

Application A → ClusterIP Service → Application B

### NodePort

Used mainly for:

* External access to applications
* Testing purposes

NodePort exposes the service through:

```
Node IP + NodePort
```

It can also be used for internal testing.

However, the nodes need network accessibility from the client.

### LoadBalancer

Used mainly for:

* Production-style external access
* Creating an external Load Balancer

With a LoadBalancer service, we don't need to manually check which worker node contains the pod.

The Load Balancer distributes traffic to the Kubernetes nodes.

## 3. PRODUCTION VS QUICK LAB SETUP

=========================================

For quick practice:

* Kubernetes CLI commands can automatically create resources.
* We don't explicitly create everything manually.

For example:

* VPC
* Subnets
* Load Balancer-related resources
* Cluster resources

### Production Approach

In production, infrastructure should normally be created using Infrastructure as Code such as Terraform.

We explicitly specify:

* VPC
* Public subnets
* Private subnets
* Worker nodes
* Networking
* Load Balancer-related configuration

Typical production architecture:

```
Internet
   |
   v
Load Balancer
   |
   v
Public Subnets
   |
   v
Private Worker Nodes
   |
   v
Kubernetes Pods
```

## 4. DOCKER AND KUBERNETES

=========================================

Important point:

When we deploy a Pod:

```
kubectl apply
      |
      v
  Kubernetes
      |
      v
    Pod
      |
      v
  Container
```

We don't normally see Docker commands while deploying a Pod.

### Does Kubernetes use Docker?

Kubernetes does NOT require Docker as its container runtime.

Earlier:

```
Kubernetes
    |
    v
Docker
    |
    v
docker-shim
```

Older Kubernetes versions used Docker through `docker-shim`.

From Kubernetes 1.24 onward, the dockershim component was removed.

Modern Kubernetes uses the Container Runtime Interface (CRI).

Examples of container runtimes include:

* containerd
* CRI-O

## 5. WHY DO WE INSTALL DOCKER IN THE LAB?

=========================================

Docker is installed on our separate server because we need to:

1. Clone application source code
2. Build the Docker image
3. Push the image to ECR
4. Tell Kubernetes to use that image

Important:

```
Docker installation/build
        |
        v
Separate server
```

NOT:

```
Kubernetes worker node
```

## 6. INSTALL DOCKER AND GIT

=========================================

Install Docker:

```
yum install docker -y
```

Start Docker:

```
systemctl start docker
```

Install Git as well.

Then clone the application repository.

Example:

```
git clone <Swiggy-NodeJS-Repository>
```

Go inside the repository:

```
cd <repository>
```

Check files:

```
ls
```

## 7. BUILD THE DOCKER IMAGE

=========================================

The repository contains a Dockerfile.

Build the image:

```
docker build -t swiggy .
```

Important:

Each Dockerfile instruction creates a layer in the Docker image.

Check the image:

```
docker images
```

Example:

```
REPOSITORY
swiggy
```

## 8. CREATE ECR REPOSITORY

=========================================

Go to AWS Console:

```
AWS Console
    |
    v
Elastic Container Registry (ECR)
    |
    v
Create Repository
```

Repository name:

```
swiggy
```

## 9. PUSH DOCKER IMAGE TO ECR

=========================================

After creating the ECR repository, AWS provides the required push commands.

Basic flow:

```
Docker Image
     |
     v
ECR Login
     |
     v
Docker Tag
     |
     v
Docker Push
     |
     v
ECR Repository
```

### Step 1 — Login to ECR

Use the AWS ECR login command provided by AWS.

Modern form:

```
aws ecr get-login-password | docker login ...
```

### Step 2 — Tag the image

Example:

```
docker tag swiggy:latest <ECR-URI>/swiggy:latest
```

### Step 3 — Push the image

```
docker push <ECR-URI>/swiggy:latest
```

Now the Docker image is available inside ECR.

## 10. USE ECR IMAGE IN KUBERNETES

=========================================

The Deployment YAML should reference the ECR image.

Example concept:

```
deployment.yml
```

Inside the Deployment:

```
image: <ECR-URI>/swiggy:latest
```

Then create the Kubernetes resources:

```
kubectl apply -f .
```

Check Pods:

```
kubectl get pods
```

Check detailed Pod information:

```
kubectl get pods -o wide
```

Describe a Pod:

```
kubectl describe pod <pod-name>
```

Check Pod logs:

```
kubectl logs <pod-name>
```

## 11. HOW TO LOGIN INTO A POD

=========================================

Interview question:

Q: How do you login into a Kubernetes Pod?

Command:

```
kubectl exec -it <pod-name> -- /bin/bash
```

After entering the container, you can execute commands.

Example:

```
ps aux
```

If a command is not available, check which Linux distribution the container is using.

Command:

```
cat /etc/os-release
```

Example:

```
Debian
```

Important:

Don't blindly assume the container uses Amazon Linux, CentOS, Ubuntu, etc.

Always check the image/Dockerfile or:

```
cat /etc/os-release
```

## 12. BASIC POD DEBUGGING

=========================================

Interview question:

Q: My Pod is running but I cannot access the application. How will you troubleshoot?

Useful commands:

```
kubectl get pods

kubectl get pods -o wide

kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl get svc

kubectl get endpoints

kubectl exec -it <pod-name> -- /bin/bash
```

Basic troubleshooting flow:

```
Pod Status
   |
   v
Pod IP
   |
   v
Container Logs
   |
   v
Service
   |
   v
Endpoints
   |
   v
Network / Security Group
   |
   v
Application
```

## 13. SERVICE LABEL MATCHING

=========================================

Important concept:

The Deployment/Pod labels and Service selector must match.

Example:

Deployment:

```
labels:
  app: swiggy
```

Service:

```
selector:
  app: swiggy
```

Because the labels match:

```
Service
   |
   v
Pods
```

If labels don't match:

```
Service
   |
   X
No matching Pods
```

Therefore, the Service may not have the expected endpoints.

## 14. LOADBALANCER SERVICE

=========================================

If the Service type is:

```
LoadBalancer
```

Kubernetes requests an external Load Balancer.

Example:

```
Internet
   |
   v
External Load Balancer
   |
   v
Kubernetes Service
   |
   v
Worker Nodes
   |
   v
Pods
```

With a LoadBalancer service:

* You don't need to identify which node is running the Pod.
* The Load Balancer handles traffic distribution.

## 15. IMPORTANT PRODUCTION NETWORKING CONCEPT

=========================================

For a production cluster:

```
Internet
   |
   v
Load Balancer
   |
   v
Public Subnets
   |
   v
Private Subnets
   |
   v
Worker Nodes
   |
   v
Pods
```

Typical approach:

* Load Balancer → public-facing
* Worker nodes → private
* Pods → run on worker nodes

Terraform can be used to explicitly define:

* VPC
* Subnets
* Routing
* Security Groups
* Kubernetes cluster
* Worker nodes
* Load Balancer configuration

# =========================================

# ARGO CD

# =========================================

## 16. WHAT IS ARGO CD?

=========================================

Argo CD is a Kubernetes-native Continuous Deployment (CD) tool.

It is commonly used to implement:

```
GitOps
```

Argo CD runs inside the Kubernetes cluster and continuously monitors the desired configuration stored in Git.

## 17. WHAT IS GITOPS?

=========================================

GitOps means:

```
Git becomes the source of truth.
```

Instead of developers directly changing the Kubernetes cluster, they make changes in Git.

Example:

```
Developer
   |
   v
Git Repository
   |
   v
Argo CD
   |
   v
Kubernetes Cluster
   |
   v
Pods
```

## 18. TRADITIONAL DEPLOYMENT PROCESS

=========================================

Traditional process:

```
Developer
   |
   v
kubectl
   |
   v
Kubernetes Control Plane
   |
   v
Worker Nodes
   |
   v
Pods
```

Here:

* Developer directly interacts with Kubernetes.
* `kubectl` sends instructions to the Kubernetes API server/control plane.
* Kubernetes performs the required operations.

## 19. GITOPS DEPLOYMENT PROCESS

=========================================

GitOps process:

```
Developer
   |
   v
Git Repository
   |
   v
Argo CD
   |
   v
Kubernetes Cluster
   |
   v
Pods
```

Here:

* Developer does not directly deploy using kubectl.
* Developer pushes YAML changes to Git.
* Argo CD monitors Git.
* Argo CD detects the changes.
* Argo CD synchronizes the changes with Kubernetes.

## 20. WHY ARGO CD?

=========================================

Argo CD helps us:

* Automate deployments
* Improve productivity
* Reduce manual Kubernetes operations
* Maintain Git as the source of truth
* Improve deployment consistency
* Reduce direct access to the Kubernetes cluster
* Support GitOps practices

## 21. CI + CD + ARGO CD

=========================================

Typical modern flow:

```
Developer
   |
   v
Source Code Git
   |
   v
CI Pipeline
   |
   +----> Build Docker Image
   |
   +----> Push Image to ECR
   |
   v
Update Kubernetes YAML
   |
   v
Git Repository
   |
   v
Argo CD
   |
   v
Kubernetes Cluster
   |
   v
Pods
```

## 22. CI PIPELINE RESPONSIBILITY

=========================================

CI pipeline can perform:

1. Get source code
2. Build application
3. Build Docker image
4. Run tests
5. Perform code analysis
6. Push Docker image to image registry
7. Update image reference in Kubernetes YAML

Image registries can include:

* Docker Hub
* Amazon ECR
* Other container registries

## 23. TWO GIT REPOSITORIES

=========================================

We can use:

### Repository 1 — Application Source Code

Contains:

* Application source code
* Dockerfile
* Application-related files

Example:

```
Source Code Git
      |
      v
Docker Build
      |
      v
Docker Image
      |
      v
ECR
```

### Repository 2 — Kubernetes Manifests

Contains:

* deployment.yml
* service.yml
* ConfigMaps
* Other Kubernetes YAML files

Example:

```
Kubernetes Git
      |
      v
   Argo CD
      |
      v
Kubernetes Cluster
```

The source-code repository and Kubernetes-manifest repository can be:

* Separate repositories

OR

* The same repository

## 24. ARGO CD ARCHITECTURE

=========================================

Important flow:

```
Git Repository
     |
     | monitors
     v
  Argo CD
     |
     | synchronizes
     v
Kubernetes API
     |
     v
Worker Nodes
     |
     v
   Pods
```

Argo CD itself runs inside Kubernetes.

Therefore:

```
Kubernetes Cluster
    |
    +-- Argo CD Namespace
    |
    +-- Application Namespace
    |
    +-- Worker Nodes
    |
    +-- Pods
```

## 25. CREATE ARGO CD NAMESPACE

=========================================

Create a separate namespace:

```
kubectl create namespace argocd
```

Install Argo CD:

```
kubectl apply -n argocd -f \
https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Check Argo CD Pods:

```
kubectl get pods -n argocd
```

This allows us to see the resources created inside the `argocd` namespace.

## 26. ACCESS ARGO CD UI

=========================================

Argo CD provides a web UI.

By default, the Argo CD server Service can be ClusterIP.

ClusterIP is internally accessible.

To access it externally for the lab, we can change it to NodePort.

Check services:

```
kubectl get svc -n argocd
```

Edit the service:

```
kubectl edit svc argocd-server -n argocd
```

Change:

```
type: ClusterIP
```

to:

```
type: NodePort
```

Check again:

```
kubectl get svc -n argocd
```

Then access:

```
Node IP + NodePort
```

## 27. ARGO CD LOGIN

=========================================

Argo CD has an admin user.

Username:

```
admin
```

The initial admin password needs to be retrieved from the Argo CD installation.

It is similar to the concept of obtaining an initial administrator password in other tools.

## 28. CREATE AN ARGO CD APPLICATION

=========================================

In the Argo CD UI:

```
+ NEW APP
```

Provide:

### Application Name

Example:

```
CD-pipeline
```

### Project

Choose the required project.

### Sync Policy

Enable:

```
Auto-Sync
```

### Source

Provide the Git repository containing Kubernetes YAML files.

Example:

```
GitHub Repository
      |
      v
deployment.yml
service.yml
```

### Revision

Specify the branch.

Example:

```
main
```

### Path

Specify the directory where the Kubernetes YAML files exist.

### Destination Namespace

Example:

```
dev
```

Create the application.

## 29. APPLICATION NAMESPACE

=========================================

Create namespace:

```
kubectl create namespace dev
```

Argo CD can deploy the application into:

```
dev
```

When a Deployment is created:

```
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
```

Therefore, when you deploy an application, a ReplicaSet is also created as part of the Deployment mechanism.

## 30. ARGO CD AUTO-SYNC

=========================================

With Auto-Sync enabled:

```
Developer
   |
   v
Update YAML
   |
   v
Git Commit
   |
   v
GitHub
   |
   v
Argo CD detects change
   |
   v
Kubernetes updated
   |
   v
New/updated Pods
```

Therefore, developers only need to push their Kubernetes YAML changes to Git.

## 31. MANUAL SYNC

=========================================

Instead of waiting for automatic synchronization, we can manually click:

```
SYNC
```

Argo CD then synchronizes the Git state with the Kubernetes cluster.

## 32. ARGO CD UI CAPABILITIES

=========================================

From the Argo CD UI, we can observe:

* Application status
* Kubernetes resources
* Pods
* ReplicaSets
* Deployments
* Services
* Logs
* Live manifests
* Sync status

We can also perform certain Kubernetes operations through the UI.

## 33. LIVE MANIFEST

=========================================

Argo CD provides a view of the Live Manifest.

The Live Manifest represents the configuration currently running in the Kubernetes cluster.

Git represents:

```
Desired State
```

Kubernetes represents:

```
Live State
```

Argo CD compares:

```
Desired State
      VS
Live State
```

If they differ, Argo CD can synchronize the cluster with Git.

## 34. GIT AS SINGLE SOURCE OF TRUTH

=========================================

Important GitOps concept:

```
Git = Source of Truth
```

Example:

Git says:

```
replicas: 3
```

But someone manually changes the cluster to:

```
replicas: 1
```

There is now a difference between:

```
Git Desired State
        VS
Kubernetes Live State
```

Argo CD detects this difference and can synchronize the cluster according to the Git configuration.

## 35. ARGO CD VS DIRECT KUBECTL

=========================================

### Traditional

```
Developer
   |
   v
kubectl
   |
   v
Kubernetes
```

### GitOps

```
Developer
   |
   v
Git
   |
   v
Argo CD
   |
   v
Kubernetes
```

Main difference:

```
Traditional:
Developer directly interacts with Kubernetes.

GitOps:
Developer interacts with Git and Argo CD handles deployment.
```

# =========================================

# IMPORTANT INTERVIEW QUESTIONS

# =========================================

### Q1. Does Kubernetes use Docker?

Answer:

Modern Kubernetes does not require Docker as its container runtime. Kubernetes uses the Container Runtime Interface (CRI), with runtimes such as containerd or CRI-O.

### Q2. Why did we install Docker on the server?

Answer:

We installed Docker on a separate server to build the application Docker image. The image was then pushed to Amazon ECR, and Kubernetes pulls/uses that image to create containers.

### Q3. How do you login to a Pod?

```
kubectl exec -it <pod-name> -- /bin/bash
```

### Q4. How do you check Pod logs?

```
kubectl logs <pod-name>
```

### Q5. How do you troubleshoot a Pod?

Use:

```
kubectl get pods

kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl get svc

kubectl get endpoints

kubectl exec -it <pod-name> -- /bin/bash
```

### Q6. What is Argo CD?

Argo CD is a Kubernetes-native Continuous Deployment tool used to implement GitOps.

### Q7. What is GitOps?

GitOps is a deployment approach where Git is treated as the source of truth for the desired state of the infrastructure/application.

### Q8. What is the role of Argo CD?

Argo CD continuously monitors the Git repository and synchronizes the desired configuration from Git with the Kubernetes cluster.

### Q9. Why is Argo CD installed inside Kubernetes?

Argo CD is a Kubernetes-native CD tool and runs inside the Kubernetes cluster to monitor Git and manage Kubernetes resources.

### Q10. Can source code and Kubernetes YAMLs be in different repositories?

Yes.

We can use:

```
Source Code Repository
        +
Kubernetes Manifest Repository
```

Or we can keep both in the same repository.

### Q11. What happens when a developer changes a Kubernetes YAML?

Typical GitOps flow:

```
Developer changes YAML
      |
      v
Git commit/push
      |
      v
Argo CD detects change
      |
      v
Argo CD synchronizes
      |
      v
Kubernetes resources updated
      |
      v
Pods updated
```

# =========================================

# DAY 7 — QUICK REVISION

# =========================================

1. Docker is used to build the application image.

2. Docker image is pushed to ECR.

3. Kubernetes Deployment references the ECR image.

4. Kubernetes creates Pods using the image.

5. Service provides network access to Pods.

6. Service selector must match Pod labels.

7. ClusterIP is mainly for internal communication.

8. NodePort can be used for external access and lab testing.

9. LoadBalancer provides external Load Balancer access.

10. Production infrastructure should preferably be managed using Terraform/IaC.

11. Modern Kubernetes uses CRI-compatible container runtimes.

12. Argo CD is a Kubernetes-native CD tool.

13. Argo CD implements GitOps.

14. Git is treated as the source of truth.

15. Developer pushes YAML changes to Git.

16. Argo CD monitors the Git repository.

17. Argo CD synchronizes Git state with Kubernetes.

18. Auto-Sync can automatically deploy changes.

19. Argo CD can show Pods, Services, Deployments, logs and manifests.

20. GitOps reduces the need for developers to directly interact with the Kubernetes control plane.

# =========================================

# DAY 7 END

# =========================================

Today we learned:

```
Private Docker Image
      |
      v
    ECR
      |
      v
Kubernetes Deployment
      |
      v
    Pods
      |
      v
  Service
```

And:

```
Developer
    |
    v
   Git
    |
    v
 Argo CD
    |
    v
```

Kubernetes
|
v
Pods

Tomorrow we will continue with Argo CD and deeper Kubernetes concepts.
