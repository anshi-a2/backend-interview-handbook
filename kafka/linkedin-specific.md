## 🔹 LinkedIn

**Q1. Explain ISR, AR, OSR and how leader election works.**\
- ISR = fully caught-up replicas.\
- AR = all assigned replicas.\
- OSR = lagging replicas.\
- Leader election only from ISR.

**Q2. What happens if a leader fails?**\
- Controller triggers leader election.\
- ISR replica becomes new leader.\
- If no ISR → risk of data loss.

**Q3. How does Kafka handle message retention?**\
- Time-based (`retention.ms`) or size-based (`retention.bytes`).\
- Log compaction keeps latest value per key.

**Q4. What happens if producer sends faster than broker can handle?**\
- Broker throttles or producer blocks.\
- Buffer may fill → TimeoutException.\
- Fix: add partitions/brokers, tune configs.

  

------------------------------------------------------------------------


# ✅ Key Takeaways

-   **LinkedIn** → Deep internals (ISR, retention, elections).\
