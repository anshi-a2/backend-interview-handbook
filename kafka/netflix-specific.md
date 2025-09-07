
## 🔹 Netflix

**Q1. How do you ensure real-time processing with millions of messages
per second?**\
- Partition topics by `user_id` or region.\
- Use **Kafka Streams** or Flink.\
- Enable **compression** (Snappy/LZ4).\
- Monitor **consumer lag** & auto-scale.

**Q2. How do you handle backpressure in a video streaming pipeline?**\
- Use **pause() / resume()** consumer APIs.\
- Implement retry queues & DLQs.\
- Batch writes to downstream systems.

**Q3. How do you ensure low latency in Kafka pipeline?**\
- Tune `linger.ms`, `batch.size`, compression.\
- Optimize consumer fetch configs.\
- Use async writes.

**Q4. How do you detect and recover from consumer lag?**\
- Monitor lag via Burrow or Kafka Lag Exporter.\
- Auto-scale consumers.\
- Alert on thresholds.\
- Throttle producers if needed.


------------------------------------------------------------------------

# ✅ Key Takeaway

-   **Netflix** → Low latency, backpressure handling.\
