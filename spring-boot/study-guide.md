# 📘 Spring Boot Study Guide

A **comprehensive guide** to learn, revise, and prepare for **Spring Boot** interviews and real-world development.

---

## 1. Introduction to Spring Boot
### Why Spring Boot?
- Reduces boilerplate code compared to Spring Framework.
- Provides auto-configuration for commonly used features.
- Comes with embedded servers (Tomcat, Jetty, Undertow).
- Easy setup for microservices and cloud-native apps.

### Key Features
- **Starter dependencies** – bundles of commonly used dependencies.
- **Auto-configuration** – configures beans automatically based on classpath.
- **Embedded servers** – run apps as standalone JARs.
- **Spring Boot CLI** – run Groovy-based Spring apps quickly.
- **Actuator** – production-ready monitoring endpoints.

---

## 2. Core Concepts
### Auto-Configuration
- Configures beans dynamically based on conditions.
- Controlled via `@EnableAutoConfiguration` (included in `@SpringBootApplication`).

### Starter Dependencies
- Example:  
  - `spring-boot-starter-web` → Web + REST.  
  - `spring-boot-starter-data-jpa` → Hibernate + JPA.  

### CLI
- Run apps directly with:  
  ```bash
  spring run app.groovy

---


## 3. Project Structure


src/main/java/com/example/demo
└── DemoApplication.java
src/main/resources
├── application.properties
└── static/



### Entry Point
```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```


## 4. Configuration
### Properties & YAML
- `application.properties`  
  ```properties
  server.port=8081
  spring.datasource.url=jdbc:mysql://localhost:3306/testdb
  ```

- `application.yml`
    ```yml
    server:
      port: 8081
    spring:
      datasource:
      url: jdbc:mysql://localhost:3306/testdb
    ```



- `profiles`
  ```java
  `@Profile("dev")
  @Configuration
    public class DevConfig {}

  ```

## 5. REST APIs

### Controller Example
```java
@RestController
@RequestMapping("/api")
public class UserController {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return new User(id, "John Doe");
    }
}
```

###Global Excpetion Handling

```java
  @ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handle(Exception e) {
        return ResponseEntity.badRequest().body(e.getMessage());
    }
}

```


### Validation

```java
  public class User {
    @NotNull @Size(min = 3)
    private String name;
}
```

## 6. Data Access

### JPA Repository
```java
public interface UserRepository extends JpaRepository<User, Long> {}
```


### Transactions

```java
@Transactional
public void updateUser(User user) {}

```

## 7. Security

### Basics
- Spring Security auto-configures login form.
- In-memory authentication for testing.

### JWT Authentication
- Token validation filter added to security chain.

### OAuth2
- Integration with Google/GitHub SSO via `spring-boot-starter-oauth2-client`.

