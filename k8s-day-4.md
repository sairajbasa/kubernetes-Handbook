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
