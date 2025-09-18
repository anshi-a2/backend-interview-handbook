# Microsoft SDE-2 REST API Interview Questions & Solutions

## Question 1: Debug concurrency issue

-   Shared HashMap in controller → race condition.
-   Fix: ConcurrentHashMap or synchronized compute.

------------------------------------------------------------------------

## Question 2: Optimize paging for large datasets

-   Cursor-based pagination.
-   Indexing on sort fields.
-   Lightweight DTOs to reduce payload.

------------------------------------------------------------------------

## Question 3: Optimistic locking in REST APIs

-   Use version fields in DB rows.
-   `If-Match` headers for concurrency control.
-   Handle conflicts with 409 Conflict.

------------------------------------------------------------------------

## Question 4: API versioning strategy

-   URI versioning (`/api/v1/`).
-   Header versioning (`Accept: vnd.myapp.v1+json`).
-   Ensure backward compatibility.

------------------------------------------------------------------------

## Question 5: Handling partial failures in batch REST requests

-   Bulk request endpoint: POST with multiple items.
-   Return per-item status (multi-status 207).
-   Retry failed items only.
