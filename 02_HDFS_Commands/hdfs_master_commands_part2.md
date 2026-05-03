#### SECTION 21 — NAMENODE ADMIN COMMANDS

# Show NameNode status
hdfs namenode -report

# Format NameNode (first-time setup only)
hdfs namenode -format

# Start NameNode manually
hdfs --daemon start namenode

# Stop NameNode
hdfs --daemon stop namenode

# Upgrade NameNode metadata
hdfs namenode -upgrade

# Rollback NameNode
hdfs namenode -rollback
SECTION 22 — DATANODE ADMIN COMMANDS
# Start DataNode
hdfs --daemon start datanode

# Stop DataNode
hdfs --daemon stop datanode

# Refresh DataNodes list
hdfs dfsadmin -refreshNodes

# Check DataNode configuration
hdfs datanode -getDatanodeInfo

#### SECTION 23 — SECONDARY NAMENODE COMMANDS

# Start Secondary NameNode
hdfs --daemon start secondarynamenode

# Stop Secondary NameNode
hdfs --daemon stop secondarynamenode

# Force checkpoint creation
hdfs secondarynamenode -checkpoint

# Check checkpoint status
hdfs secondarynamenode -getCheckpointStatus


#### SECTION 24 — CHECKPOINT MANAGEMENT

# Trigger checkpoint manually
hdfs dfsadmin -saveNamespace

# Save filesystem namespace
hdfs dfsadmin -fetchImage imagefile

# Verify namespace image
hdfs oiv -i fsimage -o output.xml


#### SECTION 25 — QUOTA MANAGEMENT (VERY IMPORTANT FOR EXAMS)

# Set namespace quota
hdfs dfsadmin -setQuota 1000 /data

# Remove namespace quota
hdfs dfsadmin -clrQuota /data

# Set diskspace quota
hdfs dfsadmin -setSpaceQuota 10g /data

# Remove diskspace quota
hdfs dfsadmin -clrSpaceQuota /data


## SECTION 26 — TRASH MANAGEMENT COMMANDS

# Move deleted files to trash (default behavior)
hdfs dfs -rm /data/file.txt

# Skip trash while deleting
hdfs dfs -rm -skipTrash /data/file.txt

# Show trash directory
hdfs dfs -ls /user/username/.Trash

# Expunge trash immediately
hdfs dfs -expunge

### SECTION 27 — STORAGE POLICY COMMANDS

# Show storage policies
hdfs storagepolicies -listPolicies

# Set storage policy
hdfs storagepolicies -setStoragePolicy \
-path /data -policy COLD

# Get storage policy
hdfs storagepolicies -getStoragePolicy \
-path /data

# Remove storage policy
hdfs storagepolicies -unsetStoragePolicy \
-path /data

Common policies:

HOT
COLD
WARM
ALL_SSD
ONE_SSD
 
 ### SECTION 28 — BLOCK MANAGEMENT COMMANDS

# Show block information
hdfs fsck /data/file.txt -files -blocks

# Locate block replicas
hdfs fsck /data/file.txt -locations

# Show missing blocks
hdfs fsck / -list-corruptfileblocks

# Recover corrupted blocks
hdfs fsck / -move
SECTION 29 — FILE LEASE MANAGEMENT
# Recover file lease
hdfs debug recoverLease /data/file.txt

# Verify lease recovery
hdfs debug verifyMeta /data/file.txt

### SECTION 30 — REPLICATION MONITORING

# List under-replicated blocks
hdfs fsck / -list-corruptfileblocks

# Trigger replication manually
hdfs dfsadmin -triggerBlockReport

# Refresh replication queues
hdfs dfsadmin -refreshNodes
SECTION 31 — CLUSTER BALANCING COMMANDS
# Start cluster balancer
hdfs balancer

# Run balancer with threshold
hdfs balancer -threshold 10

# Run balancer in background
hdfs balancer &



### SECTION 32 — DECOMMISSION DATANODES

# Add DataNode to exclude list
nano dfs.exclude

# Refresh nodes after editing exclude list
hdfs dfsadmin -refreshNodes

# Monitor decommission status
hdfs dfsadmin -report

### SECTION 33 — RECOMMISSION DATANODES

# Remove DataNode from exclude file
nano dfs.exclude

# Refresh nodes again
hdfs dfsadmin -refreshNodes


### SECTION 34 — CLUSTER HEALTH CHECK COMMANDS

# Check filesystem health
hdfs fsck /

# Check block health
hdfs fsck / -blocks

# Check replication health
hdfs fsck / -files

# Check corrupted files
hdfs fsck / -list-corruptfileblocks

### SECTION 35 — METADATA OPERATIONS

# Fetch fsimage
hdfs dfsadmin -fetchImage fsimage.img

# Convert fsimage to XML
hdfs oiv -i fsimage.img -o fsimage.xml

# View edits log
hdfs oev -i edits -o edits.xml

## SECTION 36 — SAFE MODE ADVANCED COMMANDS

# Enter safe mode
hdfs dfsadmin -safemode enter

# Leave safe mode
hdfs dfsadmin -safemode leave

# Force exit safe mode
hdfs dfsadmin -safemode forceExit

# Check safe mode state
hdfs dfsadmin -safemode get

### SECTION 37 — RACK AWARENESS COMMANDS

# Refresh rack configuration
hdfs dfsadmin -refreshNodes

# Verify rack awareness
hdfs dfsadmin -printTopology


### SECTION 38 — CLUSTER TOPOLOGY COMMANDS

# Show cluster topology
hdfs dfsadmin -printTopology

# Show DataNode placement
hdfs dfsadmin -report

### SECTION 39 — SNAPSHOT DIFF COMMANDS

# Compare snapshots
hdfs dfs -diff /data snap1 snap2

# List snapshots
hdfs dfs -lsSnapshottableDir

# Rename snapshot
hdfs dfs -renameSnapshot /data snap1 snap_new

#### SECTION 40 — HIGH AVAILABILITY (HA) COMMANDS

# Show HA service state
hdfs haadmin -getServiceState nn1

# Transition NameNode to active
hdfs haadmin -transitionToActive nn1

# Transition NameNode to standby
hdfs haadmin -transitionToStandby nn2

# Failover manually
hdfs haadmin -failover nn1 nn2

### SECTION 41 — JOURNALNODE COMMANDS (HA SETUP)

# Start JournalNode
hdfs --daemon start journalnode

# Stop JournalNode
hdfs --daemon stop journalnode

# Format JournalNode
hdfs journalnode -format

### SECTION 42 — FEDERATION COMMANDS

# Show namespace info
hdfs getconf -namenodes

# Show federation configuration
hdfs getconf -confKey dfs.nameservices
SECTION 43 — CONFIGURATION CHECK COMMANDS
# Show config parameter
hdfs getconf -confKey dfs.replication

# Show NameNode address
hdfs getconf -namenodes

# Show DataNode address
hdfs getconf -nnRpcAddresses

### SECTION 44 — LOG MANAGEMENT COMMANDS

# View NameNode logs
tail -f $HADOOP_HOME/logs/hadoop-*-namenode-*.log

# View DataNode logs
tail -f $HADOOP_HOME/logs/hadoop-*-datanode-*.log

# View SecondaryNameNode logs
tail -f $HADOOP_HOME/logs/hadoop-*-secondarynamenode-*.log
 
 ### SECTION 45 — HDFS DEBUG COMMANDS

#Debug block metadata
hdfs debug verifyMeta /data/file.txt

# Recover corrupted lease
hdfs debug recoverLease /data/file.txt

# Verify block checksum
hdfs dfs -checksum /data/file.txt
