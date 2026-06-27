# 02_Java_17_to_21_Upgrade_Guide.md

> **Goal**
>
> This guide explains everything needed to upgrade an application from **Java 17 to Java 21**. It covers the migration strategy, new language features, JVM improvements, Virtual Threads, Structured Concurrency, Garbage Collection improvements, production migration, common issues, and interview questions.
>
> **Java 21 is the latest Long-Term Support (LTS) release**, making it the recommended upgrade path for enterprise applications.

---

# Table of Contents

1. Why Upgrade to Java 21?
2. High-Level Migration Strategy
3. Major Language Features
4. Virtual Threads (Project Loom)
5. Structured Concurrency
6. Scoped Values
7. Pattern Matching Enhancements
8. Sequenced Collections
9. Performance Improvements
10. Garbage Collector Improvements
11. JVM Improvements
12. Production Migration Strategy
13. Common Migration Issues
14. Best Practices
15. Frequently Asked Interview Questions

---

# 1. Why Upgrade to Java 21?

Java 21 is an **LTS (Long-Term Support)** release with several improvements over Java 17.

## Benefits

- Long-term support
- Better application performance
- Improved JVM optimizations
- Virtual Threads
- Structured Concurrency
- Pattern Matching improvements
- Sequenced Collections
- Better Garbage Collection
- Improved developer productivity
- Better cloud-native performance

---

## Why did companies migrate?

- Java 21 is the latest LTS release.
- Better scalability for microservices.
- Improved concurrency using Virtual Threads.
- Reduced infrastructure cost.
- Better support from Spring Boot 3.x.
- Better performance with modern JVM optimizations.

---

# 2. High-Level Migration Strategy

A safe migration flow

```text
Upgrade JDK

↓

Update Maven / Gradle

↓

Update Docker Image

↓

Compile Project

↓

Fix Deprecated APIs

↓

Update Third-party Libraries

↓

Run Unit Tests

↓

Run Integration Tests

↓

Performance Testing

↓

Deploy to Staging

↓

Canary Deployment

↓

Production
```

Never upgrade Java in production without validating framework compatibility.

---

# 3. Major Language Features

## Record Patterns

Java 17

```java
if(obj instanceof Employee){

}
```

Java 21

```java
if(obj instanceof Employee(String name,int age)){

}
```

Benefits

- Less boilerplate
- Better readability
- Cleaner code

---

## Pattern Matching for switch

Old

```java
if(obj instanceof String){

}
else if(obj instanceof Integer){

}
```

New

```java
switch(obj){

case String s ->

case Integer i ->

}
```

Advantages

- Cleaner
- Easier maintenance
- Compiler checks exhaustiveness

---

## Switch Improvements

Supports

- Multiple labels
- Pattern Matching
- Better readability
- Less nesting

---

# 4. Virtual Threads (Project Loom)

One of the biggest Java 21 features.

---

## Traditional Threads

```
Request

↓

Platform Thread

↓

OS Thread
```

Every Java thread maps to an Operating System thread.

Creating thousands of OS threads is expensive.

Problems

- High memory usage
- Context switching overhead
- Thread pool tuning
- Scalability limits

---

## Virtual Threads

```
Request

↓

Virtual Thread

↓

JVM Scheduler

↓

Carrier Thread

↓

OS Thread
```

Thousands (or even millions) of Virtual Threads can share a much smaller number of platform (carrier) threads.

---

## Platform Thread vs Virtual Thread

| Platform Thread | Virtual Thread |
|----------------|----------------|
| Heavyweight | Lightweight |
| Expensive | Cheap |
| Limited | Millions possible |
| OS Managed | JVM Managed |
| Higher memory | Lower memory |

---

## Internal Architecture

```
HTTP Request

↓

Virtual Thread

↓

JVM Scheduler

↓

Carrier Thread

↓

CPU
```

The JVM schedules Virtual Threads onto a pool of carrier threads.

---

## Why are Virtual Threads Faster?

Traditional

```
10000 Requests

↓

10000 Threads

↓

Huge Memory
```

Virtual Threads

```
10000 Requests

↓

10000 Virtual Threads

↓

Few Carrier Threads
```

Less memory

Less context switching

Better scalability

---

## Where should you use Virtual Threads?

Good for

- REST APIs
- Database Calls
- File I/O
- HTTP Clients
- Kafka Consumers
- Messaging
- Microservices

Avoid for

- CPU intensive calculations
- Long-running compute-heavy tasks

---

## Virtual Thread Example

```java
Thread.startVirtualThread(() -> {

});
```

Executor

```java
Executors.newVirtualThreadPerTaskExecutor();
```

---

## Thread Pinning

Virtual Threads can become **pinned** when they cannot be unmounted from a carrier thread.

Common causes

- `synchronized` blocks
- Native code
- Certain blocking operations

Pinned threads reduce scalability.

Prefer

```
ReentrantLock
```

instead of excessive synchronized blocks when appropriate.

---

# 5. Structured Concurrency

Introduced to simplify concurrent programming.

Without

```
Task-1

Task-2

Task-3

↓

Manual Join
```

With Structured Concurrency

```
Parent Task

↓

Child Tasks

↓

Automatic Coordination

↓

Return
```

Benefits

- Easier cancellation
- Better error propagation
- Cleaner lifecycle management

> Note: Structured Concurrency is a preview feature in Java 21 and must be enabled explicitly.

---

# 6. Scoped Values

Alternative to ThreadLocal.

Traditional

```
ThreadLocal

↓

Memory Leak Risk

↓

Manual Cleanup
```

Scoped Values

```
Scoped Value

↓

Immutable

↓

Automatically Cleared
```

Benefits

- Safer
- Immutable
- Better with Virtual Threads

> Scoped Values are also a preview feature in Java 21.

---

# 7. Pattern Matching Enhancements

Java 21 expands Pattern Matching.

Example

```java
switch(obj){

case Employee e ->

case Manager m ->

}
```

Benefits

- Cleaner business logic
- Reduced casting
- Better readability

---

# 8. Sequenced Collections

New interfaces

```
SequencedCollection

SequencedMap

SequencedSet
```

New operations

```
getFirst()

getLast()

addFirst()

addLast()
```

Benefits

- Consistent APIs
- Easier collection manipulation

---

# 9. Performance Improvements

Java 21 provides JVM optimizations.

Areas improved

- Startup time
- Memory management
- JIT compilation
- Thread scheduling
- Throughput
- CPU utilization

Applications usually see improvements without code changes, though results depend on workload.

---

# 10. Garbage Collector Improvements

Java 21 continues improvements across multiple collectors.

## G1 GC

- Better pause-time prediction
- Improved memory reclamation
- Default choice for most applications

---

## ZGC

- Extremely low pause times
- Suitable for very large heaps
- Improved stability

---

## Shenandoah

- Low latency
- Concurrent garbage collection

---

## Which GC should you use?

| GC | Recommended For |
|----|------------------|
| G1 | General applications |
| ZGC | Large memory systems requiring low pauses |
| Shenandoah | Low latency workloads |

---

# 11. JVM Improvements

Java 21 includes improvements in

- Class loading
- CDS (Class Data Sharing)
- JIT compilation
- Memory allocation
- Thread scheduling
- Runtime optimizations

Benefits

- Faster startup
- Better CPU utilization
- Better container performance

---

# 12. Production Migration Strategy

Safe rollout

```text
Development

↓

Upgrade JDK

↓

Compile

↓

Fix Warnings

↓

Upgrade Libraries

↓

Unit Tests

↓

Integration Tests

↓

Performance Testing

↓

Staging

↓

Canary

↓

Production
```

---

## Things to Verify

- Docker base image
- CI/CD pipeline
- JVM options
- Monitoring tools
- Third-party libraries
- Spring Boot compatibility

---

# 13. Common Migration Issues

## Unsupported Libraries

Some libraries may not yet support Java 21.

Solution

Upgrade dependencies.

---

## Reflection Issues

Older frameworks using deep reflection may fail.

Solution

Upgrade framework versions.

---

## Deprecated APIs

Some APIs are removed or deprecated.

Review compiler warnings carefully.

---

## Docker Images

Old

```
openjdk:17
```

New

```
eclipse-temurin:21
```

Verify that your production base image uses Java 21.

---

## Build Tool Compatibility

Upgrade

- Maven plugins
- Gradle plugins
- JaCoCo
- SpotBugs
- Checkstyle

if required.

---

# 14. Best Practices

✔ Upgrade dependencies before upgrading the JDK.

✔ Validate Spring Boot compatibility.

✔ Test with production-like workloads.

✔ Benchmark latency and throughput.

✔ Monitor GC after deployment.

✔ Evaluate Virtual Threads only for I/O-bound workloads.

✔ Keep preview features disabled in production unless there is a clear business need and your organization approves their use.

✔ Maintain a rollback plan.

---

# 15. Frequently Asked Interview Questions

## Q1. Why did you upgrade from Java 17 to Java 21?

**Answer**

Java 21 is the latest LTS release. It provides improved JVM performance, better concurrency with Virtual Threads, modern language features, enhanced garbage collection, and long-term vendor support.

---

## Q2. What is the biggest feature in Java 21?

**Answer**

Virtual Threads (Project Loom) are the most significant addition because they allow applications to handle a very large number of concurrent I/O-bound tasks with far fewer operating system threads.

---

## Q3. What is the difference between Platform Threads and Virtual Threads?

| Platform Thread | Virtual Thread |
|----------------|----------------|
| Managed by OS | Managed by JVM |
| Heavyweight | Lightweight |
| Expensive | Cheap |
| Limited count | Millions possible |

---

## Q4. When should you use Virtual Threads?

**Use for**

- REST APIs
- Database operations
- File I/O
- HTTP calls
- Messaging systems
- Microservices

**Avoid for**

- CPU-intensive algorithms
- Long-running computation

---

## Q5. What are Structured Concurrency and Scoped Values?

**Answer**

Structured Concurrency helps manage related concurrent tasks as a single unit with simpler cancellation and error handling. Scoped Values provide an immutable alternative to `ThreadLocal` and work well with Virtual Threads. Both are preview features in Java 21.

---

## Q6. Did you face any migration issues?

Typical answers

- Updating third-party libraries
- Docker image upgrades
- Build plugin compatibility
- Reflection issues
- Performance regression testing

---

## Q7. How did you ensure a safe migration?

**Answer**

We upgraded the JDK in a separate branch, updated dependencies, fixed compiler warnings, executed unit and integration tests, benchmarked performance, validated staging deployments, and performed a gradual canary rollout while monitoring application metrics.

---

# Senior Engineer Takeaways

During a Java 21 migration discussion, emphasize:

- **Planning**: Dependency analysis, compatibility checks, phased rollout.
- **Performance**: JVM improvements, GC behavior, Virtual Threads.
- **Reliability**: Comprehensive testing and rollback strategy.
- **Business Value**: Better scalability, improved concurrency, and long-term support.
- **Practical Adoption**: Adopt Virtual Threads where they provide value instead of replacing every existing thread pool blindly.

A senior-level interview answer should explain **why** the migration was done, **how** it was executed safely, **what challenges were encountered**, and **how success was validated** in production.
