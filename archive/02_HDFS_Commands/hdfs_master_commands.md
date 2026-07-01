#### 🧱 SECTION 1 — BASIC HDFS COMMANDS (BEGINNER LEVEL)

# List files in HDFS root directory
hdfs dfs -ls /

# List files in specific directory
hdfs dfs -ls /user

# List recursively
hdfs dfs -ls -R /user

# Create directory in HDFS
hdfs dfs -mkdir /data

# Create nested directory
hdfs dfs -mkdir -p /project/input

# Upload file from local to HDFS
hdfs dfs -put file.txt /data

# Upload multiple files
hdfs dfs -put file1 file2 /data

# Upload directory
hdfs dfs -put folder /data

# Alternative upload command
hdfs dfs -copyFromLocal file.txt /data

# Download file from HDFS
hdfs dfs -get /data/file.txt

# Download directory
hdfs dfs -get /data/folder

# Alternative download command
hdfs dfs -copyToLocal /data/file.txt


#### 📂 SECTION 2 — FILE VIEWING COMMANDS

# Display file content
hdfs dfs -cat /data/file.txt

# View first few lines
hdfs dfs -head /data/file.txt

# View last few lines
hdfs dfs -tail /data/file.txt

# Stream file content
hdfs dfs -text /data/file.txt

# Display compressed file content
hdfs dfs -text file.gz


### 📦 SECTION 3 — FILE CREATION COMMANDS

# Create empty file
hdfs dfs -touchz /data/empty.txt

# Append content to file
hdfs dfs -appendToFile local.txt /data/file.txt

# Merge multiple files
hdfs dfs -getmerge /input output.txt
🗑 SECTION 4 — DELETE COMMANDS
# Delete file
hdfs dfs -rm /data/file.txt

# Delete directory recursively
hdfs dfs -rm -r /data

# Force delete
hdfs dfs -rm -r -f /data

# Skip trash while deleting
hdfs dfs -rm -skipTrash /data/file.txt


#### 📋 SECTION 5 — COPY & MOVE COMMANDS

# Copy file inside HDFS
hdfs dfs -cp /data/file.txt /backup

# Move file
hdfs dfs -mv /data/file.txt /backup

# Rename file
hdfs dfs -mv old.txt new.txt

# Move directory
hdfs dfs -mv /data /archive


### 📊 SECTION 6 — DISK USAGE COMMANDS

# Show disk usage
hdfs dfs -du /data

# Human readable disk usage
hdfs dfs -du -h /data

# Show total directory size
hdfs dfs -dus /data

# Show filesystem capacity
hdfs dfs -df -h


#### 🔍 SECTION 7 — FILE COUNT COMMANDS

# Count directories, files, bytes
hdfs dfs -count /

# Count specific directory
hdfs dfs -count /data


### 🔐 SECTION 8 — PERMISSION COMMANDS


# Change file permission
hdfs dfs -chmod 755 /data/file.txt

# Change permission recursively
hdfs dfs -chmod -R 777 /data

# Change owner
hdfs dfs -chown user /data/file.txt

# Change owner with group
hdfs dfs -chown user:group /data/file.txt

# Change group
hdfs dfs -chgrp group /data/file.txt

### 🔗 SECTION 9 — FILE STATUS COMMANDS

# Show file info
hdfs dfs -stat /data/file.txt

# Show block size
hdfs dfs -stat %b /data/file.txt

# Show filename only
hdfs dfs -stat %n /data/file.txt


### 📁 SECTION 10 — DIRECTORY MANAGEMENT COMMANDS

# Create directory
hdfs dfs -mkdir /input

# Create nested directory
hdfs dfs -mkdir -p /input/raw/logs

# Remove empty directory
hdfs dfs -rmdir /input
📄 SECTION 11 — FILE TEST COMMANDS
# Check file exists
hdfs dfs -test -e /data/file.txt

# Check if directory exists
hdfs dfs -test -d /data

# Check if file size > 0
hdfs dfs -test -s /data/file.txt


### 📦 SECTION 12 — ARCHIVE COMMANDS

# Copy file preserving metadata
hdfs dfs -cp -p file1 file2

# Merge small files
hdfs dfs -getmerge /smallfiles merged.txt

### 📈 SECTION 13 — REPLICATION MANAGEMENT


# Change replication factor
hdfs dfs -setrep 3 /data/file.txt

# Change replication recursively
hdfs dfs -setrep -R 2 /data

# Wait until replication completes
hdfs dfs -setrep -w 3 /data/file.txt

⭐ Exam trap: default replication factor = 3

### 🧪 SECTION 14 — CHECKSUM COMMANDS
# Verify checksum
hdfs dfs -checksum /data/file.txt

### SECTION 15 — FILE CONCAT COMMANDS
# Concatenate files
hdfs dfs -concat targetfile sourcefile

#### 🔍 SECTION 16 — SNAPSHOT COMMANDS


# Enable snapshot directory
hdfs dfsadmin -allowSnapshot /data

# Create snapshot
hdfs dfs -createSnapshot /data snap1

# Delete snapshot
hdfs dfs -deleteSnapshot /data snap1

### ⚙️ SECTION 17 — SAFE MODE COMMANDS

# Enter safe mode
hdfs dfsadmin -safemode enter

# Leave safe mode
hdfs dfsadmin -safemode leave

# Check safe mode status
hdfs dfsadmin -safemode get
📡 SECTION 18 — DATANODE REPORT COMMANDS
# Show cluster report
hdfs dfsadmin -report

# Show live datanodes
hdfs dfsadmin -report | grep Live

# Show dead datanodes
hdfs dfsadmin -report | grep Dead

### SECTION 19 — BALANCER COMMANDS

# Start balancer
hdfs balancer

# Run balancer with threshold
hdfs balancer -threshold 5


###🔍 SECTION 20 — FSCK COMMANDS (VERY IMPORTANT)


# Check filesystem health
hdfs fsck /

# Check block details
hdfs fsck / -files -blocks

# Check corrupted blocks
hdfs fsck / -list-corruptfileblocks
