# REST API Study Guide for SDE-2 (Java Backend)

## 1. REST Fundamentals

### REST Principles

-   **Statelessness**: Each request contains all information needed.
-   **Uniform Interface**: Consistent structure (HTTP methods, URIs).
-   **Client-Server**: Separation of concerns.
-   **Cacheable**: Responses can be cached.
-   **Layered System**: Components can be layered.
-   **Code on Demand** (optional).

### HTTP Methods & Idempotency

-   **GET**: Safe, idempotent. Retrieve data.
-   **POST**: Not idempotent. Create resources.
-   **PUT**: Idempotent. Replace resource.
-   **PATCH**: Not always idempotent. Partial update.
-   **DELETE**: Idempotent. Remove resource.

### HTTP Status Codes

-   **2xx**: Success (200 OK, 201 Created, 204 No Content)
-   **3xx**: Redirect (301 Moved Permanently, 302 Found)
-   **4xx**: Client error (400 Bad Request, 401 Unauthorized, 403
    Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests)
-   **5xx**: Server error (500 Internal Server Error, 502 Bad Gateway,
    503 Service Unavailable)

### Example Request/Response

``` http
GET /users/123 HTTP/1.1
Host: api.example.com
Accept: application/json
```

``` json
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com"
}
```

------------------------------------------------------------------------

## 2. Advanced API Design

### Pagination

-   **Offset-based**: `/items?page=2&size=10` → simple, but expensive on
    large datasets.
-   **Cursor-based**: `/items?cursor=abc123&limit=10` → scalable, avoids
    skipping.

### Filtering & Sorting

`GET /products?category=shoes&sort=price,asc`

### Versioning

-   **URI**: `/api/v1/users`
-   **Header**: `Accept: application/vnd.myapp.v2+json`
-   **Pros/Cons**: URI easy, Header more flexible.

### Idempotency

-   Ensure retries don't cause duplicates (e.g., payments).
-   Use **idempotency keys**.

### Rate Limiting

-   **Token bucket / Leaky bucket** algorithms.
-   Return `429 Too Many Requests`.

### Error Handling

-   Use structured error responses:

``` json
{
  "error": "InvalidParameter",
  "message": "Email format is invalid"
}
```

------------------------------------------------------------------------

## 3. Security

### Authentication

-   **Basic Auth** (not recommended).
-   **API Keys** (simple, but static).
-   **OAuth2.0** (industry standard).
-   **JWT** (stateless tokens).

### Authorization

-   **RBAC**: Role-based (Admin, User).
-   **ABAC**: Attribute-based (location, device).

### CORS

-   Control cross-origin requests via headers.

### Attacks to Prevent

-   **CSRF**: Use anti-CSRF tokens.
-   **XSS**: Escape outputs, sanitize inputs.
-   **SQL Injection**: Use parameterized queries.

------------------------------------------------------------------------

## 4. Performance & Scalability

### Caching

-   **ETag/If-None-Match** headers.
-   **Redis/CDN caching**.
-   **Last-Modified** for validation.

### Async APIs

-   **Long Polling**, **WebSockets**, **SSE** for real-time.

### Bulk Endpoints

-   Reduce HTTP overhead: `POST /users/bulk`.

### Compression

-   Enable **gzip** or **Brotli**.

------------------------------------------------------------------------

## 5. Reliability & Resilience

### Circuit Breakers

-   Prevent cascading failures (Resilience4j).

### Retries & Backoff

-   Retry with exponential backoff.

### Timeout Strategies

-   Set client + server timeouts.

### Observability

-   Metrics: latency, error rate, throughput.
-   Logs: structured JSON logs.
-   Traces: OpenTelemetry.

------------------------------------------------------------------------

## 6. Testing & Documentation

### Testing

-   **Postman** for manual testing.
-   **JUnit + RestAssured** for automation.

### Mock Servers

-   Simulate APIs during dev.

### Documentation

-   **OpenAPI/Swagger** for API contracts.

------------------------------------------------------------------------

## 7. Java + Spring Boot Best Practices

### Annotations

-   `@RestController`, `@GetMapping`, `@PostMapping`.

### Exception Handling

``` java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handle(Exception e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(e.getMessage());
    }
}
```

### Validation

``` java
public class UserDto {
    @NotNull
    @Email
    private String email;
}
```

### DTO vs Entity

-   Avoid exposing DB entities directly.
-   Use DTO for API contracts.

### HATEOAS

-   Provide links in responses for discoverability.

------------------------------------------------------------------------

## 8. System Design with REST

### Large Scale Design

-   E-commerce API → `/products`, `/orders`, `/payments`.
-   Payment API with idempotency.
-   Ride booking (Uber) with async updates.

### REST vs Alternatives

-   **GraphQL**: Avoids over-fetching.
-   **gRPC**: High performance, binary protocol.
-   **Kafka/Event-driven**: Asynchronous.

### API Gateway

-   Handle authentication, logging, throttling.
-   Tools: Zuul, Spring Cloud Gateway, Kong.

------------------------------------------------------------------------

## 9. Common Interview Questions

### Q1: Design a REST API for user management

-   Endpoints: `/users` (POST), `/users/{id}` (GET/PUT/DELETE).

### Q2: How to implement idempotency in payments?

-   Use **idempotency keys**. Store processed requests.

### Q3: Difference between PUT and PATCH?

-   PUT: replace full resource.
-   PATCH: partial update.

### Q4: Explain pagination strategies.

-   Offset-based: simple, but slow.
-   Cursor-based: scalable.

### Q5: How to secure REST APIs in production?

-   HTTPS, OAuth2, JWT, input validation, WAF.

### Q6: REST vs SOAP vs GraphQL?

-   REST: widely used, flexible.
-   SOAP: strict contracts, enterprise legacy.
-   GraphQL: flexible queries, reduces over-fetching.

------------------------------------------------------------------------

# ✅ Final Notes

-   Master **REST fundamentals** first.
-   Be ready to **discuss trade-offs** (consistency vs latency, REST vs
    gRPC).
-   Always cover **security & scalability** aspects in interviews.
