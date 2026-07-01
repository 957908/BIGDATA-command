
08_Spark_Commands/spark_master_commands_part2.md
```

---

# SECTION 51 — TRANSFORMATIONS vs ACTIONS

### Transformations (lazy execution)

Examples:

```scala
rdd.map()
rdd.filter()
rdd.flatMap()
rdd.distinct()
```

Execution:

```text
Run only when action called
```

---

### Actions (trigger execution)

Examples:

```scala
rdd.collect()
rdd.count()
rdd.first()
rdd.take(5)
```

⭐ Exam trap: transformations lazy hote hain

---

# SECTION 52 — LAZY EVALUATION CONCEPT

Spark executes job only after:

```scala
rdd.collect()
```

Example:

```scala
val data = rdd.map(x => x + 1)
```

Runs when:

```scala
data.count()
```

---

# SECTION 53 — DAG (DIRECTED ACYCLIC GRAPH)

Spark execution plan stored as:

```text
DAG
```

Purpose:

```text
Optimize execution before running tasks
```

View DAG in:

```text
Spark UI → http://localhost:4040
```

---

# SECTION 54 — STAGES IN SPARK EXECUTION

Spark job divided into:

```text
Stages
```

Stage divided into:

```text
Tasks
```

Example pipeline:

```text
Job → Stage → Task
```

---

# SECTION 55 — NARROW DEPENDENCY

Example:

```scala
rdd.map()
rdd.filter()
```

Meaning:

```text
Each partition depends on one parent partition
```

No shuffle required ✅

---

# SECTION 56 — WIDE DEPENDENCY

Example:

```scala
groupByKey()
reduceByKey()
join()
```

Meaning:

```text
Multiple partitions required
```

Shuffle required ❌

⭐ Very important exam concept

---

# SECTION 57 — SHUFFLE OPERATION

Shuffle happens during:

```scala
reduceByKey()
groupByKey()
join()
```

Moves data across nodes

Costly operation ⚠️

---

# SECTION 58 — CACHE vs PERSIST DIFFERENCE

Cache:

```scala
rdd.cache()
```

Stores in:

```text
Memory only
```

Persist:

```scala
rdd.persist()
```

Stores in:

```text
Memory + Disk
```

---

# SECTION 59 — STORAGE LEVEL OPTIONS

Example:

```scala
rdd.persist(StorageLevel.MEMORY_ONLY)
```

Other options:

```scala
MEMORY_AND_DISK
DISK_ONLY
MEMORY_ONLY_SER
```

---

# SECTION 60 — CHECK RDD STORAGE STATUS

```scala
sc.getPersistentRDDs
```

Lists cached datasets

---

# SECTION 61 — BROADCAST VARIABLE CREATION

Used to send read-only variable to all nodes

```scala
val broadcastVar = sc.broadcast(Array(1,2,3))
```

Access:

```scala
broadcastVar.value
```

Improves performance 🚀

---

# SECTION 62 — WHEN TO USE BROADCAST VARIABLE

Use when:

```text
Small lookup dataset shared across nodes
```

Example:

```text
Country code lookup table
```

---

# SECTION 63 — ACCUMULATOR VARIABLE

Used for counters

Example:

```scala
val acc = sc.longAccumulator("Counter")
```

Update:

```scala
acc.add(1)
```

Read:

```scala
acc.value
```

---

# SECTION 64 — CUSTOM ACCUMULATOR USAGE

Example:

```scala
val errorCounter = sc.longAccumulator("Errors")
```

Inside transformation:

```scala
errorCounter.add(1)
```

---

# SECTION 65 — DEFAULT PARALLELISM CONTROL

Check:

```scala
sc.defaultParallelism
```

Used to determine number of partitions

---

# SECTION 66 — REPARTITION OPERATION

Increase partitions:

```scala
rdd.repartition(6)
```

Shuffle required ⚠️

---

# SECTION 67 — COALESCE OPERATION

Reduce partitions:

```scala
rdd.coalesce(2)
```

No shuffle (default) ✅

---

# SECTION 68 — DIFFERENCE: REPARTITION vs COALESCE

| Operation   | Shuffle | Usage               |
| ----------- | ------- | ------------------- |
| repartition | Yes     | Increase partitions |
| coalesce    | No      | Reduce partitions   |

⭐ Exam favorite

---

# SECTION 69 — CHECK PARTITION COUNT

```scala
rdd.getNumPartitions
```

---

# SECTION 70 — MAP vs FLATMAP DIFFERENCE

Example:

```scala
rdd.map(line => line.split(" "))
```

Returns:

```text
Array(Array(words))
```

FlatMap:

```scala
rdd.flatMap(line => line.split(" "))
```

Returns:

```text
Array(words)
```

---

# SECTION 71 — GROUPBYKEY vs REDUCEBYKEY DIFFERENCE

groupByKey:

```scala
rdd.groupByKey()
```

Transfers full dataset ❌ slow

reduceByKey:

```scala
rdd.reduceByKey(_+_)
```

Local aggregation first ✅ fast

⭐ Important optimization concept

---

# SECTION 72 — MAPSIDE COMBINE CONCEPT

Occurs in:

```scala
reduceByKey()
```

Reduces shuffle size

Improves performance 🚀

---

# SECTION 73 — SPARK EXECUTOR CONCEPT

Executor runs:

```text
Tasks
```

Inside:

```text
Worker nodes
```

Responsible for:

```text
Task execution
Caching
Shuffle storage
```

---

# SECTION 74 — DRIVER PROGRAM ROLE

Driver controls:

```text
Task scheduling
Execution planning
DAG generation
```

Runs:

```text
Inside spark-submit process
```

---

# SECTION 75 — CLUSTER MANAGER TYPES

Supported:

```text
Standalone
YARN
Mesos
Kubernetes
```

Most common:

```text
YARN
```

---

# SECTION 76 — SPARK EXECUTION MODES

Client mode:

```text
Driver runs locally
```

Cluster mode:

```text
Driver runs inside cluster
```

---

# SECTION 77 — SET EXECUTOR MEMORY

```bash
spark-submit \
--executor-memory 4G app.jar
```

---

# SECTION 78 — SET DRIVER MEMORY

```bash
spark-submit \
--driver-memory 2G app.jar
```

---

# SECTION 79 — SET NUMBER OF EXECUTORS

```bash
spark-submit \
--num-executors 5 app.jar
```

---

# SECTION 80 — SET EXECUTOR CORES

```bash
spark-submit \
--executor-cores 2 app.jar
```

---

# SECTION 81 — ENABLE DYNAMIC RESOURCE ALLOCATION

```bash
spark.dynamicAllocation.enabled=true
```

Allocates executors automatically

---

# SECTION 82 — CHECK EXECUTOR STATUS

Open Spark UI:

```text
http://localhost:4040
```

Shows:

```text
Active executors
Memory usage
Task status
```

---

# SECTION 83 — SPARK SHUFFLE PARTITION CONTROL

```scala
spark.conf.set("spark.sql.shuffle.partitions", 10)
```

Default:

```text
200
```

⭐ Exam trap

---

# SECTION 84 — CACHE DATAFRAME

```scala
df.cache()
```

Stores dataframe in memory

---

# SECTION 85 — PERSIST DATAFRAME

```scala
df.persist()
```

Stores memory + disk

---

# SECTION 86 — CREATE GLOBAL TEMP VIEW

```scala
df.createGlobalTempView("people")
```

Accessible across sessions

---

# SECTION 87 — ACCESS GLOBAL TEMP VIEW

```scala
spark.sql("SELECT * FROM global_temp.people")
```

---

# SECTION 88 — SHOW DATABASES (SPARK SQL)

```scala
spark.sql("SHOW DATABASES").show()
```

---

# SECTION 89 — CREATE DATABASE (SPARK SQL)

```scala
spark.sql("CREATE DATABASE testdb")
```

---

# SECTION 90 — USE DATABASE

```scala
spark.sql("USE testdb")
```

---

# SECTION 91 — SHOW TABLES

```scala
spark.sql("SHOW TABLES").show()
```

---

# SECTION 92 — CREATE TABLE USING DATAFRAME

```scala
df.write.saveAsTable("employee")
```

---

# SECTION 93 — DROP TABLE

```scala
spark.sql("DROP TABLE employee")
```

---

# SECTION 94 — CHECK CURRENT DATABASE

```scala
spark.catalog.currentDatabase
```

---

# SECTION 95 — ENABLE HIVE SUPPORT IN SPARK

```scala
SparkSession.builder.enableHiveSupport().getOrCreate()
```

Integrates Spark with Hive

---

# SECTION 96 — CHECK SPARK CONFIGURATION

```scala
spark.conf.getAll
```

Displays configuration settings

---

# SECTION 97 — CHECK APPLICATION NAME

```scala
sc.appName
```

---

# SECTION 98 — STOP SPARK CONTEXT

```scala
sc.stop()
```

Stops Spark application safely

---

# SECTION 99 — CHECK EXECUTION PLAN

```scala
df.explain(true)
```

Shows logical + physical plan

⭐ Exam important

---

# SECTION 100 — MOST IMPORTANT SPARK EXAM TRAPS 🎯

Remember:

```
Transformations = lazy
Actions = trigger execution
RDD immutable
Shuffle expensive operation
reduceByKey faster than groupByKey
Broadcast variables = read-only
Accumulators = write-only worker side
Spark UI port = 4040
Default shuffle partitions = 200
```

