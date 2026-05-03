# Linux Basics Tutorial

## Table of Contents
1. [Introduction to Linux](#introduction-to-linux)
2. [File System and Navigation](#file-system-and-navigation)
3. [Working with Files and Directories](#working-with-files-and-directories)
4. [File Permissions and Ownership](#file-permissions-and-ownership)
5. [Text Processing and Searching](#text-processing-and-searching)
6. [User and Group Management](#user-and-group-management)
7. [Process Management](#process-management)
8. [Networking Basics](#networking-basics)
9. [Package Management](#package-management)
10. [Bash Scripting Fundamentals](#bash-scripting-fundamentals)

---

## Introduction to Linux

### What is Linux?
Linux is a free, open-source operating system kernel created by Linus Torvalds. It powers servers, desktops, mobile devices (Android), and embedded systems worldwide.

### Linux Distributions
- **Ubuntu**: User-friendly, great for beginners
- **CentOS/RHEL**: Enterprise-focused
- **Debian**: Stable and reliable
- **Fedora**: Cutting-edge features
- **Arch**: Minimalist and customizable

### The Linux Shell
The shell is a command-line interface that interprets user commands. The most common shell is **Bash** (Bourne Again Shell).

---

## File System and Navigation

### Understanding the Linux File System
```bash
/          # Root directory
/home      # User home directories
/etc       # System configuration files
/var       # Variable data (logs, temp files)
/usr       # User programs and libraries
/bin       # Essential command binaries
/sbin      # System administration binaries
/tmp       # Temporary files
/lib       # System libraries
/opt       # Optional software packages
```

### Basic Navigation Commands
```bash
# Print working directory
pwd

# List files and directories
ls              # Basic listing
ls -l           # Long format with details
ls -la          # Include hidden files
ls -lh          # Human-readable file sizes
ls -R           # Recursive listing

# Change directory
cd /path/to/directory
cd ~            # Go to home directory
cd ..           # Go to parent directory
cd -            # Go to previous directory

# Tree view of directories
tree
tree -L 2       # Limit depth to 2 levels
```

### Path Concepts
```bash
# Absolute path (starts from root)
/home/user/documents/file.txt

# Relative path (relative to current directory)
documents/file.txt
../documents/file.txt

# Home directory shortcut
~/documents/file.txt

# Current directory
./file.txt
```

---

## Working with Files and Directories

### Creating and Deleting
```bash
# Create directories
mkdir directory_name
mkdir -p path/to/nested/directory    # Create parent directories if needed

# Create empty files
touch filename.txt

# Remove files
rm filename.txt
rm -f filename.txt      # Force remove without confirmation
rm -i filename.txt      # Prompt before deletion

# Remove directories
rmdir empty_directory
rm -r directory_name    # Recursively remove with contents
rm -rf directory_name   # Force recursive removal
```

### Copying and Moving
```bash
# Copy files
cp source.txt destination.txt
cp source.txt /path/to/destination/
cp -r source_dir dest_dir       # Recursive copy

# Move/rename files
mv old_name.txt new_name.txt
mv file.txt /path/to/destination/
mv directory_name new_location/

# Backup files
cp important.txt important.txt.bak
```

### Viewing File Contents
```bash
# Display entire file
cat filename.txt

# View file with pagination
less filename.txt       # Navigate with arrow keys, 'q' to quit
more filename.txt       # Similar to less, older version

# View first/last lines
head filename.txt       # First 10 lines
head -n 20 filename.txt # First 20 lines
tail filename.txt       # Last 10 lines
tail -n 5 filename.txt  # Last 5 lines

# Follow file changes in real-time
tail -f logfile.log

# View entire file with line numbers
nl filename.txt
cat -n filename.txt
```

### Creating and Editing Files
```bash
# Using echo to create/append
echo "Hello, World!" > file.txt       # Create/overwrite
echo "Additional line" >> file.txt    # Append

# Using text editors
nano filename.txt       # Simple editor
vi filename.txt         # Advanced editor
vim filename.txt        # Improved vi

# View file with syntax highlighting
cat filename.txt | less
```

---

## File Permissions and Ownership

### Understanding Permissions
```bash
# Permission format: -rwxrwxrwx
# Position:         1 234567890
# 1: File type (- = regular, d = directory, l = link)
# 2-4: Owner permissions
# 5-7: Group permissions
# 8-10: Others permissions

# Permission values:
# r (read) = 4
# w (write) = 2
# x (execute) = 1

# Examples:
# 755 = rwxr-xr-x (owner: full, group/others: read+execute)
# 644 = rw-r--r-- (owner: read+write, others: read)
# 700 = rwx------ (only owner can access)
```

### Changing Permissions
```bash
# Symbolic method
chmod u+x filename.txt          # Add execute to owner
chmod g-w filename.txt          # Remove write from group
chmod o-r filename.txt          # Remove read from others
chmod a+r filename.txt          # Add read for all
chmod u=rwx,g=rx,o= filename.txt # Set specific permissions

# Numeric method
chmod 755 filename.txt          # rwxr-xr-x
chmod 644 filename.txt          # rw-r--r--
chmod 700 filename.txt          # rwx------
chmod 777 filename.txt          # rwxrwxrwx

# Recursive permission change
chmod -R 755 directory_name
```

### Changing Ownership
```bash
# Change owner
chown new_owner filename.txt

# Change owner and group
chown new_owner:new_group filename.txt

# Recursive change
chown -R user:group directory_name

# Change only group
chgrp new_group filename.txt
```

### Special Permissions
```bash
# Set-User-ID (SUID) - run as owner
chmod u+s filename.txt      # Numeric: 4xxx

# Set-Group-ID (SGID) - run as group
chmod g+s filename.txt      # Numeric: 2xxx

# Sticky bit - only owner can delete
chmod o+t directory_name    # Numeric: 1xxx

# Examples
chmod 4755 filename.txt     # SUID + rwxr-xr-x
chmod 2755 filename.txt     # SGID + rwxr-xr-x
chmod 1755 directory_name   # Sticky bit + rwxr-xr-x
```

---

## Text Processing and Searching

### Searching for Files and Content
```bash
# Find files by name
find /path -name "*.txt"
find . -name "*log*" -type f
find . -type d -name "test*"        # Find directories
find . -type f -size +100M          # Find large files

# Find with execution
find . -name "*.tmp" -delete         # Delete temp files
find . -name "*.txt" -exec cat {} \; # Execute command on matches

# Search within file contents
grep "search_term" filename.txt
grep -i "search_term" filename.txt   # Case-insensitive
grep -n "search_term" filename.txt   # Show line numbers
grep -c "search_term" filename.txt   # Count matches
grep -r "search_term" directory/     # Recursive search
grep -v "exclude" filename.txt       # Invert match (exclude)
```

### Text Processing Tools
```bash
# Sort lines
sort filename.txt
sort -n numbers.txt         # Numeric sort
sort -r filename.txt        # Reverse sort
sort -u filename.txt        # Remove duplicates

# Remove duplicate lines
uniq filename.txt
sort filename.txt | uniq    # Sort then remove duplicates
uniq -c filename.txt        # Count occurrences

# Cut columns
cut -d: -f1 /etc/passwd     # Extract first field (delimiter :)
cut -c1-10 filename.txt     # Extract characters 1-10

# Stream editor for filtering and transforming
sed 's/old/new/' filename.txt       # Replace first occurrence per line
sed 's/old/new/g' filename.txt      # Replace all occurrences
sed '5d' filename.txt               # Delete line 5
sed -n '5,10p' filename.txt         # Print lines 5-10

# AWK - powerful text processing
awk '{print $1}' filename.txt       # Print first column
awk -F: '{print $1}' /etc/passwd    # Print first field (delimiter :)
awk '{sum+=$1} END {print sum}' numbers.txt  # Sum values
```

### Pattern Matching
```bash
# Regular expressions
grep "^start" filename.txt          # Lines starting with 'start'
grep "end$" filename.txt            # Lines ending with 'end'
grep "p.ttern" filename.txt         # . matches any character
grep "colou?r" filename.txt         # ? makes preceding element optional
grep "[0-9]" filename.txt           # Contains digit
grep "[^0-9]" filename.txt          # Doesn't contain digit
```

---

## User and Group Management

### User Management
```bash
# Add new user
useradd username
useradd -m -s /bin/bash username    # Create home dir and set shell

# Remove user
userdel username
userdel -r username                 # Remove with home directory

# Modify user
usermod -s /bin/zsh username        # Change shell
usermod -d /home/newpath username   # Change home directory
usermod -G group1,group2 username   # Add to groups

# Change password
passwd username
passwd                              # Change own password

# View user information
id username
whoami                              # Current user
who                                 # Logged-in users
```

### Group Management
```bash
# Create group
groupadd groupname

# Remove group
groupdel groupname

# Add user to group
usermod -aG groupname username      # -a: append, -G: groups

# View groups
groups username                     # Show user's groups
cat /etc/group                      # List all groups
```

### Sudo Access
```bash
# Grant sudo privilege
usermod -aG sudo username           # Add to sudo group

# Edit sudoers file
sudo visudo                         # Safely edit sudoers

# Run command as another user
sudo command
sudo -u username command            # Run as specific user
```

---

## Process Management

### Viewing Processes
```bash
# List running processes
ps                                  # Current shell processes
ps aux                              # All processes with details
ps aux | grep process_name          # Search for specific process

# Real-time process viewer
top
htop                                # Enhanced version of top

# View process tree
pstree
pstree -p                           # Include process IDs

# List processes by user
ps -u username
```

### Managing Processes
```bash
# Run process in background
command &

# Suspend running process (Ctrl+Z in terminal)
# Then resume in background
bg

# Resume in foreground
fg

# List background jobs
jobs
jobs -l                             # Include process IDs

# Kill processes
kill PID                            # Send SIGTERM
kill -9 PID                         # Send SIGKILL (force)
kill -STOP PID                      # Pause process
kill -CONT PID                      # Resume process

# Kill by process name
killall process_name
pkill -f "process pattern"

# Find process ID
pidof process_name
pgrep process_name
```

### Process Priorities
```bash
# Run with specific priority
nice -n 10 command                  # Lower priority (19 is lowest)
nice -n -10 command                 # Higher priority (-20 is highest)

# Change priority of running process
renice +5 -p PID                    # Decrease priority
renice -5 -p PID                    # Increase priority
```

---

## Networking Basics

### Network Configuration
```bash
# View network interfaces
ifconfig
ip addr show                        # Modern alternative
ip link show

# View routing table
route -n
ip route show

# Check DNS settings
cat /etc/resolv.conf

# Configure IP address
sudo ifconfig eth0 192.168.1.100
sudo ip addr add 192.168.1.100/24 dev eth0
```

### Network Testing and Diagnostics
```bash
# Ping - test connectivity
ping -c 4 google.com                # 4 packets

# DNS lookup
nslookup google.com
dig google.com
host google.com

# View network connections
netstat -tuln                       # TCP/UDP listening ports
netstat -tun | grep ESTABLISHED    # Active connections
ss -tuln                            # Modern alternative

# Check open ports
netstat -an | grep LISTEN
ss -an | grep LISTEN

# Trace network route
traceroute google.com
tracepath google.com
```

### File Transfer
```bash
# Secure copy
scp local_file user@remote:/path/
scp user@remote:/path/file local_path/

# Secure shell
ssh user@remote_host
ssh -p 2222 user@remote_host        # Custom port

# File synchronization
rsync -avz local_path/ user@remote:/path/
rsync -avz --delete local_path/ user@remote:/path/  # Delete extra files

# Download files
wget https://example.com/file.tar.gz
curl -O https://example.com/file.tar.gz
```

---

## Package Management

### Debian/Ubuntu (apt)
```bash
# Update package list
sudo apt update

# Upgrade installed packages
sudo apt upgrade                    # Safe upgrade
sudo apt full-upgrade               # May remove packages

# Install package
sudo apt install package_name
sudo apt install pkg1 pkg2 pkg3     # Multiple packages

# Remove package
sudo apt remove package_name
sudo apt purge package_name         # Remove with config files

# Search packages
apt search search_term
apt-cache search search_term

# Show package info
apt show package_name
apt-cache show package_name
```

### Red Hat/CentOS (yum/dnf)
```bash
# Update packages
sudo yum update
sudo dnf upgrade

# Install package
sudo yum install package_name
sudo dnf install package_name

# Remove package
sudo yum remove package_name
sudo dnf remove package_name

# Search packages
yum search search_term
dnf search search_term

# Show package info
yum info package_name
dnf info package_name
```

### Managing Repositories
```bash
# Add repository (Ubuntu)
sudo add-apt-repository ppa:username/ppa-name
sudo apt update

# List repositories
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Remove repository
sudo add-apt-repository --remove ppa:username/ppa-name
```

---

## Bash Scripting Fundamentals

### Creating Your First Script
```bash
#!/bin/bash
# This is a comment

echo "Hello, World!"
echo "Welcome to Bash scripting"

# Make script executable
# chmod +x script.sh
# Run: ./script.sh
```

### Variables
```bash
#!/bin/bash

# Variable assignment
name="Alice"
age=30

# Using variables
echo "Name: $name"
echo "Age: $age"

# Command substitution
current_date=$(date)
echo "Current date: $current_date"

# Read user input
read -p "Enter your name: " user_name
echo "Hello, $user_name!"
```

### Conditionals
```bash
#!/bin/bash

# If-then-else
age=18
if [ $age -ge 18 ]; then
    echo "You are an adult"
else
    echo "You are a minor"
fi

# Multiple conditions
if [ $age -lt 13 ]; then
    echo "Child"
elif [ $age -lt 18 ]; then
    echo "Teenager"
else
    echo "Adult"
fi

# Test conditions
[ -f file.txt ] && echo "File exists"      # File exists
[ -d directory ] && echo "Directory exists" # Directory exists
[ -z "$var" ] && echo "Variable is empty"   # String is empty
[ -n "$var" ] && echo "Variable is not empty" # String is not empty
```

### Loops
```bash
#!/bin/bash

# For loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# For loop with range
for i in {1..10}; do
    echo "$i"
done

# While loop
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Loop through files
for file in *.txt; do
    echo "Processing: $file"
done
```

### Functions
```bash
#!/bin/bash

# Define function
greet() {
    echo "Hello, $1!"
}

# Call function
greet "Alice"
greet "Bob"

# Function with return value
add() {
    local sum=$(($1 + $2))
    echo $sum
}

result=$(add 5 3)
echo "Sum: $result"

# Function with local variables
calculate() {
    local x=10
    local y=20
    echo $((x + y))
}
```

### Arrays
```bash
#!/bin/bash

# Create array
fruits=("apple" "banana" "cherry")

# Access array elements
echo ${fruits[0]}       # First element
echo ${fruits[@]}       # All elements
echo ${#fruits[@]}      # Array length

# Loop through array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Add to array
fruits+=("date")
```

---

## Linux Best Practices

1. **Keep system updated**: Regularly run `apt update && apt upgrade`
2. **Use strong passwords**: Mix uppercase, lowercase, numbers, and symbols
3. **Manage user permissions carefully**: Don't use sudo unnecessarily
4. **Monitor disk space**: Use `df -h` to check available space
5. **Backup important files**: Use `tar` or `rsync` regularly
6. **Check logs for errors**: Review `/var/log/` files
7. **Use version control**: Track changes with Git
8. **Document your scripts**: Add comments explaining what code does
9. **Test scripts before production**: Always test on non-critical systems first
10. **Keep software updated**: Regular patches prevent security vulnerabilities

---

## Useful Linux Commands Reference

| Command | Purpose |
|---------|---------|
| `pwd` | Print working directory |
| `ls` | List files and directories |
| `cd` | Change directory |
| `mkdir` | Create directory |
| `rm` | Remove file/directory |
| `cp` | Copy file |
| `mv` | Move/rename file |
| `cat` | Display file contents |
| `grep` | Search text |
| `find` | Find files |
| `chmod` | Change permissions |
| `chown` | Change ownership |
| `ps` | List processes |
| `kill` | Terminate process |
| `sudo` | Execute as superuser |
| `apt/yum` | Package manager |
| `ssh` | Secure shell connection |
| `scp` | Secure copy |
| `tar` | Archive files |
| `zip` | Compress files |

---

## Conclusion

This tutorial covers the fundamental Linux commands and concepts. Regular practice will help you become proficient with the command line. Remember, Linux is a powerful system, and understanding these basics is essential for system administration, development, and general computer literacy.

For more information, refer to the [Linux Manual Pages](https://man7.org/linux/man-pages/) or visit your distribution's official documentation.
