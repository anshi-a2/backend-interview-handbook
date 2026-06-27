# 04_Kafka_With_Spring_Boot.md

> **Goal**
>
> Learn **how Kafka is integrated with Spring Boot**, what happens internally when a Spring Boot application publishes or consumes messages, and how to build **production-grade event-driven microservices**.
>
> This chapter covers Spring Kafka architecture, producer and consumer configuration, serialization, retries, Dead Letter Topics, transactions, exactly-once semantics, batch processing, error handling, and complete real-world examples.

---

# Table of Contents

1. Why Spring Kafka?
2. Spring Kafka Architecture
3. Maven Dependencies
4. Producer Configuration
5. KafkaTemplate Internals
6. Consumer Configuration
7. @KafkaListener Internals
8. Serialization & Deserialization
9. Message Keys & Partitioning
10. Error Handling
11. Retry Mechanisms
12. Dead Letter Topics (DLT)
13. Manual Offset Commit
14. Batch Processing
15. Transactions & Exactly Once
16. Real Production Flow
17. Common Production Problems
18. Best Practices
19. Interview Summary

---

# 1. Why Spring Kafka?

Without Spring Kafka

```
Application

↓

Kafka Producer API

↓

Kafka Consumer API

↓

Manual Configuration
```

Developer writes a lot of boilerplate code.

---

With Spring Kafka

```
Application

↓

KafkaTemplate

↓

Spring Kafka

↓

Kafka Client

↓

Kafka Broker
```

Spring Boot manages most configuration automatically.

---

# 2. Spring Kafka Architecture

```
                    Spring Boot

      ┌───────────────────────────────┐
      │                               │
      │       KafkaTemplate           │
      │            │                  │
REST API ─────────▶│                  │
      │            ▼                  │
      │      Kafka Producer           │
      │                               │
      │      Kafka Consumer           │
      │            ▲                  │
      │     @KafkaListener            │
      └───────────────────────────────┘
                     │
                     ▼
               Kafka Cluster
```

---

# 3. Maven Dependencies

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

Spring Boot automatically configures

- ProducerFactory
- ConsumerFactory
- KafkaTemplate
- ListenerContainerFactory

using application properties.

---

# 4. Producer Configuration

Example

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092

    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer

      acks: all
      retries: 3
      properties:
        enable.idempotence: true
```

### Important Settings

| Property | Purpose |
|----------|----------|
| bootstrap-servers | Kafka cluster address |
| acks | Write durability |
| retries | Retry failed sends |
| enable.idempotence | Prevent duplicate writes |
| batch.size | Batch size |
| linger.ms | Wait time before sending |
| compression.type | Compress batches |

---

# 5. KafkaTemplate Internals

Application

```java
kafkaTemplate.send("orders", order);
```

Internal Flow

```
Application

↓

KafkaTemplate

↓

ProducerFactory

↓

Kafka Producer

↓

Serializer

↓

Partitioner

↓

RecordAccumulator

↓

Sender Thread

↓

Broker
```

`KafkaTemplate` is simply a Spring wrapper around KafkaProducer.

---

## Async Nature

```java
CompletableFuture<SendResult<String, Order>>
```

Spring does **not block**.

```
Send()

↓

Future Returned

↓

Broker Response

↓

Callback
```

Example

```java
kafkaTemplate.send(topic, order)
    .whenComplete((result, ex) -> {
        if (ex != null) {
            // failure
        }
    });
```

---

# 6. Consumer Configuration

Example

```yaml
spring:
  kafka:
    consumer:
      group-id: order-group

      auto-offset-reset: earliest

      enable-auto-commit: false

      key-deserializer:
        org.apache.kafka.common.serialization.StringDeserializer

      value-deserializer:
        org.springframework.kafka.support.serializer.JsonDeserializer
```

---

Important Properties

| Property | Meaning |
|-----------|----------|
| group-id | Consumer Group |
| auto-offset-reset | earliest/latest |
| enable-auto-commit | Offset strategy |
| max-poll-records | Records per poll |
| fetch-min-size | Fetch tuning |

---

# 7. @KafkaListener Internals

Example

```java
@KafkaListener(topics="orders")
public void consume(Order order){

}
```

What happens internally?

```
Application Starts

↓

Spring Scans

↓

@KafkaListener

↓

ListenerContainer

↓

Kafka Consumer

↓

Poll Loop

↓

Broker

↓

Business Method
```

Spring creates a **listener container** that continuously calls `poll()` in the background.

You never call `poll()` yourself.

---

# 8. Serialization & Deserialization

Producer

```
Order Object

↓

JsonSerializer

↓

Byte[]
```

Broker stores bytes.

Consumer

```
Byte[]

↓

JsonDeserializer

↓

Order Object
```

Common formats

- JSON
- Avro
- Protobuf
- String
- Byte Array

For large distributed systems, Avro or Protobuf are preferred because they support schema evolution and are more compact than JSON.

---

# 9. Message Keys & Partitioning

Producer

```java
kafkaTemplate.send(
        "orders",
        orderId,
        order);
```

Flow

```
Key

↓

Hash

↓

Partition
```

Same key

↓

Same partition

↓

Ordering guaranteed

Example

```
Order-101

↓

Partition-2
```

All updates for Order-101 arrive in order.

---

# 10. Error Handling

Without handling

```
Consumer

↓

Exception

↓

Consumer Stops
```

Spring Kafka allows custom error handling.

```
Consumer

↓

Exception

↓

Retry

↓

DLT

↓

Skip
```

Example

```java
DefaultErrorHandler
```

Supports

- Retry
- Backoff
- Recovery

---

# 11. Retry Mechanisms

Suppose database is temporarily unavailable.

```
Consumer

↓

Database Error

↓

Retry

↓

Retry

↓

Success
```

Spring provides

```
Fixed Backoff

Exponential Backoff
```

Avoid

```
Infinite Retry
```

because one bad message can block the partition.

---

# 12. Dead Letter Topics (DLT)

Suppose

```
Message

↓

Retry

↓

Retry

↓

Retry

↓

Still Fails
```

Instead of blocking

↓

Move

```
orders-dlt
```

Architecture

```
Orders Topic

↓

Consumer

↓

Failure

↓

Retry

↓

DLT
```

Benefits

- No poison message blocking
- Easier debugging
- Replay later

---

# 13. Manual Offset Commit

Critical interview topic.

Auto Commit

```
Read

↓

Commit

↓

Process
```

Risk

Application crashes

↓

Message lost.

---

Manual Commit

```
Read

↓

Process

↓

Commit
```

Safer.

Spring supports

```java
AckMode.MANUAL
```

Example

```java
ack.acknowledge();
```

Only commit after successful processing.

---

# 14. Batch Processing

Instead of

```
1 Message

↓

Process
```

Spring can fetch

```
100 Messages

↓

Process Together
```

Benefits

- Higher throughput
- Lower network cost
- Fewer commits

Trade-off

- Larger memory usage
- More complex error handling

---

# 15. Transactions & Exactly Once

Producer

```
Topic-A

↓

Topic-B

↓

Commit
```

or

```
Rollback
```

Spring supports Kafka transactions.

Configuration

```yaml
transaction-id-prefix:
```

Exactly Once requires

- Idempotent producer
- Transactions
- Correct consumer design
- Idempotent downstream operations when external systems are involved

---

# 16. Real Production Flow

Imagine an Order Service.

```
REST Request

↓

Controller

↓

Order Service

↓

Save Database

↓

KafkaTemplate

↓

Orders Topic

↓

Inventory Service

↓

Payment Service

↓

Notification Service

↓

Analytics
```

Each service consumes independently.

---

## Payment Flow

```
Order Created

↓

Payment Service

↓

Payment Successful

↓

payment-events

↓

Inventory

↓

Notification
```

---

# 17. Common Production Problems

---

## Duplicate Messages

Reasons

- Producer retries
- Consumer retry
- Offset commit failure

Solution

- Idempotent producer
- Idempotent consumers
- Unique business keys

---

## Consumer Lag

Reasons

- Slow database
- External API latency
- Too few consumers
- Heavy processing

Monitor continuously.

---

## Rebalancing Storm

Too many consumers joining/leaving

↓

Frequent partition movement

↓

Performance degradation

---

## Serialization Errors

Producer sends

```
JSON
```

Consumer expects

```
Avro
```

↓

Deserialization fails.

Always keep producer and consumer schemas compatible.

---

# 18. Best Practices

### Producer

✔ Enable idempotence.

✔ Use `acks=all`.

✔ Use compression.

✔ Send meaningful message keys.

---

### Consumer

✔ Prefer manual offset commits.

✔ Handle retries with backoff.

✔ Use Dead Letter Topics.

✔ Keep business logic idempotent.

✔ Monitor consumer lag.

---

### Spring Boot

✔ Use `KafkaTemplate`.

✔ Use `@KafkaListener`.

✔ Externalize Kafka configuration.

✔ Separate producer and consumer configurations when appropriate.

✔ Use structured logging and correlation IDs for tracing.

---

# 19. Interview Summary

## How Does KafkaTemplate Work?

"`KafkaTemplate` is a Spring abstraction over the KafkaProducer. It creates a ProducerRecord, serializes the payload, determines the partition, stores the record in the RecordAccumulator, and a background Sender Thread asynchronously sends the batch to the Kafka broker."

---

## How Does @KafkaListener Work?

"When Spring Boot starts, it scans for `@KafkaListener` annotations and creates listener containers. These containers manage KafkaConsumer instances internally and continuously invoke `poll()` in the background. Whenever records are received, Spring deserializes them and invokes the annotated method."

---

## Why Use Manual Offset Commit?

"Manual commits ensure offsets are acknowledged only after successful business processing. This reduces the risk of message loss and is preferred for critical applications."

---

## What is a Dead Letter Topic?

"A Dead Letter Topic stores messages that cannot be processed successfully after configured retry attempts. This prevents a single poison message from blocking the consumer while preserving the failed event for investigation or replay."

---

## How Do You Guarantee Ordering?

"Use a meaningful message key so all related events are routed to the same partition. Kafka guarantees ordering only within an individual partition."

---

# Key Takeaways

- **Spring Kafka** simplifies Kafka integration by managing producers, consumers, and listener containers.
- **KafkaTemplate** is asynchronous and delegates work to the native Kafka producer.
- **@KafkaListener** hides the polling loop by running Kafka consumers inside managed listener containers.
- **JSON, Avro, and Protobuf** are common serialization formats; choose based on interoperability and performance requirements.
- **Manual offset commits** provide greater reliability than automatic commits for critical workloads.
- **Retries**, **backoff policies**, and **Dead Letter Topics** are essential for resilient consumers.
- **Transactions**, **idempotent producers**, and **idempotent processing** together help build reliable event-driven systems.
- A production-ready Spring Boot Kafka application requires proper monitoring, schema compatibility, retry handling, and consumer lag management.
