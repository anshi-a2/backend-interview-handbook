# REST API Interview Questions & Answers (Java SDE-2)

Here's a list of the **Top REST API Interview Questions** with sample
**answers/solutions** tailored for Java backend engineers:

------------------------------------------------------------------------

## 🔑 REST API Interview Questions & Answers (Java SDE-2)

### 1. What are REST principles?

**Answer:**\
REST (Representational State Transfer) follows 6 architectural
constraints:\
- **Statelessness** → Each request contains all info (no session).\
- **Client-Server** → Separation of concerns.\
- **Cacheable** → Responses can be cached.\
- **Uniform Interface** → Consistent URIs, verbs, representations.\
- **Layered System** → Intermediaries allowed (proxy, gateway).\
- **Code on Demand** (optional) → Server sends executable code.

👉 Example in Spring Boot:

``` java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return ResponseEntity.ok(userService.findById(id));
}
```

------------------------------------------------------------------------

### 2. What are idempotent methods in REST?

**Answer:**\
- **Idempotent** = same result no matter how many times it is called.\
- `GET`, `PUT`, `DELETE` are idempotent.\
- `POST` is **not idempotent** (creates new resource each time).

👉 Example:\
- `DELETE /users/1` multiple times → same state (user deleted).\
- `POST /users` multiple times → creates multiple users → not
idempotent.

------------------------------------------------------------------------

### 3. Difference between PUT vs PATCH vs POST?

**Answer:**\
- **POST** → Create a new resource (`/users`).\
- **PUT** → Replace entire resource (`/users/1`).\
- **PATCH** → Partially update resource
(`/users/1 { "email": "x@y.com" }`).

👉 Example in Spring Boot:

``` java
@PatchMapping("/users/{id}")
public ResponseEntity<User> updateEmail(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    return ResponseEntity.ok(userService.updatePartial(id, updates));
}
```

------------------------------------------------------------------------

### 4. How do you handle errors in REST API?

**Answer:**\
Use **standard HTTP codes + error response body**.\
- `200 OK` → success\
- `400 Bad Request` → invalid input\
- `404 Not Found` → resource missing\
- `500 Internal Server Error` → server issue

👉 Example Error JSON:

``` json
{
  "timestamp": "2025-09-18T20:00:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "User not found",
  "path": "/users/10"
}
```

👉 Spring Boot `@ControllerAdvice`:

``` java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<Map<String, String>> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(Map.of("error", ex.getMessage()));
    }
}
```

------------------------------------------------------------------------

### 5. How do you secure REST APIs?

**Answer:**\
- Use **HTTPS**.\
- Authentication → **JWT, OAuth2.0, Basic Auth**.\
- Authorization → Role-based / claims-based.\
- Input validation (avoid SQL injection, XSS).\
- Rate limiting, throttling.

👉 Example JWT filter in Spring Security:

``` java
public class JwtFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        String token = request.getHeader("Authorization");
        if (token != null && jwtService.validate(token)) {
            UsernamePasswordAuthenticationToken auth =
                new UsernamePasswordAuthenticationToken(jwtService.getUser(token), null, List.of());
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        filterChain.doFilter(request, response);
    }
}
```

------------------------------------------------------------------------

### 6. How do you handle pagination in REST APIs?

**Answer:**\
- Use query params → `GET /users?page=1&size=20`.\
- Response should contain metadata.

👉 Example Response:

``` json
{
  "data": [ ...users... ],
  "page": 1,
  "size": 20,
  "totalElements": 120,
  "totalPages": 6
}
```

👉 In Spring Boot:

``` java
@GetMapping("/users")
public Page<User> getUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

------------------------------------------------------------------------

### 7. How do you version REST APIs?

**Answer:**\
- **URI Versioning** → `/api/v1/users`\
- **Header Versioning** → `Accept: application/vnd.myapp.v1+json`\
- **Query Param Versioning** → `/users?version=1`

👉 Best Practice: URI versioning for simplicity.

------------------------------------------------------------------------

### 8. How do you improve performance of REST APIs?

**Answer:**\
- Use **caching** (ETags, Cache-Control).\
- Reduce payload (compression, projections).\
- Async processing for long tasks.\
- Database optimization (indexes, batch queries).\
- Use **Redis** for caching.\
- Pagination for large datasets.

------------------------------------------------------------------------

### 9. How do you document REST APIs?

**Answer:**\
- Swagger / OpenAPI (Springdoc, springfox).\
- Generate interactive docs → `/swagger-ui.html`.

------------------------------------------------------------------------

### 10. Scenario Question (Common at SDE-2):

👉 *"How would you design a REST API for a URL shortener like bit.ly?"*\
**Answer:**\
- `POST /shorten` → takes long URL, returns short code.\
- `GET /{shortCode}` → redirects to original URL.\
- Use DB with index on shortCode.\
- Add caching layer (Redis) for quick lookup.\
- Handle collisions with hashing + retries.\
- Security → prevent malicious URLs.\
- Analytics → count clicks, track metadata.
