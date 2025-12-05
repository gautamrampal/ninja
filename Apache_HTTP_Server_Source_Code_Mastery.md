# Apache HTTP Server Source Code Mastery - Complete Tutorial

## Table of Contents
1. [Introduction to Apache HTTP Server](#introduction)
2. [Basic Concepts](#basic-concepts)
3. [Build System and Compilation](#build-system)
4. [Core Architecture](#core-architecture)
5. [Intermediate Concepts](#intermediate-concepts)
6. [Advanced Topics](#advanced-topics)
7. [Module System Deep Dive](#module-system)
8. [Request Processing Pipeline](#request-processing)
9. [Memory Management](#memory-management)
10. [Multi-Processing Modules (MPM)](#mpm)
11. [Configuration System](#configuration-system)
12. [Hook System](#hook-system)
13. [Filter Chain Architecture](#filter-chain)
14. [Performance Optimization](#performance)
15. [Security Implementation](#security)
16. [Source Code Navigation Guide](#code-navigation)
17. [Practical Examples](#practical-examples)

---

## 1. Introduction to Apache HTTP Server {#introduction}

### What is Apache HTTP Server?

Apache HTTP Server (httpd) is the world's most popular open-source web server. Written primarily in C, it serves as a robust, commercial-grade implementation of HTTP protocol.

**Key Statistics:**
- First released: 1995
- Language: C (99%), with some components in other languages
- Architecture: Modular, extensible
- License: Apache License 2.0

### Source Code Overview

**Repository Structure:**
```
httpd/
├── server/          # Core server functionality
├── modules/         # Standard modules
├── include/         # Header files
├── os/             # OS-specific code
├── support/        # Utility programs
├── docs/           # Documentation
├── build/          # Build scripts
└── test/           # Test suites
```

### Why Study Apache Source Code?

- Master production-grade C programming
- Understand network server architecture
- Learn modular design patterns
- Study high-performance concurrent systems
- Understand HTTP protocol implementation

---

## 2. Basic Concepts {#basic-concepts}

### 2.1 Apache Portable Runtime (APR)

APR provides a portable abstraction layer for OS-specific functionality.

**Core APR Concepts:**

```c
/* APR Pool - Memory management foundation */
#include "apr_pools.h"
#include "apr_general.h"

/* Creating a memory pool */
apr_pool_t *pool;
apr_status_t status;

// Initialize APR
status = apr_initialize();
if (status != APR_SUCCESS) {
    // Handle error
}

// Create root pool
status = apr_pool_create(&pool, NULL);

// Use the pool for allocations
char *buffer = apr_palloc(pool, 1024);

// Cleanup - destroys all memory allocated from pool
apr_pool_destroy(pool);
apr_terminate();
```

**APR Key Components:**

1. **Memory Pools** (`apr_pools.h`)
   - Hierarchical memory allocation
   - Automatic cleanup
   - No individual free() needed

2. **Hash Tables** (`apr_hash.h`)
```c
#include "apr_hash.h"

apr_hash_t *hash;
hash = apr_hash_make(pool);

// Insert key-value pair
apr_hash_set(hash, "key", APR_HASH_KEY_STRING, "value");

// Retrieve value
const char *value = apr_hash_get(hash, "key", APR_HASH_KEY_STRING);
```

3. **Arrays** (`apr_tables.h`)
```c
#include "apr_tables.h"

apr_array_header_t *arr;
arr = apr_array_make(pool, 10, sizeof(const char *));

// Add element
*(const char **)apr_array_push(arr) = "element";
```

4. **File I/O** (`apr_file_io.h`)
```c
#include "apr_file_io.h"

apr_file_t *file;
apr_status_t rv;

// Open file
rv = apr_file_open(&file, "/path/to/file", 
                   APR_READ | APR_WRITE | APR_CREATE,
                   APR_OS_DEFAULT, pool);

// Read from file
char buffer[1024];
apr_size_t bytes_read = sizeof(buffer);
apr_file_read(file, buffer, &bytes_read);

// Close file
apr_file_close(file);
```

### 2.2 Basic Server Data Structures

**Server Record** (`server/core.c`):
```c
/* Main server structure */
struct server_rec {
    server_rec *next;              // Linked list of servers
    
    const char *defn_name;         // Server definition name
    unsigned defn_line_number;     // Line number in config
    
    char *server_admin;            // ServerAdmin email
    char *server_hostname;         // ServerName
    apr_port_t port;               // Port number
    
    char *path;                    // DocumentRoot path
    apr_array_header_t *names;     // ServerAlias list
    apr_array_header_t *wild_names; // Wildcard ServerAlias
    
    void *module_config;           // Per-server module config
    void *lookup_defaults;         // Per-dir config defaults
    
    int is_virtual;                // Virtual host flag
    int keep_alive_timeout;        // KeepAlive timeout
    int keep_alive;                // KeepAlive enabled
    
    const char *error_fname;       // Error log filename
    apr_file_t *error_log;         // Error log file handle
    int loglevel;                  // Log level
    
    int limit_req_line;            // Max request line size
    int limit_req_fieldsize;       // Max header field size
    int limit_req_fields;          // Max number of headers
};
```

**Request Record** (`include/httpd.h`):
```c
/* Request structure - central to Apache */
struct request_rec {
    apr_pool_t *pool;              // Request pool
    conn_rec *connection;          // Connection this request is on
    server_rec *server;            // Server handling request
    
    request_rec *next;             // For internal redirects
    request_rec *prev;             // Previous request
    request_rec *main;             // Main request (for subrequests)
    
    /* Request-Line components */
    const char *the_request;       // Full request line
    int assbackwards;              // HTTP/0.9 request flag
    
    int proxyreq;                  // Proxy request flag
    int header_only;               // HEAD request flag
    
    char *protocol;                // Protocol version "HTTP/1.1"
    int proto_num;                 // Protocol number
    const char *hostname;          // Host: header value
    
    apr_time_t request_time;       // Request timestamp
    const char *status_line;       // Status line for response
    int status;                    // HTTP status code
    
    /* Request method */
    const char *method;            // GET, POST, etc.
    int method_number;             // M_GET, M_POST, etc.
    
    /* HTTP headers */
    apr_table_t *headers_in;       // Request headers
    apr_table_t *headers_out;      // Response headers
    apr_table_t *err_headers_out;  // Error response headers
    apr_table_t *subprocess_env;   // CGI environment
    apr_table_t *notes;            // Inter-module communication
    
    /* Request URI components */
    const char *uri;               // Request URI
    char *filename;                // Filesystem path
    char *canonical_filename;      // Canonicalized filename
    char *path_info;               // PATH_INFO
    char *args;                    // Query string
    
    struct ap_conf_vector_t *per_dir_config; // Per-dir config
    struct ap_conf_vector_t *request_config;  // Request config
    
    /* Content info */
    const char *content_type;      // Content-Type
    const char *handler;           // Handler name
    const char *content_encoding;  // Content-Encoding
    apr_array_header_t *content_languages; // Content-Language
    
    /* Authentication */
    char *user;                    // Authenticated user
    char *ap_auth_type;            // Auth type used
    
    /* Response body */
    apr_off_t bytes_sent;          // Bytes sent to client
    apr_time_t mtime;              // Modification time
    
    /* Chunked encoding */
    int chunked;                   // Chunked transfer encoding
    int read_chunked;              // Reading chunked request
    
    /* Range requests */
    const char *range;             // Range header
    apr_off_t clength;             // Content-Length
    apr_off_t remaining;           // Bytes remaining to read
    apr_off_t read_length;         // Bytes read so far
    
    /* Filters */
    struct ap_filter_t *input_filters;  // Input filter chain
    struct ap_filter_t *output_filters; // Output filter chain
    
    /* ETag and validation */
    const char *etag;              // Entity tag
    
    /* Used for ap_process_request_internal */
    int eos_sent;                  // End-of-stream sent
};
```

**Connection Record** (`include/httpd.h`):
```c
struct conn_rec {
    apr_pool_t *pool;              // Connection pool
    server_rec *base_server;       // Base server
    void *vhost_lookup_data;       // Vhost lookup data
    
    /* Socket information */
    apr_sockaddr_t *local_addr;    // Local address
    apr_sockaddr_t *client_addr;   // Client address
    char *client_ip;               // Client IP string
    char *remote_host;             // Remote hostname
    char *remote_logname;          // Remote logname
    
    /* Connection ID */
    unsigned long id;              // Unique connection ID
    
    /* Per-connection config */
    struct ap_conf_vector_t *conn_config;
    
    /* Notes for modules */
    apr_table_t *notes;
    
    /* Input/Output filters */
    struct ap_filter_t *input_filters;
    struct ap_filter_t *output_filters;
    
    /* Socket bucket allocator */
    void *sbh;
    
    /* Connection state */
    int keepalive;                 // Keepalive state
    int double_reverse;            // Double-reverse DNS flag
    int aborted;                   // Connection aborted flag
    
    /* Current request being served */
    request_rec *current_thread;
    
    /* SSL/TLS info */
    int is_ssl;                    // SSL connection flag
    int ssl_session_id_len;        // SSL session ID length
    char ssl_session_id[SSL_MAX_SESSION_ID_LENGTH];
};
```

### 2.3 Module Structure

Every Apache module follows this structure:

```c
#include "httpd.h"
#include "http_config.h"
#include "http_protocol.h"
#include "ap_config.h"

/* Module configuration structure */
typedef struct {
    int enabled;
    const char *setting;
} my_config;

/* Create per-directory config */
static void *create_dir_config(apr_pool_t *p, char *dir)
{
    my_config *cfg = apr_pcalloc(p, sizeof(my_config));
    cfg->enabled = 0;
    cfg->setting = NULL;
    return cfg;
}

/* Merge configurations */
static void *merge_dir_config(apr_pool_t *p, void *base_conf, void *new_conf)
{
    my_config *merged = apr_pcalloc(p, sizeof(my_config));
    my_config *base = (my_config *)base_conf;
    my_config *new = (my_config *)new_conf;
    
    merged->enabled = new->enabled ? new->enabled : base->enabled;
    merged->setting = new->setting ? new->setting : base->setting;
    
    return merged;
}

/* Configuration directives */
static const command_rec my_directives[] = {
    AP_INIT_FLAG("MyEnable", ap_set_flag_slot,
                 (void *)APR_OFFSETOF(my_config, enabled),
                 OR_ALL, "Enable my module"),
    AP_INIT_TAKE1("MySetting", ap_set_string_slot,
                  (void *)APR_OFFSETOF(my_config, setting),
                  OR_ALL, "Set configuration value"),
    {NULL}
};

/* Register hooks */
static void register_hooks(apr_pool_t *p)
{
    ap_hook_handler(my_handler, NULL, NULL, APR_HOOK_MIDDLE);
}

/* Module declaration */
module AP_MODULE_DECLARE_DATA my_module = {
    STANDARD20_MODULE_STUFF,
    create_dir_config,     // Per-directory config creator
    merge_dir_config,      // Per-directory merger
    NULL,                  // Per-server config creator
    NULL,                  // Per-server merger
    my_directives,         // Configuration directives
    register_hooks         // Register hooks
};
```

---

## 3. Build System and Compilation {#build-system}

### 3.1 Building Apache from Source

**Prerequisites:**
```bash
# Required tools
- C compiler (GCC or Clang)
- APR (Apache Portable Runtime)
- APR-Util
- PCRE (Perl Compatible Regular Expressions)
```

**Basic Build Process:**
```bash
# Download source
wget https://downloads.apache.org/httpd/httpd-2.4.xx.tar.gz
tar -xzf httpd-2.4.xx.tar.gz
cd httpd-2.4.xx

# Configure
./configure \
    --prefix=/usr/local/apache2 \
    --enable-modules=all \
    --enable-mods-shared=all \
    --enable-mpms-shared=all \
    --with-mpm=event

# Compile
make

# Install
sudo make install
```

### 3.2 Configure Options Deep Dive

**Key Configuration Flags:**
```bash
# Installation paths
--prefix=/path/to/install      # Base installation directory
--exec-prefix=/path             # Architecture-dependent files
--bindir=/path/bin             # User executables
--sbindir=/path/sbin           # System admin executables
--sysconfdir=/path/etc         # Configuration files

# MPM selection
--with-mpm=prefork             # Prefork MPM (one process per request)
--with-mpm=worker              # Worker MPM (threads)
--with-mpm=event               # Event MPM (async I/O)

# Module compilation
--enable-modules=all           # Enable all modules
--enable-mods-shared=all       # Build all as shared (.so)
--enable-module=static         # Build specific module statically
--disable-module               # Disable specific module

# APR configuration
--with-apr=/path/to/apr        # APR location
--with-apr-util=/path          # APR-Util location

# SSL/TLS support
--enable-ssl                   # Enable mod_ssl
--with-ssl=/path/to/openssl    # OpenSSL location

# Performance features
--enable-nonportable-atomics   # Use CPU-specific atomics
--enable-http2                 # HTTP/2 support

# Debug options
--enable-maintainer-mode       # Enable debugging
--enable-debugger-mode         # Attach debugger
```

### 3.3 Understanding the Build System

**autoconf Structure:**
```
configure.in (configure.ac)
    ├── acinclude.m4        # Local M4 macros
    ├── build/
    │   ├── config.m4       # Module-specific config
    │   └── rules.mk        # Make rules
    └── modules/*/config.m4 # Per-module config
```

**Makefile Structure:**
```makefile
# Main Makefile components
SUBDIRS = server modules support

# Compilation flags
CFLAGS = -g -O2 -pthread -Wall
LDFLAGS = -lpthread -lm

# Module compilation
modules: $(MODULE_DIRS)
	for dir in $(MODULE_DIRS); do \
	    (cd $$dir && $(MAKE)); \
	done

# Server binary
httpd: $(OBJS)
	$(LINK) $(CFLAGS) -o httpd $(OBJS) $(LIBS)
```

---

## 4. Core Architecture {#core-architecture}

### 4.1 Server Lifecycle

**Initialization Sequence:**

```c
/* server/main.c - Main entry point */
int main(int argc, char *argv[])
{
    /* 1. Initialize APR */
    apr_app_initialize(&argc, &argv, NULL);
    apr_pool_create(&pconf, NULL);
    
    /* 2. Parse command line */
    ap_parse_args(argc, argv);
    
    /* 3. Read configuration */
    ap_read_config(pconf, ptemp, confname);
    
    /* 4. Open logs */
    ap_open_logs(pconf, plog, ptemp, server_conf);
    
    /* 5. Post-config hooks */
    ap_run_post_config(pconf, plog, ptemp, server_conf);
    
    /* 6. Pre-MPM hook */
    ap_run_pre_mpm(pconf, plog, ptemp);
    
    /* 7. Launch MPM */
    ap_mpm_run(pconf, plog, server_conf);
    
    /* 8. Cleanup */
    apr_pool_destroy(pconf);
    apr_terminate();
    
    return 0;
}
```

**Server Phases:**

1. **Initialization Phase**
```c
/* APR initialization */
apr_status_t apr_app_initialize(int *argc, char ***argv, char ***env)
{
    // Initialize platform-specific code
    apr_initialize();
    
    // Setup signal handling
    apr_signal_init(APR_SIGNAL_BLOCK);
    
    // Initialize timing
    apr_time_init();
    
    return APR_SUCCESS;
}
```

2. **Configuration Phase**
```c
/* server/config.c - Read configuration */
const char *ap_read_config(apr_pool_t *p, apr_pool_t *ptemp,
                           const char *filename)
{
    cmd_parms *parms;
    ap_directive_t *conftree;
    
    /* Parse configuration file into tree */
    conftree = ap_build_cont_config(p, ptemp, filename);
    
    /* Process directives */
    ap_process_config_tree(server_conf, conftree, p, ptemp);
    
    /* Run open_logs hook */
    ap_run_open_logs(p, plog, ptemp, server_conf);
    
    return NULL;
}
```

3. **Hook Registration Phase**
```c
/* server/config.c - Process config tree */
const char *ap_process_config_tree(server_rec *s,
                                   ap_directive_t *conftree,
                                   apr_pool_t *p, apr_pool_t *ptemp)
{
    /* Walk directive tree */
    for (current = conftree; current; current = current->next) {
        /* Find command handler */
        cmd = ap_find_command(current->directive);
        
        /* Invoke command */
        retval = invoke_cmd(cmd, parms, current->args);
    }
    
    return NULL;
}
```

### 4.2 Multi-Processing Models

**Prefork MPM** - Process-based:
```c
/* server/mpm/prefork/prefork.c */
static int make_child(server_rec *s, int slot)
{
    int pid;
    
    if (one_process) {
        /* Debug mode - no fork */
        signal(SIGHUP, just_die);
        child_main(slot);
        return 0;
    }
    
    /* Fork new child process */
    pid = fork();
    if (pid == -1) {
        ap_log_error(APLOG_ERR, errno, s, "fork: Unable to fork new process");
        return -1;
    }
    
    if (pid == 0) {
        /* Child process */
        child_main(slot);
        return 0;
    }
    
    /* Parent process */
    ap_scoreboard_image->parent[slot].pid = pid;
    return 0;
}

/* Child process main loop */
static void child_main(int child_num_arg)
{
    apr_pool_t *ptrans;
    apr_allocator_t *allocator;
    apr_status_t status;
    conn_rec *current_conn;
    
    /* Initialize */
    apr_allocator_create(&allocator);
    apr_pool_create(&ptrans, pconf);
    
    /* Process requests in loop */
    while (!die_now) {
        /* Accept connection */
        current_conn = ap_run_create_connection(ptrans, ap_server_conf,
                                                csd, my_child_num,
                                                sbh, bucket_alloc);
        
        /* Process connection */
        ap_process_connection(current_conn, csd);
        
        /* Cleanup */
        ap_lingering_close(current_conn);
        apr_pool_clear(ptrans);
    }
    
    apr_pool_destroy(ptrans);
}
```

**Worker MPM** - Thread-based:
```c
/* server/mpm/worker/worker.c */
static void *worker_thread(apr_thread_t *thd, void *dummy)
{
    proc_info *ti = dummy;
    int process_slot = ti->pid;
    int thread_slot = ti->tid;
    apr_pool_t *tpool = apr_thread_pool_get(thd);
    void *csd = NULL;
    apr_bucket_alloc_t *bucket_alloc;
    
    /* Initialize thread-local storage */
    bucket_alloc = apr_bucket_alloc_create(tpool);
    
    /* Main worker loop */
    while (!workers_may_exit) {
        /* Get connection from queue */
        rv = ap_queue_pop(worker_queue, &csd, &ptrans);
        
        if (rv != APR_SUCCESS) {
            continue;
        }
        
        /* Process connection */
        process_socket(ptrans, csd, process_slot, thread_slot, bucket_alloc);
        
        /* Return to accepting */
        apr_pool_clear(ptrans);
    }
    
    ap_update_child_status_from_indexes(process_slot, thread_slot,
                                        SERVER_DEAD, NULL);
    apr_thread_exit(thd, APR_SUCCESS);
    return NULL;
}
```

**Event MPM** - Async I/O:
```c
/* server/mpm/event/event.c */
static void *listener_thread(apr_thread_t *thd, void *dummy)
{
    timer_event_t *te;
    apr_status_t rc;
    proc_info *ti = dummy;
    int process_slot = ti->pid;
    apr_pool_t *tpool = apr_thread_pool_get(thd);
    
    /* Create pollset for async I/O */
    rc = apr_pollset_create(&event_pollset, num_listensocks + num_buckets,
                           tpool, APR_POLLSET_THREADSAFE | APR_POLLSET_NOCOPY);
    
    /* Main event loop */
    while (!listener_may_exit) {
        apr_int32_t num;
        const apr_pollfd_t *out_pfd;
        
        /* Wait for events */
        rc = apr_pollset_poll(event_pollset, timeout, &num, &out_pfd);
        
        if (rc != APR_SUCCESS) {
            if (APR_STATUS_IS_EINTR(rc)) {
                continue;
            }
            if (APR_STATUS_IS_TIMEUP(rc)) {
                process_timeout_queue();
                continue;
            }
        }
        
        /* Process ready connections */
        for (i = 0; i < num; i++) {
            pt = (listener_poll_type *)out_pfd[i].client_data;
            
            if (pt->type == PT_CSD) {
                /* Connection ready - pass to worker */
                push_to_worker_queue(pt);
            }
            else if (pt->type == PT_ACCEPT) {
                /* New connection */
                accept_connection(pt);
            }
        }
    }
    
    return NULL;
}
```

### 4.3 Request Processing Flow

**High-Level Flow:**
```
1. Accept Connection
2. Read Request
3. Parse Request
4. Authentication
5. Authorization  
6. Type Checking
7. Fixups
8. Handler
9. Logging
10. Cleanup
```

**Detailed Implementation:**
```c
/* server/protocol.c - Main request handler */
void ap_process_request(request_rec *r)
{
    int access_status;
    
    /* 1. Authentication phase */
    access_status = ap_run_check_user_id(r);
    if (access_status != OK) {
        ap_die(access_status, r);
        return;
    }
    
    /* 2. Authorization phase */
    access_status = ap_run_auth_checker(r);
    if (access_status != OK) {
        ap_die(access_status, r);
        return;
    }
    
    /* 3. Access control */
    access_status = ap_run_access_checker(r);
    if (access_status != OK) {
        ap_die(access_status, r);
        return;
    }
    
    /* 4. Type checking */
    access_status = ap_run_type_checker(r);
    if (access_status != OK && access_status != DECLINED) {
        ap_die(access_status, r);
        return;
    }
    
    /* 5. Fixups */
    access_status = ap_run_fixups(r);
    if (access_status != OK && access_status != DECLINED) {
        ap_die(access_status, r);
        return;
    }
    
    /* 6. Handler */
    access_status = ap_invoke_handler(r);
    if (access_status == DECLINED) {
        access_status = HTTP_NOT_FOUND;
    }
    
    if (access_status != OK) {
        ap_die(access_status, r);
    }
}
```

---

## 5. Intermediate Concepts {#intermediate-concepts}

### 5.1 Configuration Directives

**Directive Types:**

1. **NO_ARGS** - No arguments
```c
/* Example: EnableMMAP */
AP_INIT_NO_ARGS("EnableMMAP", set_enable_mmap, NULL, RSRC_CONF,
                "Controls whether memory-mapping is used for file delivery")

static const char *set_enable_mmap(cmd_parms *cmd, void *d_, int arg)
{
    core_dir_config *d = d_;
    d->enable_mmap = ENABLE_MMAP_ON;
    return NULL;
}
```

2. **FLAG** - On/Off
```c
/* Example: KeepAlive */
AP_INIT_FLAG("KeepAlive", set_keep_alive, NULL, RSRC_CONF,
             "Whether persistent connections should be enabled or disabled")

static const char *set_keep_alive(cmd_parms *cmd, void *dummy, int arg)
{
    server_rec *s = cmd->server;
    s->keep_alive = arg;
    return NULL;
}
```

3. **TAKE1/2/3** - Fixed number of arguments
```c
/* Example: DocumentRoot (TAKE1) */
AP_INIT_TAKE1("DocumentRoot", set_document_root, NULL, RSRC_CONF,
              "The directory out of which you will serve files")

static const char *set_document_root(cmd_parms *cmd, void *dummy,
                                    const char *arg)
{
    server_rec *s = cmd->server;
    s->path = apr_pstrdup(cmd->pool, arg);
    return NULL;
}

/* Example: ErrorDocument (TAKE2) */
AP_INIT_TAKE2("ErrorDocument", set_error_document, NULL, OR_FILEINFO,
              "HTTP response code and document to be returned for errors")
```

4. **RAW_ARGS** - All remaining arguments as string
```c
/* Example: Define */
AP_INIT_RAW_ARGS("Define", set_define, NULL, RSRC_CONF | ACCESS_CONF,
                 "Define a variable")

static const char *set_define(cmd_parms *cmd, void *config, const char *args)
{
    const char *name, *value;
    const char *err = ap_getword_conf(cmd->pool, &args);
    
    name = err;
    value = args;
    
    apr_table_set(defines, name, value);
    return NULL;
}
```

5. **ITERATE** - Variable arguments, called once per arg
```c
/* Example: AddType */
AP_INIT_ITERATE2("AddType", add_type, NULL, OR_FILEINFO,
                 "MIME type followed by file extensions")

static const char *add_type(cmd_parms *cmd, void *m_,
                           const char *type, const char *ext)
{
    core_dir_config *m = m_;
    apr_table_set(m->forced_types, ext, type);
    return NULL;
}
```

### 5.2 Per-Directory Configuration

**Configuration Context:**
```c
/* server/config.c - Directory walk */
int ap_directory_walk(request_rec *r)
{
    ap_conf_vector_t *per_dir_defaults = r->server->lookup_defaults;
    ap_conf_vector_t **sec_array = r->server->lookup_array;
    int num_sec = r->server->lookup_array_size;
    
    /* Start with server defaults */
    r->per_dir_config = per_dir_defaults;
    
    /* Walk filesystem path */
    for (i = 0; i < num_dirs; i++) {
        /* Find matching <Directory> section */
        for (j = 0; j < num_sec; j++) {
            core_dir_config *core_dir = sec_array[j];
            
            if (matches_path(core_dir->d, path)) {
                /* Merge configuration */
                r->per_dir_config = ap_merge_per_dir_configs(
                    r->pool, r->per_dir_config, sec_array[j]);
            }
        }
        
        /* Check for .htaccess */
        if (allowed_override) {
            ap_parse_htaccess(&htaccess_conf, r, allowed_override,
                            path, access_name);
            r->per_dir_config = ap_merge_per_dir_configs(
                r->pool, r->per_dir_config, htaccess_conf);
        }
    }
    
    return OK;
}
```

**Config Merging:**
```c
/* Example module config merge */
static void *merge_dir_config(apr_pool_t *p, void *basev, void *addv)
{
    my_config *base = (my_config *)basev;
    my_config *add = (my_config *)addv;
    my_config *new = apr_pcalloc(p, sizeof(my_config));
    
    /* Merge logic - 'add' overrides 'base' */
    new->enabled = (add->enabled != DEFAULT) ? add->enabled : base->enabled;
    new->path = add->path ? add->path : base->path;
    
    /* For arrays/tables, combine them */
    new->array = apr_array_append(p, base->array, add->array);
    new->table = apr_table_overlay(p, add->table, base->table);
    
    return new;
}
```

### 5.3 Scoreboard

The scoreboard tracks server and worker status.

**Scoreboard Structure:**
```c
/* include/scoreboard.h */
typedef struct {
    apr_time_t last_used;          // Last used time
    char client_ip[32];            // Client IP
    apr_time_t start_time;         // Request start time
    char request[64];              // Request line
    char vhost[32];                // Virtual host
    apr_off_t bytes_served;        // Bytes sent
    unsigned long access_count;    // Total requests served
    apr_off_t my_bytes_served;     // Bytes in current request
    unsigned long conn_bytes;      // Bytes in connection
    unsigned short conn_count;     // Connection count
    server_rec *the_server;        // Server handling request
} worker_score;

typedef struct {
    int pid;                       // Process ID
    apr_time_t last_used;          // Last activity
    unsigned long generation;      // Generation number
    char quiescing;                // Shutting down flag
    char not_accepting;            // Not accepting new connections
    char conn_count;               // Active connections
    short bucket;                  // Listener bucket
} process_score;

typedef struct {
    int server_limit;              // Max servers
    int thread_limit;              // Max threads per server
    ap_scoreboard_e sb_type;       // Scoreboard type
    apr_time_t restart_time;       // Last restart time
    int lb_limit;                  // Listen bucket limit
} global_score;

/* Main scoreboard structure */
typedef struct {
    global_score *global;
    process_score *parent;
    worker_score **servers;
    apr_pool_t *pool;              // Shared memory pool
} scoreboard;
```

**Scoreboard Usage:**
```c
/* Update worker status */
void ap_update_child_status(ap_sb_handle_t *sbh, int status, request_rec *r)
{
    worker_score *ws = sbh->ws;
    
    ws->status = status;
    ws->last_used = apr_time_now();
    
    if (status == SERVER_BUSY_READ) {
        /* Reading request */
        ws->start_time = apr_time_now();
        apr_cpystrn(ws->client_ip, r->connection->client_ip,
                   sizeof(ws->client_ip));
    }
    else if (status == SERVER_BUSY_WRITE) {
        /* Writing response */
        apr_cpystrn(ws->request, r->the_request, sizeof(ws->request));
        apr_cpystrn(ws->vhost, r->server->server_hostname,
                   sizeof(ws->vhost));
    }
    else if (status == SERVER_READY) {
        /* Ready for new request */
        ws->access_count++;
        ws->my_bytes_served = 0;
    }
}
```

---

## 6. Advanced Topics {#advanced-topics}

### 6.1 Bucket Brigades

Bucket brigades are Apache's I/O abstraction, allowing efficient data streaming.

**Bucket Types:**
```c
/* include/apr_buckets.h */

/* Basic bucket structure */
struct apr_bucket {
    apr_bucket_list link;          // Linked list
    const apr_bucket_type_t *type; // Bucket type
    apr_size_t length;             // Data length
    apr_off_t start;               // Start offset
    void *data;                    // Type-specific data
    apr_bucket_free_t free;        // Free function
    apr_pool_t *pool;              // Memory pool
};

/* Bucket types */
APR_BUCKET_TYPES:
- apr_bucket_file      // File bucket
- apr_bucket_mmap      // Memory-mapped file
- apr_bucket_heap      // Heap-allocated data
- apr_bucket_pool      // Pool-allocated data
- apr_bucket_immortal  // Static data
- apr_bucket_transient // Temporary data
- apr_bucket_eos       // End of stream
- apr_bucket_flush     // Flush marker
- apr_bucket_pipe      // Pipe input
- apr_bucket_socket    // Socket input
```

**Bucket Brigade Operations:**
```c
/* Creating brigade */
apr_bucket_brigade *bb = apr_brigade_create(r->pool, r->connection->bucket_alloc);

/* Adding buckets */

// 1. Add file bucket
apr_file_t *file;
apr_file_open(&file, filename, APR_READ, APR_OS_DEFAULT, r->pool);
apr_bucket *bucket = apr_bucket_file_create(file, 0, (apr_size_t)finfo.size,
                                            r->pool, r->connection->bucket_alloc);
APR_BRIGADE_INSERT_TAIL(bb, bucket);

// 2. Add heap bucket
char *data = apr_pstrdup(r->pool, "Hello World");
bucket = apr_bucket_heap_create(data, strlen(data), NULL, r->connection->bucket_alloc);
APR_BRIGADE_INSERT_TAIL(bb, bucket);

// 3. Add EOS (End of Stream)
bucket = apr_bucket_eos_create(r->connection->bucket_alloc);
APR_BRIGADE_INSERT_TAIL(bb, bucket);

/* Reading from brigade */
apr_status_t rv;
apr_size_t len;
const char *data;
apr_bucket *e;

for (e = APR_BRIGADE_FIRST(bb);
     e != APR_BRIGADE_SENTINEL(bb);
     e = APR_BUCKET_NEXT(e)) {
    
    if (APR_BUCKET_IS_EOS(e)) {
        break;
    }
    
    /* Read bucket data */
    rv = apr_bucket_read(e, &data, &len, APR_BLOCK_READ);
    if (rv != APR_SUCCESS) {
        return rv;
    }
    
    /* Process data */
    process_data(data, len);
}

/* Splitting buckets */
apr_bucket *b2;
apr_bucket_split(bucket, offset);  // Splits at offset

/* Destroying brigade */
apr_brigade_destroy(bb);
```

**Brigade Patterns:**
```c
/* Pattern 1: Flatten brigade into single buffer */
char *flatten_brigade(apr_bucket_brigade *bb, apr_pool_t *pool)
{
    apr_size_t total_len = 0;
    apr_size_t offset = 0;
    char *buffer;
    apr_bucket *e;
    
    /* Calculate total length */
    for (e = APR_BRIGADE_FIRST(bb);
         e != APR_BRIGADE_SENTINEL(bb);
         e = APR_BUCKET_NEXT(e)) {
        total_len += e->length;
    }
    
    /* Allocate buffer */
    buffer = apr_palloc(pool, total_len + 1);
    
    /* Copy data */
    for (e = APR_BRIGADE_FIRST(bb);
         e != APR_BRIGADE_SENTINEL(bb);
         e = APR_BUCKET_NEXT(e)) {
        const char *data;
        apr_size_t len;
        apr_bucket_read(e, &data, &len, APR_BLOCK_READ);
        memcpy(buffer + offset, data, len);
        offset += len;
    }
    buffer[total_len] = '\0';
    
    return buffer;
}

/* Pattern 2: Pass brigade through filter chain */
apr_status_t pass_brigade(ap_filter_t *f, apr_bucket_brigade *bb)
{
    apr_bucket *e;
    
    /* Process each bucket */
    for (e = APR_BRIGADE_FIRST(bb);
         e != APR_BRIGADE_SENTINEL(bb);
         e = APR_BUCKET_NEXT(e)) {
        
        if (APR_BUCKET_IS_METADATA(e)) {
            /* Pass metadata buckets unchanged */
            continue;
        }
        
        /* Transform data buckets */
        transform_bucket(e);
    }
    
    /* Pass to next filter */
    return ap_pass_brigade(f->next, bb);
}
```

### 6.2 Input/Output Filters

Filters process data streams between network and application.

**Filter Structure:**
```c
/* include/util_filter.h */
struct ap_filter_t {
    ap_filter_rec_t *frec;         // Filter registration
    void *ctx;                     // Filter context
    ap_filter_t *next;             // Next filter in chain
    request_rec *r;                // Associated request
    conn_rec *c;                   // Associated connection
};

/* Filter registration */
struct ap_filter_rec_t {
    const char *name;              // Filter name
    ap_filter_func filter_func;    // Filter function
    ap_init_filter_func filter_init_func;  // Init function
    ap_filter_type ftype;          // Filter type
    int debug;                     // Debug flag
};

/* Filter types */
typedef enum {
    AP_FTYPE_RESOURCE,      // Content generation (handler)
    AP_FTYPE_CONTENT_SET,   // Content modification (mod_deflate)
    AP_FTYPE_PROTOCOL,      // Protocol handling (HTTP)
    AP_FTYPE_TRANSCODE,     // Character encoding
    AP_FTYPE_CONNECTION,    // Connection handling (SSL)
    AP_FTYPE_NETWORK        // Network I/O
} ap_filter_type;
```

**Registering Filters:**
```c
/* Register filter in register_hooks */
static void register_hooks(apr_pool_t *p)
{
    /* Output filter */
    ap_register_output_filter("MY_FILTER", my_output_filter,
                             NULL, AP_FTYPE_CONTENT_SET);
    
    /* Input filter */
    ap_register_input_filter("MY_INPUT_FILTER", my_input_filter,
                            NULL, AP_FTYPE_PROTOCOL);
}

/* Insert filter into chain */
static int my_handler(request_rec *r)
{
    /* Add output filter */
    ap_add_output_filter("MY_FILTER", NULL, r, r->connection);
    
    /* Add input filter */
    ap_add_input_filter("MY_INPUT_FILTER", NULL, r, r->connection);
    
    return OK;
}
```

**Output Filter Implementation:**
```c
/* Example: Uppercase filter */
typedef struct {
    apr_bucket_brigade *tmp_bb;
} uppercase_ctx;

static apr_status_t uppercase_output_filter(ap_filter_t *f,
                                           apr_bucket_brigade *bb)
{
    uppercase_ctx *ctx = f->ctx;
    apr_bucket *bucket;
    apr_status_t rv;
    
    /* Initialize context on first call */
    if (!ctx) {
        ctx = apr_pcalloc(f->r->pool, sizeof(*ctx));
        ctx->tmp_bb = apr_brigade_create(f->r->pool,
                                        f->c->bucket_alloc);
        f->ctx = ctx;
    }
    
    /* Process each bucket */
    for (bucket = APR_BRIGADE_FIRST(bb);
         bucket != APR_BRIGADE_SENTINEL(bb);
         bucket = APR_BUCKET_NEXT(bucket)) {
        
        const char *data;
        apr_size_t len;
        char *uppercase_data;
        apr_bucket *new_bucket;
        
        /* Skip metadata buckets */
        if (APR_BUCKET_IS_METADATA(bucket)) {
            APR_BUCKET_REMOVE(bucket);
            APR_BRIGADE_INSERT_TAIL(ctx->tmp_bb, bucket);
            continue;
        }
        
        /* Read bucket data */
        rv = apr_bucket_read(bucket, &data, &len, APR_BLOCK_READ);
        if (rv != APR_SUCCESS) {
            return rv;
        }
        
        /* Convert to uppercase */
        uppercase_data = apr_palloc(f->r->pool, len);
        for (apr_size_t i = 0; i < len; i++) {
            uppercase_data[i] = apr_toupper(data[i]);
        }
        
        /* Create new bucket with uppercase data */
        new_bucket = apr_bucket_pool_create(uppercase_data, len, f->r->pool,
                                           f->c->bucket_alloc);
        APR_BRIGADE_INSERT_TAIL(ctx->tmp_bb, new_bucket);
        
        /* Delete original bucket */
        apr_bucket_delete(bucket);
    }
    
    /* Pass transformed brigade to next filter */
    rv = ap_pass_brigade(f->next, ctx->tmp_bb);
    apr_brigade_cleanup(ctx->tmp_bb);
    
    return rv;
}
```

**Input Filter Implementation:**
```c
/* Example: Request body logging filter */
static apr_status_t log_input_filter(ap_filter_t *f,
                                     apr_bucket_brigade *bb,
                                     ap_input_mode_t mode,
                                     apr_read_type_e block,
                                     apr_off_t readbytes)
{
    apr_status_t rv;
    apr_bucket *bucket;
    
    /* Get data from next filter in chain */
    rv = ap_get_brigade(f->next, bb, mode, block, readbytes);
    if (rv != APR_SUCCESS) {
        return rv;
    }
    
    /* Log bucket data */
    for (bucket = APR_BRIGADE_FIRST(bb);
         bucket != APR_BRIGADE_SENTINEL(bb);
         bucket = APR_BUCKET_NEXT(bucket)) {
        
        const char *data;
        apr_size_t len;
        
        if (APR_BUCKET_IS_METADATA(bucket)) {
            continue;
        }
        
        rv = apr_bucket_read(bucket, &data, &len, APR_BLOCK_READ);
        if (rv == APR_SUCCESS) {
            ap_log_rerror(APLOG_MARK, APLOG_DEBUG, 0, f->r,
                         "Input: %.*s", (int)len, data);
        }
    }
    
    return APR_SUCCESS;
}
```

### 6.3 Shared Memory and IPC

**Creating Shared Memory:**
```c
#include "apr_shm.h"

/* Global shared memory segment */
apr_shm_t *shm = NULL;
typedef struct {
    apr_uint32_t counter;
    apr_time_t last_access;
    char data[256];
} shared_data_t;

/* Initialize in post_config hook */
static int init_shm(apr_pool_t *p, apr_pool_t *plog,
                    apr_pool_t *ptemp, server_rec *s)
{
    apr_status_t rv;
    apr_size_t size = sizeof(shared_data_t);
    const char *shmfile = "/tmp/apache_shm";
    
    /* Remove existing file */
    apr_shm_remove(shmfile, p);
    
    /* Create shared memory */
    rv = apr_shm_create(&shm, size, shmfile, p);
    if (rv != APR_SUCCESS) {
        ap_log_error(APLOG_MARK, APLOG_ERR, rv, s,
                    "Failed to create shared memory");
        return HTTP_INTERNAL_SERVER_ERROR;
    }
    
    /* Initialize data */
    shared_data_t *data = apr_shm_baseaddr_get(shm);
    data->counter = 0;
    data->last_access = apr_time_now();
    memset(data->data, 0, sizeof(data->data));
    
    return OK;
}

/* Access shared memory */
static int my_handler(request_rec *r)
{
    shared_data_t *data = apr_shm_baseaddr_get(shm);
    
    /* Read/write shared data */
    data->counter++;
    data->last_access = apr_time_now();
    
    ap_rprintf(r, "Counter: %u\n", data->counter);
    
    return OK;
}
```

**Global Mutex for Synchronization:**
```c
#include "apr_global_mutex.h"

apr_global_mutex_t *mutex = NULL;

/* Initialize mutex */
static int init_mutex(apr_pool_t *p, apr_pool_t *plog,
                     apr_pool_t *ptemp, server_rec *s)
{
    apr_status_t rv;
    const char *lockfile = "/tmp/apache_mutex";
    
    /* Create global mutex */
    rv = apr_global_mutex_create(&mutex, lockfile,
                                APR_LOCK_DEFAULT, p);
    if (rv != APR_SUCCESS) {
        ap_log_error(APLOG_MARK, APLOG_ERR, rv, s,
                    "Failed to create mutex");
        return HTTP_INTERNAL_SERVER_ERROR;
    }
    
    /* Required for proper cleanup in child processes */
    rv = apr_global_mutex_child_init(&mutex, lockfile, p);
    
    return OK;
}

/* Use mutex to protect critical section */
static int safe_counter_increment(request_rec *r)
{
    apr_status_t rv;
    shared_data_t *data = apr_shm_baseaddr_get(shm);
    
    /* Acquire lock */
    rv = apr_global_mutex_lock(mutex);
    if (rv != APR_SUCCESS) {
        return HTTP_INTERNAL_SERVER_ERROR;
    }
    
    /* Critical section */
    data->counter++;
    apr_cpystrn(data->data, r->uri, sizeof(data->data));
    
    /* Release lock */
    apr_global_mutex_unlock(mutex);
    
    return OK;
}
```

---

## 7. Module System Deep Dive {#module-system}

### 7.1 Module Loading

**Dynamic Module Loading:**
```c
/* server/config.c - LoadModule directive */
static const char *load_module(cmd_parms *cmd, void *dummy,
                              const char *modname, const char *filename)
{
    module *mod;
    apr_dso_handle_t *modhandle;
    apr_status_t status;
    
    /* Load DSO */
    status = apr_dso_load(&modhandle, filename, cmd->pool);
    if (status != APR_SUCCESS) {
        return apr_psprintf(cmd->pool,
                          "Cannot load %s into server: %s",
                          filename, apr_dso_error(modhandle, errbuf, sizeof(errbuf)));
    }
    
    /* Find module structure */
    status = apr_dso_sym((void **)&mod, modhandle, modname);
    if (status != APR_SUCCESS) {
        apr_dso_unload(modhandle);
        return apr_psprintf(cmd->pool,
                          "Can't locate API module structure `%s' in file %s",
                          modname, filename);
    }
    
    /* Add module */
    ap_add_loaded_module(mod, cmd->pool);
    
    return NULL;
}
```

**Module Initialization Sequence:**
```c
/* 1. Pre-config hook - before configuration read */
static int my_pre_config(apr_pool_t *p, apr_pool_t *plog, apr_pool_t *ptemp)
{
    /* Early initialization */
    return OK;
}

/* 2. Check config hook - validate configuration */
static int my_check_config(apr_pool_t *p, apr_pool_t *plog,
                          apr_pool_t *ptemp, server_rec *s)
{
    /* Verify configuration is valid */
    return OK;
}

/* 3. Post config hook - after configuration read */
static int my_post_config(apr_pool_t *p, apr_pool_t *plog,
                         apr_pool_t *ptemp, server_rec *s)
{
    /* Late initialization, register resources */
    void *data;
    const char *userdata_key = "my_module_init";
    
    /* Avoid double initialization */
    apr_pool_userdata_get(&data, userdata_key, s->process->pool);
    if (!data) {
        apr_pool_userdata_set((const void *)1, userdata_key,
                            apr_pool_cleanup_null, s->process->pool);
        return OK;
    }
    
    /* Real initialization code */
    init_shared_memory(p);
    init_mutexes(p);
    
    return OK;
}

/* 4. Child init hook - called in each child process */
static void my_child_init(apr_pool_t *p, server_rec *s)
{
    /* Per-child initialization */
    apr_global_mutex_child_init(&my_mutex, lockfile, p);
}

/* Register hooks */
static void register_hooks(apr_pool_t *p)
{
    ap_hook_pre_config(my_pre_config, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_check_config(my_check_config, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_post_config(my_post_config, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_child_init(my_child_init, NULL, NULL, APR_HOOK_MIDDLE);
}
```

### 7.2 Complete Module Example

**Custom Authentication Module:**
```c
/* mod_custom_auth.c */
#include "httpd.h"
#include "http_config.h"
#include "http_core.h"
#include "http_log.h"
#include "http_protocol.h"
#include "http_request.h"
#include "apr_strings.h"
#include "apr_md5.h"

/* Module configuration */
typedef struct {
    int enabled;
    const char *auth_file;
    const char *realm;
} custom_auth_config;

/* Create per-directory config */
static void *create_auth_dir_config(apr_pool_t *p, char *d)
{
    custom_auth_config *conf = apr_pcalloc(p, sizeof(*conf));
    conf->enabled = 0;
    conf->auth_file = NULL;
    conf->realm = "Protected Area";
    return conf;
}

/* Merge configurations */
static void *merge_auth_dir_config(apr_pool_t *p, void *basev, void *overridesv)
{
    custom_auth_config *base = (custom_auth_config *)basev;
    custom_auth_config *overrides = (custom_auth_config *)overridesv;
    custom_auth_config *conf = apr_pcalloc(p, sizeof(*conf));
    
    conf->enabled = overrides->enabled ? overrides->enabled : base->enabled;
    conf->auth_file = overrides->auth_file ? overrides->auth_file : base->auth_file;
    conf->realm = overrides->realm ? overrides->realm : base->realm;
    
    return conf;
}

/* Check user credentials */
static int check_user_credentials(request_rec *r, const char *user,
                                 const char *password)
{
    custom_auth_config *conf = ap_get_module_config(r->per_dir_config,
                                                    &custom_auth_module);
    apr_file_t *f;
    char line[256];
    apr_status_t status;
    
    /* Open auth file */
    status = apr_file_open(&f, conf->auth_file, APR_READ, APR_OS_DEFAULT, r->pool);
    if (status != APR_SUCCESS) {
        ap_log_rerror(APLOG_MARK, APLOG_ERR, status, r,
                     "Could not open auth file: %s", conf->auth_file);
        return HTTP_INTERNAL_SERVER_ERROR;
    }
    
    /* Read file line by line: format is "username:md5password" */
    while (apr_file_gets(line, sizeof(line), f) == APR_SUCCESS) {
        char *file_user, *file_pass;
        apr_md5_ctx_t context;
        unsigned char digest[APR_MD5_DIGESTSIZE];
        char hash[33];
        
        /* Parse line */
        file_user = apr_strtok(line, ":", &file_pass);
        if (!file_user || !file_pass) {
            continue;
        }
        
        /* Remove newline from password */
        file_pass[strcspn(file_pass, "\r\n")] = 0;
        
        /* Check username */
        if (strcmp(user, file_user) != 0) {
            continue;
        }
        
        /* Calculate MD5 of provided password */
        apr_md5_init(&context);
        apr_md5_update(&context, (unsigned char *)password, strlen(password));
        apr_md5_final(digest, &context);
        
        /* Convert to hex string */
        for (int i = 0; i < APR_MD5_DIGESTSIZE; i++) {
            sprintf(&hash[i*2], "%02x", digest[i]);
        }
        hash[32] = 0;
        
        /* Compare passwords */
        if (strcmp(hash, file_pass) == 0) {
            apr_file_close(f);
            return OK;
        }
        
        /* User found but password wrong */
        apr_file_close(f);
        return HTTP_UNAUTHORIZED;
    }
    
    apr_file_close(f);
    return HTTP_UNAUTHORIZED;  /* User not found */
}

/* Authentication handler */
static int authenticate_user(request_rec *r)
{
    custom_auth_config *conf = ap_get_module_config(r->per_dir_config,
                                                    &custom_auth_module);
    const char *auth_header;
    char *user, *password;
    int res;
    
    /* Check if enabled */
    if (!conf->enabled) {
        return DECLINED;
    }
    
    /* Get Authorization header */
    auth_header = apr_table_get(r->headers_in, "Authorization");
    if (!auth_header) {
        /* No credentials provided - send auth challenge */
        apr_table_setn(r->err_headers_out, "WWW-Authenticate",
                      apr_psprintf(r->pool, "Basic realm=\"%s\"", conf->realm));
        return HTTP_UNAUTHORIZED;
    }
    
    /* Parse Basic auth header */
    if (strncmp(auth_header, "Basic ", 6) != 0) {
        return HTTP_UNAUTHORIZED;
    }
    
    /* Decode base64 credentials */
    char *decoded = apr_palloc(r->pool, apr_base64_decode_len(auth_header + 6));
    apr_base64_decode(decoded, auth_header + 6);
    
    /* Split user:password */
    user = apr_strtok(decoded, ":", &password);
    if (!user || !password) {
        return HTTP_UNAUTHORIZED;
    }
    
    /* Verify credentials */
    res = check_user_credentials(r, user, password);
    if (res != OK) {
        apr_table_setn(r->err_headers_out, "WWW-Authenticate",
                      apr_psprintf(r->pool, "Basic realm=\"%s\"", conf->realm));
        return res;
    }
    
    /* Set authenticated user */
    r->user = apr_pstrdup(r->pool, user);
    r->ap_auth_type = "Basic";
    
    return OK;
}

/* Configuration directives */
static const command_rec auth_directives[] = {
    AP_INIT_FLAG("CustomAuthEnable", ap_set_flag_slot,
                 (void *)APR_OFFSETOF(custom_auth_config, enabled),
                 OR_AUTHCFG, "Enable custom authentication"),
    AP_INIT_TAKE1("CustomAuthFile", ap_set_file_slot,
                  (void *)APR_OFFSETOF(custom_auth_config, auth_file),
                  OR_AUTHCFG, "Path to authentication file"),
    AP_INIT_TAKE1("CustomAuthRealm", ap_set_string_slot,
                  (void *)APR_OFFSETOF(custom_auth_config, realm),
                  OR_AUTHCFG, "Authentication realm"),
    {NULL}
};

/* Register hooks */
static void register_hooks(apr_pool_t *p)
{
    ap_hook_check_user_id(authenticate_user, NULL, NULL, APR_HOOK_MIDDLE);
}

/* Module declaration */
module AP_MODULE_DECLARE_DATA custom_auth_module = {
    STANDARD20_MODULE_STUFF,
    create_auth_dir_config,    /* Per-directory config creator */
    merge_auth_dir_config,     /* Per-directory merger */
    NULL,                      /* Per-server config creator */
    NULL,                      /* Per-server merger */
    auth_directives,           /* Configuration directives */
    register_hooks             /* Register hooks */
};
```

**Build and Install:**
```bash
# Compile module
apxs -c mod_custom_auth.c

# Install module
apxs -i -a mod_custom_auth.la

# Configuration in httpd.conf
LoadModule custom_auth_module modules/mod_custom_auth.so

<Directory "/var/www/protected">
    CustomAuthEnable On
    CustomAuthFile /etc/apache2/users.auth
    CustomAuthRealm "Secret Area"
    Require valid-user
</Directory>
```

---

## 8. Request Processing Pipeline {#request-processing}

### 8.1 Connection Processing

**Accept Loop:**
```c
/* server/mpm/prefork/prefork.c - Simplified */
static void child_main(int child_num_arg)
{
    apr_pool_t *ptrans;
    apr_socket_t *csd = NULL;
    ap_listen_rec *lr;
    
    /* Create transaction pool */
    apr_pool_create(&ptrans, pconf);
    
    while (!die_now) {
        /* Wait for connection */
        lr = find_ready_listener();
        if (!lr) {
            continue;
        }
        
        /* Accept connection */
        apr_status_t status = apr_socket_accept(&csd, lr->sd, ptrans);
        if (status != APR_SUCCESS) {
            if (APR_STATUS_IS_EINTR(status)) {
                continue;
            }
            /* Log error and continue */
            continue;
        }
        
        /* Create connection record */
        conn_rec *c = ap_run_create_connection(ptrans, ap_server_conf,
                                              csd, child_num_arg,
                                              sbh, bucket_alloc);
        if (!c) {
            apr_socket_close(csd);
            continue;
        }
        
        /* Process connection */
        ap_process_connection(c, csd);
        
        /* Cleanup */
        ap_lingering_close(c);
        apr_pool_clear(ptrans);
    }
}
```

**Connection Processing:**
```c
/* server/connection.c */
void ap_process_connection(conn_rec *c, apr_socket_t *csd)
{
    int rc;
    
    /* Set socket to connection */
    ap_set_core_module_config(c->conn_config, csd);
    
    /* Create connection bucket allocator */
    c->bucket_alloc = apr_bucket_alloc_create(c->pool);
    
    /* Run pre_connection hooks */
    rc = ap_run_pre_connection(c, csd);
    if (rc != OK && rc != DONE) {
        c->aborted = 1;
    }
    
    if (!c->aborted) {
        /* Process HTTP requests on this connection */
        ap_run_process_connection(c);
    }
}

/* HTTP protocol processing */
int ap_process_http_connection(conn_rec *c)
{
    request_rec *r;
    int csd_set = 0;
    
    /* Create core connection config */
    if (!ap_get_core_module_config(c->conn_config)) {
        csd_set = 1;
    }
    
    /* Keep-alive loop */
    while (!c->aborted) {
        /* Create request record */
        r = ap_read_request(c);
        
        if (!r) {
            /* Connection closed or error */
            break;
        }
        
        /* Process request */
        ap_process_request(r);
        
        /* Update keep-alive status */
        if (!r->connection->keepalive || c->aborted) {
            break;
        }
        
        /* Destroy request pool */
        apr_pool_destroy(r->pool);
    }
    
    return OK;
}
```

### 8.2 Request Parsing

**Read Request:**
```c
/* server/protocol.c - Read and parse request */
request_rec *ap_read_request(conn_rec *c)
{
    request_rec *r;
    apr_pool_t *p;
    const char *expect;
    int access_status;
    
    /* Create request pool */
    apr_pool_create(&p, c->pool);
    apr_pool_tag(p, "request");
    
    /* Allocate request record */
    r = apr_pcalloc(p, sizeof(request_rec));
    r->pool = p;
    r->connection = c;
    r->server = c->base_server;
    
    /* Initialize request */
    r->headers_in = apr_table_make(r->pool, 25);
    r->subprocess_env = apr_table_make(r->pool, 25);
    r->headers_out = apr_table_make(r->pool, 12);
    r->err_headers_out = apr_table_make(r->pool, 5);
    r->notes = apr_table_make(r->pool, 5);
    r->request_time = apr_time_now();
    
    /* Read request line */
    access_status = read_request_line(r);
    if (access_status != OK) {
        if (access_status == HTTP_REQUEST_TIME_OUT) {
            ap_log_rerror(APLOG_MARK, APLOG_INFO, 0, r,
                         "Request timeout");
        }
        goto error;
    }
    
    /* Parse URI */
    if (r->uri[0] != '/') {
        /* Absolute URI or authority form */
        if (r->method_number == M_CONNECT) {
            /* CONNECT method - authority form */
        }
        else {
            /* Absolute URI */
            apr_uri_t uri;
            apr_uri_parse(r->pool, r->uri, &uri);
            r->hostname = uri.hostname;
            r->uri = uri.path;
        }
    }
    
    /* Read headers */
    access_status = ap_read_request_headers(r);
    if (access_status != OK) {
        goto error;
    }
    
    /* Get Host header for virtual hosting */
    if (!r->hostname) {
        const char *host = apr_table_get(r->headers_in, "Host");
        if (host) {
            r->hostname = host;
        }
    }
    
    /* Virtual host processing */
    ap_update_vhost_from_headers(r);
    
    /* Protocol-specific processing */
    if (r->proto_num == HTTP_VERSION(1,1)) {
        /* HTTP/1.1 requires Host header */
        if (!r->hostname) {
            access_status = HTTP_BAD_REQUEST;
            goto error;
        }
    }
    
    /* Check for Expect: 100-continue */
    expect = apr_table_get(r->headers_in, "Expect");
    if (expect && ap_cstr_casecmp(expect, "100-continue") == 0) {
        r->expecting_100 = 1;
    }
    
    return r;
    
error:
    r->status = access_status;
    ap_send_error_response(r, 0);
    apr_pool_destroy(r->pool);
    return NULL;
}
```

**Parse Request Line:**
```c
/* Read request line: METHOD URI PROTOCOL */
static int read_request_line(request_rec *r)
{
    char *request_line;
    char *method, *uri, *protocol;
    apr_size_t len;
    int major, minor;
    
    /* Read line from connection */
    request_line = apr_palloc(r->pool, DEFAULT_LIMIT_REQUEST_LINE);
    len = DEFAULT_LIMIT_REQUEST_LINE;
    
    if (ap_rgetline(&request_line, len, &len, r,
                    0, r->input_filters) != OK) {
        return HTTP_REQUEST_TIME_OUT;
    }
    
    /* Save full request line */
    r->the_request = request_line;
    
    /* Parse method */
    method = ap_getword_white(r->pool, &request_line);
    if (!method || !*method) {
        return HTTP_BAD_REQUEST;
    }
    r->method = method;
    
    /* Convert method to number */
    r->method_number = ap_method_number_of(method);
    
    /* Parse URI */
    uri = ap_getword_white(r->pool, &request_line);
    if (!uri || !*uri) {
        return HTTP_BAD_REQUEST;
    }
    r->uri = uri;
    
    /* Parse protocol (optional for HTTP/0.9) */
    protocol = ap_getword_white(r->pool, &request_line);
    if (!protocol || !*protocol) {
        /* HTTP/0.9 simple request */
        r->assbackwards = 1;
        r->protocol = "HTTP/0.9";
        r->proto_num = HTTP_VERSION(0, 9);
        return OK;
    }
    
    /* Validate protocol */
    if (strncmp(protocol, "HTTP/", 5) != 0) {
        return HTTP_BAD_REQUEST;
    }
    
    r->protocol = protocol;
    
    /* Parse version number */
    if (sscanf(protocol + 5, "%d.%d", &major, &minor) != 2) {
        return HTTP_BAD_REQUEST;
    }
    
    r->proto_num = HTTP_VERSION(major, minor);
    
    return OK;
}
```

**Read Headers:**
```c
/* server/protocol.c - Read HTTP headers */
int ap_read_request_headers(request_rec *r)
{
    char *field, *value;
    apr_size_t len;
    int fields_read = 0;
    char *last_field = NULL;
    
    while (1) {
        field = apr_palloc(r->pool, DEFAULT_LIMIT_REQUEST_FIELDSIZE);
        len = DEFAULT_LIMIT_REQUEST_FIELDSIZE;
        
        /* Read header line */
        if (ap_rgetline(&field, len, &len, r,
                       0, r->input_filters) != OK) {
            return HTTP_REQUEST_TIME_OUT;
        }
        
        /* Empty line marks end of headers */
        if (field[0] == '\0') {
            break;
        }
        
        /* Check field count limit */
        if (++fields_read > r->server->limit_req_fields) {
            ap_log_rerror(APLOG_MARK, APLOG_INFO, 0, r,
                         "Number of request headers exceeds limit");
            return HTTP_REQUEST_HEADER_FIELDS_TOO_LARGE;
        }
        
        /* Handle continuation lines (obsolete in HTTP/1.1) */
        if (field[0] == ' ' || field[0] == '\t') {
            if (!last_field) {
                return HTTP_BAD_REQUEST;
            }
            /* Append to previous field */
            while (*field == ' ' || *field == '\t') {
                field++;
            }
            value = apr_pstrcat(r->pool,
                              apr_table_get(r->headers_in, last_field),
                              " ", field, NULL);
            apr_table_set(r->headers_in, last_field, value);
            continue;
        }
        
        /* Parse field-name: field-value */
        value = strchr(field, ':');
        if (!value) {
            return HTTP_BAD_REQUEST;
        }
        
        *value = '\0';
        value++;
        
        /* Skip leading whitespace in value */
        while (*value == ' ' || *value == '\t') {
            value++;
        }
        
        /* Remove trailing whitespace */
        char *end = value + strlen(value) - 1;
        while (end > value && (*end == ' ' || *end == '\t')) {
            *end = '\0';
            end--;
        }
        
        /* Add to headers table */
        apr_table_addn(r->headers_in, field, value);
        last_field = field;
    }
    
    return OK;
}
```

### 8.3 Request Hook Execution

**Hook Execution Order:**
```c
/* server/request.c - Process request through hooks */
void ap_process_request_internal(request_rec *r)
{
    int access_status;
    
    /* 1. Post-read request hook */
    access_status = ap_run_post_read_request(r);
    if (access_status != OK && access_status != DECLINED) {
        goto error;
    }
    
    /* 2. Translate name (URI to filename) */
    access_status = ap_run_translate_name(r);
    if (access_status != OK) {
        goto error;
    }
    
    /* 3. Map to storage */
    access_status = ap_run_map_to_storage(r);
    if (access_status != OK && access_status != DECLINED) {
        goto error;
    }
    
    /* 4. Directory walk - build per-dir config */
    access_status = ap_directory_walk(r);
    if (access_status != OK) {
        goto error;
    }
    
    /* 5. File walk - process <Files> sections */
    access_status = ap_file_walk(r);
    if (access_status != OK) {
        goto error;
    }
    
    /* 6. Header parser hook */
    access_status = ap_run_header_parser(r);
    if (access_status != OK && access_status != DECLINED) {
        goto error;
    }
    
    /* 7. Access control hook */
    access_status = ap_run_access_checker(r);
    if (access_status != OK) {
        goto error;
    }
    
    /* 8. Check user ID (authentication) */
    access_status = ap_run_check_user_id(r);
    if (access_status != OK && access_status != DECLINED) {
        goto error;
    }
    
    /* 9. Auth checker (authorization) */
    access_status = ap_run_auth_checker(r);
    if (access_status != OK && access_status != DECLINED) {
        goto error;
    }
    
    /* 10. Type checker - determine Content-Type */
    access_status = ap_run_type_checker(r);
    if (access_status != OK && access_status != DECLINED) {
        goto error;
    }
    
    /* 11. Fixups - last chance before handler */
    access_status = ap_run_fixups(r);
    if (access_status != OK && access_status != DECLINED) {
        goto error;
    }
    
    /* 12. Insert filters */
    ap_run_insert_filter(r);
    
    /* 13. Handler - generate response */
    access_status = ap_invoke_handler(r);
    if (access_status != OK) {
        goto error;
    }
    
    /* 14. Log transaction */
    ap_run_log_transaction(r);
    
    return;
    
error:
    r->status = access_status;
    ap_send_error_response(r, 0);
    ap_run_log_transaction(r);
}
```

---

## 9. Memory Management {#memory-management}

### 9.1 Pool Architecture

**Pool Hierarchy:**
```
Root Pool (pconf)
├── Process Pool
├── Server Pools (per virtual host)
├── Connection Pools
│   └── Request Pools
│       └── Subrequest Pools
└── Temporary Pools
```

**Pool Implementation Details:**
```c
/* apr/memory/unix/apr_pools.c - Simplified */

/* Pool structure */
struct apr_pool_t {
    apr_pool_t *parent;            /* Parent pool */
    apr_pool_t *child;             /* First child pool */
    apr_pool_t *sibling;           /* Next sibling pool */
    apr_pool_t **ref;              /* Reference to update on clear */
    
    cleanup_t *cleanups;           /* Cleanup functions */
    free_proc_t *free_proc;        /* Free callback */
    
    apr_allocator_t *allocator;    /* Memory allocator */
    
    apr_memnode_t *active;         /* Active memory node */
    apr_memnode_t *self;           /* Pool's own node */
    
    char *self_first_avail;        /* First available byte */
};

/* Memory node */
struct apr_memnode_t {
    apr_memnode_t *next;           /* Next node in freelist */
    apr_memnode_t **ref;           /* Reference for removal */
    apr_uint32_t index;            /* Size index */
    apr_uint32_t free_index;       /* Index for freeing */
    char *first_avail;             /* First available byte */
    char *endp;                    /* End pointer */
};
```

**Pool Allocation:**
```c
/* Allocate memory from pool */
void *apr_palloc(apr_pool_t *pool, apr_size_t size)
{
    apr_memnode_t *active, *node;
    void *mem;
    apr_size_t free_space;
    
    /* Align size to minimum alignment */
    size = APR_ALIGN_DEFAULT(size);
    
    active = pool->active;
    
    /* Try to allocate from active node */
    free_space = active->endp - active->first_avail;
    if (free_space >= size) {
        mem = active->first_avail;
        active->first_avail += size;
        return mem;
    }
    
    /* Active node too small - get new node */
    node = allocator_alloc(pool->allocator, size);
    if (!node) {
        return NULL;
    }
    
    /* Link new node as active */
    node->next = active;
    pool->active = node;
    
    /* Allocate from new node */
    mem = node->first_avail;
    node->first_avail += size;
    
    return mem;
}

/* Zero-initialized allocation */
void *apr_pcalloc(apr_pool_t *pool, apr_size_t size)
{
    void *mem = apr_palloc(pool, size);
    if (mem) {
        memset(mem, 0, size);
    }
    return mem;
}
```

**Pool Cleanup:**
```c
/* Register cleanup function */
void apr_pool_cleanup_register(apr_pool_t *p, void *data,
                              apr_status_t (*plain_cleanup)(void *),
                              apr_status_t (*child_cleanup)(void *))
{
    cleanup_t *c;
    
    c = apr_palloc(p, sizeof(cleanup_t));
    c->data = data;
    c->plain_cleanup_fn = plain_cleanup;
    c->child_cleanup_fn = child_cleanup;
    c->next = p->cleanups;
    p->cleanups = c;
}

/* Clear pool - run cleanups and free memory */
void apr_pool_clear(apr_pool_t *pool)
{
    cleanup_t *c;
    
    /* Destroy all child pools first */
    while (pool->child) {
        apr_pool_destroy(pool->child);
    }
    
    /* Run cleanup functions */
    for (c = pool->cleanups; c; c = c->next) {
        if (c->plain_cleanup_fn) {
            (*c->plain_cleanup_fn)(c->data);
        }
    }
    pool->cleanups = NULL;
    
    /* Return memory nodes to allocator */
    free_all_nodes(pool);
    
    /* Reset pool */
    pool->active = pool->self;
    pool->self_first_avail = pool->self->first_avail;
}

/* Destroy pool completely */
void apr_pool_destroy(apr_pool_t *pool)
{
    apr_pool_clear(pool);
    
    /* Remove from parent's child list */
    if (pool->parent) {
        if (pool->parent->child == pool) {
            pool->parent->child = pool->sibling;
        }
        else {
            apr_pool_t *p;
            for (p = pool->parent->child; p; p = p->sibling) {
                if (p->sibling == pool) {
                    p->sibling = pool->sibling;
                    break;
                }
            }
        }
    }
    
    /* Free pool's own memory */
    if (pool->allocator) {
        free_node(pool->allocator, pool->self);
    }
}
```

### 9.2 Memory Pool Best Practices

**Pattern 1: Temporary Computations**
```c
static int my_handler(request_rec *r)
{
    apr_pool_t *tmp_pool;
    char *large_buffer;
    
    /* Create temporary pool for scratch work */
    apr_pool_create(&tmp_pool, r->pool);
    
    /* Do work that needs lots of temporary memory */
    large_buffer = apr_palloc(tmp_pool, 1024 * 1024);
    process_data(large_buffer);
    
    /* Free all temporary memory at once */
    apr_pool_destroy(tmp_pool);
    
    return OK;
}
```

**Pattern 2: Resource Cleanup**
```c
/* Cleanup function */
static apr_status_t cleanup_file(void *data)
{
    apr_file_t *file = (apr_file_t *)data;
    return apr_file_close(file);
}

static int open_and_register(request_rec *r)
{
    apr_file_t *file;
    apr_status_t rv;
    
    /* Open file */
    rv = apr_file_open(&file, "/path/to/file",
                      APR_READ, APR_OS_DEFAULT, r->pool);
    if (rv != APR_SUCCESS) {
        return HTTP_INTERNAL_SERVER_ERROR;
    }
    
    /* Register cleanup - file will be closed when pool destroyed */
    apr_pool_cleanup_register(r->pool, file, cleanup_file,
                            apr_pool_cleanup_null);
    
    return OK;
}
```

**Pattern 3: Long-Lived Data**
```c
/* Global data that persists across requests */
static apr_hash_t *global_cache = NULL;

static int post_config_hook(apr_pool_t *p, apr_pool_t *plog,
                           apr_pool_t *ptemp, server_rec *s)
{
    /* Allocate from process pool - lives until server restart */
    global_cache = apr_hash_make(p);
    
    /* Load persistent data */
    load_cache_from_disk(global_cache, p);
    
    return OK;
}
```

---

## 10. Multi-Processing Modules (MPM) {#mpm}

### 10.1 MPM Comparison

| Feature | Prefork | Worker | Event |
|---------|---------|--------|-------|
| Model | Process per request | Threads + Processes | Async I/O |
| Concurrency | Low | Medium | High |
| Memory | High | Medium | Low |
| Thread-safe | Not required | Required | Required |
| Keep-alive efficiency | Poor | Good | Excellent |
| Best for | Non-thread-safe modules | Mixed workload | High concurrency |

### 10.2 Prefork MPM Deep Dive

**Process Management:**
```c
/* server/mpm/prefork/prefork.c */

/* Configuration */
typedef struct {
    int max_clients;           /* MaxRequestWorkers */
    int start_servers;         /* StartServers */
    int min_spare_servers;     /* MinSpareServers */
    int max_spare_servers;     /* MaxSpareServers */
    int max_requests_per_child; /* MaxConnectionsPerChild */
} prefork_config;

/* Server pool state */
static int num_children = 0;
static int idle_spawn_rate = 1;
static int hold_off_on_exponential_spawning;

/* Main prefork control loop */
int ap_mpm_run(apr_pool_t *pconf, apr_pool_t *plog, server_rec *s)
{
    int index;
    int remaining_children_to_start;
    
    /* Initialize */
    ap_scoreboard_image = init_scoreboard(pconf);
    
    /* Create initial children */
    remaining_children_to_start = ap_daemons_to_start;
    while (remaining_children_to_start > 0) {
        if (make_child(s, idle_spawn_rate) < 0) {
            break;
        }
        remaining_children_to_start--;
    }
    
    /* Main loop - monitor and adjust process pool */
    while (!restart_pending && !shutdown_pending) {
        apr_sleep(apr_time_from_sec(1));
        
        /* Reap dead children */
        for (index = 0; index < ap_daemons_limit; index++) {
            if (ap_scoreboard_image->servers[index][0].status == SERVER_DEAD) {
                continue;
            }
            
            /* Check if process died */
            pid_t pid = ap_scoreboard_image->parent[index].pid;
            if (pid == 0) {
                continue;
            }
            
            pid_t waitret = waitpid(pid, &status, WNOHANG);
            if (waitret == pid) {
                /* Child died - mark slot as free */
                ap_scoreboard_image->parent[index].pid = 0;
                num_children--;
            }
        }
        
        /* Count idle servers */
        int idle_count = 0;
        int active_count = 0;
        
        for (index = 0; index < ap_daemons_limit; index++) {
            worker_score *ws = &ap_scoreboard_image->servers[index][0];
            
            if (ws->status == SERVER_READY) {
                idle_count++;
            }
            else if (ws->status != SERVER_DEAD) {
                active_count++;
            }
        }
        
        /* Spawn children if needed */
        if (idle_count < ap_daemons_min_free) {
            /* Too few idle - spawn more */
            int to_spawn = ap_daemons_min_free - idle_count;
            
            if (!hold_off_on_exponential_spawning) {
                idle_spawn_rate *= 2;
                if (idle_spawn_rate > MAX_SPAWN_RATE) {
                    idle_spawn_rate = MAX_SPAWN_RATE;
                }
            }
            
            for (index = 0; index < to_spawn && num_children < ap_daemons_limit; index++) {
                make_child(s, idle_spawn_rate);
            }
        }
        else if (idle_count > ap_daemons_max_free) {
            /* Too many idle - kill some */
            idle_spawn_rate = 1;
            
            for (index = 0; index < ap_daemons_limit && idle_count > ap_daemons_max_free; index++) {
                worker_score *ws = &ap_scoreboard_image->servers[index][0];
                
                if (ws->status == SERVER_READY) {
                    /* Send graceful shutdown signal */
                    kill(ap_scoreboard_image->parent[index].pid, SIGUSR1);
                    idle_count--;
                }
            }
        }
    }
    
    /* Shutdown - kill all children */
    for (index = 0; index < ap_daemons_limit; index++) {
        pid_t pid = ap_scoreboard_image->parent[index].pid;
        if (pid > 0) {
            kill(pid, SIGTERM);
        }
    }
    
    return OK;
}
```

### 10.3 Worker MPM Deep Dive

**Thread Pool Management:**
```c
/* server/mpm/worker/worker.c */

/* Configuration */
typedef struct {
    int max_clients;               /* MaxRequestWorkers */
    int threads_per_child;         /* ThreadsPerChild */
    int max_requests_per_child;    /* MaxConnectionsPerChild */
    int server_limit;              /* ServerLimit */
    int thread_limit;              /* ThreadLimit */
} worker_config;

/* Per-process info */
typedef struct {
    pid_t pid;
    int tid;
} proc_info;

/* Thread start function */
static void *worker_thread(apr_thread_t *thd, void *dummy)
{
    proc_info *ti = dummy;
    int process_slot = ti->pid;
    int thread_slot = ti->tid;
    apr_pool_t *ptrans;
    apr_allocator_t *allocator;
    apr_bucket_alloc_t *bucket_alloc;
    conn_rec *current_conn;
    apr_socket_t *csd = NULL;
    
    /* Create thread pool */
    apr_allocator_create(&allocator);
    apr_pool_create(&ptrans, NULL);
    apr_pool_allocator_set(ptrans, allocator);
    apr_allocator_owner_set(allocator, ptrans);
    
    bucket_alloc = apr_bucket_alloc_create(ptrans);
    
    /* Mark thread as ready */
    ap_update_child_status_from_indexes(process_slot, thread_slot,
                                       SERVER_READY, NULL);
    
    /* Main worker loop */
    while (!workers_may_exit) {
        /* Get connection from queue */
        apr_status_t rv = ap_queue_pop(worker_queue, &csd, &ptrans);
        
        if (rv != APR_SUCCESS) {
            if (APR_STATUS_IS_EINTR(rv)) {
                continue;
            }
            break;
        }
        
        /* Mark as busy */
        ap_update_child_status_from_indexes(process_slot, thread_slot,
                                           SERVER_BUSY_READ, NULL);
        
        /* Create connection */
        current_conn = ap_run_create_connection(ptrans, ap_server_conf,
                                               csd, thread_slot,
                                               sbh, bucket_alloc);
        if (current_conn) {
            /* Process connection */
            ap_process_connection(current_conn, csd);
            ap_lingering_close(current_conn);
        }
        
        /* Clean up for next request */
        apr_pool_clear(ptrans);
        
        /* Back to ready state */
        ap_update_child_status_from_indexes(process_slot, thread_slot,
                                           SERVER_READY, NULL);
    }
    
    /* Thread exiting */
    ap_update_child_status_from_indexes(process_slot, thread_slot,
                                       SERVER_DEAD, NULL);
    
    apr_bucket_alloc_destroy(bucket_alloc);
    apr_pool_destroy(ptrans);
    
    return NULL;
}

/* Process initialization */
static void child_main(int child_num_arg)
{
    apr_thread_t **threads;
    apr_status_t rv;
    int i;
    
    /* Create thread array */
    threads = apr_pcalloc(pchild, sizeof(apr_thread_t *) * ap_threads_per_child);
    
    /* Start worker threads */
    for (i = 0; i < ap_threads_per_child; i++) {
        proc_info *my_info = apr_pcalloc(pchild, sizeof(proc_info));
        my_info->pid = child_num_arg;
        my_info->tid = i;
        
        rv = apr_thread_create(&threads[i], thread_attr,
                              worker_thread, my_info, pchild);
        if (rv != APR_SUCCESS) {
            ap_log_error(APLOG_MARK, APLOG_ALERT, rv, ap_server_conf,
                        "apr_thread_create: unable to create worker thread");
            exit(APEXIT_CHILDFATAL);
        }
    }
    
    /* Listener thread */
    rv = apr_thread_create(&threads[i], thread_attr,
                          listener_thread, NULL, pchild);
    
    /* Wait for threads to exit */
    for (i = 0; i < ap_threads_per_child; i++) {
        apr_thread_join(&rv, threads[i]);
    }
}
```

### 10.4 Event MPM Deep Dive

**Async I/O Event Loop:**
```c
/* server/mpm/event/event.c */

/* Connection state */
typedef enum {
    CONN_STATE_READ_REQUEST_LINE,
    CONN_STATE_CHECK_REQUEST_LINE_READABLE,
    CONN_STATE_WRITE_COMPLETION,
    CONN_STATE_KEEPALIVE,
    CONN_STATE_LINGER,
    CONN_STATE_SUSPENDED
} conn_state_e;

/* Event listener thread */
static void *listener_thread(apr_thread_t *thd, void *dummy)
{
    timer_event_t *te;
    apr_status_t rc;
    apr_pollfd_t *out_pfd;
    apr_int32_t num = 0;
    listener_poll_type *pt;
    
    /* Create pollset for event monitoring */
    rc = apr_pollset_create(&event_pollset,
                           num_listensocks + num_buckets,
                           tpool,
                           APR_POLLSET_THREADSAFE | APR_POLLSET_NOCOPY);
    
    /* Add listening sockets to pollset */
    for (lr = ap_listeners; lr != NULL; lr = lr->next) {
        apr_pollfd_t pfd = { 0 };
        
        pt = apr_pcalloc(tpool, sizeof(*pt));
        pt->type = PT_ACCEPT;
        pt->baton = lr;
        
        pfd.desc_type = APR_POLL_SOCKET;
        pfd.desc.s = lr->sd;
        pfd.reqevents = APR_POLLIN;
        pfd.client_data = pt;
        
        apr_pollset_add(event_pollset, &pfd);
    }
    
    /* Main event loop */
    while (!listener_may_exit) {
        apr_interval_time_t timeout;
        
        /* Calculate timeout based on pending timers */
        timeout = calculate_timeout();
        
        /* Poll for events */
        rc = apr_pollset_poll(event_pollset, timeout, &num, &out_pfd);
        
        if (rc != APR_SUCCESS) {
            if (APR_STATUS_IS_EINTR(rc)) {
                continue;
            }
            if (APR_STATUS_IS_TIMEUP(rc)) {
                /* Process timeout queue */
                process_timeout_queue();
                continue;
            }
        }
        
        /* Process ready descriptors */
        for (i = 0; i < num; i++) {
            pt = (listener_poll_type *)out_pfd[i].client_data;
            
            if (pt->type == PT_ACCEPT) {
                /* New connection ready to accept */
                ap_listen_rec *lr = (ap_listen_rec *)pt->baton;
                
                /* Accept in loop until would block */
                while (1) {
                    apr_socket_t *csd;
                    apr_pool_t *ptrans;
                    
                    apr_pool_create(&ptrans, pconf);
                    
                    rc = apr_socket_accept(&csd, lr->sd, ptrans);
                    if (rc != APR_SUCCESS) {
                        if (!APR_STATUS_IS_EAGAIN(rc)) {
                            ap_log_error(APLOG_MARK, APLOG_ERR, rc,
                                       ap_server_conf,
                                       "accept() failed");
                        }
                        apr_pool_destroy(ptrans);
                        break;
                    }
                    
                    /* Push connection to worker queue */
                    push_to_worker_queue(csd, ptrans);
                }
            }
            else if (pt->type == PT_CSD) {
                /* Existing connection has activity */
                process_socket(pt->baton, out_pfd[i].rtnevents);
            }
        }
    }
    
    return NULL;
}

/* Process socket event */
static void process_socket(void *baton, apr_int16_t events)
{
    conn_rec *c = (conn_rec *)baton;
    conn_state_t *cs = (conn_state_t *)c->cs;
    
    if (events & APR_POLLIN) {
        /* Data available to read */
        switch (cs->state) {
            case CONN_STATE_CHECK_REQUEST_LINE_READABLE:
                cs->state = CONN_STATE_READ_REQUEST_LINE;
                push_to_worker_queue(c);
                break;
                
            case CONN_STATE_KEEPALIVE:
                /* Keep-alive connection has new request */
                cs->state = CONN_STATE_READ_REQUEST_LINE;
                push_to_worker_queue(c);
                break;
        }
    }
    
    if (events & APR_POLLOUT) {
        /* Socket ready for writing */
        if (cs->state == CONN_STATE_WRITE_COMPLETION) {
            push_to_worker_queue(c);
        }
    }
    
    if (events & (APR_POLLHUP | APR_POLLERR)) {
        /* Connection error or closed */
        close_connection(c);
    }
}
```

---

## 11. Configuration System {#configuration-system}

### 11.1 Configuration File Parsing

**Parser Implementation:**
```c
/* server/config.c */

/* Directive node in configuration tree */
struct ap_directive_t {
    const char *directive;         /* Directive name */
    const char *args;              /* Arguments */
    ap_directive_t *next;          /* Next sibling */
    ap_directive_t *first_child;   /* First child (for containers) */
    ap_directive_t *parent;        /* Parent directive */
    const char *filename;          /* Config file */
    int line_num;                  /* Line number */
};

/* Build configuration tree from file */
static const char *ap_build_config(apr_pool_t *p, apr_pool_t *temp_pool,
                                  const char *filename,
                                  ap_directive_t **conftree)
{
    ap_configfile_t *cfp;
    char *l;