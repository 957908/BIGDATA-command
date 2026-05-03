
07_YARN_Commands/yarn_master_commands_part2.md
```

---

# SECTION 31 — CHECK YARN CONFIGURATION FILE LOCATION

```bash
# Open YARN configuration file
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

Contains:

* memory config
* CPU config
* scheduler settings
* log aggregation config

---

# SECTION 32 — NODEMANAGER MEMORY CONFIGURATION

```xml
<property>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>8192</value>
</property>
```

Defines total memory available per node

---

# SECTION 33 — NODEMANAGER CPU CONFIGURATION

```xml
<property>
<name>yarn.nodemanager.resource.cpu-vcores</name>
<value>4</value>
</property>
```

Defines CPU cores per node

---

# SECTION 34 — MINIMUM CONTAINER MEMORY

```xml
<property>
<name>yarn.scheduler.minimum-allocation-mb</name>
<value>512</value>
</property>
```

Smallest container size allowed

---

# SECTION 35 — MAXIMUM CONTAINER MEMORY

```xml
<property>
<name>yarn.scheduler.maximum-allocation-mb</name>
<value>8192</value>
</property>
```

Largest container size allowed

---

# SECTION 36 — ENABLE LOG AGGREGATION

```xml
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>
```

Stores logs in HDFS

---

# SECTION 37 — LOG AGGREGATION DIRECTORY

```xml
<property>
<name>yarn.nodemanager.remote-app-log-dir</name>
<value>/tmp/logs</value>
</property>
```

Location where logs stored

---

# SECTION 38 — CHECK AGGREGATED LOGS

```bash
yarn logs -applicationId application_123456789
```

Reads logs from HDFS

---

# SECTION 39 — ENABLE RESOURCE MANAGER HA

```xml
<property>
<name>yarn.resourcemanager.ha.enabled</name>
<value>true</value>
</property>
```

Enables High Availability mode

---

# SECTION 40 — DEFINE RESOURCE MANAGER HA IDS

```xml
<property>
<name>yarn.resourcemanager.ha.rm-ids</name>
<value>rm1,rm2</value>
</property>
```

Defines HA ResourceManagers

---

# SECTION 41 — CONFIGURE ACTIVE RESOURCE MANAGER HOST

```xml
<property>
<name>yarn.resourcemanager.hostname.rm1</name>
<value>master1</value>
</property>
```

---

# SECTION 42 — CONFIGURE STANDBY RESOURCE MANAGER HOST

```xml
<property>
<name>yarn.resourcemanager.hostname.rm2</name>
<value>master2</value>
</property>
```

---

# SECTION 43 — ENABLE AUTOMATIC FAILOVER

```xml
<property>
<name>yarn.resourcemanager.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
```

Switches RM automatically if failure occurs

---

# SECTION 44 — CHECK ACTIVE RESOURCE MANAGER

```bash
yarn rmadmin -getServiceState rm1
```

Output:

```bash
active
standby
```

---

# SECTION 45 — MANUAL FAILOVER RESOURCE MANAGER

```bash
yarn rmadmin -transitionToActive rm1
```

Switch standby → active

---

# SECTION 46 — REFRESH NODEMANAGER LIST

```bash
yarn rmadmin -refreshNodes
```

Reload nodes after config change

---

# SECTION 47 — REFRESH QUEUES

```bash
yarn rmadmin -refreshQueues
```

Reload scheduler queues

---

# SECTION 48 — REFRESH USER GROUPS

```bash
yarn rmadmin -refreshUserToGroupsMappings
```

Reload user permissions

---

# SECTION 49 — CHECK CLUSTER HEALTH

```bash
yarn rmadmin -getClusterMetrics
```

Displays:

* total nodes
* active nodes
* memory usage

---

# SECTION 50 — CHECK NODE LABELS

```bash
yarn node -list -showDetails
```

Shows label-based scheduling info

---

# SECTION 51 — ADD NODE LABEL

```bash
yarn rmadmin -addToClusterNodeLabels "gpu"
```

Used for GPU scheduling clusters

---

# SECTION 52 — REMOVE NODE LABEL

```bash
yarn rmadmin -removeFromClusterNodeLabels "gpu"
```

---

# SECTION 53 — ASSIGN NODE LABEL TO NODE

```bash
yarn rmadmin -replaceLabelsOnNode \
"node1=gpu"
```

---

# SECTION 54 — LIST NODE LABELS

```bash
yarn cluster --list-node-labels
```

---

# SECTION 55 — CHECK SCHEDULER CONFIGURATION

```bash
yarn scheduler
```

Displays:

* FIFO
* Capacity
* Fair scheduler

---

# SECTION 56 — ENABLE CAPACITY SCHEDULER

```xml
<property>
<name>yarn.resourcemanager.scheduler.class</name>
<value>
org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler
</value>
</property>
```

Default scheduler

---

# SECTION 57 — ENABLE FAIR SCHEDULER

```xml
<property>
<name>yarn.resourcemanager.scheduler.class</name>
<value>
org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler
</value>
</property>
```

Equal resource sharing

---

# SECTION 58 — ENABLE FIFO SCHEDULER

```xml
<property>
<name>yarn.resourcemanager.scheduler.class</name>
<value>
org.apache.hadoop.yarn.server.resourcemanager.scheduler.fifo.FifoScheduler
</value>
</property>
```

Simple queue scheduling

---

# SECTION 59 — CHECK APPLICATION ATTEMPTS

```bash
yarn applicationattempt -list application_123456
```

Shows retry attempts

---

# SECTION 60 — KILL STUCK CONTAINER

```bash
yarn container -kill container_123456
```

Stops failed container manually

---

# SECTION 61 — CHECK RESOURCE MANAGER LOGS

```bash
tail -f $HADOOP_HOME/logs/yarn-*-resourcemanager-*.log
```

---

# SECTION 62 — CHECK NODE MANAGER LOGS

```bash
tail -f $HADOOP_HOME/logs/yarn-*-nodemanager-*.log
```

---

# SECTION 63 — CHECK APPLICATION MASTER LOGS

```bash
yarn logs -applicationId application_123456
```

Used for debugging job failures

---

# SECTION 64 — ENABLE CONTAINER MEMORY CHECK

```xml
<property>
<name>yarn.nodemanager.pmem-check-enabled</name>
<value>true</value>
</property>
```

Prevents memory overuse

---

# SECTION 65 — ENABLE VIRTUAL MEMORY CHECK

```xml
<property>
<name>yarn.nodemanager.vmem-check-enabled</name>
<value>true</value>
</property>
```

Prevents excessive virtual memory usage

---

# SECTION 66 — CHECK CLUSTER MEMORY USAGE

```bash
yarn cluster -status
```

Displays cluster resource summary

---

# SECTION 67 — CHECK RUNNING CONTAINERS

```bash
yarn container -list application_123456
```

Shows active containers

---

# SECTION 68 — MOST IMPORTANT YARN ADMIN EXAM TRAPS 🎯

Remember these:

```
ResourceManager HA → supported
NodeManager runs per node
ApplicationMaster runs per job
Container = execution unit
Scheduler default = CapacityScheduler
Logs stored in HDFS when aggregation enabled
RM UI → 8088
NM UI → 8042
```
