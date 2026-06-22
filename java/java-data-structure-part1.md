# Java Data Structures Deep Dive (Part 1)
# Arrays, ArrayList, LinkedList, Stack, Queue, Deque

Level: SDE-2 / Senior Backend Engineer

---

# Table of Contents

1. Array Internals
2. Dynamic Array Internals (ArrayList)
3. LinkedList Internals
4. ArrayList vs LinkedList
5. Stack Internals
6. Queue Internals
7. Deque Internals
8. Interview Deep Dives

---

# 1. Array Internals

Arrays are the foundation of almost every data structure.

---

Example

```java
int[] arr = {10,20,30,40};
```

Memory Layout

```text
Address     Value

1000        10
1004        20
1008        30
1012        40
```

Elements are stored in:

```text
Contiguous Memory
```

---

# Why Array Access is O(1)

Suppose:

```java
arr[3]
```

Base Address:

```text
1000
```

Element Size:

```text
4 bytes
```

Formula:

```text
address = base + (index * size)
```

Calculation:

```text
1000 + (3 * 4)

1012
```

Direct access.

No traversal required.

Therefore:

```text
O(1)
```

---

# Why Insertion is O(n)

Insert:

```text
10 20 30 40
```

Insert 25 at index 2

Need:

```text
10 20 _ 30 40
```

Shift:

```text
30 -> right
40 -> right
```

Result:

```text
10 20 25 30 40
```

Work done:

```text
n shifts
```

Complexity:

```text
O(n)
```

---

# Why Deletion is O(n)

Delete:

```text
10 20 30 40 50
```

Delete 30

Shift:

```text
40 left
50 left
```

Complexity:

```text
O(n)
```

---

# Array Advantages

```text
Fastest Random Access
Cache Friendly
Low Memory Overhead
```

---

# Array Disadvantages

```text
Fixed Size
Costly Inserts
Costly Deletes
```

---

# 2. Dynamic Array Internals (ArrayList)

Most asked Java collection interview topic.

---

# Internal Structure

ArrayList uses:

```java
Object[] elementData;
```

Internally.

Source code simplified:

```java
class ArrayList {

    Object[] elementData;

    int size;
}
```

---

# Initial Capacity

Java 8+

```java
new ArrayList<>();
```

does NOT immediately create array.

Array allocated only when first element inserted.

---

After first insertion:

```text
Capacity = 10
```

---

Memory

```text
[ ]
[ ]
[ ]
[ ]
[ ]
[ ]
[ ]
[ ]
[ ]
[ ]
```

10 slots.

---

# What Happens When Full?

Example

Capacity:

```text
10
```

Insert:

```text
11th element
```

Array becomes full.

---

ArrayList grows using:

```java
newCapacity =
oldCapacity + (oldCapacity >> 1)
```

Equivalent:

```text
old * 1.5
```

---

Example

```text
10 -> 15

15 -> 22

22 -> 33
```

---

# Resize Process

Old Array

```text
10
20
30
```

Create New Array

```text
Capacity 15
```

Copy all elements.

Replace reference.

---

Cost

```text
O(n)
```

---

# Why Add() is O(1) Amortized?

Most additions:

```text
Append directly
```

Only occasionally:

```text
Resize
Copy
```

Therefore:

```text
Amortized O(1)
```

---

# Why ArrayList Fast?

Contiguous memory.

CPU cache friendly.

---

Example

```text
10
20
30
40
50
```

CPU fetches nearby memory together.

---

LinkedList lacks this advantage.

---

# 3. LinkedList Internals

Java LinkedList:

```java
LinkedList<E>
```

uses:

```text
Doubly Linked List
```

not singly.

---

Node Structure

```java
class Node<E> {

    E item;

    Node<E> next;

    Node<E> prev;
}
```

---

Visualization

```text
NULL

 ← prev

10

next →

20

next →

30

next →

NULL
```

---

# Insert at Beginning

Before

```text
10 -> 20 -> 30
```

Insert 5

```text
5 -> 10 -> 20 -> 30
```

Only pointer updates.

Complexity:

```text
O(1)
```

---

# Insert at End

LinkedList maintains:

```java
head
tail
```

references.

No traversal.

Complexity:

```text
O(1)
```

---

# Why Random Access is O(n)

Need:

```java
list.get(5000)
```

Must traverse nodes.

```text
1
2
3
...
5000
```

Complexity:

```text
O(n)
```

---

# Why LinkedList Uses More Memory?

ArrayList

```text
Value Only
```

---

LinkedList Node

```text
Value
Next Pointer
Prev Pointer
Object Header
```

Much larger memory footprint.

---

# Why LinkedList Often Slower Than ArrayList

Many developers assume:

```text
LinkedList faster insertions
```

Always.

Wrong.

---

Reason:

```text
Pointer Chasing
Cache Misses
Memory Allocations
```

Modern CPUs heavily favor arrays.

---

Real-world:

```text
ArrayList usually wins.
```

---

# ArrayList vs LinkedList

| Feature | ArrayList | LinkedList |
|----------|-----------|-----------|
| Random Access | O(1) | O(n) |
| Insert End | O(1) | O(1) |
| Insert Middle | O(n) | O(n) |
| Memory | Low | High |
| Cache Friendly | Yes | No |

---

Interview Answer

```text
ArrayList should be default choice.

LinkedList only when frequent insertions
and deletions happen through references.
```

---

# 4. Stack Internals

LIFO

```text
Last In First Out
```

---

Push

```text
10
20
30
```

Top

```text
30
```

---

Pop

Returns

```text
30
```

---

# Internal Implementation

Can be built using:

```text
Array
Linked List
Deque
```

---

Modern Java Recommendation

```java
Deque<Integer> stack =
new ArrayDeque<>();
```

---

Reason

```text
Faster
No synchronization overhead
```

---

# Function Call Stack

Every method call creates:

```text
Stack Frame
```

Example

```java
main()
  |
foo()
  |
bar()
```

Memory

```text
bar
foo
main
```

LIFO.

---

# Applications

```text
DFS
Undo
Recursion
Expression Parsing
Browser Back
```

---

# 5. Queue Internals

FIFO

```text
First In First Out
```

---

Example

```text
10
20
30
```

Remove

```text
10
```

---

Implementation

Usually:

```text
Linked List
Circular Array
```

---

Operations

```java
offer()
poll()
peek()
```

---

Applications

```text
Task Scheduling
Kafka
RabbitMQ
BFS
Producer Consumer
```

---

# 6. Circular Queue

Problem

Normal array queue wastes space.

---

Example

```text
[10][20][30][40]
```

Remove:

```text
10
20
```

Memory:

```text
[X][X][30][40]
```

Unused space.

---

Circular Queue wraps around.

```text
rear reaches end
```

then:

```text
returns to beginning
```

---

Used heavily in:

```text
Thread Pools
Networking
Schedulers
```

---

# 7. Deque Internals

Double Ended Queue

---

Supports

```text
Insert Front
Insert Back

Delete Front
Delete Back
```

---

Implementation

```java
ArrayDeque
```

Internally:

```text
Resizable Circular Array
```

---

Advantages

```text
O(1) insertion/removal
both ends
```

---

Applications

```text
Sliding Window Maximum
LRU Cache
Monotonic Queue
Stack Replacement
Queue Replacement
```

---

# Most Asked Interview Question

Why ArrayDeque instead of Stack?

Answer:

```text
Stack is legacy and synchronized.

ArrayDeque is faster and preferred.
```

---

# Cheat Sheet

Array

```text
O(1) access
O(n) insert/delete
```

---

ArrayList

```text
Dynamic Array
1.5x Growth
Amortized O(1) add
```

---

LinkedList

```text
Doubly Linked List
High Memory Usage
Poor Cache Locality
```

---

Stack

```text
LIFO
Use ArrayDeque
```

---

Queue

```text
FIFO
Task Processing
```

---

Deque

```text
Queue + Stack
Most Flexible
```
