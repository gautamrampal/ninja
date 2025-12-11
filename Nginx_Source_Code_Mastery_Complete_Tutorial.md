# Nginx Source Code Mastery - Complete Tutorial

## Table of Contents
1. [Introduction to Nginx Architecture](#introduction-to-nginx-architecture)
2. [Source Code Organization](#source-code-organization)
3. [Core Data Structures](#core-data-structures)
4. [Event-Driven Architecture](#event-driven-architecture)
5. [Process Model](#process-model)
6. [Memory Management](#memory-management)
7. [Configuration System](#configuration-system)
8. [HTTP Module Architecture](#http-module-architecture)
9. [Request Processing Pipeline](#request-processing-pipeline)
10. [Upstream and Load Balancing](#upstream-and-load-balancing)
11. [Caching Mechanism](#caching-mechanism)
12. [Module Development Deep Dive](#module-development-deep-dive)

---

## Introduction to Nginx Architecture

### High-Level Architecture Overview
```
┌─────────────────────────────────────────────┐
│           Master Process                     │
│  - Config parsing                           │
│  - Worker management                        │
│  - Signal handling                          │
└───────────┬─────────────────────────────────┘
            │
            ├─── Worker Process 1 (Event Loop)
            ├─── Worker Process 2 (Event Loop)
            ├─── Worker Process N (Event Loop)
            │
            └─── Cache Manager/Loader Processes
```

### Design Principles
1. **Event-Driven**: Asynchronous, non-blocking I/O
2. **Process-Based**: Master-worker process model
3. **Modular**: Pluggable module architecture
4. **Memory Efficient**: Pool-based memory allocation
5. **Zero-Copy**: Optimized data transfer paths

### Key Components
```c
// Core components in Nginx
src/core/           // Core functionality (memory, strings, lists)
src/event/          // Event handling mechanisms
src/http/           // HTTP module
src/mail/           // Mail proxy module
src/stream/         // TCP/UDP proxy module
src/os/             // OS-specific implementations
```

---

## Source Code Organization

### Directory Structure
```
nginx/
├── auto/              # Build configuration scripts
├── conf/              # Default configuration files
├── contrib/           # Contributed scripts and tools
├── docs/              # Documentation
├── misc/              # Miscellaneous files
├── man/               # Manual pages
└── src/               # Source code
    ├── core/          # Core functionality
    │   ├── nginx.c    # Main entry point
    │   ├── ngx_config.h
    │   ├── ngx_core.h
    │   ├── ngx_cycle.c        # Main cycle
    │   ├── ngx_connection.c   # Connection handling
    │   ├── ngx_palloc.c       # Memory pool
    │   └── ngx_array.c        # Array implementation
    ├── event/         # Event module
    │   ├── ngx_event.c
    │   ├── ngx_event_accept.c
    │   ├── modules/
    │   │   ├── ngx_epoll_module.c  # Linux epoll
    │   │   ├── ngx_kqueue_module.c # BSD kqueue
    │   │   └── ngx_select_module.c # select()
    ├── http/          # HTTP module
    │   ├── ngx_http.c
    │   ├── ngx_http_request.c
    │   ├── ngx_http_core_module.c
    │   ├── ngx_http_upstream.c
    │   ├── modules/           # HTTP submodules
    │   └── v2/                # HTTP/2 implementation
    ├── mail/          # Mail proxy
    ├── stream/        # Stream (TCP/UDP) module
    └── os/            # OS-specific code
        ├── unix/      # Unix/Linux implementations
        └── win32/     # Windows implementations
```

### Main Entry Point (src/core/nginx.c)
```c
// Main function in nginx.c
int ngx_cdecl
main(int argc, char *const *argv)
{
    ngx_buf_t        *b;
    ngx_log_t        *log;
    ngx_uint_t        i;
    ngx_cycle_t      *cycle, init_cycle;
    ngx_conf_dump_t  *cd;
    ngx_core_conf_t  *ccf;

    // Initialize error log
    ngx_debug_init();
    
    // Get and initialize log
    log = ngx_log_init(ngx_prefix);
    
    // Initialize time
    ngx_time_init();
    
    // Process command line arguments
    if (ngx_get_options(argc, argv) != NGX_OK) {
        return 1;
    }
    
    // Initialize OpenSSL
    if (ngx_ssl_init(log) != NGX_OK) {
        return 1;
    }
    
    // Initialize cycle (main configuration container)
    ngx_memzero(&init_cycle, sizeof(ngx_cycle_t));
    init_cycle.log = log;
    ngx_cycle = &init_cycle;
    
    // Create memory pool for cycle
    init_cycle.pool = ngx_create_pool(1024, log);
    
    // Initialize cycle
    cycle = ngx_init_cycle(&init_cycle);
    
    // Daemonize if needed
    if (ngx_daemon(log) != NGX_OK) {
        return 1;
    }
    
    // Create PID file
    if (ngx_create_pidfile(&ccf->pid, log) != NGX_OK) {
        return 1;
    }
    
    // Set signal handlers
    if (ngx_init_signals(cycle->log) != NGX_OK) {
        return 1;
    }
    
    // Start worker processes
    ngx_master_process_cycle(cycle);
    
    return 0;
}
```

---

## Core Data Structures

### ngx_cycle_t - The Configuration Cycle
```c
// src/core/ngx_cycle.h
struct ngx_cycle_s {
    void                  ****conf_ctx;      // Configuration contexts
    ngx_pool_t               *pool;          // Memory pool
    
    ngx_log_t                *log;           // Log
    ngx_log_t                 new_log;
    
    ngx_uint_t                log_use_stderr;
    
    ngx_connection_t        **files;         // Connection files
    ngx_connection_t         *free_connections; // Free connections pool
    ngx_uint_t                free_connection_n; // Number of free connections
    
    ngx_module_t            **modules;       // Active modules
    ngx_uint_t                modules_n;     // Number of modules
    ngx_uint_t                modules_used;
    
    ngx_queue_t               reusable_connections_queue; // Reusable connections
    
    ngx_array_t               listening;     // Listening sockets
    ngx_array_t               paths;         // Paths
    ngx_array_t               config_dump;   // Config dump
    ngx_rbtree_t              config_dump_rbtree;
    ngx_rbtree_node_t         config_dump_sentinel;
    
    ngx_list_t                open_files;    // Open files
    ngx_list_t                shared_memory; // Shared memory
    
    ngx_uint_t                connection_n;  // Number of connections
    ngx_uint_t                files_n;       // Number of files
    
    ngx_connection_t         *connections;   // Connections array
    ngx_event_t              *read_events;   // Read events
    ngx_event_t              *write_events;  // Write events
    
    ngx_cycle_t              *old_cycle;     // Previous cycle (for reload)
    
    ngx_str_t                 conf_file;     // Configuration file path
    ngx_str_t                 conf_param;    // Configuration parameters
    ngx_str_t                 conf_prefix;   // Configuration prefix
    ngx_str_t                 prefix;        // Prefix
    ngx_str_t                 lock_file;     // Lock file
    ngx_str_t                 hostname;      // Hostname
};
```

### ngx_pool_t - Memory Pool
```c
// src/core/ngx_palloc.h
typedef struct ngx_pool_s ngx_pool_t;

struct ngx_pool_s {
    ngx_pool_data_t       d;           // Pool data
    size_t                max;         // Maximum allocation size
    ngx_pool_t           *current;     // Current pool
    ngx_chain_t          *chain;       // Chain of buffers
    ngx_pool_large_t     *large;       // Large allocations (>max)
    ngx_pool_cleanup_t   *cleanup;     // Cleanup handlers
    ngx_log_t            *log;         // Log
};

typedef struct {
    u_char               *last;        // End of used memory
    u_char               *end;         // End of pool
    ngx_pool_t           *next;        // Next pool in chain
    ngx_uint_t            failed;      // Failed allocations count
} ngx_pool_data_t;

// Pool allocation functions
ngx_pool_t *ngx_create_pool(size_t size, ngx_log_t *log);
void ngx_destroy_pool(ngx_pool_t *pool);
void ngx_reset_pool(ngx_pool_t *pool);

void *ngx_palloc(ngx_pool_t *pool, size_t size);
void *ngx_pnalloc(ngx_pool_t *pool, size_t size);
void *ngx_pcalloc(ngx_pool_t *pool, size_t size);
void *ngx_pmemalign(ngx_pool_t *pool, size_t size, size_t alignment);
ngx_int_t ngx_pfree(ngx_pool_t *pool, void *p);
```

### ngx_str_t - String Structure
```c
// src/core/ngx_string.h
typedef struct {
    size_t      len;      // String length
    u_char     *data;     // String data (NOT null-terminated)
} ngx_str_t;

// String macros and functions
#define ngx_string(str)     { sizeof(str) - 1, (u_char *) str }
#define ngx_null_string     { 0, NULL }
#define ngx_str_set(str, text)                                               \
    (str)->len = sizeof(text) - 1; (str)->data = (u_char *) text
#define ngx_str_null(str)   (str)->len = 0; (str)->data = NULL

// String comparison
#define ngx_strcmp(s1, s2)  strcmp((const char *) s1, (const char *) s2)
#define ngx_strncmp(s1, s2, n)  strncmp((const char *) s1, (const char *) s2, n)
#define ngx_memcmp(s1, s2, n)   memcmp(s1, s2, n)
```

### ngx_array_t - Dynamic Array
```c
// src/core/ngx_array.h
typedef struct {
    void        *elts;      // Elements
    ngx_uint_t   nelts;     // Number of elements
    size_t       size;      // Size of single element
    ngx_uint_t   nalloc;    // Number of allocated elements
    ngx_pool_t  *pool;      // Memory pool
} ngx_array_t;

// Array functions
ngx_array_t *ngx_array_create(ngx_pool_t *p, ngx_uint_t n, size_t size);
void ngx_array_destroy(ngx_array_t *a);
void *ngx_array_push(ngx_array_t *a);
void *ngx_array_push_n(ngx_array_t *a, ngx_uint_t n);
```

### ngx_list_t - Linked List
```c
// src/core/ngx_list.h
typedef struct ngx_list_part_s  ngx_list_part_t;

struct ngx_list_part_s {
    void             *elts;    // Elements in this part
    ngx_uint_t        nelts;   // Number of elements
    ngx_list_part_t  *next;    // Next part
};

typedef struct {
    ngx_list_part_t  *last;    // Last part
    ngx_list_part_t   part;    // First part
    size_t            size;    // Element size
    ngx_uint_t        nalloc;  // Elements per part
    ngx_pool_t       *pool;    // Memory pool
} ngx_list_t;
```

### ngx_connection_t - Connection Structure
```c
// src/core/ngx_connection.h
struct ngx_connection_s {
    void               *data;           // Connection data
    ngx_event_t        *read;           // Read event
    ngx_event_t        *write;          // Write event
    
    ngx_socket_t        fd;             // Socket descriptor
    
    ngx_recv_pt         recv;           // Receive function pointer
    ngx_send_pt         send;           // Send function pointer
    ngx_recv_chain_pt   recv_chain;     // Receive chain
    ngx_send_chain_pt   send_chain;     // Send chain
    
    ngx_listening_t    *listening;      // Listening socket
    
    off_t               sent;           // Bytes sent
    
    ngx_log_t          *log;            // Connection log
    
    ngx_pool_t         *pool;           // Connection pool
    
    int                 type;           // Connection type
    struct sockaddr    *sockaddr;       // Socket address
    socklen_t           socklen;        // Socket address length
    ngx_str_t           addr_text;      // Text representation
    
    ngx_buf_t          *buffer;         // Buffer
    
    ngx_queue_t         queue;          // Queue node
    
    ngx_atomic_uint_t   number;         // Connection number
    
    ngx_uint_t          requests;       // Request count
    
    unsigned            log_error:3;    // Log error level
    
    unsigned            timedout:1;     // Timeout flag
    unsigned            error:1;        // Error flag
    unsigned            destroyed:1;    // Destroyed flag
    
    unsigned            idle:1;         // Idle flag
    unsigned            reusable:1;     // Reusable flag
    unsigned            close:1;        // Close flag
    
    unsigned            sendfile:1;     // Sendfile flag
    unsigned            sndlowat:1;     // Send low watermark
    unsigned            tcp_nodelay:2;  // TCP nodelay
    unsigned            tcp_nopush:2;   // TCP nopush
};
```

### ngx_event_t - Event Structure
```c
// src/event/ngx_event.h
struct ngx_event_s {
    void            *data;          // Event data
    
    unsigned         write:1;       // Write event flag
    
    unsigned         accept:1;      // Accept event flag
    
    unsigned         instance:1;    // Event instance
    
    unsigned         active:1;      // Active flag
    
    unsigned         disabled:1;    // Disabled flag
    
    unsigned         ready:1;       // Ready flag
    
    unsigned         oneshot:1;     // Oneshot flag
    
    unsigned         complete:1;    // Complete flag
    
    unsigned         eof:1;         // EOF flag
    unsigned         error:1;       // Error flag
    
    unsigned         timedout:1;    // Timeout flag
    unsigned         timer_set:1;   // Timer set flag
    
    unsigned         delayed:1;     // Delayed flag
    
    unsigned         deferred_accept:1; // Deferred accept
    
    unsigned         pending_eof:1;     // Pending EOF
    
    unsigned         posted:1;      // Posted flag
    
    unsigned         closed:1;      // Closed flag
    
    unsigned         available:1;   // Available data flag
    
    ngx_event_handler_pt  handler;  // Event handler
    
    ngx_log_t       *log;           // Event log
    
    ngx_rbtree_node_t   timer;      // Timer tree node
    
    ngx_queue_t      queue;         // Posted events queue
};
```

---

## Event-Driven Architecture

### Event Module Interface
```c
// src/event/ngx_event.h
typedef struct {
    ngx_str_t              *name;           // Module name
    
    void                 *(*create_conf)(ngx_cycle_t *cycle);
    char                 *(*init_conf)(ngx_cycle_t *cycle, void *conf);
    
    ngx_event_actions_t     actions;        // Event actions
} ngx_event_module_t;

// Event actions (function pointers for I/O operations)
typedef struct {
    ngx_int_t  (*add)(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);
    ngx_int_t  (*del)(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);
    
    ngx_int_t  (*enable)(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);
    ngx_int_t  (*disable)(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);
    
    ngx_int_t  (*add_conn)(ngx_connection_t *c);
    ngx_int_t  (*del_conn)(ngx_connection_t *c, ngx_uint_t flags);
    
    ngx_int_t  (*notify)(ngx_event_handler_pt handler);
    
    ngx_int_t  (*process_events)(ngx_cycle_t *cycle, ngx_msec_t timer,
                                  ngx_uint_t flags);
    
    ngx_int_t  (*init)(ngx_cycle_t *cycle, ngx_msec_t timer);
    void       (*done)(ngx_cycle_t *cycle);
} ngx_event_actions_t;
```

### Epoll Implementation (Linux)
```c
// src/event/modules/ngx_epoll_module.c
static ngx_int_t
ngx_epoll_init(ngx_cycle_t *cycle, ngx_msec_t timer)
{
    ngx_epoll_conf_t  *epcf;
    
    epcf = ngx_event_get_conf(cycle->conf_ctx, ngx_epoll_module);
    
    // Create epoll instance
    ep = epoll_create(cycle->connection_n / 2);
    
    if (ep == -1) {
        ngx_log_error(NGX_LOG_EMERG, cycle->log, ngx_errno,
                      "epoll_create() failed");
        return NGX_ERROR;
    }
    
    // Allocate event list
    event_list = ngx_alloc(sizeof(struct epoll_event) * epcf->events,
                           cycle->log);
    if (event_list == NULL) {
        return NGX_ERROR;
    }
    
    nevents = epcf->events;
    
    ngx_io = ngx_os_io;
    
    ngx_event_actions = ngx_epoll_module_ctx.actions;
    
    ngx_event_flags = NGX_USE_CLEAR_EVENT
                      |NGX_USE_GREEDY_EVENT
                      |NGX_USE_EPOLL_EVENT;
    
    return NGX_OK;
}

static ngx_int_t
ngx_epoll_process_events(ngx_cycle_t *cycle, ngx_msec_t timer,
    ngx_uint_t flags)
{
    int                events;
    uint32_t           revents;
    ngx_int_t          instance, i;
    ngx_uint_t         level;
    ngx_err_t          err;
    ngx_event_t       *rev, *wev;
    ngx_queue_t       *queue;
    ngx_connection_t  *c;
    
    // Wait for events
    events = epoll_wait(ep, event_list, (int) nevents, timer);
    
    err = (events == -1) ? ngx_errno : 0;
    
    if (flags & NGX_UPDATE_TIME || ngx_event_timer_alarm) {
        ngx_time_update();
    }
    
    if (err) {
        if (err == NGX_EINTR) {
            if (ngx_event_timer_alarm) {
                ngx_event_timer_alarm = 0;
                return NGX_OK;
            }
            
            level = NGX_LOG_INFO;
        } else {
            level = NGX_LOG_ALERT;
        }
        
        ngx_log_error(level, cycle->log, err, "epoll_wait() failed");
        return NGX_ERROR;
    }
    
    if (events == 0) {
        if (timer != NGX_TIMER_INFINITE) {
            return NGX_OK;
        }
        
        ngx_log_error(NGX_LOG_ALERT, cycle->log, 0,
                      "epoll_wait() returned no events without timeout");
        return NGX_ERROR;
    }
    
    // Process events
    for (i = 0; i < events; i++) {
        c = event_list[i].data.ptr;
        
        instance = (uintptr_t) c & 1;
        c = (ngx_connection_t *) ((uintptr_t) c & (uintptr_t) ~1);
        
        rev = c->read;
        
        if (c->fd == -1 || rev->instance != instance) {
            continue;
        }
        
        revents = event_list[i].events;
        
        // Handle read events
        if ((revents & EPOLLIN) && rev->active) {
            rev->ready = 1;
            
            if (flags & NGX_POST_EVENTS) {
                queue = rev->accept ? &ngx_posted_accept_events
                                    : &ngx_posted_events;
                
                ngx_post_event(rev, queue);
            } else {
                rev->handler(rev);
            }
        }
        
        wev = c->write;
        
        // Handle write events
        if ((revents & EPOLLOUT) && wev->active) {
            if (c->fd == -1 || wev->instance != instance) {
                continue;
            }
            
            wev->ready = 1;
            
            if (flags & NGX_POST_EVENTS) {
                ngx_post_event(wev, &ngx_posted_events);
            } else {
                wev->handler(wev);
            }
        }
    }
    
    return NGX_OK;
}
```

### Event Processing Loop
```c
// src/event/ngx_event.c
void
ngx_process_events_and_timers(ngx_cycle_t *cycle)
{
    ngx_uint_t  flags;
    ngx_msec_t  timer, delta;
    
    // Get timer for next expiration
    timer = ngx_event_find_timer();
    
    flags = NGX_UPDATE_TIME;
    
    if (ngx_use_accept_mutex) {
        // Try to acquire accept mutex
        if (ngx_trylock_accept_mutex(cycle) == NGX_ERROR) {
            return;
        }
        
        if (ngx_accept_mutex_held) {
            flags |= NGX_POST_EVENTS;
        } else {
            if (timer == NGX_TIMER_INFINITE
                || timer > ngx_accept_mutex_delay)
            {
                timer = ngx_accept_mutex_delay;
            }
        }
    }
    
    delta = ngx_current_msec;
    
    // Process I/O events
    (void) ngx_process_events(cycle, timer, flags);
    
    delta = ngx_current_msec - delta;
    
    // Process posted accept events
    ngx_event_process_posted(cycle, &ngx_posted_accept_events);
    
    // Release accept mutex
    if (ngx_accept_mutex_held) {
        ngx_shmtx_unlock(&ngx_accept_mutex);
    }
    
    // Expire timers
    if (delta) {
        ngx_event_expire_timers();
    }
    
    // Process regular posted events
    ngx_event_process_posted(cycle, &ngx_posted_events);
}
```

---

## Process Model

### Master Process
```c
// src/os/unix/ngx_process_cycle.c
void
ngx_master_process_cycle(ngx_cycle_t *cycle)
{
    char              *title;
    u_char            *p;
    size_t             size;
    ngx_int_t          i;
    ngx_uint_t         n, sigio;
    sigset_t           set;
    struct itimerval   itv;
    ngx_uint_t         live;
    ngx_msec_t         delay;
    ngx_listening_t   *ls;
    ngx_core_conf_t   *ccf;
    
    // Block signals
    sigemptyset(&set);
    sigaddset(&set, SIGCHLD);
    sigaddset(&set, SIGALRM);
    sigaddset(&set, SIGIO);
    sigaddset(&set, SIGINT);
    sigaddset(&set, ngx_signal_value(NGX_RECONFIGURE_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_REOPEN_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_NOACCEPT_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_TERMINATE_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_SHUTDOWN_SIGNAL));
    sigaddset(&set, ngx_signal_value(NGX_CHANGEBIN_SIGNAL));
    
    if (sigprocmask(SIG_BLOCK, &set, NULL) == -1) {
        ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                      "sigprocmask() failed");
    }
    
    sigemptyset(&set);
    
    // Set process title
    size = sizeof(master_process);
    
    for (i = 0; i < ngx_argc; i++) {
        size += ngx_strlen(ngx_argv[i]) + 1;
    }
    
    title = ngx_pnalloc(cycle->pool, size);
    if (title == NULL) {
        /* fatal */
        exit(2);
    }
    
    p = ngx_cpymem(title, master_process, sizeof(master_process) - 1);
    for (i = 0; i < ngx_argc; i++) {
        *p++ = ' ';
        p = ngx_cpystrn(p, (u_char *) ngx_argv[i], size);
    }
    
    ngx_setproctitle(title);
    
    ccf = (ngx_core_conf_t *) ngx_get_conf(cycle->conf_ctx, ngx_core_module);
    
    // Start worker processes
    ngx_start_worker_processes(cycle, ccf->worker_processes,
                               NGX_PROCESS_RESPAWN);
    
    // Start cache manager and loader
    ngx_start_cache_manager_processes(cycle, 0);
    
    ngx_new_binary = 0;
    delay = 0;
    sigio = 0;
    live = 1;
    
    // Main master loop
    for ( ;; ) {
        if (delay) {
            if (ngx_sigalrm) {
                sigio = 0;
                delay *= 2;
                ngx_sigalrm = 0;
            }
            
            ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                           "termination cycle: %M", delay);
            
            itv.it_interval.tv_sec = 0;
            itv.it_interval.tv_usec = 0;
            itv.it_value.tv_sec = delay / 1000;
            itv.it_value.tv_usec = (delay % 1000) * 1000;
            
            if (setitimer(ITIMER_REAL, &itv, NULL) == -1) {
                ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno,
                              "setitimer() failed");
            }
        }
        
        ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0, "sigsuspend");
        
        sigsuspend(&set);
        
        ngx_time_update();
        
        ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                       "wake up, sigio %i", sigio);
        
        // Handle signals
        if (ngx_reap) {
            ngx_reap = 0;
            ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0, "reap children");
            
            live = ngx_reap_children(cycle);
        }
        
        if (!live && (ngx_terminate || ngx_quit)) {
            ngx_master_process_exit(cycle);
        }
        
        if (ngx_terminate) {
            if (delay == 0) {
                delay = 50;
            }
            
            if (sigio) {
                sigio--;
                continue;
            }
            
            sigio = ccf->worker_processes + 2 /* cache processes */;
            
            if (delay > 1000) {
                ngx_signal_worker_processes(cycle, SIGKILL);
            } else {
                ngx_signal_worker_processes(cycle,
                                           ngx_signal_value(NGX_TERMINATE_SIGNAL));
            }
            
            continue;
        }
        
        if (ngx_quit) {
            ngx_signal_worker_processes(cycle,
                                       ngx_signal_value(NGX_SHUTDOWN_SIGNAL));
            
            ls = cycle->listening.elts;
            for (n = 0; n < cycle->listening.nelts; n++) {
                if (ngx_close_socket(ls[n].fd) == -1) {
                    ngx_log_error(NGX_LOG_EMERG, cycle->log, ngx_socket_errno,
                                  ngx_close_socket_n " %V failed",
                                  &ls[n].addr_text);
                }
            }
            cycle->listening.nelts = 0;
            
            continue;
        }
        
        if (ngx_reconfigure) {
            ngx_reconfigure = 0;
            
            if (ngx_new_binary) {
                ngx_start_worker_processes(cycle, ccf->worker_processes,
                                           NGX_PROCESS_RESPAWN);
                ngx_start_cache_manager_processes(cycle, 0);
                ngx_noaccepting = 0;
                
                continue;
            }
            
            ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0, "reconfiguring");
            
            cycle = ngx_init_cycle(cycle);
            if (cycle == NULL) {
                cycle = (ngx_cycle_t *) ngx_cycle;
                continue;
            }
            
            ngx_cycle = cycle;
            ccf = (ngx_core_conf_t *) ngx_get_conf(cycle->conf_ctx,
                                                    ngx_core_module);
            ngx_start_worker_processes(cycle, ccf->worker_processes,
                                       NGX_PROCESS_JUST_RESPAWN);
            ngx_start_cache_manager_processes(cycle, 1);
            
            /* allow new processes to start */
            ngx_msleep(100);
            
            live = 1;
            ngx_signal_worker_processes(cycle,
                                       ngx_signal_value(NGX_SHUTDOWN_SIGNAL));
        }
        
        if (ngx_restart) {
            ngx_restart = 0;
            ngx_start_worker_processes(cycle, ccf->worker_processes,
                                       NGX_PROCESS_RESPAWN);
            ngx_start_cache_manager_processes(cycle, 0);
            live = 1;
        }
        
        if (ngx_reopen) {
            ngx_reopen = 0;
            ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0, "reopening logs");
            ngx_reopen_files(cycle, ccf->user);
            ngx_signal_worker_processes(cycle,
                                       ngx_signal_value(NGX_REOPEN_SIGNAL));
        }
        
        if (ngx_change_binary) {
            ngx_change_binary = 0;
            ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0, "changing binary");
            ngx_new_binary = ngx_exec_new_binary(cycle, ngx_argv);
        }
        
        if (ngx_noaccept) {
            ngx_noaccept = 0;
            ngx_noaccepting = 1;
            ngx_signal_worker_processes(cycle,
                                       ngx_signal_value(NGX_SHUTDOWN_SIGNAL));
        }
    }
}
```

### Worker Process
```c
static void
ngx_worker_process_cycle(ngx_cycle_t *cycle, void *data)
{
    ngx_int_t worker = (intptr_t) data;
    
    ngx_process = NGX_PROCESS_WORKER;
    ngx_worker = worker;
    
    // Initialize worker process
    ngx_worker_process_init(cycle, worker);
    
    ngx_setproctitle("worker process");
    
    // Main worker loop
    for ( ;; ) {
        
        if (ngx_exiting) {
            ngx_event_cancel_timers();
            
            if (ngx_event_timer_rbtree.root == ngx_event_timer_rbtree.sentinel)
            {
                ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0, "exiting");
                ngx_worker_process_exit(cycle);
            }
        }
        
        ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0, "worker cycle");
        
        // Process events and timers
        ngx_process_events_and_timers(cycle);
        
        if (ngx_terminate) {
            ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0, "exiting");
            ngx_worker_process_exit(cycle);
        }
        
        if (ngx_quit) {
            ngx_quit = 0;
            ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0,
                          "gracefully shutting down");
            ngx_setproctitle("worker process is shutting down");
            
            if (!ngx_exiting) {
                ngx_exiting = 1;
                ngx_close_listening_sockets(cycle);
                ngx_close_idle_connections(cycle);
            }
        }
        
        if (ngx_reopen) {
            ngx_reopen = 0;
            ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0, "reopening logs");
            ngx_reopen_files(cycle, -1);
        }
    }
}
```

---

## Memory Management

### Memory Pool Implementation
```c
// src/core/ngx_palloc.c
ngx_pool_t *
ngx_create_pool(size_t size, ngx_log_t *log)
{
    ngx_pool_t  *p;
    
    // Allocate memory for pool structure + initial data block
    p = ngx_memalign(NGX_POOL_ALIGNMENT, size, log);
    if (p == NULL) {
        return NULL;
    }
    
    // Initialize pool data
    p->d.last = (u_char *) p + sizeof(ngx_pool_t);
    p->d.end = (u_char *) p + size;
    p->d.next = NULL;
    p->d.failed = 0;
    
    size = size - sizeof(ngx_pool_t);
    p->max = (size < NGX_MAX_ALLOC_FROM_POOL) ? size : NGX_MAX_ALLOC_FROM_POOL;
    
    p->current = p;
    p->chain = NULL;
    p->large = NULL;
    p->cleanup = NULL;
    p->log = log;
    
    return p;
}

void *
ngx_palloc(ngx_pool_t *pool, size_t size)
{
    if (size <= pool->max) {
        return ngx_palloc_small(pool, size, 1);
    }
    
    return ngx_palloc_large(pool, size);
}

static ngx_inline void *
ngx_palloc_small(ngx_pool_t *pool, size_t size, ngx_uint_t align)
{
    u_char      *m;
    ngx_pool_t  *p;
    
    p = pool->current;
    
    do {
        m = p->d.last;
        
        if (align) {
            m = ngx_align_ptr(m, NGX_ALIGNMENT);
        }
        
        if ((size_t) (p->d.end - m) >= size) {
            p->d.last = m + size;
            return m;
        }
        
        p = p->d.next;
        
    } while (p);
    
    return ngx_palloc_block(pool, size);
}

static void *
ngx_palloc_block(ngx_pool_t *pool, size_t size)
{
    u_char      *m;
    size_t       psize;
    ngx_pool_t  *p, *new;
    
    psize = (size_t) (pool->d.end - (u_char *) pool);
    
    m = ngx_memalign(NGX_POOL_ALIGNMENT, psize, pool->log);
    if (m == NULL) {
        return NULL;
    }
    
    new = (ngx_pool_t *) m;
    
    new->d.end = m + psize;
    new->d.next = NULL;
    new->d.failed = 0;
    
    m += sizeof(ngx_pool_data_t);
    m = ngx_align_ptr(m, NGX_ALIGNMENT);
    new->d.last = m + size;
    
    for (p = pool->current; p->d.next; p = p->d.next) {
        if (p->d.failed++ > 4) {
            pool->current = p->d.next;
        }
    }
    
    p->d.next = new;
    
    return m;
}

static void *
ngx_palloc_large(ngx_pool_t *pool, size_t size)
{
    void              *p;
    ngx_uint_t         n;
    ngx_pool_large_t  *large;
    
    p = ngx_alloc(size, pool->log);
    if (p == NULL) {
        return NULL;
    }
    
    n = 0;
    
    for (large = pool->large; large; large = large->next) {
        if (large->alloc == NULL) {
            large->alloc = p;
            return p;
        }
        
        if (n++ > 3) {
            break;
        }
    }
    
    large = ngx_palloc_small(pool, sizeof(ngx_pool_large_t), 1);
    if (large == NULL) {
        ngx_free(p);
        return NULL;
    }
    
    large->alloc = p;
    large->next = pool->large;
    pool->large = large;
    
    return p;
}

void
ngx_destroy_pool(ngx_pool_t *pool)
{
    ngx_pool_t          *p, *n;
    ngx_pool_large_t    *l;
    ngx_pool_cleanup_t  *c;
    
    // Run cleanup handlers
    for (c = pool->cleanup; c; c = c->next) {
        if (c->handler) {
            ngx_log_debug1(NGX_LOG_DEBUG_ALLOC, pool->log, 0,
                           "run cleanup: %p", c);
            c->handler(c->data);
        }
    }
    
    // Free large allocations
    for (l = pool->large; l; l = l->next) {
        if (l->alloc) {
            ngx_free(l->alloc);
        }
    }
    
    // Free pool chain
    for (p = pool, n = pool->d.next; /* void */; p = n, n = n->d.next) {
        ngx_free(p);
        
        if (n == NULL) {
            break;
        }
    }
}
```

### Buffer Management
```c
// src/core/ngx_buf.h
typedef struct ngx_buf_s  ngx_buf_t;

struct ngx_buf_s {
    u_char          *pos;        // Current position in buffer
    u_char          *last;       // End of data in buffer
    off_t            file_pos;   // File position
    off_t            file_last;  // End of data in file
    
    u_char          *start;      // Start of buffer memory
    u_char          *end;        // End of buffer memory
    ngx_buf_tag_t    tag;        // Buffer tag
    ngx_file_t      *file;       // File reference
    ngx_buf_t       *shadow;     // Shadow buffer
    
    /* Flags */
    unsigned         temporary:1;
    unsigned         memory:1;
    unsigned         mmap:1;
    unsigned         recycled:1;
    unsigned         in_file:1;
    unsigned         flush:1;
    unsigned         sync:1;
    unsigned         last_buf:1;
    unsigned         last_in_chain:1;
    unsigned         last_shadow:1;
    unsigned         temp_file:1;
};

// Buffer chain
typedef struct ngx_chain_s  ngx_chain_t;

struct ngx_chain_s {
    ngx_buf_t    *buf;
    ngx_chain_t  *next;
};
```

---

## Configuration System

### Configuration Parsing
```c
// src/core/ngx_conf_file.c
char *
ngx_conf_parse(ngx_conf_t *cf, ngx_str_t *filename)
{
    char             *rv;
    ngx_fd_t          fd;
    ngx_int_t         rc;
    ngx_buf_t         buf;
    ngx_conf_file_t  *prev, conf_file;
    enum {
        parse_file = 0,
        parse_block,
        parse_param
    } type;
    
    if (filename) {
        // Open configuration file
        fd = ngx_open_file(filename->data, NGX_FILE_RDONLY, NGX_FILE_OPEN, 0);
        if (fd == NGX_INVALID_FILE) {
            ngx_conf_log_error(NGX_LOG_EMERG, cf, ngx_errno,
                               ngx_open_file_n " \"%s\" failed",
                               filename->data);
            return NGX_CONF_ERROR;
        }
        
        prev = cf->conf_file;
        
        cf->conf_file = &conf_file;
        
        if (ngx_fd_info(fd, &cf->conf_file->file.info) == NGX_FILE_ERROR) {
            ngx_log_error(NGX_LOG_EMERG, cf->log, ngx_errno,
                          ngx_fd_info_n " \"%s\" failed", filename->data);
        }
        
        cf->conf_file->buffer = &buf;
        
        buf.start = ngx_alloc(NGX_CONF_BUFFER, cf->log);
        if (buf.start == NULL) {
            goto failed;
        }
        
        buf.pos = buf.start;
        buf.last = buf.start;
        buf.end = buf.last + NGX_CONF_BUFFER;
        buf.temporary = 1;
        
        cf->conf_file->file.fd = fd;
        cf->conf_file->file.name.len = filename->len;
        cf->conf_file->file.name.data = filename->data;
        cf->conf_file->file.offset = 0;
        cf->conf_file->file.log = cf->log;
        cf->conf_file->line = 1;
        
        type = parse_file;
        
    } else if (cf->conf_file->file.fd != NGX_INVALID_FILE) {
        
        type = parse_block;
        
    } else {
        type = parse_param;
    }
    
    // Parse configuration
    for ( ;; ) {
        rc = ngx_conf_read_token(cf);
        
        if (rc == NGX_ERROR) {
            goto done;
        }
        
        if (rc == NGX_CONF_BLOCK_DONE) {
            
            if (type != parse_block) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "unexpected \"}\"");
                goto failed;
            }
            
            goto done;
        }
        
        if (rc == NGX_CONF_FILE_DONE) {
            
            if (type == parse_block) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                                   "unexpected end of file, expecting \"}\"");
                goto failed;
            }
            
            goto done;
        }
        
        if (rc == NGX_CONF_BLOCK_START) {
            
            if (type == parse_param) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                                   "block directives are not supported "
                                   "in -g option");
                goto failed;
            }
        }
        
        // Process directive
        rv = ngx_conf_handler(cf, rc);
        
        if (rv == NGX_CONF_OK) {
            continue;
        }
        
        if (rv == NGX_CONF_ERROR) {
            goto failed;
        }
        
        ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "%s", rv);
        
        goto failed;
    }
    
failed:
    
    rc = NGX_ERROR;
    
done:
    
    if (filename) {
        if (cf->conf_file->buffer->start) {
            ngx_free(cf->conf_file->buffer->start);
        }
        
        if (ngx_close_file(fd) == NGX_FILE_ERROR) {
            ngx_log_error(NGX_LOG_ALERT, cf->log, ngx_errno,
                          ngx_close_file_n " %s failed",
                          filename->data);
            rc = NGX_ERROR;
        }
        
        cf->conf_file = prev;
    }
    
    if (rc == NGX_ERROR) {
        return NGX_CONF_ERROR;
    }
    
    return NGX_CONF_OK;
}
```

### Module Configuration
```c
// Module configuration structure example
typedef struct {
    ngx_flag_t  enable;
    ngx_str_t   value;
    ngx_uint_t  number;
} ngx_http_custom_loc_conf_t;

// Configuration creation
static void *
ngx_http_custom_create_loc_conf(ngx_conf_t *cf)
{
    ngx_http_custom_loc_conf_t  *conf;
    
    conf = ngx_pcalloc(cf->pool, sizeof(ngx_http_custom_loc_conf_t));
    if (conf == NULL) {
        return NULL;
    }
    
    conf->enable = NGX_CONF_UNSET;
    conf->number = NGX_CONF_UNSET_UINT;
    
    return conf;
}

// Configuration merging
static char *
ngx_http_custom_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child)
{
    ngx_http_custom_loc_conf_t *prev = parent;
    ngx_http_custom_loc_conf_t *conf = child;
    
    ngx_conf_merge_value(conf->enable, prev->enable, 0);
    ngx_conf_merge_str_value(conf->value, prev->value, "");
    ngx_conf_merge_uint_value(conf->number, prev->number, 0);
    
    return NGX_CONF_OK;
}
```

---

## HTTP Module Architecture

### HTTP Request Structure
```c
// src/http/ngx_http_request.h
struct ngx_http_request_s {
    uint32_t                          signature;         /* "HTTP" */
    
    ngx_connection_t                 *connection;
    
    void                            **ctx;
    void                            **main_conf;
    void                            **srv_conf;
    void                            **loc_conf;
    
    ngx_http_event_handler_pt         read_event_handler;
    ngx_http_event_handler_pt         write_event_handler;
    
    ngx_http_cache_t                 *cache;
    
    ngx_http_upstream_t              *upstream;
    ngx_array_t                      *upstream_states;
    
    ngx_pool_t                       *pool;
    ngx_buf_t                        *header_in;
    
    ngx_http_headers_in_t             headers_in;
    ngx_http_headers_out_t            headers_out;
    
    ngx_http_request_body_t          *request_body;
    
    time_t                            lingering_time;
    time_t                            start_sec;
    ngx_msec_t                        start_msec;
    
    ngx_uint_t                        method;
    ngx_uint_t                        http_version;
    
    ngx_str_t                         request_line;
    ngx_str_t                         uri;
    ngx_str_t                         args;
    ngx_str_t                         exten;
    ngx_str_t                         unparsed_uri;
    
    ngx_str_t                         method_name;
    ngx_str_t                         http_protocol;
    
    ngx_chain_t                      *out;
    ngx_http_request_t               *main;
    ngx_http_request_t               *parent;
    ngx_http_postponed_request_t     *postponed;
    ngx_http_post_subrequest_t       *post_subrequest;
    ngx_http_posted_request_t        *posted_requests;
    
    ngx_int_t                         phase_handler;
    ngx_http_handler_pt               content_handler;
    
    ngx_uint_t                        access_code;
    
    ngx_http_variable_value_t        *variables;
    
    /* ... many more fields ... */
    
    unsigned                          http_state:4;
    
    unsigned                          uri_changed:1;
    unsigned                          complex_uri:1;
    unsigned                          quoted_uri:1;
    unsigned                          plus_in_uri:1;
    unsigned                          space_in_uri:1;
    unsigned                          invalid_header:1;
    
    unsigned                          header_only:1;
    unsigned                          keepalive:1;
    unsigned                          lingering_close:1;
    
    unsigned                          internal:1;
    unsigned                          error_page:1;
    unsigned                          filter_finalize:1;
    unsigned                          post_action:1;
    unsigned                          request_complete:1;
    unsigned                          request_output:1;
    unsigned                          header_sent:1;
    unsigned                          expect_tested:1;
    unsigned                          root_tested:1;
    unsigned                          done:1;
    unsigned                          logged:1;
    
    unsigned                          buffered:4;
    
    unsigned                          main_filter_need_in_memory:1;
    unsigned                          filter_need_in_memory:1;
    unsigned                          filter_need_temporary:1;
    unsigned                          preserve_body:1;
    unsigned                          allow_ranges:1;
    unsigned                          subrequest_ranges:1;
    unsigned                          single_range:1;
    unsigned                          disable_not_modified:1;
    unsigned                          stat_reading:1;
    unsigned                          stat_writing:1;
    unsigned                          stat_processing:1;
    
    unsigned                          background:1;
    unsigned                          health_check:1;
    
    unsigned                          count:16;
    
    unsigned                          subrequests:8;
    unsigned                          blocked:8;
    
    unsigned                          aio:1;
    
    ngx_http_connection_t            *http_connection;
    ngx_http_v2_stream_t             *stream;
    
    ngx_http_log_handler_pt           log_handler;
    
    ngx_http_cleanup_t               *cleanup;
    
    unsigned                          subrequest_in_memory:1;
    unsigned                          waited:1;
    
    unsigned                          cached:1;
    
    unsigned                          gzip_tested:1;
    unsigned                          gzip_ok:1;
    unsigned                          gzip_vary:1;
    
    unsigned                          proxy:1;
    unsigned                          bypass_cache:1;
    unsigned                          no_cache:1;
    
    unsigned                          limit_conn_status:2;
    unsigned                          limit_req_status:3;
    
    unsigned                          pipeline:1;
    
    unsigned                          chunked:1;
    unsigned                          header_hash:1;
    
    ngx_int_t                         limit_rate;
    ngx_int_t                         limit_rate_after;
    
    ngx_msec_t                        read_event_timeout;
    ngx_msec_t                        write_event_timeout;
};
```

---

## Request Processing Pipeline

### HTTP Processing Phases
```c
// src/http/ngx_http_core_module.h
typedef enum {
    NGX_HTTP_POST_READ_PHASE = 0,
    
    NGX_HTTP_SERVER_REWRITE_PHASE,
    
    NGX_HTTP_FIND_CONFIG_PHASE,
    NGX_HTTP_REWRITE_PHASE,
    NGX_HTTP_POST_REWRITE_PHASE,
    
    NGX_HTTP_PREACCESS_PHASE,
    
    NGX_HTTP_ACCESS_PHASE,
    NGX_HTTP_POST_ACCESS_PHASE,
    
    NGX_HTTP_PRECONTENT_PHASE,
    
    NGX_HTTP_CONTENT_PHASE,
    
    NGX_HTTP_LOG_PHASE
} ngx_http_phases;
```

### Phase Handler
```c
// src/http/ngx_http_core_module.c
void
ngx_http_core_run_phases(ngx_http_request_t *r)
{
    ngx_int_t                   rc;
    ngx_http_phase_handler_t   *ph;
    ngx_http_core_main_conf_t  *cmcf;
    
    cmcf = ngx_http_get_module_main_conf(r, ngx_http_core_module);
    
    ph = cmcf->phase_engine.handlers;
    
    while (ph[r->phase_handler].checker) {
        
        rc = ph[r->phase_handler].checker(r, &ph[r->phase_handler]);
        
        if (rc == NGX_OK) {
            return;
        }
    }
}

// Generic phase checker
ngx_int_t
ngx_http_core_generic_phase(ngx_http_request_t *r, ngx_http_phase_handler_t *ph)
{
    ngx_int_t  rc;
    
    ngx_log_debug1(NGX_LOG_DEBUG_HTTP, r->connection->log, 0,
                   "generic phase: %ui", r->phase_handler);
    
    rc = ph->handler(r);
    
    if (rc == NGX_OK) {
        r->phase_handler = ph->next;
        return NGX_AGAIN;
    }
    
    if (rc == NGX_DECLINED) {
        r->phase_handler++;
        return NGX_AGAIN;
    }
    
    if (rc == NGX_AGAIN || rc == NGX_DONE) {
        return NGX_OK;
    }
    
    /* rc == NGX_ERROR || rc == NGX_HTTP_...  */
    
    ngx_http_finalize_request(r, rc);
    
    return NGX_OK;
}
```

### Request Initialization
```c
// src/http/ngx_http_request.c
static void
ngx_http_process_request(ngx_http_request_t *r)
{
    ngx_connection_t  *c;
    
    c = r->connection;
    
    if (r->plain_http) {
        ngx_log_error(NGX_LOG_INFO, c->log, 0,
                      "client sent plain HTTP request to HTTPS port");
        ngx_http_finalize_request(r, NGX_HTTP_TO_HTTPS);
        return;
    }
    
    if (c->read->timer_set) {
        ngx_del_timer(c->read);
    }
    
    c->read->handler = ngx_http_request_handler;
    c->write->handler = ngx_http_request_handler;
    r->read_event_handler = ngx_http_block_reading;
    
    ngx_http_handler(r);
    
    ngx_http_run_posted_requests(c);
}

void
ngx_http_handler(ngx_http_request_t *r)
{
    ngx_http_core_main_conf_t  *cmcf;
    
    r->connection->log->action = NULL;
    
    if (!r->internal) {
        switch (r->headers_in.connection_type) {
        case 0:
            r->keepalive = (r->http_version > NGX_HTTP_VERSION_10);
            break;
            
        case NGX_HTTP_CONNECTION_CLOSE:
            r->keepalive = 0;
            break;
            
        case NGX_HTTP_CONNECTION_KEEP_ALIVE:
            r->keepalive = 1;
            break;
        }
        
        r->lingering_close = (r->headers_in.content_length_n > 0
                              || r->headers_in.chunked);
        r->phase_handler = 0;
        
    } else {
        cmcf = ngx_http_get_module_main_conf(r, ngx_http_core_module);
        r->phase_handler = cmcf->phase_engine.server_rewrite_index;
    }
    
    r->write_event_handler = ngx_http_core_run_phases;
    ngx_http_core_run_phases(r);
}
```

---

## Upstream and Load Balancing

### Upstream Structure
```c
// src/http/ngx_http_upstream.h
struct ngx_http_upstream_s {
    ngx_http_upstream_handler_pt     read_event_handler;
    ngx_http_upstream_handler_pt     write_event_handler;
    
    ngx_peer_connection_t            peer;
    
    ngx_event_pipe_t                *pipe;
    
    ngx_chain_t                     *request_bufs;
    
    ngx_output_chain_ctx_t           output;
    ngx_chain_writer_ctx_t           writer;
    
    ngx_http_upstream_conf_t        *conf;
    ngx_http_upstream_srv_conf_t    *upstream;
    
    ngx_array_t                     *caches;
    
    ngx_http_upstream_resolved_t    *resolved;
    
    ngx_buf_t                        from_client;
    
    ngx_buf_t                        buffer;
    off_t                            length;
    
    ngx_chain_t                     *out_bufs;
    ngx_chain_t                     *busy_bufs;
    ngx_chain_t                     *free_bufs;
    
    ngx_int_t                      (*input_filter_init)(void *data);
    ngx_int_t                      (*input_filter)(void *data, ssize_t bytes);
    void                            *input_filter_ctx;
    
    ngx_int_t                      (*create_request)(ngx_http_request_t *r);
    ngx_int_t                      (*reinit_request)(ngx_http_request_t *r);
    ngx_int_t                      (*process_header)(ngx_http_request_t *r);
    void                           (*abort_request)(ngx_http_request_t *r);
    void                           (*finalize_request)(ngx_http_request_t *r,
                                                        ngx_int_t rc);
    ngx_int_t                      (*rewrite_redirect)(ngx_http_request_t *r,
                                                        ngx_table_elt_t *h, size_t prefix);
    ngx_int_t                      (*rewrite_cookie)(ngx_http_request_t *r,
                                                      ngx_table_elt_t *h);
    
    ngx_msec_t                       timeout;
    
    ngx_http_upstream_state_t       *state;
    
    ngx_str_t                        method;
    ngx_str_t                        schema;
    ngx_str_t                        uri;
    
    ngx_http_cleanup_pt             *cleanup;
    
    unsigned                         store:1;
    unsigned                         cacheable:1;
    unsigned                         accel:1;
    unsigned                         ssl:1;
    unsigned                         cache_status:3;
    
    unsigned                         buffering:1;
    unsigned                         keepalive:1;
    unsigned                         upgrade:1;
    
    unsigned                         request_sent:1;
    unsigned                         request_body_sent:1;
    unsigned                         header_sent:1;
};
```

### Round-Robin Load Balancing
```c
// src/http/ngx_http_upstream_round_robin.c
ngx_int_t
ngx_http_upstream_init_round_robin(ngx_conf_t *cf,
    ngx_http_upstream_srv_conf_t *us)
{
    ngx_url_t                      u;
    ngx_uint_t                     i, j, n, w;
    ngx_http_upstream_server_t    *server;
    ngx_http_upstream_rr_peer_t   *peer, **peerp;
    ngx_http_upstream_rr_peers_t  *peers, *backup;
    
    us->peer.init = ngx_http_upstream_init_round_robin_peer;
    
    if (us->servers == NULL || us->servers->nelts == 0) {
        return NGX_ERROR;
    }
    
    server = us->servers->elts;
    
    n = 0;
    w = 0;
    
    for (i = 0; i < us->servers->nelts; i++) {
        if (server[i].backup) {
            continue;
        }
        
        n += server[i].naddrs;
        w += server[i].naddrs * server[i].weight;
    }
    
    if (n == 0) {
        return NGX_ERROR;
    }
    
    peers = ngx_pcalloc(cf->pool, sizeof(ngx_http_upstream_rr_peers_t));
    if (peers == NULL) {
        return NGX_ERROR;
    }
    
    peer = ngx_pcalloc(cf->pool, sizeof(ngx_http_upstream_rr_peer_t) * n);
    if (peer == NULL) {
        return NGX_ERROR;
    }
    
    peers->single = (n == 1);
    peers->number = n;
    peers->weighted = (w != n);
    peers->total_weight = w;
    peers->name = &us->host;
    
    n = 0;
    peerp = &peers->peer;
    
    for (i = 0; i < us->servers->nelts; i++) {
        if (server[i].backup) {
            continue;
        }
        
        for (j = 0; j < server[i].naddrs; j++) {
            peer[n].sockaddr = server[i].addrs[j].sockaddr;
            peer[n].socklen = server[i].addrs[j].socklen;
            peer[n].name = server[i].addrs[j].name;
            peer[n].weight = server[i].weight;
            peer[n].effective_weight = server[i].weight;
            peer[n].current_weight = 0;
            peer[n].max_fails = server[i].max_fails;
            peer[n].fail_timeout = server[i].fail_timeout;
            peer[n].down = server[i].down;
            peer[n].server = server[i].name;
            
            *peerp = &peer[n];
            peerp = &peer[n].next;
            n++;
        }
    }
    
    us->peer.data = peers;
    
    return NGX_OK;
}

ngx_int_t
ngx_http_upstream_get_round_robin_peer(ngx_peer_connection_t *pc, void *data)
{
    ngx_http_upstream_rr_peer_data_t  *rrp = data;
    
    ngx_int_t                      rc;
    ngx_uint_t                     i, n;
    ngx_http_upstream_rr_peer_t   *peer;
    ngx_http_upstream_rr_peers_t  *peers;
    
    peers = rrp->peers;
    
    if (peers->single) {
        peer = peers->peer;
        
        if (peer->down) {
            goto failed;
        }
        
        rrp->current = peer;
        
    } else {
        
        /* ngx_http_upstream_get_peer */
        
        peer = ngx_http_upstream_get_peer(rrp);
        
        if (peer == NULL) {
            goto failed;
        }
        
        rrp->current = peer;
    }
    
    pc->sockaddr = peer->sockaddr;
    pc->socklen = peer->socklen;
    pc->name = &peer->name;
    
    peer->conns++;
    
    return NGX_OK;
    
failed:
    
    peers = rrp->peers;
    
    if (peers->next) {
        rrp->peers = peers->next;
        
        n = (rrp->peers->number + (8 * sizeof(uintptr_t) - 1))
                / (8 * sizeof(uintptr_t));
        
        for (i = 0; i < n; i++) {
            rrp->tried[i] = 0;
        }
        
        rc = ngx_http_upstream_get_round_robin_peer(pc, rrp);
        
        if (rc != NGX_BUSY) {
            return rc;
        }
    }
    
    return NGX_BUSY;
}
```

---

## Caching Mechanism

### Cache Manager
```c
// src/http/ngx_http_file_cache.c
static void
ngx_http_cache_manager_process_handler(ngx_event_t *ev)
{
    ngx_uint_t                     i;
    ngx_msec_t                     next, n;
    ngx_path_t                   **path;
    ngx_http_file_cache_t        **cache;
    ngx_http_cache_manager_ctx_t  *ctx;
    
    ctx = ev->data;
    
    cache = ctx->cycle->cache->elts;
    
    for (i = 0; i < ctx->cycle->cache->nelts; i++) {
        
        if (ngx_http_file_cache_manager(cache[i]) != 0) {
            
            ngx_time_update();
            
            next = (ngx_msec_t) ngx_http_file_cache_manager(cache[i]);
            
            if (next == 0) {
                next = ctx->manager;
            }
            
            ngx_add_timer(ev, next);
            
            return;
        }
    }
    
    path = ctx->cycle->paths.elts;
    
    for (i = 0; i < ctx->cycle->paths.nelts; i++) {
        
        if (ngx_path_manager_pt(path[i]) != 0) {
            
            ngx_time_update();
            
            n = (ngx_msec_t) path[i]->manager(path[i]->data);
            
            if (n == 0) {
                n = ctx->manager;
            }
            
            ngx_add_timer(ev, n);
            
            return;
        }
    }
    
    ngx_add_timer(ev, ctx->manager);
}

ngx_int_t
ngx_http_file_cache_manager(ngx_http_file_cache_t *cache)
{
    off_t                   size;
    time_t                  wait;
    ngx_uint_t              count, watermark;
    ngx_msec_t              elapsed;
    ngx_tree_ctx_t          tree;
    ngx_http_file_cache_cleanup_t  cleanup;
    
    cache->last = ngx_current_msec;
    cache->files = 0;
    
    ngx_log_debug2(NGX_LOG_DEBUG_HTTP, ngx_cycle->log, 0,
                   "http file cache manager: \"%V\" e:%d",
                   &cache->path->name, cache->last_expired);
    
    tree.init_handler = NULL;
    tree.file_handler = ngx_http_file_cache_manage_file;
    tree.pre_tree_handler = ngx_http_file_cache_manage_directory;
    tree.post_tree_handler = ngx_http_file_cache_noop;
    tree.spec_handler = ngx_http_file_cache_delete_file;
    tree.data = &cleanup;
    tree.alloc = 0;
    tree.log = ngx_cycle->log;
    
    cleanup.cache = cache;
    cleanup.current_time = ngx_time();
    
    if (ngx_walk_tree(&tree, &cache->path->name) == NGX_ABORT) {
        return cache->manager_sleep;
    }
    
    size = cache->sh->size;
    count = cache->sh->count;
    watermark = cache->max_size * cache->manager_threshold / 100;
    
    if (size >= watermark || count >= cache->max_count) {
        if (ngx_http_file_cache_forced_expire(cache, watermark) != NGX_OK) {
            return cache->manager_sleep;
        }
    }
    
    elapsed = ngx_abs((ngx_msec_int_t) (cache->last - cache->last_expired));
    
    if (elapsed < cache->inactive_time) {
        wait = cache->inactive_time - elapsed;
        return (ngx_msec_t) wait * 1000;
    }
    
    return cache->manager_sleep;
}
```

### Cache Lookup and Storage
```c
ngx_int_t
ngx_http_file_cache_open(ngx_http_request_t *r)
{
    ngx_int_t                  rc, rv;
    ngx_uint_t                 test;
    ngx_http_cache_t          *c;
    ngx_http_file_cache_t     *cache;
    ngx_http_core_loc_conf_t  *clcf;
    
    c = r->cache;
    
    if (c->waiting) {
        return NGX_AGAIN;
    }
    
    if (c->reading) {
        return ngx_http_file_cache_read(r, c);
    }
    
    cache = c->file_cache;
    
    if (c->node == NULL) {
        // Create cache key
        ngx_http_file_cache_create_key(r);
    }
    
    // Lock cache node
    rv = ngx_http_file_cache_lock(r, c);
    
    if (rv == NGX_DECLINED) {
        // Cache miss
        return NGX_DECLINED;
    }
    
    if (rv == NGX_AGAIN) {
        // Waiting for lock
        return NGX_AGAIN;
    }
    
    if (rv == NGX_OK) {
        // Open cache file
        rc = ngx_http_file_cache_read(r, c);
        
        if (rc == NGX_OK) {
            // Cache hit
            return NGX_OK;
        }
        
        if (rc == NGX_HTTP_CACHE_STALE) {
            // Stale cache entry
            return NGX_HTTP_CACHE_STALE;
        }
    }
    
    return NGX_DECLINED;
}
```

---

## Module Development Deep Dive

### Complete Module Example
```c
// ngx_http_hello_module.c
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>

// Module configuration structure
typedef struct {
    ngx_flag_t  enable;
    ngx_str_t   message;
} ngx_http_hello_loc_conf_t;

// Function declarations
static ngx_int_t ngx_http_hello_handler(ngx_http_request_t *r);
static void *ngx_http_hello_create_loc_conf(ngx_conf_t *cf);
static char *ngx_http_hello_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child);
static char *ngx_http_hello(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);

// Module directives
static ngx_command_t ngx_http_hello_commands[] = {
    {
        ngx_string("hello"),
        NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS,
        ngx_http_hello,
        NGX_HTTP_LOC_CONF_OFFSET,
        0,
        NULL
    },
    {
        ngx_string("hello_message"),
        NGX_HTTP_LOC_CONF|NGX_CONF_TAKE1,
        ngx_conf_set_str_slot,
        NGX_HTTP_LOC_CONF_OFFSET,
        offsetof(ngx_http_hello_loc_conf_t, message),
        NULL
    },
    ngx_null_command
};

// HTTP module context
static ngx_http_module_t ngx_http_hello_module_ctx = {
    NULL,                                  /* preconfiguration */
    NULL,                                  /* postconfiguration */
    NULL,                                  /* create main configuration */
    NULL,                                  /* init main configuration */
    NULL,                                  /* create server configuration */
    NULL,                                  /* merge server configuration */
    ngx_http_hello_create_loc_conf,       /* create location configuration */
    ngx_http_hello_merge_loc_conf         /* merge location configuration */
};

// Module definition
ngx_module_t ngx_http_hello_module = {
    NGX_MODULE_V1,
    &ngx_http_hello_module_ctx,           /* module context */
    ngx_http_hello_commands,              /* module directives */
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

// Request handler
static ngx_int_t
ngx_http_hello_handler(ngx_http_request_t *r)
{
    ngx_int_t                     rc;
    ngx_buf_t                    *b;
    ngx_chain_t                   out;
    ngx_http_hello_loc_conf_t    *hlcf;
    
    // Get module configuration
    hlcf = ngx_http_get_module_loc_conf(r, ngx_http_hello_module);
    
    // Only allow GET and HEAD methods
    if (!(r->method & (NGX_HTTP_GET|NGX_HTTP_HEAD))) {
        return NGX_HTTP_NOT_ALLOWED;
    }
    
    // Discard request body
    rc = ngx_http_discard_request_body(r);
    if (rc != NGX_OK) {
        return rc;
    }
    
    // Set response headers
    r->headers_out.status = NGX_HTTP_OK;
    r->headers_out.content_type_len = sizeof("text/plain") - 1;
    ngx_str_set(&r->headers_out.content_type, "text/plain");
    r->headers_out.content_length_n = hlcf->message.len;
    
    // Handle HEAD request
    if (r->method == NGX_HTTP_HEAD) {
        rc = ngx_http_send_header(r);
        if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
            return rc;
        }
    }
    
    // Allocate buffer for response
    b = ngx_pcalloc(r->pool, sizeof(ngx_buf_t));
    if (b == NULL) {
        return NGX_HTTP_INTERNAL_SERVER_ERROR;
    }
    
    // Set buffer content
    b->pos = hlcf->message.data;
    b->last = hlcf->message.data + hlcf->message.len;
    b->memory = 1;
    b->last_buf = 1;
    
    // Set chain
    out.buf = b;
    out.next = NULL;
    
    // Send headers
    rc = ngx_http_send_header(r);
    if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
        return rc;
    }
    
    // Send body
    return ngx_http_output_filter(r, &out);
}

// Create location configuration
static void *
ngx_http_hello_create_loc_conf(ngx_conf_t *cf)
{
    ngx_http_hello_loc_conf_t  *conf;
    
    conf = ngx_pcalloc(cf->pool, sizeof(ngx_http_hello_loc_conf_t));
    if (conf == NULL) {
        return NULL;
    }
    
    conf->enable = NGX_CONF_UNSET;
    
    return conf;
}

// Merge location configuration
static char *
ngx_http_hello_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child)
{
    ngx_http_hello_loc_conf_t *prev = parent;
    ngx_http_hello_loc_conf_t *conf = child;
    
    ngx_conf_merge_value(conf->enable, prev->enable, 0);
    ngx_conf_merge_str_value(conf->message, prev->message, "Hello World!");
    
    return NGX_CONF_OK;
}

// Directive handler
static char *
ngx_http_hello(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    ngx_http_core_loc_conf_t  *clcf;
    
    clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
    clcf->handler = ngx_http_hello_handler;
    
    return NGX_CONF_OK;
}
```

### Module Compilation
```bash
# config file for dynamic module
ngx_addon_name=ngx_http_hello_module

if test -n "$ngx_module_link"; then
    ngx_module_type=HTTP
    ngx_module_name=ngx_http_hello_module
    ngx_module_srcs="$ngx_addon_dir/ngx_http_hello_module.c"
    
    . auto/module
else
    HTTP_MODULES="$HTTP_MODULES ngx_http_hello_module"
    NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_hello_module.c"
fi
```

```bash
# Compile as dynamic module
./configure --prefix=/etc/nginx \
            --add-dynamic-module=/path/to/ngx_http_hello_module
make
sudo make install
```

---

## Performance Analysis Tools

### Using perf for Profiling
```bash
# Record Nginx performance
sudo perf record -p $(pgrep -f 'nginx: worker process') -g -- sleep 60

# Generate report
sudo perf report

# Flame graph
git clone https://github.com/brendangregg/FlameGraph
sudo perf script | FlameGraph/stackcollapse-perf.pl | FlameGraph/flamegraph.pl > nginx.svg
```

### Memory Leak Detection
```bash
# Using Valgrind
valgrind --leak-check=full --show-leak-kinds=all \
         --log-file=valgrind.log \
         nginx -g 'daemon off;'
```

### SystemTap Tracing
```systemtap
#!/usr/bin/env stap
# nginx_trace.stp

probe process("/usr/sbin/nginx").function("ngx_http_process_request") {
    printf("Processing request: %s\n", user_string($r->uri.data));
}

probe process("/usr/sbin/nginx").function("ngx_http_finalize_request") {
    printf("Request completed with status: %d\n", $rc);
}
```

---

## Best Practices for Source Code Study

### 1. Start with Documentation
- Read Nginx development guide
- Study module documentation
- Review commit messages

### 2. Use Code Navigation Tools
```bash
# Generate ctags
ctags -R src/

# Or use cscope
find src/ -name "*.c" -o -name "*.h" > cscope.files
cscope -b -q -k
```

### 3. Debug with GDB
```bash
# Compile with debug symbols
./configure --with-debug
make

# Debug worker process
sudo gdb -p $(pgrep -f 'nginx: worker process')

# Set breakpoints
(gdb) break ngx_http_process_request
(gdb) continue
```

### 4. Study Execution Flow
- Follow request lifecycle
- Trace event handling
- Understand phase processing

### 5. Read Tests
- Study test cases in nginx-tests repository
- Write your own tests
- Use test frameworks

---

## Recommended Reading Order

1. **Core Infrastructure**
   - Memory pools (ngx_palloc.c)
   - Strings and arrays (ngx_string.c, ngx_array.c)
   - Configuration parsing (ngx_conf_file.c)

2. **Process Model**
   - Master process (ngx_process_cycle.c)
   - Worker initialization
   - Signal handling

3. **Event Mechanism**
   - Event module interface (ngx_event.h)
   - Epoll implementation (ngx_epoll_module.c)
   - Event processing loop

4. **HTTP Module**
   - Request structure (ngx_http_request.h)
   - Request processing (ngx_http_request.c)
   - Phase handling (ngx_http_core_module.c)

5. **Advanced Features**
   - Upstream module (ngx_http_upstream.c)
   - Cache implementation (ngx_http_file_cache.c)
   - Load balancing algorithms

---

## Contributing to Nginx

### Coding Style
- Follow Nginx coding conventions
- Use consistent indentation (4 spaces)
- Write clear comments
- Keep functions small and focused

### Submitting Patches
1. Create clear commit messages
2. Test thoroughly
3. Submit to nginx-devel mailing list
4. Address review feedback

### Testing
```bash
# Run Nginx test suite
git clone https://github.com/nginx/nginx-tests.git
cd nginx-tests
prove -v .
```

---

## Conclusion

Mastering Nginx source code requires:
- Deep understanding of C programming
- Knowledge of event-driven architecture
- Experience with network programming
- Patience and systematic study

Key takeaways:
1. Nginx's modular architecture enables extensibility
2. Event-driven model provides high performance
3. Memory pool management optimizes allocations
4. Process model ensures stability and scalability
5. Well-structured code facilitates understanding

---

## Additional Resources

### Official Documentation
- https://nginx.org/en/docs/dev/development_guide.html
- https://www.nginx.com/resources/wiki/

### Books
- "Nginx HTTP Server" by Martin Fjordvald
- "Nginx Essentials" by Valery Kholodkov

### Source Code Repositories
- Main repository: https://github.com/nginx/nginx
- Test suite: https://github.com/nginx/nginx-tests
- Third-party modules: https://www.nginx.com/resources/wiki/modules/

### Community
- Mailing list: nginx-devel@nginx.org
- IRC: #nginx on Freenode
- Forums: https://forum.nginx.org/

---

*End of Nginx Source Code Mastery Tutorial*