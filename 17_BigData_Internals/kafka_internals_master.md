
17_BigData_Internals/kafka_internals_master.md
```

---

# 📌 KAFKA INTERNALS OVERVIEW

Kafka = **distributed commit log system**

Core components:

```text
Producer → Topic → Partition → Broker → Consumer
```

Key ideas:

```text
Partitioning
Replication
Offsets
Leader–Follower model
```

---

# 1️⃣ TOPIC & PARTITION (FOUNDATION)

* **Topic** = logical stream
* **Partition** = parallel unit (ordered log)

```text
Topic: orders
  ├─ Partition 0
  ├─ Partition 1
  └─ Partition 2
```

📌 Ordering **only within a partition** guaranteed

---

# 2️⃣ BROKER & CLUSTER

* **Broker** = Kafka server
* Multiple brokers = cluster

```text
Broker 0
Broker 1
Broker 2
```

📌 Topics partitions brokers me distribute hote hain

---

# 3️⃣ REPLICATION (FAULT TOLERANCE)

Each partition has:

```text
Leader
Followers (replicas)
```

Example:

```text
Partition 0:
Leader → Broker 1
Follower → Broker 2, Broker 3
```

---

# 4️⃣ ISR (IN-SYNC REPLICAS) ⭐

ISR = replicas jo leader ke saath **fully synced** hain

```text
ISR = {Leader + up-to-date followers}
```

If follower lag karta hai:

```text
ISR se remove ho jata hai
```

📌 Important:

```text
Only ISR replicas can become leader
```

---

# 5️⃣ LEADER ELECTION (VERY IMPORTANT)

Leader fail hone par:

```text
New leader selected from ISR
```

Steps:

```text
1. Leader crash
2. Controller detect failure
3. ISR me se new leader choose
```

⚠️ If ISR empty:

```text
Data loss possible (unclean leader election)
```

---

# 6️⃣ PRODUCER WRITE FLOW

```text
Producer → Leader partition → Followers replicate
```

Steps:

```text
1. Producer send message
2. Leader write to log
3. Followers replicate
4. Ack sent based on config
```

---

# 7️⃣ ACKS CONFIGURATION (CONSISTENCY CONTROL)

| acks value | Meaning          |
| ---------- | ---------------- |
| 0          | no guarantee     |
| 1          | leader only      |
| all        | all ISR replicas |

⭐ Most important:

```text
acks=all → strongest durability
```

---

# 8️⃣ OFFSET MANAGEMENT

Offset = message position inside partition

```text
Partition 0:
offset 0 → msg1
offset 1 → msg2
offset 2 → msg3
```

Consumer tracks:

```text
last read offset
```

---

# 9️⃣ CONSUMER GROUP

* Multiple consumers = **consumer group**
* Each partition → one consumer

```text
Group1:
Consumer1 → Partition 0
Consumer2 → Partition 1
```

📌 Parallel processing possible

---

# 🔟 OFFSET COMMIT TYPES

## Auto commit

```text
Kafka automatically commit offset
```

## Manual commit

```text
Consumer controls offset commit
```

📌 Manual = safer (no data loss)

---

# 1️⃣1️⃣ AT-LEAST / AT-MOST / EXACTLY-ONCE

| Type          | Meaning               |
| ------------- | --------------------- |
| At-most-once  | may lose data         |
| At-least-once | may duplicate         |
| Exactly-once  | no loss, no duplicate |

---

# 1️⃣2️⃣ IDEMPOTENT PRODUCER

Enable:

```text
enable.idempotence=true
```

Meaning:

```text
Duplicate messages avoid hoti hain
```

---

# 1️⃣3️⃣ EXACTLY-ONCE SEMANTICS (EOS)

Achieved using:

```text
Idempotent producer
Transactions
```

Example flow:

```text
Producer → send messages → commit transaction
Consumer → read committed data only
```

---

# 1️⃣4️⃣ TRANSACTIONAL PRODUCER

Steps:

```text
1. Begin transaction
2. Send messages
3. Commit / Abort
```

Guarantee:

```text
All messages processed OR none
```

---

# 1️⃣5️⃣ LOG STRUCTURE (INTERNAL STORAGE)

Kafka stores data as:

```text
Segment files
Offset index
Time index
```

Example:

```text
000000.log
000000.index
```

---

# 1️⃣6️⃣ RETENTION POLICY

Controls data deletion:

```text
Time-based
Size-based
```

Example:

```text
retention.ms = 7 days
```

---

# 1️⃣7️⃣ PARTITION REBALANCING

When:

```text
New consumer joins
Consumer leaves
```

Kafka reassigns partitions

⚠️ Issue:

```text
Rebalance causes temporary delay
```

---

# 1️⃣8️⃣ ZOOKEEPER vs KRAFT (NEW MODE)

## Old:

```text
Kafka + Zookeeper
```

## New:

```text
Kafka (KRaft mode) without Zookeeper
```

Benefits:

```text
Simpler architecture
Better performance
```

---

# 1️⃣9️⃣ HIGH THROUGHPUT REASON

Kafka fast kyun hai:

```text
Sequential disk writes
Zero-copy transfer
Batching
Partition parallelism
```

---

# 2️⃣0️⃣ REAL INTERVIEW SCENARIO

Question:

```text
How to ensure no data loss in Kafka?
```

Answer:

```text
acks=all
replication factor ≥ 3
min.insync.replicas ≥ 2
enable.idempotence=true
```

---

# 🎯 MOST IMPORTANT KAFKA INTERVIEW TRAPS

```text
ISR = in-sync replicas
Leader handles all reads/writes
Ordering only within partition
Offset = message position
acks=all ensures strong durability
Idempotent producer avoids duplicates
Exactly-once uses transactions
Consumer group enables parallelism
Kafka default = eventual consistency
```




✅ **HBase Internals Deep Dive (MemStore, WAL, HFile, Compaction, Region Splitting)** — jo CDAC + interviews me frequently poocha jaata hai 💯
