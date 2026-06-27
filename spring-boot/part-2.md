# 02_Spring_Boot_Startup_Internals.md

> **Goal**
>
> Understand **exactly what happens internally** from the moment you execute
>
> ```java
> SpringApplication.run(Application.class, args);
> ```
>
> until your application becomes ready to receive HTTP requests.
>
> This is one of the **most frequently asked SDE-2/Senior Backend interview topics.**

---

# Table of Contents

1. High-Level Startup Flow
2. What Happens When `main()` Executes?
3. SpringApplication
4. Preparing the Environment
5. Creating the ApplicationContext
6. Loading Auto Configuration
7. Component Scanning
8. Bean Creation Process
9. Dependency Injection
10. Bean Lifecycle During Startup
11. Embedded Tomcat Startup
12. DispatcherServlet Initialization
13. Application Ready
14. Startup Events
15. Internal Components
16. Common Startup Failures
17. Startup Optimization
18. Interview Summary

---

# 1. Complete Startup Flow

This entire flow happens when your application starts.

```
main()

↓

SpringApplication.run()

↓

Create SpringApplication Object

↓

Prepare Environment

↓

Read application.properties / application.yml

↓

Determine Web Application Type

↓

Create ApplicationContext

↓

Load Auto Configurations

↓

Component Scan

↓

Register Bean Definitions

↓

Create Singleton Beans

↓

Dependency Injection

↓

@PostConstruct

↓

Start Embedded Tomcat

↓

Register DispatcherServlet

↓

ApplicationReadyEvent

↓

Application Ready
```

Everything in this chapter expands on this flow.

---

# 2. What Happens Inside main()

Typical Spring Boot application

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

Although only one line is executed,

internally hundreds of operations happen.

The first object created is

```
SpringApplication
```

---

# 3. SpringApplication

`SpringApplication` is responsible for bootstrapping the application.

Internally it performs tasks such as

- Determine application type
- Read configuration
- Create ApplicationContext
- Register listeners
- Load auto-configurations
- Refresh context
- Start embedded server

Think of it as the **orchestrator** of the entire startup process.

---

# 4. Preparing the Environment

Before creating any Beans,

Spring prepares the application environment.

Sources include

```
Command Line Arguments

↓

Environment Variables

↓

application.properties

↓

application.yml

↓

JVM System Properties
```

Spring combines these into one object called

```
Environment
```

This object is later used by

```java
@Value

@ConfigurationProperties
```

---

## Property Priority (Highest → Lowest)

```
Command Line

↓

System Properties

↓

Environment Variables

↓

application.yml

↓

application.properties

↓

Default Values
```

---

# 5. Determine Application Type

Spring Boot checks the classpath.

Possible types

### Servlet Application

```
Spring MVC

↓

Embedded Tomcat
```

---

### Reactive Application

```
Spring WebFlux

↓

Netty
```

---

### Non-Web Application

```
Console Application
```

This determines which ApplicationContext will be created.

---

# 6. Creating the ApplicationContext

Spring now creates the IoC container.

For Spring MVC applications

```
AnnotationConfigServletWebServerApplicationContext
```

Responsibilities

- Store Beans
- Manage lifecycle
- Dependency Injection
- Publish Events
- Manage configuration

Everything in Spring lives inside the ApplicationContext.

---

# 7. Loading Auto Configuration

One of Spring Boot's most powerful features.

Suppose you added

```xml
spring-boot-starter-web
```

Spring Boot automatically configures

- Tomcat
- DispatcherServlet
- Jackson
- Spring MVC
- Error handling

without manual configuration.

---

## How Auto Configuration Works

Spring Boot scans

```
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

(which replaces the older `spring.factories` mechanism in modern Spring Boot versions)

This file contains hundreds of auto-configuration classes.

Example

```
DispatcherServletAutoConfiguration

JacksonAutoConfiguration

DataSourceAutoConfiguration

WebMvcAutoConfiguration

HibernateJpaAutoConfiguration
```

Each configuration is loaded only if conditions match.

---

# 8. Conditional Auto Configuration

Spring does NOT blindly create everything.

It evaluates conditions.

Examples

```
@ConditionalOnClass

↓

@ConditionalOnBean

↓

@ConditionalOnMissingBean

↓

@ConditionalOnProperty
```

Example

If Jackson library exists

↓

Configure Jackson

Otherwise

↓

Skip configuration

This keeps startup efficient.

---

# 9. Component Scanning

Spring scans packages starting from

```
@SpringBootApplication
```

Example

```
com.company

├── controller

├── service

├── repository

├── config
```

Classes annotated with

```
@Component

@Service

@Repository

@Controller

@RestController
```

are discovered.

---

# 10. Register Bean Definitions

Important Interview Point

Spring does **NOT immediately create every object**.

First it registers metadata.

```
Bean Definition

↓

Bean Name

↓

Bean Type

↓

Scope

↓

Dependencies
```

Only after registration does Spring begin creating Beans.

---

# 11. Bean Creation Process

For every Bean

```
Create Object

↓

Inject Dependencies

↓

Aware Interfaces

↓

BeanPostProcessor (Before)

↓

@PostConstruct

↓

InitializingBean

↓

Custom Init Method

↓

BeanPostProcessor (After)

↓

Bean Ready
```

---

Example

```
PaymentService

↓

Constructor

↓

Inject Repository

↓

@PostConstruct

↓

Ready
```

---

# 12. Dependency Injection

Suppose

```java
@Service
public class PaymentService {

    private final PaymentRepository repository;

    public PaymentService(PaymentRepository repository) {
        this.repository = repository;
    }
}
```

Flow

```
PaymentRepository Bean

↓

Create

↓

PaymentService Bean

↓

Inject Repository

↓

Store Bean
```

Spring resolves dependencies recursively.

---

# 13. BeanPostProcessor

One of Spring's extension points.

Purpose

Modify Beans before and after initialization.

Flow

```
Bean Created

↓

postProcessBeforeInitialization()

↓

@PostConstruct

↓

postProcessAfterInitialization()
```

Used internally for

- AOP proxies
- Transactions
- Security
- Validation

---

# 14. Embedded Tomcat Startup

Once essential Beans are ready,

Spring starts the embedded web server.

```
Tomcat

↓

Create Connector

↓

Open Port 8080

↓

Register Context

↓

Initialize Servlets
```

No external Tomcat installation required.

---

# 15. DispatcherServlet Initialization

Spring MVC's front controller.

```
HTTP Request

↓

DispatcherServlet

↓

HandlerMapping

↓

Controller

↓

Service

↓

Repository

↓

Database

↓

Response
```

DispatcherServlet is registered automatically.

We'll study this entire flow in Part 4.

---

# 16. Application Events

Spring publishes events during startup.

Major events

```
ApplicationStartingEvent

↓

ApplicationEnvironmentPreparedEvent

↓

ApplicationContextInitializedEvent

↓

ApplicationPreparedEvent

↓

ContextRefreshedEvent

↓

ApplicationStartedEvent

↓

ApplicationReadyEvent
```

Useful for

- Logging
- Metrics
- Startup tasks

---

# 17. CommandLineRunner & ApplicationRunner

Sometimes work should execute **after startup**.

Example

```java
@Component
public class StartupRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        System.out.println("Application Started");
    }
}
```

Runs after

```
ApplicationReadyEvent
```

Typical use cases

- Cache warming
- Initial data loading
- Startup validation

---

# 18. Complete Startup Timeline

```
main()

↓

SpringApplication

↓

Environment

↓

ApplicationContext

↓

Auto Configuration

↓

Component Scan

↓

Bean Definitions

↓

Bean Creation

↓

Dependency Injection

↓

@PostConstruct

↓

BeanPostProcessor

↓

Embedded Tomcat

↓

DispatcherServlet

↓

ApplicationStartedEvent

↓

CommandLineRunner

↓

ApplicationReadyEvent

↓

Ready for Requests
```

---

# 19. Internal Components

| Component | Responsibility |
|-----------|----------------|
| SpringApplication | Bootstraps application |
| Environment | Loads properties |
| ApplicationContext | IoC container |
| BeanFactory | Creates Beans |
| BeanPostProcessor | Modifies Beans |
| AutoConfiguration | Automatic setup |
| Embedded Tomcat | Web server |
| DispatcherServlet | Handles HTTP requests |

---

# 20. Common Startup Failures

## BeanCreationException

Reason

Bean could not be created.

Usually due to

- Missing dependency
- Exception in constructor
- Invalid configuration

---

## NoSuchBeanDefinitionException

Spring cannot find the required Bean.

Possible causes

- Missing `@Component`
- Wrong package scanning
- Bean not registered

---

## Port Already in Use

```
8080
```

already occupied.

Solution

Change

```properties
server.port=8081
```

---

## Circular Dependency

```
Service A

↓

Service B

↓

Service A
```

Spring cannot resolve constructor-based circular dependencies.

---

## Configuration Errors

Examples

- Missing database URL
- Invalid YAML syntax
- Missing environment variables

---

# 21. Startup Optimization

Production recommendations

✔ Use constructor injection.

✔ Avoid unnecessary Beans.

✔ Prefer lazy initialization for rarely used components when appropriate.

✔ Minimize expensive work in constructors.

✔ Keep `@PostConstruct` lightweight.

✔ Use `ApplicationRunner` or background initialization for long-running startup tasks.

✔ Remove unused starter dependencies.

---

# 22. Interview Summary

## Explain Spring Boot Startup

"When `SpringApplication.run()` is called, Spring Boot creates a `SpringApplication` instance, prepares the Environment by loading configuration from multiple property sources, determines the application type, creates an `ApplicationContext`, loads auto-configurations based on the classpath, scans components to register Bean definitions, creates singleton Beans with dependency injection, executes Bean lifecycle callbacks, starts the embedded web server (such as Tomcat), registers the `DispatcherServlet`, publishes startup events, executes any `CommandLineRunner` or `ApplicationRunner` beans, and finally publishes `ApplicationReadyEvent`, after which the application is ready to serve requests."

---

## What is Auto Configuration?

"Auto Configuration is Spring Boot's mechanism for automatically configuring infrastructure Beans based on dependencies available on the classpath and other runtime conditions. It uses conditional annotations such as `@ConditionalOnClass`, `@ConditionalOnMissingBean`, and `@ConditionalOnProperty` to decide whether a configuration should be applied."

---

## What is ApplicationContext?

"`ApplicationContext` is the central IoC container in Spring. It manages Bean creation, dependency injection, Bean lifecycle, events, configuration, and integrates various Spring modules."

---

# Key Takeaways

- `SpringApplication.run()` orchestrates the entire startup process.
- The **Environment** aggregates configuration from multiple property sources.
- **ApplicationContext** is the central container managing the application.
- Spring first registers **Bean Definitions**, then creates and wires Bean instances.
- **Auto Configuration** simplifies setup by creating infrastructure Beans only when conditions are satisfied.
- **BeanPostProcessors** allow Spring to enhance or proxy Beans for features like transactions and AOP.
- The embedded **Tomcat** starts automatically in servlet-based applications.
- **DispatcherServlet** becomes the entry point for all HTTP requests.
- Startup concludes with **ApplicationReadyEvent**, indicating the application is fully initialized and ready to process traffic.
