# 02_Kafka_Producer_Consumer_Internals.md

> **Goal**
>
> Understand **how Kafka Producers and Consumers work internally**, how messages travel from your application to a Kafka Broker, how consumers read them efficiently, how offsets are managed, and how Kafka guarantees durability and scalability.
>
> This chapter covers the complete Producer and Consumer lifecycle, internal components, acknowledgements, retries, batching, consumer groups, rebalancing, offset management, and delivery guarantees.

---

# Table of Contents

1. Producer Architecture
2. Producer Internal Flow
3. ProducerRecord
4. Serialization
5. Partition Selection
6. RecordAccumulator
7. Sender Thread
8. Batching
9. Compression
10. ACKs
11. Retries
12. Idempotent Producer
13. Producer Transactions
14. Consumer Architecture
15. Consumer Poll Model
16. Consumer Groups
17. Consumer Coordinator
18. Offset Management
19. Offset Commit Strategies
20. Consumer Rebalancing
21. Consumer Lag
22. Delivery Semantics
23. Retry & Dead Letter Queue
24. Production Best Practices
25. Interview Summary

---

# 1. Producer Architecture

A Producer is responsible for writing records into Kafka.

```
Application

↓

Kafka Producer API

↓

Serializer

↓

Partitioner

↓

RecordAccumulator

↓

Sender Thread

↓

Kafka Broker
```

The application thread **does not directly communicate with the broker**.

Instead, Kafka uses internal buffers and background threads.

---

# 2. Producer Internal Flow

Complete lifecycle

```
Application

↓

producer.send()

↓

ProducerRecord

↓

Serializer

↓

Partition Selection

↓

RecordAccumulator

↓

Batch Creation

↓

Sender Thread

↓

TCP Connection

↓

Broker Leader

↓

ACK

↓

Callback/Future
```

This asynchronous architecture is one of the reasons Kafka achieves very high throughput.

---

# 3. ProducerRecord

Every message sent to Kafka becomes a **ProducerRecord**.

It contains

```
Topic

Partition (Optional)

Key

Value

Headers

Timestamp
```

Example

```java
ProducerRecord<String, Order> record =
        new ProducerRecord<>("orders", orderId, order);
```

---

# 4. Serialization

Kafka stores **bytes**, not Java objects.

Therefore every object must be serialized.

```
Java Object

↓

Serializer

↓

Byte[]

↓

Network
```

Common serializers

- StringSerializer
- IntegerSerializer
- LongSerializer
- ByteArraySerializer

For Spring Boot

- JsonSerializer
- Avro Serializer
- Protobuf Serializer

---

# Why Serialization?

Without serialization

```
Order Object

↓

???

↓

Broker
```

Broker cannot understand Java objects.

---

# 5. Partition Selection

Every record must go to exactly one partition.

Kafka decides using the following rules.

---

## Case 1

Partition explicitly provided

```java
send(topic, 2, key, value)
```

Always goes to

```
Partition-2
```

---

## Case 2

Key provided

```
hash(key)

↓

Partition
```

Example

```
OrderID = 101

↓

Hash

↓

Partition-1
```

This guarantees

```
Same Key

↓

Same Partition

↓

Ordering
```

---

## Case 3

No key

Kafka distributes records.

Modern Kafka uses the **Sticky Partitioner**.

Instead of changing partition every message,

it sends multiple messages to the same partition until the batch fills.

Benefits

- Larger batches
- Fewer network requests
- Better throughput

---

# 6. RecordAccumulator

One of Kafka's most important internal components.

Think of it as

```
Producer Buffer
```

Flow

```
Message

↓

RecordAccumulator

↓

Batch

↓

Sender Thread
```

Instead of sending immediately,

Kafka stores records temporarily.

Benefits

- Batching
- Better throughput
- Reduced network overhead

---

# 7. Sender Thread

Kafka Producer creates a dedicated background thread.

```
Application Thread

↓

RecordAccumulator

↓

Sender Thread

↓

Broker
```

Responsibilities

- Build batches
- Send requests
- Retry failures
- Receive ACKs

The application thread continues without waiting.

---

# 8. Batching

Instead of

```
Message

↓

Network

↓

Message

↓

Network
```

Kafka sends

```
Message

Message

Message

↓

Single Batch

↓

Network
```

Benefits

- Fewer TCP requests
- Better throughput
- Lower CPU usage

---

Important producer configs

```
batch.size

linger.ms
```

---

### batch.size

Maximum batch size.

Larger values

↓

Higher throughput

↓

Higher memory usage

---

### linger.ms

How long Kafka waits

before sending an incomplete batch.

Higher value

↓

Better batching

↓

Slightly higher latency

---

# 9. Compression

Kafka compresses batches.

Supported algorithms

- GZIP
- Snappy
- LZ4
- ZSTD

Compression occurs

```
Batch

↓

Compress

↓

Network
```

Benefits

- Lower bandwidth
- Lower storage
- Higher throughput

---

# 10. ACKs

Very common interview topic.

Producer waits for acknowledgements.

---

### ACK = 0

```
Producer

↓

Broker

(No Response)
```

Fastest.

Lowest durability.

---

### ACK = 1

```
Producer

↓

Leader

↓

ACK
```

Leader confirms.

Followers may not have replicated yet.

---

### ACK = all (-1)

```
Producer

↓

Leader

↓

Followers

↓

ISR

↓

ACK
```

Safest.

Highest durability.

Slightly higher latency.

---

# 11. Retries

Suppose

```
Producer

↓

Network Failure
```

Kafka retries automatically.

Configuration

```
retries

retry.backoff.ms
```

Be careful.

Retries can produce duplicate records unless idempotence is enabled.

---

# 12. Idempotent Producer

Classic interview question.

Problem

```
Producer

↓

Timeout

↓

Retry

↓

Duplicate Message
```

Solution

Enable

```
enable.idempotence=true
```

Kafka assigns

```
Producer ID

↓

Sequence Number
```

Broker ignores duplicates.

Benefits

- Safe retries
- No duplicate writes
- Better reliability

---

# 13. Producer Transactions

Useful when producing to multiple partitions or topics.

Without transaction

```
Topic A ✔

Topic B ✘
```

Inconsistent state.

With transaction

```
Begin Transaction

↓

Topic A

↓

Topic B

↓

Commit

OR

Rollback
```

Combined with idempotence, Kafka supports **Exactly Once Semantics (EOS)** for supported workflows.

---

# 14. Consumer Architecture

Consumer reads records.

```
Broker

↓

Fetch Request

↓

Consumer

↓

Business Logic
```

Unlike many messaging systems,

Kafka consumers **pull** data.

---

# 15. Consumer Poll Model

Applications continuously poll Kafka.

```
while(true){

    consumer.poll();

}
```

Flow

```
Consumer

↓

Poll

↓

Broker

↓

Records

↓

Application
```

Advantages

- Backpressure support
- Consumer controls speed
- Better scalability

---

# Why Pull Instead of Push?

Consumer decides

```
When

How Much

How Fast
```

Prevents slow consumers from being overwhelmed.

---

# 16. Consumer Groups

Suppose topic has

```
4 Partitions
```

Consumer Group

```
Consumer-1

Consumer-2
```

Assignment

```
P0 → C1

P1 → C2

P2 → C1

P3 → C2
```

Important rule

One partition

↓

One consumer

within the same consumer group.

Different consumer groups can independently read the same topic.

---

# 17. Consumer Coordinator

Every consumer group has a coordinator.

Responsibilities

- Join group
- Assign partitions
- Monitor heartbeats
- Trigger rebalancing

Flow

```
Consumer Starts

↓

Coordinator

↓

Partition Assignment
```

---

# 18. Offset Management

Kafka stores

```
Messages

Offsets
```

Separately.

Consumer tracks

```
Last Processed Offset
```

Example

```
Offset

10

11

12

13

Consumer committed

13
```

After restart,

consumer resumes from

```
14
```

---

# 19. Offset Commit Strategies

## Auto Commit

Kafka periodically commits offsets.

Simple

↓

Risk of message loss if processing fails after commit.

---

## Manual Commit

Application decides when to commit.

```
Process Record

↓

Success

↓

Commit Offset
```

Safer.

Preferred for critical systems.

---

### commitSync()

Blocks until broker confirms.

Reliable.

Slightly slower.

---

### commitAsync()

Does not wait.

Higher throughput.

Possible commit ordering issues.

---

# 20. Consumer Rebalancing

Suppose

```
2 Consumers
```

One crashes.

Coordinator detects failure.

```
Consumer Down

↓

Rebalance

↓

Partitions Reassigned
```

Rebalancing also happens when

- Consumer joins
- Consumer leaves
- Partitions increase

Modern Kafka supports **Cooperative Rebalancing**, which reduces disruption by moving only the partitions that need reassignment.

---

# 21. Consumer Lag

Definition

```
Latest Offset

-

Committed Offset
```

Example

```
Latest

1000

Committed

900

Lag

100
```

High lag means consumers cannot keep up.

Common causes

- Slow processing
- Too few consumers
- Slow downstream systems
- Large messages

---

# 22. Delivery Semantics

---

## At Most Once

```
Commit

↓

Process
```

No duplicates.

Possible message loss.

---

## At Least Once

```
Process

↓

Commit
```

No message loss.

Duplicates possible.

Most commonly used.

---

## Exactly Once

Requires

- Idempotent Producer
- Transactions
- Compatible consumer processing

Highest consistency.

---

# 23. Retry & Dead Letter Queue

Suppose

```
Consumer

↓

Record

↓

Processing Failed
```

Retry

```
Attempt-1

↓

Attempt-2

↓

Attempt-3
```

Still fails

↓

Dead Letter Topic

```
orders-dlt
```

Benefits

- No infinite retries
- Bad records isolated
- Easier debugging

---

# 24. Production Best Practices

### Producer

✔ Use ACK = all for important data.

✔ Enable idempotence.

✔ Use compression.

✔ Tune batch size.

✔ Use keys when ordering matters.

---

### Consumer

✔ Prefer manual commits for critical processing.

✔ Monitor consumer lag.

✔ Handle retries carefully.

✔ Use Dead Letter Topics for poison messages.

✔ Keep processing idempotent.

✔ Scale by increasing partitions and consumers appropriately.

---

# 25. Interview Summary

## Explain Producer Flow

"When an application calls `producer.send()`, Kafka creates a `ProducerRecord`, serializes it into bytes, determines the target partition, stores it temporarily in the `RecordAccumulator`, batches records together, and a background Sender Thread transmits the batch to the leader broker. The broker acknowledges the write according to the configured `acks` level."

---

## Why Doesn't Producer Send Immediately?

"Sending every message individually would create excessive network overhead. Kafka batches records inside the `RecordAccumulator`, allowing the Sender Thread to send many messages in a single request, dramatically improving throughput."

---

## Why Does Kafka Use Pull Instead of Push?

"With the pull model, consumers control when and how much data they fetch. This naturally supports backpressure, prevents slow consumers from being overwhelmed, and allows consumers to process records at their own pace."

---

## Explain Consumer Groups

"A consumer group is a set of consumers cooperating to process a topic. Each partition is assigned to only one consumer within the group, enabling parallel processing while preserving ordering within each partition. Multiple consumer groups can independently consume the same topic."

---

## Auto Commit vs Manual Commit

| Auto Commit | Manual Commit |
|-------------|---------------|
| Simple | More control |
| Risk of message loss | Safer processing |
| Less application code | Preferred for critical systems |

---

## What Causes Consumer Lag?

- Slow business logic
- External service latency
- Database bottlenecks
- Too few consumer instances
- Insufficient partitions
- Large or complex messages

---

# Key Takeaways

- Producers are **asynchronous** and rely on the **RecordAccumulator** and **Sender Thread** for high throughput.
- **Serialization**, **partitioning**, **batching**, and **compression** are key parts of the producer pipeline.
- **ACKs**, **retries**, and **idempotence** determine durability and duplicate behavior.
- Consumers use a **pull model**, allowing them to control throughput and apply backpressure.
- **Consumer Groups** provide scalability while maintaining ordering within partitions.
- **Offsets** track consumer progress independently of message storage.
- **Manual offset commits**, **Dead Letter Topics**, and **idempotent processing** are essential patterns for production-grade systems.
- Understanding the internal producer and consumer architecture is critical before studying Kafka's end-to-end request lifecycle.
