# Nginx Complete Mastery Tutorial - Advanced Level

## Table of Contents
1. [Advanced Architecture Patterns](#advanced-architecture-patterns)
2. [Custom Modules Development](#custom-modules-development)
3. [Embedded Scripting with Lua](#embedded-scripting-with-lua)
4. [Advanced Security Hardening](#advanced-security-hardening)
5. [Performance Tuning Deep Dive](#performance-tuning-deep-dive)
6. [High Availability and Failover](#high-availability-and-failover)
7. [Dynamic Configuration](#dynamic-configuration)
8. [Stream Processing (TCP/UDP)](#stream-processing-tcpudp)
9. [Advanced Monitoring and Observability](#advanced-monitoring-and-observability)
10. [Troubleshooting and Debugging](#troubleshooting-and-debugging)
11. [Container and Cloud Deployments](#container-and-cloud-deployments)
12. [Advanced Use Cases](#advanced-use-cases)

---

## Advanced Architecture Patterns

### Microservices Gateway
```nginx
# Service registry using upstream blocks
upstream auth_service {
    least_conn;
    server auth1.internal:8001 max_fails=3 fail_timeout=30s;
    server auth2.internal:8001 max_fails=3 fail_timeout=30s;
    keepalive 32;
}

upstream user_service {
    least_conn;
    server user1.internal:8002 max_fails=3 fail_timeout=30s;
    server user2.internal:8002 max_fails=3 fail_timeout=30s;
    keepalive 32;
}

upstream order_service {
    least_conn;
    server order1.internal:8003 max_fails=3 fail_timeout=30s;
    server order2.internal:8003 max_fails=3 fail_timeout=30s;
    keepalive 32;
}

# API Gateway configuration
server {
    listen 443 ssl http2;
    server_name api.example.com;
    
    # SSL configuration
    ssl_certificate /etc/nginx/ssl/api.crt;
    ssl_certificate_key /etc/nginx/ssl/api.key;
    
    # Global rate limiting
    limit_req_zone $binary_remote_addr zone=api_global:10m rate=100r/s;
    limit_req zone=api_global burst=200 nodelay;
    
    # Authentication endpoint
    location /auth/ {
        limit_req_zone $binary_remote_addr zone=auth:10m rate=10r/s;
        limit_req zone=auth burst=20 nodelay;
        
        proxy_pass http://auth_service/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Caching for token validation
        proxy_cache auth_cache;
        proxy_cache_valid 200 5m;
        proxy_cache_key "$request_uri|$http_authorization";
    }
    
    # User service
    location /users/ {
        # Authentication subrequest
        auth_request /auth/validate;
        auth_request_set $user_id $upstream_http_x_user_id;
        
        proxy_pass http://user_service/;
        proxy_set_header X-User-ID $user_id;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
    
    # Order service
    location /orders/ {
        auth_request /auth/validate;
        auth_request_set $user_id $upstream_http_x_user_id;
        
        proxy_pass http://order_service/;
        proxy_set_header X-User-ID $user_id;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
    
    # Internal auth validation endpoint
    location = /auth/validate {
        internal;
        proxy_pass http://auth_service/validate;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
    }
}
```

### Multi-region Load Balancing with GeoIP
```nginx
# Requires ngx_http_geoip_module
http {
    geoip_country /usr/share/GeoIP/GeoIP.dat;
    
    # Map countries to backend regions
    map $geoip_country_code $backend_region {
        default "us-east";
        US "us-east";
        CA "us-east";
        GB "eu-west";
        DE "eu-west";
        FR "eu-west";
        JP "ap-east";
        CN "ap-east";
        AU "ap-east";
    }
    
    # Region-specific upstreams
    upstream us-east {
        server us-east-1.example.com:8080;
        server us-east-2.example.com:8080;
    }
    
    upstream eu-west {
        server eu-west-1.example.com:8080;
        server eu-west-2.example.com:8080;
    }
    
    upstream ap-east {
        server ap-east-1.example.com:8080;
        server ap-east-2.example.com:8080;
    }
    
    server {
        listen 80;
        server_name global.example.com;
        
        location / {
            # Route to appropriate region
            proxy_pass http://$backend_region;
            proxy_set_header X-Region $backend_region;
            proxy_set_header X-Country $geoip_country_code;
        }
    }
}
```

### Blue-Green Deployment
```nginx
# Using map for deployment switching
map $cookie_deployment $backend_pool {
    default "blue";
    "green" "green";
}

upstream blue {
    server blue1.internal:8080;
    server blue2.internal:8080;
}

upstream green {
    server green1.internal:8080;
    server green2.internal:8080;
}

server {
    listen 80;
    server_name app.example.com;
    
    location / {
        # Route based on backend pool
        proxy_pass http://$backend_pool;
        
        # Set deployment cookie for testing
        add_header X-Deployment $backend_pool;
    }
    
    # Admin endpoint to switch deployments
    location /admin/switch-deployment {
        allow 10.0.0.0/8;
        deny all;
        
        # Toggle deployment (requires custom logic)
        return 200 "Deployment switched to $arg_target";
    }
}
```

### Canary Releases
```nginx
# Split traffic between stable and canary
split_clients "${remote_addr}${http_user_agent}${date_gmt}" $backend_variant {
    95%     "stable";
    5%      "canary";
}

upstream stable {
    server stable1.internal:8080;
    server stable2.internal:8080;
}

upstream canary {
    server canary1.internal:8080;
    server canary2.internal:8080;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://$backend_variant;
        add_header X-Backend-Variant $backend_variant;
    }
}
```

### API Versioning
```nginx
# Extract API version from URI or header
map $request_uri $api_version {
    ~^/v1/    "v1";
    ~^/v2/    "v2";
    ~^/v3/    "v3";
    default   "v1";
}

# Version-specific upstreams
upstream api_v1 {
    server api-v1-1.internal:8001;
    server api-v1-2.internal:8001;
}

upstream api_v2 {
    server api-v2-1.internal:8002;
    server api-v2-2.internal:8002;
}

upstream api_v3 {
    server api-v3-1.internal:8003;
    server api-v3-2.internal:8003;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;
    
    location ~ ^/v[123]/ {
        # Route to version-specific backend
        proxy_pass http://api_$api_version;
        proxy_set_header X-API-Version $api_version;
    }
}
```

---

## Custom Modules Development

### Module Architecture Overview
```c
// Basic Nginx module structure
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>

// Module context
typedef struct {
    ngx_flag_t enable;
} ngx_http_custom_loc_conf_t;

// Function declarations
static ngx_int_t ngx_http_custom_handler(ngx_http_request_t *r);
static void *ngx_http_custom_create_loc_conf(ngx_conf_t *cf);
static char *ngx_http_custom_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child);

// Module directives
static ngx_command_t ngx_http_custom_commands[] = {
    {
        ngx_string("custom_enable"),
        NGX_HTTP_LOC_CONF|NGX_CONF_FLAG,
        ngx_conf_set_flag_slot,
        NGX_HTTP_LOC_CONF_OFFSET,
        offsetof(ngx_http_custom_loc_conf_t, enable),
        NULL
    },
    ngx_null_command
};

// HTTP module context
static ngx_http_module_t ngx_http_custom_module_ctx = {
    NULL,                                  /* preconfiguration */
    NULL,                                  /* postconfiguration */
    NULL,                                  /* create main configuration */
    NULL,                                  /* init main configuration */
    NULL,                                  /* create server configuration */
    NULL,                                  /* merge server configuration */
    ngx_http_custom_create_loc_conf,      /* create location configuration */
    ngx_http_custom_merge_loc_conf        /* merge location configuration */
};

// Module definition
ngx_module_t ngx_http_custom_module = {
    NGX_MODULE_V1,
    &ngx_http_custom_module_ctx,          /* module context */
    ngx_http_custom_commands,             /* module directives */
    NGX_HTTP_MODULE,                      /* module type */
    NULL,                                  /* init master */
    NULL,                                  /* init module */
    NULL,                                  /* init process */
    NULL,                                  /* init thread */
    NULL,                                  /* exit thread */
    NULL,                                  /* exit process */
    NULL,                                  /* exit master */
    NGX_MODULE_V1_PADDING
};
```

### Request Handler Example
```c
static ngx_int_t
ngx_http_custom_handler(ngx_http_request_t *r)
{
    ngx_buf_t    *b;
    ngx_chain_t   out;
    
    // Set response headers
    r->headers_out.status = NGX_HTTP_OK;
    r->headers_out.content_length_n = sizeof("Custom Module Response") - 1;
    
    // Allocate buffer for response
    b = ngx_calloc_buf(r->pool);
    if (b == NULL) {
        return NGX_HTTP_INTERNAL_SERVER_ERROR;
    }
    
    // Set buffer content
    b->pos = (u_char *) "Custom Module Response";
    b->last = b->pos + sizeof("Custom Module Response") - 1;
    b->memory = 1;
    b->last_buf = 1;
    
    // Set chain
    out.buf = b;
    out.next = NULL;
    
    // Send headers
    ngx_http_send_header(r);
    
    // Send body
    return ngx_http_output_filter(r, &out);
}
```

### Filter Module Example
```c
// Header filter
static ngx_int_t
ngx_http_custom_header_filter(ngx_http_request_t *r)
{
    // Add custom header
    ngx_table_elt_t *h;
    h = ngx_list_push(&r->headers_out.headers);
    if (h == NULL) {
        return NGX_ERROR;
    }
    
    h->hash = 1;
    ngx_str_set(&h->key, "X-Custom-Header");
    ngx_str_set(&h->value, "CustomValue");
    
    return ngx_http_next_header_filter(r);
}

// Body filter
static ngx_int_t
ngx_http_custom_body_filter(ngx_http_request_t *r, ngx_chain_t *in)
{
    // Process response body
    ngx_chain_t  *cl;
    
    for (cl = in; cl; cl = cl->next) {
        // Modify buffer content here
    }
    
    return ngx_http_next_body_filter(r, in);
}
```

### Compiling and Installing Custom Module
```bash
# Download Nginx source
wget http://nginx.org/download/nginx-1.24.0.tar.gz
tar -xzf nginx-1.24.0.tar.gz
cd nginx-1.24.0

# Configure with custom module
./configure \
    --prefix=/etc/nginx \
    --add-module=/path/to/custom_module \
    --with-http_ssl_module

# Compile and install
make
sudo make install
```

---

## Embedded Scripting with Lua

### Installing OpenResty or Lua Module
```bash
# Install OpenResty (Nginx + Lua)
wget https://openresty.org/download/openresty-1.21.4.1.tar.gz
tar -xzf openresty-1.21.4.1.tar.gz
cd openresty-1.21.4.1

./configure --prefix=/usr/local/openresty
make
sudo make install

# Or add Lua module to existing Nginx
# Download lua-nginx-module and LuaJIT, compile with Nginx
```

### Basic Lua Integration
```nginx
http {
    # Set Lua package path
    lua_package_path "/usr/local/openresty/lualib/?.lua;;";
    lua_package_cpath "/usr/local/openresty/lualib/?.so;;";
    
    # Shared dictionary for caching
    lua_shared_dict my_cache 10m;
    
    server {
        listen 80;
        
        # Inline Lua code
        location /hello {
            content_by_lua_block {
                ngx.say("Hello from Lua!")
            }
        }
        
        # External Lua file
        location /api {
            content_by_lua_file /path/to/script.lua;
        }
    }
}
```

### Advanced Lua Examples

#### Request Validation
```nginx
location /api/users {
    access_by_lua_block {
        local cjson = require "cjson"
        
        -- Validate request method
        if ngx.var.request_method ~= "POST" then
            ngx.status = ngx.HTTP_METHOD_NOT_ALLOWED
            ngx.say(cjson.encode({error = "Only POST allowed"}))
            ngx.exit(ngx.HTTP_METHOD_NOT_ALLOWED)
        end
        
        -- Validate Content-Type
        local content_type = ngx.var.http_content_type
        if content_type ~= "application/json" then
            ngx.status = ngx.HTTP_BAD_REQUEST
            ngx.say(cjson.encode({error = "Content-Type must be application/json"}))
            ngx.exit(ngx.HTTP_BAD_REQUEST)
        end
        
        -- Read and parse request body
        ngx.req.read_body()
        local body = ngx.req.get_body_data()
        
        if not body then
            ngx.status = ngx.HTTP_BAD_REQUEST
            ngx.say(cjson.encode({error = "Request body required"}))
            ngx.exit(ngx.HTTP_BAD_REQUEST)
        end
        
        local ok, data = pcall(cjson.decode, body)
        if not ok then
            ngx.status = ngx.HTTP_BAD_REQUEST
            ngx.say(cjson.encode({error = "Invalid JSON"}))
            ngx.exit(ngx.HTTP_BAD_REQUEST)
        end
        
        -- Validate required fields
        if not data.username or not data.email then
            ngx.status = ngx.HTTP_BAD_REQUEST
            ngx.say(cjson.encode({error = "username and email required"}))
            ngx.exit(ngx.HTTP_BAD_REQUEST)
        end
    }
    
    proxy_pass http://backend;
}
```

#### JWT Authentication
```nginx
location /protected {
    access_by_lua_block {
        local jwt = require "resty.jwt"
        local cjson = require "cjson"
        
        -- Get Authorization header
        local auth_header = ngx.var.http_authorization
        if not auth_header then
            ngx.status = ngx.HTTP_UNAUTHORIZED
            ngx.say(cjson.encode({error = "No authorization header"}))
            ngx.exit(ngx.HTTP_UNAUTHORIZED)
        end
        
        -- Extract token
        local _, _, token = string.find(auth_header, "Bearer%s+(.+)")
        if not token then
            ngx.status = ngx.HTTP_UNAUTHORIZED
            ngx.say(cjson.encode({error = "Invalid token format"}))
            ngx.exit(ngx.HTTP_UNAUTHORIZED)
        end
        
        -- Verify JWT
        local secret = "your-secret-key"
        local jwt_obj = jwt:verify(secret, token)
        
        if not jwt_obj.verified then
            ngx.status = ngx.HTTP_UNAUTHORIZED
            ngx.say(cjson.encode({error = "Invalid token", reason = jwt_obj.reason}))
            ngx.exit(ngx.HTTP_UNAUTHORIZED)
        end
        
        -- Set user context
        ngx.req.set_header("X-User-ID", jwt_obj.payload.sub)
        ngx.req.set_header("X-User-Role", jwt_obj.payload.role)
    }
    
    proxy_pass http://backend;
}
```

#### Rate Limiting with Redis
```nginx
location /api {
    access_by_lua_block {
        local redis = require "resty.redis"
        local cjson = require "cjson"
        
        -- Connect to Redis
        local red = redis:new()
        red:set_timeout(1000)
        
        local ok, err = red:connect("127.0.0.1", 6379)
        if not ok then
            ngx.log(ngx.ERR, "Failed to connect to Redis: ", err)
            -- Continue without rate limiting
            return
        end
        
        -- Rate limit key
        local key = "rate_limit:" .. ngx.var.remote_addr
        
        -- Increment counter
        local count, err = red:incr(key)
        if not count then
            ngx.log(ngx.ERR, "Failed to increment: ", err)
            return
        end
        
        -- Set expiration on first request
        if count == 1 then
            red:expire(key, 60)  -- 60 seconds window
        end
        
        -- Check limit (100 requests per minute)
        if count > 100 then
            ngx.status = ngx.HTTP_TOO_MANY_REQUESTS
            ngx.header["Retry-After"] = "60"
            ngx.say(cjson.encode({error = "Rate limit exceeded"}))
            ngx.exit(ngx.HTTP_TOO_MANY_REQUESTS)
        end
        
        -- Set rate limit headers
        ngx.header["X-RateLimit-Limit"] = "100"
        ngx.header["X-RateLimit-Remaining"] = tostring(100 - count)
        
        -- Close Redis connection
        red:set_keepalive(10000, 100)
    }
    
    proxy_pass http://backend;
}
```

#### Response Transformation
```nginx
location /api/data {
    proxy_pass http://backend;
    
    header_filter_by_lua_block {
        -- Modify response headers
        ngx.header["X-Processed-By"] = "Nginx-Lua"
        ngx.header["X-Response-Time"] = ngx.now() - ngx.req.start_time()
    }
    
    body_filter_by_lua_block {
        local cjson = require "cjson"
        
        -- Get response body
        local chunk = ngx.arg[1]
        local eof = ngx.arg[2]
        
        if eof then
            local ok, data = pcall(cjson.decode, chunk)
            if ok then
                -- Add metadata to response
                data.metadata = {
                    timestamp = ngx.time(),
                    server = ngx.var.hostname
                }
                
                ngx.arg[1] = cjson.encode(data)
            end
        end
    }
}
```

#### Dynamic Upstream Selection
```nginx
location / {
    set $backend '';
    
    rewrite_by_lua_block {
        local redis = require "resty.redis"
        local red = redis:new()
        
        red:connect("127.0.0.1", 6379)
        
        -- Get backend from Redis based on user
        local user_id = ngx.var.cookie_user_id
        if user_id then
            local backend, err = red:get("user:backend:" .. user_id)
            if backend and backend ~= ngx.null then
                ngx.var.backend = backend
            else
                ngx.var.backend = "default_backend"
            end
        else
            ngx.var.backend = "default_backend"
        end
        
        red:set_keepalive(10000, 100)
    }
    
    proxy_pass http://$backend;
}
```

---

## Advanced Security Hardening

### ModSecurity WAF Integration
```nginx
# Install ModSecurity module
# apt install libmodsecurity3 nginx-module-modsecurity

# Load ModSecurity module
load_module modules/ngx_http_modsecurity_module.so;

http {
    # Enable ModSecurity
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsec/main.conf;
    
    server {
        listen 443 ssl;
        
        location / {
            # ModSecurity is active
            proxy_pass http://backend;
        }
    }
}
```

ModSecurity configuration (`/etc/nginx/modsec/main.conf`):
```
# Load OWASP Core Rule Set
Include /etc/nginx/modsec/crs-setup.conf
Include /etc/nginx/modsec/rules/*.conf

# Custom rules
SecRule REQUEST_HEADERS:User-Agent "@contains nikto" \
    "id:1001,phase:1,deny,status:403,msg:'Nikto scanner detected'"

SecRule ARGS "@rx <script" \
    "id:1002,phase:2,deny,status:403,msg:'XSS attack detected'"
```

### Advanced DDoS Protection
```nginx
# Connection limiting
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;
limit_conn_zone $server_name zone=conn_limit_per_server:10m;

# Request rate limiting
limit_req_zone $binary_remote_addr zone=req_limit_per_ip:10m rate=10r/s;

# Bot protection using user agent
map $http_user_agent $limit_bots {
    default '';
    ~*(bot|crawler|spider|scraper) $binary_remote_addr;
}

limit_req_zone $limit_bots zone=bots:10m rate=1r/m;

server {
    listen 443 ssl http2;
    
    # Connection limits
    limit_conn conn_limit_per_ip 10;
    limit_conn conn_limit_per_server 1000;
    
    # Request rate limits
    limit_req zone=req_limit_per_ip burst=20 nodelay;
    limit_req zone=bots burst=2;
    
    # Slow request protection
    client_body_timeout 10s;
    client_header_timeout 10s;
    send_timeout 10s;
    
    location / {
        proxy_pass http://backend;
    }
}
```

### IP Reputation and Blacklisting
```nginx
# GeoIP blocking
http {
    geoip_country /usr/share/GeoIP/GeoIP.dat;
    
    map $geoip_country_code $allowed_country {
        default no;
        US yes;
        GB yes;
        CA yes;
        DE yes;
    }
    
    server {
        if ($allowed_country = no) {
            return 403 "Access denied from your country";
        }
    }
}
```

```nginx
# Dynamic IP blacklist using Lua
access_by_lua_block {
    local redis = require "resty.redis"
    local red = redis:new()
    
    red:connect("127.0.0.1", 6379)
    
    -- Check if IP is blacklisted
    local ip = ngx.var.remote_addr
    local blacklisted = red:get("blacklist:" .. ip)
    
    if blacklisted and blacklisted ~= ngx.null then
        ngx.exit(ngx.HTTP_FORBIDDEN)
    end
    
    red:set_keepalive(10000, 100)
}
```

### SSL Pinning and Certificate Transparency
```nginx
server {
    listen 443 ssl http2;
    
    # SSL configuration
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    # Public Key Pinning (deprecated, use Certificate Transparency)
    # add_header Public-Key-Pins 'pin-sha256="base64+primary=="; pin-sha256="base64+backup=="; max-age=5184000';
    
    # Expect-CT header for Certificate Transparency
    add_header Expect-CT 'max-age=86400, enforce, report-uri="https://example.com/ct-report"';
    
    # HSTS with preload
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
}
```

### Request/Response Filtering
```nginx
# SQL injection protection
location / {
    if ($query_string ~* "union.*select|insert.*into|drop.*table|update.*set|delete.*from") {
        return 403 "SQL injection attempt detected";
    }
    
    # XSS protection
    if ($query_string ~* "<script|javascript:|onerror=|onload=") {
        return 403 "XSS attempt detected";
    }
    
    # Path traversal protection
    if ($query_string ~* "\.\./|\.\.\\") {
        return 403 "Path traversal attempt detected";
    }
    
    proxy_pass http://backend;
}
```

---

## Performance Tuning Deep Dive

### Kernel and System Tuning
```bash
# /etc/sysctl.conf

# Increase system file descriptor limit
fs.file-max = 2097152

# Network tuning
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# TCP optimization
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15

# Enable TCP Fast Open
net.ipv4.tcp_fastopen = 3

# TCP window scaling
net.ipv4.tcp_window_scaling = 1
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# Connection tracking
net.netfilter.nf_conntrack_max = 1048576

# Apply settings
# sudo sysctl -p
```

User limits (`/etc/security/limits.conf`):
```
nginx soft nofile 1048576
nginx hard nofile 1048576
```

### Nginx Worker Optimization
```nginx
# Automatic worker processes based on CPU cores
worker_processes auto;

# Bind workers to specific CPU cores
worker_cpu_affinity auto;

# Increase worker connections
events {
    worker_connections 16384;
    use epoll;
    multi_accept on;
}

# Increase worker priority
worker_priority -5;

# Worker shutdown timeout
worker_shutdown_timeout 30s;

# Disable daemon mode for systemd
daemon off;
```

### Memory and Buffer Tuning
```nginx
http {
    # Client buffers
    client_body_buffer_size 128k;
    client_max_body_size 50m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;
    
    # Proxy buffers
    proxy_buffering on;
    proxy_buffer_size 8k;
    proxy_buffers 256 8k;
    proxy_busy_buffers_size 256k;
    proxy_temp_file_write_size 256k;
    proxy_max_temp_file_size 0;
    
    # FastCGI buffers
    fastcgi_buffering on;
    fastcgi_buffer_size 16k;
    fastcgi_buffers 256 16k;
    fastcgi_busy_buffers_size 256k;
    
    # Output buffers
    output_buffers 2 32k;
    postpone_output 1460;
    
    # Request/Response sizes
    client_body_timeout 60s;
    client_header_timeout 60s;
    send_timeout 60s;
    
    # Timeouts
    keepalive_timeout 75s;
    keepalive_requests 1000;
    
    # File caching
    open_file_cache max=200000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
}
```

### Advanced Caching Strategies
```nginx
http {
    # Proxy cache configuration
    proxy_cache_path /var/cache/nginx/hot
        levels=1:2
        keys_zone=hot_cache:100m
        max_size=10g
        inactive=24h
        use_temp_path=off;
    
    proxy_cache_path /var/cache/nginx/warm
        levels=1:2
        keys_zone=warm_cache:50m
        max_size=5g
        inactive=7d
        use_temp_path=off;
    
    proxy_cache_path /var/cache/nginx/cold
        levels=1:2
        keys_zone=cold_cache:20m
        max_size=2g
        inactive=30d
        use_temp_path=off;
    
    server {
        # Frequently accessed content (hot cache)
        location ~* ^/(api|assets)/ {
            proxy_cache hot_cache;
            proxy_cache_valid 200 1h;
            proxy_cache_valid 404 1m;
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
            proxy_cache_background_update on;
            proxy_cache_lock on;
            
            proxy_pass http://backend;
        }
        
        # Medium access content (warm cache)
        location ~* ^/content/ {
            proxy_cache warm_cache;
            proxy_cache_valid 200 24h;
            proxy_cache_valid 404 5m;
            
            proxy_pass http://backend;
        }
        
        # Rarely accessed content (cold cache)
        location ~* ^/archive/ {
            proxy_cache cold_cache;
            proxy_cache_valid 200 30d;
            proxy_cache_valid 404 1h;
            
            proxy_pass http://backend;
        }
    }
}
```

### TCP Optimization
```nginx
http {
    # Enable sendfile
    sendfile on;
    
    # Optimize TCP packet transmission
    tcp_nopush on;
    tcp_nodelay on;
    
    # Upstream keepalive
    upstream backend {
        server 127.0.0.1:8000;
        keepalive 256;
        keepalive_requests 10000;
        keepalive_timeout 60s;
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

### Compression Optimization
```nginx
http {
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_min_length 1000;
    gzip_proxied any;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/rss+xml
        application/atom+xml
        image/svg+xml;
    
    # Brotli compression (if module available)
    brotli on;
    brotli_comp_level 6;
    brotli_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        image/svg+xml;
}
```

---

## High Availability and Failover

### Active-Passive Setup with Keepalived
```bash
# Install keepalived
sudo apt install keepalived

# Master configuration (/etc/keepalived/keepalived.conf)
vrrp_script check_nginx {
    script "/usr/bin/pgrep nginx"
    interval 2
    weight 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1
    
    authentication {
        auth_type PASS
        auth_pass secret123
    }
    
    virtual_ipaddress {
        192.168.1.100/24
    }
    
    track_script {
        check_nginx
    }
}
```

Backup server configuration:
```bash
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    # ... rest same as master
}
```

### DNS-based Failover
```nginx
# Use DNS with multiple A records
# DNS configuration:
# example.com A 192.168.1.10  (primary)
# example.com A 192.168.1.11  (secondary)

# Nginx resolver configuration
resolver 8.8.8.8 8.8.4.4 valid=30s;
resolver_timeout 10s;

upstream backend {
    server backend.example.com:8080 resolve;
    # Nginx will re-resolve DNS periodically
}
```

### Health Checks and Circuit Breaker
```nginx
upstream backend {
    server backend1.example.com:8080 max_fails=3 fail_timeout=30s;
    server backend2.example.com:8080 max_fails=3 fail_timeout=30s backup;
    server backend3.example.com:8080 max_fails=3 fail_timeout=30s backup;
}

server {
    location / {
        proxy_pass http://backend;
        proxy_next_upstream error timeout http_500 http_502 http_503;
        proxy_next_upstream_tries 3;
        proxy_next_upstream_timeout 10s;
        
        # Custom error page
        error_page 502 503 504 /maintenance.html;
    }
    
    location = /maintenance.html {
        root /var/www/error;
        internal;
    }
}
```

### Session Persistence in HA Setup
```nginx
# Using shared memory for session storage
upstream backend {
    ip_hash;  # Basic session persistence
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}

# Or use sticky sessions with cookies (requires additional module)
# sticky cookie srv_id expires=1h domain=.example.com path=/;
```

---

## Dynamic Configuration

### Nginx Plus Dynamic Upstreams (Commercial)
```nginx
# Nginx Plus API for dynamic configuration
upstream backend {
    zone backend 64k;
    # Servers can be added/removed via API
}

server {
    location /api {
        api write=on;
        allow 127.0.0.1;
        deny all;
    }
}
```

API usage:
```bash
# Add server
curl -X POST -d '{"server":"192.168.1.10:8080"}' \
     http://localhost/api/6/http/upstreams/backend/servers

# Remove server
curl -X DELETE http://localhost/api/6/http/upstreams/backend/servers/0

# Get status
curl http://localhost/api/6/http/upstreams/backend
```

### Configuration Reload Strategies
```bash
# Zero-downtime configuration reload
sudo nginx -t && sudo nginx -s reload

# Automated configuration testing and reload
#!/bin/bash
NGINX_CONFIG="/etc/nginx/nginx.conf"

# Test configuration
if nginx -t -c $NGINX_CONFIG 2>/dev/null; then
    echo "Configuration valid, reloading..."
    nginx -s reload
    echo "Reload completed"
else
    echo "Configuration test failed!"
    nginx -t -c $NGINX_CONFIG
    exit 1
fi
```

### Dynamic SSL Certificate Management
```nginx
# Using Lua for dynamic certificate selection
ssl_certificate_by_lua_block {
    local ssl = require "ngx.ssl"
    local server_name = ssl.server_name()
    
    -- Load certificate based on SNI
    local cert_path = "/etc/nginx/ssl/" .. server_name .. ".crt"
    local key_path = "/etc/nginx/ssl/" .. server_name .. ".key"
    
    local cert_file = io.open(cert_path, "r")
    local key_file = io.open(key_path, "r")
    
    if cert_file and key_file then
        local cert_data = cert_file:read("*all")
        local key_data = key_file:read("*all")
        
        cert_file:close()
        key_file:close()
        
        ssl.set_cert(cert_data)
        ssl.set_priv_key(key_data)
    end
}
```

---

## Stream Processing (TCP/UDP)

### TCP Load Balancing
```nginx
stream {
    upstream database {
        least_conn;
        server db1.example.com:5432 max_fails=3 fail_timeout=30s;
        server db2.example.com:5432 max_fails=3 fail_timeout=30s;
        server db3.example.com:5432 backup;
    }
    
    server {
        listen 5432;
        proxy_pass database;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
    }
}
```

### UDP Load Balancing
```nginx
stream {
    upstream dns_servers {
        server 8.8.8.8:53;
        server 8.8.4.4:53;
    }
    
    server {
        listen 53 udp;
        proxy_pass dns_servers;
        proxy_timeout 1s;
        proxy_responses 1;
    }
}
```

### SSL/TLS Termination for TCP
```nginx
stream {
    upstream secure_backend {
        server backend.example.com:8443;
    }
    
    server {
        listen 443 ssl;
        
        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        
        proxy_pass secure_backend;
        proxy_ssl on;
    }
}
```

### Stream Module Logging
```nginx
stream {
    log_format stream_log '$remote_addr [$time_local] '
                         '$protocol $status $bytes_sent $bytes_received '
                         '$session_time "$upstream_addr" '
                         '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';
    
    access_log /var/log/nginx/stream-access.log stream_log;
    
    server {
        listen 3306;
        proxy_pass mysql_backend;
    }
}
```

---

## Advanced Monitoring and Observability

### Prometheus Exporter Integration
```nginx
# Using nginx-vts-module or nginx-prometheus-exporter

http {
    vhost_traffic_status_zone;
    
    server {
        listen 9145;
        
        location /metrics {
            vhost_traffic_status_display;
            vhost_traffic_status_display_format prometheus;
            
            allow 10.0.0.0/8;
            deny all;
        }
    }
}
```

### Structured Logging for Analysis
```nginx
log_format json_analytics escape=json
'{'
    '"timestamp":"$time_iso8601",'
    '"remote_addr":"$remote_addr",'
    '"request_method":"$request_method",'
    '"request_uri":"$request_uri",'
    '"status":$status,'
    '"body_bytes_sent":$body_bytes_sent,'
    '"request_time":$request_time,'
    '"upstream_response_time":"$upstream_response_time",'
    '"upstream_connect_time":"$upstream_connect_time",'
    '"upstream_header_time":"$upstream_header_time",'
    '"http_referrer":"$http_referer",'
    '"http_user_agent":"$http_user_agent",'
    '"http_x_forwarded_for":"$http_x_forwarded_for"'
'}';

access_log /var/log/nginx/access.json json_analytics;
```

### Distributed Tracing
```nginx
# Add trace ID to requests
map $http_x_trace_id $trace_id {
    default $http_x_trace_id;
    "" $request_id;
}

server {
    location / {
        proxy_set_header X-Trace-ID $trace_id;
        proxy_set_header X-Span-ID $request_id;
        
        add_header X-Trace-ID $trace_id;
        
        proxy_pass http://backend;
    }
}
```

### Real-time Metrics with StatsD
```nginx
# Using nginx-statsd module
location / {
    statsd_count "nginx.requests" 1;
    statsd_timing "nginx.request_time" $request_time;
    statsd_set "nginx.unique_users" $remote_addr;
    
    proxy_pass http://backend;
}
```

---

## Troubleshooting and Debugging

### Debug Logging
```nginx
# Enable debug log for specific connection
error_log /var/log/nginx/debug.log debug;

# Debug specific IP
events {
    debug_connection 192.168.1.100;
}
```

### Request Debugging Headers
```nginx
server {
    location / {
        # Add debug headers
        add_header X-Debug-Request-ID $request_id always;
        add_header X-Debug-Upstream-Addr $upstream_addr always;
        add_header X-Debug-Upstream-Status $upstream_status always;
        add_header X-Debug-Upstream-Response-Time $upstream_response_time always;
        add_header X-Debug-Cache-Status $upstream_cache_status always;
        
        proxy_pass http://backend;
    }
}
```

### Core Dump Analysis
```nginx
# Enable core dumps
worker_rlimit_core 500M;
working_directory /tmp/nginx-cores;

# Generate backtrace from core dump
# gdb /usr/sbin/nginx /tmp/nginx-cores/core
# (gdb) bt
```

### Performance Profiling
```bash
# CPU profiling with perf
sudo perf record -p $(pgrep nginx) -g -- sleep 60
sudo perf report

# Memory profiling with valgrind
valgrind --leak-check=full --show-leak-kinds=all nginx -g 'daemon off;'

# System call tracing
strace -p $(pgrep -f 'nginx: worker process')
```

---

## Container and Cloud Deployments

### Docker Configuration
```dockerfile
FROM nginx:1.24-alpine

# Copy custom configuration
COPY nginx.conf /etc/nginx/nginx.conf
COPY conf.d/ /etc/nginx/conf.d/

# Copy SSL certificates
COPY ssl/ /etc/nginx/ssl/

# Copy static files
COPY html/ /usr/share/nginx/html/

# Expose ports
EXPOSE 80 443

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

### Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
        - containerPort: 443
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: ssl
          mountPath: /etc/nginx/ssl
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: config
        configMap:
          name: nginx-config
      - name: ssl
        secret:
          secretName: nginx-ssl
```

### AWS Load Balancer Integration
```nginx
# Trust AWS ALB headers
set_real_ip_from 10.0.0.0/8;
real_ip_header X-Forwarded-For;
real_ip_recursive on;

server {
    listen 80;
    
    # Health check endpoint for ALB
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
    
    location / {
        proxy_pass http://backend;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
    }
}
```

---

## Advanced Use Cases

### API Rate Limiting with Different Tiers
```nginx
# Define rate limit zones for different tiers
limit_req_zone $tier_free zone=free:10m rate=10r/m;
limit_req_zone $tier_basic zone=basic:10m rate=100r/m;
limit_req_zone $tier_premium zone=premium:10m rate=1000r/m;

# Map API key to tier
map $http_x_api_key $tier {
    default "free";
    "key_basic_1" "basic";
    "key_basic_2" "basic";
    "key_premium_1" "premium";
}

# Apply appropriate rate limit
map $tier $tier_free {
    "free" $binary_remote_addr;
    default "";
}

map $tier $tier_basic {
    "basic" $binary_remote_addr;
    default "";
}

map $tier $tier_premium {
    "premium" $binary_remote_addr;
    default "";
}

server {
    location /api/ {
        limit_req zone=free burst=5 nodelay;
        limit_req zone=basic burst=50 nodelay;
        limit_req zone=premium burst=500 nodelay;
        
        proxy_pass http://backend;
    }
}
```

### GraphQL Gateway
```nginx
location /graphql {
    # Only allow POST requests
    if ($request_method != POST) {
        return 405;
    }
    
    # Validate Content-Type
    if ($http_content_type != "application/json") {
        return 415;
    }
    
    # Rate limiting for GraphQL
    limit_req zone=graphql_limit burst=10 nodelay;
    
    # Cache GET-like queries (introspection)
    proxy_cache graphql_cache;
    proxy_cache_methods POST;
    proxy_cache_key "$request_body";
    proxy_cache_valid 200 5m;
    
    proxy_pass http://graphql_backend;
}
```

### WebRTC Signaling Server
```nginx
upstream signaling {
    hash $remote_addr consistent;
    server signal1.example.com:8080;
    server signal2.example.com:8080;
}

server {
    listen 443 ssl http2;
    
    location /signaling {
        proxy_pass http://signaling;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Long timeout for signaling
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}
```

---

## Best Practices Summary

1. **Always test configuration** before reloading in production
2. **Use version control** for Nginx configurations
3. **Monitor performance metrics** continuously
4. **Implement proper logging** for debugging and analytics
5. **Keep Nginx updated** for security patches
6. **Use upstream keepalive** for better performance
7. **Implement health checks** for all backends
8. **Enable caching** where appropriate
9. **Use SSL/TLS** with strong ciphers
10. **Implement rate limiting** to prevent abuse

---

*End of Advanced Level Tutorial*
