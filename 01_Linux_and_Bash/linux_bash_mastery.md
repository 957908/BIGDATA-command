# 🐧 Module 01: Linux & Bash Mastery for Big Data

To successfully manage a distributed cluster containing hundreds of nodes, you must master the Linux operating system, resource limits, network troubleshooting, and shell automation. Distributed frameworks (Hadoop, Spark, Kafka) spawn thousands of concurrent threads and open thousands of files, making OS-level configuration critical.

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

# Check all limits
ulimit -a
```

---

## 2. Linux Permissions & User/Group Management

Multi-tenant clusters require strict permission controls to protect data files in HDFS and local directories.

### Common CLI commands:
```bash
# Create group for data engineers
sudo groupadd dataengineers

# Add user 'sparkadmin' to groups 'sparkadmin' and secondary group 'dataengineers'
sudo usermod -aG dataengineers sparkadmin

# Change ownership of Spark log directory (Owner: sparkadmin, Group: dataengineers)
sudo chown -R sparkadmin:dataengineers /var/log/spark

# Set permissions: Owner (Read/Write/Execute), Group (Read/Execute), Others (None)
chmod 750 /var/log/spark

# Set SetUID/SetGID/Sticky Bit
# Sticky Bit (t) ensures only the owner can delete files inside a shared directory (e.g., /tmp)
chmod +t /tmp/shared_data
```

### Permission Representation Matrix:
| Octal | Binary | File Permissions | Description |
| :--- | :--- | :--- | :--- |
| `7` | `111` | `rwx` | Read, Write, and Execute |
| `6` | `110` | `rw-` | Read and Write |
| `5` | `101` | `r-x` | Read and Execute |
| `4` | `100` | `r--` | Read Only |
| `0` | `000` | `---` | No Permissions |

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

## 4. Process Management & Signal Handling

Ecosystem services run as background JVM processes. You must know how to trace them, inspect resource consumption, and shut them down cleanly.

### Inspecting Processes:
```bash
# Find JVM process IDs running on the machine
jps -lm

# List all running processes with user, PID, CPU, Memory, and Command details
ps aux | grep java

# Monitor live CPU/Memory consumption per process
top -b -n 1 | head -n 20
# Better interactive UI
htop
```

### Signal Handling (`kill`):
When shutting down services, always use soft signals before resorting to force.
* **`kill -15 <PID>` (SIGTERM)**: Request clean shutdown. The JVM catches this signal, runs its registered shutdown hooks, flushes metadata to disk, closes open files, and exits cleanly.
* **`kill -9 <PID>` (SIGKILL)**: Force kill. The kernel terminates the process immediately. The JVM cannot run shutdown hooks, which can corrupt local transaction logs or database files.
* **`kill -3 <PID>` (SIGQUIT)**: Triggers the JVM to dump a full thread stack trace to standard output. Extremely useful for debugging deadlocks and hung applications.

---

## 5. Network Analysis & Port Troubleshooting

Distributed daemons listen on specific TCP ports. A primary task in clustering is opening firewall ports and verifying inter-node communication.

### Verifying Listening Ports:
```bash
# Show listening TCP ports with PIDs and process names
sudo netstat -tulnp

# Modern alternative using socket statistics (ss)
ss -tulnp

# Find which process is occupying port 8088 (YARN ResourceManager UI)
lsof -i :8088
```

### Checking Inter-Node Connections:
```bash
# Verify network path and measure round-trip latency
ping -c 5 192.168.1.101

# Test if port 9000 (HDFS NameNode RPC) is reachable on master node
nc -zv master-node 9000

# Inspect routing path to worker node
traceroute worker1
```

---

## 6. Advanced Bash Scripting for Cluster Automation

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

## 7. System Monitoring & Kernel Diagnostics

When nodes behave erratically (e.g., losing connection or suddenly slowing down), check system statistics and kernel buffers.

```bash
# View kernel ring buffer messages. Look for OOM (Out of Memory) kills.
dmesg -T | grep -i -E "oom|kill"

# Check system memory usage in human-readable megabytes/gigabytes
free -h

# Check swap file usage and activity
swappiness=$(cat /proc/sys/vm/swappiness)
echo "Swappiness value is: $swappiness" # High values (e.g. 60) force swapping. Keep to 10 or 0 for cluster nodes to prevent swapping executor RAM to disk.

# Monitor I/O performance on disks (requires sysstat package)
# Displays disk read/write rates and await (wait time for I/O request in ms)
iostat -xz 1 5

# Monitor system stats (processes, memory, paging, CPU activity)
vmstat 1 5
```

---

## 🎯 Exam and Interview Traps

1. **Trap: Why did my Spark application crash with an OOM error, but the logs show `java.lang.OutOfMemoryError: unable to create new native thread`?**
   * **Answer**: This is rarely a JVM heap issue. It usually means the Linux user running Spark exceeded the `nproc` limit defined in `/etc/security/limits.conf`. The OS prevented the JVM from spawning a new native thread. Set `nproc` to a higher value like `32768`.

2. **Trap: Why is it bad to use `kill -9` to stop a NameNode or Kafka Broker?**
   * **Answer**: `kill -9` stops the process instantly, meaning the service cannot flush transaction journals or state changes to disk. For NameNode, it might corrupt the `edit logs`. For Kafka, it leaves the partition state out-of-sync, forcing a long recovery scan of index files during startup. Always use `kill -15` (SIGTERM).

3. **Trap: Why does network performance degrade on worker nodes even when CPU usage is low?**
   * **Answer**: The OS may be swapping memory pages to disk because the system RAM is full. Check this with `free -h` and `vmstat`. Ensure `vm.swappiness` is set to `10` or less in `/etc/sysctl.conf`.
