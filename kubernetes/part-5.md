# 05_Scaling_Rolling_Update_Self_Healing.md

> **Goal**
>
> Understand how Kubernetes automatically scales applications, performs zero-downtime deployments, recovers from failures, and maintains the desired state.

---

# Table of Contents

1. Why Scaling & Self-Healing?
2. Deployment vs ReplicaSet
3. Manual Scaling
4. Rolling Updates
5. Rollback
6. Deployment Strategies
7. Self-Healing
8. Horizontal Pod Autoscaler (HPA)
9. Vertical Pod Autoscaler (VPA)
10. Cluster Autoscaler
11. Node Failure Handling
12. Pod Eviction
13. Cordon & Drain
14. Graceful Shutdown
15. Common Production Issues
16. Debugging Commands
17. Interview Summary

---

# 1. Why Scaling & Self-Healing?

Imagine your Spring Boot application.

Initially

```
100 Users

↓

2 Pods
```

Suddenly

```
100,000 Users
```

Now

- CPU reaches 95%
- Memory reaches 90%
- Response time increases
- Requests start timing out

Kubernetes should automatically

- Add more Pods
- Remove unhealthy Pods
- Recover failed Nodes
- Deploy new versions without downtime

---

# 2. Deployment vs ReplicaSet

Many people confuse these.

```
Deployment

↓

ReplicaSet

↓

Pods
```

## Deployment

Responsible for

- Rolling Updates
- Rollbacks
- Scaling
- Version History

Deployment NEVER creates Pods directly.

---

## ReplicaSet

Responsible only for

```
Desired Pods = Actual Pods
```

Example

Desired

```
5 Pods
```

Current

```
4 Pods
```

ReplicaSet creates

```
1 New Pod
```

---

# 3. Manual Scaling

Example

```bash
kubectl scale deployment payment --replicas=5
```

Flow

```
Deployment

↓

ReplicaSet Updated

↓

Creates New Pods

↓

Scheduler

↓

Worker Node

↓

Pods Running
```

Scaling down

```
5 Pods

↓

3 Pods
```

ReplicaSet terminates two Pods.

---

# 4. Rolling Updates

Suppose

Current Version

```
v1

3 Pods
```

Deploy

```
v2
```

Kubernetes does NOT terminate all Pods together.

Instead

```
Old Pod

↓

New Pod

↓

Ready

↓

Delete Old Pod

↓

Repeat
```

No downtime.

---

## maxSurge

Extra Pods allowed during deployment.

Example

```
Replicas = 4

maxSurge = 1
```

Maximum Pods

```
5
```

---

## maxUnavailable

Pods allowed to be unavailable.

Example

```
Replicas = 4

maxUnavailable = 1
```

Minimum healthy Pods

```
3
```

---

Rolling Update Example

```
Old

Pod1
Pod2
Pod3

↓

Create Pod4(v2)

↓

Ready

↓

Delete Pod1(v1)

↓

Repeat
```

---

# 5. Rollback

Suppose

Deployment

```
v2
```

contains a bug.

Rollback

```bash
kubectl rollout undo deployment payment
```

Flow

```
Deployment History

↓

Previous ReplicaSet

↓

Pods Created

↓

Old Version Restored
```

---

View history

```bash
kubectl rollout history deployment payment
```

---

# 6. Deployment Strategies

---

## Rolling Update (Default)

```
Old Pods

↓

Gradually Replace

↓

New Pods
```

Pros

- No downtime

Cons

- Old & new versions coexist

---

## Recreate

```
Delete All

↓

Create New
```

Pros

Simple

Cons

Downtime

---

## Blue-Green

```
Blue Environment

↓

Green Environment

↓

Switch Traffic
```

Pros

Instant rollback

Cons

Double infrastructure cost

---

## Canary

```
90% Traffic

↓

Version 1

10% Traffic

↓

Version 2
```

Observe metrics.

Increase traffic gradually.

---

# 7. Self-Healing

One of Kubernetes' biggest features.

---

## Pod Crash

```
Desired

5 Pods

↓

One Pod Crashes

↓

ReplicaSet Detects

↓

Creates New Pod
```

---

## Node Failure

Worker Node crashes.

```
Node Not Ready

↓

Pods Lost

↓

Scheduler Creates Pods

↓

Healthy Nodes
```

---

## Container Crash

```
Application Crash

↓

kubelet

↓

Restart Container
```

---

Self-healing happens automatically.

---

# 8. Horizontal Pod Autoscaler (HPA)

Scales **number of Pods**.

Example

```
CPU

85%
```

Target

```
50%
```

Flow

```
Metrics Server

↓

HPA

↓

Deployment

↓

ReplicaSet

↓

More Pods
```

Example

```
3 Pods

↓

8 Pods
```

---

HPA metrics

- CPU
- Memory
- Custom Metrics
- Prometheus Metrics

---

Example

```yaml
minReplicas: 2

maxReplicas: 10

targetCPUUtilizationPercentage: 70
```

---

# 9. Vertical Pod Autoscaler (VPA)

Instead of

More Pods

It increases

```
CPU

Memory
```

Example

Before

```
CPU

500m
```

After

```
1000m
```

Useful for

- Databases
- Stateful apps

---

# 10. Cluster Autoscaler

Suppose

Scheduler cannot find nodes.

```
Pending Pods
```

Reason

No resources.

Cluster Autoscaler

```
Adds New VM

↓

Worker Node Joins

↓

Pods Scheduled
```

Cloud providers

- AWS
- Azure
- GCP

---

# 11. Node Failure Handling

Suppose

Worker Node crashes.

Flow

```
Heartbeat Missing

↓

Node Controller

↓

Node NotReady

↓

Pods Marked Lost

↓

ReplicaSet Creates New Pods

↓

Healthy Nodes
```

---

# 12. Pod Eviction

Sometimes Kubernetes removes Pods.

Reasons

- Memory Pressure
- Disk Pressure
- Node Shutdown
- Maintenance

Eviction differs from deletion.

Kubernetes chooses which Pods to remove.

Lowest priority Pods go first.

---

# 13. Cordon & Drain

---

## Cordon

```
kubectl cordon worker-1
```

Node remains running.

No new Pods scheduled.

Existing Pods continue running.

---

## Drain

```
kubectl drain worker-1
```

Flow

```
Move Pods

↓

Other Nodes

↓

Empty Node
```

Used before

- Upgrades
- Maintenance

---

# 14. Graceful Shutdown

When Pod deleted

Flow

```
SIGTERM

↓

Application Cleanup

↓

Close Connections

↓

Complete Requests

↓

Exit

↓

SIGKILL (if timeout)
```

Default grace period

```
30 seconds
```

---

## preStop Hook

Runs before container termination.

Example

```
Stop accepting traffic

↓

Finish Requests

↓

Shutdown
```

---

# 15. Common Production Issues

---

## Rolling Update Stuck

Reasons

- Readiness Probe failing
- Image Pull Failure
- CrashLoopBackOff

---

## HPA Not Scaling

Check

```
Metrics Server

CPU Requests

HPA Events
```

---

## Pending Pods

Reasons

- No Nodes
- Taints
- Insufficient Memory

---

## Frequent Node Eviction

Reasons

- Memory Pressure
- Disk Full

---

## Continuous Rollback

Usually

New deployment unhealthy.

---

# 16. Debugging Commands

Deployments

```bash
kubectl get deployment
```

ReplicaSets

```bash
kubectl get rs
```

Rollout Status

```bash
kubectl rollout status deployment payment
```

Rollout History

```bash
kubectl rollout history deployment payment
```

HPA

```bash
kubectl get hpa
```

Nodes

```bash
kubectl get nodes
```

Node Details

```bash
kubectl describe node worker-1
```

Metrics

```bash
kubectl top pod

kubectl top node
```

Events

```bash
kubectl get events
```

---

# Production Debugging Flow

### Pod keeps restarting

Check

```
kubectl logs

↓

kubectl describe pod

↓

Events

↓

Probe Failures

↓

Memory Limits
```

---

### Deployment stuck

Check

```
Rollout Status

↓

ReplicaSet

↓

Readiness Probe

↓

Image Pull

↓

Events
```

---

### HPA not working

Check

```
Metrics Server

↓

CPU Requests

↓

HPA

↓

Pod Metrics
```

---

### Pending Pods

Check

```
Scheduler Events

↓

Node Resources

↓

Taints

↓

Affinity
```

---

# Interview Summary

### Explain Rolling Update

"When a new Deployment version is applied, Kubernetes creates a new ReplicaSet. Instead of replacing all Pods at once, it gradually creates new Pods while terminating old ones based on `maxSurge` and `maxUnavailable`. Each new Pod must pass the Readiness Probe before an old Pod is removed, ensuring zero downtime."

---

### Explain Self-Healing

"Kubernetes continuously compares the desired state with the actual state using controllers. If a Pod crashes, the ReplicaSet creates a replacement. If a container crashes, kubelet restarts it. If an entire worker node fails, the Node Controller marks it NotReady, and Pods are recreated on healthy nodes. This continuous reconciliation is known as self-healing."

---

### Explain HPA

"The Horizontal Pod Autoscaler periodically reads metrics from the Metrics Server. If CPU, memory, or custom metrics exceed the configured threshold, it updates the Deployment's replica count. The Deployment updates its ReplicaSet, which creates additional Pods. When load decreases, HPA scales the Deployment back down."

---

# Key Takeaways

- **Deployment** manages versions, scaling, and rollouts.
- **ReplicaSet** ensures the desired number of Pods are running.
- **Rolling Updates** provide zero-downtime deployments.
- **Rollbacks** restore previous ReplicaSets.
- **HPA** scales the number of Pods.
- **VPA** adjusts CPU and memory for Pods.
- **Cluster Autoscaler** adds or removes worker nodes.
- **Self-healing** automatically replaces failed Pods and recovers from node failures.
- **Cordon** stops scheduling new Pods on a node.
- **Drain** safely moves Pods off a node for maintenance.
- **Graceful shutdown** prevents dropped requests during Pod termination.
- Understanding Deployment → ReplicaSet → Pod relationships and Kubernetes' reconciliation loop is essential for senior backend interviews.
