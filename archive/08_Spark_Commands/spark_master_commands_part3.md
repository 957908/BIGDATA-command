

```bash
08_Spark_Commands/spark_master_commands_part3.md
```

---

# SECTION 101 — SPARK ARCHITECTURE OVERVIEW

Execution pipeline:

```
Driver → Cluster Manager → Executors → Tasks → Results
```

Components:

* Driver → controls execution
* Executors → run tasks
* Cluster Manager → resource allocation
* Tasks → smallest execution unit

⭐ Very common architecture question

---

# SECTION 102 — SPARK SQL ENGINE COMPONENTS

Spark SQL internally uses:

```
Parser → Analyzer → Optimizer → Physical Planner → Execution Engine
```

Optimizer used:

```
Catalyst Optimizer
```

Execution engine:

```
Tungsten Engine
```

---

# SECTION 103 — CATALYST OPTIMIZER

Catalyst performs:

```
Logical plan optimization
Predicate pushdown
Column pruning
Join reordering
Constant folding
```

Example:

```scala
df.select("name").filter("age > 25")
```

Optimizer removes unnecessary columns automatically

⭐ Exam favorite topic

---

# SECTION 104 — TUNGSTEN EXECUTION ENGINE

Purpose:

```
Improve CPU efficiency
Reduce memory usage
Optimize binary processing
```

Techniques used:

```
Binary encoding
Cache-aware computation
Whole-stage code generation
```

---

# SECTION 105 — WHOLE-STAGE CODE GENERATION

Spark converts query execution into:

```
Optimized Java bytecode
```

Benefit:

```
Faster execution
Less JVM overhead
```

---

# SECTION 106 — DATAFRAME vs RDD DIFFERENCE

| Feature      | RDD    | DataFrame |
| ------------ | ------ | --------- |
| Optimization | No     | Yes       |
| Catalyst     | No     | Yes       |
| Schema       | No     | Yes       |
| Performance  | Slower | Faster    |

⭐ Exam trap question

---

# SECTION 107 — DATASET vs DATAFRAME DIFFERENCE

| Feature        | Dataset | DataFrame |
| -------------- | ------- | --------- |
| Type safe      | Yes     | No        |
| JVM only       | Yes     | No        |
| Python support | No      | Yes       |

---

# SECTION 108 — STRUCTURED STREAMING INTRODUCTION

Structured Streaming processes:

```
Real-time streaming data
```

Sources:

```
Kafka
Socket
Files
Rate source
```

Example:

```scala
spark.readStream.format("socket")
.option("host","localhost")
.option("port",9999)
.load()
```

---

# SECTION 109 — START STREAMING QUERY

Example:

```scala
df.writeStream
.outputMode("append")
.format("console")
.start()
.awaitTermination()
```

Runs continuous streaming job

---

# SECTION 110 — STREAMING OUTPUT MODES

Three modes:

```
Append
Update
Complete
```

Difference:

| Mode     | Output         |
| -------- | -------------- |
| Append   | Only new rows  |
| Update   | Updated rows   |
| Complete | Entire dataset |

⭐ Important MCQ topic

---

# SECTION 111 — STREAMING TRIGGER INTERVAL

Example:

```scala
.trigger(processingTime="10 seconds")
```

Runs batch every 10 seconds

---

# SECTION 112 — CHECKPOINTING IN STREAMING

Used for:

```
Fault tolerance
Recovery
State management
```

Example:

```scala
.option("checkpointLocation","/tmp/checkpoint")
```

---

# SECTION 113 — WATERMARKING CONCEPT

Handles:

```
Late arriving data
```

Example:

```scala
.withWatermark("timestamp","10 minutes")
```

---

# SECTION 114 — WINDOW OPERATIONS IN STREAMING

Example:

```scala
groupBy(
window($"timestamp","10 minutes")
)
.count()
```

Used for:

```
Time-based aggregation
```

---

# SECTION 115 — SPARK SQL JOIN OPTIMIZATION

Types:

```
Broadcast Join
Sort Merge Join
Shuffle Hash Join
```

Broadcast join example:

```scala
broadcast(df_small)
```

Used when:

```
Small dataset joins large dataset
```

---

# SECTION 116 — ENABLE BROADCAST JOIN THRESHOLD

Check threshold:

```scala
spark.conf.get("spark.sql.autoBroadcastJoinThreshold")
```

Default:

```
10MB
```

---

# SECTION 117 — DISABLE BROADCAST JOIN

Example:

```scala
spark.conf.set("spark.sql.autoBroadcastJoinThreshold",-1)
```

---

# SECTION 118 — PREDICATE PUSHDOWN OPTIMIZATION

Example:

```scala
df.filter("age > 25")
```

Spark pushes filter to:

```
Data source level
```

Reduces scan size ⚡

---

# SECTION 119 — COLUMN PRUNING OPTIMIZATION

Example:

```scala
df.select("name")
```

Reads only:

```
Required columns
```

---

# SECTION 120 — CACHE TABLE IN SPARK SQL

Example:

```scala
spark.catalog.cacheTable("employee")
```

Improves repeated query speed 🚀

---

# SECTION 121 — UNCACHE TABLE

```scala
spark.catalog.uncacheTable("employee")
```

Removes cached table

---

# SECTION 122 — CHECK CACHED TABLES

```scala
spark.catalog.isCached("employee")
```

Returns:

```
true / false
```

---

# SECTION 123 — EXPLAIN QUERY PLAN

Example:

```scala
df.explain(true)
```

Shows:

```
Logical plan
Optimized plan
Physical plan
```

⭐ Very important exam question

---

# SECTION 124 — ADAPTIVE QUERY EXECUTION (AQE)

Enable:

```scala
spark.conf.set("spark.sql.adaptive.enabled",true)
```

Improves:

```
Join strategy
Partition size
Shuffle handling
```

---

# SECTION 125 — COALESCE SHUFFLE PARTITIONS (AQE FEATURE)

Example:

```scala
spark.conf.set(
"spark.sql.adaptive.coalescePartitions.enabled",
true
)
```

Reduces shuffle partitions automatically

---

# SECTION 126 — SKEW JOIN HANDLING (AQE)

Enable:

```scala
spark.conf.set(
"spark.sql.adaptive.skewJoin.enabled",
true
)
```

Handles uneven key distribution

---

# SECTION 127 — DYNAMIC PARTITION PRUNING

Enable:

```scala
spark.conf.set(
"spark.sql.optimizer.dynamicPartitionPruning.enabled",
true
)
```

Improves join performance

---

# SECTION 128 — FILE SOURCE PARTITION DISCOVERY

Example:

```
data/year=2024/month=05
```

Spark automatically detects:

```
Partition columns
```

---

# SECTION 129 — PARQUET OPTIMIZATION SETTINGS

Enable vectorized reader:

```scala
spark.conf.set(
"spark.sql.parquet.enableVectorizedReader",
true
)
```

Improves read speed

---

# SECTION 130 — ORC OPTIMIZATION SETTINGS

Enable ORC filter pushdown:

```scala
spark.conf.set(
"spark.sql.orc.filterPushdown",
true
)
```

---

# SECTION 131 — SPARK MEMORY MANAGEMENT MODEL

Memory divided into:

```
Execution memory
Storage memory
```

Execution memory used for:

```
Shuffle
Join
Sort
Aggregation
```

Storage memory used for:

```
Cache
Persist
Broadcast
```

---

# SECTION 132 — UNIFIED MEMORY MANAGER

Spark uses:

```
Unified Memory Manager
```

Allows sharing between:

```
Execution memory
Storage memory
```

---

# SECTION 133 — SPARK FAULT TOLERANCE MECHANISM

RDD fault tolerance based on:

```
Lineage graph
```

Recomputes lost partitions automatically

---

# SECTION 134 — CHECKPOINTING vs CACHING DIFFERENCE

| Feature         | Checkpoint | Cache  |
| --------------- | ---------- | ------ |
| Storage         | Disk       | Memory |
| Lineage removed | Yes        | No     |
| Speed           | Slower     | Faster |

---

# SECTION 135 — ENABLE CHECKPOINT DIRECTORY

Example:

```scala
sc.setCheckpointDir("/tmp/checkpoint")
```

---

# SECTION 136 — CREATE CHECKPOINT RDD

Example:

```scala
rdd.checkpoint()
```

---

# SECTION 137 — SPARK SHUFFLE FILE LOCATION

Stored inside:

```
Executor local disk
```

Managed by:

```
Shuffle Manager
```

---

# SECTION 138 — TASK SCHEDULER ROLE

Responsible for:

```
Assign tasks to executors
Retry failed tasks
Maintain locality
```

---

# SECTION 139 — DAG SCHEDULER ROLE

Responsible for:

```
Divide job into stages
Detect shuffle boundaries
Build execution DAG
```

---

# SECTION 140 — MOST IMPORTANT SPARK INTERNAL EXAM TRAPS 🎯

Remember these:

```
Catalyst = query optimizer
Tungsten = execution engine
Default shuffle partitions = 200
Broadcast threshold = 10MB
RDD fault tolerance = lineage
Spark UI port = 4040
AQE improves runtime optimization
Structured Streaming = micro-batch processing
```
