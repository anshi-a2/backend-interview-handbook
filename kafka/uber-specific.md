## 🔹 Uber

**Q1. Kafka vs RabbitMQ at Uber scale -- why Kafka?**\
- Kafka supports **event replay**, high throughput, partitioned logs.\
- RabbitMQ better for complex routing, but Kafka wins for scale.

**Q2. How would you build Uber Trip Event System?**\
- Topic: `trip-events` (RF=3).\
- Partition key = `trip_id`.\
- Consumers: Fare calc, Matching, Fraud detection.\
- ISR ensures failover.

**Q3. How do you ensure ordering of trip events?**\
- Partition by `trip_id`.\
- Guarantees ordering per trip.

**Q4. How would you design a surge pricing system?**\
- Topic: `ride-requests`.\
- Consumer: surge pricing service with Kafka Streams sliding window.\
- Publish updates to `surge-events` topic.

-------------------------------------------------------------------------


# ✅ Key Takeaways


-   **Uber** → Real-time pipelines, surge pricing.\
