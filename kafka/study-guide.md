# 📘 Apache Kafka -- Complete Study Guide (Java Focus, Basics to Advanced)

------------------------------------------------------------------------

## 1. Introduction to Kafka

### 1.1 What is Kafka?

Apache Kafka is a **distributed event streaming platform** capable of
handling **high-throughput, fault-tolerant, scalable** data pipelines.\
It was originally developed by **LinkedIn** and later open-sourced. Now
maintained by the **Apache Software Foundation**.

### 1.2 Why Kafka?

-   Traditional message brokers (ActiveMQ, RabbitMQ) cannot scale
    easily.
-   Databases are not optimized for real-time streaming.
-   Kafka provides:
    -   **Durability**: Messages written to disk (commit log).
    -   **Scalability**: Horizontally scalable via partitions.
    -   **Fault tolerance**: Replication across brokers.
    -   **High throughput**: Millions of messages per second.

### 1.3 Real-World Analogy

Think of Kafka as a **newspaper publisher**: - **Topics** = Different
newspapers (Economy, Sports, Tech). - **Partitions** = Multiple copies
of the newspaper printed for distribution. - **Producers** = Journalists
publishing articles. - **Consumers** = Readers who subscribe to certain
topics.

------------------------------------------------------------------------

## 2. Kafka Architecture

### 2.1 Core Components

-   **Producer** → Publishes records to Kafka topics.
-   **Consumer** → Reads records from topics.
-   **Broker** → Kafka server storing data (cluster usually has multiple
    brokers).
-   **Topic** → Logical channel for messages.
-   **Partition** → Splits topic for parallelism.
-   **Offset** → Unique ID for each message in a partition.
-   **Replication** → Copies partitions across brokers for fault
    tolerance.

### 2.2 Zookeeper vs KRaft

-   Before Kafka 2.8 → Zookeeper managed cluster metadata.
-   From Kafka 2.8+ → KRaft mode removes Zookeeper, metadata handled by
    internal controller.

### 2.3 Data Flow

1.  Producer sends message to broker.
2.  Broker writes message to topic partition (append-only log).
3.  Message replicated to follower brokers.
4.  Consumer pulls messages and tracks offset.

------------------------------------------------------------------------

## 3. Kafka Setup

### 3.1 Installation

1.  Download Kafka: [Apache Kafka
    Downloads](https://kafka.apache.org/downloads)

2.  Extract and start services:

    ``` bash
    bin/zookeeper-server-start.sh config/zookeeper.properties
    bin/kafka-server-start.sh config/server.properties
    ```

### 3.2 Create Topic

``` bash
bin/kafka-topics.sh --create --topic orders --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

### 3.3 List Topics

``` bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

------------------------------------------------------------------------

## 4. Kafka Producers in Java

### 4.1 Basic Producer Example

``` java
import org.apache.kafka.clients.producer.*;
import java.util.Properties;

public class SimpleProducer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        ProducerRecord<String, String> record =
            new ProducerRecord<>("orders", "orderId-1", "Order placed for product 101");

        producer.send(record, (metadata, exception) -> {
            if (exception == null) {
                System.out.printf("Sent to partition %d with offset %d%n",
                        metadata.partition(), metadata.offset());
            } else {
                exception.printStackTrace();
            }
        });

        producer.close();
    }
}
```

### 4.2 Advanced Producer Features

-   **Idempotent Producer**

    ``` java
    props.put("enable.idempotence", "true");
    ```

-   **Custom Partitioner**

    ``` java
    public class CustomPartitioner implements Partitioner {
        @Override
        public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
            int partitions = cluster.partitionCountForTopic(topic);
            return (key.hashCode() & Integer.MAX_VALUE) % partitions;
        }
        @Override public void close() {}
        @Override public void configure(Map<String, ?> configs) {}
    }
    ```

------------------------------------------------------------------------

## 5. Kafka Consumers in Java

### 5.1 Basic Consumer Example

``` java
import org.apache.kafka.clients.consumer.*;
import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

public class SimpleConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "order-consumers");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("auto.offset.reset", "earliest");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("orders"));

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("Consumed: key=%s value=%s partition=%d offset=%d%n",
                        record.key(), record.value(), record.partition(), record.offset());
            }
        }
    }
}
```

### 5.2 Manual Offset Commit

``` java
props.put("enable.auto.commit", "false");

// Commit manually after processing
consumer.commitSync();
```

### 5.3 Rebalance Listener

Used to handle partition rebalancing events.

------------------------------------------------------------------------

## 6. Advanced Kafka Concepts

### 6.1 Delivery Guarantees

-   **At-most-once** → No retries, message may be lost.
-   **At-least-once** → Retries enabled, duplicates possible.
-   **Exactly-once** → Transactions, idempotent producers.

### 6.2 Transactions

``` java
props.put("transactional.id", "txn-1");
KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.initTransactions();

producer.beginTransaction();
producer.send(new ProducerRecord<>("orders", "orderId-1", "Order Payment Done"));
producer.commitTransaction();
```

### 6.3 Performance Tuning

-   `batch.size` → Controls batch size.
-   `linger.ms` → Wait time before sending batch.
-   `compression.type` → gzip/snappy/lz4/zstd.

### 6.4 Security

-   **SSL Encryption**
-   **SASL Authentication**
-   **ACLs for authorization**

------------------------------------------------------------------------

## 7. Kafka Streams API

### 7.1 Word Count Example

``` java
import org.apache.kafka.streams.*;
import org.apache.kafka.streams.kstream.*;
import java.util.Properties;
import java.util.Arrays;

public class WordCountApp {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> textLines = builder.stream("input-text");
        textLines.flatMapValues(value -> Arrays.asList(value.toLowerCase().split(" ")))
                 .groupBy((key, word) -> word)
                 .count()
                 .toStream()
                 .to("word-count-output");

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();
    }
}
```

### 7.2 Features

-   **Joins** between streams.
-   **Windowing** operations.
-   **Stateful processing** with RocksDB.

------------------------------------------------------------------------

## 8. Kafka Connect

-   **Source Connectors** → Ingest data into Kafka (MySQL, Postgres,
    etc.).
-   **Sink Connectors** → Push data out (Elasticsearch, HDFS, S3).

### Example: MySQL → Kafka → ElasticSearch

1.  Use MySQL Source Connector to capture DB changes (CDC).
2.  Kafka stores events in topic.
3.  ElasticSearch Sink Connector indexes data for search.

------------------------------------------------------------------------

## 9. Monitoring Kafka

-   **Lag monitoring** → Ensures consumers keep up with producers.
-   **Prometheus + Grafana** → Visualize metrics.
-   **Kafka Manager** or **Confluent Control Center** for UI monitoring.

### Key Metrics

-   Consumer lag.
-   Broker disk usage.
-   Under-replicated partitions.

------------------------------------------------------------------------

## 10. Real-World Projects

### 10.1 Log Aggregation Pipeline

Apps → Kafka → Logstash → Elasticsearch → Kibana.

### 10.2 E-commerce Order Tracking

Microservices produce events (OrderPlaced, PaymentDone, Shipped).\
Kafka ensures reliable event delivery.

### 10.3 IoT Data Streaming

Sensors → Kafka → Spark/Flink → Dashboard.

### 10.4 Fraud Detection in Banking

Transactions streamed → Kafka → ML model → Alerts.

------------------------------------------------------------------------

## 11. Interview Q&A

1.  **Why is Kafka fast?**
    -   Sequential disk writes (commit log).
    -   Zero-copy technology.
    -   Batching.
2.  **Kafka vs RabbitMQ**
    -   RabbitMQ → Traditional message broker.
    -   Kafka → Distributed event streaming platform.
3.  **What happens if a broker fails?**
    -   Leader election via ISR (in-sync replicas).
    -   Consumers retry fetching.
4.  **At-least-once vs Exactly-once**
    -   At-least-once may duplicate messages.
    -   Exactly-once uses idempotence + transactions.
5.  **How does Kafka ensure durability?**
    -   Messages persisted to disk.
    -   Replicated across brokers.

------------------------------------------------------------------------

# ✅ Summary

This guide covered: - Kafka basics & architecture. - Producer & consumer
Java examples. - Advanced topics (EOS, transactions, tuning). - Kafka
Streams & Connect. - Monitoring best practices. - Real-world projects. -
Interview preparation.
