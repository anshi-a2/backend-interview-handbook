# Uber SDE-2 REST API Interview Questions & Solutions

## Question 1: Ride booking API (idempotency)

-   POST `/rides` with idempotency key.
-   Avoids double booking from retries.
-   Return same booking on duplicate request.

------------------------------------------------------------------------

## Question 2: Location update API for drivers

-   PUT `/drivers/{id}/location` with GPS coords.
-   High frequency updates → batch or compress data.
-   Use Kafka for async processing.

------------------------------------------------------------------------

## Question 3: Surge pricing API

-   GET `/pricing?location=...`.
-   Dynamic calculations based on demand/supply.
-   Cache results for seconds.

------------------------------------------------------------------------

## Question 4: Distributed transactions (ride + payment)

-   Use saga pattern for orchestration.
-   Rollback ride if payment fails.
-   Compensating transactions.

------------------------------------------------------------------------

## Question 5: Observability in real-time REST services

-   Metrics: request latency, success rate, error codes.
-   Distributed tracing with OpenTelemetry.
-   Real-time dashboards for SLA monitoring.
