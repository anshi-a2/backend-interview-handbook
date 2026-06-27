# 04_Spring_MVC_Request_Lifecycle_Internals.md

> **Goal**
>
> Understand **exactly what happens internally** when a client sends an HTTP request to a Spring Boot application.
>
> This is one of the **most frequently asked Spring Boot interview topics** because it covers Spring MVC internals, DispatcherServlet, Filters, Interceptors, Controllers, Jackson, Exception Handling, and the complete request lifecycle.

---

# Table of Contents

1. End-to-End Request Flow
2. What Happens When User Hits an API?
3. Embedded Tomcat
4. Servlet Fundamentals
5. DispatcherServlet Internals
6. Handler Mapping
7. Handler Adapter
8. Controller Execution
9. Service Layer
10. Repository Layer
11. Database Communication
12. Returning Response
13. Jackson Serialization
14. Filters
15. Interceptors
16. Exception Handling
17. Request Lifecycle Diagram
18. Threading Model
19. Common Production Issues
20. Best Practices
21. Interview Summary

---

# 1. Complete Request Flow

Suppose user calls

```
GET /users/101
```

Complete request journey

```
Browser

↓

DNS

↓

Load Balancer

↓

NGINX

↓

Tomcat

↓

Servlet Filter

↓

DispatcherServlet

↓

Handler Mapping

↓

Handler Adapter

↓

Controller

↓

Service

↓

Repository

↓

Hibernate

↓

MySQL

↓

Repository

↓

Service

↓

Controller

↓

Jackson

↓

DispatcherServlet

↓

Tomcat

↓

Browser
```

This is the complete lifecycle of every HTTP request.

---

# 2. What Happens When User Hits an API?

Example

```
GET /users/101
```

Imagine a Spring Boot application exposing

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id){
        ...
    }

}
```

Internally, Spring executes dozens of steps before your method runs.

---

# 3. Client Side

```
Browser

↓

DNS Lookup

↓

IP Address

↓

TCP Connection

↓

HTTP Request
```

Example

```
GET /users/101

Host: api.company.com
```

---

# 4. Load Balancer

Production systems usually have multiple application instances.

```
Client

↓

Load Balancer

↓

Spring Boot Instance 1

Spring Boot Instance 2

Spring Boot Instance 3
```

The load balancer forwards the request to one instance.

---

# 5. Embedded Tomcat

The request reaches Tomcat.

Tomcat responsibilities

- Accept TCP connection
- Parse HTTP request
- Create HttpServletRequest
- Create HttpServletResponse
- Allocate worker thread
- Invoke DispatcherServlet

```
HTTP Request

↓

Tomcat

↓

Servlet Objects

↓

DispatcherServlet
```

---

# 6. Thread Creation

Every incoming request gets a worker thread.

```
Client Request

↓

Tomcat Thread Pool

↓

Thread-21

↓

DispatcherServlet
```

This thread processes the entire request.

After completion,

the thread returns to the pool.

---

# 7. Servlet Fundamentals

Spring MVC is built on top of Java Servlets.

Every HTTP request eventually reaches a Servlet.

Without Spring

```
HttpServlet
```

With Spring

```
DispatcherServlet
```

DispatcherServlet is simply a powerful Servlet.

---

# 8. DispatcherServlet

DispatcherServlet is the **Front Controller**.

Every request enters here first.

```
Incoming Request

↓

DispatcherServlet

↓

Find Controller

↓

Execute

↓

Return Response
```

Responsibilities

- Route requests
- Validate parameters
- Call controller
- Handle exceptions
- Serialize response

Think of it as the traffic controller of Spring MVC.

---

# 9. Request Processing Pipeline

```
HTTP Request

↓

Filters

↓

DispatcherServlet

↓

Handler Mapping

↓

Handler Adapter

↓

Controller

↓

Business Logic

↓

Response
```

---

# 10. Handler Mapping

DispatcherServlet doesn't know which controller to execute.

It asks

```
HandlerMapping
```

Example

```
GET /users/101
```

HandlerMapping searches

```
@GetMapping("/users/{id}")
```

Returns

```
UserController.getUser()
```

---

# 11. Handler Adapter

Different handlers can have different signatures.

HandlerAdapter knows

how to invoke them.

```
Controller Method

↓

Reflection

↓

Execute Method

↓

Return Object
```

---

# 12. Controller Execution

Example

```java
@GetMapping("/{id}")
public UserDTO getUser(Long id){

    return service.find(id);

}
```

Controller should only

- Validate request
- Call service
- Return response

Business logic belongs in Service.

---

# 13. Service Layer

Service contains business logic.

```
Controller

↓

Service

↓

Repository
```

Example

```
Validate User

↓

Business Rules

↓

Call Repository
```

---

# 14. Repository Layer

Repository communicates with database.

```
Repository

↓

Hibernate

↓

JDBC

↓

Database
```

Controller never talks directly to database.

---

# 15. Hibernate

Repository calls Hibernate.

Hibernate

↓

Generates SQL

↓

Executes SQL

↓

Maps ResultSet

↓

Creates Entity

---

# 16. Database

```
SELECT *

FROM USERS

WHERE ID=101
```

Database returns result.

---

# 17. Response Journey

Database

↓

Repository

↓

Service

↓

Controller

↓

DispatcherServlet

↓

Jackson

↓

HTTP Response

↓

Browser

---

# 18. Jackson Serialization

Suppose controller returns

```java
UserDTO
```

Jackson converts

```
Java Object

↓

JSON
```

Example

```json
{
   "id":101,
   "name":"John"
}
```

Spring does this automatically.

---

# 19. HttpMessageConverter

DispatcherServlet uses

```
HttpMessageConverter
```

Responsibilities

- Java → JSON
- JSON → Java
- XML
- Binary

Jackson is one implementation.

---

# 20. Request Body Flow

Incoming JSON

```json
{
  "name":"John"
}
```

Flow

```
JSON

↓

Jackson

↓

Java Object

↓

Controller
```

Annotations

```java
@RequestBody
```

---

# 21. Response Body Flow

Controller

↓

Java Object

↓

Jackson

↓

JSON

↓

Client

---

# 22. Filters

Filters execute **before Spring MVC**.

```
Request

↓

Filter

↓

DispatcherServlet
```

Examples

- Authentication
- Logging
- Compression
- CORS

Lifecycle

```
Before Request

↓

Chain.doFilter()

↓

After Response
```

---

# 23. Interceptors

Interceptors execute **inside Spring MVC**.

```
DispatcherServlet

↓

Interceptor

↓

Controller
```

Methods

```
preHandle()

↓

Controller

↓

postHandle()

↓

afterCompletion()
```

Used for

- Logging
- Authorization
- Metrics

---

# Filter vs Interceptor

| Filter | Interceptor |
|----------|------------|
| Servlet API | Spring MVC |
| Before DispatcherServlet | After DispatcherServlet |
| Works for all Servlets | Only Spring MVC |
| Good for CORS | Good for authentication, logging, auditing |

---

# 24. Exception Handling

Suppose

```
Repository

↓

throws Exception
```

Flow

```
Repository

↓

Service

↓

Controller

↓

DispatcherServlet

↓

@ControllerAdvice

↓

JSON Error
```

Example

```java
@RestControllerAdvice
public class GlobalExceptionHandler{

}
```

---

# 25. Spring Request Lifecycle

```
Client

↓

Load Balancer

↓

Tomcat

↓

Filter

↓

DispatcherServlet

↓

Interceptor

↓

HandlerMapping

↓

HandlerAdapter

↓

Controller

↓

Service

↓

Repository

↓

Hibernate

↓

Database

↓

Repository

↓

Service

↓

Controller

↓

Jackson

↓

DispatcherServlet

↓

Filter

↓

HTTP Response
```

---

# 26. Threading Model

One request

↓

One thread

Example

```
Request A

↓

Thread-10

Request B

↓

Thread-12

Request C

↓

Thread-18
```

Multiple requests execute concurrently.

Therefore

Services must be stateless.

---

# 27. Why Singleton Beans Work?

Question

```
One Controller

Many Threads

Safe?
```

Yes,

because Controllers and Services should not store mutable request-specific state.

Incorrect

```java
private String username;
```

Correct

```java
String username;
```

inside method.

---

# 28. Common Production Issues

---

## Slow API

Check

```
Controller

↓

Service

↓

Database

↓

External APIs
```

Most slow APIs are not caused by Spring itself.

---

## 404 Not Found

Possible reasons

- Wrong URL
- Missing @RequestMapping
- Wrong HTTP method

---

## 415 Unsupported Media Type

Usually

```
Content-Type

application/json
```

missing.

---

## 400 Bad Request

Reasons

- Validation failure
- JSON parsing error
- Missing required field

---

## 500 Internal Server Error

Usually

Unhandled exception.

Use

```
@ControllerAdvice
```

---

## Thread Pool Exhaustion

Symptoms

- Requests hanging
- High latency
- Timeout

Possible reasons

- Long-running requests
- Blocking database calls
- External service delays

---

# 29. Best Practices

✔ Controllers should only orchestrate requests.

✔ Keep business logic in Services.

✔ Never access the database directly from Controllers.

✔ Use DTOs instead of exposing entities.

✔ Use Global Exception Handling.

✔ Validate requests using `@Valid`.

✔ Keep Controllers stateless.

✔ Return meaningful HTTP status codes.

✔ Use ResponseEntity when custom status or headers are required.

---

# 30. Interview Summary

## Explain Spring MVC Request Flow

"When an HTTP request reaches a Spring Boot application, it is first accepted by the embedded Tomcat server, which assigns a worker thread and creates the servlet request and response objects. The request passes through servlet Filters before reaching the DispatcherServlet, the front controller of Spring MVC. DispatcherServlet consults HandlerMapping to locate the appropriate controller method and uses HandlerAdapter to invoke it. The controller delegates business logic to the service layer, which interacts with the repository layer and database. The returned object is serialized into JSON by Jackson through an HttpMessageConverter, and DispatcherServlet sends the HTTP response back through Tomcat to the client."

---

## What is DispatcherServlet?

"DispatcherServlet is the central front controller in Spring MVC. It receives every incoming request, delegates it to the correct controller, coordinates handler mappings, exception handling, view resolution or message conversion, and prepares the final HTTP response."

---

## Filter vs Interceptor

- **Filter** is part of the Servlet specification and executes before the request enters Spring MVC.
- **Interceptor** is a Spring MVC feature that executes around controller invocation and is ideal for logging, authorization, and request timing.

---

# Key Takeaways

- Every HTTP request flows through **Tomcat → Filters → DispatcherServlet → Controller → Service → Repository → Database**.
- **DispatcherServlet** is the heart of Spring MVC and coordinates request processing.
- **HandlerMapping** identifies the correct controller method, while **HandlerAdapter** invokes it.
- **Jackson** and **HttpMessageConverter** handle automatic JSON serialization and deserialization.
- **Filters** operate at the servlet level, whereas **Interceptors** operate within the Spring MVC framework.
- Controllers should remain thin, delegating business logic to Services and persistence to Repositories.
- Spring MVC uses a **thread-per-request** model, making stateless singleton Beans safe and scalable.
- Understanding the complete request lifecycle is essential for designing performant REST APIs and answering senior Spring Boot interview questions.
