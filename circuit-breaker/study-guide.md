
# 📘 Circuit Breaker — Complete Study Guide (Java & Spring Boot Focus — Deep Dive)

---

## Table of Contents
1. Introduction
2. Problem statement: cascading failures
3. Circuit Breaker pattern (states & lifecycle)
4. Key components & related resilience patterns
5. Java libraries & Spring support
6. Quick start with Resilience4j (Spring Boot)
7. Deep-dive: configuration & internals (sliding windows, failure rate)
8. Combining Circuit Breaker with Retry, Bulkhead, RateLimiter, TimeLimiter
   - Patterns and interaction order
   - Practical code examples (annotations and functional)
9. Reactive (WebFlux) examples with Circuit Breaker
10. Async examples with CompletableFuture and TimeLimiter
11. Fallback strategies and designing graceful degradation
12. Monitoring, metrics, and alerting (Micrometer + Prometheus + Grafana)
13. Testing strategies (unit, integration, chaos testing)
14. Deployment & operational considerations (multi-instance, gateway)
15. Troubleshooting common issues
16. Interview Q&A (detailed answers)
17. Sample project layout & checklist
18. References & further reading

---

## 1. Introduction
A circuit breaker is a resilience pattern that detects failures and encapsulates the logic of preventing an application from performing an operation that's likely to fail. The circuit breaker improves system stability by failing fast and allowing recovery mechanisms to run instead of waiting for repeated timeouts or resource exhaustion.

---

## 2. Problem statement: cascading failures
Consider a microservice architecture where Service A calls Service B, which calls Service C. If Service C becomes slow or fails, requests back up and thread pools exhaust across Service B and A. This leads to cascading failures across the system. A circuit breaker stops repeated calls to a failing dependency and returns a fallback response quickly, protecting resources and enabling faster recovery.

---

## 3. Circuit Breaker pattern (states & lifecycle)

- **Closed**: All requests allowed through. Failure counts tracked in sliding window.
- **Open**: Requests fail fast (exceptions immediately). After `waitDurationInOpenState`, the breaker transitions to Half-Open.
- **Half-Open**: A limited number of trial requests allowed. If they succeed, breaker transitions to Closed; if they fail, back to Open.

### Sliding window types
- **Count-based**: Window size measured in number of calls (e.g., last 100 calls). Failure rate calculated from these calls.
- **Time-based**: Window measured in time duration (e.g., last 60 seconds).

### Failure conditions
- `failureRateThreshold` (percentage) triggers Open when exceeded.
- `minimumNumberOfCalls` defines how many calls must be present before evaluating thresholds.

---

## 4. Key components & related resilience patterns
- **Circuit Breaker**: Fails fast, prevents cascading failure.
- **Retry**: Reattempts failed calls (good for transient issues).
- **Bulkhead**: Isolates resources (thread pools/semaphores) to prevent resource exhaustion.
- **RateLimiter**: Controls call rate to downstream services.
- **TimeLimiter**: Enforces a timeout on operations (often used with async calls).
- **Fallback**: Alternate behavior when main call fails (default response, cached value, queue for later processing).

---

## 5. Java libraries & Spring support
- **Resilience4j** — Recommended: modern, modular, lightweight; integrates with Spring Boot and Micrometer.
- **Spring Cloud Circuit Breaker** — Abstraction that supports Resilience4j, Sentinel, Hystrix.
- **Hystrix** — Legacy from Netflix (maintenance mode).

We will use **Resilience4j** examples (annotation and functional style) and show Spring Boot integration, reactive patterns, and metrics.

---

## 6. Quick start with Resilience4j (Spring Boot)

### Maven dependencies
```xml
<dependency>
  <groupId>io.github.resilience4j</groupId>
  <artifactId>resilience4j-spring-boot3</artifactId>
  <version>2.1.0</version>
</dependency>
<dependency>
  <groupId>io.github.resilience4j</groupId>
  <artifactId>resilience4j-micrometer</artifactId>
  <version>2.1.0</version>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
> Use Spring Boot starter versions to align compatibility (adjust versions for your Spring Boot release).

### Basic configuration (application.yml)
```yaml
resilience4j:
  circuitbreaker:
    instances:
      inventoryService:
        register-health-indicator: true
        sliding-window-type: COUNT_BASED
        sliding-window-size: 20
        minimum-number-of-calls: 5
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        permitted-number-of-calls-in-half-open-state: 3
```

### Simple annotated usage (synchronous call)
```java
@Service
public class InventoryClient {

    private final RestTemplate restTemplate;

    public InventoryClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @CircuitBreaker(name = "inventoryService", fallbackMethod = "inventoryFallback")
    public String checkStock(String productId) {
        return restTemplate.getForObject("http://inventory-service/api/stock/" + productId, String.class);
    }

    public String inventoryFallback(String productId, Throwable t) {
        // fallback signature: same args + Throwable
        return "fallback-stock"; // or null / cached value / queue
    }
}
```

---

## 7. Deep-dive: configuration & internals

### Important properties (Resilience4j)
- `slidingWindowSize`: number of calls in window (count-based) or duration (time-based).
- `minimumNumberOfCalls`: minimum calls before calculating failure rate.
- `failureRateThreshold`: percent failures to open the circuit.
- `waitDurationInOpenState`: how long to stay open before trying half-open.
- `permittedNumberOfCallsInHalfOpenState`: how many trial calls allowed.
- `automaticTransitionFromOpenToHalfOpenEnabled`: whether to automatically transition after wait duration.

### Events & listeners
Resilience4j emits events (`CircuitBreakerOnStateTransitionEvent`, `CircuitBreakerOnErrorEvent`, etc.). You can register event listeners for monitoring or alerting:

```java
CircuitBreaker cb = CircuitBreaker.ofDefaults("inventoryService");
cb.getEventPublisher().onStateTransition(event -> log.info("State: {}", event.getStateTransition()));
```

### Manual creation and registry
```java
CircuitBreakerConfig config = CircuitBreakerConfig.custom()
    .failureRateThreshold(50)
    .slidingWindow(20, 20, SlidingWindowType.COUNT_BASED)
    .waitDurationInOpenState(Duration.ofSeconds(10))
    .build();

CircuitBreakerRegistry registry = CircuitBreakerRegistry.of(config);
CircuitBreaker cb = registry.circuitBreaker("inventoryService");
```

### Recording failure vs success
By default, any exception is recorded as failure. You can specify `recordExceptions` and `ignoreExceptions` in config to tune behavior.

```java
CircuitBreakerConfig.custom()
    .recordExceptions(IOException.class, TimeoutException.class)
    .ignoreExceptions(BadRequestException.class)
    ...
```

---

## 8. Combining Circuit Breaker with Retry, Bulkhead, RateLimiter, TimeLimiter

### 8.1 Principles & Order of decorators
The order in which you apply resilience patterns matters. Common recommended order (outer to inner):
```
RateLimiter -> Bulkhead -> CircuitBreaker -> Retry -> TimeLimiter -> Actual call
```
Why? Example reasoning:
- **RateLimiter** rejects excessive traffic early (cheap).
- **Bulkhead** isolates calls to limited resources.
- **CircuitBreaker** stops calling if dependency failing.
- **Retry** attempts transient recovery (but if used after CircuitBreaker, retries won't run when circuit open).
- **TimeLimiter** enforces call timeouts for async calls.

However, there is no one-size-fits-all; adapt to your requirements. Another common composition is:
```
TimeLimiter + CircuitBreaker + Retry
```

### 8.2 Annotation example (Resilience4j + Spring Boot)
```java
@Service
public class PaymentClient {

    @RateLimiter(name = "paymentService")
    @Bulkhead(name = "paymentService", type = Bulkhead.Type.SEMAPHORE)
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentRetry", fallbackMethod = "paymentFallback")
    public String chargeCard(String cardId, double amount) {
        // synchronous call to external payment gateway
    }

    public String paymentFallback(String cardId, double amount, Throwable t) {
        return "Payment gateway unavailable. Please try again later.";
    }
}
```
Notes:
- `@Bulkhead(type = THREADPOOL)` requires configuration of thread-pool bulkhead and executor.
- `@TimeLimiter` is used on methods that return `CompletableFuture` or `Future` (async).

### 8.3 Functional style (decorators) — advanced control
Functional style gives fine-grained control and is useful for non-Spring contexts or for complex flows:
```java
CircuitBreaker cb = registry.circuitBreaker("paymentService");
Retry retry = Retry.ofDefaults("paymentService");
Supplier<String> decoratedSupplier = Decorators.ofSupplier(() -> callPaymentGateway())
    .withRetry(retry)
    .withCircuitBreaker(cb)
    .decorate();

Try.ofSupplier(decoratedSupplier).get(); // from Vavr Try or CompletableFuture
```
Or using Resilience4j's `Decorators` helper:
```java
Supplier<CompletionStage<String>> supplier = () -> CompletableFuture.supplyAsync(() -> callPaymentGateway());

Supplier<CompletionStage<String>> decorated = Decorators.ofSupplier(supplier)
    .withCircuitBreaker(cb)
    .withRetry(retry)
    .withFallback(throwable -> CompletableFuture.completedFuture("fallback"))
    .decorate();
```

### 8.4 Bulkhead types: Semaphore vs ThreadPool
- **Semaphore**: simple limit on concurrent calls within same thread; best for sync calls.
- **ThreadPool**: isolates calls into a dedicated thread pool; useful for long-blocking calls to protect main thread pool.

ThreadPool Bulkhead configuration example (application.yml):
```yaml
resilience4j:
  bulkhead:
    instances:
      paymentService:
        maxThreadPoolSize: 10
        coreThreadPoolSize: 5
        queueCapacity: 50
```

And annotation:
```java
@Bulkhead(name = "paymentService", type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<String> chargeCardAsync(...) { ... }
```

### 8.5 TimeLimiter example (for async calls)
```java
@TimeLimiter(name = "inventoryService")
@CircuitBreaker(name = "inventoryService", fallbackMethod = "inventoryFallback")
public CompletableFuture<String> checkStockAsync(String productId) {
    return CompletableFuture.supplyAsync(() -> restTemplate.getForObject(...));
}
```

### 8.6 Retry configuration notes
Retries should be used carefully — they increase load. Configure backoff strategy:
```yaml
resilience4j:
  retry:
    instances:
      paymentRetry:
        maxAttempts: 3
        waitDuration: 500ms
        exponentialBackoffMultiplier: 2
```
Resilience4j `RetryConfig` supports `intervalFunction` for exponential backoff and jitter.

---

## 9. Reactive (WebFlux) examples with Circuit Breaker

Resilience4j provides `reactor` integration via `reactor-resilience4j` (operators like `transform` for Reactor `Mono`/`Flux`). Using Spring Cloud Circuit Breaker Reactor or `resilience4j-reactor` you can apply circuit breaker to reactive pipelines.

### Reactor example with circuit breaker operator
```java
@Service
public class ProductService {

    private final WebClient webClient;

    public ProductService(WebClient webClient) { this.webClient = webClient; }

    public Mono<Product> getProduct(String id) {
        return webClient.get()
            .uri("http://product-service/api/products/{id}", id)
            .retrieve()
            .bodyToMono(Product.class)
            .transformDeferred(CircuitBreakerOperator.of(circuitBreaker)); // resilience4j-reactor
    }
}
```
Where `circuitBreaker` is `io.github.resilience4j.circuitbreaker.CircuitBreaker` instance created from registry.

### Using Spring Cloud Circuit Breaker with Reactor
Spring Cloud Circuit Breaker provides `ReactiveCircuitBreakerFactory` that can wrap Reactor types and integrate with Resilience4j under the hood.

---

## 10. Async examples with CompletableFuture and TimeLimiter

Combining `CompletableFuture`, `TimeLimiter`, and Circuit Breaker for non-blocking timeouts:

```java
@Service
public class InventoryAsyncClient {

    private final ExecutorService executor = Executors.newCachedThreadPool();

    @TimeLimiter(name = "inventoryService")
    @CircuitBreaker(name = "inventoryService", fallbackMethod = "fallback")
    public CompletableFuture<String> checkStock(String productId) {
        return CompletableFuture.supplyAsync(() -> {
            // blocking HTTP call or DB call
            return restTemplate.getForObject(...);
        }, executor);
    }

    public CompletableFuture<String> fallback(String productId, Throwable t) {
        return CompletableFuture.completedFuture("fallback-stock");
    }
}
```

Note: When using `@TimeLimiter`, method must return `CompletionStage`/`CompletableFuture` to allow Resilience4j to cancel/timeout the future.

---

## 11. Fallback strategies and designing graceful degradation

Fallbacks should be designed carefully:
- Return cached data (short-term stale response).
- Queue the request for later processing (eventually consistent).
- Return a default response or prompt user to retry later.
- Use partial success: return what is available and mark missing parts as degraded.

Avoid returning sensitive error details. Ensure fallback paths are themselves resilient and don't call the failing dependency again causing loops.

---

## 12. Monitoring, metrics, and alerting (Micrometer + Prometheus + Grafana)

Resilience4j provides `resilience4j-micrometer` to expose metrics. With Micrometer Prometheus registry you can scrape metrics into Prometheus and visualize in Grafana.

### Key metrics to monitor
- `resilience4j_circuitbreaker_state{name="inventoryService"}` → 0=closed,1=open,2=half_open
- `resilience4j_circuitbreaker_calls{kind="successful|failed|not_permitted"}` → counts
- `resilience4j_circuitbreaker_buffered_calls` → number of calls in sliding window
- `resilience4j_circuitbreaker_failure_rate` → current failure rate %

### Example Prometheus queries (for Grafana panels)
- Failure rate over last 5m:
```
avg_over_time(resilience4j_circuitbreaker_failure_rate{instance="inventoryService"}[5m])
```
- Open circuit count:
```
resilience4j_circuitbreaker_state{instance="inventoryService"} == 1
```
- Not permitted (fast-failed) calls rate:
```
rate(resilience4j_circuitbreaker_calls_total{kind="NOT_PERMITTED",instance="inventoryService"}[5m])
```

### Health indicators & actuator
Enable health indicators for circuit breakers to surface in Spring Actuator `/actuator/health`.

---

## 13. Testing strategies (unit, integration, chaos testing)

### Unit tests
- Mock external dependencies, assert fallback called when exception thrown.
- Use `CircuitBreakerRegistry` with custom config for deterministic behavior.
- Example with Mockito and JUnit: verify method called fallback on exception.

### Integration tests
- Start a test server for downstream service that can simulate slow responses and failures (WireMock).
- Assert circuit breaker transitions to OPEN when threshold reached.

### Chaos testing
- Use tools to inject latency/failure (Chaostools, Gremlin) and validate system degrades gracefully.
- Test combined patterns (retry + circuit breaker) to ensure no thundering herd effect on recovery.

---

## 14. Deployment & operational considerations (multi-instance, gateway)

### Gateway-level circuit breakers
- Apply breakers in API Gateway to protect downstream services centrally.
- Prefer service-level breakers for fine-grained behavior.

### Multi-instance considerations for subscriptions and state
- Circuit breaker state in Resilience4j is in-memory per JVM by default. For multi-instance global state, you need external coordination (rarely necessary). Evaluate whether per-instance breaker suffices (commonly acceptable because failures often local to network partition or instance).

### Backpressure on recovery
- When circuit transitions to HALF_OPEN, trial calls may hit the downstream service; ensure the service can handle gradual recovery (use small permitted number of calls). Consider dynamic adjustment or ramping strategy.

---

## 15. Troubleshooting common issues

- **Circuit keeps opening immediately**: Check `minimumNumberOfCalls` too low; exceptions being recorded include expected client errors — use `ignoreExceptions` to avoid counting 4xx as failures.
- **Fallback not invoked**: Ensure fallback method signature matches (same args + Throwable) and method is visible (public) and in same bean class.
- **Retries causing overload**: Retries increase load; ensure backoff and bounds, and place retries behind circuit breaker if appropriate.
- **ThreadPool bulkhead saturates**: Tune core/max size and queue capacity; monitor RejectedExecutionException or bulkhead permitted count metrics.
- **Metrics missing**: Ensure `resilience4j-micrometer` and micrometer registry properly configured; actuator endpoints exposed.

---

## 16. Interview Q&A (detailed answers)

**Q1: Explain circuit breaker pattern and why it is useful.**  
A: Circuit breaker watches calls to a remote/expensive operation. When failures exceed threshold, it opens the circuit preventing further calls, failing fast and enabling system recovery. Useful to avoid resource exhaustion and cascading failures.

**Q2: How do Circuit Breaker and Retry interact?**  
A: Retry reattempts transient failures; circuit breaker stops calls to a service that is failing. If Retry is inside the Circuit Breaker, the circuit counts the outcome after retries; if Retry is outside, retries happen even when the circuit is open (not ideal). Usually prefer CircuitBreaker outer and Retry inner.

**Q3: Difference between Semaphore and ThreadPool Bulkhead.**  
A: Semaphore bulkhead limits concurrent calls in the calling thread (suitable for light, quick calls). ThreadPool bulkhead uses a dedicated thread pool isolating blocking operations and protecting caller threads.

**Q4: How to handle 4xx vs 5xx errors?**  
A: Treat client errors (4xx) as non-failures for circuit breaker (ignoreExceptions) because retries won't fix client request; server errors (5xx) should count as failures.

**Q5: How to monitor when to change breaker thresholds?**  
A: Use historical metrics (failure rate, latency distribution), SLOs, and alerts. Adjust thresholds based on observed normal and degraded behavior; run load tests to validate.

---

## 17. Sample project layout & checklist

```
src/main/java/com/example/resilience
  ├─ config/                 # Resilience4j beans, registry config
  ├─ client/                 # HTTP clients (RestTemplate/WebClient wrappers)
  ├─ service/                # Business services using resilience patterns
  ├─ fallback/               # Fallback implementations
  ├─ metrics/                # Metrics and listeners
  └─ App.java
src/main/resources/
  └─ application.yml
```

Checklist before production deployment:
- ✅ Fallbacks in place for user-facing flows
- ✅ Backoff + jitter for retries
- ✅ Circuit breaker metrics hooked to Prometheus
- ✅ Health indicators enabled
- ✅ Bulkhead configured for blocking calls
- ✅ Timeouts for all external calls
- ✅ Chaos tests for critical flows

---

## 18. References & further reading
- Resilience4j official docs: https://resilience4j.readme.io/
- Michael Nygard — *Release It!* (resilience patterns book)
- Spring Cloud Circuit Breaker docs
- Micrometer & Prometheus docs

---

# Appendix — Copy-ready code snippets

## 1) Resilience4j annotated combo (Retry + CircuitBreaker + Bulkhead + RateLimiter)
```java
@Service
public class PaymentService {

    @RateLimiter(name = "paymentService", fallbackMethod = "paymentFallback")
    @Bulkhead(name = "paymentService", type = Bulkhead.Type.SEMAPHORE, fallbackMethod = "paymentFallback")
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService", fallbackMethod = "paymentFallback")
    public String charge(String userId, double amount) {
        // call external payment gateway (blocking)
        return restTemplate.postForObject("http://payment-gateway/charge", request, String.class);
    }

    public String paymentFallback(String userId, double amount, Throwable t) {
        // fallback logic: queue the payment, return pending response
        return "payment-queued";
    }
}
```

## 2) ThreadPool Bulkhead configuration (YAML)
```yaml
resilience4j:
  bulkhead:
    instances:
      paymentService:
        maxThreadPoolSize: 10
        coreThreadPoolSize: 5
        queueCapacity: 100
```

## 3) Registering event listener to log circuit transition
```java
@Configuration
public class ResilienceConfig {

    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry() {
        CircuitBreakerRegistry registry = CircuitBreakerRegistry.ofDefaults();
        registry.getAllCircuitBreakers().forEach(cb -> cb.getEventPublisher()
            .onStateTransition(event -> log.info("Circuit {} changed: {}", cb.getName(), event.getStateTransition())));
        return registry;
    }
}
```

## 4) Functional decoration with Retry + CircuitBreaker
```java
CircuitBreaker cb = registry.circuitBreaker("paymentService");
Retry retry = Retry.ofDefaults("paymentService");

Supplier<String> supplier = () -> callPaymentGateway();
Supplier<String> decorated = Decorators.ofSupplier(supplier)
    .withRetry(retry)
    .withCircuitBreaker(cb)
    .withFallback(throwable -> "fallback")
    .decorate();

String result = decorated.get();
```

---


