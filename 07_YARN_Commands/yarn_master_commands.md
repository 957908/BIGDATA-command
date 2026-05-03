
07_YARN_Commands/yarn_master_commands.md
```

---

# SECTION 1 — YARN SERVICE START COMMANDS

```bash
# Start YARN services
start-yarn.sh

# Stop YARN services
stop-yarn.sh
```

Start karta hai:

* ResourceManager
* NodeManager

---

# SECTION 2 — CHECK RUNNING YARN SERVICES

```bash
# Check ResourceManager process
jps

# Expected output example
ResourceManager
NodeManager
```

---

# SECTION 3 — LIST ALL RUNNING APPLICATIONS

```bash
# List running YARN applications
yarn application -list
```

Status values:

```
RUNNING
FINISHED
FAILED
KILLED
```

⭐ Exam important

---

# SECTION 4 — CHECK APPLICATION STATUS

```bash
# Show application details
yarn application -status application_123456789
```

Shows:

* state
* queue
* start time
* tracking URL

---

# SECTION 5 — KILL RUNNING APPLICATION

```bash
# Kill running application
yarn application -kill application_123456789
```

Used when job stuck

---

# SECTION 6 — LIST CLUSTER NODES

```bash
# Show NodeManager nodes
yarn node -list
```

Shows:

* node hostname
* memory usage
* CPU usage
* node state

---

# SECTION 7 — CHECK NODE STATUS

```bash
# Show detailed node info
yarn node -status node_id
```

Example:

```
node_id = hostname:port
```

---

# SECTION 8 — LIST ALL YARN QUEUES

```bash
# Show scheduler queues
yarn queue -list
```

Used in multi-user clusters

---

# SECTION 9 — CHECK QUEUE DETAILS

```bash
# Show specific queue status
yarn queue -status default
```

Displays:

* capacity
* running apps
* pending apps

---

# SECTION 10 — VIEW APPLICATION LOGS

```bash
# View application logs
yarn logs -applicationId application_123456789
```

Most used debugging command ⭐

---

# SECTION 11 — CHECK CONTAINER LOGS

```bash
# View container logs
yarn logs -applicationId application_123456789 \
-containerId container_12345
```

Used when reducer/map fails

---

# SECTION 12 — RESOURCE MANAGER WEB UI

Open:

```
http://localhost:8088
```

Shows:

* running jobs
* cluster memory
* CPU usage
* node status

⭐ Very important exam question

---

# SECTION 13 — NODE MANAGER WEB UI

Open:

```
http://localhost:8042
```

Shows:

* containers
* logs
* memory usage

---

# SECTION 14 — JOB HISTORY SERVER UI

Open:

```
http://localhost:19888
```

Shows completed jobs history

---

# SECTION 15 — CHECK CLUSTER RESOURCE STATUS

```bash
# Show cluster metrics
yarn cluster -status
```

Displays:

* total memory
* used memory
* active nodes

---

# SECTION 16 — LIST SCHEDULER INFORMATION

```bash
# Show scheduler info
yarn scheduler
```

Displays scheduling policy

Example:

```
CapacityScheduler
FairScheduler
FIFO Scheduler
```

---

# SECTION 17 — SUBMIT APPLICATION MANUALLY

```bash
# Submit application
yarn jar application.jar MainClass input output
```

Example:

```
yarn jar wordcount.jar WordCount input output
```

---

# SECTION 18 — CHECK APPLICATION ATTEMPTS

```bash
# Show attempts of application
yarn applicationattempt -list application_123456789
```

Useful when retries occur

---

# SECTION 19 — CHECK CONTAINERS OF APPLICATION

```bash
# Show containers used by app
yarn container -list application_123456789
```

Displays:

* container IDs
* node allocation

---

# SECTION 20 — CHECK CONTAINER STATUS

```bash
# Show container details
yarn container -status container_123456789
```

---

# SECTION 21 — RESOURCE MANAGER ROLE (EXAM THEORY)

Responsibilities:

```
Resource allocation
Scheduler management
Application tracking
Cluster monitoring
```

Single per cluster

---

# SECTION 22 — NODE MANAGER ROLE (EXAM THEORY)

Responsibilities:

```
Launch containers
Monitor resource usage
Report node status
Manage logs
```

One per worker node

---

# SECTION 23 — APPLICATION MASTER ROLE

Responsibilities:

```
Negotiate resources
Monitor task execution
Communicate with RM
Manage containers
```

One per application

⭐ Very important concept

---

# SECTION 24 — CONTAINER CONCEPT

Container provides:

```
CPU
RAM
Disk
Network
```

Used to execute:

```
Mapper
Reducer
Spark tasks
Hive jobs
```

---

# SECTION 25 — TYPES OF SCHEDULERS IN YARN

Schedulers:

```
FIFO Scheduler
Capacity Scheduler
Fair Scheduler
```

Default:

```
Capacity Scheduler
```

⭐ Exam trap

---

# SECTION 26 — CHECK MEMORY CONFIGURATION

```bash
# Show node memory allocation
yarn node -list
```

Also check config:

```bash
yarn-site.xml
```

Key property:

```
yarn.nodemanager.resource.memory-mb
```

---

# SECTION 27 — CHECK CPU CONFIGURATION

Property:

```
yarn.nodemanager.resource.cpu-vcores
```

Controls CPU allocation per node

---

# SECTION 28 — CHECK MAXIMUM CONTAINER MEMORY

Property:

```
yarn.scheduler.maximum-allocation-mb
```

Defines max container memory

---

# SECTION 29 — CHECK MINIMUM CONTAINER MEMORY

Property:

```
yarn.scheduler.minimum-allocation-mb
```

Defines min container size

---

# SECTION 30 — MOST IMPORTANT YARN EXAM TRAPS 🎯

Remember these:

```
ResourceManager → single per cluster
NodeManager → one per node
ApplicationMaster → one per job
Container → resource allocation unit
Default scheduler → Capacity Scheduler
ResourceManager UI → port 8088
NodeManager UI → port 8042
JobHistory UI → port 19888
```

---


Jo **CDAC exam + Hadoop admin level** ke liye required hota hai.
