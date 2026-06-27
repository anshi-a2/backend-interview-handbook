# 08_Kubernetes_Troubleshooting_Interview_Guide.md

> **Goal**
>
> Learn how senior engineers debug Kubernetes production issues and confidently answer Kubernetes interview questions.

---

# Table of Contents

1. Troubleshooting Methodology
2. Pod Issues
3. Deployment Issues
4. Node Issues
5. Networking Issues
6. Storage Issues
7. Performance Issues
8. Production Incident Examples
9. Debugging Flowcharts
10. Essential kubectl Commands
11. Top Kubernetes Interview Questions
12. Senior Interview Tips

---

# 1. Troubleshooting Methodology

Never randomly execute commands.

Always troubleshoot layer by layer.

```
Application

↓

Container

↓

Pod

↓

Deployment

↓

Service

↓

Ingress

↓

Node

↓

Cluster
```

Ask these questions:

1. Is the application running?
2. Is the Pod healthy?
3. Is the Deployment healthy?
4. Is the Service routing traffic?
5. Is the Ingress configured correctly?
6. Is the Node healthy?
7. Is the Cluster healthy?

---

# Production Debugging Workflow

```
User reports issue

↓

Can Pod be found?

↓

YES

↓

Running?

↓

YES

↓

Ready?

↓

YES

↓

Service endpoints?

↓

YES

↓

Ingress?

↓

YES

↓

Application logs

↓

Database

↓

Resolved
```

---

# 2. Pod Issues

---

## Problem 1

### Pod stuck in Pending

```
kubectl describe pod <pod>
```

Possible Reasons

- Insufficient CPU
- Insufficient Memory
- Node Selector mismatch
- Taints
- PVC Pending

Check

```
Events

↓

Scheduler Message
```

Example

```
0/5 nodes available

Insufficient memory
```

Solution

- Add nodes
- Increase cluster size
- Reduce requests

---

## Problem 2

### ImagePullBackOff

Reason

Image cannot be downloaded.

Check

```
Image Name

↓

Registry

↓

Credentials

↓

Image Tag
```

Commands

```bash
kubectl describe pod
```

---

## Problem 3

### CrashLoopBackOff

Meaning

```
Start

↓

Crash

↓

Restart

↓

Crash

↓

Restart
```

Common Reasons

- Application exception
- Missing Secret
- Wrong ConfigMap
- Invalid environment variable

Commands

```bash
kubectl logs pod

kubectl logs pod --previous
```

---

## Problem 4

### OOMKilled

Meaning

Memory limit exceeded.

Symptoms

```
Exit Code 137
```

Check

```
kubectl describe pod
```

Solution

- Increase memory
- Fix memory leak
- Tune JVM

---

## Problem 5

### Pod Running but Users Cannot Access

Check

```
Readiness Probe
```

Remember

```
Running

≠

Ready
```

---

# 3. Deployment Issues

---

## Deployment Not Updating

Check

```bash
kubectl rollout status deployment app
```

Possible Reasons

- Readiness failure
- Image pull failure
- CrashLoopBackOff

---

## Rolling Update Stuck

Check

```
kubectl get rs

↓

kubectl describe pod
```

Usually

New Pods never become Ready.

---

## Wrong Version Running

Check

```
kubectl describe deployment
```

Possible Reasons

- Old image tag
- Image cache
- GitOps sync delay

---

## Rollback

```bash
kubectl rollout undo deployment app
```

---

# 4. Node Issues

---

## Node NotReady

Check

```bash
kubectl get nodes
```

Possible Reasons

- kubelet stopped
- Network issue
- Disk pressure
- Memory pressure

---

## Disk Pressure

Symptoms

Pods evicted.

Check

```bash
kubectl describe node
```

---

## Memory Pressure

Node starts evicting Pods.

Lowest QoS Pods removed first.

---

## CPU Pressure

Symptoms

- Slow Pods
- High latency

Check

```bash
kubectl top node
```

---

# 5. Networking Issues

---

## Service Not Working

Check

```bash
kubectl get svc
```

Then

```bash
kubectl get endpoints
```

If endpoints empty

Problem is

- Labels
- Selector
- Readiness

---

## DNS Failure

Test

```bash
kubectl exec pod -- nslookup payment
```

Possible Reasons

- CoreDNS
- Wrong Service Name

---

## Ingress Returns 503

Check

```
Ingress

↓

Service

↓

Endpoints

↓

Readiness
```

Usually

No healthy endpoints.

---

## Pod Cannot Reach Database

Check

- Network Policy
- Service
- DNS
- Firewall

---

# 6. Storage Issues

---

## PVC Pending

Check

```bash
kubectl describe pvc
```

Possible Reasons

- StorageClass
- PV unavailable
- Capacity mismatch

---

## Volume Mount Failed

Check

```
Pod Events
```

Possible Reasons

- Missing Secret
- Missing ConfigMap
- Missing PVC

---

# 7. Performance Issues

---

## High CPU

Check

```bash
kubectl top pod
```

Possible Reasons

- Infinite loop
- Heavy traffic
- Missing autoscaling

---

## High Memory

Check

```
Heap Usage

GC

Memory Leak
```

---

## Slow Response

Investigate

```
Application

↓

Redis

↓

Database

↓

Network

↓

CPU
```

---

## High Restart Count

Check

```bash
kubectl get pods
```

Then

```
Describe

↓

Logs

↓

Events
```

---

# 8. Production Incident Examples

---

## Incident 1

### Pod Restarts Every 5 Minutes

Investigation

```
kubectl describe pod

↓

Liveness Probe Failed
```

Root Cause

Health endpoint timing out.

Solution

Increase timeout.

---

## Incident 2

### Application Returns 503

Investigation

```
Service

↓

Endpoints Empty
```

Reason

Readiness Probe failing.

---

## Incident 3

### Deployment Never Completes

```
kubectl rollout status
```

Reason

New Pods not Ready.

---

## Incident 4

### Users Experience Slow API

Investigation

```
CPU

↓

Database

↓

Trace

↓

GC
```

Found

Database queries taking

```
5 seconds
```

Not Kubernetes issue.

---

## Incident 5

### All Pods Suddenly Restart

Possible Reasons

- Node reboot
- Cluster upgrade
- Manual rollout
- OOM

Always verify

```
Events
```

---

# 9. Debugging Flowcharts

---

## Pod Crash

```
Pod Restart

↓

Logs

↓

Events

↓

Describe

↓

OOM?

↓

Probe?

↓

Application?
```

---

## Service Issue

```
Service

↓

Endpoints

↓

Readiness

↓

Labels

↓

Pods
```

---

## Deployment Issue

```
Deployment

↓

ReplicaSet

↓

Pods

↓

Scheduler

↓

Node
```

---

## Node Issue

```
Node

↓

kubelet

↓

containerd

↓

Disk

↓

Memory

↓

Network
```

---

# 10. Essential kubectl Commands

Pods

```bash
kubectl get pods

kubectl describe pod

kubectl logs pod

kubectl logs pod --previous
```

Deployments

```bash
kubectl get deploy

kubectl rollout status

kubectl rollout history
```

ReplicaSets

```bash
kubectl get rs
```

Nodes

```bash
kubectl get nodes

kubectl describe node

kubectl top node
```

Services

```bash
kubectl get svc

kubectl get endpoints
```

Storage

```bash
kubectl get pv

kubectl get pvc
```

Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

Shell Access

```bash
kubectl exec -it pod -- sh
```

---

# 11. Top Kubernetes Interview Questions

---

## Q1

Explain Kubernetes Architecture.

---

## Q2

Explain complete Pod creation flow.

---

## Q3

What happens after

```
kubectl apply
```

---

## Q4

Deployment vs ReplicaSet

---

## Q5

Pod vs Container

---

## Q6

Why does Kubernetes use Pause Container?

---

## Q7

How Scheduler selects Node?

Mention

- Filtering
- Scoring
- Binding

---

## Q8

Readiness vs Liveness

---

## Q9

Why is Pod Running but not serving traffic?

---

## Q10

Why Pod restarting every few minutes?

Possible answers

- Liveness
- OOMKilled
- Crash
- Node Restart
- Manual rollout

---

## Q11

How Service finds Pods?

Answer

```
Labels

↓

Selectors

↓

Endpoints
```

---

## Q12

How does kube-proxy work?

Explain

- Service IP
- iptables/IPVS
- Endpoint selection

---

## Q13

Explain Rolling Update.

Mention

```
maxSurge

maxUnavailable
```

---

## Q14

How does Kubernetes Self-Heal?

Explain

- Deployment Controller
- ReplicaSet
- kubelet
- Node Controller

---

## Q15

How does HPA work?

Mention

```
Metrics Server

↓

HPA

↓

Deployment

↓

ReplicaSet

↓

Pods
```

---

## Q16

How is Configuration Managed?

Answer

```
ConfigMap

Secret
```

---

## Q17

Authentication vs Authorization

Authentication

```
Who are you?
```

Authorization

```
What can you do?
```

---

## Q18

How does traffic reach your Spring Boot application?

Mention

```
DNS

↓

Load Balancer

↓

Ingress

↓

Service

↓

kube-proxy

↓

Pod

↓

Application
```

---

## Q19

What happens if Worker Node dies?

Mention

- Node Controller
- NotReady
- ReplicaSet
- New Pods

---

## Q20

What happens if etcd fails?

Mention

- Source of Truth
- Quorum
- HA

---

# 12. Senior Interview Tips

When answering Kubernetes questions,

don't stop at

```
Deployment creates Pods.
```

Explain

```
Deployment

↓

ReplicaSet

↓

Pod Object

↓

Scheduler

↓

Node

↓

kubelet

↓

containerd

↓

Container

↓

Readiness

↓

Service

↓

Ingress
```

This demonstrates a deep understanding of Kubernetes internals.

---

# Kubernetes Interview Cheat Sheet

| Topic | Remember |
|---------|----------|
| API Server | Entry point of cluster |
| etcd | Source of truth |
| Scheduler | Selects node only |
| kubelet | Creates and monitors Pods |
| containerd | Runs containers |
| ReplicaSet | Maintains Pod count |
| Deployment | Manages ReplicaSets |
| Service | Stable networking |
| Ingress | HTTP/HTTPS routing |
| Readiness | Controls traffic |
| Liveness | Restarts containers |
| Startup | Protects slow startups |
| HPA | Scales Pods |
| VPA | Scales resources |
| Cluster Autoscaler | Scales Nodes |
| ConfigMap | Configuration |
| Secret | Sensitive data |
| PV | Physical storage |
| PVC | Storage request |
| RBAC | Authorization |
| Service Account | Pod identity |

---

# Complete Kubernetes Flow (30-Second Revision)

```
Developer writes code
        │
Git Push
        │
CI/CD builds Docker image
        │
Push image to Registry
        │
kubectl/Helm/ArgoCD
        │
API Server
        │
Authentication
        │
Authorization
        │
Admission Controllers
        │
etcd stores desired state
        │
Deployment Controller
        │
ReplicaSet
        │
Pod Object
        │
Scheduler selects Node
        │
kubelet watches Pod
        │
containerd creates Pod
        │
Pause Container
        │
Application Container
        │
Startup Probe
        │
Readiness Probe
        │
Service Endpoints
        │
Ingress
        │
Load Balancer
        │
Client Request
        │
Response
```

---

# Final Key Takeaways

## Kubernetes Philosophy

- Kubernetes is **declarative**, not imperative.
- You specify the **desired state**; controllers continuously reconcile the actual state.
- Everything revolves around **Pods**, **Controllers**, and the **API Server**.

## Production Debugging Order

Whenever something goes wrong, investigate in this order:

```
Application
        ↓
Container
        ↓
Pod
        ↓
Deployment
        ↓
Service
        ↓
Ingress
        ↓
Node
        ↓
Cluster
```

## Golden Rules for Interviews

- Never say **Deployment creates Pods directly**.
- Remember: **Deployment → ReplicaSet → Pod**.
- **Scheduler assigns Nodes**; it does **not** create containers.
- **kubelet** starts containers via **containerd**.
- **Readiness** controls traffic; **Liveness** controls restarts.
- **Services** route traffic using **labels and selectors**.
- Every answer should explain **what happens internally**, not just define the resource.

---

# Congratulations 🎉

You now have a complete Kubernetes handbook covering:

- ✅ Kubernetes Fundamentals & Architecture
- ✅ End-to-End Deployment Pipeline
- ✅ Pod Lifecycle & Container Internals
- ✅ Health Checks & Networking
- ✅ Scaling, Rolling Updates & Self-Healing
- ✅ Storage, Security & Configuration
- ✅ Production Kubernetes
- ✅ Troubleshooting & Interview Guide

This sequence is designed to take you from understanding **how Kubernetes works internally** to being able to **debug real production issues** and answer **senior SDE-2/SDE-3 interview questions** with confidence.
