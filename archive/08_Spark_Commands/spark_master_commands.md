
```bash
08_Spark_Commands/spark_master_commands.md
```

---

# SECTION 1 — START SPARK SHELL

```bash
# Start Scala Spark shell
spark-shell
```

Default language:

```text
Scala
```

⭐ Exam trap

---

# SECTION 2 — START PYSPARK SHELL

```bash
# Start Python Spark shell
pyspark
```

Used for Python-based Spark programs

---

# SECTION 3 — START SPARK SQL CLI

```bash
# Start Spark SQL interface
spark-sql
```

Used to execute SQL queries directly

---

# SECTION 4 — CHECK SPARK VERSION

```bash
# Show Spark version
spark-submit --version
```

---

# SECTION 5 — CREATE SPARK SESSION

Scala:

```scala
val spark = SparkSession.builder.appName("TestApp").getOrCreate()
```

Python:

```python
spark = SparkSession.builder.appName("TestApp").getOrCreate()
```

⭐ Entry point of Spark application

---

# SECTION 6 — CREATE RDD (PARALLELIZE METHOD)

```scala
val data = sc.parallelize(List(1,2,3,4))
```

Creates distributed dataset

---

# SECTION 7 — CREATE RDD FROM FILE

```scala
val fileRDD = sc.textFile("data.txt")
```

Reads file into distributed memory

---

# SECTION 8 — DISPLAY RDD CONTENT

```scala
fileRDD.collect()
```

Collects data to driver node

⚠️ Avoid for large datasets

---

# SECTION 9 — COUNT RDD RECORDS

```scala
fileRDD.count()
```

Returns number of elements

---

# SECTION 10 — FIRST ELEMENT OF RDD

```scala
fileRDD.first()
```

Returns first record

---

# SECTION 11 — TAKE FIRST N RECORDS

```scala
fileRDD.take(5)
```

Returns first 5 elements

---

# SECTION 12 — APPLY MAP TRANSFORMATION

```scala
fileRDD.map(x => x.toUpperCase())
```

Transforms each element

---

# SECTION 13 — APPLY FILTER TRANSFORMATION

```scala
fileRDD.filter(x => x.contains("Spark"))
```

Filters dataset

---

# SECTION 14 — APPLY FLATMAP TRANSFORMATION

```scala
fileRDD.flatMap(line => line.split(" "))
```

Splits words into tokens

⭐ Used in WordCount

---

# SECTION 15 — DISTINCT VALUES FROM RDD

```scala
fileRDD.distinct()
```

Removes duplicates

---

# SECTION 16 — SORT RDD DATA

```scala
fileRDD.sortBy(x => x)
```

Sorts dataset

---

# SECTION 17 — UNION TWO RDDS

```scala
rdd1.union(rdd2)
```

Combines datasets

---

# SECTION 18 — INTERSECTION OF RDDS

```scala
rdd1.intersection(rdd2)
```

Common elements only

---

# SECTION 19 — CARTESIAN PRODUCT

```scala
rdd1.cartesian(rdd2)
```

Returns pair combinations

---

# SECTION 20 — SAVE RDD TO FILE

```scala
fileRDD.saveAsTextFile("output")
```

Writes output to HDFS/local

⚠️ Output directory must not exist

---

# SECTION 21 — CREATE DATAFRAME

```scala
val df = spark.read.json("data.json")
```

Loads structured dataset

---

# SECTION 22 — SHOW DATAFRAME CONTENT

```scala
df.show()
```

Displays table preview

---

# SECTION 23 — PRINT DATAFRAME SCHEMA

```scala
df.printSchema()
```

Shows structure of dataset

---

# SECTION 24 — SELECT COLUMN FROM DATAFRAME

```scala
df.select("name").show()
```

Select specific column

---

# SECTION 25 — FILTER DATAFRAME ROWS

```scala
df.filter(df("age") > 25).show()
```

Apply condition

---

# SECTION 26 — GROUP DATAFRAME

```scala
df.groupBy("age").count()
```

Aggregation operation

---

# SECTION 27 — ORDER DATAFRAME

```scala
df.orderBy("age").show()
```

Sort rows

---

# SECTION 28 — CREATE TEMP VIEW

```scala
df.createOrReplaceTempView("people")
```

Used for SQL queries

---

# SECTION 29 — RUN SQL QUERY ON DATAFRAME

```scala
spark.sql("SELECT * FROM people").show()
```

Executes Spark SQL

---

# SECTION 30 — READ CSV FILE

```scala
spark.read.csv("file.csv")
```

Loads CSV dataset

---

# SECTION 31 — READ PARQUET FILE

```scala
spark.read.parquet("file.parquet")
```

Columnar optimized format

---

# SECTION 32 — READ ORC FILE

```scala
spark.read.orc("file.orc")
```

Hive-compatible format

---

# SECTION 33 — WRITE DATAFRAME TO CSV

```scala
df.write.csv("output")
```

Exports dataset

---

# SECTION 34 — WRITE DATAFRAME TO PARQUET

```scala
df.write.parquet("output")
```

Efficient storage format

---

# SECTION 35 — WRITE DATAFRAME TO ORC

```scala
df.write.orc("output")
```

Compressed storage

---

# SECTION 36 — CHECK NUMBER OF PARTITIONS

```scala
fileRDD.getNumPartitions
```

Returns partition count

---

# SECTION 37 — CHANGE NUMBER OF PARTITIONS

```scala
fileRDD.repartition(4)
```

Increase partitions

---

# SECTION 38 — REDUCE NUMBER OF PARTITIONS

```scala
fileRDD.coalesce(2)
```

Decrease partitions efficiently

---

# SECTION 39 — CACHE RDD IN MEMORY

```scala
fileRDD.cache()
```

Stores dataset in RAM

⭐ Performance optimization

---

# SECTION 40 — PERSIST RDD

```scala
fileRDD.persist()
```

Stores dataset in memory/disk

---

# SECTION 41 — SPARK WEB UI

Open:

```text
http://localhost:4040
```

Shows:

* jobs
* stages
* tasks
* storage usage

⭐ Very important exam question

---

# SECTION 42 — SPARK SUBMIT COMMAND

```bash
spark-submit app.jar
```

Runs Spark application

Example:

```bash
spark-submit --class WordCount app.jar input output
```

---

# SECTION 43 — RUN SPARK APPLICATION IN LOCAL MODE

```bash
spark-submit --master local app.jar
```

Runs locally

---

# SECTION 44 — RUN SPARK APPLICATION ON YARN

```bash
spark-submit --master yarn app.jar
```

Runs on cluster

---

# SECTION 45 — RUN SPARK APPLICATION ON CLUSTER MODE

```bash
spark-submit --deploy-mode cluster app.jar
```

Driver runs inside cluster

---

# SECTION 46 — RUN SPARK APPLICATION CLIENT MODE

```bash
spark-submit --deploy-mode client app.jar
```

Driver runs locally

⭐ Exam trap difference

---

# SECTION 47 — CHECK DEFAULT PARALLELISM

```scala
sc.defaultParallelism
```

Returns default task parallelism

---

# SECTION 48 — CHECK SPARK CONTEXT INFO

```scala
sc.appName
```

Returns application name

---

# SECTION 49 — STOP SPARK SESSION

```scala
spark.stop()
```

Stops application safely

---

# SECTION 50 — MOST IMPORTANT SPARK EXAM TRAPS 🎯

Remember:

```text
spark-shell default language = Scala
Spark works in-memory
Spark faster than MapReduce
Spark entry point = SparkSession
RDD = Resilient Distributed Dataset
Spark UI port = 4040
Cluster mode → driver inside cluster
Client mode → driver outside cluster
```


* Partition tuning
* Performance optimization settings 🚀
