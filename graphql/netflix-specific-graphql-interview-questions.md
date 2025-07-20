

## 🏢 Netflix-Specific GraphQL Interview Questions (SDE-2)

### 1. How would you secure a public-facing GraphQL endpoint serving millions of users?

* Use persisted queries to avoid arbitrary query execution.
* Apply rate limiting per IP/token.
* Validate query depth and complexity.
* Enable authentication for write operations.

### 2. Explain how persisted queries and client safelisting help reduce attack surface.

* Only allow pre-registered queries.
* Prevent malicious introspective or recursive queries.
* Reduces bandwidth and improves caching.

### 3. Describe a caching strategy for GraphQL data across CDNs, edge nodes, and client devices.

* Cache full queries or partial responses.
* Use cache-control headers and surrogate keys.
* Use Apollo Client’s normalized cache for client-side.
* Use edge computing (e.g., Cloudflare Workers) to cache at edge.

### 4. How do you log, trace, and monitor deeply nested GraphQL queries in real-time?

* Use Apollo Engine or OpenTelemetry.
* Log resolver timings, errors.
* Use request IDs and parent-child spans for nested traces.

### 5. GraphQL vs REST for streaming metadata APIs — what are the trade-offs?

* GraphQL: Better aggregation and nesting, but complex caching.
* REST: Simpler CDN support.
* Hybrid approach: Use GraphQL for query UI, REST for bulk/stream endpoints.


---

Prepared for Backend SDE-2 Interviews — GraphQL Focused
