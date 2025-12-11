# Nginx Complete Mastery Tutorial - Intermediate Level

## Table of Contents
1. [Reverse Proxy Fundamentals](#reverse-proxy-fundamentals)
2. [Load Balancing](#load-balancing)
3. [Caching Strategies](#caching-strategies)
4. [SSL/TLS Configuration](#ssltls-configuration)
5. [HTTP/2 and HTTP/3](#http2-and-http3)
6. [URL Rewriting and Redirects](#url-rewriting-and-redirects)
7. [Headers Manipulation](#headers-manipulation)
8. [Performance Optimization](#performance-optimization)
9. [Monitoring and Metrics](#monitoring-and-metrics)
10. [Advanced Location Matching](#advanced-location-matching)

---

## Reverse Proxy Fundamentals

### What is a Reverse Proxy?
A reverse proxy sits in front of backend servers and forwards client requests to those servers. Clients interact with the reverse proxy, unaware of the backend infrastructure.

```
Client → Nginx (Reverse Proxy) → Backend Server(s)
```

### Basic Reverse Proxy Configuration
```nginx
server {
    listen 80;
    server_name app.example.com;
    
    location / {
        # Forward requests to backend server
        proxy_pass http://localhost:3000;
        
        # Preserve original host header
        proxy_set_header Host $host;
        
        # Forward client IP
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Forward protocol information
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Proxying to Multiple Backends
```nginx
server {
    listen 80;
    server_name example.com;
    
    # API requests to backend API server
    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # Admin panel to separate backend
    location /admin/ {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
    }
    
    # Static files served directly
    location / {
        root /var/www/html;
        index index.html;
    }
}
```

### Upstream Definitions
```nginx
# Define backend servers
upstream backend_servers {
    server 192.168.1.10:8000;
    server 192.168.1.11:8000;
    server 192.168.1.12:8000;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Proxy Timeouts
```nginx
location /api/ {
    proxy_pass http://backend;
    
    # Connection timeout to backend
    proxy_connect_timeout 60s;
    
    # Timeout for sending request to backend
    proxy_send_timeout 60s;
    
    # Timeout for reading response from backend
    proxy_read_timeout 60s;
}
```

### Proxy Buffering
```nginx
location / {
    proxy_pass http://backend;
    
    # Enable buffering (default: on)
    proxy_buffering on;
    
    # Buffer size for response headers
    proxy_buffer_size 4k;
    
    # Number and size of buffers for response body
    proxy_buffers 8 4k;
    
    # Maximum buffer size
    proxy_busy_buffers_size 8k;
}
```

### WebSocket Proxying
```nginx
# WebSocket support requires HTTP/1.1 and Connection upgrade
upstream websocket_backend {
    server localhost:8080;
}

server {
    listen 80;
    
    location /ws/ {
        proxy_pass http://websocket_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        
        # Increase timeout for WebSocket connections
        proxy_read_timeout 86400;
    }
}
```

### Proxying to HTTPS Backend
```nginx
location / {
    proxy_pass https://backend.example.com;
    
    # Verify SSL certificate
    proxy_ssl_verify on;
    proxy_ssl_trusted_certificate /etc/ssl/certs/ca-bundle.crt;
    
    # Or disable verification (not recommended)
    # proxy_ssl_verify off;
    
    proxy_set_header Host $host;
}
```

### Error Handling in Reverse Proxy
```nginx
upstream backend {
    server 192.168.1.10:8000;
    server 192.168.1.11:8000 backup;  # Only used if primary fails
}

server {
    location / {
        proxy_pass http://backend;
        
        # Retry on specific errors
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        proxy_next_upstream_tries 3;
        proxy_next_upstream_timeout 10s;
        
        # Custom error pages
        proxy_intercept_errors on;
        error_page 502 503 504 /50x.html;
    }
    
    location = /50x.html {
        root /var/www/error;
    }
}
```

---

## Load Balancing

### Load Balancing Methods

#### Round Robin (Default)
```nginx
# Requests distributed evenly across servers
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}
```

#### Least Connections
```nginx
# Send requests to server with fewest active connections
upstream backend {
    least_conn;
    
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}
```

#### IP Hash
```nginx
# Same client always routed to same server
upstream backend {
    ip_hash;
    
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}
```

#### Generic Hash
```nginx
# Hash based on any variable
upstream backend {
    hash $request_uri consistent;
    
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}
```

#### Weighted Load Balancing
```nginx
# Distribute based on server capacity
upstream backend {
    server backend1.example.com weight=3;  # Gets 3x traffic
    server backend2.example.com weight=2;  # Gets 2x traffic
    server backend3.example.com weight=1;  # Gets 1x traffic
}
```

### Server Parameters

#### max_fails and fail_timeout
```nginx
upstream backend {
    server backend1.example.com max_fails=3 fail_timeout=30s;
    server backend2.example.com max_fails=3 fail_timeout=30s;
    server backend3.example.com max_fails=3 fail_timeout=30s;
}
# After 3 failures within 30s, server marked as unavailable for 30s
```

#### backup
```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backup.example.com backup;  # Only used when others fail
}
```

#### down
```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com down;  # Temporarily disabled
    server backend3.example.com;
}
```

#### max_conns
```nginx
upstream backend {
    server backend1.example.com max_conns=100;  # Limit concurrent connections
    server backend2.example.com max_conns=100;
}
```

### Complete Load Balancer Example
```nginx
upstream app_servers {
    least_conn;
    
    server app1.example.com:8000 weight=2 max_fails=3 fail_timeout=30s;
    server app2.example.com:8000 weight=2 max_fails=3 fail_timeout=30s;
    server app3.example.com:8000 weight=1 max_fails=3 fail_timeout=30s;
    server backup.example.com:8000 backup;
    
    # Health check (requires ngx_http_upstream_module)
    keepalive 32;
}

server {
    listen 80;
    server_name www.example.com;
    
    location / {
        proxy_pass http://app_servers;
        
        # Headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 10s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
        
        # Buffering
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        
        # Error handling
        proxy_next_upstream error timeout http_502 http_503;
        proxy_next_upstream_tries 2;
    }
}
```

### Session Persistence
```nginx
# Using ip_hash for session stickiness
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}

# Using cookie for session persistence (requires commercial version)
# upstream backend {
#     server backend1.example.com;
#     server backend2.example.com;
#     sticky cookie srv_id expires=1h domain=.example.com path=/;
# }
```

### Health Checks (Active)
```nginx
# Passive health checks (built-in)
upstream backend {
    server backend1.example.com max_fails=3 fail_timeout=30s;
    server backend2.example.com max_fails=3 fail_timeout=30s;
}

# Active health checks (requires NGINX Plus or third-party module)
# upstream backend {
#     zone backend 64k;
#     server backend1.example.com;
#     server backend2.example.com;
# }
# 
# server {
#     location / {
#         proxy_pass http://backend;
#         health_check interval=10 fails=3 passes=2;
#     }
# }
```

---

## Caching Strategies

### Proxy Cache Configuration
```nginx
# Define cache path and settings (in http block)
http {
    proxy_cache_path /var/cache/nginx 
                     levels=1:2 
                     keys_zone=my_cache:10m 
                     max_size=1g 
                     inactive=60m 
                     use_temp_path=off;
    
    server {
        location / {
            proxy_cache my_cache;
            proxy_pass http://backend;
            
            # Cache status header (for debugging)
            add_header X-Cache-Status $upstream_cache_status;
        }
    }
}
```

### Cache Parameters Explained
```nginx
proxy_cache_path /var/cache/nginx    # Cache directory
    levels=1:2                        # Directory structure (16*256 subdirs)
    keys_zone=my_cache:10m           # Shared memory zone name and size
    max_size=1g                      # Maximum cache size
    inactive=60m                     # Remove items not accessed in 60min
    use_temp_path=off;              # Write directly to cache path
```

### Cache Key Configuration
```nginx
location / {
    proxy_cache my_cache;
    
    # Default cache key
    # proxy_cache_key "$scheme$request_method$host$request_uri";
    
    # Custom cache key (ignore query parameters)
    proxy_cache_key "$scheme$request_method$host$uri";
    
    # Include specific query parameters
    proxy_cache_key "$scheme$request_method$host$uri$arg_page$arg_sort";
    
    proxy_pass http://backend;
}
```

### Cache Control Directives
```nginx
location / {
    proxy_cache my_cache;
    proxy_pass http://backend;
    
    # Cache valid responses
    proxy_cache_valid 200 302 10m;   # Cache 200 and 302 for 10 minutes
    proxy_cache_valid 404 1m;        # Cache 404 for 1 minute
    proxy_cache_valid any 5m;        # Cache anything else for 5 minutes
    
    # Minimum uses before caching
    proxy_cache_min_uses 2;          # Cache after 2 requests
    
    # Cache methods
    proxy_cache_methods GET HEAD;    # Only cache GET and HEAD
}
```

### Bypass Cache Conditions
```nginx
# Define cache bypass conditions
map $request_method $skip_cache {
    default         0;
    POST            1;
    PUT             1;
}

map $http_cookie $skip_cache {
    default         0;
    ~*session       1;  # Don't cache if session cookie exists
}

server {
    location / {
        proxy_cache my_cache;
        proxy_pass http://backend;
        
        # Skip cache based on conditions
        proxy_cache_bypass $skip_cache;
        proxy_no_cache $skip_cache;
        
        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

### Cache Purging
```nginx
# Manual cache purge location
location ~ /purge(/.*) {
    allow 127.0.0.1;
    deny all;
    
    proxy_cache_purge my_cache "$scheme$request_method$host$1";
}

# Usage:
# curl http://localhost/purge/page.html
```

### Cache Locking
```nginx
location / {
    proxy_cache my_cache;
    proxy_pass http://backend;
    
    # Prevent cache stampede
    proxy_cache_lock on;
    proxy_cache_lock_timeout 5s;
    proxy_cache_lock_age 10s;
}
```

### Slice Caching (for Large Files)
```nginx
location / {
    # Split large files into 1MB slices
    slice 1m;
    
    proxy_cache my_cache;
    proxy_pass http://backend;
    
    # Include range in cache key
    proxy_cache_key "$uri$is_args$args$slice_range";
    
    # Handle range requests
    proxy_set_header Range $slice_range;
    proxy_cache_valid 200 206 1h;
}
```

### FastCGI Cache (for PHP)
```nginx
# Define FastCGI cache path
fastcgi_cache_path /var/cache/nginx/fastcgi
                   levels=1:2
                   keys_zone=php_cache:10m
                   max_size=1g
                   inactive=60m;

server {
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_cache php_cache;
        
        # Cache configuration
        fastcgi_cache_valid 200 60m;
        fastcgi_cache_methods GET HEAD;
        
        # Cache key
        fastcgi_cache_key "$scheme$request_method$host$request_uri";
        
        # Bypass conditions
        fastcgi_cache_bypass $skip_cache;
        fastcgi_no_cache $skip_cache;
        
        # Headers
        add_header X-FastCGI-Cache $upstream_cache_status;
        
        include fastcgi_params;
    }
}
```

### Cache Status Values
```
- MISS: Response not found in cache, fetched from backend
- HIT: Response served from cache
- EXPIRED: Cached item expired, fetched from backend
- STALE: Serving stale content (when backend unavailable)
- UPDATING: Content being updated in background
- REVALIDATED: Expired content still valid
- BYPASS: Cache bypassed by directive
```

---

## SSL/TLS Configuration

### Obtaining SSL Certificates

#### Using Let's Encrypt (Certbot)
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d example.com -d www.example.com

# Certificate auto-renewal (cron job)
sudo certbot renew --dry-run
```

### Basic SSL Configuration
```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    # Certificate files
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # Basic SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    
    root /var/www/html;
}

# HTTP to HTTPS redirect
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}
```

### Strong SSL Configuration
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    
    # SSL Protocols
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;
    
    # Ciphers
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    
    # SSL Session Cache
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    
    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    root /var/www/html;
}
```

### SSL Configuration Snippet
```nginx
# /etc/nginx/snippets/ssl-params.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers off;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;

# Usage in server block:
server {
    listen 443 ssl http2;
    include snippets/ssl-params.conf;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
}
```

### Diffie-Hellman Parameters
```bash
# Generate DH parameters
sudo openssl dhparam -out /etc/nginx/dhparam.pem 2048
```

```nginx
server {
    listen 443 ssl http2;
    
    ssl_dhparam /etc/nginx/dhparam.pem;
    # ... other SSL settings
}
```

### Client Certificate Authentication
```nginx
server {
    listen 443 ssl;
    
    # Server certificates
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    
    # Client certificate verification
    ssl_client_certificate /etc/nginx/ssl/ca.crt;
    ssl_verify_client on;  # or 'optional'
    ssl_verify_depth 2;
    
    location / {
        # Client certificate DN available in $ssl_client_s_dn
        proxy_set_header X-SSL-Client-DN $ssl_client_s_dn;
        proxy_pass http://backend;
    }
}
```

### SSL Testing Tools
```bash
# Test SSL configuration
openssl s_client -connect example.com:443

# Check certificate expiration
echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Test SSL/TLS protocols
nmap --script ssl-enum-ciphers -p 443 example.com

# Online tools:
# https://www.ssllabs.com/ssltest/
```

---

## HTTP/2 and HTTP/3

### Enabling HTTP/2
```nginx
server {
    # Enable HTTP/2 with SSL
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    root /var/www/html;
}
```

### HTTP/2 Server Push
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    location = /index.html {
        # Push resources to client
        http2_push /css/style.css;
        http2_push /js/app.js;
        http2_push /images/logo.png;
    }
}
```

### HTTP/2 Configuration Parameters
```nginx
http {
    # Maximum concurrent streams per connection
    http2_max_concurrent_streams 128;
    
    # Maximum size of request body
    http2_body_preread_size 64k;
    
    # Chunk size for response body
    http2_chunk_size 8k;
    
    server {
        listen 443 ssl http2;
        # ... SSL config
    }
}
```

### HTTP/3 (QUIC) Setup
```nginx
# Requires Nginx compiled with QUIC support
server {
    listen 443 ssl http2;
    listen 443 quic reuseport;  # HTTP/3
    
    server_name example.com;
    
    # SSL certificates
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    # QUIC-specific settings
    ssl_early_data on;
    quic_retry on;
    
    # Advertise HTTP/3 support
    add_header Alt-Svc 'h3=":443"; ma=86400';
    
    root /var/www/html;
}
```

---

## URL Rewriting and Redirects

### Simple Redirects
```nginx
# Permanent redirect (301)
server {
    listen 80;
    server_name old-domain.com;
    return 301 https://new-domain.com$request_uri;
}

# Temporary redirect (302)
location /old-page {
    return 302 /new-page;
}

# Redirect with custom message
location /maintenance {
    return 503 "Site under maintenance";
}
```

### Rewrite Directive
```nginx
# Syntax: rewrite regex replacement [flag];

# Flags:
# - last: stop processing, search for new location
# - break: stop processing, continue in current location
# - redirect: return 302 temporary redirect
# - permanent: return 301 permanent redirect

location /old/ {
    # Rewrite and continue processing
    rewrite ^/old/(.*)$ /new/$1 last;
}

location /products/ {
    # Rewrite URL format
    rewrite ^/products/([0-9]+)$ /products?id=$1 last;
}
```

### Common Rewrite Examples
```nginx
# Remove www prefix
server {
    listen 80;
    server_name www.example.com;
    return 301 $scheme://example.com$request_uri;
}

# Add www prefix
server {
    listen 80;
    server_name example.com;
    return 301 $scheme://www.example.com$request_uri;
}

# Remove trailing slash
rewrite ^/(.*)/$ /$1 permanent;

# Add trailing slash
rewrite ^([^.]*[^/])$ $1/ permanent;

# Remove index.php from URL
if ($request_uri ~* "^(.*/)index\.php$") {
    return 301 $1;
}
```

### Clean URLs for Applications
```nginx
# WordPress
location / {
    try_files $uri $uri/ /index.php?$args;
}

# Laravel/Modern PHP frameworks
location / {
    try_files $uri $uri/ /index.php?$query_string;
}

# Node.js SPA (React, Vue, Angular)
location / {
    try_files $uri $uri/ /index.html;
}
```

### Conditional Rewrites
```nginx
# Based on user agent
if ($http_user_agent ~* (mobile|android|iphone)) {
    rewrite ^/$ /mobile/ permanent;
}

# Based on query parameter
if ($arg_lang = "es") {
    rewrite ^/$ /es/ permanent;
}

# Based on request method
if ($request_method = POST) {
    return 405;
}
```

### Named Captures in Rewrites
```nginx
location /user/ {
    # Capture named groups
    rewrite ^/user/(?<username>[a-z0-9]+)/profile$ /profile.php?user=$username last;
}
```

---

## Headers Manipulation

### Adding Headers
```nginx
location / {
    # Add custom header
    add_header X-Custom-Header "CustomValue";
    
    # Add header always (even on error responses)
    add_header X-Always-Present "Value" always;
    
    # Multiple headers
    add_header Cache-Control "public, max-age=3600";
    add_header Pragma "cache";
}
```

### Setting Headers
```nginx
location / {
    # Set request headers (to backend)
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    proxy_pass http://backend;
}
```

### Removing Headers
```nginx
# Remove header from response
proxy_hide_header X-Powered-By;
proxy_hide_header Server;

# Pass specific headers through
proxy_pass_header Set-Cookie;
```

### Security Headers
```nginx
# HSTS (HTTP Strict Transport Security)
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

# Prevent clickjacking
add_header X-Frame-Options "SAMEORIGIN" always;

# XSS Protection
add_header X-XSS-Protection "1; mode=block" always;

# Prevent MIME sniffing
add_header X-Content-Type-Options "nosniff" always;

# Content Security Policy
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" always;

# Referrer Policy
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# Permissions Policy
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

### CORS Headers
```nginx
location /api/ {
    # Allow all origins
    add_header Access-Control-Allow-Origin * always;
    
    # Or specific origin
    # add_header Access-Control-Allow-Origin "https://example.com" always;
    
    # Allow credentials
    add_header Access-Control-Allow-Credentials "true" always;
    
    # Allowed methods
    add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
    
    # Allowed headers
    add_header Access-Control-Allow-Headers "Authorization, Content-Type, X-Requested-With" always;
    
    # Max age for preflight request
    add_header Access-Control-Max-Age "3600" always;
    
    # Handle preflight requests
    if ($request_method = OPTIONS) {
        return 204;
    }
    
    proxy_pass http://backend;
}
```

### Conditional Headers with Map
```nginx
# Define mapping
map $sent_http_content_type $cache_control {
    default "no-cache";
    ~image/ "public, max-age=31536000";
    ~text/css "public, max-age=31536000";
    ~application/javascript "public, max-age=31536000";
}

server {
    location / {
        add_header Cache-Control $cache_control;
        root /var/www/html;
    }
}
```

---

## Performance Optimization

### Gzip Compression
```nginx
http {
    # Enable gzip
    gzip on;
    
    # Compression level (1-9, higher = more CPU)
    gzip_comp_level 6;
    
    # Minimum file size to compress
    gzip_min_length 1000;
    
    # File types to compress
    gzip_types 
        text/plain
        text/css
        text/javascript
        text/xml
        application/json
        application/javascript
        application/xml+rss
        application/xml
        image/svg+xml;
    
    # Enable for proxied requests
    gzip_proxied any;
    
    # Add Vary: Accept-Encoding header
    gzip_vary on;
    
    # Disable for IE6
    gzip_disable "msie6";
    
    # Buffer size
    gzip_buffers 16 8k;
}
```

### Brotli Compression (requires module)
```nginx
http {
    brotli on;
    brotli_comp_level 6;
    brotli_types 
        text/plain
        text/css
        text/javascript
        application/json
        application/javascript
        image/svg+xml;
}
```

### Static File Caching
```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|otf)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

### Buffer Optimization
```nginx
http {
    # Client request body buffer
    client_body_buffer_size 16k;
    
    # Client request header buffer
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;
    
    # Output buffering
    output_buffers 2 32k;
    postpone_output 1460;
    
    # Sendfile optimization
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
}
```

### Keepalive Connections
```nginx
http {
    # Client keepalive
    keepalive_timeout 65;
    keepalive_requests 100;
    
    # Upstream keepalive
    upstream backend {
        server 127.0.0.1:8000;
        keepalive 32;  # Keep 32 idle connections
    }
    
    server {
        location / {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
    }
}
```

### File Descriptor Cache
```nginx
http {
    # Cache file metadata
    open_file_cache max=10000 inactive=30s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
}
```

### Worker Process Optimization
```nginx
# Set to number of CPU cores
worker_processes auto;

# Bind workers to CPU cores
worker_cpu_affinity auto;

# Increase worker connections
events {
    worker_connections 2048;
    use epoll;  # Linux
    multi_accept on;
}

# Priority (nice value)
worker_priority -5;
```

### Rate Limiting for Performance
```nginx
http {
    # Limit request rate
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    
    # Limit concurrent connections
    limit_conn_zone $binary_remote_addr zone=conn:10m;
    
    server {
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            limit_conn conn 10;
            proxy_pass http://backend;
        }
    }
}
```

---

## Monitoring and Metrics

### Stub Status Module
```nginx
# Enable stub_status
server {
    listen 8080;
    server_name localhost;
    
    location /nginx_status {
        stub_status;
        allow 127.0.0.1;
        deny all;
    }
}
```

Access: `curl http://localhost:8080/nginx_status`

Output:
```
Active connections: 291
server accepts handled requests
 16630948 16630948 31070465
Reading: 6 Writing: 179 Waiting: 106
```

### Custom Log Format for Metrics
```nginx
# Detailed logging format
log_format detailed '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'rt=$request_time uct="$upstream_connect_time" '
                    'uht="$upstream_header_time" urt="$upstream_response_time"';

access_log /var/log/nginx/detailed.log detailed;
```

### Prometheus Metrics (requires module)
```nginx
# nginx-prometheus-exporter or nginx-vts-module
server {
    listen 9145;
    
    location /metrics {
        vhost_traffic_status_display;
        vhost_traffic_status_display_format prometheus;
        allow 127.0.0.1;
        deny all;
    }
}
```

### Real-time Monitoring with GoAccess
```bash
# Install GoAccess
sudo apt install goaccess

# Real-time HTML report
sudo goaccess /var/log/nginx/access.log -o /var/www/html/report.html --real-time-html --daemonize

# Terminal dashboard
sudo goaccess /var/log/nginx/access.log -c
```

### Log Analysis Scripts
```bash
# Top 10 IPs by request count
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Top 10 requested URLs
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Response status distribution
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Average request time
awk '{sum+=$NF; count++} END {print sum/count}' /var/log/nginx/access.log

# Requests per hour
awk '{print $4}' /var/log/nginx/access.log | cut -d: -f2 | sort | uniq -c

# 4xx and 5xx errors
awk '$9 ~ /^[45]/ {print $0}' /var/log/nginx/access.log
```

---

## Advanced Location Matching

### Location Priority Order
```
1. Exact match:              location = /path
2. Preferential prefix:      location ^~ /path
3. Regular expression:       location ~ /path
4. Regular (insensitive):    location ~* /path
5. Prefix match:             location /path
```

### Complex Location Examples
```nginx
server {
    # 1. Exact match (highest priority)
    location = / {
        return 200 "exact match for /";
    }
    
    # 2. Preferential prefix (stops regex matching)
    location ^~ /static/ {
        root /var/www;
    }
    
    # 3. Case-sensitive regex
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm.sock;
    }
    
    # 4. Case-insensitive regex
    location ~* \.(jpg|jpeg|png|gif)$ {
        expires 30d;
    }
    
    # 5. Prefix match (lowest priority)
    location / {
        proxy_pass http://backend;
    }
}
```

### Nested Locations
```nginx
location /api/ {
    # Applies to all /api/* requests
    proxy_set_header X-API-Version "v1";
    
    location /api/public/ {
        # Specific handling for public API
        limit_req zone=public_api;
        proxy_pass http://backend;
    }
    
    location /api/admin/ {
        # Admin API with authentication
        auth_basic "Admin API";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://backend;
    }
}
```

### Named Locations
```nginx
location / {
    try_files $uri $uri/ @fallback;
}

location @fallback {
    proxy_pass http://backend;
    proxy_set_header Host $host;
}
```

---

## Practice Projects

### Project 1: Multi-tier Web Application
Set up Nginx as reverse proxy for:
- React frontend on port 3000
- Node.js API on port 8000
- Admin panel on port 8080
With SSL, caching, and load balancing.

### Project 2: High-Performance Static Site
Configure Nginx for maximum performance:
- Gzip/Brotli compression
- Browser caching
- HTTP/2
- Security headers

### Project 3: API Gateway
Build an API gateway with:
- Rate limiting
- Authentication
- Request/response transformation
- Load balancing across multiple backends

---

*End of Intermediate Level Tutorial*
