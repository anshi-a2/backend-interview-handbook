# 01_Introduction_To_Spring_Boot.md

> **Goal**
>
> Build a strong foundation of Spring Boot by understanding **why it exists, how Spring works internally, what IoC and Dependency Injection are, how Beans are managed, and what happens when a Spring Boot application starts.**
>
> **This chapter lays the foundation for all subsequent Spring Boot topics.**

---

# Table of Contents

1. Why Spring?
2. Evolution of Java Enterprise Development
3. What is Spring Framework?
4. What is Spring Boot?
5. Spring Framework vs Spring Boot
6. Spring Boot Architecture
7. Modules of Spring
8. Inversion of Control (IoC)
9. Dependency Injection (DI)
10. Bean & IoC Container
11. Bean Scopes
12. Bean Lifecycle
13. BeanFactory vs ApplicationContext
14. Component Scanning
15. Auto Configuration
16. Spring Boot Starter Dependencies
17. Spring Boot Project Structure
18. Common Annotations
19. Best Practices
20. Common Interview Questions
21. Key Takeaways

---

# 1. Why Spring?

Before Spring, building enterprise Java applications was difficult.

Developers had to manually manage:

- Object creation
- Dependency management
- Database connections
- Transactions
- Security
- Configuration
- Web server setup

Example (Without Spring)

```java
public class UserService {

    private UserRepository repository = new UserRepository();

}
```

Problems

- Tight coupling
- Difficult testing
- Hard to replace implementations
- Difficult maintenance

As applications grew,

```
Controller

↓

Service

↓

Repository

↓

Database
```

every class created another class using `new`.

The code became tightly coupled and difficult to scale.

Spring solved this problem using **IoC (Inversion of Control)**.

---

# 2. Evolution of Java Enterprise Development

```
Java Application

↓

Servlet

↓

JSP

↓

EJB (Java EE)

↓

Spring Framework

↓

Spring Boot
```

---

## Servlets

Developers manually handled:

- Request parsing
- Response creation
- Database connection
- Transactions
- Error handling

Large applications became difficult to maintain.

---

## EJB (Enterprise JavaBeans)

Attempted to simplify enterprise development.

Problems

- Heavyweight
- XML configuration
- Difficult deployment
- Complex APIs

---

## Spring Framework

Introduced

- Dependency Injection
- IoC
- POJO programming
- Lightweight architecture

---

## Spring Boot

Further simplified Spring by providing

- Auto Configuration
- Embedded Tomcat
- Starter Dependencies
- Minimal configuration
- Production-ready features

---

# 3. What is Spring Framework?

Spring is an **application framework** that helps build loosely coupled, maintainable, and testable Java applications.

It provides modules for

- Dependency Injection
- Web
- Security
- Transactions
- Data Access
- AOP
- Messaging
- Integration

Spring itself is **not an application server**.

---

# 4. What is Spring Boot?

Spring Boot is an opinionated framework built on top of Spring Framework.

It reduces boilerplate configuration and enables rapid application development.

Instead of configuring everything manually,

Spring Boot automatically configures components based on dependencies present in the project.

Example

Instead of configuring

- Tomcat
- Jackson
- DispatcherServlet
- Spring MVC

Spring Boot does it automatically.

---

# 5. Spring Framework vs Spring Boot

| Spring Framework | Spring Boot |
|------------------|-------------|
| Manual configuration | Auto Configuration |
| External Tomcat required | Embedded Tomcat |
| More XML/Java config | Minimal configuration |
| Individual dependencies | Starter dependencies |
| Longer setup | Quick setup |

Think of it as

```
Spring

↓

Framework

↓

Foundation

Spring Boot

↓

Framework + Automation
```

---

# 6. Spring Boot Architecture

```
Application

↓

SpringApplication.run()

↓

ApplicationContext

↓

Auto Configuration

↓

Component Scan

↓

Bean Creation

↓

Dependency Injection

↓

Embedded Tomcat

↓

Application Ready
```

Each block in this flow will be explained in detail in later chapters.

---

# 7. Spring Modules

Spring consists of many independent modules.

```
Spring Framework

├── Core
├── Beans
├── Context
├── AOP
├── MVC
├── JDBC
├── Data JPA
├── Security
├── WebFlux
├── Messaging
└── Test
```

Most applications use only the required modules.

---

# 8. Inversion of Control (IoC)

## Traditional Programming

The application controls object creation.

```java
UserRepository repository = new UserRepository();
```

Every class creates its own dependencies.

---

## Spring IoC

Spring controls object creation.

```
Spring Container

↓

Creates Objects

↓

Injects Dependencies

↓

Application Uses Them
```

Instead of

```
Developer

↓

Creates Objects
```

Spring becomes responsible.

This inversion of responsibility is called **Inversion of Control (IoC)**.

---

# 9. Dependency Injection (DI)

Dependency Injection is the mechanism used to implement IoC.

Instead of creating dependencies,

they are injected by Spring.

Without DI

```java
class UserService {

    private UserRepository repository = new UserRepository();

}
```

With DI

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Spring automatically provides the `UserRepository`.

---

## Benefits

- Loose coupling
- Better testing
- Easier maintenance
- Easy to replace implementations
- Better scalability

---

# 10. Bean & IoC Container

## What is a Bean?

A Bean is simply an object managed by Spring.

Example

```java
@Service
public class PaymentService {

}
```

Spring creates this object and stores it in the IoC container.

---

## IoC Container

The IoC container manages

- Bean creation
- Dependency injection
- Lifecycle
- Configuration
- Destruction

```
ApplicationContext

↓

Bean1

Bean2

Bean3

↓

Dependency Injection
```

---

# 11. Bean Scopes

By default,

every Bean is a **Singleton**.

```
One Bean

↓

Shared Everywhere
```

---

Common scopes

| Scope | Description |
|---------|-------------|
| Singleton | One instance per Spring container |
| Prototype | New instance every request from the container |
| Request | One instance per HTTP request |
| Session | One instance per user session |
| Application | One instance per ServletContext |

Singleton is the most commonly used scope.

---

# 12. Bean Lifecycle

Complete lifecycle

```
Bean Definition

↓

Object Creation

↓

Dependency Injection

↓

@PostConstruct

↓

Bean Ready

↓

Application Running

↓

@PreDestroy

↓

Bean Destroyed
```

Spring manages this lifecycle automatically.

---

# 13. BeanFactory vs ApplicationContext

## BeanFactory

Basic IoC container.

Features

- Bean creation
- Dependency Injection

Lazy initialization by default.

---

## ApplicationContext

Built on top of BeanFactory.

Adds

- Event publishing
- Internationalization
- Auto configuration support
- Environment management
- Bean post processing

Almost every Spring Boot application uses **ApplicationContext**.

---

# 14. Component Scanning

Spring automatically discovers Beans.

Example

```java
@Component
@Service
@Repository
@Controller
@RestController
```

Flow

```
Application Starts

↓

Component Scan

↓

Find Classes

↓

Create Beans

↓

Store in Container
```

No manual registration is required for these classes.

---

# 15. Auto Configuration

One of Spring Boot's biggest features.

Suppose your project includes

```xml
spring-boot-starter-web
```

Spring Boot automatically configures

- Embedded Tomcat
- DispatcherServlet
- Jackson
- Spring MVC
- Error handling

No manual configuration required.

Auto Configuration uses

- `@ConditionalOnClass`
- `@ConditionalOnBean`
- `@ConditionalOnMissingBean`
- `@ConditionalOnProperty`

These will be covered in detail in the Startup Internals chapter.

---

# 16. Spring Boot Starter Dependencies

Instead of adding dozens of individual libraries,

Spring Boot provides starter dependencies.

Examples

| Starter | Purpose |
|----------|---------|
| spring-boot-starter-web | REST APIs & MVC |
| spring-boot-starter-data-jpa | JPA & Hibernate |
| spring-boot-starter-security | Security |
| spring-boot-starter-test | Testing |
| spring-boot-starter-validation | Bean Validation |
| spring-boot-starter-actuator | Monitoring |

Benefits

- Compatible dependency versions
- Less configuration
- Faster setup

---

# 17. Typical Spring Boot Project Structure

```
src/main/java

├── controller
├── service
├── repository
├── entity
├── dto
├── config
├── exception
├── util
└── Application.java
```

Resources

```
src/main/resources

application.properties

application.yml

static/

templates/
```

---

# 18. Common Annotations

| Annotation | Purpose |
|------------|----------|
| @SpringBootApplication | Main application entry point |
| @Component | Generic Spring Bean |
| @Service | Business logic Bean |
| @Repository | Database access Bean |
| @Controller | MVC Controller |
| @RestController | REST Controller |
| @Configuration | Configuration class |
| @Bean | Creates a Bean manually |
| @Autowired | Injects dependency |
| @Value | Injects property values |

---

# 19. Best Practices

✔ Prefer **Constructor Injection** over field injection.

✔ Keep business logic inside Services.

✔ Keep Controllers lightweight.

✔ Avoid using `new` for Spring-managed components.

✔ Follow layered architecture:

```
Controller

↓

Service

↓

Repository

↓

Database
```

✔ Use interfaces where appropriate for loose coupling.

---

# 20. Common Interview Questions

### What is Spring?

A lightweight framework for building enterprise Java applications with features like IoC, Dependency Injection, AOP, transaction management, and web development.

---

### What is Spring Boot?

Spring Boot is built on top of Spring Framework and provides auto-configuration, starter dependencies, embedded servers, and production-ready features to simplify application development.

---

### What is IoC?

Inversion of Control is a design principle where the framework, instead of the application, controls object creation and lifecycle.

---

### What is Dependency Injection?

Dependency Injection is the mechanism through which Spring provides required objects (Beans) to other Beans instead of having classes create them manually.

---

### What is a Bean?

A Bean is an object that is created, configured, managed, and destroyed by the Spring IoC container.

---

### BeanFactory vs ApplicationContext?

`BeanFactory` is the basic IoC container with lazy initialization.

`ApplicationContext` extends `BeanFactory` with enterprise features such as event publishing, environment support, internationalization, and easier integration with Spring Boot.

---

### Why Constructor Injection?

- Ensures required dependencies are available.
- Makes classes immutable.
- Easier to unit test.
- Avoids partially initialized objects.

---

# 21. Key Takeaways

- **Spring Framework** provides the core infrastructure for enterprise Java applications.
- **Spring Boot** simplifies Spring development with auto-configuration, starter dependencies, and embedded servers.
- **IoC** means Spring manages object creation instead of application code.
- **Dependency Injection** enables loose coupling by injecting dependencies rather than creating them manually.
- A **Bean** is any object managed by the Spring IoC container.
- **ApplicationContext** is the primary container used in Spring Boot applications.
- **Component Scanning** automatically detects and registers Beans.
- **Auto Configuration** configures common infrastructure based on the project's dependencies.
- Understanding these concepts is essential because every advanced Spring Boot feature—MVC, Security, Data JPA, Transactions, AOP, and Microservices—builds on this foundation.
