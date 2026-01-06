# Deadlocks in Java – SDE-2 Interview Notes

## 1. What is a Deadlock?

**Deadlock** is a situation where two or more threads are blocked forever, each waiting for a resource held by another thread.

Key idea:

* Threads are **waiting indefinitely**
* No progress is possible

---

## 2. Real Backend Scenario

### Order + Inventory System

* Thread T1:

  * Locks **Order**
  * Waits for **Inventory**

* Thread T2:

  * Locks **Inventory**
  * Waits for **Order**

Result: **Circular dependency → Deadlock**

---

## 3. Classic Java Deadlock Example

```java
class DeadlockExample {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    void method1() {
        synchronized (lockA) {
            sleep();
            synchronized (lockB) {
                System.out.println("Method 1");
            }
        }
    }

    void method2() {
        synchronized (lockB) {
            sleep();
            synchronized (lockA) {
                System.out.println("Method 2");
            }
        }
    }
}
```

---

## 4. Coffman Conditions (Must Know)

Deadlock occurs **only if all four conditions are satisfied**:

1. **Mutual Exclusion** – Resource can be held by only one thread
2. **Hold and Wait** – Thread holds one resource and waits for another
3. **No Preemption** – Resource cannot be forcibly taken
4. **Circular Wait** – Circular chain of waiting threads

Breaking **any one** condition prevents deadlock.

---

## 5. Deadlock Prevention Techniques

### 5.1 Lock Ordering (Most Important)

Always acquire locks in the **same global order**.

```java
synchronized (lockA) {
    synchronized (lockB) {
        // safe
    }
}
```

Used heavily in:

* Database transactions
* Distributed locking
* Order–Inventory workflows

---

### 5.2 tryLock with Timeout

```java
if (lock1.tryLock(2, TimeUnit.SECONDS)) {
    try {
        if (lock2.tryLock(2, TimeUnit.SECONDS)) {
            try {
                // critical section
            } finally {
                lock2.unlock();
            }
        }
    } finally {
        lock1.unlock();
    }
}
```

Benefits:

* Avoids infinite waiting
* Improves system availability

---

### 5.3 Reduce Lock Scope

❌ Bad Practice:

```java
synchronized(this) {
    callExternalService();
}
```

✔ Good Practice:

```java
synchronized(this) {
    updateState();
}
callExternalService();
```

---

### 5.4 Use High-Level Concurrency APIs

Prefer:

* `ConcurrentHashMap`
* `AtomicInteger`, `LongAdder`
* `ExecutorService`

Less manual locking → fewer deadlocks.

---

## 6. Deadlock vs Livelock vs Starvation

| Problem    | Meaning                        |
| ---------- | ------------------------------ |
| Deadlock   | Threads stuck forever          |
| Livelock   | Threads active but no progress |
| Starvation | Thread never gets resource     |

---

## 7. Detecting Deadlocks in Java

### JVM Tool

```bash
jstack <pid>
```

Output:

```
Found one Java-level deadlock
```

---

## 8. Interview-Ready One-Liners

* "Deadlock requires all four Coffman conditions."
* "Breaking circular wait prevents deadlock."
* "Lock ordering is the simplest prevention strategy."
* "tryLock with timeout avoids infinite waiting."

---

## 9. Common Interview Questions

**Q: Can synchronized cause deadlock?**
Yes, if locks are acquired in different order.

**Q: Does ConcurrentHashMap cause deadlock?**
No, it avoids circular waits through internal design.

---

## 10. SDE-2 Tip

When answering deadlock questions:

1. Give definition
2. Explain Coffman conditions
3. Show prevention strategy

This structure signals senior-level understanding.
