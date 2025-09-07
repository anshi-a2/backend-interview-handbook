## 🔹 PayPal / FinTech

**Q1. How does Kafka ensure Exactly-Once semantics (EOS)?**\
- Enable idempotence + transactional producers.\
- Kafka assigns PID + sequence numbers.\
- Offsets committed atomically with messages.

**Q2. How do you prevent double charging a customer?**\
- Use EOS producers.\
- Deduplication at DB layer.\
- Process offsets + DB updates in same transaction.

**Q3. How do you guarantee ordering in financial transactions?**\
- Partition by `account_id`.\
- Use EOS + transactional producers.

**Q4. Log compaction vs retention -- why important in finance?**\
- Retention: deletes old data after X time/size.\
- Compaction: keeps last value per key forever.\
- Finance: compaction ensures latest balances persist.

  ------------------------------------------------------------------------




# ✅ Key Takeaway

-   **PayPal** → Exactly-once semantics, ordering.\
