# Linux Kernel Mastery: Complete Guide from C Programming Perspective

## Table of Contents
1. [Linux Kernel Architecture](#linux-kernel-architecture)
2. [Kernel Source Code Structure](#kernel-source-code-structure)
3. [Process Management](#process-management)
4. [Memory Management](#memory-management)
5. [File System & VFS](#file-system--vfs)
6. [Device Drivers](#device-drivers)
7. [Linux Networking Stack](#linux-networking-stack)
8. [System Calls](#system-calls)
9. [Kernel Modules](#kernel-modules)
10. [Interrupt Handling](#interrupt-handling)
11. [Synchronization Mechanisms](#synchronization-mechanisms)
12. [Advanced Topics](#advanced-topics)

---

## 1. Linux Kernel Architecture

### Kernel Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    User Space Applications                   │
│         (bash, firefox, nginx, custom programs)              │
└─────────────────────────────────────────────────────────────┘
                            ↕ (System Calls)
┌─────────────────────────────────────────────────────────────┐
│                      Kernel Space                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              System Call Interface (SCI)              │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌─────────────┬──────────────┬──────────────┬──────────┐  │
│  │  Process    │   Memory     │ File System  │ Network  │  │
│  │  Scheduler  │  Management  │     VFS      │  Stack   │  │
│  └─────────────┴──────────────┴──────────────┴──────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │            Device Drivers & Modules                   │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │         Architecture Dependent Code (x86, ARM)        │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    Hardware Layer                            │
│         (CPU, RAM, Disk, Network Cards, etc.)               │
└─────────────────────────────────────────────────────────────┘
```

### Kernel Types

**Monolithic Kernel (Linux)**
- All kernel services run in kernel space
- Better performance due to direct function calls
- Larger memory footprint

**Microkernel**
- Minimal kernel with services in user space
- Better stability and security
- Performance overhead due to IPC

---

## 2. Kernel Source Code Structure

### Linux Kernel Directory Layout

```
linux/
├── arch/           # Architecture-specific code (x86, ARM, etc.)
├── block/          # Block device layer
├── crypto/         # Cryptographic API
├── drivers/        # Device drivers
├── fs/             # File systems (ext4, btrfs, proc, sysfs)
├── include/        # Header files
├── init/           # Kernel initialization code
├── ipc/            # Inter-process communication
├── kernel/         # Core kernel code (scheduler, process mgmt)
├── lib/            # Library routines (string operations, etc.)
├── mm/             # Memory management
├── net/            # Networking stack
├── scripts/        # Build scripts
├── security/       # Security modules (SELinux, AppArmor)
└── tools/          # Kernel development tools
```

---

## 3. Process Management

### Process Structure (task_struct)

```c
/*
 * task_struct - The core process descriptor
 * Location: include/linux/sched.h
 */
struct task_struct {
    volatile long state;        // Process state (TASK_RUNNING, etc.)
    void *stack;               // Kernel stack pointer
    
    /* Process identification */
    pid_t pid;                 // Process ID
    pid_t tgid;                // Thread group ID
    
    /* Process relationships */
    struct task_struct *parent;        // Parent process
    struct list_head children;         // List of children
    
    /* Scheduling information */
    int prio;                  // Dynamic priority
    unsigned int policy;       // Scheduling policy
    
    /* Memory management */
    struct mm_struct *mm;      // Memory descriptor
    
    /* File system info */
    struct files_struct *files; // Open file descriptors
};
```

### Process Creation with fork()

```c
/*
 * fork() - Create new process
 */
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid;
    
    printf("Before fork: PID = %d\n", getpid());
    
    /* Create new process */
    pid = fork();
    
    if (pid < 0) {
        perror("fork failed");
        return 1;
    } 
    else if (pid == 0) {
        /* Child process */
        printf("Child: PID = %d, Parent = %d\n", getpid(), getppid());
    } 
    else {
        /* Parent process */
        printf("Parent: PID = %d, Child = %d\n", getpid(), pid);
    }
    
    return 0;
}
```

---

## 4. Memory Management

### Virtual Memory Layout

```
┌─────────────────────────────────────────────────────────────┐
│  0xFFFFFFFF  ┌────────────────────────────────────┐         │
│              │       Kernel Space (1GB)           │         │
│  0xC0000000  ├────────────────────────────────────┤         │
│              │         Stack  (grows down)        │         │
│              │              ↓                     │         │
│              │          (unmapped)                │         │
│              │              ↑                     │         │
│              │         Heap (grows up)            │         │
│              ├────────────────────────────────────┤         │
│              │       BSS (uninitialized data)     │         │
│              ├────────────────────────────────────┤         │
│              │       Data (initialized data)      │         │
│              ├────────────────────────────────────┤         │
│  0x08048000  │       Text (program code)          │         │
│  0x00000000  └────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### Memory Allocation Examples

```c
/*
 * Kernel memory allocation
 */
#include <linux/slab.h>

void *ptr;

/* Allocate physically contiguous memory */
ptr = kmalloc(1024, GFP_KERNEL);
if (!ptr) {
    printk(KERN_ERR "kmalloc failed\n");
    return -ENOMEM;
}

/* Use memory */
memset(ptr, 0, 1024);

/* Free memory */
kfree(ptr);

/*
 * User space memory allocation
 */
#include <stdlib.h>

void *ptr = malloc(1024);
if (!ptr) {
    perror("malloc failed");
    return NULL;
}
memset(ptr, 0, 1024);
free(ptr);
```

---

## 5. File System & VFS

### VFS Core Structures

```c
/*
 * Superblock - Mounted filesystem
 */
struct super_block {
    dev_t s_dev;                    // Device identifier
    unsigned long s_blocksize;      // Block size
    struct dentry *s_root;          // Root dentry
};

/*
 * Inode - File or directory
 */
struct inode {
    umode_t i_mode;                 // File type and permissions
    kuid_t i_uid;                   // User ID
    kgid_t i_gid;                   // Group ID
    unsigned long i_ino;            // Inode number
    loff_t i_size;                  // File size
    struct timespec i_atime;        // Access time
    struct timespec i_mtime;        // Modification time
};

/*
 * File - Open file descriptor
 */
struct file {
    struct path f_path;             // Path (dentry + vfsmount)
    const struct file_operations *f_op;
    loff_t f_pos;                   // File position
};
```

---

## 6. Device Drivers

### Character Device Driver

```c
/*
 * Simple character device driver
 */
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>

#define DEVICE_NAME "mychardev"

static dev_t dev_num;
static struct cdev my_cdev;

/* Device open */
static int my_open(struct inode *inode, struct file *filp) {
    printk(KERN_INFO "Device opened\n");
    return 0;
}

/* Device read */
static ssize_t my_read(struct file *filp, char __user *buf,
                       size_t count, loff_t *f_pos) {
    printk(KERN_INFO "Device read\n");
    return 0;
}

/* Device write */
static ssize_t my_write(struct file *filp, const char __user *buf,
                        size_t count, loff_t *f_pos) {
    printk(KERN_INFO "Device write\n");
    return count;
}

/* Device close */
static int my_release(struct inode *inode, struct file *filp) {
    printk(KERN_INFO "Device closed\n");
    return 0;
}

/* File operations */
static struct file_operations my_fops = {
    .owner = THIS_MODULE,
    .open = my_open,
    .read = my_read,
    .write = my_write,
    .release = my_release,
};

/* Module initialization */
static int __init my_init(void) {
    int ret;
    
    /* Allocate device number */
    ret = alloc_chrdev_region(&dev_num, 0, 1, DEVICE_NAME);
    if (ret < 0) {
        printk(KERN_ERR "Failed to allocate device number\n");
        return ret;
    }
    
    /* Initialize and add cdev */
    cdev_init(&my_cdev, &my_fops);
    ret = cdev_add(&my_cdev, dev_num, 1);
    if (ret < 0) {
        unregister_chrdev_region(dev_num, 1);
        return ret;
    }
    
    printk(KERN_INFO "Device registered: MAJOR=%d, MINOR=%d\n",
           MAJOR(dev_num), MINOR(dev_num));
    
    return 0;
}

/* Module cleanup */
static void __exit my_exit(void) {
    cdev_del(&my_cdev);
    unregister_chrdev_region(dev_num, 1);
    printk(KERN_INFO "Device unregistered\n");
}

module_init(my_init);
module_exit(my_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Simple character device driver");
```

---

## 7. Linux Networking Stack

### Networking Stack Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              User Space Applications                         │
│         (curl, firefox, ssh, custom apps)                   │
└─────────────────────────────────────────────────────────────┘
                    ↕ (Socket API)
┌─────────────────────────────────────────────────────────────┐
│                  Socket Layer                                │
│         socket(), bind(), listen(), accept()                │
└─────────────────────────────────────────────────────────────┘
                    ↕
┌─────────────────────────────────────────────────────────────┐
│              Transport Layer                                 │
│         TCP (Transmission Control Protocol)                  │
│         UDP (User Datagram Protocol)                        │
└─────────────────────────────────────────────────────────────┘
                    ↕
┌─────────────────────────────────────────────────────────────┐
│              Network Layer                                   │
│         IP (Internet Protocol)                              │
│         Routing, Fragmentation                              │
└─────────────────────────────────────────────────────────────┘
                    ↕
┌─────────────────────────────────────────────────────────────┐
│              Data Link Layer                                 │
│         Ethernet, ARP, Network Device Drivers               │
└─────────────────────────────────────────────────────────────┘
                    ↕
┌─────────────────────────────────────────────────────────────┐
│              Physical Layer                                  │
│         Network Interface Card (NIC)                        │
└─────────────────────────────────────────────────────────────┘
```

### Socket Buffer (sk_buff)

```c
/*
 * sk_buff - Socket buffer structure
 * Location: include/linux/skbuff.h
 * 
 * This is the fundamental data structure for network packets
 */
struct sk_buff {
    /* Linked list pointers */
    struct sk_buff *next;           // Next buffer in list
    struct sk_buff *prev;           // Previous buffer in list
    
    /* Time stamp */
    ktime_t tstamp;                 // Packet timestamp
    
    /* Socket association */
    struct sock *sk;                // Owning socket
    
    /* Device information */
    struct net_device *dev;         // Device we arrived on/are leaving
    
    /* Protocol headers */
    union {
        struct tcphdr *th;          // TCP header
        struct udphdr *uh;          // UDP header
        struct icmphdr *icmph;      // ICMP header
        unsigned char *raw;         // Raw pointer
    } h;                            // Transport layer header
    
    union {
        struct iphdr *iph;          // IPv4 header
        struct ipv6hdr *ipv6h;      // IPv6 header
        unsigned char *raw;         // Raw pointer
    } nh;                           // Network layer header
    
    union {
        struct ethhdr *ethernet;    // Ethernet header
        unsigned char *raw;         // Raw pointer
    } mac;                          // Link layer header
    
    /* Data pointers */
    unsigned char *head;            // Start of allocated buffer
    unsigned char *data;            // Start of actual data
    unsigned char *tail;            // End of actual data
    unsigned char *end;             // End of allocated buffer
    
    /* Buffer size information */
    unsigned int len;               // Actual data length
    unsigned int data_len;          // Data length (for fragments)
    
    /* Protocol and checksum */
    __be16 protocol;                // Packet protocol
    __u8 ip_summed;                 // Checksum status
    __u32 priority;                 // Packet priority
    
    /* Routing information */
    struct dst_entry *dst;          // Destination cache entry
};

/*
 * SKB memory layout:
 * 
 * head                data                tail              end
 *  ↓                   ↓                   ↓                 ↓
 *  ┌──────────────────┬───────────────────┬────────────────┐
 *  │   Headroom       │   Actual Data     │   Tailroom     │
 *  │  (for headers)   │                   │  (for growth)  │
 *  └──────────────────┴───────────────────┴────────────────┘
 */

/*
 * Allocate sk_buff
 */
struct sk_buff *alloc_skb_example(void)
{
    struct sk_buff *skb;
    unsigned int size = 1500;  // Standard Ethernet MTU
    
    /* Allocate socket buffer */
    skb = alloc_skb(size, GFP_KERNEL);
    if (!skb) {
        printk(KERN_ERR "Failed to allocate skb\n");
        return NULL;
    }
    
    /* Reserve space for headers */
    skb_reserve(skb, 64);  // Reserve 64 bytes for headers
    
    /* Put data into buffer */
    unsigned char *data = skb_put(skb, 100);  // Add 100 bytes of data
    memcpy(data, "Hello, Network!", 15);
    
    /* Free when done */
    kfree_skb(skb);
    
    return skb;
}
```

### TCP/IP Socket Programming

```c
/*
 * TCP Server Example - User Space
 */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

void tcp_server_example(void)
{
    int server_fd, client_fd;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_len = sizeof(client_addr);
    char buffer[BUFFER_SIZE];
    ssize_t n;
    
    /* Create socket - AF_INET (IPv4), SOCK_STREAM (TCP) */
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd < 0) {
        perror("socket failed");
        return;
    }
    
    /* Set socket options - allow address reuse */
    int opt = 1;
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, 
                   &opt, sizeof(opt)) < 0) {
        perror("setsockopt failed");
        close(server_fd);
        return;
    }
    
    /* Prepare server address structure */
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;           // IPv4
    server_addr.sin_addr.s_addr = INADDR_ANY;   // Any interface
    server_addr.sin_port = htons(PORT);         // Port (host to network byte order)
    
    /* Bind socket to address */
    if (bind(server_fd, (struct sockaddr *)&server_addr, 
             sizeof(server_addr)) < 0) {
        perror("bind failed");
        close(server_fd);
        return;
    }
    
    /* Listen for connections (backlog = 5) */
    if (listen(server_fd, 5) < 0) {
        perror("listen failed");
        close(server_fd);
        return;
    }
    
    printf("Server listening on port %d...\n", PORT);
    
    /* Accept client connection */
    client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &client_len);
    if (client_fd < 0) {
        perror("accept failed");
        close(server_fd);
        return;
    }
    
    printf("Client connected: %s:%d\n",
           inet_ntoa(client_addr.sin_addr),
           ntohs(client_addr.sin_port));
    
    /* Receive data from client */
    n = recv(client_fd, buffer, BUFFER_SIZE - 1, 0);
    if (n > 0) {
        buffer[n] = '\0';
        printf("Received: %s\n", buffer);
        
        /* Send response */
        const char *response = "Hello from server!";
        send(client_fd, response, strlen(response), 0);
    }
    
    /* Cleanup */
    close(client_fd);
    close(server_fd);
}

/*
 * TCP Client Example
 */
void tcp_client_example(void)
{
    int sock_fd;
    struct sockaddr_in server_addr;
    char buffer[BUFFER_SIZE];
    const char *message = "Hello from client!";
    ssize_t n;
    
    /* Create socket */
    sock_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (sock_fd < 0) {
        perror("socket failed");
        return;
    }
    
    /* Prepare server address */
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    
    /* Convert IP address from text to binary */
    if (inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr) <= 0) {
        perror("inet_pton failed");
        close(sock_fd);
        return;
    }
    
    /* Connect to server */
    if (connect(sock_fd, (struct sockaddr *)&server_addr, 
                sizeof(server_addr)) < 0) {
        perror("connect failed");
        close(sock_fd);
        return;
    }
    
    printf("Connected to server\n");
    
    /* Send message */
    send(sock_fd, message, strlen(message), 0);
    printf("Message sent: %s\n", message);
    
    /* Receive response */
    n = recv(sock_fd, buffer, BUFFER_SIZE - 1, 0);
    if (n > 0) {
        buffer[n] = '\0';
        printf("Received: %s\n", buffer);
    }
    
    /* Cleanup */
    close(sock_fd);
}
```

### Kernel Network Device

```c
/*
 * Network device structure
 * Location: include/linux/netdevice.h
 */
struct net_device {
    char name[IFNAMSIZ];            // Interface name (eth0, wlan0, etc.)
    
    /* Device state */
    unsigned long state;            // Device state flags
    
    /* Hardware information */
    unsigned char dev_addr[MAX_ADDR_LEN];  // Hardware address (MAC)
    unsigned char broadcast[MAX_ADDR_LEN]; // Broadcast address
    
    /* MTU (Maximum Transmission Unit) */
    unsigned int mtu;               // Maximum packet size
    unsigned short type;            // Hardware type (Ethernet, etc.)
    unsigned short hard_header_len; // Hardware header length
    
    /* Network device operations */
    const struct net_device_ops *netdev_ops;
    
    /* Statistics */
    struct net_device_stats stats;  // Device statistics
    
    /* Transmit queue */
    struct netdev_queue *_tx;       // TX queue
    unsigned int num_tx_queues;     // Number of TX queues
    
    /* Device features */
    netdev_features_t features;     // Device capabilities
    netdev_features_t hw_features;  // Hardware features
    
    /* Private data */
    void *priv;                     // Device-specific data
};

/*
 * Network device operations
 */
struct net_device_ops {
    int (*ndo_init)(struct net_device *dev);
    void (*ndo_uninit)(struct net_device *dev);
    
    int (*ndo_open)(struct net_device *dev);
    int (*ndo_stop)(struct net_device *dev);
    
    netdev_tx_t (*ndo_start_xmit)(struct sk_buff *skb,
                                  struct net_device *dev);
    
    void (*ndo_set_rx_mode)(struct net_device *dev);
    int (*ndo_set_mac_address)(struct net_device *dev, void *addr);
    int (*ndo_validate_addr)(struct net_device *dev);
    
    int (*ndo_do_ioctl)(struct net_device *dev,
                       struct ifreq *ifr, int cmd);
    
    struct net_device_stats *(*ndo_get_stats)(struct net_device *dev);
};
```

### Netfilter/iptables Hook

```c
/*
 * Netfilter hook - Packet filtering framework
 * Location: include/linux/netfilter.h
 */

/* Netfilter hook points */
enum nf_inet_hooks {
    NF_INET_PRE_ROUTING = 0,    // Before routing decision
    NF_INET_LOCAL_IN,           // Destined for local system
    NF_INET_FORWARD,            // Forwarded packets
    NF_INET_LOCAL_OUT,          // Originated from local system
    NF_INET_POST_ROUTING,       // After routing decision
    NF_INET_NUMHOOKS
};

/*
 * Netfilter hook operations structure
 */
struct nf_hook_ops {
    struct list_head list;
    
    /* Hook function */
    nf_hookfn *hook;
    
    /* Protocol family (PF_INET for IPv4) */
    u_int8_t pf;
    
    /* Hook number */
    unsigned int hooknum;
    
    /* Priority */
    int priority;
};

/*
 * Netfilter hook return values
 */
#define NF_DROP 0       // Drop the packet
#define NF_ACCEPT 1     // Accept the packet
#define NF_STOLEN 2     // Packet stolen (don't continue)
#define NF_QUEUE 3      // Queue packet to userspace
#define NF_REPEAT 4     // Call hook again

/*
 * Example: Simple packet filter
 */
static unsigned int my_hook_func(void *priv,
                                 struct sk_buff *skb,
                                 const struct nf_hook_state *state)
{
    struct iphdr *iph;
    struct tcphdr *tcph;
    
    if (!skb)
        return NF_ACCEPT;
    
    /* Get IP header */
    iph = ip_hdr(skb);
    
    /* Check if TCP packet */
    if (iph->protocol == IPPROTO_TCP) {
        /* Get TCP header */
        tcph = tcp_hdr(skb);
        
        /* Block packets to port 80 (HTTP) */
        if (ntohs(tcph->dest) == 80) {
            printk(KERN_INFO "Blocking HTTP packet\n");
            return NF_DROP;
        }
    }
    
    return NF_ACCEPT;
}

/*
 * Register netfilter hook
 */
static struct nf_hook_ops my_nfho = {
    .hook = my_hook_func,
    .pf = PF_INET,                      // IPv4
    .hooknum = NF_INET_PRE_ROUTING,     // Before routing
    .priority = NF_IP_PRI_FIRST,        // Highest priority
};

static int __init my_init(void)
{
    /* Register hook */
    nf_register_net_hook(&init_net, &my_nfho);
    printk(KERN_INFO "Netfilter hook registered\n");
    return 0;
}

static void __exit my_exit(void)
{
    /* Unregister hook */
    nf_unregister_net_hook(&init_net, &my_nfho);
    printk(KERN_INFO "Netfilter hook unregistered\n");
}
```

### Network Packet Flow

```c
/*
 * Packet Receive Flow:
 * 
 * 1. Hardware receives packet → DMA to memory
 * 2. Interrupt handler called
 * 3. NAPI (New API) polling scheduled
 * 4. Driver allocates sk_buff and copies data
 * 5. netif_receive_skb() called
 * 6. Netfilter PRE_ROUTING hook
 * 7. Routing decision (local delivery or forward)
 * 8. If local: Netfilter LOCAL_IN hook
 * 9. Transport layer (TCP/UDP) processing
 * 10. Socket buffer queued to application
 */

/*
 * Simplified packet receive function
 */
int netif_receive_skb(struct sk_buff *skb)
{
    struct net_device *dev = skb->dev;
    struct packet_type *ptype;
    
    /* Update statistics */
    dev->stats.rx_packets++;
    dev->stats.rx_bytes += skb->len;
    
    /* Get packet type (IP, ARP, etc.) */
    skb->protocol = eth_type_trans(skb, dev);
    
    /* Netfilter hook - PRE_ROUTING */
    if (NF_HOOK(NFPROTO_IPV4, NF_INET_PRE_ROUTING,
                skb, dev, NULL, ip_rcv) != NF_ACCEPT) {
        kfree_skb(skb);
        return NET_RX_DROP;
    }
    
    /* Deliver to protocol handler (IP) */
    list_for_each_entry_rcu(ptype, &ptype_base[ntohs(skb->protocol)], list) {
        if (ptype->type == skb->protocol) {
            return ptype->func(skb, dev, ptype, dev);
        }
    }
    
    kfree_skb(skb);
    return NET_RX_DROP;
}
```

---

## 8. System Calls

### System Call Mechanism

```
┌─────────────────────────────────────────────────────────────┐
│                  User Space                                  │
│                                                              │
│   Application calls: read(fd, buf, size);                   │
│                           ↓                                  │
│   C Library (glibc): Wrapper function                       │
│                           ↓                                  │
│   Invoke syscall: syscall(__NR_read, fd, buf, size);       │
│                           ↓                                  │
├───────────────────────────────────────────────────────────────┤
│   CPU executes: int 0x80 (x86) or syscall (x86_64)          │
├───────────────────────────────────────────────────────────────┤
│                  Kernel Space                                │
│                           ↓                                  │
│   System call handler: system_call()                        │
│                           ↓                                  │
│   System call table: sys_call_table[__NR_read]             │
│                           ↓                                  │
│   Kernel function: sys_read(fd, buf, size)                 │
│                           ↓                                  │
│   VFS layer: vfs_read()                                     │
│                           ↓                                  │
│   File system specific: ext4_file_read()                   │
│                           ↓                                  │
│   Return to user space                                      │
└─────────────────────────────────────────────────────────────┘
```

### System Call Definition

```c
/*
 * System call definition macro
 * Location: include/linux/syscalls.h
 */

/*
 * SYSCALL_DEFINE macros for defining system calls
 * The number (0-6) indicates number of arguments
 */

/* System call with no arguments */
SYSCALL_DEFINE0(getpid)
{
    return task_tgid_vnr(current);  // current = current process
}

/* System call with one argument */
SYSCALL_DEFINE1(close, unsigned int, fd)
{
    int retval;
    struct file *filp;
    
    /* Get file structure from file descriptor */
    filp = fget(fd);
    if (!filp)
        return -EBADF;
    
    /* Close the file */
    retval = filp_close(filp, current->files);
    
    return retval;
}

/* System call with three arguments */
SYSCALL_DEFINE3(read, unsigned int, fd,
                char __user *, buf, size_t, count)
{
    struct fd f;
    ssize_t ret = -EBADF;
    
    /* Get file descriptor */
    f = fdget_pos(fd);
    if (f.file) {
        loff_t pos = file_pos_read(f.file);
        
        /* Perform read operation */
        ret = vfs_read(f.file, buf, count, &pos);
        if (ret >= 0)
            file_pos_write(f.file, pos);
        
        fdput_pos(f);
    }
    
    return ret;
}

/* System call with six arguments */
SYSCALL_DEFINE6(mmap, unsigned long, addr, unsigned long, len,
                unsigned long, prot, unsigned long, flags,
                unsigned long, fd, unsigned long, offset)
{
    return sys_mmap_pgoff(addr, len, prot, flags, fd, offset >> PAGE_SHIFT);
}
```

### System Call Table

```c
/*
 * System call table
 * Location: arch/x86/entry/syscalls/syscall_64.tbl (x86_64)
 */

/*
 * Format: <number> <abi> <name> <entry point>
 * 
 * 0    common  read            sys_read
 * 1    common  write           sys_write
 * 2    common  open            sys_open
 * 3    common  close           sys_close
 * 4    common  stat            sys_newstat
 * 5    common  fstat           sys_newfstat
 * ...
 */

/*
 * System call numbers (x86_64)
 * Location: arch/x86/include/generated/uapi/asm/unistd_64.h
 */
#define __NR_read 0
#define __NR_write 1
#define __NR_open 2
#define __NR_close 3
#define __NR_stat 4
#define __NR_fstat 5
#define __NR_fork 57
#define __NR_execve 59
#define __NR_exit 60

/*
 * System call table array
 */
const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = {
    [0] = sys_read,
    [1] = sys_write,
    [2] = sys_open,
    [3] = sys_close,
    // ... more entries
};
```

### Making System Calls from User Space

```c
/*
 * Direct system call invocation
 */
#include <unistd.h>
#include <sys/syscall.h>
#include <stdio.h>

int main() {
    pid_t pid;
    
    /* Method 1: Using wrapper function */
    pid = getpid();
    printf("PID (wrapper): %d\n", pid);
    
    /* Method 2: Using syscall() directly */
    pid = syscall(SYS_getpid);
    printf("PID (syscall): %d\n", pid);
    
    /* Method 3: Custom syscall (if added to kernel) */
    long result = syscall(548);  // Custom syscall number
    printf("Custom syscall result: %ld\n", result);
    
    return 0;
}

/*
 * System call with arguments
 */
void syscall_with_args_example() {
    int fd;
    char buffer[100];
    ssize_t bytes_read;
    
    /* open() system call */
    fd = syscall(SYS_open, "/tmp/test.txt", O_RDONLY);
    if (fd < 0) {
        perror("open failed");
        return;
    }
    
    /* read() system call */
    bytes_read = syscall(SYS_read, fd, buffer, sizeof(buffer) - 1);
    if (bytes_read > 0) {
        buffer[bytes_read] = '\0';
        printf("Read: %s\n", buffer);
    }
    
    /* close() system call */
    syscall(SYS_close, fd);
}
```

### Adding a Custom System Call

```c
/*
 * Step 1: Define system call in kernel
 * Location: kernel/sys.c
 */
#include <linux/syscalls.h>
#include <linux/kernel.h>

SYSCALL_DEFINE2(my_custom_syscall, int, arg1, int, arg2)
{
    printk(KERN_INFO "Custom syscall called: arg1=%d, arg2=%d\n", 
           arg1, arg2);
    
    return arg1 + arg2;
}

/*
 * Step 2: Add to system call table
 * Location: arch/x86/entry/syscalls/syscall_64.tbl
 * 
 * 548  64  my_custom_syscall  sys_my_custom_syscall
 */

/*
 * Step 3: Add system call number
 * Location: include/uapi/asm-generic/unistd.h
 * 
 * #define __NR_my_custom_syscall 548
 */

/*
 * Step 4: Rebuild and install kernel
 * 
 * make -j$(nproc)
 * sudo make modules_install
 * sudo make install
 * sudo reboot
 */

/*
 * Step 5: Use from user space
 */
#include <unistd.h>
#include <sys/syscall.h>
#include <stdio.h>

#define __NR_my_custom_syscall 548

int main() {
    long result;
    
    result = syscall(__NR_my_custom_syscall, 10, 20);
    printf("Result: %ld\n", result);  // Should print 30
    
    return 0;
}
```

---

## 9. Kernel Modules

### Kernel Module Basics

```c
/*
 * Simple kernel module example
 * Filename: hello_module.c
 */
#include <linux/module.h>     // Core module support
#include <linux/kernel.h>     // Kernel types and macros
#include <linux/init.h>       // Initialization macros

/*
 * Module initialization function
 * Called when module is loaded (insmod)
 */
static int __init hello_init(void)
{
    printk(KERN_INFO "Hello, Kernel World!\n");
    printk(KERN_INFO "Module loaded successfully\n");
    return 0;  // Return 0 on success
}

/*
 * Module cleanup function
 * Called when module is unloaded (rmmod)
 */
static void __exit hello_exit(void)
{
    printk(KERN_INFO "Goodbye, Kernel World!\n");
    printk(KERN_INFO "Module unloaded\n");
}

/* Register init and exit functions */
module_init(hello_init);
module_exit(hello_exit);

/* Module metadata */
MODULE_LICENSE("GPL");              // License type
MODULE_AUTHOR("Your Name");         // Author
MODULE_DESCRIPTION("Simple Hello World Module");  // Description
MODULE_VERSION("1.0");              // Version

/*
 * Makefile for building the module
 * Filename: Makefile
 */
/*
obj-m += hello_module.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
*/

/*
 * Build and load module:
 * 
 * $ make                        # Build module
 * $ sudo insmod hello_module.ko # Load module
 * $ lsmod | grep hello          # List loaded modules
 * $ dmesg | tail                # View kernel messages
 * $ sudo rmmod hello_module     # Unload module
 * $ dmesg | tail                # View exit messages
 */
```

### Module with Parameters

```c
/*
 * Kernel module with parameters
 */
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/moduleparam.h>

/* Module parameters */
static int debug_level = 0;
static char *device_name = "default_device";
static int irq_number = 7;
static int port_array[4] = {0, 0, 0, 0};
static int port_count = 0;

/* Declare parameters */
module_param(debug_level, int, S_IRUGO);
MODULE_PARM_DESC(debug_level, "Debug level (0-5)");

module_param(device_name, charp, S_IRUGO);
MODULE_PARM_DESC(device_name, "Name of the device");

module_param(irq_number, int, S_IRUGO | S_IWUSR);
MODULE_PARM_DESC(irq_number, "IRQ number for device");

module_param_array(port_array, int, &port_count, S_IRUGO);
MODULE_PARM_DESC(port_array, "Array of port numbers");

static int __init param_init(void)
{
    int i;
    
    printk(KERN_INFO "Module loaded with parameters:\n");
    printk(KERN_INFO "Debug level: %d\n", debug_level);
    printk(KERN_INFO "Device name: %s\n", device_name);
    printk(KERN_INFO "IRQ number: %d\n", irq_number);
    printk(KERN_INFO "Port count: %d\n", port_count);
    
    for (i = 0; i < port_count; i++) {
        printk(KERN_INFO "Port[%d]: %d\n", i, port_array[i]);
    }
    
    return 0;
}

static void __exit param_exit(void)
{
    printk(KERN_INFO "Module with parameters unloaded\n");
}

module_init(param_init);
module_exit(param_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Module with parameters");

/*
 * Load module with parameters:
 * 
 * sudo insmod param_module.ko debug_level=3 device_name="eth0" \
 *      irq_number=10 port_array=80,443,8080,3000
 * 
 * View parameters:
 * cat /sys/module/param_module/parameters/debug_level
 * 
 * Change writable parameter:
 * echo 5 | sudo tee /sys/module/param_module/parameters/irq_number
 */
```

### Module Dependencies

```c
/*
 * Module with symbol export/import
 */

/* ========== PROVIDER MODULE ========== */
/* Filename: symbol_provider.c */
#include <linux/module.h>
#include <linux/kernel.h>

int shared_variable = 42;
EXPORT_SYMBOL(shared_variable);  // Export symbol for other modules

int shared_function(int x, int y)
{
    printk(KERN_INFO "shared_function called: %d + %d = %d\n", 
           x, y, x + y);
    return x + y;
}
EXPORT_SYMBOL(shared_function);  // Export function

static int __init provider_init(void)
{
    printk(KERN_INFO "Symbol provider module loaded\n");
    return 0;
}

static void __exit provider_exit(void)
{
    printk(KERN_INFO "Symbol provider module unloaded\n");
}

module_init(provider_init);
module_exit(provider_exit);

MODULE_LICENSE("GPL");

/* ========== CONSUMER MODULE ========== */
/* Filename: symbol_consumer.c */
#include <linux/module.h>
#include <linux/kernel.h>

/* Declare external symbols */
extern int shared_variable;
extern int shared_function(int x, int y);

static int __init consumer_init(void)
{
    int result;
    
    printk(KERN_INFO "Symbol consumer module loaded\n");
    printk(KERN_INFO "Shared variable: %d\n", shared_variable);
    
    result = shared_function(10, 20);
    printk(KERN_INFO "Function result: %d\n", result);
    
    /* Modify shared variable */
    shared_variable = 100;
    printk(KERN_INFO "Modified shared variable: %d\n", shared_variable);
    
    return 0;
}

static void __exit consumer_exit(void)
{
    printk(KERN_INFO "Symbol consumer module unloaded\n");
}

module_init(consumer_init);
module_exit(consumer_exit);

MODULE_LICENSE("GPL");

/*
 * Load modules in order:
 * 
 * sudo insmod symbol_provider.ko
 * sudo insmod symbol_consumer.ko
 * 
 * Unload in reverse order:
 * 
 * sudo rmmod symbol_consumer
 * sudo rmmod symbol_provider
 */
```

---

## 10. Interrupt Handling

### Interrupt Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Hardware Device                           │
│         (Network Card, Disk Controller, etc.)               │
└─────────────────────────────────────────────────────────────┘
                           ↓ (IRQ Signal)
┌─────────────────────────────────────────────────────────────┐
│              Interrupt Controller (PIC/APIC)                 │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                    CPU                                       │
│   1. Save context (registers, flags)                        │
│   2. Lookup interrupt vector                                │
│   3. Call interrupt handler                                 │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│              Interrupt Handler (ISR)                         │
│   1. Top Half - Quick processing                            │
│      - Acknowledge interrupt                                │
│      - Read/Write hardware registers                        │
│      - Schedule bottom half                                 │
│                                                              │
│   2. Bottom Half - Deferred work                            │
│      - Softirq / Tasklet / Workqueue                        │
│      - Process data                                         │
│      - Wake up waiting processes                            │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│              Resume interrupted task                         │
└─────────────────────────────────────────────────────────────┘
```

### Interrupt Handler Registration

```c
/*
 * Interrupt handler example
 */
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/interrupt.h>
#include <linux/workqueue.h>

#define IRQ_NUMBER 7  // Example IRQ number

static int dev_id = 123;  // Device ID for shared interrupts

/*
 * Top half - Interrupt Service Routine (ISR)
 * MUST be fast! No sleeping allowed!
 */
static irqreturn_t my_interrupt_handler(int irq, void *dev_id)
{
    /* Quick hardware interaction */
    printk(KERN_INFO "Interrupt %d received\n", irq);
    
    /* Read device status register (example) */
    // status = readl(device_base + STATUS_REG);
    
    /* Clear interrupt flag (example) */
    // writel(CLEAR_INT, device_base + CONTROL_REG);
    
    /* Schedule bottom half for deferred work */
    // schedule_work(&my_work);
    
    /* Return IRQ_HANDLED if we handled it, IRQ_NONE if not */
    return IRQ_HANDLED;
}

/*
 * Register interrupt handler
 */
static int __init irq_init(void)
{
    int result;
    
    /* Request IRQ line */
    result = request_irq(IRQ_NUMBER,                // IRQ number
                        my_interrupt_handler,       // Handler function
                        IRQF_SHARED,               // Flags (shared IRQ)
                        "my_device",               // Device name
                        &dev_id);                  // Device ID
    
    if (result) {
        printk(KERN_ERR "Failed to request IRQ %d: %d\n", 
               IRQ_NUMBER, result);
        return result;
    }
    
    printk(KERN_INFO "IRQ %d registered successfully\n", IRQ_NUMBER);
    return 0;
}

/*
 * Unregister interrupt handler
 */
static void __exit irq_exit(void)
{
    /* Free IRQ line */
    free_irq(IRQ_NUMBER, &dev_id);
    printk(KERN_INFO "IRQ %d freed\n", IRQ_NUMBER);
}

module_init(irq_init);
module_exit(irq_exit);

MODULE_LICENSE("GPL");

/*
 * IRQ flags:
 * 
 * IRQF_SHARED      - IRQ can be shared with other devices
 * IRQF_DISABLED    - Keep interrupts disabled during handler
 * IRQF_TRIGGER_RISING   - Trigger on rising edge
 * IRQF_TRIGGER_FALLING  - Trigger on falling edge
 * IRQF_TRIGGER_HIGH     - Trigger on high level
 * IRQF_TRIGGER_LOW      - Trigger on low level
 */
```

### Bottom Half - Tasklets

```c
/*
 * Tasklet - Deferred work mechanism
 */
#include <linux/interrupt.h>

/* Tasklet function - runs in interrupt context (cannot sleep) */
void my_tasklet_function(unsigned long data)
{
    printk(KERN_INFO "Tasklet executed with data: %lu\n", data);
    
    /* Process interrupt-related work here */
    /* Still cannot sleep, but not as time-critical as top half */
}

/* Declare and initialize tasklet */
DECLARE_TASKLET(my_tasklet, my_tasklet_function, 0);

/* Alternative: Dynamic initialization */
struct tasklet_struct my_dynamic_tasklet;

static irqreturn_t my_irq_handler(int irq, void *dev_id)
{
    /* Top half - quick work */
    printk(KERN_INFO "Interrupt received, scheduling tasklet\n");
    
    /* Schedule tasklet for execution */
    tasklet_schedule(&my_tasklet);
    
    return IRQ_HANDLED;
}

static int __init tasklet_init(void)
{
    /* Dynamic tasklet initialization */
    tasklet_init(&my_dynamic_tasklet, my_tasklet_function, 123);
    
    /* Register interrupt with tasklet scheduling */
    request_irq(IRQ_NUMBER, my_irq_handler, IRQF_SHARED, 
                "my_device", &dev_id);
    
    return 0;
}

static void __exit tasklet_exit(void)
{
    /* Kill pending tasklets */
    tasklet_kill(&my_tasklet);
    tasklet_kill(&my_dynamic_tasklet);
    
    free_irq(IRQ_NUMBER, &dev_id);
}
```

### Bottom Half - Work Queues

```c
/*
 * Work queue - Deferred work in process context (can sleep)
 */
#include <linux/workqueue.h>

/* Work function - runs in process context (CAN sleep) */
static void my_work_function(struct work_struct *work)
{
    printk(KERN_INFO "Work queue function executing\n");
    
    /* Can perform blocking operations here */
    msleep(100);  // This is OK in work queues!
    
    /* Process data, access user space, etc. */
    printk(KERN_INFO "Work completed\n");
}

/* Declare and initialize work */
DECLARE_WORK(my_work, my_work_function);

/* Alternative: Dynamic initialization */
struct work_struct my_dynamic_work;

static irqreturn_t my_work_irq_handler(int irq, void *dev_id)
{
    /* Top half */
    printk(KERN_INFO "Interrupt received, scheduling work\n");
    
    /* Schedule work for execution */
    schedule_work(&my_work);
    
    return IRQ_HANDLED;
}

/*
 * Delayed work - Execute after a delay
 */
static void my_delayed_work_function(struct work_struct *work)
{
    printk(KERN_INFO "Delayed work executed\n");
}

DECLARE_DELAYED_WORK(my_delayed_work, my_delayed_work_function);

static int __init workqueue_init(void)
{
    /* Schedule work immediately */
    schedule_work(&my_work);
    
    /* Schedule delayed work (1 second delay) */
    schedule_delayed_work(&my_delayed_work, HZ);  // HZ = 1 second
    
    /* Schedule on specific CPU */
    schedule_work_on(0, &my_work);  // CPU 0
    
    return 0;
}

static void __exit workqueue_exit(void)
{
    /* Cancel pending work */
    cancel_work_sync(&my_work);
    cancel_delayed_work_sync(&my_delayed_work);
}

/*
 * Custom work queue
 */
static struct workqueue_struct *my_wq;

static int __init custom_wq_init(void)
{
    /* Create custom work queue */
    my_wq = create_workqueue("my_queue");
    if (!my_wq) {
        printk(KERN_ERR "Failed to create work queue\n");
        return -ENOMEM;
    }
    
    /* Queue work to custom queue */
    queue_work(my_wq, &my_work);
    
    /* Queue delayed work to custom queue */
    queue_delayed_work(my_wq, &my_delayed_work, HZ * 2);
    
    return 0;
}

static void __exit custom_wq_exit(void)
{
    /* Flush and destroy work queue */
    flush_workqueue(my_wq);
    destroy_workqueue(my_wq);
}
```

---

## 11. Synchronization Mechanisms

### Synchronization Overview

```
Synchronization Primitives:

1. Atomic Operations    - CPU-level atomic ops (no locks)
2. Spinlocks           - Busy-wait locks (short critical sections)
3. Semaphores          - Sleep-wait locks (long operations)
4. Mutex               - Mutual exclusion (one owner)
5. RCU                 - Read-Copy-Update (mostly reads)
6. Sequential Locks    - Reader-writer with sequence counters
```

### Atomic Operations

```c
/*
 * Atomic operations - Guaranteed atomicity at CPU level
 */
#include <linux/atomic.h>

/* Atomic integer */
atomic_t counter = ATOMIC_INIT(0);

void atomic_operations_example(void)
{
    int value;
    
    /* Read atomic variable */
    value = atomic_read(&counter);
    printk(KERN_INFO "Counter: %d\n", value);
    
    /* Set atomic variable */
    atomic_set(&counter, 10);
    
    /* Increment/decrement */
    atomic_inc(&counter);     // counter++
    atomic_dec(&counter);     // counter--
    
    /* Add/subtract */
    atomic_add(5, &counter);  // counter += 5
    atomic_sub(3, &counter);  // counter -= 3
    
    /* Increment and test */
    if (atomic_inc_and_test(&counter)) {
        printk(KERN_INFO "Counter reached zero\n");
    }
    
    /* Decrement and test */
    if (atomic_dec_and_test(&counter)) {
        printk(KERN_INFO "Counter reached zero\n");
    }
    
    /* Compare and swap */
    int old_val = 10;
    int new_val = 20;
    if (atomic_cmpxchg(&counter, old_val, new_val) == old_val) {
        printk(KERN_INFO "Swap successful\n");
    }
}

/*
 * Atomic bit operations
 */
void atomic_bit_operations(void)
{
    unsigned long flags = 0;
    
    /* Set bit */
    set_bit(0, &flags);           // flags |= (1 << 0)
    
    /* Clear bit */
    clear_bit(1, &flags);         // flags &= ~(1 << 1)
    
    /* Toggle bit */
    change_bit(2, &flags);        // flags ^= (1 << 2)
    
    /* Test bit */
    if (test_bit(0, &flags)) {
        printk(KERN_INFO "Bit 0 is set\n");
    }
    
    /* Test and set */
    if (test_and_set_bit(3, &flags)) {
        printk(KERN_INFO "Bit 3 was already set\n");
    }
    
    /* Test and clear */
    if (test_and_clear_bit(4, &flags)) {
        printk(KERN_INFO "Bit 4 was set, now cleared\n");
    }
}
```

### Spinlocks

```c
/*
 * Spinlock - Busy-wait lock (use for SHORT critical sections)
 */
#include <linux/spinlock.h>

/* Define and initialize spinlock */
DEFINE_SPINLOCK(my_spinlock);

/* Alternative: Dynamic initialization */
spinlock_t my_dynamic_spinlock;
spin_lock_init(&my_dynamic_spinlock);

void spinlock_example(void)
{
    unsigned long flags;
    
    /* Basic spinlock (interrupts enabled) */
    spin_lock(&my_spinlock);
    /* Critical section - keep SHORT! */
    // shared_data++;
    spin_unlock(&my_spinlock);
    
    /* Spinlock with interrupt disable (safer in interrupt context) */
    spin_lock_irqsave(&my_spinlock, flags);
    /* Critical section */
    // shared_data++;
    spin_unlock_irqrestore(&my_spinlock, flags);
    
    /* Try to acquire lock (non-blocking) */
    if (spin_trylock(&my_spinlock)) {
        /* Got the lock */
        // shared_data++;
        spin_unlock(&my_spinlock);
    } else {
        /* Lock is busy */
        printk(KERN_INFO "Lock is busy\n");
    }
}

/*
 * Reader-Writer Spinlock
 */
DEFINE_RWLOCK(my_rwlock);

void rwlock_example(void)
{
    unsigned long flags;
    
    /* Reader lock (multiple readers allowed) */
    read_lock(&my_rwlock);
    /* Read shared data */
    // value = shared_data;
    read_unlock(&my_rwlock);
    
    /* Writer lock (exclusive access) */
    write_lock_irqsave(&my_rwlock, flags);
    /* Modify shared data */
    // shared_data = new_value;
    write_unlock_irqrestore(&my_rwlock, flags);
}
```

### Mutexes

```c
/*
 * Mutex - Mutual exclusion lock (can sleep)
 * Use for LONGER critical sections
 */
#include <linux/mutex.h>

/* Define and initialize mutex */
DEFINE_MUTEX(my_mutex);

/* Alternative: Dynamic initialization */
struct mutex my_dynamic_mutex;
mutex_init(&my_dynamic_mutex);

void mutex_example(void)
{
    /* Lock mutex (sleeps if unavailable) */
    mutex_lock(&my_mutex);
    /* Critical section - can sleep here */
    // perform_long_operation();
    mutex_unlock(&my_mutex);
    
    /* Interruptible lock (can be interrupted by signals) */
    if (mutex_lock_interruptible(&my_mutex)) {
        /* Interrupted by signal */
        return -ERESTARTSYS;
    }
    /* Critical section */
    mutex_unlock(&my_mutex);
    
    /* Try to lock (non-blocking) */
    if (mutex_trylock(&my_mutex)) {
        /* Got the lock */
        // critical_section();
        mutex_unlock(&my_mutex);
    } else {
        /* Lock is busy */
        printk(KERN_INFO "Mutex is locked\n");
    }
    
    /* Check if mutex is locked */
    if (mutex_is_locked(&my_mutex)) {
        printk(KERN_INFO "Mutex is currently locked\n");
    }
}
```

### Semaphores

```c
/*
 * Semaphore - Counting semaphore (can sleep)
 */
#include <linux/semaphore.h>

/* Define and initialize semaphore */
struct semaphore my_sem;

void semaphore_example(void)
{
    /* Initialize semaphore with count */
    sema_init(&my_sem, 1);  // Binary semaphore (like mutex)
    // sema_init(&my_sem, 5);  // Counting semaphore (max 5)
    
    /* Down (acquire, decrement) - sleeps if count is 0 */
    down(&my_sem);
    /* Critical section */
    // access_resource();
    
    /* Up (release, increment) */
    up(&my_sem);
    
    /* Interruptible down */
    if (down_interruptible(&my_sem)) {
        /* Interrupted by signal */
        return -ERESTARTSYS;
    }
    /* Critical section */
    up(&my_sem);
    
    /* Try down (non-blocking) */
    if (down_trylock(&my_sem) == 0) {
        /* Successfully acquired */
        // critical_section();
        up(&my_sem);
    } else {
        /* Semaphore busy */
        printk(KERN_INFO "Semaphore busy\n");
    }
}

/*
 * Use case: Limiting concurrent access
 */
struct semaphore resource_sem;

static int __init sem_init(void)
{
    /* Allow maximum 3 concurrent users */
    sema_init(&resource_sem, 3);
    return 0;
}

void use_limited_resource(void)
{
    /* Try to acquire resource */
    if (down_interruptible(&resource_sem)) {
        return;  // Interrupted
    }
    
    /* Use resource (max 3 processes can be here) */
    printk(KERN_INFO "Using limited resource\n");
    msleep(1000);  // Simulate work
    
    /* Release resource */
    up(&resource_sem);
}
```

### Read-Copy-Update (RCU)

```c
/*
 * RCU - Read-Copy-Update
 * Optimized for read-mostly scenarios
 */
#include <linux/rcupdate.h>

struct my_data {
    int value;
    struct rcu_head rcu;
};

static struct my_data *global_data;

/*
 * RCU read operation (fast, no locks)
 */
void rcu_read_example(void)
{
    struct my_data *data;
    
    /* Enter RCU read-side critical section */
    rcu_read_lock();
    
    /* Read shared data */
    data = rcu_dereference(global_data);
    if (data) {
        printk(KERN_INFO "Value: %d\n", data->value);
    }
    
    /* Exit RCU read-side critical section */
    rcu_read_unlock();
}

/*
 * RCU callback for delayed free
 */
static void my_data_free_rcu(struct rcu_head *rcu)
{
    struct my_data *data = container_of(rcu, struct my_data, rcu);
    kfree(data);
    printk(KERN_INFO "RCU: Data freed\n");
}

/*
 * RCU update operation
 */
void rcu_update_example(void)
{
    struct my_data *old_data, *new_data;
    
    /* Allocate new data structure */
    new_data = kmalloc(sizeof(*new_data), GFP_KERNEL);
    if (!new_data)
        return;
    
    new_data->value = 42;
    
    /* Update pointer (atomic) */
    old_data = global_data;
    rcu_assign_pointer(global_data, new_data);
    
    /* Wait for all readers to complete */
    synchronize_rcu();
    
    /* Now safe to free old data */
    if (old_data)
        kfree(old_data);
}

/*
 * RCU update with callback
 */
void rcu_update_with_callback(void)
{
    struct my_data *old_data, *new_data;
    
    new_data = kmalloc(sizeof(*new_data), GFP_KERNEL);
    if (!new_data)
        return;
    
    new_data->value = 100;
    
    /* Update pointer */
    old_data = global_data;
    rcu_assign_pointer(global_data, new_data);
    
    /* Schedule callback to free old data after grace period */
    if (old_data)
        call_rcu(&old_data->rcu, my_data_free_rcu);
}
```

---

## 12. Advanced Topics

### Kernel Timers

```c
/*
 * Kernel timers - Schedule functions for future execution
 */
#include <linux/timer.h>

struct timer_list my_timer;

/* Timer callback function */
void timer_callback(struct timer_list *t)
{
    printk(KERN_INFO "Timer expired!\n");
    
    /* Optionally reschedule timer */
    mod_timer(&my_timer, jiffies + HZ);  // Fire again in 1 second
}

static int __init timer_init(void)
{
    /* Initialize timer */
    timer_setup(&my_timer, timer_callback, 0);
    
    /* Set expiration time (jiffies + delay) */
    my_timer.expires = jiffies + (HZ * 5);  // 5 seconds from now
    
    /* Activate timer */
    add_timer(&my_timer);
    
    printk(KERN_INFO "Timer started\n");
    return 0;
}

static void __exit timer_exit(void)
{
    /* Delete timer */
    del_timer_sync(&my_timer);  // Wait for callback to complete
    printk(KERN_INFO "Timer deleted\n");
}

/*
 * High-resolution timers (hrtimer)
 */
#include <linux/hrtimer.h>

struct hrtimer my_hrtimer;

enum hrtimer_restart hrtimer_callback(struct hrtimer *timer)
{
    printk(KERN_INFO "High-resolution timer expired\n");
    
    /* Return HRTIMER_NORESTART to not restart automatically */
    return HRTIMER_NORESTART;
    
    /* Or restart with new time */
    // hrtimer_forward_now(timer, ktime_set(1, 0));  // 1 second
    // return HRTIMER_RESTART;
}

static int __init hrtimer_init_module(void)
{
    ktime_t ktime;
    
    /* Initialize high-resolution timer */
    hrtimer_init(&my_hrtimer, CLOCK_MONOTONIC, HRTIMER_MODE_REL);
    my_hrtimer.function = hrtimer_callback;
    
    /* Set timer for 500 milliseconds */
    ktime = ktime_set(0, 500 * 1000000);  // 0 sec, 500 million nanosec
    hrtimer_start(&my_hrtimer, ktime, HRTIMER_MODE_REL);
    
    printk(KERN_INFO "High-resolution timer started\n");
    return 0;
}

static void __exit hrtimer_exit_module(void)
{
    hrtimer_cancel(&my_hrtimer);
    printk(KERN_INFO "High-resolution timer cancelled\n");
}
```

### Kernel Linked Lists

```c
/*
 * Kernel linked list implementation
 */
#include <linux/list.h>

/* Data structure with list node */
struct my_node {
    int data;
    char name[32];
    struct list_head list;  // List node
};

/* List head */
LIST_HEAD(my_list);

void list_example(void)
{
    struct my_node *node, *tmp;
    struct list_head *pos;
    
    /* Add nodes to list */
    for (int i = 0; i < 5; i++) {
        node = kmalloc(sizeof(*node), GFP_KERNEL);
        if (!node)
            continue;
        
        node->data = i * 10;
        snprintf(node->name, sizeof(node->name), "node_%d", i);
        
        /* Add to tail of list */
        list_add_tail(&node->list, &my_list);
        
        /* Alternative: Add to head */
        // list_add(&node->list, &my_list);
    }
    
    /* Iterate through list */
    list_for_each(pos, &my_list) {
        node = list_entry(pos, struct my_node, list);
        printk(KERN_INFO "Node: %s, data=%d\n", node->name, node->data);
    }
    
    /* Safer iteration (allows deletion) */
    list_for_each_entry_safe(node, tmp, &my_list, list) {
        printk(KERN_INFO "Processing: %s\n", node->name);
        
        /* Remove from list */
        list_del(&node->list);
        kfree(node);
    }
    
    /* Check if list is empty */
    if (list_empty(&my_list)) {
        printk(KERN_INFO "List is empty\n");
    }
}
```

### DMA (Direct Memory Access)

```c
/*
 * DMA - Direct Memory Access
 */
#include <linux/dma-mapping.h>

void dma_example(struct device *dev)
{
    void *cpu_addr;
    dma_addr_t dma_handle;
    size_t size = PAGE_SIZE;
    
    /* Allocate DMA buffer */
    cpu_addr = dma_alloc_coherent(dev, size, &dma_handle, GFP_KERNEL);
    if (!cpu_addr) {
        printk(KERN_ERR "DMA allocation failed\n");
        return;
    }
    
    printk(KERN_INFO "DMA buffer allocated\n");
    printk(KERN_INFO "CPU address: %p\n", cpu_addr);
    printk(KERN_INFO "DMA address: %pad\n", &dma_handle);
    
    /* Use DMA buffer */
    memset(cpu_addr, 0, size);
    
    /* Program device to use DMA address */
    // writel(dma_handle, device_base + DMA_ADDR_REG);
    // writel(size, device_base + DMA_SIZE_REG);
    // writel(DMA_START, device_base + DMA_CONTROL_REG);
    
    /* Free DMA buffer when done */
    dma_free_coherent(dev, size, cpu_addr, dma_handle);
}

/*
 * Streaming DMA mapping
 */
void streaming_dma_example(struct device *dev)
{
    void *buffer;
    dma_addr_t dma_addr;
    size_t size = 1024;
    
    /* Allocate regular buffer */
    buffer = kmalloc(size, GFP_KERNEL);
    if (!buffer)
        return;
    
    /* Map for DMA */
    dma_addr = dma_map_single(dev, buffer, size, DMA_TO_DEVICE);
    if (dma_mapping_error(dev, dma_addr)) {
        printk(KERN_ERR "DMA mapping failed\n");
        kfree(buffer);
        return;
    }
    
    /* Use for DMA transfer */
    // Setup device DMA transfer
    
    /* Wait for transfer to complete */
    // ...
    
    /* Unmap DMA */
    dma_unmap_single(dev, dma_addr, size, DMA_TO_DEVICE);
    
    /* Free buffer */
    kfree(buffer);
}
```

### procfs and sysfs

```c
/*
 * procfs - Process filesystem interface
 */
#include <linux/proc_fs.h>
#include <linux/seq_file.h>

static struct proc_dir_entry *proc_entry;

/* Show function for procfs */
static int my_proc_show(struct seq_file *m, void *v)
{
    seq_printf(m, "Hello from procfs!\n");
    seq_printf(m, "Current PID: %d\n", current->pid);
    seq_printf(m, "Current process: %s\n", current->comm);
    return 0;
}

/* Open function */
static int my_proc_open(struct inode *inode, struct file *file)
{
    return single_open(file, my_proc_show, NULL);
}

/* File operations for procfs */
static const struct file_operations my_proc_fops = {
    .owner = THIS_MODULE,
    .open = my_proc_open,
    .read = seq_read,
    .llseek = seq_lseek,
    .release = single_release,
};

static int __init proc_init(void)
{
    /* Create /proc/my_module entry */
    proc_entry = proc_create("my_module", 0444, NULL, &my_proc_fops);
    if (!proc_entry) {
        printk(KERN_ERR "Failed to create proc entry\n");
        return -ENOMEM;
    }
    
    printk(KERN_INFO "procfs entry created\n");
    return 0;
}

static void __exit proc_exit(void)
{
    proc_remove(proc_entry);
    printk(KERN_INFO "procfs entry removed\n");
}

/*
 * sysfs - System filesystem interface
 */
#include <linux/kobject.h>
#include <linux/sysfs.h>

static struct kobject *my_kobj;
static int my_value = 0;

/* Show attribute */
static ssize_t my_show(struct kobject *kobj, struct kobj_attribute *attr,
                       char *buf)
{
    return sprintf(buf, "%d\n", my_value);
}

/* Store attribute */
static ssize_t my_store(struct kobject *kobj, struct kobj_attribute *attr,
                        const char *buf, size_t count)
{
    sscanf(buf, "%d", &my_value);
    return count;
}

/* Define attribute */
static struct kobj_attribute my_attribute =
    __ATTR(my_value, 0664, my_show, my_store);

static int __init sysfs_init(void)
{
    int retval;
    
    /* Create kobject under /sys/kernel/ */
    my_kobj = kobject_create_and_add("my_module", kernel_kobj);
    if (!my_kobj)
        return -ENOMEM;
    
    /* Create attribute file */
    retval = sysfs_create_file(my_kobj, &my_attribute.attr);
    if (retval)
        kobject_put(my_kobj);
    
    printk(KERN_INFO "sysfs entry created at /sys/kernel/my_module/my_value\n");
    return retval;
}

static void __exit sysfs_exit(void)
{
    kobject_put(my_kobj);
    printk(KERN_INFO "sysfs entry removed\n");
}
```

---

## Summary

This guide covered:

1. **Linux Kernel Architecture** - Overall structure and components
2. **Source Code Structure** - Directory organization
3. **Process Management** - task_struct, scheduling, fork()
4. **Memory Management** - Virtual memory, paging, allocation
5. **File System & VFS** - Superblock, inode, dentry, file operations
6. **Device Drivers** - Character devices, file operations
7. **Network Stack** - Socket buffers, TCP/IP, netfilter
8. **System Calls** - Mechanism, implementation, adding custom calls
9. **Kernel Modules** - Loading, parameters, dependencies
10. **Interrupt Handling** - ISR, tasklets, work queues
11. **Synchronization** - Atomic ops, spinlocks, mutexes, RCU
12. **Advanced Topics** - Timers, lists, DMA, procfs, sysfs

### Key Concepts

- **Kernel vs User Space**: Protection rings, system calls as interface
- **Process Context**: Current process, can sleep
- **Interrupt Context**: Cannot sleep, must be fast
- **Synchronization**: Choose right primitive for the job
- **Memory**: Physical vs virtual, paging, caching
- **Networking**: Layered architecture, sk_buff lifecycle

### Best Practices

1. **Always check return values**
2. **Free resources in reverse order of allocation**
3. **Use appropriate synchronization primitives**
4. **Keep interrupt handlers fast**
5. **Validate user space data with copy_from_user/copy_to_user**
6. **Use kernel logging (printk) for debugging**
7. **Follow kernel coding style**

### Further Study

- Read kernel source code in `/usr/src/linux/`
- Study "Linux Kernel Development" by Robert Love
- Explore "Understanding the Linux Kernel" by Bovet & Cesati
- Practice with kernel modules and debugging
- Contribute to open source kernel projects

---

**End of Linux Kernel Mastery Guide**
