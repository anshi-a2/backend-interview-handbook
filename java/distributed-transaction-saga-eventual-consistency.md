# Distributed Transactions, Saga & Eventual Consistency – SDE-2 Interview Notes

## 1. Why Distributed Transactions Exist

In real-world backend systems, different services own their own databases:

* Order Service → Order DB
* Inventory Service → Inventory DB
* Payment Service → Payment DB

A single ACID transaction **cannot span multiple databases**.

This leads to the **Distributed Transaction problem**.

---

## 2. Why 2-Phase Commit (2PC) Is Avoided

### How 2PC Works (Theory)

1. Prepare phase
2. Commit phase

### Why It Fails at Scale

* Blocking protocol
* Poor performance
* Single point of failure (coordinator)
* Low availability

> Modern high-scale systems avoid 2PC in production.

---

## 3. Eventual Consistency (Key Concept)

Instead of immediate consistency, systems:

* Become consistent **over time**
* Allow temporary inconsistencies

Example:

* Order created
* Inventory reserved shortly after
* Payment confirmed later

This is acceptable for scalable systems.

---

## 4. Saga Pattern (Core Solution)

A **Saga** is a sequence of **local transactions**, where each step has a **compensating transaction**.

Instead of rollback:

* Perform an **undo** action

---

## 5. Order Placement as a Saga

### Normal Flow

1. Create Order
2. Reserve Inventory
3. Process Payment
4. Confirm Order

### Failure Handling

* If payment fails → release inventory
* If inventory fails → cancel order

---

## 6. Types of Saga Patterns

### 6.1 Choreography Saga

Each service reacts to events.

Flow:

* OrderCreated event
* InventoryService reserves stock
* InventoryReserved event
* PaymentService charges

#### Pros

* Decoupled
* Simple to start

#### Cons

* Hard to debug
* Event explosion

---

### 6.2 Orchestration Saga (Recommended)

A central orchestrator controls the flow.

Flow:

* Order Service calls Inventory
* Inventory responds
* Order Service calls Payment
* On failure, triggers compensation

#### Pros

* Clear control flow
* Easier debugging
* Preferred in interviews

---

## 7. Saga Flow Example

```
CREATE_ORDER
   ↓
RESERVE_INVENTORY
   ↓
PAYMENT_SUCCESS ?
   ↓ yes              ↓ no
CONFIRM_ORDER    RELEASE_INVENTORY
```

---

## 8. Compensation Transactions

Each step must have an undo operation:

| Step              | Compensation      |
| ----------------- | ----------------- |
| Create Order      | Cancel Order      |
| Reserve Inventory | Release Inventory |
| Charge Payment    | Refund Payment    |

---

## 9. Idempotency (Critical for Reliability)

Failures cause retries, retries cause duplicates.

Solution:

* Idempotency keys
* Deduplication logic

Example:

```http
POST /payments
Idempotency-Key: order_123
```

---

## 10. Handling Partial Failures

| Scenario         | Handling                |
| ---------------- | ----------------------- |
| Inventory down   | Retry / Circuit breaker |
| Payment timeout  | Retry payment           |
| Duplicate events | Idempotent consumers    |
| Consumer crash   | Retry / DLQ             |

---

## 11. Eventual Consistency Mindset

> Systems prioritize availability and scalability over immediate consistency.

Temporary inconsistency is resolved using compensations.

---

## 12. Common Interview Mistakes

❌ Using @Transactional across services
❌ Expecting global rollback
❌ Demanding strict consistency

✔ Using Saga pattern
✔ Using compensation logic
✔ Accepting eventual consistency

---

## 13. When to Use What

| Scenario                 | Approach              |
| ------------------------ | --------------------- |
| Single database          | ACID transaction      |
| Multiple databases       | Saga pattern          |
| Money-related operations | Idempotency + retries |
| High-scale systems       | Eventual consistency  |

---

## 14. Interview One-Liners

* "Distributed transactions do not scale."
* "Saga replaces rollback with compensation."
* "Eventual consistency is a design choice."
* "Orchestration Saga is easier to reason about."

---

## 15. SDE-2 Takeaway

For order and inventory systems:

* Avoid 2PC
* Use Saga pattern
* Implement idempotency
* Design for failures
