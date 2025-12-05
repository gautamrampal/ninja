# Linux Commands Complete Mastery Tutorial - Part 2
## Advanced Topics: Networking, System Administration & Security

---

## Table of Contents - Part 2

- [17. Networking Commands](#17-networking-commands)
- [18. Network Diagnostics and Troubleshooting](#18-network-diagnostics-and-troubleshooting)
- [19. System Monitoring and Performance](#19-system-monitoring-and-performance)
- [20. Shell Scripting Essentials](#20-shell-scripting-essentials)
- [21. Advanced Text Processing](#21-advanced-text-processing)
- [22. System Administration](#22-system-administration)
- [23. Security and Firewall](#23-security-and-firewall)
- [24. Kernel and System Tuning](#24-kernel-and-system-tuning)
- [25. Advanced Networking](#25-advanced-networking)

---

# Part 3: Advanced Operations (Advanced Level)

## 16. Disk and Storage Management

### `mount` - Mount Filesystems
**Description**: Mounts a filesystem to a directory.

**Syntax**:
```bash
mount [OPTIONS] DEVICE MOUNTPOINT
```

**Options**:
- `-t TYPE`: Filesystem type (ext4, xfs, ntfs, vfat, etc.)
- `-o OPTIONS`: Mount options
- `-a`: Mount all filesystems in /etc/fstab
- `-r`: Read-only
- `-w`: Read-write (default)

**Common Mount Options**:
- `ro`: Read-only
- `rw`: Read-write
- `noexec`: Don't allow execution
- `nosuid`: Ignore setuid/setgid bits
- `noatime`: Don't update access times
- `remount`: Remount with different options
- `loop`: Mount as loop device

**Examples**:
```bash
# List mounted filesystems
mount

# Mount device to directory
sudo mount /dev/sdb1 /mnt/usb

# Mount with specific type
sudo mount -t ntfs /dev/sdb1 /mnt/windows

# Mount read-only
sudo mount -o ro /dev/sdb1 /mnt/backup

# Mount ISO file
sudo mount -o loop image.iso /mnt/iso

# Remount with different options
sudo mount -o remount,rw /dev/sdb1

# Mount with multiple options
sudo mount -o rw,noexec,nosuid /dev/sdb1 /mnt/data

# Mount all from /etc/fstab
sudo mount -a

# Mount by UUID
sudo mount UUID=1234-5678 /mnt/disk

# Mount by label
sudo mount LABEL=BACKUP /mnt/backup

# Bind mount (mount directory to another location)
sudo mount --bind /source /destination

# Network filesystem (NFS)
sudo mount -t nfs server:/path /mnt/nfs

# CIFS/SMB (Windows share)
sudo mount -t cifs //server/share /mnt/share -o username=user,password=pass
```

---

### `umount` - Unmount Filesystems
**Description**: Unmounts a mounted filesystem.

**Syntax**:
```bash\numount [OPTIONS] DEVICE|MOUNTPOINT
```

**Options**:
- `-f`: Force unmount
- `-l`: Lazy unmount (detach now, cleanup later)
- `-a`: Unmount all

**Examples**:
```bash
# Unmount by mount point
sudo umount /mnt/usb

# Unmount by device
sudo umount /dev/sdb1

# Force unmount
sudo umount -f /mnt/usb

# Lazy unmount
sudo umount -l /mnt/usb

# Unmount all
sudo umount -a

# Check what's using the filesystem
lsof /mnt/usb
fuser -m /mnt/usb

# Kill processes using filesystem
sudo fuser -km /mnt/usb
```

---

### `fdisk` - Partition Disk
**Description**: Manipulates disk partition table.

**Syntax**:
```bash
fdisk [OPTIONS] DEVICE
```

**⚠️ Warning**: fdisk modifies partitions. Data loss risk!

**Examples**:
```bash
# List partitions
sudo fdisk -l
sudo fdisk -l /dev/sda

# Interactive mode
sudo fdisk /dev/sdb

# Inside fdisk:
m       # Help
p       # Print partition table
n       # New partition
d       # Delete partition
t       # Change partition type
w       # Write changes and exit
q       # Quit without saving
```

---

### `parted` - Advanced Partitioning
**Description**: More advanced partitioning tool.

**Examples**:
```bash
# List partitions
sudo parted -l

# Interactive mode
sudo parted /dev/sdb

# Inside parted:
print          # Show partitions
mkpart         # Create partition
rm <number>    # Delete partition
resize         # Resize partition
quit           # Exit

# Non-interactive
sudo parted /dev/sdb mkpart primary ext4 0% 100%
```

---

### `mkfs` - Create Filesystem
**Description**: Creates a filesystem on a partition.

**Syntax**:
```bash
mkfs [OPTIONS] DEVICE
```

**Filesystem Types**:
```bash
# ext4
sudo mkfs.ext4 /dev/sdb1

# ext3
sudo mkfs.ext3 /dev/sdb1

# xfs
sudo mkfs.xfs /dev/sdb1

# FAT32
sudo mkfs.vfat /dev/sdb1

# NTFS
sudo mkfs.ntfs /dev/sdb1

# With label
sudo mkfs.ext4 -L MyDisk /dev/sdb1

# With parameters
sudo mkfs.ext4 -b 4096 -m 1 /dev/sdb1
```

---

### `fsck` - Check and Repair Filesystem
**Description**: Checks and repairs filesystem errors.

**Syntax**:
```bash
fsck [OPTIONS] DEVICE
```

**Options**:
- `-a`: Automatically repair
- `-r`: Interactive repair
- `-y`: Yes to all prompts
- `-n`: No modifications (check only)
- `-f`: Force check

**⚠️ Warning**: Never run fsck on mounted filesystem!

**Examples**:
```bash
# Check filesystem
sudo fsck /dev/sdb1

# Auto repair
sudo fsck -y /dev/sdb1

# Force check
sudo fsck -f /dev/sdb1

# Check all filesystems in /etc/fstab
sudo fsck -A

# Specific filesystem type
sudo fsck.ext4 /dev/sdb1
sudo fsck.xfs /dev/sdb2
```

---

### `blkid` - Block Device Attributes
**Description**: Shows block device attributes (UUID, filesystem type).

**Examples**:
```bash
# List all block devices
sudo blkid

# Specific device
sudo blkid /dev/sda1

# Output:
# /dev/sda1: UUID="1234-5678" TYPE="ext4" PARTUUID="..."

# Get UUID only
sudo blkid -s UUID -o value /dev/sda1
```

---

### `dd` - Convert and Copy Files
**Description**: Low-level copy and convert data.

**Syntax**:
```bash
dd if=INPUT of=OUTPUT [OPTIONS]
```

**Options**:
- `if=FILE`: Input file
- `of=FILE`: Output file
- `bs=BYTES`: Block size
- `count=N`: Copy only N blocks
- `status=progress`: Show progress
- `conv=OPTIONS`: Convert options

**⚠️ Warning**: dd is powerful and dangerous! Double-check before running!

**Examples**:
```bash
# Create bootable USB
sudo dd if=ubuntu.iso of=/dev/sdb bs=4M status=progress

# Backup partition
sudo dd if=/dev/sda1 of=backup.img bs=4M

# Restore partition
sudo dd if=backup.img of=/dev/sda1 bs=4M

# Clone entire disk
sudo dd if=/dev/sda of=/dev/sdb bs=64K conv=noerror,sync

# Create file filled with zeros
dd if=/dev/zero of=file.txt bs=1M count=100

# Create file filled with random data
dd if=/dev/urandom of=random.dat bs=1M count=10

# Wipe disk (DANGEROUS!)
sudo dd if=/dev/zero of=/dev/sdb bs=1M status=progress

# Benchmark disk write speed
dd if=/dev/zero of=testfile bs=1G count=1 oflag=dsync

# Create ISO from CD/DVD
dd if=/dev/cdrom of=disk.iso bs=2048
```

---

### `rsync` - Remote Sync
**Description**: Efficiently syncs files and directories.

**Syntax**:
```bash
rsync [OPTIONS] SOURCE DESTINATION
```

**Options**:
- `-a`: Archive mode (preserves everything)
- `-v`: Verbose
- `-z`: Compress during transfer
- `-h`: Human-readable
- `-P`: Show progress
- `-r`: Recursive
- `-n`: Dry run
- `--delete`: Delete files in destination
- `--exclude=PATTERN`: Exclude pattern
- `--include=PATTERN`: Include pattern

**Examples**:
```bash
# Sync directories locally
rsync -av /source/ /destination/

# Sync with progress
rsync -avP /source/ /destination/

# Dry run (test without changes)
rsync -avn /source/ /destination/

# Sync to remote server
rsync -avz /local/ user@remote:/path/

# Sync from remote server
rsync -avz user@remote:/path/ /local/

# Delete files in destination not in source
rsync -av --delete /source/ /destination/

# Exclude files
rsync -av --exclude='*.log' /source/ /destination/
rsync -av --exclude-from=exclude.txt /source/ /destination/

# Include specific files
rsync -av --include='*.txt' --exclude='*' /source/ /destination/

# Sync over SSH on different port
rsync -av -e "ssh -p 2222" /local/ user@remote:/path/

# Show what would be transferred
rsync -avn --stats /source/ /destination/

# Bandwidth limit (KB/s)
rsync -av --bwlimit=1000 /source/ /destination/

# Partial transfers (resume interrupted)
rsync -avP --partial /source/ /destination/

# Backup with hardlinks (save space)
rsync -av --link-dest=/backup/previous /source/ /backup/current/
```

---

## 17. Networking Commands

### `ip` - Network Configuration
**Description**: Modern network configuration tool (replaces ifconfig).

**Syntax**:
```bash
ip [OPTIONS] OBJECT COMMAND
```

**Common Objects**:
- `link`: Network devices
- `addr`: IP addresses
- `route`: Routing table
- `neigh`: ARP/NDISC cache

**Examples**:
```bash
# Show all interfaces
ip link show
ip link

# Show IP addresses
ip addr show
ip addr
ip a            # Short form

# Specific interface
ip addr show eth0

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Add IP address
sudo ip addr add 192.168.1.100/24 dev eth0

# Delete IP address
sudo ip addr del 192.168.1.100/24 dev eth0

# Show routing table
ip route show
ip route
ip r            # Short form

# Add default route
sudo ip route add default via 192.168.1.1

# Add static route
sudo ip route add 10.0.0.0/8 via 192.168.1.254

# Delete route
sudo ip route del 10.0.0.0/8

# Show ARP cache
ip neigh show
ip neigh

# Show statistics
ip -s link
ip -s link show eth0

# Monitor real-time changes
ip monitor
```

---

### `ifconfig` - Interface Configuration (Legacy)
**Description**: Network interface configuration (older tool).

**Examples**:
```bash
# Show all interfaces
ifconfig

# Show specific interface
ifconfig eth0

# Bring interface up/down
sudo ifconfig eth0 up
sudo ifconfig eth0 down

# Assign IP address
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# Add alias interface
sudo ifconfig eth0:0 192.168.1.101
```

**Note**: `ip` command is recommended over `ifconfig`.

---

### `ping` - Test Network Connectivity
**Description**: Sends ICMP echo requests to test connectivity.

**Syntax**:
```bash
ping [OPTIONS] DESTINATION
```

**Options**:
- `-c COUNT`: Number of packets to send
- `-i INTERVAL`: Wait interval between packets
- `-s SIZE`: Packet size
- `-w DEADLINE`: Timeout in seconds
- `-W TIMEOUT`: Wait time for response
- `-4`: Force IPv4
- `-6`: Force IPv6
- `-f`: Flood ping (requires root)

**Examples**:
```bash
# Ping host (Ctrl+C to stop)
ping google.com

# Ping 4 times
ping -c 4 google.com

# Ping with interval
ping -c 10 -i 0.5 google.com    # Every 0.5 seconds

# Ping with timeout
ping -c 4 -W 2 192.168.1.1       # 2 second timeout

# Ping with larger packets
ping -c 4 -s 1000 google.com     # 1000 byte packets

# IPv6 ping
ping6 ipv6.google.com
ping -6 ipv6.google.com

# Flood ping (test maximum throughput)
sudo ping -f google.com

# Ping and timestamp
ping -c 4 -D google.com

# Quiet output (summary only)
ping -c 4 -q google.com
```

---

### `traceroute` / `tracepath` - Trace Route
**Description**: Shows network path to destination.

**Syntax**:
```bash
traceroute [OPTIONS] DESTINATION
tracepath DESTINATION
```

**Examples**:
```bash
# Trace route to host
traceroute google.com

# With ICMP (requires root)
sudo traceroute -I google.com

# With UDP (default)
traceroute google.com

# Max hops
traceroute -m 15 google.com

# No DNS resolution
traceroute -n google.com

# IPv6
traceroute6 ipv6.google.com

# tracepath (doesn't require root)
tracepath google.com
```

---

### `netstat` - Network Statistics (Legacy)
**Description**: Displays network connections and statistics.

**Syntax**:
```bash
netstat [OPTIONS]
```

**Options**:
- `-a`: Show all sockets
- `-t`: TCP connections
- `-u`: UDP connections
- `-l`: Listening sockets
- `-n`: Numeric (don't resolve names)
- `-p`: Show program/PID
- `-r`: Routing table
- `-i`: Interface statistics
- `-s`: Protocol statistics

**Examples**:
```bash
# All connections
netstat -a

# TCP connections
netstat -t

# Listening ports
netstat -l
netstat -lt     # TCP listening
netstat -lu     # UDP listening

# Listening with programs
sudo netstat -tlnp

# All connections with programs
sudo netstat -tunap

# Routing table
netstat -r

# Interface statistics
netstat -i

# Protocol statistics
netstat -s

# Continuous output
netstat -c
```

**Note**: `ss` command is the modern replacement for `netstat`.

---

### `ss` - Socket Statistics
**Description**: Modern tool for socket statistics (replaces netstat).

**Syntax**:
```bash
ss [OPTIONS]
```

**Options**:
- `-a`: Show all sockets
- `-t`: TCP sockets
- `-u`: UDP sockets
- `-l`: Listening sockets
- `-n`: Numeric (don't resolve)
- `-p`: Show processes
- `-s`: Summary statistics
- `-4`: IPv4 only
- `-6`: IPv6 only

**Examples**:
```bash
# All sockets
ss -a

# Listening TCP ports
ss -tln

# Listening TCP ports with programs
sudo ss -tlnp

# All TCP connections
ss -ta

# All UDP connections
ss -ua

# Summary statistics
ss -s

# Specific port
ss -tan sport = :80
ss -tan dport = :443

# Connections to specific IP
ss dst 192.168.1.100

# State filter
ss state established
ss state listening
ss state close-wait

# Socket memory usage
ss -tm
```

---

### `nslookup` - DNS Lookup
**Description**: Queries DNS servers.

**Syntax**:
```bash
nslookup [OPTIONS] DOMAIN [SERVER]
```

**Examples**:
```bash
# Basic lookup
nslookup google.com

# Use specific DNS server
nslookup google.com 8.8.8.8

# Reverse lookup
nslookup 8.8.8.8

# Query specific record type
nslookup -type=MX google.com    # Mail exchange
nslookup -type=NS google.com    # Name servers
nslookup -type=A google.com     # IPv4 address
nslookup -type=AAAA google.com  # IPv6 address
nslookup -type=TXT google.com   # Text records
nslookup -type=SOA google.com   # Start of authority

# Interactive mode
nslookup
> server 8.8.8.8
> google.com
> exit
```

---

### `dig` - DNS Lookup (Advanced)
**Description**: Advanced DNS lookup tool.

**Syntax**:
```bash
dig [OPTIONS] DOMAIN [TYPE]
```

**Examples**:
```bash
# Basic lookup
dig google.com

# Short answer
dig google.com +short

# Specific record type
dig google.com A
dig google.com MX
dig google.com NS
dig google.com TXT
dig google.com AAAA

# Use specific DNS server
dig @8.8.8.8 google.com

# Reverse lookup
dig -x 8.8.8.8

# Trace DNS delegation path
dig google.com +trace

# Query all record types
dig google.com ANY

# No comments in output
dig google.com +nocomments

# Only answer section
dig google.com +noall +answer

# Multiple queries
dig google.com facebook.com

# Query with specific port
dig @8.8.8.8 -p 5353 google.com
```

---

### `host` - DNS Lookup (Simple)
**Description**: Simple DNS lookup tool.

**Examples**:
```bash
# Basic lookup
host google.com

# Reverse lookup
host 8.8.8.8

# Specific record type
host -t MX google.com
host -t NS google.com
host -t A google.com

# Use specific DNS server
host google.com 8.8.8.8

# Verbose output
host -v google.com

# All record types
host -a google.com
```

---

### `wget` - Download Files
**Description**: Downloads files from the web.

**Syntax**:
```bash
wget [OPTIONS] URL
```

**Options**:
- `-O FILE`: Save as filename
- `-P DIR`: Save to directory
- `-c`: Continue partial download
- `-r`: Recursive download
- `-np`: No parent directories
- `-q`: Quiet mode
- `-b`: Background download
- `--limit-rate=RATE`: Limit download speed
- `--user=USER`: HTTP authentication
- `--password=PASS`: HTTP authentication

**Examples**:
```bash
# Download file
wget https://example.com/file.zip

# Save with different name
wget -O newname.zip https://example.com/file.zip

# Save to directory
wget -P /downloads https://example.com/file.zip

# Continue interrupted download
wget -c https://example.com/largefile.iso

# Download in background
wget -b https://example.com/file.zip

# Limit download speed
wget --limit-rate=500k https://example.com/file.zip

# Download entire website
wget -r -np https://example.com/

# Download with authentication
wget --user=username --password=pass https://example.com/file.zip

# Read URLs from file
wget -i urls.txt

# Mirror website
wget --mirror -p --convert-links -P ./website https://example.com/

# Download with custom user agent
wget --user-agent="Mozilla/5.0" https://example.com/file.zip

# Retry on failure
wget --tries=10 https://example.com/file.zip

# Timeout
wget --timeout=30 https://example.com/file.zip
```

---

### `curl` - Transfer Data
**Description**: Transfers data from or to a server.

**Syntax**:
```bash
curl [OPTIONS] URL
```

**Options**:
- `-o FILE`: Save to file
- `-O`: Save with remote filename
- `-C -`: Resume download
- `-L`: Follow redirects
- `-I`: Headers only
- `-H HEADER`: Custom header
- `-d DATA`: POST data
- `-X METHOD`: HTTP method
- `-u USER:PASS`: Authentication
- `-s`: Silent mode
- `-v`: Verbose

**Examples**:
```bash
# Download file
curl -O https://example.com/file.zip

# Save with custom name
curl -o custom.zip https://example.com/file.zip

# Follow redirects
curl -L https://example.com/redirect

# Show headers only
curl -I https://example.com

# Download and show progress
curl -# -O https://example.com/file.zip

# Resume download
curl -C - -O https://example.com/file.zip

# POST request
curl -X POST -d "param1=value1&param2=value2" https://api.example.com

# POST JSON
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com

# Custom header
curl -H "Authorization: Bearer TOKEN" https://api.example.com

# Basic authentication
curl -u username:password https://example.com

# Multiple URLs
curl -O https://example.com/file1.zip -O https://example.com/file2.zip

# Follow redirects and save cookies
curl -L -c cookies.txt https://example.com

# Send cookies
curl -b cookies.txt https://example.com

# Upload file
curl -F "file=@/path/to/file" https://example.com/upload

# Test API endpoint
curl -X GET https://api.example.com/users

# Show only response body (silent)
curl -s https://api.example.com/data

# Verbose (show request/response)
curl -v https://example.com
```

---

## 18. Network Diagnostics and Troubleshooting

### `tcpdump` - Packet Analyzer
**Description**: Captures and analyzes network packets.

**Syntax**:
```bash
tcpdump [OPTIONS]
```

**Options**:
- `-i INTERFACE`: Capture on interface
- `-c COUNT`: Capture COUNT packets
- `-w FILE`: Write to file
- `-r FILE`: Read from file
- `-n`: Don't resolve hostnames
- `-nn`: Don't resolve hostnames or ports
- `-v`, `-vv`, `-vvv`: Verbosity levels
- `-A`: Print packets in ASCII
- `-X`: Print packets in hex and ASCII

**Examples**:
```bash
# Capture on all interfaces
sudo tcpdump

# Specific interface
sudo tcpdump -i eth0

# Capture 100 packets
sudo tcpdump -c 100

# Don't resolve names
sudo tcpdump -n

# Save to file
sudo tcpdump -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Specific host
sudo tcpdump host 192.168.1.100

# Specific port
sudo tcpdump port 80
sudo tcpdump port 22 or port 80

# TCP traffic only
sudo tcpdump tcp

# UDP traffic
sudo tcpdump udp

# ICMP traffic (ping)
sudo tcpdump icmp

# Source/destination
sudo tcpdump src 192.168.1.100
sudo tcpdump dst 192.168.1.100

# Network range
sudo tcpdump net 192.168.1.0/24

# Capture HTTP traffic
sudo tcpdump -i eth0 port 80 -A

# Capture and show ASCII
sudo tcpdump -A -s 0 port 80

# Complex filter
sudo tcpdump 'tcp port 80 and (host 192.168.1.100 or host 192.168.1.101)'
```

---

### `nmap` - Network Scanner
**Description**: Network exploration and security auditing.

**Installation**:
```bash
sudo apt install nmap    # Ubuntu/Debian
sudo yum install nmap    # CentOS/RHEL
```

**Examples**:
```bash
# Scan single host
nmap 192.168.1.1

# Scan multiple hosts
nmap 192.168.1.1 192.168.1.2
nmap 192.168.1.1-10

# Scan subnet
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 22,80,443 192.168.1.1
nmap -p 1-1000 192.168.1.1
nmap -p- 192.168.1.1        # All ports

# Service version detection
nmap -sV 192.168.1.1

# OS detection
sudo nmap -O 192.168.1.1

# Aggressive scan
sudo nmap -A 192.168.1.1

# Fast scan (common ports)
nmap -F 192.168.1.1

# Ping scan (check if hosts are up)
nmap -sn 192.168.1.0/24

# TCP SYN scan (stealth)
sudo nmap -sS 192.168.1.1

# UDP scan
sudo nmap -sU 192.168.1.1

# No ping (assume host is up)
nmap -Pn 192.168.1.1

# Save output
nmap -oN output.txt 192.168.1.1
nmap -oX output.xml 192.168.1.1

# Verbose output
nmap -v 192.168.1.1
```

---

### `netcat` / `nc` - Network Swiss Army Knife
**Description**: Versatile networking tool.

**Examples**:
```bash
# Connect to port
nc example.com 80

# Listen on port
nc -l 8080

# Port scanning
nc -zv 192.168.1.1 20-80

# Transfer file
# Receiver:
nc -l 8080 > received_file
# Sender:
nc 192.168.1.100 8080 < file_to_send

# Chat between machines
# Machine 1:
nc -l 8080
# Machine 2:
nc 192.168.1.100 8080

# Simple web server
while true; do echo -e "HTTP/1.1 200 OK\n\n$(date)" | nc -l 8080; done

# Test if port is open
nc -zv google.com 443

# UDP mode
nc -u host port

# Keep connection open
nc -k -l 8080
```

---

### `telnet` - Telnet Client
**Description**: Tests connectivity to TCP ports.

**Examples**:
```bash
# Connect to port
telnet example.com 80

# Test SMTP
telnet mail.example.com 25

# Test HTTP
telnet example.com 80
GET / HTTP/1.1
Host: example.com
# (Press Enter twice)

# Exit telnet
Ctrl+]
quit
```

---

### `mtr` - Network Diagnostic Tool
**Description**: Combines ping and traceroute.

**Installation**:
```bash
sudo apt install mtr    # Ubuntu/Debian
```

**Examples**:
```bash
# Interactive mode
mtr google.com

# Report mode (10 cycles)
mtr --report google.com
mtr -c 10 --report google.com

# No DNS resolution
mtr -n google.com

# TCP instead of ICMP
mtr --tcp google.com

# Specific port
mtr --tcp --port 443 google.com
```

---

## 19. System Monitoring and Performance

### `vmstat` - Virtual Memory Statistics
**Description**: Reports virtual memory statistics.

**Syntax**:
```bash
vmstat [delay [count]]
```

**Examples**:
```bash
# One-time report
vmstat

# Update every 2 seconds
vmstat 2

# 5 reports, 2 seconds apart
vmstat 2 5

# Memory statistics in MB
vmstat -S M 2

# Disk statistics
vmstat -d

# Full statistics
vmstat -s
```

**Output Columns**:
- **procs r**: Running processes
- **procs b**: Blocked processes
- **memory**: Memory usage
- **swap**: Swap usage
- **io**: I/O operations
- **system**: System calls/context switches
- **cpu**: CPU usage percentages

---

### `iostat` - I/O Statistics
**Description**: Reports CPU and I/O statistics.

**Installation**:
```bash
sudo apt install sysstat
```

**Examples**:
```bash
# Basic report
iostat

# Extended statistics
iostat -x

# Human-readable
iostat -h

# Update every 2 seconds
iostat 2

# Device statistics only
iostat -d

# CPU statistics only
iostat -c

# Specific devices
iostat sda sdb

# MB instead of blocks
iostat -m
```

---

### `iotop` - I/O Monitor (Top-like)
**Description**: Shows I/O usage by process (like top for I/O).

**Installation**:
```bash
sudo apt install iotop
```

**Examples**:
```bash
# Launch iotop
sudo iotop

# Only show processes doing I/O
sudo iotop -o

# Non-interactive (batch mode)
sudo iotop -b -n 5

# Show accumulated I/O
sudo iotop -a
```

---

### `sar` - System Activity Reporter
**Description**: Collects and reports system activity.

**Installation**:
```bash
sudo apt install sysstat
```

**Examples**:
```bash
# CPU usage (every 2 seconds, 5 times)
sar 2 5

# Memory usage
sar -r

# Swap usage
sar -S

# I/O statistics
sar -b

# Network statistics
sar -n DEV

# All statistics
sar -A

# Historical data (from logs)
sar -f /var/log/sysstat/sa$(date +%d)

# CPU usage from specific time
sar -s 10:00:00 -e 11:00:00
```

---

### `lsof` - List Open Files
**Description**: Lists open files and processes using them.

**Examples**:
```bash
# All open files
sudo lsof

# Files opened by user
lsof -u username

# Files opened by process
lsof -p 1234

# Files opened by command
lsof -c nginx

# Network connections
sudo lsof -i

# Specific port
sudo lsof -i :80
sudo lsof -i :22

# TCP connections
sudo lsof -i TCP

# UDP connections
sudo lsof -i UDP

# Listening ports
sudo lsof -i -sTCP:LISTEN

# Files in directory
lsof +D /var/log

# Deleted files still open
lsof | grep deleted

# Check what's using a mount point
lsof /mnt/usb

# Multiple conditions
sudo lsof -u username -i TCP:80
```

---

### `strace` - Trace System Calls
**Description**: Traces system calls and signals.

**Examples**:
```bash
# Trace command
strace ls

# Trace running process
sudo strace -p 1234

# Save to file
strace -o output.txt ls

# Trace specific syscalls
strace -e open,read,write ls

# Show timestamps
strace -t ls
strace -tt ls       # With microseconds

# Count syscalls
strace -c ls

# Follow child processes
strace -f command

# Show syscall statistics
strace -c command
```

---

### `ltrace` - Library Call Tracer
**Description**: Traces library function calls.

**Examples**:
```bash
# Trace library calls
ltrace ls

# Save to file
ltrace -o output.txt command

# Count calls
ltrace -c command

# Trace specific functions
ltrace -e malloc,free command
```

---

## 20. Shell Scripting Essentials

### Script Basics

**Shebang**:
```bash
#!/bin/bash       # Use bash
#!/bin/sh         # Use sh
#!/usr/bin/env python3  # Use python3
```

**Make Executable**:
```bash
chmod +x script.sh
./script.sh
```

---

### Variables

**Examples**:
```bash
#!/bin/bash

# Declare variables
NAME="John"
AGE=30
readonly PI=3.14159    # Read-only variable

# Use variables
echo "Name: $NAME"
echo "Name: ${NAME}"   # Preferred

# Command output
DATE=$(date)
FILES=`ls`             # Old style (backticks)

# Arithmetic
COUNT=$((5 + 3))
((COUNT++))
((COUNT += 5))

# User input
read -p "Enter your name: " username
echo "Hello, $username"

# Environment variables
echo $HOME
echo $USER
echo $PATH

# Special variables
echo $0    # Script name
echo $1    # First argument
echo $@    # All arguments
echo $#    # Number of arguments
echo $$    # Process ID
echo $?    # Exit status of last command
```

---

### Conditionals

**Examples**:
```bash
#!/bin/bash

# If statement
if [ "$1" = "start" ]; then
    echo "Starting..."
elif [ "$1" = "stop" ]; then
    echo "Stopping..."
else
    echo "Usage: $0 {start|stop}"
fi

# File tests
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

if [ -d "/var/log" ]; then
    echo "Directory exists"
fi

if [ -r "file.txt" ]; then
    echo "File is readable"
fi

# Numeric comparison
if [ $AGE -gt 18 ]; then
    echo "Adult"
else
    echo "Minor"
fi

# String comparison
if [ "$NAME" = "John" ]; then
    echo "Hello John"
fi

if [ -z "$VAR" ]; then
    echo "Variable is empty"
fi

if [ -n "$VAR" ]; then
    echo "Variable is not empty"
fi

# Logical operators
if [ $AGE -gt 18 ] && [ "$NAME" = "John" ]; then
    echo "Adult named John"
fi

if [ $AGE -lt 18 ] || [ $AGE -gt 65 ]; then
    echo "Not working age"
fi

# Modern test (bash)
if [[ $NAME == "John" ]]; then
    echo "Hello John"
fi

if [[ $NAME == J* ]]; then
    echo "Name starts with J"
fi

# Case statement
case "$1" in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

**Test Operators**:
```
File tests:
  -f file    # File exists and is regular file
  -d dir     # Directory exists
  -e path    # Path exists
  -r file    # File is readable
  -w file    # File is writable
  -x file    # File is executable
  -s file    # File exists and not empty
  -L file    # File is symbolic link

Numeric comparison:
  -eq        # Equal
  -ne        # Not equal
  -gt        # Greater than
  -ge        # Greater than or equal
  -lt        # Less than
  -le        # Less than or equal

String comparison:
  =          # Equal
  !=         # Not equal
  -z         # String is empty
  -n         # String is not empty
  <          # Less than (lexicographically)
  >          # Greater than (lexicographically)
```

---

### Loops

**Examples**:
```bash
#!/bin/bash

# For loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# For with range
for i in {1..10}; do
    echo $i
done

# For with step
for i in {0..100..10}; do
    echo $i
done

# C-style for
for ((i=1; i<=10; i++)); do
    echo $i
done

# Iterate over files
for file in *.txt; do
    echo "Processing $file"
done

# While loop
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# Until loop
COUNT=1
until [ $COUNT -gt 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Break and continue
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue    # Skip 5
    fi
    if [ $i -eq 8 ]; then
        break       # Stop at 8
    fi
    echo $i
done
```

---

### Functions

**Examples**:
```bash
#!/bin/bash

# Define function
greet() {
    echo "Hello, $1!"
}

# Call function
greet "John"

# Function with return value
add() {
    local sum=$(($1 + $2))
    echo $sum
}

result=$(add 5 3)
echo "Result: $result"

# Function with return code
check_file() {
    if [ -f "$1" ]; then
        return 0    # Success
    else
        return 1    # Failure
    fi
}

if check_file "/etc/passwd"; then
    echo "File exists"
fi

# Function with local variables
calculate() {
    local a=$1
    local b=$2
    local result=$((a * b))
    echo $result
}

# Function arguments
process() {
    echo "Function name: $0"
    echo "First arg: $1"
    echo "Second arg: $2"
    echo "All args: $@"
    echo "Number of args: $#"
}

process arg1 arg2 arg3
```

---

### Arrays

**Examples**:
```bash
#!/bin/bash

# Declare array
fruits=("apple" "banana" "orange")

# Access elements
echo ${fruits[0]}       # First element
echo ${fruits[1]}       # Second element
echo ${fruits[@]}       # All elements
echo ${#fruits[@]}      # Number of elements

# Add element
fruits+=("grape")

# Iterate over array
for fruit in "${fruits[@]}"; do
    echo $fruit
done

# Iterate with index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done

# Associative array (bash 4+)
declare -A user
user[name]="John"
user[age]=30
user[city]="NYC"

echo ${user[name]}
echo ${user[@]}         # All values
echo ${!user[@]}        # All keys
```

---

### Input/Output Redirection

**Examples**:
```bash
# Redirect stdout to file
echo "Hello" > file.txt         # Overwrite
echo "World" >> file.txt        # Append

# Redirect stderr
command 2> error.log

# Redirect both stdout and stderr
command > output.log 2>&1
command &> output.log           # Bash shorthand

# Redirect stdin
command < input.txt

# Here document
cat << EOF
Line 1
Line 2
Line 3
EOF

# Here string
command <<< "input data"

# Pipe output
command1 | command2

# Tee (write to file and stdout)
command | tee output.txt

# Suppress output
command > /dev/null 2>&1
```

---

### Error Handling

**Examples**:
```bash
#!/bin/bash

# Exit on error
set -e

# Exit on undefined variable
set -u

# Return error on pipe failure
set -o pipefail

# Combined
set -euo pipefail

# Custom error handling
command || {
    echo "Command failed"
    exit 1
}

# Check exit status
if command; then
    echo "Success"
else
    echo "Failed"
fi

# Trap signals
trap 'echo "Script interrupted"; exit 1' INT TERM

# Cleanup on exit
trap 'rm -f temp_file' EXIT

# Error function
error_exit() {
    echo "Error: $1" >&2
    exit 1
}

[ -f "file.txt" ] || error_exit "File not found"
```

---

### Complete Script Example

```bash
#!/bin/bash
# Backup script

set -euo pipefail

# Configuration
BACKUP_DIR="/backup"
SOURCE_DIR="/data"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"

# Functions
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

error_exit() {
    log "ERROR: $1"
    exit 1
}

# Check if source exists
[ -d "$SOURCE_DIR" ] || error_exit "Source directory not found"

# Create backup directory
mkdir -p "$BACKUP_DIR" || error_exit "Cannot create backup directory"

# Perform backup
log "Starting backup..."
tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "$SOURCE_DIR" || error_exit "Backup failed"

# Verify backup
if [ -f "${BACKUP_DIR}/${BACKUP_FILE}" ]; then
    SIZE=$(du -h "${BACKUP_DIR}/${BACKUP_FILE}" | cut -f1)
    log "Backup completed successfully: ${BACKUP_FILE} (${SIZE})"
else
    error_exit "Backup file not created"
fi

# Clean old backups (keep last 7)
log "Cleaning old backups..."
cd "$BACKUP_DIR"
ls -t backup_*.tar.gz | tail -n +8 | xargs -r rm
log "Cleanup completed"

log "Backup process finished"
exit 0
```

---

## 21. Advanced Text Processing (Continued)

### Regular Expressions

**Basic Regex**:
```
.          # Any single character
^          # Start of line
$          # End of line
*          # Zero or more
+          # One or more (extended)
?          # Zero or one (extended)
[abc]      # Character class
[^abc]     # Negated class
[a-z]      # Range
\          # Escape
|          # Or (extended)
()         # Grouping (extended)
{n}        # Exactly n times
{n,}       # n or more times
{n,m}      # Between n and m times
```

**Examples**:
```bash
# Match email addresses
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt

# Match IP addresses
grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" file.txt

# Match phone numbers
grep -E "\([0-9]{3}\) [0-9]{3}-[0-9]{4}" file.txt

# Match URLs
grep -E "https?://[a-zA-Z0-9./?=_-]+" file.txt

# Match dates (YYYY-MM-DD)
grep -E "[0-9]{4}-[0-9]{2}-[0-9]{2}" file.txt
```

---

## 22. System Administration

### `systemctl` - Systemd Control
**Description**: Controls systemd services.

**Examples**:
```bash
# Start service
sudo systemctl start nginx

# Stop service
sudo systemctl stop nginx

# Restart service
sudo systemctl restart nginx

# Reload configuration
sudo systemctl reload nginx

# Enable at boot
sudo systemctl enable nginx

# Disable at boot
sudo systemctl disable nginx

# Check status
systemctl status nginx

# Check if active
systemctl is-active nginx

# Check if enabled
systemctl is-enabled nginx

# List all services
systemctl list-units --type=service

# List failed services
systemctl --failed

# Show service file
systemctl cat nginx

# Edit service file
sudo systemctl edit nginx

# Reload systemd daemon
sudo systemctl daemon-reload

# Show service dependencies
systemctl list-dependencies nginx

# Power management
sudo systemctl reboot
sudo systemctl poweroff
sudo systemctl suspend
sudo systemctl hibernate
```

---

### `journalctl` - Systemd Logs
**Description**: Views systemd journal logs.

**Examples**:
```bash
# All logs
sudo journalctl

# Follow logs (like tail -f)
sudo journalctl -f

# Specific service
sudo journalctl -u nginx

# Kernel messages
sudo journalctl -k

# Boot messages
sudo journalctl -b

# Previous boot
sudo journalctl -b -1

# Since time
sudo journalctl --since "2025-12-01"
sudo journalctl --since "1 hour ago"
sudo journalctl --since today

# Until time
sudo journalctl --until "2025-12-04 12:00"

# Time range
sudo journalctl --since "2025-12-01" --until "2025-12-04"

# Priority level
sudo journalctl -p err
sudo journalctl -p warning

# Reverse order (newest first)
sudo journalctl -r

# Show only last N lines
sudo journalctl -n 50

# Output format
sudo journalctl -o json
sudo journalctl -o json-pretty
sudo journalctl -o cat

# Disk usage
sudo journalctl --disk-usage

# Clean old logs
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=1G
```

---

### `cron` - Schedule Tasks
**Description**: Schedules recurring tasks.

**Crontab Format**:
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─ Day of week (0-7, Sunday=0 or 7)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

**Examples**:
```bash
# Edit crontab
crontab -e

# List crontab
crontab -l

# Remove crontab
crontab -r

# Examples:
# Every minute
* * * * * /path/to/script.sh

# Every hour
0 * * * * /path/to/script.sh

# Every day at midnight
0 0 * * * /path/to/script.sh

# Every Sunday at 2 AM
0 2 * * 0 /path/to/script.sh

# Every 15 minutes
*/15 * * * * /path/to/script.sh

# First day of month
0 0 1 * * /path/to/script.sh

# Monday to Friday at 6 PM
0 18 * * 1-5 /path/to/script.sh

# Multiple times
0 9,12,18 * * * /path/to/script.sh

# System-wide cron
# /etc/crontab
# /etc/cron.d/
# /etc/cron.daily/
# /etc/cron.weekly/
# /etc/cron.monthly/
```

---

### `at` - Schedule One-time Task
**Description**: Schedules one-time task.

**Examples**:
```bash
# Schedule for specific time
at 10:00 PM
at> /path/to/script.sh
at> Ctrl+D

# Schedule relative time
at now + 1 hour
at now + 30 minutes
at now + 1 day
at midnight
at noon

# List scheduled jobs
atq

# Remove job
atrm JOB_NUMBER

# View job details
at -c JOB_NUMBER
```

---

## 23. Security and Firewall

### `ufw` - Uncomplicated Firewall (Ubuntu)
**Description**: Simple firewall frontend.

**Examples**:
```bash
# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Check status
sudo ufw status
sudo ufw status verbose

# Allow port
sudo ufw allow 22
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow service
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow from subnet
sudo ufw allow from 192.168.1.0/24

# Allow port from specific IP
sudo ufw allow from 192.168.1.100 to any port 22

# Deny port
sudo ufw deny 23

# Delete rule
sudo ufw delete allow 80

# Numbered rules
sudo ufw status numbered
sudo ufw delete 2

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Reset firewall
sudo ufw reset

# Application profiles
sudo ufw app list
sudo ufw allow 'Nginx Full'
```

---

### `iptables` - Firewall (Advanced)
**Description**: Advanced firewall configuration.

**Examples**:
```bash
# List rules
sudo iptables -L
sudo iptables -L -v -n

# Allow incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow incoming HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Drop all other incoming
sudo iptables -P INPUT DROP

# Allow from specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Delete rule
sudo iptables -D INPUT 3

# Flush all rules
sudo iptables -F

# Save rules
sudo iptables-save > /etc/iptables/rules.v4

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4
```

---

### `fail2ban` - Intrusion Prevention
**Description**: Bans IPs with suspicious activity.

**Installation**:
```bash
sudo apt install fail2ban
```

**Configuration**: `/etc/fail2ban/jail.local`

**Examples**:
```bash
# Status
sudo fail2ban-client status

# Status of specific jail
sudo fail2ban-client status sshd

# Ban IP
sudo fail2ban-client set sshd banip 192.168.1.100

# Unban IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Reload
sudo fail2ban-client reload

# Start/stop
sudo systemctl start fail2ban
sudo systemctl stop fail2ban
```

---

## 24. Kernel and System Tuning

### `sysctl` - Kernel Parameters
**Description**: Configures kernel parameters at runtime.

**Examples**:
```bash
# List all parameters
sysctl -a

# Show specific parameter
sysctl net.ipv4.ip_forward

# Set parameter
sudo sysctl -w net.ipv4.ip_forward=1

# Load from file
sudo sysctl -p
sudo sysctl -p /etc/sysctl.d/custom.conf

# Common parameters:
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Increase file descriptors
sudo sysctl -w fs.file-max=65536

# Increase network buffers
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216

# Make permanent: Add to /etc/sysctl.conf
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
```

---

## 25. Advanced Networking

### `iptables` NAT and Port Forwarding

**Examples**:
```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# NAT (masquerading)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forwarding
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80

# Port forwarding to different machine
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80
sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```

---

## Conclusion and Best Practices

### Command Line Best Practices

1. **Always backup before making changes**
2. **Use sudo carefully**
3. **Test commands with dry-run when available**
4. **Read man pages for unfamiliar commands**
5. **Use tab completion**
6. **Keep system updated**
7. **Use version control for scripts**
8. **Add comments to complex commands**
9. **Use meaningful variable names in scripts**
10. **Check exit codes in scripts**

### Security Best Practices

1. **Keep system and packages updated**
2. **Use SSH keys instead of passwords**
3. **Configure firewall properly**
4. **Disable unnecessary services**
5. **Use fail2ban for intrusion prevention**
6. **Regular security audits**
7. **Monitor system logs**
8. **Use strong passwords**
9. **Limit sudo access**
10. **Enable SELinux/AppArmor**

### Performance Best Practices

1. **Monitor system resources regularly**
2. **Clean up old logs and temporary files**
3. **Optimize disk I/O**
4. **Tune kernel parameters**
5. **Use appropriate filesystem**
6. **Monitor network bandwidth**
7. **Schedule resource-intensive tasks wisely**
8. **Use caching where appropriate**
9. **Optimize database queries**
10. **Profile applications for bottlenecks**

---

## Additional Resources

### Learning Resources
- Linux Documentation Project: https://www.tldp.org/
- GNU Coreutils Manual: https://www.gnu.org/software/coreutils/manual/
- Bash Reference Manual: https://www.gnu.org/software/bash/manual/
- Linux Journey: https://linuxjourney.com/
- Arch Linux Wiki: https://wiki.archlinux.org/

### Command References
- `man command` - Manual pages
- `info command` - Info pages
- `command --help` - Built-in help
- `tldr command` - Simplified examples
- https://explainshell.com/ - Command explanation

### Online Sandboxes
- JSLinux: https://bellard.org/jslinux/
- WebMinal: https://www.webminal.org/
- OverTheWire: https://overthewire.org/wargames/

---

## Appendix: Quick Reference

### Essential Commands Cheat Sheet
```bash
# Navigation
pwd, cd, ls, tree

# Files
cat, less, head, tail, touch, mkdir, cp, mv, rm

# Search
find, locate, grep, which

# Permissions
chmod, chown, chgrp

# Process
ps, top, htop, kill, pkill

# Network
ping, curl, wget, netstat, ss

# System
uname, hostname, uptime, free, df, du

# Users
whoami, who, w, id, sudo

# Package Management
apt, yum, dnf (depends on distribution)
```

---

**End of Linux Commands Complete Mastery Tutorial - Part 2**

For Part 1, see: `Linux_Commands_Complete_Mastery_Tutorial.md`
