# Transactions, Isolation Levels & Locking – SDE-2 Interview Notes

## 1. What is a Transaction?

A **transaction** is a sequence of database operations executed as a **single logical unit of work**.

Either:

* All operations succeed → COMMIT
* Any operation fails → ROLLBACK

### Example (Order System)

* Create Order
* Reserve Inventory
* Deduct Payment
* Confirm Order

Failure in any step should rollback everything.

---

## 2. ACID Properties (Must-Know)

### A — Atomicity

* All or nothing execution
* Prevents partial updates

### C — Consistency

* Database moves from one valid state to another
* Constraints are preserved

### I — Isolation

* Concurrent transactions do not interfere incorrectly

### D — Durability

* Committed data survives crashes

---

## 3. Transactions in Spring Boot

```java
@Transactional
public void placeOrder() {
    createOrder();
    reserveInventory();
    chargePayment();
}
```

* Commit on success
* Rollback on runtime exception

---

## 4. Isolation Levels (SQL Standard)

### 4.1 READ UNCOMMITTED

* Dirty reads allowed
* Rarely used

---

### 4.2 READ COMMITTED (Default in most DBs)

* Only committed data is visible
* Prevents dirty reads
* Allows non-repeatable reads

---

### 4.3 REPEATABLE READ (MySQL Default)

* Same row read remains consistent
* Prevents dirty + non-repeatable reads
* Phantom reads possible

---

### 4.4 SERIALIZABLE

* Transactions behave sequentially
* Highest isolation
* Lowest concurrency

---

## 5. Read Phenomena

### Dirty Read

* Reading uncommitted data

### Non-repeatable Read

* Same row changes between reads

### Phantom Read

* New rows appear between queries

---

## 6. Locking Basics

### Shared Lock (S)

* Used for read operations
* Multiple transactions allowed

### Exclusive Lock (X)

* Used for write operations
* Only one transaction allowed

---

## 7. Row-Level vs Table-Level Locking

| Lock Type   | Scope        | Performance |
| ----------- | ------------ | ----------- |
| Row-level   | Single row   | High        |
| Table-level | Entire table | Low         |

InnoDB supports row-level locking.

---

## 8. Inventory Reservation (Concurrency Safe)

### ❌ Unsafe Approach

```sql
SELECT available_qty FROM inventory;
UPDATE inventory SET available_qty = available_qty - 1;
```

### ✔ Safe Atomic Update

```sql
UPDATE inventory
SET available_qty = available_qty - 1
WHERE product_id = ? AND available_qty >= 1;
```

Check affected rows to confirm success.

---

## 9. Locking Strategies

### 9.1 Pessimistic Locking

```sql
SELECT * FROM inventory WHERE product_id = ? FOR UPDATE;
```

* Locks row immediately
* Suitable for high contention

---

### 9.2 Optimistic Locking

```java
@Version
private Long version;
```

* Conflict detected at update time
* Better scalability

---

## 10. Deadlocks

### Scenario

* Tx1 locks A → waits for B
* Tx2 locks B → waits for A

Database resolves by aborting one transaction.
Backend should retry.

---

## 11. Transaction Propagation (Spring)

### REQUIRED (Default)

* Join existing transaction or create new

### REQUIRES_NEW

* Suspends parent transaction
* Starts new transaction

### NESTED

* Uses savepoints
* Partial rollback support

---

## 12. Common Backend Mistakes

* Long-running transactions
* External API calls inside transactions
* Overusing SERIALIZABLE isolation
* No retry logic for deadlocks

---

## 13. Interview One-Liners

* "Isolation controls visibility, not locking"
* "Optimistic locking scales better"
* "Atomic updates prevent race conditions"
* "Transactions should be short-lived"

---

## 14. Backend Design Tip

For order & inventory systems:

* Use READ COMMITTED
* Use atomic update queries
* Implement retries for deadlocks

This balances consistency and performance.
