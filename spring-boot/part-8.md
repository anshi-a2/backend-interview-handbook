# 08_Spring_Boot_Production_Internals_Performance_Best_Practices.md

> **Goal**
>
> This is the final chapter of the Spring Boot guide. It focuses on **how Spring Boot applications behave in production**, covering **performance tuning, JVM internals, connection pooling, caching, multithreading, graceful shutdown, deployment, monitoring, debugging, and common production issues**.
>
> These topics are **frequently discussed in SDE-2/Senior Backend interviews** because they demonstrate experience operating real-world backend systems.

---

# Table of Contents

1. Production Request Journey
2. Threading Model
3. Database Connection Pooling
4. Caching
5. Asynchronous Processing
6. Scheduling
7. Graceful Shutdown
8. Memory Management & JVM
9. Garbage Collection
10. Monitoring & Observability
11. Logging
12. Health Checks
13. Docker Deployment
14. Kubernetes Deployment
15. Common Production Issues
16. Performance Tuning Checklist
17. Best Practices
18. Senior Interview Summary

---

# 1. Production Request Journey

A request in production typically follows this path:

```
Client

↓

DNS

↓

Load Balancer

↓

API Gateway / NGINX

↓

Kubernetes Service

↓

Pod

↓

Tomcat

↓

Spring Security

↓

DispatcherServlet

↓

Controller

↓

Service

↓

Redis Cache (Optional)

↓

Database

↓

Response
```

Every layer adds latency, so optimizing the complete path is more important than optimizing only application code.

---

# 2. Threading Model

Spring Boot (Servlet stack) uses a **thread-per-request** model.

```
Request A

↓

Thread-21

↓

Response

-----------------

Request B

↓

Thread-22

↓

Response
```

Tomcat maintains a **thread pool**.

```
Incoming Requests

↓

Tomcat Thread Pool

↓

Worker Threads
```

When a request finishes, its thread returns to the pool.

### Production Issues

- Long database queries
- Slow external API calls
- Blocking I/O
- Thread pool exhaustion

### Best Practices

✔ Keep request processing fast.

✔ Avoid blocking operations inside request threads.

✔ Configure appropriate thread pool sizes.

---

# 3. Database Connection Pooling

Opening a database connection for every request is expensive.

Instead

```
Application

↓

Connection Pool

↓

Database
```

Spring Boot uses **HikariCP** by default.

```
Request

↓

Borrow Connection

↓

Execute SQL

↓

Return Connection
```

### Benefits

- Low latency
- Better throughput
- Reduced database overhead

### Common Problems

- Connection leaks
- Pool exhaustion
- Incorrect pool sizing

---

# 4. Caching

Without cache

```
Request

↓

Database

↓

Response
```

With cache

```
Request

↓

Redis

↓

Found?

↓

Yes → Response

↓

No

↓

Database

↓

Redis

↓

Response
```

### Common Cache Strategies

- Cache Aside
- Read Through
- Write Through
- Write Behind

### What to Cache

- Product catalog
- User profile
- Configuration
- Frequently accessed reference data

Avoid caching frequently changing data unless invalidation is well designed.

---

# 5. Asynchronous Processing

Not every task should execute in the request thread.

Example

```
User Registration

↓

Save User

↓

Return Response

↓

Send Email (Async)
```

Spring provides

```
@Async
```

Typical use cases

- Emails
- Notifications
- Report generation
- Background processing

---

# 6. Scheduling

Execute recurring jobs automatically.

Spring provides

```
@Scheduled
```

Examples

- Cleanup jobs
- Report generation
- Cache refresh
- Daily reconciliation

Example

```java
@Scheduled(cron = "0 0 * * * *")
```

Runs every hour.

---

# 7. Graceful Shutdown

Suppose Kubernetes terminates a Pod.

Without graceful shutdown

```
Request

↓

Pod Deleted

↓

Request Lost
```

With graceful shutdown

```
SIGTERM

↓

Stop Accepting Requests

↓

Complete Active Requests

↓

Close Connections

↓

Application Stops
```

Benefits

- No dropped requests
- Cleaner deployments
- Better rolling updates

---

# 8. JVM Memory Model

```
Heap

├── Young Generation
├── Old Generation

Non-Heap

├── Metaspace
├── Code Cache
├── Thread Stacks
```

### Heap

Stores Java objects.

### Metaspace

Stores class metadata.

### Thread Stack

Stores method calls and local variables.

---

# 9. Garbage Collection

Objects no longer referenced become eligible for garbage collection.

```
Object Created

↓

Used

↓

Unreachable

↓

Garbage Collector Removes It
```

Modern JVMs commonly use collectors such as

- G1 GC (default in many JDK versions)
- ZGC
- Shenandoah

Selection depends on latency and throughput requirements.

---

# 10. Monitoring & Observability

A production application should expose

### Metrics

```
Micrometer

↓

Prometheus

↓

Grafana
```

Track

- CPU
- Memory
- JVM
- HTTP latency
- Error rates
- Database metrics

---

### Distributed Tracing

```
Gateway

↓

User Service

↓

Order Service

↓

Payment Service
```

Use

- OpenTelemetry
- Jaeger
- Zipkin

---

# 11. Logging

Good logs should answer

- What happened?
- When?
- Which request?
- Which user?
- Which service?

Log levels

```
ERROR

WARN

INFO

DEBUG

TRACE
```

Best Practices

- Use structured logging (JSON where appropriate)
- Include Correlation ID / Trace ID
- Avoid logging passwords or sensitive data

---

# 12. Health Checks

Spring Boot Actuator provides

```
/actuator/health
```

Typical checks

- Database
- Redis
- Kafka
- Disk space
- Custom dependencies

---

Kubernetes uses

### Liveness Probe

```
Is application alive?
```

If

```
No

↓

Restart Pod
```

---

### Readiness Probe

```
Can application receive traffic?
```

If

```
No

↓

Remove from Load Balancer

↓

Do NOT Restart
```

---

### Startup Probe

Useful for applications with long startup times.

```
Application Starting

↓

Startup Probe

↓

Passes

↓

Enable Liveness & Readiness
```

---

# 13. Docker Deployment

Typical flow

```
Source Code

↓

Build JAR

↓

Docker Image

↓

Container

↓

Kubernetes Pod
```

Benefits

- Consistent environments
- Easy deployment
- Immutable artifacts

---

# 14. Kubernetes Deployment

Typical deployment pipeline

```
Git Push

↓

CI Pipeline

↓

Run Tests

↓

Build JAR

↓

Build Docker Image

↓

Push Image Registry

↓

Update Deployment

↓

Rolling Update

↓

New Pods

↓

Readiness Check

↓

Traffic Shift

↓

Old Pods Removed
```

This enables zero or minimal downtime deployments.

---

# 15. Common Production Issues

## High CPU

Possible causes

- Infinite loops
- Excessive serialization
- Too many threads
- Heavy computations

---

## High Memory Usage

Possible causes

- Memory leaks
- Large caches
- Huge collections
- Long-lived objects

---

## OutOfMemoryError

Check

- Heap size
- Heap dump
- Object retention
- Cache growth

---

## Slow APIs

Investigate

```
Controller

↓

Service

↓

Database

↓

External APIs
```

Measure before optimizing.

---

## Database Bottleneck

Check

- Slow queries
- Missing indexes
- Connection pool
- Locks
- N+1 queries

---

## Frequent Pod Restarts

Common reasons

- Liveness probe failures
- OutOfMemoryError (OOMKilled)
- Application crash
- Misconfigured health checks
- CrashLoopBackOff after repeated failures

---

## Thread Pool Exhaustion

Symptoms

- Requests hanging
- Increased latency
- Timeouts

Investigate

- Long-running tasks
- External service latency
- Database contention

---

# 16. Performance Tuning Checklist

### JVM

✔ Appropriate heap sizing

✔ Modern garbage collector

✔ Monitor GC pauses

---

### Database

✔ Index frequently queried columns

✔ Optimize SQL

✔ Use pagination

✔ Tune connection pool

---

### Cache

✔ Cache hot data

✔ Use TTL

✔ Avoid stale data

---

### API

✔ Compress responses where appropriate

✔ Return only required fields

✔ Batch operations when practical

---

### Application

✔ Minimize object creation in hot paths

✔ Avoid unnecessary synchronization

✔ Prefer constructor injection

✔ Keep Beans stateless

---

# 17. Production Best Practices

✔ Externalize configuration.

✔ Use environment-specific profiles.

✔ Enable health checks and metrics.

✔ Configure graceful shutdown.

✔ Secure Actuator endpoints.

✔ Monitor JVM, application, and infrastructure metrics.

✔ Use centralized logging.

✔ Keep services stateless.

✔ Use retries with exponential backoff only where appropriate.

✔ Test disaster recovery and rollback procedures.

✔ Automate deployments using CI/CD pipelines.

---

# 18. Senior Interview Summary

## Explain the Production Request Flow

"A production request typically passes through DNS, a load balancer, an API gateway or reverse proxy, Kubernetes services, and finally a Spring Boot Pod. Inside the application, it flows through the Security Filter Chain, DispatcherServlet, Controller, Service, Repository, and Database before the response is returned. Each layer contributes to latency, security, and resilience."

---

## Why Use Connection Pooling?

"Creating database connections is expensive. A connection pool such as HikariCP maintains reusable connections so requests can borrow and return them efficiently, significantly improving throughput and reducing latency."

---

## Readiness vs Liveness Probe

- **Liveness Probe** determines whether the application is alive. If it fails repeatedly, Kubernetes restarts the container.
- **Readiness Probe** determines whether the application is ready to receive traffic. If it fails, Kubernetes temporarily removes the Pod from service without restarting it.
- **Startup Probe** delays liveness checks until a slow-starting application has initialized successfully.

---

## Why Do Pods Restart Every 5 Minutes?

Common causes include

- Liveness probe failures
- OutOfMemoryError leading to OOMKilled
- Unhandled application exceptions
- Dependency failures causing startup crashes
- Misconfigured health endpoints
- Resource limits that are too low

A typical investigation involves checking

```
kubectl describe pod

↓

kubectl logs

↓

Events

↓

Health Probes

↓

Memory/CPU Metrics

↓

Application Logs
```

---

## How Would You Improve Spring Boot Performance?

- Optimize SQL queries and indexes.
- Use connection pooling.
- Introduce caching for frequently accessed data.
- Keep APIs stateless.
- Reduce blocking operations.
- Tune thread pools.
- Monitor JVM and GC.
- Use asynchronous processing for long-running tasks.
- Continuously measure with metrics before optimizing.

---

# Key Takeaways

- Production performance depends on the **entire request path**, not just application code.
- **HikariCP** provides efficient database connection pooling.
- **Redis** and well-designed caching strategies reduce database load.
- **Graceful shutdown** ensures zero or minimal downtime during deployments.
- Understanding **JVM memory**, **Garbage Collection**, and **thread pools** is essential for diagnosing production issues.
- **Spring Boot Actuator**, **Prometheus**, **Grafana**, and **OpenTelemetry** form the foundation of production observability.
- Kubernetes **Readiness**, **Liveness**, and **Startup Probes** are critical for reliable deployments.
- Senior engineers focus not only on writing code but also on **performance, scalability, resilience, monitoring, and operational excellence**.
