# HashMap vs Map vs ConcurrentHashMap - Complete Deep Dive

# 1. First Understand: What is a Map?

Map is an Interface.

```java
public interface Map<K,V>
```

It defines the contract:

```java
put(K key, V value)
get(K key)
remove(K key)
containsKey(K key)
size()
```

A Map stores:

```text
Key -> Value
```

Example:

```java
Map<String,Integer> map = new HashMap<>();

map.put("A",100);
map.put("B",200);
```

Internally, Map itself stores nothing.

It only defines behavior.

Think:

```text
Map = Blueprint

HashMap = Implementation
ConcurrentHashMap = Implementation
TreeMap = Implementation
Hashtable = Implementation
```

---

# Collection Hierarchy

```text
Iterable
    |
Collection
    |
--------------------------------
|              |              |
List           Set          Queue

Map (Separate Hierarchy)
```

Map is NOT part of Collection hierarchy.

---

# Why HashMap Exists

Without hashing:

```text
A -> 100
B -> 200
C -> 300
```

Searching requires:

```text
A -> B -> C
```

Complexity:

O(n)

HashMap converts:

```text
Key
 ↓
Hash Function
 ↓
Bucket Index
```

so lookup becomes nearly O(1).

---

# Core Components of HashMap

Internally HashMap uses:

```text
Array
+
Hash Function
+
Linked List
+
Red Black Tree (Java 8+)
```

Structure:

```text
table[]

0
1
2 -> Node -> Node
3
4 -> Node
5
```

---

# Internal Node Structure

```java
static class Node<K,V> {

    final int hash;
    final K key;
    V value;

    Node<K,V> next;
}
```

Every entry becomes a Node.

Example:

```java
map.put("A",100);
```

Creates:

```text
Node
 hash
 key=A
 value=100
 next=null
```

---

# Put Operation Internals

Example:

```java
map.put("apple",100);
```

Step 1

Compute hash.

```java
int hash = key.hashCode();
```

Suppose:

```text
hash = 12345
```

Step 2

Find bucket.

```java
index = hash % capacity
```

Assume:

```text
12345 % 16 = 9
```

Step 3

Store inside bucket 9.

```text
table[9]
   |
Node(A,100)
```

---

# What is Capacity?

Default:

```java
new HashMap<>();
```

creates:

```text
capacity = 16
```

Meaning:

```text
16 buckets
```

```text
0
1
2
...
15
```

---

# What is Collision?

Two different keys may generate same bucket.

Example:

```text
apple -> bucket 5

banana -> bucket 5
```

Both go to same location.

This is Collision.

---

# Collision Handling

HashMap uses Separate Chaining.

```text
Bucket[5]

apple -> banana -> cat
```

Linked list inside bucket.

Example:

```text
table[5]
     |
     v

[A]
 |
 v
[B]
 |
 v
[C]
```

---

# Search During Collision

Need:

```java
map.get("B")
```

Go to bucket.

```text
bucket 5
```

Traverse:

```text
A
↓
B
```

Found.

---

# Java 8 Treeification

Suppose collisions become huge.

```text
A -> B -> C -> D -> E -> F -> G -> H -> I
```

Linked list search:

```text
O(n)
```

Java 8 converts bucket into:

```text
Red Black Tree
```

Condition:

```text
Bucket size >= 8
```

Now complexity becomes:

```text
O(log n)
```

---

# Load Factor

Most important interview topic.

Formula:

```text
loadFactor = size / capacity
```

Example:

```text
size = 12

capacity = 16

loadFactor = 0.75
```

Default:

```java
0.75f
```

---

# Why Load Factor Matters

As bucket occupancy increases:

```text
Collisions increase
```

Performance decreases.

---

# Rehashing

When:

```text
loadFactor > 0.75
```

Resize.

```text
16 -> 32

32 -> 64

64 -> 128
```

---

# Why Rehashing Needed

Index formula:

```text
hash % capacity
```

If capacity changes:

```text
16 -> 32
```

all bucket positions change.

Therefore every node must be redistributed.

---

# HashMap Complexity

| Operation | Average | Worst |
| --------- | ------- | ----- |
| Put       | O(1)    | O(n)  |
| Get       | O(1)    | O(n)  |
| Remove    | O(1)    | O(n)  |

Worst case:

```text
All keys collide
```

---

# HashMap Thread Safety

HashMap is NOT thread-safe.

Example:

Thread 1:

```java
put(A)
```

Thread 2:

```java
put(B)
```

Both modify bucket simultaneously.

Possible:

```text
Lost updates
Corrupted links
Incorrect size
```

---

# ConcurrentHashMap

Designed for Multi-threading.

```java
ConcurrentHashMap<K,V>
```

Goals:

1. Thread Safety
2. High Throughput
3. Fine-grained Locking

---

# Java 7 ConcurrentHashMap

Used Segments.

```text
ConcurrentHashMap

Segment 1
Segment 2
Segment 3
Segment 4
```

Each segment:

```text
Lock
+
Hash Table
```

Multiple threads can work on different segments.

---

# Java 8 ConcurrentHashMap

Segments removed.

Uses:

```text
Array<Node>
```

similar to HashMap.

But operations differ.

---

# Put in ConcurrentHashMap

Step 1

Calculate bucket.

Step 2

If bucket empty:

```text
CAS Operation
```

No lock.

---

# What is CAS?

Compare And Swap.

```text
Expected = null

Actual = null

Update succeeds
```

Atomic CPU instruction.

---

# If Bucket Already Occupied

Lock only bucket.

```java
synchronized(firstNode)
```

Not whole map.

Only bucket is blocked.

---

# Reads in ConcurrentHashMap

```java
get(key)
```

Mostly lock-free.

Multiple readers can access simultaneously.

Huge performance benefit.

---

# ConcurrentHashMap Resize

Multiple threads help resize.

```text
Thread 1 -> Move bucket range 1
Thread 2 -> Move bucket range 2
Thread 3 -> Move bucket range 3
```

Faster than single-thread migration.

---

# Why ConcurrentHashMap Is Faster

HashMap + synchronized:

```text
One global lock
```

ConcurrentHashMap:

```text
Bucket level locking
CAS
Lock-free reads
```

Higher concurrency.

---

# HashMap vs ConcurrentHashMap

| Feature            | HashMap       | ConcurrentHashMap      |
| ------------------ | ------------- | ---------------------- |
| Thread Safe        | No            | Yes                    |
| Reads              | Fast          | Fast                   |
| Writes             | Fast          | Fast under concurrency |
| Locking            | None          | Bucket-level           |
| Null Key           | Yes           | No                     |
| Null Value         | Yes           | No                     |
| Collision Handling | List + Tree   | List + Tree            |
| Resize             | Single Thread | Cooperative Resize     |

---

# HashMap vs Hashtable

| Feature     | HashMap | Hashtable   |
| ----------- | ------- | ----------- |
| Thread Safe | No      | Yes         |
| Locking     | None    | Whole Table |
| Performance | Faster  | Slower      |
| Null Key    | Yes     | No          |
| Null Value  | Yes     | No          |

---

# Interview Question:

Why does HashMap use Power of 2 Capacities?

Example:

```text
16
32
64
128
```

Because index calculation becomes:

```java
(hash & (capacity - 1))
```

instead of:

```java
hash % capacity
```

Bitwise operation is much faster.

---

# Most Important Interview Summary

HashMap internally uses:

```text
Array
 +
Hashing
 +
Linked List
 +
Red Black Tree
```

Flow:

```text
Key
 ↓
hashCode()
 ↓
Bucket Index
 ↓
Store Node
```

Collision:

```text
Linked List
```

Heavy Collision:

```text
Red Black Tree
```

Load Factor > 0.75:

```text
Rehash
```

ConcurrentHashMap adds:

```text
CAS
Bucket Locks
Lock-Free Reads
Cooperative Resize
```

to make the structure thread-safe while maintaining near O(1) operations.
