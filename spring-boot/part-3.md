# 03_Dependency_Injection_Beans_IoC_Internals.md

> **Goal**
>
> Master one of the most important Spring topics for SDE-2/Senior interviews:
>
> - IoC Container
> - Dependency Injection
> - Bean Creation
> - Bean Lifecycle
> - BeanFactory Internals
> - ApplicationContext Internals
> - Circular Dependency
> - Constructor vs Setter Injection
> - Singleton Thread Safety
>
> This chapter explains **how Spring actually creates and manages objects internally.**

---

# Table of Contents

1. Why Dependency Injection?
2. What is IoC?
3. What is Dependency Injection?
4. Spring Bean
5. BeanFactory Internals
6. ApplicationContext Internals
7. Bean Definition
8. Bean Creation Process
9. Dependency Resolution
10. Constructor vs Setter vs Field Injection
11. Bean Scopes
12. Bean Lifecycle
13. BeanPostProcessor
14. Circular Dependency
15. Lazy Initialization
16. Singleton Thread Safety
17. Common Bean Annotations
18. Common Production Issues
19. Best Practices
20. Interview Summary

---

# 1. Why Dependency Injection?

Imagine an Order Service.

Without Spring

```java
class OrderService {

    private PaymentService paymentService = new PaymentService();

}
```

PaymentService

↓

creates

↓

Repository

↓

creates

↓

Database

Every object creates another object.

Problems

- Tight coupling
- Difficult testing
- Hard to replace implementation
- Difficult mocking
- Poor maintainability

---

Instead

Spring creates everything.

```
Spring Container

↓

PaymentRepository

↓

PaymentService

↓

OrderService
```

OrderService simply receives the dependency.

---

# 2. What is IoC?

IoC stands for

```
Inversion of Control
```

Normally

```
Application

↓

Creates Objects
```

With Spring

```
Spring

↓

Creates Objects

↓

Injects Objects

↓

Manages Lifecycle
```

The control of object creation is inverted.

Hence

```
Inversion of Control
```

---

# 3. What is Dependency Injection?

Dependency Injection is the implementation of IoC.

Instead of

```java
new UserRepository()
```

Spring automatically injects it.

Example

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Flow

```
Spring Container

↓

Create Repository

↓

Create Service

↓

Inject Repository

↓

Ready
```

---

# 4. What is a Bean?

A Bean is simply

```
An object managed by Spring
```

Example

```java
@Service

@Repository

@Controller

@Component
```

All become Spring Beans.

---

Without Spring

```
Object

↓

Developer manages
```

With Spring

```
Bean

↓

Spring manages
```

---

# 5. BeanFactory Internals

BeanFactory is the core IoC container.

Responsibilities

- Bean creation
- Dependency resolution
- Lifecycle management

Flow

```
Request Bean

↓

Bean Exists?

↓

YES

Return Bean

↓

NO

Create Bean
```

---

Default implementation

```
DefaultListableBeanFactory
```

Almost every Spring Boot application uses this internally.

---

# 6. ApplicationContext Internals

ApplicationContext extends BeanFactory.

Adds

- Event Publishing
- Environment
- Resource Loading
- Internationalization
- Auto Configuration
- Bean Post Processing

Hierarchy

```
BeanFactory

↓

ApplicationContext

↓

WebApplicationContext
```

Spring Boot creates

```
AnnotationConfigServletWebServerApplicationContext
```

---

# 7. Bean Definition

Important Interview Point

Spring first creates

```
Bean Definitions
```

NOT actual objects.

Bean Definition stores

```
Bean Name

Class

Constructor

Scope

Dependencies

Init Method

Destroy Method
```

Later

BeanFactory creates the actual object.

---

# 8. Complete Bean Creation Process

```
Component Scan

↓

Bean Definition

↓

Constructor Selection

↓

Instantiate Bean

↓

Populate Dependencies

↓

Aware Interfaces

↓

BeanPostProcessor Before

↓

@PostConstruct

↓

afterPropertiesSet()

↓

Custom Init Method

↓

BeanPostProcessor After

↓

Bean Ready
```

---

# 9. Dependency Resolution

Suppose

```java
@Service
class OrderService {

    OrderService(PaymentService paymentService){}
}
```

PaymentService requires

```
Repository
```

Spring recursively resolves

```
Repository

↓

PaymentService

↓

OrderService
```

This process is called

```
Dependency Graph Resolution
```

---

# 10. Constructor Injection

Recommended approach.

```java
@Service
class PaymentService{

    private final Repository repository;

    PaymentService(Repository repository){
        this.repository=repository;
    }

}
```

Advantages

- Immutable objects
- Mandatory dependencies
- Easier testing
- Thread safety
- No partially initialized Bean

---

# 11. Setter Injection

```java
@Service
class PaymentService{

    private Repository repository;

    @Autowired
    void setRepository(Repository repository){
        this.repository=repository;
    }

}
```

Advantages

- Optional dependency

Disadvantages

- Mutable object
- May create incomplete Bean

---

# 12. Field Injection

```java
@Autowired

Repository repository;
```

Easy to write.

But

Not recommended.

Reasons

- Difficult testing
- Reflection
- Hidden dependency
- Mutable

Constructor Injection is preferred.

---

# Constructor vs Setter vs Field

| Type | Recommended | Immutable | Testable |
|--------|-------------|-----------|----------|
| Constructor | ✅ Yes | Yes | Excellent |
| Setter | Sometimes | No | Good |
| Field | No | No | Poor |

---

# 13. Bean Scopes

Default

```
Singleton
```

One object

Entire application.

---

Prototype

Every request to the container creates a new Bean.

---

Request Scope

One Bean

Per HTTP request.

---

Session Scope

One Bean

Per user session.

---

Application Scope

One Bean

Per ServletContext.

---

# 14. Bean Lifecycle

```
Bean Definition

↓

Object Created

↓

Dependency Injection

↓

BeanNameAware

↓

ApplicationContextAware

↓

BeanPostProcessor Before

↓

@PostConstruct

↓

InitializingBean

↓

Custom Init

↓

BeanPostProcessor After

↓

Ready

↓

Application Running

↓

@PreDestroy

↓

Destroy
```

---

# 15. Aware Interfaces

Spring can inject framework objects.

Examples

```
BeanNameAware

ApplicationContextAware

EnvironmentAware

ResourceLoaderAware
```

Useful when Bean needs framework information.

---

# 16. BeanPostProcessor

One of Spring's most powerful extension points.

```
Bean

↓

Before Initialization

↓

Modify Bean

↓

Initialization

↓

After Initialization

↓

Proxy if required
```

Used by

- Spring AOP
- Spring Security
- Transactions
- Validation

---

# 17. Circular Dependency

Example

```
A

↓

B

↓

A
```

Code

```java
class A{

    A(B b){}

}

class B{

    B(A a){}

}
```

Spring cannot create either Bean.

Result

```
BeanCurrentlyInCreationException
```

---

Solutions

- Redesign
- Extract common dependency
- Use `@Lazy` (only if appropriate)

---

# 18. Lazy Initialization

Default

```
Singleton Beans

↓

Created During Startup
```

Lazy

```
Created

↓

Only When First Used
```

Useful for

- Heavy initialization
- Rarely used Beans

---

# 19. Singleton Thread Safety

Very common interview question.

Question

```
Singleton Bean

↓

Many Threads

↓

Safe?
```

Answer

Depends.

Example

```java
@Service
class Counter{

    int count=0;

    public void increment(){
        count++;
    }

}
```

Not thread-safe.

Reason

Multiple threads modify shared state.

---

Safe Singleton

```java
@Service
class UserService{

    Repository repository;

}
```

Repository reference is immutable.

No shared mutable state.

Thread-safe.

---

Rule

Singleton Beans should be

```
Stateless
```

Store request-specific data in local variables, not instance fields.

---

# 20. @Primary vs @Qualifier

Suppose

```
PaypalPaymentService

StripePaymentService
```

Both implement

```
PaymentService
```

Spring becomes confused.

---

Use

```java
@Primary
```

Default implementation.

---

Use

```java
@Qualifier("stripe")
```

Specific implementation.

---

# 21. @Component vs @Bean

@Component

```
Spring discovers automatically

↓

Component Scan
```

---

@Bean

```
Developer manually creates Bean

↓

@Configuration
```

Useful for third-party libraries.

---

# 22. Common Production Issues

## NoSuchBeanDefinitionException

Bean not found.

Reasons

- Missing annotation
- Wrong package scan

---

## BeanCreationException

Constructor failed.

Possible reasons

- Exception
- Missing configuration
- Invalid dependency

---

## Circular Dependency

Usually caused by

Service A

↓

Service B

↓

Service A

---

## Multiple Bean Exception

```
NoUniqueBeanDefinitionException
```

Solution

```
@Primary

or

@Qualifier
```

---

## Slow Startup

Reasons

- Too many Beans
- Heavy constructors
- Expensive @PostConstruct logic

---

# 23. Best Practices

✔ Prefer Constructor Injection

✔ Avoid Field Injection

✔ Keep Singleton Beans stateless

✔ Avoid Circular Dependencies

✔ Keep `@PostConstruct` lightweight

✔ Use interfaces for abstractions

✔ Do not manually create Spring-managed objects using `new`

✔ Use `@Qualifier` when multiple implementations exist

---

# 24. Interview Summary

## Explain IoC

"Inversion of Control means that the responsibility for creating, configuring, and managing objects is transferred from the application code to the Spring container. Instead of classes creating their own dependencies, Spring manages the entire object lifecycle."

---

## Explain Dependency Injection

"Dependency Injection is the mechanism through which the Spring container provides required dependencies to a Bean. Spring first resolves the dependency graph, creates dependent Beans, and injects them using constructor, setter, or field injection. Constructor injection is the preferred approach because it promotes immutability and easier testing."

---

## Explain Bean Lifecycle

"When Spring starts, it scans components and registers Bean definitions. During context initialization, the BeanFactory creates Bean instances, injects dependencies, invokes Aware callbacks, executes BeanPostProcessors before initialization, calls `@PostConstruct`, `afterPropertiesSet()`, and custom init methods, then executes BeanPostProcessors after initialization. On shutdown, Spring invokes `@PreDestroy` and custom destroy methods."

---

## Why Constructor Injection?

- Ensures mandatory dependencies are available.
- Makes objects immutable.
- Improves unit testing.
- Prevents partially initialized Beans.
- Encourages better design.

---

## Is Singleton Bean Thread Safe?

"A Singleton Bean is not automatically thread-safe. Since a single instance serves multiple requests concurrently, any mutable shared state can lead to race conditions. Singleton Beans should therefore be designed as stateless components, with request-specific data stored in local variables rather than instance fields."

---

# Key Takeaways

- **IoC** shifts object creation and lifecycle management to the Spring container.
- **Dependency Injection** enables loose coupling and better testability.
- A **Bean** is an object managed by Spring.
- **BeanFactory** performs Bean creation and dependency resolution, while **ApplicationContext** adds enterprise capabilities such as events and environment support.
- Spring first registers **Bean Definitions**, then creates Bean instances.
- **Constructor Injection** is the recommended dependency injection style.
- **BeanPostProcessors** enable advanced framework features such as AOP, transactions, and security by modifying or proxying Beans.
- **Singleton** is the default Bean scope and should remain stateless for thread safety.
- Understanding the Bean lifecycle is fundamental to understanding how Spring Boot works internally.
