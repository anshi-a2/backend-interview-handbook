# Microsoft Spring Boot Interview Questions and Answers (SDE-2)

This document covers Spring Boot interview questions commonly asked at Microsoft for backend engineering roles (SDE-2).  
Focus areas: **scalable APIs, distributed systems, cloud integration (Azure), and security**.

---

## 1. How would you design a Spring Boot API for Outlook Calendar integration?
**Answer:**
- Use **Spring Boot REST controllers** to expose endpoints like `/calendar/events`.
- Integrate with **Microsoft Graph API**.
- Use **Spring Security OAuth2 Client** for authentication.
- Store tokens securely in **Azure Key Vault**.
- Retry failed API calls with exponential backoff.

---

## 2. How do you integrate Spring Boot services with Azure?
**Answer:**
- Use **Spring Cloud Azure** starter dependencies.
- Examples:
  - **Azure Cosmos DB** for NoSQL.
  - **Azure Service Bus** for messaging.
  - **Azure Blob Storage** for file handling.
- Deploy services on **Azure Kubernetes Service (AKS)**.
- Use **Managed Identity** for secure resource access.

---

## 3. How would you handle multi-tenant architecture in Spring Boot?
**Answer:**
- Use **schema-based or discriminator-based multi-tenancy** with Hibernate.
- Implement a `TenantContext` to store tenant ID.
- Interceptor filters requests and assigns tenant dynamically.
- Example: `@Filter("tenantId")` for queries.
- Scale via Azure SQL elastic pools.

---

## 4. How do you ensure reliability in distributed Spring Boot services?
**Answer:**
- Apply **Resilience4j**:
  - Circuit breaker
  - Retry
  - Rate limiter
- Use **Azure Service Bus DLQ** for failed messages.
- Implement **idempotency keys** to avoid duplicate operations.
- Distributed tracing with **OpenTelemetry + Azure Monitor**.

---

## 5. How do you optimize performance of high-traffic APIs?
**Answer:**
- Use **WebFlux** for async/non-blocking APIs.
- Cache frequently accessed results in **Azure Redis Cache**.
- Use **pagination** for large queries.
- Profile with **Spring Boot Actuator + Application Insights**.

---

## 6. How do you secure microservices communication?
**Answer:**
- Use **mTLS** (mutual TLS) between services.
- JWT validation for API requests.
- Integrate with **Azure AD** for authentication/authorization.
- Rotate secrets using **Azure Key Vault**.

---

## 7. How would you design a file storage service in Spring Boot?
**Answer:**
- REST API endpoints:
  - `POST /files` → upload
  - `GET /files/{id}` → download
- Use **Azure Blob Storage SDK**.
- Metadata stored in SQL Server / Cosmos DB.
- Large file uploads: use **chunked uploads** with async processing.

---

## 8. How do you handle database transactions in microservices?
**Answer:**
- Prefer **event-driven saga pattern** over distributed transactions.
- Example:
  - `payment-service` reserves funds.
  - `order-service` places order.
  - If failure, publish compensation event.
- Use **Outbox pattern** with Kafka/Azure Service Bus.

---

## 9. How do you monitor Spring Boot applications in production?
**Answer:**
- Use **Spring Boot Actuator** for health checks.
- Collect metrics via **Micrometer + Application Insights**.
- Centralized logs with **Azure Log Analytics**.
- Distributed tracing with **OpenTelemetry**.

---

## 10. How would you design a notification system in Spring Boot?
**Answer:**
- Event-driven architecture with **Azure Event Hub**.
- Consumers for email, SMS, and push notifications.
- Template-based rendering for messages.
- Store preferences in DB for opt-in/out.
- Retry failed notifications with DLQ.

---

## 11. How do you implement role-based access control (RBAC)?
**Answer:**
- Use **Spring Security with Azure AD roles**.
- Map roles in JWT claims → Spring Security authorities.
- Example:
  ```java
  @PreAuthorize("hasRole('ADMIN')")
  public ResponseEntity<String> adminOnly() { ... }
