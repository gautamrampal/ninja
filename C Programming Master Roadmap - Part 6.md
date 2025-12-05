# The Complete C Programming Master Roadmap - PART 6
## Threads, Synchronization & Network Programming

---

## Table of Contents - Part 6
1. [POSIX Threads (pthread)](#1-posix-threads)
2. [Thread Synchronization](#2-thread-synchronization)
3. [Thread-Safe Programming](#3-thread-safe-programming)
4. [Network Programming - Sockets](#4-network-programming-sockets)
5. [Advanced Socket Programming](#5-advanced-socket-programming)

---

## 1. POSIX Threads

**Thread vs Process:**
```
Process:                     Thread:
- Own memory space           - Shared memory space
- Heavy-weight              - Light-weight
- IPC needed                - Direct communication
- Slower context switch     - Fast context switch

Memory Layout with Threads:
┌──────────────────┐
│   Code (Text)    │  ← Shared
├──────────────────┤
│   Data/BSS       │  ← Shared
├──────────────────┤
│   Heap           │  ← Shared
├──────────────────┤
│   Thread 1 Stack │  ← Private
├──────────────────┤
│   Thread 2 Stack │  ← Private
└──────────────────┘
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

// Basic thread creation
void* thread_function(void *arg) {
    int thread_id = *(int *)arg;
    
    printf("Thread %d starting\n", thread_id);
    sleep(2);
    printf("Thread %d finishing\n", thread_id);
    
    return NULL;
}

void basic_thread_example() {
    pthread_t threads[3];
    int thread_ids[3];
    
    // Create threads
    for (int i = 0; i < 3; i++) {
        thread_ids[i] = i;
        if (pthread_create(&threads[i], NULL, thread_function, &thread_ids[i]) != 0) {
            perror("pthread_create failed");
            exit(1);
        }
    }
    
    // Wait for threads to finish
    for (int i = 0; i < 3; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("All threads completed\n");
}

// Thread with return value
void* compute_sum(void *arg) {
    int n = *(int *)arg;
    int *result = malloc(sizeof(int));
    
    *result = 0;
    for (int i = 1; i <= n; i++) {
        *result += i;
    }
    
    return result;
}

void thread_return_value() {
    pthread_t thread;
    int n = 100;
    
    pthread_create(&thread, NULL, compute_sum, &n);
    
    void *result;
    pthread_join(thread, &result);
    
    printf("Sum of 1 to %d = %d\n", n, *(int *)result);
    free(result);
}

// Thread attributes
void thread_attributes() {
    pthread_t thread;
    pthread_attr_t attr;
    
    // Initialize attributes
    pthread_attr_init(&attr);
    
    // Set detached state (thread cleans up automatically)
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
    
    // Set stack size (8MB)
    size_t stack_size = 8 * 1024 * 1024;
    pthread_attr_setstacksize(&attr, stack_size);
    
    void* detached_thread(void *arg) {
        printf("Detached thread running\n");
        sleep(1);
        return NULL;
    }
    
    pthread_create(&thread, &attr, detached_thread, NULL);
    
    // No need to join detached thread
    sleep(2);
    
    pthread_attr_destroy(&attr);
}

// Thread cancellation
void* cancellable_thread(void *arg) {
    printf("Thread starting (cancellable)\n");
    
    // Set cancellation type
    pthread_setcanceltype(PTHREAD_CANCEL_ASYNCHRONOUS, NULL);
    
    while (1) {
        printf("Working...\n");
        sleep(1);
    }
    
    return NULL;
}

void thread_cancellation() {
    pthread_t thread;
    
    pthread_create(&thread, NULL, cancellable_thread, NULL);
    
    sleep(3);
    
    printf("Cancelling thread\n");
    pthread_cancel(thread);
    
    pthread_join(thread, NULL);
    printf("Thread cancelled\n");
}

int main() {
    printf("=== Basic Thread Example ===\n");
    basic_thread_example();
    
    printf("\n=== Thread Return Value ===\n");
    thread_return_value();
    
    printf("\n=== Thread Attributes ===\n");
    thread_attributes();
    
    // printf("\n=== Thread Cancellation ===\n");
    // thread_cancellation();  // Uncomment to test
    
    return 0;
}

// Compile: gcc -pthread program.c -o program
```

---

## 2. Thread Synchronization

### 2.1 Mutexes (Mutual Exclusion)

**Race Condition Problem:**
```
Without Mutex:
Thread 1: Read count=0    Thread 2: Read count=0
Thread 1: count++         Thread 2: count++
Thread 1: Write count=1   Thread 2: Write count=1
Result: count=1 (WRONG! Should be 2)

With Mutex:
Thread 1: Lock mutex
Thread 1: Read count=0
Thread 1: count++
Thread 1: Write count=1
Thread 1: Unlock mutex
Thread 2: Lock mutex
Thread 2: Read count=1
Thread 2: count++
Thread 2: Write count=2
Thread 2: Unlock mutex
Result: count=2 (CORRECT)
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

// Race condition without mutex
int counter_unsafe = 0;

void* increment_unsafe(void *arg) {
    for (int i = 0; i < 100000; i++) {
        counter_unsafe++;  // Race condition!
    }
    return NULL;
}

void race_condition_demo() {
    pthread_t threads[5];
    
    for (int i = 0; i < 5; i++) {
        pthread_create(&threads[i], NULL, increment_unsafe, NULL);
    }
    
    for (int i = 0; i < 5; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("Counter (unsafe): %d (expected 500000)\n", counter_unsafe);
}

// Safe version with mutex
int counter_safe = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void* increment_safe(void *arg) {
    for (int i = 0; i < 100000; i++) {
        pthread_mutex_lock(&mutex);
        counter_safe++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void mutex_demo() {
    pthread_t threads[5];
    
    for (int i = 0; i < 5; i++) {
        pthread_create(&threads[i], NULL, increment_safe, NULL);
    }
    
    for (int i = 0; i < 5; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("Counter (safe): %d (expected 500000)\n", counter_safe);
    
    pthread_mutex_destroy(&mutex);
}

// Deadlock example
pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex2 = PTHREAD_MUTEX_INITIALIZER;

void* thread1_func(void *arg) {
    pthread_mutex_lock(&mutex1);
    printf("Thread 1: Locked mutex1\n");
    sleep(1);
    
    printf("Thread 1: Waiting for mutex2\n");
    pthread_mutex_lock(&mutex2);  // Deadlock!
    
    pthread_mutex_unlock(&mutex2);
    pthread_mutex_unlock(&mutex1);
    return NULL;
}

void* thread2_func(void *arg) {
    pthread_mutex_lock(&mutex2);
    printf("Thread 2: Locked mutex2\n");
    sleep(1);
    
    printf("Thread 2: Waiting for mutex1\n");
    pthread_mutex_lock(&mutex1);  // Deadlock!
    
    pthread_mutex_unlock(&mutex1);
    pthread_mutex_unlock(&mutex2);
    return NULL;
}

void deadlock_demo() {
    pthread_t t1, t2;
    
    pthread_create(&t1, NULL, thread1_func, NULL);
    pthread_create(&t2, NULL, thread2_func, NULL);
    
    // This will hang due to deadlock
    // pthread_join(t1, NULL);
    // pthread_join(t2, NULL);
    
    sleep(3);
    printf("Deadlock occurred (threads not joined)\n");
}

// Trylock to avoid deadlock
void* thread_trylock(void *arg) {
    while (1) {
        pthread_mutex_lock(&mutex1);
        
        if (pthread_mutex_trylock(&mutex2) == 0) {
            // Got both locks
            printf("Thread acquired both locks\n");
            pthread_mutex_unlock(&mutex2);
            pthread_mutex_unlock(&mutex1);
            break;
        } else {
            // Couldn't get second lock, release first
            pthread_mutex_unlock(&mutex1);
            usleep(100);  // Small delay before retry
        }
    }
    return NULL;
}

int main() {
    printf("=== Race Condition Demo ===\n");
    race_condition_demo();
    
    printf("\n=== Mutex Demo ===\n");
    mutex_demo();
    
    printf("\n=== Deadlock Demo ===\n");
    // deadlock_demo();  // Uncomment to see deadlock (will hang)
    
    return 0;
}
```

### 2.2 Condition Variables

**Producer-Consumer Pattern:**
```
Producer:                  Consumer:
1. Lock mutex              1. Lock mutex
2. Produce item            2. While buffer empty:
3. Signal consumer            - Wait on condition
4. Unlock mutex            3. Consume item
                          4. Signal producer
                          5. Unlock mutex
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define BUFFER_SIZE 5

typedef struct {
    int buffer[BUFFER_SIZE];
    int count;
    int in;
    int out;
    pthread_mutex_t mutex;
    pthread_cond_t not_full;
    pthread_cond_t not_empty;
} BoundedBuffer;

void buffer_init(BoundedBuffer *buf) {
    buf->count = 0;
    buf->in = 0;
    buf->out = 0;
    pthread_mutex_init(&buf->mutex, NULL);
    pthread_cond_init(&buf->not_full, NULL);
    pthread_cond_init(&buf->not_empty, NULL);
}

void buffer_destroy(BoundedBuffer *buf) {
    pthread_mutex_destroy(&buf->mutex);
    pthread_cond_destroy(&buf->not_full);
    pthread_cond_destroy(&buf->not_empty);
}

void buffer_put(BoundedBuffer *buf, int item) {
    pthread_mutex_lock(&buf->mutex);
    
    // Wait while buffer is full
    while (buf->count == BUFFER_SIZE) {
        pthread_cond_wait(&buf->not_full, &buf->mutex);
    }
    
    // Add item to buffer
    buf->buffer[buf->in] = item;
    buf->in = (buf->in + 1) % BUFFER_SIZE;
    buf->count++;
    
    printf("Produced: %d (count=%d)\n", item, buf->count);
    
    // Signal that buffer is not empty
    pthread_cond_signal(&buf->not_empty);
    
    pthread_mutex_unlock(&buf->mutex);
}

int buffer_get(BoundedBuffer *buf) {
    pthread_mutex_lock(&buf->mutex);
    
    // Wait while buffer is empty
    while (buf->count == 0) {
        pthread_cond_wait(&buf->not_empty, &buf->mutex);
    }
    
    // Remove item from buffer
    int item = buf->buffer[buf->out];
    buf->out = (buf->out + 1) % BUFFER_SIZE;
    buf->count--;
    
    printf("Consumed: %d (count=%d)\n", item, buf->count);
    
    // Signal that buffer is not full
    pthread_cond_signal(&buf->not_full);
    
    pthread_mutex_unlock(&buf->mutex);
    
    return item;
}

BoundedBuffer shared_buffer;

void* producer(void *arg) {
    int id = *(int *)arg;
    
    for (int i = 0; i < 10; i++) {
        int item = id * 100 + i;
        buffer_put(&shared_buffer, item);
        usleep(100000);  // 100ms
    }
    
    return NULL;
}

void* consumer(void *arg) {
    int id = *(int *)arg;
    
    for (int i = 0; i < 10; i++) {
        int item = buffer_get(&shared_buffer);
        usleep(150000);  // 150ms
    }
    
    return NULL;
}

void producer_consumer_demo() {
    buffer_init(&shared_buffer);
    
    pthread_t prod_threads[2], cons_threads[2];
    int ids[2] = {1, 2};
    
    // Create producers
    for (int i = 0; i < 2; i++) {
        pthread_create(&prod_threads[i], NULL, producer, &ids[i]);
    }
    
    // Create consumers
    for (int i = 0; i < 2; i++) {
        pthread_create(&cons_threads[i], NULL, consumer, &ids[i]);
    }
    
    // Wait for all threads
    for (int i = 0; i < 2; i++) {
        pthread_join(prod_threads[i], NULL);
        pthread_join(cons_threads[i], NULL);
    }
    
    buffer_destroy(&shared_buffer);
}

int main() {
    printf("=== Producer-Consumer Demo ===\n");
    producer_consumer_demo();
    
    return 0;
}
```

### 2.3 Semaphores

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

// Binary semaphore (mutex alternative)
sem_t binary_sem;
int shared_resource = 0;

void* use_resource(void *arg) {
    int id = *(int *)arg;
    
    sem_wait(&binary_sem);  // Lock
    
    printf("Thread %d accessing resource\n", id);
    shared_resource++;
    sleep(1);
    printf("Thread %d releasing resource\n", id);
    
    sem_post(&binary_sem);  // Unlock
    
    return NULL;
}

void binary_semaphore_demo() {
    sem_init(&binary_sem, 0, 1);  // Initial value = 1
    
    pthread_t threads[3];
    int ids[3] = {1, 2, 3};
    
    for (int i = 0; i < 3; i++) {
        pthread_create(&threads[i], NULL, use_resource, &ids[i]);
    }
    
    for (int i = 0; i < 3; i++) {
        pthread_join(threads[i], NULL);
    }
    
    sem_destroy(&binary_sem);
}

// Counting semaphore (limited resource pool)
#define MAX_RESOURCES 3
sem_t counting_sem;

void* use_limited_resource(void *arg) {
    int id = *(int *)arg;
    
    printf("Thread %d waiting for resource\n", id);
    sem_wait(&counting_sem);
    
    printf("Thread %d got resource\n", id);
    sleep(2);
    
    printf("Thread %d releasing resource\n", id);
    sem_post(&counting_sem);
    
    return NULL;
}

void counting_semaphore_demo() {
    sem_init(&counting_sem, 0, MAX_RESOURCES);
    
    pthread_t threads[10];
    int ids[10];
    
    for (int i = 0; i < 10; i++) {
        ids[i] = i;
        pthread_create(&threads[i], NULL, use_limited_resource, &ids[i]);
    }
    
    for (int i = 0; i < 10; i++) {
        pthread_join(threads[i], NULL);
    }
    
    sem_destroy(&counting_sem);
}

int main() {
    printf("=== Binary Semaphore Demo ===\n");
    binary_semaphore_demo();
    
    printf("\n=== Counting Semaphore Demo ===\n");
    counting_semaphore_demo();
    
    return 0;
}
```

---

## 3. Thread-Safe Programming

**Thread-Local Storage:**
```c
#include <stdio.h>
#include <pthread.h>

// Thread-local variable (each thread has its own copy)
__thread int thread_local_var = 0;

// Global variable (shared by all threads)
int global_var = 0;

void* thread_function(void *arg) {
    int id = *(int *)arg;
    
    // Each thread modifies its own thread_local_var
    thread_local_var = id * 10;
    
    // All threads modify shared global_var
    global_var += id;
    
    printf("Thread %d: thread_local=%d, global=%d\n", 
           id, thread_local_var, global_var);
    
    return NULL;
}

int main() {
    pthread_t threads[3];
    int ids[3] = {1, 2, 3};
    
    for (int i = 0; i < 3; i++) {
        pthread_create(&threads[i], NULL, thread_function, &ids[i]);
    }
    
    for (int i = 0; i < 3; i++) {
        pthread_join(threads[i], NULL);
    }
    
    return 0;
}
```

**Reader-Writer Locks:**
```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;
int shared_data = 0;

void* reader(void *arg) {
    int id = *(int *)arg;
    
    pthread_rwlock_rdlock(&rwlock);  // Read lock (multiple readers allowed)
    
    printf("Reader %d: reading data = %d\n", id, shared_data);
    sleep(1);
    
    pthread_rwlock_unlock(&rwlock);
    
    return NULL;
}

void* writer(void *arg) {
    int id = *(int *)arg;
    
    pthread_rwlock_wrlock(&rwlock);  // Write lock (exclusive)
    
    shared_data += 10;
    printf("Writer %d: wrote data = %d\n", id, shared_data);
    sleep(1);
    
    pthread_rwlock_unlock(&rwlock);
    
    return NULL;
}

int main() {
    pthread_t readers[5], writers[2];
    int reader_ids[5] = {1, 2, 3, 4, 5};
    int writer_ids[2] = {1, 2};
    
    for (int i = 0; i < 2; i++) {
        pthread_create(&writers[i], NULL, writer, &writer_ids[i]);
    }
    
    for (int i = 0; i < 5; i++) {
        pthread_create(&readers[i], NULL, reader, &reader_ids[i]);
    }
    
    for (int i = 0; i < 2; i++) {
        pthread_join(writers[i], NULL);
    }
    
    for (int i = 0; i < 5; i++) {
        pthread_join(readers[i], NULL);
    }
    
    pthread_rwlock_destroy(&rwlock);
    
    return 0;
}
```

---

## 4. Network Programming - Sockets

**Socket Communication Flow:**
```
Server:                      Client:
1. socket()                  1. socket()
2. bind()                    
3. listen()                  
4. accept() ←────────────── 2. connect()
5. recv/send ←──────────→   3. send/recv
6. close()                   4. close()
```

### 4.1 TCP Server/Client

**Server:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

void tcp_server() {
    int server_fd, client_fd;
    struct sockaddr_in server_addr, client_addr;
    socklen_t addr_len = sizeof(client_addr);
    char buffer[BUFFER_SIZE];
    
    // 1. Create socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd < 0) {
        perror("Socket creation failed");
        exit(1);
    }
    
    // Set socket options (reuse address)
    int opt = 1;
    setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    
    // 2. Bind socket to address
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);
    
    if (bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind failed");
        exit(1);
    }
    
    // 3. Listen for connections
    if (listen(server_fd, 5) < 0) {
        perror("Listen failed");
        exit(1);
    }
    
    printf("Server listening on port %d\n", PORT);
    
    // 4. Accept connection
    client_fd = accept(server_fd, (struct sockaddr*)&client_addr, &addr_len);
    if (client_fd < 0) {
        perror("Accept failed");
        exit(1);
    }
    
    printf("Client connected: %s:%d\n", 
           inet_ntoa(client_addr.sin_addr), 
           ntohs(client_addr.sin_port));
    
    // 5. Receive and send data
    int bytes_read = recv(client_fd, buffer, BUFFER_SIZE - 1, 0);
    if (bytes_read > 0) {
        buffer[bytes_read] = '\0';
        printf("Received: %s\n", buffer);
        
        // Echo back to client
        send(client_fd, buffer, bytes_read, 0);
    }
    
    // 6. Close sockets
    close(client_fd);
    close(server_fd);
}

int main() {
    tcp_server();
    return 0;
}
```

**Client:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

void tcp_client() {
    int sock_fd;
    struct sockaddr_in server_addr;
    char buffer[BUFFER_SIZE];
    
    // 1. Create socket
    sock_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (sock_fd < 0) {
        perror("Socket creation failed");
        exit(1);
    }
    
    // 2. Connect to server
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    
    if (inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr) <= 0) {
        perror("Invalid address");
        exit(1);
    }
    
    if (connect(sock_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("Connection failed");
        exit(1);
    }
    
    printf("Connected to server\n");
    
    // 3. Send message
    const char *message = "Hello from client!";
    send(sock_fd, message, strlen(message), 0);
    printf("Sent: %s\n", message);
    
    // 4. Receive response
    int bytes_read = recv(sock_fd, buffer, BUFFER_SIZE - 1, 0);
    if (bytes_read > 0) {
        buffer[bytes_read] = '\0';
        printf("Received: %s\n", buffer);
    }
    
    // 5. Close socket
    close(sock_fd);
}

int main() {
    tcp_client();
    return 0;
}
```

### 4.2 UDP Server/Client

**Server:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

void udp_server() {
    int sock_fd;
    struct sockaddr_in server_addr, client_addr;
    socklen_t addr_len = sizeof(client_addr);
    char buffer[BUFFER_SIZE];
    
    // Create UDP socket
    sock_fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock_fd < 0) {
        perror("Socket creation failed");
        exit(1);
    }
    
    // Bind socket
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);
    
    if (bind(sock_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind failed");
        exit(1);
    }
    
    printf("UDP Server listening on port %d\n", PORT);
    
    // Receive datagram
    int bytes_read = recvfrom(sock_fd, buffer, BUFFER_SIZE - 1, 0,
                               (struct sockaddr*)&client_addr, &addr_len);
    
    if (bytes_read > 0) {
        buffer[bytes_read] = '\0';
        printf("Received from %s:%d: %s\n",
               inet_ntoa(client_addr.sin_addr),
               ntohs(client_addr.sin_port),
               buffer);
        
        // Send response
        sendto(sock_fd, buffer, bytes_read, 0,
               (struct sockaddr*)&client_addr, addr_len);
    }
    
    close(sock_fd);
}

int main() {
    udp_server();
    return 0;
}
```

**Client:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

void udp_client() {
    int sock_fd;
    struct sockaddr_in server_addr;
    socklen_t addr_len = sizeof(server_addr);
    char buffer[BUFFER_SIZE];
    
    // Create UDP socket
    sock_fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock_fd < 0) {
        perror("Socket creation failed");
        exit(1);
    }
    
    // Server address
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr);
    
    // Send datagram
    const char *message = "Hello UDP!";
    sendto(sock_fd, message, strlen(message), 0,
           (struct sockaddr*)&server_addr, sizeof(server_addr));
    
    printf("Sent: %s\n", message);
    
    // Receive response
    int bytes_read = recvfrom(sock_fd, buffer, BUFFER_SIZE - 1, 0,
                               (struct sockaddr*)&server_addr, &addr_len);
    
    if (bytes_read > 0) {
        buffer[bytes_read] = '\0';
        printf("Received: %s\n", buffer);
    }
    
    close(sock_fd);
}

int main() {
    udp_client();
    return 0;
}
```

---

## CONTINUED IN PART 7

This is Part 6 of the Complete C Programming Master Roadmap. It covers:
- POSIX Threads (creation, attributes, cancellation)
- Thread Synchronization (Mutexes, Condition Variables, Semaphores)
- Thread-Safe Programming
- Network Programming (TCP/UDP Sockets)

**Part 7 will include:**
- Advanced Socket Programming (select, poll, epoll)
- Linux Internals (System Calls, Kernel Modules)
- Memory-Mapped I/O
- Performance Optimization Techniques

Save this file and continue with Part 7!