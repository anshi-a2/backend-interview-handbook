# Spring Boot Interview Questions and Answers (SDE-2 Level)

This guide covers advanced-level Spring Boot interview questions with detailed answers for SDE-2 backend developer interviews.

---

## 1. What are the main differences between Spring and Spring Boot?
**Answer:**
- **Spring**: Provides a comprehensive framework but requires manual configuration (XML/Java config). Developers must set up server, dependency management, etc.
- **Spring Boot**: Provides auto-configuration, embedded servers (Tomcat, Jetty, Undertow), starter dependencies, actuator, and production-ready features with minimal configuration.

---

## 2. What is Spring Boot Auto-Configuration and how does it work?
**Answer:**
- Auto-configuration automatically configures Spring Beans based on dependencies available in the classpath.
- Uses `@EnableAutoConfiguration` and `spring.factories`.
- Works with **Conditional Annotations** such as:
  - `@ConditionalOnClass`
  - `@ConditionalOnMissingBean`
  - `@ConditionalOnProperty`
- Example:
  ```java
  @Configuration
  @ConditionalOnClass(DataSource.class)
  public class DataSourceAutoConfiguration { ... }
  ```

---

## 3. How does Spring Boot manage dependency injection?
**Answer:**
- Uses **Spring IoC Container** with annotations like `@Component`, `@Service`, `@Repository`.
- Beans are injected using:
  - `@Autowired`
  - Constructor Injection (preferred for immutability and testing).
- SDE-2 expectation: Discuss **circular dependencies** and resolution (`@Lazy`, redesigning bean relationships).

---

## 4. What are Spring Boot Starters?
**Answer:**
- **Starter POMs**: Predefined dependencies for specific use cases (e.g., web, data, security).
- Examples:
  - `spring-boot-starter-web` → Spring MVC + Tomcat + Jackson.
  - `spring-boot-starter-data-jpa` → Hibernate + JPA.
- Benefits: Reduces boilerplate dependency management, avoids version conflicts.

---

## 5. Explain Spring Boot Actuator and its use cases.
**Answer:**
- Provides **production-ready monitoring** endpoints (health, metrics, info, env).
- Example endpoints:
  - `/actuator/health`
  - `/actuator/metrics`
- Use Cases:
  - Application monitoring
  - Integration with **Prometheus/Grafana**
  - Debugging runtime issues
- Can secure endpoints using Spring Security.

---

## 6. How do you secure a Spring Boot application?
**Answer:**
- Use **Spring Security** dependency (`spring-boot-starter-security`).
- Common approaches:
  - Basic Authentication
  - JWT (JSON Web Tokens)
  - OAuth2/OpenID Connect
- Example: JWT Security
  ```java
  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
      http.csrf().disable()
          .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
          .oauth2ResourceServer().jwt();
      return http.build();
  }
  ```

---

## 7. What is the difference between @Component, @Service, and @Repository?
**Answer:**
- All are **stereotype annotations** for Spring Beans.
- `@Component`: Generic bean.
- `@Service`: Business logic layer bean.
- `@Repository`: DAO layer bean, adds exception translation for persistence errors.

---

## 8. How do you configure and use caching in Spring Boot?
**Answer:**
- Enable caching with `@EnableCaching`.
- Use caching annotations:
  - `@Cacheable`
  - `@CachePut`
  - `@CacheEvict`
- Example:
  ```java
  @Cacheable("users")
  public User getUserById(Long id) {
      return userRepository.findById(id).orElseThrow();
  }
  ```
- Backend options: **EhCache, Redis, Caffeine**.

---

## 9. How does Spring Boot handle exception management?
**Answer:**
- Use `@ControllerAdvice` + `@ExceptionHandler`.
- Example:
  ```java
  @ControllerAdvice
  public class GlobalExceptionHandler {
      @ExceptionHandler(ResourceNotFoundException.class)
      public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
          return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
      }
  }
  ```
- Benefits: Centralized error handling, clean controllers, custom error responses.

---

## 10. How do you handle transactions in Spring Boot?
**Answer:**
- Spring provides **transaction management** with `@Transactional`.
- Example:
  ```java
  @Service
  public class PaymentService {
      @Transactional
      public void processPayment(Order order) {
          orderRepository.save(order);
          paymentRepository.save(order.getPayment());
      }
  }
  ```
- Key Points:
  - Default rollback on runtime exceptions.
  - Can configure rollback for checked exceptions.
  - Transaction propagation levels (e.g., `REQUIRES_NEW`, `MANDATORY`).

---

## 11. How do you deploy a Spring Boot application in production?
**Answer:**
- **Ways to deploy:**
  - JAR with embedded Tomcat (most common).
  - WAR on external server.
  - Docker container.
  - Kubernetes (for scaling).
- Considerations:
  - Externalized configuration (`application.yml`, environment variables).
  - Monitoring with Actuator.
  - CI/CD integration.

---

## 12. How does Spring Boot support Microservices architecture?
**Answer:**
- Provides:
  - Embedded servers (independent deployability).
  - Spring Cloud integration (service discovery, config server, API gateway).
  - Easy REST API development.
- Features:
  - **Resilience4j/Hystrix** for fault tolerance.
  - **Sleuth + Zipkin** for distributed tracing.
  - **Spring Cloud Config** for centralized configuration.

---

## 13. What are Profiles in Spring Boot and how are they used?
**Answer:**
- **Profiles** allow environment-specific configurations.
- Example files:
  - `application-dev.yml`
  - `application-prod.yml`
- Activate via:
  - Command-line: `--spring.profiles.active=dev`
  - Config file
- Example usage:
  ```java
  @Profile("dev")
  @Bean
  public DataSource devDataSource() { ... }
  ```

---

## 14. How does Spring Boot integrate with databases?
**Answer:**
- Using `spring-boot-starter-data-jpa` for ORM.
- Configuration:
  ```yaml
  spring:
    datasource:
      url: jdbc:mysql://localhost:3306/testdb
      username: root
      password: root
    jpa:
      hibernate:
        ddl-auto: update
      show-sql: true
  ```
- Supports:
  - JPA/Hibernate
  - JDBC
  - Reactive Repositories (Spring Data R2DBC)

---

## 15. How do you test Spring Boot applications?
**Answer:**
- Testing tools:
  - `@SpringBootTest` → Full integration test.
  - `@WebMvcTest` → Test controllers only.
  - `@DataJpaTest` → Test JPA repositories.
- Example:
  ```java
  @SpringBootTest
  public class UserServiceTest {
      @Autowired
      private UserService userService;

      @Test
      public void testUserCreation() {
          User user = userService.createUser("Alice");
          assertNotNull(user.getId());
      }
  }
  ```

---

# ✅ Final Tips for SDE-2 Interviews:
- Expect **system design + Spring Boot integration questions** (e.g., designing a URL shortener, payment system).
- Be ready to discuss **scalability, caching, resilience, and observability** in microservices.
- Prepare for **hands-on coding tasks** (REST API, database integration).
