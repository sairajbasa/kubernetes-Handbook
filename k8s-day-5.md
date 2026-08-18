# DAY - 5

## Kubernetes Services, NodePort, LoadBalancer, Endpoints & Namespaces

---

## 1. Career / Learning Approach

* Apply for **minimum 20 jobs every day**.
* Don't wait until the course is completed to start applying.
* In the current market, **Cloud + DevOps** should be the primary focus.
* Don't try to learn everything at the same time.
* First become strong in one primary skill.
* Once you are comfortable with around **60% of DevOps**, then start exploring:

  * GenAI
  * Agentic AI
  * AIOps
  * AI Agents
* GenAI/Agentic AI applications still require:

  * Docker
  * Kubernetes
  * Cloud
  * CI/CD
  * Monitoring
  * Observability
  * Security
  * Networking
* The same Cloud/DevOps fundamentals are repeatedly used in:

  * AI Agent deployment
  * Cybersecurity
  * Cloud security
  * Application security
  * OS security
  * Network security
  * Vulnerability scanning
* Don't get distracted by multiple areas before becoming strong in the primary skill.
* Once you get a job, focus deeply on the technologies required by your project.

---

# 2. Why Kubernetes Service?

### Kubernetes architecture

```text
Kubernetes Cluster
│
├── Control Plane
│
└── Worker Nodes
    │
    ├── Pod
    │   └── Application
    │
    └── Pod
        └── Application
```

* Application runs inside a **Pod**.
* Users should not directly access Pods.
* Pod IPs are generally not used as the stable access point for applications.
* Kubernetes provides a **Service** to expose applications.

```text
User
  ↓
Service
  ↓
Selected Pod
  ↓
Application
```

### Main purpose of Service

* Provides a stable way to access Pods.
* Selects the required Pods using **labels**.
* Provides a stable Service IP/DNS.
* Can expose applications:

  * Internally → ClusterIP
  * Externally → NodePort
  * Externally through cloud Load Balancer → LoadBalancer

---

# 3. Types of Kubernetes Services

There are mainly 4 Service types:

```text
1. ClusterIP
2. NodePort
3. LoadBalancer
4. Headless Service
```

In this class:

* ClusterIP
* NodePort
* LoadBalancer

Headless Service will be discussed separately.

---

# 4. ClusterIP

```yaml
type: ClusterIP
```

* **ClusterIP is the default Service type.**
* Used mainly for **internal communication** between applications/services inside the cluster.
* It is not directly exposed to external users.

Example:

```text
Frontend Pod
     ↓
Backend Service
     ↓
Backend Pods
```

The frontend can communicate with the backend through the backend Service.

### Simple understanding

```text
ClusterIP
    ↓
Internal access
    ↓
Inside Kubernetes cluster
```

---

# 5. NodePort

```yaml
type: NodePort
```

* Used to provide external access through a **worker node IP + NodePort**.
* Commonly useful for:

  * Development
  * Testing
  * Limited external access

Access format:

```text
NodePublicIP:NodePort
```

Example:

```text
http://13.x.x.x:30007
```

---

# 6. LoadBalancer

```yaml
type: LoadBalancer
```

* Used to expose an application externally through a **cloud Load Balancer**.
* In AWS/EKS, Kubernetes can provision the corresponding AWS load-balancing resources.
* This is commonly used for production-style external access.

Basic flow:

```text
Internet User
      ↓
AWS Load Balancer
      ↓
Kubernetes Service
      ↓
Pod
      ↓
Application
```

---

# 7. Service Manifest

Service YAML follows the same basic approach as other Kubernetes objects.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: my-app-svc
  labels:
    app: my-app

spec:
  type: NodePort

  selector:
    app: my-pods

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

---

# 8. Most Important Concept — Labels

Suppose Deployment creates Pods with:

```yaml
spec:
  template:
    metadata:
      labels:
        app: my-pods
```

Service:

```yaml
spec:
  selector:
    app: my-pods
```

These values match:

```text
Pod label
    ↓
app: my-pods

        MATCH

Service selector
    ↓
app: my-pods
```

Therefore, the Service selects those Pods.

### Remember

```text
Service selector
        ↓
matches
        ↓
Pod labels
```

**The Service does not select Pods using the Deployment's `metadata.labels`.**

It selects Pods using the labels actually present on the Pods, which are normally defined under:

```yaml
spec:
  template:
    metadata:
      labels:
```

---

# 9. Why Labels Are Important

Suppose:

```text
100 Pods
50 different applications
```

How does a Service know which Pods belong to it?

Through **labels and selectors**.

Example:

```text
Pod 1 → app: frontend
Pod 2 → app: frontend
Pod 3 → app: backend
Pod 4 → app: database
```

Frontend Service:

```yaml
selector:
  app: frontend
```

Therefore:

```text
Frontend Service
      ↓
Pod 1
Pod 2
```

It does not send traffic to backend/database Pods.

---

# 10. Deployment + Service Example

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-deployment

spec:
  replicas: 2

  template:
    metadata:
      labels:
        app: my-pods
```

Apply:

```bash
kubectl apply -f deploy.yml
```

Check Pods:

```bash
kubectl get pods
```

Now create Service:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: my-service

spec:
  type: NodePort

  selector:
    app: my-pods

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

Apply:

```bash
kubectl apply -f svc.yml
```

---

# 11. Three Important Ports

For NodePort, remember these three ports:

```text
nodePort
   ↓
port
   ↓
targetPort
```

| Port         | Purpose                                                |
| ------------ | ------------------------------------------------------ |
| `nodePort`   | Port exposed on the worker node for external access    |
| `port`       | Port exposed by the Kubernetes Service                 |
| `targetPort` | Port where the application is listening inside the Pod |

---

# 12. targetPort

```yaml
targetPort: 80
```

* This represents the port where the application is actually listening inside the Pod.

Example:

```text
Pod
└── nginx
    └── listening on port 80
```

Therefore:

```yaml
targetPort: 80
```

---

# 13. Service port

```yaml
port: 80
```

* This is the port exposed by the Kubernetes Service.
* Other applications inside the cluster can communicate with the Service using this port.

---

# 14. nodePort

```yaml
nodePort: 30007
```

* This is the port exposed on the worker nodes.
* Default NodePort range:

```text
30000 - 32767
```

If you don't specify `nodePort`, Kubernetes can automatically allocate one from the allowed range.

---

# 15. NodePort Request Flow

Suppose:

```yaml
nodePort: 30007
port: 80
targetPort: 80
```

User accesses:

```text
NodePublicIP:30007
```

Conceptually:

```text
User
 │
 │ NodePublicIP:30007
 ↓
Worker Node
 │
 │ NodePort 30007
 ↓
Service
 │
 │ Service port 80
 ↓
Selected Pod
 │
 │ targetPort 80
 ↓
Application
```

### Important clarification

NodePort does **not literally redirect** the request.

Kubernetes networking forwards/translates the traffic through the Service to one of the Service's eligible endpoints.

---

# 16. NodePort vs Docker Port Mapping

Docker:

```text
-p 8080:80
```

Conceptually:

```text
Host Port
    ↓
Container Port
```

Kubernetes NodePort has an additional Service layer:

```text
NodePort
   ↓
Service port
   ↓
Pod targetPort
```

So:

```text
Docker:
Host → Container

Kubernetes:
Node → Service → Pod
```

---

# 17. Worker Node = EC2 Instance

In an AWS EKS environment:

```text
Kubernetes Worker Node
        ≈
AWS EC2 Instance
```

Therefore, if you want to access a NodePort externally:

```text
Node Public IP + NodePort
```

Example:

```text
http://<NodePublicIP>:30007
```

You can find the nodes using:

```bash
kubectl get nodes
```

More information:

```bash
kubectl get nodes -o wide
```

---

# 18. Find Which Node Is Running the Pod

```bash
kubectl get pods -o wide
```

This shows additional information such as:

* Pod IP
* Node where the Pod is running

Example:

```text
NAME        READY   STATUS    IP          NODE
my-pod      1/1     Running   10.0.2.15   node-1
```

---

# 19. Security Group Issue with NodePort

Suppose:

```text
Browser
   ↓
NodePublicIP:30007
```

But the application is not accessible.

One possible reason is the **AWS Security Group**.

The Security Group attached to the worker node must allow the required inbound traffic.

Example:

```text
Internet
   ↓
Node Public IP
   ↓
Security Group
   ↓
NodePort
```

If the Security Group blocks the traffic:

```text
Request ❌
```

Therefore, when debugging NodePort access, check the relevant network/security rules.

---

# 20. `kubectl get svc`

```bash
kubectl get svc
```

Used to display Kubernetes Services.

Example:

```text
NAME         TYPE        CLUSTER-IP     PORT(S)
kubernetes   ClusterIP   10.x.x.x       443/TCP
my-service   NodePort    10.x.x.x       80:30007/TCP
```

The `kubernetes` Service is normally present by default in a cluster.

---

# 21. Delete a Service

```bash
kubectl delete svc <service-name>
```

Example:

```bash
kubectl delete svc my-service
```

After deleting the Service:

```text
Service ❌
   ↓
Service-based access ❌

Pods ✅
```

### Important

Deleting a Service **does not delete the Pods** selected by that Service.

The Pods continue running, but they are no longer reachable through that Service.

---

# 22. LoadBalancer Service

Change:

```yaml
type: NodePort
```

to:

```yaml
type: LoadBalancer
```

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: my-service

spec:
  type: LoadBalancer

  selector:
    app: my-pods

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f svc.yml
```

Then:

```bash
kubectl get svc
```

Kubernetes requests a cloud Load Balancer through the cloud provider integration.

---

# 23. LoadBalancer Automatically Handles Cloud Integration

With an AWS-managed Kubernetes environment:

```text
Kubernetes Service
type: LoadBalancer
        ↓
AWS Load Balancer
        ↓
Kubernetes networking
        ↓
Service
        ↓
Pods
```

You don't normally manually create the Load Balancer and register every Pod as a target when using the Kubernetes Service abstraction.

Kubernetes/cloud integration handles the required infrastructure and configuration.

---

# 24. LoadBalancer and NodePort

A traditional Kubernetes `LoadBalancer` Service has historically been implemented on top of NodePort:

```text
LoadBalancer
     ↓
NodePort
     ↓
Service
     ↓
Pod
```

However, modern Kubernetes/cloud integrations can use different load-balancing implementations and may avoid NodePort when configured to do so.

For basic learning:

```text
LoadBalancer
    ↓
NodePort
    ↓
Service
    ↓
Pod
```

is a useful model.

---

# 25. External vs Internal Load Balancer

Conceptually:

### External Load Balancer

```text
Internet
   ↓
External LB
   ↓
Kubernetes Service
   ↓
Pods
```

Used when external users need access.

### Internal Load Balancer

```text
Internal Network
      ↓
Internal LB
      ↓
Kubernetes Service
      ↓
Pods
```

Used when access should remain private.

> The exact AWS annotations/configuration used to request an internal load balancer depend on the AWS load-balancer integration being used. Don't rely on `type: internal` as a generic Kubernetes field.

---

# 26. Public and Private Subnets

For an AWS architecture, commonly:

```text
VPC
│
├── Public Subnets
│   ├── AZ-1
│   └── AZ-2
│
└── Private Subnets
    ├── AZ-1
    └── AZ-2
```

Before deploying EKS, plan the networking properly.

A common production-style setup includes:

```text
2+ Availability Zones
Public subnets
Private subnets
```

The exact subnet selection for an AWS Load Balancer is determined by AWS networking configuration, subnet tagging, and the load-balancer controller/integration—not simply by Kubernetes automatically treating every `LoadBalancer` Service as "public."

---

# 27. Terraform vs Kubernetes Responsibilities

A useful separation:

```text
Terraform
   ↓
Create Infrastructure
   ↓
VPC
Subnets
EKS
IAM
Security Groups
etc.
```

Then:

```text
Kubernetes
   ↓
Deploy Applications
   ↓
Pods
Deployments
Services
Ingress
etc.
```

Example project flow:

```text
Terraform
   ↓
Create AWS Infrastructure
   ↓
Dockerfile
   ↓
Build Container Image
   ↓
CI/CD Pipeline
   ↓
Push Image
   ↓
Deploy to Kubernetes
   ↓
Service exposes Application
```

---

# 28. `kubectl describe svc`

```bash
kubectl describe svc <service-name>
```

Very useful for understanding what the Service is doing.

Example:

```bash
kubectl describe svc my-service
```

Important information includes:

```text
Selector
Port
TargetPort
NodePort
Endpoints
```

---

# 29. Service IP vs Pod IP vs Endpoints

Suppose:

```text
Service IP = 10.100.20.50
```

Pods:

```text
Pod 1 = 10.0.1.10
Pod 2 = 10.0.2.20
Pod 3 = 10.0.3.30
```

The Service maintains a set of eligible endpoints corresponding to the selected/ready Pods.

Conceptually:

```text
Service
10.100.20.50:80
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
Pod1  Pod2  Pod3
```

---

# 30. Endpoints

Check endpoints:

```bash
kubectl get endpoints
```

Specific Service:

```bash
kubectl get endpoints <service-name>
```

Example:

```bash
kubectl get endpoints my-service
```

If 4 eligible Pods are selected:

```text
my-service
   ↓
Endpoint 1
Endpoint 2
Endpoint 3
Endpoint 4
```

### Important debugging point

If the Service exists but has **no endpoints**, check the Service selector and Pod labels first.

---

# 31. Very Important Debugging Scenario

Question:

> Pods are running and Service is running, but I cannot access my application. How do you debug?

Use a logical sequence:

### Step 1 — Check Service

```bash
kubectl get svc
```

### Step 2 — Check Pods

```bash
kubectl get pods
```

### Step 3 — Check Pod labels

```bash
kubectl get pods --show-labels
```

### Step 4 — Check Service selector

```bash
kubectl describe svc <service-name>
```

### Step 5 — Check endpoints

```bash
kubectl get endpoints <service-name>
```

### Step 6 — Check Service ports

Verify:

```text
nodePort
port
targetPort
```

### Step 7 — Check application/listening port

Make sure the application is actually listening on the expected `targetPort`.

### Step 8 — For NodePort, check AWS Security Group/networking

Verify the required inbound traffic is allowed.

---

# 32. Most Important Debugging Rule

Remember:

```text
Service
   ↓
Selector
   ↓
Pod Labels
   ↓
Endpoints
   ↓
Pod
```

If labels don't match:

```text
Service selector ❌
       ↓
No matching Pods
       ↓
No endpoints
       ↓
Request cannot reach the Pods
```

---

# 33. Example of Wrong Selector

Pod:

```yaml
labels:
  app: my-pods
```

Service:

```yaml
selector:
  app: my-podsp
```

Notice:

```text
my-pods
   ≠
my-podsp
```

Therefore:

```text
Service
   ↓
No matching Pod
   ↓
Endpoints = none
```

Application won't receive traffic through that Service.

---

# 34. Fixing the Selector

Edit the Service:

```bash
kubectl edit svc <service-name>
```

Correct the selector.

Then check:

```bash
kubectl get endpoints <service-name>
```

Once the selector matches the Pod labels:

```text
Service
   ↓
Matching Pods
   ↓
Endpoints populated
   ↓
Traffic can reach Pods
```

---

# 35. Scaling Deployment Does Not Require Recreating Service

Suppose:

```yaml
replicas: 2
```

Service already exists.

Change:

```yaml
replicas: 4
```

Apply:

```bash
kubectl apply -f deploy.yml
```

You **do not need to recreate the Service**.

The existing Service automatically discovers the additional eligible Pods.

```text
Service
   ↓
Pod 1
Pod 2
Pod 3
Pod 4
```

---

# 36. One Service Can Expose Multiple Pods

Suppose:

```text
10 Pods
   ↓
same application
```

You don't need:

```text
10 Services
```

You can use:

```text
1 Service
   ↓
10 matching Pods
```

Example:

```text
             ┌── Pod 1
             ├── Pod 2
Service ─────┼── Pod 3
             ├── Pod 4
             └── Pod 5
```

The Service distributes traffic among its eligible endpoints according to the Kubernetes networking implementation.

---

# 37. Multiple Applications

Suppose:

```text
Application A → Pods
Application B → Pods
Application C → Pods
```

Normally each application has its own Service.

```text
Service A → Application A Pods

Service B → Application B Pods

Service C → Application C Pods
```

For HTTP/HTTPS applications, another common architecture is to use a single Load Balancer with **path-based or host-based routing** through an Ingress/Gateway layer.

---

# 38. Multiple Kubernetes Objects in One YAML

You can put multiple Kubernetes objects in one YAML file.

Use:

```yaml
---
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
...
---
apiVersion: v1
kind: Service
...
```

Then:

```bash
kubectl apply -f application.yml
```

Kubernetes processes the objects in the file.

You can also keep them in separate files:

```text
deploy.yml
svc.yml
```

Both approaches are valid.

---

# 39. Apply All YAML Files in Current Directory

```bash
kubectl apply -f .
```

The `.` means:

```text
current directory
```

Kubernetes processes the applicable manifest files in that directory.

---

# 40. ClusterIP Testing

Change:

```yaml
type: LoadBalancer
```

to:

```yaml
type: ClusterIP
```

Then:

```bash
kubectl apply -f svc.yml
```

Check:

```bash
kubectl get svc
```

The Service receives a ClusterIP.

---

# 41. How to Test ClusterIP

ClusterIP is intended for internal cluster communication.

Conceptually:

```text
Client inside cluster
       ↓
ClusterIP Service
       ↓
Pod
```

If your client machine is outside the cluster/network, you cannot simply open the ClusterIP from your laptop.

You can test from a location that has network access to the cluster, such as a worker node or a temporary test Pod, depending on your cluster setup.

Example from a worker node:

```bash
curl http://<pod-ip>
```

For Service testing, more appropriately test the **Service IP/DNS**:

```bash
curl http://<service-ip>:80
```

---

# 42. Namespaces

Check namespaces:

```bash
kubectl get ns
```

Namespaces provide logical separation within a Kubernetes cluster.

Example:

```text
default
kube-system
kube-public
kube-node-lease
```

You can create your own:

```bash
kubectl create ns veera
```

Check:

```bash
kubectl get ns
```

---

# 43. Deploy into a Specific Namespace

You can specify the namespace while applying:

```bash
kubectl apply -f dep.yml -n veera
```

Here:

```text
-n veera
```

means:

```text
use namespace = veera
```

---

# 44. Namespace in YAML

Instead of specifying `-n` every time, you can specify the namespace in metadata:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-deployment
  namespace: veera
```

Then:

```bash
kubectl apply -f dep.yml
```

The Deployment will be created in:

```text
veera
```

---

# 45. Creating Namespace Through YAML

You can also create a Namespace using YAML:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: nit
```

Apply:

```bash
kubectl apply -f namespace.yml
```

Then:

```bash
kubectl get ns
```

---

# 46. HPA — Introduction

Kubernetes also provides **Horizontal Pod Autoscaler (HPA)**.

Basic idea:

```text
Load increases
      ↓
Pod count increases
```

and:

```text
Load decreases
      ↓
Pod count decreases
```

Example:

```text
2 Pods
  ↓
High CPU/load
  ↓
4 Pods
  ↓
8 Pods
```

---

# 47. Node Autoscaling vs Pod Autoscaling

### HPA

```text
Application load increases
        ↓
Pod count increases
```

### Node Autoscaling

```text
Pods cannot be scheduled
        ↓
Pods become Pending
        ↓
More worker nodes required
        ↓
Node count increases
        ↓
Pending Pods can be scheduled
```

Simple comparison:

```text
HPA
→ scales Pods

Node Autoscaler
→ scales Worker Nodes
```

---

# 48. Complete Request Flow to Remember

## ClusterIP

```text
Internal Client
      ↓
ClusterIP Service
      ↓
Service Port
      ↓
Target Pod
      ↓
Application
```

## NodePort

```text
External User
      ↓
Node Public IP : NodePort
      ↓
Kubernetes Service
      ↓
Service Port
      ↓
Target Pod : targetPort
      ↓
Application
```

## LoadBalancer

```text
External User
      ↓
Cloud Load Balancer
      ↓
Kubernetes Service
      ↓
Eligible Pod/Endpoint
      ↓
Application
```

---

# 49. AWS ↔ Kubernetes Comparison

This is a very useful way to understand Kubernetes if you already know AWS.

### Traditional AWS approach

```text
User
 ↓
Load Balancer
 ↓
Target Group
 ↓
EC2 Instances
 ↓
Application
```

### Kubernetes approach

```text
User
 ↓
LoadBalancer Service
 ↓
Kubernetes networking
 ↓
Pod
 ↓
Application
```

Kubernetes abstracts much of the manual target registration and service discovery work.

---

# 50. Interview Important Points

For Kubernetes interviews, focus strongly on:

* Kubernetes architecture
* Control Plane
* Worker Nodes
* Pods
* Deployments
* ReplicaSets
* Services
* ClusterIP
* NodePort
* LoadBalancer
* Labels and Selectors
* Service endpoints
* `port`
* `targetPort`
* `nodePort`
* Service debugging
* How applications are exposed
* Namespace
* HPA
* Node Autoscaling

### Very important interview question:

> How does a Service know which Pods should receive traffic?

Answer:

```text
Service uses a selector.
The selector matches labels on Pods.
The matching/eligible Pods become Service endpoints.
Traffic is then forwarded to those endpoints.
```

### Another important question:

> Pods are running and Service is running, but application is not accessible. What do you check?

Think:

```text
Service
   ↓
Selector
   ↓
Pod Labels
   ↓
Endpoints
   ↓
Ports
   ↓
Pod/Application
   ↓
Network/Security Group
```

---

# 51. Day-5 Quick Revision

```text
Pod
 ↓
Application runs inside Pod
 ↓
Service provides stable access
```

```text
ClusterIP
→ Internal communication
```

```text
NodePort
→ NodeIP + NodePort
→ External/limited access
```

```text
LoadBalancer
→ Cloud Load Balancer
→ External access
```

```text
Service selector
        ↓
Pod labels
        ↓
Endpoints
        ↓
Traffic reaches Pod
```

### Three ports

```text
nodePort
   ↓
port
   ↓
targetPort
```

### Most important relationship

```text
spec.template.metadata.labels
              =
spec.selector
```

### Scaling

```text
HPA
→ Pod count

Node Autoscaler
→ Worker node count
```

### Namespaces

```text
kubectl get ns

kubectl create ns veera

kubectl apply -f dep.yml -n veera
```

### Multiple objects

```text
Object 1
---
Object 2
---
Object 3
```

### Most important debugging command

```bash
kubectl get endpoints <service-name>
```

If endpoints are empty, **first investigate why the Service is not selecting the expected Pods**.
