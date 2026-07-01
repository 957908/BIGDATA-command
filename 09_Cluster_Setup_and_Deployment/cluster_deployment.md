# 🛠️ Module 09: Cluster Setup & Deployment Guide

Deploying a distributed big data cluster requires precise configuration of core XML files, setup of operating system prerequisites, and formatting of name systems. This guide provides the complete XML configuration blueprints for production, a step-by-step multi-node deployment protocol, and a functional `docker-compose.yml` file to run a Hadoop/Spark environment locally.

---

## 1. Core Hadoop XML Configuration Blueprints

Hadoop configurations are stored in XML files under the `$HADOOP_HOME/etc/hadoop/` directory.

### 1. `core-site.xml` (Common Cluster Settings)
Defines the default filesystem URI and temporary directory paths.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <!-- The entrypoint URL of the active NameNode -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master-node:9000</value>
    </property>

    <!-- Base directory for local temp files (spills, tokens) -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/app/hadoop/tmp</value>
    </property>

    <!-- Enable trash bin for safe HDFS file recovery (7-day retention) -->
    <property>
        <name>fs.trash.interval</name>
        <value>10080</value>
    </property>
</configuration>
```

### 2. `hdfs-site.xml` (Storage Configuration)
Defines block replication factor, directory metadata storage, and high availability settings.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Default replication factor -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>

    <!-- Directory where NameNode stores transaction logs (Edits/FsImage) -->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///app/hadoop/metadata</value>
    </property>

    <!-- Directory where DataNodes write block files -->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///app/hadoop/data</value>
    </property>

    <!-- Default Block Size (256 MB for enterprise warehousing) -->
    <property>
        <name>dfs.blocksize</name>
        <value>268435456</value>
    </property>

    <!-- Disable permission checking (convenient for testing; enable in production) -->
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>
</configuration>
```

### 3. `yarn-site.xml` (Resource Allocation Configuration)
Configures NodeManager memory capacity and map-shuffle auxiliary services.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Hostname of the central YARN ResourceManager -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master-node</value>
    </property>

    <!-- Auxiliary shuffle service required for MapReduce execution -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- Memory capacity allocated to YARN containers on each NodeManager -->
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>16384</value> <!-- 16 GB -->
    </property>

    <!-- Virtual Cores allocated to YARN on each NodeManager -->
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>8</value>
    </property>

    <!-- Memory limits per container -->
    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>1024</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>16384</value>
    </property>
</configuration>
```

---

## 2. Multi-Node Cluster Installation Protocol

Follow these steps sequentially to configure a physical 3-node cluster: `master-node`, `worker-node-1`, and `worker-node-2`.

### Step 1: Operating System Prerequisites (All Nodes)
* Configure `/etc/hosts` to resolve node names:
  ```text
  192.168.1.100 master-node
  192.168.1.101 worker-node-1
  192.168.1.102 worker-node-2
  ```
* Install Java 8 JDK:
  ```bash
  sudo apt-get update && sudo apt-get install openjdk-8-jdk -y
  ```
* Ensure passwordless SSH is configured from `master-node` to both workers (Refer to Module 01).

### Step 2: Extract Hadoop & Configure Environment Variables (All Nodes)
* Download and extract Hadoop binaries to `/usr/local/hadoop`:
  ```bash
  tar -xvf hadoop-3.3.6.tar.gz
  sudo mv hadoop-3.3.6 /usr/local/hadoop
  ```
* Add environment variables to `~/.bashrc`:
  ```bash
  export HADOOP_HOME=/usr/local/hadoop
  export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
  export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
  ```
  Run `source ~/.bashrc` to load the changes.

---

## 3. Complete Cluster Administration & Verification Reference

This library details docker container commands and diagnostic checks.

### A. Docker Compose CLI Operations
* **Launch Cluster**:
  ```bash
  docker-compose up -d
  ```
* **Stop and Purge Container Data (Clear Volumes)**:
  ```bash
  docker-compose down -v
  ```
* **Enter Container Shell**:
  ```bash
  docker exec -it namenode bash
  ```
* **Monitor Container Daemon Logs**:
  ```bash
  docker logs -f resourcemanager
  ```

### B. Daemon Health Verification Commands
* **JVM Check**: `jps -lm` (Shows PIDs, class paths, and startup parameters).
* **Storage Check**: `hdfs dfsadmin -report` (Shows dead datanodes, raw space occupancy).
* **Resource Check**: `yarn node -list -all` (Shows RAM/Core allocations per NodeManager).
* **HBase Check**: `hbase shell` -> `status` (Shows live regions and server loads).
* **Cassandra Check**: `nodetool status` (Shows Up/Normal statuses of token rings).

---

## 4. Containerized Cluster: Docker Compose Blueprint

Use this `docker-compose.yml` file to quickly spin up a local 3-node testing cluster (NameNode, DataNode, and ResourceManager).

```yaml
version: "3.7"

services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    restart: always
    ports:
      - "9870:9870"
      - "9000:9000"
    volumes:
      - hadoop_namenode:/hadoop/dfs/name
    environment:
      - CLUSTER_NAME=test-cluster
    env_file:
      - ./hadoop.env

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    container_name: datanode
    restart: always
    ports:
      - "9864:9864"
    volumes:
      - hadoop_datanode:/hadoop/dfs/data
    environment:
      - SERVICE_PRECONDITION=namenode:9870
    env_file:
      - ./hadoop.env
    depends_on:
      - namenode

  resourcemanager:
    image: bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8
    container_name: resourcemanager
    restart: always
    ports:
      - "8088:8088"
    environment:
      - SERVICE_PRECONDITION=namenode:9000
    env_file:
      - ./hadoop.env
    depends_on:
      - namenode

volumes:
  hadoop_namenode:
  hadoop_datanode:
```

### Verification Endpoints:
After executing `docker-compose up -d`, access these interfaces from your browser:
* **HDFS NameNode UI**: `http://localhost:9870`
* **YARN ResourceManager UI**: `http://localhost:8088`

---

## 5. Enterprise Job Interview Q&A (Cluster Setup & Deployment)

This section prepares you for production-level interview questions.

### Q1: Why does formatting the NameNode on a running cluster cause data loss? Explain the low-level mechanism.
* **How to explain this to the interviewer**:
  Explain that formatting is not just deleting files; it generates a brand new Cluster ID. Detail what happens to the connection handshake between DataNodes (which retain the old ID) and the newly formatted NameNode.

* **Model Answer**:
  "Formatting the NameNode (`hdfs namenode -format`) is a destructive operation because it creates a new **Cluster ID** and **Namespace ID** in the NameNode's local metadata folders (`VERSION` file).
  
  The physical blocks on the DataNodes still hold the **old** Cluster ID in their local block metadata. 
  
  When HDFS starts:
  1. DataNodes establish connection handshakes with the NameNode.
  2. DataNodes transmit their local Cluster IDs.
  3. The NameNode notices the mismatch (old Cluster ID on DataNodes vs. new Cluster ID on NameNode).
  4. The NameNode immediately rejects the DataNodes' connections, preventing them from joining the cluster.
  
  As a result, even though the raw block files still physically exist on the DataNodes' hard drives, the newly formatted NameNode will report them as lost, rendering all historical data completely unreadable. To resolve this, you would have to manually edit the `VERSION` file on all DataNodes to match the new Cluster ID, which is highly risky and prone to corruption."

---

### Q2: How do you identify and troubleshoot long JVM Garbage Collection (GC) pauses on a NameNode? What JVM flags would you configure to optimize this?
* **How to explain this to the interviewer**:
  Explain how to identify GC pauses from JVM logs or thread dumps, then specify modern garbage collectors (like G1GC) and explain how to configure target pause times to prevent cluster timeouts.

* **Model Answer**:
  "Long JVM GC pauses (Stop-the-World events) on the NameNode can cause ZKFC heartbeats to timeout, triggering accidental cluster failovers.
  
  **Identification**:
  * I inspect the NameNode logs for warnings like `GC pool 'G1 Old Generation' spent 12 seconds collecting`.
  * I monitor active heap usage using JVM tools like `jstat -gcutil <PID> 1000` or attach VisualVM.
  
  **Optimization**:
  To resolve this, I switch the default collector to **G1GC** (Garbage-First Garbage Collector) and configure optimizations in `hadoop-env.sh`:
  1. **Enable G1GC**:
     `-XX:+UseG1GC`
  2. **Set Max GC Pause Target**:
     `-XX:MaxGCPauseMillis=200` (Instructs the JVM to keep individual GC cycles under 200 milliseconds).
  3. **Initiating Heap Occupancy**:
     `-XX:InitiatingHeapOccupancyPercent=45` (Starts background cleaning cycles early when 45% of the heap is occupied, preventing the heap from filling up and forcing fallback Full GC cycles).
  4. **Dump on OOM**:
     `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/` (Ensures that if the NameNode crashes, a heap dump is captured for root-cause analysis)."

---

### Q3: How do you execute a high-volume data migration from an on-premise HDFS cluster to an AWS S3 bucket? What tool is used, and how is it optimized?
* **How to explain this to the interviewer**:
  Name the exact tool used (**DistCp**) and explain that it runs as a Map-only MapReduce job. Explain how to configure security credentials and how to optimize parallel mapping bandwidth.

* **Model Answer**:
  "To execute high-volume data migrations from on-premise HDFS to AWS S3, I use the **DistCp** (Distributed Copy) tool. DistCp launches a Map-only MapReduce job where each mapper copies a subset of files in parallel.
  
  **Configuration Steps**:
  1. I configure the S3 connector in `core-site.xml` by defining the **S3A filesystem client** properties:
     ```xml
     <property>
         <name>fs.s3a.impl</name>
         <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
     </property>
     ```
  2. I supply security credentials (either IAM roles or AWS access/secret keys configured securely in Hadoop Credentials Provider).
  
  **Execution and Optimization**:
  I run the command:
  `hadoop distcp -update -skipcrccheck -bandwidth 50 -m 100 hdfs:///data/sales s3a://my-bucket/data/sales`
  
  * **Optimizations used**:
    * `-update`: Copies only files that have changed or do not exist in S3 (incremental sync).
    * `-skipcrccheck`: S3 uses ETag headers for checksums, which differ from HDFS's block checksums. Skipping CRC comparison avoids verification failures.
    * `-bandwidth 50`: Limits copy bandwidth per mapper to 50MB/s to prevent saturated network connections on on-premise switches.
    * `-m 100`: Allocates exactly 100 mapper tasks in YARN to control cluster parallel load."
