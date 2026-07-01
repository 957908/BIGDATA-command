# 🌊 Module 06: Apache Spark Structured Streaming

Spark Structured Streaming is a scalable and fault-tolerant stream processing engine built on the Spark SQL engine. It allows developers to express streaming computations the same way they express batch computations over static data. The engine treats live data streams as an **Unbounded Table** that is constantly appended.

---

## 1. Structured Streaming Execution Models

Structured Streaming offers two main processing modes depending on the latency requirements of the business logic.

```mermaid
flowchart TD
    subgraph Micro-Batch Processing (Default)
        In_MB[Live Stream Input] --> Trigger_MB{Trigger Every X ms}
        Trigger_MB -- Yes --> Batch_Job[Executor Spark SQL Job]
        Batch_Job --> Sink_MB[Output Sink]
    end

    subgraph Continuous Processing (Low Latency)
        In_CP[Live Stream Input] --> Long_Running[Continuous Task Threads]
        Long_Running -- Asynchronous Epochs --> Sink_CP[Output Sink]
    end

    style Trigger_MB fill:#ff9,stroke:#333
    style Long_Running fill:#bbf,stroke:#333
```

### Micro-Batch Processing (Default):
* **Mechanism**: The streaming engine periodically checks the input source (e.g., every 500ms) for new data, creates a micro-batch Spark SQL query, executes it across the cluster executors, and writes the output.
* **Latency**: Typical latencies range from `10ms` to `100ms`.
* **Guarantees**: End-to-end **Exactly-Once** processing when combined with replayable sources (like Kafka) and idempotent sinks.

### Continuous Processing (Spark 2.3+):
* **Mechanism**: Spark launches long-running tasks that continuously poll the sources for events. It commits state and offsets asynchronously using checkpoint epochs.
* **Latency**: Sub-millisecond end-to-end latency (`<1ms`).
* **Guarantees**: **At-Least-Once** processing. Some stateful operations are restricted.

---

## 2. Event-Time, Processing-Time, and Windowing

Processing time series data requires separating when an event occurred from when it was processed by the cluster.

* **Event Time**: The timestamp embedded within the record itself, generated at the source device.
* **Processing Time**: The local time of the Spark executor node executing the task.

### Windowing Semantics:
1. **Tumbling Windows**: Non-overlapping, fixed-duration time blocks (e.g., every 10 minutes: 12:00-12:10, 12:10-12:20).
2. **Sliding Windows**: Overlapping, fixed-duration time blocks that slide at a specified interval (e.g., 10-minute window sliding every 5 minutes: 12:00-12:10, 12:05-12:15).
3. **Session Windows**: Dynamic, data-driven windows defined by gaps of inactivity (e.g., group web-logs until a user goes idle for 30 minutes).

---

## 3. Watermarking: Handling Late Data & State Cleanup

In real-world streaming, network lags cause events to arrive out of order (late data). Spark maintains the aggregate state of windows in memory (StateStore). Without watermarking, memory will eventually overflow.

### What is a Watermark?
A watermark is a sliding threshold that trails behind the maximum event time seen by Spark. It defines how late a record can be before it is rejected.

$$\text{Watermark} = \max(\text{Event Time seen so far}) - \text{Allowed Late Time}$$

```text
Timeline of Watermarking Example (Allowed Late Time = 10 mins):

Event Time Seen:  12:15                  12:25                  12:30
Watermark:       12:05                  12:15                  12:20
                  │                      │                      │
                  └─ [Records < 12:05]   └─ [Records < 12:15]   └─ [Records < 12:20]
                     are DISCARDED          are DISCARDED          are DISCARDED
```

* **State Cleanup**: Windows whose end-times are older than the watermark are finalized and flushed from memory, preventing memory leaks.
* **Late Data**: If a record arrives with a timestamp greater than the current watermark, the corresponding window is updated in memory. If the timestamp is less than the watermark, the record is dropped.

---

## 4. Fault Tolerance: Checkpointing & Exactly-Once

To guarantee exactly-once processing across application failures, Spark uses **Metadata Checkpointing** and **Write-Ahead Logs (WAL)**.

```xml
<!-- Pre-requisites for Exactly-Once -->
1. Replayable Source: The source must support seeking back to offsets (e.g., Kafka).
2. Metadata Checkpoint: Spark writes state and partition offsets to a secure filesystem (HDFS/S3).
3. Idempotent Sink: The sink must handle duplicate writes without corrupting final values.
```

### Specifying Checkpoint Directory:
When starting a streaming query, configure `checkpointLocation`. If the application crashes, restarting it with the same checkpoint location allows Spark to read the WAL, rebuild the memory state, and resume from the exact offsets where it stopped.

---

## 5. Complete Spark Streaming API & Configuration Reference

This library covers the streaming connector properties and query controls.

### A. Kafka Streaming Source Options
```python
stream_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "iot-events") \
    .option("startingOffsets", "latest") \
    .option("failOnDataLoss", "false") \
    .option("maxOffsetsPerTrigger", "50000") \
    .load()
```
* **`startingOffsets`**: Read position at startup (`earliest`, `latest`, or JSON map of offsets).
* **`failOnDataLoss`**: Raise alert/fail if offsets disappear from Kafka brokers (e.g., due to log deletion retention). Set to `false` for robust recovery.
* **`maxOffsetsPerTrigger`**: Sets rate-limiting bounds (max messages per micro-batch). Prevents cluster crash during peak load restarts.

### B. Output Modes & Write Stream APIs
```python
query = processed_df.writeStream \
    .format("parquet") \
    .outputMode("append") \
    .option("path", "hdfs:///warehouse/outputs") \
    .option("checkpointLocation", "hdfs:///checkpoints/stream_1") \
    .trigger(processingTime="30 seconds") \
    .start()
```
* **`outputMode()`**: Specifies write policies.
  * `append` (default): Only new rows added to the result table are written. Best for raw ingestion.
  * `update`: Only rows updated in the result table since the last trigger are written. Best for windowed aggregations.
  * `complete`: The entire updated result table is rewritten to the sink. Best for global summaries.
* **`trigger()`**: Sets epoch boundaries.
  * `processingTime="30 seconds"`: Micro-batch trigger.
  * `once=True`: Evaluates the query once as a single batch and exits (useful for nightly delta syncs).
  * `continuous="1 second"`: Low-latency continuous mode.

### C. Streaming Query Runtime Operations
* **`query.id`**: Unique static query ID.
* **`query.runId`**: Unique instance ID for the current execution run.
* **`query.status`**: Check state (e.g., `{'message': 'Active', 'isDataAvailable': true, 'isTriggerActive': false}`).
* **`query.lastProgress`**: Inspect metrics (throughput rates, processed offsets, state storage size).
* **`query.stop()`**: Stop execution.
* **`spark.streams.active`**: Return a list of all active streaming queries running under the current SparkSession.

---

## 6. Structured Streaming PySpark Code Implementation

Here is a master-level PySpark template reading from Kafka, applying watermarking and windowed aggregations, and writing results to a console sink.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, from_json, window, current_timestamp
from pyspark.sql.types import StructType, StructField, StringType, DoubleType

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("KafkaStructuredStreamingMaster") \
    .config("spark.sql.shuffle.partitions", "4") \
    .getOrCreate()

# Define schema for incoming JSON payloads
iot_schema = StructType([
    StructField("device_id", StringType(), True),
    StructField("timestamp", StringType(), True), # ISO-8601 string
    StructField("temperature", DoubleType(), True)
])

# 1. Read Stream from Apache Kafka
raw_stream = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "iot-sensor-data") \
    .option("startingOffsets", "latest") \
    .load()

# 2. Deserialize Value byte array and cast schema
deserialized_stream = raw_stream \
    .selectExpr("CAST(value AS STRING) as json_payload") \
    .select(from_json(col("json_payload"), iot_schema).alias("data")) \
    .select("data.*")

# 3. Cast Event Time column to Timestamp type
processed_stream = deserialized_stream \
    .withColumn("event_time", col("timestamp").cast("timestamp"))

# 4. Window Aggregation with 10-Minute Watermark
# Group by 5-minute sliding windows every 1 minute
windowed_aggregates = processed_stream \
    .withWatermark("event_time", "10 minutes") \
    .groupBy(
        window(col("event_time"), "5 minutes", "1 minute"),
        col("device_id")
    ) \
    .agg({"temperature": "avg"}) \
    .select(
        col("window.start").alias("window_start"),
        col("window.end").alias("window_end"),
        col("device_id"),
        col("avg(temperature)").alias("avg_temp")
    )

# 5. Write Output Stream to Console (Update Mode for aggregates)
query = windowed_aggregates.writeStream \
    .format("console") \
    .outputMode("update") \
    .option("checkpointLocation", "hdfs:///tmp/kafka_spark_checkpoint") \
    .trigger(processingTime="10 seconds") \
    .start()

# Block until termination signal
query.awaitTermination()
```

---

## 7. Enterprise Job Interview Q&A (Spark Streaming)

This section prepares you for production-level interview questions.

### Q1: What is a Watermark in Spark Structured Streaming? How does it handle late data, and how does it clean up the StateStore from memory leaks?
* **How to explain this to the interviewer**:
  Explain that watermarking tracks event time (embedded in records) and subtracts the late threshold. Walk through the math of how incoming records are either processed or discarded, and how finished windows are deleted from the StateStore JVM memory.

* **Model Answer**:
  "A watermark is a dynamic event-time threshold used in stateful streaming queries to handle late-arriving data and prevent memory exhaustion.
  
  Mathematically, Spark tracks the maximum event-time value seen across all partitions. The watermark is calculated as:
  $$\text{Watermark} = \max(\text{Event Time seen so far}) - \text{Allowed Late Time}$$
  
  **How it handles late data**:
  When a new record arrives:
  1. If its event-time is **greater** than the current watermark, Spark accepts it, updates the aggregated window state in the StateStore, and writes updates to the sink.
  2. If its event-time is **less** than the watermark, Spark classifies it as 'too late' and discards it.
  
  **How it prevents memory leaks**:
  Without watermarking, Spark must keep the state for every past time window in memory forever, as it assumes records for any past date could eventually arrive. This causes the StateStore (usually running inside HDFS-backed RocksDB or JVM RAM) to overflow. When the watermark progresses past the end-time of a specific window, Spark knows no more records for that window will be accepted. It deletes the window state from memory, preventing memory leaks."

---

### Q2: Compare the three output modes in Spark Structured Streaming: Append, Update, and Complete. When would you use each?
* **How to explain this to the interviewer**:
  Clearly describe what is written to the sink at each micro-batch trigger, and match each mode to a real-world scenario.

* **Model Answer**:
  "Structured Streaming supports three output modes depending on the query logic:
  
  1. **Append Mode** (Default):
     * *Behavior*: Only new rows added to the result table since the last micro-batch are written to the sink. Existing rows cannot be modified.
     * *Use Case*: Raw ingestion pipelines (e.g. reading from Kafka and writing directly to Parquet files). It is the only mode supported for file-based sinks because files are append-only.
     
  2. **Update Mode**:
     * *Behavior*: Only rows that were updated or newly added in the result table since the last trigger are written. If a row is unchanged, it is not output.
     * *Use Case*: Real-time aggregates (e.g. hourly page views per device). It writes updates to database sinks (like Cassandra or JDBC) without rewriting the entire table.
     
  3. **Complete Mode**:
     * *Behavior*: The entire updated result table is rewritten to the sink every time a trigger runs.
     * *Use Case*: Dashboard summaries (e.g. overall count of errors). Since it rewrites all states, it is only supported if the query contains aggregations (otherwise the result table would grow indefinitely)."

---

### Q3: How does Spark Structured Streaming guarantee end-to-end exactly-once processing? Detail the requirements.
* **How to explain this to the interviewer**:
  Explain that exactly-once is a coordinated effort between the input source, the processing engine's checkpointing system, and the output sink. Walk through the failure-and-recovery sequence.

* **Model Answer**:
  "End-to-end exactly-once processing requires a coordinated handshake across three components:
  
  1. **Replay-able Source**: The input source must support re-reading data from specified offsets (like Kafka partition offsets). If a crash occurs, Spark must be able to pull the exact same data again.
  2. **Engine Checkpointing & WAL**: Spark must write metadata checkpoints and a Write-Ahead Log (WAL) to a persistent, secure file system (HDFS/S3). This logs the exact partition offsets processed in each micro-batch trigger.
  3. **Idempotent Sink**: The output sink must be able to handle duplicate writes from the same offset without corrupting the final state. This can be achieved through:
     * *Idempotency*: Overwriting keys based on unique IDs (e.g. database upsert).
     * *Atomic Transactions*: A two-phase commit protocol (like writing files to a temporary directory and committing them via atomic file renames when the batch completes).
  
  **Recovery Protocol**:
  If an executor crashes mid-batch:
  * The Spark Driver restarts, reads the checkpoint WAL, and identifies the uncommitted batch offsets.
  * It pulls the data from those exact offsets from the replay-able source.
  * It re-executes the processing.
  * The idempotent sink filters out the duplicates or overwrites them, guaranteeing no double-counting."
