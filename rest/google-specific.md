# Google SDE-2 REST API Interview Questions & Solutions

## Question 1: IPC between microservices

-   REST vs gRPC vs Pub/Sub.
-   Serialization & schema evolution.
-   Retries, error handling.

------------------------------------------------------------------------

## Question 2: API design for user events

Endpoint: `GET /users/{id}/events?start=...&end=...&page=...` - Index on
(userId, eventTime). - Partition by userId/date. - Pagination with
metadata.

------------------------------------------------------------------------

## Question 3: Design Docs API (collaboration)

-   Endpoints: create doc, fetch doc, update doc.
-   Use optimistic concurrency (versioning).
-   Consider real-time updates with WebSocket, fallback REST.

------------------------------------------------------------------------

## Question 4: Rate limiting & quota enforcement API

-   Token bucket algorithm.
-   Enforce per-user/per-IP quotas.
-   Return 429 Too Many Requests.

------------------------------------------------------------------------

## Question 5: Search API optimization

-   Autocomplete: prefix search with trie/index.
-   Cache popular queries.
-   Pagination with cursors.
