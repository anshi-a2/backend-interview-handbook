
## 🏢 Uber-Specific GraphQL Interview Questions (SDE-2)

### 1. Uber’s systems are very latency-sensitive — how would you ensure GraphQL doesn’t increase p99 latency?

* Use persisted queries.
* Avoid nested queries with high fan-out.
* Use DataLoader to batch DB calls.
* Monitor and cache aggressively.

### 2. Describe how you’d expose real-time vehicle tracking data with GraphQL.

* Use GraphQL Subscriptions with WebSocket.
* Push data updates to client as driver moves.
* Use throttle/debounce logic.

### 3. How would you model driver-partner and customer relationship in a GraphQL schema?

```graphql
type Ride {
  id: ID!
  driver: Driver!
  customer: Customer!
  status: String!
  location: Location
}
```

### 4. What tools would you use to monitor GraphQL resolver health in a critical system?

* Apollo Studio or OpenTelemetry.
* Track resolver timings, error rates.
* Export Prometheus metrics.

### 5. If a backend team owns a microservice and the GraphQL schema exposes it, how would you coordinate schema changes?

* Schema registry (Apollo/Git).
* Contract-first approach.
* Versioned schemas or directives.

---

Prepared for Backend SDE-2 Interviews — GraphQL Focused
