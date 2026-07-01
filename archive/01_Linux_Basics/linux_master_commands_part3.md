#### SECTION 46 — LVM (LOGICAL VOLUME MANAGER)


# Show physical volumes
pvdisplay

# Create physical volume
pvcreate /dev/sdb

# Show volume groups
vgdisplay

# Create volume group
vgcreate vg_data /dev/sdb

# Show logical volumes
lvdisplay

# Create logical volume
lvcreate -L 5G -n lv_data vg_data

# Format logical volume
mkfs.ext4 /dev/vg_data/lv_data

# Extend logical volume
lvextend -L +2G /dev/vg_data/lv_data

# Resize filesystem
resize2fs /dev/vg_data/lv_data

# Remove logical volume
lvremove /dev/vg_data/lv_data

#### SECTION 47 — ADVANCED DISK MONITORING

# Show disk usage summary
iostat

# Monitor disk IO continuously
iostat 2

# Show disk partitions
lsblk -f

# Show disk health (SMART)
smartctl -a /dev/sda

# Monitor disk activity
iotop


#### SECTION 48 — FIREWALL MANAGEMENT (UFW)


# Enable firewall
ufw enable

# Disable firewall
ufw disable

# Show firewall status
ufw status

# Allow SSH
ufw allow ssh

# Allow specific port
ufw allow 8080

# Deny port
ufw deny 21

# Delete firewall rule
ufw delete allow 8080


##### SECTION 49 — FIREWALL MANAGEMENT (IPTABLES)


# Show rules
iptables -L

# Allow traffic on port 80
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Block IP address
iptables -A INPUT -s 192.168.1.10 -j DROP

# Allow loopback interface
iptables -A INPUT -i lo -j ACCEPT

# Save rules
iptables-save

#### SECTION 50 — SSH CONFIGURATION & HARDENING

# Connect remote server
ssh user@host

# Connect using custom port
ssh -p 2222 user@host

# Generate SSH key
ssh-keygen

# Copy SSH key to server
ssh-copy-id user@host

# Edit SSH config
nano /etc/ssh/sshd_config

# Restart SSH service
systemctl restart ssh


#### SECTION 51 — SYSTEM PERFORMANCE MONITORING

# Real-time CPU usage
top

# Better process viewer
htop

# CPU statistics
mpstat

# Virtual memory stats
vmstat

# System load average
uptime


### SECTION 52 — MEMORY DEBUGGING


# Show memory usage
free -m

# Show detailed memory info
cat /proc/meminfo

# Monitor memory per process
top

# Track swap usage
swapon --show


### SECTION 53 — CPU INFORMATION & CONTROL

# Show CPU details
lscpu

# Monitor CPU usage
mpstat -P ALL

# Show CPU architecture
uname -p

# Limit CPU usage
cpulimit -p PID -l 50


#### SECTION 54 — SYSTEM CALL TRACING

# Trace system calls
strace ls

# Trace process by PID
strace -p PID

# Count system calls
strace -c ls
SECTION 55 — LIBRARY DEPENDENCY CHECKING
# Show shared libraries
ldd /bin/ls

# Show dynamic dependencies
ldd program_name


#### SECTION 56 — KERNEL MODULE MANAGEMENT


# Show loaded modules
lsmod

# Load module
modprobe module_name

# Remove module
modprobe -r module_name

# Module details
modinfo module_name


### SECTION 57 — SYSTEM BOOT ANALYSIS

# Show boot time
uptime

# Boot performance
systemd-analyze

# Show boot critical chain
systemd-analyze critical-chain

# View boot logs
journalctl -b

#### SECTION 58 — SERVICE DEPENDENCY MANAGEMENT

# List services
systemctl list-units --type=service

# Enable service
systemctl enable apache2

# Disable service
systemctl disable apache2

# Mask service
systemctl mask apache2
SECTION 59 — ADVANCED PROCESS FILTERING
# Search process
pgrep ssh

# Kill process by name
pkill apache2

# Show process tree
pstree

# Show thread-level info
ps -eLf


### SECTION 60 — NETWORK TRAFFIC ANALYSIS

# Monitor traffic
iftop

# Packet capture
tcpdump -i eth0

# Capture specific port
tcpdump port 80

# Save capture to file
tcpdump -w capture.pcap
SECTION 61 — DNS DEBUGGING
# DNS lookup
nslookup google.com

# Detailed DNS info
dig google.com

# Reverse lookup
dig -x 8.8.8.8


### SECTION 62 — ROUTING MANAGEMENT


# Show routing table
route -n

# Add route
ip route add default via 192.168.1.1

# Delete route
ip route del default

#### SECTION 63 — NFS (NETWORK FILE SYSTEM)

# Install NFS server
apt install nfs-kernel-server

# Export directory
nano /etc/exports

# Restart NFS
systemctl restart nfs-kernel-server

# Mount NFS share
mount server:/dir /mnt

#### SECTION 64 — SAMBA FILE SHARING

# Install samba
apt install samba

# Edit samba config
nano /etc/samba/smb.conf

# Restart samba
systemctl restart smbd

# Check samba status
systemctl status smbd

#### SECTION 65 — CRASH DEBUGGING

# Kernel logs
dmesg

# Monitor logs live
tail -f /var/log/syslog

# Check failed services
systemctl --failed

### SECTION 66 — USER LIMIT CONFIGURATION

# Edit limits config
nano /etc/security/limits.conf

# Show limits
ulimit -a

# Set open file limit
ulimit -n 4096
SECTION 67 — ENVIRONMENT DEBUGGING
# Show shell
echo $SHELL

# Show user variables
env

# Export variable permanently
nano ~/.bashrc


### SECTION 68 — TIME SYNCHRONIZATION (NTP)

# Install NTP
apt install ntp

# Restart NTP
systemctl restart ntp

# Show sync status
timedatectl status

#### SECTION 69 — DOCKER BASIC CONTROL (LINUX ADMIN BONUS)

# Check docker version
docker --version

# Pull image
docker pull ubuntu

# Run container
docker run ubuntu

# List containers
docker ps

# Stop container
docker stop container_id


#### SECTION 70 — SYSTEM RESOURCE LIMIT TRACKING

# Show resource usage summary
sar

# CPU stats history
sar -u

# Memory stats history
sar -r

# Disk stats history
sar -d


### SECTION 71 — FILE DESCRIPTOR MANAGEMENT

# Show open files
lsof

# Show process open files
lsof -p PID

# Show open port usage
lsof -i :80
SECTION 72 — SECURITY AUDITING
# Install audit daemon
apt install auditd

# Start audit service
systemctl start auditd

# Show audit logs
ausearch -m LOGIN

# Generate audit report
aureport

#### SECTION 73 — PASSWORD POLICY MANAGEMENT

# Edit password policy
nano /etc/login.defs

# Force password change
passwd -e username

# Lock user account
passwd -l username

# Unlock user account
passwd -u username
SECTION 74 — ADVANCED ARCHIVE CONTROL
# Create compressed tar.gz
tar -czvf backup.tar.gz folder

# Extract tar.gz
tar -xzvf backup.tar.gz

# Create bz2 archive
tar -cjvf backup.tar.bz2 folder

# Extract bz2 archive
tar -xjvf backup.tar.bz2

### SECTION 75 — SYSTEM INFORMATION (ENTERPRISE LEVEL)

# Hardware summary
lshw

# BIOS info
dmidecode

# Show uptime history
last reboot

# Kernel release info
uname -mrs
