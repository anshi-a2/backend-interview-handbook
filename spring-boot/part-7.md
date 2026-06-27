# 07_Spring_Boot_Microservices_Distributed_Systems.md

> **Goal**
>
> Learn how Spring Boot applications are designed and deployed in a **microservices architecture**.
>
> This chapter covers **service communication, API Gateway, Service Discovery, Configuration Management, Circuit Breakers, Distributed Transactions, Kafka Integration, Resilience Patterns, and Production Best Practices**.
>
> These concepts are commonly discussed in **SDE-2/Senior Backend interviews** at companies like Amazon, Walmart, Oracle, Blink Health, Uber, and other product companies.

---

# Table of Contents

1. Monolith vs Microservices
2. Microservices Architecture
3. Service Communication
4. REST vs gRPC
5. Feign Client
6. WebClient
7. API Gateway
8. Service Discovery
9. Centralized Configuration
10. Circuit Breaker
11. Retry Pattern
12. Timeout Management
13. Bulkhead Pattern
14. Rate Limiting
15. Distributed Transactions
16. Saga Pattern
17. Idempotency
18. Event-Driven Architecture
19. Kafka Integration
20. Observability
21. Common Production Issues
22. Best Practices
23. Interview Summary

---

# 1. Why Microservices?

Traditional Monolith

```
                Monolith

    ┌───────────────────────────┐
    │                           │
    │  User                     │
    │  Order                    │
    │  Payment                  │
    │  Inventory                │
    │  Notification             │
    │                           │
    └───────────────────────────┘
```

Problems

- Large codebase
- Slow deployments
- Difficult scaling
- One failure can impact the whole application
- Technology lock-in

---

Microservices

```
           API Gateway
                 │
   ┌─────────────┼─────────────┐
   │             │             │
 User        Order        Payment
 Service     Service      Service
   │             │             │
 Inventory   Notification   Kafka
 Service      Service
```

Each service

- Has its own codebase
- Can be deployed independently
- Can scale independently
- Owns its own data

---

# 2. Typical Spring Boot Microservices Architecture

```
                 Client
                    │
             Load Balancer
                    │
              API Gateway
                    │
 ┌──────────┬──────────┬──────────┐
 │          │          │          │
User     Order     Payment   Inventory
Service   Service   Service    Service
 │          │          │          │
MySQL    MySQL      MySQL     MySQL
 │          │
 Redis    Kafka
```

Supporting Infrastructure

```
Config Server

↓

Service Registry

↓

Prometheus

↓

Grafana

↓

ELK

↓

Jaeger/OpenTelemetry
```

---

# 3. Service Communication

Microservices communicate in two ways.

### Synchronous

```
Service A

↓

HTTP/gRPC

↓

Service B
```

Caller waits for response.

---

### Asynchronous

```
Service A

↓

Kafka

↓

Service B
```

No immediate response required.

---

# 4. REST vs gRPC

### REST

- Human-readable JSON
- Easy integration
- Browser friendly

```
HTTP

↓

JSON
```

---

### gRPC

- Binary protocol (Protocol Buffers)
- Faster
- Lower network usage
- Strongly typed contracts

```
HTTP/2

↓

Protocol Buffers
```

---

| REST | gRPC |
|-------|------|
| JSON | Binary |
| Slower | Faster |
| Easy debugging | More efficient |
| Public APIs | Internal service-to-service |

---

# 5. OpenFeign

Calling another service manually

```java
RestTemplate

↓

HTTP

↓

Response
```

OpenFeign simplifies this.

```java
@FeignClient(name = "payment-service")
public interface PaymentClient {

}
```

Internally

```
Proxy

↓

HTTP Client

↓

Remote Service
```

Benefits

- Less boilerplate
- Declarative API
- Easy integration with load balancing and resilience libraries

---

# 6. WebClient

Modern non-blocking HTTP client.

Flow

```
Request

↓

Event Loop

↓

Remote Service

↓

Async Response
```

Advantages

- Reactive
- Better scalability for I/O-heavy workloads
- Supports streaming

Use for high-concurrency applications or when adopting Spring WebFlux.

---

# 7. API Gateway

Instead of exposing every service directly

```
Client

↓

Gateway

↓

Services
```

Responsibilities

- Authentication
- Authorization
- Routing
- SSL termination
- Rate limiting
- Logging
- Request aggregation

Popular gateways

- Spring Cloud Gateway
- Kong
- NGINX
- Envoy

---

# 8. Service Discovery

Problem

Service IPs change frequently.

Instead of hardcoding

```
10.0.1.25
```

use

```
payment-service
```

Flow

```
Service

↓

Register

↓

Service Registry

↓

Gateway Finds Service
```

Historically, tools like **Eureka** were popular.

In Kubernetes,

```
Kubernetes Service

↓

Built-in Service Discovery
```

is typically used instead.

---

# 9. Centralized Configuration

Instead of

```
application.properties

in every service
```

Use

```
Config Server

↓

Configuration Repository

↓

Services
```

Benefits

- Centralized configuration
- Easier updates
- Consistent environments

In Kubernetes, ConfigMaps and Secrets often replace Spring Cloud Config.

---

# 10. Circuit Breaker

Suppose

```
Order

↓

Payment
```

Payment service becomes slow.

Without protection

```
Order waits

↓

Threads blocked

↓

Application fails
```

Circuit Breaker

```
Failures

↓

Threshold reached

↓

Circuit Opens

↓

Fail Fast
```

Popular library

```
Resilience4j
```

---

# 11. Retry Pattern

Temporary failures happen.

```
Call

↓

Fail

↓

Retry

↓

Success
```

Retry only for

- Network issues
- Temporary outages
- Timeouts

Avoid retrying non-idempotent operations blindly.

---

# 12. Timeout Management

Never wait forever.

```
HTTP Call

↓

Timeout

↓

Fallback
```

Always configure

- Connection timeout
- Read timeout

---

# 13. Bulkhead Pattern

Prevent one slow dependency from consuming all resources.

```
Thread Pool A

↓

Payment

Thread Pool B

↓

Inventory
```

Failure in one service should not exhaust threads needed by others.

---

# 14. Rate Limiting

Protect APIs from abuse.

```
100 Requests

↓

Allowed

101st

↓

Rejected
```

Common algorithms

- Token Bucket
- Leaky Bucket
- Sliding Window

---

# 15. Distributed Transactions

Suppose

```
Order

↓

Payment

↓

Inventory
```

One service succeeds,

another fails.

Traditional database transactions do not span multiple independent services.

---

# 16. Saga Pattern

Preferred solution.

```
Create Order

↓

Reserve Inventory

↓

Charge Payment

↓

Send Notification
```

If one step fails

↓

Execute compensation

```
Refund Payment

↓

Release Inventory

↓

Cancel Order
```

Sagas can be

- Choreography-based (events)
- Orchestration-based (central coordinator)

---

# 17. Idempotency

Very important interview topic.

Problem

```
Client retries

↓

Payment API called twice
```

Duplicate payment.

Solution

```
Idempotency Key
```

Flow

```
Request

↓

Unique Key

↓

Already Processed?

↓

Return Previous Result
```

Useful for

- Payments
- Order creation
- External APIs

---

# 18. Event-Driven Architecture

Instead of direct calls

```
Order Service

↓

Kafka

↓

Inventory

↓

Notification

↓

Analytics
```

Advantages

- Loose coupling
- Scalability
- Better resilience
- Independent consumers

---

# 19. Kafka Integration

Typical Spring Boot flow

```
Producer

↓

Kafka Topic

↓

Consumer

↓

Business Logic
```

Producer

```java
KafkaTemplate
```

Consumer

```java
@KafkaListener
```

Production considerations

- Consumer groups
- Partitioning
- Ordering
- Offset management
- Dead Letter Topics (DLTs)

---

# 20. Observability

A production microservices system should include

### Metrics

```
Micrometer

↓

Prometheus

↓

Grafana
```

---

### Logs

```
Application

↓

Fluent Bit

↓

OpenSearch/Elasticsearch

↓

Kibana
```

---

### Tracing

```
Client

↓

Gateway

↓

Order

↓

Payment

↓

Inventory
```

Use

- OpenTelemetry
- Jaeger
- Zipkin

A Trace ID follows the request across services.

---

# 21. Common Production Issues

## Cascading Failure

One slow service

↓

All services become slow.

Solution

- Circuit Breaker
- Timeouts
- Bulkheads

---

## Duplicate Events

Cause

Producer retries.

Solution

- Idempotent consumers
- Deduplication
- Exactly-once where appropriate

---

## Message Lag

Symptoms

Kafka consumers behind.

Check

- Consumer lag
- Partitions
- Consumer throughput

---

## Configuration Drift

Different services running with different configurations.

Solution

- Centralized configuration
- Version-controlled configs
- Automated deployments

---

## Service Unavailable

Check

```
Gateway

↓

Service Discovery

↓

Health Check

↓

Pod Status

↓

Application Logs
```

---

# 22. Production Best Practices

✔ One database per service.

✔ Prefer asynchronous communication where appropriate.

✔ Keep APIs backward compatible.

✔ Implement retries with exponential backoff.

✔ Always configure timeouts.

✔ Use Circuit Breakers for remote calls.

✔ Use idempotency for payment and order APIs.

✔ Monitor every service with metrics, logs, and traces.

✔ Use correlation IDs to trace requests across services.

✔ Design services around business capabilities rather than technical layers.

---

# 23. Interview Summary

## Explain Microservices Architecture

"A microservices architecture decomposes an application into independently deployable services, each responsible for a specific business capability. Services communicate through synchronous protocols such as REST or gRPC, or asynchronously through messaging systems like Kafka. Infrastructure components such as an API Gateway, service discovery, centralized configuration, observability, and resilience mechanisms ensure scalability, fault tolerance, and maintainability."

---

## REST vs gRPC

"REST uses JSON over HTTP and is easy to integrate with external clients, making it suitable for public APIs. gRPC uses HTTP/2 and Protocol Buffers, offering lower latency, smaller payloads, and stronger contracts, making it ideal for internal service-to-service communication."

---

## What is a Circuit Breaker?

"A Circuit Breaker monitors failures when calling remote services. After a configurable threshold of failures, it opens the circuit and immediately rejects new requests, preventing thread exhaustion and cascading failures. After a cooldown period, it allows limited test requests to determine whether the dependency has recovered."

---

## Explain Saga Pattern

"The Saga Pattern manages distributed transactions across multiple microservices by breaking a business transaction into a sequence of local transactions. If a step fails, compensating transactions undo previously completed work, ensuring eventual consistency without using distributed database transactions."

---

## Why is Idempotency Important?

"Idempotency ensures that repeating the same request produces the same result without unintended side effects. It is especially important for payment processing, order creation, and distributed systems where retries may occur due to network failures."

---

# Key Takeaways

- Microservices improve scalability, independent deployments, and fault isolation compared to monoliths.
- Services communicate using **REST**, **gRPC**, or **event-driven messaging** depending on the use case.
- **API Gateways** centralize routing, security, and cross-cutting concerns.
- **Service Discovery** enables services to locate each other dynamically; Kubernetes provides this capability natively.
- **Circuit Breakers**, **Retries**, **Timeouts**, and **Bulkheads** are essential resilience patterns.
- **Saga Pattern** is the preferred approach for distributed transactions across multiple services.
- **Kafka** enables asynchronous communication and event-driven architectures.
- Production-ready microservices require comprehensive **metrics, logging, tracing, health checks, and monitoring** to operate reliably at scale.
