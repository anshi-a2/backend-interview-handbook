# Thread Pools & ExecutorService – SDE-2 Interview Notes

## 1. Why not create threads manually?

Creating threads directly using `new Thread()` is discouraged in backend systems.

### Problems with manual thread creation

* Thread creation is expensive
* No upper bound on threads
* Risk of **OutOfMemoryError**
* No task queue or lifecycle management

```java
new Thread(() -> processOrder()).start(); // ❌ Not recommended
```

Backend servers always use **thread pools**.

---

## 2. What is a Thread Pool?

A **thread pool** is a collection of reusable worker threads that execute submitted tasks.

### Benefits

* Thread reuse (performance)
* Controlled concurrency
* Task queueing
* Better resource utilization

---

## 3. Executor Framework Overview

Java provides the **Executor Framework** for thread pool management.

```
Executor
  └── ExecutorService
        └── ScheduledExecutorService
```

---

## 4. ExecutorService – Basic Usage

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

executor.submit(() -> processOrder());

executor.shutdown();
```

### What happens internally?

* Task is submitted to a queue
* One thread from the pool executes it
* Thread is reused for next task

---

## 5. Types of Thread Pools (Must Know)

### 5.1 Fixed Thread Pool

```java
Executors.newFixedThreadPool(10);
```

* Fixed number of threads
* Uses **unbounded queue**
* Risk of memory growth if tasks pile up

Used when workload is predictable.

---

### 5.2 Cached Thread Pool

```java
Executors.newCachedThreadPool();
```

* Creates threads as needed
* Reuses idle threads
* ❌ Can create unlimited threads

Not recommended for backend services.

---

### 5.3 Single Thread Executor

```java
Executors.newSingleThreadExecutor();
```

* Single worker thread
* Tasks executed sequentially

Used when order of execution must be preserved.

---

### 5.4 Scheduled Thread Pool

```java
Executors.newScheduledThreadPool(5);
```

* Supports delayed and periodic execution
* Replacement for `Timer`

Used for cleanup jobs, retries, and monitoring.

---

## 6. Why Executors Factory is discouraged in Production

`Executors.newFixedThreadPool()` and others use:

* **Unbounded task queues**

This can lead to:

* High memory usage
* OutOfMemoryError under load

👉 Production systems should use **ThreadPoolExecutor**.

---

## 7. ThreadPoolExecutor (Production-Grade Control)

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    10,                       // corePoolSize
    20,                       // maxPoolSize
    60, TimeUnit.SECONDS,     // keepAliveTime
    new ArrayBlockingQueue<>(100),
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

---

## 8. ThreadPoolExecutor Parameters Explained

| Parameter       | Description                             |
| --------------- | --------------------------------------- |
| corePoolSize    | Minimum number of threads kept alive    |
| maxPoolSize     | Maximum threads allowed                 |
| workQueue       | Queue holding waiting tasks             |
| keepAliveTime   | Time extra threads stay alive when idle |
| rejectionPolicy | Strategy when pool is full              |

---

## 9. Rejection Policies (Very Important)

| Policy              | Behavior                          |
| ------------------- | --------------------------------- |
| AbortPolicy         | Throws RejectedExecutionException |
| CallerRunsPolicy    | Calling thread executes task      |
| DiscardPolicy       | Silently drops task               |
| DiscardOldestPolicy | Drops oldest queued task          |

**CallerRunsPolicy** is preferred for backend systems because it provides backpressure.

---

## 10. Real Backend Example (Order Service)

```java
@PostMapping("/orders")
public ResponseEntity<?> createOrder() {
    executor.submit(() -> orderService.process());
    return ResponseEntity.accepted().build();
}
```

Benefits:

* Non-blocking API
* Controlled concurrency
* Graceful degradation under load

---

## 11. ExecutorService vs ForkJoinPool

| ExecutorService      | ForkJoinPool        |
| -------------------- | ------------------- |
| Independent tasks    | Recursive tasks     |
| IO-bound workloads   | CPU-bound workloads |
| Predictable behavior | Work-stealing       |

---

## 12. Interview One-Liners

* "Never create threads manually in backend services."
* "Prefer ThreadPoolExecutor over Executors factory methods."
* "Unbounded queues can cause OutOfMemoryError."
* "CallerRunsPolicy provides natural backpressure."

---

## 13. Common Interview Questions

**Q: How many threads should a thread pool have?**

* CPU-bound tasks → number of cores
* IO-bound tasks → more than cores

**Q: What happens when the queue is full?**

* Rejection policy is triggered

---

## 14. SDE-2 Tip

When explaining thread pools in interviews:

1. Explain why manual threads are bad
2. Describe ExecutorService
3. Show ThreadPoolExecutor configuration

This signals production-level experience.
