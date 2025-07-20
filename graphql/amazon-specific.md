

## 🏢 Amazon-Specific GraphQL Interview Questions (SDE-2)

### 1. How would you design a fault-tolerant GraphQL gateway that talks to multiple downstream services?

* Use **Apollo Gateway** or **BFF pattern**.
* Implement retries, timeouts, and circuit breakers (e.g., Resilience4j).
* Fallbacks for failed services.
* Use health checks and service discovery (e.g., AWS Cloud Map).
* Deploy in multiple AZs for HA.

### 2. Explain how you’d monitor and alert on errors in a live GraphQL service in production.

* Use **structured logging** with trace IDs.
* Integrate **Datadog**, **CloudWatch**, or **Prometheus**.
* Expose metrics like query latency, resolver errors.
* Setup alerts for spikes in error rates or degraded latencies.
* Include tracing headers for distributed tracing.

### 3. Have you worked on GraphQL federation or schema stitching in a microservice architecture?

* Yes, used **Apollo Federation** to separate ownership of types.
* Services define subgraphs with `@key`, `@external`, `@provides`.
* Gateway composes and routes requests.
* Enables modularity and independent deployability.

### 4. Describe a time you simplified a REST-heavy system by introducing GraphQL.

* Replaced multiple REST endpoints with a single GraphQL query.
* Reduced round-trips and payload size.
* Improved frontend developer productivity.
* Example: Consolidated user profile + orders + cart endpoints.

### 5. How would you reduce cold-start latency in a Lambda + GraphQL setup?

* Use **provisioned concurrency** for critical Lambdas.
* Bundle only needed dependencies.
* Lazy-load modules inside resolvers.
* Keep connection pools warm for DBs.
* Use AWS Lambda Powertools for metrics/logging without heavy SDKs.

---

Prepared for Backend SDE-2 Interviews — GraphQL Focused
