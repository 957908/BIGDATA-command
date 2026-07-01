

# 📁 HIVE ADMIN COMMANDS (MASTER SECTION)

## SECTION 101 — HIVE SERVER CONTROL COMMANDS

```sql
# Start HiveServer2 service
hiveserver2

# Check HiveServer2 running status
ps -ef | grep hiveserver2

# Restart HiveServer2
hiveserver2 --hiveconf hive.server2.restart=true

# Connect HiveServer2 using Beeline
beeline -u jdbc:hive2://localhost:10000/default
```

📌 Used in production clusters instead of Hive CLI

---

# SECTION 102 — METASTORE SERVICE MANAGEMENT

```sql
# Start Hive metastore service
hive --service metastore

# Check metastore process
ps -ef | grep metastore

# Restart metastore service
hive --service metastore &
```

📌 Required for multi-user Hive access

---

# SECTION 103 — METASTORE DATABASE VALIDATION

```sql
# Show metastore DB connection URL
set javax.jdo.option.ConnectionURL;

# Show metastore driver
set javax.jdo.option.ConnectionDriverName;

# Show metastore username
set javax.jdo.option.ConnectionUserName;
```

📌 Confirms Derby vs MySQL metastore

---

# SECTION 104 — HIVE WAREHOUSE DIRECTORY MANAGEMENT

```sql
# Show warehouse directory location
set hive.metastore.warehouse.dir;
```

Default:

```
/user/hive/warehouse
```

⭐ Exam trap

---

# SECTION 105 — CHECK EXECUTION ENGINE

```sql
# Show execution engine
set hive.execution.engine;
```

Possible values:

```
mr
tez
spark
```

---

# SECTION 106 — CHECK DEFAULT FILE FORMAT

```sql
# Show default file format
set hive.default.fileformat;
```

Common default:

```
TEXTFILE
```

---

# SECTION 107 — ENABLE COST BASED OPTIMIZER

```sql
# Enable CBO optimizer
set hive.cbo.enable=true;
```

📌 Improves join order automatically

---

# SECTION 108 — ENABLE PARALLEL EXECUTION

```sql
# Enable parallel execution
set hive.exec.parallel=true;
```

Speeds up multi-stage queries 🚀

---

# SECTION 109 — ENABLE LOCAL MODE EXECUTION

```sql
# Enable local execution for small datasets
set hive.exec.mode.local.auto=true;
```

---

# SECTION 110 — ENABLE DYNAMIC PARTITIONING

```sql
# Enable dynamic partitioning
set hive.exec.dynamic.partition=true;

# Enable non-strict mode
set hive.exec.dynamic.partition.mode=nonstrict;
```

⭐ Exam trap: default = strict

---

# SECTION 111 — CHECK REPLICATION FACTOR (HDFS THROUGH HIVE)

```sql
# Show replication factor
set dfs.replication;
```

Default:

```
3
```

---

# SECTION 112 — ENABLE QUERY RESULT CACHE

```sql
# Enable caching
set hive.query.results.cache.enabled=true;
```

---

# SECTION 113 — REDUCER AUTO TUNING

```sql
# Set reducer size threshold
set hive.exec.reducers.bytes.per.reducer=256000000;

# Set max reducers
set hive.exec.reducers.max=64;
```

---

# SECTION 114 — ENABLE VECTORIZED EXECUTION

```sql
# Enable vectorized execution
set hive.vectorized.execution.enabled=true;
```

Column batch processing optimization ⚡

---

# SECTION 115 — ENABLE PARTITION PRUNING

```sql
# Enable partition pruning
set hive.optimize.ppd=true;
```

Skips unnecessary partitions

---

# SECTION 116 — ENABLE MAPJOIN AUTO CONVERSION

```sql
# Enable automatic MapJoin
set hive.auto.convert.join=true;
```

Small table joins faster

---

# SECTION 117 — ENABLE SKEW JOIN HANDLING

```sql
# Enable skew join optimization
set hive.optimize.skewjoin=true;
```

Handles uneven key distribution

---

# SECTION 118 — ENABLE BUCKET ENFORCEMENT

```sql
# Enforce bucketing
set hive.enforce.bucketing=true;
```

---

# SECTION 119 — ENABLE SORT ENFORCEMENT

```sql
# Enforce sorting
set hive.enforce.sorting=true;
```

---

# SECTION 120 — STRICT MODE ENABLE/DISABLE

```sql
# Enable strict mode
set hive.mapred.mode=strict;

# Disable strict mode
set hive.mapred.mode=nonstrict;
```

Strict mode prevents risky queries

---

# SECTION 121 — ENABLE AUTHORIZATION

```sql
# Enable SQL authorization
set hive.security.authorization.enabled=true;
```

Cluster security control 🔐

---

# SECTION 122 — ROLE MANAGEMENT (ADMIN LEVEL)

```sql
# Create role
create role admin_role;

# Assign role to user
grant role admin_role to user user1;

# Show roles
show roles;

# Drop role
drop role admin_role;
```

---

# SECTION 123 — PERMISSION CONTROL

```sql
# Grant select permission
grant select on table employee to user user1;

# Grant all permissions
grant all on table employee to user user1;
```

---

# SECTION 124 — REVOKE ACCESS CONTROL

```sql
# Revoke select permission
revoke select on table employee from user user1;
```

---

# SECTION 125 — LOCK MANAGEMENT (TRANSACTION TABLES)

```sql
# Show locks
show locks acid_table;

# Unlock table
unlock table acid_table;
```

Used in ACID tables

---

# SECTION 126 — COMPACTION MANAGEMENT

```sql
# Trigger minor compaction
alter table acid_table compact 'minor';

# Trigger major compaction
alter table acid_table compact 'major';

# Check compaction status
show compactions;
```

---

# SECTION 127 — METADATA REPAIR (VERY IMPORTANT)

```sql
# Repair partition metadata
msck repair table sales;
```

Used when partitions added manually in HDFS

---

# SECTION 128 — EXPLAIN DEPENDENCY GRAPH

```sql
# Show execution dependency graph
explain dependency
select * from employee;
```

---

# SECTION 129 — SHOW CONFIGURATION VALUES

```sql
# Show all configuration values
set;

# Show specific config
set hive.exec.parallel;
```

---

# SECTION 130 — CHECK TEMP DIRECTORY LOCATION

```sql
# Show temp scratch directory
set hive.exec.scratchdir;
```

---

Agar tum chaho to next main **Hive Troubleshooting Admin Commands (job fail fix, reducer stuck fix, metastore error fix, permission error fix)** bhi add kar deta hoon — jo real cluster me use hote hain 🔧
