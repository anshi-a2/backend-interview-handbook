# volatile vs synchronized (Java) – SDE-2 Interview Notes

## 1. What problem do they solve?

Both `volatile` and `synchronized` deal with **concurrency issues** in multi-threaded programs, but they solve **different aspects** of the problem.

Concurrency problems mainly come from:

* **Visibility issues** (one thread cannot see updates from another)
* **Atomicity issues** (operations are partially executed and interleaved)
* **Ordering issues** (instructions executed out of order)

---

## 2. volatile – What it guarantees

### ✅ Guarantees provided by `volatile`

1. **Visibility**

   * When a thread writes to a `volatile` variable, the updated value is immediately visible to all other threads.
   * Reads always fetch the latest value from main memory.

2. **Ordering (happens-before)**

   * A write to a volatile variable happens-before every subsequent read of that variable.

### ❌ What `volatile` does NOT guarantee

* **No atomicity**
  Operations like `count++` are NOT atomic.

Example:

```java
volatile int count = 0;
count++; // read → increment → write (3 steps)
```

Multiple threads can interleave these steps and cause incorrect results.

---

## 3. synchronized – What it guarantees

### ✅ Guarantees provided by `synchronized`

1. **Mutual exclusion (locking)**

   * Only one thread can execute the synchronized block/method at a time.

2. **Visibility**

   * Changes made by one thread are visible to the next thread that acquires the same lock.

3. **Atomicity**

   * Compound operations are executed as a single atomic unit.

Example:

```java
synchronized void increment() {
    count++;
}
```

---

## 4. Key Difference Summary

| Aspect      | volatile    | synchronized          |
| ----------- | ----------- | --------------------- |
| Visibility  | ✅ Yes       | ✅ Yes                 |
| Atomicity   | ❌ No        | ✅ Yes                 |
| Locking     | ❌ No        | ✅ Yes                 |
| Performance | Very fast   | Slower due to locking |
| Use case    | State flags | Critical sections     |

---

## 5. When to use volatile (Very Important)

Use `volatile` when:

* There is **one writer and multiple readers**
* No compound operations are involved
* Variable represents a **state flag**

### Common examples:

```java
volatile boolean isRunning = true;
volatile boolean shutdownRequested;
```

---

## 6. When volatile is NOT enough

❌ Counters
❌ Accumulators
❌ Read-modify-write operations

Example (WRONG):

```java
volatile int activeUsers;
activeUsers++; // NOT thread-safe
```

Correct alternatives:

* `synchronized`
* `AtomicInteger`
* `LongAdder`

---

## 7. Interview-Ready One-Liners

* "volatile guarantees visibility, not atomicity."
* "Use volatile for state flags, not counters."
* "synchronized provides both visibility and atomicity via locking."

---

## 8. Typical Interview Follow-ups

**Q: Why is volatile faster than synchronized?**
A: Because it avoids locking and context switching.

**Q: Can volatile replace synchronized?**
A: No, only when atomicity is not required.

**Q: Is volatile thread-safe?**
A: Only for simple read/write operations.

---

## 9. SDE-2 Interview Tip

If interviewer asks:

> "Why not use volatile everywhere?"

Answer:

> "Because volatile cannot protect compound operations, which require atomicity."
