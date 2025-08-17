
# Circuit Breaker with Resilience4j – Interview Q&A (SDE-2 Level)

---

## 🔹 1. What is a Circuit Breaker and why do we need it?
**Answer:**  
A circuit breaker is a **fault tolerance pattern** that prevents cascading failures in distributed systems.  
- It monitors calls to a downstream service.  
- If failures exceed a threshold, it “opens” and stops sending requests, failing fast.  
- After a cool-down, it moves to “half-open” to test recovery.  
- Protects the system from **latency spikes, timeouts, and outages**.  

---

## 🔹 2. What are the states of a Circuit Breaker in Resilience4j?
**Answer:**  
- **Closed** → Normal state, all calls go through.  
- **Open** → All calls fail immediately (breaker tripped).  
- **Half-Open** → Limited calls allowed; if successful, goes back to Closed.  

---

## 🔹 3. Difference between Retry and Circuit Breaker? Can we combine them?
**Answer:**  
- **Retry:** Attempts failed calls again (good for transient failures).  
- **Circuit Breaker:** Stops calls after consistent failures (good for persistent issues).  
👉 They can be combined → Retry first, and if failure continues, Circuit Breaker opens.  

---

## 🔹 4. Difference between Hystrix and Resilience4j?
**Answer:**  
- **Hystrix** (Netflix OSS) → Deprecated, heavyweight, based on RxJava.  
- **Resilience4j** → Lightweight, modular, Java 8 functional style, integrates with Spring Boot easily.  

---

## 🔹 5. What metrics does Resilience4j use to trip the Circuit Breaker?
**Answer:**  
- **Failure rate threshold** (% of failed calls).  
- **Slow call rate threshold** (% of calls slower than a threshold).  
- **Sliding window size** (time-based or count-based).  
- **Minimum number of calls** before evaluating.  

---

## 🔹 6. What’s the difference between Count-based and Time-based sliding windows?
**Answer:**  
- **Count-based:** Calculates failure rate on last N calls.  
- **Time-based:** Calculates failure rate on calls made in the last X seconds.  

---

## 🔹 7. How do you configure Circuit Breaker in Spring Boot?
**Answer:**  
- Use `@CircuitBreaker(name="serviceA", fallbackMethod="fallback")` on methods.  
- Configure in `application.yml`.  

```yaml
resilience4j:
  circuitbreaker:
    instances:
      serviceA:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 5s
        permittedNumberOfCallsInHalfOpenState: 3
```

---

## 🔹 8. How do you implement fallback in Resilience4j?
**Answer:**  
By defining a method with the same signature plus an `Exception` parameter.  

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "fallbackPayment")
public String placeOrder() {
    return restTemplate.getForObject("http://payment-service/pay", String.class);
}

public String fallbackPayment(Exception ex) {
    return "Payment service is unavailable. Please try again later!";
}
```

---

## 🔹 9. How to integrate Circuit Breaker with RestTemplate / WebClient / Feign?
**Answer:**  
- With **RestTemplate** → Annotate service methods with `@CircuitBreaker`.  
- With **WebClient** → Use `resilience4j-reactor` integration.  
- With **Feign** → Use Spring Cloud OpenFeign with Resilience4j.  

---

## 🔹 10. What is the role of `@RateLimiter`, `@Bulkhead`, and `@Retry` in Resilience4j?
**Answer:**  
- **@RateLimiter:** Controls request rate (throttling).  
- **@Bulkhead:** Limits concurrent calls (like thread isolation).  
- **@Retry:** Retries failed calls before marking them as failure.  

---

## 🔹 11. How do you configure multiple Circuit Breakers for different services?
**Answer:**  
Define multiple instances in `application.yml`:  

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        failureRateThreshold: 50
      inventoryService:
        failureRateThreshold: 30
```

---

## 🔹 12. Scenario: Payment service is intermittently failing. How would you design fault tolerance?
**Answer:**  
- Use **@Retry** with exponential backoff for transient failures.  
- Protect with **Circuit Breaker** to stop overloading if failures persist.  
- Add a **fallback** (e.g., cached response, queue request).  
- Ensure **idempotency** to avoid duplicate payments.  

---

## 🔹 13. What happens if the fallback itself fails?
**Answer:**  
- If fallback fails, the original exception is propagated.  
- Best practice → Use lightweight, reliable fallbacks (cached response, default message).  

---

## 🔹 14. Can Circuit Breaker cause data inconsistency?
**Answer:**  
Yes, if retries cause duplicate actions (like payment).  
👉 Solution: Design **idempotent APIs** (e.g., order IDs prevent duplicate charges).  

---

## 🔹 15. When NOT to use a Circuit Breaker?
**Answer:**  
- Batch jobs (failures don’t cascade).  
- Services where **availability > consistency** (you prefer retries).  
- Low-latency in-memory calls (breaker overhead isn’t worth it).  




