# C Programming Master Roadmap: From Beginner to Ninja

**Your Complete Guide to Mastering C, Data Structures, Algorithms, Linux Internals, and Open Source**

---

## Table of Contents

1. [Introduction & Setup](#1-introduction--setup)
2. [Core C Programming](#2-core-c-programming)
3. [Memory Management](#3-memory-management)
4. [Pointers Mastery](#4-pointers-mastery)
5. [Advanced C](#5-advanced-c)
6. [Data Structures](#6-data-structures)
7. [Algorithms](#7-algorithms)
8. [File I/O & System Programming](#8-file-io--system-programming)
9. [Linux System Programming](#9-linux-system-programming)
10. [Linux Internals](#10-linux-internals)
11. [Open Source Analysis](#11-open-source-analysis)
12. [Advanced Topics](#12-advanced-topics)

---

## 1. Introduction & Setup

### 1.1 Compilation Process

```
Source (.c) → Preprocessor → Compiler → Assembler → Linker → Executable
```

**Steps:**
1. **Preprocessing**: Remove comments, expand macros, include headers
2. **Compilation**: Convert to assembly language
3. **Assembly**: Convert to machine code (object files)
4. **Linking**: Combine object files and libraries

```bash
# Compile with debugging and warnings
gcc -Wall -Wextra -g -std=c11 program.c -o program

# See preprocessing output
gcc -E program.c -o program.i

# See assembly code
gcc -S program.c -o program.s

# Create object file
gcc -c program.c -o program.o

# Link object files
gcc program.o helper.o -o program
```

---

## 2. Core C Programming

### 2.1 Data Types & Sizes

```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    // Integer types
    char c = 'A';              // 1 byte: -128 to 127
    unsigned char uc = 255;    // 1 byte: 0 to 255
    short s = 32767;           // 2 bytes
    int i = 2147483647;        // 4 bytes
    long l = 9223372036854775807L;  // 8 bytes
    
    // Floating point
    float f = 3.14f;           // 4 bytes (7 digits precision)
    double d = 3.14159265359;  // 8 bytes (15 digits precision)
    
    printf("Sizes: char=%zu int=%zu float=%zu double=%zu\n",
           sizeof(char), sizeof(int), sizeof(float), sizeof(double));
    
    return 0;
}
```

### 2.2 Bitwise Operations

```c
// Bitwise operators
unsigned int a = 60;  // 0011 1100
unsigned int b = 13;  // 0000 1101

a & b;   // AND: 0000 1100 = 12
a | b;   // OR:  0011 1101 = 61
a ^ b;   // XOR: 0011 0001 = 49
~a;      // NOT: 1100 0011
a << 2;  // Left shift: 1111 0000 = 240
a >> 2;  // Right shift: 0000 1111 = 15

// Practical applications
#define SET_BIT(n, pos) ((n) | (1 << (pos)))
#define CLEAR_BIT(n, pos) ((n) & ~(1 << (pos)))
#define TOGGLE_BIT(n, pos) ((n) ^ (1 << (pos)))
#define CHECK_BIT(n, pos) (((n) >> (pos)) & 1)

// Check if power of 2
int is_power_of_2(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}

// Count set bits (Brian Kernighan's algorithm)
int count_set_bits(int n) {
    int count = 0;
    while (n) {
        n &= (n - 1);
        count++;
    }
    return count;
}

// Swap without temp variable
void swap_xor(int *a, int *b) {
    if (a != b) {
        *a ^= *b;
        *b ^= *a;
        *a ^= *b;
    }
}
```

### 2.3 Control Flow

```c
// if-else-if ladder
if (score >= 90) {
    printf("A\n");
} else if (score >= 80) {
    printf("B\n");
} else {
    printf("C\n");
}

// switch-case
switch (choice) {
    case 1:
        // code
        break;
    case 2:
        // code
        break;
    default:
        // code
}

// Loops
for (int i = 0; i < 10; i++) { }
while (condition) { }
do { } while (condition);

// break, continue, goto
for (int i = 0; i < 100; i++) {
    if (i % 2 == 0) continue;  // Skip even
    if (i > 50) break;         // Exit loop
}

// goto (use sparingly)
error_cleanup:
    free(ptr);
    return -1;
```

### 2.4 Functions

```c
// Function declaration
int add(int a, int b);

// Function definition
int add(int a, int b) {
    return a + b;
}

// Pass by value vs pass by reference
void modify_value(int x) {
    x = 10;  // Original unchanged
}

void modify_reference(int *x) {
    *x = 10;  // Original changed
}

// Recursion
int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

// Tail recursion (optimizable)
int factorial_tail(int n, int acc) {
    return (n <= 1) ? acc : factorial_tail(n - 1, n * acc);
}
```

---

## 3. Memory Management

### 3.1 Memory Layout

```
High Address
┌─────────────────┐
│ Cmd Args/Env    │
├─────────────────┤
│ Stack ↓         │  Local vars, function calls
├─────────────────┤
│ ↑ Heap          │  Dynamic allocation (malloc)
├─────────────────┤
│ BSS (uninit)    │  Uninitialized global/static
├─────────────────┤
│ Data (init)     │  Initialized global/static
├─────────────────┤
│ Text (code)     │  Program instructions
└─────────────────┘
Low Address
```

### 3.2 Dynamic Memory

```c
#include <stdlib.h>

// malloc - allocate uninitialized memory
int *arr = (int *)malloc(10 * sizeof(int));
if (arr == NULL) {
    // Handle error
}

// calloc - allocate zero-initialized memory
int *arr2 = (int *)calloc(10, sizeof(int));

// realloc - resize memory
arr = (int *)realloc(arr, 20 * sizeof(int));

// free - deallocate memory
free(arr);
arr = NULL;  // Avoid dangling pointer

// Dynamic 2D array
int **matrix = (int **)malloc(rows * sizeof(int *));
for (int i = 0; i < rows; i++) {
    matrix[i] = (int *)malloc(cols * sizeof(int));
}

// Free 2D array
for (int i = 0; i < rows; i++) {
    free(matrix[i]);
}
free(matrix);
```

### 3.3 Common Memory Errors

```c
// 1. Memory leak
void leak() {
    int *p = malloc(sizeof(int));
    // Forgot free(p);
}

// 2. Use after free
int *p = malloc(sizeof(int));
free(p);
*p = 10;  // BAD!

// 3. Double free
free(p);
free(p);  // BAD!

// 4. Dangling pointer
int *func() {
    int x = 10;
    return &x;  // BAD! x is destroyed
}

// 5. Buffer overflow
char buf[10];
strcpy(buf, "This is too long");  // BAD!
```

---

## 4. Pointers Mastery

### 4.1 Pointer Basics

```c
int value = 42;
int *ptr = &value;  // ptr stores address of value

printf("Value: %d\n", value);          // 42
printf("Address: %p\n", (void*)&value);
printf("Pointer: %p\n", (void*)ptr);
printf("Dereference: %d\n", *ptr);     // 42

*ptr = 100;  // Modify through pointer
printf("Value: %d\n", value);          // 100

// Pointer arithmetic
int arr[] = {10, 20, 30, 40};
int *p = arr;

printf("%d\n", *p);       // 10
printf("%d\n", *(p+1));   // 20
printf("%d\n", p[2]);     // 30

p++;
printf("%d\n", *p);       // 20
```

### 4.2 Advanced Pointer Types

```c
// Pointer to pointer
int val = 42;
int *p = &val;
int **pp = &p;
printf("%d\n", **pp);  // 42

// Void pointer (generic)
void *vp;
int x = 10;
vp = &x;
printf("%d\n", *(int*)vp);

// Function pointer
int add(int a, int b) { return a + b; }
int (*func_ptr)(int, int) = add;
printf("%d\n", func_ptr(5, 3));  // 8

// Array of function pointers
int (*ops[4])(int, int) = {add, sub, mul, div};
int result = ops[0](10, 5);  // Calls add

// Const pointers
const int *p1;      // Pointer to const data
int *const p2;      // Const pointer
const int *const p3; // Const pointer to const data
```

### 4.3 Pointer & Arrays

```c
int arr[] = {1, 2, 3, 4, 5};

// These are equivalent
arr[2];
*(arr + 2);
2[arr];  // Yes, this works!

// Pointer to array
int (*ptr)[5] = &arr;  // Points to entire array

// Array of pointers
int *ptrs[5];

// 2D array with pointers
int matrix[3][4];
int (*p)[4] = matrix;  // Pointer to array of 4 ints
printf("%d\n", *(*(p + 1) + 2));  // matrix[1][2]
```

---

## 5. Advanced C

### 5.1 Structures

```c
// Structure definition
struct Person {
    char name[50];
    int age;
    float salary;
};

// Initialization
struct Person p1 = {"John", 30, 50000.0};

// Designated initializers (C99)
struct Person p2 = {
    .name = "Jane",
    .age = 25,
    .salary = 60000.0
};

// Nested structures
struct Date {
    int day, month, year;
};

struct Employee {
    char name[50];
    struct Date join_date;
};

// Structure padding
struct Example {
    char c;    // 1 byte + 3 padding
    int i;     // 4 bytes
    short s;   // 2 bytes + 2 padding
};  // Total: 12 bytes

// Packed structure (no padding)
struct __attribute__((packed)) Packed {
    char c;
    int i;
    short s;
};  // Total: 7 bytes
```

### 5.2 Unions

```c
// Union - all members share same memory
union Data {
    int i;
    float f;
    char str[20];
};

union Data d;
d.i = 10;
printf("%d\n", d.i);

d.f = 3.14;  // Overwrites i
printf("%f\n", d.f);

// Tagged union (discriminated)
typedef struct {
    enum { INT, FLOAT, STRING } type;
    union {
        int i;
        float f;
        char *str;
    } value;
} Variant;

void print_variant(Variant *v) {
    switch (v->type) {
        case INT: printf("%d\n", v->value.i); break;
        case FLOAT: printf("%f\n", v->value.f); break;
        case STRING: printf("%s\n", v->value.str); break;
    }
}
```

### 5.3 Enums & Typedef

```c
// Enumeration
enum Color {
    RED,      // 0
    GREEN,    // 1
    BLUE      // 2
};

enum Status {
    OK = 0,
    ERROR = -1,
    NOT_FOUND = -2
};

// Bit flags
enum Permissions {
    READ = 1 << 0,    // 1
    WRITE = 1 << 1,   // 2
    EXECUTE = 1 << 2  // 4
};

unsigned int perms = READ | WRITE;
if (perms & WRITE) {
    printf("Can write\n");
}

// Typedef
typedef unsigned long long ull;
typedef struct Point {
    int x, y;
} Point;

typedef int (*MathFunc)(int, int);
```

### 5.4 Preprocessor

```c
// Object macros
#define PI 3.14159
#define MAX_SIZE 100

// Function macros
#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))

// Multi-line macros
#define SWAP(a, b, T) do { \
    T temp = (a); \
    (a) = (b); \
    (b) = temp; \
} while(0)

// Stringification
#define STR(x) #x
printf("%s\n", STR(hello));  // Prints: hello

// Token pasting
#define CONCAT(a, b) a##b
int CONCAT(var, 123) = 10;  // Creates var123

// Variadic macros
#define DEBUG(...) fprintf(stderr, __VA_ARGS__)
DEBUG("Error: %d\n", errno);

// Conditional compilation
#ifdef DEBUG
    printf("Debug mode\n");
#endif

#ifndef MAX_SIZE
    #define MAX_SIZE 100
#endif

// Predefined macros
__FILE__    // Current file name
__LINE__    // Current line number
__DATE__    // Compilation date
__TIME__    // Compilation time
__func__    // Current function name (C99)

// Include guards
#ifndef HEADER_H
#define HEADER_H
// Header contents
#endif
```

---

## 6. Data Structures

### 6.1 Dynamic Array (Vector)

```c
typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} Vector;

Vector* vector_create() {
    Vector *v = malloc(sizeof(Vector));
    v->size = 0;
    v->capacity = 4;
    v->data = malloc(v->capacity * sizeof(int));
    return v;
}

void vector_push(Vector *v, int value) {
    if (v->size >= v->capacity) {
        v->capacity *= 2;
        v->data = realloc(v->data, v->capacity * sizeof(int));
    }
    v->data[v->size++] = value;
}

int vector_pop(Vector *v) {
    return v->data[--v->size];
}

void vector_free(Vector *v) {
    free(v->data);
    free(v);
}
```

### 6.2 Linked List

```c
typedef struct Node {
    int data;
    struct Node *next;
} Node;

typedef struct {
    Node *head;
    size_t size;
} LinkedList;

void list_insert_front(LinkedList *list, int data) {
    Node *new = malloc(sizeof(Node));
    new->data = data;
    new->next = list->head;
    list->head = new;
    list->size++;
}

void list_insert_back(LinkedList *list, int data) {
    Node *new = malloc(sizeof(Node));
    new->data = data;
    new->next = NULL;
    
    if (!list->head) {
        list->head = new;
    } else {
        Node *curr = list->head;
        while (curr->next) curr = curr->next;
        curr->next = new;
    }
    list->size++;
}

void list_delete(LinkedList *list, int value) {
    if (!list->head) return;
    
    if (list->head->data == value) {
        Node *temp = list->head;
        list->head = list->head->next;
        free(temp);
        list->size--;
        return;
    }
    
    Node *curr = list->head;
    while (curr->next && curr->next->data != value) {
        curr = curr->next;
    }
    
    if (curr->next) {
        Node *temp = curr->next;
        curr->next = curr->next->next;
        free(temp);
        list->size--;
    }
}

void list_reverse(LinkedList *list) {
    Node *prev = NULL, *curr = list->head, *next;
    while (curr) {
        next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    list->head = prev;
}

// Detect cycle (Floyd's algorithm)
int list_has_cycle(Node *head) {
    Node *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return 1;
    }
    return 0;
}

// Find middle
Node* list_find_middle(Node *head) {
    Node *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
```

### 6.3 Stack

```c
#define MAX 100

typedef struct {
    int data[MAX];
    int top;
} Stack;

void stack_init(Stack *s) {
    s->top = -1;
}

int stack_is_empty(Stack *s) {
    return s->top == -1;
}

int stack_is_full(Stack *s) {
    return s->top == MAX - 1;
}

void stack_push(Stack *s, int value) {
    if (!stack_is_full(s)) {
        s->data[++s->top] = value;
    }
}

int stack_pop(Stack *s) {
    if (!stack_is_empty(s)) {
        return s->data[s->top--];
    }
    return -1;
}

int stack_peek(Stack *s) {
    return stack_is_empty(s) ? -1 : s->data[s->top];
}

// Application: Balanced parentheses
int is_balanced(const char *expr) {
    Stack s;
    stack_init(&s);
    
    for (int i = 0; expr[i]; i++) {
        char ch = expr[i];
        if (ch == '(' || ch == '[' || ch == '{') {
            stack_push(&s, ch);
        } else if (ch == ')' || ch == ']' || ch == '}') {
            if (stack_is_empty(&s)) return 0;
            char top = stack_pop(&s);
            if ((ch == ')' && top != '(') ||
                (ch == ']' && top != '[') ||
                (ch == '}' && top != '{')) {
                return 0;
            }
        }
    }
    return stack_is_empty(&s);
}
```

### 6.4 Queue

```c
#define MAX 100

typedef struct {
    int data[MAX];
    int front, rear, size;
} Queue;

void queue_init(Queue *q) {
    q->front = 0;
    q->rear = -1;
    q->size = 0;
}

int queue_is_empty(Queue *q) {
    return q->size == 0;
}

int queue_is_full(Queue *q) {
    return q->size == MAX;
}

void queue_enqueue(Queue *q, int value) {
    if (!queue_is_full(q)) {
        q->rear = (q->rear + 1) % MAX;
        q->data[q->rear] = value;
        q->size++;
    }
}

int queue_dequeue(Queue *q) {
    if (!queue_is_empty(q)) {
        int value = q->data[q->front];
        q->front = (q->front + 1) % MAX;
        q->size--;
        return value;
    }
    return -1;
}
```

### 6.5 Binary Tree

```c
typedef struct TreeNode {
    int data;
    struct TreeNode *left, *right;
} TreeNode;

TreeNode* create_node(int data) {
    TreeNode *node = malloc(sizeof(TreeNode));
    node->data = data;
    node->left = node->right = NULL;
    return node;
}

// Traversals
void inorder(TreeNode *root) {
    if (root) {
        inorder(root->left);
        printf("%d ", root->data);
        inorder(root->right);
    }
}

void preorder(TreeNode *root) {
    if (root) {
        printf("%d ", root->data);
        preorder(root->left);
        preorder(root->right);
    }
}

void postorder(TreeNode *root) {
    if (root) {
        postorder(root->left);
        postorder(root->right);
        printf("%d ", root->data);
    }
}

// Height
int tree_height(TreeNode *root) {
    if (!root) return 0;
    int lh = tree_height(root->left);
    int rh = tree_height(root->right);
    return 1 + (lh > rh ? lh : rh);
}

// Count nodes
int count_nodes(TreeNode *root) {
    return root ? 1 + count_nodes(root->left) + count_nodes(root->right) : 0;
}
```

### 6.6 Binary Search Tree

```c
TreeNode* bst_insert(TreeNode *root, int data) {
    if (!root) return create_node(data);
    
    if (data < root->data)
        root->left = bst_insert(root->left, data);
    else if (data > root->data)
        root->right = bst_insert(root->right, data);
    
    return root;
}

TreeNode* bst_search(TreeNode *root, int data) {
    if (!root || root->data == data) return root;
    return (data < root->data) ? 
           bst_search(root->left, data) : 
           bst_search(root->right, data);
}

TreeNode* bst_min(TreeNode *root) {
    while (root && root->left) root = root->left;
    return root;
}

TreeNode* bst_delete(TreeNode *root, int data) {
    if (!root) return NULL;
    
    if (data < root->data) {
        root->left = bst_delete(root->left, data);
    } else if (data > root->data) {
        root->right = bst_delete(root->right, data);
    } else {
        if (!root->left) {
            TreeNode *temp = root->right;
            free(root);
            return temp;
        } else if (!root->right) {
            TreeNode *temp = root->left;
            free(root);
            return temp;
        }
        
        TreeNode *temp = bst_min(root->right);
        root->data = temp->data;
        root->right = bst_delete(root->right, temp->data);
    }
    return root;
}
```

### 6.7 Hash Table

```c
#define TABLE_SIZE 100

typedef struct Entry {
    char *key;
    int value;
    struct Entry *next;
} Entry;

typedef struct {
    Entry *table[TABLE_SIZE];
} HashTable;

unsigned int hash(const char *key) {
    unsigned int hash = 0;
    while (*key) {
        hash = (hash << 5) + *key++;
    }
    return hash % TABLE_SIZE;
}

void ht_insert(HashTable *ht, const char *key, int value) {
    unsigned int index = hash(key);
    Entry *entry = malloc(sizeof(Entry));
    entry->key = strdup(key);
    entry->value = value;
    entry->next = ht->table[index];
    ht->table[index] = entry;
}

int ht_search(HashTable *ht, const char *key, int *value) {
    unsigned int index = hash(key);
    Entry *entry = ht->table[index];
    
    while (entry) {
        if (strcmp(entry->key, key) == 0) {
            *value = entry->value;
            return 1;
        }
        entry = entry->next;
    }
    return 0;
}

void ht_delete(HashTable *ht, const char *key) {
    unsigned int index = hash(key);
    Entry *entry = ht->table[index];
    Entry *prev = NULL;
    
    while (entry) {
        if (strcmp(entry->key, key) == 0) {
            if (prev) {
                prev->next = entry->next;
            } else {
                ht->table[index] = entry->next;
            }
            free(entry->key);
            free(entry);
            return;
        }
        prev = entry;
        entry = entry->next;
    }
}
```

### 6.8 Graph

```c
#define MAX_VERTICES 100

// Adjacency Matrix
typedef struct {
    int matrix[MAX_VERTICES][MAX_VERTICES];
    int num_vertices;
} GraphMatrix;

void graph_add_edge_matrix(GraphMatrix *g, int src, int dest) {
    g->matrix[src][dest] = 1;
    g->matrix[dest][src] = 1;  // For undirected
}

// Adjacency List
typedef struct AdjNode {
    int dest;
    struct AdjNode *next;
} AdjNode;

typedef struct {
    AdjNode *head[MAX_VERTICES];
    int num_vertices;
} GraphList;

void graph_add_edge_list(GraphList *g, int src, int dest) {
    AdjNode *node = malloc(sizeof(AdjNode));
    node->dest = dest;
    node->next = g->head[src];
    g->head[src] = node;
    
    // For undirected
    node = malloc(sizeof(AdjNode));
    node->dest = src;
    node->next = g->head[dest];
    g->head[dest] = node;
}

// DFS
void dfs_util(GraphList *g, int v, int visited[]) {
    visited[v] = 1;
    printf("%d ", v);
    
    AdjNode *node = g->head[v];
    while (node) {
        if (!visited[node->dest]) {
            dfs_util(g, node->dest, visited);
        }
        node = node->next;
    }
}

void dfs(GraphList *g, int start) {
    int visited[MAX_VERTICES] = {0};
    dfs_util(g, start, visited);
}

// BFS
void bfs(GraphList *g, int start) {
    int visited[MAX_VERTICES] = {0};
    Queue q;
    queue_init(&q);
    
    visited[start] = 1;
    queue_enqueue(&q, start);
    
    while (!queue_is_empty(&q)) {
        int v = queue_dequeue(&q);
        printf("%d ", v);
        
        AdjNode *node = g->head[v];
        while (node) {
            if (!visited[node->dest]) {
                visited[node->dest] = 1;
                queue_enqueue(&q, node->dest);
            }
            node = node->next;
        }
    }
}
```

---

## 7. Algorithms

### 7.1 Sorting Algorithms

#### Bubble Sort
```c
void bubble_sort(int arr[], int n) {
    for (int i = 0; i < n-1; i++) {
        int swapped = 0;
        for (int j = 0; j < n-i-1; j++) {
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
                swapped = 1;
            }
        }
        if (!swapped) break;  // Already sorted
    }
}
// Time: O(n²), Space: O(1)
```

#### Selection Sort
```c
void selection_sort(int arr[], int n) {
    for (int i = 0; i < n-1; i++) {
        int min_idx = i;
        for (int j = i+1; j < n; j++) {
            if (arr[j] < arr[min_idx]) {
                min_idx = j;
            }
        }
        int temp = arr[i];
        arr[i] = arr[min_idx];
        arr[min_idx] = temp;
    }
}
// Time: O(n²), Space: O(1)
```

#### Insertion Sort
```c
void insertion_sort(int arr[], int n) {
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j+1] = arr[j];
            j--;
        }
        arr[j+1] = key;
    }
}
// Time: O(n²), Space: O(1)
// Best for small/nearly sorted arrays
```

#### Merge Sort
```c
void merge(int arr[], int l, int m, int r) {
    int n1 = m - l + 1;
    int n2 = r - m;
    
    int L[n1], R[n2];
    for (int i = 0; i < n1; i++) L[i] = arr[l + i];
    for (int i = 0; i < n2; i++) R[i] = arr[m + 1 + i];
    
    int i = 0, j = 0, k = l;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }
    
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void merge_sort(int arr[], int l, int r) {
    if (l < r) {
        int m = l + (r - l) / 2;
        merge_sort(arr, l, m);
        merge_sort(arr, m + 1, r);
        merge(arr, l, m, r);
    }
}
// Time: O(n log n), Space: O(n)
// Stable, good for linked lists
```

#### Quick Sort
```c
int partition(int arr[], int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }
    
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    return i + 1;
}

void quick_sort(int arr[], int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quick_sort(arr, low, pi - 1);
        quick_sort(arr, pi + 1, high);
    }
}
// Time: O(n log n) average, O(n²) worst
// Space: O(log n)
// In-place, cache-friendly
```

#### Heap Sort
```c
void heapify(int arr[], int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    
    if (left < n && arr[left] > arr[largest])
        largest = left;
    
    if (right < n && arr[right] > arr[largest])
        largest = right;
    
    if (largest != i) {
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;
        heapify(arr, n, largest);
    }
}

void heap_sort(int arr[], int n) {
    // Build heap
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);
    
    // Extract elements
    for (int i = n - 1; i > 0; i--) {
        int temp = arr[0];
        arr[0] = arr[i];
        arr[i] = temp;
        heapify(arr, i, 0);
    }
}
// Time: O(n log n), Space: O(1)
```

#### Counting Sort
```c
void counting_sort(int arr[], int n) {
    int max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] > max) max = arr[i];
    }
    
    int *count = calloc(max + 1, sizeof(int));
    int *output = malloc(n * sizeof(int));
    
    for (int i = 0; i < n; i++) {
        count[arr[i]]++;
    }
    
    for (int i = 1; i <= max; i++) {
        count[i] += count[i - 1];
    }
    
    for (int i = n - 1; i >= 0; i--) {
        output[count[arr[i]] - 1] = arr[i];
        count[arr[i]]--;
    }
    
    for (int i = 0; i < n; i++) {
        arr[i] = output[i];
    }
    
    free(count);
    free(output);
}
// Time: O(n + k), Space: O(k)
// Where k is range of input
```

### 7.2 Searching Algorithms

#### Linear Search
```c
int linear_search(int arr[], int n, int key) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == key) return i;
    }
    return -1;
}
// Time: O(n)
```

#### Binary Search
```c
int binary_search(int arr[], int n, int key) {
    int left = 0, right = n - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == key) return mid;
        if (arr[mid] < key) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
// Time: O(log n) - Array must be sorted
```

#### Binary Search (Recursive)
```c
int binary_search_rec(int arr[], int l, int r, int key) {
    if (l <= r) {
        int mid = l + (r - l) / 2;
        
        if (arr[mid] == key) return mid;
        if (arr[mid] > key) 
            return binary_search_rec(arr, l, mid - 1, key);
        return binary_search_rec(arr, mid + 1, r, key);
    }
    return -1;
}
```

### 7.3 Dynamic Programming

#### Fibonacci (Memoization)
```c
int fib_memo(int n, int memo[]) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo);
    return memo[n];
}
```

#### Longest Common Subsequence
```c
int lcs(char *X, char *Y, int m, int n) {
    int dp[m+1][n+1];
    
    for (int i = 0; i <= m; i++) {
        for (int j = 0; j <= n; j++) {
            if (i == 0 || j == 0) {
                dp[i][j] = 0;
            } else if (X[i-1] == Y[j-1]) {
                dp[i][j] = dp[i-1][j-1] + 1;
            } else {
                dp[i][j] = (dp[i-1][j] > dp[i][j-1]) ? 
                           dp[i-1][j] : dp[i][j-1];
            }
        }
    }
    return dp[m][n];
}
```

#### Knapsack Problem
```c
int knapsack(int W, int wt[], int val[], int n) {
    int dp[n+1][W+1];
    
    for (int i = 0; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            if (i == 0 || w == 0) {
                dp[i][w] = 0;
            } else if (wt[i-1] <= w) {
                int include = val[i-1] + dp[i-1][w - wt[i-1]];
                int exclude = dp[i-1][w];
                dp[i][w] = (include > exclude) ? include : exclude;
            } else {
                dp[i][w] = dp[i-1][w];
            }
        }
    }
    return dp[n][W];
}
```

### 7.4 Graph Algorithms

#### Dijkstra's Shortest Path
```c
#define INF 999999

void dijkstra(int graph[MAX][MAX], int n, int src) {
    int dist[MAX];
    int visited[MAX] = {0};
    
    for (int i = 0; i < n; i++) {
        dist[i] = INF;
    }
    dist[src] = 0;
    
    for (int count = 0; count < n - 1; count++) {
        int min = INF, min_idx;
        
        for (int v = 0; v < n; v++) {
            if (!visited[v] && dist[v] < min) {
                min = dist[v];
                min_idx = v;
            }
        }
        
        int u = min_idx;
        visited[u] = 1;
        
        for (int v = 0; v < n; v++) {
            if (!visited[v] && graph[u][v] && 
                dist[u] + graph[u][v] < dist[v]) {
                dist[v] = dist[u] + graph[u][v];
            }
        }
    }
}
```

#### Floyd-Warshall (All Pairs Shortest Path)
```c
void floyd_warshall(int graph[MAX][MAX], int n) {
    int dist[MAX][MAX];
    
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            dist[i][j] = graph[i][j];
        }
    }
    
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
}
```

---

## 8. File I/O & System Programming

### 8.1 File Operations

```c
#include <stdio.h>

// Write to file
void write_file() {
    FILE *fp = fopen("data.txt", "w");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    fprintf(fp, "Hello, World!\n");
    fprintf(fp, "Number: %d\n", 42);
    fclose(fp);
}

// Read from file
void read_file() {
    FILE *fp = fopen("data.txt", "r");
    if (fp == NULL) {
        perror("Error");
        return;
    }
    
    char buffer[256];
    while (fgets(buffer, sizeof(buffer), fp)) {
        printf("%s", buffer);
    }
    fclose(fp);
}

// Binary file I/O
typedef struct {
    int id;
    char name[50];
    float salary;
} Employee;

void write_binary() {
    FILE *fp = fopen("employees.dat", "wb");
    Employee e = {1, "John", 50000.0};
    fwrite(&e, sizeof(Employee), 1, fp);
    fclose(fp);
}

void read_binary() {
    FILE *fp = fopen("employees.dat", "rb");
    Employee e;
    while (fread(&e, sizeof(Employee), 1, fp)) {
        printf("ID: %d, Name: %s, Salary: %.2f\n", 
               e.id, e.name, e.salary);
    }
    fclose(fp);
}

// File positioning
void file_seek_example() {
    FILE *fp = fopen("data.txt", "r");
    
    fseek(fp, 0, SEEK_END);   // Go to end
    long size = ftell(fp);     // Get position
    fseek(fp, 0, SEEK_SET);    // Go to start
    
    rewind(fp);  // Reset to beginning
    fclose(fp);
}
```

### 8.2 Command Line Arguments

```c
int main(int argc, char *argv[]) {
    printf("Program name: %s\n", argv[0]);
    printf("Arguments: %d\n", argc - 1);
    
    for (int i = 1; i < argc; i++) {
        printf("Arg %d: %s\n", i, argv[i]);
    }
    
    return 0;
}

// Usage: ./program arg1 arg2 arg3
```

---

## 9. Linux System Programming

### 9.1 Process Management

```c
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

// Fork - create child process
void fork_example() {
    pid_t pid = fork();
    
    if (pid < 0) {
        perror("Fork failed");
    } else if (pid == 0) {
        // Child process
        printf("Child PID: %d\n", getpid());
        printf("Parent PID: %d\n", getppid());
    } else {
        // Parent process
        printf("Parent PID: %d\n", getpid());
        printf("Child PID: %d\n", pid);
        wait(NULL);  // Wait for child
    }
}

// Exec - replace process image
void exec_example() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Execute new program
        execl("/bin/ls", "ls", "-l", NULL);
        perror("execl failed");  // Only if execl fails
    } else {
        wait(NULL);
    }
}

// System call
int ret = system("ls -l");
```

### 9.2 Inter-Process Communication (IPC)

#### Pipes
```c
#include <unistd.h>

void pipe_example() {
    int fd[2];  // fd[0] = read, fd[1] = write
    pipe(fd);
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child - write to pipe
        close(fd[0]);
        char msg[] = "Hello from child";
        write(fd[1], msg, sizeof(msg));
        close(fd[1]);
    } else {
        // Parent - read from pipe
        close(fd[1]);
        char buffer[100];
        read(fd[0], buffer, sizeof(buffer));
        printf("Received: %s\n", buffer);
        close(fd[0]);
    }
}
```

#### Shared Memory
```c
#include <sys/shm.h>
#include <sys/ipc.h>

void shared_memory_example() {
    // Create shared memory
    key_t key = ftok("shmfile", 65);
    int shmid = shmget(key, 1024, 0666|IPC_CREAT);
    
    // Attach to shared memory
    char *str = (char*) shmat(shmid, NULL, 0);
    
    // Write to shared memory
    sprintf(str, "Hello from shared memory");
    
    // Detach
    shmdt(str);
    
    // Destroy shared memory
    shmctl(shmid, IPC_RMID, NULL);
}
```

#### Message Queues
```c
#include <sys/msg.h>

struct message {
    long msg_type;
    char msg_text[100];
};

void message_queue_example() {
    key_t key = ftok("msgfile", 65);
    int msgid = msgget(key, 0666 | IPC_CREAT);
    
    struct message msg;
    msg.msg_type = 1;
    strcpy(msg.msg_text, "Hello");
    
    // Send message
    msgsnd(msgid, &msg, sizeof(msg.msg_text), 0);
    
    // Receive message
    msgrcv(msgid, &msg, sizeof(msg.msg_text), 1, 0);
    printf("Received: %s\n", msg.msg_text);
    
    // Destroy queue
    msgctl(msgid, IPC_RMID, NULL);
}
```

### 9.3 Threads (POSIX)

```c
#include <pthread.h>

void* thread_function(void *arg) {
    int id = *(int*)arg;
    printf("Thread %d running\n", id);
    return NULL;
}

int main() {
    pthread_t threads[5];
    int ids[5];
    
    // Create threads
    for (int i = 0; i < 5; i++) {
        ids[i] = i;
        pthread_create(&threads[i], NULL, thread_function, &ids[i]);
    }
    
    // Join threads
    for (int i = 0; i < 5; i++) {
        pthread_join(threads[i], NULL);
    }
    
    return 0;
}

// Mutex for synchronization
pthread_mutex_t lock;

void* safe_increment(void *arg) {
    pthread_mutex_lock(&lock);
    // Critical section
    shared_counter++;
    pthread_mutex_unlock(&lock);
    return NULL;
}
```

### 9.4 Signals

```c
#include <signal.h>

void signal_handler(int signum) {
    printf("Caught signal %d\n", signum);
    exit(signum);
}

int main() {
    // Register signal handler
    signal(SIGINT, signal_handler);  // Ctrl+C
    signal(SIGTERM, signal_handler);
    
    while (1) {
        printf("Running...\n");
        sleep(1);
    }
    
    return 0;
}

// Send signal to process
kill(pid, SIGTERM);
```

### 9.5 Network Programming (Sockets)

#### Server
```c
#include <sys/socket.h>
#include <netinet/in.h>

void tcp_server() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    
    struct sockaddr_in address;
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(8080);
    
    bind(server_fd, (struct sockaddr*)&address, sizeof(address));
    listen(server_fd, 3);
    
    int client_fd = accept(server_fd, NULL, NULL);
    
    char buffer[1024] = {0};
    read(client_fd, buffer, 1024);
    printf("Received: %s\n", buffer);
    
    char *response = "Hello from server";
    send(client_fd, response, strlen(response), 0);
    
    close(client_fd);
    close(server_fd);
}
```

#### Client
```c
void tcp_client() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    
    struct sockaddr_in serv_addr;
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(8080);
    inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr);
    
    connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    
    char *message = "Hello from client";
    send(sock, message, strlen(message), 0);
    
    char buffer[1024] = {0};
    read(sock, buffer, 1024);
    printf("Received: %s\n", buffer);
    
    close(sock);
}
```

---

## 10. Linux Internals

### 10.1 Memory Management

#### Virtual Memory Architecture

**Concepts:**
- **Virtual Address Space**: Each process sees contiguous memory (0x0 to 0x7FFFFFFFFFFFFFFF on 64-bit)
- **Physical Memory**: Actual RAM divided into frames (typically 4KB)
- **MMU (Memory Management Unit)**: Hardware that translates virtual → physical addresses
- **TLB (Translation Lookaside Buffer)**: Cache for page table entries (fast lookups)
- **Page Fault**: Exception when page not in memory (triggers OS to load from disk)

**Page Table Hierarchy (x86-64):**
```
Virtual Address (48 bits used):
[PML4 | Directory Ptr | Directory | Table | Offset]
  9bit    9bit          9bit       9bit    12bit

Translation Process:
1. PML4 index → PML4 entry
2. PDP index → Page Directory Pointer
3. PD index → Page Directory
4. PT index → Page Table Entry (PTE)
5. Offset → Physical address
```

**Memory Zones:**
```c
// From Linux kernel
ZONE_DMA      // 0-16MB (ISA devices)
ZONE_DMA32    // 0-4GB (32-bit DMA)
ZONE_NORMAL   // Normal memory
ZONE_HIGHMEM  // >896MB on 32-bit systems
ZONE_MOVABLE  // For memory hotplug
```

**Page Allocation Example:**
```c
#include <sys/mman.h>
#include <stdio.h>
#include <string.h>

int main() {
    // Anonymous mapping (no file backing)
    void *addr = mmap(NULL, 4096,
                     PROT_READ | PROT_WRITE,
                     MAP_PRIVATE | MAP_ANONYMOUS,
                     -1, 0);
    
    if (addr == MAP_FAILED) {
        perror("mmap failed");
        return 1;
    }
    
    // Use the memory
    strcpy(addr, "Hello from mmap!");
    printf("%s\n", (char*)addr);
    
    // Unmap when done
    munmap(addr, 4096);
    return 0;
}
```

**Memory-Mapped Files:**
```c
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>

void mmap_file_example() {
    int fd = open("data.txt", O_RDWR);
    if (fd < 0) {
        perror("open");
        return;
    }
    
    // Get file size
    struct stat sb;
    fstat(fd, &sb);
    
    // Map file to memory
    char *mapped = mmap(NULL, sb.st_size,
                       PROT_READ | PROT_WRITE,
                       MAP_SHARED,  // Changes reflect to file
                       fd, 0);
    
    if (mapped == MAP_FAILED) {
        perror("mmap");
        close(fd);
        return;
    }
    
    // Modify memory = modify file!
    mapped[0] = 'X';
    
    // Ensure changes written to disk
    msync(mapped, sb.st_size, MS_SYNC);
    
    munmap(mapped, sb.st_size);
    close(fd);
}
```

**Advanced mmap Flags:**
```c
// MAP_SHARED: Share mapping with other processes
void *shared = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                    MAP_SHARED | MAP_ANONYMOUS, -1, 0);

// MAP_PRIVATE: Copy-on-write (COW)
void *private = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                     MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);

// MAP_FIXED: Map at exact address (dangerous!)
void *fixed = mmap((void*)0x100000, 4096, PROT_READ | PROT_WRITE,
                    MAP_PRIVATE | MAP_ANONYMOUS | MAP_FIXED, -1, 0);

// MAP_LOCKED: Lock pages in memory (no swap)
void *locked = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                    MAP_PRIVATE | MAP_ANONYMOUS | MAP_LOCKED, -1, 0);

// MAP_HUGETLB: Use huge pages (2MB or 1GB)
void *huge = mmap(NULL, 2 * 1024 * 1024,
                  PROT_READ | PROT_WRITE,
                  MAP_PRIVATE | MAP_ANONYMOUS | MAP_HUGETLB,
                  -1, 0);
```

**Memory Protection:**
```c
#include <sys/mman.h>

void protection_example() {
    void *page = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                      MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    
    // Write some data
    strcpy(page, "Secret data");
    
    // Make read-only
    mprotect(page, 4096, PROT_READ);
    
    // This will crash with SIGSEGV!
    // strcpy(page, "Try to modify");  
    
    // Make executable (for JIT compilers)
    mprotect(page, 4096, PROT_READ | PROT_EXEC);
    
    // Make no access (guard page)
    mprotect(page, 4096, PROT_NONE);
    
    munmap(page, 4096);
}
```

**brk/sbrk (Heap Management):**
```c
#include <unistd.h>

void heap_example() {
    // Get current program break
    void *current_brk = sbrk(0);
    printf("Current break: %p\n", current_brk);
    
    // Increase heap by 1024 bytes
    void *new_brk = sbrk(1024);
    if (new_brk == (void*)-1) {
        perror("sbrk failed");
        return;
    }
    
    // Use the memory
    char *heap_mem = (char*)new_brk;
    strcpy(heap_mem, "Allocated on heap");
    
    // Shrink heap (decrease by 1024)
    sbrk(-1024);
}
```

**Page Locking (Prevent Swapping):**
```c
#include <sys/mman.h>

void lock_memory_example() {
    char sensitive_data[4096];
    
    // Lock page in RAM (won't be swapped to disk)
    if (mlock(sensitive_data, sizeof(sensitive_data)) == 0) {
        // Store passwords, encryption keys, etc.
        strcpy(sensitive_data, "password123");
        
        // Clear before unlocking
        memset(sensitive_data, 0, sizeof(sensitive_data));
        munlock(sensitive_data, sizeof(sensitive_data));
    }
    
    // Lock all current and future pages
    mlockall(MCL_CURRENT | MCL_FUTURE);
    
    // Unlock all
    munlockall();
}

### 10.2 System Calls Deep Dive

**What is a System Call?**
- Controlled entry point into kernel mode
- Transition from user space (Ring 3) to kernel space (Ring 0)
- Mechanism: Software interrupt (int 0x80 on x86, syscall instruction on x86-64)

**System Call Flow:**
```
User Program
    ↓ (call wrapper function)
C Library (glibc)
    ↓ (setup registers, invoke syscall)
Kernel System Call Handler
    ↓ (sys_call_table lookup)
Specific System Call (e.g., sys_read)
    ↓ (perform operation)
Return to User Space
```

**Common System Calls:**
```c
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>

// File Operations
int fd = open("file.txt", O_RDWR | O_CREAT | O_TRUNC, 0644);
if (fd < 0) {
    perror("open");
    return -1;
}

// Read (system call)
char buffer[1024];
ssize_t bytes_read = read(fd, buffer, sizeof(buffer));
if (bytes_read < 0) {
    perror("read");
}

// Write (system call)
const char *msg = "Hello, Kernel!\n";
ssize_t bytes_written = write(fd, msg, strlen(msg));
if (bytes_written < 0) {
    perror("write");
}

// Seek (system call)
off_t new_pos = lseek(fd, 0, SEEK_SET);  // Go to beginning
lseek(fd, 10, SEEK_CUR);  // Skip 10 bytes forward
off_t file_size = lseek(fd, 0, SEEK_END);  // Get file size

// Close (system call)
close(fd);
```

**File Metadata:**
```c
#include <sys/stat.h>
#include <time.h>

void examine_file(const char *path) {
    struct stat st;
    
    if (stat(path, &st) < 0) {
        perror("stat");
        return;
    }
    
    // File type
    if (S_ISREG(st.st_mode))  printf("Regular file\n");
    if (S_ISDIR(st.st_mode))  printf("Directory\n");
    if (S_ISLNK(st.st_mode))  printf("Symbolic link\n");
    if (S_ISFIFO(st.st_mode)) printf("FIFO/pipe\n");
    if (S_ISSOCK(st.st_mode)) printf("Socket\n");
    if (S_ISCHR(st.st_mode))  printf("Character device\n");
    if (S_ISBLK(st.st_mode))  printf("Block device\n");
    
    // Permissions
    printf("Permissions: ");
    printf((st.st_mode & S_IRUSR) ? "r" : "-");
    printf((st.st_mode & S_IWUSR) ? "w" : "-");
    printf((st.st_mode & S_IXUSR) ? "x" : "-");
    printf((st.st_mode & S_IRGRP) ? "r" : "-");
    printf((st.st_mode & S_IWGRP) ? "w" : "-");
    printf((st.st_mode & S_IXGRP) ? "x" : "-");
    printf((st.st_mode & S_IROTH) ? "r" : "-");
    printf((st.st_mode & S_IWOTH) ? "w" : "-");
    printf((st.st_mode & S_IXOTH) ? "x" : "-");
    printf("\n");
    
    // Size and links
    printf("Size: %ld bytes\n", st.st_size);
    printf("Links: %ld\n", st.st_nlink);
    printf("Inode: %ld\n", st.st_ino);
    printf("Device: %ld\n", st.st_dev);
    
    // Timestamps
    printf("Last access: %s", ctime(&st.st_atime));
    printf("Last modification: %s", ctime(&st.st_mtime));
    printf("Last status change: %s", ctime(&st.st_ctime));
    
    // Ownership
    printf("UID: %d, GID: %d\n", st.st_uid, st.st_gid);
}
```

**Advanced File Operations:**
```c
#include <unistd.h>
#include <fcntl.h>

// File locking (advisory)
void file_locking_example() {
    int fd = open("data.txt", O_RDWR);
    
    struct flock lock;
    lock.l_type = F_WRLCK;    // Write lock
    lock.l_whence = SEEK_SET;
    lock.l_start = 0;
    lock.l_len = 0;           // Lock entire file
    
    // Blocking lock
    if (fcntl(fd, F_SETLKW, &lock) < 0) {
        perror("fcntl lock");
        return;
    }
    
    // Critical section - file is locked
    write(fd, "protected data", 14);
    
    // Unlock
    lock.l_type = F_UNLCK;
    fcntl(fd, F_SETLK, &lock);
    
    close(fd);
}

// File descriptor duplication
void dup_example() {
    int fd = open("output.txt", O_WRONLY | O_CREAT, 0644);
    
    // Save stdout
    int saved_stdout = dup(STDOUT_FILENO);
    
    // Redirect stdout to file
    dup2(fd, STDOUT_FILENO);
    
    printf("This goes to file\n");  // Written to file
    
    // Restore stdout
    dup2(saved_stdout, STDOUT_FILENO);
    close(saved_stdout);
    
    printf("This goes to terminal\n");  // Displayed
    
    close(fd);
}

// Scatter-gather I/O (vectored I/O)
#include <sys/uio.h>

void vectored_io_example() {
    int fd = open("data.txt", O_WRONLY | O_CREAT, 0644);
    
    // Multiple buffers in one system call
    char buf1[] = "First part ";
    char buf2[] = "Second part ";
    char buf3[] = "Third part\n";
    
    struct iovec iov[3];
    iov[0].iov_base = buf1;
    iov[0].iov_len = strlen(buf1);
    iov[1].iov_base = buf2;
    iov[1].iov_len = strlen(buf2);
    iov[2].iov_base = buf3;
    iov[2].iov_len = strlen(buf3);
    
    // Single system call for all buffers
    ssize_t written = writev(fd, iov, 3);
    printf("Wrote %zd bytes\n", written);
    
    close(fd);
}

// Asynchronous I/O hint
void async_io_hint() {
    int fd = open("file.txt", O_RDWR);
    
    // Advise kernel about access pattern
    posix_fadvise(fd, 0, 0, POSIX_FADV_SEQUENTIAL);  // Sequential
    // POSIX_FADV_RANDOM      - Random access
    // POSIX_FADV_WILLNEED    - Will need soon (prefetch)
    // POSIX_FADV_DONTNEED    - Won't need (drop cache)
    
    close(fd);
}
```

**Direct System Call Invocation:**
```c
#include <sys/syscall.h>
#include <unistd.h>

// Bypass C library wrapper
void direct_syscall_example() {
    // Direct syscall (Linux x86-64)
    long result = syscall(SYS_write, STDOUT_FILENO, "Hello\n", 6);
    
    // Get process ID
    pid_t pid = syscall(SYS_getpid);
    
    // Get thread ID
    pid_t tid = syscall(SYS_gettid);
    
    printf("PID: %d, TID: %d\n", pid, tid);
}
```

### 10.3 Process Information & Management

**Process Identifiers:**
```c
#include <sys/types.h>
#include <unistd.h>

void process_ids() {
    // Process ID
    pid_t pid = getpid();
    printf("My PID: %d\n", pid);
    
    // Parent process ID
    pid_t ppid = getppid();
    printf("Parent PID: %d\n", ppid);
    
    // Process group ID
    pid_t pgid = getpgrp();
    printf("Process group: %d\n", pgid);
    
    // Session ID
    pid_t sid = getsid(0);
    printf("Session ID: %d\n", sid);
    
    // User IDs
    uid_t ruid = getuid();   // Real user ID
    uid_t euid = geteuid();  // Effective user ID
    printf("UID: %d, EUID: %d\n", ruid, euid);
    
    // Group IDs
    gid_t rgid = getgid();
    gid_t egid = getegid();
    printf("GID: %d, EGID: %d\n", rgid, egid);
}
```

**Resource Limits:**
```c
#include <sys/resource.h>

void show_limits() {
    struct rlimit limit;
    
    // Stack size
    getrlimit(RLIMIT_STACK, &limit);
    printf("Stack: soft=%ld, hard=%ld\n",
           limit.rlim_cur, limit.rlim_max);
    
    // Max open files
    getrlimit(RLIMIT_NOFILE, &limit);
    printf("Open files: soft=%ld, hard=%ld\n",
           limit.rlim_cur, limit.rlim_max);
    
    // Max processes
    getrlimit(RLIMIT_NPROC, &limit);
    printf("Max processes: %ld\n", limit.rlim_cur);
    
    // Virtual memory
    getrlimit(RLIMIT_AS, &limit);
    printf("Address space: %ld bytes\n", limit.rlim_cur);
    
    // Core dump size
    getrlimit(RLIMIT_CORE, &limit);
    printf("Core dump: %ld bytes\n", limit.rlim_cur);
    
    // CPU time
    getrlimit(RLIMIT_CPU, &limit);
    printf("CPU time: %ld seconds\n", limit.rlim_cur);
}

void set_limits() {
    struct rlimit limit;
    
    // Increase file descriptor limit
    limit.rlim_cur = 4096;  // Soft limit
    limit.rlim_max = 8192;  // Hard limit
    
    if (setrlimit(RLIMIT_NOFILE, &limit) == 0) {
        printf("Limit increased\n");
    }
}
```

**Resource Usage:**
```c
#include <sys/resource.h>
#include <sys/time.h>

void show_resource_usage() {
    struct rusage usage;
    
    // Get resource usage for this process
    getrusage(RUSAGE_SELF, &usage);
    
    // CPU time
    printf("User CPU time: %ld.%06ld sec\n",
           usage.ru_utime.tv_sec, usage.ru_utime.tv_usec);
    printf("System CPU time: %ld.%06ld sec\n",
           usage.ru_stime.tv_sec, usage.ru_stime.tv_usec);
    
    // Memory
    printf("Max RSS: %ld KB\n", usage.ru_maxrss);
    printf("Page faults (minor): %ld\n", usage.ru_minflt);
    printf("Page faults (major): %ld\n", usage.ru_majflt);
    
    // I/O
    printf("Block input ops: %ld\n", usage.ru_inblock);
    printf("Block output ops: %ld\n", usage.ru_oublock);
    
    // Context switches
    printf("Voluntary ctx switches: %ld\n", usage.ru_nvcsw);
    printf("Involuntary ctx switches: %ld\n", usage.ru_nivcsw);
}
```

**Environment Variables:**
```c
#include <stdlib.h>

void environment_example() {
    // Get variable
    char *home = getenv("HOME");
    char *path = getenv("PATH");
    
    printf("HOME: %s\n", home);
    printf("PATH: %s\n", path);
    
    // Set variable (0 = don't overwrite if exists)
    setenv("MY_VAR", "my_value", 1);
    
    // Modify variable
    setenv("MY_VAR", "new_value", 1);
    
    // Remove variable
    unsetenv("MY_VAR");
    
    // Alternative: putenv (variable becomes part of environment)
    putenv("CUSTOM=value");
}

// Access entire environment
extern char **environ;

void print_all_env() {
    for (char **env = environ; *env != NULL; env++) {
        printf("%s\n", *env);
    }
}
```

**Process Priority (Nice Values):**
```c
#include <sys/resource.h>

void priority_example() {
    // Get current priority (-20 to 19)
    // Lower value = higher priority
    int nice_val = getpriority(PRIO_PROCESS, 0);
    printf("Current nice value: %d\n", nice_val);
    
    // Increase niceness (lower priority)
    nice(10);  // Add 10 to nice value
    
    // Set specific priority (requires privileges for negative values)
    setpriority(PRIO_PROCESS, 0, 5);
    
    // Set priority for process group
    setpriority(PRIO_PGRP, getpgrp(), 10);
    
    // Set priority for user
    setpriority(PRIO_USER, getuid(), 15);
}
```

**Process Memory Information:**
```c
#include <stdio.h>

void read_proc_info() {
    // Read /proc/self/status
    FILE *fp = fopen("/proc/self/status", "r");
    char line[256];
    
    while (fgets(line, sizeof(line), fp)) {
        if (strncmp(line, "VmSize:", 7) == 0 ||
            strncmp(line, "VmRSS:", 6) == 0 ||
            strncmp(line, "VmData:", 7) == 0 ||
            strncmp(line, "VmStk:", 6) == 0) {
            printf("%s", line);
        }
    }
    fclose(fp);
    
    // Read /proc/self/maps (memory mappings)
    fp = fopen("/proc/self/maps", "r");
    printf("\nMemory mappings:\n");
    while (fgets(line, sizeof(line), fp)) {
        printf("%s", line);
    }
    fclose(fp);
}
```

### 10.4 Kernel Data Structures & Concepts

#### Process Control Block (task_struct)

**The task_struct** is the heart of process management in Linux. It contains everything the kernel needs to know about a process.

**Key Fields in task_struct (from include/linux/sched.h):**
```c
struct task_struct {
    // Process state
    volatile long state;    // TASK_RUNNING, TASK_INTERRUPTIBLE, etc.
    
    // Process identification
    pid_t pid;             // Process ID
    pid_t tgid;            // Thread group ID (main thread PID)
    
    // Process relationships
    struct task_struct *parent;     // Parent process
    struct list_head children;       // Child processes
    struct list_head sibling;        // Sibling processes
    
    // Scheduling information
    int prio;                       // Dynamic priority
    int static_prio;                // Static priority
    unsigned int policy;            // Scheduling policy
    unsigned int time_slice;        // Time quantum
    
    // Memory management
    struct mm_struct *mm;           // Memory descriptor
    struct mm_struct *active_mm;
    
    // File system information
    struct fs_struct *fs;           // Filesystem info
    struct files_struct *files;     // Open file descriptors
    
    // Signal handling
    struct signal_struct *signal;
    sigset_t blocked;               // Blocked signals
    sigset_t pending;               // Pending signals
    
    // CPU context
    struct thread_struct thread;    // CPU registers, stack pointer
    
    // Namespaces (containers)
    struct nsproxy *nsproxy;
    
    // Credentials
    const struct cred *cred;        // UIDs, GIDs, capabilities
    
    // Statistics
    u64 utime;                      // User mode time
    u64 stime;                      // Kernel mode time
    unsigned long nvcsw;            // Voluntary context switches
    unsigned long nivcsw;           // Involuntary context switches
};
```

**Process States:**
```c
// From include/linux/sched.h
#define TASK_RUNNING            0  // Running or ready to run
#define TASK_INTERRUPTIBLE      1  // Sleeping, can be woken by signals
#define TASK_UNINTERRUPTIBLE    2  // Sleeping, cannot be interrupted
#define __TASK_STOPPED          4  // Stopped (SIGSTOP)
#define __TASK_TRACED           8  // Being traced (ptrace)
#define EXIT_ZOMBIE            16  // Terminated, waiting for parent
#define EXIT_DEAD              32  // Final state

/*
Process State Transitions:

  ┌─────────────┐
  │  TASK_NEW   │  (Process creation)
  └──────┬──────┘
         │
         ↓
  ┌─────────────┐     Context Switch      ┌──────────────┐
  │TASK_RUNNING │ ←───────────────────────→│ TASK_READY   │
  └──────┬──────┘                          └──────────────┘
         │                                         ↑
         │ Wait for I/O                           │
         ↓                                         │
  ┌──────────────────┐    I/O Complete           │
  │ TASK_INTERRUPTIBLE│────────────────────────────┘
  └──────────────────┘         or
                        ┌──────────────────┐
                        │TASK_UNINTERRUPTIBLE│
                        └──────────────────┘
         │
         │ Process termination
         ↓
  ┌─────────────┐
  │EXIT_ZOMBIE  │  (Waiting for parent to reap)
  └──────┬──────┘
         │ wait()/waitpid()
         ↓
  ┌─────────────┐
  │ EXIT_DEAD   │
  └─────────────┘
*/
```

#### Memory Descriptor (mm_struct)

```c
struct mm_struct {
    // Virtual memory areas (segments)
    struct vm_area_struct *mmap;    // List of VMAs
    struct rb_root mm_rb;           // Red-black tree of VMAs
    
    unsigned long start_code;       // Text segment start
    unsigned long end_code;         // Text segment end
    unsigned long start_data;       // Data segment start
    unsigned long end_data;         // Data segment end
    unsigned long start_brk;        // Heap start
    unsigned long brk;              // Current heap end
    unsigned long start_stack;      // Stack start
    
    // Page table
    pgd_t *pgd;                     // Page global directory
    
    atomic_t mm_users;              // Number of users
    atomic_t mm_count;              // Reference count
    
    unsigned long total_vm;         // Total pages mapped
    unsigned long locked_vm;        // Pages locked in memory
    unsigned long pinned_vm;        // Pages pinned
};
```

**Virtual Memory Area (VMA):**
```c
struct vm_area_struct {
    unsigned long vm_start;         // Start address
    unsigned long vm_end;           // End address
    struct vm_area_struct *vm_next; // Next VMA
    
    pgprot_t vm_page_prot;         // Access permissions
    unsigned long vm_flags;         // Flags (VM_READ, VM_WRITE, etc.)
    
    struct file *vm_file;           // Mapped file (or NULL)
    unsigned long vm_pgoff;         // Offset in file
    
    const struct vm_operations_struct *vm_ops;  // Operations
};
```

#### File Descriptor Table

```c
struct files_struct {
    atomic_t count;                 // Reference count
    struct fdtable *fdt;            // Pointer to fd table
    struct fdtable fdtab;           // Base fd table
    spinlock_t file_lock;           // Lock
    int next_fd;                    // Next available fd
};

struct fdtable {
    unsigned int max_fds;           // Max number of fds
    struct file **fd;               // Array of file pointers
    unsigned long *close_on_exec;   // Close-on-exec bitmap
    unsigned long *open_fds;        // Open fds bitmap
};

/*
File Descriptor Layout:

┌──────────────────────────────────┐
│  Process (task_struct)           │
└────────────┬─────────────────────┘
             │
             │ files
             ↓
┌──────────────────────────────────┐
│  files_struct                    │
│  ┌────────────────────────────┐ │
│  │ fd[0] → stdin  (file*)     │ │
│  │ fd[1] → stdout (file*)     │ │
│  │ fd[2] → stderr (file*)     │ │
│  │ fd[3] → opened_file (file*)│ │
│  │ ...                        │ │
│  └────────────────────────────┘ │
└──────────────────────────────────┘
             │
             ↓
┌──────────────────────────────────┐
│  struct file                     │
│  - f_mode (read/write)           │
│  - f_pos (file position)         │
│  - f_op (file operations)        │
│  - f_path (dentry, vfsmount)    │
└──────────────────────────────────┘
*/
```

**Standard File Descriptors:**
```c
#define STDIN_FILENO   0    // Standard input
#define STDOUT_FILENO  1    // Standard output
#define STDERR_FILENO  2    // Standard error

// Usage example
write(STDOUT_FILENO, "Hello\n", 6);  // Write to stdout
read(STDIN_FILENO, buffer, 100);     // Read from stdin
```

#### Scheduler Run Queue

```c
// Simplified representation
struct rq {
    unsigned int nr_running;        // Number of runnable tasks
    u64 clock;                      // Scheduler clock
    
    struct cfs_rq cfs;              // CFS (Completely Fair Scheduler) queue
    struct rt_rq rt;                // Real-time queue
    
    struct task_struct *curr;       // Currently running task
    struct task_struct *idle;       // Idle task
    
    // Per-CPU data
    int cpu;
    
    // Load balancing
    unsigned long cpu_load[5];
};
```

#### Signal Handling Structures

```c
struct signal_struct {
    atomic_t sigcnt;                // Signal count
    atomic_t live;                  // Live processes
    
    struct list_head thread_head;   // List of threads
    
    // Signal statistics
    unsigned long nvcsw, nivcsw;    // Context switches
    unsigned long utime, stime;     // CPU time
};

struct sighand_struct {
    atomic_t count;
    struct k_sigaction action[_NSIG];  // Signal handlers
    spinlock_t siglock;
};
```

#### Kernel Linked Lists

```c
// From include/linux/list.h
struct list_head {
    struct list_head *next, *prev;
};

// Example usage
struct my_data {
    int value;
    struct list_head list;  // Embedded list node
};

// Kernel list operations
LIST_HEAD(my_list);  // Define and initialize list

struct my_data *item = kmalloc(sizeof(*item), GFP_KERNEL);
item->value = 42;
list_add(&item->list, &my_list);  // Add to list

// Iterate through list
struct my_data *pos;
list_for_each_entry(pos, &my_list, list) {
    printk("Value: %d\n", pos->value);
}
```

---

## 11. Open Source Analysis

### 11.1 Linux Kernel Code Study

**Kernel Source Organization:**
```
linux/
├── arch/           # Architecture-specific code
│   ├── x86/         # Intel/AMD processors
│   ├── arm/         # ARM processors
│   └── arm64/       # 64-bit ARM
├── block/          # Block device I/O layer
├── crypto/         # Cryptographic API
├── drivers/        # Device drivers
│   ├── char/        # Character devices
│   ├── block/       # Block devices
│   ├── net/         # Network drivers
│   └── usb/         # USB drivers
├── fs/             # File systems
│   ├── ext4/        # ext4 filesystem
│   ├── proc/        # /proc filesystem
│   └── sysfs/       # /sys filesystem
├── include/        # Header files
│   ├── linux/       # Core kernel headers
│   └── uapi/        # User-space API headers
├── init/           # Kernel initialization
├── ipc/            # Inter-process communication
│   ├── msg.c        # Message queues
│   ├── sem.c        # Semaphores
│   └── shm.c        # Shared memory
├── kernel/         # Core kernel code
│   ├── sched/       # Process scheduler
│   ├── fork.c       # Process creation
│   ├── signal.c     # Signal handling
│   └── time/        # Time management
├── lib/            # Library routines
├── mm/             # Memory management
│   ├── mmap.c       # Memory mapping
│   ├── slab.c       # Slab allocator
│   └── vmalloc.c    # Virtual memory allocation
├── net/            # Networking stack
│   ├── ipv4/        # IPv4 protocol
│   ├── ipv6/        # IPv6 protocol
│   └── socket.c     # Socket interface
└── security/       # Security modules (SELinux, AppArmor)
```

#### Kernel String Functions (lib/string.c)

```c
// Simple strlen implementation from Linux kernel
size_t strlen(const char *s) {
    const char *sc;
    
    for (sc = s; *sc != '\0'; ++sc)
        /* nothing */;
    return sc - s;
}

// Optimized memcpy (simplified)
void *memcpy(void *dest, const void *src, size_t count) {
    char *tmp = dest;
    const char *s = src;
    
    while (count--)
        *tmp++ = *s++;
    return dest;
}

// Kernel-specific string copy with size limit
size_t strlcpy(char *dest, const char *src, size_t size) {
    size_t ret = strlen(src);
    
    if (size) {
        size_t len = (ret >= size) ? size - 1 : ret;
        memcpy(dest, src, len);
        dest[len] = '\0';
    }
    return ret;
}
```

#### Process Creation (kernel/fork.c)

**The fork() system call flow:**
```c
/*
User Space:               Kernel Space:

fork()                    sys_fork()
  │                          │
  │                          ↓
  │                       _do_fork()
  │                          │
  │                          ↓
  │                       copy_process()
  │                          │
  │                          ├────────> dup_task_struct()
  │                          │         (Allocate new task_struct)
  │                          │
  │                          ├────────> copy_mm()
  │                          │         (Copy/share memory descriptor)
  │                          │
  │                          ├────────> copy_files()
  │                          │         (Copy file descriptor table)
  │                          │
  │                          ├────────> copy_signal()
  │                          │         (Copy signal handlers)
  │                          │
  │                          ├────────> sched_fork()
  │                          │         (Setup scheduler)
  │                          │
  │                          └────────> wake_up_new_task()
  │                                    (Add to run queue)
  │
  ←────── Return child PID to parent
           Return 0 to child
*/

// Simplified fork implementation concept
long do_fork(unsigned long clone_flags,
             unsigned long stack_start,
             struct pt_regs *regs,
             unsigned long stack_size) {
    struct task_struct *p;
    int trace = 0;
    long nr;
    
    // Allocate new task_struct
    p = copy_process(clone_flags, stack_start, regs, stack_size);
    
    if (!IS_ERR(p)) {
        // Get new PID
        nr = task_pid_vnr(p);
        
        // Wake up new process
        wake_up_new_task(p);
    }
    
    return nr;
}
```

#### Scheduler (kernel/sched/core.c)

**Completely Fair Scheduler (CFS):**
```c
/*
CFS Scheduling Algorithm:

1. Virtual Runtime (vruntime):
   - Each task accumulates vruntime as it runs
   - vruntime = actual_runtime * (NICE_0_LOAD / task_weight)
   - Lower nice value → higher weight → slower vruntime growth

2. Red-Black Tree:
   - Tasks stored in rb-tree ordered by vruntime
   - Leftmost node = task with smallest vruntime (runs next)
   
3. Time Slice:
   - Calculated based on: target_latency / nr_running
   - Minimum granularity enforced

Visualization:

        RB-Tree (ordered by vruntime)
                ┌──────────┐
                │ vruntime:  │
                │   1000     │
                └───┬────┬──┘
                   │      │
           ┌───────┴─┐    └───────┬────┐
           │   800   │            │ 1500 │
           └───┬─┬───┘            └──────┘
               │ │
          ┌────┘ └────┐
          │ 700 │   │ 850 │  ← Pick leftmost (700)
          └─────┘   └─────┘
*/

// Simplified CFS pick next task
struct task_struct *pick_next_task_fair(struct rq *rq) {
    struct cfs_rq *cfs_rq = &rq->cfs;
    struct sched_entity *se;
    
    // Get leftmost (minimum vruntime) entity
    se = __pick_first_entity(cfs_rq);
    if (!se)
        return NULL;
    
    // Return task
    return task_of(se);
}
```

#### Memory Allocator (mm/slab.c)

**SLAB/SLUB Allocator:**
```c
/*
SLAB Allocator Purpose:
- Efficient allocation of small, frequently-used objects
- Reduces fragmentation
- Cache-hot objects (recently freed objects reused)

Structure:

┌────────────────────────────────────────┐
│ kmem_cache (e.g., task_struct cache)      │
│                                          │
│  ┌────────────┐   ┌────────────┐       │
│  │   Slab 1    │   │   Slab 2    │  ...  │
│  │ (full)     │   │ (partial)  │       │
│  └────────────┘   └────────────┘       │
│       │                  │                │
│       └──────────────────┘                │
│                    │                       │
│     ┌──────────────┴───────────────┐   │
│     │ Objects in Slab:              │   │
│     │  [obj1][obj2][obj3][free]...   │   │
│     └──────────────────────────────┘   │
└────────────────────────────────────────┘
*/

// Create a cache for specific object type
struct kmem_cache *task_cache;

void init_task_cache(void) {
    task_cache = kmem_cache_create("task_struct_cache",
                                   sizeof(struct task_struct),
                                   0,  // alignment
                                   SLAB_HWCACHE_ALIGN,
                                   NULL);  // constructor
}

// Allocate object from cache
struct task_struct *alloc_task(void) {
    return kmem_cache_alloc(task_cache, GFP_KERNEL);
}

// Free object back to cache
void free_task(struct task_struct *task) {
    kmem_cache_free(task_cache, task);
}
```

#### System Call Implementation

**Example: sys_getpid (kernel/sys.c):**
```c
// System call to get process ID
SYSCALL_DEFINE0(getpid) {
    // task_tgid_vnr() gets thread group ID
    // (main thread's PID)
    return task_tgid_vnr(current);
}

// Expands to:
asmlinkage long sys_getpid(void) {
    return task_tgid_vnr(current);
}

/*
'current' is a macro that returns pointer to current task_struct:

#define current get_current()

static inline struct task_struct *get_current(void) {
    return this_cpu_read_stable(current_task);
}

On x86-64, current task stored in per-CPU variable
*/
```

**Adding Your Own System Call (Educational):**
```c
// 1. Add to kernel/sys.c
SYSCALL_DEFINE1(hello, char __user *, name) {
    char kernel_name[256];
    
    // Copy from user space to kernel space
    if (copy_from_user(kernel_name, name, 256)) {
        return -EFAULT;
    }
    
    printk(KERN_INFO "Hello, %s!\n", kernel_name);
    return 0;
}

// 2. Add to system call table (arch/x86/entry/syscalls/syscall_64.tbl)
// 548    common  hello     sys_hello

// 3. Add prototype to include/linux/syscalls.h
asmlinkage long sys_hello(char __user *name);

// 4. User space usage
#include <unistd.h>
#include <sys/syscall.h>

int main() {
    syscall(548, "World");  // Call your syscall
    return 0;
}
```

### 11.2 Redis Source Code Analysis

**Redis Architecture Overview:**
```
Redis is a single-threaded, in-memory data structure store.

Core Components:
1. Event Loop (ae.c) - Handles I/O multiplexing
2. Data Structures (t_*.c) - String, List, Hash, Set, Sorted Set
3. Persistence (rdb.c, aof.c) - RDB snapshots, AOF logs
4. Networking (networking.c) - Client connections
5. Commands (commands/*.c) - Command implementations
```

#### Simple Dynamic String (SDS)

**Why SDS instead of C strings?**
- O(1) length retrieval
- Binary safe (can store \0)
- Prevents buffer overflows
- Efficient memory pre-allocation

```c
// From sds.h
typedef char *sds;

struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags;  // 3 lsb = type, 5 msb = length
    char buf[];
};

struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len;         // Used length
    uint8_t alloc;       // Allocated space (excluding header and null)
    unsigned char flags; // Type identifier
    char buf[];          // Actual string
};

struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len;
    uint16_t alloc;
    unsigned char flags;
    char buf[];
};

// More types: sdshdr32, sdshdr64

/*
Memory Layout:

┌──────────┬──────────┬───────┬──────────────────────────┬───┐
│   len    │  alloc   │ flags │     buf (string)       │\0 │
│  (used)  │ (total)  │ (type)│                        │   │
└──────────┴──────────┴───────┴──────────────────────────┴───┘
                            ↑
                           sds pointer points here
*/

// Create SDS
sds sdsnew(const char *init) {
    size_t initlen = (init == NULL) ? 0 : strlen(init);
    return sdsnewlen(init, initlen);
}

sds sdsnewlen(const void *init, size_t initlen) {
    void *sh;
    sds s;
    char type = sdsReqType(initlen);
    int hdrlen = sdsHdrSize(type);
    unsigned char *fp;  // Flags pointer
    
    // Allocate header + string + null terminator
    sh = malloc(hdrlen + initlen + 1);
    if (sh == NULL) return NULL;
    
    // Initialize header based on type
    fp = ((unsigned char*)sh) + hdrlen - 1;
    switch(type) {
        case SDS_TYPE_8: {
            SDS_HDR_VAR(8, s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        // ... other types
    }
    
    s = (char*)sh + hdrlen;
    if (initlen && init)
        memcpy(s, init, initlen);
    s[initlen] = '\0';
    return s;
}

// Get length (O(1))
size_t sdslen(const sds s) {
    unsigned char flags = s[-1];  // Access header
    switch(flags & SDS_TYPE_MASK) {
        case SDS_TYPE_8:
            return SDS_HDR(8, s)->len;
        case SDS_TYPE_16:
            return SDS_HDR(16, s)->len;
        // ... other types
    }
    return 0;
}

// Concatenate
sds sdscat(sds s, const char *t) {
    return sdscatlen(s, t, strlen(t));
}

sds sdscatlen(sds s, const void *t, size_t len) {
    size_t curlen = sdslen(s);
    
    // Expand if needed
    s = sdsMakeRoomFor(s, len);
    if (s == NULL) return NULL;
    
    // Append
    memcpy(s + curlen, t, len);
    sdssetlen(s, curlen + len);
    s[curlen + len] = '\0';
    return s;
}
```

#### Redis Object System

```c
// From server.h
typedef struct redisObject {
    unsigned type:4;      // Type: STRING, LIST, SET, ZSET, HASH
    unsigned encoding:4;  // Encoding: RAW, INT, HT, ZIPLIST, etc.
    unsigned lru:24;      // LRU time or LFU data
    int refcount;         // Reference counting
    void *ptr;            // Pointer to actual data
} robj;

// Object types
#define OBJ_STRING 0
#define OBJ_LIST 1
#define OBJ_SET 2
#define OBJ_ZSET 3
#define OBJ_HASH 4

// Encodings
#define OBJ_ENCODING_RAW 0        // Raw SDS string
#define OBJ_ENCODING_INT 1        // Encoded as integer
#define OBJ_ENCODING_HT 2         // Hash table
#define OBJ_ENCODING_ZIPLIST 5    // Compressed list
#define OBJ_ENCODING_INTSET 6     // Integer set
#define OBJ_ENCODING_SKIPLIST 7   // Skip list + hash table

// Create string object
robj *createStringObject(const char *ptr, size_t len) {
    if (len <= OBJ_ENCODING_EMBSTR_SIZE_LIMIT)
        return createEmbeddedStringObject(ptr, len);
    else
        return createRawStringObject(ptr, len);
}

robj *createRawStringObject(const char *ptr, size_t len) {
    return createObject(OBJ_STRING, sdsnewlen(ptr, len));
}

// Try to encode as integer
robj *tryObjectEncoding(robj *o) {
    long value;
    sds s = o->ptr;
    size_t len;
    
    if (o->encoding != OBJ_ENCODING_RAW)
        return o;
    
    len = sdslen(s);
    if (len <= 20 && string2l(s, len, &value)) {
        // Can encode as integer
        if ((value >= 0 && value < OBJ_SHARED_INTEGERS) ||
            (value >= LONG_MIN && value <= LONG_MAX)) {
            decrRefCount(o);
            return createStringObjectFromLongLong(value);
        }
    }
    
    return o;
}
```

#### Skip List (Sorted Set Implementation)

```c
// From server.h
#define ZSKIPLIST_MAXLEVEL 32
#define ZSKIPLIST_P 0.25

typedef struct zskiplistNode {
    sds ele;                      // Member element
    double score;                  // Score for sorting
    struct zskiplistNode *backward;  // Backward pointer
    struct zskiplistLevel {
        struct zskiplistNode *forward;  // Forward pointer
        unsigned long span;              // Span to next node
    } level[];                     // Flexible array
} zskiplistNode;

typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;          // Number of nodes
    int level;                     // Max level
} zskiplist;

/*
Skip List Structure (4 levels):

Level 3:  head -------> [30] --------------------------------> NULL
Level 2:  head -> [10] -> [30] -------> [60] --------------> NULL
Level 1:  head -> [10] -> [30] -> [40] -> [60] -> [80] ----> NULL
Level 0:  head -> [10] -> [30] -> [40] -> [60] -> [80] -> [90] -> NULL
*/

// Create skip list
zskiplist *zslCreate(void) {
    int j;
    zskiplist *zsl;
    
    zsl = zmalloc(sizeof(*zsl));
    zsl->level = 1;
    zsl->length = 0;
    
    // Create header node with max level
    zsl->header = zslCreateNode(ZSKIPLIST_MAXLEVEL, 0, NULL);
    for (j = 0; j < ZSKIPLIST_MAXLEVEL; j++) {
        zsl->header->level[j].forward = NULL;
        zsl->header->level[j].span = 0;
    }
    zsl->header->backward = NULL;
    zsl->tail = NULL;
    return zsl;
}

// Random level for new node
int zslRandomLevel(void) {
    int level = 1;
    while ((random() & 0xFFFF) < (ZSKIPLIST_P * 0xFFFF))
        level += 1;
    return (level < ZSKIPLIST_MAXLEVEL) ? level : ZSKIPLIST_MAXLEVEL;
}

// Insert node
zskiplistNode *zslInsert(zskiplist *zsl, double score, sds ele) {
    zskiplistNode *update[ZSKIPLIST_MAXLEVEL], *x;
    unsigned int rank[ZSKIPLIST_MAXLEVEL];
    int i, level;
    
    x = zsl->header;
    // Find insertion point at each level
    for (i = zsl->level - 1; i >= 0; i--) {
        rank[i] = i == (zsl->level - 1) ? 0 : rank[i + 1];
        while (x->level[i].forward &&
               (x->level[i].forward->score < score ||
                (x->level[i].forward->score == score &&
                 sdscmp(x->level[i].forward->ele, ele) < 0))) {
            rank[i] += x->level[i].span;
            x = x->level[i].forward;
        }
        update[i] = x;
    }
    
    // Random level for new node
    level = zslRandomLevel();
    if (level > zsl->level) {
        for (i = zsl->level; i < level; i++) {
            rank[i] = 0;
            update[i] = zsl->header;
            update[i]->level[i].span = zsl->length;
        }
        zsl->level = level;
    }
    
    // Create and insert new node
    x = zslCreateNode(level, score, ele);
    for (i = 0; i < level; i++) {
        x->level[i].forward = update[i]->level[i].forward;
        update[i]->level[i].forward = x;
        
        x->level[i].span = update[i]->level[i].span - (rank[0] - rank[i]);
        update[i]->level[i].span = (rank[0] - rank[i]) + 1;
    }
    
    // Update backward pointer
    x->backward = (update[0] == zsl->header) ? NULL : update[0];
    if (x->level[0].forward)
        x->level[0].forward->backward = x;
    else
        zsl->tail = x;
    
    zsl->length++;
    return x;
}
```

#### Event Loop (ae.c)

```c
// Redis uses a simple event-driven model

typedef struct aeEventLoop {
    int maxfd;                  // Highest file descriptor
    int setsize;                // Max number of tracked fds
    long long timeEventNextId;
    time_t lastTime;
    aeFileEvent *events;        // File events
    aeFiredEvent *fired;        // Fired events
    aeTimeEvent *timeEventHead; // Time events (linked list)
    int stop;
    void *apidata;              // Used by specific API (epoll, kqueue)
    aeBeforeSleepProc *beforesleep;
    aeBeforeSleepProc *aftersleep;
} aeEventLoop;

// Main event loop
void aeMain(aeEventLoop *eventLoop) {
    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        // Execute before-sleep callback
        if (eventLoop->beforesleep != NULL)
            eventLoop->beforesleep(eventLoop);
        
        // Process events
        aeProcessEvents(eventLoop, AE_ALL_EVENTS | AE_CALL_AFTER_SLEEP);
    }
}

int aeProcessEvents(aeEventLoop *eventLoop, int flags) {
    int processed = 0, numevents;
    
    // Get time events
    if (flags & AE_TIME_EVENTS) {
        // Process time events
    }
    
    // Get file events from OS (epoll_wait, kqueue, select)
    numevents = aeApiPoll(eventLoop, tvp);
    
    // Execute after-sleep callback
    if (eventLoop->aftersleep != NULL && flags & AE_CALL_AFTER_SLEEP)
        eventLoop->aftersleep(eventLoop);
    
    // Process file events
    for (j = 0; j < numevents; j++) {
        aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
        int mask = eventLoop->fired[j].mask;
        int fd = eventLoop->fired[j].fd;
        
        // Readable event
        if (fe->mask & mask & AE_READABLE) {
            fe->rfileProc(eventLoop, fd, fe->clientData, mask);
            fired++;
        }
        
        // Writable event
        if (fe->mask & mask & AE_WRITABLE) {
            if (!fired || fe->wfileProc != fe->rfileProc) {
                fe->wfileProc(eventLoop, fd, fe->clientData, mask);
                fired++;
            }
        }
        
        processed++;
    }
    
    return processed;
}
```

#### Command Execution Flow

```c
/*
Client sends: SET key value

1. readQueryFromClient() - Read from socket
   ↓
2. processInputBuffer() - Parse protocol
   ↓
3. processCommand() - Lookup command in command table
   ↓
4. call() - Execute command function
   ↓
5. setCommand() - Actual SET implementation
   ↓
6. addReply() - Prepare response
   ↓
7. writeToClient() - Send response to socket
*/

// Command table
struct redisCommand redisCommandTable[] = {
    {"set", setCommand, -3, "wm", 0, NULL, 1, 1, 1, 0, 0},
    {"get", getCommand, 2, "rF", 0, NULL, 1, 1, 1, 0, 0},
    {"del", delCommand, -2, "w", 0, NULL, 1, -1, 1, 0, 0},
    // ... more commands
};

// SET command implementation
void setCommand(client *c) {
    int j;
    robj *expire = NULL;
    int unit = UNIT_SECONDS;
    int flags = OBJ_SET_NO_FLAGS;
    
    // Parse options (EX, PX, NX, XX, etc.)
    for (j = 3; j < c->argc; j++) {
        // ... parse options
    }
    
    // Set the key-value
    c->argv[2] = tryObjectEncoding(c->argv[2]);
    setGenericCommand(c, flags, c->argv[1], c->argv[2], expire, unit, NULL, NULL);
}

void setGenericCommand(client *c, int flags, robj *key, robj *val,
                      robj *expire, int unit, robj *ok_reply, robj *abort_reply) {
    long long milliseconds = 0;
    
    // Handle expiration
    if (expire) {
        if (getLongLongFromObjectOrReply(c, expire, &milliseconds, NULL) != C_OK)
            return;
        if (milliseconds <= 0) {
            addReplyErrorFormat(c, "invalid expire time");
            return;
        }
        if (unit == UNIT_SECONDS) milliseconds *= 1000;
    }
    
    // Check NX|XX conditions
    if ((flags & OBJ_SET_NX && lookupKeyWrite(c->db, key) != NULL) ||
        (flags & OBJ_SET_XX && lookupKeyWrite(c->db, key) == NULL)) {
        addReply(c, abort_reply ? abort_reply : shared.null[c->resp]);
        return;
    }
    
    // Set key in database
    setKey(c->db, key, val);
    server.dirty++;
    
    // Set expiration
    if (expire) setExpire(c, c->db, key, mstime() + milliseconds);
    
    notifyKeyspaceEvent(NOTIFY_STRING, "set", key, c->db->id);
    
    // Send reply
    addReply(c, ok_reply ? ok_reply : shared.ok);
}
```

### 11.3 SQLite Database Engine

**Virtual Machine approach:**
- SQL compiled to bytecode
- VDBE (Virtual Database Engine) executes bytecode
- B-tree for storage
- Single file database

**Key concepts to study:**
```c
// Parse SQL → Generate bytecode → Execute
// B-tree operations
// Transaction management (ACID)
// Page cache
```

### 11.4 Git Internals

**Objects:**
```
- Blob: File contents
- Tree: Directory structure
- Commit: Snapshot with metadata
- Tag: Named reference to commit
```

**Storage:**
```c
// Objects stored as: .git/objects/ab/cdef123...
// SHA-1 hash of content
// Compressed with zlib
```

### 11.5 Nginx Architecture

**Key concepts:**
```
- Event-driven, asynchronous, non-blocking
- Master/worker process model
- Connection handling without threads
- State machines for request processing
```

---

## 12. Advanced Topics

### 12.1 Function Pointers & Callbacks

```c
// Calculator with function pointers
typedef int (*Operation)(int, int);

int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }
int div(int a, int b) { return a / b; }

int calculate(int a, int b, Operation op) {
    return op(a, b);
}

int main() {
    printf("%d\n", calculate(10, 5, add));  // 15
    printf("%d\n", calculate(10, 5, mul));  // 50
    
    // Array of function pointers
    Operation ops[] = {add, sub, mul, div};
    printf("%d\n", ops[2](6, 7));  // 42
    
    return 0;
}
```

### 12.2 Variadic Functions

```c
#include <stdarg.h>

int sum(int count, ...) {
    va_list args;
    va_start(args, count);
    
    int total = 0;
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);
    }
    
    va_end(args);
    return total;
}

void log_message(const char *fmt, ...) {
    va_list args;
    va_start(args, fmt);
    vprintf(fmt, args);
    va_end(args);
}

int main() {
    printf("%d\n", sum(3, 10, 20, 30));  // 60
    log_message("Error: %s at line %d\n", "null pointer", 42);
    return 0;
}
```

### 12.3 Inline Assembly

```c
// GCC inline assembly
int add_asm(int a, int b) {
    int result;
    __asm__ ("addl %%ebx, %%eax;"
             : "=a" (result)
             : "a" (a), "b" (b));
    return result;
}

// Atomics with assembly
static inline int atomic_increment(int *ptr) {
    int result = 1;
    __asm__ __volatile__(
        "lock; xaddl %0, %1"
        : "+r" (result), "+m" (*ptr)
        :
        : "memory"
    );
    return result + 1;
}
```

### 12.4 Compiler Optimizations

```c
// Optimization flags
// -O0: No optimization (default)
// -O1: Basic optimization
// -O2: Recommended optimization
// -O3: Aggressive optimization
// -Os: Optimize for size

// Loop unrolling
// Compiler may transform:
for (int i = 0; i < 4; i++) {
    arr[i] *= 2;
}
// Into:
arr[0] *= 2;
arr[1] *= 2;
arr[2] *= 2;
arr[3] *= 2;

// Inline functions
static inline int max(int a, int b) {
    return a > b ? a : b;
}

// Likely/unlikely (GCC)
if (__builtin_expect(error_condition, 0)) {
    // Unlikely path
}
```

### 12.5 Bit Fields

```c
struct Flags {
    unsigned int is_ready : 1;
    unsigned int has_error : 1;
    unsigned int mode : 3;
    unsigned int priority : 4;
    unsigned int : 0;  // Align to next boundary
};

struct Flags f = {0};
f.is_ready = 1;
f.mode = 5;
f.priority = 10;
```

### 12.6 Memory Alignment

```c
#include <stddef.h>

// Alignment requirements
struct Aligned {
    char c;       // 1 byte + 7 padding
    double d;     // 8 bytes (aligned to 8)
    int i;        // 4 bytes + 4 padding
};  // Total: 24 bytes

struct Compact {
    double d;     // 8 bytes
    int i;        // 4 bytes
    char c;       // 1 byte + 3 padding
};  // Total: 16 bytes

// Check alignment
printf("Alignment of int: %zu\n", _Alignof(int));
printf("Alignment of double: %zu\n", _Alignof(double));

// Force alignment (C11)
_Alignas(16) char buffer[64];
```

### 12.7 Memory Pools

```c
#define POOL_SIZE 1024
#define BLOCK_SIZE 32

typedef struct MemoryPool {
    char memory[POOL_SIZE];
    char *free_list;
    size_t block_size;
} MemoryPool;

void pool_init(MemoryPool *pool) {
    pool->block_size = BLOCK_SIZE;
    pool->free_list = pool->memory;
    
    // Link free blocks
    for (int i = 0; i < POOL_SIZE - BLOCK_SIZE; i += BLOCK_SIZE) {
        *(char**)(pool->memory + i) = pool->memory + i + BLOCK_SIZE;
    }
    *(char**)(pool->memory + POOL_SIZE - BLOCK_SIZE) = NULL;
}

void* pool_alloc(MemoryPool *pool) {
    if (pool->free_list == NULL) return NULL;
    
    void *ptr = pool->free_list;
    pool->free_list = *(char**)pool->free_list;
    return ptr;
}

void pool_free(MemoryPool *pool, void *ptr) {
    *(char**)ptr = pool->free_list;
    pool->free_list = ptr;
}
```

### 12.8 Lock-Free Data Structures

```c
#include <stdatomic.h>

// Lock-free queue (simplified)
typedef struct Node {
    int data;
    struct Node *next;
} Node;

typedef struct {
    atomic_uintptr_t head;
    atomic_uintptr_t tail;
} LockFreeQueue;

void queue_push(LockFreeQueue *q, int value) {
    Node *node = malloc(sizeof(Node));
    node->data = value;
    node->next = NULL;
    
    Node *tail;
    do {
        tail = (Node*)atomic_load(&q->tail);
    } while (!atomic_compare_exchange_weak(&tail->next, &(Node*){NULL}, node));
    
    atomic_compare_exchange_weak(&q->tail, &tail, node);
}
```

---

## 13. Best Practices & Optimization

### 13.1 Code Style

```c
// Use meaningful names
int calculate_average(int *array, size_t count);  // Good
int calc(int *a, size_t c);  // Bad

// Const correctness
void process_data(const int *input, int *output, size_t size);

// Error handling
int open_file(const char *path, FILE **fp) {
    *fp = fopen(path, "r");
    if (*fp == NULL) {
        return -1;  // Error code
    }
    return 0;  // Success
}

// Defensive programming
if (ptr != NULL && size > 0) {
    // Safe to proceed
}
```

### 13.2 Common Pitfalls

```c
// 1. Uninitialized variables
int x;  // Contains garbage
int y = 0;  // Properly initialized

// 2. Array out of bounds
int arr[10];
arr[10] = 5;  // BAD! Index 10 doesn't exist

// 3. Null pointer dereference
int *ptr = NULL;
*ptr = 10;  // CRASH!

// 4. Memory leaks
int *p = malloc(sizeof(int));
// ... forgot to free(p);

// 5. Dangling pointers
int *p = malloc(sizeof(int));
free(p);
*p = 10;  // BAD! Use after free

// 6. String buffer overflow
char buf[10];
strcpy(buf, "This is a very long string");  // OVERFLOW!

// 7. Forgetting null terminator
char str[5];
strncpy(str, "Hello", 5);  // No null terminator!
str[4] = '\0';  // Add it manually

// 8. Integer overflow
int x = INT_MAX;
x++;  // Undefined behavior

// 9. Comparing floats with ==
float a = 0.1 + 0.2;
if (a == 0.3) {  // May be false due to precision!
    // Use: fabs(a - 0.3) < EPSILON
}

// 10. Modifying string literals
char *str = "Hello";
str[0] = 'h';  // CRASH! String literal is read-only
```

### 13.3 Performance Optimization

```c
// 1. Use const for read-only data
const int lookup_table[] = {1, 2, 3, 4, 5};

// 2. Minimize function calls in loops
// Bad:
for (int i = 0; i < strlen(str); i++) { }
// Good:
size_t len = strlen(str);
for (size_t i = 0; i < len; i++) { }

// 3. Cache-friendly code
// Bad: Column-major access
for (int j = 0; j < N; j++)
    for (int i = 0; i < N; i++)
        sum += matrix[i][j];

// Good: Row-major access
for (int i = 0; i < N; i++)
    for (int j = 0; j < N; j++)
        sum += matrix[i][j];

// 4. Use bitwise operations
x = x * 2;   // Slow
x = x << 1;  // Fast

x = x / 2;   // Slow
x = x >> 1;  // Fast

// 5. Avoid repeated dereferencing
// Bad:
for (int i = 0; i < 100; i++) {
    ptr->member->value = i;
}
// Good:
int *val = &ptr->member->value;
for (int i = 0; i < 100; i++) {
    *val = i;
}

// 6. Use static for internal functions
static void internal_helper() { }

// 7. Profile before optimizing
// gcc -pg program.c -o program
// ./program
// gprof program gmon.out
```

### 13.4 Debugging Techniques

```c
// 1. Assertions
#include <assert.h>
assert(ptr != NULL);
assert(size > 0);

// 2. Debug macros
#ifdef DEBUG
    #define DEBUG_PRINT(fmt, ...) \
        fprintf(stderr, "DEBUG %s:%d: " fmt, __FILE__, __LINE__, ##__VA_ARGS__)
#else
    #define DEBUG_PRINT(fmt, ...) /* nothing */
#endif

DEBUG_PRINT("Value: %d\n", x);

// 3. Valgrind (memory debugging)
// valgrind --leak-check=full ./program

// 4. GDB (debugger)
// gcc -g program.c -o program
// gdb ./program
/*
(gdb) break main
(gdb) run
(gdb) print variable
(gdb) next
(gdb) step
(gdb) continue
(gdb) backtrace
*/

// 5. Static analysis
// gcc -Wall -Wextra -Werror
// clang --analyze
// cppcheck program.c
```

---

## 14. Learning Path & Resources

### Phase 1: Fundamentals (Weeks 1-4)

**Goal:** Master basic syntax, data types, and control flow

#### Week 1: Setup & Basic Syntax

**Tutorial 1: Hello World & Compilation**
```c
// hello.c - Your first C program
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

**Compile and Run:**
```bash
# Basic compilation
gcc hello.c -o hello
./hello

# With warnings and debugging symbols
gcc -Wall -Wextra -g -std=c11 hello.c -o hello

# See preprocessing output
gcc -E hello.c -o hello.i
cat hello.i  # See expanded code

# See assembly code
gcc -S hello.c -o hello.s
cat hello.s  # See assembly instructions
```

**Tutorial 2: Variables & Data Types**
```c
// datatypes.c - Understanding sizes and ranges
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main(void) {
    // Integer types
    char c = 'A';
    short s = 1000;
    int i = 100000;
    long l = 1000000000L;
    long long ll = 1000000000000LL;
    
    // Unsigned types
    unsigned int ui = 4000000000U;
    
    // Floating point
    float f = 3.14f;
    double d = 3.14159265359;
    
    // Print sizes
    printf("=== Type Sizes ===\n");
    printf("char: %zu bytes\n", sizeof(char));
    printf("short: %zu bytes\n", sizeof(short));
    printf("int: %zu bytes\n", sizeof(int));
    printf("long: %zu bytes\n", sizeof(long));
    printf("long long: %zu bytes\n", sizeof(long long));
    printf("float: %zu bytes\n", sizeof(float));
    printf("double: %zu bytes\n", sizeof(double));
    
    // Print ranges
    printf("\n=== Integer Ranges ===\n");
    printf("char: %d to %d\n", CHAR_MIN, CHAR_MAX);
    printf("short: %d to %d\n", SHRT_MIN, SHRT_MAX);
    printf("int: %d to %d\n", INT_MIN, INT_MAX);
    printf("long: %ld to %ld\n", LONG_MIN, LONG_MAX);
    
    // Print precision
    printf("\n=== Float Precision ===\n");
    printf("float digits: %d\n", FLT_DIG);
    printf("double digits: %d\n", DBL_DIG);
    
    return 0;
}
```

**Exercise:**
1. Write a program to swap two variables without using a third variable
2. Create a temperature converter (Celsius ↔ Fahrenheit)
3. Calculate area and perimeter of different shapes

#### Week 2: Control Flow & Operators

**Tutorial 3: Operators & Expressions**
```c
// operators.c - Mastering C operators
#include <stdio.h>

int main(void) {
    // Arithmetic operators
    int a = 10, b = 3;
    printf("a + b = %d\n", a + b);    // 13
    printf("a - b = %d\n", a - b);    // 7
    printf("a * b = %d\n", a * b);    // 30
    printf("a / b = %d\n", a / b);    // 3 (integer division)
    printf("a %% b = %d\n", a % b);   // 1 (modulo)
    
    // Increment/Decrement
    int x = 5;
    printf("x = %d\n", x);          // 5
    printf("x++ = %d\n", x++);      // 5, then x becomes 6
    printf("x = %d\n", x);          // 6
    printf("++x = %d\n", ++x);      // 7, x is now 7
    
    // Comparison operators
    printf("\n=== Comparisons ===\n");
    printf("10 > 5: %d\n", 10 > 5);   // 1 (true)
    printf("10 < 5: %d\n", 10 < 5);   // 0 (false)
    printf("10 == 10: %d\n", 10 == 10); // 1
    printf("10 != 5: %d\n", 10 != 5);  // 1
    
    // Logical operators
    printf("\n=== Logical ===\n");
    printf("1 && 1: %d\n", 1 && 1);   // 1 (true)
    printf("1 && 0: %d\n", 1 && 0);   // 0 (false)
    printf("1 || 0: %d\n", 1 || 0);   // 1 (true)
    printf("!1: %d\n", !1);           // 0 (false)
    
    // Bitwise operators
    printf("\n=== Bitwise ===\n");
    unsigned int p = 60;  // 0011 1100
    unsigned int q = 13;  // 0000 1101
    printf("p & q = %u\n", p & q);    // 12 (0000 1100)
    printf("p | q = %u\n", p | q);    // 61 (0011 1101)
    printf("p ^ q = %u\n", p ^ q);    // 49 (0011 0001)
    printf("~p = %u\n", ~p);          // Bitwise NOT
    printf("p << 2 = %u\n", p << 2);  // 240 (left shift)
    printf("p >> 2 = %u\n", p >> 2);  // 15 (right shift)
    
    return 0;
}
```

**Tutorial 4: Control Structures**
```c
// control.c - Decision making and loops
#include <stdio.h>

// Function prototypes
void if_else_demo(void);
void switch_demo(void);
void loop_demo(void);
void nested_loop_demo(void);

int main(void) {
    if_else_demo();
    switch_demo();
    loop_demo();
    nested_loop_demo();
    return 0;
}

void if_else_demo(void) {
    printf("=== If-Else Demo ===\n");
    
    int score = 85;
    
    if (score >= 90) {
        printf("Grade: A\n");
    } else if (score >= 80) {
        printf("Grade: B\n");
    } else if (score >= 70) {
        printf("Grade: C\n");
    } else if (score >= 60) {
        printf("Grade: D\n");
    } else {
        printf("Grade: F\n");
    }
    
    // Ternary operator
    int max = (10 > 5) ? 10 : 5;
    printf("Max: %d\n", max);
}

void switch_demo(void) {
    printf("\n=== Switch Demo ===\n");
    
    int day = 3;
    
    switch (day) {
        case 1:
            printf("Monday\n");
            break;
        case 2:
            printf("Tuesday\n");
            break;
        case 3:
            printf("Wednesday\n");
            break;
        case 4:
            printf("Thursday\n");
            break;
        case 5:
            printf("Friday\n");
            break;
        case 6:
        case 7:
            printf("Weekend\n");
            break;
        default:
            printf("Invalid day\n");
    }
}

void loop_demo(void) {
    printf("\n=== Loop Demo ===\n");
    
    // For loop
    printf("For loop: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", i);
    }
    printf("\n");
    
    // While loop
    printf("While loop: ");
    int j = 0;
    while (j < 5) {
        printf("%d ", j);
        j++;
    }
    printf("\n");
    
    // Do-while loop
    printf("Do-while loop: ");
    int k = 0;
    do {
        printf("%d ", k);
        k++;
    } while (k < 5);
    printf("\n");
    
    // Break and continue
    printf("Break demo: ");
    for (int i = 0; i < 10; i++) {
        if (i == 5) break;
        printf("%d ", i);
    }
    printf("\n");
    
    printf("Continue demo: ");
    for (int i = 0; i < 10; i++) {
        if (i % 2 == 0) continue;
        printf("%d ", i);
    }
    printf("\n");
}

void nested_loop_demo(void) {
    printf("\n=== Nested Loops ===\n");
    
    // Multiplication table
    for (int i = 1; i <= 5; i++) {
        for (int j = 1; j <= 5; j++) {
            printf("%4d", i * j);
        }
        printf("\n");
    }
}
```

**Exercises:**
1. Prime number checker
2. Fibonacci sequence generator
3. Pattern printing (pyramids, diamonds)
4. Number guessing game
5. Simple calculator

#### Week 3: Functions & Arrays

**Tutorial 5: Functions**
```c
// functions.c - Function mastery
#include <stdio.h>
#include <stdbool.h>

// Function declarations
int add(int a, int b);
int factorial(int n);
int factorial_iterative(int n);
bool is_prime(int n);
void swap_by_value(int a, int b);
void swap_by_reference(int *a, int *b);
int fibonacci(int n);
void print_array(int arr[], int size);

int main(void) {
    // Basic function call
    printf("=== Basic Functions ===\n");
    int sum = add(10, 20);
    printf("10 + 20 = %d\n", sum);
    
    // Recursive function
    printf("\n=== Recursion ===\n");
    printf("5! = %d\n", factorial(5));
    printf("5! (iterative) = %d\n", factorial_iterative(5));
    
    // Boolean function
    printf("\n=== Prime Check ===\n");
    for (int i = 2; i <= 20; i++) {
        if (is_prime(i)) {
            printf("%d ", i);
        }
    }
    printf("\n");
    
    // Pass by value vs reference
    printf("\n=== Pass by Value vs Reference ===\n");
    int x = 10, y = 20;
    printf("Before swap: x=%d, y=%d\n", x, y);
    swap_by_value(x, y);
    printf("After swap_by_value: x=%d, y=%d\n", x, y);
    swap_by_reference(&x, &y);
    printf("After swap_by_reference: x=%d, y=%d\n", x, y);
    
    // Fibonacci
    printf("\n=== Fibonacci ===\n");
    for (int i = 0; i < 10; i++) {
        printf("%d ", fibonacci(i));
    }
    printf("\n");
    
    return 0;
}

int add(int a, int b) {
    return a + b;
}

int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

int factorial_iterative(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

bool is_prime(int n) {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    
    for (int i = 5; i * i <= n; i += 6) {
        if (n % i == 0 || n % (i + 2) == 0)
            return false;
    }
    return true;
}

void swap_by_value(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    // Changes don't affect original variables
}

void swap_by_reference(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
    // Changes affect original variables
}

int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

void print_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}
```

**Tutorial 6: Arrays**
```c
// arrays.c - Array operations
#include <stdio.h>
#include <string.h>

#define SIZE 10

void print_array(int arr[], int size);
int find_max(int arr[], int size);
int find_min(int arr[], int size);
float calculate_average(int arr[], int size);
void reverse_array(int arr[], int size);
int linear_search(int arr[], int size, int key);
void bubble_sort(int arr[], int size);

int main(void) {
    // Array initialization
    int numbers[SIZE] = {64, 34, 25, 12, 22, 11, 90, 88, 45, 50};
    
    printf("Original array: ");
    print_array(numbers, SIZE);
    
    // Find max and min
    printf("\nMax: %d\n", find_max(numbers, SIZE));
    printf("Min: %d\n", find_min(numbers, SIZE));
    printf("Average: %.2f\n", calculate_average(numbers, SIZE));
    
    // Search
    int key = 22;
    int index = linear_search(numbers, SIZE, key);
    if (index != -1) {
        printf("\nFound %d at index %d\n", key, index);
    }
    
    // Reverse
    reverse_array(numbers, SIZE);
    printf("\nReversed array: ");
    print_array(numbers, SIZE);
    
    // Sort
    bubble_sort(numbers, SIZE);
    printf("\nSorted array: ");
    print_array(numbers, SIZE);
    
    // 2D array
    printf("\n=== 2D Array ===\n");
    int matrix[3][3] = {
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9}
    };
    
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d ", matrix[i][j]);
        }
        printf("\n");
    }
    
    return 0;
}

void print_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int find_max(int arr[], int size) {
    int max = arr[0];
    for (int i = 1; i < size; i++) {
        if (arr[i] > max) {
            max = arr[i];
        }
    }
    return max;
}

int find_min(int arr[], int size) {
    int min = arr[0];
    for (int i = 1; i < size; i++) {
        if (arr[i] < min) {
            min = arr[i];
        }
    }
    return min;
}

float calculate_average(int arr[], int size) {
    int sum = 0;
    for (int i = 0; i < size; i++) {
        sum += arr[i];
    }
    return (float)sum / size;
}

void reverse_array(int arr[], int size) {
    for (int i = 0; i < size / 2; i++) {
        int temp = arr[i];
        arr[i] = arr[size - 1 - i];
        arr[size - 1 - i] = temp;
    }
}

int linear_search(int arr[], int size, int key) {
    for (int i = 0; i < size; i++) {
        if (arr[i] == key) {
            return i;
        }
    }
    return -1;
}

void bubble_sort(int arr[], int size) {
    for (int i = 0; i < size - 1; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

**Exercises:**
1. Matrix addition and multiplication
2. Array rotation
3. Remove duplicates from array
4. Find second largest element
5. Merge two sorted arrays

#### Week 4: Strings & Practice

**Tutorial 7: String Manipulation**
```c
// strings.c - String operations
#include <stdio.h>
#include <string.h>
#include <ctype.h>

void custom_strlen_demo(void);
void custom_strcpy_demo(void);
void custom_strcmp_demo(void);
void string_reverse_demo(void);
void palindrome_check_demo(void);
void word_count_demo(void);
void char_frequency_demo(void);

// Custom string functions
size_t my_strlen(const char *str);
void my_strcpy(char *dest, const char *src);
int my_strcmp(const char *str1, const char *str2);
void reverse_string(char *str);
int is_palindrome(const char *str);

int main(void) {
    custom_strlen_demo();
    custom_strcpy_demo();
    custom_strcmp_demo();
    string_reverse_demo();
    palindrome_check_demo();
    word_count_demo();
    char_frequency_demo();
    return 0;
}

void custom_strlen_demo(void) {
    printf("=== String Length ===\n");
    char str[] = "Hello, World!";
    printf("String: %s\n", str);
    printf("Length (built-in): %zu\n", strlen(str));
    printf("Length (custom): %zu\n", my_strlen(str));
}

void custom_strcpy_demo(void) {
    printf("\n=== String Copy ===\n");
    char src[] = "Hello";
    char dest[50];
    my_strcpy(dest, src);
    printf("Source: %s\n", src);
    printf("Destination: %s\n", dest);
}

void custom_strcmp_demo(void) {
    printf("\n=== String Compare ===\n");
    char str1[] = "Apple";
    char str2[] = "Banana";
    char str3[] = "Apple";
    
    printf("%s vs %s: %d\n", str1, str2, my_strcmp(str1, str2));
    printf("%s vs %s: %d\n", str1, str3, my_strcmp(str1, str3));
}

void string_reverse_demo(void) {
    printf("\n=== String Reverse ===\n");
    char str[] = "Hello";
    printf("Original: %s\n", str);
    reverse_string(str);
    printf("Reversed: %s\n", str);
}

void palindrome_check_demo(void) {
    printf("\n=== Palindrome Check ===\n");
    char *words[] = {"radar", "hello", "level", "world", "madam"};
    
    for (int i = 0; i < 5; i++) {
        printf("%s: %s\n", words[i], 
               is_palindrome(words[i]) ? "palindrome" : "not palindrome");
    }
}

void word_count_demo(void) {
    printf("\n=== Word Count ===\n");
    char sentence[] = "The quick brown fox jumps over the lazy dog";
    
    int count = 0;
    int in_word = 0;
    
    for (int i = 0; sentence[i] != '\0'; i++) {
        if (isspace(sentence[i])) {
            in_word = 0;
        } else if (!in_word) {
            in_word = 1;
            count++;
        }
    }
    
    printf("Sentence: %s\n", sentence);
    printf("Word count: %d\n", count);
}

void char_frequency_demo(void) {
    printf("\n=== Character Frequency ===\n");
    char str[] = "hello world";
    int freq[256] = {0};
    
    for (int i = 0; str[i] != '\0'; i++) {
        freq[(unsigned char)str[i]]++;
    }
    
    printf("String: %s\n", str);
    printf("Frequencies:\n");
    for (int i = 0; i < 256; i++) {
        if (freq[i] > 0 && !isspace(i)) {
            printf("%c: %d\n", i, freq[i]);
        }
    }
}

// Custom implementations
size_t my_strlen(const char *str) {
    size_t len = 0;
    while (str[len] != '\0') {
        len++;
    }
    return len;
}

void my_strcpy(char *dest, const char *src) {
    int i = 0;
    while ((dest[i] = src[i]) != '\0') {
        i++;
    }
}

int my_strcmp(const char *str1, const char *str2) {
    while (*str1 && (*str1 == *str2)) {
        str1++;
        str2++;
    }
    return *(unsigned char *)str1 - *(unsigned char *)str2;
}

void reverse_string(char *str) {
    int len = strlen(str);
    for (int i = 0; i < len / 2; i++) {
        char temp = str[i];
        str[i] = str[len - 1 - i];
        str[len - 1 - i] = temp;
    }
}

int is_palindrome(const char *str) {
    int left = 0;
    int right = strlen(str) - 1;
    
    while (left < right) {
        if (str[left] != str[right]) {
            return 0;
        }
        left++;
        right--;
    }
    return 1;
}
```

**Week 4 Project: Student Management System**
```c
// student_mgmt.c - Simple database
#include <stdio.h>
#include <string.h>

#define MAX_STUDENTS 100

typedef struct {
    int id;
    char name[50];
    float marks;
} Student;

Student students[MAX_STUDENTS];
int student_count = 0;

void add_student(void);
void display_students(void);
void search_student(void);
void calculate_average(void);

int main(void) {
    int choice;
    
    while (1) {
        printf("\n=== Student Management System ===\n");
        printf("1. Add Student\n");
        printf("2. Display All Students\n");
        printf("3. Search Student\n");
        printf("4. Calculate Average\n");
        printf("5. Exit\n");
        printf("Enter choice: ");
        scanf("%d", &choice);
        
        switch (choice) {
            case 1: add_student(); break;
            case 2: display_students(); break;
            case 3: search_student(); break;
            case 4: calculate_average(); break;
            case 5: return 0;
            default: printf("Invalid choice!\n");
        }
    }
    
    return 0;
}

void add_student(void) {
    if (student_count >= MAX_STUDENTS) {
        printf("Database full!\n");
        return;
    }
    
    Student *s = &students[student_count];
    s->id = student_count + 1;
    
    printf("Enter name: ");
    scanf(" %[^\n]", s->name);
    printf("Enter marks: ");
    scanf("%f", &s->marks);
    
    student_count++;
    printf("Student added successfully!\n");
}

void display_students(void) {
    printf("\nID\tName\t\tMarks\n");
    printf("================================\n");
    for (int i = 0; i < student_count; i++) {
        printf("%d\t%-15s\t%.2f\n", 
               students[i].id, 
               students[i].name, 
               students[i].marks);
    }
}

void search_student(void) {
    int id;
    printf("Enter student ID: ");
    scanf("%d", &id);
    
    for (int i = 0; i < student_count; i++) {
        if (students[i].id == id) {
            printf("Found: %s (Marks: %.2f)\n", 
                   students[i].name, students[i].marks);
            return;
        }
    }
    printf("Student not found!\n");
}

void calculate_average(void) {
    if (student_count == 0) {
        printf("No students!\n");
        return;
    }
    
    float sum = 0;
    for (int i = 0; i < student_count; i++) {
        sum += students[i].marks;
    }
    printf("Average marks: %.2f\n", sum / student_count);
}
```

**Phase 1 Milestones:**
- ✅ Understand compilation process
- ✅ Master all operators
- ✅ Write 50+ programs
- ✅ Complete one small project
- ✅ Debug programs using GDB basics

### Phase 2: Intermediate (Weeks 5-8)

**Goal:** Master pointers, memory management, and advanced language features

#### Week 5: Pointers Deep Dive

**Tutorial 8: Pointer Fundamentals**
```c
// pointers_basics.c
#include <stdio.h>

void pointer_fundamentals(void);
void pointer_arithmetic(void);
void pointer_and_arrays(void);
void pointer_to_pointer(void);

int main(void) {
    pointer_fundamentals();
    pointer_arithmetic();
    pointer_and_arrays();
    pointer_to_pointer();
    return 0;
}

void pointer_fundamentals(void) {
    printf("=== Pointer Basics ===\n");
    
    int value = 42;
    int *ptr = &value;
    
    printf("value = %d\n", value);
    printf("&value = %p\n", (void*)&value);
    printf("ptr = %p\n", (void*)ptr);
    printf("*ptr = %d\n", *ptr);
    
    // Modify through pointer
    *ptr = 100;
    printf("After *ptr = 100:\n");
    printf("value = %d\n", value);
    printf("*ptr = %d\n", *ptr);
}

void pointer_arithmetic(void) {
    printf("\n=== Pointer Arithmetic ===\n");
    
    int arr[] = {10, 20, 30, 40, 50};
    int *p = arr;
    
    printf("Array: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    // Accessing through pointer
    printf("\nUsing pointer arithmetic:\n");
    printf("*p = %d\n", *p);         // 10
    printf("*(p+1) = %d\n", *(p+1)); // 20
    printf("*(p+2) = %d\n", *(p+2)); // 30
    
    // Incrementing pointer
    printf("\nIncrementing pointer:\n");
    for (int i = 0; i < 5; i++) {
        printf("%d ", *p);
        p++;
    }
    printf("\n");
    
    // Pointer difference
    int *start = arr;
    int *end = &arr[4];
    printf("\nPointer difference: %ld\n", end - start);
}

void pointer_and_arrays(void) {
    printf("\n=== Pointers and Arrays ===\n");
    
    int arr[] = {1, 2, 3, 4, 5};
    
    // These are equivalent:
    printf("arr[2] = %d\n", arr[2]);
    printf("*(arr+2) = %d\n", *(arr+2));
    printf("2[arr] = %d\n", 2[arr]);  // Yes, this works!
    
    // 2D array with pointers
    int matrix[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };
    
    int (*ptr)[4] = matrix;  // Pointer to array of 4 ints
    
    printf("\nAccessing 2D array with pointer:\n");
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            printf("%3d ", *(*(ptr + i) + j));
        }
        printf("\n");
    }
}

void pointer_to_pointer(void) {
    printf("\n=== Pointer to Pointer ===\n");
    
    int value = 42;
    int *ptr = &value;
    int **ptr2 = &ptr;
    
    printf("value = %d\n", value);
    printf("*ptr = %d\n", *ptr);
    printf("**ptr2 = %d\n", **ptr2);
    
    // Modify through double pointer
    **ptr2 = 100;
    printf("After **ptr2 = 100:\n");
    printf("value = %d\n", value);
}
```

**Tutorial 9: Function Pointers**
```c
// function_pointers.c
#include <stdio.h>

// Function prototypes
int add(int a, int b);
int subtract(int a, int b);
int multiply(int a, int b);
int divide(int a, int b);

// Function that takes function pointer
int calculate(int a, int b, int (*operation)(int, int));

// Callback example
void process_array(int arr[], int size, void (*callback)(int));
void print_value(int val);
void square_value(int val);

int main(void) {
    printf("=== Function Pointers ===\n");
    
    // Function pointer declaration
    int (*func_ptr)(int, int);
    
    // Assign function to pointer
    func_ptr = add;
    printf("10 + 5 = %d\n", func_ptr(10, 5));
    
    func_ptr = multiply;
    printf("10 * 5 = %d\n", func_ptr(10, 5));
    
    // Using calculate function
    printf("\n=== Using calculate() ===\n");
    printf("15 + 5 = %d\n", calculate(15, 5, add));
    printf("15 - 5 = %d\n", calculate(15, 5, subtract));
    printf("15 * 5 = %d\n", calculate(15, 5, multiply));
    printf("15 / 5 = %d\n", calculate(15, 5, divide));
    
    // Array of function pointers (calculator)
    int (*operations[4])(int, int) = {add, subtract, multiply, divide};
    char *op_names[] = {"+", "-", "*", "/"};
    
    printf("\n=== Calculator with Array of Function Pointers ===\n");
    for (int i = 0; i < 4; i++) {
        printf("20 %s 4 = %d\n", op_names[i], operations[i](20, 4));
    }
    
    // Callback example
    printf("\n=== Callbacks ===\n");
    int numbers[] = {1, 2, 3, 4, 5};
    
    printf("Print values:\n");
    process_array(numbers, 5, print_value);
    
    printf("\nSquare values:\n");
    process_array(numbers, 5, square_value);
    
    return 0;
}

int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }
int multiply(int a, int b) { return a * b; }
int divide(int a, int b) { return a / b; }

int calculate(int a, int b, int (*operation)(int, int)) {
    return operation(a, b);
}

void process_array(int arr[], int size, void (*callback)(int)) {
    for (int i = 0; i < size; i++) {
        callback(arr[i]);
    }
}

void print_value(int val) {
    printf("%d ", val);
}

void square_value(int val) {
    printf("%d ", val * val);
}
```

#### Week 6: Dynamic Memory Management

**Tutorial 10: malloc, calloc, realloc, free**
```c
// memory_management.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void malloc_demo(void);
void calloc_demo(void);
void realloc_demo(void);
void dynamic_array_demo(void);
void dynamic_2d_array_demo(void);
void memory_leak_demo(void);

int main(void) {
    malloc_demo();
    calloc_demo();
    realloc_demo();
    dynamic_array_demo();
    dynamic_2d_array_demo();
    return 0;
}

void malloc_demo(void) {
    printf("=== malloc Demo ===\n");
    
    // Allocate memory for 5 integers
    int *arr = (int *)malloc(5 * sizeof(int));
    
    if (arr == NULL) {
        fprintf(stderr, "Memory allocation failed!\n");
        return;
    }
    
    // Initialize and print
    for (int i = 0; i < 5; i++) {
        arr[i] = i * 10;
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    // Free memory
    free(arr);
    arr = NULL;  // Avoid dangling pointer
}

void calloc_demo(void) {
    printf("\n=== calloc Demo ===\n");
    
    // Allocate and zero-initialize
    int *arr = (int *)calloc(5, sizeof(int));
    
    if (arr == NULL) {
        fprintf(stderr, "Memory allocation failed!\n");
        return;
    }
    
    printf("calloc initializes to zero: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);  // All zeros
    }
    printf("\n");
    
    free(arr);
}

void realloc_demo(void) {
    printf("\n=== realloc Demo ===\n");
    
    // Initial allocation
    int *arr = (int *)malloc(3 * sizeof(int));
    if (arr == NULL) return;
    
    for (int i = 0; i < 3; i++) {
        arr[i] = i + 1;
    }
    
    printf("Original: ");
    for (int i = 0; i < 3; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    // Resize to 6 elements
    int *temp = (int *)realloc(arr, 6 * sizeof(int));
    if (temp == NULL) {
        free(arr);  // Original still valid
        return;
    }
    arr = temp;
    
    // Initialize new elements
    for (int i = 3; i < 6; i++) {
        arr[i] = i + 1;
    }
    
    printf("After realloc: ");
    for (int i = 0; i < 6; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    free(arr);
}

void dynamic_array_demo(void) {
    printf("\n=== Dynamic Array (Vector) ===\n");
    
    int capacity = 4;
    int size = 0;
    int *arr = (int *)malloc(capacity * sizeof(int));
    
    if (arr == NULL) return;
    
    // Add elements (with dynamic resizing)
    for (int i = 0; i < 10; i++) {
        if (size >= capacity) {
            // Double capacity
            capacity *= 2;
            int *temp = (int *)realloc(arr, capacity * sizeof(int));
            if (temp == NULL) {
                free(arr);
                return;
            }
            arr = temp;
            printf("Resized to capacity: %d\n", capacity);
        }
        arr[size++] = i * 10;
    }
    
    printf("Final array: ");
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    free(arr);
}

void dynamic_2d_array_demo(void) {
    printf("\n=== Dynamic 2D Array ===\n");
    
    int rows = 3, cols = 4;
    
    // Allocate array of row pointers
    int **matrix = (int **)malloc(rows * sizeof(int *));
    if (matrix == NULL) return;
    
    // Allocate each row
    for (int i = 0; i < rows; i++) {
        matrix[i] = (int *)malloc(cols * sizeof(int));
        if (matrix[i] == NULL) {
            // Cleanup on failure
            for (int j = 0; j < i; j++) {
                free(matrix[j]);
            }
            free(matrix);
            return;
        }
    }
    
    // Initialize
    int value = 1;
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            matrix[i][j] = value++;
        }
    }
    
    // Print
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%3d ", matrix[i][j]);
        }
        printf("\n");
    }
    
    // Free memory
    for (int i = 0; i < rows; i++) {
        free(matrix[i]);
    }
    free(matrix);
}
```

#### Week 7: Structures and File I/O

**Tutorial 11: Structures**
```c
// structures.c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// Basic structure
typedef struct {
    int x;
    int y;
} Point;

// Nested structure
typedef struct {
    int day;
    int month;
    int year;
} Date;

typedef struct {
    char name[50];
    int age;
    Date birth_date;
    float salary;
} Employee;

// Linked list node
typedef struct Node {
    int data;
    struct Node *next;
} Node;

void structure_basics(void);
void nested_structures(void);
void array_of_structures(void);
void structure_pointers(void);
void linked_list_demo(void);

int main(void) {
    structure_basics();
    nested_structures();
    array_of_structures();
    structure_pointers();
    linked_list_demo();
    return 0;
}

void structure_basics(void) {
    printf("=== Structure Basics ===\n");
    
    Point p1 = {10, 20};
    Point p2;
    p2.x = 30;
    p2.y = 40;
    
    printf("p1: (%d, %d)\n", p1.x, p1.y);
    printf("p2: (%d, %d)\n", p2.x, p2.y);
    
    // Designated initializers (C99)
    Point p3 = {.y = 100, .x = 50};
    printf("p3: (%d, %d)\n", p3.x, p3.y);
}

void nested_structures(void) {
    printf("\n=== Nested Structures ===\n");
    
    Employee emp = {
        .name = "John Doe",
        .age = 30,
        .birth_date = {15, 6, 1993},
        .salary = 50000.0
    };
    
    printf("Name: %s\n", emp.name);
    printf("Age: %d\n", emp.age);
    printf("Birth Date: %d/%d/%d\n", 
           emp.birth_date.day,
           emp.birth_date.month,
           emp.birth_date.year);
    printf("Salary: $%.2f\n", emp.salary);
}

void array_of_structures(void) {
    printf("\n=== Array of Structures ===\n");
    
    Employee employees[3] = {
        {"Alice", 25, {10, 3, 1998}, 45000.0},
        {"Bob", 30, {20, 7, 1993}, 55000.0},
        {"Charlie", 28, {5, 12, 1995}, 48000.0}
    };
    
    printf("%-15s %-5s %-15s %s\n", "Name", "Age", "Birth Date", "Salary");
    printf("====================================================\n");
    
    for (int i = 0; i < 3; i++) {
        printf("%-15s %-5d %02d/%02d/%04d      $%.2f\n",
               employees[i].name,
               employees[i].age,
               employees[i].birth_date.day,
               employees[i].birth_date.month,
               employees[i].birth_date.year,
               employees[i].salary);
    }
}

void structure_pointers(void) {
    printf("\n=== Structure Pointers ===\n");
    
    Employee *emp = (Employee *)malloc(sizeof(Employee));
    if (emp == NULL) return;
    
    strcpy(emp->name, "David");  // Arrow operator
    emp->age = 35;
    emp->salary = 60000.0;
    
    printf("Name: %s\n", emp->name);
    printf("Age: %d\n", emp->age);
    printf("Salary: $%.2f\n", emp->salary);
    
    free(emp);
}

void linked_list_demo(void) {
    printf("\n=== Linked List Demo ===\n");
    
    // Create nodes
    Node *head = (Node *)malloc(sizeof(Node));
    Node *second = (Node *)malloc(sizeof(Node));
    Node *third = (Node *)malloc(sizeof(Node));
    
    if (!head || !second || !third) {
        free(head); free(second); free(third);
        return;
    }
    
    // Link nodes
    head->data = 1;
    head->next = second;
    
    second->data = 2;
    second->next = third;
    
    third->data = 3;
    third->next = NULL;
    
    // Traverse and print
    printf("Linked List: ");
    Node *current = head;
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
    
    // Free memory
    free(head);
    free(second);
    free(third);
}
```

**Tutorial 12: File I/O**
```c
// file_io.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int id;
    char name[50];
    float marks;
} Student;

void text_file_write(void);
void text_file_read(void);
void binary_file_write(void);
void binary_file_read(void);
void file_append(void);
void file_copy(void);

int main(void) {
    text_file_write();
    text_file_read();
    binary_file_write();
    binary_file_read();
    file_append();
    file_copy();
    return 0;
}

void text_file_write(void) {
    printf("=== Writing to Text File ===\n");
    
    FILE *fp = fopen("students.txt", "w");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    fprintf(fp, "ID\tName\t\tMarks\n");
    fprintf(fp, "1\tAlice\t\t85.5\n");
    fprintf(fp, "2\tBob\t\t92.0\n");
    fprintf(fp, "3\tCharlie\t\t78.5\n");
    
    fclose(fp);
    printf("Data written to students.txt\n");
}

void text_file_read(void) {
    printf("\n=== Reading from Text File ===\n");
    
    FILE *fp = fopen("students.txt", "r");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    char line[256];
    while (fgets(line, sizeof(line), fp)) {
        printf("%s", line);
    }
    
    fclose(fp);
}

void binary_file_write(void) {
    printf("\n=== Writing to Binary File ===\n");
    
    FILE *fp = fopen("students.dat", "wb");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    Student students[] = {
        {1, "Alice", 85.5},
        {2, "Bob", 92.0},
        {3, "Charlie", 78.5}
    };
    
    fwrite(students, sizeof(Student), 3, fp);
    fclose(fp);
    printf("Binary data written to students.dat\n");
}

void binary_file_read(void) {
    printf("\n=== Reading from Binary File ===\n");
    
    FILE *fp = fopen("students.dat", "rb");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    Student s;
    printf("ID\tName\t\tMarks\n");
    while (fread(&s, sizeof(Student), 1, fp) == 1) {
        printf("%d\t%-15s\t%.2f\n", s.id, s.name, s.marks);
    }
    
    fclose(fp);
}

void file_append(void) {
    printf("\n=== Appending to File ===\n");
    
    FILE *fp = fopen("students.txt", "a");
    if (fp == NULL) {
        perror("Error opening file");
        return;
    }
    
    fprintf(fp, "4\tDavid\t\t88.0\n");
    fclose(fp);
    printf("Data appended to students.txt\n");
}

void file_copy(void) {
    printf("\n=== Copying File ===\n");
    
    FILE *src = fopen("students.txt", "r");
    FILE *dest = fopen("students_copy.txt", "w");
    
    if (src == NULL || dest == NULL) {
        perror("Error opening file");
        if (src) fclose(src);
        if (dest) fclose(dest);
        return;
    }
    
    char ch;
    while ((ch = fgetc(src)) != EOF) {
        fputc(ch, dest);
    }
    
    fclose(src);
    fclose(dest);
    printf("File copied to students_copy.txt\n");
}
```

#### Week 8: Complete Intermediate Project

**Project: Contact Management System**
```c
// contact_manager.c - Complete intermediate project
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_CONTACTS 100
#define FILENAME "contacts.dat"

typedef struct {
    int id;
    char name[50];
    char phone[15];
    char email[50];
} Contact;

Contact contacts[MAX_CONTACTS];
int contact_count = 0;

// Function prototypes
void load_contacts(void);
void save_contacts(void);
void add_contact(void);
void display_contacts(void);
void search_contact(void);
void delete_contact(void);
void update_contact(void);
int find_contact_by_id(int id);

int main(void) {
    load_contacts();
    
    int choice;
    while (1) {
        printf("\n=== Contact Management System ===\n");
        printf("1. Add Contact\n");
        printf("2. Display All Contacts\n");
        printf("3. Search Contact\n");
        printf("4. Update Contact\n");
        printf("5. Delete Contact\n");
        printf("6. Save and Exit\n");
        printf("Enter choice: ");
        scanf("%d", &choice);
        
        switch (choice) {
            case 1: add_contact(); break;
            case 2: display_contacts(); break;
            case 3: search_contact(); break;
            case 4: update_contact(); break;
            case 5: delete_contact(); break;
            case 6:
                save_contacts();
                printf("Contacts saved. Goodbye!\n");
                return 0;
            default:
                printf("Invalid choice!\n");
        }
    }
    
    return 0;
}

void load_contacts(void) {
    FILE *fp = fopen(FILENAME, "rb");
    if (fp == NULL) {
        printf("No existing contacts file. Starting fresh.\n");
        return;
    }
    
    fread(&contact_count, sizeof(int), 1, fp);
    fread(contacts, sizeof(Contact), contact_count, fp);
    fclose(fp);
    printf("Loaded %d contacts.\n", contact_count);
}

void save_contacts(void) {
    FILE *fp = fopen(FILENAME, "wb");
    if (fp == NULL) {
        perror("Error saving contacts");
        return;
    }
    
    fwrite(&contact_count, sizeof(int), 1, fp);
    fwrite(contacts, sizeof(Contact), contact_count, fp);
    fclose(fp);
}

void add_contact(void) {
    if (contact_count >= MAX_CONTACTS) {
        printf("Contact list is full!\n");
        return;
    }
    
    Contact *c = &contacts[contact_count];
    c->id = contact_count + 1;
    
    printf("Enter name: ");
    scanf(" %[^\n]", c->name);
    printf("Enter phone: ");
    scanf("%s", c->phone);
    printf("Enter email: ");
    scanf("%s", c->email);
    
    contact_count++;
    printf("Contact added successfully! (ID: %d)\n", c->id);
}

void display_contacts(void) {
    if (contact_count == 0) {
        printf("No contacts to display!\n");
        return;
    }
    
    printf("\n%-5s %-20s %-15s %-30s\n", "ID", "Name", "Phone", "Email");
    printf("===================================================================\n");
    
    for (int i = 0; i < contact_count; i++) {
        printf("%-5d %-20s %-15s %-30s\n",
               contacts[i].id,
               contacts[i].name,
               contacts[i].phone,
               contacts[i].email);
    }
}

void search_contact(void) {
    char name[50];
    printf("Enter name to search: ");
    scanf(" %[^\n]", name);
    
    int found = 0;
    for (int i = 0; i < contact_count; i++) {
        if (strstr(contacts[i].name, name) != NULL) {
            if (!found) {
                printf("\n%-5s %-20s %-15s %-30s\n", "ID", "Name", "Phone", "Email");
                printf("===================================================================\n");
                found = 1;
            }
            printf("%-5d %-20s %-15s %-30s\n",
                   contacts[i].id,
                   contacts[i].name,
                   contacts[i].phone,
                   contacts[i].email);
        }
    }
    
    if (!found) {
        printf("No contacts found!\n");
    }
}

void delete_contact(void) {
    int id;
    printf("Enter contact ID to delete: ");
    scanf("%d", &id);
    
    int index = find_contact_by_id(id);
    if (index == -1) {
        printf("Contact not found!\n");
        return;
    }
    
    // Shift contacts
    for (int i = index; i < contact_count - 1; i++) {
        contacts[i] = contacts[i + 1];
    }
    contact_count--;
    
    printf("Contact deleted successfully!\n");
}

void update_contact(void) {
    int id;
    printf("Enter contact ID to update: ");
    scanf("%d", &id);
    
    int index = find_contact_by_id(id);
    if (index == -1) {
        printf("Contact not found!\n");
        return;
    }
    
    Contact *c = &contacts[index];
    printf("Enter new name (current: %s): ", c->name);
    scanf(" %[^\n]", c->name);
    printf("Enter new phone (current: %s): ", c->phone);
    scanf("%s", c->phone);
    printf("Enter new email (current: %s): ", c->email);
    scanf("%s", c->email);
    
    printf("Contact updated successfully!\n");
}

int find_contact_by_id(int id) {
    for (int i = 0; i < contact_count; i++) {
        if (contacts[i].id == id) {
            return i;
        }
    }
    return -1;
}
```

**Phase 2 Milestones:**
- ✅ Master all pointer types
- ✅ Understand memory management
- ✅ Use structures effectively
- ✅ Perform file I/O operations
- ✅ Complete contact management project
- ✅ Debug with Valgrind (memory leaks)

### Phase 3: Data Structures (Weeks 9-12)

**Goal:** Implement fundamental data structures from scratch

#### Week 9: Linked Lists

**Tutorial 13: Singly Linked List Implementation**
```c
// singly_linked_list.c - Complete implementation
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

typedef struct {
    Node *head;
    int size;
} LinkedList;

// Function prototypes
LinkedList* list_create(void);
void list_insert_front(LinkedList *list, int data);
void list_insert_back(LinkedList *list, int data);
void list_insert_at(LinkedList *list, int data, int position);
int list_delete_front(LinkedList *list);
int list_delete_back(LinkedList *list);
int list_delete_value(LinkedList *list, int value);
int list_search(LinkedList *list, int value);
void list_reverse(LinkedList *list);
Node* list_find_middle(LinkedList *list);
int list_has_cycle(LinkedList *list);
void list_remove_duplicates(LinkedList *list);
void list_print(LinkedList *list);
void list_free(LinkedList *list);

int main(void) {
    LinkedList *list = list_create();
    
    printf("=== Singly Linked List Operations ===\n");
    
    // Insert operations
    printf("\nInserting elements...\n");
    list_insert_back(list, 10);
    list_insert_back(list, 20);
    list_insert_back(list, 30);
    list_insert_front(list, 5);
    list_insert_at(list, 15, 2);
    
    printf("List: ");
    list_print(list);
    
    // Search
    int key = 20;
    if (list_search(list, key)) {
        printf("Found %d in list\n", key);
    }
    
    // Find middle
    Node *middle = list_find_middle(list);
    if (middle) {
        printf("Middle element: %d\n", middle->data);
    }
    
    // Delete operations
    printf("\nDeleting front element...\n");
    list_delete_front(list);
    list_print(list);
    
    printf("Deleting value 20...\n");
    list_delete_value(list, 20);
    list_print(list);
    
    // Reverse
    printf("\nReversing list...\n");
    list_reverse(list);
    list_print(list);
    
    // Test with duplicates
    list_insert_back(list, 10);
    list_insert_back(list, 15);
    printf("\nList with duplicates: ");
    list_print(list);
    
    list_remove_duplicates(list);
    printf("After removing duplicates: ");
    list_print(list);
    
    list_free(list);
    return 0;
}

LinkedList* list_create(void) {
    LinkedList *list = (LinkedList*)malloc(sizeof(LinkedList));
    if (list) {
        list->head = NULL;
        list->size = 0;
    }
    return list;
}

void list_insert_front(LinkedList *list, int data) {
    Node *new_node = (Node*)malloc(sizeof(Node));
    if (!new_node) return;
    
    new_node->data = data;
    new_node->next = list->head;
    list->head = new_node;
    list->size++;
}

void list_insert_back(LinkedList *list, int data) {
    Node *new_node = (Node*)malloc(sizeof(Node));
    if (!new_node) return;
    
    new_node->data = data;
    new_node->next = NULL;
    
    if (list->head == NULL) {
        list->head = new_node;
    } else {
        Node *current = list->head;
        while (current->next != NULL) {
            current = current->next;
        }
        current->next = new_node;
    }
    list->size++;
}

void list_insert_at(LinkedList *list, int data, int position) {
    if (position < 0 || position > list->size) return;
    
    if (position == 0) {
        list_insert_front(list, data);
        return;
    }
    
    Node *new_node = (Node*)malloc(sizeof(Node));
    if (!new_node) return;
    
    new_node->data = data;
    
    Node *current = list->head;
    for (int i = 0; i < position - 1; i++) {
        current = current->next;
    }
    
    new_node->next = current->next;
    current->next = new_node;
    list->size++;
}

int list_delete_front(LinkedList *list) {
    if (list->head == NULL) return 0;
    
    Node *temp = list->head;
    int data = temp->data;
    list->head = list->head->next;
    free(temp);
    list->size--;
    return data;
}

int list_delete_back(LinkedList *list) {
    if (list->head == NULL) return 0;
    
    if (list->head->next == NULL) {
        int data = list->head->data;
        free(list->head);
        list->head = NULL;
        list->size--;
        return data;
    }
    
    Node *current = list->head;
    while (current->next->next != NULL) {
        current = current->next;
    }
    
    int data = current->next->data;
    free(current->next);
    current->next = NULL;
    list->size--;
    return data;
}

int list_delete_value(LinkedList *list, int value) {
    if (list->head == NULL) return 0;
    
    if (list->head->data == value) {
        Node *temp = list->head;
        list->head = list->head->next;
        free(temp);
        list->size--;
        return 1;
    }
    
    Node *current = list->head;
    while (current->next != NULL && current->next->data != value) {
        current = current->next;
    }
    
    if (current->next == NULL) return 0;
    
    Node *temp = current->next;
    current->next = current->next->next;
    free(temp);
    list->size--;
    return 1;
}

int list_search(LinkedList *list, int value) {
    Node *current = list->head;
    while (current != NULL) {
        if (current->data == value) return 1;
        current = current->next;
    }
    return 0;
}

void list_reverse(LinkedList *list) {
    Node *prev = NULL;
    Node *current = list->head;
    Node *next = NULL;
    
    while (current != NULL) {
        next = current->next;
        current->next = prev;
        prev = current;
        current = next;
    }
    
    list->head = prev;
}

Node* list_find_middle(LinkedList *list) {
    if (list->head == NULL) return NULL;
    
    Node *slow = list->head;
    Node *fast = list->head;
    
    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
    }
    
    return slow;
}

int list_has_cycle(LinkedList *list) {
    if (list->head == NULL) return 0;
    
    Node *slow = list->head;
    Node *fast = list->head;
    
    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return 1;
    }
    
    return 0;
}

void list_remove_duplicates(LinkedList *list) {
    if (list->head == NULL) return;
    
    Node *current = list->head;
    while (current != NULL && current->next != NULL) {
        Node *runner = current;
        while (runner->next != NULL) {
            if (runner->next->data == current->data) {
                Node *temp = runner->next;
                runner->next = runner->next->next;
                free(temp);
                list->size--;
            } else {
                runner = runner->next;
            }
        }
        current = current->next;
    }
}

void list_print(LinkedList *list) {
    Node *current = list->head;
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}

void list_free(LinkedList *list) {
    Node *current = list->head;
    while (current != NULL) {
        Node *temp = current;
        current = current->next;
        free(temp);
    }
    free(list);
}
```

**Tutorial 14: Doubly Linked List**
```c
// doubly_linked_list.c
#include <stdio.h>
#include <stdlib.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

typedef struct {
    DNode *head;
    DNode *tail;
    int size;
} DoublyLinkedList;

// Function prototypes
DoublyLinkedList* dlist_create(void);
void dlist_insert_front(DoublyLinkedList *list, int data);
void dlist_insert_back(DoublyLinkedList *list, int data);
void dlist_delete_front(DoublyLinkedList *list);
void dlist_delete_back(DoublyLinkedList *list);
void dlist_print_forward(DoublyLinkedList *list);
void dlist_print_backward(DoublyLinkedList *list);
void dlist_free(DoublyLinkedList *list);

int main(void) {
    DoublyLinkedList *list = dlist_create();
    
    printf("=== Doubly Linked List ===\n");
    
    dlist_insert_back(list, 10);
    dlist_insert_back(list, 20);
    dlist_insert_back(list, 30);
    dlist_insert_front(list, 5);
    
    printf("Forward: ");
    dlist_print_forward(list);
    
    printf("Backward: ");
    dlist_print_backward(list);
    
    dlist_delete_front(list);
    printf("\nAfter deleting front: ");
    dlist_print_forward(list);
    
    dlist_delete_back(list);
    printf("After deleting back: ");
    dlist_print_forward(list);
    
    dlist_free(list);
    return 0;
}

DoublyLinkedList* dlist_create(void) {
    DoublyLinkedList *list = (DoublyLinkedList*)malloc(sizeof(DoublyLinkedList));
    if (list) {
        list->head = NULL;
        list->tail = NULL;
        list->size = 0;
    }
    return list;
}

void dlist_insert_front(DoublyLinkedList *list, int data) {
    DNode *new_node = (DNode*)malloc(sizeof(DNode));
    if (!new_node) return;
    
    new_node->data = data;
    new_node->prev = NULL;
    new_node->next = list->head;
    
    if (list->head == NULL) {
        list->head = list->tail = new_node;
    } else {
        list->head->prev = new_node;
        list->head = new_node;
    }
    list->size++;
}

void dlist_insert_back(DoublyLinkedList *list, int data) {
    DNode *new_node = (DNode*)malloc(sizeof(DNode));
    if (!new_node) return;
    
    new_node->data = data;
    new_node->next = NULL;
    new_node->prev = list->tail;
    
    if (list->tail == NULL) {
        list->head = list->tail = new_node;
    } else {
        list->tail->next = new_node;
        list->tail = new_node;
    }
    list->size++;
}

void dlist_delete_front(DoublyLinkedList *list) {
    if (list->head == NULL) return;
    
    DNode *temp = list->head;
    list->head = list->head->next;
    
    if (list->head == NULL) {
        list->tail = NULL;
    } else {
        list->head->prev = NULL;
    }
    
    free(temp);
    list->size--;
}

void dlist_delete_back(DoublyLinkedList *list) {
    if (list->tail == NULL) return;
    
    DNode *temp = list->tail;
    list->tail = list->tail->prev;
    
    if (list->tail == NULL) {
        list->head = NULL;
    } else {
        list->tail->next = NULL;
    }
    
    free(temp);
    list->size--;
}

void dlist_print_forward(DoublyLinkedList *list) {
    DNode *current = list->head;
    while (current != NULL) {
        printf("%d <-> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}

void dlist_print_backward(DoublyLinkedList *list) {
    DNode *current = list->tail;
    while (current != NULL) {
        printf("%d <-> ", current->data);
        current = current->prev;
    }
    printf("NULL\n");
}

void dlist_free(DoublyLinkedList *list) {
    DNode *current = list->head;
    while (current != NULL) {
        DNode *temp = current;
        current = current->next;
        free(temp);
    }
    free(list);
}
```

#### Week 10: Stacks and Queues

**Tutorial 15: Stack Implementation**
```c
// stack.c - Array and linked list implementations
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_SIZE 100

// Array-based stack
typedef struct {
    int data[MAX_SIZE];
    int top;
} ArrayStack;

// Linked list-based stack
typedef struct StackNode {
    int data;
    struct StackNode *next;
} StackNode;

typedef struct {
    StackNode *top;
    int size;
} LinkedStack;

// Array stack functions
ArrayStack* astack_create(void);
bool astack_is_empty(ArrayStack *stack);
bool astack_is_full(ArrayStack *stack);
void astack_push(ArrayStack *stack, int data);
int astack_pop(ArrayStack *stack);
int astack_peek(ArrayStack *stack);

// Linked stack functions
LinkedStack* lstack_create(void);
void lstack_push(LinkedStack *stack, int data);
int lstack_pop(LinkedStack *stack);
int lstack_peek(LinkedStack *stack);
bool lstack_is_empty(LinkedStack *stack);
void lstack_free(LinkedStack *stack);

// Applications
bool is_balanced_parentheses(const char *expr);
void reverse_string_with_stack(char *str);
int evaluate_postfix(const char *expr);

int main(void) {
    // Test array stack
    printf("=== Array-Based Stack ===\n");
    ArrayStack *as = astack_create();
    
    astack_push(as, 10);
    astack_push(as, 20);
    astack_push(as, 30);
    
    printf("Popped: %d\n", astack_pop(as));
    printf("Peek: %d\n", astack_peek(as));
    
    // Test linked stack
    printf("\n=== Linked List-Based Stack ===\n");
    LinkedStack *ls = lstack_create();
    
    lstack_push(ls, 100);
    lstack_push(ls, 200);
    lstack_push(ls, 300);
    
    printf("Popped: %d\n", lstack_pop(ls));
    printf("Peek: %d\n", lstack_peek(ls));
    
    // Applications
    printf("\n=== Stack Applications ===\n");
    
    const char *expressions[] = {
        "(())",
        "({[]})",
        "((}",
        "{[()]}"
    };
    
    for (int i = 0; i < 4; i++) {
        printf("%s: %s\n", expressions[i],
               is_balanced_parentheses(expressions[i]) ? "Balanced" : "Not Balanced");
    }
    
    char str[] = "Hello";
    printf("\nOriginal: %s\n", str);
    reverse_string_with_stack(str);
    printf("Reversed: %s\n", str);
    
    const char *postfix = "23*5+";
    printf("\nPostfix %s = %d\n", postfix, evaluate_postfix(postfix));
    
    free(as);
    lstack_free(ls);
    return 0;
}

// Array Stack Implementation
ArrayStack* astack_create(void) {
    ArrayStack *stack = (ArrayStack*)malloc(sizeof(ArrayStack));
    if (stack) stack->top = -1;
    return stack;
}

bool astack_is_empty(ArrayStack *stack) {
    return stack->top == -1;
}

bool astack_is_full(ArrayStack *stack) {
    return stack->top == MAX_SIZE - 1;
}

void astack_push(ArrayStack *stack, int data) {
    if (!astack_is_full(stack)) {
        stack->data[++stack->top] = data;
    }
}

int astack_pop(ArrayStack *stack) {
    if (!astack_is_empty(stack)) {
        return stack->data[stack->top--];
    }
    return -1;
}

int astack_peek(ArrayStack *stack) {
    if (!astack_is_empty(stack)) {
        return stack->data[stack->top];
    }
    return -1;
}

// Linked Stack Implementation
LinkedStack* lstack_create(void) {
    LinkedStack *stack = (LinkedStack*)malloc(sizeof(LinkedStack));
    if (stack) {
        stack->top = NULL;
        stack->size = 0;
    }
    return stack;
}

void lstack_push(LinkedStack *stack, int data) {
    StackNode *new_node = (StackNode*)malloc(sizeof(StackNode));
    if (!new_node) return;
    
    new_node->data = data;
    new_node->next = stack->top;
    stack->top = new_node;
    stack->size++;
}

int lstack_pop(LinkedStack *stack) {
    if (lstack_is_empty(stack)) return -1;
    
    StackNode *temp = stack->top;
    int data = temp->data;
    stack->top = stack->top->next;
    free(temp);
    stack->size--;
    return data;
}

int lstack_peek(LinkedStack *stack) {
    if (lstack_is_empty(stack)) return -1;
    return stack->top->data;
}

bool lstack_is_empty(LinkedStack *stack) {
    return stack->top == NULL;
}

void lstack_free(LinkedStack *stack) {
    while (!lstack_is_empty(stack)) {
        lstack_pop(stack);
    }
    free(stack);
}

// Applications
bool is_balanced_parentheses(const char *expr) {
    ArrayStack *stack = astack_create();
    
    for (int i = 0; expr[i] != '\0'; i++) {
        char ch = expr[i];
        
        if (ch == '(' || ch == '[' || ch == '{') {
            astack_push(stack, ch);
        } else if (ch == ')' || ch == ']' || ch == '}') {
            if (astack_is_empty(stack)) {
                free(stack);
                return false;
            }
            
            char top = astack_pop(stack);
            if ((ch == ')' && top != '(') ||
                (ch == ']' && top != '[') ||
                (ch == '}' && top != '{')) {
                free(stack);
                return false;
            }
        }
    }
    
    bool balanced = astack_is_empty(stack);
    free(stack);
    return balanced;
}

void reverse_string_with_stack(char *str) {
    LinkedStack *stack = lstack_create();
    
    int i = 0;
    while (str[i] != '\0') {
        lstack_push(stack, str[i]);
        i++;
    }
    
    i = 0;
    while (!lstack_is_empty(stack)) {
        str[i++] = lstack_pop(stack);
    }
    
    lstack_free(stack);
}

int evaluate_postfix(const char *expr) {
    ArrayStack *stack = astack_create();
    
    for (int i = 0; expr[i] != '\0'; i++) {
        char ch = expr[i];
        
        if (ch >= '0' && ch <= '9') {
            astack_push(stack, ch - '0');
        } else {
            int b = astack_pop(stack);
            int a = astack_pop(stack);
            
            switch (ch) {
                case '+': astack_push(stack, a + b); break;
                case '-': astack_push(stack, a - b); break;
                case '*': astack_push(stack, a * b); break;
                case '/': astack_push(stack, a / b); break;
            }
        }
    }
    
    int result = astack_pop(stack);
    free(stack);
    return result;
}
```

**Tutorial 16: Queue Implementation**
```c
// queue.c - Circular queue and linked queue
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_SIZE 100

// Circular Queue (Array-based)
typedef struct {
    int data[MAX_SIZE];
    int front;
    int rear;
    int size;
} CircularQueue;

// Linked Queue
typedef struct QueueNode {
    int data;
    struct QueueNode *next;
} QueueNode;

typedef struct {
    QueueNode *front;
    QueueNode *rear;
    int size;
} LinkedQueue;

// Circular Queue functions
CircularQueue* cqueue_create(void);
bool cqueue_is_empty(CircularQueue *queue);
bool cqueue_is_full(CircularQueue *queue);
void cqueue_enqueue(CircularQueue *queue, int data);
int cqueue_dequeue(CircularQueue *queue);
int cqueue_peek(CircularQueue *queue);

// Linked Queue functions
LinkedQueue* lqueue_create(void);
void lqueue_enqueue(LinkedQueue *queue, int data);
int lqueue_dequeue(LinkedQueue *queue);
bool lqueue_is_empty(LinkedQueue *queue);
void lqueue_free(LinkedQueue *queue);

int main(void) {
    printf("=== Circular Queue ===\n");
    CircularQueue *cq = cqueue_create();
    
    cqueue_enqueue(cq, 10);
    cqueue_enqueue(cq, 20);
    cqueue_enqueue(cq, 30);
    
    printf("Dequeued: %d\n", cqueue_dequeue(cq));
    printf("Front: %d\n", cqueue_peek(cq));
    
    cqueue_enqueue(cq, 40);
    cqueue_enqueue(cq, 50);
    
    while (!cqueue_is_empty(cq)) {
        printf("%d ", cqueue_dequeue(cq));
    }
    printf("\n");
    
    printf("\n=== Linked Queue ===\n");
    LinkedQueue *lq = lqueue_create();
    
    lqueue_enqueue(lq, 100);
    lqueue_enqueue(lq, 200);
    lqueue_enqueue(lq, 300);
    
    printf("Dequeued: %d\n", lqueue_dequeue(lq));
    
    while (!lqueue_is_empty(lq)) {
        printf("%d ", lqueue_dequeue(lq));
    }
    printf("\n");
    
    free(cq);
    lqueue_free(lq);
    return 0;
}

// Circular Queue Implementation
CircularQueue* cqueue_create(void) {
    CircularQueue *queue = (CircularQueue*)malloc(sizeof(CircularQueue));
    if (queue) {
        queue->front = 0;
        queue->rear = -1;
        queue->size = 0;
    }
    return queue;
}

bool cqueue_is_empty(CircularQueue *queue) {
    return queue->size == 0;
}

bool cqueue_is_full(CircularQueue *queue) {
    return queue->size == MAX_SIZE;
}

void cqueue_enqueue(CircularQueue *queue, int data) {
    if (cqueue_is_full(queue)) return;
    
    queue->rear = (queue->rear + 1) % MAX_SIZE;
    queue->data[queue->rear] = data;
    queue->size++;
}

int cqueue_dequeue(CircularQueue *queue) {
    if (cqueue_is_empty(queue)) return -1;
    
    int data = queue->data[queue->front];
    queue->front = (queue->front + 1) % MAX_SIZE;
    queue->size--;
    return data;
}

int cqueue_peek(CircularQueue *queue) {
    if (cqueue_is_empty(queue)) return -1;
    return queue->data[queue->front];
}

// Linked Queue Implementation
LinkedQueue* lqueue_create(void) {
    LinkedQueue *queue = (LinkedQueue*)malloc(sizeof(LinkedQueue));
    if (queue) {
        queue->front = NULL;
        queue->rear = NULL;
        queue->size = 0;
    }
    return queue;
}

void lqueue_enqueue(LinkedQueue *queue, int data) {
    QueueNode *new_node = (QueueNode*)malloc(sizeof(QueueNode));
    if (!new_node) return;
    
    new_node->data = data;
    new_node->next = NULL;
    
    if (queue->rear == NULL) {
        queue->front = queue->rear = new_node;
    } else {
        queue->rear->next = new_node;
        queue->rear = new_node;
    }
    queue->size++;
}

int lqueue_dequeue(LinkedQueue *queue) {
    if (lqueue_is_empty(queue)) return -1;
    
    QueueNode *temp = queue->front;
    int data = temp->data;
    queue->front = queue->front->next;
    
    if (queue->front == NULL) {
        queue->rear = NULL;
    }
    
    free(temp);
    queue->size--;
    return data;
}

bool lqueue_is_empty(LinkedQueue *queue) {
    return queue->front == NULL;
}

void lqueue_free(LinkedQueue *queue) {
    while (!lqueue_is_empty(queue)) {
        lqueue_dequeue(queue);
    }
    free(queue);
}
```

#### Week 11: Trees

**Tutorial 17: Binary Tree & BST**
```c
// binary_tree.c - Binary tree and BST operations
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

typedef struct TreeNode {
    int data;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

// Tree creation
TreeNode* create_node(int data);

// Tree traversals
void inorder(TreeNode *root);
void preorder(TreeNode *root);
void postorder(TreeNode *root);
void level_order(TreeNode *root);

// Tree properties
int tree_height(TreeNode *root);
int count_nodes(TreeNode *root);
int count_leaves(TreeNode *root);
int find_max(TreeNode *root);
int find_min(TreeNode *root);

// BST operations
TreeNode* bst_insert(TreeNode *root, int data);
TreeNode* bst_search(TreeNode *root, int data);
TreeNode* bst_delete(TreeNode *root, int data);
TreeNode* bst_find_min(TreeNode *root);
TreeNode* bst_find_max(TreeNode *root);
bool is_bst(TreeNode *root);

// Helper functions
bool is_bst_util(TreeNode *root, int min, int max);
void free_tree(TreeNode *root);

int main(void) {
    printf("=== Binary Search Tree Operations ===\n");
    
    TreeNode *root = NULL;
    
    // Insert nodes
    root = bst_insert(root, 50);
    root = bst_insert(root, 30);
    root = bst_insert(root, 70);
    root = bst_insert(root, 20);
    root = bst_insert(root, 40);
    root = bst_insert(root, 60);
    root = bst_insert(root, 80);
    
    printf("\nInorder traversal: ");
    inorder(root);
    printf("\n");
    
    printf("Preorder traversal: ");
    preorder(root);
    printf("\n");
    
    printf("Postorder traversal: ");
    postorder(root);
    printf("\n");
    
    printf("Level order traversal: ");
    level_order(root);
    printf("\n");
    
    printf("\nTree height: %d\n", tree_height(root));
    printf("Total nodes: %d\n", count_nodes(root));
    printf("Leaf nodes: %d\n", count_leaves(root));
    
    int key = 40;
    TreeNode *found = bst_search(root, key);
    printf("\nSearch %d: %s\n", key, found ? "Found" : "Not found");
    
    printf("Is BST: %s\n", is_bst(root) ? "Yes" : "No");
    
    printf("\nDeleting 30...\n");
    root = bst_delete(root, 30);
    printf("Inorder after deletion: ");
    inorder(root);
    printf("\n");
    
    free_tree(root);
    return 0;
}

TreeNode* create_node(int data) {
    TreeNode *node = (TreeNode*)malloc(sizeof(TreeNode));
    if (node) {
        node->data = data;
        node->left = NULL;
        node->right = NULL;
    }
    return node;
}

// Traversals
void inorder(TreeNode *root) {
    if (root != NULL) {
        inorder(root->left);
        printf("%d ", root->data);
        inorder(root->right);
    }
}

void preorder(TreeNode *root) {
    if (root != NULL) {
        printf("%d ", root->data);
        preorder(root->left);
        preorder(root->right);
    }
}

void postorder(TreeNode *root) {
    if (root != NULL) {
        postorder(root->left);
        postorder(root->right);
        printf("%d ", root->data);
    }
}

void level_order(TreeNode *root) {
    if (root == NULL) return;
    
    // Simple queue using array
    TreeNode *queue[100];
    int front = 0, rear = 0;
    
    queue[rear++] = root;
    
    while (front < rear) {
        TreeNode *node = queue[front++];
        printf("%d ", node->data);
        
        if (node->left) queue[rear++] = node->left;
        if (node->right) queue[rear++] = node->right;
    }
}

// Tree properties
int tree_height(TreeNode *root) {
    if (root == NULL) return 0;
    
    int left_height = tree_height(root->left);
    int right_height = tree_height(root->right);
    
    return 1 + (left_height > right_height ? left_height : right_height);
}

int count_nodes(TreeNode *root) {
    if (root == NULL) return 0;
    return 1 + count_nodes(root->left) + count_nodes(root->right);
}

int count_leaves(TreeNode *root) {
    if (root == NULL) return 0;
    if (root->left == NULL && root->right == NULL) return 1;
    return count_leaves(root->left) + count_leaves(root->right);
}

int find_max(TreeNode *root) {
    if (root == NULL) return INT_MIN;
    
    int max = root->data;
    int left_max = find_max(root->left);
    int right_max = find_max(root->right);
    
    if (left_max > max) max = left_max;
    if (right_max > max) max = right_max;
    
    return max;
}

// BST operations
TreeNode* bst_insert(TreeNode *root, int data) {
    if (root == NULL) {
        return create_node(data);
    }
    
    if (data < root->data) {
        root->left = bst_insert(root->left, data);
    } else if (data > root->data) {
        root->right = bst_insert(root->right, data);
    }
    
    return root;
}

TreeNode* bst_search(TreeNode *root, int data) {
    if (root == NULL || root->data == data) {
        return root;
    }
    
    if (data < root->data) {
        return bst_search(root->left, data);
    } else {
        return bst_search(root->right, data);
    }
}

TreeNode* bst_find_min(TreeNode *root) {
    if (root == NULL) return NULL;
    while (root->left != NULL) {
        root = root->left;
    }
    return root;
}

TreeNode* bst_find_max(TreeNode *root) {
    if (root == NULL) return NULL;
    while (root->right != NULL) {
        root = root->right;
    }
    return root;
}

TreeNode* bst_delete(TreeNode *root, int data) {
    if (root == NULL) return NULL;
    
    if (data < root->data) {
        root->left = bst_delete(root->left, data);
    } else if (data > root->data) {
        root->right = bst_delete(root->right, data);
    } else {
        // Node found
        if (root->left == NULL) {
            TreeNode *temp = root->right;
            free(root);
            return temp;
        } else if (root->right == NULL) {
            TreeNode *temp = root->left;
            free(root);
            return temp;
        }
        
        // Node with two children
        TreeNode *temp = bst_find_min(root->right);
        root->data = temp->data;
        root->right = bst_delete(root->right, temp->data);
    }
    
    return root;
}

bool is_bst_util(TreeNode *root, int min, int max) {
    if (root == NULL) return true;
    
    if (root->data < min || root->data > max) {
        return false;
    }
    
    return is_bst_util(root->left, min, root->data - 1) &&
           is_bst_util(root->right, root->data + 1, max);
}

bool is_bst(TreeNode *root) {
    return is_bst_util(root, INT_MIN, INT_MAX);
}

void free_tree(TreeNode *root) {
    if (root != NULL) {
        free_tree(root->left);
        free_tree(root->right);
        free(root);
    }
}
```

#### Week 12: Hash Tables & Graphs

**Tutorial 18: Hash Table**
```c
// hash_table.c - Chaining implementation
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TABLE_SIZE 10

typedef struct Entry {
    char *key;
    int value;
    struct Entry *next;
} Entry;

typedef struct {
    Entry *table[TABLE_SIZE];
    int size;
} HashTable;

// Hash table operations
HashTable* ht_create(void);
unsigned int hash(const char *key);
void ht_insert(HashTable *ht, const char *key, int value);
int ht_search(HashTable *ht, const char *key, int *value);
void ht_delete(HashTable *ht, const char *key);
void ht_print(HashTable *ht);
void ht_free(HashTable *ht);

int main(void) {
    printf("=== Hash Table Operations ===\n");
    
    HashTable *ht = ht_create();
    
    // Insert key-value pairs
    ht_insert(ht, "apple", 100);
    ht_insert(ht, "banana", 200);
    ht_insert(ht, "orange", 300);
    ht_insert(ht, "grape", 400);
    ht_insert(ht, "mango", 500);
    
    printf("\nHash Table contents:\n");
    ht_print(ht);
    
    // Search
    int value;
    if (ht_search(ht, "banana", &value)) {
        printf("\nFound banana: %d\n", value);
    }
    
    // Delete
    printf("\nDeleting 'orange'...\n");
    ht_delete(ht, "orange");
    ht_print(ht);
    
    ht_free(ht);
    return 0;
}

HashTable* ht_create(void) {
    HashTable *ht = (HashTable*)malloc(sizeof(HashTable));
    if (ht) {
        for (int i = 0; i < TABLE_SIZE; i++) {
            ht->table[i] = NULL;
        }
        ht->size = 0;
    }
    return ht;
}

unsigned int hash(const char *key) {
    unsigned int hash_value = 0;
    while (*key) {
        hash_value = (hash_value << 5) + *key++;
    }
    return hash_value % TABLE_SIZE;
}

void ht_insert(HashTable *ht, const char *key, int value) {
    unsigned int index = hash(key);
    
    // Check if key exists
    Entry *current = ht->table[index];
    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            current->value = value;  // Update existing
            return;
        }
        current = current->next;
    }
    
    // Insert new entry
    Entry *new_entry = (Entry*)malloc(sizeof(Entry));
    if (!new_entry) return;
    
    new_entry->key = strdup(key);
    new_entry->value = value;
    new_entry->next = ht->table[index];
    ht->table[index] = new_entry;
    ht->size++;
}

int ht_search(HashTable *ht, const char *key, int *value) {
    unsigned int index = hash(key);
    Entry *current = ht->table[index];
    
    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            *value = current->value;
            return 1;
        }
        current = current->next;
    }
    return 0;
}

void ht_delete(HashTable *ht, const char *key) {
    unsigned int index = hash(key);
    Entry *current = ht->table[index];
    Entry *prev = NULL;
    
    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            if (prev == NULL) {
                ht->table[index] = current->next;
            } else {
                prev->next = current->next;
            }
            free(current->key);
            free(current);
            ht->size--;
            return;
        }
        prev = current;
        current = current->next;
    }
}

void ht_print(HashTable *ht) {
    for (int i = 0; i < TABLE_SIZE; i++) {
        if (ht->table[i] != NULL) {
            printf("[%d]: ", i);
            Entry *current = ht->table[i];
            while (current != NULL) {
                printf("(%s: %d) ", current->key, current->value);
                current = current->next;
            }
            printf("\n");
        }
    }
}

void ht_free(HashTable *ht) {
    for (int i = 0; i < TABLE_SIZE; i++) {
        Entry *current = ht->table[i];
        while (current != NULL) {
            Entry *temp = current;
            current = current->next;
            free(temp->key);
            free(temp);
        }
    }
    free(ht);
}
```

**Phase 3 Milestones:**
- ✅ Implement all linked list variants
- ✅ Create stack and queue (array & linked)
- ✅ Build binary trees and BST
- ✅ Implement hash table
- ✅ Understand graph representations
- ✅ Solve 50+ data structure problems

### Phase 4: Algorithms (Weeks 13-16)

**Goal:** Master sorting, searching, and algorithmic problem-solving

**Week 13-14:** Implement all sorting algorithms (bubble, selection, insertion, merge, quick, heap, counting)
**Week 15:** Binary search, recursion, backtracking
**Week 16:** Dynamic programming (Fibonacci, LCS, Knapsack, Coin Change)

**Phase 4 Milestones:**
- ✅ Implement all major sorting algorithms  
- ✅ Understand time/space complexity
- ✅ Master binary search variants
- ✅ Solve 100+ LeetCode problems
- ✅ Complete dynamic programming problems

### Phase 5: System Programming (Weeks 17-20)

**Goal:** Master processes, threads, and IPC

**Week 17:** Process creation (fork, exec), wait, zombies
**Week 18:** POSIX threads, mutexes, semaphores
**Week 19:** IPC (pipes, shared memory, message queues, signals)
**Week 20:** Network programming (sockets, TCP/UDP)

**Phase 5 Milestones:**
- ✅ Create and manage processes
- ✅ Implement multi-threaded programs
- ✅ Use pipes, shared memory, message queues
- ✅ Handle signals properly
- ✅ Build client-server socket programs

### Phase 6: Linux Internals (Weeks 21-24)

**Goal:** Understand kernel internals and system-level programming

**Week 21:** System calls (open, read, write, mmap, brk)
**Week 22:** Virtual memory, page tables, memory zones
**Week 23:** Process management (task_struct, scheduler, context switching)
**Week 24:** File systems (VFS, inodes, dentries)

**Phase 6 Milestones:**
- ✅ Understand system call mechanism
- ✅ Study memory management subsystem
- ✅ Read scheduler source code (CFS)
- ✅ Understand VFS layer
- ✅ Trace process creation (fork/exec)

### Phase 7: Open Source (Weeks 25-28)

**Goal:** Analyze production C codebases

**Week 25:** Redis - SDS, Objects, Event Loop
**Week 26:** SQLite - B-tree, VDBE, Query planner
**Week 27:** Git - Objects (blob, tree, commit), Pack files
**Week 28:** Nginx - Event-driven architecture, HTTP parser

**Phase 7 Milestones:**
- ✅ Read and understand Redis source
- ✅ Study SQLite architecture
- ✅ Analyze Git internals
- ✅ Contribute to open source project
- ✅ Write technical blog posts

### Phase 8: Mastery (Ongoing)

**Goal:** Become a C expert and systems programmer

**Advanced Topics:**
- Lock-free data structures (atomics, CAS)
- Memory pools and custom allocators
- Compiler optimizations (-O2, -O3, inline assembly)
- Performance profiling (perf, gprof, cachegrind)
- Debugging (GDB, Valgrind, AddressSanitizer)

**Phase 8 Milestones:**
- ✅ Build complete projects from scratch
- ✅ Optimize for performance
- ✅ Write production-quality code
- ✅ Mentor others
- ✅ Never stop learning!

### Recommended Books
1. "The C Programming Language" - Kernighan & Ritchie
2. "C Programming: A Modern Approach" - K.N. King
3. "Expert C Programming" - Peter van der Linden
4. "Computer Systems: A Programmer's Perspective" - Bryant & O'Hallaron
5. "Linux System Programming" - Robert Love
6. "Understanding the Linux Kernel" - Bovet & Cesati
7. "Advanced Programming in the UNIX Environment" - Stevens

### Online Resources
- Linux kernel source: https://kernel.org
- Redis source: https://github.com/redis/redis
- SQLite source: https://www.sqlite.org/src/doc/trunk/README.md
- Git source: https://github.com/git/git
- Practice: LeetCode, HackerRank, CodeForces

### Project Ideas
1. Custom shell (like bash)
2. Text editor (like vim basics)
3. Memory allocator (like malloc)
4. Database engine (simple SQL)
5. Web server (HTTP/1.1)
6. File compression utility
7. Network packet sniffer
8. Process monitor (like htop)
9. File system implementation
10. Debugger (basic GDB clone)

---

## Conclusion

Mastering C requires dedication, practice, and deep understanding of computer systems. This roadmap covers everything from basics to advanced topics including DSA, Linux internals, and open-source analysis.

**Key Principles:**
1. Practice daily - code every day
2. Read source code - learn from masters
3. Understand concepts deeply - don't just memorize
4. Build real projects - apply your knowledge
5. Debug extensively - learn from errors
6. Optimize wisely - measure before optimizing
7. Stay curious - keep exploring

**Remember:** C is not just a language, it's a way of thinking about computers. Understanding C makes you a better programmer in any language.

Good luck on your journey to becoming a C ninja! 🥷

---

*This roadmap is comprehensive but not exhaustive. Keep learning, keep coding, and never stop exploring!*
