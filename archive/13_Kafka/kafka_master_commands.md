

# 4️⃣ KAFKA BASICS MASTER COMMANDS

📁 Suggested file:

```bash
13_Kafka/kafka_master_commands.md
```

---

## SECTION 1 — START KAFKA SERVICES

Start Zookeeper

```bash
zookeeper-server-start.sh config/zookeeper.properties
```

Start Kafka broker

```bash
kafka-server-start.sh config/server.properties
```

---

## SECTION 2 — CREATE TOPIC

```bash
kafka-topics.sh \
--create \
--topic test-topic \
--bootstrap-server localhost:9092 \
--partitions 1 \
--replication-factor 1
```

---

## SECTION 3 — LIST TOPICS

```bash
kafka-topics.sh \
--list \
--bootstrap-server localhost:9092
```

---

## SECTION 4 — DESCRIBE TOPIC

```bash
kafka-topics.sh \
--describe \
--topic test-topic \
--bootstrap-server localhost:9092
```

---

## SECTION 5 — DELETE TOPIC

```bash
kafka-topics.sh \
--delete \
--topic test-topic \
--bootstrap-server localhost:9092
```

---

## SECTION 6 — START PRODUCER

```bash
kafka-console-producer.sh \
--topic test-topic \
--bootstrap-server localhost:9092
```

Send message:

```
Hello Kafka
```

---

## SECTION 7 — START CONSUMER

```bash
kafka-console-consumer.sh \
--topic test-topic \
--from-beginning \
--bootstrap-server localhost:9092
```

Reads all messages

---

## SECTION 8 — CONSUMER GROUP LIST

```bash
kafka-consumer-groups.sh \
--list \
--bootstrap-server localhost:9092
```

---

## SECTION 9 — DESCRIBE CONSUMER GROUP

```bash
kafka-consumer-groups.sh \
--describe \
--group group1 \
--bootstrap-server localhost:9092
```



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



# 4️⃣ KAFKA — ADVANCED PRACTICAL COMMANDS

## Check broker API versions

```bash
kafka-broker-api-versions.sh \
--bootstrap-server localhost:9092
```

---

## Increase topic partitions

```bash
kafka-topics.sh \
--alter \
--topic test-topic \
--partitions 3 \
--bootstrap-server localhost:9092
```

---

## Check topic offsets

```bash
kafka-run-class.sh kafka.tools.GetOffsetShell \
--broker-list localhost:9092 \
--topic test-topic
```

---

## Reset consumer group offset

```bash
kafka-consumer-groups.sh \
--group group1 \
--reset-offsets \
--to-earliest \
--execute \
--topic test-topic \
--bootstrap-server localhost:9092
```

---

## Start consumer from latest message

```bash
kafka-console-consumer.sh \
--topic test-topic \
--bootstrap-server localhost:9092
```

(no `--from-beginning`)

---

## Check Kafka configs

```bash
kafka-configs.sh \
--describe \
--bootstrap-server localhost:9092
```

---

## Check active brokers

```bash
zookeeper-shell.sh localhost:2181 ls /brokers/ids
```

---



