-> Your Day 10 notes are actually covering a **very important Kubernetes production concept: Ingress + Ingress Controller + path-based routing**, with an AWS Load Balancer in front.

I would clean them up like this.

# Day 10 — Kubernetes Ingress & Path-Based Routing

## 1. Why do we need Ingress?

Suppose one application has multiple components:

```text
Application
│
├── Home Page
├── Fashion
├── Mobiles
└── Electronics
```

Each component has its own:

* Deployment
* Pods
* Service

For example:

```text
Fashion
  ├── Deployment
  ├── 3 Pods
  └── ClusterIP Service

Mobiles
  ├── Deployment
  ├── 3 Pods
  └── ClusterIP Service

Electronics
  ├── Deployment
  ├── 3 Pods
  └── ClusterIP Service

Home
  ├── Deployment
  ├── 2 Pods
  └── ClusterIP Service
```

---

# 2. What happens if we use Service type LoadBalancer?

If you expose every application using:

```yaml
type: LoadBalancer
```

you can end up with:

```text
Fashion Service ──> AWS Load Balancer
Mobiles Service ──> AWS Load Balancer
Electronics Service ──> AWS Load Balancer
Home Service ──> AWS Load Balancer
```

Potentially **4 external load balancers**.

Users would then need different endpoints such as:

```text
fashion.example.com
mobiles.example.com
electronics.example.com
example.com
```

or provider-generated load-balancer DNS names.

That is usually not the architecture you want when a single public entry point can route requests to multiple applications.

---

# 3. The problem with LoadBalancer Service

A Kubernetes `Service` of type `LoadBalancer` primarily exposes a service externally.

For example:

```text
Internet
   |
   v
AWS Load Balancer
   |
   v
Fashion Service
   |
   v
Fashion Pods
```

But your requirement is:

```text
example.com/
        |
        +---- /fashions       → Fashion
        |
        +---- /mobiles        → Mobiles
        |
        +---- /electronics    → Electronics
```

Now we need **HTTP-aware routing**.

This is where **Ingress** comes in.

---

# 4. What is Kubernetes Ingress?

**Ingress is a Kubernetes API object used to define HTTP/HTTPS routing rules for traffic entering the cluster.**

For example:

```text
example.com/
      |
      +---- /fashions
      |         ↓
      |    Fashion Service
      |
      +---- /mobiles
      |         ↓
      |    Mobiles Service
      |
      +---- /electronics
                ↓
           Electronics Service
```

The important point:

> **Ingress defines the routing rules.**

But an Ingress object by itself doesn't actually process traffic.

We need an **Ingress Controller**.

---

# 5. What is an Ingress Controller?

The **Ingress Controller is the component that actually watches Ingress resources and implements the routing rules.**

Examples include:

* NGINX Ingress Controller
* AWS Load Balancer Controller
* Traefik
* HAProxy
* Kong

In your trainer's example, you're using:

> **NGINX Ingress Controller**

So the architecture becomes:

```text
                    INTERNET
                       |
                       v
              AWS Load Balancer
                       |
                       v
             NGINX Ingress Controller
                       |
            +----------+----------+
            |          |          |
            v          v          v
       Fashion      Mobiles   Electronics
       ClusterIP    ClusterIP   ClusterIP
            |          |          |
            v          v          v
          Pods       Pods       Pods
```

---

# 6. Very important: Ingress vs Ingress Controller

This is a common interview question.

### Ingress

Ingress is the **configuration/rules**.

Example:

```yaml
/fashions     → fashion-service
/mobiles      → mobile-service
/electronics  → electronics-service
```

### Ingress Controller

The controller is the **actual software that implements those rules**.

For example:

```text
Ingress rules
     ↓
NGINX Ingress Controller
     ↓
Routes HTTP request
```

### Simple memory trick

> **Ingress = What should happen?**
> **Ingress Controller = Actually makes it happen.**

---

# 7. Path-Based Routing

This is the main concept from Day 10.

Suppose the user accesses:

```text
https://example.com/fashions
```

The request reaches NGINX.

NGINX checks the Ingress rules:

```text
/fashions → fashion-service
```

Then:

```text
NGINX
  ↓
fashion-service
  ↓
fashion pods
```

Similarly:

```text
https://example.com/mobiles
```

becomes:

```text
NGINX
  ↓
mobile-service
  ↓
mobile pods
```

And:

```text
https://example.com/electronics
```

becomes:

```text
NGINX
  ↓
electronics-service
  ↓
electronics pods
```

---

# 8. Why ClusterIP Services?

This is another important part of your trainer's explanation.

Your application services can remain:

```yaml
spec:
  type: ClusterIP
```

because they don't need to be directly exposed to the Internet.

For example:

```text
Internet
   |
   v
AWS Load Balancer
   |
   v
NGINX Ingress Controller
   |
   +---- ClusterIP → Fashion Pods
   |
   +---- ClusterIP → Mobile Pods
   |
   +---- ClusterIP → Electronics Pods
```

The **Ingress Controller is the external entry point**, while the application services remain internal.

---

# 9. Complete request flow

This is the flow I recommend memorizing for interviews:

```text
User
 |
 | https://example.com/mobiles
 v
DNS
 |
 v
AWS Load Balancer
 |
 v
NGINX Ingress Controller
 |
 | checks Ingress rules
 |
 | /mobiles
 v
mobile-service
 |
 v
Mobile Pod
```

For another request:

```text
User
 |
 | https://example.com/fashions
 v
AWS Load Balancer
 |
 v
NGINX
 |
 | /fashions
 v
fashion-service
 |
 v
Fashion Pod
```

---

# 10. L4 vs L7 — VERY IMPORTANT

Your trainer mentioned this, and you should understand it properly.

### Layer 4

Layer 4 works primarily with:

* TCP
* UDP
* IP
* Port

It doesn't understand application-level HTTP paths such as:

```text
/fashions
/mobiles
/electronics
```

### Layer 7

Layer 7 understands HTTP/HTTPS information such as:

```text
Host
Path
Headers
HTTP method
```

Therefore NGINX can make decisions such as:

```text
/fashions → fashion-service

/mobiles → mobile-service

/electronics → electronics-service
```

---

# 11. One important clarification about AWS Load Balancers

Your notes say:

> NLB, CLB are Layer-4 oriented whereas NGINX is doing Layer-7 routing.

For the **architecture your trainer demonstrated**, this is a useful mental model:

```text
AWS Load Balancer
        ↓
NGINX
        ↓
ClusterIP
        ↓
Pods
```

But don't memorize **"AWS Load Balancer can never do Layer 7"**.

AWS has different load-balancing products. For example, **Application Load Balancer (ALB)** operates at Layer 7 and can itself perform HTTP path/host-based routing.

So the better interview statement is:

> "In this NGINX Ingress architecture, the external AWS load balancer gets traffic into the cluster, while the NGINX Ingress Controller performs the Kubernetes HTTP routing based on the Ingress rules."

That's much more accurate.

---

# 12. Ingress Resource Example

A simplified example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
spec:
  ingressClassName: nginx

  rules:
  - host: example.com
    http:
      paths:

      - path: /fashions
        pathType: Prefix
        backend:
          service:
            name: fashion-service
            port:
              number: 80

      - path: /mobiles
        pathType: Prefix
        backend:
          service:
            name: mobile-service
            port:
              number: 80

      - path: /electronics
        pathType: Prefix
        backend:
          service:
            name: electronics-service
            port:
              number: 80
```

The important part is:

```yaml
/fashions
/mobiles
/electronics
```

These are the **routing rules**.

---

# 13. What does `pathType: Prefix` mean?

Suppose:

```yaml
path: /mobiles
pathType: Prefix
```

Then requests such as:

```text
/mobiles
/mobiles/iphone
/mobiles/samsung
/mobiles/products/123
```

can match the `/mobiles` prefix.

So:

```text
/mobiles/*
```

goes to:

```text
mobile-service
```

---

# 14. Host-Based Routing vs Path-Based Routing

You mentioned this in your notes, and it's important.

### Path-based routing

Same host:

```text
example.com/fashions
example.com/mobiles
example.com/electronics
```

Routing decision:

```text
PATH
```

---

### Host-based routing

Different hosts:

```text
fashion.example.com
mobile.example.com
electronics.example.com
```

Routing decision:

```text
HOSTNAME
```

Example:

```text
fashion.example.com → fashion-service

mobile.example.com → mobile-service

electronics.example.com → electronics-service
```

For host-based routing, DNS records need to point the hostnames toward your ingress entry point.

---

# 15. What happens when you install NGINX Ingress Controller?

This is an important architecture point from your class.

You install:

```text
NGINX Ingress Controller
```

The controller is exposed externally, commonly through a:

```text
Service
type: LoadBalancer
```

Then AWS provisions the corresponding external load-balancing infrastructure supported by that controller/service configuration.

Conceptually:

```text
AWS Load Balancer
       |
       v
NGINX Ingress Controller
       |
       +---- Fashion Service
       +---- Mobile Service
       +---- Electronics Service
```

**This is the key difference:**

You don't create one LoadBalancer service for every application.

Instead:

```text
                ONE external entry point
                         |
                         v
                 NGINX Ingress
                  /     |      \
                 /      |       \
                v       v        v
           Fashion   Mobiles   Electronics
```

---

# 16. Commands from your class

Check the Ingress:

```bash
kubectl get ingress
```

More detailed information:

```bash
kubectl describe ingress <ingress-name>
```

Check all resources in the ingress-controller namespace:

```bash
kubectl get all -n ingress-controller
```

Check services:

```bash
kubectl get svc -n ingress-controller
```

Check available IngressClasses:

```bash
kubectl get ingressclass
```

Check Ingress resources:

```bash
kubectl get ingress
```

⚠️ Your note says:

```bash
kubectl get ingress -a
```

`-a` isn't generally needed for `kubectl get ingress`; use:

```bash
kubectl get ingress -A
```

if you want to see Ingress resources across **all namespaces**.

---

# 17. AWS CLI

For Application/Network Load Balancers:

```bash
aws elbv2 describe-load-balancers
```

For Classic Load Balancers:

```bash
aws elb describe-load-balancers
```

Notice your original note has a typo:

```bash
aws elb describe-load-balanceres
```

It should be:

```bash
aws elb describe-load-balancers
```

---

# 18. ECR part of the class

Your trainer also mentioned **ECR** because the Kubernetes manifests were using old images.

The production flow is:

```text
Source Code
    |
    v
Docker Build
    |
    v
Docker Image
    |
    v
Amazon ECR
    |
    v
Kubernetes Deployment
    |
    v
Pods
```

For example:

```bash
docker build -t fashion-app .
```

Tag:

```bash
docker tag fashion-app:latest \
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/fashion-app:latest
```

Push:

```bash
docker push \
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/fashion-app:latest
```

Then update the Deployment:

```yaml
containers:
- name: fashion
  image: ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/fashion-app:latest
```

In a real CI/CD pipeline, this image build/tag/push/update process would normally be automated rather than manually performed.

---

# 19. Complete Day 10 Architecture

This is the diagram I would keep in your notes:

```text
                         USER
                           |
                           | https://example.com
                           |
                           v
                     DNS / Route 53
                           |
                           v
                AWS Load Balancer
                           |
                           v
              NGINX Ingress Controller
                           |
                    Ingress Rules
                           |
             +-------------+-------------+
             |             |             |
             |             |             |
       /fashions       /mobiles    /electronics
             |             |             |
             v             v             v
       Fashion SVC    Mobile SVC    Electronics SVC
        ClusterIP      ClusterIP       ClusterIP
             |             |             |
             v             v             v
          Pods          Pods           Pods
        +-----+       +-----+        +-----+
        |     |       |     |        |     |
        |     |       |     |        |     |
        +-----+       +-----+        +-----+
```

---

# 20. Production mental model

As a DevOps engineer, remember these responsibilities:

| Component                | Responsibility                                      |
| ------------------------ | --------------------------------------------------- |
| DNS                      | Converts domain name to entry-point address         |
| AWS Load Balancer        | Gets external traffic toward the ingress layer      |
| Ingress                  | Stores HTTP/HTTPS routing rules                     |
| NGINX Ingress Controller | Implements those routing rules                      |
| ClusterIP Service        | Provides stable internal access to application pods |
| Deployment               | Maintains desired pod replicas                      |
| Pods                     | Run the application                                 |
| ECR                      | Stores container images                             |

The most important flow:

```text
Internet
   ↓
DNS
   ↓
Load Balancer
   ↓
Ingress Controller
   ↓
Ingress Rule
   ↓
ClusterIP Service
   ↓
Pod
```

### 🔥 Interview answer to memorize

If the interviewer asks:

**"Why do we use Ingress in Kubernetes?"**

You can answer:

> "Ingress provides a common entry point for HTTP and HTTPS traffic entering the Kubernetes cluster. Instead of exposing every application through a separate LoadBalancer service, we can keep application services as ClusterIP and use an Ingress Controller such as NGINX to route requests to the appropriate service based on the host or URL path."

That is the **core of Day 10**.

---

## Practical Day 10 Lab

For your own practice, build exactly what your trainer described:

```text
                example.com
                     |
                     v
              NGINX Ingress
               /     |      \
              /      |       \
             v       v        v
         /fashion /mobile /electronics
             |       |        |
             v       v        v
          ClusterIP ClusterIP ClusterIP
             |       |        |
             v       v        v
           Pods     Pods      Pods
```

Create:

* 4 Deployments
* 4 ClusterIP Services
* 1 NGINX Ingress Controller
* 1 Ingress resource
* Path-based routing
* Then test `/`, `/fashions`, `/mobiles`, `/electronics`
* Check what happens when a backend pod is deleted
* Scale each Deployment
* Test routing when multiple replicas exist
* Check the Ingress Controller logs
* Inspect the AWS Load Balancer created for the controller

That lab will make the Day 10 concepts much easier to remember than only reading the notes.
