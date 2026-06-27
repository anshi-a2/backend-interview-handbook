# 01_REST_to_GraphQL_Migration_Guide.md

> **Goal**
>
> This guide explains how to migrate a production application from **REST APIs to GraphQL**, why organizations adopt GraphQL, migration strategies, architectural changes, schema design, common challenges, production best practices, and interview questions.
>
> This document is targeted towards **SDE-2 / Senior Backend Engineers**.

---

# Table of Contents

1. Why Migrate from REST to GraphQL?
2. REST Architecture
3. GraphQL Architecture
4. REST vs GraphQL
5. Migration Strategy
6. Designing a GraphQL Schema
7. Resolver Architecture
8. Authentication & Authorization
9. Common Migration Challenges
10. Production Migration Strategy
11. Best Practices
12. Frequently Asked Interview Questions

---

# 1. Why Migrate from REST to GraphQL?

Many organizations start with REST because it is simple and widely adopted. As applications grow, especially mobile applications and microservices, REST APIs often become inefficient.

## Problems with REST

### Over Fetching

REST returns more data than required.

Example

```
GET /users/101
```

Response

```json
{
  "id":101,
  "name":"John",
  "email":"john@test.com",
  "phone":"12345",
  "address":"USA",
  "profilePicture":"...",
  "orders":[...],
  "preferences":{...}
}
```

Client only needs

```
Name
```

but receives everything.

---

### Under Fetching

Sometimes one API doesn't provide enough data.

```
GET /orders/100

↓

Customer Id

↓

GET /customers/50

↓

Address

↓

GET /addresses/20
```

Three REST calls for one screen.

---

### Multiple Network Calls

Mobile application

```
Dashboard

↓

User API

↓

Order API

↓

Notification API

↓

Product API
```

Four API calls.

Higher latency.

---

### Versioning Problems

REST

```
/v1/users

/v2/users

/v3/users
```

Older versions must remain available for backward compatibility, increasing maintenance effort.

---

## Why GraphQL?

GraphQL allows the client to request exactly what it needs.

```
Client

↓

GraphQL

↓

Single Response
```

Benefits

- One endpoint
- No over-fetching
- No under-fetching
- Better mobile performance
- Strongly typed schema
- Flexible queries
- Faster frontend development

---

# 2. REST Architecture

Typical Spring Boot REST architecture

```
Client

↓

REST Controller

↓

Service

↓

Repository

↓

Database
```

Each URL represents a resource.

Examples

```
GET /users

GET /orders

POST /payments

DELETE /products
```

Characteristics

- Multiple endpoints
- HTTP verb based
- Versioned APIs
- Resource-oriented

---

# 3. GraphQL Architecture

GraphQL exposes **one endpoint**.

```
Client

↓

POST /graphql

↓

GraphQL Engine

↓

Query Resolver

↓

Service Layer

↓

Repository

↓

Database
```

Instead of URLs, clients send **queries**.

Example

```graphql
query{
  user(id:1){
      name
      email
  }
}
```

Only requested fields are returned.

---

# 4. REST vs GraphQL

| REST | GraphQL |
|------|----------|
| Multiple endpoints | Single endpoint |
| Fixed response | Client decides fields |
| Over-fetching possible | No over-fetching |
| Under-fetching possible | Single query can fetch related data |
| URL based | Schema based |
| Versioning required | Schema evolution |
| Easy HTTP caching | Requires different caching strategies |
| Simple learning curve | More flexible but more complex |

---

## Example

REST

```
GET /orders/100

↓

GET /customer/20

↓

GET /payments/10
```

GraphQL

```graphql
query{
 order(id:100){
   id
   customer{
      name
   }
   payment{
      status
   }
 }
}
```

One request.

---

# 5. Migration Strategy

Never replace REST overnight.

Recommended approach

```
REST APIs

↓

Introduce GraphQL

↓

Keep Both APIs

↓

New Features in GraphQL

↓

Frontend Migration

↓

Monitor

↓

Deprecate REST

↓

Remove REST
```

Benefits

- Zero downtime
- Low risk
- Easy rollback
- Incremental migration

---

## Step 1

Keep existing REST APIs.

---

## Step 2

Introduce GraphQL alongside REST.

```
REST

↓

GraphQL

↓

Same Service Layer
```

Business logic remains unchanged.

---

## Step 3

Frontend gradually migrates.

Old screen

↓

REST

New screen

↓

GraphQL

---

## Step 4

After all consumers migrate,

remove unused REST endpoints.

---

# 6. Designing a GraphQL Schema

GraphQL is schema-first.

Example

```graphql
type User{

   id:ID!

   name:String!

   email:String!

}
```

---

## Query

```graphql
type Query{

   user(id:ID!):User

}
```

---

## Mutation

```graphql
type Mutation{

   createUser(input:UserInput):User

}
```

---

## Input Types

```graphql
input UserInput{

   name:String!

   email:String!

}
```

---

## Best Practices

✔ Keep schema simple.

✔ Avoid exposing database entities directly.

✔ Use meaningful names.

✔ Make fields nullable only when necessary.

✔ Design APIs around business concepts, not database tables.

---

# 7. Resolver Architecture

Resolvers are similar to REST Controllers.

REST

```
Controller

↓

Service

↓

Repository
```

GraphQL

```
Resolver

↓

Service

↓

Repository
```

---

## Query Resolver

Responsible for reading data.

```
Client Query

↓

Resolver

↓

Service
```

---

## Mutation Resolver

Responsible for

- Insert
- Update
- Delete

---

## Nested Resolver

Example

```
Order

↓

Customer

↓

Address
```

Each field may invoke another resolver.

---

# 8. Authentication & Authorization

GraphQL usually uses the same authentication mechanisms as REST.

Example

```
Client

↓

JWT Token

↓

GraphQL

↓

Authentication

↓

Authorization

↓

Resolver
```

Authorization should be enforced at the resolver or service layer.

Example

```
Admin

↓

Delete User

Allowed

------------

Customer

↓

Delete User

Denied
```

---

# 9. Common Migration Challenges

---

## N+1 Query Problem

Suppose

```
100 Orders

↓

100 Customer Queries
```

Total

```
101 Database Queries
```

Solution

```
DataLoader
```

which batches and caches related lookups within a request.

---

## Deep Queries

Clients can request

```
User

↓

Orders

↓

Products

↓

Supplier

↓

Warehouse
```

Very deep queries may affect performance.

Solutions

- Depth limits
- Query complexity limits

---

## Large Responses

Clients may request hundreds of fields.

Solutions

- Query complexity analysis
- Pagination
- Limits

---

## Circular References

Example

```
User

↓

Orders

↓

Customer

↓

Orders

↓

Customer
```

Can create recursive query structures.

Design schema carefully to avoid infinite nesting.

---

## Caching

REST

```
HTTP Cache
```

GraphQL

Requires different strategies such as

- Persisted queries
- Client-side caching
- Response caching
- Redis

---

# 10. Production Migration Strategy

Recommended deployment

```
Development

↓

GraphQL Service

↓

Unit Testing

↓

Integration Testing

↓

Performance Testing

↓

Staging

↓

Feature Flag

↓

Canary Deployment

↓

Production
```

---

## Migration Tips

✔ Keep REST and GraphQL together initially.

✔ Reuse existing services.

✔ Avoid duplicate business logic.

✔ Monitor query performance.

✔ Add query depth limits.

✔ Log slow queries.

✔ Monitor resolver execution time.

---

# 11. Best Practices

### Schema

✔ Design schema around business objects.

✔ Keep naming consistent.

✔ Avoid exposing database models directly.

---

### Performance

✔ Use DataLoader.

✔ Enable pagination.

✔ Limit query depth.

✔ Cache expensive queries.

---

### Security

✔ JWT Authentication.

✔ Resolver-level authorization.

✔ Rate limiting.

✔ Query complexity limits.

---

### Migration

✔ Migrate screen by screen.

✔ Keep REST as fallback.

✔ Monitor production metrics.

✔ Remove REST only after confirming all consumers have migrated.

---

# 12. Frequently Asked Interview Questions

---

## Q1. Why migrate from REST to GraphQL?

**Answer**

REST APIs often suffer from over-fetching, under-fetching, and multiple network calls. GraphQL allows clients to request exactly the data they need in a single request, improving flexibility and reducing network overhead.

---

## Q2. Should every application use GraphQL?

**Answer**

No. GraphQL adds complexity. It is beneficial for applications with complex data relationships, multiple clients (web/mobile), or frequent UI changes. Simple CRUD applications often work well with REST.

---

## Q3. What is the biggest challenge during migration?

Typical answers

- Schema design
- N+1 query problem
- Authentication
- Caching
- Monitoring
- Frontend migration

---

## Q4. How would you migrate without downtime?

**Answer**

Introduce GraphQL alongside existing REST APIs. Reuse the same service layer, migrate clients gradually using feature flags, monitor production traffic, and deprecate REST endpoints only after all consumers have switched.

---

## Q5. What is a Resolver?

**Answer**

A resolver is responsible for fetching data for a GraphQL field. It acts similarly to a REST controller but operates at the field level, allowing GraphQL to resolve nested objects dynamically.

---

## Q6. What is the N+1 problem?

**Answer**

The N+1 problem occurs when fetching related data results in one query for the parent records and additional queries for each child record, causing excessive database calls. GraphQL applications typically solve this using DataLoader.

---

## Q7. How do you secure GraphQL APIs?

**Answer**

Authentication is commonly handled using JWT or OAuth2, while authorization is enforced in resolvers or the service layer. Additional protections include query depth limits, query complexity analysis, rate limiting, and disabling introspection in production if appropriate.

---

## Q8. When would you avoid GraphQL?

**Answer**

Avoid GraphQL when:
- The application is simple CRUD.
- There are few relationships between entities.
- HTTP caching is sufficient.
- The added complexity does not provide meaningful value.

---

# Senior Engineer Takeaways

When discussing a REST-to-GraphQL migration, focus on:

- **Business Motivation**: Reduce over-fetching, improve client flexibility, simplify frontend development.
- **Architecture**: Reuse the existing service layer while introducing GraphQL incrementally.
- **Migration Strategy**: Run REST and GraphQL side by side, use feature flags, and perform a phased rollout.
- **Performance**: Address N+1 queries with DataLoader, implement pagination, and monitor query complexity.
- **Security & Operations**: Enforce authentication and authorization, apply query limits, monitor resolver performance, and maintain rollback plans.

A senior-level interview answer should demonstrate not only **how GraphQL works**, but also **how to safely introduce it into a production system while minimizing risk and avoiding disruption to existing clients**.


# 02_GraphQL_Internals_Execution_Flow_Production.md

> **Goal**
>
> This guide explains **how GraphQL works internally**, from the moment a client sends a request until the response is returned. It covers GraphQL execution, resolvers, DataLoader, caching, security, pagination, subscriptions, Spring Boot integration, production issues, performance tuning, and senior interview questions.
>
> This is intended for **SDE-2 / Senior Backend Engineer** interviews.

---

# Table of Contents

1. End-to-End GraphQL Request Flow
2. GraphQL Execution Engine
3. GraphQL Schema Internals
4. Resolver Lifecycle
5. DataLoader & N+1 Problem
6. Pagination
7. Caching
8. Authentication & Authorization
9. GraphQL Subscriptions
10. Spring Boot Integration
11. Performance Optimization
12. Production Issues
13. Monitoring & Observability
14. Best Practices
15. Frequently Asked Interview Questions

---

# 1. End-to-End GraphQL Request Flow

A GraphQL request goes through several stages before data is returned.

```
Client

↓

POST /graphql

↓

GraphQL Controller

↓

Parser

↓

Query AST

↓

Validator

↓

Execution Engine

↓

Resolvers

↓

Service Layer

↓

Repository

↓

Database

↓

Response Builder

↓

JSON Response
```

Unlike REST, GraphQL always hits the **same endpoint**, and the query itself determines what data is fetched.

---

## Example Request

```graphql
query {
  user(id: 1) {
    id
    name
    email
  }
}
```

---

## Response

```json
{
  "data": {
    "user": {
      "id": 1,
      "name": "John",
      "email": "john@test.com"
    }
  }
}
```

---

# 2. GraphQL Execution Engine

The execution engine processes a request in multiple phases.

```
Receive Query

↓

Parse

↓

Create AST

↓

Validate

↓

Execute

↓

Resolve Fields

↓

Build Response
```

---

## Step 1 – Parsing

GraphQL converts the query into an **Abstract Syntax Tree (AST)**.

```
query {

 user {

   name

 }

}
```

↓

AST

```
Query

└── User

      └── name
```

---

## Step 2 – Validation

The engine validates:

✔ Query syntax

✔ Requested fields

✔ Field types

✔ Required arguments

✔ Schema compatibility

Example

```graphql
user{

salary
}
```

If **salary** doesn't exist,

↓

Validation Error

---

## Step 3 – Execution

The execution engine starts resolving fields.

```
Query

↓

Resolver

↓

Business Logic

↓

Database
```

---

# 3. GraphQL Schema Internals

Everything revolves around the schema.

```
Schema

├── Query

├── Mutation

├── Subscription

├── Types

├── Input Types

├── Enums

├── Interfaces

└── Unions
```

---

## Query

Read data.

```graphql
type Query{

 user(id:ID!):User

}
```

---

## Mutation

Modify data.

```graphql
type Mutation{

 createUser(input:UserInput):User

}
```

---

## Subscription

Real-time updates.

```graphql
type Subscription{

 orderCreated:Order

}
```

---

# 4. Resolver Lifecycle

Resolvers are GraphQL's equivalent of controllers.

```
Query

↓

Root Resolver

↓

Child Resolver

↓

Nested Resolver

↓

Database
```

---

## Example

```
User

↓

Orders

↓

Products

↓

Category
```

Execution

```
User Resolver

↓

Orders Resolver

↓

Products Resolver

↓

Category Resolver
```

Each field may trigger another resolver.

---

## Parallel Execution

Independent fields execute in parallel where possible.

```
User

↓

Email

Phone

Orders

Preferences
```

GraphQL can resolve sibling fields concurrently, improving response time.

---

# 5. DataLoader & N+1 Problem

One of the most important GraphQL interview topics.

---

## N+1 Problem

Suppose

```
100 Orders
```

Each order loads its customer.

```
Orders Query

↓

Customer Query

↓

Customer Query

↓

Customer Query

↓

100 Times
```

Total

```
101 SQL Queries
```

Very inefficient.

---

## Solution — DataLoader

Instead of

```
1

1

1

1

1
```

GraphQL batches requests.

```
1

↓

Batch

↓

SELECT * FROM customer

WHERE id IN(...)
```

Only

```
2 Queries
```

are executed.

---

## Internal Flow

```
Resolvers

↓

DataLoader

↓

Batch Requests

↓

Single SQL Query

↓

Return Map

↓

Resolvers
```

Benefits

✔ Eliminates N+1

✔ Request-scoped cache

✔ Faster execution

---

# 6. Pagination

Never return unlimited data.

---

## Offset Pagination

```
?page=2

&size=20
```

Simple but inefficient for very large datasets.

---

## Cursor Pagination

GraphQL commonly uses cursors.

```
after:"abc123"

first:20
```

Advantages

- Stable results
- Better performance
- Recommended for large datasets

---

## Relay Style

```
edges

↓

node

↓

cursor
```

Industry standard for GraphQL pagination.

---

# 7. Caching

Unlike REST,

GraphQL cannot rely solely on HTTP caching because different queries can request different fields through the same endpoint.

---

## Client Cache

```
Apollo Client

↓

Memory Cache
```

---

## Response Cache

```
GraphQL

↓

Redis

↓

Response
```

---

## Persisted Queries

Instead of sending

```
Huge Query
```

Client sends

```
Query Id
```

Server retrieves the stored query.

Benefits

- Smaller payloads
- Faster execution
- Better security

---

# 8. Authentication & Authorization

Authentication

```
Client

↓

JWT

↓

GraphQL

↓

Authenticated User
```

Authorization

```
Resolver

↓

Role Check

↓

Business Logic
```

Example

```
Admin

↓

Delete User

Allowed

---------

Customer

↓

Delete User

Denied
```

Authorization should be enforced at the resolver or service layer.

---

# 9. GraphQL Subscriptions

Used for real-time communication.

Architecture

```
Client

↓

WebSocket

↓

Subscription

↓

GraphQL

↓

Publisher

↓

Response
```

Example

```
Order Created

↓

Subscription Event

↓

Client Receives Update
```

Common use cases

- Chat
- Notifications
- Live dashboards
- Stock prices

---

# 10. Spring Boot Integration

Spring Boot uses **Spring for GraphQL**.

Architecture

```
Controller

↓

@QueryMapping

↓

Service

↓

Repository

↓

Database
```

---

## Query Mapping

```java
@QueryMapping
public User user(Long id)
```

---

## Mutation Mapping

```java
@MutationMapping
```

---

## Schema Mapping

```java
@SchemaMapping
```

Used to resolve nested fields.

---

## Exception Handling

```
Exception

↓

GraphQL Exception Handler

↓

Error Response
```

GraphQL returns both

```
data

errors
```

in the same response when applicable.

---

# 11. Performance Optimization

---

## Query Complexity

Prevent expensive queries.

Example

```
User

↓

Orders

↓

Products

↓

Reviews

↓

Comments

↓

Replies
```

This may overload the server.

Solutions

- Complexity scoring
- Depth limiting

---

## Batch Database Calls

Always batch related queries.

Avoid

```
100 SQL Queries
```

Prefer

```
2 SQL Queries
```

---

## Persisted Queries

Reduces

- Payload size
- Parsing time
- Security risk

---

## Response Caching

Cache expensive responses.

---

## Monitor Slow Resolvers

Track

- Execution time
- SQL count
- Memory usage

---

# 12. Production Issues

---

## Slow Resolver

Reasons

- External API
- Database latency
- Blocking calls

---

## N+1 Problem

Most common production issue.

Solution

DataLoader.

---

## Deep Queries

Clients request excessive nesting.

Solution

Depth limit.

---

## Large Responses

Return thousands of objects.

Solution

Pagination.

---

## Expensive Queries

Need complexity analysis.

---

## Memory Usage

Large object graphs

↓

High memory consumption.

---

# 13. Monitoring & Observability

Monitor

- Query execution time
- Resolver execution time
- Error rate
- Request rate
- Query complexity
- Database calls
- Cache hit ratio

Typical stack

```
Spring Boot

↓

Micrometer

↓

Prometheus

↓

Grafana
```

Also log:

- Slow queries
- Failed resolvers
- Timeout errors

---

# 14. Best Practices

### Schema

✔ Design around business entities.

✔ Avoid exposing database models.

---

### Performance

✔ Use DataLoader.

✔ Enable pagination.

✔ Limit query depth.

✔ Cache expensive operations.

✔ Persist frequently used queries.

---

### Security

✔ JWT Authentication.

✔ Resolver-level authorization.

✔ Rate limiting.

✔ Complexity analysis.

✔ Disable introspection in production if not required.

---

### Monitoring

✔ Track resolver latency.

✔ Track SQL queries.

✔ Monitor cache performance.

✔ Alert on slow GraphQL queries.

---

# 15. Frequently Asked Interview Questions

---

## Q1. How does GraphQL execute a query?

**Answer**

The request is parsed into an Abstract Syntax Tree (AST), validated against the schema, executed by the GraphQL engine, and resolved field by field through resolvers. The execution engine collects all resolved values and constructs a JSON response.

---

## Q2. What is a Resolver?

**Answer**

A resolver is a function responsible for fetching data for a GraphQL field. Every field in a GraphQL query can have its own resolver, allowing nested and flexible data retrieval.

---

## Q3. Explain the N+1 Problem.

**Answer**

The N+1 problem occurs when one query fetches parent records and then executes an additional query for each child record. For example, loading 100 orders and executing 100 separate customer queries results in 101 database queries. DataLoader batches these requests into a single query, significantly improving performance.

---

## Q4. What is DataLoader?

**Answer**

DataLoader is a batching and caching utility that groups multiple data-fetch requests into a single database call and caches results within the scope of a single GraphQL request.

---

## Q5. Why is GraphQL harder to cache than REST?

**Answer**

REST uses different URLs for different resources, making HTTP caching straightforward. GraphQL uses a single endpoint, and each query can request a different combination of fields, so caching is typically implemented with persisted queries, client-side caches, response caches, or Redis.

---

## Q6. How do you secure GraphQL?

**Answer**

Authentication is usually implemented with JWT or OAuth2. Authorization is enforced within resolvers or the service layer. Additional protections include rate limiting, query complexity analysis, depth limits, persisted queries, and disabling schema introspection in production when appropriate.

---

## Q7. How do you improve GraphQL performance?

**Answer**

- Use DataLoader to eliminate N+1 queries.
- Implement pagination.
- Batch database operations.
- Cache expensive responses.
- Use persisted queries.
- Limit query depth and complexity.
- Monitor slow resolvers and optimize database access.

---

## Q8. What are GraphQL Subscriptions?

**Answer**

Subscriptions enable real-time communication using WebSockets. Clients subscribe to events, and the server pushes updates whenever those events occur, making subscriptions suitable for chat applications, notifications, and live dashboards.

---

# Senior Engineer Takeaways

When discussing GraphQL internals, focus on:

- **Execution Flow**: Parsing → AST → Validation → Resolver execution → Response.
- **Resolvers**: Field-level execution and nested resolution.
- **Performance**: DataLoader, batching, pagination, caching, and query complexity limits.
- **Security**: Authentication, authorization, depth limiting, and persisted queries.
- **Operations**: Monitoring resolver latency, database calls, and production metrics.

A senior-level explanation should demonstrate not only **how GraphQL executes queries**, but also **how to build, optimize, secure, and operate GraphQL APIs reliably in production**.
