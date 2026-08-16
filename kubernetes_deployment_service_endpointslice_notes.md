# Kubernetes: Deployment, Service & EndpointSlice

## 1. Deployment → Pod relationship

A `Deployment` manages Pods using labels.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
```

The important relationship is:

```text
Deployment
    |
    | selector: app=nginx
    ↓
Pods
├── Pod-1  app=nginx
├── Pod-2  app=nginx
└── Pod-3  app=nginx
```

The Deployment is effectively saying:

> "I am responsible for Pods having `app=nginx`."

---

## 2. Service does NOT directly care about the Deployment

When we want to expose the application running inside the Pods, we create a `Service`.

For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

Notice:

```yaml
selector:
  app: nginx
```

The Service does **not** say:

> "I want Pods from `nginx-deployment`."

Instead, it says:

> "I want all Pods having the label `app=nginx`."

### Key distinction

| Object | Selector means |
|---|---|
| Deployment | "Which Pods do I manage?" |
| Service | "Which Pods should receive traffic?" |

---

## 3. How does a Service find the correct Pods?

Suppose we have:

```text
Pod-1   app=nginx
Pod-2   app=nginx
Pod-3   app=nginx
...
Pod-100 app=nginx
```

And the Service has:

```yaml
selector:
  app: nginx
```

The Service selects all matching Pods.

Conceptually:

```text
                 nginx-service
                   ClusterIP
                 10.96.10.50
                       |
               selector: app=nginx
                       |
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
    Pod-1           Pod-2           Pod-3
     :80              :80             :80
       .
       .
       .
    Pod-100
```

When another Pod accesses:

```text
http://nginx-service:80
```

the traffic can be distributed among the matching backend Pods.

---

## 4. Service does NOT know about the Deployment

This is the most important mental model:

```text
             Deployment
                 |
          selector: app=nginx
                 |
          manages these Pods
                 |
       ┌─────────┼─────────┐
       ↓         ↓         ↓
     Pod-1     Pod-2     Pod-3
       ↑         ↑         ↑
       └─────────┼─────────┘
                 |
        Service selector
          app=nginx
```

There is no direct relationship:

```text
Service → Deployment
```

Instead:

```text
Service → matching Pod labels
```

and:

```text
Deployment → matching Pod labels
```

They are loosely connected through the **Pod labels**.

---

## 5. What if there are 100 Pods?

Suppose:

```text
Pod-1    app=nginx
Pod-2    app=nginx
Pod-3    app=nginx
...
Pod-100  app=nginx
```

The Service selector:

```yaml
selector:
  app: nginx
```

matches all of them.

The Service therefore has access to all matching backend Pods.

---

## 6. Important example: Service doesn't care who created the Pod

Suppose we have:

```text
nginx-deployment
       |
       ├── Pod-1 app=nginx
       ├── Pod-2 app=nginx
       └── Pod-3 app=nginx

Manually created Pod
       |
       └── Pod-999 app=nginx
```

The Service has:

```yaml
selector:
  app: nginx
```

Therefore, **Pod-999 can also become a Service backend**.

Why?

Because the Service does not care:

> "Who created this Pod?"

It only cares:

> "Does this Pod match my selector?"

This is a very important Kubernetes principle.

---

# 7. Where does EndpointSlice come into the picture?

**EndpointSlice comes primarily with the Service concept, not the Deployment concept.**

The overall relationship is:

```text
Deployment
    |
    | manages
    ↓
  Pods
    ↑
    | selected by
    |
 Service
    |
    | creates/uses
    ↓
EndpointSlice
```

### Before a Service

The Deployment creates/manages Pods:

```text
Deployment
    ↓
Pod-1
Pod-2
Pod-3
```

At this point, EndpointSlice is not the important concept.

### After creating a Service

Suppose the Service has:

```yaml
selector:
  app: nginx
```

Kubernetes finds the Pods matching that selector and maintains their network endpoints in EndpointSlice objects.

```text
Service
  selector: app=nginx
        |
        ↓
 Find matching Pods
        |
        ↓
 EndpointSlice
 ├── 10.0.1.10:80
 ├── 10.0.1.11:80
 └── 10.0.1.12:80
```

---

# 8. What is an endpoint?

An **endpoint** represents an actual network destination where traffic can go.

For example:

```text
Pod IP: 10.0.2.15
Port:   8080
```

This can be represented as an endpoint:

```text
10.0.2.15:8080
```

So an EndpointSlice can contain multiple backend endpoints:

```text
EndpointSlice
├── 10.0.1.10:8080
├── 10.0.1.11:8080
├── 10.0.2.15:8080
└── 10.0.2.16:8080
```

---

# 9. Why is it called EndpointSlice?

The name is literal:

> **Endpoint + Slice = a slice/portion of the complete set of Service endpoints.**

Think of one huge apple:

```text
             🍎
              |
              ↓
        Chop it into slices
              |
       ┌──────┼──────┐
       ↓      ↓      ↓
     Slice 1 Slice 2 Slice 3
```

In Kubernetes:

```text
One huge Endpoints object
        ↓
EndpointSlice 1
EndpointSlice 2
EndpointSlice 3
...
```

Each EndpointSlice contains a portion of the complete set of endpoints.

### Example

If a Service has 10,000 backend Pods:

```text
Service
   ↓
EndpointSlice 1 → endpoints 1–100
EndpointSlice 2 → endpoints 101–200
EndpointSlice 3 → endpoints 201–300
...
EndpointSlice N → remaining endpoints
```

This improves scalability because Kubernetes doesn't have to constantly manipulate one enormous object containing every endpoint.

---

# 10. Older `Endpoints` vs modern `EndpointSlice`

Earlier Kubernetes used the `Endpoints` API object:

```text
Service
   ↓
Endpoints
├── Pod IP 1
├── Pod IP 2
├── Pod IP 3
└── ...
```

For a large Service, this could become a very large single object.

Kubernetes introduced `EndpointSlice` as the preferred and more scalable mechanism:

```text
Service
   ↓
EndpointSlice
├── Slice 1 → endpoints 1–100
├── Slice 2 → endpoints 101–200
├── Slice 3 → endpoints 201–300
└── ...
```

### Important clarification

`Endpoints` has **not simply disappeared**. It is deprecated, while `EndpointSlice` is the recommended API for discovering Service backends.

---

# 11. Who creates and maintains EndpointSlice?

Normally, **you do not manually create EndpointSlices** when using a normal selector-based Service.

You write:

```yaml
kind: Service
spec:
  selector:
    app: nginx
```

Kubernetes automatically finds matching Pods and creates/updates the EndpointSlice objects.

For example:

```text
Service
  selector: app=nginx
       |
       ↓
Kubernetes finds matching Pods
       |
       ↓
EndpointSlice
       |
       ├── Pod IP 1
       ├── Pod IP 2
       └── Pod IP 3
```

---

# 12. What happens when a Pod dies?

Suppose:

```text
Before:

EndpointSlice
├── 10.0.1.15:80
├── 10.0.1.16:80
└── 10.0.2.21:80
```

Then:

```text
Pod-2 ❌
```

Kubernetes updates the EndpointSlice:

```text
After:

EndpointSlice
├── 10.0.1.15:80
└── 10.0.2.21:80
```

If the Deployment creates a replacement Pod:

```text
New Pod
IP: 10.0.3.44
label: app=nginx
```

the EndpointSlice is updated to include the new backend.

---

# 13. Complete mental model

This is the most important picture to remember:

```text
                    Deployment
                        |
                 selector: app=nginx
                        |
                 manages these Pods
                        |
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       Pod-1          Pod-2          Pod-3
     app=nginx       app=nginx       app=nginx
     10.0.1.10       10.0.1.11       10.0.1.12
          ↑             ↑             ↑
          └─────────────┼─────────────┘
                        |
                 Service selector
                    app=nginx
                        |
                        ↓
                  EndpointSlice
                        |
              10.0.1.10:80
              10.0.1.11:80
              10.0.1.12:80
                        |
                        ↓
                   ClusterIP
                  10.96.10.50
                        |
                        ↓
                   Client Pod
```

---

# 14. Final revision points

### Deployment

> **"Which Pods do I manage?"**

Uses:

```yaml
spec:
  selector:
    matchLabels:
      app: nginx
```

### Pod Template

Creates Pods with:

```yaml
template:
  metadata:
    labels:
      app: nginx
```

### Service

> **"Which Pods should receive traffic?"**

Uses:

```yaml
spec:
  selector:
    app: nginx
```

### EndpointSlice

> **"Here are the actual network endpoints of the Pods selected by the Service."**

Conceptually:

```text
Service
   ↓
Service selector
   ↓
Matching Pods
   ↓
EndpointSlice
   ↓
Pod IP:Port
```

### One-line memory trick

> **Deployment manages Pods → Service selects Pods → EndpointSlice records their network endpoints → Service traffic reaches those endpoints.**
