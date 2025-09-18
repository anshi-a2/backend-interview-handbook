# Amazon SDE-2 REST API Interview Questions & Solutions

## Question 1: Design a distributed cache service

-   Use consistent hashing to distribute keys.
-   Replication for availability, LRU/LFU eviction.
-   TTL for expiry, write-through or lazy invalidation.
-   Monitoring: hit ratio, latency, eviction count.

**Trade-offs:** strong vs eventual consistency, memory vs latency.

------------------------------------------------------------------------

## Question 2: High-throughput REST API for product details

-   Cache product details in Redis or CDN.
-   Use read replicas for DB.
-   Non-blocking I/O in Java (Spring WebFlux).
-   Circuit breakers, bulkheads.

------------------------------------------------------------------------

## Question 3: Pagination & filtering for large catalog APIs

-   Use query params: `/products?page=1&size=20&category=shoes`.
-   Cursor-based pagination for efficiency.
-   DB indexing + caching.

------------------------------------------------------------------------

## Question 4: Handling retries & idempotency in payment APIs

-   Use idempotency keys (e.g., requestId).
-   Ensure PUT/DELETE remain idempotent.
-   Handle retries with exponential backoff.

------------------------------------------------------------------------

## Question 5: Monitoring/observability at scale

-   Structured logs, metrics (Prometheus), traces (OpenTelemetry).
-   Dashboards (Grafana).
-   SLO/SLA/SLA reporting.
