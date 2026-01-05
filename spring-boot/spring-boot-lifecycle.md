# 🌱 Spring Boot Lifecycle – Complete In-Depth Guide

This document provides a **complete, interview-ready explanation of the Spring Boot lifecycle**, covering **startup, bean lifecycle, runtime, Actuator integration, and shutdown**. It is suitable for **SDE-2 / Senior Backend Engineer interviews**.

---

## 1. JVM Startup & `main()` Method

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**What happens:**

* JVM starts
* ClassLoader loads Spring Boot & application classes
* `SpringApplication.run()` triggers the entire lifecycle

---

## 2. SpringApplication Bootstrap Phase

Internally:

* Determines application type (SERVLET / REACTIVE / NONE)
* Loads `ApplicationContextInitializer`
* Loads `ApplicationListener`
* Sets up logging & banner

📌 **No beans are created yet**

---

## 3. Environment Preparation

Spring prepares the `Environment` object:

* `application.yml / application.properties`
* Profile-specific configs
* OS environment variables
* JVM system properties

**Order of precedence (high → low):**

1. Command-line args
2. JVM system properties
3. OS env vars
4. Profile configs
5. Default application config

---

## 4. ApplicationContext Creation

Spring Boot creates the IoC container based on app type:

| App Type   | Context                                              |
| ---------- | ---------------------------------------------------- |
| Spring MVC | `AnnotationConfigServletWebServerApplicationContext` |
| WebFlux    | `ReactiveWebServerApplicationContext`                |
| Non-Web    | `AnnotationConfigApplicationContext`                 |

---

## 5. Bean Definition Registration Phase

Spring scans and registers **bean metadata** (not objects):

* `@Component`, `@Service`, `@Repository`
* `@Configuration`, `@Bean`
* Auto-configuration classes

📌 **Only definitions are registered, objects are not created yet**

---

## 6. BeanFactoryPostProcessor Phase

Runs **before bean instantiation**.

Used to:

* Modify bean definitions
* Resolve placeholders
* Change scopes

📌 Runs once, affects metadata only

---

## 7. Bean Instantiation Phase

* Singleton beans created eagerly
* Prototype beans created on demand

---

## 8. Dependency Injection Phase

Spring injects dependencies using:

* Constructor injection (preferred)
* Setter injection
* Field injection

At this stage:

* Objects exist
* Dependencies are wired
* Beans are not fully initialized yet

---

## 9. Aware Interfaces Phase

If a bean implements:

* `ApplicationContextAware`
* `EnvironmentAware`
* `BeanNameAware`

Spring injects container-level metadata

---

## 10. BeanPostProcessor Phase (VERY IMPORTANT)

Runs **before and after initialization**.

Used for:

* AOP
* Transaction management
* Async execution
* Caching

📌 Proxies for `@Transactional`, `@Async`, `@Cacheable` are created here

---

## 11. Bean Initialization Phase

Execution order:

1. `@PostConstruct`
2. `InitializingBean.afterPropertiesSet()`
3. Custom `initMethod`

📌 Best place for startup logic

---

## 12. Embedded Server Startup

Spring Boot:

* Starts embedded Tomcat/Jetty/Netty
* Initializes DispatcherServlet
* Registers filters & interceptors

---

## 13. ApplicationReadyEvent

Triggered when:

* Context is fully initialized
* Server is started
* Endpoints are active

Used for:

* Cache warmup
* Kafka consumer startup
* Preloading data

---

## 14. Runtime Phase

During runtime Spring manages:

* Thread pools
* Transactions
* Exception handling
* Serialization/deserialization

Request flow:

```
Client → Filter → DispatcherServlet → Controller → Service → Repository → DB
```

---

## 15. Spring Boot Actuator in Lifecycle

### Where Actuator Fits

Actuator integrates across **startup, runtime, and shutdown**.

### Startup

* Auto-configured during auto-configuration phase
* Registers health, metrics, info beans

### Bean Lifecycle

* Hooks into `BeanPostProcessor`
* Instruments beans using Micrometer

### Runtime

* Exposes endpoints:

  * `/actuator/health`
  * `/actuator/metrics`
  * `/actuator/info`
* Monitors JVM, DB, thread pools

### Shutdown

* Updates readiness to `OUT_OF_SERVICE`
* Supports graceful shutdown (K8s, LB)

---

## 16. Graceful Shutdown Phase

Triggered by:

* SIGTERM
* `context.close()`

Shutdown order:

1. Stop accepting new requests
2. Complete in-flight requests
3. Execute destroy callbacks

Destroy callbacks:

* `@PreDestroy`
* `DisposableBean.destroy()`
* Custom `destroyMethod`

---

## 17. Complete Lifecycle Flow (Summary)

```
JVM Start
 ↓
SpringApplication.run()
 ↓
Environment Prepared
 ↓
ApplicationContext Created
 ↓
Bean Definitions Registered
 ↓
BeanFactoryPostProcessor
 ↓
Bean Instantiation
 ↓
Dependency Injection
 ↓
Aware Interfaces
 ↓
BeanPostProcessor (AOP, Tx)
 ↓
@PostConstruct
 ↓
Embedded Server Start
 ↓
ApplicationReadyEvent
 ↓
Application Running
 ↓
@PreDestroy
 ↓
Shutdown
```

---

## 18. Spring Boot vs Spring Framework (Lifecycle View)

| Aspect   | Spring Framework | Spring Boot           |
| -------- | ---------------- | --------------------- |
| Focus    | Bean lifecycle   | App + Bean lifecycle  |
| Startup  | Manual           | Auto                  |
| Server   | External         | Embedded              |
| Events   | Limited          | Rich lifecycle events |
| Shutdown | Manual           | Graceful              |

📌 **Bean lifecycle is identical in both**

---

## 19. Interview One-Liners (Must Remember)

* "Spring Boot does not change Spring lifecycle; it automates and extends it."
* "Bean lifecycle remains the same; application lifecycle is enhanced."
* "Spring manages beans, Spring Boot manages the application."

---

## 20. Final Takeaway

Spring Boot lifecycle is a **superset** of Spring Framework lifecycle.
It adds:

* Auto-configuration
* Embedded server lifecycle
* Application-level events
* Production readiness via Actuator

While **reusing Spring Core’s bean lifecycle internally**.

---

✅ This document can be directly used for **interview prep, revision, or sharing as a `.md` file**.
