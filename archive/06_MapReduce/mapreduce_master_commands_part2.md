
```bash
06_MapReduce/mapreduce_master_commands_part2.md
```

---

# SECTION 41 — MAPREDUCE EXECUTION FLOW (PIPELINE UNDERSTANDING)

```bash
Input → InputSplit → Mapper → Combiner → Partitioner → Shuffle → Sort → Reducer → OutputFormat → HDFS
```

⭐ Exam favorite architecture pipeline

---

# SECTION 42 — INPUT SPLIT SIZE CONTROL

```bash
# Set minimum split size
-D mapreduce.input.fileinputformat.split.minsize=67108864

# Set maximum split size
-D mapreduce.input.fileinputformat.split.maxsize=134217728
```

Controls number of mapper tasks indirectly

---

# SECTION 43 — CUSTOM INPUT FORMAT CLASS

```bash
# Set custom input format
-D mapreduce.job.inputformat.class=TextInputFormat
```

Other types:

```
KeyValueTextInputFormat
SequenceFileInputFormat
NLineInputFormat
```

---

# SECTION 44 — CUSTOM OUTPUT FORMAT CLASS

```bash
# Set output format
-D mapreduce.job.outputformat.class=TextOutputFormat
```

---

# SECTION 45 — COMBINER CLASS CONFIGURATION

```java
job.setCombinerClass(MyReducer.class);
```

📌 Reduces shuffle traffic between mapper and reducer

---

# SECTION 46 — PARTITIONER CLASS CONFIGURATION

```java
job.setPartitionerClass(MyPartitioner.class);
```

Controls reducer assignment logic

Example:

```
key % number_of_reducers
```

---

# SECTION 47 — SHUFFLE PHASE CONTROL

```bash
# Enable shuffle compression
-D mapreduce.map.output.compress=true
```

Improves network transfer speed 🚀

---

# SECTION 48 — SORT BUFFER SIZE CONTROL

```bash
# Set sort buffer memory
-D mapreduce.task.io.sort.mb=256
```

Larger buffer → fewer disk spills

---

# SECTION 49 — SPILL PERCENTAGE CONTROL

```bash
# Define spill threshold
-D mapreduce.map.sort.spill.percent=0.80
```

Controls when mapper writes intermediate data to disk

---

# SECTION 50 — MAP OUTPUT COMPRESSION CODEC

```bash
# Set compression codec
-D mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec
```

Common codecs:

```
SnappyCodec
GzipCodec
LzoCodec
```

---

# SECTION 51 — REDUCE SHUFFLE PARALLEL COPIES

```bash
# Increase parallel shuffle fetch threads
-D mapreduce.reduce.shuffle.parallelcopies=10
```

Speeds reducer input transfer

---

# SECTION 52 — MAP TASK JVM REUSE

```bash
# Enable JVM reuse
-D mapreduce.job.jvm.numtasks=-1
```

Improves performance significantly

---

# SECTION 53 — SPECULATIVE EXECUTION ENABLE

```bash
# Enable speculative execution for mapper
-D mapreduce.map.speculative=true

# Enable speculative execution for reducer
-D mapreduce.reduce.speculative=true
```

Handles slow nodes automatically

---

# SECTION 54 — SPECULATIVE EXECUTION DISABLE

```bash
-D mapreduce.map.speculative=false
-D mapreduce.reduce.speculative=false
```

Used in debugging scenarios

---

# SECTION 55 — DISTRIBUTED CACHE FILE ADD

```bash
# Add file to distributed cache
-D mapreduce.job.cache.files=hdfs:///config.txt
```

Makes file available to all nodes

---

# SECTION 56 — DISTRIBUTED CACHE ARCHIVE ADD

```bash
# Add archive dependency
-D mapreduce.job.cache.archives=hdfs:///lib.zip
```

Used for libraries/scripts

---

# SECTION 57 — MAPREDUCE COUNTERS (BUILT-IN)

Example counters:

```
Map input records
Map output records
Reduce input records
Reduce output records
Spilled Records
```

Check via:

```bash
mapred job -history output_dir
```

---

# SECTION 58 — CUSTOM COUNTERS (JAVA)

```java
context.getCounter("MyCounterGroup","InvalidRecords").increment(1);
```

Tracks custom statistics inside job

---

# SECTION 59 — CHECK FAILED TASK ATTEMPTS

```bash
mapred job -list-attempt-ids job_123456 map
```

---

# SECTION 60 — FETCH FAILED TASK LOGS

```bash
mapred job -logs job_123456
```

Useful for debugging errors

---

# SECTION 61 — CHECK APPLICATION MASTER STATUS

```bash
yarn application -status application_123456
```

Shows execution progress

---

# SECTION 62 — VIEW CONTAINER LOGS

```bash
yarn logs -applicationId application_123456
```

Important debugging command ⭐

---

# SECTION 63 — ENABLE MAP MEMORY SETTINGS

```bash
-D mapreduce.map.memory.mb=2048
```

---

# SECTION 64 — ENABLE REDUCE MEMORY SETTINGS

```bash
-D mapreduce.reduce.memory.mb=4096
```

---

# SECTION 65 — SET MAP JAVA HEAP SIZE

```bash
-D mapreduce.map.java.opts=-Xmx1500m
```

---

# SECTION 66 — SET REDUCE JAVA HEAP SIZE

```bash
-D mapreduce.reduce.java.opts=-Xmx3000m
```

---

# SECTION 67 — CONTROL TASK TIMEOUT

```bash
-D mapreduce.task.timeout=600000
```

Default = 10 minutes

---

# SECTION 68 — ENABLE OUTPUT FILE COMPRESSION

```bash
-D mapreduce.output.fileoutputformat.compress=true
```

---

# SECTION 69 — OUTPUT COMPRESSION CODEC

```bash
-D mapreduce.output.fileoutputformat.compress.codec=SnappyCodec
```

---

# SECTION 70 — CHECK JOB HISTORY SERVER

```
http://localhost:19888
```

Shows completed job details

---
---

Next I can generate **MapReduce Part 3 (Mapper lifecycle + Reducer lifecycle + Shuffle-Sort deep theory + CDAC exam traps)** which is the most asked conceptual section.
