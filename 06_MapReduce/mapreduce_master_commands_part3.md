
06_MapReduce/mapreduce_master_commands_part3.md
```

---

# SECTION 71 — MAPREDUCE JOB COMPLETE EXECUTION PIPELINE

```text
Client → ResourceManager → NodeManager → ApplicationMaster
→ InputSplit → Mapper → Combiner → Partitioner
→ Shuffle → Sort → Reducer → OutputFormat → HDFS
```

⭐ Most important architecture flow (exam favorite)

---

# SECTION 72 — INPUT SPLIT CONCEPT

```text
InputSplit ≠ HDFS Block
```

Difference:

| Component  | Purpose                 |
| ---------- | ----------------------- |
| HDFS Block | Physical storage        |
| InputSplit | Logical processing unit |

Example:

```text
128MB block → 1 mapper usually
```

---

# SECTION 73 — NUMBER OF MAPPERS DECISION FORMULA

```text
Number of Mappers = InputSplit count
```

Controlled by:

```bash
-D mapreduce.input.fileinputformat.split.maxsize
```

---

# SECTION 74 — NUMBER OF REDUCERS DECISION FORMULA

Default formula:

```text
Reducers = 0.95 × (Nodes × ReduceSlots)
```

Manual override:

```bash
-D mapreduce.job.reduces=5
```

---

# SECTION 75 — MAPPER LIFECYCLE (STEP-BY-STEP)

Execution order:

```text
setup()
map()
cleanup()
```

Example flow:

```java
setup()
map(key,value)
cleanup()
```

---

# SECTION 76 — REDUCER LIFECYCLE (STEP-BY-STEP)

Execution order:

```text
setup()
reduce()
cleanup()
```

Reducer receives:

```text
(key, list(values))
```

---

# SECTION 77 — SHUFFLE PHASE (MOST IMPORTANT)

Shuffle means:

```text
Mapper output transfer → Reducer input
```

Includes:

* partitioning
* sorting
* merging
* copying

Exam trap:

```text
Shuffle + Sort happen together
```

---

# SECTION 78 — SORT PHASE INTERNAL WORKING

Sorting happens:

```text
Before reducer execution
```

Example:

```text
Mapper output:
dog 1
cat 1
dog 1

Reducer receives:
cat → [1]
dog → [1,1]
```

---

# SECTION 79 — PARTITIONER WORKING LOGIC

Default partition logic:

```java
hash(key) % number_of_reducers
```

Purpose:

```text
Decide reducer destination
```

---

# SECTION 80 — COMBINER ROLE

Combiner runs:

```text
Between Mapper and Reducer
```

Purpose:

```text
Reduce network traffic
```

Example:

Without combiner:

```text
dog 1
dog 1
dog 1
```

With combiner:

```text
dog 3
```

---

# SECTION 81 — WHEN COMBINER IS NOT SAFE

Combiner NOT used when:

```text
Operation is non-associative
```

Example:

```text
Average calculation ❌
```

Safe operations:

```text
Sum
Count
Min
Max
```

---

# SECTION 82 — MAP-ONLY JOB

Disable reducer:

```bash
-D mapreduce.job.reduces=0
```

Example usage:

```text
Filtering dataset
Format conversion
Data cleaning
```

---

# SECTION 83 — REDUCE-ONLY JOB (RARE CASE)

Mapper acts as identity mapper:

```text
Input → Reducer directly
```

Used for:

```text
Aggregation jobs
```

---

# SECTION 84 — APPLICATION MASTER ROLE

Responsibilities:

```text
Coordinate job execution
Track progress
Manage containers
Communicate with ResourceManager
```

Runs on:

```text
NodeManager container
```

---

# SECTION 85 — RESOURCE MANAGER ROLE

Responsibilities:

```text
Cluster resource allocation
Scheduler control
Application tracking
```

Single per cluster

---

# SECTION 86 — NODE MANAGER ROLE

Responsibilities:

```text
Launch containers
Monitor resource usage
Report status to RM
```

One per node

---

# SECTION 87 — CONTAINER CONCEPT

Container provides:

```text
CPU
RAM
Disk
Network
```

Used to execute:

```text
Mapper
Reducer
ApplicationMaster
```

---

# SECTION 88 — DATA LOCALITY TYPES

Types:

```text
Data Local
Rack Local
Off Rack
```

Priority order:

```text
Data Local > Rack Local > Off Rack
```

---

# SECTION 89 — SPECULATIVE EXECUTION PURPOSE

Used when:

```text
Slow node detected
```

Solution:

```text
Duplicate task launched elsewhere
```

Command:

```bash
-D mapreduce.map.speculative=true
```

---

# SECTION 90 — MAPREDUCE COUNTERS TYPES

Two categories:

### Built-in counters

```text
Map input records
Map output records
Reduce input records
Spilled records
```

### Custom counters

Example:

```java
context.getCounter("Group","InvalidRows").increment(1);
```

---

# SECTION 91 — SPILLED RECORDS MEANING

Spill happens when:

```text
Mapper buffer full
```

Data written to disk temporarily

Counter:

```text
Spilled Records
```

---

# SECTION 92 — BLOCK SIZE IMPACT ON MAPPERS

Example:

```text
File size = 256MB
Block size = 128MB
```

Result:

```text
2 mappers
```

---

# SECTION 93 — DISTRIBUTED CACHE PURPOSE

Used for:

```text
Lookup tables
Config files
Libraries
```

Command:

```bash
-D mapreduce.job.cache.files=hdfs:///config.txt
```

---

# SECTION 94 — JOB FAILURE COMMON REASONS

Typical causes:

```text
Output directory exists
Permission denied
Memory insufficient
ClassNotFoundException
Wrong input path
```

Most common:

```text
Output directory exists
```

Fix:

```bash
hdfs dfs -rm -r output
```

---

# SECTION 95 — MAPREDUCE MEMORY TUNING STRATEGY

Mapper memory:

```bash
-D mapreduce.map.memory.mb=2048
```

Reducer memory:

```bash
-D mapreduce.reduce.memory.mb=4096
```

---

# SECTION 96 — JVM REUSE PERFORMANCE BOOST

Enable reuse:

```bash
-D mapreduce.job.jvm.numtasks=-1
```

Benefit:

```text
Avoid JVM restart overhead
```

---

# SECTION 97 — SHUFFLE COMPRESSION BENEFIT

Enable compression:

```bash
-D mapreduce.map.output.compress=true
```

Improves:

```text
Network speed
Disk usage
Execution time
```

---

# SECTION 98 — MAPREDUCE JOB HISTORY SERVER ROLE

Access UI:

```text
http://localhost:19888
```

Shows:

```text
Completed jobs
Counters
Logs
Execution time
```

---

# SECTION 99 — YARN APPLICATION STATUS CHECK

Command:

```bash
yarn application -list
```

Details:

```text
RUNNING
FINISHED
FAILED
KILLED
```

---

# SECTION 100 — MOST IMPORTANT MAPREDUCE EXAM TRAPS 🎯

Remember these:

```text
Mapper count = InputSplit count
Reducer receives sorted keys
Shuffle + Sort simultaneous phase
Default partitioner = HashPartitioner
Combiner optional
Speculative execution handles slow nodes
Output directory must not exist
ApplicationMaster manages job lifecycle
