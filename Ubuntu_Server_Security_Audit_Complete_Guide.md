# Ubuntu Server Security Audit & Hardening - Complete Master Guide

## Table of Contents
1. [Introduction to Server Security](#introduction-to-server-security)
2. [Pre-Audit Preparation](#pre-audit-preparation)
3. [Initial Security Assessment](#initial-security-assessment)
4. [User Account Security Audit](#user-account-security-audit)
5. [File System Security](#file-system-security)
6. [Network Security Audit](#network-security-audit)
7. [Service and Process Hardening](#service-and-process-hardening)
8. [Firewall Configuration](#firewall-configuration)
9. [SSH Hardening](#ssh-hardening)
10. [Logging and Monitoring](#logging-and-monitoring)
11. [Kernel Security](#kernel-security)
12. [Application Security](#application-security)
13. [Automated Security Tools](#automated-security-tools)
14. [Compliance and Best Practices](#compliance-and-best-practices)
15. [Incident Response Planning](#incident-response-planning)
16. [Continuous Security Monitoring](#continuous-security-monitoring)

---

## 1. Introduction to Server Security

### 1.1 Why Security Audits Matter

Security audits are systematic evaluations of your server's security posture. They help:
- **Identify vulnerabilities** before attackers do
- **Ensure compliance** with security standards (PCI-DSS, HIPAA, ISO 27001)
- **Prevent data breaches** and unauthorized access
- **Maintain system integrity** and availability
- **Build trust** with users and stakeholders

### 1.2 Security Audit Methodology

```
┌─────────────────────────────────────────┐
│    1. Planning & Scope Definition       │
├─────────────────────────────────────────┤
│    2. Information Gathering             │
├─────────────────────────────────────────┤
│    3. Vulnerability Assessment          │
├─────────────────────────────────────────┤
│    4. Risk Analysis & Prioritization    │
├─────────────────────────────────────────┤
│    5. Hardening & Remediation           │
├─────────────────────────────────────────┤
│    6. Verification & Testing            │
├─────────────────────────────────────────┤
│    7. Documentation & Reporting         │
├─────────────────────────────────────────┤
│    8. Continuous Monitoring             │
└─────────────────────────────────────────┘
```

### 1.3 Key Security Principles

- **Least Privilege**: Grant minimum permissions necessary
- **Defense in Depth**: Multiple layers of security controls
- **Fail Secure**: Systems should fail in a secure state
- **Separation of Duties**: No single person has complete control
- **Security by Design**: Build security from the ground up

---

## 2. Pre-Audit Preparation

### 2.1 Gather System Information

```bash
# Check Ubuntu version and kernel
lsb_release -a
uname -a

# Check system architecture
dpkg --print-architecture

# Get system uptime and load
uptime
cat /proc/loadavg

# Check hardware information
lscpu
free -h
df -h
```

**Output Analysis:**
- Note Ubuntu version for known vulnerabilities
- Verify kernel version is up-to-date
- Document system resources for capacity planning

### 2.2 Create Audit Checklist

Create a structured checklist to track your audit progress:

```bash
# Create audit workspace
mkdir -p ~/security-audit/{logs,reports,configs,scripts}
cd ~/security-audit

# Create audit log file with timestamp
AUDIT_DATE=$(date +%Y%m%d_%H%M%S)
AUDIT_LOG="logs/audit_${AUDIT_DATE}.log"
touch $AUDIT_LOG

# Function to log audit findings
audit_log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $AUDIT_LOG
}

audit_log "Security Audit Started"
```

### 2.3 Backup Critical Configurations

**Always backup before making changes:**

```bash
# Create backup directory
BACKUP_DIR=~/security-audit/backups/$(date +%Y%m%d)
mkdir -p $BACKUP_DIR

# Backup critical configuration files
sudo cp -p /etc/ssh/sshd_config $BACKUP_DIR/
sudo cp -p /etc/sudoers $BACKUP_DIR/
sudo cp -p /etc/pam.d/* $BACKUP_DIR/pam.d/
sudo cp -p /etc/login.defs $BACKUP_DIR/
sudo cp -p /etc/security/limits.conf $BACKUP_DIR/
sudo cp -pr /etc/ufw $BACKUP_DIR/
sudo cp -p /etc/sysctl.conf $BACKUP_DIR/

# Create system snapshot (if using LVM)
sudo lvdisplay
# sudo lvcreate -L 5G -s -n root_snapshot /dev/ubuntu-vg/root

audit_log "Configuration backups created in $BACKUP_DIR"
```

---

## 3. Initial Security Assessment

### 3.1 Check for Rootkits and Malware

```bash
# Install security scanning tools
sudo apt update
sudo apt install -y rkhunter chkrootkit clamav clamav-daemon

# Update virus definitions
sudo freshclam

# Run rootkit hunter
sudo rkhunter --update
sudo rkhunter --propupd  # First time setup
sudo rkhunter --check --skip-keypress

# Run chkrootkit
sudo chkrootkit

# Scan for malware with ClamAV
sudo clamscan -r -i /home
sudo clamscan -r -i /var/www  # If web server

audit_log "Rootkit and malware scan completed"
```

**Analysis:**
- Review warnings in `/var/log/rkhunter.log`
- Investigate any suspicious files or hidden processes
- False positives are common; verify before taking action

### 3.2 Check for Unauthorized Users and Groups

```bash
# List all users
cat /etc/passwd | cut -d: -f1

# List users with shell access
grep -E '/bin/(bash|sh|zsh|fish)$' /etc/passwd

# Find users with UID 0 (root privileges)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# List all groups
cat /etc/group

# Check for users with empty passwords (CRITICAL!)
sudo awk -F: '$2 == "" {print $1}' /etc/shadow

# Find accounts with no password aging
sudo awk -F: '$5 == "" {print $1}' /etc/shadow

audit_log "User account enumeration completed"
```

**Red Flags:**
- Multiple UID 0 accounts (only root should have UID 0)
- Empty passwords
- Unfamiliar user accounts
- Shell access for service accounts

### 3.3 Check Running Processes and Services

```bash
# List all running processes
ps aux

# Check for suspicious processes
ps aux | grep -E 'nc|ncat|netcat|/dev/tcp'

# List all enabled services
systemctl list-unit-files --type=service --state=enabled

# List all running services
systemctl list-units --type=service --state=running

# Check for services listening on network ports
sudo ss -tulpn
sudo netstat -tulpn  # Alternative

# Find processes listening on privileged ports (< 1024)
sudo ss -tulpn | awk '$5 ~ /:([0-9]{1,3})$/ {split($5,a,":"); if(a[2] < 1024) print}'

audit_log "Process and service enumeration completed"
```

**Look for:**
- Unnecessary services running
- Unknown processes with network connections
- Processes running as root that shouldn't be

---

## 4. User Account Security Audit

### 4.1 Password Policy Audit

```bash
# Check current password policies
sudo cat /etc/login.defs | grep -E '^PASS_'

# Check PAM password requirements
sudo cat /etc/pam.d/common-password

# Check password quality requirements
sudo cat /etc/security/pwquality.conf

audit_log "Password policy audit completed"
```

**Recommended Settings for `/etc/login.defs`:**

```bash
PASS_MAX_DAYS   90      # Maximum password age
PASS_MIN_DAYS   1       # Minimum days between password changes
PASS_WARN_AGE   7       # Days warning before password expires
PASS_MIN_LEN    14      # Minimum password length
```

### 4.2 Strengthen Password Policy

```bash
# Install password quality checking library
sudo apt install -y libpam-pwquality

# Configure password complexity
sudo tee /etc/security/pwquality.conf <<EOF
# Minimum password length
minlen = 14

# Require at least one digit
dcredit = -1

# Require at least one uppercase character
ucredit = -1

# Require at least one lowercase character
lcredit = -1

# Require at least one special character
ocredit = -1

# Number of character classes required
minclass = 4

# Maximum consecutive characters
maxrepeat = 3

# Check against dictionary
dictcheck = 1

# Reject passwords containing username
usercheck = 1

# Remember last N passwords (prevents reuse)
remember = 5
EOF

audit_log "Password policy hardened"
```

### 4.3 Configure Account Lockout Policy

```bash
# Edit PAM configuration for account lockout
sudo tee -a /etc/pam.d/common-auth <<EOF

# Account lockout after failed attempts
auth required pam_tally2.so deny=5 unlock_time=900 onerr=fail audit
EOF

# Add to common-account
sudo tee -a /etc/pam.d/common-account <<EOF
account required pam_tally2.so
EOF

# Check failed login attempts
sudo pam_tally2 --user=username

# Reset failed login counter
# sudo pam_tally2 --user=username --reset

audit_log "Account lockout policy configured"
```

**Policy Explanation:**
- `deny=5`: Lock account after 5 failed attempts
- `unlock_time=900`: Auto-unlock after 15 minutes (900 seconds)
- `onerr=fail`: Fail secure if error occurs

### 4.4 Disable Inactive Accounts

```bash
# Find accounts that haven't logged in for 90+ days
sudo lastlog -b 90

# Disable inactive accounts
# sudo usermod -L inactive_username
# sudo chage -E 0 inactive_username

# Remove unnecessary system accounts
# sudo userdel -r username

audit_log "Inactive account audit completed"
```

### 4.5 Implement Sudo Security

```bash
# Backup sudoers file
sudo cp /etc/sudoers /etc/sudoers.backup

# Edit sudoers safely
sudo visudo

# Add audit logging for sudo commands
echo "Defaults logfile=/var/log/sudo.log" | sudo tee -a /etc/sudoers.d/logging

# Require password for sudo (disable NOPASSWD)
# Remove or comment out any NOPASSWD entries

# Set sudo timeout
echo "Defaults timestamp_timeout=5" | sudo tee -a /etc/sudoers.d/timeout

# Log sudo commands to syslog
echo "Defaults syslog=authpriv" | sudo tee -a /etc/sudoers.d/syslog

# Review sudo access
sudo grep -E '^[^#].*ALL.*ALL' /etc/sudoers /etc/sudoers.d/*

audit_log "Sudo configuration hardened"
```

---

## 5. File System Security

### 5.1 Find Files with Dangerous Permissions

```bash
# Find world-writable files
sudo find / -xdev -type f -perm -0002 -ls 2>/dev/null

# Find world-writable directories
sudo find / -xdev -type d -perm -0002 -ls 2>/dev/null

# Find files with SUID bit set (run as owner)
sudo find / -xdev -type f -perm -4000 -ls 2>/dev/null

# Find files with SGID bit set (run as group)
sudo find / -xdev -type f -perm -2000 -ls 2>/dev/null

# Find files with no owner
sudo find / -xdev -nouser -ls 2>/dev/null

# Find files with no group
sudo find / -xdev -nogroup -ls 2>/dev/null

audit_log "Dangerous file permissions audit completed"
```

**Action Items:**
- Remove SUID/SGID from unnecessary files
- Fix ownership of orphaned files
- Remove world-writable permissions

### 5.2 Secure Critical System Files

```bash
# Set secure permissions on critical files
sudo chmod 600 /etc/shadow
sudo chmod 600 /etc/gshadow
sudo chmod 644 /etc/passwd
sudo chmod 644 /etc/group
sudo chmod 600 /etc/ssh/sshd_config
sudo chmod 600 /boot/grub/grub.cfg

# Protect important directories
sudo chmod 700 /root
sudo chmod 755 /etc
sudo chmod 755 /bin
sudo chmod 755 /sbin

# Make critical files immutable (prevents accidental changes)
# sudo chattr +i /etc/passwd
# sudo chattr +i /etc/shadow
# To remove: sudo chattr -i /etc/passwd

audit_log "Critical file permissions secured"
```

### 5.3 Configure Filesystem Mount Options

```bash
# Check current mount options
mount | column -t

# Edit /etc/fstab for secure mount options
sudo cp /etc/fstab /etc/fstab.backup

# Example secure mount options:
# /tmp with noexec,nosuid,nodev
# /var/tmp with noexec,nosuid,nodev
# /dev/shm with noexec,nosuid,nodev

sudo tee -a /etc/fstab <<EOF
# Secure mount options
tmpfs /tmp tmpfs defaults,noexec,nosuid,nodev,mode=1777 0 0
tmpfs /var/tmp tmpfs defaults,noexec,nosuid,nodev,mode=1777 0 0
tmpfs /dev/shm tmpfs defaults,noexec,nosuid,nodev 0 0
EOF

# Remount with new options
# sudo mount -o remount /tmp
# sudo mount -o remount /var/tmp
# sudo mount -o remount /dev/shm

audit_log "Filesystem mount options configured"
```

**Mount Option Explanations:**
- `noexec`: Prevents execution of binaries
- `nosuid`: Ignores SUID/SGID bits
- `nodev`: Prevents device file creation

### 5.4 Implement File Integrity Monitoring

```bash
# Install AIDE (Advanced Intrusion Detection Environment)
sudo apt install -y aide

# Initialize AIDE database (takes time)
sudo aideinit

# Move database to active location
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Run AIDE check
sudo aide --check

# Update AIDE database after legitimate changes
# sudo aide --update
# sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Schedule daily AIDE checks
echo "0 5 * * * root /usr/bin/aide --check | mail -s 'AIDE Report' admin@example.com" | sudo tee -a /etc/crontab

audit_log "File integrity monitoring configured"
```

---

## 6. Network Security Audit

### 6.1 Network Interface and IP Configuration Audit

```bash
# List network interfaces
ip addr show
ip link show

# Check routing table
ip route show

# Check DNS configuration
cat /etc/resolv.conf

# Check for promiscuous mode (potential sniffer)
ip link | grep PROMISC

# Disable IPv6 if not needed
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1

audit_log "Network configuration audit completed"
```

### 6.2 Open Ports and Services Audit

```bash
# Comprehensive port scan on localhost
sudo ss -tulpn

# Check listening TCP ports
sudo netstat -tlnp

# Check listening UDP ports
sudo netstat -ulnp

# Check established connections
sudo netstat -anp | grep ESTABLISHED

# Identify unusual connections
sudo lsof -i -n -P

# Check for services listening on all interfaces (0.0.0.0)
sudo ss -tulpn | grep "0.0.0.0"

audit_log "Open ports audit completed"
```

**Analysis:**
- Verify each open port has a legitimate purpose
- Close unnecessary ports
- Bind services to localhost if not needed externally

### 6.3 Network Security Parameters (sysctl)

```bash
# Current network parameters
sudo sysctl -a | grep net.

# Harden network stack
sudo tee /etc/sysctl.d/99-security.conf <<EOF
# Disable IP forwarding
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# Disable source routing
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0

# Disable ICMP redirect acceptance
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0

# Enable bad error message protection
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Enable reverse path filtering (prevent IP spoofing)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Log suspicious packets (martians)
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Ignore ICMP ping requests
net.ipv4.icmp_echo_ignore_all = 0  # Set to 1 to disable ping

# Ignore broadcast pings
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Enable TCP SYN cookies (protection against SYN floods)
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Disable IPv6 router advertisements
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.accept_ra = 0

# Increase TCP security
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1

# Protect against time-wait assassination
net.ipv4.tcp_rfc1337 = 1

# Kernel hardening
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
kernel.yama.ptrace_scope = 1
kernel.kexec_load_disabled = 1

# Core dump restrictions
fs.suid_dumpable = 0
kernel.core_uses_pid = 1
EOF

# Apply settings
sudo sysctl -p /etc/sysctl.d/99-security.conf

audit_log "Network security parameters hardened"
```

---

## 7. Service and Process Hardening

### 7.1 Disable Unnecessary Services

```bash
# List all enabled services
systemctl list-unit-files --type=service --state=enabled

# Common services to disable if not needed:
SERVICES_TO_REVIEW=(
    "bluetooth.service"
    "cups.service"           # Printing
    "avahi-daemon.service"   # Zeroconf networking
    "isc-dhcp-server.service"
    "nfs-server.service"
    "rpcbind.service"
    "snmpd.service"
)

# Review and disable unnecessary services
for service in "${SERVICES_TO_REVIEW[@]}"; do
    if systemctl is-enabled $service 2>/dev/null; then
        echo "Service $service is enabled. Disable? (y/n)"
        # sudo systemctl disable $service
        # sudo systemctl stop $service
    fi
done

audit_log "Unnecessary services disabled"
```

### 7.2 Harden Apache/Nginx Web Server

**Apache Hardening:**

```bash
# Install mod_security (Web Application Firewall)
sudo apt install -y libapache2-mod-security2

# Enable modules
sudo a2enmod security2
sudo a2enmod headers
sudo a2enmod ssl

# Configure security headers
sudo tee /etc/apache2/conf-available/security-headers.conf <<EOF
<IfModule mod_headers.c>
    # Prevent clickjacking
    Header always set X-Frame-Options "SAMEORIGIN"
    
    # XSS Protection
    Header always set X-XSS-Protection "1; mode=block"
    
    # Prevent MIME type sniffing
    Header always set X-Content-Type-Options "nosniff"
    
    # Referrer Policy
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    
    # Content Security Policy
    Header always set Content-Security-Policy "default-src 'self'"
    
    # Hide server information
    Header always unset X-Powered-By
    Header always unset Server
</IfModule>

ServerTokens Prod
ServerSignature Off
TraceEnable Off
EOF

sudo a2enconf security-headers
sudo systemctl reload apache2

audit_log "Apache web server hardened"
```

**Nginx Hardening:**

```bash
# Edit nginx configuration
sudo tee -a /etc/nginx/nginx.conf <<EOF
# Hide nginx version
server_tokens off;

# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self'" always;

# Limit request size
client_max_body_size 10M;

# Timeouts
client_body_timeout 10s;
client_header_timeout 10s;
keepalive_timeout 5s 5s;
send_timeout 10s;

# Buffer overflow protection
client_body_buffer_size 1K;
client_header_buffer_size 1k;
large_client_header_buffers 2 1k;
EOF

sudo nginx -t
sudo systemctl reload nginx

audit_log "Nginx web server hardened"
```

### 7.3 Harden Database Services

**MySQL/MariaDB Hardening:**

```bash
# Run MySQL secure installation
sudo mysql_secure_installation

# Additional hardening
sudo mysql -u root -p <<EOF
-- Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- Disable remote root login
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');

-- Remove test database
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';

-- Create limited privilege user instead of using root
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT SELECT, INSERT, UPDATE, DELETE ON appdb.* TO 'appuser'@'localhost';

-- Flush privileges
FLUSH PRIVILEGES;
EOF

# Edit MySQL configuration for security
sudo tee -a /etc/mysql/mysql.conf.d/mysqld.cnf <<EOF

[mysqld]
# Bind to localhost only
bind-address = 127.0.0.1

# Disable LOAD DATA LOCAL INFILE
local-infile=0

# Enable logging
log_error = /var/log/mysql/error.log
log_warnings = 2
log_slow_queries = /var/log/mysql/slow.log
long_query_time = 2

# Limit connections
max_connections = 100
max_connect_errors = 10

# Set timeouts
wait_timeout = 300
interactive_timeout = 300
EOF

sudo systemctl restart mysql
audit_log "MySQL/MariaDB hardened"
```

**PostgreSQL Hardening:**

```bash
# Edit PostgreSQL configuration
sudo nano /etc/postgresql/*/main/postgresql.conf

# Set these parameters:
# listen_addresses = 'localhost'
# ssl = on
# log_connections = on
# log_disconnections = on
# log_duration = on
# log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '

# Edit pg_hba.conf for access control
sudo nano /etc/postgresql/*/main/pg_hba.conf

# Use md5 or scram-sha-256 instead of trust
# local   all             postgres                                peer
# host    all             all             127.0.0.1/32            scram-sha-256

sudo systemctl restart postgresql
audit_log "PostgreSQL hardened"
```

---

## 8. Firewall Configuration

### 8.1 UFW (Uncomplicated Firewall) Setup

```bash
# Install UFW
sudo apt install -y ufw

# Set default policies (deny all incoming, allow all outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw default deny routed

# Allow SSH before enabling (CRITICAL - prevents lockout)
sudo ufw allow 22/tcp comment 'SSH'
# Or limit SSH to prevent brute force
sudo ufw limit 22/tcp comment 'SSH rate limited'

# Allow specific services
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'

# Allow from specific IP addresses only
# sudo ufw allow from 192.168.1.100 to any port 22

# Enable UFW
sudo ufw enable

# Check status
sudo ufw status verbose
sudo ufw status numbered

# View application profiles
sudo ufw app list
sudo ufw app info 'Apache Full'

audit_log "UFW firewall configured and enabled"
```

### 8.2 Advanced Firewall Rules

```bash
# Block specific IP address
sudo ufw deny from 192.168.1.50

# Allow specific subnet
sudo ufw allow from 192.168.1.0/24

# Allow port range
sudo ufw allow 6000:6007/tcp

# Allow specific interface
sudo ufw allow in on eth1 to any port 3306

# Delete rule by number
sudo ufw status numbered
sudo ufw delete 5

# Enable logging
sudo ufw logging on
sudo ufw logging medium  # or high

# Check logs
sudo tail -f /var/log/ufw.log

audit_log "Advanced firewall rules configured"
```

### 8.3 IPTables (Advanced Alternative)

```bash
# Install iptables-persistent to save rules
sudo apt install -y iptables-persistent

# Flush existing rules (CAREFUL!)
# sudo iptables -F
# sudo iptables -X

# Set default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

# Rate limit SSH to prevent brute force
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# Allow HTTP and HTTPS
sudo iptables -A INPUT -p tcp -m multiport --dports 80,443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

# Drop invalid packets
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Protection against port scanning
sudo iptables -N port-scanning
sudo iptables -A port-scanning -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s --limit-burst 2 -j RETURN
sudo iptables -A port-scanning -j DROP

# Save rules
sudo netfilter-persistent save

# View current rules
sudo iptables -L -n -v --line-numbers

audit_log "IPTables configured"
```

### 8.4 Fail2Ban Installation and Configuration

```bash
# Install Fail2Ban
sudo apt install -y fail2ban

# Create local configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit jail.local
sudo tee /etc/fail2ban/jail.local <<EOF
[DEFAULT]
# Ban IP for 1 hour
bantime = 3600

# 5 failures within 10 minutes triggers ban
findtime = 600
maxretry = 5

# Destination email for alerts
destemail = admin@example.com
sendername = Fail2Ban
action = %(action_mwl)s

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log
maxretry = 3
bantime = 7200

[apache-auth]
enabled = true
port = http,https
logpath = /var/log/apache2/error.log

[apache-badbots]
enabled = true
port = http,https
logpath = /var/log/apache2/access.log
bantime = 86400
maxretry = 2

[apache-noscript]
enabled = true
port = http,https
logpath = /var/log/apache2/error.log

[apache-overflows]
enabled = true
port = http,https
logpath = /var/log/apache2/error.log

[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log

[nginx-botsearch]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
EOF

# Start and enable Fail2Ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Unban an IP
# sudo fail2ban-client set sshd unbanip 192.168.1.100

audit_log "Fail2Ban configured and running"
```

---

## 9. SSH Hardening

### 9.1 SSH Configuration Best Practices

```bash
# Backup SSH config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Apply hardened SSH configuration
sudo tee /etc/ssh/sshd_config <<EOF
# Port and protocol
Port 22  # Consider changing to non-standard port
Protocol 2
AddressFamily inet  # Use inet6 for IPv6

# Listen on specific IP only
#ListenAddress 192.168.1.10

# Authentication
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM yes

# Allow specific users only
AllowUsers deployuser adminuser
# Or allow specific groups
# AllowGroups sshusers

# Key-based authentication only
AuthorizedKeysFile .ssh/authorized_keys

# Disable dangerous features
PermitUserEnvironment no
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
PermitTunnel no

# Login grace time
LoginGraceTime 30

# Maximum authentication attempts
MaxAuthTries 3
MaxSessions 2

# Connection settings
ClientAliveInterval 300
ClientAliveCountMax 2
TCPKeepAlive yes

# Strong ciphers only (disable weak algorithms)
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha256

# Host keys
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# Banner
Banner /etc/ssh/banner

# Subsystem (for SFTP)
Subsystem sftp /usr/lib/openssh/sftp-server -f AUTHPRIV -l INFO
EOF

# Test configuration
sudo sshd -t

# Restart SSH service
sudo systemctl restart sshd

audit_log "SSH hardening completed"
```

### 9.2 SSH Key-Based Authentication Setup

```bash
# Generate SSH key pair (on client machine)
# ssh-keygen -t ed25519 -C "your_email@example.com"
# ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Copy public key to server (from client)
# ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server

# Or manually:
# On server:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Add public key to authorized_keys
# echo "ssh-ed25519 AAAA... user@client" >> ~/.ssh/authorized_keys

# Disable password authentication after confirming key works
# sudo sed -i 's/^PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
# sudo systemctl restart sshd

audit_log "SSH key-based authentication configured"
```

### 9.3 SSH Login Banner

```bash
# Create warning banner
sudo tee /etc/ssh/banner <<EOF
***************************************************************************
                            AUTHORIZED ACCESS ONLY
***************************************************************************

This system is for authorized use only. All activity is monitored and logged.
Unauthorized access attempts will be prosecuted to the fullest extent of the law.

By continuing, you acknowledge that you have no expectation of privacy and
consent to monitoring of all activities on this system.

Disconnect IMMEDIATELY if you are not an authorized user.
***************************************************************************
EOF

audit_log "SSH banner created"
```

### 9.4 Two-Factor Authentication (2FA) for SSH

```bash
# Install Google Authenticator PAM module
sudo apt install -y libpam-google-authenticator

# Configure PAM for SSH
sudo tee -a /etc/pam.d/sshd <<EOF
# Google Authenticator
auth required pam_google_authenticator.so
EOF

# Edit SSH config to enable challenge-response
sudo sed -i 's/^ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/' /etc/ssh/sshd_config
sudo sed -i 's/^@include common-auth/#@include common-auth/' /etc/pam.d/sshd

# Add to sshd_config
echo "AuthenticationMethods publickey,keyboard-interactive" | sudo tee -a /etc/ssh/sshd_config

sudo systemctl restart sshd

# Each user runs this to set up 2FA:
# google-authenticator
# Answer questions and scan QR code with authenticator app

audit_log "SSH 2FA configured"
```

---

## 10. Logging and Monitoring

### 10.1 Configure Centralized Logging

```bash
# Configure rsyslog
sudo tee -a /etc/rsyslog.conf <<EOF

# Log all authentication messages
auth,authpriv.* /var/log/auth.log

# Log all kernel messages
kern.* /var/log/kern.log

# Log cron messages
cron.* /var/log/cron.log

# Remote logging (optional)
# *.* @@log-server.example.com:514
EOF

sudo systemctl restart rsyslog

audit_log "Centralized logging configured"
```

### 10.2 Log Rotation Configuration

```bash
# Configure logrotate for security logs
sudo tee /etc/logrotate.d/security <<EOF
/var/log/auth.log
/var/log/sudo.log
/var/log/ufw.log
{
    daily
    rotate 90
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
    postrotate
        /usr/lib/rsyslog/rsyslog-rotate
    endscript
}
EOF

audit_log "Log rotation configured"
```

### 10.3 Audit Daemon (auditd) Setup

```bash
# Install auditd
sudo apt install -y auditd audispd-plugins

# Enable and start auditd
sudo systemctl enable auditd
sudo systemctl start auditd

# Add audit rules
sudo tee -a /etc/audit/rules.d/audit.rules <<EOF
# Delete all existing rules
-D

# Buffer size
-b 8192

# Failure mode (0=silent, 1=printk, 2=panic)
-f 1

# Monitor unauthorized access attempts
-a always,exit -F arch=b64 -S open,openat -F exit=-EACCES -k access
-a always,exit -F arch=b64 -S open,openat -F exit=-EPERM -k access

# Monitor changes to system configuration
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/sudoers -p wa -k sudoers
-w /etc/sudoers.d/ -p wa -k sudoers

# Monitor SSH configuration
-w /etc/ssh/sshd_config -p wa -k sshd

# Monitor system calls
-a always,exit -F arch=b64 -S adjtimex,settimeofday -k time-change
-a always,exit -F arch=b64 -S mount,umount2 -k mounts
-a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat -k delete

# Monitor user/group modifications
-w /usr/sbin/useradd -p x -k user_modification
-w /usr/sbin/userdel -p x -k user_modification
-w /usr/sbin/usermod -p x -k user_modification
-w /usr/sbin/groupadd -p x -k group_modification
-w /usr/sbin/groupdel -p x -k group_modification
-w /usr/sbin/groupmod -p x -k group_modification

# Monitor sudo usage
-w /var/log/sudo.log -p wa -k sudo_log

# Monitor kernel module loading
-a always,exit -F arch=b64 -S init_module,delete_module -k modules

# Make configuration immutable
-e 2
EOF

# Load rules
sudo augenrules --load

# Check audit rules
sudo auditctl -l

# Search audit logs
# sudo ausearch -k access
# sudo ausearch -k sshd
# sudo aureport -au

audit_log "Audit daemon configured"
```

### 10.4 Real-time Log Monitoring

```bash
# Install logwatch for daily reports
sudo apt install -y logwatch

# Configure logwatch
sudo tee /etc/logwatch/conf/logwatch.conf <<EOF
Output = mail
Format = html
MailTo = admin@example.com
MailFrom = logwatch@$(hostname)
Detail = High
Service = All
Range = yesterday
EOF

# Test logwatch
sudo logwatch --detail High --mailto admin@example.com --range today

# Install OSSEC HIDS (Host Intrusion Detection)
# wget -q -O - https://updates.atomicorp.com/installers/atomic | sudo bash
# sudo apt install -y ossec-hids

audit_log "Log monitoring tools configured"
```

### 10.5 Monitor Critical System Changes

```bash
# Create monitoring script
sudo tee /usr/local/bin/monitor_system.sh <<'EOF'
#!/bin/bash

LOG_FILE="/var/log/system_monitor.log"
ALERT_EMAIL="admin@example.com"

log_alert() {
    echo "[$(date)] $1" >> $LOG_FILE
    echo "$1" | mail -s "Security Alert: $(hostname)" $ALERT_EMAIL
}

# Check for new SUID files
find / -xdev -type f -perm -4000 2>/dev/null > /tmp/suid_current
if [ -f /tmp/suid_baseline ]; then
    diff /tmp/suid_baseline /tmp/suid_current > /dev/null
    if [ $? -ne 0 ]; then
        log_alert "New SUID files detected"
    fi
fi
cp /tmp/suid_current /tmp/suid_baseline

# Check for failed login attempts
FAILED_LOGINS=$(grep "Failed password" /var/log/auth.log | grep $(date +%Y-%m-%d) | wc -l)
if [ $FAILED_LOGINS -gt 10 ]; then
    log_alert "High number of failed login attempts: $FAILED_LOGINS"
fi

# Check for new users
USER_COUNT=$(wc -l < /etc/passwd)
if [ -f /tmp/user_count ]; then
    OLD_COUNT=$(cat /tmp/user_count)
    if [ $USER_COUNT -ne $OLD_COUNT ]; then
        log_alert "User count changed from $OLD_COUNT to $USER_COUNT"
    fi
fi
echo $USER_COUNT > /tmp/user_count

# Check for listening ports changes
ss -tulpn > /tmp/ports_current
if [ -f /tmp/ports_baseline ]; then
    diff /tmp/ports_baseline /tmp/ports_current > /dev/null
    if [ $? -ne 0 ]; then
        log_alert "Listening ports have changed"
    fi
fi
cp /tmp/ports_current /tmp/ports_baseline

EOF

sudo chmod +x /usr/local/bin/monitor_system.sh

# Run hourly
echo "0 * * * * root /usr/local/bin/monitor_system.sh" | sudo tee -a /etc/crontab

audit_log "System monitoring script installed"
```

---

## 11. Kernel Security

### 11.1 AppArmor Configuration

```bash
# Check AppArmor status
sudo aa-status

# Install AppArmor utilities
sudo apt install -y apparmor-profiles apparmor-utils

# Enable AppArmor
sudo systemctl enable apparmor
sudo systemctl start apparmor

# List profiles
sudo aa-status

# Set profile to enforce mode
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx

# Set profile to complain mode (logging only)
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx

# Generate profile for application
# sudo aa-genprof /path/to/application

audit_log "AppArmor configured"
```

### 11.2 Kernel Module Management

```bash
# List loaded kernel modules
lsmod

# Disable unused kernel modules
sudo tee /etc/modprobe.d/blacklist-security.conf <<EOF
# Disable uncommon network protocols
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true

# Disable uncommon filesystems
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install udf /bin/true

# Disable USB storage if not needed
# install usb-storage /bin/true

# Disable firewire if not needed
install firewire-core /bin/true

# Disable Bluetooth if not needed
install bluetooth /bin/true
EOF

# Update initramfs
sudo update-initramfs -u

audit_log "Kernel modules hardened"
```

### 11.3 Kernel Parameters Hardening

```bash
# Additional kernel hardening (added to earlier sysctl config)
sudo tee -a /etc/sysctl.d/99-security.conf <<EOF

# Disable magic SysRq key
kernel.sysrq = 0

# Controls the System Request debugging functionality of the kernel
kernel.core_uses_pid = 1

# Restrict access to kernel logs
kernel.dmesg_restrict = 1

# Restrict kernel pointer visibility
kernel.kptr_restrict = 2

# Restrict perf subsystem usage
kernel.perf_event_paranoid = 3

# Disable kexec (prevents kernel crashdump exploitation)
kernel.kexec_load_disabled = 1

# Restrict BPF JIT compiler
net.core.bpf_jit_harden = 2

# Increase ASLR effectiveness
kernel.randomize_va_space = 2

# Restrict PTRACE usage
kernel.yama.ptrace_scope = 1

# Restrict access to /proc
# kernel.pid_max = 65536
EOF

sudo sysctl -p /etc/sysctl.d/99-security.conf

audit_log "Additional kernel hardening applied"
```

---

## 12. Application Security

### 12.1 Secure PHP Configuration

```bash
# Edit php.ini
PHP_INI=$(php -i | grep "Loaded Configuration File" | awk '{print $5}')

sudo tee -a $PHP_INI <<EOF

; Security settings
expose_php = Off
allow_url_fopen = Off
allow_url_include = Off
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log

; Disable dangerous functions
disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,phpinfo

; File upload security
file_uploads = On
upload_max_filesize = 2M
max_file_uploads = 3
upload_tmp_dir = /var/php_upload_tmp

; Session security
session.cookie_httponly = 1
session.cookie_secure = 1
session.use_only_cookies = 1
session.cookie_samesite = "Strict"
session.use_strict_mode = 1

; Memory limits
memory_limit = 128M
max_execution_time = 30
max_input_time = 60

; Open basedir restriction
open_basedir = /var/www:/tmp
EOF

sudo systemctl restart php*-fpm
audit_log "PHP hardened"
```

### 12.2 Secure Node.js Applications

```bash
# Install security packages
npm install helmet express-rate-limit

# Example secure Express.js configuration
cat > secure_express_example.js <<'EOF'
const express = require('express');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Use Helmet for security headers
app.use(helmet());

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
});
app.use(limiter);

// Disable X-Powered-By header
app.disable('x-powered-by');

// Trust proxy (if behind reverse proxy)
app.set('trust proxy', 1);

// Start server
app.listen(3000, '127.0.0.1', () => {
    console.log('Server running securely on localhost:3000');
});
EOF

audit_log "Node.js security example created"
```

### 12.3 Docker Container Security

```bash
# If using Docker
if command -v docker &> /dev/null; then
    # Run containers as non-root user
    # docker run --user 1000:1000 image_name
    
    # Use read-only filesystems
    # docker run --read-only image_name
    
    # Limit container resources
    # docker run --memory="512m" --cpus="1.0" image_name
    
    # Use security options
    # docker run --security-opt=no-new-privileges image_name
    
    # Scan images for vulnerabilities
    # docker scan image_name
    
    # Use Docker Bench Security
    docker run -it --net host --pid host --userns host --cap-add audit_control \
        -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
        -v /var/lib:/var/lib \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -v /usr/lib/systemd:/usr/lib/systemd \
        -v /etc:/etc --label docker_bench_security \
        docker/docker-bench-security
        
    audit_log "Docker security audit completed"
fi
```

---

## 13. Automated Security Tools

### 13.1 Lynis Security Audit

```bash
# Install Lynis
sudo apt install -y lynis

# Run comprehensive security audit
sudo lynis audit system

# View report
sudo cat /var/log/lynis.log
sudo cat /var/log/lynis-report.dat

# Schedule regular audits
echo "0 3 * * 0 root lynis audit system --quiet" | sudo tee -a /etc/crontab

audit_log "Lynis security audit completed"
```

### 13.2 OpenVAS Vulnerability Scanner

```bash
# Install OpenVAS (now part of Greenbone Vulnerability Management)
sudo apt install -y gvm

# Setup OpenVAS
sudo gvm-setup

# Start services
sudo gvm-start

# Access web interface at https://localhost:9392
# Default credentials are created during setup

audit_log "OpenVAS installed"
```

### 13.3 OSSEC HIDS Installation

```bash
# Install OSSEC Host Intrusion Detection System
wget -q -O - https://updates.atomicorp.com/installers/atomic | sudo bash
sudo apt install -y ossec-hids-server

# Start OSSEC
sudo /var/ossec/bin/ossec-control start

# Add agent (if monitoring multiple servers)
# sudo /var/ossec/bin/manage_agents

audit_log "OSSEC HIDS installed"
```

### 13.4 Tiger Security Audit Tool

```bash
# Install Tiger
sudo apt install -y tiger

# Run security audit
sudo tiger

# View report
sudo cat /var/log/tiger/security.report.*

audit_log "Tiger security audit completed"
```

### 13.5 Nmap Security Scanning

```bash
# Install Nmap
sudo apt install -y nmap

# Scan localhost for open ports
sudo nmap -sT -O localhost

# Comprehensive scan
sudo nmap -sS -sV -O -p- localhost

# Scan for vulnerabilities
sudo nmap --script vuln localhost

audit_log "Nmap security scan completed"
```

---

## 14. Compliance and Best Practices

### 14.1 CIS Benchmark Compliance

The Center for Internet Security (CIS) provides comprehensive security benchmarks:

```bash
# Download CIS Benchmark for Ubuntu
# Visit: https://www.cisecurity.org/cis-benchmarks/

# Key CIS recommendations implemented:
# [x] Filesystem integrity checking (AIDE)
# [x] Secure boot settings
# [x] Remove unnecessary services
# [x] Configure host firewall
# [x] Configure logging and auditing
# [x] Harden SSH
# [x] Configure PAM
# [x] User account security
# [x] System maintenance

audit_log "CIS Benchmark compliance reviewed"
```

### 14.2 PCI-DSS Compliance Checklist

For systems handling payment card data:

```bash
# PCI-DSS Requirements:
# 1. Install and maintain firewall - COMPLETED
# 2. Change vendor defaults - COMPLETED (SSH, passwords)
# 3. Protect stored cardholder data - APPLICATION SPECIFIC
# 4. Encrypt transmission of cardholder data - SSL/TLS
# 5. Use and update anti-virus - ClamAV installed
# 6. Develop secure systems - COMPLETED
# 7. Restrict access by business need - COMPLETED (least privilege)
# 8. Assign unique ID to each person - COMPLETED
# 9. Restrict physical access - PHYSICAL SECURITY
# 10. Track and monitor access - COMPLETED (auditd, logs)
# 11. Regularly test security - COMPLETED (tools installed)
# 12. Maintain security policy - DOCUMENTATION

audit_log "PCI-DSS compliance checklist reviewed"
```

### 14.3 GDPR Data Protection

```bash
# GDPR compliance considerations:
# - Encrypt sensitive data at rest and in transit
# - Implement access controls and authentication
# - Maintain audit logs
# - Regular security assessments
# - Data breach notification procedures
# - Privacy by design

# Enable full disk encryption (if not already done)
# Use LUKS for encryption

audit_log "GDPR compliance considerations documented"
```

### 14.4 Security Documentation

```bash
# Create security documentation directory
mkdir -p ~/security-docs

# Generate system security report
sudo tee ~/security-docs/security_report_$(date +%Y%m%d).md <<EOF
# Security Audit Report - $(date)

## System Information
- Hostname: $(hostname)
- OS: $(lsb_release -d | cut -f2)
- Kernel: $(uname -r)
- IP Address: $(hostname -I)

## Security Measures Implemented
1. Firewall configured (UFW/IPTables)
2. SSH hardened with key-based authentication
3. Fail2Ban installed and configured
4. User account policies enforced
5. File integrity monitoring (AIDE)
6. Audit daemon (auditd) configured
7. AppArmor enabled
8. Kernel hardening applied
9. Security updates automated
10. Logging and monitoring configured

## Open Ports
$(sudo ss -tulpn)

## Enabled Services
$(systemctl list-unit-files --type=service --state=enabled)

## User Accounts
$(cat /etc/passwd | cut -d: -f1)

## Recent Failed Logins
$(sudo grep "Failed password" /var/log/auth.log | tail -20)

## Recommendations
- Schedule regular security audits
- Review logs daily
- Update software weekly
- Test backups monthly
- Review user access quarterly
EOF

audit_log "Security documentation generated"
```

---

## 15. Incident Response Planning

### 15.1 Create Incident Response Plan

```bash
# Create incident response directory
mkdir -p ~/incident-response/{playbooks,evidence,reports}

# Create basic incident response playbook
cat > ~/incident-response/playbooks/incident_response.md <<'EOF'
# Incident Response Playbook

## 1. Preparation
- [ ] Maintain security audit logs
- [ ] Keep contact list updated
- [ ] Ensure backups are current
- [ ] Document baseline system state

## 2. Identification
- [ ] Monitor alerts from IDS/IPS
- [ ] Review log files for anomalies
- [ ] Investigate suspicious activity
- [ ] Determine incident severity

## 3. Containment
### Short-term
- [ ] Isolate affected systems
- [ ] Block malicious IPs
- [ ] Disable compromised accounts
- [ ] Preserve evidence

### Long-term
- [ ] Apply patches
- [ ] Change passwords
- [ ] Review access controls

## 4. Eradication
- [ ] Remove malware
- [ ] Close vulnerabilities
- [ ] Strengthen security controls

## 5. Recovery
- [ ] Restore from clean backups
- [ ] Verify system integrity
- [ ] Monitor for re-infection
- [ ] Gradual return to production

## 6. Lessons Learned
- [ ] Document incident timeline
- [ ] Identify root cause
- [ ] Update security measures
- [ ] Train staff on findings
EOF

audit_log "Incident response plan created"
```

### 15.2 Evidence Collection Script

```bash
# Create evidence collection script
sudo tee /usr/local/bin/collect_evidence.sh <<'EOF'
#!/bin/bash

EVIDENCE_DIR="/root/evidence/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$EVIDENCE_DIR"

echo "[*] Collecting incident evidence to $EVIDENCE_DIR"

# System information
echo "[*] Collecting system information..."
uname -a > "$EVIDENCE_DIR/system_info.txt"
hostname >> "$EVIDENCE_DIR/system_info.txt"
date >> "$EVIDENCE_DIR/system_info.txt"

# Network connections
echo "[*] Collecting network connections..."
ss -tupan > "$EVIDENCE_DIR/network_connections.txt"
ip addr > "$EVIDENCE_DIR/ip_addresses.txt"
ip route > "$EVIDENCE_DIR/routing_table.txt"

# Running processes
echo "[*] Collecting process list..."
ps auxf > "$EVIDENCE_DIR/processes.txt"

# User sessions
echo "[*] Collecting user sessions..."
who > "$EVIDENCE_DIR/logged_in_users.txt"
last > "$EVIDENCE_DIR/login_history.txt"
lastb > "$EVIDENCE_DIR/failed_logins.txt"

# System logs
echo "[*] Copying system logs..."
cp -r /var/log "$EVIDENCE_DIR/logs"

# Memory dump (requires significant space)
# dd if=/dev/mem of="$EVIDENCE_DIR/memory.dump" bs=1M

# Loaded kernel modules
echo "[*] Collecting loaded modules..."
lsmod > "$EVIDENCE_DIR/kernel_modules.txt"

# Open files
echo "[*] Collecting open files..."
lsof > "$EVIDENCE_DIR/open_files.txt"

# Scheduled tasks
echo "[*] Collecting scheduled tasks..."
crontab -l > "$EVIDENCE_DIR/crontab.txt" 2>&1
ls -la /etc/cron* > "$EVIDENCE_DIR/cron_jobs.txt"

# Create tarball
echo "[*] Creating evidence archive..."
tar -czf "/root/evidence_$(date +%Y%m%d_%H%M%S).tar.gz" -C /root/evidence .

echo "[*] Evidence collection complete: $EVIDENCE_DIR"
EOF

sudo chmod 700 /usr/local/bin/collect_evidence.sh

audit_log "Evidence collection script created"
```

### 15.3 Emergency Response Procedures

```bash
# Create emergency response quick reference
cat > ~/incident-response/emergency_procedures.txt <<'EOF'
EMERGENCY SECURITY RESPONSE - QUICK REFERENCE

=== SUSPECTED BREACH ===
1. DO NOT PANIC
2. Collect evidence: sudo /usr/local/bin/collect_evidence.sh
3. Disconnect from network (if severe): sudo ip link set eth0 down
4. Contact security team
5. Preserve logs and memory

=== BLOCK MALICIOUS IP ===
sudo ufw deny from <IP_ADDRESS>
sudo iptables -I INPUT -s <IP_ADDRESS> -j DROP

=== DISABLE COMPROMISED USER ===
sudo usermod -L <username>
sudo pkill -KILL -u <username>

=== ISOLATE SYSTEM ===
sudo ufw default deny incoming
sudo ufw default deny outgoing
# Or disconnect network:
sudo ip link set eth0 down

=== CHECK FOR ROOTKITS ===
sudo rkhunter --check --skip-keypress
sudo chkrootkit

=== SYSTEM RECOVERY ===
1. Boot from live USB
2. Mount encrypted filesystems
3. Run integrity checks
4. Restore from clean backup
5. Apply all security patches
6. Change all passwords
7. Review all user accounts
8. Audit all logs

=== CONTACTS ===
Security Team: security@example.com
Emergency Phone: +1-XXX-XXX-XXXX
Incident Response: incident@example.com
EOF

audit_log "Emergency procedures documented"
```

---

## 16. Continuous Security Monitoring

### 16.1 Automated Security Checks

```bash
# Create daily security check script
sudo tee /usr/local/bin/daily_security_check.sh <<'EOF'
#!/bin/bash

REPORT="/var/log/daily_security_$(date +%Y%m%d).log"
ALERT_EMAIL="admin@example.com"

echo "Daily Security Check - $(date)" > $REPORT
echo "======================================" >> $REPORT

# Check for system updates
echo "\n[*] Checking for system updates..." >> $REPORT
apt list --upgradable 2>/dev/null >> $REPORT

# Check for failed login attempts
echo "\n[*] Failed login attempts (last 24h):" >> $REPORT
grep "Failed password" /var/log/auth.log | grep $(date +%Y-%m-%d) >> $REPORT

# Check listening ports
echo "\n[*] Listening ports:" >> $REPORT
ss -tulpn >> $REPORT

# Check user accounts
echo "\n[*] User accounts with shell access:" >> $REPORT
grep -E '/bin/(bash|sh|zsh)$' /etc/passwd >> $REPORT

# Check SUID files
echo "\n[*] SUID files check:" >> $REPORT
find / -xdev -type f -perm -4000 2>/dev/null | wc -l >> $REPORT

# Check disk usage
echo "\n[*] Disk usage:" >> $REPORT
df -h >> $REPORT

# Check for rootkits (quick check)
echo "\n[*] Quick rootkit check:" >> $REPORT
chkrootkit -q >> $REPORT 2>&1

# Email report
cat $REPORT | mail -s "Daily Security Report - $(hostname)" $ALERT_EMAIL

EOF

sudo chmod +x /usr/local/bin/daily_security_check.sh

# Schedule daily execution
echo "0 6 * * * root /usr/local/bin/daily_security_check.sh" | sudo tee -a /etc/crontab

audit_log "Daily security check script configured"
```

### 16.2 Real-time Monitoring with Monit

```bash
# Install Monit
sudo apt install -y monit

# Configure Monit
sudo tee /etc/monit/monitrc <<EOF
set daemon 60

set logfile /var/log/monit.log

set mailserver smtp.gmail.com port 587
    username "your_email@gmail.com" password "your_password"
    using tlsv13

set alert admin@example.com

set httpd port 2812 and
    use address localhost
    allow localhost
    allow admin:monit

# Monitor system resources
check system \$HOST
    if loadavg (1min) > 4 then alert
    if loadavg (5min) > 2 then alert
    if cpu usage > 95% for 10 cycles then alert
    if memory usage > 90% then alert
    if swap usage > 50% then alert

# Monitor SSH
check process sshd with pidfile /var/run/sshd.pid
    start program = "/bin/systemctl start ssh"
    stop program = "/bin/systemctl stop ssh"
    if failed port 22 protocol ssh then restart
    if 5 restarts within 5 cycles then timeout

# Monitor Apache (if installed)
check process apache with pidfile /var/run/apache2/apache2.pid
    start program = "/bin/systemctl start apache2"
    stop program = "/bin/systemctl stop apache2"
    if failed host localhost port 80 protocol http then restart
    if 5 restarts within 5 cycles then timeout

# Monitor filesystem
check filesystem rootfs with path /
    if space usage > 90% then alert
    if inode usage > 90% then alert

EOF

sudo chmod 600 /etc/monit/monitrc
sudo systemctl enable monit
sudo systemctl start monit

# Check Monit status
sudo monit status

audit_log "Monit monitoring configured"
```

### 16.3 Security Update Automation

```bash
# Install unattended-upgrades
sudo apt install -y unattended-upgrades apt-listchanges

# Configure automatic security updates
sudo tee /etc/apt/apt.conf.d/50unattended-upgrades <<EOF
Unattended-Upgrade::Allowed-Origins {
    "\${distro_id}:\${distro_codename}-security";
    "\${distro_id}ESMApps:\${distro_codename}-apps-security";
    "\${distro_id}ESM:\${distro_codename}-infra-security";
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";
Unattended-Upgrade::SyslogEnable "true";
Unattended-Upgrade::Mail "admin@example.com";
Unattended-Upgrade::MailReport "only-on-error";
EOF

# Enable automatic updates
sudo tee /etc/apt/apt.conf.d/20auto-upgrades <<EOF
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
EOF

# Test configuration
sudo unattended-upgrade --dry-run --debug

audit_log "Automatic security updates configured"
```

### 16.4 Security Metrics Dashboard

```bash
# Create security metrics collection script
sudo tee /usr/local/bin/security_metrics.sh <<'EOF'
#!/bin/bash

METRICS_FILE="/var/log/security_metrics.json"

cat > $METRICS_FILE <<METRICS
{
    "timestamp": "$(date -Iseconds)",
    "hostname": "$(hostname)",
    "metrics": {
        "failed_logins_24h": $(grep "Failed password" /var/log/auth.log | grep $(date +%Y-%m-%d) | wc -l),
        "active_users": $(who | wc -l),
        "listening_ports": $(ss -tulpn | grep LISTEN | wc -l),
        "firewall_status": "$(sudo ufw status | head -1 | awk '{print $2}')",
        "updates_available": $(apt list --upgradable 2>/dev/null | grep -c upgradable),
        "disk_usage_percent": $(df / | tail -1 | awk '{print $5}' | tr -d '%'),
        "cpu_load_1min": $(uptime | awk -F'load average:' '{print $2}' | awk -F',' '{print $1}' | tr -d ' '),
        "memory_usage_percent": $(free | grep Mem | awk '{printf("%.0f", $3/$2 * 100.0)}')
    },
    "security_checks": {
        "auditd_running": "$(systemctl is-active auditd)",
        "fail2ban_running": "$(systemctl is-active fail2ban)",
        "aide_configured": $([ -f /var/lib/aide/aide.db ] && echo "true" || echo "false"),
        "apparmor_enabled": "$(sudo aa-status --enabled && echo 'true' || echo 'false')"
    }
}
METRICS

EOF

sudo chmod +x /usr/local/bin/security_metrics.sh

# Run every hour
echo "0 * * * * root /usr/local/bin/security_metrics.sh" | sudo tee -a /etc/crontab

audit_log "Security metrics collection configured"
```

---

## Conclusion and Final Checklist

### Complete Security Audit Checklist

```bash
# Generate final security checklist
cat > ~/security-audit/final_checklist.md <<'EOF'
# Ubuntu Server Security Hardening - Final Checklist

## System Updates
- [ ] All security patches applied
- [ ] Automatic updates configured
- [ ] System reboot scheduled if required

## User Accounts
- [ ] Root login disabled
- [ ] Strong password policy enforced
- [ ] Inactive accounts disabled
- [ ] Account lockout policy configured
- [ ] Sudo access reviewed and limited

## Authentication
- [ ] SSH key-based authentication enabled
- [ ] Password authentication disabled for SSH
- [ ] SSH configured securely
- [ ] Two-factor authentication implemented (optional)

## Network Security
- [ ] Firewall (UFW/IPTables) configured
- [ ] Fail2Ban installed and configured
- [ ] Unnecessary services disabled
- [ ] Open ports minimized
- [ ] Network parameters hardened (sysctl)

## File System Security
- [ ] Critical file permissions secured
- [ ] SUID/SGID files reviewed
- [ ] World-writable files removed
- [ ] File integrity monitoring (AIDE) configured
- [ ] Mount options hardened

## Kernel and System
- [ ] Kernel parameters hardened
- [ ] AppArmor enabled
- [ ] Unnecessary kernel modules disabled
- [ ] Core dumps disabled

## Logging and Monitoring
- [ ] Audit daemon (auditd) configured
- [ ] Log rotation configured
- [ ] Real-time monitoring enabled
- [ ] Security alerts configured
- [ ] Daily security reports automated

## Application Security
- [ ] Web server hardened
- [ ] Database server secured
- [ ] Application-specific security applied
- [ ] Security headers configured

## Backup and Recovery
- [ ] Regular backups scheduled
- [ ] Backup integrity tested
- [ ] Incident response plan documented
- [ ] Recovery procedures tested

## Compliance
- [ ] Security documentation completed
- [ ] Compliance requirements reviewed
- [ ] Security audit scheduled
- [ ] Staff training planned

## Continuous Monitoring
- [ ] Automated security scans scheduled
- [ ] Log monitoring active
- [ ] Intrusion detection running
- [ ] Vulnerability scanning configured

EOF

audit_log "Final security checklist generated"
```

### Post-Hardening Verification

```bash
# Run comprehensive security verification
sudo tee /usr/local/bin/verify_hardening.sh <<'EOF'
#!/bin/bash

echo "======================================"
echo "Security Hardening Verification"
echo "======================================"
echo ""

# Check firewall
echo "[*] Firewall Status:"
sudo ufw status
echo ""

# Check SSH configuration
echo "[*] SSH Root Login:"
sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
echo "[*] SSH Password Authentication:"
sudo grep "^PasswordAuthentication" /etc/ssh/sshd_config
echo ""

# Check fail2ban
echo "[*] Fail2Ban Status:"
sudo systemctl is-active fail2ban
echo ""

# Check auditd
echo "[*] Auditd Status:"
sudo systemctl is-active auditd
echo ""

# Check AppArmor
echo "[*] AppArmor Status:"
sudo aa-status | head -5
echo ""

# Check automatic updates
echo "[*] Automatic Updates:"
sudo systemctl is-active unattended-upgrades
echo ""

# Check listening services
echo "[*] Listening Services:"
sudo ss -tulpn | grep LISTEN
echo ""

# Check password policy
echo "[*] Password Policy:"
sudo grep "^PASS" /etc/login.defs
echo ""

echo "======================================"
echo "Verification Complete"
echo "======================================"
EOF

sudo chmod +x /usr/local/bin/verify_hardening.sh
sudo /usr/local/bin/verify_hardening.sh

audit_log "Hardening verification completed"
```

### Maintenance Schedule

```markdown
## Recommended Security Maintenance Schedule

### Daily
- Review security alerts and logs
- Monitor system resources
- Check for suspicious activity

### Weekly  
- Review failed login attempts
- Check for available updates
- Verify backup completion
- Review firewall logs

### Monthly
- Run full security audit (Lynis)
- Review user accounts and permissions
- Test backup restoration
- Update security documentation
- Review and update firewall rules

### Quarterly
- Conduct vulnerability assessment
- Review and update security policies
- Security awareness training
- Test incident response procedures
- Review compliance requirements

### Annually
- Comprehensive security audit
- Penetration testing
- Review and update security architecture
- Disaster recovery testing
- Security policy review and update
```

---

## Additional Resources

### Security References

1. **CIS Benchmarks**: https://www.cisecurity.org/cis-benchmarks/
2. **Ubuntu Security Guide**: https://ubuntu.com/security
3. **NIST Cybersecurity Framework**: https://www.nist.gov/cyberframework
4. **OWASP**: https://owasp.org/
5. **SANS Security Resources**: https://www.sans.org/security-resources/

### Useful Commands Reference

```bash
# Security Status Quick Check
sudo ufw status                  # Firewall status
sudo systemctl status fail2ban   # Fail2ban status
sudo systemctl status auditd     # Audit daemon status
sudo aa-status                   # AppArmor status
sudo fail2ban-client status      # Banned IPs
sudo ausearch -ts today          # Today's audit events
sudo lynis audit system          # Full security audit

# Log Analysis
sudo tail -f /var/log/auth.log          # Real-time auth logs
sudo tail -f /var/log/syslog             # System logs
sudo grep "Failed password" /var/log/auth.log  # Failed logins
sudo journalctl -u ssh -f                # SSH service logs

# Network Security
sudo ss -tulpn                   # Listening ports
sudo netstat -antp               # Active connections
sudo lsof -i                     # Open network files
sudo tcpdump -i eth0             # Packet capture

# User Management
sudo lastlog                     # Last login times
sudo last                        # Login history
sudo who                         # Current users
sudo w                           # Who is doing what

# File Security
sudo find / -perm -4000 2>/dev/null     # SUID files
sudo find / -perm -2000 2>/dev/null     # SGID files
sudo find / -perm -0002 2>/dev/null     # World-writable
```

---

## Summary

This comprehensive guide has covered:

✅ **Complete security audit methodology**
✅ **User account and authentication hardening**  
✅ **File system and kernel security**
✅ **Network security and firewall configuration**
✅ **Service hardening (SSH, web servers, databases)**
✅ **Logging, monitoring, and intrusion detection**
✅ **Automated security tools and scanning**
✅ **Compliance frameworks (CIS, PCI-DSS, GDPR)**
✅ **Incident response planning**
✅ **Continuous security monitoring**

### Key Takeaways

1. **Security is a continuous process**, not a one-time task
2. **Defense in depth** - implement multiple layers of security
3. **Least privilege principle** - grant minimum necessary permissions  
4. **Monitor everything** - comprehensive logging is crucial
5. **Stay updated** - apply security patches promptly
6. **Test regularly** - verify backups and incident response procedures
7. **Document thoroughly** - maintain security policies and procedures
8. **Automate where possible** - reduce human error

### Next Steps

1. Review this guide and adapt to your specific requirements
2. Implement hardening measures in a test environment first  
3. Create backups before making production changes
4. Document all security configurations
5. Schedule regular security audits
6. Train team members on security best practices
7. Stay informed about new security threats and mitigations

**Remember**: Security is an ongoing journey. Regular audits, updates, and vigilance are essential to maintaining a secure server environment.

---

*Last Updated: December 2025*  
*Version: 1.0*  
*Author: Security Operations Team*
```