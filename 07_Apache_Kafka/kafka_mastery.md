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
  * **Follower**: Passive replicas that sync data from the leader.
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
  * `acks=0`: Fire-and-forget. The producer does not wait for broker acknowledgments.
  * `acks=1`: The leader writes the record to its local log and acknowledges.
  * `acks=all` (or `-1`): The leader waits for all in-sync replicas (ISR) to acknowledge the write.
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
  * **Auto-Commit (`enable.auto.commit=true`)**: Automatically commits offsets every 5 seconds. High risk of **Data Loss** or **Duplicate Processing**.
  * **Manual Commit**: Disable auto-commit and call `commitSync()` (blocking) or `commitAsync()` (non-blocking) in code after processing a batch of records.

---

## 4. Complete Apache Kafka CLI Reference Library

This library lists all the essential CLI commands for managing an enterprise Kafka cluster.

### A. Topic Administration (`kafka-topics.sh`)
* **Create Topic**:
  ```bash
  kafka-topics.sh --bootstrap-server localhost:9092 --create --topic app-logs --partitions 6 --replication-factor 3
  ```
* **List and Describe Topics**:
  ```bash
  kafka-topics.sh --bootstrap-server localhost:9092 --list
  kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic app-logs
  ```
* **Alter Topic (Increase Partitions)**:
  ```bash
  kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic app-logs --partitions 12
  ```
* **Delete Topic**:
  ```bash
  kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic app-logs
  ```

### B. Dynamic Configurations (`kafka-configs.sh`)
* **Update Log Retention Time (e.g. set to 24 hours)**:
  ```bash
  kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name app-logs --alter --add-config retention.ms=86400000
  ```
* **Describe Configuration Overrides**:
  ```bash
  kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name app-logs --describe
  ```

### C. Console Producing & Consuming
* **Console Producer with Keys & Snappy Compression**:
  ```bash
  kafka-console-producer.sh --bootstrap-server localhost:9092 --topic app-logs --property parse.key=true --property key.separator=: --compression-codec snappy --producer-property acks=all
  ```
* **Console Consumer showing Keys and Timestamps**:
  ```bash
  kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic app-logs --from-beginning --property print.key=true --property print.timestamp=true
  ```
* **Consume from a Specific Partition and Offset**:
  ```bash
  kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic app-logs --partition 0 --offset 12345 --max-messages 100
  ```

### D. Consumer Group Diagnostics (`kafka-consumer-groups.sh`)
* **List Active Groups**:
  ```bash
  kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
  ```
* **Describe Group Lag**:
  ```bash
  kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group analytics-group
  ```
* **Reset Offsets to Earliest (Reprocess everything)**:
  ```bash
  kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group analytics-group --reset-offsets --to-earliest --topic app-logs --execute
  ```
* **Shift Offsets backward by 10 positions**:
  ```bash
  kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group analytics-group --reset-offsets --shift-by -10 --topic app-logs --execute
  ```

### E. Security Access Control List (`kafka-acls.sh`)
* **Grant Read Access to a Consumer Application**:
  ```bash
  kafka-acls.sh --bootstrap-server localhost:9092 --add --allow-principal User:analytics_app --operation Read --group analytics-group --topic app-logs
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

## 6. Enterprise Job Interview Q&A (Apache Kafka)

This section prepares you for production-level interview questions.

### Q1: What triggers a Consumer Group Rebalance, what are the hazards during a rebalance, and how do you mitigate them?
* **How to explain this to the interviewer**:
  Define the rebalance trigger conditions (consumers joining/leaving, heartbeat loss, long processing times). Explain that partition consumption is suspended during rebalancing, and detail configurations like `max.poll.interval.ms`.

* **Model Answer**:
  "A Consumer Group Rebalance is triggered by the Broker Coordinator when:
  1. A new consumer joins the group, or an active consumer leaves/disconnects.
  2. A consumer fails to send heartbeats within `session.timeout.ms` (default 45 seconds).
  3. A consumer takes too long to process a single batch of records retrieved from a `poll()` call, exceeding the `max.poll.interval.ms` threshold (default 5 minutes).
  
  **Hazards**:
  * During a rebalance, all partition reading is suspended (Stop-the-World).
  * If a consumer was evicted because it exceeded `max.poll.interval.ms`, it will attempt to commit offsets for the batch once processing finishes. This commit will fail because the partition has already been reassigned to another consumer. The new consumer will read the exact same batch, leading to **duplicate data processing loops** and increased lag.
  
  **Mitigations**:
  1. Increase `max.poll.interval.ms` if tasks are slow, or decrease `max.poll.records` (default 500) to process smaller batches per poll.
  2. Implement manual asynchronous commits (`commitAsync()`) or commit offsets at logical milestones in code to minimize duplicate windows.
  3. Use newer Cooperative Sticky Assignors to rebalance only affected partitions instead of freezing the entire group."

---

### Q2: Explain Kafka's internal storage layout. How do log segments and index files work, and how does Kafka achieve high throughput on commodity hardware?
* **How to explain this to the interviewer**:
  Describe how partition folders contain `.log` and `.index` files. Detail the binary search lookup process, and explain how OS Page Cache and Zero-Copy bypass JVM bottlenecks.

* **Model Answer**:
  "In Kafka's data directory, each partition corresponds to a folder. Within this folder, Kafka splits data into **Segments** (typically 1GB files). Each segment has:
  * A `.log` file containing the serialized record payloads appended sequentially.
  * A `.index` file containing offset-to-physical-byte mappings.
  
  **Read Operations**: When a consumer requests offset 5000, Kafka doesn't scan the log file. It performs a binary search on the sparse `.index` file to locate the closest physical byte position (e.g. offset 4990 starts at byte 50024), jumps to that address in the `.log` file, and streams from that point.
  
  **High-Throughput Architecture**:
  1. **Sequential Writes**: Appending to a commit log is an $O(1)$ operation that works at sequential disk write speeds (as fast as network transfers), avoiding random disk seeks.
  2. **OS Page Cache**: Kafka doesn't cache messages in JVM memory (which would cause GC pauses). It writes directly to the OS page cache. Reads are served directly from RAM cache without hitting disk.
  3. **Zero-Copy (`sendfile`)**: When transmitting data to the network card, Kafka uses the Linux `sendfile` system call. The OS transfers bytes directly from the page cache to the network socket, avoiding copying data into JVM application memory and user-space context switches."

---

### Q3: How do you configure a Kafka pipeline for end-to-end exactly-once semantics (EOS)?
* **How to explain this to the interviewer**:
  Explain that exactly-once requires a three-way configuration: idempotent/transactional producers, transaction coordinators on the brokers, and read_committed isolation on consumers.

* **Model Answer**:
  "End-to-End Exactly-Once Semantics (EOS) in Kafka is achieved by configuring the Producer, Broker, and Consumer:
  
  1. **Producer Configuration**:
     * Set `enable.idempotence=true` (assigns sequence numbers to packets to discard duplicates).
     * Configure a `transactional.id` in the producer properties.
     * Use the transactional APIs: `initTransactions()`, `beginTransaction()`, `sendOffsetsToTransaction()` (to commit source offsets and output records in a single atomic transaction), and `commitTransaction()`.
     
  2. **Broker Configuration**:
     * Brokers run a **Transaction Coordinator** that writes atomic commit markers to a special internal topic `__transaction_state`.
     * Set `min.insync.replicas=2` and `acks=all` on topics to prevent partition leader crashes from losing transaction records.
     
  3. **Consumer Configuration**:
     * Set `isolation.level=read_committed`.
     * The consumer will block reading messages that are part of an open transaction, and will only read records once the broker's Transaction Coordinator appends the final commit marker."
