
# Java Data Structures Deep Dive (Part 2)
# HashMap, HashSet, LinkedHashMap, TreeMap, ConcurrentHashMap & Set Internals

Level: SDE-2 / Senior Backend Engineer

---

# Table of Contents

1. Why Maps Exist
2. Hashing Fundamentals
3. hashCode() and equals()
4. HashMap Internal Architecture
5. Put Operation Internals
6. Get Operation Internals
7. Collision Handling
8. Java 7 vs Java 8 HashMap
9. Treeification
10. Resize & Rehashing
11. Load Factor
12. HashSet Internals
13. LinkedHashMap Internals
14. LinkedHashSet Internals
15. TreeMap Internals
16. TreeSet Internals
17. Hashtable Internals
18. ConcurrentHashMap Internals
19. WeakHashMap
20. IdentityHashMap
21. Fail Fast Iterators
22. Interview Questions
23. Cheat Sheet

---

# 1. Why Maps Exist

Problem:

```java
List<User> users;
```

Find user:

```java
userId = 1001
```

Need linear scan.

Complexity:

```text
O(n)
```

---

Map solves this.

```java
Map<Integer, User> users;
```

Lookup:

```java
users.get(1001);
```

Average:

```text
O(1)
```

---

# 2. Hashing Fundamentals

HashMap works using:

```text
Hashing
```

---

Goal:

Convert

```text
Key
```

into

```text
Array Index
```

---

Example

```java
key = 123
```

Hash Function

```text
hash(123)
```

Result

```text
5
```

Store inside:

```text
bucket[5]
```

---

Visualization

```text
Bucket Array

0
1
2
3
4
5 -> User(123)
6
7
```

---

# Why Hashing?

Without hashing:

```text
Search O(n)
```

With hashing:

```text
Search O(1)
```

Average.

---

# 3. hashCode() and equals()

Most important HashMap interview topic.

---

Example

```java
class Employee {

    int id;
}
```

---

Object

```java
Employee e1 =
new Employee(1);
```

---

HashMap calls:

```java
e1.hashCode()
```

to determine bucket.

---

# hashCode()

Produces integer.

Example

```java
"JAVA".hashCode()
```

returns:

```text
2301506
```

---

HashMap uses hash value.

Not actual object.

---

# equals()

Used after bucket found.

Checks:

```text
Are two keys actually equal?
```

---

Interview Rule

Always override:

```java
hashCode()
```

and

```java
equals()
```

together.

---

Wrong

```java
equals()
```

without

```java
hashCode()
```

breaks HashMap.

---

# HashMap Contract

If:

```java
a.equals(b)
```

then:

```java
a.hashCode() == b.hashCode()
```

must be true.

---

Reverse not required.

---

Possible:

```java
hashCode same

equals false
```

Collision.

---

# 4. HashMap Internal Architecture

Java 8+

Structure:

```text
Bucket Array
      ↓
Node
      ↓
Linked List
      ↓
Red Black Tree
```

---

Simplified Source

```java
transient Node<K,V>[] table;
```

---

Node Structure

```java
class Node<K,V> {

    final int hash;

    final K key;

    V value;

    Node<K,V> next;
}
```

---

Visualization

```text
table

0
1
2
3
4 -> Node
5
6
```

---

# Initial Capacity

Default:

```text
16
```

Buckets.

---

Indexes

```text
0-15
```

---

# 5. Put Operation Internals

Example

```java
map.put("JAVA", 100);
```

---

Step 1

Calculate hash.

```java
hash("JAVA")
```

Suppose:

```text
2301506
```

---

Step 2

Calculate bucket.

Formula:

```java
(n - 1) & hash
```

---

Example

```text
capacity = 16

15 & hash
```

Result:

```text
bucket 10
```

---

Step 3

Check bucket.

Empty?

```text
YES
```

Insert.

---

Result

```text
bucket[10]

JAVA -> 100
```

---

# 6. Get Operation Internals

Example

```java
map.get("JAVA");
```

---

Step 1

Hash key.

---

Step 2

Find bucket.

---

Step 3

Traverse bucket.

---

Step 4

Compare:

```java
hash
```

and

```java
equals()
```

---

Return value.

---

Average:

```text
O(1)
```

---

# 7. Collision Handling

Most important concept.

---

Collision:

```text
Different keys
Same bucket
```

---

Example

```java
"A"
```

and

```java
"B"
```

land in bucket 5.

---

Before Java 8

```text
Bucket
   ↓
Linked List
```

---

Visualization

```text
Bucket 5

A
↓
B
↓
C
↓
D
```

---

Search:

```text
O(n)
```

Worst case.

---

# 8. Java 7 vs Java 8 HashMap

---

## Java 7

Bucket:

```text
Linked List Only
```

Worst Case:

```text
O(n)
```

---

## Java 8

Bucket:

```text
Linked List
        ↓
Red Black Tree
```

Worst Case:

```text
O(log n)
```

---

Huge improvement.

---

# 9. Treeification

When bucket becomes large.

---

Threshold

```text
8 Nodes
```

---

Condition

```java
bucket size > 8
```

AND

```java
capacity >= 64
```

---

Convert

```text
Linked List
```

into

```text
Red Black Tree
```

---

Before

```text
A
↓
B
↓
C
↓
D
```

---

After

```text
       C
      / \
     B   D
    /
   A
```

---

Search Complexity

Before:

```text
O(n)
```

After:

```text
O(log n)
```

---

# Untreeify

If nodes reduce below:

```text
6
```

Convert back to list.

---

# 10. Resize & Rehashing

Capacity:

```text
16
```

---

Load Factor:

```text
0.75
```

---

Threshold:

```text
16 * 0.75

12
```

---

Insert 13th element.

Resize triggered.

---

New Capacity

```text
32
```

---

Process

```text
Create new array
```

```text
Recalculate buckets
```

```text
Move nodes
```

---

Cost

```text
O(n)
```

---

# 11. Load Factor

Definition:

```text
How full HashMap can become
before resizing.
```

---

Default:

```java
0.75f
```

---

Why not:

```text
1.0
```

?

Too many collisions.

---

Why not:

```text
0.25
```

?

Too much memory waste.

---

0.75 chosen as balance.

---

# 12. HashSet Internals

Most asked question.

---

HashSet internally uses:

```java
HashMap
```

---

Source

```java
private transient HashMap<E,Object> map;
```

---

Dummy Value

```java
private static final Object PRESENT
```

---

Insertion

```java
set.add("JAVA");
```

Actually

```java
map.put("JAVA", PRESENT);
```

---

Therefore:

```text
HashSet = HashMap Keys Only
```

---

# 13. LinkedHashMap Internals

HashMap +

```text
Doubly Linked List
```

---

Node

```java
before
after
```

pointers.

---

Structure

```text
HashMap

JAVA
SPRING
AWS
```

---

Plus

```text
JAVA <-> SPRING <-> AWS
```

---

Maintains:

```text
Insertion Order
```

---

Complexity

```text
Still O(1)
```

Average.

---

# Why LinkedHashMap Used in LRU Cache

Supports:

```java
removeEldestEntry()
```

---

Automatically removes oldest entry.

---

# 14. LinkedHashSet Internals

Internally uses:

```java
LinkedHashMap
```

---

Benefits

```text
Unique Values
Insertion Order
```

---

# 15. TreeMap Internals

Implemented using:

```text
Red Black Tree
```

---

NOT Hashing.

---

Structure

```text
         50
        /  \
      25    75
```

---

Properties

```text
Sorted Keys
```

---

Complexities

| Operation | Complexity |
|------------|------------|
| Put | O(log n) |
| Get | O(log n) |
| Remove | O(log n) |

---

# Why TreeMap Slower Than HashMap

HashMap

```text
O(1)
```

---

TreeMap

```text
O(log n)
```

---

But TreeMap provides:

```text
Sorting
```

---

# 16. TreeSet Internals

Internally:

```java
TreeMap
```

---

Stores:

```text
Keys Only
```

---

Properties

```text
Sorted
Unique
```

---

Complexities

```text
O(log n)
```

for add/remove/search.

---

# 17. Hashtable Internals

Legacy class.

---

Thread Safe.

---

Implementation

```text
Every method synchronized.
```

---

Example

```java
public synchronized V put()
```

---

Problem

```text
One lock for entire table.
```

---

Performance poor.

---

Modern replacement:

```java
ConcurrentHashMap
```

---

# Hashtable vs HashMap

| Feature | HashMap | Hashtable |
|----------|----------|----------|
| Thread Safe | No | Yes |
| Null Key | Yes | No |
| Null Value | Yes | No |
| Performance | Fast | Slow |

---

# 18. ConcurrentHashMap Internals

Most important SDE-2 topic.

---

Problem

HashMap:

```text
Not Thread Safe
```

---

Hashtable:

```text
Too Much Locking
```

---

Solution:

```text
ConcurrentHashMap
```

---

# Java 7 Design

Used:

```text
Segment Locking
```

---

Structure

```text
Segment 1
Segment 2
Segment 3
Segment 4
```

Each segment locked independently.

---

# Java 8 Design

Removed segments.

Uses:

```text
CAS
Bucket Locks
Tree Nodes
```

---

# Reads

Mostly:

```text
Lock Free
```

---

# Writes

Lock only:

```text
Affected Bucket
```

---

Much better concurrency.

---

# Why Null Not Allowed?

Question:

```java
map.get(key)
```

returns:

```text
null
```

---

Cannot distinguish:

```text
Key Missing
```

vs

```text
Key Exists with Null Value
```

under concurrent access.

---

Therefore:

```text
Null keys and values disallowed.
```

---

# 19. WeakHashMap

Uses:

```text
Weak References
```

---

When key no longer referenced.

GC removes entry automatically.

---

Used for:

```text
Caches
Metadata
```

---

# 20. IdentityHashMap

Uses:

```java
==
```

instead of:

```java
equals()
```

---

Example

```java
new String("A")
```

and

```java
new String("A")
```

considered different.

---

Rarely used.

---

# 21. Fail Fast Iterators

Example

```java
ArrayList
HashMap
```

---

Iterator stores:

```text
modCount
```

---

If collection modified during iteration:

```java
ConcurrentModificationException
```

---

Example

```java
for(Integer i : list) {

   list.remove(i);
}
```

Throws exception.

---

Purpose:

```text
Detect bugs early.
```

---

# 22. Common Interview Questions

---

## Why HashMap O(1)?

```text
Hashing maps key directly
to bucket.
```

---

## Why Override hashCode and equals Together?

```text
HashMap depends on both
for correct key lookup.
```

---

## Why Treeification?

```text
Prevent O(n) bucket traversal.
```

---

## Why Load Factor 0.75?

```text
Best tradeoff between
memory and collisions.
```

---

## Why ConcurrentHashMap Faster Than Hashtable?

```text
Fine-grained locking,
CAS operations,
lock-free reads.
```

---

## Why ConcurrentHashMap Doesn't Allow Null?

```text
Avoid ambiguity during
concurrent lookups.
```

---

## Difference Between HashMap and TreeMap?

HashMap

```text
O(1)
Unordered
```

TreeMap

```text
O(log n)
Sorted
```

---

# 23. Final Cheat Sheet

## HashMap

```text
Array
+
Linked List
+
Red Black Tree
```

Average:

```text
O(1)
```

---

## HashSet

```text
HashMap Keys Only
```

---

## LinkedHashMap

```text
HashMap
+
Doubly Linked List
```

Maintains order.

---

## TreeMap

```text
Red Black Tree
```

Sorted.

```text
O(log n)
```

---

## TreeSet

```text
TreeMap Keys Only
```

---

## Hashtable

```text
Fully Synchronized
```

Legacy.

---

## ConcurrentHashMap

```text
CAS
Bucket Locking
Treeification
```

Thread Safe.

---

# Golden Interview Answer

> HashMap internally uses an array of buckets where each bucket contains either a linked list or a Red-Black Tree (Java 8+). Keys are mapped to buckets using `hashCode()`, and collisions are resolved through chaining. When a bucket grows beyond 8 nodes and capacity exceeds 64, it is treeified into a Red-Black Tree, reducing worst-case lookup from O(n) to O(log n). ConcurrentHashMap extends this concept with lock-free reads, CAS operations, and bucket-level locking, providing high-performance thread-safe access without the contention of Hashtable.
