# The Complete C Programming Master Roadmap - PART 5
## Systems Programming

---

## Table of Contents - Part 5
1. [File I/O Operations](#1-file-io-operations)
2. [Process Management](#2-process-management)
3. [Inter-Process Communication (IPC)](#3-inter-process-communication)
4. [Signals](#4-signals)
5. [Threads and Synchronization](#5-threads-and-synchronization)
6. [Network Programming Basics](#6-network-programming-basics)

---

## 1. File I/O Operations

### 1.1 Text File Operations

**File Modes:**
```
"r"   - Read (file must exist)
"w"   - Write (creates/truncates file)
"a"   - Append (creates if not exists)
"r+"  - Read/Write (file must exist)
"w+"  - Read/Write (creates/truncates)
"a+"  - Read/Append

Binary modes: "rb", "wb", "ab", "rb+", "wb+", "ab+"
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void write_text_file() {
    FILE *fp = fopen("example.txt", "w");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    fprintf(fp, "Hello, World!\n");
    fprintf(fp, "Line number: %d\n", 2);
    fputs("This is a line\n", fp);
    
    fclose(fp);
    printf("File written successfully\n");
}

void read_text_file() {
    FILE *fp = fopen("example.txt", "r");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    char buffer[256];
    
    printf("Reading file:\n");
    while (fgets(buffer, sizeof(buffer), fp) != NULL) {
        printf("%s", buffer);
    }
    
    fclose(fp);
}

void read_file_character_by_character() {
    FILE *fp = fopen("example.txt", "r");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    int c;
    while ((c = fgetc(fp)) != EOF) {
        putchar(c);
    }
    
    fclose(fp);
}

void read_formatted_data() {
    FILE *fp = fopen("data.txt", "w");
    fprintf(fp, "John 25 85.5\n");
    fprintf(fp, "Jane 30 92.3\n");
    fclose(fp);
    
    fp = fopen("data.txt", "r");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    char name[50];
    int age;
    float score;
    
    printf("\nFormatted data:\n");
    while (fscanf(fp, "%s %d %f", name, &age, &score) == 3) {
        printf("Name: %s, Age: %d, Score: %.1f\n", name, age, score);
    }
    
    fclose(fp);
}

void file_positioning() {
    FILE *fp = fopen("example.txt", "r");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    // Get current position
    long pos = ftell(fp);
    printf("Current position: %ld\n", pos);
    
    // Move to position 5
    fseek(fp, 5, SEEK_SET);
    
    // Read from new position
    char buffer[50];
    fgets(buffer, sizeof(buffer), fp);
    printf("After seeking: %s", buffer);
    
    // Rewind to beginning
    rewind(fp);
    
    fclose(fp);
}

int main() {
    write_text_file();
    read_text_file();
    read_formatted_data();
    file_positioning();
    
    return 0;
}
```

### 1.2 Binary File Operations

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Student {
    int id;
    char name[50];
    float gpa;
} Student;

void write_binary_file() {
    FILE *fp = fopen("students.dat", "wb");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    Student students[] = {
        {1, "Alice", 3.8},
        {2, "Bob", 3.5},
        {3, "Charlie", 3.9}
    };
    
    size_t count = sizeof(students) / sizeof(students[0]);
    size_t written = fwrite(students, sizeof(Student), count, fp);
    
    printf("Written %zu records\n", written);
    fclose(fp);
}

void read_binary_file() {
    FILE *fp = fopen("students.dat", "rb");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    Student student;
    
    printf("\nReading binary file:\n");
    while (fread(&student, sizeof(Student), 1, fp) == 1) {
        printf("ID: %d, Name: %s, GPA: %.2f\n", 
               student.id, student.name, student.gpa);
    }
    
    fclose(fp);
}

void random_access_binary() {
    FILE *fp = fopen("students.dat", "rb");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    // Read second record
    fseek(fp, sizeof(Student), SEEK_SET);
    
    Student student;
    if (fread(&student, sizeof(Student), 1, fp) == 1) {
        printf("\nSecond record: %s (GPA: %.2f)\n", 
               student.name, student.gpa);
    }
    
    fclose(fp);
}

void update_binary_record() {
    FILE *fp = fopen("students.dat", "rb+");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    // Update first record's GPA
    fseek(fp, 0, SEEK_SET);
    
    Student student;
    fread(&student, sizeof(Student), 1, fp);
    
    student.gpa = 4.0;
    
    fseek(fp, 0, SEEK_SET);
    fwrite(&student, sizeof(Student), 1, fp);
    
    printf("\nUpdated first record\n");
    fclose(fp);
}

int main() {
    write_binary_file();
    read_binary_file();
    random_access_binary();
    update_binary_record();
    read_binary_file();
    
    return 0;
}
```

### 1.3 Low-Level File I/O (POSIX)

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

void low_level_write() {
    int fd = open("lowlevel.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("Error opening file");
        return;
    }
    
    const char *text = "Hello from low-level I/O\n";
    ssize_t bytes_written = write(fd, text, strlen(text));
    
    printf("Wrote %zd bytes\n", bytes_written);
    close(fd);
}

void low_level_read() {
    int fd = open("lowlevel.txt", O_RDONLY);
    if (fd == -1) {
        perror("Error opening file");
        return;
    }
    
    char buffer[100];
    ssize_t bytes_read = read(fd, buffer, sizeof(buffer) - 1);
    
    if (bytes_read > 0) {
        buffer[bytes_read] = '\0';
        printf("Read: %s", buffer);
    }
    
    close(fd);
}

void file_copy() {
    int src_fd = open("lowlevel.txt", O_RDONLY);
    int dst_fd = open("copy.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    
    if (src_fd == -1 || dst_fd == -1) {
        perror("Error opening files");
        return;
    }
    
    char buffer[1024];
    ssize_t bytes;
    
    while ((bytes = read(src_fd, buffer, sizeof(buffer))) > 0) {
        write(dst_fd, buffer, bytes);
    }
    
    close(src_fd);
    close(dst_fd);
    printf("File copied successfully\n");
}

int main() {
    low_level_write();
    low_level_read();
    file_copy();
    
    return 0;
}
```

---

## 2. Process Management

### 2.1 Creating Processes with fork()

**Process Creation:**
```
Parent Process (PID: 1000)
        |
        | fork()
        |
        ├─────────────┐
        ▼             ▼
  Parent (1000)   Child (1001)
  returns child   returns 0
  PID
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

void basic_fork() {
    pid_t pid = fork();
    
    if (pid < 0) {
        perror("Fork failed");
        exit(1);
    } else if (pid == 0) {
        // Child process
        printf("Child process: PID=%d, Parent PID=%d\n", 
               getpid(), getppid());
        exit(0);
    } else {
        // Parent process
        printf("Parent process: PID=%d, Child PID=%d\n", 
               getpid(), pid);
        wait(NULL);  // Wait for child to finish
    }
}

void fork_with_exec() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process - execute ls command
        printf("Child executing ls:\n");
        execlp("ls", "ls", "-l", NULL);
        perror("execlp failed");
        exit(1);
    } else {
        // Parent waits
        wait(NULL);
        printf("Child finished\n");
    }
}

void multiple_processes() {
    printf("Creating process tree:\n");
    
    for (int i = 0; i < 3; i++) {
        pid_t pid = fork();
        
        if (pid == 0) {
            printf("Child %d: PID=%d, Parent=%d\n", 
                   i, getpid(), getppid());
            sleep(1);
            exit(0);
        }
    }
    
    // Parent waits for all children
    for (int i = 0; i < 3; i++) {
        wait(NULL);
    }
    
    printf("All children finished\n");
}

void zombie_orphan_demo() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process
        printf("Child running\n");
        sleep(1);
        printf("Child exiting\n");
        exit(0);
        // Child becomes zombie until parent calls wait()
    } else {
        // Parent doesn't call wait immediately
        printf("Parent sleeping (child becomes zombie)\n");
        sleep(3);
        
        int status;
        wait(&status);  // Reap zombie
        
        if (WIFEXITED(status)) {
            printf("Child exited with status %d\n", WEXITSTATUS(status));
        }
    }
}

int main() {
    printf("=== Basic Fork ===\n");
    basic_fork();
    
    printf("\n=== Fork with Exec ===\n");
    fork_with_exec();
    
    printf("\n=== Multiple Processes ===\n");
    multiple_processes();
    
    printf("\n=== Zombie Demo ===\n");
    zombie_orphan_demo();
    
    return 0;
}
```

### 2.2 Process Execution (exec family)

**exec Family Functions:**
```
execl()  - List of arguments
execv()  - Vector of arguments
execlp() - Search PATH
execvp() - Vector + search PATH
execle() - List + environment
execve() - Vector + environment

All replace current process image with new program
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void exec_examples() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Method 1: execl (list of arguments)
        execl("/bin/echo", "echo", "Hello", "World", NULL);
        perror("execl failed");
        exit(1);
    }
    wait(NULL);
    
    pid = fork();
    if (pid == 0) {
        // Method 2: execvp (array of arguments, search PATH)
        char *args[] = {"echo", "Using", "execvp", NULL};
        execvp("echo", args);
        perror("execvp failed");
        exit(1);
    }
    wait(NULL);
    
    pid = fork();
    if (pid == 0) {
        // Method 3: execle (with environment)
        char *env[] = {"USER=testuser", "PATH=/bin:/usr/bin", NULL};
        execle("/usr/bin/env", "env", NULL, env);
        perror("execle failed");
        exit(1);
    }
    wait(NULL);
}

int main() {
    exec_examples();
    return 0;
}
```

---

## 3. Inter-Process Communication (IPC)

### 3.1 Pipes

**Pipe Concept:**
```
Process A ──→ [Pipe] ──→ Process B
          write     read

Pipe: Unidirectional communication channel
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

void simple_pipe() {
    int pipefd[2];  // pipefd[0] = read, pipefd[1] = write
    
    if (pipe(pipefd) == -1) {
        perror("pipe failed");
        exit(1);
    }
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process - reads from pipe
        close(pipefd[1]);  // Close write end
        
        char buffer[100];
        read(pipefd[0], buffer, sizeof(buffer));
        printf("Child received: %s\n", buffer);
        
        close(pipefd[0]);
        exit(0);
    } else {
        // Parent process - writes to pipe
        close(pipefd[0]);  // Close read end
        
        const char *msg = "Hello from parent!";
        write(pipefd[1], msg, strlen(msg) + 1);
        
        close(pipefd[1]);
        wait(NULL);
    }
}

void pipe_command_pipeline() {
    // Simulate: ls | wc -l
    int pipefd[2];
    pipe(pipefd);
    
    if (fork() == 0) {
        // First child: ls
        close(pipefd[0]);  // Close read end
        dup2(pipefd[1], STDOUT_FILENO);  // Redirect stdout to pipe
        close(pipefd[1]);
        
        execlp("ls", "ls", NULL);
        perror("execlp ls failed");
        exit(1);
    }
    
    if (fork() == 0) {
        // Second child: wc -l
        close(pipefd[1]);  // Close write end
        dup2(pipefd[0], STDIN_FILENO);  // Redirect stdin from pipe
        close(pipefd[0]);
        
        execlp("wc", "wc", "-l", NULL);
        perror("execlp wc failed");
        exit(1);
    }
    
    // Parent closes both ends and waits
    close(pipefd[0]);
    close(pipefd[1]);
    wait(NULL);
    wait(NULL);
}

void bidirectional_pipe() {
    int pipe1[2], pipe2[2];
    
    pipe(pipe1);  // Parent → Child
    pipe(pipe2);  // Child → Parent
    
    if (fork() == 0) {
        // Child
        close(pipe1[1]);  // Close write of pipe1
        close(pipe2[0]);  // Close read of pipe2
        
        char buffer[100];
        read(pipe1[0], buffer, sizeof(buffer));
        printf("Child received: %s\n", buffer);
        
        const char *response = "Child's response";
        write(pipe2[1], response, strlen(response) + 1);
        
        close(pipe1[0]);
        close(pipe2[1]);
        exit(0);
    } else {
        // Parent
        close(pipe1[0]);
        close(pipe2[1]);
        
        const char *msg = "Parent's message";
        write(pipe1[1], msg, strlen(msg) + 1);
        
        char buffer[100];
        read(pipe2[0], buffer, sizeof(buffer));
        printf("Parent received: %s\n", buffer);
        
        close(pipe1[1]);
        close(pipe2[0]);
        wait(NULL);
    }
}

int main() {
    printf("=== Simple Pipe ===\n");
    simple_pipe();
    
    printf("\n=== Command Pipeline ===\n");
    pipe_command_pipeline();
    
    printf("\n=== Bidirectional Pipe ===\n");
    bidirectional_pipe();
    
    return 0;
}
```

### 3.2 Shared Memory

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/wait.h>

#define SHM_SIZE 1024

void shared_memory_example() {
    // Create shared memory segment
    key_t key = ftok(".", 'A');
    int shmid = shmget(key, SHM_SIZE, IPC_CREAT | 0666);
    
    if (shmid == -1) {
        perror("shmget failed");
        exit(1);
    }
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process - writes to shared memory
        char *shm = (char *)shmat(shmid, NULL, 0);
        if (shm == (char *)-1) {
            perror("shmat failed");
            exit(1);
        }
        
        sprintf(shm, "Hello from child process!");
        printf("Child wrote: %s\n", shm);
        
        shmdt(shm);  // Detach
        exit(0);
    } else {
        // Parent process - reads from shared memory
        wait(NULL);  // Wait for child to write
        
        char *shm = (char *)shmat(shmid, NULL, 0);
        if (shm == (char *)-1) {
            perror("shmat failed");
            exit(1);
        }
        
        printf("Parent read: %s\n", shm);
        
        shmdt(shm);
        shmctl(shmid, IPC_RMID, NULL);  // Delete shared memory
    }
}

int main() {
    shared_memory_example();
    return 0;
}
```

### 3.3 Message Queues

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <unistd.h>

struct message {
    long msg_type;
    char text[100];
};

void message_queue_example() {
    key_t key = ftok(".", 'B');
    int msgid = msgget(key, IPC_CREAT | 0666);
    
    if (msgid == -1) {
        perror("msgget failed");
        exit(1);
    }
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child - sends message
        struct message msg;
        msg.msg_type = 1;
        strcpy(msg.text, "Hello from message queue!");
        
        if (msgsnd(msgid, &msg, sizeof(msg.text), 0) == -1) {
            perror("msgsnd failed");
            exit(1);
        }
        
        printf("Child sent: %s\n", msg.text);
        exit(0);
    } else {
        // Parent - receives message
        wait(NULL);
        
        struct message msg;
        if (msgrcv(msgid, &msg, sizeof(msg.text), 1, 0) == -1) {
            perror("msgrcv failed");
            exit(1);
        }
        
        printf("Parent received: %s\n", msg.text);
        
        msgctl(msgid, IPC_RMID, NULL);  // Delete queue
    }
}

int main() {
    message_queue_example();
    return 0;
}
```

---

## 4. Signals

**Common Signals:**
```
SIGINT  (2)  - Interrupt (Ctrl+C)
SIGTERM (15) - Termination request
SIGKILL (9)  - Kill (cannot be caught)
SIGSEGV (11) - Segmentation fault
SIGCHLD (17) - Child process terminated
SIGALRM (14) - Alarm clock
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

volatile sig_atomic_t keep_running = 1;

void signal_handler(int signum) {
    printf("\nReceived signal %d\n", signum);
    
    if (signum == SIGINT) {
        printf("Ctrl+C pressed. Exiting gracefully...\n");
        keep_running = 0;
    }
}

void basic_signal_handling() {
    // Register signal handler
    signal(SIGINT, signal_handler);
    
    printf("Running... Press Ctrl+C to stop\n");
    
    while (keep_running) {
        printf("Working...\n");
        sleep(1);
    }
    
    printf("Program terminated\n");
}

void alarm_signal() {
    void alarm_handler(int sig) {
        printf("Alarm triggered!\n");
    }
    
    signal(SIGALRM, alarm_handler);
    
    printf("Setting alarm for 3 seconds\n");
    alarm(3);
    
    // Wait for alarm
    pause();
    
    printf("After alarm\n");
}

void send_signal_between_processes() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process
        void child_handler(int sig) {
            printf("Child received signal %d\n", sig);
            exit(0);
        }
        
        signal(SIGUSR1, child_handler);
        
        printf("Child waiting for signal...\n");
        pause();
    } else {
        // Parent process
        sleep(2);
        printf("Parent sending signal to child\n");
        kill(pid, SIGUSR1);
        wait(NULL);
    }
}

void sigaction_example() {
    struct sigaction sa;
    
    void handler(int sig, siginfo_t *info, void *context) {
        printf("Signal %d from PID %d\n", sig, info->si_pid);
    }
    
    sa.sa_sigaction = handler;
    sa.sa_flags = SA_SIGINFO;
    sigemptyset(&sa.sa_mask);
    
    sigaction(SIGINT, &sa, NULL);
    
    printf("Press Ctrl+C (sigaction example)\n");
    pause();
}

int main() {
    printf("=== Basic Signal Handling ===\n");
    // basic_signal_handling();  // Uncomment to test
    
    printf("\n=== Alarm Signal ===\n");
    alarm_signal();
    
    printf("\n=== Signal Between Processes ===\n");
    send_signal_between_processes();
    
    return 0;
}
```

---

## CONTINUED IN PART 6

This is Part 5 of the Complete C Programming Master Roadmap. It covers:
- File I/O (Text, Binary, Low-Level)
- Process Management (fork, exec, wait)
- Inter-Process Communication (Pipes, Shared Memory, Message Queues)
- Signals (Handling, Sending, Advanced)

**Part 6 will include:**
- Threads and Synchronization (POSIX threads)
- Mutexes, Semaphores, Condition Variables
- Network Programming (Sockets)
- Linux Internals Deep Dive

Save this file and continue with Part 6!