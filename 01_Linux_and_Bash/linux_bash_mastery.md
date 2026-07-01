# 🐧 Module 01: Linux & Bash Mastery for Big Data

To successfully manage a distributed cluster containing hundreds of nodes, you must master the Linux operating system, resource limits, network troubleshooting, text parsing, and shell automation. Distributed frameworks (Hadoop, Spark, Kafka) spawn thousands of concurrent threads and open thousands of files, making OS-level configuration critical.

---

## 1. Operating System Resource Limits (`ulimit`)

Big Data services run as user processes, which Linux limits by default to prevent a single process from hogging resources. In production clusters, default limits will cause "Too many open files" or "Out of memory (Unable to create new native thread)" errors.

### Configuring `/etc/security/limits.conf`
To permanently increase limits for Hadoop/Spark users, edit `/etc/security/limits.conf` as `root`:

```text
# /etc/security/limits.conf
# <domain>    <type>    <item>        <value>
hdfs          soft      nofile        65536
hdfs          hard      nofile        65536
hdfs          soft      nproc         32768
hdfs          hard      nproc         32768
spark         soft      nofile        65536
spark         hard      nofile        65536
spark         soft      nproc         32768
spark         hard      nproc         32768
```

### Explaining the Limits:
* **`nofile`**: Maximum number of open file descriptors. A NameNode or Kafka broker opens index files, socket connections, and log files concurrently. Default is usually `1024`, which is too low.
* **`nproc`**: Maximum number of processes/threads. Spark executors spawn thread pools. A low limit prevents thread creation.
* **`soft`**: The threshold warning limit that users can change up to the hard limit.
* **`hard`**: The absolute maximum limit enforced by the kernel.

### Check Current Limits on a Node:
```bash
# Check maximum open files limit for current user
ulimit -n

# Check maximum user processes limit
ulimit -u

# Check all limits (including stack size, memory locks, core dump size)
ulimit -a
```

---

## 2. Complete Linux Command Reference Library

This library lists the exact commands and flags essential for managing and troubleshooting nodes in an enterprise Big Data environment.

### A. Navigation & Advanced File Operations
* **`ls`**: Lists directory contents.
  * `ls -la`: Show all files (including hidden ones starting with `.`) in list format with size, permissions, and owners.
  * `ls -lh`: Show file sizes in human-readable formats (e.g., KB, MB, GB).
  * `ls -lt`: Sort files by modification time (newest first). Great for finding recent log entries.
  * `ls -lrS`: Sort files by size in reverse order (largest last).
* **`find`**: Searches for files in a directory hierarchy.
  * `find . -name "*.log"`: Find files matching wildcard pattern.
  * `find /var/log -type f -size +100M`: Find files larger than 100 megabytes.
  * `find /app/hadoop -mtime -2`: Find files modified within the last 2 days.
  * `find . -type f -perm 777`: Find files with insecure permissions.
  * `find /var/log/spark -name "*.log" -exec gzip {} \;`: Compress all logs found in Spark log directory.
* **`xargs`**: Builds and executes command lines from standard input.
  * `find . -name "*.tmp" | xargs rm -f`: Efficiently delete thousands of temporary files.
* **`tar`**: Archive utility.
  * `tar -cvzf logs_backup.tar.gz /var/log/hadoop`: Create a compressed gzip tarball.
  * `tar -xvzf logs_backup.tar.gz -C /tmp`: Extract tarball to a specific directory.
* **`rsync`**: Fast, incremental file transfer tool.
  * `rsync -avz /local/data/ user@remote-node:/remote/data/`: Sync directories over SSH with compression and verbose details.

### B. High-Performance Text Processing & Log Parsing
* **`grep`**: Pattern matching utility.
  * `grep "OutOfMemory" /var/log/spark/*.log`: Search for errors in log files.
  * `grep -i "error" hive-cli.log`: Case-insensitive search.
  * `grep -rn "Failed" /var/log/hadoop/`: Recursive search showing line numbers.
  * `grep -c "Exception" app.log`: Count occurrences of a word.
  * `grep -A 5 -B 2 "NullPointerException" app.log`: Print 5 lines after (After) and 2 lines before (Before) the match.
* **`awk`**: Pattern scanning and processing language.
  * `awk '{print $1}' access.log`: Print the first column (e.g., IP addresses) of a space-delimited log.
  * `awk -F',' '{if($3 > 50000) print $1, $3}' emp.csv`: Read CSV files (comma delimiter) and filter records based on value comparisons.
  * `awk '/ERROR/ {count++} END {print count}' spark.log`: Scan log for matching string and output cumulative count.
* **`sed`**: Stream editor for filtering and transforming text.
  * `sed -i 's/localhost/namenode-host/g' core-site.xml`: Inline search-and-replace string in configurations.
  * `sed -n '10,20p' app.log`: Print lines 10 to 20 of a file.
* **`sort` & `uniq`**:
  * `sort -k3 -n -r data.txt`: Sort data by third column numerically in reverse order.
  * `uniq -c`: Count consecutive duplicate lines.
  * *Combination*: `awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 10` (Finds the top 10 IP addresses hitting your server).
* **`cut`**: Remove sections from each line of files.
  * `cut -d',' -f1,3 data.csv`: Extract fields 1 and 3 separated by commas.

### C. System Performance & Disk Monitoring
* **`df`**: Reports file system disk space usage.
  * `df -h`: Human-readable format.
  * `df -i`: Show inode usage. If inodes are 100% full, you cannot create new files even if you have terabytes of free space.
* **`du`**: Estimates file space usage.
  * `du -sh /var/log/*`: Display total size of directories.
  * `du -ah /var/log | sort -rh | head -n 10`: Find the top 10 largest files in log directory.
* **`free`**: Displays amount of free and used memory in the system.
  * `free -h`: Human-readable statistics.
  * `free -m`: Displays stats in megabytes. Focus on the `available` column rather than `free`, as Linux uses unused RAM for file caching.
* **`top` / `htop`**: Real-time display of process resource usage.
  * Press `M` in `top` to sort processes by memory usage.
  * Press `P` in `top` to sort by CPU usage.
* **`iostat`**: Report Central Processing Unit (CPU) statistics and input/output statistics for devices.
  * `iostat -xz 1 5`: Show extended disk I/O metrics every second, 5 times. High `%util` indicates disk bottleneck.
* **`vmstat`**: Reports virtual memory statistics.
  * `vmstat 1 5`: Inspect page swapping (`si`/`so` columns). If swapping is high, the node is out of physical memory.

### D. Network Analysis & Port Diagnostics
* **`netstat` / `ss`**: Print network connections, routing tables, and interface statistics.
  * `ss -tulnp`: Display all listening TCP/UDP ports with process names and PIDs.
* **`lsof`**: List open files.
  * `lsof -i :8088`: Find the process ID listening on port 8088 (YARN ResourceManager UI).
* **`nc` (netcat)**: Arbitrary data transmission utility.
  * `nc -zv 192.168.1.100 9000`: Scan if port 9000 (HDFS NameNode RPC) is open and reachable.
* **`ping` & `traceroute`**:
  * `ping -c 5 remote-host`: Test basic network connection.
  * `traceroute remote-host`: Trace network routing path hops.
* **`curl` & `wget`**:
  * `curl -I http://localhost:50070`: Test connection and retrieve HTTP headers from NameNode UI.
  * `wget http://repo.maven.org/spark-core.jar`: Download binaries directly on cluster nodes.

### E. User, Group & Permission Management
* **`chmod`**: Change file mode bits.
  * `chmod 755 script.sh`: Read/Write/Execute for owner, Read/Execute for group and others.
  * `chmod +t /tmp/shared`: Apply sticky bit.
* **`chown`**: Change file owner and group.
  * `chown -R hdfs:hadoop /hadoop/data`: Change owner and group recursively.
* **`useradd` / `usermod` / `userdel`**:
  * `sudo useradd -m -g hadoop spark`: Create user 'spark' belonging to primary group 'hadoop'.
  * `sudo usermod -aG wheels user`: Append user to secondary group 'wheels'.
* **`visudo`**: Safely edit the sudoers file.
  * Add `spark ALL=(ALL) NOPASSWD: ALL` to allow running commands as root without passwords.

---

## 3. SSH Configuration & Passwordless Login

Daemons like Hadoop start scripts (`start-dfs.sh`) that SSH into every worker node listed in the `workers` file. Passwordless login is mandatory.

### Setting Up Passwordless SSH:
1. **Generate SSH Key Pair** on the master node:
   ```bash
   # Generates 4096-bit RSA key pair without passphrase
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
   ```
2. **Copy the Public Key** to all worker nodes:
   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub user@worker-node-1
   ssh-copy-id -i ~/.ssh/id_rsa.pub user@worker-node-2
   ```
3. **Configure SSH Client** (`~/.ssh/config`) for easier navigation and custom ports:
   ```text
   # ~/.ssh/config
   Host worker1
       HostName 192.168.1.101
       User sparkadmin
       Port 22
       IdentityFile ~/.ssh/id_rsa

   Host worker2
       HostName 192.168.1.102
       User sparkadmin
       Port 22
       IdentityFile ~/.ssh/id_rsa
   ```
4. **Test SSH connection**:
   ```bash
   ssh worker1
   ```

---

## 4. Advanced Bash Scripting for Cluster Automation

Writing clean shell scripts is essential for scheduling tasks, validating environments, and cleaning logs.

### Master Script Structure: Log Cleanup & Health Daemon
This production-grade script runs daily to check Spark/Hadoop log directories, delete logs older than 7 days, and alert if disk usage exceeds 90%.

```bash
#!/usr/bin/env bash

# Set strict mode: exit immediately if a command fails, or if using unset variables
set -euo pipefail

# Configurations
LOG_DIR="/var/log/spark"
BACKUP_DIR="/var/log/spark/archive"
THRESHOLD_PERCENT=90
EMAIL_ALERT="admin@datacompany.com"

# Logging function
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') [INFO] - $1"
}

log_error() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') [ERROR] - $1" >&2
}

# 1. Check Disk Space Usage
check_disk_usage() {
    log_message "Checking disk space usage..."
    # Extracts the partition percentage of the directory partition
    local disk_usage
    disk_usage=$(df -h "$LOG_DIR" | awk 'NR==2 {print $5}' | sed 's/%//')
    
    if [ "$disk_usage" -gt "$THRESHOLD_PERCENT" ]; then
        log_error "Disk space usage has exceeded threshold! Current usage: ${disk_usage}%"
        # Alert mechanism placeholder
        # mail -s "Disk Space Alert: ${disk_usage}%" "$EMAIL_ALERT" <<< "Disk space warning on node $(hostname)"
    else
        log_message "Disk space usage is healthy: ${disk_usage}%"
    fi
}

# 2. Archive and Purge Old Logs
clean_logs() {
    log_message "Purging files older than 7 days from $LOG_DIR..."
    
    # Ensure backup directory exists
    mkdir -p "$BACKUP_DIR"
    
    # Compress logs older than 3 days
    find "$LOG_DIR" -maxdepth 1 -name "*.log" -mtime +3 -exec gzip {} \;
    
    # Delete compressed logs older than 7 days
    find "$LOG_DIR" -maxdepth 1 -name "*.gz" -mtime +7 -delete
    
    log_message "Log cleanup completed successfully."
}

# Main Execution block
main() {
    log_message "Starting log maintenance tasks..."
    check_disk_usage
    clean_logs
    log_message "Maintenance completed."
}

# Execute main function
main
```

---

## 5. Enterprise Job Interview Q&A (Linux & Bash)

This section prepares you for production-level interview questions.

### Q1: How do you identify a process that is causing a memory leak on a cluster node, and how do you terminate it safely?
* **How to explain this to the interviewer**:
  Start by stating the tool you would use to identify the process (`top` or `htop`), explain how to sort the output to pinpoint the culprit, and detail the step-by-step transition from safe termination signals to forced signals. Do not just say "I would run `kill -9`."

* **Model Answer**:
  "To identify the memory-leaking process, I would log into the host node and run `top` or `htop`. In `top`, I would press `Shift + M` to sort all running processes by physical memory usage (`%MEM`). 
  
  Once the process ID (PID) is identified, I check its type. If it is a JVM application (like Hadoop or Spark), I run `jps -lv` to retrieve the JVM arguments and identify the class name.
  
  To terminate it:
  1. I start with a graceful termination request using **`kill -15 <PID>` (SIGTERM)**. This allows the application to trigger its JVM shutdown hooks, flush transient memory buffers to disk, release file handles, and close socket connections.
  2. If the process does not shut down within a reasonable timeout, it indicates the process is hung in an IO state or deadlock. I will then execute **`kill -9 <PID>` (SIGKILL)** to force the operating system kernel to clean up the process resources immediately."

---

### Q2: What is the difference between hard limits and soft limits in `ulimit`, and how does it impact a running Spark cluster?
* **How to explain this to the interviewer**:
  Explain that `ulimit` controls resource thresholds for processes started by users. Define soft limits as warnings/changeable boundaries, and hard limits as absolute maximum bounds set by the administrator. Link this directly to a common Spark runtime failure.

* **Model Answer**:
  "The **soft limit** is the current value enforced by the operating system for a resource (like open files or processes). Users can increase their soft limits up to the threshold of the **hard limit** without admin permissions. The **hard limit** is the absolute ceiling set by the system administrator (root) in `/etc/security/limits.conf`.
  
  In a Spark cluster, executors spawn hundreds of concurrent worker threads and stream blocks. If the `nproc` limit is set to a low default (e.g., `1024`), the executor JVM will fail when trying to spawn a thread pool thread, throwing `java.lang.OutOfMemoryError: unable to create new native thread`. 
  
  Similarly, if `nofile` is too low, when Spark opens connections to copy partition blocks during shuffles, it fails with `java.io.FileNotFoundException (Too many open files)`. In production, both values must be permanently increased to at least `32768` (processes) and `65536` (file descriptors)."

---

### Q3: How do you troubleshoot a connection timeout error between a Spark Driver on node A and its Executors on node B?
* **How to explain this to the interviewer**:
  Establish a systematic troubleshooting hierarchy: Physical Link/DNS -> Port Binding -> Firewall Rules -> Application Configuration.

* **Model Answer**:
  "I troubleshoot network connectivity timeouts step-by-step:
  1. **Ping Test**: Run `ping -c 5 node-b` from Node A to ensure the physical link is active and DNS hostnames resolve to the correct IP addresses.
  2. **Port Binding Check**: On Node B, I run `ss -tulnp | grep <port>` or `lsof -i :<port>` to verify the Executor daemon is actually running and listening on the expected port (not bound only to the local loopback `127.0.0.1`).
  3. **Port Reachability**: From Node A, I use netcat (`nc -zv Node-B <port>`) to check if the port is reachable. If the ping succeeds but netcat hangs or returns connection refused, it indicates a firewall block.
  4. **Firewall Verification**: I check local firewall rules on both hosts using `sudo ufw status` (Ubuntu) or `sudo firewall-cmd --list-all` (CentOS/RedHat) and ensure security groups permit communication on the Spark port ranges."
