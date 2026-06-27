# Pod Lifecycle & Container Internals

> **Goal**
>
> Understand what a Pod actually is, how Kubernetes creates it, how containers run inside it, why Pods restart, and how to debug common Pod lifecycle issues.

---

# Table of Contents

1. What is a Pod?
2. Why Kubernetes Uses Pods
3. Pod Architecture
4. Pod Lifecycle
5. Pod Creation Flow
6. Pause Container
7. Init Containers
8. Sidecar Containers
9. Container Runtime (containerd)
10. Linux Namespaces & cgroups
11. Container Restart Policy
12. Resource Requests & Limits
13. QoS Classes
14. Common Pod States
15. Common Pod Failures
16. Pod Deletion
17. Debugging Pods
18. Interview Summary

---

# 1. What is a Pod?

A **Pod** is the **smallest deployable unit** in Kubernetes.

A Pod wraps one or more containers that must:

- Share the same network namespace
- Share storage (volumes)
- Start and stop together
- Be scheduled together

Example

```
Pod

├── Spring Boot App
└── Fluent Bit (Log Collector)
```

Although a Pod can contain multiple containers, **one application container per Pod** is the recommended pattern.

---

# 2. Why Kubernetes Uses Pods Instead of Containers

A container is just a process.

Many supporting processes need to stay close to the application.

Example

```
Application

↓

Log Collector

↓

Monitoring Agent

↓

Security Agent
```

Instead of scheduling each container independently, Kubernetes groups them into one Pod.

Benefits

- Shared localhost
- Shared storage
- Same lifecycle
- Single scheduling unit

---

# 3. Pod Architecture

```
                    Pod
+------------------------------------------------+
|                                                |
|      Shared Network Namespace                  |
|                                                |
|  localhost                                     |
|                                                |
|  +--------------------+                        |
|  | Spring Boot        |                        |
|  +--------------------+                        |
|                                                |
|  +--------------------+                        |
|  | Fluent Bit         |                        |
|  +--------------------+                        |
|                                                |
|     Shared Volumes                            |
+------------------------------------------------+
```

Inside a Pod:

- One IP
- One hostname
- One network namespace
- Shared volumes

---

# 4. Pod Lifecycle

```
Pending

↓

ContainerCreating

↓

Running

↓

Ready

↓

Terminating

↓

Succeeded / Failed
```

---

## Pending

Pod object exists.

But container has not started.

Possible reasons

- Waiting for scheduler
- Image pull
- Volume attachment

---

## ContainerCreating

Node selected.

kubelet preparing

- Pull image
- Mount volumes
- Configure networking
- Start container

---

## Running

Container process has started.

**Important**

Running ≠ Ready

The application may still be initializing.

---

## Ready

Readiness Probe succeeds.

Service can now send traffic.

---

## Terminating

Pod deletion requested.

Kubernetes

- Sends SIGTERM
- Waits for grace period
- Sends SIGKILL if necessary

---

## Succeeded

Container exits successfully.

Mostly for Jobs.

---

## Failed

Container terminated unexpectedly.

---

# 5. Pod Creation Flow

```
Deployment

↓

ReplicaSet

↓

Pod Object

↓

Scheduler

↓

Worker Node

↓

kubelet

↓

containerd

↓

Pause Container

↓

Application Container

↓

Running
```

Detailed flow

1. Deployment creates ReplicaSet.
2. ReplicaSet creates Pod object.
3. Scheduler assigns a node.
4. kubelet detects assigned Pod.
5. kubelet calls containerd.
6. containerd creates Pod Sandbox.
7. Image is pulled.
8. Containers start.
9. Health probes begin.

---

# 6. Pause Container

Every Pod has a hidden **Pause Container**.

```
Pod

├── Pause Container
├── Spring Boot
└── Fluent Bit
```

Purpose

- Owns network namespace
- Owns Pod IP
- Holds shared namespaces alive

Without it, every container restart would destroy the Pod network.

Think of it as the Pod's "anchor."

---

# 7. Init Containers

Init Containers run **before** application containers.

Example

```
Init Container

↓

Download Config

↓

Exit

↓

Application Starts
```

Common uses

- Database migration
- Waiting for dependency
- Downloading files
- Creating directories

Characteristics

- Run sequentially
- Must complete successfully
- Never run again

---

# 8. Sidecar Containers

Runs alongside the main application.

Example

```
Pod

├── Application
├── Log Collector
├── Metrics Exporter
└── Security Agent
```

Common uses

- Logging
- Monitoring
- Reverse proxy
- Certificate renewal

---

# 9. Container Runtime

Kubernetes doesn't start containers directly.

It communicates through the **Container Runtime Interface (CRI)**.

Typical runtime

```
kubelet

↓

CRI

↓

containerd

↓

runc

↓

Linux Kernel
```

containerd responsibilities

- Pull image
- Create container
- Start container
- Stop container
- Delete container

---

# 10. Linux Namespaces & cgroups

Containers are ordinary Linux processes.

Isolation comes from Linux features.

---

## Namespaces

Provide isolation.

Examples

| Namespace | Isolates |
|-----------|----------|
| PID | Processes |
| NET | Networking |
| IPC | Shared memory |
| MNT | File system |
| UTS | Hostname |
| USER | Users |

Each Pod gets its own network namespace.

---

## cgroups

Limit resources.

Examples

- CPU
- Memory
- Disk IO

Without cgroups, one container could consume the entire machine.

---

# 11. Container Restart Policy

Possible values

```
Always

OnFailure

Never
```

Deployment Pods usually use

```
Always
```

Jobs generally use

```
OnFailure
```

---

# 12. Resource Requests & Limits

## Requests

Minimum resources guaranteed.

Example

```
CPU: 500m

Memory: 512Mi
```

Scheduler uses Requests while selecting nodes.

---

## Limits

Maximum resources allowed.

Example

```
Memory: 1Gi
```

If exceeded

Container may be terminated.

---

# 13. QoS Classes

Kubernetes assigns a Quality of Service class.

---

## Guaranteed

Requests == Limits

Highest priority.

Least likely to be evicted.

---

## Burstable

Requests < Limits

Most common.

---

## BestEffort

No requests.

No limits.

Lowest priority.

First to be evicted during resource pressure.

---

# 14. Common Pod States

## CrashLoopBackOff

Meaning

Application starts

↓

Crashes

↓

Restart

↓

Crash again

Repeated failures increase restart delay.

Common causes

- Application exception
- Missing environment variable
- Invalid configuration

---

## ImagePullBackOff

Image cannot be downloaded.

Reasons

- Wrong image name
- Registry authentication
- Image doesn't exist

---

## ErrImagePull

Initial image pull failure.

May later become

```
ImagePullBackOff
```

---

## Pending

Scheduler cannot assign node.

Possible reasons

- CPU shortage
- Memory shortage
- Taints
- PVC unavailable

---

## OOMKilled

Container exceeded memory limit.

Linux kernel kills the process.

Symptoms

```
Exit Code 137
```

Solution

Increase memory limit or fix memory leak.

---

## ContainerCreating

Usually temporary.

If stuck

Check

- Image pull
- Volume mount
- Network plugin

---

# 15. Why Pods Restart

## Application Crash

```
NullPointerException

↓

Process exits

↓

kubelet restarts
```

---

## Liveness Probe Failure

Probe fails repeatedly.

kubelet kills container.

Creates new one.

---

## OOMKilled

Memory exceeded.

Kernel terminates process.

---

## Node Restart

Worker node reboots.

Pod recreated.

---

## Manual Rollout

New Deployment version.

Old Pods terminated.

New Pods created.

---

# 16. Pod Deletion

Command

```bash
kubectl delete pod app-123
```

Flow

```
Delete Request

↓

API Server

↓

Pod marked Terminating

↓

SIGTERM

↓

Grace Period

↓

SIGKILL

↓

Pod Removed
```

Default grace period

```
30 seconds
```

---

# 17. Debugging Pods

Useful commands

View Pods

```bash
kubectl get pods
```

Describe Pod

```bash
kubectl describe pod <pod-name>
```

Logs

```bash
kubectl logs <pod-name>
```

Previous crashed container

```bash
kubectl logs <pod-name> --previous
```

Open shell

```bash
kubectl exec -it <pod-name> -- sh
```

Events

```bash
kubectl get events
```

Node information

```bash
kubectl describe node
```

---

# Debugging Checklist

| Problem | Check |
|----------|-------|
| Pending | Scheduler events |
| ImagePullBackOff | Image name, registry |
| CrashLoopBackOff | Application logs |
| OOMKilled | Memory usage |
| ContainerCreating | Volumes, image, CNI |
| Readiness Failure | Probe endpoint |
| Liveness Failure | Health endpoint |
| Frequent Restart | Logs + describe output |

---

# Interview Summary

### Explain Pod Lifecycle

"A Deployment creates a ReplicaSet, which creates Pod objects. The Scheduler assigns each Pod to a worker node. The kubelet on that node watches for assigned Pods and communicates with the container runtime (containerd) through the CRI. The runtime creates a Pod Sandbox, starts the Pause container, configures networking and volumes, then launches Init containers (if any) followed by the application containers. Kubernetes executes Startup, Readiness, and Liveness probes. Once the Readiness Probe succeeds, the Pod is added to the Service endpoints and starts receiving traffic. Throughout its lifetime, kubelet monitors the Pod and restarts containers according to the restart policy if failures occur."

---

# Key Takeaways

- A Pod is the smallest deployable unit in Kubernetes.
- Pods share networking and storage.
- Kubernetes schedules Pods, not individual containers.
- Every Pod contains a hidden Pause Container.
- Init Containers always finish before application containers start.
- Sidecars provide supporting functionality like logging and monitoring.
- containerd is responsible for creating and running containers.
- Linux namespaces provide isolation; cgroups enforce resource limits.
- Requests influence scheduling; Limits restrict resource usage.
- `Running` does not mean `Ready`.
- `CrashLoopBackOff`, `ImagePullBackOff`, and `OOMKilled` are among the most common production issues.
- `kubectl describe`, `kubectl logs`, and Kubernetes Events are the first tools to use when debugging Pod failures.
