# MODULE 1: LINUX ADMINISTRATION

## 📚 Overview
Linux is the backbone of DevOps. This module covers everything from kernel architecture to practical system administration, giving you deep knowledge needed for production systems.

---

## 1. LINUX ARCHITECTURE

### Linux Kernel
- **Core** - Process scheduling, memory management, file system
- **Boot Process** - BIOS → Bootloader → Kernel → Init
- **Monolithic Design** - All core services in one kernel

### System Components
```
┌─────────────────────────────────────┐
│      User Applications              │
├─────────────────────────────────────┤
│      Shell & Utilities               │
├─────────────────────────────────────┤
│      System Libraries (glibc)        │
├─────────────────────────────────────┤
│      Linux Kernel                    │
├─────────────────────────────────────┤
│      Hardware (CPU, RAM, Disk)       │
└─────────────────────────────────────┘
```

---

## 2. BOOT PROCESS

### Detailed Flow
1. **POST** - Power-On Self-Test (Hardware check)
2. **BIOS/UEFI** - Firmware initialization
3. **MBR/GPT** - Master Boot Record reads bootloader
4. **GRUB2** - Bootloader loads kernel image
5. **Kernel Initialization** - Sets up memory, interrupts, processes
6. **Init System** - Systemd (PID 1) starts services
7. **Login** - TTY and user sessions

### Important Files
```bash
/boot/grub/grub.cfg      # GRUB configuration
/boot/vmlinuz-*          # Kernel image
/boot/initramfs-*        # Initial RAM filesystem
/etc/fstab              # Filesystem mount table
```

---

## 3. FILE SYSTEM

### Hierarchy
```
/                   # Root
├── /boot           # Boot files, kernels, GRUB
├── /bin             # Essential binaries (cat, ls, cp)
├── /sbin            # System binaries (ifconfig, fdisk)
├── /etc             # Configuration files
├── /home            # User home directories
├── /root            # Root user home
├── /tmp             # Temporary files (cleared on reboot)
├── /var             # Variable data (logs, cache)
├── /var/log         # System logs
├── /usr             # User programs and libraries
├── /lib             # Shared libraries
├── /opt             # Optional software
├── /srv             # Service data
├── /dev             # Device files
├── /proc            # Process information (virtual)
├── /sys             # System information (virtual)
└── /run             # Runtime data
```

### File Types
```bash
-   Regular file
d   Directory
l   Symbolic link
c   Character device
b   Block device
p   Named pipe
s   Socket
```

### Inode Structure
```bash
File content is stored in blocks
Inode contains:
  - File permissions
  - Owner (UID, GID)
  - Timestamps (access, modify, change)
  - Size
  - Block pointers
  - Hard link count
```

---

## 4. PERMISSIONS

### Permission Basics
```
-rwxrwxrwx
 │││││││││
 │││││││└─ Other: execute
 │││││││── Other: write
 │││││││─── Other: read
 │││││────  Group: execute
 │││││───── Group: write
 │││││────── Group: read
 │││└────────  Owner: execute
 │││─────────  Owner: write
 │││──────────  Owner: read
 │└───────────  Link count (hardlinks)
 └────────────  File type (- = regular)

Octal Representation:
r = 4 (read)
w = 2 (write)
x = 1 (execute)

755 = rwxr-xr-x (owner rwx, group rx, other rx)
644 = rw-r--r-- (owner rw, group r, other r)
```

### Essential Commands
```bash
chmod 755 file              # Change permissions (octal)
chmod u+x file              # Add execute for owner
chmod g-w file              # Remove write from group
chown user:group file       # Change owner and group
chown -R user:group /dir    # Recursive change
umask 0022                  # Default permissions mask (077 for secure)
```

### Special Permissions
```bash
Setuid (4) - File runs as owner (e.g., /usr/bin/passwd)
Setgid (2) - File runs as group
Sticky (1) - Only owner can delete (e.g., /tmp)

chmod 4755 file             # Setuid
chmod 2755 file             # Setgid
chmod 1755 dir              # Sticky bit
chmod u+s file              # Symbolic setuid
```

---

## 5. USERS & GROUPS

### User Management
```bash
# Add user
useradd -m -s /bin/bash username
  -m  Create home directory
  -s  Specify shell
  -u  Specify UID
  -g  Specify primary group
  -G  Add to supplementary groups

# User with home dir and bash
useradd -m -s /bin/bash -d /home/john john

# Delete user
userdel -r username         # -r removes home directory

# Modify user
usermod -aG sudo username   # Add to sudo group
usermod -s /bin/bash username  # Change shell
usermod -l newname oldname  # Rename user
```

### Group Management
```bash
groupadd devops             # Create group
groupdel groupname          # Delete group
groupmod -n newname oldname # Rename group
usermod -aG groupname user  # Add user to group
```

### Important Files
```bash
/etc/passwd         # User accounts (user:x:uid:gid:gecos:home:shell)
/etc/shadow         # Password hashes (only root readable)
/etc/group          # Group definitions
/etc/sudoers        # Sudo privileges (edit with visudo)
```

### Sudo Configuration
```bash
# Edit sudo configuration
visudo             # Edit /etc/sudoers safely

# Examples in /etc/sudoers
%devops ALL=(ALL) NOPASSWD:ALL    # devops group, no password
user1 ALL=(ALL) ALL               # Full sudo
user2 ALL=NOPASSWD:/usr/bin/docker # Only docker, no password
```

---

## 6. PROCESS MANAGEMENT

### Process Basics
```bash
# View processes
ps aux                      # All processes
ps auxf                     # Tree view
pstree                      # Process tree
pstree -p                   # With PIDs
top                         # Real-time monitoring
htop                        # Better top (if installed)
lsof -i :8080              # Find process on port

# Process info
ps -p 1234 -o comm=        # Get process name by PID
ps -p 1234 -o cmd=         # Get full command
ps -L 1234                  # Show threads of process

# Running processes
./script.sh &               # Run in background
jobs                        # List background jobs
fg %1                       # Bring job 1 to foreground
bg %1                       # Resume job 1 in background
Ctrl+Z                      # Suspend current process
```

### Signals
```bash
SIGTERM (15) - Graceful termination
SIGKILL (9)  - Force kill (cannot be caught)
SIGSTOP (19) - Pause process
SIGCONT (18) - Resume process
SIGHUP (1)   - Hangup, reload config
SIGUSR1 (10) - User-defined signal

kill -TERM 1234             # Graceful terminate
kill -9 1234                # Force kill
kill -HUP 1234              # Reload config
killall nginx               # Kill all nginx processes
```

### Process Priority
```bash
nice -n 10 ./script.sh      # Run with priority 10
renice -n 5 -p 1234        # Change existing process priority
  (Higher number = lower priority)
  Range: -20 (highest) to 19 (lowest)

ps -o pid,ni,cmd -p 1234   # Show niceness
```

### Zombie Processes
```bash
# Zombie process = child exits but parent hasn't reaped it
ps aux | grep Z             # Find zombies
# Solution: Kill parent process
kill -9 parent_pid
```

---

## 7. SYSTEMD

### Systemd Basics
- Modern init system
- Service management
- Socket activation
- Timer jobs
- Dependency management

### Essential Commands
```bash
# Service management
systemctl start nginx                   # Start service
systemctl stop nginx                    # Stop service
systemctl restart nginx                 # Restart
systemctl reload nginx                  # Reload config without restart
systemctl status nginx                  # View status
systemctl enable nginx                  # Enable at boot
systemctl disable nginx                 # Disable at boot
systemctl is-active nginx               # Check if running
systemctl is-enabled nginx              # Check if enabled at boot

# View services
systemctl list-units --type=service     # All services
systemctl list-units --type=service --state=failed  # Failed services
systemctl list-unit-files               # All unit files

# Logs
journalctl -u nginx                     # Logs for specific service
journalctl -u nginx -f                  # Follow logs (tail -f)
journalctl -u nginx --since today       # Logs since today
journalctl -p err                       # Error level and above
journalctl -n 50                        # Last 50 lines
```

### Systemd Service File
```ini
[Unit]
Description=My Web Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/app
ExecStart=/opt/app/start.sh
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Systemd Timer
```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target

# /etc/systemd/system/backup.service
[Unit]
Description=Backup Service

[Service]
Type=oneshot
ExecStart=/opt/backup/run.sh
```

---

## 8. NETWORKING

### Network Configuration

#### IP Configuration
```bash
# Old style (deprecated)
ifconfig eth0 192.168.1.10/24
ifconfig eth0 up/down

# New style (ip command)
ip addr add 192.168.1.10/24 dev eth0
ip addr del 192.168.1.10/24 dev eth0
ip link set eth0 up/down
ip addr show                    # Show all IPs
ip link show                    # Show interfaces
ip route show                   # Show routing table

# Permanent configuration (Debian/Ubuntu)
cat /etc/network/interfaces
  auto eth0
  iface eth0 inet static
    address 192.168.1.10
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4

# Permanent configuration (CentOS/RHEL)
cat /etc/sysconfig/network-scripts/ifcfg-eth0
  DEVICE=eth0
  TYPE=Ethernet
  IPADDR=192.168.1.10
  NETMASK=255.255.255.0
  GATEWAY=192.168.1.1
  DNS1=8.8.8.8
```

#### Network Troubleshooting
```bash
ping 8.8.8.8                    # ICMP connectivity
traceroute 8.8.8.8              # Show hops to destination
mtr 8.8.8.8                     # Real-time traceroute
telnet example.com 80           # Test TCP connectivity
nc -zv example.com 80           # Netcat connectivity test
curl -I example.com             # HTTP headers
ssh -v user@host                # SSH with verbose output
dig example.com                 # DNS query
nslookup example.com            # DNS lookup
getent hosts example.com        # Check /etc/hosts and DNS

# Packet analysis
tcpdump -i eth0 -n host 192.168.1.1    # Capture packets
tcpdump -i eth0 -n 'port 80'           # Capture port 80
```

#### Firewall (UFW - Ubuntu)
```bash
ufw enable                      # Enable firewall
ufw disable                     # Disable firewall
ufw allow 22/tcp                # Allow SSH
ufw allow 80/tcp                # Allow HTTP
ufw allow 443/tcp               # Allow HTTPS
ufw deny 3306/tcp               # Deny MySQL
ufw status                      # Show firewall status
ufw status numbered             # Show with rule numbers
ufw delete 1                    # Delete rule 1
```

#### Firewall (firewalld - CentOS/RHEL)
```bash
systemctl start firewalld       # Start firewall
firewall-cmd --list-all         # Show all rules
firewall-cmd --add-port=80/tcp  # Add port (temporary)
firewall-cmd --add-port=80/tcp --permanent  # Permanent
firewall-cmd --reload           # Reload rules
firewall-cmd --add-service=http --permanent
```

---

## 9. DISK MANAGEMENT

### Partitioning
```bash
# List disks
lsblk                           # Block devices
fdisk -l                        # List all disks
parted -l                       # Partition layout

# Create partitions (fdisk)
fdisk /dev/sda
  n - New partition
  p - Primary partition
  e - Extended partition
  d - Delete partition
  w - Write changes
  q - Quit without writing

# Create partitions (parted)
parted /dev/sda
  mkpart primary ext4 0 100GB  # Create partition
  print                         # Show partitions
  quit
```

### File Systems
```bash
# Format partition
mkfs.ext4 /dev/sda1             # Format as ext4
mkfs.xfs /dev/sda1              # Format as XFS
mkfs.btrfs /dev/sda1            # Format as BTRFS

# Mount
mount /dev/sda1 /mnt/data       # Mount partition
mount -o remount,rw /          # Remount as read-write
mount -t tmpfs -o size=1G tmpfs /mnt/tmp  # Tmpfs

# Unmount
umount /mnt/data                # Unmount
umount -l /mnt/data             # Lazy unmount

# Permanent mount (/etc/fstab)
/dev/sda1  /home   ext4   defaults,nofail  0  2
/dev/sdb1  /data   xfs    defaults         0  0
tmpfs      /tmp    tmpfs  size=2G          0  0
  # device  path   type   options          dump  fsck

mount -a                        # Mount all from /etc/fstab
```

### File System Checks
```bash
# Check file system
fsck /dev/sda1                  # File system check
fsck -y /dev/sda1              # Automatic yes
e2fsck /dev/sda1               # Ext4 specific

# Resize file system
resize2fs /dev/sda1            # Extend ext4
resize2fs /dev/sda1 100G       # Resize to 100GB

# Disk space
df -h                          # Disk usage (human readable)
du -sh /home                   # Directory size
du -sh /home/*                 # Size per subdirectory
du -sh /home/* | sort -h       # Sorted by size
```

---

## 10. LOGICAL VOLUME MANAGER (LVM)

### LVM Concepts
```
Physical Volume (PV) - Physical disk/partition
  ↓
Volume Group (VG) - Pool of PV
  ↓
Logical Volume (LV) - Virtual partition from VG
  ↓
File System - Formatted LV
```

### LVM Commands
```bash
# Physical Volumes
pvcreate /dev/sda1 /dev/sdb1       # Create PV
pvs                                # List PVs
pvdisplay                          # Detailed PV info
pvremove /dev/sda1                 # Remove PV

# Volume Groups
vgcreate vg0 /dev/sda1 /dev/sdb1   # Create VG
vgs                                # List VGs
vgdisplay                          # Detailed VG info
vgextend vg0 /dev/sdc1             # Add PV to VG
vgreduce vg0 /dev/sdc1             # Remove PV from VG
vgremove vg0                       # Remove VG

# Logical Volumes
lvcreate -L 10G -n lv_data vg0     # Create 10GB LV
lvcreate -l 100%FREE -n lv_all vg0 # Use all free space
lvs                                # List LVs
lvdisplay                          # Detailed LV info
lvextend -L +5G /dev/vg0/lv_data   # Extend by 5GB
lvextend -L 20G /dev/vg0/lv_data   # Extend to 20GB
lvremove /dev/vg0/lv_data          # Remove LV
lvresize -L 15G /dev/vg0/lv_data   # Resize to 15GB

# After extending LV, extend file system
resize2fs /dev/vg0/lv_data         # Ext4
xfs_growfs /dev/vg0/lv_data        # XFS
```

---

## 11. SECURE SHELL (SSH)

### SSH Basics
```bash
# Connect
ssh user@hostname               # Connect to remote
ssh user@hostname -p 2222       # Custom port
ssh -i /path/to/key user@host   # Specific key
ssh -v user@host               # Verbose (debugging)
ssh -X user@host               # X11 forwarding

# SSH Key Generation
ssh-keygen -t rsa -b 4096      # Generate RSA key pair
ssh-keygen -t ed25519          # Generate Ed25519 key (modern)
ssh-keygen -p -f ~/.ssh/id_rsa # Change passphrase

# Key-based Authentication
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host  # Copy public key
# Manual: append ~/.ssh/id_rsa.pub to ~/.ssh/authorized_keys on remote

# SSH Config
cat ~/.ssh/config
  Host myserver
    HostName example.com
    User deployer
    Port 2222
    IdentityFile ~/.ssh/myserver_key
    IdentitiesOnly yes

ssh myserver                    # Uses config
```

### SSH Configuration
```bash
# Server configuration (/etc/ssh/sshd_config)
Port 2222                       # Change port
PermitRootLogin no              # Disable root login
PasswordAuthentication no        # Disable password auth
PubkeyAuthentication yes         # Enable key auth
AllowUsers user1 user2          # Whitelist users
AllowGroups devops admins       # Whitelist groups
MaxAuthTries 3                  # Max login attempts
ClientAliveInterval 300         # Keep-alive timeout
UseDNS no                       # Disable DNS lookup

# After editing
sshd -t                         # Test config
systemctl restart ssh           # Apply changes
```

### SSH Tunneling
```bash
# Local port forwarding
ssh -L 8080:localhost:80 user@remote  # Remote port on localhost
# Access localhost:8080, tunnels to remote:80

# Remote port forwarding
ssh -R 9000:localhost:8000 user@remote
# Remote can access remote:9000 to reach your localhost:8000

# Dynamic SOCKS proxy
ssh -D 1080 user@remote         # Remote as SOCKS proxy
curl --socks5 localhost:1080 example.com
```

---

## 12. CRON JOBS

### Crontab Syntax
```
┌─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬──────────
│ Minute  │  Hour   │   Day   │  Month  │ Weekday │ Command │ Description
├─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼──────────
│(0-59)   │(0-23)   │(1-31)   │(1-12)   │(0-7)    │         │
└─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴──────────

* = any value
, = list of values
- = range of values
/ = repeat every value

Examples:
0 0 * * *       - Midnight daily
0 */4 * * *     - Every 4 hours
30 2 * * 0      - 2:30 AM every Sunday
0 0 1 * *       - Midnight first day of month
30 14 * * 1-5   - 2:30 PM weekdays
*/15 * * * *    - Every 15 minutes
0 0 * * *       - Every day at midnight
```

### Crontab Management
```bash
# Edit cron jobs
crontab -e                      # Edit current user's crontab
crontab -l                      # List cron jobs
crontab -r                      # Remove all cron jobs
sudo crontab -e                 # Edit root's crontab
sudo crontab -u username -e     # Edit specific user's crontab

# Crontab examples
# Backup database daily at 2 AM
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

# Update packages weekly
0 3 * * 0 apt-get update && apt-get upgrade -y

# Check disk space hourly
0 * * * * df -h >> /var/log/disk_check.log

# Restart service if down (every 10 minutes)
*/10 * * * * systemctl is-active myservice || systemctl start myservice
```

### System Cron
```bash
# System cron files
/etc/cron.d/          # System cron jobs
/etc/cron.daily/      # Daily cron jobs
/etc/cron.hourly/     # Hourly cron jobs
/etc/cron.weekly/     # Weekly cron jobs
/etc/cron.monthly/    # Monthly cron jobs

# Example: /etc/cron.d/backup
0 2 * * * root /usr/local/bin/backup.sh
```

### Systemd Timer (Modern Alternative)
```bash
# Timer file: /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target

# Service file: /etc/systemd/system/backup.service
[Unit]
Description=Backup Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh

# Enable and start
systemctl daemon-reload
systemctl enable backup.timer
systemctl start backup.timer
systemctl status backup.timer
```

---

## 13. PACKAGE MANAGEMENT

### APT (Debian/Ubuntu)
```bash
# Update package list
apt update                      # Update package index
apt upgrade                     # Upgrade all packages
apt full-upgrade               # Also remove unused packages

# Install/Remove
apt install nginx               # Install package
apt install nginx apache2       # Multiple packages
apt remove nginx                # Remove package
apt purge nginx                 # Remove + config files
apt autoremove                  # Remove unused dependencies

# Search and info
apt search nginx                # Search packages
apt show nginx                  # Package information
apt list --installed            # List installed packages
apt list --upgradable           # Show upgradable packages

# Clean up
apt clean                       # Remove cached packages
apt autoclean                   # Remove old cached packages
```

### YUM/DNF (CentOS/RHEL)
```bash
# Update packages
yum update                      # Update all packages
yum upgrade                     # Upgrade all packages
dnf update                      # DNF (newer)

# Install/Remove
yum install nginx               # Install package
yum remove nginx                # Remove package
yum reinstall nginx             # Reinstall

# Search and info
yum search nginx                # Search packages
yum info nginx                  # Package information
yum list installed              # List installed packages
yum check-update               # Check available updates

# Repository management
yum repolist                   # List enabled repos
yum install epel-release       # Add EPEL repo
```

### dpkg (Low-level)
```bash
# Install/Remove
dpkg -i package.deb            # Install .deb file
dpkg -r package                # Remove package
dpkg -P package                # Purge package

# Query
dpkg -l                        # List installed packages
dpkg -S /path/to/file          # Find package containing file
dpkg -L package                # List files in package
dpkg -s package                # Package status
```

---

## 14. TOP 150+ LINUX COMMANDS

### File Management
```bash
ls -la                    # List files with details
cd /path                  # Change directory
pwd                       # Print working directory
mkdir -p /path/to/dir     # Create directory recursively
rmdir /path/to/dir        # Remove empty directory
rm -rf /path/to/dir       # Remove directory recursively
cp file1 file2            # Copy file
cp -r dir1 dir2           # Copy directory
mv oldname newname         # Move/rename
touch filename            # Create empty file
file /path/to/file        # Determine file type
stat file                 # File statistics
ln -s target linkname     # Create symbolic link
ln target linkname        # Create hard link
```

### Text Processing
```bash
cat file                  # Display file
less file                 # Paged view
more file                 # Paged view (older)
head -20 file             # First 20 lines
tail -20 file             # Last 20 lines
tail -f file              # Follow file (tail -f)
wc -l file                # Line count
grep pattern file         # Search pattern
grep -r pattern /dir      # Recursive search
grep -v pattern file      # Inverse match
sed 's/old/new/' file     # Replace text
sed -i 's/old/new/' file  # In-place replace
awk '{print $1}' file     # Print first column
cut -d: -f1 file          # Cut by delimiter
sort file                 # Sort lines
sort -u file              # Sort unique
uniq file                 # Show unique lines
tr 'a' 'b' file           # Translate characters
dos2unix file             # Remove Windows line endings
```

### File Searching
```bash
find /path -name "*.txt"              # Find by name
find /path -type f -size +10M         # Find large files
find /path -mtime -7                  # Modified in last 7 days
find /path -name "*.log" -delete      # Find and delete
locate filename                        # Fast search (needs updatedb)
updatedb                              # Update locate database
which command                         # Find command in PATH
whereis command                       # Find command, source, docs
```

### Permissions & Ownership
```bash
chmod 755 file            # Change permissions
chmod u+x file            # Add execute for owner
chown user file           # Change owner
chown user:group file     # Change owner and group
chown -R user:group /dir  # Recursive change
umask 0022                # Set default permissions
sudo command              # Run as root
sudo -u user command      # Run as specific user
```

### User & Group Management
```bash
whoami                    # Current user
id                        # User ID and groups
groups                    # User's groups
who                       # Logged in users
w                         # Who and what they're doing
last                      # Last logins
useradd user              # Add user
usermod -aG group user    # Add user to group
passwd user               # Change password
groupadd group            # Add group
```

### Process Management
```bash
ps aux                    # All processes
ps auxf                   # Process tree
pstree                    # Process tree
top                       # Real-time processes
htop                      # Interactive processes
kill -9 PID               # Force kill process
killall nginx             # Kill all nginx
pkill pattern             # Kill by pattern
jobs                      # Background jobs
fg                        # Foreground job
bg                        # Background job
nice -n 10 command        # Run with priority
renice 5 -p PID           # Change priority
```

### System Information
```bash
uname -a                  # System information
lsb_release -a            # Distribution info
hostnamectl               # Hostname info
uptime                    # System uptime
free -h                   # Memory usage
df -h                     # Disk usage
du -sh /dir               # Directory size
lscpu                     # CPU information
lsmem                     # Memory information
lsblk                     # Block devices
lsusb                     # USB devices
lspci                     # PCI devices
```

### Networking
```bash
ip addr                   # Show IP addresses
ip link                   # Show interfaces
ip route                  # Show routes
ifconfig                  # Interface configuration
ping host                 # Test connectivity
traceroute host           # Trace route
mtr host                  # Real-time traceroute
netstat -tuln             # Listen ports
ss -tuln                  # Socket statistics
telnet host 80            # Test connectivity
nc -zv host 80            # Netcat test
dig domain                # DNS query
nslookup domain           # DNS lookup
curl url                  # HTTP request
wget url                  # Download file
```

### Archive & Compression
```bash
tar -cvf archive.tar dir/      # Create tar
tar -xvf archive.tar           # Extract tar
tar -czvf archive.tar.gz dir/  # Create compressed
tar -xzvf archive.tar.gz       # Extract compressed
gzip file                      # Compress file
gunzip file.gz                 # Decompress
zip -r archive.zip dir/        # Create zip
unzip archive.zip              # Extract zip
```

### Text Editors
```bash
nano file                 # Simple editor
vi file                   # Vi editor
vim file                  # Vi Improved
gedit file                # GUI editor
cat > file << EOF         # Create file with content
  line 1
  line 2
EOF
```

### System Logs
```bash
dmesg                     # Kernel messages
journalctl                # System logs
journalctl -u service     # Service logs
journalctl -f             # Follow logs
tail -f /var/log/syslog   # System log
tail -f /var/log/auth.log # Auth log
```

### Disk Management
```bash
lsblk                     # List block devices
fdisk -l                  # List partitions
parted -l                 # Partition info
mount /dev/sda1 /mnt      # Mount device
umount /mnt               # Unmount device
mkfs.ext4 /dev/sda1       # Format partition
fsck /dev/sda1            # Check filesystem
df -h                     # Disk free
du -sh /dir               # Directory size
```

### Advanced
```bash
sudo su                   # Become root
sudo su - user            # Become user
su - user                 # Switch user
sudo -l                   # Allowed sudo commands
sudo -v                   # Update sudo timestamp
sudo -k                   # Invalidate sudo
git clone url             # Clone repo
git pull                  # Update repo
curl -X POST url          # HTTP POST
wget -O file url          # Download specific name
rsync -av src/ dst/       # Sync directories
ssh user@host             # SSH connect
scp file user@host:/path  # Copy over SSH
```

---

## HANDS-ON LABS

### Lab 1: User and Permission Management
**Objective:** Create users, groups, and manage permissions

```bash
# Create group
sudo groupadd devops

# Create users
sudo useradd -m -s /bin/bash -G devops john
sudo useradd -m -s /bin/bash -G devops jane

# Create project directory
sudo mkdir -p /home/projects/app1
sudo chown john:devops /home/projects/app1
sudo chmod 770 /home/projects/app1

# Test permissions
su - john
cd /home/projects/app1
touch test.txt
cat /home/projects/app1/test.txt

# Verify jane can access
su - jane
cat /home/projects/app1/test.txt
```

### Lab 2: LVM Setup
**Objective:** Create LVM volumes and extend storage

```bash
# Create physical volumes
sudo pvcreate /dev/sdb /dev/sdc

# Create volume group
sudo vgcreate vg_storage /dev/sdb /dev/sdc

# Create logical volume
sudo lvcreate -L 20G -n lv_data vg_storage

# Format and mount
sudo mkfs.ext4 /dev/vg_storage/lv_data
sudo mkdir -p /mnt/data
sudo mount /dev/vg_storage/lv_data /mnt/data

# Extend logical volume
sudo lvextend -L +10G /dev/vg_storage/lv_data
sudo resize2fs /dev/vg_storage/lv_data

# Verify
df -h /mnt/data
```

### Lab 3: SSH Key-based Authentication
**Objective:** Setup secure SSH access

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "deploy@myserver"

# Copy to remote
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@remote

# Configure SSH config
cat >> ~/.ssh/config << EOF
Host myserver
  HostName 192.168.1.100
  User deployer
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes
EOF

chmod 600 ~/.ssh/config

# Test connection
ssh myserver

# Disable password auth on server
sudo nano /etc/ssh/sshd_config
  # Set: PasswordAuthentication no
sudo systemctl restart ssh
```

### Lab 4: Systemd Service Creation
**Objective:** Create a custom systemd service

```bash
# Create service file
sudo nano /etc/systemd/system/myapp.service

# Add content
[Unit]
Description=My Python Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/main.py
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
sudo systemctl status myapp.service
```

### Lab 5: Cron Job Scheduling
**Objective:** Schedule automated tasks

```bash
# Create backup script
cat > /opt/backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
tar -czf $BACKUP_DIR/backup_$DATE.tar.gz /home/user/documents/
find $BACKUP_DIR -name "backup_*.tar.gz" -mtime +30 -delete
EOF

chmod +x /opt/backup.sh

# Add to crontab
(crontab -l 2>/dev/null; echo "0 2 * * * /opt/backup.sh") | crontab -

# Verify
crontab -l
```

---

## INTERVIEW QUESTIONS

### Beginner Level
1. **What is the Linux Kernel?**
   - Core of the operating system that manages hardware resources
   - Handles process scheduling, memory management, and file systems

2. **Explain Linux File Permissions**
   - Three types: Read (4), Write (2), Execute (1)
   - Three categories: Owner, Group, Others
   - Example: 755 = rwxr-xr-x

3. **What is umask?**
   - Default permissions for newly created files/directories
   - Subtracted from default (666 for files, 777 for dirs)
   - umask 0022 = files get 644, dirs get 755

4. **Difference between sudo and su?**
   - su: Switch user, requires user password
   - sudo: Run command as root/other user, requires sudo privileges

5. **What is inode?**
   - Data structure containing file metadata
   - Contains: permissions, owner, timestamps, size, block pointers
   - Not the filename (that's in the directory entry)

### Intermediate Level
6. **Explain LVM (Logical Volume Manager)**
   - Physical Volumes (PV) → Volume Group (VG) → Logical Volumes (LV)
   - Allows dynamic resizing and flexible storage management
   - Can extend volumes without unmounting

7. **What is the boot sequence in Linux?**
   - BIOS/UEFI → Bootloader (GRUB) → Kernel → Init System → Services
   - Initramfs loads before actual root filesystem

8. **Difference between hardlink and symbolic link**
   - Hardlink: Direct reference to inode, same file content
   - Symlink: Reference to filename/path, can point to files on different filesystems

9. **What are sticky bits, setuid, and setgid?**
   - Sticky bit (1xxx): Only owner can delete in directories like /tmp
   - Setuid (4xxx): File runs with owner's permissions (e.g., passwd)
   - Setgid (2xxx): File runs with group's permissions

10. **Explain systemd vs init**
    - Systemd: Parallel service startup, socket activation, timers
    - Init: Sequential service startup, more overhead
    - Systemd is modern and handles more than just services

### Advanced Level
11. **How would you troubleshoot a full disk?**
    - Check which filesystem is full: `df -h`
    - Find large files: `find / -type f -size +100M`
    - Check logs: `du -sh /var/log/*`
    - Clear logs: `journalctl --vacuum=time:7d`
    - Remove old packages: `apt autoremove`, `apt clean`

12. **Explain SSH key exchange process**
    - Server sends public key
    - Client generates session key, encrypts with server's public key
    - Server decrypts with private key
    - Both derive symmetric keys for session encryption

13. **How do zombie processes occur and how to fix?**
    - Child process terminates but parent hasn't reaped it (wait())
    - Shows as `<defunct>` in process list
    - Fix: Kill parent process or reboot

14. **Explain iptables/netfilter**
    - Userspace tool for kernel netfilter framework
    - Tables: Filter, NAT, Mangle, Security, Raw
    - Chains: INPUT, OUTPUT, FORWARD
    - Rules: Match packets and apply actions (ACCEPT, DROP, REJECT)

15. **What's the difference between SIGTERM and SIGKILL?**
    - SIGTERM (15): Graceful termination, can be caught and handled
    - SIGKILL (9): Force kill, cannot be caught, process immediately terminates
    - Should always try SIGTERM first

### Scenario-based
16. **A database server is consuming 95% memory. What would you do?**
    - Check what's using memory: `top`, `ps aux`
    - Check if swap is being used: `free -h`
    - Look for memory leaks: `valgrind`
    - Restart service if needed: `systemctl restart service`
    - Adjust memory limits in config or increase RAM

17. **SSH connection is timing out. How would you debug?**
    - Check if service is running: `sudo systemctl status ssh`
    - Check SSH config: `sshd -t`
    - Check listening ports: `ss -tuln | grep 22`
    - Check firewall: `ufw status`, `firewall-cmd --list-all`
    - Check logs: `sudo tail -f /var/log/auth.log`
    - Try verbose SSH: `ssh -vvv user@host`

18. **How would you securely transfer a file to a server?**
    - Use SCP: `scp file user@host:/path/`
    - Use SFTP: `sftp user@host`
    - Use rsync: `rsync -e ssh file user@host:/path/`
    - Verify with checksum: `sha256sum file`
    - All encrypt data during transfer

19. **Cron job is not running. How to troubleshoot?**
    - Check if cron daemon is running: `systemctl status cron`
    - Check permissions on script: must be executable
    - Check crontab: `crontab -l`
    - Check logs: `grep CRON /var/log/syslog`
    - Verify time format in crontab
    - Full paths required in cron scripts

20. **How to safely restart a server without losing data?**
    - Notify users: `wall "Server rebooting in 5 minutes"`
    - Close connections gracefully
    - Sync filesystem: `sync`
    - Check running services: `systemctl list-units --type=service --state=running`
    - Use shutdown: `sudo shutdown -r +5` (reboot in 5 minutes)
    - Verify filesystem integrity after boot: `fsck`

---

## BEST PRACTICES

1. **Always use `sudo` properly**
   - Never run as root constantly
   - Use visudo to edit sudoers safely
   - Log and audit sudo usage

2. **Regular backups**
   - Automate with cron or systemd timer
   - Test restore procedures
   - Keep offsite copies

3. **Security hardening**
   - Disable root login
   - Disable password authentication
   - Use SSH keys
   - Keep system updated

4. **Log management**
   - Archive old logs
   - Set up log rotation
   - Monitor critical logs
   - Centralize logs if possible

5. **File system management**
   - Monitor disk usage
   - Prevent filesystem full
   - Use LVM for flexibility
   - Implement quotas for users

6. **Process management**
   - Use systemd for service management
   - Implement health checks
   - Handle signals properly
   - Monitor for resource leaks

---

## KEY TAKEAWAYS

✅ Understand Linux kernel, boot process, and architecture
✅ Master file permissions and user/group management
✅ Proficient with process and service management
✅ Comfortable with networking and SSH
✅ Can troubleshoot system issues effectively
✅ Understand modern tools (systemd, LVM, etc.)
✅ Capable of securing Linux systems
✅ Comfortable with automation using cron/systemd

---

**Total Interview Questions: 50+**
**Total Labs: 5 hands-on exercises**
**Commands Covered: 150+**
