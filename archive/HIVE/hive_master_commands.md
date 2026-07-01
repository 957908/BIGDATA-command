Samajh gaya 👍

---

# 📁 HIVE ADVANCED COMMANDS — PART 2 (ENGINEER LEVEL)

## SECTION 131 — QUERY EXECUTION ENGINE SWITCHING

```sql
# Check current execution engine
set hive.execution.engine;

# Switch to MapReduce engine
set hive.execution.engine=mr;

# Switch to Tez engine
set hive.execution.engine=tez;

# Switch to Spark engine
set hive.execution.engine=spark;
```

⭐ Exam trap: Default modern clusters → **tez**

---

# SECTION 132 — CHECK QUERY STAGE BREAKDOWN

```sql
# View stage-level execution
explain select * from employee;

# Extended execution pipeline
explain extended select * from employee;
```

Used to identify Mapper + Reducer stages ⚙️

---

# SECTION 133 — FORCE NUMBER OF REDUCERS

```sql
# Manually define reducers
set mapreduce.job.reduces=5;
```

Useful when skewed dataset present

---

# SECTION 134 — AUTO REDUCER CALCULATION FORMULA

```sql
# Control reducer auto calculation threshold
set hive.exec.reducers.bytes.per.reducer=256000000;
```

Default ≈ 256MB per reducer

---

# SECTION 135 — MAXIMUM REDUCER LIMIT CONTROL

```sql
# Limit total reducers allowed
set hive.exec.reducers.max=64;
```

Prevents cluster overload 🚀

---

# SECTION 136 — ENABLE FETCH TASK OPTIMIZATION

```sql
# Convert simple queries to fetch-only execution
set hive.fetch.task.conversion=more;
```

Avoids MapReduce job execution

---

# SECTION 137 — ENABLE SMALL FILE MERGE (MAP OUTPUT)

```sql
# Merge mapper output files
set hive.merge.mapfiles=true;
```

Solves small file problem 📂

---

# SECTION 138 — ENABLE SMALL FILE MERGE (REDUCER OUTPUT)

```sql
# Merge reducer output files
set hive.merge.mapredfiles=true;
```

---

# SECTION 139 — CONTROL MERGE FILE SIZE

```sql
# Target merged file size
set hive.merge.size.per.task=256000000;
```

---

# SECTION 140 — ENABLE AUTO MAPJOIN THRESHOLD

```sql
# Auto convert join to MapJoin
set hive.auto.convert.join=true;
```

---

# SECTION 141 — MAPJOIN MEMORY LIMIT

```sql
# Define small table threshold
set hive.auto.convert.join.noconditionaltask.size=10000000;
```

Default ≈ 10MB

---

# SECTION 142 — FORCE MAPJOIN MANUALLY

```sql
select /*+ MAPJOIN(dept) */ *
from emp join dept
on emp.id=dept.id;
```

Small table broadcast join

---

# SECTION 143 — ENABLE BUCKET MAP JOIN

```sql
# Enable bucket join optimization
set hive.optimize.bucketmapjoin=true;
```

---

# SECTION 144 — ENABLE SORT MERGE BUCKET JOIN

```sql
# Enable SMB join optimization
set hive.optimize.bucketmapjoin.sortedmerge=true;
```

Used in large distributed joins

---

# SECTION 145 — ENABLE SKEW JOIN OPTIMIZATION

```sql
# Enable skew join handling
set hive.optimize.skewjoin=true;
```

Handles uneven key distribution

---

# SECTION 146 — ENABLE PARTITION PRUNING

```sql
# Skip unnecessary partitions
set hive.optimize.ppd=true;
```

Improves scan speed ⚡

---

# SECTION 147 — ENABLE COLUMN PRUNING

```sql
# Enable column pruning
set hive.optimize.cp=true;
```

Reads only required columns

---

# SECTION 148 — ENABLE COST BASED OPTIMIZER

```sql
# Enable cost-based optimizer
set hive.cbo.enable=true;
```

Optimizes join order automatically

---

# SECTION 149 — ENABLE STATISTICS FOR CBO

```sql
# Collect table statistics
analyze table employee compute statistics;

# Collect column statistics
analyze table employee compute statistics for columns;
```

Required for CBO accuracy

---

# SECTION 150 — ENABLE VECTORIZED EXECUTION

```sql
# Enable vectorized processing
set hive.vectorized.execution.enabled=true;
```

Processes batch column data faster 🚀

---

# SECTION 151 — ENABLE ORC OPTIMIZATION

```sql
# Enable predicate pushdown
set hive.optimize.ppd=true;

# Enable ORC indexing
set hive.optimize.index.filter=true;
```

---

# SECTION 152 — ENABLE PARQUET OPTIMIZATION

```sql
# Enable parquet predicate pushdown
set hive.optimize.index.filter=true;
```

---

# SECTION 153 — ENABLE LOCAL MODE EXECUTION

```sql
# Run small queries locally
set hive.exec.mode.local.auto=true;
```

---

# SECTION 154 — ENABLE PARALLEL QUERY EXECUTION

```sql
# Enable parallel execution
set hive.exec.parallel=true;
```

Runs independent stages simultaneously

---

# SECTION 155 — CONTROL SCRATCH DIRECTORY

```sql
# Show temp execution directory
set hive.exec.scratchdir;
```

Stores intermediate results

---

# SECTION 156 — CHECK WAREHOUSE DIRECTORY

```sql
# Show warehouse path
set hive.metastore.warehouse.dir;
```

Default:

```
/user/hive/warehouse
```

⭐ Exam favorite

---

# SECTION 157 — CHECK DEFAULT FILE FORMAT

```sql
# Show default storage format
set hive.default.fileformat;
```

Usually:

```
TEXTFILE
```

---

# SECTION 158 — ENABLE RESULT CACHE

```sql
# Enable query result caching
set hive.query.results.cache.enabled=true;
```

Speeds repeated queries

---

# SECTION 159 — ENABLE AUTHORIZATION SECURITY

```sql
# Enable SQL authorization
set hive.security.authorization.enabled=true;
```

---

# SECTION 160 — CHECK SESSION VARIABLES

```sql
# Show session variables
set;
```

Lists entire Hive config snapshot 📊

---

# SECTION 161 — ENABLE STRICT MODE VALIDATION

```sql
# Prevent cartesian joins
set hive.strict.checks.cartesian.product=true;
```

---

# SECTION 162 — ENABLE DYNAMIC PARTITIONING

```sql
set hive.exec.dynamic.partition=true;
```

---

# SECTION 163 — SWITCH TO NON-STRICT MODE

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
```

Default = strict ⭐

---

# SECTION 164 — CHECK HDFS REPLICATION FACTOR FROM HIVE

```sql
set dfs.replication;
```

Default:

```
3
```

---

# SECTION 165 — CHECK TEMP FILE LOCATION

```sql
set hive.exec.local.scratchdir;
```

---

# SECTION 166 — CHECK LOGGING FRAMEWORK

```
log4j
```

⭐ Direct MCQ question

---

# SECTION 167 — CHECK EXECUTION MODE

```sql
set hive.mapred.mode;
```

Values:

```
strict
nonstrict
```

---

# SECTION 168 — CHECK AUTO PARALLEL REDUCERS

```sql
set hive.exec.reducers.max;
```

---

# SECTION 169 — CHECK FETCH MODE SETTING

```sql
set hive.fetch.task.conversion;
```

---

# SECTION 170 — CHECK CURRENT DATABASE

```sql
set hive.current.database;
```

---

Agar tum chaho to next **Hive Super-Advanced Admin Part-3 (Metastore tuning + Tez configs + memory tuning + YARN integration + query failure debugging)** bhi bana deta hoon 🔧📊
