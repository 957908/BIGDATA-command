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
* Install Java 8 JDK (Hadoop is built primarily on Java 8):
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

### Step 3: Populate Cluster Workers File (Master Node Only)
* Edit `/usr/local/hadoop/etc/hadoop/workers` and list worker hostnames:
  ```text
  worker-node-1
  worker-node-2
  ```

### Step 4: Format and Launch HDFS & YARN (Master Node Only)
* **Format NameNode HDFS Namespace** (Run once before the first startup):
  ```bash
  hdfs namenode -format
  ```
* **Start HDFS Daemons** (Spins up NameNode locally, and DataNodes on worker nodes):
  ```bash
  start-dfs.sh
  ```
* **Start YARN Daemons** (Spins up ResourceManager locally, and NodeManagers on workers):
  ```bash
  start-yarn.sh
  ```

### Step 5: Verify Daemon Operations (All Nodes)
* Run `jps` on master node:
  * Output should show: `NameNode`, `ResourceManager`, `DFSZKFailoverController` (if HA).
* Run `jps` on worker nodes:
  * Output should show: `DataNode`, `NodeManager`.

---

## 3. Containerized Cluster: Docker Compose Blueprint

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

## 🎯 Exam and Interview Traps

1. **Trap: Why does formatting the NameNode on a running cluster cause data loss?**
   * **Answer**: Formatting the NameNode (`hdfs namenode -format`) generates a brand new **Cluster ID** in the NameNode metadata directory. The existing blocks on the DataNodes still hold the old Cluster ID. When DataNodes send block reports, the NameNode rejects them due to mismatching IDs, rendering all previously stored block files completely unreadable.

2. **Trap: Why do worker nodes fail to join the cluster with "Retrying connect to server: master-node/192.168.1.100:9000"?**
   * **Answer**: Two typical causes:
     1. The system firewall (`iptables` or `ufw`) is active on `master-node` and blocking port `9000`.
     2. The IP address bind configurations are incorrect. If `fs.defaultFS` in `core-site.xml` is set to `hdfs://127.0.0.1:9000` instead of the master's physical LAN IP, the NameNode binds only to the local loopback adapter, preventing external worker nodes from connecting.

3. **Trap: What is the purpose of `yarn.nodemanager.aux-services` and what happens if it is misconfigured?**
   * **Answer**: The auxiliary service enables MapReduce to perform shuffles by running a small auxiliary web server on every NodeManager to serve map task output files. If it is omitted or spelled incorrectly (e.g., missing `mapreduce_shuffle`), YARN container tasks will compile but fail immediately during the shuffle execution stage with a connection refused exception.
