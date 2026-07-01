

# SECTION 1 — MAPREDUCE INTRO EXECUTION COMMANDS

```bash
# Run example MapReduce WordCount job
hadoop jar hadoop-mapreduce-examples.jar wordcount input output
```

📌 Most common Hadoop exam command

---

# SECTION 2 — CHECK AVAILABLE EXAMPLE JOBS

```bash
# List all built-in MapReduce example programs
hadoop jar hadoop-mapreduce-examples.jar
```

Common outputs:

```
wordcount
pi
grep
randomwriter
teragen
terasort
teravalidate
```

---

# SECTION 3 — WORDCOUNT JOB EXECUTION (STEP-BY-STEP)

```bash
# Upload file to HDFS
hdfs dfs -put data.txt /input

# Run wordcount job
hadoop jar hadoop-mapreduce-examples.jar wordcount /input /output

# View result
hdfs dfs -cat /output/part-r-00000
```

---

# SECTION 4 — REMOVE OLD OUTPUT DIRECTORY (VERY IMPORTANT)

```bash
# Remove previous output directory before rerun
hdfs dfs -rm -r /output
```

⭐ Exam trap: MapReduce job fails if output folder exists

---

# SECTION 5 — RUN GREP MAPREDUCE JOB

```bash
# Search pattern inside dataset
hadoop jar hadoop-mapreduce-examples.jar grep input output 'error'
```

---

# SECTION 6 — RUN PI ESTIMATION JOB

```bash
# Run Pi calculation example
hadoop jar hadoop-mapreduce-examples.jar pi 2 5
```

Format:

```
pi <maps> <samples>
```

---

# SECTION 7 — RANDOM DATA GENERATION JOB

```bash
# Generate random dataset
hadoop jar hadoop-mapreduce-examples.jar randomwriter output
```

Used for testing cluster performance

---

# SECTION 8 — TERAGEN BENCHMARK JOB

```bash
# Generate large dataset
hadoop jar hadoop-mapreduce-examples.jar teragen 100000 output
```

Creates benchmarking dataset

---

# SECTION 9 — TERASORT JOB

```bash
# Sort generated dataset
hadoop jar hadoop-mapreduce-examples.jar terasort input output
```

Cluster performance benchmark

---

# SECTION 10 — TERAVALIDATE JOB

```bash
# Validate sorted dataset
hadoop jar hadoop-mapreduce-examples.jar teravalidate input output
```

---

# SECTION 11 — CUSTOM MAPREDUCE PROGRAM EXECUTION

```bash
# Run custom MapReduce jar
hadoop jar myprogram.jar DriverClass input output
```

Format:

```
hadoop jar <jarfile> <driverclass> <input> <output>
```

---

# SECTION 12 — CHECK RUNNING MAPREDUCE JOBS

```bash
# List running jobs
mapred job -list
```

---

# SECTION 13 — CHECK JOB STATUS

```bash
# Show job status
mapred job -status job_123456789
```

---

# SECTION 14 — KILL RUNNING JOB

```bash
# Kill running job
mapred job -kill job_123456789
```

Useful when job stuck 🚨

---

# SECTION 15 — VIEW JOB COUNTERS

```bash
# Display job counters
mapred job -history output_dir
```

Counters include:

```
Map input records
Map output records
Reduce input records
Reduce output records
```

---

# SECTION 16 — VIEW JOB LOGS

```bash
# View MapReduce job logs
yarn logs -applicationId application_123456
```

Used for debugging failures

---

# SECTION 17 — SET NUMBER OF MAPPERS

```bash
# Define number of mapper tasks
-D mapreduce.job.maps=4
```

Example:

```bash
hadoop jar job.jar Driver \
-D mapreduce.job.maps=4 input output
```

---

# SECTION 18 — SET NUMBER OF REDUCERS

```bash
# Define reducers manually
-D mapreduce.job.reduces=2
```

Example:

```bash
hadoop jar job.jar Driver \
-D mapreduce.job.reduces=2 input output
```

---

# SECTION 19 — DISABLE REDUCER (MAP-ONLY JOB)

```bash
# Run mapper-only job
-D mapreduce.job.reduces=0
```

---

# SECTION 20 — VIEW JOB TRACKER WEB UI

```
http://localhost:8088
```

Shows:

* running jobs
* completed jobs
* failed jobs
* counters
* logs

---

# SECTION 21 — CHECK CLUSTER NODES

```bash
# Show active cluster nodes
yarn node -list
```

---

# SECTION 22 — LIST APPLICATIONS IN YARN

```bash
# List running applications
yarn application -list
```

---

# SECTION 23 — CHECK APPLICATION STATUS

```bash
# Show application status
yarn application -status application_123456
```

---

# SECTION 24 — KILL APPLICATION

```bash
# Kill running application
yarn application -kill application_123456
```

---

# SECTION 25 — VIEW JOB HISTORY SERVER

```
http://localhost:19888
```

Shows completed MapReduce jobs history

---

# SECTION 26 — SET INPUT FORMAT CLASS

```bash
# Define input format
-D mapreduce.job.inputformat.class=TextInputFormat
```

Other formats:

```
KeyValueTextInputFormat
SequenceFileInputFormat
```

---

# SECTION 27 — SET OUTPUT FORMAT CLASS

```bash
# Define output format
-D mapreduce.job.outputformat.class=TextOutputFormat
```

---

# SECTION 28 — SET COMBINER CLASS

```bash
# Define combiner class
job.setCombinerClass(MyReducer.class);
```

Improves performance ⚡

---

# SECTION 29 — SET PARTITIONER CLASS

```bash
# Define custom partitioner
job.setPartitionerClass(MyPartitioner.class);
```

Controls reducer distribution

---

# SECTION 30 — ENABLE COMPRESSION OUTPUT

```bash
# Enable output compression
-D mapreduce.output.fileoutputformat.compress=true
```

---

# SECTION 31 — ENABLE MAP OUTPUT COMPRESSION

```bash
# Compress intermediate output
-D mapreduce.map.output.compress=true
```

Improves shuffle performance 🚀

---

# SECTION 32 — SET BLOCK SIZE FOR JOB

```bash
# Define input split size
-D mapreduce.input.fileinputformat.split.maxsize=134217728
```

Controls mapper count indirectly

---

# SECTION 33 — SET MEMORY FOR MAP TASK

```bash
# Allocate mapper memory
-D mapreduce.map.memory.mb=1024
```

---

# SECTION 34 — SET MEMORY FOR REDUCE TASK

```bash
# Allocate reducer memory
-D mapreduce.reduce.memory.mb=2048
```

---

# SECTION 35 — VIEW FAILED TASK DETAILS

```bash
# View failed attempts
mapred job -history job_output_dir
```

---

# SECTION 36 — CHECK JOB PRIORITY

```bash
# Show job priority
mapred job -list
```

---

# SECTION 37 — CHANGE JOB PRIORITY

```bash
# Set job priority
mapred job -set-priority job_123456 HIGH
```

Levels:

```
VERY_HIGH
HIGH
NORMAL
LOW
VERY_LOW
```

---

# SECTION 38 — CHECK MAP TASK REPORT

```bash
# View map task report
mapred job -list-attempt-ids job_123456 map
```

---

# SECTION 39 — CHECK REDUCE TASK REPORT

```bash
# View reduce task report
mapred job -list-attempt-ids job_123456 reduce
```

---

# SECTION 40 — VIEW TASK LOGS

```bash
# Fetch task logs
mapred job -logs job_123456
```

---

* Performance tuning parameters

Jo tumhara **MapReduce Master Handbook (CDAC exam level)** complete karega 📘🚀
