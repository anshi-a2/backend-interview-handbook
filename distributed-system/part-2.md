
## 1. What Is Concurrency in a Database?

**Concurrency** refers to **multiple transactions accessing or modifying data at the same time**.

In backend systems (especially banking), concurrency is unavoidable due to:

* Multiple users
* Parallel services
* Distributed deployments

> Goal: **Maximize parallelism without breaking data correctness**.

---

## 2. Why Concurrency Control Is Needed

Without control, concurrent transactions can lead to:

* Incorrect balances
* Lost updates
* Reading invalid data

> Concurrency control ensures **ACID properties**, especially **Isolation**.

---

## 3. Isolation Levels (SQL Standard)

**Isolation** defines *how visible one transaction’s changes are to others*.

### SQL Isolation Levels (from weakest to strongest)

| Level            | Prevents               | Allows                    |
| ---------------- | ---------------------- | ------------------------- |
| Read Uncommitted | Nothing                | Dirty reads, lost updates |
| Read Committed   | Dirty reads            | Non-repeatable reads      |
| Repeatable Read  | Dirty + non-repeatable | Phantom reads             |
| Serializable     | All anomalies          | None                      |

---

## 4. Common Concurrency Problems

### 4.1 Dirty Read

**Definition:**
Reading data written by a transaction that has **not yet committed**.

**Example:**

```text
T1: Update balance to 500 (not committed)
T2: Reads balance = 500
T1: Rolls back
→ T2 read invalid data
```

❌ **Unacceptable in banking**

---

### 4.2 Lost Update

**Definition:**
Two transactions update the same data, and **one overwrites the other**.

**Example:**

```text
Balance = 1000
T1: Read 1000 → subtract 100 → write 900
T2: Read 1000 → subtract 200 → write 800

Final balance = 800 (lost T1 update)
```

❌ Leads to money loss

---

### 4.3 Non-Repeatable Read

**Definition:**
Same row read twice gives different values due to another committed transaction.

---

### 4.4 Phantom Read

**Definition:**
Re-running a query returns a different *set of rows*.

---

## 5. How Isolation Levels Prevent These Issues

| Problem             | Min Isolation Level Needed |
| ------------------- | -------------------------- |
| Dirty read          | Read Committed             |
| Lost update         | Repeatable Read / Locks    |
| Non-repeatable read | Repeatable Read            |
| Phantom read        | Serializable               |

---

## 6. Locking Mechanisms

### 6.1 Pessimistic Locking

**Idea:**

> "Assume conflict will happen → lock first"

**How it works:**

* Locks rows/tables before update
* Other transactions must wait

**SQL Example:**

```sql
SELECT balance FROM account WHERE id=1 FOR UPDATE;
```

**Pros:**

* Strong correctness
* Prevents lost updates

**Cons:**

* Reduced throughput
* Risk of deadlocks

---

### 6.2 Optimistic Locking

**Idea:**

> "Assume conflicts are rare → detect later"

**How it works:**

* No locks during read
* Uses version / timestamp
* Fails if data changed before update

**Example:**

```sql
UPDATE account
SET balance = ?, version = version + 1
WHERE id = ? AND version = ?;
```

If rows updated = 0 → conflict detected.

**Pros:**

* High concurrency
* No blocking

**Cons:**

* Retry logic required

---

## 7. Optimistic vs Pessimistic Locking (Comparison)

| Aspect           | Optimistic          | Pessimistic        |
| ---------------- | ------------------- | ------------------ |
| Assumption       | Conflicts rare      | Conflicts frequent |
| Locking          | No                  | Yes                |
| Performance      | High                | Lower              |
| Failure handling | Retry               | Wait               |
| Typical use      | Reads-heavy systems | Money movement     |

---

## 8. Where Each Is Used in Banking Systems

### Core Banking Ledger

* Isolation: **Serializable / Repeatable Read**
* Locking: **Pessimistic**
* Reason: No lost updates, no dirty reads

---

### Account Balance Updates

* Uses **row-level locks**
* Prevents double withdrawal

---

### Transaction History / Statements

* Isolation: **Read Committed**
* Locking: None
* Reason: Slight delay acceptable

---

### User Profile / KYC Data

* Locking: **Optimistic**
* Reason: Conflicts rare

---

### Summary Table (Banking)

| Component   | Isolation       | Locking     |
| ----------- | --------------- | ----------- |
| Ledger      | Serializable    | Pessimistic |
| Payments    | Repeatable Read | Pessimistic |
| Statements  | Read Committed  | None        |
| Profile/KYC | Read Committed  | Optimistic  |

---


## 9. SQL vs NoSQL – First-Level Decision

### Use SQL When

* Strong consistency is required
* Complex joins & transactions exist
* Schema is stable

### Use NoSQL When

* Massive scale
* High write/read throughput
* Flexible schema

---

## 10. MySQL vs PostgreSQL

### Core Difference

| Aspect         | MySQL                   | PostgreSQL                 |
| -------------- | ----------------------- | -------------------------- |
| Philosophy     | Simplicity & speed      | Correctness & features     |
| SQL compliance | Partial                 | Very strong                |
| Transactions   | Strong                  | Strong                     |
| Performance    | Faster for simple reads | Better for complex queries |
| Extensibility  | Limited                 | Highly extensible          |

---

### When to Use MySQL

* Simple CRUD applications
* Read-heavy workloads
* SaaS products
* When operational simplicity matters

**Examples:**

* User management
* E-commerce catalog
* CMS systems

---

### When to Use PostgreSQL

* Complex queries
* Financial systems
* Analytics-heavy workloads
* JSON + relational hybrid needs

**Examples:**

* Banking systems
* Reporting platforms
* Geo-spatial apps

---

## 11. Cassandra vs MongoDB

### Data Model

| Aspect       | Cassandra       | MongoDB             |
| ------------ | --------------- | ------------------- |
| Model        | Wide-column     | Document (JSON)     |
| Schema       | Fixed per table | Flexible            |
| Joins        | No              | Limited             |
| Transactions | Very limited    | Multi-doc (limited) |

---

### Cassandra – When to Use

**Strengths:**

* Massive write throughput
* Horizontal scalability
* High availability (AP system)

**Use Cases:**

* Time-series data
* Event logging
* Metrics & monitoring

**Example:**

* IoT sensor data
* Clickstream analytics

---

### MongoDB – When to Use

**Strengths:**

* Flexible schema
* Faster development
* JSON-like documents

**Use Cases:**

* Content management
* User profiles
* Product catalogs

---

## 12. Redis vs Memcached

### Core Purpose

Both are **in-memory key-value stores**, mainly used for caching.

---

### Feature Comparison

| Aspect          | Redis                  | Memcached        |
| --------------- | ---------------------- | ---------------- |
| Data structures | Rich (list, set, zset) | Simple key-value |
| Persistence     | Yes (RDB/AOF)          | No               |
| Replication     | Yes                    | No               |
| Transactions    | Yes                    | No               |
| Pub/Sub         | Yes                    | No               |

---

### When to Use Redis

* Distributed caching
* Session storage
* Rate limiting
* Leaderboards

**Why backend teams love Redis:**

* Can act as DB + cache + queue

---

### When to Use Memcached

* Extremely simple caching
* Temporary data
* Minimal overhead

---

### Interview Line

> "Redis is chosen when we need richer data structures or durability, while Memcached fits very simple, ephemeral caching."

---

## 13. When NOT to Use NoSQL

This is a **very important interview question**.

---

### Avoid NoSQL When

* Strong ACID transactions are required
* Data integrity is critical
* Complex joins are needed
* Reporting & analytics are heavy

---

### Banking Example

❌ Using NoSQL for core ledger

* No joins
* Eventual consistency
* Hard to enforce constraints

✅ Use RDBMS instead

---

### Common Anti-Patterns

* Choosing NoSQL "for scale" prematurely
* Using MongoDB as a replacement for RDBMS
* Ignoring consistency needs

---

## 14. Decision Table (Quick Recall)

| Scenario           | Recommended DB      |
| ------------------ | ------------------- |
| Banking ledger     | PostgreSQL / Oracle |
| User profiles      | MongoDB             |
| Product catalog    | MySQL / MongoDB     |
| Time-series data   | Cassandra           |
| Caching / sessions | Redis               |
| Simple cache       | Memcached           |

---



