# Kafka SDE-2 Interview Questions with Solutions

At this level, interviewers usually test not just the basics (what is
Kafka, why use it) but also **deep internals, real-world use cases,
troubleshooting, and design questions**.

------------------------------------------------------------------------

## 🔹 1. Kafka Basics & Core Concepts

**Q1. What is Kafka and why is it used?**\
- Kafka is a **distributed event streaming platform** used for
high-throughput, low-latency, real-time data pipelines and event-driven
applications.\
- Advantages:\
- Durability (messages written to disk & replicated)\
- Scalability (partitioning + brokers)\
- Fault-tolerance (replication, ISR)\
- Decoupling producers & consumers

**Q2. Explain Kafka Topic, Partition, and Offset.**\
- **Topic**: Named log where messages are published.\
- **Partition**: Split of topic for scalability. Each partition is an
ordered, immutable sequence of records.\
- **Offset**: Unique ID within a partition (acts like cursor). Consumers
use offsets to track read progress.

**Q3. Difference between Kafka Consumer Groups and Partitions?**\
- A **consumer group** is a set of consumers that share work.\
- Each partition in a topic is consumed by only **one consumer in a
group** (but can be read by many groups independently).\
- Ensures **parallelism** + **fault tolerance**.

------------------------------------------------------------------------

## 🔹 2. Kafka Internals & Advanced

**Q4. How does Kafka ensure reliability of message delivery?**\
- **Replication factor (RF)**: Each partition has leader + followers.\
- **acks settings**:\
- `acks=0` → Fire and forget\
- `acks=1` → Leader ack only\
- `acks=all` → All replicas in ISR must ack (strongest)\
- **min.insync.replicas**: Ensures enough replicas are in sync before
accepting writes.

**Q5. Explain ISR, AR, and OSR in Kafka.**\
- **ISR (In-Sync Replicas)**: Set of replicas fully caught up with the
leader.\
- **AR (Assigned Replicas)**: All replicas assigned to a partition.\
- **OSR (Out of Sync Replicas)**: Replicas that are lagging behind
leader.\
- Only ISR members are eligible to become leader.

**Q6. What is Kafka Exactly-Once Semantics (EOS)?**\
- Kafka guarantees **at-least-once** delivery by default.\
- With idempotent producers (`enable.idempotence=true`) + transactional
writes, you can achieve **exactly-once** processing.\
- Uses **Producer ID (PID)** and **Sequence Numbers** to deduplicate
messages.

------------------------------------------------------------------------

## 🔹 3. Kafka in System Design

**Q7. How would you design a system using Kafka for real-time analytics
(e.g., user activity tracking)?**\
- Producers → Kafka (partitioned by `user_id`) → Consumers → Processing
Layer (Spark/Flink) → Storage/DB.\
- Key points:\
- Use partition key = user_id for ordering.\
- Consumer group for scaling.\
- Set RF=3, acks=all for durability.

**Q8. How do you handle backpressure in Kafka consumers?**\
- Tune **max.poll.interval.ms** & **max.poll.records**.\
- Use **pause() and resume()** APIs for consumers.\
- Use Kafka Streams / reactive frameworks to handle load gracefully.\
- Employ batching & async processing.

**Q9. How to ensure message ordering in Kafka?**\
- Ordering is guaranteed **only within a single partition**.\
- To preserve ordering → use a consistent key for partitioning.\
- Global ordering (across partitions) is not supported.

------------------------------------------------------------------------

## 🔹 4. Performance & Troubleshooting

**Q10. Kafka producer is slow, what do you check?**\
- Check producer configs: `batch.size`, `linger.ms`,
`compression.type`.\
- Network issues between producer → broker.\
- Broker overloaded (CPU/disk).\
- GC pauses in JVM.\
- Partitions skewed (bad partitioning logic).

**Q11. Kafka consumer is lagging, how to debug?**\
- Check **consumer lag** using Kafka tools / metrics.\
- Tune `fetch.min.bytes`, `max.poll.records`, `fetch.max.wait.ms`.\
- Increase number of partitions or consumers.\
- Check for slow downstream systems (DB writes).

**Q12. Difference between Kafka vs RabbitMQ? Why choose Kafka?**\
- Kafka is **distributed log storage** optimized for throughput &
scaling.\
- RabbitMQ is **message queue** optimized for complex routing & lower
latency.\
- Kafka is preferred for **streaming, event sourcing, big data
pipelines**.\
- RabbitMQ for **real-time transactional workflows**.

------------------------------------------------------------------------

## 🔹 5. Coding/Problem Solving with Kafka

**Q13. Write a producer-consumer example in Java using Kafka.**

Producer example:

``` java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

producer.send(new ProducerRecord<>("user-events", "user1", "login"));
producer.close();
```

Consumer example:

``` java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "analytics-group");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Collections.singletonList("user-events"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record.key() + " → " + record.value());
    }
}
```

------------------------------------------------------------------------

✅ This set covers **conceptual, system design, internals, debugging,
and coding** questions---exactly what's expected at **SDE-2 level Kafka
interviews**.
