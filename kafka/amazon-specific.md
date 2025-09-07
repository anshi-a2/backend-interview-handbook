## 🔹 Amazon

**Q1. How does Kafka guarantee durability of events?**\
- Kafka writes data to disk (commit log).\
- Uses **replication factor** with leader--follower model.\
- Controlled via `acks` + `min.insync.replicas`.\
- Messages remain even after consumption (until retention policy).

**Q2. How would you design Amazon Order Tracking using Kafka?**\
- **Producers**: Order service → push events to Kafka topic (`orders`).\
- **Partition Key**: `order_id` to preserve ordering.\
- **Consumers**: Payment, Shipping, Notifications.\
- **Durability**: RF=3, acks=all.\
- **Scaling**: More partitions = more consumer parallelism.

**Q3. What happens if a consumer in a consumer group crashes?**\
- Kafka triggers a **rebalance**.\
- Partitions are reassigned to remaining consumers.\
- Group coordinator manages consumer state.

**Q4. How do you handle duplicate messages in Kafka consumers?**\
- Kafka guarantees *at least once* delivery.\
- Solutions: idempotent operations, deduplication in DB, or enable EOS.


------------------------------------------------------------------------

# ✅ Key Takeaway

-   **Amazon** → Durability, scaling, fault-tolerance.\
