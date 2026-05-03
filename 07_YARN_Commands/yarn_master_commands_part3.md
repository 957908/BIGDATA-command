
```bash
07_YARN_Commands/yarn_master_commands_part3.md
```

---

# SECTION 69 — YARN COMPLETE EXECUTION FLOW (ARCHITECTURE PIPELINE)

```text
Client → ResourceManager → Scheduler → ApplicationMaster
→ NodeManager → Container → Task Execution
```

⭐ Very important exam architecture flow

---

# SECTION 70 — APPLICATION SUBMISSION WORKFLOW

Execution steps:

```text
1. Client submits job to ResourceManager
2. RM allocates container for ApplicationMaster
3. ApplicationMaster starts
4. AM negotiates containers from RM
5. Containers launched on NodeManagers
6. Tasks executed
7. Results stored in HDFS
```

---

# SECTION 71 — RESOURCE REQUEST NEGOTIATION PROCESS

ApplicationMaster requests:

```text
Memory
CPU cores
Node preference
Rack preference
```

From:

```text
ResourceManager Scheduler
```

---

# SECTION 72 — CONTAINER LIFECYCLE (STEP-BY-STEP)

```text
Requested → Allocated → Launched → Running → Completed
```

States:

```text
NEW
LOCALIZING
RUNNING
COMPLETED
FAILED
KILLED
```

---

# SECTION 73 — APPLICATION STATES IN YARN

```bash
yarn application -list
```

Possible states:

```text
NEW
SUBMITTED
ACCEPTED
RUNNING
FINISHED
FAILED
KILLED
```

⭐ Frequently asked in exams

---

# SECTION 74 — APPLICATION ATTEMPT RETRY LOGIC

Check retries:

```bash
yarn applicationattempt -list application_123456
```

Max attempts controlled by:

```xml
yarn.resourcemanager.am.max-attempts
```

Default:

```text
2
```

---

# SECTION 75 — NODE STATES IN YARN

Check node state:

```bash
yarn node -list
```

Possible states:

```text
RUNNING
UNHEALTHY
DECOMMISSIONED
LOST
REBOOTED
```

---

# SECTION 76 — NODE DECOMMISSION PROCESS

Step 1:

```bash
nano yarn.exclude
```

Add node hostname

Step 2:

```bash
yarn rmadmin -refreshNodes
```

Node removed safely

---

# SECTION 77 — NODE RECOMMISSION PROCESS

Remove hostname from:

```bash
yarn.exclude
```

Then refresh:

```bash
yarn rmadmin -refreshNodes
```

---

# SECTION 78 — RESOURCE MANAGER SCHEDULER ROLE

Scheduler handles:

```text
Queue allocation
Priority handling
Fair sharing
Capacity guarantees
```

Does NOT monitor execution

⭐ Exam trap question

---

# SECTION 79 — APPLICATION MASTER FAILURE HANDLING

If AM crashes:

```text
RM restarts ApplicationMaster
```

Controlled by:

```xml
yarn.resourcemanager.am.max-attempts
```

---

# SECTION 80 — HEARTBEAT MECHANISM

NodeManager sends heartbeat to:

```text
ResourceManager
```

Contains:

```text
Health status
Container usage
Memory usage
CPU usage
```

Frequency controlled by:

```xml
yarn.resourcemanager.nodemanagers.heartbeat-interval-ms
```

---

# SECTION 81 — NODE HEALTH CHECK SCRIPT

Configure script:

```xml
yarn.nodemanager.health-checker.script.path
```

Marks node:

```text
Healthy
Unhealthy
```

---

# SECTION 82 — CONTAINER RESOURCE ISOLATION

Controlled using:

```text
LinuxContainerExecutor
```

Provides:

```text
CPU isolation
Memory isolation
User isolation
```

---

# SECTION 83 — MEMORY OVERUSE TERMINATION

If container exceeds memory:

```text
NodeManager kills container
```

Controlled by:

```xml
yarn.nodemanager.pmem-check-enabled
```

---

# SECTION 84 — VIRTUAL MEMORY LIMIT CONTROL

Property:

```xml
yarn.nodemanager.vmem-pmem-ratio
```

Default:

```text
2.1
```

Meaning:

```text
Virtual memory = 2.1 × physical memory
```

---

# SECTION 85 — LOG AGGREGATION WORKFLOW

When enabled:

```text
NodeManager logs → HDFS → Aggregated logs directory
```

Access using:

```bash
yarn logs -applicationId application_123456
```

---

# SECTION 86 — CONTAINER LOCALIZATION PROCESS

Before execution:

```text
Download jars
Download configs
Download libraries
```

Stored in:

```text
NodeManager local directory
```

---

# SECTION 87 — YARN LOCAL DIRECTORIES CONFIG

Property:

```xml
yarn.nodemanager.local-dirs
```

Stores:

```text
Temporary container files
Localized resources
Intermediate outputs
```

---

# SECTION 88 — YARN LOG DIRECTORY CONFIG

Property:

```xml
yarn.nodemanager.log-dirs
```

Stores:

```text
Container logs
Application logs
Error logs
```

---

# SECTION 89 — QUEUE CAPACITY CONFIGURATION

Example:

```xml
yarn.scheduler.capacity.root.default.capacity=50
```

Means:

```text
Queue gets 50% cluster resources
```

---

# SECTION 90 — FAIR SCHEDULER RESOURCE SHARING RULE

FairScheduler ensures:

```text
Equal resource distribution between jobs
```

Even if:

```text
Jobs submitted at different times
```

---

# SECTION 91 — FIFO SCHEDULER BEHAVIOR

FIFO executes jobs:

```text
First submitted → First executed
```

No fairness guarantee

---

# SECTION 92 — CAPACITY SCHEDULER BEHAVIOR

CapacityScheduler ensures:

```text
Minimum resource guarantee per queue
```

Supports:

```text
Multi-tenant clusters
```

⭐ Default scheduler

---

# SECTION 93 — YARN SECURITY AUTHENTICATION

Enable Kerberos:

```xml
hadoop.security.authentication=kerberos
```

Used in production clusters

---

# SECTION 94 — CHECK ACTIVE RESOURCE MANAGER (HA MODE)

```bash
yarn rmadmin -getServiceState rm1
```

Output:

```text
active
standby
```

---

# SECTION 95 — FORCE RESOURCE MANAGER FAILOVER

```bash
yarn rmadmin -transitionToActive rm1
```

Switch standby → active

---

# SECTION 96 — APPLICATION PRIORITY CONTROL

Set priority:

```bash
yarn application -appId application_123 \
-updatePriority 5
```

Higher value = higher priority

---

# SECTION 97 — CHECK SCHEDULER QUEUE METRICS

```bash
yarn queue -status default
```

Displays:

```text
Queue capacity
Used memory
Pending jobs
Running jobs
```

---

# SECTION 98 — CONTAINER EXECUTION ENVIRONMENT VARIABLES

Example:

```text
CLASSPATH
JAVA_HOME
HADOOP_CONF_DIR
```

Configured inside:

```text
yarn-env.sh
```

---

# SECTION 99 — RESOURCE CALCULATOR TYPE

Property:

```xml
yarn.scheduler.capacity.resource-calculator
```

Options:

```text
DefaultResourceCalculator
DominantResourceCalculator
```

Used in multi-resource scheduling

---

# SECTION 100 — MOST IMPORTANT YARN INTERNAL EXAM TRAPS 🎯

Remember:

```text
ResourceManager schedules resources only
ApplicationMaster manages job execution
NodeManager launches containers
Container = smallest execution unit
Scheduler default = CapacityScheduler
Heartbeat sent by NodeManager
AM restart attempts default = 2
Log aggregation stores logs in HDFS
```



Next logical module (CDAC Big Data sequence) is usually **Spark Master Commands** — want me to build that next?
