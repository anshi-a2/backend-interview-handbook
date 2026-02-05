# Multi-Data Center Data Synchronization & Write Optimization

## 1. Problem Statement (Interview Framing)

You have **multiple data centers (DCs) in different geographic locations**. Data is **continuously updated in all DCs**, and you must ensure:

* Data stays in sync across DCs
* System scales with **huge write volume on large tables**
* Low latency for local users
* High availability even if one DC goes down

This is a **classic senior backend / distributed systems question**.

---

## 2. Core Challenges Interviewers Expect You to Identify

1. Network latency between DCs
2. Network partitions
3. Conflicting writes
4. Massive write amplification
5. Replication lag
6. Cost of syncing large tables

---

## 3. High-Level Architecture Options

```
        +-------------+        +-------------+
        |   DC-1      | <----> |    DC-2     |
        | (US-East)   |        | (EU-West)   |
        +-------------+        +-------------+
              |                      |
              v                      v
        Local Writes           Local Writes
```

Key question:

> Do we want **strong consistency** or **eventual consistency**?

For most large-scale systems → **Eventual Consistency**

---

## 4. Data Synchronization Strategies

### 4.1 Active-Active Replication (Preferred)

Both DCs accept writes.

How it works:

* Each DC writes locally
* Changes are asynchronously replicated to other DCs

Pros:

* Low latency writes
* High availability

Cons:

* Conflict resolution needed

Used by:

* Cassandra
* DynamoDB Global Tables

---

### 4.2 Active-Passive Replication

One DC is primary, others are read replicas.

Pros:

* Simpler consistency

Cons:

* Write latency for remote users
* Failover complexity

Used by:

* Traditional banking systems

---

## 5. How Data Is Synced (Core Mechanisms)

### 5.1 Change Data Capture (CDC)

Instead of syncing entire tables:

* Capture **row-level changes**
* Stream changes to other DCs

Tools:

* Debezium
* Database WAL / binlog

Why interviewers love this:

* Efficient
* Scales well

---

### 5.2 Event Streaming Layer

```
DB Write → CDC → Kafka → Consumer → Remote DB
```

Benefits:

* Back-pressure handling
* Replayability
* Loose coupling

---

## 6. Conflict Resolution (Hard Part)

When both DCs update same row:

### Common Strategies

1. **Last Write Wins (LWW)**

   * Based on timestamp
   * Simple but risky

2. **Version Vectors / Vector Clocks**

   * Detect concurrent writes
   * Used in Dynamo-style systems

3. **Business-Level Resolution**

   * Domain rules decide winner
   * Required in banking

---

## 7. Optimizing Huge Write Volumes on Large Tables

### 7.1 Sharding

Split data horizontally.

Shard key examples:

* User ID
* Region

Benefits:

* Parallel writes
* Reduced contention

---

### 7.2 Write-Ahead Log Based Replication

* Append-only writes
* Sequential I/O

Why faster:

* Avoid random disk writes

---

### 7.3 Batch Writes

Instead of per-row updates:

* Buffer
* Write in batches

Used by:

* Analytics systems

---

### 7.4 Asynchronous Writes

* Acknowledge local write
* Replicate in background

Tradeoff:

* Eventual consistency

---

### 7.5 Avoid Cross-DC Transactions

Never do distributed 2PC across DCs.

Why:

* Latency explosion
* Failure-prone

---

## 8. Database Choices (Interview Gold)

| Requirement           | Best Choice   |
| --------------------- | ------------- |
| Multi-DC writes       | Cassandra     |
| Global tables         | DynamoDB      |
| Strong consistency    | Spanner       |
| High write throughput | LSM-based DBs |

---

## 9. Consistency Model

* **Eventual consistency** across DCs
* **Strong consistency** within a single shard or DC

This hybrid model scales best.

---

## 10. Failure Handling

### DC Failure

* Traffic routed to healthy DC
* Replication resumes when DC recovers

### Network Partition

* Continue accepting writes locally
* Resolve conflicts post-partition

---

## 11. Banking vs Internet-Scale Systems

| Aspect              | Banking        | Global Internet Apps |
| ------------------- | -------------- | -------------------- |
| Consistency         | Strong         | Eventual             |
| DC Writes           | Active-Passive | Active-Active        |
| Conflict Resolution | Strict         | LWW / Vectors        |

---

## 12. Model Interview Answer (Say This)

> "I would design an active-active multi-DC system where each data center accepts local writes and uses CDC with an event streaming platform like Kafka to replicate changes asynchronously. Data would be sharded to handle high write throughput, conflicts resolved using timestamps or domain rules, and large tables optimized through append-only logs and batch writes. This ensures scalability, availability, and acceptable eventual consistency across regions."

---


## 14. When This Design Is Used

* Global user profiles
* Messaging systems
* E-commerce catalogs
* Analytics platforms

---

