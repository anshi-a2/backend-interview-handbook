# Database Indexing & Query Optimization – SDE-2 Interview Notes

## 1. Why Indexing Exists

Indexes are used to **reduce the amount of data scanned** by the database while executing a query.

* Without index → Full table scan (O(N))
* With index → Tree lookup (O(log N))

Indexes improve **read performance** at the cost of **write overhead**.

---

## 2. What is an Index (Internals)

An index is a **separate data structure** that stores:

* Indexed column value
* Pointer to the actual table row

Key points:

* Index ≠ table data
* Index consumes additional memory
* Index must be updated on INSERT/UPDATE/DELETE

---

## 3. B+ Tree (Why Databases Use It)

Most relational databases use **B+ Trees** for indexing.

### Why not Hash Index?

* Hash index ❌ does not support range queries
* B+ Tree ✅ supports range, sorting, and prefix queries

### B+ Tree properties

* Balanced tree
* All data stored at leaf nodes
* Leaf nodes are linked (fast range scan)

Supports:

* `=` `>` `<` `BETWEEN`
* `ORDER BY`

---

## 4. Types of Indexes (Must Know)

### 4.1 Primary / Clustered Index

* Usually created on **primary key**
* Determines physical data order
* Only **one clustered index** per table

```sql
PRIMARY KEY (order_id)
```

In MySQL InnoDB, the primary key is the clustered index.

---

### 4.2 Secondary / Non-Clustered Index

* Separate structure from table
* Points to primary key

```sql
CREATE INDEX idx_user_id ON orders(user_id);
```

---

### 4.3 Composite Index (Very Important)

```sql
CREATE INDEX idx_user_status ON orders(user_id, status);
```

⚠️ Order matters

Works for:

* `(user_id)`
* `(user_id, status)`

Does NOT work for:

* `(status)` alone

---

## 5. Cardinality

**Cardinality** = number of unique values in a column.

| Column  | Cardinality |
| ------- | ----------- |
| gender  | Low         |
| status  | Low         |
| user_id | High        |

Indexes are most effective on **high-cardinality columns**.

---

## 6. When Index is NOT Used

Indexes may be ignored in the following cases:

### 6.1 Function on Column

```sql
WHERE LOWER(email) = 'a@b.com'
```

### 6.2 Leading Wildcard

```sql
WHERE name LIKE '%abc'
```

### 6.3 Type Mismatch

```sql
WHERE user_id = '123'  -- user_id is INT
```

### 6.4 Small Tables

Query optimizer may prefer full scan.

---

## 7. Query Optimization Basics

### 7.1 Use EXPLAIN

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 10;
```

Key fields:

* `type` → ALL = full table scan (bad)
* `key` → index used
* `rows` → estimated rows scanned

Goal: minimize rows scanned.

---

### 7.2 Avoid SELECT *

❌ Bad:

```sql
SELECT * FROM orders;
```

✔ Good:

```sql
SELECT order_id, status FROM orders;
```

---

### 7.3 Avoid N+1 Query Problem

❌ Problem:

```java
for (Order o : orders) {
    fetchOrderItems(o.id);
}
```

✔ Solutions:

* JOIN queries
* Batch fetching
* Hibernate fetch join

---

## 8. Index vs Write Performance

Each index:

* Slows INSERT
* Slows UPDATE
* Slows DELETE

Index only columns that are:

* Frequently queried
* Used in WHERE / ORDER BY

---

## 9. Covering Index (Advanced)

A **covering index** contains all columns required by the query.

```sql
CREATE INDEX idx_user_order
ON orders(user_id, status, created_at);
```

Query:

```sql
SELECT status, created_at FROM orders WHERE user_id = 10;
```

✔ No table lookup required
✔ Faster execution

---

## 10. ORDER BY and Index

Index helps ORDER BY only if:

* Column order matches index
* Sort direction matches

```sql
ORDER BY created_at DESC
```

Index:

```sql
(created_at DESC)
```

---

## 11. Pagination Optimization

❌ Offset-based pagination:

```sql
LIMIT 100000, 20
```

✔ Cursor-based pagination:

```sql
WHERE id > last_seen_id
LIMIT 20
```

More scalable for large datasets.

---

## 12. Real Backend Example

Query:

```sql
SELECT * FROM orders
WHERE user_id = ?
AND status = 'PLACED'
ORDER BY created_at DESC
LIMIT 20;
```

Optimal index:

```sql
(user_id, status, created_at DESC)
```

---

## 13. Common Interview Questions

**Q: How many indexes are too many?**
When write performance degrades noticeably.

**Q: Does index always improve performance?**
No, not for small tables or low-cardinality columns.

**Q: Clustered vs Non-clustered?**
Clustered defines physical row order.

---

## 14. Interview One-Liners

* "Indexes trade write performance for read speed."
* "Composite index order matters."
* "Use EXPLAIN to validate index usage."
* "Avoid SELECT * in production queries."

---

## 15. SDE-2 Tip

In interviews:

1. Explain why index exists
2. Talk about B+ tree
3. Discuss composite index & EXPLAIN

This signals strong backend fundamentals.
