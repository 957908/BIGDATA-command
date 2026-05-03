SECTION 46 — DISTCP COMMANDS (CLUSTER-TO-CLUSTER COPY)
# Copy data between two HDFS clusters
hadoop distcp hdfs://srccluster/data hdfs://destcluster/data

# Copy directory recursively
hadoop distcp /input /backup

# Overwrite existing files
hadoop distcp -overwrite /input /backup

# Update only changed files
hadoop distcp -update /input /backup

# Preserve metadata (permission, replication)
hadoop distcp -p /input /backup


SECTION 47 — WEBHDFS REST COMMANDS
# List directory via REST API
curl "http://localhost:9870/webhdfs/v1/?op=LISTSTATUS"

# Create directory via REST
curl -i -X PUT \
"http://localhost:9870/webhdfs/v1/data?op=MKDIRS"

# Upload file via REST
curl -i -X PUT \
-T file.txt \
"http://localhost:9870/webhdfs/v1/data/file.txt?op=CREATE"

# Read file via REST
curl \
"http://localhost:9870/webhdfs/v1/data/file.txt?op=OPEN"



SECTION 48 — ERASURE CODING COMMANDS (STORAGE OPTIMIZATION)
# Enable erasure coding policy
hdfs ec -enablePolicy -policy RS-6-3-1024k

# Set erasure coding on directory
hdfs ec -setPolicy -path /data \
-policy RS-6-3-1024k

# Get erasure coding policy
hdfs ec -getPolicy -path /data

# List available EC policies
hdfs ec -listPolicies

# Remove erasure coding policy
hdfs ec -unsetPolicy -path /data

⭐ Exam trap: Erasure Coding reduces storage compared to replication factor 3




SECTION 49 — ENCRYPTION ZONE COMMANDS (SECURITY)
# Create encryption key
hadoop key create mykey

# Create encryption zone
hdfs crypto -createZone \
-keyName mykey \
-path /securedata

# List encryption zones
hdfs crypto -listZones

# Get encryption info
hdfs crypto -getFileEncryptionInfo \
/securedata/file.txt
SECTION 50 — TRASH INTERVAL CONFIGURATION
# Set trash interval (minutes)
hdfs dfs -Dfs.trash.interval=60 -rm file.txt

# Expunge trash manually
hdfs dfs -expunge
SECTION 51 — BLOCK RECOVERY COMMANDS
# Recover corrupted block metadata
hdfs debug verifyMeta /data/file.txt

# Recover lease of file
hdfs debug recoverLease /data/file.txt

# Trigger block report
hdfs dfsadmin -triggerBlockReport
SECTION 52 — HDFS PERFORMANCE TUNING COMMANDS
# Show replication configuration
hdfs getconf -confKey dfs.replication

# Show block size configuration
hdfs getconf -confKey dfs.blocksize

# Increase replication factor
hdfs dfs -setrep 5 /data/file.txt

# Check cluster utilization
hdfs dfsadmin -report
SECTION 53 — SMALL FILE PROBLEM MANAGEMENT
# Merge small files
hdfs dfs -getmerge /smallfiles merged.txt

# Archive small files
hadoop archive \
-archiveName data.har \
-p /input /archive

⭐ Important concept: HAR = Hadoop Archive format







SECTION 54 — HDFS ARCHIVE COMMANDS (HAR FILE)
# Create HAR archive
hadoop archive \
-archiveName logs.har \
-p /logs /archive

# View HAR archive content
hdfs dfs -ls /archive/logs.har
SECTION 55 — SNAPSHOT ADVANCED OPERATIONS
# Allow snapshot creation
hdfs dfsadmin -allowSnapshot /data

# Create snapshot
hdfs dfs -createSnapshot /data snap1

# Compare snapshots
hdfs dfs -diff /data snap1 snap2

# Delete snapshot
hdfs dfs -deleteSnapshot /data snap1
SECTION 56 — HDFS CACHE MANAGEMENT COMMANDS
# Add cache directive
hdfs cacheadmin -addDirective \
-path /data \
-pool default

# Remove cache directive
hdfs cacheadmin -removeDirective 1

# List cache pools
hdfs cacheadmin -listPools





SECTION 57 — BALANCER ADVANCED OPTIONS
# Run balancer with bandwidth limit
hdfs balancer -bandwidth 1048576

# Run balancer with threshold
hdfs balancer -threshold 5
SECTION 58 — DATANODE BLOCK REPORT COMMANDS
# Trigger block report
hdfs dfsadmin -triggerBlockReport

# Refresh DataNode configuration
hdfs dfsadmin -refreshNodes
SECTION 59 — HDFS STORAGE TYPE COMMANDS
# Set storage policy HOT
hdfs storagepolicies \
-setStoragePolicy \
-path /data \
-policy HOT

# Set storage policy COLD
hdfs storagepolicies \
-setStoragePolicy \
-path /data \
-policy COLD




SECTION 60 — QUOTA REPORT COMMANDS
# Show quota usage
hdfs dfs -count -q /data

# Show space quota usage
hdfs dfs -count -h /data
SECTION 61 — JAVA API BASIC COMMANDS (HDFS PROGRAMMING)
# Import HDFS FileSystem class
import org.apache.hadoop.fs.FileSystem;

# Import Path class
import org.apache.hadoop.fs.Path;

# Create FileSystem object
FileSystem fs = FileSystem.get(conf);

# Open file
FSDataInputStream in = fs.open(new Path("file.txt"));

# Create file
FSDataOutputStream out =
fs.create(new Path("file.txt"));




SECTION 62 — JAVA API FILE OPERATIONS
# Check file exists
fs.exists(path);

# Delete file
fs.delete(path,true);

# Rename file
fs.rename(src,dst);

# Create directory
fs.mkdirs(path);



SECTION 63 — JAVA API DIRECTORY LISTING
# List directory content
FileStatus[] status = fs.listStatus(path);

# Get file block locations
fs.getFileBlockLocations(status,0,length);



SECTION 64 — JAVA API FILE COPY OPERATIONS
# Copy local file to HDFS
fs.copyFromLocalFile(src,dst);

# Copy HDFS file to local
fs.copyToLocalFile(src,dst);
SECTION 65 — WEB UI MONITORING COMMANDS
# Open NameNode UI
http://localhost:9870

# Open DataNode UI
http://localhost:9864

# View cluster summary
http://localhost:9870/dfshealth.html


SECTION 66 — HDFS LOG DEBUGGING COMMANDS
# View NameNode logs
tail -f $HADOOP_HOME/logs/*namenode*.log

# View DataNode logs
tail -f $HADOOP_HOME/logs/*datanode*.log

# View ResourceManager logs
tail -f $HADOOP_HOME/logs/*resourcemanager*.log



SECTION 67 — NAME NODE RECOVERY COMMANDS
# Save namespace
hdfs dfsadmin -saveNamespace

# Restore fsimage backup
cp fsimage_backup fsimage

# Restart NameNode
hdfs --daemon restart namenode
SECTION 68 — EDIT LOG OPERATIONS
# Convert edit logs to XML
hdfs oev -i edits -o edits.xml

# View edit logs
cat edits.xml





SECTION 69 — FEDERATION STATUS COMMANDS
# Show federation namespaces
hdfs getconf -namenodes

# Show nameservices
hdfs getconf -confKey dfs.nameservices
SECTION 70 — CLUSTER CONFIGURATION VALIDATION
# Validate configuration
hdfs getconf -confKey dfs.blocksize

# Check replication setting
hdfs getconf -confKey dfs.replication
