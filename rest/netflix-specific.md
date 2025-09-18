# Netflix SDE-2 REST API Interview Questions & Solutions

## Question 1: Video metadata API

-   GET `/movies/{id}` returns metadata (not binary).
-   Cache frequently accessed metadata in Redis.

------------------------------------------------------------------------

## Question 2: Recommendations API

-   GET `/users/{id}/recommendations`.
-   Async fetch from ML service.
-   Cache results for short TTL.

------------------------------------------------------------------------

## Question 3: Resilience in REST APIs

-   Use circuit breakers (Resilience4j).
-   Retries with exponential backoff.
-   Bulkheads for isolation.

------------------------------------------------------------------------

## Question 4: Global API gateway design

-   Multi-region deployments.
-   DNS-based routing + load balancing.
-   API Gateway handles auth, logging, throttling.

------------------------------------------------------------------------

## Question 5: Authentication in distributed REST

-   JWT tokens with expiry.
-   OAuth2 for external apps.
-   Refresh tokens for session renewal.
