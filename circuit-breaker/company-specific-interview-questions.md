
# Company-Specific Circuit Breaker Interview Q&A (SDE-2)

---

## **1. Amazon (SDE-2)**
👉 *Amazon likes behavioral + system design + trade-off discussions.*  

**Q:** You’re designing the **Order Service** which calls **Payment Service**. The Payment Service sometimes takes too long. How would you design a resilient solution?  

**Answer:**  
- Use **Circuit Breaker** with **slow-call threshold** to fail fast.  
- Add **Retry with exponential backoff** for transient network issues.  
- If retries + breaker fail → push order to **SQS/Kafka queue** for async processing.  
- Implement **idempotency keys** (e.g., orderId) to avoid duplicate payments.  
- Add **metrics and alarms (CloudWatch)** to monitor breaker states.  

👉 **Key Amazon Angle:** Tie the solution to **customer obsession** (never double charge) + **operational excellence** (CloudWatch monitoring).  

---

## **2. Walmart (SDE-2, Global Tech)**  
👉 *They test practical coding with Spring Boot.*  

**Q:** Show me how you’d use Resilience4j Circuit Breaker for an Inventory Service call in Spring Boot.  

**Answer (Code):**  

```java
@Service
public class InventoryServiceClient {

    private final RestTemplate restTemplate;

    public InventoryServiceClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @CircuitBreaker(name = "inventoryService", fallbackMethod = "inventoryFallback")
    public String checkInventory(String productId) {
        return restTemplate.getForObject("http://inventory-service/check/" + productId, String.class);
    }

    public String inventoryFallback(String productId, Exception ex) {
        return "Inventory data unavailable. Please retry later.";
    }
}
```

**Config in `application.yml`:**

```yaml
resilience4j:
  circuitbreaker:
    instances:
      inventoryService:
        slidingWindowSize: 20
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 5
```

👉 **Key Walmart Angle:** Show **Spring Boot + YAML + fallback code** → they often ask for runnable snippets.  

---

## **3. Goldman Sachs (Backend SDE-2)**  
👉 *They focus on **low-latency** and **financial consistency***.  

**Q:** What happens if you use Circuit Breaker in a **payment transaction system**?  

**Answer:**  
- Circuit Breaker prevents overwhelming Payment Gateway during outages.  
- **Risk:** If retries are not idempotent → may cause **duplicate transactions**.  
- **Solution:**  
  - Design all payment APIs **idempotent** (unique transaction ID).  
  - Use **at-least-once semantics** with reconciliation service.  
  - Ensure **timeouts** are shorter than upstream SLA (e.g., 2s vs. 5s).  

👉 **Key Goldman Angle:** Show **financial safety, idempotency, low-latency trade-offs**.  

---

## **4. PayPal / Fintech Interviews**  
👉 *They check how you handle failure + customer impact.*  

**Q:** If your downstream service is failing and Circuit Breaker opens, what will the customer experience?  

**Answer:**  
- Customer request fails **immediately** (fast failure).  
- Return **graceful fallback** (e.g., “Transaction is being processed, please check later”).  
- Log + alert team.  
- Optionally, queue the request for **eventual consistency** (background retries).  

👉 **Key PayPal Angle:** Always emphasize **graceful degradation** → “never lose customer request, ensure traceability”.  

---

## **5. Microsoft / Azure Interviews**  
👉 *They check if you understand cloud-native integration.*  

**Q:** How would you use Circuit Breaker in a microservices system deployed on Kubernetes + Azure?  

**Answer:**  
- Use **Resilience4j** at app-level.  
- Combine with **Kubernetes HPA (Horizontal Pod Autoscaler)** for scaling under load.  
- Use **Azure Application Insights** for Circuit Breaker metrics.  
- Use **ConfigMaps** for dynamic breaker thresholds (instead of hardcoding).  

👉 **Key Microsoft Angle:** Show **cloud-native + monitoring integration**.  

---

## **6. Uber / Swiggy / Zomato (High Traffic Systems)**  
👉 *They love traffic-spike + SLA questions.*  

**Q:** If Order Service gets **10k RPS**, but Payment Service can handle only 2k RPS, how would you protect it?  

**Answer:**  
- Use **Circuit Breaker** to cut off failing calls.  
- Use **Bulkhead pattern** to restrict concurrent threads calling Payment Service.  
- Use **Rate Limiter** (`@RateLimiter`) to throttle requests to 2k RPS.  
- Queue overflow requests into **Kafka** for async processing.  

👉 **Key Uber Angle:** Mention **Bulkhead + RateLimiter + CircuitBreaker together**.  

---

## ✅ Quick Company-Specific Summary
- **Amazon** → Customer obsession + async retries + monitoring.  
- **Walmart** → Show code in Spring Boot (Resilience4j configs).  
- **Goldman Sachs** → Idempotency, financial consistency, low-latency.  
- **PayPal** → Graceful degradation + no data loss.  
- **Microsoft** → Cloud-native, monitoring, config management.  
- **Uber/Swiggy** → High RPS, Bulkhead + RateLimiter + Kafka + breaker combo.  





