# ConcurrentHashMap – SDE-2 Interview Notes

This document summarizes **HashMap problems**, **ConcurrentHashMap internals**, and key concurrency concepts at **SDE-2 interview depth**.

---

## 1. Problems with HashMap in Multi-threaded Environment

`HashMap` is **not thread-safe**. When accessed concurrently without synchronization, it can lead to:

* **Data loss** (lost updates)
* **Internal structure corruption**
* **Infinite loops** during resize (especially pre-Java 8)

### Why this happens

* No synchronization on `put`, `get`, or resize
* Multiple threads can modify buckets simultaneously
* Resize operation is not atomic

➡️ **Conclusion:** HashMap should never be used for concurrent writes.

---

## 2. What is ConcurrentHashMap?

`ConcurrentHashMap` is a thread-safe alternative to `HashMap` designed for **high concurrency**.

Key goals:

* Allow **multiple concurrent reads**
* Allow **multiple concurrent writes**
* Avoid locking the entire map

---

## 3. ConcurrentHashMap Internals (Java 8+)

Internally uses:

* `Node[] table` (buckets)
* **CAS (Compare-And-Swap)** for atomic updates
* `volatile` variables for visibility
* **Fine-grained locking** (bucket-level)

No global map-level lock is used.

---

## 4. Bucket-Level Locking

### What is a Bucket?

A bucket is a single index in the internal `Node[] table`.

Each bucket may contain:

* A single node
* A linked list
* A Red-Black tree (for high collision cases)

### Bucket-Level Locking Explained

Instead of locking the entire map:

* Only the **specific bucket** being modified is locked
* Other buckets remain accessible to other threads

### How `put()` Works

1. Compute hash
2. Locate bucket index
3. **If bucket is empty** → CAS insert (no lock)
4. **If bucket has nodes** → synchronize on bucket head

➡️ This allows parallel writes to different buckets.

---

## 5. Lock-Free Reads

### Why `get()` Does Not Use Locks

* Buckets and nodes are accessed via `volatile` references
* Ensures **visibility** of latest writes
* No structural modification during read

### Benefits

* Extremely fast reads
* No thread contention
* Ideal for read-heavy workloads

➡️ `get()` is **lock-free** but **thread-safe**.

---

## 6. CAS (Compare-And-Swap) vs Synchronization

### CAS

* Optimistic, non-blocking
* Used when inserting into empty buckets
* Implemented at CPU level

Example:

```text
If value == expected → update
Else → retry
```

### Synchronization

* Used when bucket already contains nodes
* Locks only the bucket head
* Prevents race conditions during collision handling

### Why Both Are Used

| Scenario         | Mechanism    |
| ---------------- | ------------ |
| Empty bucket     | CAS          |
| Collision bucket | synchronized |

➡️ Best balance between **safety and performance**.

---

## 7. Interview-Ready Summary

* HashMap is unsafe for concurrent writes
* ConcurrentHashMap avoids global locking
* Uses bucket-level locking for writes
* Uses volatile reads for lock-free gets
* CAS improves performance in uncontended scenarios

> **One-liner:**
> ConcurrentHashMap achieves thread safety using CAS, volatile reads, and bucket-level synchronization instead of locking the entire map.

---

## 8. Common Interview Questions

**Q: Why not use Collections.synchronizedMap?**
A: It locks the entire map, causing high contention.

**Q: Can two threads write simultaneously?**
A: Yes, if they write to different buckets.

**Q: Why is get() lock-free?**
A: Because volatile reads guarantee visibility without synchronization.

---

✔ Prepared for SDE-2 backend interviews
