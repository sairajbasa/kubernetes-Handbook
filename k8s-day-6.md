Absolutely. I cleaned and organized your **Day 6 running notes** into structured Kubernetes notes while keeping the trainer's concepts, flow, examples, commands, and important interview/KT points.

# =========================

# DAY - 6

# Kubernetes Services, kube-proxy, Endpoints & HPA

# =========================

## 1. CREATE AN EKS CLUSTER

============================

Let's start by creating a Kubernetes cluster.

Basic approach:

1. Launch an EC2 instance
2. Become root

```bash
sudo su -
```

3. Install:

   * kubectl
   * eksctl
4. Attach the required IAM Role
5. Create the EKS cluster quickly using eksctl

### Important Learning Point

While learning Kubernetes, continuously track:

* What concept am I learning?
* Why is this component required?
* How does it work internally?
* How does one Kubernetes component communicate with another?

Otherwise, Kubernetes concepts can become confusing.

> Terraform-based EKS cluster creation will be covered in upcoming classes.

# 2. CONNECT TO THE EKS CLUSTER

===============================

After creating the cluster:

```bash
kubectl get nodes
```

If kubectl is unable to connect to the cluster, update the kubeconfig:

```bash
aws eks update-kubeconfig \
--region <region-name> \
--name <cluster-name>
```

Then:

```bash
kubectl get nodes
```

This configures kubectl to communicate with the EKS cluster.

# 3. KUBERNETES SERVICE – BASIC CONCEPT

========================================

Let's understand how a Kubernetes Service works internally.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 1
```

And a Service:

```yaml
apiVersion: v1
kind: Service
```

Apply the configuration:

```bash
kubectl apply -f deploy.yml
```

Check the pods:

```bash
kubectl get pods
```

Check services:

```bash
kubectl get svc
```

### VERY IMPORTANT

The Service selector labels must match the Pod labels.

Example:

Pod:

```yaml
labels:
  app: nginx
```

Service:

```yaml
selector:
  app: nginx
```

If the labels do not match:

```text
Service
   |
   X
   |
Pod
```

The Service will not have the expected Pod endpoints.

# 4. DEFAULT SERVICE

====================

Check Services:

```bash
kubectl get svc
```

A ClusterIP Service is used for internal Kubernetes communication.

ClusterIP means:

```text
Pod/Application
      |
      v
 ClusterIP Service
      |
      v
 Other Pod/Application
```

It is primarily used for communication inside the cluster.

# 5. ENDPOINTS

==============

To check the endpoints associated with Services:

```bash
kubectl get endpoints
```

A Service needs to know:

```text
Which Pods should receive traffic?
```

The endpoint information represents the Pod IPs that are currently associated with the Service.

# 6. SERVICE INFORMATION AND ETCD

==================================

When a Service is created:

```bash
kubectl apply -f demo-svc.yml
```

Kubernetes stores cluster state information in:

```text
ETCD
```

The important idea is:

```text
kubectl
   |
   v
API Server
   |
   v
ETCD
```

The API Server is the main communication point for Kubernetes components.

> Don't think of the client directly modifying ETCD. Kubernetes components communicate through the API Server, while ETCD stores the cluster state.

# 7. KUBE-PROXY

===============

## VERY IMPORTANT COMPONENT

One of the most important components for understanding Kubernetes Services is:

```text
kube-proxy
```

kube-proxy runs on the worker nodes.

If there are:

```text
2 Worker Nodes
```

then kube-proxy runs on both nodes.

Architecture:

```text
Worker Node 1
 └── kube-proxy

Worker Node 2
 └── kube-proxy
```

kube-proxy continuously watches the Kubernetes API Server for relevant changes.

# 8. WHAT HAPPENS WHEN A POD IS CREATED?

========================================

Suppose we create a Pod.

Basic flow:

```text
Pod created
     |
     v
Kubelet
     |
     v
API Server
     |
     v
ETCD
```

The Kubernetes control-plane components maintain the desired/current cluster state.

If the Pod belongs to a Service, its IP needs to become an endpoint for that Service.

# 9. WHAT HAPPENS WHEN A POD IS DELETED?

========================================

Suppose:

```text
Service
   |
   +----> Pod-1
```

Pod-1 gets deleted.

The Service should NOT continue sending traffic to the old Pod IP.

Otherwise:

```text
Client
  |
  v
Service
  |
  v
Old Pod IP
  |
  X
Pod no longer exists
```

The application will fail to respond correctly.

Therefore, the old Pod must be removed from the Service endpoints.

# 10. POD DELETION FLOW

=======================

When something happens to a Pod on a worker node, kubelet is an important component involved in reporting the state.

Simplified flow:

```text
Pod deleted
     |
     v
Kubelet
     |
     v
API Server
     |
     +----> ETCD
     |
     +----> Other Kubernetes components
     |
     v
Endpoint information updated
     |
     v
kube-proxy receives the change
     |
     v
kube-proxy updates networking rules
```

The important concept:

```text
Pod changes
     ↓
Endpoint changes
     ↓
kube-proxy reacts
     ↓
Networking rules are updated
```

# 11. ENDPOINT CONTROLLER / ENDPOINTSLICE CONTROLLER

====================================================

Kubernetes has an Endpoint Controller / EndpointSlice Controller responsible for continuously watching relevant:

* Pods
* Services

Its job is to maintain the relationship between:

```text
Service
   |
   v
Healthy/Matching Pods
```

When the matching Pods change, endpoint information changes.

# 12. KUBEPROXY + ENDPOINTSLICE

===============================

Every kube-proxy watches the API Server.

When EndpointSlice information changes:

```text
EndpointSlice changed
        |
        v
API Server
        |
        v
kube-proxy
        |
        v
Update networking rules
```

This allows Service traffic to be directed toward the currently valid Pod endpoints.

# 13. NODEPORT – MAIN CONCEPT

=============================

Now comes one of the most important concepts from today's class.

Suppose:

```text
Worker Node 1
Worker Node 2
```

And the Pod is running only on:

```text
Worker Node 2
```

If the Service is a NodePort Service, you can access it using:

```text
Node-1-IP:<NodePort>
```

even though the Pod is running on Node 2.

Example:

```text
Node 1
Public IP: X.X.X.X
       |
       | :30007
       v
   NodePort
       |
       v
   Node 2
       |
       v
     Pod
```

This is the important Service networking behavior.

# 14. WHY CAN WE ACCESS THE POD THROUGH ANOTHER NODE?

======================================================

The main component involved here is:

```text
kube-proxy
```

kube-proxy runs on every worker node.

For example:

```text
Node 1
 └── kube-proxy

Node 2
 └── kube-proxy
```

When the NodePort Service is created, the NodePort is exposed at the cluster/node level rather than only on the node where the Pod happens to be running.

Therefore:

```text
Node 1 IP + NodePort
```

can ultimately reach a Pod running on:

```text
Node 2
```

# 15. IPTABLES

==============

kube-proxy manages networking rules on Linux nodes.

One important mechanism discussed here is:

```text
iptables
```

The simplified flow is:

```text
Client
  |
  v
Node 1 IP : NodePort
  |
  v
kube-proxy-managed rules
  |
  v
Pod endpoint
```

If the selected Pod is on another node:

```text
Node 1
  |
  | kube-proxy networking rules
  v
Node 2
  |
  v
Pod
```

So Node-to-Node traffic can occur internally through the networking rules configured by kube-proxy.

# 16. NODEPORT EXISTS ON ALL WORKER NODES

==========================================

Suppose the cluster has:

```text
10 Worker Nodes
```

and only:

```text
1 Pod
```

is running on Worker Node 5.

The NodePort is still available on the worker nodes.

Conceptually:

```text
Node 1  → NodePort
Node 2  → NodePort
Node 3  → NodePort
Node 4  → NodePort
Node 5  → NodePort → Pod
Node 6  → NodePort
...
Node 10 → NodePort
```

Even a node that currently has zero Pods can listen for the NodePort traffic.

# 17. CHECK NODE INFORMATION

============================

Check nodes:

```bash
kubectl get nodes
```

Get additional information:

```bash
kubectl get nodes -o wide
```

This helps identify:

* Node IP
* Internal IP
* External/Public IP where applicable
* Node details

# 18. NODEPORT ACCESS EXAMPLE

=============================

Suppose:

```text
Node 1 Public IP = 10.10.10.10
Node 2 Public IP = 20.20.20.20

NodePort = 30007
```

Pod is running on Node 2.

You can try:

```text
http://10.10.10.10:30007
```

and:

```text
http://20.20.20.20:30007
```

The request can reach the Service/POD even when the Pod itself is running only on one node.

This is one of the important points to understand about Kubernetes Service networking.

# 19. SECURITY GROUP

====================

When using EKS worker nodes, make sure the required inbound traffic is permitted by the relevant security configuration.

For lab purposes, you may add the required inbound rule.

Example:

```text
NodePort
30007
```

Then access:

```text
http://<node-public-ip>:30007
```

In production, do NOT blindly allow all traffic. Use the minimum required ports and sources.

# 20. SCALING THE DEPLOYMENT

============================

Initially:

```yaml
replicas: 1
```

Check:

```bash
kubectl get pods
```

Now change:

```yaml
replicas: 2
```

Apply:

```bash
kubectl apply -f deploy.yml
```

Check:

```bash
kubectl get deploy
kubectl get pods
kubectl get endpoints
```

Now there should be:

```text
2 Pods
2 Service endpoints
```

Conceptually:

```text
Service
  |
  +----> Pod 1
  |
  +----> Pod 2
```

# 21. REDUCING REPLICAS

=======================

Edit the Deployment:

```bash
kubectl edit deploy my-deployment-np
```

Change:

```yaml
replicas: 1
```

One Pod will be removed.

Check:

```bash
kubectl get endpoints
```

Now only the remaining Pod should be represented as a Service endpoint.

This demonstrates:

```text
Pod count changes
      ↓
Endpoint count changes
      ↓
kube-proxy/networking information updates
```

# 22. OPENSHIFT

===============

An alternative Kubernetes-based platform is:

```text
OpenShift
```

Important points:

* OpenShift is built on Kubernetes.
* Many Kubernetes concepts are similar.
* OpenShift adds features and tooling to simplify certain enterprise Kubernetes use cases.
* Kubernetes commonly uses:

```bash
kubectl
```

OpenShift commonly uses:

```bash
oc
```

Conceptual comparison:

```text
Kubernetes → kubectl

OpenShift  → oc
```

Similar idea:

```text
Terraform
   |
   +---- OpenTofu
```

The concepts overlap significantly, but the tools/platform features differ.

# 23. IMPORTANT KUBERNETES SERVICE FLOW

========================================

Remember this simplified flow:

```text
                 API SERVER
                     |
          +----------+----------+
          |                     |
         ETCD              Controllers
          |                     |
          |                EndpointSlice
          |                 information
          |                     |
          +----------+----------+
                     |
                kube-proxy
                     |
               iptables/rules
                     |
                  Service
                     |
                     v
                   Pod
```

The key point:

```text
API Server
   ↓
Cluster state / endpoint changes
   ↓
kube-proxy
   ↓
Networking rules
   ↓
Service traffic
   ↓
Pod
```

# 24. DIFFERENT APPLICATIONS AND PORTS

=======================================

A machine cannot normally have different applications independently listening on the exact same IP address and port combination.

For example:

```text
Application A → 8080
Application B → 8080
```

on the same IP and protocol cannot both simply bind to the same port.

Kubernetes uses its networking/service mechanisms to provide different ways of exposing applications.

# =========================================================

# HORIZONTAL POD AUTOSCALER (HPA)

# =========================================================

# 25. WHAT IS HPA?

==================

HPA stands for:

```text
Horizontal Pod Autoscaler
```

Its responsibility is to automatically increase or decrease the number of Pods based on a metric such as CPU utilization.

Conceptually:

```text
Low Load
   ↓
1 Pod

High Load
   ↓
Multiple Pods
```

This is similar in concept to:

```text
AWS Auto Scaling Group
```

but HPA operates at the Kubernetes Pod level.

# 26. WHY DO WE NEED HPA?

=========================

A Deployment with:

```yaml
replicas: 1
```

is static.

The Deployment controller knows:

```text
Desired replicas = 1
```

It does NOT automatically decide:

```text
Traffic increased
→ create more Pods
```

unless an autoscaling mechanism such as HPA is configured.

HPA provides that additional intelligence.

# 27. HPA WORKING

=================

Simplified architecture:

```text
                 Metrics
                    |
                    v
                  HPA
                    |
             Increase/Decrease
                replicas
                    |
                    v
          Deployment Controller
                    |
                    v
                  Pods
```

Example:

```text
CPU increases
     |
     v
HPA detects increased utilization
     |
     v
HPA changes desired replica count
     |
     v
Deployment Controller
     |
     v
Creates additional Pods
```

# 28. HPA SPECIFICATION

=======================

Example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
```

The important HPA settings include:

```text
minReplicas
maxReplicas
scaleTargetRef
resource
target
```

Example concept:

```yaml
minReplicas: 1
maxReplicas: 10
```

Meaning:

```text
Minimum Pods = 1
Maximum Pods = 10
```

# 29. SCALE TARGET REF

======================

HPA needs to know:

```text
Which Deployment should I scale?
```

This is specified through:

```yaml
scaleTargetRef:
```

It points to the target Deployment.

The Deployment's:

```yaml
metadata:
  name:
```

is important.

Example:

```yaml
metadata:
  name: my-deployment
```

HPA can target:

```text
my-deployment
```

# 30. WHY NAMES ARE IMPORTANT

=============================

Kubernetes resources use names for identification.

Example:

```text
Deployment
Name: my-deployment
```

HPA:

```text
Target: my-deployment
```

The HPA must know exactly which Deployment it should control.

Names are important because Kubernetes resources communicate with and reference each other using resource identities, names, labels, and selectors depending on the relationship.

# 31. CPU UTILIZATION

=====================

Example HPA target:

```yaml
resource:
  name: cpu
  target:
    type: Utilization
    averageUtilization: 1
```

The important question is:

```text
1% of what?
```

The reference point comes from the Pod's CPU resource request.

Therefore, CPU requests should be defined in the Deployment.

# 32. CPU REQUESTS

==================

Example:

```yaml
resources:
  requests:
    cpu: "250m"
```

Here:

```text
250m = 250 millicores
```

Conceptually:

```text
1 CPU = 1000m

250m = 0.25 CPU
```

If HPA uses CPU utilization, the CPU request provides the reference value.

# 33. UNDERSTANDING 1% UTILIZATION

==================================

Suppose:

```text
CPU request = 250m
```

If the HPA target is:

```text
1%
```

then conceptually:

```text
250m × 1%
= 2.5m
```

So approximately:

```text
2.5m CPU usage
```

would represent 1% of a 250m CPU request.

> The exact HPA behavior also depends on Kubernetes metrics availability and how the metric is reported.

# 34. WHY CPU REQUESTS MATTER

=============================

If you do not define CPU requests, there is no useful request-based reference point for CPU utilization-based HPA behavior.

Therefore, define resource requests:

```yaml
resources:
  requests:
    cpu: "250m"
```

This gives Kubernetes a reference point for resource management and CPU utilization calculations.

# 35. CLEANING THE PREVIOUS LAB

===============================

To delete resources created in the current directory:

```bash
kubectl delete -f .
```

Then, if required:

```bash
rm -rf *
```

Be careful with:

```bash
rm -rf *
```

because it permanently deletes files in the current directory.

# 36. CREATE THE DEPLOYMENT

===========================

Create:

```bash
vi deploy.yml
```

Initially, the image may have been incorrect.

If you see:

```text
ErrImagePull
```

or:

```text
ImagePullBackOff
```

check the image name.

For example:

```yaml
image: nginx
```

Docker will use the appropriate default/latest tag when no explicit tag is provided.

# 37. APPLY THE DEPLOYMENT

==========================

```bash
kubectl apply -f deploy.yml
```

Check:

```bash
kubectl get pods
```

At this stage you can have:

```text
1 Pod
```

and no Service yet.

# 38. SERVICE LABEL MATCHING

============================

When creating the Service, remember:

```text
Service selector
        ↓
must match
        ↓
Pod labels
```

Example:

```yaml
Pod:
  labels:
    app: nginx
```

Service:

```yaml
selector:
  app: nginx
```

If they do not match:

```text
Service → No correct endpoints
```

Therefore always verify labels/selectors.

# 39. LOADBALANCER SERVICE

==========================

Create:

```bash
vi svc.yml
```

Example Service type:

```yaml
type: LoadBalancer
```

Apply:

```bash
kubectl apply -f svc.yml
```

Check:

```bash
kubectl get svc
```

Kubernetes will provision/expose a load balancer through the cloud provider integration in a managed cloud environment such as EKS.

# 40. CREATE HPA

===============

Create:

```bash
vi hpa.yml
```

The HPA should target the correct Deployment name.

Apply:

```bash
kubectl apply -f hpa.yml
```

Check:

```bash
kubectl get hpa
```

You may initially see:

```text
CPU: <unknown>/1%
```

This can happen when the required metrics are not yet available.

You should investigate metrics availability rather than assuming the HPA itself is broken.

# 41. GENERATE LOAD

===================

To test HPA, generate traffic against the application's endpoint.

For example:

```text
Load Balancer DNS
```

can be used in a test shell script.

Example concept:

```bash
vi test.sh
```

The script repeatedly sends requests to the Load Balancer endpoint.

You can also use an appropriate stress/load-generation tool in a lab environment.

# 42. WATCH HPA

==============

Use:

```bash
kubectl get hpa
```

Also check:

```bash
kubectl top pods
```

And:

```bash
kubectl get pods
```

You are looking for:

```text
CPU utilization increases
        ↓
HPA detects the load
        ↓
Desired replicas increase
        ↓
Deployment Controller creates Pods
        ↓
Pod count increases
```

# 43. HPA RESPONSIBILITY

========================

Remember:

```text
HPA
 |
 +----> increases/decreases Pod replicas
```

HPA does NOT directly create EC2 worker nodes.

The Deployment Controller is responsible for maintaining the requested number of Pods.

HPA changes the desired replica count.

# 44. NODE AUTOSCALER

=====================

Now consider a different situation.

Suppose HPA increases the number of Pods:

```text
1 Pod
 ↓
5 Pods
 ↓
10 Pods
```

But the worker nodes don't have enough resources.

Some Pods may become:

```text
Pending
```

because Kubernetes cannot schedule them on the existing nodes.

# 45. NODE AUTOSCALER RESPONSIBILITY

====================================

A Node Autoscaler is responsible for increasing worker-node capacity when additional nodes are required.

Conceptually:

```text
High application load
       |
       v
HPA increases Pods
       |
       v
Not enough node capacity
       |
       v
Pods become Pending
       |
       v
Node Autoscaler
       |
       v
Additional worker node
       |
       v
Pending Pods get scheduled
```

# 46. HPA VS NODE AUTOSCALER

============================

| Component                | Responsibility                                              |
| ------------------------ | ----------------------------------------------------------- |
| Deployment               | Maintains desired Pod count                                 |
| HPA                      | Increases/decreases Pod count based on metrics              |
| Node Autoscaler          | Increases/decreases worker-node capacity                    |
| kubelet                  | Manages Pods on its worker node and reports node/Pod status |
| kube-proxy               | Implements Service networking rules on nodes                |
| EndpointSlice Controller | Maintains Service-to-Pod endpoint information               |
| API Server               | Main communication interface for Kubernetes components      |
| ETCD                     | Stores Kubernetes cluster state                             |

### Easy memory trick:

```text
HPA = More/Fewer Pods

Node Autoscaler = More/Fewer Nodes
```

And:

```text
HPA → Pod scaling

Node Autoscaler → Node scaling
```

# 47. IMPORTANT END-TO-END FLOW

===============================

The complete scenario can be remembered like this:

```text
User Traffic
     |
     v
Service / LoadBalancer
     |
     v
Kubernetes Service
     |
     v
Endpoint / EndpointSlice
     |
     v
Pod
```

When traffic increases:

```text
Traffic increases
       |
       v
CPU utilization increases
       |
       v
HPA detects metric
       |
       v
HPA increases replicas
       |
       v
Deployment Controller
       |
       v
More Pods
```

If nodes don't have enough capacity:

```text
More Pods
    |
    v
Pods Pending
    |
    v
Node Autoscaler
    |
    v
More Worker Nodes
    |
    v
Pods scheduled
```

# 48. MOST IMPORTANT DAY-6 CONCEPT

===================================

### Service + kube-proxy + Endpoints

Remember:

```text
Service
   |
   v
EndpointSlice / Endpoints
   |
   v
Pod IPs
```

When Pods change:

```text
Pod created/deleted
       |
       v
Endpoint information changes
       |
       v
API Server
       |
       v
kube-proxy watches the change
       |
       v
Networking rules updated
       |
       v
Service traffic reaches valid Pods
```

For NodePort:

```text
Client
  |
  v
Any Worker Node IP : NodePort
  |
  v
kube-proxy networking rules
  |
  v
Available Pod endpoint
```

Therefore:

> **The Pod does NOT need to be running on the same node whose IP the client is accessing.**

# 49. IMPORTANT COMMANDS FROM DAY 6

====================================

```bash
# Configure EKS access
aws eks update-kubeconfig --region <region> --name <cluster>

# Check nodes
kubectl get nodes

# Detailed node information
kubectl get nodes -o wide

# Check Pods
kubectl get pods

# Check Deployments
kubectl get deploy

# Check Services
kubectl get svc

# Check endpoints
kubectl get endpoints

# Check HPA
kubectl get hpa

# Check Pod resource usage
kubectl top pods

# Apply YAML
kubectl apply -f deploy.yml

# Delete resources
kubectl delete -f .

# Edit Deployment
kubectl edit deploy <deployment-name>
```

# 50. DAY-6 INTERVIEW / KT POINTS

==================================

1. **What is kube-proxy?**
   A node-level Kubernetes component involved in implementing Service networking.

2. **Where does kube-proxy run?**
   On worker nodes.

3. **What does kube-proxy watch?**
   It watches relevant Kubernetes state through the API Server.

4. **What happens when a Pod is deleted?**
   The Service endpoint information is updated so traffic is not directed to the deleted Pod.

5. **What is an endpoint?**
   An address representing a backend Pod that can receive Service traffic.

6. **What is EndpointSlice?**
   A Kubernetes mechanism for representing groups of Service endpoints.

7. **Why must Service selectors match Pod labels?**
   So Kubernetes can identify the correct Pods for the Service.

8. **Can NodePort work when the Pod is running on another node?**
   Yes.

9. **Why?**
   NodePort is exposed on worker nodes and kube-proxy implements the required networking behavior.

10. **Does kube-proxy run only on the node where the Pod exists?**
    No. kube-proxy runs on the worker nodes.

11. **What is iptables?**
    Linux networking/firewall functionality that can be used for packet filtering and routing rules.

12. **What is HPA?**
    Horizontal Pod Autoscaler.

13. **What does HPA scale?**
    Pods/Deployment replicas.

14. **What does Node Autoscaler scale?**
    Worker-node capacity.

15. **What happens if HPA creates more Pods but nodes have insufficient resources?**
    Pods can remain Pending until sufficient node capacity becomes available.

16. **Why are CPU requests important for CPU-based HPA?**
    CPU utilization is evaluated relative to the configured CPU request.

17. **What does `250m` CPU mean?**
    250 millicores, or 0.25 CPU.

18. **How does HPA know which Deployment to scale?**
    Through `scaleTargetRef`, which identifies the target resource.

19. **Where is Kubernetes cluster state stored?**
    ETCD.

20. **What is the simple difference between HPA and Node Autoscaler?**

```text
HPA
↓
Scale Pods

Node Autoscaler
↓
Scale Nodes
```

# 51. DAY-6 FINAL MEMORY MAP

============================

```text
                  KUBERNETES
                      |
          +-----------+-----------+
          |                       |
       SERVICE                   HPA
          |                       |
          v                       v
      Endpoints              Pod Scaling
          |
          v
    EndpointSlice
          |
          v
     kube-proxy
          |
          v
      iptables
          |
          v
       Pod / Node
```

For scaling:

```text
Traffic
   ↓
CPU increases
   ↓
HPA
   ↓
More Pods
   ↓
If capacity is insufficient
   ↓
Pods Pending
   ↓
Node Autoscaler
   ↓
More Nodes
   ↓
Pods scheduled
```

### ONE-LINE REVISION

```text
Service finds Pods through selectors/endpoints,
kube-proxy implements Service networking on nodes,
HPA scales Pods based on metrics,
and Node Autoscaler provides additional worker-node capacity.
```

# =========================================================

# DAY-6 PRACTICE GOAL

# =========================================================

Practice the complete flow yourself:

```text
1. Create EKS cluster
2. Connect using kubectl
3. Create Deployment
4. Create Service
5. Check endpoints
6. Increase replicas
7. Check endpoints again
8. Reduce replicas
9. Check endpoints again
10. Create NodePort/LoadBalancer Service
11. Access through different worker-node IPs
12. Understand kube-proxy behavior
13. Create HPA
14. Configure CPU requests
15. Generate application load
16. Monitor HPA
17. Watch Pod count increase
18. Understand Pending Pods
19. Understand Node Autoscaler
```

> **Day-6 main focus:** Don't just memorize the commands. Understand the internal flow of **Service → EndpointSlice → kube-proxy → networking rules → Pod**, and the scaling flow of **HPA → Pods → Node Autoscaler → Nodes**.
