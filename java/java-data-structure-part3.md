
# Java Data Structures Deep Dive (Part 3)
# Trees, Heaps, PriorityQueue, Trie, AVL Tree, Red-Black Tree & B+ Tree

Level: SDE-2 / Senior Backend Engineer

---

# Table of Contents

1. Why Trees Exist
2. Binary Tree
3. Binary Search Tree (BST)
4. BST Operations Internals
5. BST Deletion Cases
6. Self-Balancing Trees
7. AVL Tree Internals
8. AVL Rotations
9. Red-Black Tree Internals
10. Why Java Uses Red-Black Trees
11. Heap Fundamentals
12. Min Heap Internals
13. Max Heap Internals
14. Heap Insert Operation
15. Heap Delete Operation
16. Heapify Algorithm
17. Why Build Heap is O(n)
18. PriorityQueue Internals
19. Trie Internals
20. Trie Operations
21. Autocomplete Systems
22. Segment Tree
23. Fenwick Tree (Binary Indexed Tree)
24. B Tree
25. B+ Tree
26. Why Databases Use B+ Trees
27. B+ Tree vs BST
28. Interview Questions
29. Cheat Sheet

---

# 1. Why Trees Exist

Problem with Arrays

```text
Search = O(n)
```

Problem with Linked Lists

```text
Search = O(n)
```

---

Need:

```text
Fast Search
Fast Insert
Fast Delete
```

---

Trees help achieve:

```text
O(log n)
```

operations.

---

# 2. Binary Tree

A tree where each node has:

```text
At most 2 children
```

---

Structure

```text
        10
       /  \
      5   15
```

---

Node Structure

```java
class TreeNode {

    int value;

    TreeNode left;

    TreeNode right;
}
```

---

Terminology

```text
Root
Parent
Child
Leaf
Height
Depth
```

---

Example

```text
        10
       /  \
      5   15
```

Root:

```text
10
```

Leaf Nodes:

```text
5
15
```

---

# 3. Binary Search Tree (BST)

Special Binary Tree.

Rule:

```text
Left < Root < Right
```

---

Example

```text
          50
         /  \
       30    70
      / \   / \
    20 40 60 80
```

---

Advantages

```text
Fast Search
Fast Insert
Fast Delete
```

---

# 4. BST Search Internals

Search:

```text
60
```

---

Traversal

```text
50
 ↓
70
 ↓
60
```

---

Operations

```text
50 < 60 → go right

70 > 60 → go left
```

---

Nodes visited:

```text
3
```

instead of:

```text
7
```

---

Complexity

Balanced BST:

```text
O(log n)
```

---

Worst Case

```text
O(n)
```

---

# BST Degeneration Problem

Insert:

```text
10
20
30
40
50
```

BST becomes:

```text
10
 \
 20
   \
   30
     \
     40
       \
       50
```

---

Tree becomes:

```text
Linked List
```

---

Complexity

```text
O(n)
```

---

Solution:

```text
Self-Balancing Trees
```

---

# 5. BST Deletion Cases

Most asked interview topic.

---

Case 1: Leaf Node

Delete:

```text
20
```

```text
 30
 /
20
```

Simply remove.

---

Complexity:

```text
O(log n)
```

---

Case 2: One Child

```text
30
 /
20
 /
10
```

Delete:

```text
20
```

Promote:

```text
10
```

---

Case 3: Two Children

Most important.

---

Example

```text
       50
      /  \
    30    70
         /  \
       60   80
```

Delete:

```text
70
```

---

Replace with:

```text
Inorder Successor
```

Smallest node in right subtree.

---

Successor:

```text
80
```

or predecessor:

```text
60
```

depending on implementation.

---

# 6. Self-Balancing Trees

Goal:

```text
Maintain O(log n)
```

always.

---

Popular Trees

```text
AVL Tree
Red Black Tree
B Tree
B+ Tree
```

---

# 7. AVL Tree Internals

AVL = Adelson-Velsky Landis Tree.

---

Rule

For every node:

```text
Height Difference <= 1
```

---

Balance Factor

```text
Height(left)
-
Height(right)
```

---

Valid

```text
-1
0
1
```

---

Example

```text
     30
    / \
  20  40
```

Balanced.

---

# AVL Violation

Insert

```text
30
20
10
```

Tree

```text
    30
   /
 20
 /
10
```

Balance Factor:

```text
2
```

Invalid.

---

Needs rotation.

---

# 8. AVL Rotations

Most asked AVL topic.

---

## Left Rotation

Before

```text
10
  \
  20
    \
    30
```

After

```text
    20
   /  \
 10   30
```

---

## Right Rotation

Before

```text
    30
   /
 20
 /
10
```

After

```text
    20
   /  \
 10   30
```

---

## Left Right Rotation

```text
30
 /
10
  \
  20
```

Requires:

```text
Left Rotation
then
Right Rotation
```

---

## Right Left Rotation

```text
10
  \
  30
 /
20
```

Requires:

```text
Right Rotation
then
Left Rotation
```

---

# AVL Complexity

```text
Search O(log n)
Insert O(log n)
Delete O(log n)
```

---

# Drawback

Too many rotations.

---

# 9. Red Black Tree Internals

Used by:

```text
TreeMap
TreeSet
HashMap Treeified Buckets
```

---

Each node has:

```text
Color
```

---

Colors

```text
RED
BLACK
```

---

Rules

1.

```text
Root always BLACK
```

---

2.

```text
No two consecutive RED nodes
```

---

3.

```text
Every path contains same
number of BLACK nodes.
```

---

Example

```text
        B50
       /   \
     R30   R70
```

---

# Why Red Black Tree?

AVL:

```text
More Balanced
```

---

But

```text
More Rotations
```

---

Red Black Tree:

```text
Slightly Less Balanced
Much Faster Inserts
```

---

Used heavily in production.

---

# 10. Why Java Uses Red Black Tree

Used in:

```java
TreeMap
TreeSet
HashMap
```

because:

```text
Good Balance
Fewer Rotations
Better Overall Throughput
```

---

# 11. Heap Fundamentals

Heap = Complete Binary Tree.

---

Complete means:

```text
Filled Left to Right
```

---

# Min Heap

Rule

```text
Parent <= Children
```

---

Example

```text
      10
     /  \
   20   30
```

---

Smallest element at root.

---

# Max Heap

Rule

```text
Parent >= Children
```

---

Example

```text
      50
     /  \
   30   40
```

---

Largest element at root.

---

# 12. Heap Internal Representation

Most important interview concept.

---

Heap uses:

```text
Array
```

not nodes.

---

Example

```text
       10
      /  \
    20   30
   /
 40
```

Stored as:

```text
[10,20,30,40]
```

---

# Parent Formula

```java
(i - 1) / 2
```

---

# Left Child

```java
2*i + 1
```

---

# Right Child

```java
2*i + 2
```

---

Example

```text
Index 0 = 10
```

Left:

```text
1
```

Right:

```text
2
```

---

# 13. Heap Insert Operation

Insert:

```text
5
```

---

Step 1

Place at end.

```text
10
20
30
40
5
```

---

Step 2

Bubble Up.

```text
5 < 20
```

Swap.

---

Again

```text
5 < 10
```

Swap.

---

Result

```text
       5
      / \
    10 30
   /
 40
```

---

Complexity

```text
O(log n)
```

---

# 14. Heap Delete Operation

Delete root.

---

Replace root with last element.

---

Example

```text
Root = 5
```

Replace with:

```text
40
```

---

Heapify Down.

---

Swap with smaller child.

---

Complexity

```text
O(log n)
```

---

# 15. Heapify Algorithm

Most important heap operation.

---

Purpose

```text
Restore Heap Property
```

---

Two Types

```text
Heapify Up
Heapify Down
```

---

Used in:

```text
PriorityQueue
Heap Sort
```

---

# 16. Why Build Heap is O(n)

Interview favorite.

---

Wrong assumption:

```text
n inserts
×
log n

=
O(n log n)
```

---

Actual Build Heap:

```text
Bottom Up Heapify
```

---

Complexity

```text
O(n)
```

---

Reason:

Most nodes are near leaves.

Require very little heapify work.

---

# 17. PriorityQueue Internals

Java

```java
PriorityQueue<Integer>
```

uses:

```text
Min Heap
```

---

Example

```java
pq.add(50);
pq.add(10);
pq.add(30);
```

---

Removal Order

```text
10
30
50
```

---

Operations

| Operation | Complexity |
|------------|------------|
| Peek | O(1) |
| Insert | O(log n) |
| Remove | O(log n) |

---

# Why PriorityQueue Not Sorted?

Interview Trick Question.

---

Internal Array

```text
[5,10,50,20]
```

Not fully sorted.

---

Only guarantee:

```text
Minimum at root.
```

---

# 18. Trie Internals

Trie = Prefix Tree.

---

Stores strings efficiently.

---

Example

```text
cat
car
care
```

---

Structure

```text
          root
            |
            c
            |
            a
           / \
          t   r
               \
                e
```

---

Node Structure

```java
class TrieNode {

    TrieNode[] children =
        new TrieNode[26];

    boolean isWord;
}
```

---

# 19. Trie Search

Search

```text
care
```

---

Traversal

```text
c
↓
a
↓
r
↓
e
```

---

Complexity

```text
O(length)
```

---

Not dependent on:

```text
Number of Words
```

---

# 20. Autocomplete Systems

Google Search.

---

Input

```text
ca
```

---

Navigate Trie:

```text
c
↓
a
```

---

Collect descendants.

```text
cat
car
care
cart
camera
```

---

Very efficient.

---

# 21. Segment Tree

Used for:

```text
Range Queries
```

---

Example

```text
Sum(10-500)
```

---

Complexities

```text
Query O(log n)
Update O(log n)
```

---

Used in:

```text
Competitive Programming
Analytics Systems
```

---

# 22. Fenwick Tree (BIT)

Simpler than Segment Tree.

---

Supports

```text
Prefix Sum
```

queries.

---

Complexities

```text
Update O(log n)
Query O(log n)
```

---

Memory:

```text
Less than Segment Tree
```

---

# 23. B Tree

Used in databases.

---

Unlike BST:

```text
Many Children
```

---

Example

```text
[10|20|30]
 / | | \
```

---

Reduces tree height.

---

Ideal for disks.

---

# 24. B+ Tree

Most important database index structure.

---

Used by:

```text
MySQL
PostgreSQL
Oracle
SQL Server
```

---

Structure

```text
Internal Nodes
```

store:

```text
Keys Only
```

---

Leaf Nodes

store:

```text
Actual Data
```

---

Example

```text
           [20|40]
          /   |   \
     [10] [30] [50]

Leaf Level:

10 <-> 20 <-> 30 <-> 40 <-> 50
```

---

# Why B+ Tree Better Than BST

BST

```text
Millions of nodes
```

Deep tree.

Many disk reads.

---

B+ Tree

```text
Huge branching factor
```

Example

```text
1000 children per node
```

---

Height

```text
2 or 3
```

for millions of rows.

---

Huge performance gain.

---

# 25. Leaf Node Linking

Most important B+ Tree feature.

---

Leaf Nodes

```text
10 ↔ 20 ↔ 30 ↔ 40 ↔ 50
```

---

Range Query

```sql
WHERE id BETWEEN
1000 AND 5000
```

Fast sequential scan.

---

# Why Databases Prefer B+ Tree

Supports:

```text
Equality Search
Range Search
Sorting
Sequential Reads
```

---

Perfect for:

```sql
ORDER BY
BETWEEN
>
<
>=
<=
```

---

# 26. B+ Tree vs BST

| Feature | BST | B+ Tree |
|----------|----------|----------|
| Height | High | Very Low |
| Disk Reads | Many | Few |
| Range Query | Slow | Fast |
| Sequential Scan | Poor | Excellent |
| DB Usage | Rare | Standard |

---

# 27. Common Interview Questions

---

## Why Heap Uses Array?

```text
Complete tree allows
easy index calculation.
```

---

## Difference Between Heap and BST?

Heap:

```text
Parent relation only.
```

BST:

```text
Global ordering.
```

---

## Why PriorityQueue O(log n)?

```text
Insert/Delete require
heapify operations.
```

---

## Why Trie Search O(length)?

```text
Characters directly determine path.
```

---

## Why Databases Use B+ Trees?

```text
Low height,
few disk reads,
excellent range queries.
```

---

## Why TreeMap Uses Red Black Tree?

```text
Balanced performance with
fewer rotations than AVL.
```

---

# 28. Final Cheat Sheet

## BST

```text
Average O(log n)
Worst O(n)
```

---

## AVL Tree

```text
Strictly Balanced
More Rotations
```

---

## Red Black Tree

```text
Loosely Balanced
Production Friendly
```

---

## Heap

```text
Insert O(log n)
Delete O(log n)
Peek O(1)
```

---

## PriorityQueue

```text
Implemented using Min Heap
```

---

## Trie

```text
Search O(length)
Autocomplete
```

---

## Segment Tree

```text
Range Query O(log n)
```

---

## B+ Tree

```text
Database Indexes
Very Low Height
Leaf Nodes Linked
```

---

# Golden Interview Answer

> Trees provide logarithmic search, insert, and delete operations by organizing data hierarchically. Java's TreeMap and TreeSet use Red-Black Trees because they maintain balance with fewer rotations than AVL trees. PriorityQueue is implemented using a binary min-heap stored as an array, enabling O(log n) insertion and deletion. Tries optimize prefix-based searches and power autocomplete systems. Modern databases use B+ Trees instead of BSTs because their high branching factor minimizes disk I/O and their linked leaf nodes make range queries extremely efficient.
