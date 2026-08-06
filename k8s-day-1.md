Perfect. We'll do it the right way.

We'll create **Day-1 Kubernetes Notes - Part 1** as if a **20+ years experienced DevOps Architect** is teaching. Every topic will answer:

* **Why did this technology come?**
* **What problem existed before?**
* **How is it solved?**
* **What happens in production?**
* **Interview perspective**

We'll not simply define concepts.

---

# Kubernetes Day-1 Notes

# Part-1

## Introduction to Kubernetes

---

# Why should every DevOps Engineer learn Kubernetes?

Among all DevOps tools, **Kubernetes is one of the most important skills**.

Today almost every company uses one of the following:

* Amazon EKS
* Azure AKS
* Google GKE
* Red Hat OpenShift
* On-Prem Kubernetes

Terraform is used to create infrastructure.

Docker is used to package applications.

**Kubernetes is used to run and manage those applications in production.**

Think of a typical production environment.

```text
Developer writes code
        │
        ▼
Docker packages application
        │
        ▼
Terraform creates Infrastructure
        │
        ▼
Kubernetes runs application
        │
        ▼
Millions of users access it
```

Without Kubernetes, managing thousands of containers becomes extremely difficult.

That is why Kubernetes has become a mandatory skill for DevOps Engineers. Your trainer also emphasized that Kubernetes is a more difficult skill than Terraform and requires consistent practice. 

---

# Before learning Kubernetes

Many beginners think Kubernetes is difficult.

Actually Kubernetes is **not difficult**.

The difficult part is understanding

* Networking
* Load Balancing
* Linux
* Docker

Once these fundamentals are strong, Kubernetes becomes much easier.

That is why AWS Networking is considered one of the important prerequisites before learning Kubernetes. 

---

# What is Kubernetes?

## Definition

Kubernetes (K8s) is an **Open-Source Container Orchestration Platform** developed by Google and now maintained by the Cloud Native Computing Foundation (CNCF).

Its purpose is to automate the deployment, management, scaling and recovery of containerized applications.

Simply,

> Docker creates containers.

> Kubernetes manages containers.

---

# Important Keywords

Whenever someone asks

**What is Kubernetes?**

Remember these words.

* Open Source
* Container Platform
* Orchestration
* Automation
* High Availability
* Self Healing
* Auto Scaling

These words should naturally appear in your explanation.

---

# What does Open Source mean?

Open Source means

* Source code is publicly available.
* Anyone can use it.
* Anyone can contribute.
* No licensing cost for the software itself.

Google developed Kubernetes and later donated it to CNCF so the community could continue improving it. Your notes describe Kubernetes as an open-source tool that anyone can use. 

---

# What is Orchestration?

This is the most important concept in today's class.

Many people memorize

"Kubernetes is an orchestration tool."

Very few understand

"What exactly is orchestration?"

---

## Real World Example

Imagine a company.

There are 100 employees.

Without a manager,

* nobody knows what to do
* tasks are not assigned
* problems remain unresolved
* work is delayed

Now imagine a good manager.

The manager

* assigns work
* monitors employees
* replaces absent employees
* hires new employees
* balances work across the team

The manager doesn't perform every task personally.

He ensures the entire team works correctly.

Kubernetes behaves exactly like this manager for applications. Your notes describe orchestration as a "good manager" or "well application management." 

---

# Orchestration in Production

Suppose Flipkart has

* Payment Service
* Cart Service
* Search Service
* Orders Service
* Notification Service

Every service runs inside containers.

Now imagine

Payment container crashes.

Without orchestration,

an engineer receives an alert,

logs into the server,

finds the failed container,

starts another one,

checks networking,

checks health,

and verifies users can access it again.

Now imagine this happens

500 times a day.

Manual management becomes impossible.

This is exactly why orchestration platforms were introduced.

---

# Simple Definition of Orchestration

Orchestration means

> Automatically managing multiple containerized applications throughout their lifecycle.

It includes

* Deployment
* Monitoring
* Recovery
* Scaling
* Scheduling
* Networking
* Load Balancing

---

# Kubernetes is NOT...

Many beginners misunderstand Kubernetes.

It is **NOT**

* a programming language
* a Docker replacement
* multiple tools combined
* a Continuous Integration tool
* a Continuous Deployment tool

Instead,

it coordinates multiple components to keep applications running in the desired state. This aligns with your trainer's emphasis that Kubernetes is coordination rather than a collection of tools or a continuous process. 

---

# Why was Docker introduced?

Before understanding Kubernetes,

we must understand

**Why Docker became popular.**

Every technology solves a problem left by the previous generation.

So the journey is

```text
Physical Servers
        │
        ▼
Virtual Machines
        │
        ▼
Containers (Docker)
        │
        ▼
Kubernetes
```

Each technology solved some problems.

Each technology also introduced new challenges.

The next technology came to solve those new challenges.

---

# Physical Server Era

Initially,

one application was deployed on one physical server.

Example

```text
Server-1
    └── Banking Application

Server-2
    └── HR Application

Server-3
    └── Inventory Application
```

Problems

* Very expensive
* Hardware underutilized
* Difficult to scale
* Long provisioning time
* Maintenance cost was high

---

# Virtualization Era

Virtualization solved hardware utilization.

One physical server could host multiple Virtual Machines.

```text
Physical Server

│

├── VM-1
│     └── Banking

├── VM-2
│     └── HR

├── VM-3
│     └── Inventory
```

Advantages

* Better hardware utilization
* Isolation between applications
* Easy server provisioning

---

# Problems with Virtual Machines

Every VM contains

* Guest Operating System
* Kernel
* Libraries
* Application

This means

* More RAM usage
* More CPU usage
* Larger storage
* Slower startup

Organizations wanted something lighter.

That is why containerization became popular.

---

# Kubernetes Day-1 Notes

# Part-2

## Docker, Containerization and Why Kubernetes Was Introduced

---

# Why Docker?

Before understanding Kubernetes, we must first understand Docker.

**Interview Question**

> Why was Docker introduced?

The answer is **not** "to create containers."

Docker was introduced to solve the problems of Virtual Machines.

---

# Problems with Virtual Machines

Suppose a company has **100 applications**.

Using Virtual Machines, the architecture looks like this.

```text
                    Physical Server

-------------------------------------------------------

VM-1
├── Guest OS
├── Kernel
├── Libraries
└── Application-1

VM-2
├── Guest OS
├── Kernel
├── Libraries
└── Application-2

VM-3
├── Guest OS
├── Kernel
├── Libraries
└── Application-3
```

Notice something.

Every VM has

* Complete Operating System
* Kernel
* System Libraries

Even though applications are small, every VM consumes a large amount of RAM, CPU and Storage.

This increases infrastructure cost.

---

# Solution → Containerization

Instead of creating a complete Operating System for every application,

Docker introduced **Containers**.

Containers share the Host Operating System Kernel.

```text
                 Host Machine

---------------------------------------------------

Host Linux Kernel

│

├── Container-1 → Banking Application

├── Container-2 → Payment Application

├── Container-3 → Inventory Application

├── Container-4 → Search Application

└── Container-5 → Notification Application
```

Unlike Virtual Machines,

Containers contain only

* Application
* Required Libraries
* Dependencies

They reuse the Host Kernel.

Your trainer highlighted that VMs include the full OS and kernel, whereas Docker provides a base OS and shares the host kernel. 

---

# Why are Containers Lightweight?

Because

they don't install another Operating System.

Instead,

they directly use the Host Kernel.

Therefore,

* Less RAM
* Less CPU
* Less Storage
* Faster Startup
* More Containers on same server

---

# Interview Question

## Can Linux run Windows Containers?

Answer

**No**

Why?

Because

Containers always use the Host Operating System Kernel.

Example

```text
Host Machine

Operating System → Linux

Kernel → Linux Kernel
```

Now try to run

```text
Windows Container
```

Will it work?

No.

Reason

Linux Kernel cannot understand Windows Kernel APIs.

Windows Containers require Windows Kernel.

Similarly,

Windows Host cannot directly execute Linux Containers.

Your trainer explained exactly this kernel dependency. 

---

# One Container = One Application

Docker best practice

One application should run inside one container.

Example

```text
Container-1

Payment Service

----------------

Container-2

Orders Service

----------------

Container-3

Inventory Service

----------------

Container-4

Cart Service
```

This provides

* Isolation
* Easy upgrades
* Independent deployments
* Better scalability

Your trainer emphasized that each application runs in its own individual container. 

---

# Did Docker Solve Everything?

**No.**

Docker solved

✔ Packaging Applications

✔ Running Applications

✔ Lightweight Deployments

But Docker does **not** manage applications automatically.

This is the biggest limitation.

---

# Production Scenario

Imagine Flipkart.

It has

* 800 Microservices
* 15,000 Containers

Everything is running perfectly.

Suddenly

Payment Container crashes.

What happens?

```text
Payment Container

↓

Stopped
```

Will Docker automatically create another container?

**No.**

Docker simply runs containers.

It does **not** monitor their health continuously.

If a container stops,

someone must manually restart it (or another external tool must do it).

Your trainer explicitly pointed out that if a container is deleted or stopped, Docker does not automatically recreate it. 

---

# Another Production Problem

Suppose

your application runs on EC2.

```text
EC2 Instance

↓

Running 25 Containers
```

Suddenly

EC2 crashes.

```text
EC2 Deleted
```

Question

Will another EC2 automatically come?

No.

Unless

AWS Auto Scaling Group (ASG)

is configured,

AWS will not create another EC2 automatically.

Your trainer used this comparison to explain the lack of High Availability without ASG. 

---

# Problems Before Kubernetes

Docker cannot automatically

❌ Restart failed containers

❌ Replace deleted containers

❌ Create new servers

❌ Balance workload

❌ Scale applications

❌ Maintain High Availability

Imagine managing

5000 containers manually.

Impossible.

Companies needed a platform that continuously watches applications.

---

# Birth of Kubernetes

Kubernetes was introduced to solve these production problems.

Docker creates containers.

Kubernetes manages containers.

Think of them as

```text
Docker

↓

"I know how to create containers."

-----------------------------------

Kubernetes

↓

"I know how to manage containers."
```

That is why Kubernetes is called

**Container Orchestration Platform.**

---

# What Does Kubernetes Actually Do?

Suppose you write one YAML file.

Inside that file you define

* Application Image
* Number of Replicas
* Container Port
* Labels
* Networking
* Resources

You simply submit the YAML to Kubernetes.

```text
manifest.yaml

↓

Kubernetes

↓

Reads Configuration

↓

Creates Required Pods

↓

Monitors Them Forever
```

Your trainer explained that instead of manually creating containers, we hand over a manifest YAML describing replicas, networking, ports, and other settings, and Kubernetes takes care of deployment. 

---

# Self-Healing

One of Kubernetes' biggest advantages.

Suppose

You requested

```text
Replicas = 5
```

Current Situation

```text
Pod-1

Pod-2

Pod-3

Pod-4

Pod-5
```

Everything is healthy.

Now

Pod-3 crashes.

Immediately Kubernetes notices

```text
Desired State = 5

Current State = 4
```

Mismatch detected.

Immediately

```text
Create New Pod
```

Final State

```text
Pod-1

Pod-2

Pod-3 (New)

Pod-4

Pod-5
```

Nobody logs into the server.

Nobody manually starts containers.

Kubernetes automatically restores the desired state.

This feature is called **Self-Healing**, exactly as highlighted in your trainer's notes. 

---

# Node Self-Healing

Container recovery is only one level.

Suppose

Entire Worker Node crashes.

```text
Worker Node-2

↓

Unavailable
```

All Pods on that node disappear.

Kubernetes immediately

* Detects unhealthy node
* Schedules Pods on healthy nodes
* If Cluster Autoscaler is configured, provisions a replacement worker node
* Moves workloads back according to scheduling rules

From the application's perspective,

service continues running.

Your trainer explained this as node-level self-healing and pod movement during node failures. 

---

# High Availability

High Availability means

The application should remain available even if

* Pods fail
* Containers fail
* Nodes fail

Kubernetes achieves this through

* Multiple Replicas
* Self-Healing
* Scheduling
* Load Balancing

Users should ideally not notice individual infrastructure failures.

---

# Important Day-1 Interview Questions

### Q1. Why was Docker introduced?

To solve the resource and management problems of Virtual Machines by introducing lightweight containers.

---

### Q2. Why was Kubernetes introduced?

Because Docker can create containers but cannot automatically manage large-scale containerized applications in production.

---

### Q3. Does Docker automatically recreate deleted containers?

No. Docker alone does not recreate stopped or deleted containers automatically. 

---

### Q4. What is Self-Healing?

Self-Healing is Kubernetes' ability to automatically detect failed Pods and recreate them so that the actual state matches the desired state.

---

### Q5. What is the relationship between Docker and Kubernetes?

Docker packages applications into images and runs containers.

Kubernetes uses those images to create and manage Pods, ensuring the applications remain healthy and available. 

---

Excellent. Let's continue with the same trainer perspective.

---

# Kubernetes Day-1 Notes

# Part-3

# HPA, ASG, Complete DevOps Flow & Why Kubernetes Became the Standard

---

# Kubernetes Features

Kubernetes became the industry standard because it provides several production-ready capabilities out of the box.

The major features are:

* Self-Healing
* High Availability
* Auto Scaling
* Load Balancing
* Service Discovery
* Rolling Updates
* Rollbacks
* Declarative Configuration

In today's class, the trainer mainly discussed **Self-Healing**, **High Availability**, and **Auto Scaling**. 

---

# Horizontal Pod Autoscaler (HPA)

## Production Problem

Suppose you deployed an E-Commerce application.

Initially,

```text
Users = 500

Pods = 3
```

Everything works perfectly.

Now suddenly,

Flipkart announces

**Big Billion Days Sale**

Within a few minutes

```text
500 Users

↓

5,00,000 Users
```

What happens?

The existing three Pods cannot handle all incoming requests.

Result

* High CPU Utilization
* High Memory Usage
* Slow Response
* Request Timeouts
* Users see errors

Without auto scaling,

the application may become unavailable.

---

# Solution

Kubernetes introduced

**Horizontal Pod Autoscaler (HPA)**

HPA continuously monitors Pod metrics such as

* CPU Utilization
* Memory Utilization (when configured)
* Custom Metrics
* External Metrics

Suppose

```text
Minimum Pods = 3

Maximum Pods = 15

Target CPU = 70%
```

Current Situation

```text
Pod-1

Pod-2

Pod-3
```

CPU becomes

```text
85%

90%

88%
```

Immediately HPA detects

```text
Current CPU > Target CPU
```

Kubernetes automatically creates additional Pods.

```text
Pod-1

Pod-2

Pod-3

Pod-4

Pod-5

Pod-6

Pod-7
```

Traffic is now distributed across more Pods.

When traffic reduces,

HPA automatically removes unnecessary Pods while respecting the configured minimum replica count.

Your trainer summarized this by explaining that Kubernetes controls Pods according to load—creating more Pods during heavy traffic and reducing them when demand decreases. 

---

# Real-Time Example

Suppose Amazon receives

```text
2 AM

Users = 2,000
```

Pods running

```text
5 Pods
```

During

Prime Day Sale

```text
Users = 20,00,000
```

HPA automatically increases Pods.

```text
5

↓

20

↓

50

↓

100
```

After the sale

Traffic decreases.

Kubernetes automatically scales back.

```text
100

↓

50

↓

20

↓

5
```

No DevOps Engineer needs to manually start or stop Pods.

---

# Interview Question

### Why do we need HPA?

Because application traffic is never constant.

Some applications experience

* Festival sales
* Product launches
* Cricket matches
* Flash sales
* Breaking news

Manual scaling is too slow.

HPA allows Kubernetes to automatically adjust the number of Pods based on demand.

---

# Difference between ASG and HPA

This is one of the most frequently asked interview questions.

| AWS Auto Scaling Group (ASG)               | Kubernetes Horizontal Pod Autoscaler (HPA)     |
| ------------------------------------------ | ---------------------------------------------- |
| Works at EC2 Instance Level                | Works at Pod Level                             |
| Creates or Terminates EC2 Instances        | Creates or Terminates Pods                     |
| Managed by AWS                             | Managed by Kubernetes                          |
| Used when compute capacity is insufficient | Used when application capacity is insufficient |
| Infrastructure Scaling                     | Application Scaling                            |

### Visual Understanding

```text
                    AWS

              Auto Scaling Group

                       │

      Creates More EC2 Instances

                       │

                       ▼

        Worker Node-1

        Worker Node-2

        Worker Node-3

---------------------------------------------------

                Kubernetes

          Horizontal Pod Autoscaler

                       │

       Creates More Application Pods

                       │

                       ▼

Pod-1

Pod-2

Pod-3

Pod-4

Pod-5
```

Simple way to remember

> ASG scales Servers.

> HPA scales Applications.

This matches your trainer's explanation comparing ASG at the instance level and HPA at the Pod level. 

---

# Kubernetes Deployment Philosophy

Unlike Docker,

where you manually execute commands,

Kubernetes follows

**Declarative Configuration**

Instead of saying

```text
Run this container.
```

You simply provide a YAML file.

Example

```yaml
replicas: 5

image: payment:v1

port: 8080
```

Kubernetes continuously ensures that

Actual State

always matches

Desired State.

This is called

**Desired State Management**

Your trainer explained that we simply hand over a manifest YAML containing replicas, networking, ports, and other configuration, and Kubernetes takes care of deployment and management. 

---

# Why Kubernetes Needs Docker Images

One common interview question is

**Can Kubernetes run source code directly?**

No.

Developers write

```text
Python

Java

NodeJS

.NET

Go
```

Kubernetes cannot execute source code.

It requires

Container Images.

Therefore,

Docker first converts the application into an image.

Example

```text
Application

↓

Dockerfile

↓

Docker Build

↓

Docker Image

↓

Container Registry (ECR)

↓

Kubernetes

↓

Pods
```

Your trainer summarized this by stating that Docker builds images, stores them in ECR (or another artifact repository), and Kubernetes consumes those images to create Pods. 

---

# Docker vs Kubernetes

Many beginners think Kubernetes replaces Docker.

This is incorrect.

Docker and Kubernetes solve different problems.

| Docker                 | Kubernetes            |
| ---------------------- | --------------------- |
| Builds Images          | Uses Images           |
| Creates Containers     | Creates Pods          |
| Packages Applications  | Manages Applications  |
| Single Machine Focus   | Cluster Focus         |
| No Native Self-Healing | Built-in Self-Healing |
| Limited Scaling        | Automatic Scaling     |

Simple Interview Answer

Docker is responsible for packaging the application.

Kubernetes is responsible for managing the packaged application in production.

---

# Pod vs Container

Your trainer gave a very simple way to remember this.

```text
Docker

↓

Image

↓

Container

----------------------------

Kubernetes

↓

Image

↓

Pod
```

You can think of

Pod

as the smallest deployable unit in Kubernetes.

For beginners,

A Pod behaves similarly to a container, although internally it provides additional Kubernetes-specific functionality.

Your trainer emphasized this comparison during the introduction. 

---

# Day-1 Interview Questions

### Q1. Why is HPA required?

To automatically increase or decrease the number of Pods based on application load.

---

### Q2. Difference between ASG and HPA?

ASG scales infrastructure (EC2 Instances).

HPA scales application Pods.

---

### Q3. Can Kubernetes build Docker Images?

No.

Docker or another container build tool creates the image.

Kubernetes only deploys and manages containers from those images.

---

### Q4. Why does Kubernetes use YAML?

Because Kubernetes follows a declarative model where users describe the desired state, and Kubernetes continuously works to maintain it.

---

### Q5. What is the relationship between Docker and Kubernetes?

Docker packages applications into images.

Kubernetes deploys those images as Pods and continuously manages them.

---

## End of Part-3

The next part will cover the complete **production DevOps workflow** from your notes:

* Developer → GitHub
* Dockerfile
* CI Pipeline
* ECR
* Kubernetes Manifest Files
* Argo CD
* Terraform Pipeline
* Why Terraform is executed from pipelines
* The complete 3-pipeline architecture (Terraform → CI → CD)

This section is one of the most important parts of your Day-1 notes because it connects all the tools into a real production deployment flow. 

---

Excellent. This is the most important part of Day-1 because now all the DevOps tools connect together. Your trainer shifted from individual tools to **how a real production deployment works**. The notes below are based on your uploaded notes and preserve the trainer's flow while expanding the production reasoning. 

---

# Kubernetes Day-1 Notes

# Part-4

# Complete DevOps Workflow (Developer → Production)

---

# Big Picture

Most beginners learn DevOps tools separately.

```
Git
Docker
Terraform
Kubernetes
Jenkins
GitHub Actions
ArgoCD
AWS
```

This is the wrong approach.

In a real company, these tools work together to deploy an application from a developer's laptop to production.

The objective of DevOps is **not learning tools**.

The objective is

> **Deliver application changes to production quickly, reliably and consistently.**

---

# Complete Production Flow

```
Developer
     │
     ▼
GitHub Repository
     │
     ▼
CI Pipeline
(GitHub Actions / Jenkins)
     │
     ▼
Docker Image Build
     │
     ▼
Amazon ECR
(Image Repository)
     │
     ▼
ArgoCD
     │
     ▼
Kubernetes Cluster
     │
     ▼
Pods
     │
     ▼
Service
     │
     ▼
Load Balancer
     │
     ▼
End Users
```

Everything in DevOps revolves around this flow.

---

# Step-1 Developer Writes Application

A developer writes application code.

Example

```
app.py

requirements.txt

pom.xml

package.json
```

These files belong to the development team.

Example

```
GitHub Repository

│

├── app.py

├── requirements.txt

├── package.json
```

The application still **cannot run inside Kubernetes**.

Why?

Because Kubernetes does not understand source code.

It only understands **Container Images**. 

---

# Step-2 DevOps Engineer's Responsibility

Once the developer finishes the application,

the DevOps Engineer prepares deployment files.

Typical DevOps files are

```
Dockerfile

CI Pipeline

Terraform Files

Kubernetes YAML Files
```

Notice the separation of responsibilities.

| Developer                 | DevOps Engineer        |
| ------------------------- | ---------------------- |
| Writes Application        | Creates Infrastructure |
| Fixes Bugs                | Builds Images          |
| Implements Business Logic | Creates CI/CD          |
| Writes Source Code        | Deploys Application    |

Your trainer explicitly listed Dockerfile, CI/CD files and Terraform files as DevOps-owned artifacts. 

---

# Step-3 Dockerfile

Now we must convert

```
Python Code

↓

Container Image
```

Docker cannot build an image without instructions.

Those instructions are written in

```
Dockerfile
```

Example

```
FROM python:3.12

COPY .

RUN pip install

CMD python app.py
```

Docker reads the Dockerfile

↓

Builds Image

↓

Produces

```
payment:v1

inventory:v1

cart:v1
```

Docker images are immutable packages containing the application and its dependencies.

---

# Why Images?

Interview Question

**Why can't Kubernetes deploy source code directly?**

Because

source code

```
app.py
```

cannot execute by itself.

It first needs

* Python Runtime
* Dependencies
* Libraries
* Operating System Components

Docker packages everything together into one image.

Now Kubernetes simply runs that image.

---

# Step-4 Push Image to Registry

Building an image locally is not enough.

Imagine

```
Developer Laptop

↓

Docker Image
```

Can Kubernetes running inside AWS access your laptop?

No.

Therefore,

the image must be stored in a centralized repository.

Example

```
Amazon ECR

Docker Hub

JFrog Artifactory

Harbor
```

Your trainer used Amazon ECR as the example artifact repository. 

---

# Why ECR?

Suppose

10 Worker Nodes exist.

Every Worker Node needs the same image.

Instead of copying images manually,

all Worker Nodes pull images from ECR.

```
Amazon ECR

│

├── payment:v1

├── inventory:v1

├── cart:v1

└── search:v1
```

Worker Nodes download images whenever Pods are created.

---

# Step-5 Kubernetes Deployment

Now Kubernetes receives

```
deployment.yaml
```

Example

```
Image

Replicas

Ports

Resources

Labels

Environment Variables
```

Kubernetes reads

```
deployment.yaml

↓

Image = payment:v1

↓

Pull Image From ECR

↓

Create Pods
```

Everything is automated.

Your trainer emphasized that Kubernetes requires images and uses manifest YAML files to deploy them. 

---

# Why Manifest YAML?

Interview Question

Why do we write YAML?

Because Kubernetes follows

**Declarative Configuration.**

Instead of saying

```
Create Pod
```

we say

```
I want

3 Pods

Image payment:v1

Port 8080
```

Kubernetes continuously ensures that the cluster matches the desired configuration.

---

# Step-6 Service

Pods are temporary.

Suppose

```
Pod-2

↓

Deleted
```

Another Pod is created.

Its IP changes.

Users should never access Pods directly.

Instead,

they access

```
Service

↓

Stable Endpoint
```

Although your Day-1 notes only briefly mention users accessing Kubernetes services, this is the production reason behind that statement. The notes themselves state that end users access the service created by Kubernetes. 

---

# Step-7 Load Balancer

Finally

Traffic reaches

```
Internet

↓

Load Balancer

↓

Kubernetes Service

↓

Pods
```

The Load Balancer distributes requests among healthy Pods.

Users never know

which Pod handled their request.

---

# Continuous Integration (CI)

Now imagine

Developer changes one line of code.

Should the DevOps Engineer manually

* Build Docker Image?
* Push Image?
* Verify Everything?

No.

This becomes repetitive.

So we automate it.

This automation is called

**Continuous Integration (CI).**

CI responsibilities

```
Source Code

↓

Build

↓

Test

↓

Docker Build

↓

Docker Push

↓

Store Image in ECR
```

GitHub Actions and Jenkins are common CI tools. Your trainer specifically mentioned GitHub Actions, GitLab CI/CD and Jenkins for CI. 

---

# Continuous Deployment (CD)

Once the image reaches ECR,

someone still needs to deploy it.

This responsibility belongs to

**Continuous Deployment (CD).**

Flow

```
ECR

↓

New Image

↓

ArgoCD Detects Change

↓

Syncs Kubernetes

↓

Deploys New Version
```

Your trainer highlighted Argo CD as the preferred Kubernetes-native CD tool. 

---

# Why Argo CD?

Interview Question

Can GitHub Actions deploy to Kubernetes?

Yes.

Can Jenkins deploy to Kubernetes?

Yes.

Then why Argo CD?

Because

Argo CD is

**Kubernetes Native.**

It understands

* Kubernetes Resources
* Desired State
* GitOps
* Automatic Synchronization

It continuously compares

```
Git Repository

vs

Kubernetes Cluster
```

If both differ,

Argo CD synchronizes them automatically.

Your trainer also pointed out that Argo CD focuses on deployment and is not intended for Docker build or image push operations. 

---

# Interview Questions

### Q1. Why do we need Dockerfile?

To convert application source code into a Docker Image.

---

### Q2. Why do we push images to ECR?

Because Kubernetes Worker Nodes cannot pull images directly from a developer's laptop. A centralized image registry is required.

---

### Q3. What is the responsibility of CI?

* Build Application
* Test Application
* Build Docker Image
* Push Image to Registry

---

### Q4. What is the responsibility of CD?

Deploy the image from the registry into Kubernetes.

---

### Q5. Why is Argo CD preferred?

Because it is Kubernetes-native and continuously synchronizes the desired state defined in Git with the actual state of the Kubernetes cluster.

---

## End of Part-4

The **next and final part of Day-1** will cover your trainer's most practical production discussion:

* **Why Terraform runs in pipelines (not just automation)**
* **Three-pipeline architecture (Terraform → CI → CD)**
* **10 applications scenario**
* **10 Dockerfiles, 10 Images, 10 YAMLs**
* **1 Pipeline vs 10 Pipelines**
* **Path-based routing using a single Load Balancer**

These are excellent interview topics because they explain **how large production systems are actually organized**. 

---

# Kubernetes Day-1 Notes

# Part-5 (Final)

# Terraform Pipeline, CI/CD Strategy & Realtime Architecture

This is one of the most important discussions from Day-1 because your trainer explained **how DevOps works in real production** instead of just teaching tools individually. These concepts are frequently discussed in interviews for DevOps Engineers with 2–8 years of experience. 

---

# Why do we need Terraform Pipeline?

This is one of the most misunderstood concepts.

Many beginners think

> We use pipelines to automate Terraform.

Your trainer said

> **No. That is not the real reason.**

Automation is only a secondary benefit.

The primary reason is **Consistency**.

---

# Problem Before Terraform Pipeline

Suppose there are three DevOps Engineers.

```text
Sairaj

Arun

Hitesh
```

All three need to create AWS Infrastructure.

Without a pipeline,

everyone executes Terraform from their laptops.

```text
             AWS Infrastructure

                   ▲

      -----------------------------

      ▲             ▲            ▲

Sairaj Laptop   Arun Laptop   Hitesh Laptop
```

Looks fine.

But in production this creates many problems.

---

# Production Problems

## Problem-1 Different Terraform Versions

```text
Sairaj

Terraform 1.7

------------------------

Arun

Terraform 1.9

------------------------

Hitesh

Terraform 1.10
```

Each version may behave differently.

Result

* Different execution
* Unexpected warnings
* Provider incompatibility
* Production failures

---

## Problem-2 Different Provider Versions

Suppose

```text
Sairaj

AWS Provider

5.45

------------------------

Arun

AWS Provider

6.0
```

Now

same Terraform code

different outputs.

---

## Problem-3 Missing Credentials

Suppose

```text
Sairaj

AWS Credentials Configured

✔

------------------------

Arun

Credentials Missing

✘
```

Terraform execution fails.

---

## Problem-4 Operating System Difference

```text
Windows

Linux

Mac
```

Different environments.

Different plugins.

Different PATH variables.

Different execution.

---

## Problem-5 Human Dependency

Imagine

Arun knows how to deploy infrastructure.

Today

Arun is on leave.

Nobody else knows his laptop configuration.

Deployment gets delayed.

Production teams never want deployments to depend on one person's laptop.

---

# Solution

Instead of running Terraform locally,

everyone uses

ONE common pipeline.

```text
                 GitHub

                   │

                   ▼

        Terraform Pipeline

────────────────────────────────

Terraform Version

AWS CLI

Provider Version

Backend

Credentials

Plugins

────────────────────────────────

                   │

                   ▼

             AWS Infrastructure
```

Now

Sairaj

↓

Arun

↓

Hitesh

all execute the same pipeline.

Everybody uses

* Same Terraform Version
* Same Provider Version
* Same Credentials
* Same Backend
* Same Plugins

Result

Consistent Infrastructure.

This is exactly what your trainer explained—that Terraform pipelines are primarily about providing a fixed/common environment rather than simply automating `terraform init`, `plan`, and `apply`. 

---

# Interview Question

### Why do companies execute Terraform through CI Pipelines?

Answer

Not only for automation.

Main reasons

* Consistent Environment
* Version Control
* Team Collaboration
* Auditability
* Security
* Standardized Execution

Automation is an additional benefit.

---

# Three Pipelines in Production

Suppose a completely new project starts.

What pipelines are required?

Many freshers answer

CI Pipeline.

Wrong.

Infrastructure must exist before applications can be deployed.

Therefore

Pipeline-1

comes first.

---

# Pipeline-1

## Terraform Pipeline

Purpose

Create Infrastructure.

Creates

* VPC
* Subnets
* Route Tables
* Internet Gateway
* NAT Gateway
* IAM Roles
* EKS Cluster
* Worker Nodes
* Load Balancer Prerequisites
* Security Groups

Flow

```text
GitHub

↓

Terraform Pipeline

↓

Terraform Init

↓

Terraform Plan

↓

Terraform Apply

↓

AWS Infrastructure Ready
```

Without infrastructure,

nothing else can be deployed.

Your trainer clearly stated that the Terraform pipeline is the first pipeline in a project. 

---

# Pipeline-2

## Continuous Integration (CI)

Infrastructure is ready.

Developers now push source code.

Pipeline responsibilities

```text
Source Code

↓

Compile

↓

Unit Tests

↓

Docker Build

↓

Docker Image

↓

Push to Amazon ECR
```

Notice

CI stops after pushing the image.

No deployment yet.

Your trainer described the CI pipeline's responsibility as building the image and pushing it to ECR. 

---

# Pipeline-3

## Continuous Deployment (CD)

Once the image is available,

Deployment starts.

Flow

```text
Amazon ECR

↓

ArgoCD

↓

Kubernetes

↓

Pods Updated
```

Responsibilities

* Pull latest image
* Update Deployment
* Rolling Update
* Health Verification
* Sync Desired State

Your trainer recommended Argo CD as the preferred Kubernetes-native CD tool. 

---

# Complete Production Architecture

```text
                Developer

                     │

                     ▼

                 GitHub

                     │

        -------------------------

        │                       │

        ▼                       ▼

Terraform Pipeline         CI Pipeline

        │                       │

        ▼                       ▼

AWS Infrastructure      Docker Image

                                │

                                ▼

                            Amazon ECR

                                │

                                ▼

                         Argo CD (CD)

                                │

                                ▼

                        Kubernetes Cluster

                                │

                                ▼

                               Pods

                                │

                                ▼

                           End Users
```

---

# Can CI and CD be Combined?

Yes.

Suppose the company wants

every code change

↓

immediately deployed.

Flow

```text
Developer Push

↓

Build Image

↓

Push Image

↓

Deploy

↓

Done
```

Only one pipeline.

Your trainer explained that in some projects CI and CD are combined. 

---

# When Do We Separate CI and CD?

Large organizations usually separate them.

Reason

Sometimes

Image is built today.

Deployment happens tomorrow.

Example

Developer finishes coding.

↓

Image stored in ECR.

↓

Testing Team approves.

↓

Release Team approves.

↓

Deployment during midnight maintenance window.

Therefore

Separate CD Pipeline.

This aligns with your trainer's explanation that images can be stored in ECR first and deployed later using a dedicated CD pipeline. 

---

# Why Argo CD?

Argo CD only performs

Deployment.

It does NOT

* Build Docker Images
* Push Images

Those responsibilities belong to CI.

Argo CD continuously synchronizes Kubernetes with the desired configuration stored in Git.

Your trainer specifically pointed out that Argo CD is not used for Docker build or push operations. 

---

# Realtime Scenario

Suppose your company has

10 Applications.

```text
Payment

Orders

Cart

Inventory

Search

Profile

Offers

Reviews

Notifications

Admin
```

---

# How many Dockerfiles?

Each application has

different

* Runtime
* Dependencies
* Libraries

Therefore

```text
10 Applications

↓

10 Dockerfiles
```

---

# How many Images?

Every Dockerfile

↓

One Docker Image.

```text
10 Applications

↓

10 Images
```

---

# How many Git Repositories?

Most companies follow

```text
One Application

↓

One Repository
```

Result

```text
10 Applications

↓

10 Git Repositories
```

Your trainer explained this approach as the common case for independently managed applications. 

---

# How many Kubernetes YAML Files?

Every application requires

its own Deployment.

Therefore

```text
10 Applications

↓

10 Deployment YAML Files
```

Each YAML references

one Docker Image.

Exactly as your trainer noted. 

---

# How many CI Pipelines?

Interview Question

Answer

Depends on Business.

## Case-1

Independent Applications

```text
Payment

Orders

Inventory
```

Each team deploys independently.

Then

```text
10 Applications

↓

10 Pipelines
```

One application changes.

Only one pipeline executes.

---

## Case-2

Single Product

Suppose

all applications belong

to one product.

Then

one master pipeline

can coordinate builds and deployments.

Therefore

```text
10 Applications

↓

1 Pipeline

or

10 Pipelines

depends on Business Requirement.
```

Exactly matching your trainer's explanation that pipeline count depends on business logic and deployment independence. 

---

# How many Load Balancers?

Many beginners answer

10.

Wrong.

Usually

One Load Balancer

is enough.

It performs

Path-Based Routing.

Example

```text
https://shop.com/cart

↓

Cart Service

--------------------------------

https://shop.com/payment

↓

Payment Service

--------------------------------

https://shop.com/orders

↓

Orders Service
```

One Load Balancer

routes traffic

to different services.

Your trainer specifically highlighted using a single Load Balancer with path-based routing. 

---

# Day-1 Summary

By the end of Day-1, the complete story is:

```text
Physical Servers
        │
        ▼
Virtual Machines
        │
        ▼
Docker (Containerization)
        │
        ▼
Problems Managing Containers
        │
        ▼
Kubernetes (Orchestration)
        │
        ▼
Infrastructure Created by Terraform
        │
        ▼
Application Packaged by Docker
        │
        ▼
CI Builds Image
        │
        ▼
Amazon ECR
        │
        ▼
Argo CD Deploys
        │
        ▼
Kubernetes Runs Pods
        │
        ▼
Load Balancer
        │
        ▼
End Users
```

This completes your **Day-1 Kubernetes Notes**, covering all the major discussions from your uploaded running notes in a clearer, production-oriented trainer style while preserving the original flow and intent. 

