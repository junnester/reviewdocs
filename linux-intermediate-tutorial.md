# Linux Intermediate Tutorial

## Table of Contents
1. [Advanced File Management](#advanced-file-management)
2. [Advanced Permissions and Access Control](#advanced-permissions-and-access-control)
3. [System Administration](#system-administration)
4. [Storage and Disk Management](#storage-and-disk-management)
5. [Advanced Networking](#advanced-networking)
6. [System Monitoring and Logging](#system-monitoring-and-logging)
7. [Advanced Shell Scripting](#advanced-shell-scripting)
8. [Software Installation and Compilation](#software-installation-and-compilation)
9. [Security and Firewalls](#security-and-firewalls)
10. [System Scheduling and Automation](#system-scheduling-and-automation)

---

## Advanced File Management

### File Archiving and Compression
```bash
# Tar archives
tar -cvf archive.tar file1 file2 directory/    # Create archive
tar -tvf archive.tar                           # List contents
tar -xvf archive.tar                           # Extract archive
tar -xvf archive.tar -C /path/to/extract/     # Extract to specific directory

# Compression with tar
tar -czf archive.tar.gz file1 file2            # Gzip compression
tar -cjf archive.tar.bz2 file1 file2           # Bzip2 compression
tar -cJf archive.tar.xz file1 file2            # XZ compression

# Extract compressed archives
tar -xzf archive.tar.gz
tar -xjf archive.tar.bz2
tar -xJf archive.tar.xz

# Zip files
zip -r archive.zip file1 file2 directory/      # Create zip
unzip archive.zip                              # Extract zip
unzip -l archive.zip                           # List contents
unzip -d /path/to/extract archive.zip          # Extract to specific location

# Compression tools
gzip filename                  # Compress to filename.gz
gunzip filename.gz             # Decompress
bzip2 filename                 # Compress with bzip2
bunzip2 filename.bz2           # Decompress bzip2
```

### Advanced File Search
```bash
# Find with regex patterns
find . -type f -regex ".*\.(txt|log)$"         # Find by extension

# Find by time
find . -type f -mtime -7                       # Modified in last 7 days
find . -type f -atime +30                      # Accessed more than 30 days ago
find . -type f -cmin -30                       # Changed in last 30 minutes

# Find by size
find . -type f -size +100M                     # Larger than 100MB
find . -type f -size -1M                       # Smaller than 1MB
find . -type f -size 50M                       # Exactly 50MB

# Find by permissions
find . -type f -perm 644                       # Exact permissions
find . -type f -perm /u=x                      # Has execute for owner
find . -type f -perm -004                      # Readable by others

# Find and execute
find . -name "*.tmp" -type f -delete
find . -name "*.log" -type f -exec gzip {} \;  # Gzip all logs
find . -type f -name "*.py" -exec wc -l {} \+  # Count lines in Python files

# Advanced grep patterns
grep -E "pattern1|pattern2" file               # Multiple patterns (extended)
grep -P "(?<=prefix)pattern" file              # Perl regex (lookahead)
grep -r "pattern" . --include="*.txt"          # Search with file type filter
grep -r "pattern" . --exclude-dir=".git"       # Exclude directories
```

### Working with Large Files
```bash
# Process large files efficiently
head -c 1G largefile.bin > first_gigabyte.bin  # Copy first gigabyte
tail -c 100M largefile.bin > last_100mb.bin    # Copy last 100MB

# Split large files
split -b 100M largefile.bin split_             # Split by size
split -l 10000 largefile.txt split_            # Split by lines
split -d largefile.bin part_                   # Use numeric suffixes

# Join split files
cat split_* > original_file

# Sparse files (efficient storage)
truncate -s 1G sparse_file.img                 # Create sparse file
du -sh sparse_file.img                         # Show disk usage
du --apparent-size -sh sparse_file.img         # Show actual size

# Memory-mapped file operations
mmap_tool file.dat                             # Tool for mmap operations
```

---

## Advanced Permissions and Access Control

### POSIX Access Control Lists (ACL)
```bash
# View ACL
getfacl filename
getfacl -R directory/

# Set ACL for user
setfacl -m u:username:rwx filename             # Grant permissions
setfacl -m u:username:rx filename              # Read and execute
setfacl -x u:username filename                 # Remove user ACL

# Set ACL for group
setfacl -m g:groupname:rwx filename
setfacl -m g:groupname:rx filename

# Set default ACL (for new files in directory)
setfacl -d -m u:username:rwx directory/
setfacl -d -m g:groupname:rx directory/

# Remove all ACLs
setfacl -b filename                            # Remove all ACL entries
setfacl -k directory/                          # Remove default ACLs

# Copy ACLs between files
getfacl file1 | setfacl -f - file2             # Copy ACL from file1 to file2
```

### SELinux and AppArmor

#### SELinux (Red Hat/CentOS)
```bash
# Check SELinux status
getenforce                                     # Get current mode
selinuxenabled && echo "SELinux is enabled"

# View SELinux contexts
ls -Z filename                                 # Show context
ps -Z                                          # Process contexts
netstat -Z                                     # Network contexts

# Change context
chcon -t var_t file.txt                        # Change type
chcon -u system_u:role_r:type_t file.txt       # Change full context

# Restore default contexts
restorecon filename
restorecon -R directory/

# SELinux booleans
getsebool -a                                   # List all booleans
setsebool -P sebool_name on                    # Set permanently
```

#### AppArmor (Ubuntu/Debian)
```bash
# Check AppArmor status
aa-status                                      # Show profiles
aa-enabled                                     # Check if enabled

# Profile modes
aa-complain /path/to/profile                   # Set to complain mode
aa-enforce /path/to/profile                    # Set to enforce mode

# View violations
tail -f /var/log/audit/audit.log               # Check audit logs
grep apparmor /var/log/syslog
```

### File Locking and Immutability
```bash
# Make files immutable (no changes, no deletion)
sudo chattr +i filename                        # Set immutable
sudo chattr -i filename                        # Remove immutable

# Append-only files
sudo chattr +a filename                        # Only append allowed
sudo chattr -a filename                        # Remove append-only

# No dump (skip during backup)
sudo chattr +d filename

# List file attributes
lsattr filename
lsattr -a directory/
```

---

## System Administration

### System Information
```bash
# CPU information
lscpu                                          # CPU details
cat /proc/cpuinfo                              # Detailed CPU info
nproc                                          # Number of CPUs

# Memory information
free -h                                        # Human-readable memory
free -m                                        # Memory in MB
cat /proc/meminfo                              # Detailed memory info

# Disk information
lsblk                                          # Block devices
fdisk -l                                       # Partition table
parted -l                                      # Partition information
df -h                                          # Disk usage
du -sh directory/                              # Directory size

# System information
uname -a                                       # All system info
hostnamectl                                    # Hostname and OS
uptime                                         # System uptime
lsb_release -a                                 # Linux version
cat /proc/version                              # Kernel version
```

### Systemd Service Management
```bash
# Service control
sudo systemctl start service_name              # Start service
sudo systemctl stop service_name               # Stop service
sudo systemctl restart service_name            # Restart service
sudo systemctl reload service_name             # Reload configuration
sudo systemctl enable service_name             # Start at boot
sudo systemctl disable service_name            # Don't start at boot

# Service status
systemctl status service_name                  # Current status
systemctl is-active service_name               # Check if running
systemctl is-enabled service_name              # Check if enabled
systemctl list-units --type=service            # List all services

# Service dependencies
systemctl list-dependencies service_name       # Show dependencies

# Create custom service
sudo nano /etc/systemd/system/myservice.service

# Service file example
[Unit]
Description=My Custom Service
After=network.target

[Service]
Type=simple
User=serviceuser
ExecStart=/usr/local/bin/myservice
Restart=on-failure

[Install]
WantedBy=multi-user.target

# Enable custom service
sudo systemctl daemon-reload
sudo systemctl enable myservice
sudo systemctl start myservice
```

### Running with Elevated Privileges
```bash
# Execute single command with sudo
sudo command

# Execute as specific user
sudo -u username command

# Execute as different group
sudo -g groupname command

# Preserve environment
sudo -E command

# Non-interactive (useful for scripts)
sudo -n command                                # No password prompt

# View sudo permissions
sudo -l

# Edit sudoers safely
sudo visudo

# Example sudoers entry
username ALL=(ALL) NOPASSWD: /usr/bin/service
```

---

## Storage and Disk Management

### Partition Management
```bash
# View partitions
fdisk -l                                       # List all partitions
parted -l                                      # Detailed partition info
lsblk                                          # Block device tree
blkid                                          # Block device UUID/LABEL

# Create partitions with fdisk
sudo fdisk /dev/sda                            # Interactive mode
# Commands: n (new), d (delete), t (type), w (write)

# Create partitions with parted
sudo parted /dev/sda mklabel gpt               # Create GPT table
sudo parted /dev/sda mkpart primary ext4 0% 50%  # Create 50% partition
sudo parted /dev/sda mkpart primary ext4 50% 100%  # Remaining space

# Remove partitions
sudo parted /dev/sda rm 1                      # Remove partition 1
```

### File System Management
```bash
# Create file systems
sudo mkfs.ext4 /dev/sda1                       # Ext4 file system
sudo mkfs.xfs /dev/sda1                        # XFS file system
sudo mkfs.btrfs /dev/sda1                      # Btrfs file system

# File system repair
sudo fsck /dev/sda1                            # Check and repair
sudo fsck.ext4 -f /dev/sda1                    # Force check

# Mount file systems
sudo mount /dev/sda1 /mnt/data
sudo mount -o ro /dev/sda1 /mnt/data           # Read-only
sudo mount -o remount,rw /mnt/data             # Remount as read-write

# Persistent mounts (/etc/fstab)
# /dev/sda1  /mnt/data  ext4  defaults  0  2

# Unmount
sudo umount /mnt/data
sudo umount -l /mnt/data                       # Lazy unmount

# View mounted file systems
mount
df -h
findmnt
```

### Logical Volume Management (LVM)
```bash
# Create physical volume
sudo pvcreate /dev/sda1 /dev/sda2

# Create volume group
sudo vgcreate myvg /dev/sda1 /dev/sda2

# Create logical volume
sudo lvcreate -L 10G -n mylv myvg

# Format and mount
sudo mkfs.ext4 /dev/myvg/mylv
sudo mount /dev/myvg/mylv /mnt/data

# Extend logical volume
sudo lvextend -L +5G /dev/myvg/mylv
sudo resize2fs /dev/myvg/mylv                  # Resize file system

# List LVM components
pvs                                            # Physical volumes
vgs                                            # Volume groups
lvs                                            # Logical volumes
```

---

## Advanced Networking

### Network Configuration
```bash
# Persistent network configuration (Netplan - Ubuntu)
sudo nano /etc/netplan/01-netcfg.yaml

# Example configuration
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
    eth1:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# Apply configuration
sudo netplan apply

# Traditional network config (systemd-networkd)
sudo systemctl restart systemd-networkd
```

### Network Troubleshooting
```bash
# DNS troubleshooting
nslookup domain.com                            # Query DNS
dig domain.com                                 # Detailed query
dig @8.8.8.8 domain.com                        # Query specific DNS server
host domain.com

# Trace network path
traceroute domain.com                          # ICMP trace
traceroute -T domain.com                       # TCP trace
mtr domain.com                                 # Real-time trace

# Check network latency
ping -c 4 -i 0.2 domain.com
fping host1 host2 host3                        # Parallel ping

# Capture network traffic
sudo tcpdump -i eth0                           # Capture packets
sudo tcpdump -i eth0 -w capture.pcap           # Save to file
sudo tcpdump -r capture.pcap                   # Read from file
sudo tcpdump -i eth0 -n port 80                # Filter by port
```

### Port Forwarding and Tunneling
```bash
# SSH tunneling
ssh -L 8080:localhost:80 user@remotehost       # Local port forward
ssh -R 8080:localhost:80 user@remotehost       # Remote port forward
ssh -D 1080 user@remotehost                    # SOCKS proxy

# Port forwarding with iptables
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-port 8080

# View iptables rules
sudo iptables -t nat -L -n -v

# Save iptables rules
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
```

---

## System Monitoring and Logging

### System Monitoring Tools
```bash
# Real-time monitoring
top                                            # CPU and memory
htop                                           # Enhanced top
iotop                                          # Disk I/O
nethogs                                        # Network per process
glances                                        # All-in-one monitor

# Performance monitoring
vmstat 1                                       # Virtual memory stats
iostat 1                                       # I/O statistics
mpstat 1                                       # Multiprocessor stats
pidstat 1                                      # Per-process stats

# System load
uptime                                         # Load average
w                                              # Logged in users and load
cat /proc/loadavg                              # Load average details
```

### Log Management
```bash
# System logs location
/var/log/syslog                                # General system log (Debian/Ubuntu)
/var/log/messages                              # General system log (RHEL/CentOS)
/var/log/auth.log                              # Authentication logs
/var/log/kernel.log                            # Kernel messages

# View logs
tail -f /var/log/syslog                        # Follow log updates
tail -n 50 /var/log/syslog                     # Last 50 lines
head -n 20 /var/log/syslog                     # First 20 lines

# Search logs
grep "ERROR" /var/log/syslog
grep -i "failed" /var/log/auth.log             # Case-insensitive
grep "password" /var/log/auth.log | tail -n 10

# Journal logs (systemd)
journalctl                                     # View all journal logs
journalctl -u service_name                     # Specific service
journalctl -f                                  # Follow journal
journalctl --since "2 hours ago"               # Time range
journalctl -p err                              # Error priority

# Log rotation with logrotate
sudo nano /etc/logrotate.d/myapp

# Example logrotate configuration
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 appuser appuser
}

# Compression logs
gzip /var/log/old.log
bzip2 /var/log/old.log
```

### Syslog Configuration
```bash
# Configure syslog (rsyslog)
sudo nano /etc/rsyslog.d/50-default.conf

# Log specific service
if $programname == 'myapp' then /var/log/myapp.log
& stop

# Send logs to remote server
*.* @@remote.server:514

# Apply configuration
sudo systemctl restart rsyslog
```

---

## Advanced Shell Scripting

### Function Best Practices
```bash
#!/bin/bash

# Function with local variables
calculate() {
    local num1=$1
    local num2=$2
    local result=$((num1 + num2))
    echo $result
}

# Function with error handling
safe_divide() {
    if [ $2 -eq 0 ]; then
        echo "Error: Division by zero" >&2
        return 1
    fi
    echo $(($1 / $2))
}

# Call with error checking
if result=$(safe_divide 10 2); then
    echo "Result: $result"
else
    echo "Division failed"
fi

# Recursive function
factorial() {
    local n=$1
    if [ $n -le 1 ]; then
        echo 1
    else
        echo $(( n * $(factorial $((n-1))) ))
    fi
}

# Function documentation
# Description: Calculate factorial
# Arguments: $1 - number
# Returns: factorial result
```

### Advanced Control Flow
```bash
#!/bin/bash

# Case statement
case $1 in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    *)
        echo "Unknown command"
        exit 1
        ;;
esac

# Select menu
select choice in "Option 1" "Option 2" "Option 3"; do
    case $choice in
        "Option 1") echo "Selected 1"; break ;;
        "Option 2") echo "Selected 2"; break ;;
        "Option 3") echo "Selected 3"; break ;;
    esac
done

# While loop with conditions
while [ -f "lockfile" ] && [ $counter -lt 10 ]; do
    sleep 1
    ((counter++))
done
```

### String Manipulation
```bash
#!/bin/bash

string="Hello World"

# String length
echo ${#string}                        # 11

# Substring
echo ${string:0:5}                     # "Hello"
echo ${string:6}                       # "World"

# Replace
echo ${string/World/Bash}              # "Hello Bash"
echo ${string//l/L}                    # "HeLLo WorLd"

# Remove pattern
echo ${string#H*}                      # "ello World"
echo ${string%d}                       # "Hello Worl"

# Convert case
echo ${string,,}                       # lowercase
echo ${string^^}                       # UPPERCASE

# Parameter expansion
param=${1:-default_value}              # Default if unset
param=${2:?Error message}              # Error if unset
```

### Error Handling and Debugging
```bash
#!/bin/bash
set -euo pipefail                      # Exit on error, undefined vars, pipe errors

# Trap errors
trap 'echo "Error on line $LINENO"' ERR

# Trap signals
trap 'echo "Received SIGINT"; exit 130' INT

# Cleanup on exit
cleanup() {
    rm -f "$TMPFILE"
    echo "Cleaned up"
}
trap cleanup EXIT

# Debug mode
# Run: bash -x script.sh
# Or: set -x in script

# Conditional debugging
if [ "${DEBUG:-0}" = "1" ]; then
    set -x
fi
```

---

## Software Installation and Compilation

### Building from Source
```bash
# Download source
wget https://example.com/software-1.0.tar.gz
tar -xzf software-1.0.tar.gz
cd software-1.0

# Typical build process
./configure                            # Configure build
./configure --prefix=/usr/local        # Install to /usr/local
make                                   # Compile
sudo make install                      # Install

# Build with specific options
./configure --enable-feature --disable-other
make -j$(nproc)                        # Parallel build

# View configure options
./configure --help
```

### Package Management Advanced
```bash
# Hold package version (Debian)
sudo apt-mark hold package_name        # Prevent upgrade
sudo apt-mark unhold package_name      # Allow upgrade

# Install specific version
apt-cache policy package_name          # Show available versions
sudo apt install package_name=version

# Create local repository
dpkg-scanpackages . > Packages
gzip -c Packages > Packages.gz

# Pin package version
sudo nano /etc/apt/preferences
# Package: package_name
# Pin: version 1.0
# Pin-Priority: 1001
```

### Dependency Management
```bash
# Check dependencies
apt-cache depends package_name         # Show dependencies
apt-cache rdepends package_name        # Show reverse dependencies
dpkg -s package_name | grep Depends    # Check installed dependencies

# Resolve dependency issues
sudo apt --fix-broken install
sudo apt autoremove                    # Remove unused packages
sudo apt autoclean                     # Clean old package files
```

---

## Security and Firewalls

### UFW Firewall (Ubuntu)
```bash
# Enable/disable firewall
sudo ufw enable
sudo ufw disable
sudo ufw status

# Allow/deny ports
sudo ufw allow 22/tcp                  # Allow SSH
sudo ufw allow 80/tcp                  # Allow HTTP
sudo ufw allow 443/tcp                 # Allow HTTPS
sudo ufw deny 23/tcp                   # Deny Telnet

# Allow from specific IP
sudo ufw allow from 192.168.1.100 to any port 22

# Remove rules
sudo ufw delete allow 22/tcp
sudo ufw reset                         # Reset firewall

# Firewall status details
sudo ufw show added                    # Show added rules
sudo ufw show raw                      # Show raw rules
```

### iptables (Advanced Firewall)
```bash
# List rules
sudo iptables -L                       # List all rules
sudo iptables -L -n                    # Numeric output
sudo iptables -L -n -v                 # Verbose output
sudo iptables -t nat -L                # NAT table

# Basic rules
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT   # Allow SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # Allow HTTP
sudo iptables -A INPUT -j DROP                        # Default drop

# Stateful firewall
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT

# Save rules
sudo netfilter-persistent save
# Or
sudo iptables-save > /etc/iptables/rules.v4
```

### SSH Security
```bash
# SSH configuration
sudo nano /etc/ssh/sshd_config

# Recommended settings
Port 2222                              # Change default port
PermitRootLogin no                     # Disable root login
PasswordAuthentication no               # Use key-only auth
PubkeyAuthentication yes
X11Forwarding no
PrintMotd no

# Generate SSH keys
ssh-keygen -t rsa -b 4096
ssh-keygen -t ed25519                  # More secure

# Copy public key to server
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host

# SSH with specific key
ssh -i ~/.ssh/custom_key user@host

# SSH config file
nano ~/.ssh/config
# Host myserver
#     HostName 192.168.1.100
#     User username
#     Port 2222
#     IdentityFile ~/.ssh/id_rsa

# Connect using config
ssh myserver
```

### File Integrity Monitoring
```bash
# AIDE (Advanced Intrusion Detection Environment)
sudo aideinit                          # Initialize database
sudo aide --check                      # Check for changes

# Tripwire
sudo tripwire --init                   # Initialize
sudo tripwire --check                  # Check integrity
sudo tripwire --update                 # Update database
```

---

## System Scheduling and Automation

### Cron Jobs
```bash
# Edit crontab
crontab -e                             # Edit user crontab
sudo crontab -e                        # Edit root crontab

# Crontab format
# MIN HOUR DOM MON DOW COMMAND
# 0   0    1   1   0   /path/to/script.sh

# Common examples
0 0 * * *    /usr/local/bin/backup.sh       # Daily at midnight
*/5 * * * *  /usr/local/bin/check.sh        # Every 5 minutes
0 2 * * 0    /usr/local/bin/weekly-task.sh  # Weekly Sunday 2AM
0 0 1 * *    /usr/local/bin/monthly.sh      # Monthly 1st at midnight

# View crontabs
crontab -l                             # User crontab
sudo crontab -l                        # Root crontab
cat /etc/cron.d/                       # System cron jobs

# Cron environment
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

### Systemd Timers
```bash
# Create service
sudo nano /etc/systemd/system/backup.service
[Unit]
Description=Backup Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=backupuser

# Create timer
sudo nano /etc/systemd/system/backup.timer
[Unit]
Description=Backup Timer
Requires=backup.service

[Timer]
OnCalendar=daily
OnCalendar=*-*-01 02:00:00          # Monthly 1st 2AM
OnBootSec=5min                      # 5 min after boot
OnUnitActiveSec=1day                # 1 day after last run

[Install]
WantedBy=timers.target

# Enable and start timer
sudo systemctl enable backup.timer
sudo systemctl start backup.timer
sudo systemctl list-timers          # View active timers
```

### At Command (One-time Scheduling)
```bash
# Schedule one-time job
at 3:00 PM tomorrow               # Interactive mode
at now + 2 hours                  # In 2 hours
at 14:30 Friday                   # Specific date/time

# Script execution
echo "/usr/local/bin/backup.sh" | at 2:00 AM

# List scheduled jobs
atq

# View job details
at -c job_id

# Remove job
atrm job_id

# Enable at daemon
sudo systemctl enable atd
sudo systemctl start atd
```

---

## Best Practices for Intermediate Linux

1. **Automate routine tasks**: Use cron/systemd timers for backups and maintenance
2. **Monitor system health**: Regularly check disk, memory, and CPU usage
3. **Keep audit logs**: Monitor authentication and security events
4. **Use version control for scripts**: Track changes to automation scripts
5. **Implement backups**: Multiple backup strategies (local, remote, cloud)
6. **Plan disk space**: Monitor and manage disk usage proactively
7. **Use LVM**: For flexible storage management and resizing
8. **Document changes**: Keep detailed records of system modifications
9. **Test configurations**: Test changes on staging systems first
10. **Stay updated**: Keep systems patched and current
11. **Use firewalls**: Implement proper firewall rules and policies
12. **Secure SSH**: Disable passwords, use key-based authentication

---

## Conclusion

Intermediate Linux skills involve system administration, networking, security, and automation. Practice these concepts regularly and explore your system's logs and configurations to deepen your understanding.

For more information, refer to the [Linux man pages](https://man7.org/linux/man-pages/), [Linux Foundation](https://www.linuxfoundation.org/), or your distribution's official documentation.
