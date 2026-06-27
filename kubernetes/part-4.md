# 04_Health_Checks_Request_Flow_Networking.md

> **Goal**
>
> Understand how Kubernetes determines whether an application is healthy, why Pods restart, how requests travel from the Internet to your application, and how Kubernetes networking works internally.

---

# Table of Contents

1. Why Health Checks?
2. Startup Probe
3. Readiness Probe
4. Liveness Probe
5. Probe Comparison
6. Why Pods Restart
7. Request Flow (End-to-End)
8. Kubernetes Networking
9. Services
10. kube-proxy
11. DNS (CoreDNS)
12. Ingress
13. Complete Network Flow
14. Common Production Issues
15. Debugging Commands
16. Interview Summary

---

# 1. Why Health Checks?

Starting a container **does not mean the application is ready**.

Example:

```
Spring Boot Container Started

↓

Loading Configuration

↓

Creating Beans

↓

Connecting Database

↓

Connecting Redis

↓

Loading Cache

↓

Application Ready
```

Although the container is running, the application cannot serve requests yet.

Kubernetes needs a way to determine:

- Has the application started?
- Can it receive traffic?
- Is it still healthy?

This is the purpose of **Probes**.

---

# 2. Startup Probe

## Purpose

Determines whether the application has **started successfully**.

Useful for applications that take a long time to initialize.

Example

```
Spring Boot

Hibernate

Flyway

Redis

Kafka

↓

Startup Time = 2 minutes
```

Without Startup Probe

```
Container starts

↓

Liveness runs immediately

↓

Health endpoint unavailable

↓

Container killed

↓

Restart

↓

Infinite loop
```

Startup Probe prevents this.

---

## Flow

```
Container Starts

↓

Startup Probe

↓

Success?

↓

YES

↓

Enable Readiness

↓

Enable Liveness
```

Until Startup Probe succeeds

- Readiness disabled
- Liveness disabled

---

Example

```yaml
startupProbe:
  httpGet:
    path: /actuator/health
    port: 8080
```

---

# 3. Readiness Probe

## Purpose

Determines

```
Can this Pod receive traffic?
```

Readiness affects **only routing**, not container lifecycle.

---

Flow

```
Container Running

↓

Readiness Probe

↓

Success

↓

Add Pod to Service

↓

Traffic Starts
```

---

If probe fails

```
Pod Running

↓

Removed from Service

↓

No Traffic
```

Container is NOT restarted.

---

Common checks

- Database connectivity
- Redis connectivity
- Kafka connectivity
- Internal initialization complete

---

Example

```yaml
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
```

---

# 4. Liveness Probe

## Purpose

Determines

```
Is the application still healthy?
```

Unlike Readiness,

failure results in **container restart**.

---

Flow

```
Running Application

↓

Liveness Probe

↓

Failure

↓

kubelet

↓

Kill Container

↓

Restart Container
```

---

Example

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
```

---

Common failures

- Deadlock
- Infinite loop
- Hung thread pool
- Application frozen
- JVM no longer responding

---

# 5. Probe Comparison

| Probe | Purpose | Restart Container | Receive Traffic |
|---------|----------|------------------|-----------------|
| Startup | Has app started? | No | No |
| Readiness | Can serve requests? | No | Yes/No |
| Liveness | Is app healthy? | Yes | No |

---

# 6. Why Pods Restart?

One of the most common interview questions.

## Case 1 - Application Crash

Example

```
NullPointerException

↓

JVM exits

↓

Exit Code = 1

↓

kubelet restarts container
```

---

## Case 2 - Liveness Failure

```
Application hangs

↓

Health endpoint stops responding

↓

Liveness fails

↓

Restart
```

---

## Case 3 - OOMKilled

Container exceeds memory limit.

Example

```
Memory Limit

1 GB

↓

Application uses

1.4 GB

↓

Linux Kernel

↓

Kill Process

↓

Exit Code 137

↓

Restart
```

---

## Case 4 - Node Restart

Worker node rebooted.

Pods recreated.

---

## Case 5 - Deployment Update

Rolling Update

```
Old Pod

↓

Terminate

↓

New Pod
```

Expected restart.

---

## Case 6 - Failed Startup

Application never starts.

Startup Probe keeps failing.

Eventually

Container restarted.

---

## Case 7 - Manual Restart

```
kubectl rollout restart deployment
```

---

# Why Pod Restarts Every 5 Minutes

Usually caused by

- Liveness Probe
- Memory Leak
- Deadlock
- External Dependency Timeout
- Application Crash
- Scheduled Job crashing application
- JVM Full GC freeze
- Database timeout causing health failure

Always inspect

```
kubectl logs

kubectl describe pod

kubectl get events
```

---

# 7. End-to-End Request Flow

Suppose a user opens

```
https://shop.company.com
```

Complete flow

```
Browser

↓

DNS

↓

Load Balancer

↓

Ingress Controller

↓

Ingress Rule

↓

Service

↓

kube-proxy

↓

Pod IP

↓

Spring Boot

↓

Database

↓

Response
```

---

# 8. Kubernetes Networking

Fundamental rule

**Every Pod gets its own IP address.**

Example

```
Pod A

10.10.1.2

Pod B

10.10.4.7
```

Pods communicate directly.

No NAT required inside the cluster.

---

Communication

```
Pod A

↓

Pod B

↓

Pod C
```

---

# 9. Services

Pods are temporary.

Every restart changes Pod IP.

Clients cannot depend on Pod IP.

Solution

```
Service
```

Example

```
Service

↓

Pod1

Pod2

Pod3
```

Service provides

- Stable Virtual IP
- Stable DNS
- Load balancing

---

## Service Types

### ClusterIP

Default.

Accessible only inside cluster.

---

### NodePort

Exposes application on every node.

```
NodeIP:30080
```

---

### LoadBalancer

Cloud provider creates external Load Balancer.

---

### ExternalName

Maps Service to external DNS.

---

# 10. kube-proxy

Service itself doesn't route packets.

kube-proxy programs Linux networking.

Modes

```
iptables

or

IPVS
```

Flow

```
Service IP

↓

iptables Rule

↓

Random Healthy Pod
```

---

# 11. DNS (CoreDNS)

Every Service automatically gets DNS.

Example

```
payment.default.svc.cluster.local
```

Application can simply call

```
http://payment
```

instead of Pod IP.

---

# 12. Ingress

Ingress manages HTTP routing.

Example

```
shop.company.com

↓

Ingress

↓

Service

↓

Pods
```

Supports

- SSL
- Path Routing
- Host Routing
- Rewrite
- Authentication

Popular controllers

- NGINX
- Traefik
- HAProxy
- AWS ALB

---

# 13. Complete Request Flow

```
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

Service

↓

kube-proxy

↓

Pod

↓

Container

↓

Spring Boot

↓

Redis

↓

Database

↓

Business Logic

↓

HTTP Response

↓

Browser
```

---

# 14. Common Production Issues

## Pod Running but No Traffic

Reason

Readiness Probe failing.

---

## Pod Restarting Every Few Minutes

Possible reasons

- Liveness Probe
- OOMKilled
- CrashLoopBackOff

---

## Service Cannot Reach Pod

Possible reasons

- Wrong Labels
- Selector mismatch
- Readiness failure

---

## DNS Not Working

Check

```
CoreDNS

Network Policy

Service Name
```

---

## 503 from Ingress

Usually

- No healthy endpoints
- Readiness failure
- Service misconfiguration

---

# 15. Debugging Commands

Check Pods

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

Previous crash

```bash
kubectl logs <pod-name> --previous
```

Services

```bash
kubectl get svc
```

Endpoints

```bash
kubectl get endpoints
```

Ingress

```bash
kubectl get ingress
```

DNS

```bash
kubectl exec -it pod -- nslookup payment
```

Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# Troubleshooting Checklist

| Problem | First Thing to Check |
|----------|----------------------|
| Pod restarting | Logs + Events |
| Running but inaccessible | Readiness Probe |
| Continuous restart | Liveness Probe |
| Exit Code 137 | OOMKilled |
| No Service endpoints | Labels + Readiness |
| 503 from Ingress | Endpoints |
| DNS failure | CoreDNS |
| Slow startup | Startup Probe |
| Connection refused | Service Port Mapping |

---

# Interview Summary

### Difference between Startup, Readiness and Liveness Probes

**Startup Probe** determines whether the application has finished starting. While it is failing, Kubernetes does not run Readiness or Liveness probes.

**Readiness Probe** determines whether the application is ready to receive requests. If it fails, the Pod remains running but is removed from the Service endpoints, so users stop sending traffic to it.

**Liveness Probe** determines whether the application is still healthy. If it fails, kubelet kills and restarts the container.

---

### Explain Request Flow in Kubernetes

"When a user sends a request, DNS resolves the application's domain to the external Load Balancer. The Load Balancer forwards traffic to the Ingress Controller, which matches the request against Ingress rules and forwards it to the appropriate Service. The Service uses kube-proxy (iptables/IPVS) to route the request to one of the healthy Pods selected by its labels. Only Pods with successful Readiness Probes are included in the Service endpoints. The application processes the request, communicates with downstream services like Redis or databases if necessary, and sends the response back through the same path."

---

# Key Takeaways

- A **Running** Pod is not necessarily **Ready**.
- **Startup Probe** protects slow-starting applications.
- **Readiness Probe** controls traffic.
- **Liveness Probe** controls container restarts.
- Every Pod has its own IP.
- Services provide stable networking despite changing Pod IPs.
- kube-proxy implements Service load balancing using iptables or IPVS.
- CoreDNS enables service discovery through DNS names.
- Ingress exposes HTTP/HTTPS applications to external users.
- Most production networking issues are caused by readiness failures, label mismatches, or incorrect Service configuration—not by Kubernetes networking itself.
