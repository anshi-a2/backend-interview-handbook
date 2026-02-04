# Zero-Downtime Deployment – Java Backend Interview Guide

## 1. What Is the Problem We Are Solving?

When deploying to production, the biggest risks are:

* Service downtime
* Failed deployments
* Partial system outages

> Interviewers want to know: **How do you deploy without taking the system down?**

This tests:

* Production maturity
* Distributed systems thinking
* Real-world backend experience

---

## 2. Core Principle: Separate Deployment from Availability

A well-designed system ensures:

* **Instances can come and go**
* Traffic is routed only to healthy instances
* Failures are isolated

> If one instance dies during deployment, users should not notice.

---

## 3. Key Building Blocks for Safe Deployment

### 3.1 Load Balancer

* Routes traffic to multiple instances
* Removes unhealthy instances automatically

**Examples:**

* NGINX
* AWS ALB / ELB
* Kubernetes Service

---

### 3.2 Health Checks

Each service exposes a health endpoint:

```http
GET /health
```

Deployment systems rely on this to:

* Add instance to traffic only when ready
* Remove instance before shutdown

---

### 3.3 Stateless Services (Very Important)

To support zero downtime:

* No session data in memory
* No local file storage

**Instead:**

* Sessions → Redis
* Files → S3

---

## 4. Deployment Strategies (Most Important Section)

### 4.1 Rolling Deployment

**How it works:**

* Deploy new version to one instance at a time
* Keep remaining instances serving traffic

**Flow:**

```text
Instance A → Deploy v2 → Healthy → Add to LB
Instance B → Deploy v2 → Healthy → Add to LB
```

**Pros:**

* Simple
* No extra infrastructure

**Cons:**

* Old and new versions run together

---

### 4.2 Blue-Green Deployment

**How it works:**

* Two environments: Blue (current), Green (new)
* Switch traffic instantly after validation

**Flow:**

```text
Users → Blue (v1)
Deploy v2 → Green
Switch LB → Green
```

**Pros:**

* Instant rollback
* No mixed versions

**Cons:**

* Doubles infrastructure

---

### 4.3 Canary Deployment

**How it works:**

* Release to small % of users first
* Monitor metrics
* Gradually increase traffic

**Flow:**

```text
5% → v2
25% → v2
100% → v2
```

**Pros:**

* Lowest risk
* Real user validation

**Cons:**

* More complex

---

### Strategy Comparison

| Strategy   | Downtime | Rollback | Cost   | Risk     |
| ---------- | -------- | -------- | ------ | -------- |
| Rolling    | None     | Medium   | Low    | Medium   |
| Blue-Green | None     | Easy     | High   | Low      |
| Canary     | None     | Easy     | Medium | Very Low |

---

## 5. Handling Database Changes Safely

### Problem

Application and database must evolve together.

---

### Best Practices

1. **Backward-compatible schema changes**

   * Add columns (nullable)
   * Avoid dropping columns

2. **Two-phase deployment**

   * Deploy DB changes first
   * Deploy app changes later

3. **Feature flags**

---

### Tools

* Flyway / Liquibase
* Versioned migrations

---

## 6. Feature Flags (Hidden Superpower)

Feature flags allow:

* Deploying code without enabling it
* Turning features on/off instantly

**Use cases:**

* Emergency rollback
* Gradual rollout

---

## 7. Graceful Shutdown (Critical for Java Services)

### Why It Matters

During deployment:

* In-flight requests must finish
* New requests should stop

---

### Java / Spring Boot

* Handle `SIGTERM`
* Configure graceful shutdown

```yaml
server:
  shutdown: graceful
```

---

## 8. Observability During Deployment

### What to Monitor

* Error rate
* Latency
* CPU / memory
* Business metrics

---

### Tools

* Prometheus + Grafana
* CloudWatch
* ELK

---

## 9. Real-World Banking Example

### Scenario: Deploying Payment Service

* Canary deployment to 5% traffic
* Strict error monitoring
* Immediate rollback on anomaly

**Why:**

* Financial correctness > speed

---

## 10. Failure & Rollback Strategy

A deployment plan is incomplete without rollback.

**Rollback options:**

* Switch LB (Blue-Green)
* Disable feature flag
* Redeploy previous version

---

## 11. Interview-Ready One-Paragraph Answer

> "To ensure zero downtime during deployment, systems use load balancers, health checks, and stateless services. Deployments are done using rolling, blue-green, or canary strategies. Database changes are backward-compatible and controlled via migrations. Feature flags and graceful shutdown ensure safe rollouts and fast rollbacks, so the system remains available even during failures."

---

## 12. One-Line Takeaways

* Never deploy to a single instance
* Health checks control traffic
* Canary is safest
* Backward-compatible DB changes are mandatory

---

