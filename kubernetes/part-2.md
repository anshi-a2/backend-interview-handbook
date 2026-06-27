# End-to-End Kubernetes Deployment Pipeline

> **Goal**
>
> Understand the complete lifecycle of an application—from writing code locally to serving requests in Kubernetes. This chapter explains every component involved and how they collaborate.

---

# Table of Contents

1. Deployment Overview
2. Step 1: Write Code
3. Step 2: Git Push
4. Step 3: CI/CD Pipeline
5. Step 4: Docker Build
6. Step 5: Push Image to Registry
7. Step 6: Deploy to Kubernetes
8. Step 7: API Server Request Flow
9. Step 8: Controllers & Reconciliation
10. Step 9: Scheduling
11. Step 10: Pod Creation
12. Step 11: Health Checks
13. Step 12: Service & Traffic
14. Complete Sequence Diagram
15. Common Failures
16. Interview Summary

---

# 1. Deployment Overview

A production deployment typically looks like this:

```
Developer

↓

Git Push

↓

GitHub

↓

CI/CD

↓

Build & Test

↓

Docker Image

↓

Container Registry

↓

Helm / kubectl

↓

Kubernetes API Server

↓

Deployment

↓

ReplicaSet

↓

Pods

↓

Scheduler

↓

Worker Node

↓

kubelet

↓

containerd

↓

Running Pod

↓

Readiness Probe

↓

Service

↓

Ingress

↓

Load Balancer

↓

Users
```

Every arrow represents communication between components.

---

# 2. Step 1: Developer Writes Code

Example

```
Spring Boot

NodeJS

Python

Go
```

Developer creates

```
Application Code

Dockerfile

deployment.yaml

service.yaml
```

Example

```yaml
apiVersion: apps/v1
kind: Deployment

spec:
  replicas: 3
```

This YAML does **not** create Pods directly.

It simply declares the desired state.

---

# 3. Step 2: Git Push

Developer commits code.

```
git add .

git commit

git push
```

Repository

```
GitHub

GitLab

Bitbucket
```

A webhook triggers the CI/CD pipeline.

---

# 4. Step 3: CI/CD Pipeline

Typical stages

```
Checkout Code

↓

Compile

↓

Unit Tests

↓

Static Analysis

↓

Integration Tests

↓

Docker Build

↓

Push Image

↓

Deploy
```

Common tools

- Jenkins
- GitHub Actions
- GitLab CI
- Azure DevOps

If any stage fails, deployment stops.

---

# 5. Step 4: Docker Build

Docker reads the Dockerfile.

Example

```dockerfile
FROM eclipse-temurin:21

COPY target/app.jar app.jar

ENTRYPOINT ["java","-jar","app.jar"]
```

Docker creates

```
Image Layers
```

Benefits

- Reusable
- Cached
- Immutable

Example image

```
payment-service:v2
```

---

# 6. Step 5: Push Image to Registry

Image uploaded to

```
Docker Hub

AWS ECR

Google GCR

Azure ACR
```

Later, Kubernetes pulls the image from here.

---

# 7. Step 6: Deploy to Kubernetes

Deployment can happen using

```
kubectl apply

or

Helm

or

ArgoCD
```

Example

```bash
kubectl apply -f deployment.yaml
```

`kubectl` converts the YAML into REST API requests.

Example

```
POST /apis/apps/v1/deployments
```

Request goes to the API Server.

---

# 8. API Server Request Flow

The API Server is the entry point for all Kubernetes operations.

Flow

```
kubectl

↓

API Server

↓

Authentication

↓

Authorization

↓

Admission Controllers

↓

Validation

↓

Persist to etcd

↓

Response
```

## Authentication

Verifies the identity of the caller.

Examples

- Certificate
- Token
- Service Account

---

## Authorization

Checks permissions.

Usually using RBAC.

Example

```
Can this user create Deployments?
```

---

## Admission Controllers

Modify or reject requests.

Examples

- Default values
- Image policy
- Security validation

---

## Validation

Checks

- YAML syntax
- Required fields
- API version

Invalid requests are rejected.

---

## Store in etcd

Deployment object stored.

Important:

Only metadata is stored.

No Pods are running yet.

---

# 9. Controllers & Reconciliation

Deployment Controller continuously watches the API Server.

It notices

```
New Deployment
```

It creates

```
ReplicaSet
```

ReplicaSet notices

```
Desired Pods = 3

Current Pods = 0
```

Creates

```
Pod1

Pod2

Pod3
```

Initially

```
Node = null
```

Pods are created but not scheduled.

---

# 10. Scheduling

Scheduler watches

```
Pods without node assignment
```

Scheduling phases

```
Filter

↓

Score

↓

Bind
```

## Filter

Remove unsuitable nodes.

Example

```
Need 4 CPU

Node has 2 CPU

Reject
```

---

## Score

Remaining nodes are scored.

Factors

- Available CPU
- Memory
- Affinity
- Taints
- Image locality

Highest score wins.

---

## Bind

Scheduler updates Pod

```
nodeName = worker-2
```

Only metadata changes.

Container still hasn't started.

---

# 11. Pod Creation

kubelet running on worker-2 notices

```
New Pod assigned
```

kubelet asks containerd

```
Create Pod
```

containerd performs

```
Pull Image

↓

Create Sandbox

↓

Setup Network

↓

Mount Volumes

↓

Create Container

↓

Start Container
```

If image already exists

Image pull is skipped.

---

# 12. Health Checks

Container starts.

Pod status

```
Running
```

Running **does not mean ready**.

Kubernetes now executes

```
Startup Probe

↓

Readiness Probe

↓

Liveness Probe
```

---

## Startup Probe

Used for slow-starting applications.

Until successful

- Liveness disabled
- Readiness disabled

---

## Readiness Probe

Determines

```
Can this Pod receive traffic?
```

If probe fails

```
Container Running

Service ignores Pod
```

Application is alive

Users cannot access it.

---

## Liveness Probe

Determines

```
Is application healthy?
```

Failure

↓

kubelet kills container

↓

Restarts container

---

# 13. Service & Traffic

Readiness succeeds.

Endpoints Controller adds Pod to Service.

```
Client

↓

Ingress

↓

Service

↓

Pod
```

Traffic begins.

---

# Complete Deployment Flow

```
Developer

↓

Git Push

↓

CI/CD

↓

Docker Build

↓

Registry

↓

kubectl apply

↓

API Server

↓

Authentication

↓

Authorization

↓

Admission Controllers

↓

Validation

↓

etcd

↓

Deployment Controller

↓

ReplicaSet

↓

Pods

↓

Scheduler

↓

Worker Node

↓

kubelet

↓

containerd

↓

Image Pull

↓

Container

↓

Startup Probe

↓

Readiness Probe

↓

Service Endpoint

↓

Ingress

↓

Load Balancer

↓

User Request
```

---

# What Happens When a Request Comes?

```
Browser

↓

DNS

↓

Load Balancer

↓

Ingress Controller

↓

Service

↓

kube-proxy

↓

Pod IP

↓

Application

↓

Database

↓

Response
```

The response follows the reverse path back to the client.

---

# Common Deployment Failures

## ImagePullBackOff

Reason

- Wrong image
- Registry unavailable
- Authentication issue

---

## Pending Pod

Reason

- No resources
- Scheduler unable to find node
- Taints

---

## CrashLoopBackOff

Reason

- Application exits immediately
- Configuration issue
- Missing environment variable

---

## Readiness Probe Failure

Application starts

But

Service never sends traffic.

---

## Liveness Probe Failure

Application becomes unhealthy.

kubelet repeatedly restarts it.

---

# Interview Summary

### Explain Kubernetes Deployment Internally

1. Developer pushes code to Git.
2. CI/CD builds the Docker image and pushes it to a registry.
3. `kubectl` (or Helm/ArgoCD) sends the Deployment manifest to the Kubernetes API Server.
4. The API Server authenticates, authorizes, validates, and stores the Deployment in etcd.
5. The Deployment Controller creates a ReplicaSet.
6. The ReplicaSet creates the required Pod objects.
7. The Scheduler selects suitable worker nodes and binds Pods to them.
8. The kubelet on each node observes the assigned Pods and asks the container runtime to create them.
9. The runtime pulls the image, creates networking, mounts volumes, and starts the container.
10. After the Readiness Probe succeeds, the Pod is added to the Service endpoints and starts receiving traffic through the Service and Ingress.
11. Kubernetes continuously monitors the Deployment. If Pods fail or nodes become unavailable, controllers reconcile the actual state back to the desired state automatically.

---

# Key Takeaways

- `kubectl apply` **does not create containers directly**.
- The API Server is the central entry point.
- etcd stores the desired cluster state.
- Controllers create Kubernetes resources.
- The Scheduler only assigns nodes.
- kubelet creates Pods on worker nodes.
- containerd runs the actual containers.
- Readiness controls **traffic**.
- Liveness controls **restarts**.
- Services expose Pods using stable networking.
- Kubernetes continuously reconciles actual state with desired state.
