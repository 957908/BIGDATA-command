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

## 5. Hive Performance Optimization Techniques

* **Tez Execution Engine**:
  * Set `set hive.execution.engine=tez;`. Tez executes jobs using a directed acyclic graph (DAG) and passes data in-memory between tasks, avoiding writing intermediate data to HDFS like legacy MapReduce.
* **Vectorization**:
  * Set `set hive.vectorized.execution.enabled=true;` and `set hive.vectorized.execution.reduce.enabled=true;`. Vectorization allows the execution engine to process a batch of 1024 rows together instead of single rows. This improves CPU cache utilization and reduces instruction pipelines.
* **Cost-Based Optimizer (CBO)**:
  * Uses database statistics (row count, histograms) to optimize queries (e.g., reordering joins, choosing physical algorithms).
  * Run stats calculation:
    ```sql
    ANALYZE TABLE orders_optimized PARTITION(year, month) COMPUTE STATISTICS;
    ```
* **Join Optimizations**:
  * **Map Join**: If joining a large table and a small table (default threshold: `<25MB` defined by `hive.auto.convert.join.noconditionaltask.size`), the small table is loaded into memory (DistributedCache) on all map nodes. The join is executed entirely inside the map phase without a shuffle stage.
  * **SMB (Sort Merge Bucket) Join**: If both tables are bucketed on the same join key, sorted, and have matching/compatible bucket counts, Hive performs a Sorted Merge Bucket Join. It reads buckets sequentially and merges them, bypassing sorting and shuffling completely.

---

## 6. ACID & Transactional Tables (v2.x/3.x)

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

### DDL and Operation:
```sql
CREATE TABLE customer_accounts (
    account_id INT,
    balance DOUBLE
)
CLUSTERED BY (account_id) INTO 4 BUCKETS
STORED AS ORC
TBLPROPERTIES ('transactional'='true');

-- Supports standard transactional queries
INSERT INTO customer_accounts VALUES (101, 5000.0);
UPDATE customer_accounts SET balance = 5500.0 WHERE account_id = 101;
DELETE FROM customer_accounts WHERE account_id = 101;
```

---

## 🎯 Exam and Interview Traps

1. **Trap: Why does a query with a `WHERE` condition on a partitioned column run extremely slowly on a newly loaded table?**
   * **Answer**: If the partition data was copied directly to HDFS (e.g., using `hdfs dfs -put`), Hive's metastore is unaware of the new directory paths. Hive will scan the entire root table directory. Run the metastore repair command to sync HDFS partition paths with the Metastore:
     ```sql
     MSCK REPAIR TABLE table_name;
     ```

2. **Trap: What is the difference between Partitioning and Bucketing, and how do you choose?**
   * **Answer**: Partitioning is logical directory splitting; it should be used on low-cardinality columns (e.g., Date, Region, Department) where you filter queries directly. Bucketing is hash-based file splitting; it should be used on high-cardinality columns (e.g., UserID, TransactionID) to prevent folder bloat, optimize join algorithms, and enforce file size caps.

3. **Trap: Why are there thousands of tiny files in my ACID transactional table directories, and how do we resolve this?**
   * **Answer**: Every `INSERT`, `UPDATE`, or `DELETE` statement in Hive ACID writes a new `delta` directory in HDFS containing small incremental files. To resolve this, Hive running compactors must merge these files. Check transaction compaction status:
     ```sql
     SHOW COMPACTIONS;
     ```
     If stalled, trigger manual compaction:
     ```sql
     ALTER TABLE customer_accounts COMPACT 'major';
     ```
