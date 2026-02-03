
## 1. What is a Consistency Model?

A **consistency model** defines the **guarantees a distributed system provides about the visibility and ordering of data updates** across multiple nodes.

In simpler terms:

> *If I write some data, when and how will other users/threads/services see it?*

---

## 2. Strong Consistency vs Eventual Consistency

### Strong Consistency

**Definition:**
Once a write is acknowledged, **all subsequent reads return the latest value**.

**Characteristics:**

* Linearizable behavior
* Feels like a single-node system
* Easier to reason about
* Slower writes & reads due to coordination

**Example:**

```text
Balance = 100
User A withdraws 20 → Balance = 80
Immediately, any read returns 80
```

**Used in:**

* Banking systems
* Payment ledgers
* Order state machines

---

### Eventual Consistency

**Definition:**
Writes are propagated asynchronously. **Reads may return stale data**, but eventually all nodes converge.

**Characteristics:**

* High availability
* Low latency
* Better scalability
* Temporary inconsistencies

**Example:**

```text
User updates profile name
Some regions still see old name for a few seconds
Eventually, all see updated name
```

**Used in:**

* Social feeds
* Product catalogs
* Analytics systems

---

### Comparison Table

| Aspect         | Strong Consistency      | Eventual Consistency    |
| -------------- | ----------------------- | ----------------------- |
| Read freshness | Always latest           | May be stale            |
| Latency        | Higher                  | Lower                   |
| Availability   | Lower during partitions | High                    |
| Complexity     | Simpler logic           | Needs conflict handling |
| Typical DBs    | RDBMS, Spanner          | Cassandra, DynamoDB     |

---

## 3. Read-after-Write Consistency

### Definition

A **client-specific consistency guarantee**:

> After a client writes data, **its subsequent reads will see that write**.

This does NOT guarantee that *other clients* will see it immediately.

---

**Example:**

```text
User updates email
Immediately reloads profile page
User must see updated email
```

---

### Implementation Techniques

* Sticky sessions (same replica)
* Client-side caching
* Read-your-writes routing
* Versioning (vector clocks, timestamps)

---

### Relation to Strong vs Eventual

| Model                | Supports Read-after-Write? |
| -------------------- | -------------------------- |
| Strong consistency   | Always                     |
| Eventual consistency | Often (per-client)         |

---

## 4. What is the CAP Theorem?

**CAP Theorem** (by Eric Brewer) states that in a **distributed system**, when a **network partition** occurs, the system can guarantee **only one** of the following two:

* **C – Consistency**: All nodes return the same (latest) data
* **A – Availability**: Every request receives a response (success/failure, not timeout)

👉 **Partition Tolerance (P)** is non-negotiable in real distributed systems.

> So the real choice is always **CP vs AP**.

---

## 5. What Is a Network Partition (P)?

### Definition

A **partition** happens when:

* Nodes are alive
* But **cannot communicate** with each other

This could be due to:

* Network failure
* Data center outage
* Packet loss / latency spikes

---

### What Exactly Breaks During a Partition?

During partition:

* Replicas cannot sync writes
* Nodes have **diverging state**
* System must decide **how to respond to requests**

**Key breaking point:**

> You cannot know if another node has newer data.

---

## 6. Why You Must Choose Between C and A During Partition

### Option 1: Choose Consistency (CP)

**Behavior:**

* Reject requests if latest data cannot be guaranteed
* System may become partially unavailable

**Result:**

* No stale reads
* Correctness preserved

---

### Option 2: Choose Availability (AP)

**Behavior:**

* Serve requests from any reachable node
* May return stale or conflicting data

**Result:**

* System always responds
* Temporary inconsistencies

---

## 7. Why CA Is Impossible in Distributed Systems

### CA = Consistency + Availability

To maintain both:

* All nodes must agree on data
* All requests must be served

---

### What Breaks CA?

During partition:

* Node A cannot reach Node B
* If A serves request → might be stale ❌ (breaks Consistency)
* If A waits for B → request times out ❌ (breaks Availability)

> There is **no third option**.

Hence:

> ❌ **CA systems cannot exist once partition happens**

---

### Important Interview Line

> "CA is achievable only in a single-node or non-partitioned system, not in a real distributed system."

---

## 8. How Do Systems Achieve Consistency or Availability?

### Achieving Consistency (CP Systems)

Techniques:

* Quorum-based writes (majority consensus) - Quorum-based writes ensure consistency in replicated systems by requiring a majority of nodes to acknowledge a write, guaranteeing that any subsequent read overlapping the quorum will see the latest value. (R + W > N)
* Distributed transactions - Either everything commits or everything rolls back — across systems.
* Leader-based replication - Leader-based replication is a data replication strategy where one node (Leader/Primary) handles all writes, and replica/follower nodes replicate data from the leader and usually serve reads.
* Synchronous replication - write is considered successful only after it has been persisted on the leader and acknowledged by one or more replicas.

**Trade-off:**

* Higher latency
* Possible request failures

---

### Achieving Availability (AP Systems)

Techniques:

* Async replication
* Multi-leader or leaderless setups
* Conflict resolution (last-write-wins, vector clocks)
* Read-local, write-local

**Trade-off:**

* Stale reads
* Conflict handling complexity

---

## 9. Real Examples – Banking Systems

### Why Banking Is Special

* Money must not be duplicated
* Double-spend is unacceptable
* Regulatory & audit constraints

---

### What Happens During Partition in Banks?

* Transactions may fail
* System may block writes
* Read-only mode may be enabled

**But:**

> Incorrect balance is **never** shown

---

### Example Scenario

```text
Account balance = ₹10,000
Network partition occurs

Option A: Allow withdrawal using stale replica ❌ (money loss)
Option B: Reject transaction ✅ (temporary unavailability)
```

Banks always choose **Option B**.

---

## 8. Where Do Banks Fall on CAP?

### Interview-Grade Answer

> **Banks are CP systems (Consistency + Partition Tolerance)**

---

### CAP Mapping for Banking

| System Component    | CAP Choice | Reason                  |
| ------------------- | ---------- | ----------------------- |
| Core banking ledger | CP         | Financial correctness   |
| Payment settlement  | CP         | No stale reads          |
| ATM withdrawal      | CP         | Prevent double-spend    |
| Analytics / reports | AP         | Slight delay acceptable |

---

