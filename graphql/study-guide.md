
# 📘 GraphQL — Complete Study Guide (Java Focus — Deep Dive: DataLoader, Subscriptions, Federation, Security)

---

## Table of Contents
1. Introduction
2. Why GraphQL?
3. GraphQL vs REST — quick comparison
4. Setup options (libraries & tooling) for Java
5. Schema design best practices
6. Implementing GraphQL in Spring Boot (detailed)
   - Dependencies
   - Project structure
   - Schema files location
   - Controllers / Resolvers with `graphql-java` and Spring GraphQL
7. DataLoader — solving N+1 and batching in Java
   - What is the N+1 problem
   - DataLoader concepts: BatchLoader, DataLoaderRegistry
   - Example: Users → Orders batched fetch (code)
   - Integration with Spring GraphQL
8. Subscriptions — real-time GraphQL in Java
   - Protocol overview (WebSocket + GraphQL over WS / graphql-ws)
   - Spring GraphQL subscription support (Flux-based)
   - Example: chat messages subscription (code with Reactor Flux)
   - Deploy & scaling considerations
9. Federation — composing multiple GraphQL services
   - What is Apollo Federation vs schema-stitching
   - How federation works (entities, @key, _entities)
   - Java options: `graphql-java-federation` and implementing federated services
   - Example: product and review services + gateway (high-level & code snippets)
10. Security — authentication & authorization
    - Transport vs field-level security
    - JWT authentication with Spring Security + GraphQL
    - Role-based access control in resolvers
    - Query complexity, depth limiting & rate-limiting to mitigate abuse
    - Example: configuring JWT filter + method-level security for resolvers
11. Error handling & validation
12. Pagination, filtering, and cursor-based pagination (Relay-style)
13. Performance & caching strategies
14. Testing GraphQL APIs in Java
15. Deployment & operational concerns
16. Real-world project examples & templates
17. Interview Q&A (focused on Java/GraphQL topics)
18. References & further reading

---

## 1. Introduction
GraphQL is a query language and runtime for APIs that allows clients to request exactly the data they need. In modern architectures it is widely used as a flexible API layer, often acting as an API gateway or a BFF (Backend for Frontend).

This guide focuses on building, scaling, and securing GraphQL services in Java — especially using **Spring Boot** and **graphql-java / Spring GraphQL** libraries. We'll go deep on DataLoader, Subscriptions, Federation, and Security with complete code snippets and patterns you can copy into a project.

---

## 2. Why GraphQL?
- Single endpoint for queries, mutations and subscriptions.
- Clients request exactly the fields they need — avoids over/under fetching.
- Strongly-typed schema enables better tooling (autocomplete, validation).
- Good fit as an orchestration layer over microservices.

---

## 3. GraphQL vs REST — quick comparison
(Short table — see previous materials; focus here is on implementation details)

---

## 4. Setup options (libraries & tooling) for Java
Popular approaches for Java & Spring:
- **Spring GraphQL** (spring-boot-starter-graphql) — high level, integrates with Spring MVC, Spring Security, DataLoader, Reactor, and provides annotations like `@QueryMapping`, `@MutationMapping`, `@SubscriptionMapping`.
- **graphql-java** — the core GraphQL engine for the JVM. Spring GraphQL uses graphql-java under the hood.
- **graphql-java-tools** — helps map schema to resolvers (older pattern).
- **DataLoader (java-dataloader)** — batching & caching helper ported from JS DataLoader concept.
- **graphql-java-federation** / `apollo-federation-jvm` — libraries to support Apollo Federation in Java.
- **GraphiQL / GraphQL Playground / Altair** — developer UI tools.

Which to pick?
- For Spring Boot apps, use **Spring GraphQL** (easier integration). Use `graphql-java` primitives for low-level customization.

---

## 5. Schema design best practices (short recap)
- Design small focused types; prefer composition over huge monolithic types.
- Use `@deprecated` for deprecated fields, not new endpoints.
- Use pagination for lists; implement `first/after` Relay-style cursors when necessary.
- Keep mutation side-effects explicit (idempotency & validation).

---

## 6. Implementing GraphQL in Spring Boot (detailed)

### 6.1 Maven dependencies (Spring GraphQL + WebFlux + DataLoader + Security + WebSocket)
```xml
<!-- Spring Boot parent omitted for brevity -->
<dependencies>
    <!-- Spring Boot + GraphQL -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-graphql</artifactId>
    </dependency>

    <!-- If you prefer reactive stack (WebFlux) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <!-- DataLoader (graphql-java dataloader integration) -->
    <dependency>
        <groupId>com.graphql-java</groupId>
        <artifactId>java-dataloader</artifactId>
        <version>3.1.0</version>
    </dependency>

    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- JWT (example: jjwt) -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>

    <!-- Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

> Note: Versions change over time. If you use Spring Boot's starter, prefer aligning versions via Spring Boot's dependency management (parent POM).

### 6.2 Project structure (recommended)
```
src/main/java/com/example/graphql
  ├─ config/
  ├─ controller/         # GraphQL controllers / resolvers (annotated handlers)
  ├─ service/            # Business logic
  ├─ repository/         # Data access (JPA / reactive repos / DAO)
  ├─ dataloader/         # DataLoader batch functions & registry config
  └─ security/           # JWT filters, security config
src/main/resources/
  └─ graphql/
     └─ schema.graphqls   # schema files are automatically picked up by Spring GraphQL
```

### 6.3 Basic schema example (`src/main/resources/graphql/schema.graphqls`)
```graphql
type Query {
  user(id: ID!): User
  users: [User!]!
}

type Mutation {
  createUser(input: NewUserInput!): User!
}

type Subscription {
  messageAdded(channelId: ID!): Message!
}

type User {
  id: ID!
  name: String!
  orders: [Order!]!
}

type Order {
  id: ID!
  amount: Float!
  userId: ID!
}

input NewUserInput {
  name: String!
}
```

### 6.4 Simple resolver using Spring GraphQL (annotation style)
```java
package com.example.graphql.controller;

import org.springframework.graphql.data.method.annotation.QueryMapping;
import org.springframework.graphql.data.method.annotation.MutationMapping;
import org.springframework.stereotype.Controller;

@Controller
public class UserController {
    private final UserService userService;
    public UserController(UserService userService) { this.userService = userService; }

    @QueryMapping
    public User user(String id) {
        return userService.findById(id);
    }

    @MutationMapping
    public User createUser(NewUserInput input) {
        return userService.createUser(input.getName());
    }
}
```

---

## 7. DataLoader — solving N+1 and batching in Java

### 7.1 The N+1 problem explained
When resolving a list of parent objects (e.g. `users`), and for each user you load `orders` individually, you might make N DB calls (one per user) instead of a single batched query.

Example N+1:
```
Query users -> load 100 users
For each user -> SELECT * FROM orders WHERE userId = ?  (100 queries)
```

### 7.2 DataLoader concepts
- **BatchLoader<K, V>**: a function that receives a list of keys and returns list of values in the same order (or a Map/CompletableFuture).
- **DataLoader**: holds a BatchLoader and performs batching/caching per request context.
- **DataLoaderRegistry**: register DataLoaders per request so resolvers can access them.

### 7.3 Example: BatchLoader for orders by userId (JPA example)
```java
// dataloader/OrderBatchLoader.java
package com.example.graphql.dataloader;

import org.dataloader.BatchLoader;
import org.springframework.stereotype.Component;

import java.util.*;
import java.util.concurrent.CompletableFuture;
import java.util.stream.Collectors;

@Component
public class OrderBatchLoader implements BatchLoader<String, List<Order>> {

    private final OrderRepository orderRepository;

    public OrderBatchLoader(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Override
    public CompletableFuture<List<List<Order>>> load(List<String> userIds) {
        // fetch all orders for the list of userIds in single query
        List<Order> all = orderRepository.findByUserIdIn(userIds);
        // group by userId
        Map<String, List<Order>> grouped = all.stream().collect(Collectors.groupingBy(Order::getUserId));
        // prepare result in same order as userIds
        List<List<Order>> result = userIds.stream()
            .map(id -> grouped.getOrDefault(id, Collections.emptyList()))
            .collect(Collectors.toList());
        return CompletableFuture.completedFuture(result);
    }
}
```

### 7.4 Registering DataLoader per request (Spring GraphQL integration)
Spring GraphQL provides hooks for wiring DataLoaderRegistry per request. Example config:
```java
// config/DataLoaderConfig.java
package com.example.graphql.config;

import org.dataloader.DataLoader;
import org.dataloader.DataLoaderRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.example.graphql.dataloader.OrderBatchLoader;

@Configuration
public class DataLoaderConfig {
    private final OrderBatchLoader orderBatchLoader;
    public DataLoaderConfig(OrderBatchLoader orderBatchLoader) {
        this.orderBatchLoader = orderBatchLoader;
    }

    @Bean
    public DataLoaderRegistry dataLoaderRegistry() {
        DataLoaderRegistry registry = new DataLoaderRegistry();
        DataLoader<String, List<Order>> ordersByUserLoader = DataLoader.newMappedDataLoader(keys -> 
            orderBatchLoader.load(new ArrayList<>(keys)).thenApply(list -> {
                // convert list-of-lists into Map keyed by userId if using newMappedDataLoader
                Map<String, List<Order>> map = new HashMap<>();
                // this example assumes orderBatchLoader returns list aligned with keys order
                // adapt as needed
                return map;
            })
        );
        registry.register("ordersByUser", ordersByUserLoader);
        return registry;
    }
}
```
> Note: The exact DataLoader API usage may differ depending on `java-dataloader` version — above is illustrative. The goal: create a DataLoader named `ordersByUser` and register it per request.

### 7.5 Using DataLoader in resolvers
```java
package com.example.graphql.controller;

import org.dataloader.DataLoader;
import org.springframework.graphql.data.method.annotation.SchemaMapping;
import org.springframework.stereotype.Controller;

@Controller
public class UserResolver {
    @SchemaMapping(typeName = "User", field = "orders")
    public CompletableFuture<List<Order>> orders(User user, DataLoader<String, List<Order>> ordersLoader) {
        return ordersLoader.load(user.getId());
    }
}
```

### 7.6 Notes on caching & lifecycle
- DataLoader caches results for the lifetime of a request (DataLoaderRegistry per request).
- Do not use a global DataLoaderRegistry (would mix requests & cache problems).
- Clear cache if needed; consider cache size for large requests.

---

## 8. Subscriptions — real-time GraphQL in Java

### 8.1 Protocols & transport
GraphQL subscriptions typically use WebSockets and one of the protocols: `graphql-ws` (modern) or `subscriptions-transport-ws` (older). Spring GraphQL implements subscription handling and integrates with Reactor `Flux` for streaming results to clients.

### 8.2 Spring GraphQL subscription example (chat messages)
**Schema portion**
```graphql
type Subscription {
  messageAdded(channelId: ID!): Message!
}

type Message {
  id: ID!
  channelId: ID!
  content: String!
  sender: String!
  timestamp: String!
}
```

**Publisher (service) using Reactor Sinks**
```java
// service/MessageService.java
package com.example.graphql.service;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Sinks;
import org.springframework.stereotype.Service;

@Service
public class MessageService {
    private final Sinks.Many<Message> sink = Sinks.many().multicast().onBackpressureBuffer();

    public void publish(Message message) {
        sink.tryEmitNext(message);
    }

    public Flux<Message> getMessagesForChannel(String channelId) {
        return sink.asFlux()
                   .filter(msg -> msg.getChannelId().equals(channelId));
    }
}
```

**Subscription resolver**
```java
// controller/SubscriptionController.java
package com.example.graphql.controller;

import org.springframework.graphql.data.method.annotation.SubscriptionMapping;
import reactor.core.publisher.Flux;
import org.springframework.stereotype.Controller;

@Controller
public class SubscriptionController {
    private final MessageService messageService;

    public SubscriptionController(MessageService messageService) {
        this.messageService = messageService;
    }

    @SubscriptionMapping
    public Flux<Message> messageAdded(String channelId) {
        return messageService.getMessagesForChannel(channelId);
    }
}
```

**Client (GraphQL over WebSocket) example**
A client can connect using a GraphQL WS client (Apollo, graphql-ws) and subscribe:
```graphql
subscription OnMessageAdded($channelId: ID!) {
  messageAdded(channelId: $channelId) {
    id
    content
    sender
    timestamp
  }
}
```

### 8.3 Backpressure & scaling
- Use Reactor `Sinks` and handle backpressure carefully.
- For scaling across instances, use a distributed pub/sub (Redis, Kafka, or other messaging system) — each instance should publish and subscribe to the channel so all subscribers receive events regardless of which instance they connect to.
- For guaranteed delivery, persist messages and allow replay via query endpoints (subscriptions are typically transient streams).

---

## 9. Federation — composing multiple GraphQL services

### 9.1 What is Federation?
Apollo Federation is a pattern and set of rules for composing multiple GraphQL services into a single graph. Services export "entities" and the Gateway composes schemas and resolves cross-service references.

Key concepts:
- `@key` directive on entity types
- `_Entity` and `_Service` types used by the gateway
- Gateway performs entity resolution by delegating to subgraphs

### 9.2 Java support for Federation
- `graphql-java-federation` - community libraries that add federation support on top of `graphql-java`.
- Implementations include support for transforming local schema to include federation directives and implementing entity resolvers.

### 9.3 Example: Product & Review services (high-level)
**Product service schema (product-service)**
```graphql
type Product @key(fields: "id") {
  id: ID!
  name: String!
  price: Float
}
type Query { productById(id: ID!): Product }
```
**Review service schema (review-service)**
```graphql
type Review {
  id: ID!
  productId: ID!
  content: String!
}

extend type Product @key(fields: "id") {
  id: ID! @external
  reviews: [Review!]!
}
```
**Gateway**
- Runs Apollo Gateway (Node) or Java-based gateway compatible with federation spec.
- Gateway composes schemas and resolves `Product.reviews` by calling review-service with `productId`.

### 9.4 Implementing entity resolvers in Java
- Annotate entities with `@key` directive in schema.
- Provide resolvers for `_entities` type to resolve references passed by gateway.

> Federation requires compatible gateway — many teams use Apollo Gateway (Node.js). There are JVM gateway implementations too; evaluate based on your stack.

---

## 10. Security — authentication & authorization

### 10.1 Overview: Where to secure
- **Transport**: HTTPS, secure WebSocket (wss://)
- **Authentication**: who is the caller? (JWT / OAuth2 / mTLS)
- **Authorization**: what operations can they do? (RBAC / ABAC)
- **Field-level security**: restrict sensitive fields
- **Query complexity limits & depth limiting**: prevent expensive queries and DoS

### 10.2 JWT + Spring Security example (high-level)

**1) Add Spring Security dependency** (already shown in pom).

**2) Configure a JWT filter that authenticates HTTP requests and WebSocket handshake requests**:
```java
// security/JwtAuthenticationFilter.java
package com.example.graphql.security;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import org.springframework.security.core.Authentication;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Collections;

public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final String secretKey = "replace-with-secure-secret";

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws IOException, javax.servlet.ServletException {
        String authHeader = request.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            try {
                Claims claims = Jwts.parserBuilder().setSigningKey(secretKey.getBytes()).build().parseClaimsJws(token).getBody();
                String username = claims.getSubject();
                Authentication auth = new UsernamePasswordAuthenticationToken(username, null, Collections.emptyList());
                SecurityContextHolder.getContext().setAuthentication(auth);
            } catch (Exception e) {
                // invalid token - ignore or set error
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

**3) Security configuration**
```java
// security/SecurityConfig.java
package com.example.graphql.security;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.context.annotation.Bean;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests(auth -> auth
                .antMatchers("/graphiql", "/graphql").permitAll() // allow playground if needed
                .anyRequest().authenticated()
            )
            .addFilterBefore(new JwtAuthenticationFilter(), org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }
}
```

**4) Accessing authenticated user in GraphQL resolvers**
Spring Security's `SecurityContextHolder` works normally inside resolvers. Alternatively inject `Principal`:
```java
@QueryMapping
public User me(Principal principal) {
    String username = principal.getName();
    return userService.findByUsername(username);
}
```

### 10.3 Field-level security & method-level protection
Use Spring Security annotations:
```java
@PreAuthorize("hasRole('ADMIN')")
@MutationMapping
public Product createProduct( ... ) { ... }
```

### 10.4 Query complexity and depth limiting
- Use `MaxQueryComplexityInstrumentation` and `MaxQueryDepthInstrumentation` from `graphql-java` instrumentation APIs to reject overly expensive queries.
- Example integration:
```java
import graphql.execution.instrumentation.Instrumentation;
import graphql.execution.instrumentation.dataloader.DataLoaderDispatcherInstrumentation;
import graphql.execution.instrumentation.tracing.TracingInstrumentation;
import graphql.validation.rules.OnValidationErrorStrategy;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GraphQLInstrumentationConfig {
    @Bean
    public Instrumentation instrumentation() {
        // compose instrumentations: DataLoader dispatcher, tracing, custom complexity limiter...
        return new DataLoaderDispatcherInstrumentation();
    }
}
```

> Implement a custom complexity estimator to assign costs to fields, then fail queries that exceed a threshold.

---

## 11. Error handling & validation
- Throw `GraphQLError` (or implement `DataFetcherExceptionResolver`) to control GraphQL error output (avoid leaking stack traces).
- Validate input DTOs (use `javax.validation` annotations) and map validation errors to user-friendly GraphQL errors.
- Distinguish between user errors (client mistakes) and system errors (server exceptions).

Example: custom exception resolver
```java
@Component
public class GraphqlExceptionResolver implements DataFetcherExceptionResolver {
    @Override
    public CompletableFuture<List<GraphQLError>> resolveException(DataFetcherExceptionResolverParameters params) {
        Throwable ex = params.getException();
        if (ex instanceof MyDomainException) {
            return CompletableFuture.completedFuture(List.of(new GenericGraphQLError(ex.getMessage())));
        }
        return CompletableFuture.completedFuture(List.of(new GenericGraphQLError("Internal server error")));
    }
}
```

---

## 12. Pagination, filtering, and cursor-based pagination (Relay-style)
- For large lists use cursor-based pagination to avoid offset-based problems.
- Relay-style: `edges`, `node`, `cursor`, and `pageInfo` fields.
- Implementation approach: use opaque cursor (base64 encode of `id:timestamp`) and a deterministic ordering field (createdAt + id).

Schema example (simplified):
```graphql
type Query {
  posts(first: Int, after: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}
```

---

## 13. Performance & caching strategies
- Use DataLoader for batching.
- Cache frequently used read-only fields at the application or CDN layer (Response cache).
- Use persisted queries to reduce query parsing overhead and prevent malicious queries.
- Use HTTP caching for queries that are safe and cacheable.
- Use CDN and edge caching for public data.

---

## 14. Testing GraphQL APIs in Java
- Unit test resolvers with mocked services.
- Integration tests using `WebTestClient` (for WebFlux) or `MockMvc` (for servlet stack) to send GraphQL queries to `/graphql` endpoint.
- Use `graphql-java` `ExecutionInput` directly for unit testing `GraphQL` objects.

Example with WebTestClient (simplified):
```java
webTestClient.post().uri("/graphql")
   .contentType(MediaType.APPLICATION_JSON)
   .bodyValue(Map.of("query", "{ users { id name } }"))
   .exchange()
   .expectStatus().isOk()
   .expectBody().jsonPath("$.data.users").isArray();
```

---

## 15. Deployment & operational concerns
- Use HTTPS / WSS for secure transport.
- Horizontal scaling requires session-less resolvers or shared state (Redis/Kafka) for subscriptions; prefer a distributed pub/sub for cross-instance subscriptions.
- Monitor metrics: request rate, error rate, subscription counts, DataLoader cache misses, slow resolvers.
- Use rate-limiting and query cost analysis to protect from abusive clients.

---

## 16. Real-world project examples & templates
- **Microservices Gateway**: GraphQL gateway (Spring GraphQL) that aggregates services: product-service, user-service, order-service. Use DataLoader for batching cross-service calls.
- **Realtime Collaboration**: GraphQL subscriptions for updates and Kafka/Redis as persistent event backbone to scale across nodes.
- **BFF for Mobile**: GraphQL tailors responses to mobile clients reducing bandwidth.

---

## 17. Interview Q&A (focused on Java/GraphQL)

**Q: How do you fix N+1 in GraphQL?**  
A: Use DataLoader to batch and cache requests, implementing a `BatchLoader` that accepts a list of keys and returns values in one DB call.

**Q: How do subscriptions scale in a multi-instance deployment?**  
A: Use an external pub/sub (Kafka/Redis) so each app instance publishes and subscribes; use sticky sessions or stateless authentication for websockets; consider message replay persistence for reliability.

**Q: How to protect GraphQL API from costly queries?**  
A: Use query complexity and depth limit instrumentation, rate limiting, persisted queries, and field-level cost assignments.

**Q: How to secure field-level access?**  
A: Apply authorization checks inside resolvers or use method-level `@PreAuthorize` annotations; for fine-grained control, implement field resolvers that consult an authorization service based on the authenticated principal.

---

## 18. References & further reading
- graphql-java docs (official)
- Spring GraphQL project documentation
- DataLoader (original concept by Facebook)
- Apollo Federation spec

---

# Appendix — Copy-ready Code Snippets (compressed)

Below are the important copy-ready snippets from above so you can paste directly into your project.

## DataLoader BatchLoader (simplified)
```java
// dataloader/OrderBatchLoader.java
public class OrderBatchLoader implements BatchLoader<String, List<Order>> {
    private final OrderRepository repo;
    public OrderBatchLoader(OrderRepository repo) { this.repo = repo; }
    @Override
    public CompletableFuture<List<List<Order>>> load(List<String> userIds) {
        List<Order> all = repo.findByUserIdIn(userIds);
        Map<String, List<Order>> grouped = all.stream().collect(Collectors.groupingBy(Order::getUserId));
        List<List<Order>> out = userIds.stream().map(id -> grouped.getOrDefault(id, Collections.emptyList())).collect(Collectors.toList());
        return CompletableFuture.completedFuture(out);
    }
}
```

## Subscription using Reactor `Flux`
```java
// service/MessageService.java
@Service
public class MessageService {
    private final Sinks.Many<Message> sink = Sinks.many().multicast().onBackpressureBuffer();
    public void publish(Message m) { sink.tryEmitNext(m); }
    public Flux<Message> forChannel(String channelId) {
        return sink.asFlux().filter(msg -> msg.getChannelId().equals(channelId));
    }
}
```

## JWT Filter (simplified)
```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final String secretKey = "replace-with-secure-secret";
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            Claims claims = Jwts.parserBuilder().setSigningKey(secretKey.getBytes()).build().parseClaimsJws(token).getBody();
            Authentication auth = new UsernamePasswordAuthenticationToken(claims.getSubject(), null, Collections.emptyList());
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        filterChain.doFilter(request, response);
    }
}
```



