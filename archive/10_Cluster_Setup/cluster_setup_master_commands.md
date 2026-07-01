
# 1️⃣ CLUSTER SETUP MASTER COMMANDS

📁 Suggested file:

```bash
10_Cluster_Setup/cluster_setup_master_commands.md
```

## SECTION 1 — PASSWORDLESS SSH SETUP (MULTI-NODE REQUIRED)

Generate SSH key

```bash
ssh-keygen -t rsa
```

Copy key to worker nodes

```bash
ssh-copy-id worker1
ssh-copy-id worker2
```

Test connection

```bash
ssh worker1
```

⭐ Required for distributed Hadoop cluster

---

## SECTION 2 — HOSTNAME CONFIGURATION

Edit hosts file

```bash
sudo nano /etc/hosts
```

Example:

```bash
192.168.1.10 master
192.168.1.11 worker1
192.168.1.12 worker2
```

Check hostname

```bash
hostname
```

---

## SECTION 3 — JAVA INSTALLATION CHECK

Check Java version

```bash
java -version
```

Set JAVA_HOME

```bash
nano ~/.bashrc
```

Example:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Reload config

```bash
source ~/.bashrc
```

---

## SECTION 4 — HADOOP CONFIGURATION FILES

Important files:

```bash
core-site.xml
hdfs-site.xml
mapred-site.xml
yarn-site.xml
```

Open config file

```bash
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

---

## SECTION 5 — FORMAT NAMENODE

Run only once (first-time setup)

```bash
hdfs namenode -format
```

⭐ Exam trap command

---

## SECTION 6 — START HDFS SERVICES

Start HDFS

```bash
start-dfs.sh
```

Starts:

```
NameNode
DataNode
SecondaryNameNode
```

Check services

```bash
jps
```

---

## SECTION 7 — START YARN SERVICES

Start YARN

```bash
start-yarn.sh
```

Starts:

```
ResourceManager
NodeManager
```

---

## SECTION 8 — STOP HADOOP CLUSTER

Stop HDFS

```bash
stop-dfs.sh
```

Stop YARN

```bash
stop-yarn.sh
```

Stop everything

```bash
stop-all.sh
```

---

## SECTION 9 — VERIFY CLUSTER STATUS

Check running services

```bash
jps
```

Check nodes

```bash
hdfs dfsadmin -report
```

---

# 2️⃣ HBASE MASTER COMMANDS (VERY IMPORTANT)

📁 Suggested file:

```bash
11_HBase/hbase_master_commands.md
```

Start HBase shell

```bash
hbase shell
```

---

## SECTION 1 — TABLE COMMANDS

Create table

```bash
create 'emp','info'
```

List tables

```bash
list
```

Describe table

```bash
describe 'emp'
```

Disable table

```bash
disable 'emp'
```

Drop table

```bash
drop 'emp'
```

---

## SECTION 2 — INSERT DATA

Insert record

```bash
put 'emp','101','info:name','Rahul'
```

Insert another column

```bash
put 'emp','101','info:age','25'
```

---

## SECTION 3 — READ DATA

Scan full table

```bash
scan 'emp'
```

Get specific row

```bash
get 'emp','101'
```

Get specific column

```bash
get 'emp','101','info:name'
```

---

## SECTION 4 — DELETE DATA

Delete column

```bash
delete 'emp','101','info:name'
```

Delete entire row

```bash
deleteall 'emp','101'
```

---

## SECTION 5 — COUNT RECORDS

```bash
count 'emp'
```

---

## SECTION 6 — FILTER DATA

Prefix filter example

```bash
scan 'emp',{FILTER=>"PrefixFilter('10')"}
```

Column filter example

```bash
scan 'emp',{COLUMNS=>'info:name'}
```

---

## SECTION 7 — ALTER TABLE

Add column family

```bash
alter 'emp','contact'
```

Delete column family

```bash
alter 'emp',{NAME=>'contact',METHOD=>'delete'}
```

---

## SECTION 8 — ENABLE / DISABLE TABLE

Disable

```bash
disable 'emp'
```

Enable

```bash
enable 'emp'
```

---

# 3️⃣ ETL / DATA LAKE COMMANDS MASTER SECTION

📁 Suggested file:

```bash
12_ETL_DataLake/etl_datalake_commands.md
```

ETL pipeline tools:

```
Sqoop
Hive
Spark
Flume
Kafka
```

---

## SECTION 1 — SQOOP IMPORT

Import MySQL → HDFS

```bash
sqoop import \
--connect jdbc:mysql://localhost/company \
--username root \
--password root \
--table employee \
--target-dir /sqoop/emp
```

---

## SECTION 2 — SQOOP EXPORT

Export HDFS → MySQL

```bash
sqoop export \
--connect jdbc:mysql://localhost/company \
--table employee \
--export-dir /sqoop/emp
```

---

## SECTION 3 — IMPORT SINGLE COLUMN

```bash
sqoop import \
--columns id,name \
--table employee
```

---

## SECTION 4 — IMPORT WITH CONDITION

```bash
sqoop import \
--table employee \
--where "salary > 50000"
```

---

## SECTION 5 — FLUME AGENT START

Start flume agent

```bash
flume-ng agent \
--conf conf \
--conf-file example.conf \
--name agent1
```

Pipeline:

```
Source → Channel → Sink
```

---

## SECTION 6 — SPARK ETL READ DATA

Read CSV

```python
df = spark.read.csv("file.csv")
```

Write parquet

```python
df.write.parquet("output")
```

---

## SECTION 7 — HIVE ETL LOAD DATA

```sql
LOAD DATA INPATH '/input/data.txt'
INTO TABLE employee;
```

---
Samajh gaya 👍
Tum bol rahe ho **Cluster Setup, HBase, ETL/Data Lake, Kafka basics ke jo commands diye the unko chhodkar aur bhi important commands chahiye** — matlab **advanced / admin / real-cluster / exam-level extra commands**.

Neeche main **extra commands (new set)** de raha hoon jo typically notes me miss ho jaate hain but exams + interviews me useful hote hain 🚀

---

# 1️⃣ CLUSTER SETUP — ADVANCED ADMIN COMMANDS

## Check Hadoop environment variables

```bash
echo $HADOOP_HOME
echo $JAVA_HOME
echo $PATH
```

---

## Sync cluster time (multi-node cluster requirement)

```bash
sudo apt install ntp
sudo service ntp start
```

⏱️ Time sync important hota hai distributed clusters me

---

## Check Hadoop config directory

```bash
echo $HADOOP_CONF_DIR
```

---

## Verify Namenode web UI

```bash
http://localhost:9870
```

---

## Verify ResourceManager web UI

```bash
http://localhost:8088
```

---

## Check DataNode storage directories

```bash
cat hdfs-site.xml
```

Look for:

```xml
dfs.datanode.data.dir
```

---

## Safe mode operations (very important)

Check safemode status

```bash
hdfs dfsadmin -safemode get
```

Leave safemode

```bash
hdfs dfsadmin -safemode leave
```

Enter safemode

```bash
hdfs dfsadmin -safemode enter
```

⭐ Exam favorite

---

## Refresh Hadoop configuration without restart

```bash
hdfs dfsadmin -refreshNodes
```

---

# 2️⃣ HBASE — ADVANCED COMMANDS (EXAM + ADMIN LEVEL)

## Check HBase status

```bash
status
```

Shows:

* region servers
* live nodes
* dead nodes

---

## Check cluster detailed status

```bash
status 'detailed'
```

---

## List namespace

```bash
list_namespace
```

---

## Create namespace

```bash
create_namespace 'company'
```

---

## Create table inside namespace

```bash
create 'company:emp','info'
```

---

## List tables inside namespace

```bash
list_namespace_tables 'company'
```

---

## Truncate table (delete + recreate)

```bash
truncate 'emp'
```

⭐ Faster than drop + create

---

## Scan with limit

```bash
scan 'emp',{LIMIT=>5}
```

---

## Scan with start row

```bash
scan 'emp',{STARTROW=>'101'}
```

---

## Scan with stop row

```bash
scan 'emp',{STOPROW=>'200'}
```

---

## Scan with timestamp filter

```bash
scan 'emp',{TIMERANGE=>[1000,2000]}
```

---

## Check table existence

```bash
exists 'emp'
```

---

# 3️⃣ ETL / DATA LAKE — EXTRA PRACTICAL COMMANDS

## Sqoop incremental import

```bash
sqoop import \
--table emp \
--incremental append \
--check-column id \
--last-value 100
```

Used for:

📊 Daily data sync jobs

---

## Sqoop import all tables

```bash
sqoop import-all-tables \
--connect jdbc:mysql://localhost/company
```

---

## Sqoop import with split-by

```bash
sqoop import \
--table emp \
--split-by id
```

Improves performance ⚡

---

## Hive external table ETL load

```sql
CREATE EXTERNAL TABLE emp_ext(
id INT,
name STRING
)
LOCATION '/external/emp';
```

---

## Spark ETL JSON read

```python
df = spark.read.json("data.json")
```

---

## Spark write partitioned dataset

```python
df.write.partitionBy("year").parquet("output")
```

Data lake optimization 🚀

---

## Convert CSV → Parquet (classic ETL step)

```python
df = spark.read.csv("file.csv")
df.write.parquet("output")
```

---
