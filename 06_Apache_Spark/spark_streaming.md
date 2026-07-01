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

* **Event Time**: The timestamp embedded within the record itself, generated at the source device (e.g., log time, sensor timestamp).
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

## 5. Structured Streaming PySpark Code Implementation

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

## 🎯 Exam and Interview Traps

1. **Trap: Why are my watermark aggregates not emitting any output in `Append` output mode?**
   * **Answer**: In `Append` mode, rows are only written to the sink once they are final. Because a window can receive late-arriving data, Spark cannot write the window data until the watermark progresses past the window's end-time. If the stream stops receiving new events, the watermark does not progress, and no data is written. Use `Update` mode to see real-time updates as records arrive.

2. **Trap: Can we join a streaming DataFrame with a static DataFrame, and what are the limitations?**
   * **Answer**: Yes, Stream-Static joins are supported (e.g., lookup table joined to transaction stream). However, the static DataFrame is loaded once at query startup and is not automatically refreshed if the underlying source table changes. If you need dynamic static updates, you must use custom stateful processing (`mapGroupsWithState`) or restart the streaming query.

3. **Trap: Why does my streaming application crash with Out-Of-Memory (OOM) errors even when I have watermarking enabled?**
   * **Answer**: Watermarking only cleans up state when grouping by columns containing the event-time timestamp. If you group by `device_id` alone (without referencing the `window` or event-time column), Spark cannot discard older states because it expects device records to arrive indefinitely. Ensure `event_time` is part of the `groupBy` fields.
