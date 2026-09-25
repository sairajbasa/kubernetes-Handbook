# Day 11 — Kubernetes Deployment Strategies & Rollback

## 1. Deployment Strategies

In Kubernetes, application updates can be performed using different deployment strategies.

The four important strategies are:

1. **Recreate**
2. **Rolling Update**
3. **Blue-Green**
4. **Canary**

### Easy memory

> **Recreate = Delete all → Create all**
> **Rolling = Replace gradually**
> **Blue-Green = Prepare new → Switch all traffic**
> **Canary = Release gradually → Increase traffic**

---

# 2. ReplicaSet vs Deployment

This is an important interview question.

### ReplicaSet

A ReplicaSet's primary responsibility is:

> **Maintain the desired number of Pods.**

For example:

```yaml
replicas: 4
```

If one Pod crashes:

```text
4 Pods
 ↓
1 Pod crashes
 ↓
3 Pods
 ↓
ReplicaSet creates 1 new Pod
 ↓
4 Pods
```

However, a ReplicaSet is **not designed to manage application version rollouts**.

Suppose the image changes:

```text
nginx:1.25
        ↓
nginx:1.26
```

Changing the image in the ReplicaSet does not give you Deployment-style rolling-update management.

You generally need to update/apply the ReplicaSet configuration, and the ReplicaSet itself does not provide the rollout history and controlled rollout mechanisms that a Deployment provides.

---

# 3. Deployment

A Deployment manages ReplicaSets and provides:

* Rolling updates
* Recreate strategy
* Rollout history
* Rollback
* Version management

Architecture:

```text
Deployment
    |
    +---- ReplicaSet v1
    |         |
    |       Pods
    |
    +---- ReplicaSet v2
              |
            Pods
```

When you update the image:

```text
v1 → v2
```

the Deployment creates/manages a new ReplicaSet and gradually replaces Pods according to the configured strategy.

---

# 4. Default Deployment Strategy

If you don't specify a strategy:

```yaml
spec:
  strategy:
    type: RollingUpdate
```

Kubernetes Deployment uses:

> **RollingUpdate by default**

So:

```yaml
spec:
  replicas: 4
```

without an explicit strategy means the Deployment performs a rolling update when the Pod template changes.

---

# 5. Recreate Strategy

Recreate means:

> **Delete all old Pods first, then create new Pods.**

Example:

```yaml
spec:
  strategy:
    type: Recreate
```

Suppose:

```text
Old version
Pod 1
Pod 2
Pod 3
Pod 4
```

During update:

```text
Pod 1 ─┐
Pod 2 ─┤
Pod 3 ─┼──> DELETE
Pod 4 ─┘

        ↓

New version
Pod 1
Pod 2
Pod 3
Pod 4
```

### Disadvantage

There can be **downtime**, because there is a period when no application Pods are available.

### When useful?

Recreate can be useful when running old and new versions simultaneously is unsafe or undesirable.

---

# 6. Rolling Update Strategy

RollingUpdate means:

> **Gradually replace old Pods with new Pods.**

Instead of:

```text
Delete all → Create all
```

we do:

```text
Old Pods → New Pods
       gradually
```

Example:

```text
Before:

v1  v1  v1  v1
```

During rollout:

```text
v1  v1  v1  v2

v1  v1  v2  v2

v1  v2  v2  v2

v2  v2  v2  v2
```

The exact behavior depends on:

* `maxSurge`
* `maxUnavailable`

---

# 7. maxSurge

`maxSurge` controls:

> **How many Pods above the desired replica count can temporarily exist during a rolling update.**

Suppose:

```yaml
replicas: 4
```

and:

```yaml
maxSurge: 25%
```

25% of 4 = 1.

Therefore Kubernetes can temporarily have:

```text
Desired = 4
Maximum = 5
```

Example:

```text
4 old Pods
    ↓
Create 1 new Pod
    ↓
5 Pods temporarily
```

---

# 8. maxUnavailable

`maxUnavailable` controls:

> **How many Pods can be unavailable during the rolling update.**

Suppose:

```yaml
replicas: 4

maxUnavailable: 25%
```

25% of 4 = 1.

Therefore up to **1 Pod** can be unavailable during the rollout.

### Important

`maxSurge` and `maxUnavailable` are controls for the **number of Pods**, not direct controls for traffic percentage.

---

# 9. Example — maxSurge and maxUnavailable

Suppose:

```yaml
replicas: 4

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

Conceptually:

```text
Initial:

v1 v1 v1 v1
```

Kubernetes can create an additional new Pod:

```text
v1 v1 v1 v1 v2
```

Once the new Pod is ready, an old Pod can be removed:

```text
v1 v1 v1 v2
```

Then another new Pod:

```text
v1 v1 v2 v2
```

Eventually:

```text
v2 v2 v2 v2
```

The exact sequence can vary depending on readiness and controller behavior.

---

# 10. What if maxSurge = 100%?

For:

```yaml
replicas: 4

maxSurge: 100%
```

Kubernetes may temporarily create up to:

```text
4 + 4 = 8 Pods
```

So:

```text
4 old
+
4 new
=
8 maximum temporarily
```

This can make the rollout faster but requires additional capacity.

---

# 11. What if maxUnavailable = 100%?

With:

```yaml
replicas: 4

maxUnavailable: 100%
```

all 4 desired Pods are allowed to be unavailable during the rollout.

This gives Kubernetes much more freedom to remove old Pods.

### Production consideration

Large values can reduce availability, while small values provide a more gradual rollout.

---

# 12. Useful Commands

Check Pods:

```bash
kubectl get pods
```

Check the image used by a Pod:

```bash
kubectl describe pod <pod-name>
```

Check Deployment:

```bash
kubectl get deployment
```

Check Deployment details:

```bash
kubectl describe deployment <deployment-name>
```

Check rollout:

```bash
kubectl rollout status deployment/<deployment-name>
```

Check rollout history:

```bash
kubectl rollout history deployment/<deployment-name>
```

Rollback:

```bash
kubectl rollout undo deployment/<deployment-name>
```

---

# 13. Blue-Green Deployment

Blue-Green means:

> **Run the old and new versions simultaneously, then switch traffic from old to new.**

Suppose Blue is the current production version:

```text
Blue Deployment
    |
  4 Pods
  v1
```

Service:

```text
Users
  |
Service
  |
Blue Pods
```

Now developers release v2.

Create Green:

```text
Blue Deployment          Green Deployment
     |                         |
  4 Pods v1                 4 Pods v2
```

Both environments are running.

But initially:

```text
Users
  |
Service
  |
Blue v1
```

After testing Green, change the Service selector so it points to Green.

```text
Users
  |
Service
  |
Green v2
```

Now traffic moves to the new version.

---

# 14. Blue-Green Example

Blue:

```yaml
app: myapp
version: blue
```

Green:

```yaml
app: myapp
version: green
```

Service initially:

```yaml
selector:
  app: myapp
  version: blue
```

After validation:

```yaml
selector:
  app: myapp
  version: green
```

The Service's endpoints then change to the Pods matching the new selector.

You can inspect endpoints with:

```bash
kubectl get endpoints
```

or, on newer Kubernetes versions, preferably:

```bash
kubectl get endpointslices
```

---

# 15. Blue-Green Rollback

One major advantage is that the old environment can remain available temporarily.

If Green has a serious issue:

```text
Service
   |
Green ❌
```

Switch the Service selector back:

```text
Service
   |
Blue ✅
```

Traffic returns to the old version.

After confirming the new version is stable, the old Blue environment can be decommissioned.

---

# 16. Blue-Green Considerations

### Additional resources

During the transition:

```text
Blue → running
Green → running
```

Therefore you temporarily need capacity for both environments.

### Application-level strategy

Blue-Green is generally referring to switching **application environments**, not creating an entirely separate Kubernetes cluster.

You could use separate clusters in some architectures, but that is a different infrastructure design decision.

---

# 17. Canary Deployment

Canary means:

> **Release the new version to a small portion of traffic/users first, validate it, then gradually increase exposure.**

Example:

```text
Old version = v1
New version = v2
```

Suppose there are:

```text
4 Pods v1
1 Pod v2
```

If both versions match the same Service selector:

```text
             Service
                |
       +--------+--------+
       |                 |
    v1 Pods            v2 Pod
```

The Service can send traffic to Pods from both versions.

### Important correction

With a normal Kubernetes Service, you should **not describe this as an exact 80% / 20% traffic split merely because there are 4 old Pods and 1 new Pod**.

Kubernetes Service load balancing does not provide a guaranteed percentage-based traffic split.

For controlled percentage-based canary traffic, tools such as:

* Ingress controllers
* Service meshes
* Argo Rollouts

can provide more sophisticated traffic management.

---

# 18. Canary Ramp-Up

**Ramp-up** means:

> Gradually increase exposure to the new version.

Conceptually:

```text
v1 = 100%
v2 =   0%

      ↓

v1 = 80%
v2 = 20%

      ↓

v1 = 50%
v2 = 50%

      ↓

v1 = 20%
v2 = 80%

      ↓

v1 = 0%
v2 = 100%
```

The exact traffic percentages require an appropriate traffic-management mechanism.

---

# 19. Canary Ramp-Down

**Ramp-down** means:

> Reduce exposure to the new version.

Suppose v2 starts showing errors:

```text
v1 = 80%
v2 = 20%
```

You can reduce v2 exposure:

```text
v1 = 90%
v2 = 10%
```

or completely remove it:

```text
v1 = 100%
v2 = 0%
```

This is useful for limiting the impact of a problematic release.

---

# 20. Blue-Green vs Canary

| Feature          | Blue-Green                                         | Canary                                   |
| ---------------- | -------------------------------------------------- | ---------------------------------------- |
| Old version      | Running                                            | Running                                  |
| New version      | Running                                            | Running                                  |
| Initial exposure | New version usually receives no production traffic | Small production exposure                |
| Traffic movement | Switch traffic to new version                      | Gradually increase exposure              |
| Rollback         | Switch back to old version                         | Reduce/remove new-version exposure       |
| Main idea        | Environment switch                                 | Gradual release                          |
| Resource usage   | Higher during transition                           | Depends on implementation                |
| Risk exposure    | New version can receive all traffic after switch   | New version starts with limited exposure |

### Easy memory

> **Blue-Green = Switch**
> **Canary = Gradually expose**

---

# 21. Deployment Strategy vs Kubernetes Deployment

A common interview trap:

**Are Blue-Green and Canary built-in Kubernetes Deployment strategies like Recreate and RollingUpdate?**

Not exactly.

Kubernetes `Deployment.spec.strategy.type` directly supports:

```text
Recreate
RollingUpdate
```

Blue-Green and Canary are **deployment patterns** that can be implemented using Kubernetes objects and/or additional tools.

For example:

```text
Blue-Green
Deployment + Service
```

and:

```text
Canary
Deployment + Service + traffic management
```

Tools such as **Argo Rollouts** can provide advanced Blue-Green and Canary rollout capabilities.

---

# 22. Argo CD and Deployment Strategies

With Argo CD, instead of manually running:

```bash
kubectl apply -f deployment.yaml
```

you can follow GitOps.

Flow:

```text
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
Deployment
    |
    v
Pods
```

You update the Kubernetes manifest in Git:

```yaml
image: myapp:v2
```

commit and push:

```bash
git add .
git commit -m "Update application image to v2"
git push
```

Argo CD detects the Git change and synchronizes the cluster according to the configured policies.

---

# 23. Important Argo CD Distinction

Argo CD is primarily a **GitOps continuous delivery tool**.

It does not automatically mean:

```text
Git → Canary traffic
```

For advanced progressive delivery, you can use:

```text
Git
 ↓
Argo CD
 ↓
Argo Rollouts
 ↓
Canary / Blue-Green
```

This distinction is important in interviews.

---

# 24. Production-Level KT

Think about deployment strategies from the perspective of **risk**:

```text
Recreate
   ↓
High downtime risk

Rolling
   ↓
Gradual replacement

Blue-Green
   ↓
Fast traffic switch + easy rollback

Canary
   ↓
Gradual production exposure
```

The strategy depends on:

* Application architecture
* Availability requirements
* Database compatibility
* Cost
* Rollback requirements
* Traffic management
* Observability
* Release risk

---

# 25. Production Scenario

### Scenario

You have:

```text
Production
4 Pods → v1
```

Developer releases:

```text
v2
```

### Recreate

```text
Delete v1
   ↓
Create v2
```

Potential downtime.

### Rolling

```text
v1 v1 v1 v1
 ↓
v2 v1 v1 v1
 ↓
v2 v2 v1 v1
 ↓
v2 v2 v2 v1
 ↓
v2 v2 v2 v2
```

Gradual replacement.

### Blue-Green

```text
Blue  = v1
Green = v2
```

Test Green, then:

```text
Service → Blue
```

changes to:

```text
Service → Green
```

Traffic switches.

### Canary

Start:

```text
v1 → majority
v2 → small exposure
```

Monitor:

```text
Error rate
Latency
CPU/memory
Application metrics
Logs
```

If healthy:

```text
Increase v2 exposure
```

If unhealthy:

```text
Reduce/remove v2 exposure
```

---

# 26. Interview Questions

### Q1. Why do we use Deployment instead of ReplicaSet?

**Answer:**

> ReplicaSet maintains the desired number of Pods, while Deployment provides higher-level application lifecycle management such as rolling updates, rollout history and rollback.

---

### Q2. What is the default Deployment strategy?

> **RollingUpdate.**

---

### Q3. What is Recreate?

> Recreate terminates all existing Pods before creating Pods with the new version. It can cause downtime.

---

### Q4. What is RollingUpdate?

> RollingUpdate gradually replaces old Pods with new Pods while controlling availability and additional Pods through `maxUnavailable` and `maxSurge`.

---

### Q5. What is maxSurge?

> It defines how many additional Pods can temporarily be created above the desired replica count during a rolling update.

---

### Q6. What is maxUnavailable?

> It defines how many Pods can be unavailable during a rolling update.

---

### Q7. What is Blue-Green?

> Blue-Green runs old and new environments simultaneously and switches the Service or traffic from the old environment to the new environment after validation.

---

### Q8. What is Canary?

> Canary gradually exposes a new version to a limited portion of production traffic or users, monitors the result, and increases exposure if the release is healthy.

---

### Q9. What is ramp-up?

> Increasing exposure to the new version gradually.

### Q10. What is ramp-down?

> Decreasing exposure to the new version, usually when the new version shows problems.

---

# 27. One-Line Memory

> **Recreate = Delete all → Create all | Rolling = Replace gradually | Blue-Green = Prepare new → Switch | Canary = Small exposure → Monitor → Ramp up**

This is the **core Day 11 knowledge** to remember.
