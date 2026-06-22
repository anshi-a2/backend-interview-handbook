# Java Concurrency, Thread Safety & Extensibility - Complete Study Guide

# Table of Contents

1. What is Concurrency?
2. Process vs Thread
3. Thread Lifecycle
4. Creating Threads in Java
5. Runnable vs Callable
6. Thread Pools
7. Executor Framework
8. Synchronization
9. Race Conditions
10. What is Thread Safety?
11. Thread-Safe Collections
12. Volatile Keyword
13. Atomic Variables
14. Locks and ReentrantLock
15. ConcurrentHashMap Internals
16. Deadlock
17. Livelock & Starvation
18. Extensibility in Java
19. Open Closed Principle (OCP)
20. Extensible Design using Interfaces
21. Extensible Design using Abstract Classes
22. Strategy Pattern
23. Dependency Injection
24. Interview Questions
25. Cheat Sheet

---

# 1. What is Concurrency?

Concurrency means:

```text
Multiple tasks making progress
during overlapping time periods.
```

---

Example

```text
Download File
Play Music
Receive Notifications
```

All appear to run simultaneously.

---

In Java:

```text
Concurrency = Multiple Threads
```

working together.

---

# Concurrency vs Parallelism

---

## Concurrency

```text
Task A
Task B
Task A
Task B
```

Single CPU switching between tasks.

---

## Parallelism

```text
Core 1 → Task A

Core 2 → Task B
```

Actual simultaneous execution.

---

Interview Answer:

```text
Concurrency is about dealing with
multiple tasks at once.

Parallelism is about executing
multiple tasks simultaneously.
```

---

# 2. Process vs Thread

| Feature | Process | Thread |
|----------|----------|----------|
| Memory | Separate | Shared |
| Creation Cost | High | Low |
| Communication | IPC | Shared Memory |
| Isolation | Strong | Weak |

---

Example

```text
Chrome Browser
```

Process

```text
Tab 1 Thread
Tab 2 Thread
Tab 3 Thread
```

---

# 3. Thread Lifecycle

```text
NEW
 ↓
RUNNABLE
 ↓
RUNNING
 ↓
BLOCKED / WAITING
 ↓
TERMINATED
```

---

Example

```java
Thread t = new Thread();
```

State:

```text
NEW
```

---

```java
t.start();
```

State:

```text
RUNNABLE
```

---

JVM Scheduler selects thread.

```text
RUNNING
```

---

# 4. Creating Threads

---

## Method 1: Extend Thread

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("Running");
    }
}
```

---

Usage

```java
MyThread t = new MyThread();
t.start();
```

---

## Method 2: Implement Runnable

Preferred.

```java
class Task implements Runnable {

    @Override
    public void run() {
        System.out.println("Running");
    }
}
```

---

Usage

```java
Thread t =
new Thread(new Task());

t.start();
```

---

Why Preferred?

```text
Supports Inheritance
Better Design
Loose Coupling
```

---

# 5. Runnable vs Callable

---

## Runnable

```java
void run()
```

No return value.

---

## Callable

```java
V call()
```

Returns value.

Can throw exceptions.

---

Example

```java
Callable<Integer> task =
() -> 100;
```

---

Usage

```java
Future<Integer> future =
executor.submit(task);

future.get();
```

---

# 6. Thread Pools

Bad Practice

```java
new Thread()
new Thread()
new Thread()
```

Thousands of threads.

---

Problems

```text
Memory Overhead
Context Switching
CPU Waste
```

---

Solution

```text
Thread Pool
```

---

Example

```java
ExecutorService executor =
Executors.newFixedThreadPool(10);
```

---

Benefits

```text
Thread Reuse
Better Performance
Controlled Resources
```

---

# 7. Executor Framework

Most production systems use:

```java
ExecutorService
```

---

Example

```java
ExecutorService executor =
Executors.newFixedThreadPool(5);

executor.submit(() ->
System.out.println("Task"));

executor.shutdown();
```

---

Advantages

```text
Task Management
Thread Pooling
Future Support
Scheduling
```

---

# 8. Synchronization

Used to prevent:

```text
Race Conditions
```

---

Example

```java
class Counter {

    int count = 0;

    void increment() {
        count++;
    }
}
```

---

Two threads:

```text
T1
T2
```

may update simultaneously.

Incorrect result.

---

Solution

```java
synchronized void increment() {
    count++;
}
```

---

# 9. Race Condition

Occurs when:

```text
Multiple threads access
shared mutable data
without synchronization.
```

---

Example

```java
count++
```

actually:

```text
Read
Modify
Write
```

---

Two threads:

```text
Read 5
Read 5

Write 6
Write 6
```

Expected:

```text
7
```

Actual:

```text
6
```

Race condition.

---

# 10. What is Thread Safety?

Definition

```text
A class is thread-safe if it behaves
correctly when accessed by multiple
threads simultaneously.
```

---

Thread Safe Examples

```java
String
ConcurrentHashMap
AtomicInteger
```

---

Not Thread Safe

```java
HashMap
ArrayList
StringBuilder
```

---

# Ways to Achieve Thread Safety

```text
Synchronization
Immutable Objects
Atomic Classes
Concurrent Collections
Locks
```

---

# 11. Thread Safe Collections

---

## HashMap

```java
Not Thread Safe
```

---

## Hashtable

```java
Thread Safe
```

Uses:

```text
Method-Level Synchronization
```

Slow.

---

## ConcurrentHashMap

```java
Thread Safe
```

High performance.

---

Preferred in modern applications.

---

# 12. Volatile Keyword

Guarantees:

```text
Visibility
```

between threads.

---

Example

```java
volatile boolean running = true;
```

---

Thread 1

```java
running = false;
```

---

Thread 2 immediately sees change.

---

Without volatile

```text
CPU Cache may hold stale value.
```

---

# What Volatile Does NOT Provide

```text
Atomicity
```

---

Wrong

```java
volatile int count;

count++;
```

Still unsafe.

---

# 13. Atomic Variables

Used for lock-free thread safety.

---

Example

```java
AtomicInteger count =
new AtomicInteger(0);
```

---

Increment

```java
count.incrementAndGet();
```

---

Internally uses:

```text
CAS
(Compare And Swap)
```

---

Advantages

```text
Fast
Thread Safe
No Locking
```

---

# 14. Locks and ReentrantLock

Alternative to synchronized.

---

Example

```java
Lock lock =
new ReentrantLock();

lock.lock();

try {
    // critical section
}
finally {
    lock.unlock();
}
```

---

Advantages

```text
Try Lock
Timeout
Interruptibility
Fair Locking
```

---

More flexible than synchronized.

---

# 15. ConcurrentHashMap Internals

Java 8+

---

Uses:

```text
CAS
Synchronized Buckets
Node Locking
```

---

Read Operations

```text
Mostly Lock Free
```

---

Write Operations

```text
Lock only affected bucket.
```

---

Advantages

```text
High Concurrency
Better Throughput
```

---

Compared to Hashtable

```text
Much Faster
```

---

# 16. Deadlock

Most asked concurrency question.

---

Example

Thread 1

```java
lock(A)
lock(B)
```

---

Thread 2

```java
lock(B)
lock(A)
```

---

Result

```text
T1 waiting for B
T2 waiting for A
```

Forever.

---

Deadlock.

---

Conditions

```text
Mutual Exclusion
Hold and Wait
No Preemption
Circular Wait
```

---

# Prevention

Acquire locks:

```text
Same Order
```

Always.

---

# 17. Livelock & Starvation

---

## Livelock

Threads keep reacting.

No progress.

---

Example

```text
You move left.
I move right.

You move right.
I move left.
```

Nobody passes.

---

## Starvation

One thread never gets CPU.

---

Example

```text
High Priority Threads
```

always dominate.

---

# 18. What is Extensibility?

Definition

```text
Ability to add new functionality
without modifying existing code.
```

---

Goal

```text
Easy Maintenance
Easy Enhancement
Minimal Changes
```

---

Bad Design

```java
if(type.equals("UPI"))

else if(type.equals("CARD"))

else if(type.equals("NETBANKING"))
```

Every new payment method:

```text
Modify Existing Code
```

---

Not extensible.

---

# 19. Open Closed Principle (OCP)

Most important extensibility principle.

---

Definition

```text
Open For Extension
Closed For Modification
```

---

Bad

```java
if(paymentType.equals("UPI"))
```

---

Good

```java
Payment payment;
payment.pay();
```

Add new implementation.

No changes.

---

# 20. Extensible Design Using Interfaces

---

Interface

```java
interface Payment {

    void pay();
}
```

---

Implementation

```java
class UpiPayment
implements Payment {

    public void pay() {}
}
```

---

Add new payment type.

```java
class CryptoPayment
implements Payment {

    public void pay() {}
}
```

---

Existing code unchanged.

---

# 21. Extensible Design Using Abstract Classes

---

Example

```java
abstract class Vehicle {

    abstract void start();

    void stop() {}
}
```

---

New vehicle

```java
class Car extends Vehicle
```

---

New vehicle

```java
class Bike extends Vehicle
```

---

Framework easily extended.

---

# 22. Strategy Pattern

Most common extensibility pattern.

---

Interface

```java
interface PaymentStrategy {

    void pay();
}
```

---

Implementations

```java
CardPayment
UPIPayment
CryptoPayment
```

---

Usage

```java
paymentStrategy.pay();
```

---

Add new strategy.

No code modification.

---

# 23. Dependency Injection

Bad

```java
Payment payment =
new CardPayment();
```

---

Good

```java
Payment payment;
```

Injected externally.

---

Benefits

```text
Loose Coupling
Testability
Extensibility
```

---

Spring heavily uses DI.

---

# 24. Common Interview Questions

---

## Difference Between Concurrency and Parallelism?

```text
Concurrency = Multiple tasks progressing.

Parallelism = Multiple tasks executing simultaneously.
```

---

## What is Thread Safety?

```text
Correct behavior under
concurrent access.
```

---

## Difference Between Volatile and Synchronization?

Volatile

```text
Visibility Only
```

Synchronization

```text
Visibility + Atomicity
```

---

## Why ConcurrentHashMap?

```text
Thread Safe
High Performance
Fine-Grained Locking
```

---

## What is Extensibility?

```text
Ability to add features
without changing existing code.
```

---

# 25. Interview Cheat Sheet

## Concurrency

```text
Multiple Tasks
Shared Resources
Threads
```

---

## Thread Safety

Achieved using:

```text
Synchronization
Locks
Atomic Classes
Immutable Objects
Concurrent Collections
```

---

## Volatile

```text
Visibility
Not Atomicity
```

---

## AtomicInteger

```text
Thread Safe Counter
CAS Based
```

---

## ConcurrentHashMap

```text
Thread Safe HashMap
High Concurrency
```

---

## Deadlock

```text
Thread A waiting for B
Thread B waiting for A
```

---

## Extensibility

```text
Interfaces
Abstract Classes
Strategy Pattern
Dependency Injection
OCP
```

---

# Golden Interview Answer

> Concurrency in Java enables multiple threads to make progress simultaneously, while thread safety ensures correct behavior when multiple threads access shared data. Thread safety can be achieved through synchronization, locks, atomic variables, immutable objects, and concurrent collections like ConcurrentHashMap. Extensibility refers to designing systems that can accommodate new functionality without modifying existing code, typically achieved using interfaces, abstract classes, dependency injection, and design patterns such as Strategy, following the Open-Closed Principle.
