# Database Internals - Complete Study Guide

# Table of Contents

1. Database Storage Fundamentals
2. Why Indexes Exist
3. B+ Tree Internals
4. Clustered Index
5. Non-Clustered Index
6. Composite Index
7. Query Optimization
8. Query Execution Lifecycle
9. Cost Based Optimizer
10. EXPLAIN Plan Analysis
11. Real Production Examples
12. Common Interview Questions
13. End-to-End Query Flow
14. Summary

---

# 1. Database Storage Fundamentals

Before understanding indexes, we need to understand how databases store data.

Consider:

```sql
CREATE TABLE employee(
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary BIGINT
);
```

Rows are stored on disk inside:

```text
Pages (Blocks)
```

Typical page size:

```text
8KB
16KB
32KB
```

Example:

```text
Page 1

1, Anshi, Eng, 100
2, John, Eng, 120
3, Alex, HR, 90
```

```text
Page 2

4, Mike, Sales, 110
5, Sarah, HR, 95
```

---

Without an index:

```sql
SELECT *
FROM employee
WHERE id = 5;
```

Database performs:

```text
Full Table Scan
```

Meaning:

```text
Page1 → Page2 → Page3 → ...
```

Time Complexity:

```text
O(N)
```

Problem:

```text
1 Billion rows
```

becomes expensive.

---

# 2. Why Indexes Exist

Indexes are special data structures that allow fast lookup.

Think:

```text
Book
```

Without index:

```text
Read all pages.
```

With index:

```text
Jump directly.
```

Database indexes work similarly.

Goal:

```text
Reduce disk reads.
```

Because:

```text
Disk I/O >> CPU Cost
```

---

# 3. B+ Tree Internals

Most relational databases use:

```text
B+ Tree
```

for indexes.

Examples:

```text
MySQL InnoDB
PostgreSQL
SQL Server
Oracle
```

---

# Why Not Binary Search Tree?

Normal BST:

```text
50
 \
  60
    \
     70
       \
        80
```

Can become:

```text
O(N)
```

in worst case.

---

# Why Not Red-Black Tree?

Memory efficient.

But databases store data on disk.

Disk access is expensive.

Need fewer disk reads.

---

# B+ Tree Design Goal

Minimize:

```text
Disk I/O
```

instead of:

```text
CPU Operations
```

---

## Structure

Example:

```text
                [40]
              /      \
       [10,20,30]   [50,60,70]
```

Internal nodes:

```text
Store keys only.
```

Leaf nodes:

```text
Store actual row references.
```

---

## Realistic Example

```text
                   [100]
               /           \
         [20,40,60]      [120,140]
         /    |    \         |
```

Leaves:

```text
10
20
30

40
50
60

70
80
90

100
110
120
```

---

# Key Property

Leaf nodes are linked.

```text
10 -> 20 -> 30 -> 40 -> 50 -> 60
```

This is why:

```sql
WHERE id BETWEEN 1000 AND 5000
```

is extremely efficient.

---

# Search Complexity

Height usually:

```text
3 or 4
```

even for billions of rows.

Lookup:

```text
O(log n)
```

---

# Example Search

```sql
SELECT *
FROM employee
WHERE id=4500;
```

Traversal:

```text
Root
 ↓
Intermediate Node
 ↓
Leaf Node
 ↓
Record
```

Typically:

```text
3-4 page reads
```

instead of:

```text
Millions
```

---

# B+ Tree Insertion

Insert:

```text
55
```

Leaf full?

Split.

Example:

Before:

```text
[10 20 30 40]
```

After:

```text
[10 20]
[30 40]
```

Parent updated.

This keeps tree balanced.

---

# B+ Tree Deletion

Delete value.

If node underflows:

```text
Borrow from sibling
```

or

```text
Merge nodes
```

Tree remains balanced.

---

# Why B+ Tree Over Hash Index?

Hash:

```text
WHERE id=10
```

Excellent.

But:

```sql
WHERE id > 10
```

Poor.

B+ Tree supports:

```text
=
>
<
BETWEEN
ORDER BY
GROUP BY
```

efficiently.

---

# 4. Clustered Index

Most misunderstood topic in interviews.

---

# Definition

A Clustered Index determines:

```text
Physical storage order
of rows on disk.
```

---

Example:

```sql
CREATE TABLE employee(
   id BIGINT PRIMARY KEY
);
```

In InnoDB:

```text
PRIMARY KEY
=
Clustered Index
```

---

Rows stored physically:

```text
1
2
3
4
5
6
7
```

ordered by:

```text
id
```

---

# Visualization

Without Clustered Index

```text
10
2
90
5
1
```

Random storage.

---

With Clustered Index

```text
1
2
5
10
90
```

Stored sorted.

---

# Why Important?

Query:

```sql
SELECT *
FROM employee
WHERE id BETWEEN 1 AND 1000;
```

Database reads:

```text
Continuous Pages
```

Very fast.

---

# Characteristics

Only:

```text
ONE
```

clustered index per table.

Reason:

```text
Data can be physically sorted
only one way.
```

---

# Primary Key Lookup

```sql
SELECT *
FROM employee
WHERE id=100;
```

Traversal:

```text
Clustered Index
 ↓
Row Found
```

No extra lookup.

---

# 5. Non-Clustered Index

Example:

```sql
CREATE INDEX idx_name
ON employee(name);
```

Data remains:

```text
Sorted by Primary Key
```

Index separately stores:

```text
name → primary key
```

Example:

```text
Alex -> 4
John -> 7
Mike -> 2
```

---

Query:

```sql
SELECT *
FROM employee
WHERE name='Alex';
```

Steps:

```text
Index Lookup
      ↓
Get PK
      ↓
Clustered Index Lookup
      ↓
Actual Row
```

Called:

```text
Double Lookup
```

or

```text
Bookmark Lookup
```

---

# 6. Composite Index

Definition:

```text
Single index built on
multiple columns.
```

Example:

```sql
CREATE INDEX idx_emp
ON employee(department,salary);
```

---

Stored As

```text
(Engineering,50000)
(Engineering,60000)
(Engineering,70000)

(HR,40000)
(HR,50000)
```

Sorted first by:

```text
department
```

then:

```text
salary
```

---

# Leftmost Prefix Rule

Most important interview concept.

Index:

```sql
(department,salary)
```

Works for:

```sql
WHERE department='Engineering'
```

YES

---

```sql
WHERE department='Engineering'
AND salary=50000
```

YES

---

```sql
WHERE department='Engineering'
AND salary>50000
```

YES

---

```sql
WHERE salary=50000
```

NO (usually)

because first column missing.

---

# Why?

Index order:

```text
department -> salary
```

Database cannot directly jump using salary alone.

---

# Good Composite Index

Query:

```sql
WHERE country='IN'
AND city='Bangalore'
```

Index:

```sql
(country, city)
```

Excellent.

---

# Bad Composite Index

Index:

```sql
(city,country)
```

when most queries filter:

```sql
WHERE country='IN'
```

Suboptimal.

---

# Covering Index

Example:

```sql
SELECT department,salary
FROM employee
WHERE department='Engineering';
```

Index:

```sql
(department,salary)
```

Everything available in index.

No table lookup required.

Huge optimization.

---

# 7. Query Optimization

The optimizer decides:

```text
Best execution plan.
```

It answers:

```text
Which index?
Join order?
Scan type?
```

---

Input:

```sql
SELECT *
FROM employee
WHERE department='Engineering';
```

Possible choices:

```text
Full Scan
Index Scan
Index Seek
```

Optimizer picks cheapest.

---

# Query Execution Lifecycle

```text
SQL Query
   ↓
Parser
   ↓
Optimizer
   ↓
Execution Plan
   ↓
Execution Engine
   ↓
Result
```

---

# 8. Cost Based Optimizer (CBO)

Modern databases use:

```text
Cost Based Optimizer
```

Not rule-based.

---

Optimizer estimates:

```text
Rows
CPU
Disk I/O
Memory
Network
```

---

Example

Table:

```text
100 Million Rows
```

Query:

```sql
WHERE gender='M'
```

50% rows match.

Optimizer may choose:

```text
Table Scan
```

instead of index.

---

Why?

Reading half table through index is slower.

---

# Selectivity

Definition:

```text
How unique a column is.
```

---

High Selectivity

```text
email
aadhaar_number
employee_id
```

Excellent index candidates.

---

Low Selectivity

```text
gender
status
boolean flags
```

Poor index candidates.

---

# 9. EXPLAIN Plan Analysis

Use:

```sql
EXPLAIN
SELECT *
FROM employee
WHERE id=100;
```

Shows execution plan.

---

Common Terms

## Index Seek

Best.

```text
Directly jumps.
```

Complexity:

```text
O(log n)
```

---

## Index Scan

Reads many index entries.

Not as good.

---

## Table Scan

Worst.

Reads entire table.

---

# Example

```sql
EXPLAIN
SELECT *
FROM employee
WHERE department='Engineering';
```

Possible output:

```text
Index Seek
```

meaning optimizer used index.

---

# 10. Real Production Examples

---

## Example 1

```sql
SELECT *
FROM users
WHERE email='abc@gmail.com';
```

Index:

```sql
(email)
```

Excellent.

---

## Example 2

```sql
SELECT *
FROM orders
WHERE user_id=10
AND status='PAID';
```

Index:

```sql
(user_id,status)
```

Excellent.

---

## Example 3

```sql
SELECT *
FROM orders
WHERE status='PAID';
```

Index on status alone:

Usually poor.

Reason:

```text
Low Selectivity
```

---

# 11. Common Interview Questions

---

## Why B+ Tree Instead of Hash?

Answer:

```text
Supports range queries,
sorting,
order traversal,
BETWEEN,
ORDER BY.
```

---

## Why Only One Clustered Index?

Answer:

```text
Rows can be physically ordered
only one way.
```

---

## What is Leftmost Prefix Rule?

Answer:

```text
Composite index can only be used
starting from its leftmost column.
```

---

## Clustered vs Non-Clustered?

| Clustered | Non Clustered |
|------------|------------|
| Stores actual rows | Stores pointers |
| One per table | Multiple |
| Faster lookup | Extra lookup required |

---

## What is Covering Index?

Answer:

```text
Query can be satisfied entirely
from index without reading table.
```

---

## What is Selectivity?

Answer:

```text
Uniqueness of indexed values.
Higher selectivity generally means
better index performance.
```

---

# 12. End-to-End Query Flow

Query:

```sql
SELECT *
FROM employee
WHERE id=500;
```

Flow:

```text
SQL Arrives
      ↓
Parser
      ↓
Optimizer
      ↓
Choose Clustered Index
      ↓
B+ Tree Traversal
      ↓
Root Page
      ↓
Internal Page
      ↓
Leaf Page
      ↓
Row Found
      ↓
Return Result
```

Disk Reads:

```text
3-4 Pages
```

instead of:

```text
Millions of Rows
```

---

# 13. Summary

### B+ Tree

- Balanced tree structure
- Optimized for disk storage
- O(log n) lookup
- Supports range scans
- Used by most databases

### Clustered Index

- Determines physical row order
- Usually Primary Key in InnoDB
- One per table
- Fastest lookup path

### Composite Index

- Multiple columns in one index
- Follows Leftmost Prefix Rule
- Great for multi-column filtering
- Can become a Covering Index

### Query Optimization

- Optimizer chooses cheapest execution plan
- Uses statistics and cost estimation
- May choose table scan over index scan
- Selectivity heavily influences decisions

### Golden Interview Statement

> A query typically gets optimized by the Cost-Based Optimizer, which evaluates available indexes (often implemented as B+ Trees), estimates costs using statistics, and chooses the most efficient execution plan such as an Index Seek, Index Scan, or Table Scan.
