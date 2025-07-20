# GraphQL Interview Questions & Answers for Backend SDE-2

## 🔧 Core GraphQL Concepts

### 1. What is GraphQL and how is it different from REST?

* GraphQL is a query language and runtime for APIs that enables clients to request exactly the data they need.
* **Differences:**

  * Over-fetching/Under-fetching: Avoided in GraphQL.
  * Single Endpoint: Unlike REST's multiple endpoints.
  * No Versioning: Use deprecations instead.
  * Strongly Typed Schema: Defined using SDL.

### 2. Explain the components of a GraphQL schema.

* **Types**: Define shape of data.
* **Queries**: For fetching data.
* **Mutations**: For modifying data.
* **Resolvers**: Functions that fetch data for fields.
* **Scalars/Enums**: Base and custom types.
* **Input Types**: Structured inputs for mutations.
* **Directives**: Annotations like `@deprecated`, `@include`.

### 3. What are resolvers in GraphQL?

* Functions that fetch data for each field.
* Use async logic, access arguments, context, and parent fields.

### 4. Differences between Query, Mutation, and Subscription?

* **Query**: Read-only.
* **Mutation**: Write operations.
* **Subscription**: Real-time data (WebSocket).

### 5. What is the purpose of `__typename`?

* Identifies the type of object returned.
* Useful for unions, caching, introspection.

---

## 🛠 Advanced Usage / Backend Integration

### 6. Integrating GraphQL with backend

* **Node.js**: Apollo Server, Express-GraphQL.
* **Python**: Graphene, Ariadne.
* Define schema, implement resolvers, expose `/graphql`.

### 7. Handling errors in GraphQL responses

* Partial `data` and `errors` array.
* Use `formatError` and `extensions`.

```json
{
  "data": { "user": null },
  "errors": [{
    "message": "User not found",
    "path": ["user"],
    "extensions": { "code": "NOT_FOUND" }
  }]
}
```

### 8. Avoiding N+1 queries

* Use **DataLoader** to batch and cache DB calls.

### 9. Authentication and authorization

* **Auth**: Token/session-based, injected into context.
* **Authorization**: Role checks in resolvers or directives.

### 10. Securing GraphQL APIs

* Depth limiting, query complexity analysis.
* Rate limiting, disable introspection (prod), persisted queries.

---

## 📈 Performance and Best Practices

### 11. Pagination

* **Offset**:

```graphql
users(offset: 0, limit: 10) { id name }
```

* **Cursor-based**:

```graphql
users(first: 10, after: "cursor123") {
  edges { node { id name } }
  pageInfo { hasNextPage endCursor }
}
```

### 12. What is schema stitching?

* Combines multiple schemas into one.
* Use in monorepos or with legacy APIs.

### 13. Apollo Federation

* Compose GraphQL services into one graph.
* Gateway and subgraphs.

### 14. Persisted queries

* Predefined on server.
* Send query ID, not full query.
* Improves performance and security.

### 15. Managing breaking changes

* Use `@deprecated`, evolve schema.
* Tools: GraphQL Inspector, Apollo Studio.

---

## 🧩 Testing and Tooling

### 16. Testing GraphQL APIs

* **Unit tests**: Resolvers with mocks.
* **Integration tests**: End-to-end queries.
* Tools: Jest, Pytest, Apollo TestClient, Postman, GraphiQL.

### 17. Documentation Tools

* Introspection: GraphiQL, Apollo Studio.
* SDL-based, auto-generated docs.
* Typed clients generate documentation.

---

## 🧐 Scenario-based / Design Questions

### 18. Schema design for e-commerce app

```graphql
type User { id: ID!, name: String!, orders: [Order!]! }
type Order { id: ID!, total: Float!, products: [Product!]! }
type Product { id: ID!, name: String!, price: Float! }
```

* Trade-offs: Avoid deep nesting, batch resolvers, paginate lists.

### 19. Debugging slow queries

* Analyze resolver logs, DB performance.
* Use DataLoader.
* Add tracing/logging and limit complexity.

### 20. Composing GraphQL schemas in microservices

* Use Apollo Federation.
* Gateway composes subgraphs.
* Use `@key`, `@extends` for shared types.

---

Prepared for Backend SDE-2 Interviews — GraphQL Focused
