# 📨 Module 07: Apache Kafka Event Streaming Mastery

Apache Kafka is a distributed event streaming platform designed to handle high-throughput, real-time data feeds. Built as a distributed commit log, Kafka stores streams of records in categories called **Topics**, partitioned across multiple cluster nodes (Brokers), allowing parallel writes and reads with fault tolerance.

---

## 1. Architectural Components & Segment Storage Internals

Kafka is designed for sequential file access and relies heavily on the operating system page cache.

```mermaid
flowchart TD
    subgraph Producer Layer
        P[Producer Client] -- Partition Key --> KeyPart{Partitioner}
    end

    subgraph Kafka Broker Cluster
        subgraph Broker 1 (Leader P0)
            P0_L[Partition 0 Leader]
            P0_L --> Seg_Log[Segment file: 0000.log]
            P0_L --> Seg_Idx[Segment index: 0000.index]
        end
        subgraph Broker 2 (Follower P0 / Leader P1)
            P0_F[Partition 0 Follower]
            P1_L[Partition 1 Leader]
        end
    end

    KeyPart -- Route to Partition 0 --> P0_L
    P0_F -- Fetch / Sync --> P0_L
```

### Components Detailed:
* **Topic & Partition**: A topic is a stream of messages. Topics are split into **Partitions**, which are the unit of scalability in Kafka. Each partition is an ordered, immutable sequence of messages appended to a commit log.
* **Log Segment**: In the broker's data directory, partitions are divided into **Segments** (default: `1GB` files).
  * **`.log` File**: Stores actual raw message byte payloads.
  * **`.index` File**: Maps message offsets to physical file positions. This allows Kafka to perform \(O(1)\) disk seeks to read specific offsets.
* **Leader vs. Follower Replicas**:
  * **Leader**: Handles all read and write requests from clients for a partition.
  * **Follower**: Passive replicas that sync data from the leader. They do not serve client traffic (unless explicitly configured for rack-aware reads in newer versions).
* **In-Sync Replicas (ISR)**: The set of follower replicas that are caught up with the leader's log within the allowed threshold (`replica.lag.time.max.ms`, default 30 seconds).

---

## 2. Producer Internals & Data Durability Guarantees

The producer client compiles and batches records before transmitting them over TCP.

```text
Producer Record Accumulator Pipeline:

Record (Key, Value) ──> Serializer ──> Partitioner ──> Record Accumulator (Memory Batching)
                                                            ├── Partition 0 Batch (16KB) ──> Network Thread ──> Broker
                                                            └── Partition 1 Batch (16KB)
```

### Key Configurations:
* **Record Accumulator**: Accumulates messages in memory batches per partition. When a batch is full (`batch.size`, default 16KB) or the wait limit is reached (`linger.ms`, default 0ms), the network thread sends the batch to the broker. Set `linger.ms=10` to enable micro-batching and increase throughput.
* **Acks Configuration (`acks`)**:
  * `acks=0`: Fire-and-forget. The producer does not wait for broker acknowledgments. Maximum performance, but high risk of data loss.
  * `acks=1`: The leader writes the record to its local log and acknowledges. Data is safe unless the leader crashes before followers sync.
  * `acks=all` (or `-1`): The leader waits for all in-sync replicas (ISR) to acknowledge the write. Provides maximum durability.
* **Idempotent Producer**:
  * Set `enable.idempotence=true`. This assigns a unique **Producer ID (PID)** and an incremental **Sequence Number** to every message. If the network drops during ACK transmission and the producer resends the message, the broker discards the duplicate sequence number, guaranteeing exactly-once delivery.

---

## 3. Consumer Internals & Group Rebalancing

Consumers read messages by polling brokers and tracking their read position (Offset) inside each partition.

```text
Consumer Group Partition Load Balancing:

Topic (4 Partitions):      Partition 0      Partition 1      Partition 2      Partition 3
                               │                │                │                │
Consumer Group (2 Consumers):  └───────┬────────┘                └───────┬────────┘
                                       ▼                                 ▼
                                  Consumer A                        Consumer B
```

* **Consumer Group**: Multiple consumers sharing the same `group.id` divide the partitions of a topic among themselves. One partition is consumed by exactly one consumer in the group at a time.
* **Rebalancing**: If a consumer crashes or a new consumer joins, the group coordinator broker triggers a **Rebalance**, reassigning partitions among active members.
* **Offset Commits**:
  * **Auto-Commit (`enable.auto.commit=true`)**: Automatically commits offsets every 5 seconds. High risk of **Data Loss** (if consumer crashes after offset is committed but before processing completes) or **Duplicate Processing** (if consumer crashes midway).
  * **Manual Commit**: Disable auto-commit and call `commitSync()` (blocking) or `commitAsync()` (non-blocking) in code after processing a batch of records.

---

## 4. Administrative & Consumer Group CLI Commands

### Topic Management (`kafka-topics.sh`):
```bash
# Create a topic with 6 partitions and a 3x replication factor
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic production-logs --partitions 6 --replication-factor 3

# Describe a topic (shows partition layout, leaders, and ISR status)
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic production-logs
```

### Producing & Consuming (`kafka-console-producer.sh` / `kafka-console-consumer.sh`):
```bash
# Produce messages from console (sending keys separated by a colon)
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic production-logs --property parse.key=true --property key.separator=:

# Consume all historical messages showing keys and timestamps
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic production-logs --from-beginning --property print.key=true --property print.timestamp=true
```

### Managing Consumer Groups (`kafka-consumer-groups.sh`):
```bash
# List all active consumer groups in the cluster
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# Describe group status (shows offset lag per partition)
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group analytics-consumer-group

# Reset offsets of a group to the earliest available offset (for reprocessing)
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group analytics-consumer-group --reset-offsets --to-earliest --topic production-logs --execute
```

---

## 5. Performance Tuning & Configurations

Edit configurations in `server.properties` (Broker) or client code:

* **`min.insync.replicas`**: 
  * Default: `1`. 
  * Tuning: Set to `2` on a 3x replication topic. Combined with `acks=all`, this ensures writes fail if less than two brokers are in sync, preventing split-brain data loss.
* **Zero-Copy Optimization**:
  * > [!TIP]
    * Kafka reads segments from disk and writes directly to the network socket using the Linux kernel `sendfile` system call. This bypasses copying data into the user-space JVM application buffer, avoiding CPU overhead and JVM garbage collection cycles.
* **Partition Count Formula**:
  * Aim to match partition counts to consumer throughput capacity:
    $$\text{Partitions} = \max\left(\frac{\text{Target Throughput}}{\text{Producer Throughput}}, \frac{\text{Target Throughput}}{\text{Consumer Throughput}}\right)$$

---

## 🎯 Exam and Interview Traps

1. **Trap: Why is my consumer group lag increasing even when I add more consumers to the group?**
   * **Answer**: You cannot have more consumers than partitions in a consumer group. If a topic has `6` partitions and you run `8` consumers, `2` consumers will sit completely idle. To resolve lag, you must first increase the partition count of the topic using `kafka-topics.sh --alter --partitions 8` and then spin up the additional consumers.

2. **Trap: What causes a "Duplicate Offset Commit" or constant consumer rebalancing loops?**
   * **Answer**: If a consumer takes too long to process a polled batch of records (exceeding `max.poll.interval.ms`, default 5 minutes), the group coordinator assumes the consumer is dead. It evicts the consumer and triggers a rebalance. When the evicted consumer finishes processing and tries to commit its offsets, the commit fails because its partition ownership has been revoked, leading to duplicate processing. Fix this by decreasing `max.poll.records` or optimizing processing speed.

3. **Trap: Why does my Kafka cluster fail to write messages even when the leader broker is up?**
   * **Answer**: If `min.insync.replicas=2` is set, and the topic replication factor is `3`, and two follower brokers go offline, only the leader is in the ISR. If a producer writes with `acks=all` (or `acks=-1`), the write will fail with a `NotEnoughReplicasException` because the leader cannot satisfy the minimum in-sync replica count requirement.
