# Amazon Spring Boot Interview Questions and Answers (SDE-2)

This document covers Spring Boot interview questions commonly asked at Amazon for SDE-2 backend roles.  
Focus areas: **scalability, fault tolerance, distributed systems, microservices, and clean coding**.

---

## 1. How would you design a scalable microservice in Spring Boot for Amazon-scale traffic?
**Answer:**
- Use **Spring Boot** with embedded Tomcat/Netty for independent deployability.
- Break services by bounded context (e.g., Order Service, Payment Service).
- Use **Spring Cloud Netflix/Consul** for service discovery.
- Enable **load balancing** with Spring Cloud LoadBalancer.
- Deploy on **Kubernetes/ECS** with autoscaling.
- Ensure **idempotent APIs** for retries.

---

## 2. How does Spring Boot support distributed configuration in microservices?
**Answer:**
- Use **Spring Cloud Config Server** for centralized configuration.
- Store configs in Git/S3.
- Client services fetch configs via `/actuator/refresh`.
- Amazon use-case: rotate API keys and secrets without redeploying services.

---

## 3. Explain how you would implement request tracing in Spring Boot for distributed systems.
**Answer:**
- Use **Spring Cloud Sleuth** + **Zipkin/X-Ray**.
- Adds `traceId` and `spanId` to logs automatically.
- Every microservice call propagates trace headers.
- Helps debug latency issues across Amazon’s distributed services.

---

## 4. How would you design a fault-tolerant API with Spring Boot?
**Answer:**
- Use **Resilience4j** (preferred over Hystrix).
- Features: Retry, Circuit Breaker, Bulkhead.
- Example:
  ```java
  @CircuitBreaker(name = "inventoryService", fallbackMethod = "fallbackInventory")
  public Inventory checkInventory(String productId) {
      return restTemplate.getForObject(inventoryUrl + "/" + productId, Inventory.class);
  }
  ```

---

## 5. How do you secure Spring Boot microservices at Amazon scale?
**Answer:**
- Use **JWT-based authentication**.
- Integrate with **AWS Cognito / IAM roles**.
- Use API Gateway as a single entry point.
- Services validate JWT locally → no bottleneck on auth server.
- Encrypt secrets with AWS KMS.

---

## 6. How does Amazon expect you to handle database scaling with Spring Boot?
**Answer:**
- Use **Amazon Aurora/MySQL/Postgres** with **read replicas**.
- Configure datasource routing in Spring Boot:
  - Write → primary DB
  - Read → replicas
- Use **Spring Data JPA** with custom `@RoutingDataSource`.
- Cache frequently accessed data with **Redis/ElastiCache**.

---

## 7. How would you integrate a Spring Boot service with AWS services?
**Answer:**
- AWS SDK integration inside Spring Boot:
  - S3 → File storage
  - SQS → Message queue
  - DynamoDB → Key-value store
- Example: SQS Listener
  ```java
  @SqsListener("order-queue")
  public void processOrder(String message) {
      // process order event
  }
  ```

---

## 8. How do you manage transactions in distributed systems?
**Answer:**
- Avoid 2PC (not scalable).
- Use **Saga Pattern**:
  - Orchestration (central coordinator service).
  - Choreography (events via SQS/Kafka).
- In Spring Boot:
  - Publish domain events after transaction commit.
  - Handle compensating transactions for failures.

---

## 9. How do you ensure observability of Spring Boot services?
**Answer:**
- **Monitoring**: Spring Boot Actuator → `/metrics`, `/health`.
- **Tracing**: Sleuth + Zipkin/X-Ray.
- **Logging**: Centralized logging (CloudWatch/ELK).
- **Alerts**: Amazon CloudWatch alarms on latency/throughput.

---

## 10. How do you implement caching at Amazon scale?
**Answer:**
- Use **Redis/ElastiCache**.
- Annotate with `@Cacheable`, `@CacheEvict`.
- Example:
  ```java
  @Cacheable(value = "productCache", key = "#productId")
  public Product getProduct(String productId) {
      return productRepository.findById(productId).orElseThrow();
  }
  ```
- Eviction policy: LRU.
- Multi-region: Use **DynamoDB DAX** for high throughput caching.

---

## 11. How would you design a URL shortener in Spring Boot (Amazon-style design)?
**Answer:**
- **Requirements**: billions of requests, low latency.
- **Tech Stack**:
  - Spring Boot REST API
  - DynamoDB (NoSQL) for mapping
  - Redis for caching hot links
- **Flow**:
  - POST `/shorten` → generate hash + store in DB
  - GET `/{hash}` → lookup Redis → fallback to DB
- Ensure **unique ID generation** using Snowflake algorithm.

---

## 12. How would you test Spring Boot services for production readiness?
**Answer:**
- Unit Tests: JUnit + Mockito.
- Integration Tests: `@SpringBootTest` + TestContainers (DB, Kafka).
- Contract Testing: Pact (for microservices).
- Load Testing: JMeter/Gatling.

---

# ✅ Amazon Interview Tips:
- Expect **system design + Spring Boot coding**.
- Focus on **scalable, fault-tolerant architectures**.
- Be ready to **integrate with AWS services (S3, SQS, DynamoDB)**.
- Discuss trade-offs (SQL vs NoSQL, sync vs async, cache invalidation).
