# 🐝 Module 05: Apache Hive Mastery

Apache Hive is a distributed data warehousing system built on top of Apache Hadoop. It provides data summarization, ad-hoc querying, and analysis of large datasets stored in HDFS using HiveQL, a SQL-like dialect. Hive translates SQL queries into directed acyclic graphs (DAGs) executed on distributed compute frameworks (Tez, Spark, or MapReduce).

---

## 1. Managed (Internal) vs. External Tables

The primary difference lies in data ownership and safety during deletions.

```sql
-- Create Managed Table
CREATE TABLE employee_managed (
    id INT,
    name STRING,
    salary DOUBLE
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

-- Create External Table pointing to pre-existing HDFS path
CREATE EXTERNAL TABLE employee_external (
    id INT,
    name STRING,
    salary DOUBLE
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/data/raw_employees';
```

| Feature | Managed (Internal) Table | External Table |
| :--- | :--- | :--- |
| **Data Ownership** | Hive manages both schema metadata and raw data. | Hive only manages metadata; data files are independent. |
| **Default Location** | Stored in HDFS `/user/hive/warehouse/` directory. | Stored at the user-defined HDFS location. |
| **Drop Behavior** | Dropping table deletes **both** metadata and HDFS data files. | Dropping table deletes **only** metadata; raw files remain in HDFS. |
| **Use Case** | Temporary/sandbox tables where Hive manages the lifecycle. | Raw ingestion directories, shared tables used by Spark/Presto. |

---

## 2. Partitioning vs. Bucketing (Clustering)

Proper physical table design is the most critical factor for Hive performance.

```text
HDFS Directory Layout for Partitioned + Bucketed Table:

/user/hive/warehouse/sales_db.db/orders/            <-- Table Directory
├── year=2023/month=10/                             <-- Partitions (Logical Sub-dirs)
│   ├── 000000_0                                    <-- Bucket 0 File (Hash-based Split)
│   └── 000001_0                                    <-- Bucket 1 File
└── year=2023/month=11/
    ├── 000000_0
    └── 000001_0
```

### Partitioning:
* **Concept**: Creates physical subdirectories in HDFS based on the values of the partitioning columns (e.g., `/orders/year=2023/month=10/`).
* **Benefit**: Enables **Partition Pruning**. A query with `WHERE year=2023 AND month=10` instructs Hive to read only that specific HDFS subdirectory, completely skipping files in other directories.
* **Anti-Pattern**: Do not partition on high-cardinality fields (e.g., `user_id` or `timestamp`). This creates millions of directories containing tiny files, bloating NameNode metadata memory and hurting MapReduce/Spark performance.

### Bucketing:
* **Concept**: Partitions files within a directory based on a hash of a bucketing column (e.g., `hash(user_id) % num_buckets`). Data is split into a fixed number of files (buckets).
* **Benefit**:
  1. Prevents small-files problems by forcing a fixed number of files.
  2. Optimizes join performance by enabling **Bucket Map Joins** and **Sorted Merge Bucket (SMB) Joins** (which avoid sorting/shuffling large datasets).
  3. Optimizes sampling operations.

### DDL Example:
```sql
CREATE TABLE orders_optimized (
    order_id BIGINT,
    customer_id BIGINT,
    order_date TIMESTAMP,
    amount DOUBLE
)
PARTITIONED BY (year INT, month INT)
CLUSTERED BY (customer_id) INTO 16 BUCKETS
STORED AS ORC;
```

---

## 3. Columnar File Formats & Compression

For master-level querying, never use plain text. Store data using columnar file formats and compression.

### Row-Oriented (TEXTFILE) vs. Columnar (ORC / Parquet):
* **Row-Oriented**: Stores entire rows consecutively. Good for writing data rapidly, but requires scanning the entire file even if only selecting two columns.
* **Columnar (ORC/Parquet)**: Groups data by columns.
  * **Projection Pruning**: Reads only the columns specified in the `SELECT` list.
  * **Index skipping**: ORC files store metadata (min, max, sum, count) for every group of 10,000 rows (Row Index) and at the file level (Stripe Index). If querying `WHERE order_id = 999999` and a Stripe's min/max does not contain it, the entire Stripe is skipped without reading it.

### Compression Codecs:
* **Snappy**: Low CPU usage, extremely fast compression/decompression, but not splittable by default unless wrapped in container file formats like ORC or Parquet.
* **Gzip**: High compression ratio, but NOT splittable. Avoid using plain Gzip on text files in HDFS, as a single map task will have to process the entire un-splittable file.
* **Zlib**: High compression ratio, default for ORC format.

```sql
-- DDL for ORC table with Snappy compression
CREATE TABLE web_logs (
    ip STRING,
    url STRING,
    response_code INT
)
STORED AS ORC
TBLPROPERTIES ("orc.compress"="SNAPPY");
```

---

## 4. Advanced Window Functions

Window functions are essential for complex aggregations without self-joining tables.

```sql
-- Sample Query: Rank employees by salary within their departments and calculate running totals
SELECT 
    dept_id,
    emp_name,
    salary,
    -- Rank without gaps (1, 2, 2, 3)
    DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as salary_rank,
    -- Lag: Fetch previous row's salary in the same department
    LAG(salary, 1, 0.0) OVER (PARTITION BY dept_id ORDER BY salary DESC) as prev_higher_salary,
    -- Cumulative running total of salaries inside the department
    SUM(salary) OVER (
        PARTITION BY dept_id 
        ORDER BY salary DESC 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_dept_total
FROM employees;
```

---

## 5. ACID & Transactional Tables (v2.x/3.x)

Modern Hive supports ACID (Atomicity, Consistency, Isolation, Durability) transactions over HDFS using write-ahead Delta files and transaction managers.

### Key Preconditions:
1. File format must be **ORC**.
2. Table must be **Bucketed**.
3. Table property must set `transactional = true`.

### Configurations (`hive-site.xml`):
```xml
<property>
    <name>hive.support.concurrency</name>
    <value>true</value>
</property>
<property>
    <name>hive.txn.manager</name>
    <value>org.apache.hadoop.hive.ql.lockmgr.DbTxnManager</value>
</property>
<property>
    <name>hive.compactor.initiator.on</name>
    <value>true</value>
</property>
<property>
    <name>hive.compactor.worker.threads</name>
    <value>2</value>
</property>
```

---

## 6. Complete Hive SQL & CLI Reference Library

This section covers the exhaustive DDL/DML queries and performance variables.

### A. Data Definition Language (DDL)
* **`CREATE TABLE`**:
  ```sql
  -- Managed Table with ORC Format
  CREATE TABLE customers (id INT, name STRING) STORED AS ORC;
  
  -- Partitioned External Table
  CREATE EXTERNAL TABLE logs (ip STRING, status INT) 
  PARTITIONED BY (dt STRING) 
  STORED AS PARQUET 
  LOCATION '/data/logs/';
  ```
* **`ALTER TABLE`**:
  ```sql
  -- Add a partition manually
  ALTER TABLE logs ADD PARTITION (dt='2026-07-01');
  
  -- Change partition location path
  ALTER TABLE logs PARTITION (dt='2026-07-01') SET LOCATION '/data/new_logs/2026-07-01';
  
  -- Replace columns schema definition
  ALTER TABLE customers REPLACE COLUMNS (id INT, full_name STRING, country STRING);
  ```
* **`DESCRIBE`**:
  ```sql
  DESCRIBE customers;            -- Simple schema view
  DESCRIBE FORMATTED customers;  -- Exhaustive metadata view (type, location, statistics, compression properties)
  ```

### B. Data Manipulation Language (DML)
* **`LOAD DATA`**:
  ```sql
  -- Load a local file to HDFS Hive warehouse (moves local file)
  LOAD DATA INPATH '/tmp/local_data.csv' INTO TABLE customers;
  
  -- Load file directly into a specific partition
  LOAD DATA INPATH '/tmp/logs.txt' INTO TABLE logs PARTITION (dt='2026-07-01');
  ```
* **`INSERT`**:
  ```sql
  -- Appends data
  INSERT INTO TABLE customers SELECT id, name FROM temp_table;
  
  -- Overwrites entire table/partition
  INSERT OVERWRITE TABLE logs PARTITION(dt='2026-07-01') SELECT ip, status FROM source_table;
  
  -- Write query output directly to HDFS directory as text files
  INSERT OVERWRITE DIRECTORY '/tmp/output_data' 
  ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
  SELECT * FROM customers;
  ```

### C. Metastore Synchronization & Repairs
* **`MSCK REPAIR TABLE`**: Syncs HDFS partition directories with the metastore.
  ```sql
  MSCK REPAIR TABLE logs;
  ```

### D. ACID Compaction Commands
* **`COMPACT`**: Merge delta folders.
  ```sql
  ALTER TABLE customer_accounts COMPACT 'major'; -- Complete merge of base and delta files
  ALTER TABLE customer_accounts COMPACT 'minor'; -- Merge delta files into one delta file
  ```
* **`SHOW`**:
  ```sql
  SHOW COMPACTIONS;  -- Inspect compaction status (INITIATED, RUNNING, READY, FAILED)
  SHOW TRANSACTIONS;  -- Inspect active locks and transactional IDs
  ```

### E. Hive Performance Optimization Parameters
```sql
-- Switch execution engine from MapReduce to Tez
set hive.execution.engine=tez;

-- Enable CPU optimization via Vectorization
set hive.vectorized.execution.enabled=true;
set hive.vectorized.execution.reduce.enabled=true;

-- Enable Cost-Based Optimizer (CBO)
set hive.cbo.enable=true;

-- Enable automatic conversion of joins to Map Joins
set hive.auto.convert.join=true;
set hive.auto.convert.join.noconditionaltask.size=26214400; -- 25 MB in bytes

-- Enable Sort-Merge Bucket (SMB) joins
set hive.optimize.bucketmapjoin=true;
set hive.optimize.bucketmapjoin.sortedmerge=true;

-- Enable dynamic partitioning inserts
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict; -- Allows inserting without specifying any static partition
```

---

## 7. Enterprise Job Interview Q&A (Apache Hive)

This section prepares you for production-level interview questions.

### Q1: What is the `MSCK REPAIR TABLE` command, and why is it necessary? What occurs under the hood?
* **How to explain this to the interviewer**:
  Explain that Hive has a separation of concerns: data resides in HDFS, but schemas/directories reside in the Metastore database (RDBMS). Explain that copying files bypasses the Metastore, and MSCK repairs this link.

* **Model Answer**:
  "The `MSCK REPAIR TABLE` (Metastore Check Table) command synchronizes the Hive Metastore metadata with the actual directories present in HDFS.
  
  When an external tool (like a Spark job or an HDFS CLI command) creates a new partition directory in HDFS (e.g., `/user/hive/warehouse/logs/dt=2026-07-02/`) and uploads files, the Hive Metastore database remains completely unaware of this folder. Consequently, running `SELECT * FROM logs WHERE dt='2026-07-02'` will return zero records.
  
  When `MSCK REPAIR TABLE logs` is executed:
  1. Hive queries the NameNode for the directory structure under the table's location.
  2. It parses the directory names looking for partition keys (e.g., `dt=value`).
  3. It compares these HDFS directories against the partition records in the Metastore RDBMS.
  4. It registers any missing partition directories into the Metastore, making the data queryable immediately."

---

### Q2: Compare Partitioning vs. Bucketing. What are the performance hazards of over-partitioning, and how does bucketing optimize joins?
* **How to explain this to the interviewer**:
  Define both clearly, contrast them, and focus on the NameNode memory bloat caused by over-partitioning. Then show how bucketing enables the Sort-Merge Bucket (SMB) Join.

* **Model Answer**:
  "**Partitioning** is the division of data into physical subdirectories based on specific columns (e.g., Date or Region). It is designed to speed up queries via partition pruning. **Bucketing** is the division of files *within* directories based on a hash of a column (e.g., UserID) modulo a fixed number of buckets.
  
  **Over-Partitioning Hazard**:
  If a table is partitioned on a high-cardinality column like `timestamp`, Hive will create a separate HDFS directory for every unique second. This causes:
  1. Severe NameNode memory bloat, as each directory and file takes ~150 bytes in NameNode JVM heap.
  2. High MapReduce/Spark overhead, as each small file spawns a separate task, wasting cluster resource scheduling time.
  
  **Bucketing Join Optimization**:
  If we join two tables bucketed on the same join key (e.g., `user_id`) with compatible bucket counts (e.g. 16 and 32), Hive can perform a **Bucket Map Join** or a **Sorted Merge Bucket (SMB) Join**. Instead of shuffling both tables across the network, the engine loads corresponding bucket files into memory (Bucket 1 of Table A joined to Bucket 1 of Table B) and performs a local merge-join, completely bypassing the network shuffle."

---

### Q3: Explain how ACID Transactions are implemented in Apache Hive on top of HDFS, which is append-only.
* **How to explain this to the interviewer**:
  Explain how Hive achieves updates and deletes on HDFS by writing delta directories (using transaction IDs) instead of updating files in place, and explain the compaction daemon's role in merging these delta files.

* **Model Answer**:
  "Since HDFS is an append-only file system, Hive cannot modify blocks in place. Instead, Hive achieves ACID functionality through **Write-Ahead Delta Files**.
  
  1. **Insert Path**: Hive writes the records into a new directory named `delta_xxxx_xxxx` (containing transaction IDs) as ORC files.
  2. **Update Path**: Since blocks are immutable, Hive performs a 'Delete-then-Insert'. It writes the deleted row keys into a `delete_delta_xxxx_xxxx` directory, and writes the new updated row values into a standard `delta` directory.
  3. **Read Path**: When a client queries the table, Hive's transactional reader reads the base table files, overlays all active `delta` and `delete_delta` directories, filters out deleted keys, and returns the net active records.
  
  **Compaction**:
  To prevent HDFS from filling up with thousands of delta files:
  * **Minor Compaction**: A background daemon merges multiple small `delta` directories into a single unified delta directory.
  * **Major Compaction**: Merges all `delta` and `delete_delta` files into the base ORC data file, permanently removing deleted records."
