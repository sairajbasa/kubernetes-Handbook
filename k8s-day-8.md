Let’s rebuild Day 8 as one connected story, from **Git → CI → Image → GitOps → Argo CD → Kubernetes**, and then understand **Auto-Sync, Prune, Self-Heal, OutOfSync, Rollback, and private repositories** with practical examples.

# DAY 8 — Kubernetes + Argo CD GitOps

## 1. First understand the big picture

The most important thing from today's class is this:

> **Git is the Source of Truth. Argo CD continuously makes the Kubernetes cluster match what is declared in Git.**

Think of the architecture like this:

```text
Developer
   |
   | 1. Push application code
   ↓
GitHub
   |
   ↓
CI Pipeline
(Jenkins / GitHub Actions / GitLab CI)
   |
   | 2. Build Docker image
   ↓
Docker Image
   |
   | 3. Push image
   ↓
Amazon ECR
```

Then comes the **CD/GitOps part**:

```text
GitHub
   |
   | Kubernetes YAML / Helm
   | Deployment
   | Service
   | Ingress
   ↓
Argo CD
   |
   | Compare Git vs Cluster
   |
   | Synchronize desired state
   ↓
Kubernetes Cluster
   |
   ├── Deployment
   ├── Pods
   ├── Service
   └── Other resources
```

### Very important distinction

There are actually **two different things in Git** that we need to think about:

```text
Application Source Code Repository
        |
        | Dockerfile
        ↓
       CI
        |
        ↓
      ECR
```

and:

```text
Kubernetes Manifest / GitOps Repository
        |
        | deployment.yaml
        | service.yaml
        | ingress.yaml
        ↓
     Argo CD
        |
        ↓
   Kubernetes Cluster
```

Sometimes companies keep both in one repository; sometimes they use separate repositories.

---

# 2. What does CI do?

Suppose your developer changes application code.

```text
developer changes code
        ↓
       GitHub
        ↓
      Jenkins
        ↓
    Docker build
        ↓
 Docker image created
        ↓
       ECR
```

For example:

```text
myapp:v1
```

gets created and pushed to:

```text
123456789.dkr.ecr.ap-south-1.amazonaws.com/myapp:v1
```

CI's primary responsibility is:

> **Build, test, scan and package the application, then publish the artifact/image.**

Examples of CI tools:

* Jenkins
* GitHub Actions
* GitLab CI/CD

---

# 3. What does CD do?

Now Kubernetes needs to know:

> "Which image should I deploy?"

For example:

```yaml
containers:
  - name: myapp
    image: 123456789.dkr.ecr.ap-south-1.amazonaws.com/myapp:v1
```

Suppose CI creates:

```text
myapp:v2
```

The Kubernetes YAML needs to change:

```yaml
image: myapp:v1
```

to:

```yaml
image: myapp:v2
```

That updated YAML is committed to Git.

Then:

```text
GitHub
   ↓
Argo CD detects change
   ↓
Argo CD compares Git with cluster
   ↓
Argo CD synchronizes
   ↓
Kubernetes Deployment updated
   ↓
New Pods created
   ↓
Pods pull myapp:v2 from ECR
```

That is the **GitOps CD flow**.

---

# 4. Why Argo CD?

Argo CD is designed specifically for Kubernetes GitOps.

The basic idea is:

```text
Git = Desired State
Kubernetes = Actual State
Argo CD = Reconciliation mechanism
```

For example, Git says:

```yaml
replicas: 3
```

But Kubernetes currently has:

```text
replicas: 2
```

Argo CD notices:

```text
Desired State = 3
Actual State  = 2

Difference detected
        ↓
     OutOfSync
```

If automatic synchronization is enabled:

```text
Argo CD
   ↓
Sync
   ↓
Kubernetes
   ↓
3 replicas
```

Now:

```text
Desired State = 3
Actual State  = 3

        ↓
      Synced
```

---

# 5. The most important concept: Desired State vs Actual State

This will make the rest of Argo CD much easier.

### Git says:

```text
Deployment
replicas = 3
```

This is the:

> **Desired State**

### Kubernetes currently has:

```text
Deployment
replicas = 2
```

This is:

> **Actual State**

Argo CD continuously compares them.

```text
             GitHub
          Desired State
               |
               |
               ↓
           Argo CD
               |
        Compare states
               |
               ↓
       Kubernetes Cluster
          Actual State
```

If they are different:

```text
OutOfSync
```

If they match:

```text
Synced
```

---

# 6. Argo CD Sync

**Sync means:**

> Make the Kubernetes cluster match the configuration stored in Git.

Example:

Git:

```yaml
replicas: 5
```

Cluster:

```text
replicas: 3
```

Argo CD:

```text
OutOfSync
```

You click:

```text
SYNC
```

Argo CD applies the Git configuration.

Cluster becomes:

```text
replicas: 5
```

Now:

```text
Synced
```

---

# 7. Auto-Sync

Normally you could manually click:

```text
SYNC
```

But with:

```text
Enable Auto-Sync
```

Argo CD automatically synchronizes changes from Git.

Example:

### Before

Git:

```yaml
replicas: 2
```

Cluster:

```text
2 Pods
```

Everything is:

```text
Synced
```

---

Developer changes Git:

```yaml
replicas: 3
```

and pushes:

```text
git push
```

Argo CD detects the Git change.

```text
Git
replicas: 3
       ↓
Argo CD
       ↓
Auto Sync
       ↓
Kubernetes
       ↓
3 Pods
```

You don't have to manually click Sync.

### Simple definition

> **Auto-Sync automatically applies Git changes to the Kubernetes cluster.**

---

# 8. Self-Heal

This is different from Auto-Sync.

Self-Heal deals with:

> **Someone manually changing Kubernetes resources directly in the cluster.**

Suppose Git says:

```yaml
replicas: 2
```

Cluster:

```text
2 Pods
```

Everything is fine.

Now someone runs:

```bash
kubectl edit deployment myapp
```

and changes:

```yaml
replicas: 2
```

to:

```yaml
replicas: 5
```

Kubernetes now has:

```text
5 Pods
```

But Git still says:

```text
2 Pods
```

Therefore:

```text
Git = 2
Cluster = 5

       ↓
   OutOfSync
```

If **Self-Heal is enabled**, Argo CD detects the drift and restores the cluster to Git.

```text
Git
replicas = 2
     |
     ↓
Argo CD
     |
     | detects manual drift
     ↓
Kubernetes
replicas = 5
     |
     ↓
Argo CD changes it back
     ↓
replicas = 2
```

### Simple definition

> **Self-Heal automatically corrects manual changes made directly in the Kubernetes cluster.**

---

# 9. Auto-Sync vs Self-Heal

This is one of the most important interview concepts.

Think about **where the change originated**.

### Scenario 1 — Change happens in Git

```text
Developer
   ↓
Git
   ↓
Argo CD
   ↓
Kubernetes
```

This is handled by:

> **Auto-Sync**

---

### Scenario 2 — Change happens manually in Kubernetes

```text
Engineer
   ↓
kubectl edit
   ↓
Kubernetes
```

Now Kubernetes differs from Git.

This is handled by:

> **Self-Heal**

---

# 10. Why do we need both?

Imagine:

```text
Git = replicas: 2
Cluster = replicas: 2
```

Now someone manually changes:

```text
Cluster = replicas: 5
```

Argo CD detects:

```text
OutOfSync
```

If:

```text
Self-Heal = OFF
```

Argo CD won't automatically fix that manual change.

You may need to synchronize.

If:

```text
Self-Heal = ON
```

Argo CD automatically restores:

```text
5 → 2
```

---

# 11. Prune

Prune is another concept and it is very easy to confuse with Self-Heal.

### Suppose Git contains:

```text
deployment.yaml
service.yaml
configmap.yaml
```

Therefore Kubernetes has:

```text
Deployment
Service
ConfigMap
```

Everything matches.

Now you delete:

```text
configmap.yaml
```

from Git.

Git now contains:

```text
deployment.yaml
service.yaml
```

But Kubernetes still has:

```text
Deployment
Service
ConfigMap
```

So:

```text
Git:
Deployment
Service

Cluster:
Deployment
Service
ConfigMap
```

There is an extra resource in the cluster.

---

## Prune OFF

If:

```text
Prune = OFF
```

Argo CD does **not automatically delete** that extra ConfigMap.

So:

```text
Git                  Cluster

Deployment  ───────→ Deployment
Service     ───────→ Service
                         |
                         └── ConfigMap remains
```

---

## Prune ON

If:

```text
Prune = ON
```

Argo CD sees:

```text
ConfigMap exists in cluster
BUT
ConfigMap no longer exists in Git
```

Therefore:

```text
Argo CD
   ↓
Prune
   ↓
Delete ConfigMap
```

Now cluster matches Git.

---

# 12. The easiest way to remember Prune

Think:

> **"If I remove something from Git, should Argo CD remove the corresponding Kubernetes resource?"**

If:

```text
Prune ON
```

➡️ Yes.

If:

```text
Prune OFF
```

➡️ No.

---

# 13. Self-Heal vs Prune

This distinction is extremely important.

| Feature   | Problem it solves                |
| --------- | -------------------------------- |
| Auto-Sync | Git changed                      |
| Self-Heal | Someone manually changed cluster |
| Prune     | Resource was removed from Git    |

### Example

Git:

```text
Deployment
Service
ConfigMap
```

Cluster:

```text
Deployment
Service
ConfigMap
```

---

### Case A

Developer changes:

```yaml
replicas: 2
```

to:

```yaml
replicas: 4
```

➡️ **Auto-Sync**

---

### Case B

Someone runs:

```bash
kubectl edit deployment
```

and changes:

```text
2 → 5
```

➡️ **Self-Heal**

---

### Case C

Developer deletes:

```text
configmap.yaml
```

from Git.

➡️ **Prune**

---

# 14. Three Sync Policy options together

Your trainer emphasized these three:

```text
1. Enable Auto-Sync
2. Prune Resources
3. Self-Heal
```

Think of them as three separate questions.

### Auto-Sync

> "When Git changes, should Argo CD automatically apply the change?"

```text
Git → Cluster
```

### Self-Heal

> "When someone manually changes the cluster, should Argo CD restore Git state?"

```text
Cluster manual change → Git state
```

### Prune

> "When something is deleted from Git, should Argo CD delete it from the cluster?"

```text
Deleted from Git → Delete from Cluster
```

---

# 15. Recommended practical lab

Create:

```text
deployment.yaml
service.yaml
configmap.yaml
```

Git:

```text
3 resources
```

Cluster:

```text
3 resources
```

Then test each feature independently.

### Lab 1 — Auto-Sync

Git:

```yaml
replicas: 2
```

Change to:

```yaml
replicas: 3
```

Commit and push.

Observe:

```text
Git change
   ↓
Argo CD
   ↓
Auto Sync
   ↓
3 Pods
```

---

### Lab 2 — Self-Heal

Git:

```text
replicas = 3
```

Cluster:

```text
3
```

Run:

```bash
kubectl scale deployment myapp --replicas=5
```

Observe:

```text
Git = 3
Cluster = 5
       ↓
OutOfSync
       ↓
Self-Heal
       ↓
Cluster = 3
```

---

### Lab 3 — Prune

Git:

```text
deployment.yaml
service.yaml
configmap.yaml
```

Delete:

```text
configmap.yaml
```

Commit and push.

With:

```text
Prune OFF
```

ConfigMap remains.

Then enable:

```text
Prune ON
```

and sync.

ConfigMap gets deleted.

---

# 16. What does OutOfSync actually mean?

**OutOfSync does NOT simply mean "deployment failed."**

It means:

> **The desired state in Git and the actual state in Kubernetes are different.**

For example:

```text
Git:
replicas = 3

Cluster:
replicas = 2
```

➡️ OutOfSync.

Or:

```text
Git:
ConfigMap exists

Cluster:
ConfigMap doesn't exist
```

➡️ OutOfSync.

Or:

```text
Git:
Service type = ClusterIP

Cluster:
Service type = NodePort
```

➡️ OutOfSync.

---

# 17. Synced

Synced means:

```text
Desired State == Actual State
```

Example:

```text
Git:
replicas = 3

Cluster:
replicas = 3
```

Therefore:

```text
Synced
```

---

# 18. Rollback

Now imagine your application has versions:

```text
Commit A
   ↓
Deployment v1
```

Then developer makes another change:

```text
Commit B
   ↓
Deployment v2
```

Then another:

```text
Commit C
   ↓
Deployment v3
```

Suppose v3 causes a problem.

Argo CD keeps track of the application history/previous revisions.

You can select an earlier revision and perform a rollback.

Conceptually:

```text
v1
 ↓
v2
 ↓
v3  ← current problematic deployment
```

Rollback:

```text
v3
 ↓
rollback
 ↓
v2
```

The important point is:

> **Rollback means returning the application to a previous known Git/application revision rather than keeping the current revision.**

In a GitOps workflow, you should also understand the distinction between an **Argo CD rollback action** and making a **Git revert**. For long-term GitOps consistency, teams commonly make Git itself reflect the intended state rather than leaving Git and the cluster permanently divergent.

---

# 19. Private GitHub Repository + Argo CD

Your trainer also explained authentication.

Suppose your repository is:

```text
Private GitHub Repository
```

Argo CD cannot simply clone it anonymously.

You need authentication.

Your trainer showed SSH authentication.

Generate SSH keys:

```bash
ssh-keygen
```

You get:

```text
private key
public key
```

Conceptually:

```text
GitHub
   ↑
Public Key
```

and:

```text
Argo CD
   |
Private Key
```

The relationship is:

```text
Argo CD
  |
  | private key
  ↓
GitHub
  |
  | matching public key
  ↓
Authentication successful
```

Then Argo CD can access the private repository.

### Very important

Never commit the private key into Git.

The private key belongs in the appropriate secure credential configuration of Argo CD.

---

# 20. Public repository vs Private repository

### Public repository

Argo CD can generally access the repository without private repository credentials.

```text
Argo CD
   ↓
Public GitHub
```

### Private repository

Authentication is required.

For example:

```text
Argo CD
   ↓
SSH private key / other supported credentials
   ↓
Private GitHub repository
```

---

# 21. Argo CD Application

When you create an application in Argo CD, you're basically telling Argo CD:

> "Watch this repository, this branch/revision, and this directory, and deploy its Kubernetes configuration into this cluster and namespace."

Think of the configuration as:

```text
Application Name
        ↓
Git Repository
        ↓
Revision
        ↓
Path
        ↓
Destination Cluster
        ↓
Namespace
        ↓
Sync Policy
```

For example:

```text
Application:
    dev

Repository:
    github.com/company/k8s-manifests

Revision:
    HEAD

Path:
    dev/

Destination:
    Kubernetes cluster

Namespace:
    dev
```

Argo CD now knows exactly what it should monitor.

---

# 22. What does HEAD mean?

Your trainer mentioned:

```text
Revision → HEAD
```

In this context, HEAD generally refers to the current/latest revision being tracked for the selected Git reference.

For a simple example:

```text
Git commits:

A
B
C
D ← current HEAD
```

Argo CD tracks the configured revision/ref and sees the corresponding state.

---

# 23. What does Path mean?

Suppose your Git repository looks like:

```text
k8s-manifests/
│
├── dev/
│   ├── deployment.yaml
│   └── service.yaml
│
├── staging/
│   ├── deployment.yaml
│   └── service.yaml
│
└── prod/
    ├── deployment.yaml
    └── service.yaml
```

You create an Argo CD application:

```text
Path = dev/
```

Argo CD manages the Kubernetes manifests under:

```text
dev/
```

It doesn't automatically manage the `prod/` manifests just because they're in the same repository.

---

# 24. Namespace

Suppose you create:

```bash
kubectl create namespace dev
```

Then Argo CD Application:

```text
Destination Namespace:
dev
```

Argo CD deploys the application's resources into that namespace, subject to the manifests and configuration.

---

# 25. Argo CD access through LoadBalancer

Your trainer used:

```bash
kubectl patch svc argocd-server -n argocd \
-p '{"spec": {"type": "LoadBalancer"}}'
```

Originally the service may be:

```text
ClusterIP
```

You changed it to:

```text
LoadBalancer
```

Kubernetes then requests a cloud load balancer from the cloud provider's integration.

You can check:

```bash
kubectl get svc -n argocd
```

You may see something like:

```text
NAME            TYPE           EXTERNAL-IP
argocd-server   LoadBalancer   xxx.elb.amazonaws.com
```

Then you can use the external endpoint to reach the Argo CD server.

---

# 26. `kubectl patch` vs `kubectl edit`

Your trainer showed two methods.

### Method 1

```bash
kubectl patch svc argocd-server -n argocd \
-p '{"spec":{"type":"LoadBalancer"}}'
```

This directly modifies the resource.

### Method 2

```bash
kubectl edit svc argocd-server -n argocd
```

Then manually change:

```yaml
type: ClusterIP
```

to:

```yaml
type: LoadBalancer
```

### Easy understanding

```text
kubectl edit
    ↓
Open resource configuration
    ↓
Manually modify

kubectl patch
    ↓
Directly apply specific change
```

---

# 27. Important correction about LoadBalancer + Security Groups

Your trainer mentioned that a separate security group can be associated with the cloud load balancer and allow HTTP traffic.

The exact behavior depends on the Kubernetes/cloud integration and AWS configuration, so don't memorize:

> "Every LoadBalancer always creates a separate SG with port 80."

Instead remember:

> **A Kubernetes `LoadBalancer` Service can provision a cloud load balancer, and the resulting network/security configuration depends on the cloud provider and controller/configuration.**

Also, Argo CD itself is not what creates that AWS security group. Kubernetes/cloud-provider integration is responsible for provisioning the cloud resources.

---

# 28. Why Kubernetes concepts can be practiced without AWS

Your trainer also made an important distinction.

Some Kubernetes concepts don't require AWS.

For example:

```text
Deployment
ReplicaSet
Pod
Service
ConfigMap
Secret
HPA
Debugging
kubectl commands
```

You can practice these using:

* kind
* Minikube
* KillerCoda / similar Kubernetes labs

For example:

```bash
kubectl create deployment nginx --image=nginx
```

No AWS account is necessary.

---

# 29. What actually needs cloud infrastructure?

Some concepts involve cloud-specific infrastructure.

For example:

```text
AWS Load Balancer
AWS networking
AWS IAM
EKS
Cloud NAT
Cloud-specific storage
```

And your trainer specifically distinguished:

### HPA

Horizontal Pod Autoscaler:

```text
CPU increases
     ↓
HPA
     ↓
Pods increase
```

This can be practiced in a local Kubernetes environment.

### Node Autoscaling

Now suppose:

```text
Pod requires resources
        ↓
No worker node has enough capacity
        ↓
Need another node
```

Now you're talking about:

```text
Cluster Autoscaler
or
Karpenter
```

On AWS/EKS, this involves provisioning cloud infrastructure.

---

# 30. Complete real-world example

Now let's connect **everything your trainer taught today**.

Imagine you're working on an LMS application.

A developer changes the application:

```text
Video API code changed
```

### Step 1 — Developer

```text
Developer
    ↓
git push
    ↓
GitHub
```

### Step 2 — CI

Jenkins detects the change:

```text
Jenkins
   ↓
Run tests
   ↓
Docker build
   ↓
Security scan
   ↓
Push image
   ↓
ECR
```

New image:

```text
lms-video-api:v2
```

### Step 3 — Update GitOps repository

The Kubernetes manifest changes:

```yaml
image: lms-video-api:v1
```

to:

```yaml
image: lms-video-api:v2
```

Commit:

```text
Update video API to v2
```

### Step 4 — Argo CD

Argo CD sees:

```text
Git = v2
Cluster = v1
```

Therefore:

```text
OutOfSync
```

With Auto-Sync enabled:

```text
Argo CD
   ↓
Sync
```

### Step 5 — Kubernetes

Deployment gets updated:

```text
v1 → v2
```

Kubernetes creates new Pods.

New Pods pull:

```text
lms-video-api:v2
```

from ECR.

Now:

```text
Git = v2
Cluster = v2

      ↓
    Synced
```

---

# 31. Now suppose an engineer manually changes the cluster

Engineer runs:

```bash
kubectl scale deployment lms-video-api --replicas=10
```

But Git says:

```yaml
replicas: 3
```

Now:

```text
Git       = 3
Cluster   = 10
```

Argo CD:

```text
OutOfSync
```

If Self-Heal is enabled:

```text
10 → 3
```

---

# 32. Now suppose someone deletes a YAML from Git

Git initially:

```text
deployment.yaml
service.yaml
configmap.yaml
```

Someone deletes:

```text
configmap.yaml
```

Git:

```text
deployment.yaml
service.yaml
```

Cluster:

```text
Deployment
Service
ConfigMap
```

If:

```text
Prune OFF
```

ConfigMap remains.

If:

```text
Prune ON
```

Argo CD removes the ConfigMap from the cluster.

---

# 33. Now suppose v2 has a problem

History:

```text
v1 → v2 → v3
```

Current:

```text
v3
```

Something goes wrong.

You can return to an earlier application revision using Argo CD's rollback/history mechanisms.

Conceptually:

```text
v3
 ↓
Rollback
 ↓
v2
```

But in a mature GitOps workflow, you should also ensure the **Git repository reflects the intended final state**, otherwise you can create a Git-vs-cluster discrepancy.

---

# 34. The complete mental model

If you remember only one diagram from Day 8, remember this:

```text
                    GITHUB
               Source of Truth
                     |
          +----------+----------+
          |                     |
    Application Code       K8s YAML/Helm
          |                     |
          ↓                     ↓
         CI                  Argo CD
   Jenkins/GHA/GitLab           |
          |                     |
          ↓                     |
         ECR                    |
          |                     |
          |                     ↓
          +--------------→ Kubernetes
                              Cluster
                                 |
                              Pods etc.
```

And Argo CD continuously thinks:

```text
"What does Git say?"
        ↓
"What does Kubernetes currently have?"
        ↓
"Are they different?"
        ↓
      YES
        ↓
   OutOfSync
        ↓
Should I sync?
        ↓
Auto-Sync / Manual Sync
        ↓
Make cluster match Git
```

---

# 35. Your three most important Argo CD settings

Memorize this table:

| Setting       | Meaning                           | Example                        |
| ------------- | --------------------------------- | ------------------------------ |
| **Auto-Sync** | Automatically apply Git changes   | Git replicas 2 → 3             |
| **Self-Heal** | Fix manual cluster changes        | `kubectl scale` 3 → 5          |
| **Prune**     | Delete resources removed from Git | Delete `service.yaml` from Git |

### One-line memory trick

```text
AUTO-SYNC = Git changed → update Cluster

SELF-HEAL = Cluster changed → restore Git state

PRUNE = Resource removed from Git → remove from Cluster
```

---

# 36. Interview scenario questions from today's class

### Q1. What is the source of truth in Argo CD?

**Answer:**

> In a GitOps model, the Git repository containing the Kubernetes desired-state configuration is treated as the source of truth. Argo CD compares that desired state with the actual state in the Kubernetes cluster and reconciles differences.

---

### Q2. What is OutOfSync?

> OutOfSync means the desired state stored in Git differs from the actual state of the Kubernetes resources in the cluster.

---

### Q3. What is Auto-Sync?

> Auto-Sync allows Argo CD to automatically apply changes detected in the configured Git repository to the Kubernetes cluster.

---

### Q4. What is Self-Heal?

> Self-Heal allows Argo CD to automatically correct configuration drift caused by manual changes made directly in the Kubernetes cluster.

---

### Q5. What is Prune?

> Prune allows Argo CD to delete Kubernetes resources that are no longer defined in Git.

---

### Q6. Someone manually changes replicas from 3 to 5. Which feature handles it?

```text
Self-Heal
```

assuming Self-Heal is enabled.

---

### Q7. Developer changes replicas from 3 to 5 in Git. Which feature handles it?

```text
Auto-Sync
```

assuming Auto-Sync is enabled.

---

### Q8. Developer deletes `service.yaml` from Git. Which feature ensures the Service is deleted from Kubernetes?

```text
Prune
```

assuming pruning is enabled.

---

### Q9. Can Auto-Sync and Self-Heal be enabled independently?

Yes.

For example:

```text
Auto-Sync = ON
Self-Heal  = OFF
```

Git changes can automatically flow to the cluster, but manual drift is not automatically corrected by Self-Heal.

---

### Q10. Does Prune mean Argo CD deletes every manually created resource?

Not simply every resource in the cluster.

Pruning concerns resources that **Argo CD manages** and that are no longer represented by the desired application state.

This distinction is important in real environments.

---

# 37. Day 8 practical lab sequence

I recommend doing today's class again practically in this order:

```text
LAB 1
Create local Kubernetes cluster
        ↓
Create namespace
        ↓
Install Argo CD
```

```text
LAB 2
Connect Argo CD to public GitHub repository
        ↓
Create Application
        ↓
Manual Sync
        ↓
Verify Pods
```

```text
LAB 3
Enable Auto-Sync
        ↓
Change replicas in Git
        ↓
git push
        ↓
Observe automatic deployment
```

```text
LAB 4
Enable Self-Heal
        ↓
kubectl scale deployment
        ↓
Observe OutOfSync
        ↓
Observe Argo CD restoring Git state
```

```text
LAB 5
Enable Prune
        ↓
Delete YAML from Git
        ↓
git push
        ↓
Observe resource deletion
```

```text
LAB 6
Disable each option one at a time
        ↓
Repeat the same experiment
        ↓
Observe the difference
```

```text
LAB 7
Connect a PRIVATE GitHub repository
        ↓
Generate SSH key
        ↓
Public key → GitHub
Private key → Argo CD
        ↓
Test repository connection
```

```text
LAB 8
Change replicas:
1 → 2 → 3
        ↓
Observe Git history
        ↓
Practice rollback/history
```

That sequence will make the concepts much easier than just reading the Argo CD UI.

### The core idea of Day 8

```text
                GIT
          Source of Truth
                 |
                 ↓
             ARGO CD
                 |
       Compare Desired State
          vs Actual State
                 |
          +------+------+
          |             |
      Different       Same
          |             |
      OutOfSync       Synced
          |
    Auto-Sync / Sync
          |
          ↓
      KUBERNETES
```

Then remember the three controls:

```text
             ARGO CD
                |
     +----------+----------+
     |          |          |
 Auto-Sync   Self-Heal   Prune
     |          |          |
 Git change   Manual     Deleted
 → Cluster    drift      Git resource
                          → Cluster deletion
```

This is the **real connection between almost everything your trainer discussed today**.
