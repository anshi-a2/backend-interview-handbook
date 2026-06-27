# 01_Spring_Boot_2_to_3_Migration_Guide.md

> **Goal**
>
> This guide explains everything required to migrate an application from **Spring Boot 2.x to Spring Boot 3.x**. It covers the migration strategy, breaking changes, dependency upgrades, Jakarta migration, Spring Security changes, Hibernate 6 updates, production challenges, and interview questions.

---

# Table of Contents

1. Why Upgrade to Spring Boot 3?
2. Prerequisites
3. High-Level Migration Strategy
4. Major Changes
5. Jakarta EE Migration
6. Spring Framework 6 Changes
7. Spring Security Migration
8. Hibernate 6 Migration
9. Configuration Changes
10. Actuator & Observability
11. Testing Changes
12. Common Migration Issues
13. Production Migration Strategy
14. Best Practices
15. Interview Questions

---

# 1. Why Upgrade to Spring Boot 3?

Spring Boot 3 is built on **Spring Framework 6** and is a major release with several improvements.

### Benefits

- Long-term support
- Better performance
- Java 17+ support
- Native Image (GraalVM) support
- Improved Observability
- Better Security
- Hibernate 6 support
- Jakarta EE 9+ compatibility

---

## Why did companies migrate?

- Spring Boot 2.x reached End of Life.
- Security patches are focused on Boot 3.
- Better cloud-native support.
- Improved monitoring with Micrometer.
- Better performance and memory usage.
- Compatibility with modern Java versions (17, 21).

---

# 2. Prerequisites

Before upgrading:

| Requirement | Version |
|------------|---------|
| Java | 17 or above |
| Spring Framework | 6.x |
| Maven | 3.8+ |
| Gradle | 7.5+ |
| Hibernate | 6.x |

---

# 3. High-Level Migration Strategy

A recommended migration flow:

```text
Upgrade JDK

↓

Upgrade Build Tool

↓

Upgrade Spring Boot Version

↓

Update Dependencies

↓

Replace javax.* with jakarta.*

↓

Upgrade Hibernate

↓

Migrate Spring Security

↓

Fix Deprecated APIs

↓

Run Unit Tests

↓

Run Integration Tests

↓

Deploy to Staging

↓

Production Deployment
```

Never perform all upgrades at once in production.

---

# 4. Major Changes

| Spring Boot 2 | Spring Boot 3 |
|---------------|---------------|
| Java 8/11 | Java 17+ |
| Spring Framework 5 | Spring Framework 6 |
| javax.* | jakarta.* |
| Hibernate 5 | Hibernate 6 |
| Older Micrometer | New Observability APIs |

---

# 5. Jakarta EE Migration

This is the **biggest breaking change**.

### Before (Boot 2)

```java
import javax.persistence.Entity;
import javax.validation.Valid;
import javax.servlet.Filter;
```

### After (Boot 3)

```java
import jakarta.persistence.Entity;
import jakarta.validation.Valid;
import jakarta.servlet.Filter;
```

### Packages Changed

| Old | New |
|------|------|
| javax.persistence | jakarta.persistence |
| javax.validation | jakarta.validation |
| javax.servlet | jakarta.servlet |
| javax.annotation | jakarta.annotation |
| javax.transaction | jakarta.transaction |

---

## Why was this change made?

Oracle transferred Java EE to the Eclipse Foundation, which renamed it to **Jakarta EE**. Spring Boot 3 aligns with this ecosystem.

---

# 6. Spring Framework 6 Changes

Spring Framework 6 introduced:

- Java 17 baseline
- Native Image support
- Better AOT (Ahead-of-Time) processing
- Improved reflection handling
- Better startup performance

---

# 7. Spring Security Migration

One of the biggest interview topics.

---

## WebSecurityConfigurerAdapter Removed

### Spring Boot 2

```java
public class SecurityConfig
        extends WebSecurityConfigurerAdapter {

}
```

### Spring Boot 3

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    return http.build();
}
```

---

### Why was it removed?

Spring now favors **component-based configuration** over inheritance.

Benefits

- Cleaner configuration
- Better modularity
- Easier testing

---

## AuthenticationManager

Earlier

```java
authenticationManagerBean()
```

Now

```java
@Bean
AuthenticationManager
```

---

## Authorization Changes

Instead of

```java
authorizeRequests()
```

Use

```java
authorizeHttpRequests()
```

---

# 8. Hibernate 6 Migration

Spring Boot 3 upgrades to **Hibernate 6**.

### Changes

- Improved SQL generation
- Better query performance
- Updated dialects
- New type system
- Better batch processing

---

### Possible Issues

- Native queries may fail.
- Deprecated APIs removed.
- Custom dialects may require updates.
- HQL syntax changes.

---

# 9. Configuration Changes

Some properties were renamed or removed.

### Example

Old

```properties
spring.redis.host
```

May require updated property structures depending on the Spring Boot version and starter.

Always review the migration guide for deprecated properties.

---

# 10. Actuator & Observability

Spring Boot 3 introduces better observability.

### Features

- Micrometer improvements
- OpenTelemetry integration
- Better metrics
- Better tracing
- Better logging

Architecture

```text
Application

↓

Micrometer

↓

Prometheus

↓

Grafana
```

Benefits

- Distributed tracing
- Better dashboards
- Production monitoring

---

# 11. Testing Changes

Most existing tests continue to work.

Verify compatibility for:

- JUnit 5
- Mockito
- MockMvc
- Testcontainers

Run both:

- Unit Tests
- Integration Tests

before deployment.

---

# 12. Common Migration Issues

## 1. ClassNotFoundException

Example

```
javax.servlet.Filter
```

Cause

Package renamed to

```
jakarta.servlet.Filter
```

---

## 2. NoSuchMethodError

Reason

Older dependency incompatible with Spring Boot 3.

Solution

Upgrade dependency versions.

---

## 3. Bean Creation Exception

Reason

Third-party library still depends on `javax.*`.

---

## 4. Security Configuration Failure

Reason

Still using

```java
WebSecurityConfigurerAdapter
```

Solution

Replace with

```java
SecurityFilterChain
```

---

## 5. Hibernate Exceptions

Possible reasons

- Query syntax changes
- Deprecated APIs
- Custom dialect incompatibility

---

# 13. Production Migration Strategy

A safe rollout strategy:

```text
Development

↓

Upgrade Branch

↓

Compile

↓

Fix Build Errors

↓

Unit Testing

↓

Integration Testing

↓

Performance Testing

↓

Staging

↓

Canary Deployment

↓

Production
```

---

## Deployment Tips

- Upgrade one service at a time.
- Monitor logs closely.
- Enable rollback strategy.
- Validate database compatibility.
- Test security flows thoroughly.

---

# 14. Best Practices

✔ Upgrade to Java 17 before Boot 3.

✔ Upgrade dependencies first.

✔ Replace all `javax.*` imports.

✔ Review deprecated APIs.

✔ Update Spring Security configuration.

✔ Test Hibernate queries.

✔ Monitor application metrics after deployment.

✔ Use staging and canary deployments before full rollout.

---

# 15. Frequently Asked Interview Questions

## Q1. Why did you upgrade to Spring Boot 3?

**Answer**

Spring Boot 3 provides long-term support, requires Java 17+, improves performance, supports Jakarta EE 9+, integrates Hibernate 6, enhances observability, and offers better cloud-native capabilities.

---

## Q2. What was the biggest breaking change?

**Answer**

The migration from `javax.*` to `jakarta.*` was the largest change. All persistence, validation, servlet, and transaction imports needed updating, along with any incompatible third-party libraries.

---

## Q3. Why was WebSecurityConfigurerAdapter removed?

**Answer**

Spring Security moved to a component-based configuration model using `SecurityFilterChain`, which is more modular, flexible, and easier to test than inheritance-based configuration.

---

## Q4. What challenges did you face?

Typical answers:

- Updating `javax.*` imports
- Third-party dependency incompatibilities
- Hibernate query changes
- Spring Security migration
- Deprecated configuration properties

---

## Q5. How did you ensure a safe migration?

**Answer**

We upgraded Java first, updated dependencies, migrated framework changes incrementally, executed comprehensive unit and integration tests, validated performance in staging, performed a canary deployment, and continuously monitored logs and metrics during rollout.

---

## Q6. Why is Java 17 mandatory?

**Answer**

Spring Framework 6, which underpins Spring Boot 3, uses modern Java language features and APIs that require Java 17 or later.

---

## Q7. How would you explain the migration in an interview?

**Sample Answer**

> "We first upgraded the application to Java 17, then updated the Spring Boot version and compatible dependencies. The largest effort was migrating from `javax.*` to `jakarta.*`, followed by updating Spring Security to use `SecurityFilterChain` and validating Hibernate 6 compatibility. We ran extensive automated tests, deployed to staging, monitored application metrics, and finally rolled out the changes gradually using a canary deployment strategy."

---

# Senior Engineer Takeaways

When discussing a Spring Boot 3 migration, focus on:

- **Planning**: Upgrade strategy, dependency analysis, staged rollout.
- **Breaking Changes**: Jakarta migration, Spring Security, Hibernate 6.
- **Testing**: Unit, integration, performance, and regression testing.
- **Operations**: Monitoring, rollback plans, staged deployment.
- **Business Impact**: Reduced risk, improved security, better observability, and long-term maintainability.

A strong interview answer should demonstrate not only **what changed**, but also **how you planned, executed, validated, and safely released** the migration in a production environment.
