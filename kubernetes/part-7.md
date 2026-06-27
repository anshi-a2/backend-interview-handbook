# 07_Production_Kubernetes.md

> **Goal**
>
> Understand how Kubernetes is used in a real production environment. This chapter covers CI/CD, GitOps, observability, logging, monitoring, tracing, high availability, disaster recovery, and production best practices.

---

# Table of Contents

1. Production Architecture
2. CI/CD Pipeline
3. GitOps
4. Production Cluster Architecture
5. Observability
6. Logging
7. Monitoring
8. Distributed Tracing
9. High Availability
10. Disaster Recovery
11. Resource Optimization
12. Cost Optimization
13. Production Checklist
14. Common Production Issues
15. Interview Summary

---

# 1. Production Architecture

A typical production deployment looks like this:

```
                    Developers
                         │
                    Git Push
                         │
                    GitHub/GitLab
                         │
                GitHub Actions/Jenkins
                         │
               Build + Test + SonarQube
                         │
                  Docker Image Build
                         │
              Push Image → ECR/DockerHub
                         │
                 Update Helm Values
                         │
                      ArgoCD
                         │
                  Kubernetes Cluster
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   Ingress          Services         Monitoring
        │                │                │
       Pods         ConfigMaps       Prometheus
        │                │                │
     Databases       Secrets         Grafana
        │
    Fluent Bit
        │
 ElasticSearch/OpenSearch
        │
     Kibana
```

---

# 2. CI/CD Pipeline

Modern deployment pipeline

```
Developer

↓

Git Push

↓

CI Trigger

↓

Compile

↓

Unit Tests

↓

Static Analysis

↓

Docker Build

↓

Push Image

↓

Deploy
```

Typical tools

- GitHub Actions
- Jenkins
- GitLab CI
- Azure DevOps

---

## Build Stage

Example

```
mvn clean package
```

Artifacts

```
JAR

↓

Docker Image
```

---

## Test Stage

Pipeline should include

- Unit Tests
- Integration Tests
- API Tests
- Security Scan

Never deploy untested code.

---

## Docker Build

Produces immutable images

Example

```
payment-service:v1.3.5
```

Never use

```
latest
```

in production.

---

# 3. GitOps

Traditional deployment

```
Engineer

↓

kubectl apply
```

Problem

Manual changes.

No audit trail.

---

GitOps

```
Git Repository

↓

ArgoCD

↓

Kubernetes
```

Git becomes the source of truth.

Benefits

- Version history
- Rollback
- Automatic synchronization
- Auditability

Popular tools

- ArgoCD
- FluxCD

---

# 4. Production Cluster Architecture

Typical cluster

```
                Load Balancer
                      │
         ┌────────────┴────────────┐
         │                         │
     API Server               API Server
         │                         │
         └────────────┬────────────┘
                      │
                    etcd
          ┌───────────┼───────────┐
          │           │           │
     Worker 1    Worker 2    Worker 3
          │           │           │
       kubelet     kubelet     kubelet
          │           │           │
     containerd  containerd  containerd
          │           │           │
         Pods        Pods        Pods
```

Production recommendations

- Multiple API Servers
- 3 or 5 etcd nodes
- Multiple Worker Nodes
- Multiple Availability Zones

---

# 5. Observability

Observability answers

```
What happened?

Why did it happen?

Where did it happen?
```

Three pillars

```
Metrics

Logs

Tracing
```

---

# 6. Logging

Applications generate logs.

```
Application

↓

stdout

↓

Container

↓

Fluent Bit

↓

ElasticSearch

↓

Kibana
```

Why centralized logging?

Pods are ephemeral.

If a Pod dies

Logs disappear.

Centralized logging preserves history.

---

Logging Best Practices

✔ Structured JSON logs

✔ Correlation IDs

✔ Log levels

```
INFO

WARN

ERROR
```

Avoid

```
System.out.println()
```

---

# 7. Monitoring

Monitoring measures cluster health.

Architecture

```
Pods

↓

Metrics

↓

Prometheus

↓

Grafana
```

---

Metrics collected

- CPU
- Memory
- Disk
- Network
- Pod Restarts
- Response Time
- JVM Metrics

---

Example alerts

```
CPU > 80%

Memory > 90%

Pod Restart > 5

Disk > 85%

API Error Rate > 2%
```

---

Alerting

```
Prometheus

↓

AlertManager

↓

Slack

↓

PagerDuty

↓

Email
```

---

# 8. Distributed Tracing

Suppose

```
Frontend

↓

Order Service

↓

Payment Service

↓

Inventory

↓

Notification
```

One request touches many services.

Tracing follows the request.

Popular tools

- Jaeger
- Zipkin
- OpenTelemetry

---

Example

```
Request

↓

Trace ID

↓

Every Service

↓

Complete Timeline
```

Benefits

- Find slow service
- Root cause analysis
- Latency measurement

---

# 9. High Availability

Avoid single points of failure.

Recommended

```
3 API Servers

3 etcd Nodes

Multiple Worker Nodes

Multiple AZs
```

---

Node Failure

```
Worker 1

↓

Crash

↓

Pods recreated

↓

Worker 2
```

---

API Server Failure

Load Balancer routes to another API Server.

---

etcd Failure

Quorum keeps cluster operational.

---

# 10. Disaster Recovery

Things that should be backed up

- etcd
- Persistent Volumes
- Helm Charts
- Kubernetes Manifests
- Secrets

---

Recovery Plan

```
Backup

↓

Failure

↓

Restore etcd

↓

Restore PV

↓

Deploy Applications
```

---

Production recommendation

Regular automated backups.

---

# 11. Resource Optimization

Every Pod should define

```
Requests

Limits
```

---

Without limits

One Pod may consume

```
100% CPU

100% Memory
```

affecting other applications.

---

Use HPA

instead of

creating many idle Pods.

---

Image Optimization

Bad

```
Ubuntu

2 GB Image
```

Better

```
Alpine

or

Distroless
```

Smaller images

- Faster pull
- Faster deployment
- Lower storage

---

# 12. Cost Optimization

Common mistakes

- Too many replicas
- No autoscaling
- Overestimated CPU
- Overestimated Memory
- Large Docker images

---

Best Practices

✔ Enable HPA

✔ Enable Cluster Autoscaler

✔ Remove unused namespaces

✔ Delete unused images

✔ Use Spot instances (where appropriate)

✔ Right-size requests and limits

---

# 13. Production Checklist

## Security

- RBAC enabled
- Secrets encrypted
- Non-root containers
- Network Policies
- Image scanning

---

## Reliability

- Multiple replicas
- Liveness Probe
- Readiness Probe
- Startup Probe
- Pod Disruption Budget

---

## Monitoring

- Prometheus
- Grafana
- AlertManager

---

## Logging

- Fluent Bit
- ElasticSearch/OpenSearch
- Kibana

---

## Deployment

- Rolling Updates
- Rollbacks
- GitOps
- Versioned Images

---

## Storage

- Persistent Volumes
- StorageClass
- Automated backups

---

# 14. Common Production Issues

## High CPU

Check

```
kubectl top pod
```

Possible reasons

- Infinite loop
- High traffic
- Missing autoscaling

---

## Memory Leak

Symptoms

```
OOMKilled

Restart Loop
```

Investigate

- Heap dumps
- JVM metrics
- GC logs

---

## Frequent Restarts

Check

- Liveness Probe
- Application logs
- Memory limits

---

## Slow Deployment

Possible reasons

- Large Docker image
- Slow image pull
- Readiness failures

---

## No Logs

Check

```
Fluent Bit

Log volume

stdout
```

---

## High API Latency

Check

- CPU
- Database
- Redis
- Network
- Trace data

---

# Useful Commands

Cluster

```bash
kubectl cluster-info
```

Resource Usage

```bash
kubectl top node

kubectl top pod
```

Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

Logs

```bash
kubectl logs <pod-name>
```

Rollout

```bash
kubectl rollout status deployment payment
```

---

# Production Best Practices

### Deployment

✔ Always use Rolling Updates

✔ Never use `latest` image tag

✔ Keep deployments declarative

---

### Security

✔ Principle of Least Privilege

✔ Dedicated Service Accounts

✔ Rotate Secrets regularly

---

### Reliability

✔ At least 2-3 replicas

✔ Configure Pod Disruption Budgets

✔ Define resource requests & limits

---

### Observability

✔ Every request should have a Correlation ID

✔ Enable distributed tracing

✔ Alert before failures become outages

---

# Interview Summary

## Explain a Production Kubernetes Architecture

"A production Kubernetes environment begins with developers pushing code to Git. A CI/CD pipeline builds, tests, scans, and packages the application into a versioned Docker image stored in a container registry. GitOps tools like ArgoCD synchronize Kubernetes manifests from Git to the cluster. The Kubernetes control plane schedules workloads across multiple worker nodes, while Ingress exposes applications externally. Prometheus collects metrics, Grafana visualizes dashboards, AlertManager sends alerts, Fluent Bit forwards logs to Elasticsearch/OpenSearch, and Kibana enables log analysis. Distributed tracing tools like OpenTelemetry and Jaeger help diagnose latency across microservices. High availability is achieved using multiple API Servers, an etcd quorum, worker nodes across multiple availability zones, rolling updates, autoscaling, and automated backups."

---

# Key Takeaways

- Production Kubernetes extends beyond running Pods—it includes **CI/CD, GitOps, observability, security, and operational excellence**.
- Git should be the **single source of truth** for deployments.
- Use **versioned Docker images**, not `latest`.
- Collect **metrics (Prometheus)**, **logs (Fluent Bit + Elasticsearch/OpenSearch)**, and **traces (OpenTelemetry/Jaeger)** for complete observability.
- Run clusters with **multiple control plane components and worker nodes** to eliminate single points of failure.
- Use **HPA, Cluster Autoscaler, and right-sized resource requests** for efficient scaling and cost optimization.
- Regularly **back up etcd and persistent storage** to prepare for disaster recovery.
- Production success depends not only on deployment but also on **monitoring, security, reliability, and operational discipline**.
