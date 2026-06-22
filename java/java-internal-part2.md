# Singleton Design Pattern - Complete Study Guide

# Table of Contents

1. What is Singleton?
2. Why Do We Need Singleton?
3. Real-World Examples
4. Characteristics of Singleton
5. Basic Singleton Implementation
6. Problems in Multithreaded Environments
7. Thread-Safe Singleton Solutions
8. Eager Initialization Singleton
9. Synchronized Method Singleton
10. Double Checked Locking (DCL)
11. Bill Pugh Singleton
12. Enum Singleton
13. Breaking Singleton
14. Preventing Singleton Breaks
15. Singleton in Spring Boot
16. Interview Questions
17. Cheat Sheet

---

# 1. What is Singleton?

Singleton is a Creational Design Pattern that ensures:

```text
Only One Object
```

of a class exists throughout the application.

and provides:

```text
Global Access Point
```

to that object.

---

# Definition

```text
Singleton Pattern restricts
object creation to exactly one instance.
```

---

# 2. Why Do We Need Singleton?

Some resources should have only one instance.

Examples:

```text
Database Connection Manager
Logger
Cache Manager
Configuration Manager
Thread Pool Manager
Kafka Producer
```

---

Imagine:

```java
new Logger();
new Logger();
new Logger();
new Logger();
```

Thousands of objects.

Waste of memory.

Instead:

```text
One Shared Instance
```

---

# 3. Real World Examples

---

## Logger

Entire application uses:

```text
One Logger
```

---

## Configuration Manager

Reads:

```text
application.properties
```

once.

Used everywhere.

---

## Cache Manager

```text
Redis Cache
Local Cache
```

Single shared object.

---

## Thread Pool

```java
Executors.newFixedThreadPool(10);
```

Usually managed as singleton.

---

# 4. Characteristics of Singleton

A Singleton class should:

---

## Private Constructor

Prevent:

```java
new Singleton()
```

outside class.

---

## Static Instance

Stores single object.

---

## Public Getter

Provides access.

---

Diagram

```text
Application
      ↓
getInstance()
      ↓
Singleton Object
```

---

# 5. Basic Singleton Implementation

---

## Lazy Initialization

```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {

        if(instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

---

Usage

```java
Singleton s1 =
Singleton.getInstance();

Singleton s2 =
Singleton.getInstance();
```

---

Verification

```java
System.out.println(s1 == s2);
```

Output

```text
true
```

---

# Problem

This implementation is:

```text
NOT Thread Safe
```

---

# 6. Problems in Multithreaded Environments

Most asked interview question.

---

Consider:

```java
if(instance == null)
```

Two threads arrive simultaneously.

---

Thread 1

```java
instance == null
```

TRUE

---

Before creating object

CPU switches.

---

Thread 2

```java
instance == null
```

TRUE

Creates object.

---

Thread 1 resumes.

Creates another object.

---

Result

```text
Two Singleton Objects
```

Singleton broken.

---

Visualization

```text
Thread 1
    ↓
instance == null

Thread 2
    ↓
instance == null

Thread 2
    ↓
new Singleton()

Thread 1
    ↓
new Singleton()
```

---

Output

```java
false
```

for:

```java
s1 == s2
```

---

# 7. Thread Safe Singleton Solutions

Multiple solutions exist.

---

# 8. Eager Initialization Singleton

Create object at startup.

---

Implementation

```java
public class Singleton {

    private static final Singleton INSTANCE =
            new Singleton();

    private Singleton(){}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

---

Why Thread Safe?

Class loading in Java is:

```text
Thread Safe
```

JVM guarantees:

```text
INSTANCE created once.
```

---

Advantages

```text
Simple
Fast
Thread Safe
```

---

Disadvantages

```text
Object created even if never used.
```

---

# 9. Synchronized Method Singleton

---

Implementation

```java
public class Singleton {

    private static Singleton instance;

    private Singleton(){}

    public static synchronized Singleton getInstance() {

        if(instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

---

How It Works

Only one thread enters:

```java
getInstance()
```

at a time.

---

Advantages

```text
Thread Safe
Easy to Understand
```

---

Disadvantages

Every call:

```java
getInstance()
```

acquires lock.

Even after object already exists.

---

Performance suffers.

---

# 10. Double Checked Locking (DCL)

Most important interview implementation.

---

Goal

```text
Lock only once.
```

---

Implementation

```java
public class Singleton {

    private static volatile Singleton instance;

    private Singleton(){}

    public static Singleton getInstance() {

        if(instance == null) {

            synchronized(Singleton.class) {

                if(instance == null) {

                    instance =
                            new Singleton();
                }
            }
        }

        return instance;
    }
}
```

---

Why Double Check?

First Check

```java
if(instance == null)
```

Avoids locking after initialization.

---

Second Check

```java
if(instance == null)
```

Prevents multiple object creation.

---

# Why Volatile Required?

Most asked follow-up.

---

Object creation is not atomic.

Actually:

```java
instance = new Singleton();
```

becomes:

```text
1. Allocate Memory

2. Initialize Object

3. Assign Reference
```

---

JVM may reorder:

```text
1. Allocate Memory

3. Assign Reference

2. Initialize Object
```

---

Another thread may see:

```text
Partially Constructed Object
```

Very dangerous.

---

`volatile` prevents:

```text
Instruction Reordering
```

and ensures:

```text
Memory Visibility
```

---

# Interview Answer

```text
volatile guarantees visibility
and prevents instruction reordering
during singleton initialization.
```

---

# 11. Bill Pugh Singleton

Recommended for interviews.

---

Uses:

```text
Static Inner Class
```

---

Implementation

```java
public class Singleton {

    private Singleton(){}

    private static class Holder {

        private static final Singleton INSTANCE =
                new Singleton();
    }

    public static Singleton getInstance() {

        return Holder.INSTANCE;
    }
}
```

---

How It Works

Outer class loads:

```text
Singleton
```

No object created.

---

When first call happens:

```java
getInstance()
```

JVM loads:

```text
Holder Class
```

---

Creates instance.

---

Benefits

```text
Lazy Loading
Thread Safe
No Synchronization
High Performance
```

---

# 12. Enum Singleton

Most robust implementation.

---

Implementation

```java
public enum Singleton {

    INSTANCE;

    public void print() {
        System.out.println("Hello");
    }
}
```

---

Usage

```java
Singleton.INSTANCE.print();
```

---

Advantages

```text
Thread Safe
Serialization Safe
Reflection Safe
Simple
```

---

Joshua Bloch recommends:

```text
Enum Singleton
```

for most cases.

---

# 13. Breaking Singleton

Many candidates forget this.

---

Singleton can be broken using:

```text
Reflection
Serialization
Cloning
Multiple Class Loaders
```

---

# Reflection Attack

Private constructor accessed.

---

Example

```java
Constructor<Singleton> c =
Singleton.class.getDeclaredConstructor();

c.setAccessible(true);

Singleton s1 =
Singleton.getInstance();

Singleton s2 =
c.newInstance();
```

---

Result

```text
Two Objects
```

Singleton broken.

---

# Serialization Attack

Serialize object.

Deserialize object.

---

Creates:

```text
New Object
```

---

Example

```java
ObjectInputStream
```

returns different instance.

---

# Cloning Attack

```java
clone()
```

creates new object.

---

Singleton broken.

---

# 14. Preventing Singleton Breaks

---

## Reflection Protection

```java
private Singleton() {

    if(instance != null) {
        throw new RuntimeException(
                "Singleton already created");
    }
}
```

---

## Serialization Protection

Implement:

```java
readResolve()
```

---

Example

```java
protected Object readResolve() {
    return instance;
}
```

---

## Cloning Protection

```java
@Override
protected Object clone()
throws CloneNotSupportedException {

    throw new CloneNotSupportedException();
}
```

---

# Best Solution

```text
Enum Singleton
```

Automatically protects against:

```text
Reflection
Serialization
Multiple Instances
```

---

# 15. Singleton in Spring Boot

Many candidates wrongly say:

```text
Spring Bean = Singleton Pattern
```

Not exactly.

---

Default Scope

```java
@Component
public class UserService {
}
```

---

Spring creates:

```text
One Bean Per Application Context
```

called:

```text
Singleton Scope
```

---

Difference

Traditional Singleton

```text
One Object Per JVM
```

---

Spring Singleton

```text
One Bean Per Spring Container
```

---

# 16. Common Interview Questions

---

## What is Singleton?

```text
A design pattern that ensures
only one instance exists.
```

---

## Why Constructor Private?

```text
Prevent external object creation.
```

---

## Why Basic Singleton Fails?

```text
Race Condition.
```

---

## Why Volatile Used?

```text
Prevent instruction reordering
and guarantee visibility.
```

---

## Best Singleton Implementation?

Most Practical:

```text
Bill Pugh Singleton
```

---

Most Robust:

```text
Enum Singleton
```

---

## Difference Between Lazy and Eager Singleton?

Lazy:

```text
Object created when needed.
```

Eager:

```text
Object created at startup.
```

---

# 17. Cheat Sheet

## Basic Singleton

```java
private static Singleton instance;
```

Not Thread Safe.

---

## Eager Singleton

```java
private static final Singleton INSTANCE
```

Thread Safe.

---

## Synchronized Singleton

```java
synchronized getInstance()
```

Thread Safe.

Slow.

---

## Double Checked Locking

```java
volatile
+
synchronized
```

Thread Safe.

High Performance.

---

## Bill Pugh

```java
Static Inner Class
```

Lazy.

Thread Safe.

Recommended.

---

## Enum Singleton

```java
INSTANCE
```

Best Protection.

Most Robust.

---

# Golden Interview Answer

> A Singleton ensures only one instance of a class exists throughout the application. A naive lazy-loaded implementation fails in multithreaded environments because multiple threads can simultaneously create separate instances. This can be solved using synchronized methods, Double-Checked Locking with `volatile`, Bill Pugh's static inner class approach, or Enum-based Singleton. For modern Java applications, Bill Pugh Singleton is typically preferred for performance and lazy loading, while Enum Singleton provides the strongest protection against reflection and serialization attacks.
