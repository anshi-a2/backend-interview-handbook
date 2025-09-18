# Meta (Facebook) SDE-2 REST API Interview Questions & Solutions

## Question 1: GraphQL vs REST for social feed

-   REST: multiple round trips, simple.
-   GraphQL: single query, avoids over-fetching.
-   Use batching to reduce load.

------------------------------------------------------------------------

## Question 2: Comments API design

-   POST `/posts/{id}/comments`.
-   GET `/posts/{id}/comments?page=...`.
-   Cache hot comment threads.
-   Anti-abuse checks (rate limiting).

------------------------------------------------------------------------

## Question 3: Handling fan-out in newsfeed APIs

-   Push vs pull model.
-   Use queues for fan-out writes.
-   Cache precomputed feeds.

------------------------------------------------------------------------

## Question 4: Abuse prevention in APIs

-   Rate limit by IP/user/token.
-   Use captchas or ML models to detect spam.
-   Throttling suspicious clients.

------------------------------------------------------------------------

## Question 5: Reels upload & streaming API

-   POST `/reels` → upload metadata + video chunks.
-   Async processing (transcoding, compression).
-   GET `/reels/{id}` for streaming URLs.
