

## 🔹 Goldman Sachs / Banking

**Q1. How do you ensure no data loss in Kafka (finance)?**\
- acks=all, min.insync.replicas ≥ 2.\
- Disable unclean leader election.\
- Use log compaction for critical topics.

**Q2. How do you design audit logging with Kafka?**\
- Producers push events into `audit-logs`.\
- RF=3, acks=all.\
- Consumers: compliance, fraud detection.\
- Compaction enabled.

**Q3. Role of unclean leader election?**\
- Promotes OSR as leader if no ISR.\
- Causes possible data loss.\
- In finance: must be disabled.

**Q4. How do you design reconciliation system?**\
- Producers → `transactions` topic.\
- Consumers → DB + compacted topic (`latest-balance`).\
- Reconciliation service cross-checks.\
- DLQ for mismatches.

  ------------------------------------------------------------------------

# ✅ Key Takeaways

-   **Goldman Sachs** → Compliance, reconciliation, strict durability.
