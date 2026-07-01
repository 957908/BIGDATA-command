### SECTION 1 - BASIC NAVIGATION COMMANDS



# Show current directory path
pwd

# List files in directory
ls

# List hidden files
ls -a

# List detailed file info
ls -l

# List files with human readable size
ls -lh

# List files recursively
ls -R

# Change directory
cd foldername

# Go to home directory
cd ~

# Go to parent directory
cd ..

# Go to previous directory
cd -

# Clear terminal screen
clear

# Display hostname
hostname

# Display current logged user
whoami

# Show logged users
who

# Show system info
uname

# Show kernel version
uname -r

# Show complete system info
uname -a






#### SECTION 2 - FILE MANGEMENT COMMANDS


# Create empty file
touch file.txt

# Create multiple files
touch file1 file2 file3

# Copy file
cp file.txt backup.txt

# Copy directory
cp -r folder backup/

# Move file
mv file.txt newfile.txt

# Move directory
mv dir1 dir2

# Delete file
rm file.txt

# Delete multiple files
rm file1 file2

# Delete directory
rm -r folder

# Force delete directory
rm -rf folder

# Show file content
cat file.txt

# Show first 10 lines
head file.txt

# Show last 10 lines
tail file.txt

# Show live file updates
tail -f logfile.log

# Count lines words characters
wc file.txt




### SECTION 3 - DIRECTORY MANAGEMENT


# Create directory
mkdir test

# Create multiple directories
mkdir dir1 dir2

# Create nested directory
mkdir -p a/b/c

# Remove empty directory
rmdir test

# Remove directory recursively
rm -r folder

# Copy directory
cp -r source destination

# Move directory
mv olddir newdir





### FILE VIEWING & EDITING 


# View file page by page
less file.txt

# View file quickly
more file.txt

# Open file editor
nano file.txt

# Open vi editor
vi file.txt

# Open vim editor
vim file.txt

# Show file type
file file.txt

# Compare two files
diff file1 file2

# Merge sorted files
sort file.txt

# Remove duplicate lines
uniq file.txt


### SECTION 4 -- FILE VIEWING & EDITING


# View file page by page
less file.txt

# View file quickly
more file.txt

# Open file editor
nano file.txt

# Open vi editor
vi file.txt

# Open vim editor
vim file.txt

# Show file type
file file.txt

# Compare two files
diff file1 file2

# Merge sorted files
sort file.txt

# Remove duplicate lines
uniq file.txt

SECTION 5 --SEARCH COMMANDS

# Search file
find . -name file.txt

# Search directory
find /home -type d

# Search by size
find / -size +100M

# Search by permission
find / -perm 777

# Search text inside file
grep word file.txt

# Case insensitive search
grep -i word file.txt

# Recursive search
grep -r word folder

# Show line number
grep -n word file.txt

# Count matches
grep -c word file.txt

### SECTION 6 -- PERMISSION COMMANDS

# Change file permission
chmod 777 file.txt

# Read only permission
chmod 444 file.txt

# Execute permission
chmod +x script.sh

# Change owner
chown user file.txt

# Change group
chgrp group file.txt

# Change owner recursively
chown -R user folder


### SECTION 7 -- USER MANAGEMENT 



# Add new user
useradd username

# Set password
passwd username

# Delete user
userdel username

# Delete user with home
userdel -r username

# Modify user
usermod username

# Switch user
su username

# Switch root user
sudo su

# Show user id
id username


### SECTION 8 -- GROUP MANAGEMENT

# Create group
groupadd dev

# Delete group
groupdel dev

# Modify group
groupmod dev

# Add user to group
usermod -aG dev user


### SECTION 9 -- PROCESS MANGEMENT

# Show running processes
ps

# Detailed process list
ps -ef

# Real time processes
top

# Enhanced top
htop

# Kill process
kill PID

# Force kill process
kill -9 PID

# Kill by name
pkill processname

# Background process
command &

# Foreground process
fg


### SECTION 10--DISK MANAGEMENT

# Show disk usage
df

# Human readable disk usage
df -h

# Directory size
du

# Directory size human readable
du -h

# Total directory size
du -sh folder

# Show mounted disks
mount

# Unmount disk
umount /dev/sdb1

### SECTION 11--NETWORK COMMANDS


# Show IP address
ip addr

# Show routing table
ip route

# Ping server
ping google.com

# Check open ports
netstat -tuln

# Show network connections
ss

# DNS lookup
nslookup google.com

# Trace route
traceroute google.com

# Download file
wget url

# Transfer file
scp file user@host:/path

#### SECTION  12 -- ARCHIVE & COMPRESSION

# Create tar archive
tar -cvf file.tar folder

# Extract tar
tar -xvf file.tar

# Create gzip file
gzip file.txt

# Extract gzip
gunzip file.txt.gz

# Create zip
zip file.zip file.txt

# Extract zip
unzip file.zip


### SECTION 13 -- SYSTEM MONITORING


# System uptime
uptime

# Memory usage
free

# Human readable memory
free -h

# CPU info
lscpu

# Block devices
lsblk

# PCI devices
lspci

# USB devices
lsusb

# Kernel messages
dmesg


### SECTION 14 --- PACKAGE MANGEMENT(apt)


# Update packages
apt update

# Upgrade packages
apt upgrade

# Install package
apt install nginx

# Remove package
apt remove nginx

# Search package
apt search nginx


### SECTION 15 -- SERVICE MANAGEMENT


# Start service
systemctl start nginx

# Stop service
systemctl stop nginx

# Restart service
systemctl restart nginx

# Enable service
systemctl enable nginx

# Disable service
systemctl disable nginx

# Check status
systemctl status nginx

### SECTION 16 -- ENVIRONMENT VARIABLES


# Show variables
printenv

# Show PATH
echo $PATH

# Set variable
export VAR=value

# Remove variable
unset VAR



#### SECTION 17 --- FILE TRANSFER)(ADNANCED)
# Remote login
ssh user@host

# Secure copy
scp file user@host:/dir

# Sync directories
rsync -av source dest

# Download via curl
curl url

#### SECTION 18 ---CRON JOBS (AUTOMATION)

# Edit cron jobs
crontab -e

# List cron jobs
crontab -l

# Remove cron jobs
crontab -r

# Run every minute
* * * * * command
       



### SECTION 19 --LOG MANAGEMENT

# View system log
cat /var/log/syslog

# View auth log
cat /var/log/auth.log

# Live monitoring
tail -f /var/log/syslog



### SECTION 20 -- SUPER ADVANCED ADMIN COMMANDSS


# Change hostname
hostnamectl set-hostname newname

# Show SELinux status
sestatus

# Change runlevel
init 3

# Shutdown system
shutdown now

# Restart system
reboot

# Schedule shutdown
shutdown +10

