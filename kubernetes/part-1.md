# Kubernetes Fundamentals & Architecture

> **Goal**
>
> Build a solid mental model of Kubernetes before diving into deployments, networking, scaling, and troubleshooting.

---

# Table of Contents

1. Why Kubernetes?
2. Evolution of Deployment
3. What is Kubernetes?
4. Kubernetes Core Concepts
5. Kubernetes Architecture
6. Control Plane Components
7. Worker Node Components
8. Kubernetes Objects
9. Desired State & Reconciliation
10. High Availability
11. Common Misconceptions
12. Production Best Practices
13. Interview Summary

---

# 1. Why Kubernetes?

Imagine you have a Spring Boot application.

Running locally is easy:

```bash
java -jar app.jar
```

or

```bash
docker run app
```

But production introduces new challenges:

- Thousands of users
- Multiple servers
- Hardware failures
- Application crashes
- Zero downtime deployments
- Auto scaling
- Service discovery
- Configuration management

Managing containers manually becomes impossible.

Example:

```
100 Microservices

×

20 replicas each

=

2000 containers
```

Questions arise:

- Which server should run each container?
- What happens if a server crashes?
- How do users find the correct container?
- How do we deploy a new version without downtime?

Kubernetes solves these problems.

---

# 2. Evolution of Deployments

## Physical Servers

```
Application
Operating System
Hardware
```

Problems:

- Wasted resources
- Difficult scaling
- Hardware dependency

---

## Virtual Machines

```
App

Guest OS

Hypervisor

Hardware
```

Advantages

- Isolation
- Better utilization

Problems

- Heavy
- Slow boot
- Multiple OS overhead

---

## Containers

```
Application

Libraries

Container Runtime

Host OS
```

Advantages

- Lightweight
- Fast startup
- Portable

Problems

- No orchestration
- Manual recovery
- Manual scaling
- Manual networking

---

## Kubernetes

Kubernetes orchestrates containers.

Instead of saying:

> Run this container.

You declare:

> I always want **5 healthy instances** of this application.

Kubernetes continuously works to maintain that state.

---

# 3. What is Kubernetes?

Kubernetes is a **container orchestration platform**.

Responsibilities:

- Deploy applications
- Restart failed containers
- Scale automatically
- Load balancing
- Rolling updates
- Rollbacks
- Service discovery
- Secret management
- Storage orchestration

Kubernetes is **declarative**.

Example:

```yaml
replicas: 5
```

You never write code to create five Pods.

You simply declare the desired state.

---

# 4. Kubernetes Core Concepts

## Cluster

A Kubernetes cluster consists of:

```
Control Plane

+

Worker Nodes
```

---

## Node

A machine capable of running Pods.

Can be:

- Physical
- Virtual Machine
- Cloud Instance

---

## Pod

Smallest deployable unit.

A Pod contains one or more containers sharing:

- Network
- Storage
- Lifecycle

Example

```
Pod

├── Spring Boot
└── FluentBit Sidecar
```

---

## Container

An isolated process created from an image.

Containers never run directly inside Kubernetes.

They always run inside Pods.

---

## Desired State

Example

```
Replicas = 3
```

Current state:

```
2 Pods
```

Kubernetes notices the difference and creates one more Pod.

---

## Control Loop

Every controller continuously performs:

```
Observe

↓

Compare

↓

Correct
```

This loop is called **Reconciliation**.

---

# 5. Kubernetes Architecture

```
                 kubectl
                    │
                    ▼
             API Server
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
    etcd      Scheduler     Controller Manager
                    │
             Worker Nodes
                    │
     ┌──────────────┴──────────────┐
     ▼                             ▼
  kubelet                     kube-proxy
     │
 containerd
     │
    Pods
```

---

# 6. Control Plane Components

## API Server

The entry point of Kubernetes.

Everything communicates through it.

Responsibilities:

- Authentication
- Authorization
- Validation
- REST APIs
- Persist objects
- Watch mechanism

Think of it as the front door of the cluster.

---

## etcd

Distributed key-value database.

Stores the entire cluster state.

Examples:

- Pods
- Deployments
- Services
- Secrets
- ConfigMaps
- Nodes

If etcd is lost, Kubernetes loses its cluster state.

---

## Scheduler

Responsible only for selecting a node.

It answers:

> Which worker node should run this Pod?

Factors considered:

- CPU
- Memory
- Node affinity
- Taints/Tolerations
- Resource requests
- Policies

The Scheduler **does not create Pods**.

---

## Controller Manager

Runs many controllers.

Examples:

- Deployment Controller
- ReplicaSet Controller
- Job Controller
- Node Controller

Each controller continuously watches for changes and reconciles actual state with desired state.

---

# 7. Worker Node Components

## kubelet

Runs on every worker node.

Responsibilities:

- Watches assigned Pods
- Pulls images
- Starts containers
- Restarts failed containers
- Reports node health

---

## containerd

Container runtime responsible for:

- Pulling images
- Creating containers
- Managing namespaces
- Managing cgroups

Kubernetes talks to containerd through the Container Runtime Interface (CRI).

---

## kube-proxy

Responsible for networking.

Programs iptables/IPVS rules.

Provides Service load balancing.

---

# 8. Kubernetes Objects

| Object | Purpose |
|---------|---------|
| Pod | Runs containers |
| ReplicaSet | Maintains Pod count |
| Deployment | Manages ReplicaSets and rolling updates |
| Service | Stable network endpoint |
| ConfigMap | Non-sensitive configuration |
| Secret | Sensitive configuration |
| Namespace | Logical isolation |
| Ingress | HTTP/HTTPS routing |
| Job | One-time task |
| CronJob | Scheduled task |
| StatefulSet | Stateful workloads |
| DaemonSet | One Pod per node |

---

# 9. Desired State & Reconciliation

This is Kubernetes' core principle.

Example:

Desired:

```
5 Pods
```

Actual:

```
4 Pods
```

Controller notices the difference.

Creates one more Pod.

If a Pod crashes later:

```
Desired = 5

Actual = 4
```

Controller immediately creates another Pod.

This continuous correction is called **Reconciliation**.

---

# 10. High Availability

Production clusters avoid single points of failure.

Typical setup:

```
3 API Servers

↓

3 etcd Nodes

↓

Multiple Controller Managers

↓

Multiple Worker Nodes
```

Leader election ensures only one active controller performs reconciliation while others remain on standby.

---

# 11. Common Misconceptions

### Deployment creates Pods.

❌ Incorrect

Deployment creates a ReplicaSet.

ReplicaSet creates Pods.

---

### Scheduler starts containers.

❌ Incorrect

Scheduler only selects the node.

kubelet starts containers.

---

### Services own Pods.

❌ Incorrect

Services only route traffic.

Deployments/ReplicaSets own Pods.

---

### Pods are permanent.

❌ Incorrect

Pods are ephemeral.

They can be recreated at any time.

---

# 12. Production Best Practices

- Always define Resource Requests and Limits.
- Use Readiness and Liveness Probes.
- Store secrets in Secrets, not ConfigMaps.
- Organize applications using Namespaces.
- Use Labels consistently.
- Enable RBAC.
- Monitor with Prometheus and Grafana.
- Aggregate logs using Fluent Bit/ELK.

---

# 13. Interview Summary

### Explain Kubernetes in 2 minutes

"Kubernetes is a container orchestration platform that manages containerized applications using a declarative model. Developers specify the desired state, such as the number of replicas, and Kubernetes continuously reconciles the actual state to match it.

The Control Plane consists of the API Server, etcd, Scheduler, and Controller Manager. The API Server is the central entry point, etcd stores cluster state, the Scheduler assigns Pods to worker nodes, and Controllers ensure the desired state is maintained.

Each Worker Node runs kubelet, kube-proxy, and a container runtime like containerd. kubelet creates and monitors Pods, while kube-proxy provides networking and load balancing.

This architecture enables automatic deployment, scaling, self-healing, service discovery, and rolling updates, making Kubernetes the standard platform for running containerized applications in production."

---

# Key Takeaways

- Kubernetes manages **desired state**, not individual containers.
- Pods are the smallest deployable unit.
- API Server is the central communication hub.
- etcd is the source of truth.
- Scheduler selects nodes.
- kubelet runs Pods.
- Controllers continuously reconcile state.
- Services provide stable networking.
- Kubernetes is built around automation and self-healing.
