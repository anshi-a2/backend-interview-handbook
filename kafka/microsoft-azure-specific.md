## 🔹 Microsoft / Azure

**Q1. How do you deploy Kafka in multi-region setup?**\
- Use MirrorMaker 2 for replication.\
- Async replication for throughput, sync for critical data.\
- Partition by region.

**Q2. How do you secure Kafka in cloud?**\
- SASL_SSL or Kerberos for auth.\
- TLS for encryption.\
- ACLs for authorization.

**Q3. Challenges of running Kafka on Kubernetes?**\
- Stateful workloads need PVs.\
- Zookeeper/KRaft management.\
- Networking overhead.\
- Solution: Strimzi/Confluent operators.

**Q4. How do you optimize Kafka in cloud for cost?**\
- Tiered storage (S3/Blob).\
- Compression.\
- Auto-scale consumers.\
- Use serverless Kafka.

------------------------------------------------------------------------


# ✅ Key Takeaways

-   **Microsoft** → Multi-region, Kubernetes, cloud security.\
