# 09_Advanced_Spring_Internals_AOP_Transactions_Events.md

> **Goal**
>
> This chapter covers the advanced Spring Framework concepts that distinguish a senior backend engineer. It focuses on **how Spring implements AOP, transactions, proxies, events, bean customization, and advanced container features internally**.
>
> These topics are frequently discussed in **SDE-2/Senior Backend interviews** because they demonstrate an understanding of Spring beyond basic application development.

---

# Table of Contents

1. Spring AOP Architecture
2. Dynamic Proxies (JDK vs CGLIB)
3. How `@Transactional` Works Internally
4. Transaction Propagation
5. Transaction Isolation Levels
6. Rollback Rules
7. Why Self-Invocation Breaks Transactions
8. BeanFactoryPostProcessor
9. BeanPostProcessor vs BeanFactoryPostProcessor
10. FactoryBean
11. ObjectProvider
12. Spring Events
13. Spring Profiles
14. Conditional Beans
15. Environment & Property Resolution
16. Spring Expression Language (SpEL)
17. Validation (`@Valid`)
18. ThreadLocal & Request Context
19. Common Production Issues
20. Best Practices
21. Interview Summary

---

# 1. Spring AOP

AOP stands for

```
Aspect Oriented Programming
```

It allows Spring to add behavior **without modifying your business code**.

Example

```
Logging

↓

Security

↓

Transactions

↓

Caching

↓

Business Logic
```

Instead of writing logging code in every service,

Spring executes it automatically.

---

# 2. AOP Terminology

```
Aspect
```

Cross-cutting concern

Examples

- Logging
- Security
- Transactions

---

```
Advice
```

Action to perform.

Examples

- Before
- After
- Around
- After Throwing

---

```
Join Point
```

A method execution where advice can run.

---

```
Pointcut
```

Rule defining where advice should execute.

---

# 3. How Spring AOP Works

Suppose

```java
@Transactional
public void placeOrder(){}
```

Spring does **not** modify your class.

Instead

```
Client

↓

Proxy

↓

Transaction Start

↓

Original Method

↓

Commit/Rollback

↓

Return
```

Everything happens through a proxy.

---

# 4. Dynamic Proxies

Spring creates proxy objects at runtime.

### JDK Dynamic Proxy

Used when the Bean implements an interface.

```
Interface

↓

JDK Proxy

↓

Target Bean
```

---

### CGLIB Proxy

Used when no interface exists.

```
Class

↓

Subclass Generated

↓

Target Bean
```

---

| JDK Proxy | CGLIB |
|------------|--------|
| Interface required | No interface required |
| Uses `java.lang.reflect.Proxy` | Generates subclass |
| Default when interface exists | Used otherwise |

---

# 5. How `@Transactional` Works

Example

```java
@Transactional
public void transferMoney(){}
```

Internal flow

```
Client

↓

Proxy

↓

Open Transaction

↓

Execute Method

↓

Success?

↓

Commit

↓

Else Rollback
```

Without the proxy,

`@Transactional` has no effect.

---

# 6. Transaction Propagation

Propagation determines how nested method calls behave.

Common modes

### REQUIRED (Default)

```
Existing Transaction?

↓

Yes → Join

↓

No → Create New
```

---

### REQUIRES_NEW

Always creates a new transaction.

```
Suspend Existing

↓

Create New

↓

Commit

↓

Resume Old
```

---

### SUPPORTS

Uses an existing transaction if one exists.

Otherwise

Runs without a transaction.

---

### MANDATORY

Requires an existing transaction.

Otherwise

Throws an exception.

---

### NEVER

Fails if a transaction already exists.

---

### NOT_SUPPORTED

Suspends the current transaction and executes without one.

---

# 7. Transaction Isolation Levels

Isolation controls how concurrent transactions interact.

### READ_UNCOMMITTED

May read uncommitted data.

Highest concurrency.

Lowest consistency.

---

### READ_COMMITTED

Cannot read uncommitted changes.

Common default in many databases.

---

### REPEATABLE_READ

Repeated reads return consistent results.

Prevents non-repeatable reads.

---

### SERIALIZABLE

Highest isolation.

Transactions behave as if executed one after another.

Highest consistency.

Lowest concurrency.

---

# 8. Rollback Rules

By default

Spring rolls back

```
RuntimeException

Error
```

Checked exceptions do **not** trigger rollback unless configured.

Example

```java
@Transactional(rollbackFor = Exception.class)
```

---

# 9. Why Self-Invocation Breaks Transactions

Classic interview question.

Example

```java
@Service
class PaymentService{

    public void methodA(){
        methodB();
    }

    @Transactional
    public void methodB(){}

}
```

Flow

```
methodA()

↓

Direct Call

↓

methodB()

↓

No Proxy

↓

No Transaction
```

Because the call stays inside the same object,

the proxy is bypassed.

Solutions

- Move transactional logic to another Bean.
- Inject the proxied Bean.
- Refactor responsibilities.

---

# 10. BeanFactoryPostProcessor

Executes **before Bean creation**.

Responsibilities

- Modify Bean Definitions
- Change Bean metadata
- Customize container behavior

Flow

```
Bean Definitions

↓

BeanFactoryPostProcessor

↓

Modified Definitions

↓

Bean Creation
```

---

# 11. BeanPostProcessor vs BeanFactoryPostProcessor

| BeanPostProcessor | BeanFactoryPostProcessor |
|-------------------|--------------------------|
| Works on Bean instances | Works on Bean definitions |
| Executes after object creation | Executes before object creation |
| Can wrap Beans with proxies | Can modify Bean metadata |

---

# 12. FactoryBean

FactoryBean creates **other Beans**.

Instead of

```
Spring

↓

Creates Bean
```

Flow becomes

```
Spring

↓

FactoryBean

↓

Creates Bean

↓

Application Uses Bean
```

Useful for complex object creation.

---

# 13. ObjectProvider

Useful when dependency is optional or lazily resolved.

Instead of

```java
@Autowired
MyService service;
```

Use

```java
ObjectProvider<MyService>
```

Benefits

- Lazy lookup
- Optional dependency
- Avoid eager initialization

---

# 14. Spring Events

Spring supports an internal event mechanism.

Flow

```
Publisher

↓

ApplicationEventPublisher

↓

Event

↓

Listener

↓

Business Logic
```

Example use cases

- Send email after registration
- Audit logging
- Cache refresh
- Notifications

Can be synchronous or asynchronous.

---

# 15. Spring Profiles

Profiles allow environment-specific configuration.

```
dev

test

stage

prod
```

Example

```
application-dev.yml

application-prod.yml
```

Activate

```
spring.profiles.active=prod
```

---

# 16. Conditional Beans

Create Beans only when conditions match.

Examples

```
@ConditionalOnClass

@ConditionalOnProperty

@ConditionalOnMissingBean

@ConditionalOnBean
```

Spring Boot auto-configuration relies heavily on these annotations.

---

# 17. Environment & Property Resolution

Spring loads configuration from multiple sources.

Priority (highest to lowest)

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

The `Environment` abstraction resolves the effective value.

---

# 18. Spring Expression Language (SpEL)

SpEL allows runtime expression evaluation.

Example

```java
@Value("#{systemProperties['user.home']}")
```

Common use cases

- Conditional expressions
- Bean references
- Property calculations

---

# 19. Validation

Spring integrates with Jakarta Bean Validation.

Example

```java
@NotNull

@Email

@Size

@Positive
```

Flow

```
Incoming Request

↓

Validation

↓

Valid?

↓

Controller

↓

Else

400 Bad Request
```

Use

```java
@Valid
```

---

# 20. ThreadLocal & Request Context

Spring stores request-scoped information using ThreadLocal.

Examples

- SecurityContext
- Locale
- RequestAttributes
- Logging MDC

Flow

```
Request

↓

Thread

↓

ThreadLocal

↓

Controller

↓

Response

↓

Clear ThreadLocal
```

Important

Failure to clear ThreadLocal in custom code can cause memory leaks in thread pools.

---

# 21. Common Production Issues

## `@Transactional` Not Working

Possible reasons

- Self-invocation
- Method not public
- Bean not managed by Spring
- Proxy bypassed

---

## `@Async` Not Working

Common causes

- Self-invocation
- Missing `@EnableAsync`
- Calling from the same Bean

---

## Duplicate Event Handling

If listeners are retried,

design handlers to be idempotent.

---

## Wrong Profile Loaded

Check

```
spring.profiles.active
```

and environment variables.

---

## Bean Not Created

Investigate

- Conditional annotations
- Component scanning
- Auto-configuration exclusions

---

# 22. Production Best Practices

✔ Prefer constructor injection.

✔ Keep transactions small.

✔ Avoid long-running transactions.

✔ Use `REQUIRES_NEW` only when necessary.

✔ Prefer events for loosely coupled communication.

✔ Keep AOP focused on cross-cutting concerns.

✔ Avoid putting business logic in aspects.

✔ Use profiles for environment-specific behavior.

✔ Validate input at API boundaries.

✔ Be careful when using ThreadLocal in custom code.

---

# 23. Interview Summary

## How Does Spring AOP Work?

"Spring AOP uses runtime-generated proxy objects to intercept method calls. The proxy executes additional behavior, such as logging, security, or transaction management, before and/or after delegating to the target object. If a Bean implements an interface, Spring typically uses JDK Dynamic Proxies; otherwise, it creates a CGLIB subclass."

---

## How Does `@Transactional` Work?

"`@Transactional` is implemented using Spring AOP. A proxy intercepts calls to transactional methods, starts a transaction before invoking the target method, commits the transaction if execution succeeds, and rolls it back if an applicable exception occurs."

---

## Why Does Self-Invocation Break Transactions?

"When one method in a class directly calls another method in the same class, the call bypasses the Spring proxy because it does not leave the object. Since the proxy is responsible for transaction management, the transactional advice is never applied."

---

## BeanPostProcessor vs BeanFactoryPostProcessor

- **BeanFactoryPostProcessor** modifies Bean definitions before Beans are instantiated.
- **BeanPostProcessor** modifies or wraps Bean instances after they are created, commonly by creating proxies.

---

## What is ThreadLocal Used For?

"ThreadLocal stores data that is specific to the current thread. Spring uses it internally for components such as the SecurityContext, request attributes, locale information, and logging correlation data, allowing each request thread to maintain isolated contextual information."

---

# Key Takeaways

- **Spring AOP** implements cross-cutting concerns using runtime-generated proxies.
- **JDK Dynamic Proxies** require interfaces, while **CGLIB** creates subclasses for concrete classes.
- **`@Transactional`** works only when method calls pass through the Spring proxy.
- **Transaction propagation** and **isolation levels** determine transactional behavior in nested and concurrent operations.
- **BeanFactoryPostProcessor** customizes Bean definitions; **BeanPostProcessor** customizes Bean instances.
- **Spring Events** enable loosely coupled communication between application components.
- **Profiles** and **conditional Beans** support environment-specific configuration and auto-configuration.
- **ThreadLocal** underpins many Spring framework features but must be used carefully in custom code.
- Understanding Spring's proxy-based architecture is essential for debugging advanced production issues and succeeding in senior backend interviews.
