# Java Map, HashMap & ConcurrentHashMap - Complete Deep Dive

# Table of Contents

1. What is a Map?
2. Map Hierarchy
3. Why HashMap Exists
4. HashMap Internal Structure
5. HashMap Put Operation
6. HashMap Get Operation
7. hashCode() Explained
8. equals() Explained
9. hashCode() and equals() Contract
10. Collision Handling
11. Load Factor
12. Rehashing / Resizing
13. Treeification (Java 8)
14. HashMap Complexity
15. Why Capacity is Power of 2
16. Null Keys and Values
17. HashMap Thread Safety
18. ConcurrentHashMap Internals
19. Why ConcurrentHashMap Disallows Null
20. HashMap vs ConcurrentHashMap
21. Interview Questions
22. Quick Revision Notes

---

# 1. What is a Map?

Map is an interface that stores data as:

```text
Key -> Value
```

Example:

```java
Map<String,Integer> map = new HashMap<>();

map.put("A",100);
map.put("B",200);
```

Map defines operations:

```java
put(K,V)
get(K)
remove(K)
containsKey(K)
size()
```

Map itself contains no implementation.

It only defines a contract.

---

# 2. Map Hierarchy

```text
Map (Interface)
 |
 +-- HashMap
 |
 +-- ConcurrentHashMap
 |
 +-- TreeMap
 |
 +-- Hashtable
 |
 +-- LinkedHashMap
```

Map is separate from Collection hierarchy.

---

# 3. Why HashMap Exists

Without hashing:

```text
A -> 100
B -> 200
C -> 300
```

Search requires:

```text
A -> B -> C
```

Complexity:

```text
O(n)
```

HashMap uses hashing to achieve:

```text
O(1)
```

average lookup.

---

# 4. HashMap Internal Structure

HashMap internally uses:

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
4
5 -> Node
```

Every entry becomes a Node.

```java
static class Node<K,V> {

    int hash;
    K key;
    V value;
    Node<K,V> next;
}
```

---

# 5. HashMap Put Operation

Example:

```java
map.put("apple",100);
```

### Step 1

Calculate hash.

```java
int hash = key.hashCode();
```

Example:

```text
12345
```

### Step 2

Find bucket.

```java
index = hash & (capacity - 1);
```

Assume:

```text
capacity = 16

index = 9
```

### Step 3

Store in bucket.

```text
table[9]

(apple,100)
```

### Step 4

If bucket already contains nodes:

* Compare keys
* Update if found
* Otherwise append new node

---

# 6. HashMap Get Operation

```java
map.get("apple");
```

Steps:

1. Calculate hash
2. Find bucket
3. Traverse bucket
4. Match key
5. Return value

Average complexity:

```text
O(1)
```

---

# 7. hashCode() Explained

hashCode determines:

```text
Which bucket should be searched?
```

Example:

```java
hash("apple") = 12345
```

Bucket:

```java
12345 % 16 = 9
```

HashMap jumps directly to bucket 9.

Without hashCode:

```text
Search every element
```

O(n)

---

# 8. equals() Explained

Many developers misunderstand this.

HashMap uses:

```text
hashCode()
to locate bucket

equals()
to locate exact key
```

Think:

```text
hashCode() -> Building Number

equals() -> Apartment Number
```

---

## Example

```java
String s1 = new String("ABC");
String s2 = new String("ABC");
```

Different objects:

```java
s1 == s2
```

returns:

```text
false
```

But:

```java
s1.equals(s2)
```

returns:

```text
true
```

Therefore:

```java
map.put(s1,100);

map.get(s2);
```

returns:

```text
100
```

---

# 9. hashCode() and equals() Contract

If:

```java
a.equals(b)
```

returns:

```text
true
```

then:

```java
a.hashCode() == b.hashCode()
```

MUST be true.

Otherwise HashMap breaks.

---

## Bad Example

Override equals only:

```java
@Override
public boolean equals(...)
```

but not:

```java
@Override
public int hashCode()
```

Result:

```text
Put goes to one bucket

Get searches another bucket
```

Lookup fails.

---

# 10. Collision Handling

Collision:

```text
Two keys map to same bucket
```

Example:

```text
apple -> bucket 5

banana -> bucket 5
```

Structure:

```text
Bucket[5]

apple
  |
banana
```

This is called:

```text
Separate Chaining
```

---

# 11. Load Factor

Formula:

```text
loadFactor = size / capacity
```

Default:

```java
0.75f
```

Example:

```text
size = 12
capacity = 16

loadFactor = 0.75
```

---

# Why Load Factor Matters

Higher load factor:

```text
More collisions
```

Lower performance.

---

# 12. Rehashing / Resizing

When:

```text
loadFactor > 0.75
```

HashMap resizes.

Example:

```text
16 -> 32

32 -> 64

64 -> 128
```

Every node must be redistributed.

Why?

Because:

```java
index = hash & (capacity - 1)
```

changes when capacity changes.

---

# 13. Treeification (Java 8)

Heavy collision:

```text
A -> B -> C -> D -> E -> F -> G -> H -> I
```

Search:

```text
O(n)
```

Java 8 converts bucket into:

```text
Red Black Tree
```

Condition:

```text
Bucket Size >= 8
```

Now:

```text
O(log n)
```

lookup.

---

# 14. HashMap Complexity

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

# 15. Why Capacity is Power of 2

Capacities:

```text
16
32
64
128
```

Reason:

HashMap uses:

```java
hash & (capacity - 1)
```

instead of:

```java
hash % capacity
```

Bitwise operation is much faster.

---

# 16. Null Keys and Values

HashMap allows:

```java
map.put(null,100);
map.put("A",null);
```

Valid.

HashMap stores null key in bucket 0.

---

# 17. HashMap Thread Safety

HashMap is NOT thread-safe.

Problems:

```text
Lost Updates
Corrupted Structure
Incorrect Size
Concurrent Resize Issues
```

Example:

```java
Thread1 -> put(A)

Thread2 -> put(B)
```

Can corrupt data.

---

# 18. ConcurrentHashMap Internals

Designed for multi-threading.

Goals:

```text
Thread Safety
High Throughput
Low Lock Contention
```

---

## Java 7

Used Segmented Locking.

```text
Segment 1
Segment 2
Segment 3
Segment 4
```

Each segment had its own lock.

---

## Java 8

Segments removed.

Uses:

```text
Array<Node>
```

similar to HashMap.

---

## Put Operation

### Empty Bucket

Uses CAS

```text
Compare And Swap
```

No lock required.

---

## Occupied Bucket

Lock only bucket.

```java
synchronized(firstNode)
```

Not entire map.

---

## Reads

```java
get()
```

Mostly lock-free.

Multiple readers work simultaneously.

---

## Resize

Multiple threads help transfer data.

```text
Thread1 -> Move Range A

Thread2 -> Move Range B

Thread3 -> Move Range C
```

Called cooperative resizing.

---

# 19. Why ConcurrentHashMap Disallows Null

HashMap:

```java
map.put("A",null);
```

Valid.

ConcurrentHashMap:

```java
map.put("A",null);
```

Throws exception.

---

## Reason

Suppose:

```java
Integer value = map.get("A");
```

returns:

```text
null
```

Question:

Did key exist with null value?

OR

Did key not exist?

OR

Was key removed by another thread?

Impossible to know.

Therefore:

```text
Null Keys -> Not Allowed

Null Values -> Not Allowed
```

Now:

```java
map.get(key) == null
```

always means:

```text
Key absent
```

---

# 20. HashMap vs ConcurrentHashMap

| Feature            | HashMap       | ConcurrentHashMap |
| ------------------ | ------------- | ----------------- |
| Thread Safe        | No            | Yes               |
| Null Key           | Yes           | No                |
| Null Value         | Yes           | No                |
| Reads              | Fast          | Lock Free         |
| Writes             | Fast          | Bucket Locked     |
| Collision Handling | List + Tree   | List + Tree       |
| Resize             | Single Thread | Cooperative       |
| Synchronization    | None          | Fine Grained      |

---

# 21. Interview Questions

### Why does HashMap use equals()?

Answer:

hashCode finds bucket.

equals finds exact key inside bucket.

---

### Why override hashCode and equals together?

Answer:

If equal objects generate different hashes, lookup fails.

---

### Why collisions occur?

Different keys can generate same bucket index.

---

### Why treeify at 8 nodes?

To improve lookup from O(n) to O(log n).

---

### Why load factor 0.75?

Balances memory usage and performance.

---

### Why ConcurrentHashMap doesn't allow null?

Because null return values become ambiguous in concurrent environments.

---

### Why power-of-two capacity?

To use:

```java
hash & (capacity - 1)
```

which is faster than modulo.

---

# 22. Quick Revision Notes

HashMap:

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
Bucket
 ↓
equals()
 ↓
Value
```

Collision:

```text
Separate Chaining
```

Resize:

```text
Load Factor > 0.75
```

Treeification:

```text
Bucket Size >= 8
```

ConcurrentHashMap:

```text
CAS
+
Bucket Locking
+
Lock-Free Reads
+
Cooperative Resize
```

Golden Rule:

hashCode() determines WHERE to search.

equals() determines WHAT to match.
