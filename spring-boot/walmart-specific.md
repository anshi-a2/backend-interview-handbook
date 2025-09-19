# Walmart Spring Boot Interview Questions and Answers (SDE-2)

This document covers Spring Boot interview questions commonly asked at Walmart Global Tech for SDE-2 backend roles.  
Focus areas: **large-scale data systems, performance optimization, real-time systems, and reliability**.

---

## 1. How would you design a Spring Boot service to handle millions of concurrent retail transactions?
**Answer:**
- Use **Spring Boot** with **reactive stack (WebFlux)** for async, non-blocking I/O.
- Horizontal scaling with Kubernetes.
- **Event-driven architecture** using Kafka for transaction events.
- Use **Redis** for session management and caching.
- Apply **CQRS pattern** for separating read/write workloads.

---

## 2. How do you ensure high performance in Spring Boot applications?
**Answer:**
- Use **connection pooling** (HikariCP).
- Optimize Hibernate with:
  - Batch fetching
  - 2nd level cache (EhCache/Redis)
- Use **async calls** with `@Async` for background tasks.
- Profile and monitor using **Actuator + Prometheus/Grafana**.

---

## 3. How would you design an inventory management microservice using Spring Boot?
**Answer:**
- REST endpoints for CRUD inventory operations.
- Use **Spring Data JPA** for database interaction.
- Event sourcing with Kafka to track inventory changes.
- Expose APIs like:
  - `POST /inventory/update`
  - `GET /inventory/{productId}`
- Scale reads with **read replicas** and caching.

---

## 4. How do you handle eventual consistency in Spring Boot microservices?
**Answer:**
- Use **event-driven messaging** with Kafka.
- Services publish/consume events asynchronously.
- Implement **compensating transactions** for failure recovery.
- Example:
  - Order placed → `order-service` emits event → `inventory-service` adjusts stock asynchronously.

---

## 5. How do you handle failures in a Spring Boot distributed system?
**Answer:**
- Use **Resilience4j**:
  - Circuit breaker
  - Bulkhead isolation
  - Timeouts + retries
- Store failed events in a **Dead Letter Queue (DLQ)** for replay.
- Walmart case: retry payment gateway failures with exponential backoff.

---

## 6. How do you secure APIs in a retail system?
**Answer:**
- JWT authentication + role-based access control.
- Use **OAuth2.0** for external partner APIs.
- API Gateway for rate limiting and throttling.
- Encrypt sensitive data (payment info) using **Vault/KMS**.

---

## 7. How do you optimize database performance in Spring Boot?
**Answer:**
- Use **read-write split** (primary for writes, replicas for reads).
- Partition large datasets (sharding).
- Index frequently queried fields.
- Use **Spring Data Redis** for hot data caching.
- Batch inserts with `saveAll()`.

---

## 8. How do you integrate a Spring Boot service with Kafka?
**Answer:**
- Add `spring-kafka` dependency.
- Example Consumer:
  ```java
  @KafkaListener(topics = "orders", groupId = "order-group")
  public void consume(String message) {
      log.info("Received: " + message);
  }
  ```
- Configure **producer retries, acknowledgements, and partition strategy**.
- Ensure **idempotency** in consumers.

---

## 9. How do you implement logging and monitoring?
**Answer:**
- Centralized logging with **ELK stack (Elasticsearch, Logstash, Kibana)**.
- Metrics with **Micrometer + Prometheus**.
- Trace requests with **Spring Cloud Sleuth**.
- Business KPIs (e.g., orders/sec, latency) reported via Actuator custom endpoints.

---

## 10. How would you design a pricing engine in Spring Boot?
**Answer:**
- **Rule-based engine** to calculate final price (discounts, offers, taxes).
- Use caching for frequently accessed products.
- Event-driven updates when promotions change.
- Expose `GET /price/{productId}`.
- Ensure low latency with precomputed values in Redis.

---

## 11. How do you handle schema changes in production databases?
**Answer:**
- Use **Liquibase/Flyway** for versioned migrations.
- Roll forward with new schema.
- Backward compatibility: keep old fields until all services migrate.
- Zero-downtime deployments with rolling updates.

---

## 12. How do you test Walmart-scale Spring Boot applications?
**Answer:**
- Unit testing with JUnit5 + Mockito.
- Integration testing with `@SpringBootTest`.
- Use **TestContainers** for DB/Kafka.
- Performance/load testing with JMeter/Gatling.
- Contract testing for microservices with Pact.

---

# ✅ Walmart Interview Tips:
- Expect **real-time retail transaction system design**.
- Strong focus on **Kafka, caching, and DB optimization**.
- Be ready to discuss **event-driven microservices**.
- Emphasize **resiliency, high availability, and zero-downtime deployments**.
