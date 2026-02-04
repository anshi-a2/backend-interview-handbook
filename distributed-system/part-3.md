
## 1. What Is an Index and Why It Matters

An **index** is a data structure that **improves read performance** by avoiding full table scans.

> Think of it like a book’s index — you don’t read every page to find a topic.


---

## 2. How Indexes Work (High-Level)

Without index:

```text
SELECT * FROM orders WHERE user_id = 123;
→ Full table scan (O(N))
```

With index:

```text
Index lookup → Pointer to row
→ O(log N) or O(1)
```

---

## 3. B-Tree Index vs Hash Index

### B-Tree Index (Most Common)

**Used by:** MySQL (InnoDB), PostgreSQL, Oracle

**Characteristics:**

* Balanced tree
* Sorted order
* Supports range queries

**Good for:**

* `=`, `<`, `>`, `BETWEEN`
* `ORDER BY`
* `LIKE 'abc%'`

**Example:**

```sql
SELECT * FROM orders WHERE created_at BETWEEN '2024-01-01' AND '2024-01-31';
```

---

### Hash Index

**Characteristics:**

* Key → hash → bucket
* No ordering
* Very fast equality lookup

**Good for:**

* Exact match (`=` only)

**Limitations:**

* No range queries
* No sorting

---

### Comparison Table

| Aspect                 | B-Tree | Hash      |
| ---------------------- | ------ | --------- |
| Supports range queries | Yes    | No        |
| Supports ORDER BY      | Yes    | No        |
| Equality lookup        | Fast   | Very fast |
| Most DB default        | Yes    | Rare      |


---

## 4. Composite Index (Multi-Column Index)

### What Is a Composite Index?

An index on **multiple columns**:

```sql
CREATE INDEX idx_user_status_date
ON orders(user_id, status, created_at);
```

---

### Composite Index Order (Very Important)

Index follows the **leftmost prefix rule**.

Valid usages:

* `(user_id)`
* `(user_id, status)`
* `(user_id, status, created_at)`

Invalid usages:

* `(status)`
* `(status, created_at)`

---

### How to Decide Column Order

Order columns by:

1. **Highest selectivity first**
2. Equality conditions before range conditions

---

### Example

Query:

```sql
SELECT * FROM orders
WHERE user_id = 10 AND status = 'PAID'
ORDER BY created_at DESC;
```

Best index:

```text
(user_id, status, created_at)
```

---

## 5. Index Selectivity

### Definition

**Selectivity** = fraction of rows filtered by the index.

```text
selectivity = distinct_values / total_rows
```

---

### High vs Low Selectivity

| Column    | Selectivity | Index Useful? |
| --------- | ----------- | ------------- |
| user_id   | High        | ✅ Yes         |
| order_id  | Very High   | ✅ Yes         |
| gender    | Low         | ❌ Usually no  |
| is_active | Very Low    | ❌ Often no    |

---

## 6. When Indexes Hurt Writes

### Why Writes Become Slower

Every write must:

* Insert/update table row
* Update all related indexes

> More indexes = more write overhead

---

### Operations Impacted

| Operation               | Impact      |
| ----------------------- | ----------- |
| INSERT                  | Slower      |
| UPDATE (indexed column) | Much slower |
| DELETE                  | Slower      |

---

### Example

```text
Table with 6 indexes
INSERT → 1 row + 6 index updates
```

---

### When Indexing Hurts

* Write-heavy systems
* High-frequency updates
* Low-selectivity indexes

**Bad example:**

```sql
INDEX(is_active)
```

---

## 7. Indexing in Banking Systems

### Core Ledger Tables

* Index on account_id
* Index on transaction_id
* Carefully controlled index count

**Why:**

* Reads must be fast
* Writes must still be safe

---

### Audit & Reporting Tables

* More indexes allowed
* Read-heavy

---


✅ Optimized for SDE-2 / Backend / System Design interviews.
