# Nginx Complete Mastery Tutorial - Basic Level

## Table of Contents
1. [Introduction to Nginx](#introduction-to-nginx)
2. [Installation and Setup](#installation-and-setup)
3. [Basic Configuration](#basic-configuration)
4. [Serving Static Content](#serving-static-content)
5. [Directory Structure](#directory-structure)
6. [Basic Commands](#basic-commands)
7. [Understanding Configuration Files](#understanding-configuration-files)
8. [Virtual Hosts (Server Blocks)](#virtual-hosts-server-blocks)
9. [Access and Error Logs](#access-and-error-logs)
10. [Basic Security](#basic-security)

---

## Introduction to Nginx

### What is Nginx?
Nginx (pronounced "engine-x") is a high-performance web server, reverse proxy server, and load balancer. Created by Igor Sysoev in 2004, it's known for its stability, rich feature set, simple configuration, and low resource consumption.

### Key Features
- **High Performance**: Handles thousands of concurrent connections efficiently
- **Low Resource Usage**: Uses event-driven, asynchronous architecture
- **Reverse Proxy**: Can proxy requests to backend servers
- **Load Balancing**: Distributes traffic across multiple servers
- **Static Content Serving**: Extremely efficient at serving static files
- **SSL/TLS Support**: Native HTTPS support

### Nginx vs Apache
```
Nginx:
- Event-driven architecture
- Better performance for static content
- Lower memory footprint
- Asynchronous, non-blocking processing

Apache:
- Process-driven architecture
- More flexible with .htaccess
- Larger ecosystem of modules
- Synchronous, blocking processing
```

### Use Cases
1. Web server for static content
2. Reverse proxy for application servers
3. Load balancer for distributed systems
4. API gateway
5. Caching server
6. Media streaming server

---

## Installation and Setup

### Installing on Ubuntu/Debian
```bash
# Update package list
sudo apt update

# Install Nginx
sudo apt install nginx -y

# Check Nginx version
nginx -v

# Check installation status
systemctl status nginx
```

### Installing on CentOS/RHEL
```bash
# Install EPEL repository (if needed)
sudo yum install epel-release -y

# Install Nginx
sudo yum install nginx -y

# Start Nginx service
sudo systemctl start nginx

# Enable Nginx to start on boot
sudo systemctl enable nginx
```

### Installing on macOS
```bash
# Using Homebrew
brew install nginx

# Start Nginx
brew services start nginx
```

### Installing from Source
```bash
# Install dependencies
sudo apt install build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev -y

# Download Nginx source
wget http://nginx.org/download/nginx-1.24.0.tar.gz
tar -xzf nginx-1.24.0.tar.gz
cd nginx-1.24.0

# Configure compilation
./configure --prefix=/etc/nginx \
            --sbin-path=/usr/sbin/nginx \
            --conf-path=/etc/nginx/nginx.conf \
            --error-log-path=/var/log/nginx/error.log \
            --http-log-path=/var/log/nginx/access.log \
            --pid-path=/var/run/nginx.pid \
            --lock-path=/var/run/nginx.lock \
            --with-http_ssl_module \
            --with-http_v2_module

# Compile and install
make
sudo make install
```

### Post-Installation Verification
```bash
# Check if Nginx is running
sudo systemctl status nginx

# Test Nginx configuration
sudo nginx -t

# Access default page
curl http://localhost

# Check which port Nginx is listening on
sudo netstat -tlnp | grep nginx
# Or using ss
sudo ss -tlnp | grep nginx
```

---

## Basic Configuration

### Main Configuration File Structure
The main configuration file is typically located at `/etc/nginx/nginx.conf`.

```nginx
# Global context - applies to entire Nginx instance
user www-data;
worker_processes auto;
pid /run/nginx.pid;

# Events context - connection processing
events {
    worker_connections 768;
}

# HTTP context - web server configuration
http {
    # Basic settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # MIME types
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Gzip compression
    gzip on;

    # Include virtual host configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### Configuration Contexts
```nginx
# Main/Global Context - affects entire server
user nginx;
worker_processes 4;

# Events Context - connection processing configuration
events {
    worker_connections 1024;
    use epoll;  # efficient connection processing method
}

# HTTP Context - HTTP server settings
http {
    # settings apply to all virtual hosts
    
    # Server Context - individual virtual host
    server {
        listen 80;
        server_name example.com;
        
        # Location Context - URL-specific settings
        location / {
            root /var/www/html;
            index index.html;
        }
    }
}
```

### Basic Directives Explained

#### worker_processes
```nginx
# Auto-detect number of CPU cores
worker_processes auto;

# Or specify exact number
worker_processes 4;
```
Determines how many worker processes Nginx spawns. Use `auto` to match CPU cores.

#### worker_connections
```nginx
events {
    worker_connections 1024;
}
```
Maximum number of simultaneous connections per worker process.

#### keepalive_timeout
```nginx
keepalive_timeout 65;
```
Time in seconds to keep connection alive for subsequent requests.

---

## Serving Static Content

### Basic Static File Server
```nginx
server {
    listen 80;
    server_name example.com;
    
    # Document root
    root /var/www/html;
    
    # Default index files
    index index.html index.htm;
    
    # Serve static files
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Serving Different File Types
```nginx
server {
    listen 80;
    server_name static.example.com;
    root /var/www/static;
    
    # HTML files
    location ~* \.html$ {
        expires 1h;
        add_header Cache-Control "public, immutable";
    }
    
    # CSS and JavaScript
    location ~* \.(css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Images
    location ~* \.(jpg|jpeg|png|gif|ico|svg)$ {
        expires 1M;
        add_header Cache-Control "public, immutable";
    }
    
    # Fonts
    location ~* \.(woff|woff2|ttf|otf|eot)$ {
        expires 1M;
        add_header Cache-Control "public, immutable";
        add_header Access-Control-Allow-Origin *;
    }
}
```

### Directory Listing
```nginx
location /downloads/ {
    root /var/www;
    autoindex on;              # Enable directory listing
    autoindex_exact_size off;  # Show file sizes in human-readable format
    autoindex_localtime on;    # Show local time instead of GMT
}
```

### Serving Files from Multiple Locations
```nginx
server {
    listen 80;
    server_name files.example.com;
    
    # Main document root
    location / {
        root /var/www/main;
        index index.html;
    }
    
    # Images from different location
    location /images/ {
        root /var/www/assets;
    }
    
    # Downloads from another location
    location /downloads/ {
        alias /mnt/storage/files/;
        autoindex on;
    }
}
```

### Root vs Alias
```nginx
# root - appends location to root path
location /images/ {
    root /var/www;
    # File path: /var/www/images/photo.jpg
}

# alias - replaces location with alias path
location /images/ {
    alias /var/www/pictures/;
    # File path: /var/www/pictures/photo.jpg
}
```

---

## Directory Structure

### Default Installation Paths (Ubuntu/Debian)
```
/etc/nginx/
├── nginx.conf              # Main configuration file
├── conf.d/                 # Additional configuration files
├── sites-available/        # Available virtual host configs
├── sites-enabled/          # Enabled virtual host configs (symlinks)
├── modules-available/      # Available modules
├── modules-enabled/        # Enabled modules
├── snippets/              # Reusable configuration snippets
└── mime.types             # MIME type mappings

/var/log/nginx/
├── access.log             # Access logs
└── error.log              # Error logs

/var/www/
└── html/                  # Default web root
    └── index.nginx-debian.html

/usr/share/nginx/
└── html/                  # Alternative default web root
```

### CentOS/RHEL Directory Structure
```
/etc/nginx/
├── nginx.conf
├── conf.d/
├── default.d/
└── mime.types

/usr/share/nginx/
└── html/

/var/log/nginx/
```

### Important Files

#### /etc/nginx/nginx.conf
Main configuration file containing global settings and HTTP block.

#### /etc/nginx/sites-available/
Contains individual virtual host configuration files.

#### /etc/nginx/sites-enabled/
Contains symbolic links to enabled sites from sites-available.

#### /etc/nginx/conf.d/
Additional configuration files (typically for specific functionalities).

#### /var/log/nginx/access.log
Records all incoming requests to the server.

#### /var/log/nginx/error.log
Records server errors and issues.

---

## Basic Commands

### Service Management

#### Start Nginx
```bash
# Using systemctl (systemd)
sudo systemctl start nginx

# Using service command
sudo service nginx start

# Direct binary execution
sudo nginx
```

#### Stop Nginx
```bash
# Graceful stop (wait for workers to finish)
sudo systemctl stop nginx
sudo nginx -s quit

# Fast stop (immediate)
sudo nginx -s stop
```

#### Restart Nginx
```bash
# Full restart
sudo systemctl restart nginx

# Reload configuration (graceful, no downtime)
sudo systemctl reload nginx
sudo nginx -s reload
```

#### Check Status
```bash
# Service status
sudo systemctl status nginx

# Check if Nginx is running
ps aux | grep nginx

# Check process tree
pstree -p | grep nginx
```

### Configuration Management

#### Test Configuration
```bash
# Test configuration syntax
sudo nginx -t

# Test and show configuration details
sudo nginx -T
```

#### View Configuration
```bash
# Display current configuration
sudo nginx -T

# Show compiled modules
nginx -V
```

#### Reload Configuration
```bash
# Reload without downtime
sudo nginx -s reload

# Using systemctl
sudo systemctl reload nginx
```

### Log Management

#### View Access Logs
```bash
# View recent access logs
sudo tail -f /var/log/nginx/access.log

# View last 100 lines
sudo tail -n 100 /var/log/nginx/access.log

# Search for specific IP
sudo grep "192.168.1.100" /var/log/nginx/access.log
```

#### View Error Logs
```bash
# Follow error log in real-time
sudo tail -f /var/log/nginx/error.log

# View errors from last hour
sudo journalctl -u nginx --since "1 hour ago"
```

#### Rotate Logs
```bash
# Send signal to reopen log files
sudo nginx -s reopen

# Using logrotate (automated)
sudo logrotate -f /etc/logrotate.d/nginx
```

### Enable/Disable Sites

#### Enable a Site
```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

#### Disable a Site
```bash
# Remove symbolic link
sudo rm /etc/nginx/sites-enabled/example.com

# Reload Nginx
sudo systemctl reload nginx
```

---

## Understanding Configuration Files

### Configuration File Syntax

#### Basic Structure
```nginx
# Comments start with #

# Simple directive (name + parameters + semicolon)
worker_processes 4;

# Block directive (contains other directives)
events {
    worker_connections 1024;
}

# Nested blocks
http {
    server {
        location / {
            # directives here
        }
    }
}
```

#### Variables
```nginx
# Built-in variables
server {
    # $uri - current request URI
    # $args - query string parameters
    # $host - hostname from request
    # $remote_addr - client IP address
    # $request_method - HTTP method (GET, POST, etc.)
    
    location / {
        return 200 "URI: $uri\nHost: $host\nIP: $remote_addr";
    }
}
```

### Common Directives

#### listen
```nginx
# Listen on port 80
listen 80;

# Listen on specific IP and port
listen 192.168.1.100:80;

# Listen on IPv6
listen [::]:80;

# Enable SSL
listen 443 ssl;

# Set as default server
listen 80 default_server;
```

#### server_name
```nginx
# Single domain
server_name example.com;

# Multiple domains
server_name example.com www.example.com;

# Wildcard subdomain
server_name *.example.com;

# Regular expression
server_name ~^www\d+\.example\.com$;

# Default catch-all
server_name _;
```

#### root and index
```nginx
server {
    # Document root directory
    root /var/www/html;
    
    # Default index files (checked in order)
    index index.html index.htm index.php;
}
```

#### location
```nginx
# Exact match
location = /exact {
    # matches only /exact
}

# Prefix match
location /prefix {
    # matches /prefix, /prefix/path, etc.
}

# Regular expression (case-sensitive)
location ~ \.php$ {
    # matches .php files
}

# Regular expression (case-insensitive)
location ~* \.(jpg|jpeg|png)$ {
    # matches image files
}

# Priority prefix match
location ^~ /priority {
    # higher priority than regex
}
```

### Include Files
```nginx
# Include external configuration
http {
    # Include MIME types
    include /etc/nginx/mime.types;
    
    # Include all .conf files
    include /etc/nginx/conf.d/*.conf;
    
    # Include site configurations
    include /etc/nginx/sites-enabled/*;
}
```

### Creating Reusable Snippets
```nginx
# File: /etc/nginx/snippets/ssl-params.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;

# Usage in server block:
server {
    listen 443 ssl;
    include snippets/ssl-params.conf;
}
```

---

## Virtual Hosts (Server Blocks)

### Basic Virtual Host
```nginx
# /etc/nginx/sites-available/example.com
server {
    listen 80;
    server_name example.com www.example.com;
    
    root /var/www/example.com;
    index index.html index.htm;
    
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Multiple Virtual Hosts
```nginx
# Site 1: example.com
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com;
    index index.html;
}

# Site 2: test.com
server {
    listen 80;
    server_name test.com www.test.com;
    root /var/www/test.com;
    index index.html;
}

# Site 3: blog.example.com (subdomain)
server {
    listen 80;
    server_name blog.example.com;
    root /var/www/blog;
    index index.html;
}
```

### Name-Based Virtual Hosts
```nginx
# HTTP block
http {
    # Virtual host 1
    server {
        listen 80;
        server_name site1.com;
        root /var/www/site1;
    }
    
    # Virtual host 2
    server {
        listen 80;
        server_name site2.com;
        root /var/www/site2;
    }
    
    # Default virtual host (catches unmatched domains)
    server {
        listen 80 default_server;
        server_name _;
        return 444;  # Close connection without response
    }
}
```

### IP-Based Virtual Hosts
```nginx
# Virtual host on first IP
server {
    listen 192.168.1.10:80;
    server_name site1.com;
    root /var/www/site1;
}

# Virtual host on second IP
server {
    listen 192.168.1.11:80;
    server_name site2.com;
    root /var/www/site2;
}
```

### Port-Based Virtual Hosts
```nginx
# Site on port 80
server {
    listen 80;
    server_name example.com;
    root /var/www/main;
}

# Admin panel on port 8080
server {
    listen 8080;
    server_name admin.example.com;
    root /var/www/admin;
}

# API on port 8000
server {
    listen 8000;
    server_name api.example.com;
    root /var/www/api;
}
```

### Setting Up a Virtual Host Step-by-Step

#### Step 1: Create Directory Structure
```bash
# Create web root directory
sudo mkdir -p /var/www/example.com/public_html

# Create log directory
sudo mkdir -p /var/log/nginx/example.com

# Set proper ownership
sudo chown -R $USER:$USER /var/www/example.com
```

#### Step 2: Create Sample Content
```bash
# Create index.html
cat > /var/www/example.com/public_html/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to example.com</title>
</head>
<body>
    <h1>Success! example.com virtual host is working!</h1>
</body>
</html>
EOF
```

#### Step 3: Create Server Block Configuration
```bash
# Create configuration file
sudo nano /etc/nginx/sites-available/example.com
```

```nginx
server {
    listen 80;
    listen [::]:80;
    
    server_name example.com www.example.com;
    
    root /var/www/example.com/public_html;
    index index.html index.htm;
    
    access_log /var/log/nginx/example.com/access.log;
    error_log /var/log/nginx/example.com/error.log;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### Step 4: Enable the Virtual Host
```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

#### Step 5: Update Hosts File (for testing)
```bash
# Add to /etc/hosts for local testing
sudo nano /etc/hosts

# Add line:
127.0.0.1 example.com www.example.com
```

#### Step 6: Test the Virtual Host
```bash
# Using curl
curl http://example.com

# Using browser
# Navigate to http://example.com
```

---

## Access and Error Logs

### Log File Locations
```nginx
# Default locations
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;

# Custom locations
access_log /var/log/nginx/mysite.access.log;
error_log /var/log/nginx/mysite.error.log;
```

### Access Log Format

#### Default Combined Format
```
192.168.1.100 - - [11/Dec/2025:10:15:30 +0000] "GET /index.html HTTP/1.1" 200 1024 "https://google.com" "Mozilla/5.0"

Fields:
- 192.168.1.100: Client IP address
- [11/Dec/2025:10:15:30 +0000]: Timestamp
- GET /index.html HTTP/1.1: Request method, URI, protocol
- 200: HTTP status code
- 1024: Response size in bytes
- "https://google.com": Referer
- "Mozilla/5.0": User agent
```

#### Custom Log Format
```nginx
# Define custom format
log_format custom '$remote_addr - $remote_user [$time_local] '
                  '"$request" $status $body_bytes_sent '
                  '"$http_referer" "$http_user_agent" '
                  'RT=$request_time';

# Use custom format
access_log /var/log/nginx/access.log custom;
```

#### JSON Log Format
```nginx
log_format json_combined escape=json
    '{'
        '"time_local":"$time_local",'
        '"remote_addr":"$remote_addr",'
        '"request":"$request",'
        '"status": "$status",'
        '"body_bytes_sent":"$body_bytes_sent",'
        '"request_time":"$request_time",'
        '"http_referrer":"$http_referer",'
        '"http_user_agent":"$http_user_agent"'
    '}';

access_log /var/log/nginx/access.json json_combined;
```

### Error Log Levels
```nginx
# Available levels (from least to most verbose):
# emerg, alert, crit, error, warn, notice, info, debug

# Set error level
error_log /var/log/nginx/error.log warn;

# Different levels for different purposes
error_log /var/log/nginx/error.log error;      # Production
error_log /var/log/nginx/debug.log debug;      # Development
```

### Conditional Logging
```nginx
# Map to create logging condition
map $status $loggable {
    ~^[23]  0;  # Don't log 2xx and 3xx
    default 1;  # Log everything else
}

# Use condition
access_log /var/log/nginx/access.log combined if=$loggable;
```

### Disable Logging
```nginx
# Disable access logging for specific location
location /health-check {
    access_log off;
    return 200 "OK";
}

# Disable error logging
error_log /dev/null crit;
```

### Analyzing Logs

#### Using tail
```bash
# Follow access log
tail -f /var/log/nginx/access.log

# Follow error log
tail -f /var/log/nginx/error.log

# View last 100 lines
tail -n 100 /var/log/nginx/access.log
```

#### Using grep
```bash
# Find specific IP
grep "192.168.1.100" /var/log/nginx/access.log

# Find 404 errors
grep " 404 " /var/log/nginx/access.log

# Find POST requests
grep "POST" /var/log/nginx/access.log

# Find errors in last hour
grep "$(date -d '1 hour ago' '+%d/%b/%Y:%H')" /var/log/nginx/error.log
```

#### Using awk
```bash
# Count requests by IP
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head

# Count requests by status code
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Average response time
awk '{sum+=$NF; count++} END {print sum/count}' /var/log/nginx/access.log
```

### Log Rotation
```nginx
# /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    daily                    # Rotate daily
    missingok               # Don't error if log missing
    rotate 14               # Keep 14 days of logs
    compress                # Compress old logs
    delaycompress          # Compress on next rotation
    notifempty             # Don't rotate empty logs
    create 0640 nginx nginx # Create new log with permissions
    sharedscripts          # Run scripts once for all logs
    postrotate
        if [ -f /var/run/nginx.pid ]; then
            kill -USR1 `cat /var/run/nginx.pid`
        fi
    endscript
}
```

---

## Basic Security

### Hiding Nginx Version
```nginx
# In http or server block
http {
    server_tokens off;
}
```

### Restricting Access by IP
```nginx
# Allow specific IPs
location /admin {
    allow 192.168.1.0/24;
    allow 10.0.0.1;
    deny all;
    
    root /var/www/admin;
}

# Deny specific IPs
location / {
    deny 192.168.1.100;
    deny 10.0.0.0/8;
    allow all;
}
```

### Basic HTTP Authentication
```nginx
# Install htpasswd utility
# sudo apt install apache2-utils

# Create password file
# sudo htpasswd -c /etc/nginx/.htpasswd username

# Configure authentication
location /admin {
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    root /var/www/admin;
}
```

### Preventing Clickjacking
```nginx
# Add X-Frame-Options header
add_header X-Frame-Options "SAMEORIGIN" always;
```

### XSS Protection
```nginx
# Enable XSS filter
add_header X-XSS-Protection "1; mode=block" always;
```

### Content Type Options
```nginx
# Prevent MIME type sniffing
add_header X-Content-Type-Options "nosniff" always;
```

### Disabling Unwanted HTTP Methods
```nginx
# Allow only GET, HEAD, POST
if ($request_method !~ ^(GET|HEAD|POST)$ ) {
    return 405;
}
```

### Limiting Request Size
```nginx
# Limit client body size to 10MB
client_max_body_size 10m;

# Limit in specific location
location /upload {
    client_max_body_size 50m;
}
```

### Rate Limiting
```nginx
# Define rate limit zone (in http block)
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
    
    server {
        location / {
            # Apply rate limit
            limit_req zone=mylimit burst=20 nodelay;
        }
    }
}
```

### Connection Limiting
```nginx
# Limit concurrent connections per IP
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    
    server {
        location /download/ {
            limit_conn addr 2;  # Max 2 connections per IP
        }
    }
}
```

### Blocking User Agents
```nginx
# Block specific user agents
if ($http_user_agent ~* (badbot|malicious|scraper)) {
    return 403;
}
```

### Securing File Uploads
```nginx
location /uploads {
    # Prevent PHP execution in upload directory
    location ~ \.php$ {
        deny all;
    }
    
    # Allow only specific file types
    location ~ \.(jpg|jpeg|png|gif|pdf)$ {
        root /var/www/uploads;
    }
}
```

### SSL/TLS Configuration (Basic)
```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    # SSL certificates
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    
    # SSL protocols
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Strong ciphers
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512';
}
```

### Redirect HTTP to HTTPS
```nginx
# Redirect all HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com;
    # SSL configuration...
}
```

---

## Practice Exercises

### Exercise 1: Basic Static Website
Create a virtual host for a static website with proper logging and caching.

### Exercise 2: Multiple Virtual Hosts
Set up three different virtual hosts on the same server, each serving different content.

### Exercise 3: Access Control
Configure a virtual host with IP-based access control and basic authentication.

### Exercise 4: Log Analysis
Analyze Nginx access logs to find the top 10 IP addresses and most requested URLs.

### Exercise 5: Security Headers
Configure a server with all basic security headers enabled.

---

## Common Issues and Troubleshooting

### Port Already in Use
```bash
# Check what's using port 80
sudo lsof -i :80
sudo netstat -tlnp | grep :80

# Stop conflicting service
sudo systemctl stop apache2
```

### Permission Denied Errors
```bash
# Check Nginx user
ps aux | grep nginx

# Fix permissions
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

### 502 Bad Gateway
- Check if backend service is running
- Verify proxy_pass configuration
- Check error logs for details

### Configuration Test Fails
```bash
# Test configuration and show error details
sudo nginx -t

# Check syntax
sudo nginx -T | less
```

---

## Next Steps

After mastering the basics, proceed to:
1. **Intermediate Level**: Reverse proxy, load balancing, caching
2. **Advanced Level**: Performance tuning, security hardening, modules
3. **Source Code**: Understanding Nginx internals

---

## Quick Reference Commands

```bash
# Start/Stop/Restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx

# Test configuration
sudo nginx -t

# View logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Enable site
sudo ln -s /etc/nginx/sites-available/site /etc/nginx/sites-enabled/

# Disable site
sudo rm /etc/nginx/sites-enabled/site
```

---

*End of Basic Level Tutorial*
