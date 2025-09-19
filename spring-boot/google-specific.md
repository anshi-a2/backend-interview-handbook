# Google Spring Boot Interview Questions and Answers (SDE-2)

This document covers Spring Boot interview questions commonly asked at Google for backend engineering roles (SDE-2).  
Focus areas: **high-scale distributed systems, consistency, cloud-native apps (GCP), and performance optimization**.

---

## 1. How would you design a Spring Boot service to handle billions of requests per day?
**Answer:**
- Use **Spring WebFlux** (reactive, non-blocking).
- Deploy on **Google Kubernetes Engine (GKE)**.
- Scale horizontally with auto-scaling pods.
- Cache hot data in **Google Cloud Memorystore (Redis)**.
- Use **Pub/Sub** for async event processing.

---

## 2. How do you implement global consistency in a distributed Spring Boot system?
**Answer:**
- Use **Google Spanner** for strong consistency across regions.
- For eventual consistency, rely on **Pub/Sub** event-driven updates.
- Apply **Idempotent consumers** to handle retries.
- Use **versioning / optimistic locking** in JPA entities.

---

## 3. How do you optimize a Spring Boot API running at Google-scale?
**Answer:**
- Use **gRPC** instead of REST for inter-service communication.
- Apply **CDN caching** via Cloud CDN.
- Profile performance using **Java Flight Recorder** + Actuator metrics.
- Reduce GC pauses with G1GC or ZGC.

---

## 4. How would you design a recommendation service in Spring Boot?
**Answer:**
- REST/gRPC API with `GET /recommendations/{userId}`.
- ML model hosted in **TensorFlow Serving**.
- Call from Spring Boot service using gRPC.
- Cache top-N results in Redis for popular users.
- Retrain models using **Dataflow/BigQuery**.

---

## 5. How do you handle eventual consistency in microservices?
**Answer:**
- Use **event-driven architecture** with GCP Pub/Sub.
- Services publish domain events asynchronously.
- Retry failures with **Dead Letter Topics**.
- Apply **saga pattern** for distributed transactions.

---

## 6. How do you secure Spring Boot microservices on GCP?
**Answer:**
- Authenticate using **Google Identity Platform / OAuth2.0**.
- Use **mTLS** for internal communication.
- Secrets management with **Google Secret Manager**.
- IAM roles for fine-grained access control.

---

## 7. How would you integrate a Spring Boot app with BigQuery?
**Answer:**
- Use `spring-cloud-gcp-starter-bigquery`.
- Example:
  ```java
  @Autowired
  private BigQueryTemplate bigQueryTemplate;

  public void runQuery() {
      TableResult result = bigQueryTemplate.query("SELECT * FROM users LIMIT 10");
  }
