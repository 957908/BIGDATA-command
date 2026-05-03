### SECTION 21 -- FILE PERMISSION(ADVANCED SYMBOLIC MODE)


# Add execute permission to user
chmod u+x script.sh

# Remove write permission from group
chmod g-w file.txt

# Add read permission to others
chmod o+r file.txt

# Assign exact permission
chmod u=rwx,g=rx,o=r file.txt

# Apply permission recursively
chmod -R 755 folder/

# Copy permission from another file
chmod --reference=file1 file2


### SECTION 22 -- OWNERSHIP MANGEMENT (ADVANCED)

# Change owner and group together
chown user:group file.txt

# Change owner recursively
chown -R user folder/

# Change group recursively
chgrp -R dev folder/

# Preserve ownership while copying
cp -p file.txt backup.txt

### SECTION 23 -- ADVANCED FILE SEARCHING

# Find files modified in last 24 hours
find / -mtime -1

# Find files older than 7 days
find / -mtime +7

# Find empty files
find / -empty

# Find executable files
find / -type f -executable

# Delete matched files automatically
find /tmp -name "*.log" -delete

# Execute command on found files
find . -name "*.txt" -exec rm {} \;


### SECTION 24 ---ADVANCED GREP FILTERING

# Match exact word
grep -w error logfile.txt

# Invert match
grep -v error logfile.txt

# Show only matched text
grep -o error logfile.txt

# Match multiple patterns
grep -E "error|warning" logfile.txt

# Recursive search with filename
grep -rn "password" /etc


### SECTION 25 -- SORTING & TEXT PROCESSING



# Sort alphabetically
sort file.txt

# Sort numerically
sort -n numbers.txt

# Reverse sort
sort -r file.txt

# Remove duplicate entries
uniq file.txt

# Count duplicate entries
uniq -c file.txt

# Combine sort and unique
sort file.txt | uniq

### SECTION 26 -- CUT COMMAND (COLUMN EXTRACTION)


# Extract first column
cut -d',' -f1 data.csv

# Extract multiple columns
cut -d',' -f1,3 data.csv

# Extract characters from position
cut -c1-5 file.txt

# Extract username from /etc/passwd
cut -d: -f1 /etc/passwd

### SECTION 27 -- AWK COMMAND (SUPER IMPORTANT)

# Print specific column
awk '{print $1}' file.txt

# Print multiple columns
awk '{print $1,$3}' file.txt

# Filter rows
awk '$3 > 50' marks.txt

# Sum column values
awk '{sum+=$1} END {print sum}' file.txt

# Print line number
awk '{print NR,$0}' file.txt

### SECTION 28 -- SED COMMAND(STREAM EDITOR)


# Replace word
sed 's/linux/unix/' file.txt

# Replace globally
sed 's/linux/unix/g' file.txt

# Delete line
sed '2d' file.txt

# Print specific line
sed -n '3p' file.txt

# Replace and save file
sed -i 's/error/fixed/g' file.txt


### SECTION 29 --- HARD LINK & SOFT LINK 


# Create hard link
ln file.txt hardlink.txt

# Create symbolic link
ln -s file.txt softlink.txt

# Remove symbolic link
rm softlink.txt

### SECTION 30 -- FILE ATTRIBUTE MANGEMENT

# Make file immutable
chattr +i file.txt

# Remove immutable flag
chattr -i file.txt

# View attributes
lsattr file.txt

### SECTION 31 --- DISK PARTITION COMMANDS

# Partition disk
fdisk /dev/sda

# Show partition table
fdisk -l

# Partition using parted
parted /dev/sda

# Display block devices
lsblk

# Show disk UUID
blkid

### SECTION 32 --FILESYSTEM CREATION

# Create ext4 filesystem
mkfs.ext4 /dev/sdb1

# Create ext3 filesystem
mkfs.ext3 /dev/sdb1

# Create xfs filesystem
mkfs.xfs /dev/sdb1

# Check filesystem
fsck /dev/sdb1

### SECTION 33 -- MOUNT MANGEMENT 

# Mount partition
mount /dev/sdb1 /mnt

# Unmount partition
umount /mnt

# Mount all filesystems
mount -a

# Show mounted devices
mount | column -t

# Temporary mount ISO
mount -o loop file.iso /mnt

### SECTION 34 -- SWAP MEMORY MANAGEMENT 


# Create swap file
fallocate -l 1G swapfile

# Set swap area
mkswap swapfile

# Enable swap
swapon swapfile

# Disable swap
swapoff swapfile

# Show swap usage
swapon --show


### SECTION 35 --- NETWORK INTERFACE MANGEMENT 


# Show interfaces
ip link show

# Enable interface
ip link set eth0 up

# Disable interface
ip link set eth0 down

# Assign IP address
ip addr add 192.168.1.10/24 dev eth0

# Remove IP address
ip addr del 192.168.1.10/24 dev eth0


### SECTION 36 -- ADVANCED NETWORK DEBUGGING

# Test connectivity
ping -c 4 google.com

# Show ARP table
arp -a

# Show open sockets
ss -tulnp

# Capture packets
tcpdump -i eth0

# Monitor bandwidth
iftop

### SECTION 37 -- USER SESSION MONITORING


# Show login history
last

# Show failed login attempts
lastb

# Show current users
who

# Show detailed session info
w


### SECTION 38 — JOB CONTROL COMMANDS


# Run job in background
sleep 100 &

# List background jobs
jobs

# Bring job to foreground
fg %1

# Stop foreground job
Ctrl+Z


#### SECTION 39 — PROCESS PRIORITY MANAGEMENT

# Start process with priority
nice -n 10 processname

# Change running process priority
renice 5 PID

# Show process tree
pstree

#### SECTION 40 — PACKAGE MANAGEMENT (RPM SYSTEMS)

# Install package
rpm -ivh package.rpm

# Remove package
rpm -e package

# Query installed packages
rpm -qa

# Package info
rpm -qi package


### SECTION 41 — LOG ANALYSIS COMMANDS


# Show kernel logs
dmesg

# View boot logs
journalctl -b

# View service logs
journalctl -u ssh

# Live log monitoring
journalctl -f

### SECTION 42 — ENVIRONMENT CONFIGURATION FILES

# Edit bash config
nano ~/.bashrc

# Reload bash config
source ~/.bashrc

# System-wide environment variables
nano /etc/environment


### SECTION 43 — SYSTEM TIME MANAGEMENT


# Show current date
date

# Set system date
date -s "2026-01-01"

# Show hardware clock
hwclock

# Sync hardware clock
hwclock --systohc


#### SECTION 44 — HOST MANAGEMENT


# Edit hosts file
nano /etc/hosts

# Show hostname
hostname

# Change hostname temporarily
hostname newname


### SECTION 45 — SHUTDOWN & POWER CONTROL (ADVANCED)


# Shutdown immediately
shutdown now

# Shutdown after 5 minutes
shutdown +5

# Restart system
reboot

# Power off system
poweroff





