# The Complete C Programming Master Roadmap
## From Beginner to Linux Kernel Developer

---

## Table of Contents
1. [Foundation & Basics](#1-foundation--basics)
2. [Advanced C Concepts](#2-advanced-c-concepts)
3. [Memory Management Mastery](#3-memory-management-mastery)
4. [Data Structures & Algorithms](#4-data-structures--algorithms)
5. [Systems Programming](#5-systems-programming)
6. [Linux Internals & Kernel Programming](#6-linux-internals--kernel-programming)
7. [Advanced Topics & Optimization](#7-advanced-topics--optimization)
8. [Real-world Projects & Open Source](#8-real-world-projects--open-source)

---

## 1. Foundation & Basics

### 1.1 Understanding the C Compilation Process

```
Source Code (.c) → Preprocessor → Compiler → Assembler → Linker → Executable

┌─────────────┐     ┌──────────────┐     ┌──────────┐     ┌─────────┐
│  hello.c    │────→│ Preprocessor │────→│ Compiler │────→│Assembler│
│ (C Source)  │     │  (cpp)       │     │  (cc)    │     │  (as)   │
└─────────────┘     └──────────────┘     └──────────┘     └─────────┘
                           ↓                    ↓               ↓
                    hello.i (expanded)    hello.s (asm)   hello.o (object)
                                                                ↓
                                                          ┌─────────┐
                                                          │ Linker  │
                                                          │  (ld)   │
                                                          └─────────┘
                                                                ↓
                                                           a.out/exe
```

**Example:**
```c
// hello.c
#include <stdio.h>
#define MAX 100

int main() {
    printf("Hello, World!\n");
    return 0;
}

// Compilation steps:
// 1. gcc -E hello.c -o hello.i    (Preprocessing)
// 2. gcc -S hello.i -o hello.s    (Compilation to assembly)
// 3. gcc -c hello.s -o hello.o    (Assembly to object code)
// 4. gcc hello.o -o hello         (Linking)
// OR: gcc hello.c -o hello        (All at once)
```

### 1.2 Data Types & Memory Layout

```
┌─────────────────────────────────────────────────┐
│            Memory Layout (32-bit)               │
├─────────────┬──────────┬─────────┬──────────────┤
│   Type      │   Size   │  Range  │  Format      │
├─────────────┼──────────┼─────────┼──────────────┤
│   char      │  1 byte  │ -128 to │   %c, %d     │
│             │          │  127    │              │
├─────────────┼──────────┼─────────┼──────────────┤
│   short     │  2 bytes │ -32768  │   %hd        │
│             │          │  32767  │              │
├─────────────┼──────────┼─────────┼──────────────┤
│   int       │  4 bytes │ -2^31   │   %d         │
│             │          │  2^31-1 │              │
├─────────────┼──────────┼─────────┼──────────────┤
│   long      │ 4/8 bytes│ varies  │   %ld        │
├─────────────┼──────────┼─────────┼──────────────┤
│   float     │  4 bytes │ 3.4E-38 │   %f         │
│             │          │ 3.4E+38 │              │
├─────────────┼──────────┼─────────┼──────────────┤
│   double    │  8 bytes │ 1.7E-308│   %lf        │
│             │          │ 1.7E+308│              │
└─────────────┴──────────┴─────────┴──────────────┘
```

**Example:**
```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main() {
    printf("Size of char: %zu bytes, Range: %d to %d\n", 
           sizeof(char), CHAR_MIN, CHAR_MAX);
    printf("Size of int: %zu bytes, Range: %d to %d\n", 
           sizeof(int), INT_MIN, INT_MAX);
    printf("Size of float: %zu bytes, Range: %E to %E\n", 
           sizeof(float), FLT_MIN, FLT_MAX);
    
    // Understanding signed vs unsigned
    unsigned int u = 4294967295;  // Max unsigned int
    int s = -1;
    printf("Unsigned: %u, Signed: %d\n", u, s);
    printf("Are they equal in memory? %s\n", 
           (*(unsigned int*)&s == u) ? "Yes" : "No");
    
    return 0;
}
```

### 1.3 Operators & Expression Evaluation

**Operator Precedence (High to Low):**
```
1.  () [] -> .                    (Postfix)
2.  ! ~ ++ -- + - * &             (Unary)
3.  * / %                         (Multiplicative)
4.  + -                           (Additive)
5.  << >>                         (Shift)
6.  < <= > >=                     (Relational)
7.  == !=                         (Equality)
8.  &                             (Bitwise AND)
9.  ^                             (Bitwise XOR)
10. |                             (Bitwise OR)
11. &&                            (Logical AND)
12. ||                            (Logical OR)
13. ?:                            (Ternary)
14. = += -= *= /= %= &= ^= |=     (Assignment)
15. ,                             (Comma)
```

**Example:**
```c
#include <stdio.h>

int main() {
    int a = 5, b = 10, c = 15;
    
    // Expression evaluation
    int result = a + b * c;  // 5 + (10 * 15) = 155, not (5 + 10) * 15
    printf("Result: %d\n", result);
    
    // Bitwise operations
    printf("5 & 3 = %d\n", 5 & 3);    // 0101 & 0011 = 0001 = 1
    printf("5 | 3 = %d\n", 5 | 3);    // 0101 | 0011 = 0111 = 7
    printf("5 ^ 3 = %d\n", 5 ^ 3);    // 0101 ^ 0011 = 0110 = 6
    printf("~5 = %d\n", ~5);          // ~0101 = 1010 (in 2's complement)
    printf("5 << 1 = %d\n", 5 << 1);  // 0101 << 1 = 1010 = 10
    printf("5 >> 1 = %d\n", 5 >> 1);  // 0101 >> 1 = 0010 = 2
    
    // Short-circuit evaluation
    int x = 0;
    if (x != 0 && 10/x > 1) {  // Second condition never evaluated
        printf("This won't print\n");
    }
    
    return 0;
}
```

### 1.4 Control Flow

**Switch Statement Internals:**
```
Switch is often implemented as a jump table for dense cases:

switch(x) {
    case 0:  // code_0
    case 1:  // code_1
    case 2:  // code_2
}

Jump Table:
┌───────┬─────────┐
│ Value │ Address │
├───────┼─────────┤
│   0   │ code_0  │
│   1   │ code_1  │
│   2   │ code_2  │
└───────┴─────────┘

For sparse cases, compiler may use if-else chain.
```

**Example:**
```c
#include <stdio.h>

// Demonstrating fall-through and jump table optimization
int main() {
    int day = 3;
    
    // Jump table efficient for dense cases
    switch(day) {
        case 1:
        case 2:
        case 3:
        case 4:
        case 5:
            printf("Weekday\n");
            break;
        case 6:
        case 7:
            printf("Weekend\n");
            break;
        default:
            printf("Invalid day\n");
    }
    
    // Loop constructs
    for (int i = 0; i < 5; i++) {
        if (i == 2) continue;  // Skip iteration
        if (i == 4) break;     // Exit loop
        printf("%d ", i);       // Prints: 0 1 3
    }
    printf("\n");
    
    // Do-while ensures at least one execution
    int count = 0;
    do {
        printf("Executed at least once\n");
    } while (count > 0);  // Condition checked after execution
    
    return 0;
}
```

### 1.5 Functions & Stack Frames

```
Stack Frame Layout:

High Memory
    │
    ├─────────────────┐
    │  Arguments      │  (Parameters passed to function)
    ├─────────────────┤
    │  Return Address │  (Where to return after function)
    ├─────────────────┤
    │  Old Frame Ptr  │  (Previous function's base pointer)
    ├─────────────────┤ ← Frame Pointer (FP/EBP)
    │  Local Vars     │
    ├─────────────────┤
    │  Saved Regs     │
    ├─────────────────┤ ← Stack Pointer (SP/ESP)
    │        ↓        │  (Stack grows downward)
Low Memory
```

**Example:**
```c
#include <stdio.h>

// Function with various parameter types
void demonstrate_stack(int a, int b, int arr[], int *ptr) {
    int local = 10;
    
    printf("Stack Frame Analysis:\n");
    printf("Address of parameter a: %p\n", (void*)&a);
    printf("Address of parameter b: %p\n", (void*)&b);
    printf("Address of local var:   %p\n", (void*)&local);
    printf("Array pointer value:    %p\n", (void*)arr);
    printf("Pointer ptr value:      %p\n", (void*)ptr);
    
    // Notice: local variables have lower addresses than parameters
    // This shows stack grows downward in memory
}

// Recursion and stack depth
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);  // Each call creates new stack frame
}

// Tail recursion (optimizable by compiler)
int factorial_tail(int n, int accumulator) {
    if (n <= 1) return accumulator;
    return factorial_tail(n - 1, n * accumulator);
}

int main() {
    int x = 5, y = 10;
    int nums[] = {1, 2, 3};
    
    demonstrate_stack(x, y, nums, &x);
    
    printf("\nFactorial of 5: %d\n", factorial(5));
    printf("Factorial of 5 (tail): %d\n", factorial_tail(5, 1));
    
    return 0;
}
```

---

## 2. Advanced C Concepts

### 2.1 Pointers - The Heart of C

**Pointer Memory Model:**
```
Variable:  int x = 10;
Memory:    [Address: 0x1000] [Value: 10]

Pointer:   int *ptr = &x;
Memory:    [Address: 0x2000] [Value: 0x1000]
                                      │
                                      └──→ [Address: 0x1000] [Value: 10]

Pointer to Pointer: int **pptr = &ptr;
Memory:    [Address: 0x3000] [Value: 0x2000]
                                      │
                                      └──→ [Address: 0x2000] [Value: 0x1000]
                                                                     │
                                                                     └──→ [Value: 10]
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

void pointer_arithmetic() {
    int arr[] = {10, 20, 30, 40, 50};
    int *ptr = arr;
    
    printf("Array elements using pointer arithmetic:\n");
    for (int i = 0; i < 5; i++) {
        printf("*(ptr + %d) = %d, Address: %p\n", i, *(ptr + i), (void*)(ptr + i));
    }
    
    // Important: ptr++ moves by sizeof(int) bytes, not 1 byte
    printf("\nPointer increment: ptr++ moves %zu bytes\n", sizeof(int));
}

void multi_dimensional_pointers() {
    int matrix[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };
    
    // Three ways to access elements:
    printf("matrix[1][2] = %d\n", matrix[1][2]);
    printf("*(*(matrix + 1) + 2) = %d\n", *(*(matrix + 1) + 2));
    printf("*(&matrix[0][0] + 1*4 + 2) = %d\n", *(&matrix[0][0] + 1*4 + 2));
    
    // Pointer to array
    int (*ptr_to_row)[4] = matrix;  // Points to array of 4 ints
    printf("Using pointer to array: %d\n", ptr_to_row[1][2]);
}

void function_pointers() {
    int (*operation)(int, int);  // Pointer to function
    
    int add(int a, int b) { return a + b; }
    int multiply(int a, int b) { return a * b; }
    
    operation = add;
    printf("5 + 3 = %d\n", operation(5, 3));
    
    operation = multiply;
    printf("5 * 3 = %d\n", operation(5, 3));
    
    // Array of function pointers
    int (*ops[])(int, int) = {add, multiply};
    printf("Using array: %d\n", ops[0](10, 20));
}

int main() {
    int a = 10, b = 20;
    printf("Before swap: a=%d, b=%d\n", a, b);
    swap(&a, &b);
    printf("After swap: a=%d, b=%d\n", a, b);
    
    pointer_arithmetic();
    multi_dimensional_pointers();
    function_pointers();
    
    return 0;
}
```

### 2.2 Arrays and Strings

**Array Memory Layout:**
```
int arr[5] = {10, 20, 30, 40, 50};

Memory Layout:
┌──────┬──────┬──────┬──────┬──────┐
│  10  │  20  │  30  │  40  │  50  │
└──────┴──────┴──────┴──────┴──────┘
↑
arr (base address)

arr[i] ≡ *(arr + i) ≡ *(i + arr) ≡ i[arr]  // All equivalent!
```

**Example:**
```c
#include <stdio.h>
#include <string.h>

void array_decay() {
    int arr[5] = {1, 2, 3, 4, 5};
    
    printf("sizeof(arr) in function scope: %zu\n", sizeof(arr));  // 20 bytes
    
    void print_array(int a[]) {
        printf("sizeof(a) in function: %zu\n", sizeof(a));  // 8 bytes (pointer!)
        // Array decays to pointer when passed to function
    }
    
    print_array(arr);
}

void string_manipulation() {
    // String literals are stored in read-only memory
    char *str1 = "Hello";  // Pointer to read-only memory
    // str1[0] = 'h';  // SEGMENTATION FAULT!
    
    char str2[] = "Hello";  // Array in stack (modifiable)
    str2[0] = 'h';  // OK
    printf("Modified string: %s\n", str2);
    
    // String operations
    char dest[50];
    strcpy(dest, "Hello");
    strcat(dest, " World");
    printf("Concatenated: %s, Length: %zu\n", dest, strlen(dest));
    
    // Manual string copy (understanding implementation)
    char *my_strcpy(char *dest, const char *src) {
        char *ret = dest;
        while ((*dest++ = *src++));  // Copy until null terminator
        return ret;
    }
    
    char buffer[20];
    my_strcpy(buffer, "Test");
    printf("Copied: %s\n", buffer);
}

void multidimensional_arrays() {
    int matrix[2][3] = {{1, 2, 3}, {4, 5, 6}};
    
    // Memory is contiguous:
    // [1][2][3][4][5][6]
    
    // Accessing as 1D array
    int *flat = (int *)matrix;
    for (int i = 0; i < 6; i++) {
        printf("%d ", flat[i]);
    }
    printf("\n");
    
    // Variable length arrays (C99)
    int rows = 3, cols = 4;
    int vla[rows][cols];  // Size determined at runtime
    
    // Initialize VLA
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            vla[i][j] = i * cols + j;
        }
    }
}

int main() {
    array_decay();
    string_manipulation();
    multidimensional_arrays();
    
    return 0;
}
```

### 2.3 Structures and Unions

**Memory Alignment & Padding:**
```
struct Example {
    char a;      // 1 byte
    // 3 bytes padding
    int b;       // 4 bytes
    char c;      // 1 byte
    // 3 bytes padding
};
Total: 12 bytes (not 6!)

Memory Layout:
┌──┬──┬──┬──┬────────┬──┬──┬──┬──┐
│a │ padding │   b    │c │ padding │
└──┴──┴──┴──┴────────┴──┴──┴──┴──┘
 1      3       4      1      3
```

**Example:**
```c
#include <stdio.h>
#include <stddef.h>
#include <string.h>

// Structure with padding
struct Unoptimized {
    char a;      // 1 byte
    int b;       // 4 bytes (3 bytes padding before)
    char c;      // 1 byte (3 bytes padding after)
};

// Optimized structure (reordered for minimal padding)
struct Optimized {
    int b;       // 4 bytes
    char a;      // 1 byte
    char c;      // 1 byte (2 bytes padding after)
};

void demonstrate_padding() {
    printf("Unoptimized size: %zu bytes\n", sizeof(struct Unoptimized));  // 12
    printf("Optimized size: %zu bytes\n", sizeof(struct Optimized));      // 8
    
    struct Unoptimized u;
    printf("Offset of a: %zu\n", offsetof(struct Unoptimized, a));  // 0
    printf("Offset of b: %zu\n", offsetof(struct Unoptimized, b));  // 4
    printf("Offset of c: %zu\n", offsetof(struct Unoptimized, c));  // 8
}

// Bit fields for memory-efficient flags
struct Flags {
    unsigned int is_valid : 1;
    unsigned int is_active : 1;
    unsigned int priority : 3;   // 0-7
    unsigned int reserved : 27;
};

void demonstrate_bitfields() {
    struct Flags f = {0};
    f.is_valid = 1;
    f.priority = 5;
    
    printf("Flags size: %zu bytes\n", sizeof(f));  // 4 bytes for all flags
    printf("is_valid: %u, priority: %u\n", f.is_valid, f.priority);
}

// Union - all members share same memory
union Data {
    int i;
    float f;
    char str[20];
};

void demonstrate_union() {
    union Data d;
    
    d.i = 10;
    printf("d.i: %d\n", d.i);
    
    d.f = 220.5;  // Overwrites d.i
    printf("d.f: %.1f\n", d.f);
    printf("d.i now: %d (corrupted)\n", d.i);  // Garbage value
    
    printf("Union size: %zu (largest member)\n", sizeof(d));  // 20
}

// Flexible array member (C99)
struct Buffer {
    int size;
    char data[];  // Flexible array member (must be last)
};

void demonstrate_flexible_array() {
    struct Buffer *buf = malloc(sizeof(struct Buffer) + 100);
    buf->size = 100;
    strcpy(buf->data, "Flexible array member");
    printf("Buffer: %s\n", buf->data);
    free(buf);
}

// Self-referential structures (linked lists, trees)
struct Node {
    int data;
    struct Node *next;
};

void create_linked_list() {
    struct Node n1 = {1, NULL};
    struct Node n2 = {2, NULL};
    struct Node n3 = {3, NULL};
    
    n1.next = &n2;
    n2.next = &n3;
    
    struct Node *current = &n1;
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}

int main() {
    demonstrate_padding();
    demonstrate_bitfields();
    demonstrate_union();
    demonstrate_flexible_array();
    create_linked_list();
    
    return 0;
}
```

### 2.4 Dynamic Memory Allocation

**Heap Memory Layout:**
```
┌──────────────────────────────────┐
│         Stack (grows down)       │
│               ↓                  │
├──────────────────────────────────┤
│            ...                   │
├──────────────────────────────────┤
│         Heap (grows up)          │
│               ↑                  │
│   malloc/free manage this area   │
├──────────────────────────────────┤
│         BSS (uninitialized)      │
├──────────────────────────────────┤
│         Data (initialized)       │
├──────────────────────────────────┤
│         Text (code)              │
└──────────────────────────────────┘
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void basic_allocation() {
    // malloc - allocates uninitialized memory
    int *ptr = (int *)malloc(5 * sizeof(int));
    if (ptr == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return;
    }
    
    // Initialize allocated memory
    for (int i = 0; i < 5; i++) {
        ptr[i] = i * 10;
    }
    
    // calloc - allocates zero-initialized memory
    int *ptr2 = (int *)calloc(5, sizeof(int));
    printf("Calloc values (should be 0): ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", ptr2[i]);
    }
    printf("\n");
    
    // realloc - resize allocated memory
    ptr = (int *)realloc(ptr, 10 * sizeof(int));
    if (ptr == NULL) {
        fprintf(stderr, "Reallocation failed\n");
        free(ptr2);
        return;
    }
    
    // Initialize new elements
    for (int i = 5; i < 10; i++) {
        ptr[i] = i * 10;
    }
    
    free(ptr);
    free(ptr2);
}

// Memory leak demonstration
void memory_leak_example() {
    for (int i = 0; i < 5; i++) {
        int *leak = malloc(sizeof(int));
        *leak = i;
        // Forgot to free! Memory leak
    }
    printf("Memory leaked: %zu bytes\n", 5 * sizeof(int));
}

// Double free demonstration (dangerous!)
void double_free_danger() {
    int *ptr = malloc(sizeof(int));
    free(ptr);
    // free(ptr);  // UNDEFINED BEHAVIOR! May crash
    
    // Use after free (also dangerous!)
    // *ptr = 10;  // UNDEFINED BEHAVIOR!
}

// Dynamic 2D array
int** create_2d_array(int rows, int cols) {
    int **arr = (int **)malloc(rows * sizeof(int *));
    if (arr == NULL) return NULL;
    
    for (int i = 0; i < rows; i++) {
        arr[i] = (int *)malloc(cols * sizeof(int));
        if (arr[i] == NULL) {
            // Cleanup on failure
            for (int j = 0; j < i; j++) {
                free(arr[j]);
            }
            free(arr);
            return NULL;
        }
    }
    
    return arr;
}

void free_2d_array(int **arr, int rows) {
    for (int i = 0; i < rows; i++) {
        free(arr[i]);
    }
    free(arr);
}

// Memory alignment example
void alignment_example() {
    // aligned_alloc requires C11
    void *ptr = aligned_alloc(64, 128);  // 128 bytes aligned to 64-byte boundary
    if (ptr != NULL) {
        printf("Aligned address: %p\n", ptr);
        printf("Is 64-byte aligned: %s\n", 
               ((uintptr_t)ptr % 64 == 0) ? "Yes" : "No");
        free(ptr);
    }
}

// Custom memory pool (simple implementation)
typedef struct MemoryPool {
    void *memory;
    size_t size;
    size_t used;
} MemoryPool;

MemoryPool* create_pool(size_t size) {
    MemoryPool *pool = malloc(sizeof(MemoryPool));
    if (pool == NULL) return NULL;
    
    pool->memory = malloc(size);
    if (pool->memory == NULL) {
        free(pool);
        return NULL;
    }
    
    pool->size = size;
    pool->used = 0;
    return pool;
}

void* pool_alloc(MemoryPool *pool, size_t size) {
    if (pool->used + size > pool->size) {
        return NULL;  // Pool exhausted
    }
    
    void *ptr = (char *)pool->memory + pool->used;
    pool->used += size;
    return ptr;
}

void destroy_pool(MemoryPool *pool) {
    if (pool != NULL) {
        free(pool->memory);
        free(pool);
    }
}

int main() {
    basic_allocation();
    memory_leak_example();
    
    int **matrix = create_2d_array(3, 4);
    if (matrix != NULL) {
        // Use matrix
        matrix[1][2] = 42;
        printf("matrix[1][2] = %d\n", matrix[1][2]);
        free_2d_array(matrix, 3);
    }
    
    alignment_example();
    
    // Memory pool example
    MemoryPool *pool = create_pool(1024);
    if (pool != NULL) {
        int *a = pool_alloc(pool, sizeof(int) * 10);
        char *b = pool_alloc(pool, sizeof(char) * 50);
        printf("Pool used: %zu / %zu bytes\n", pool->used, pool->size);
        destroy_pool(pool);
    }
    
    return 0;
}
```

---

## 3. Memory Management Mastery

### 3.1 Understanding Virtual Memory

```
Process Virtual Address Space (32-bit):

0xFFFFFFFF  ┌──────────────────────┐
            │    Kernel Space      │  (1GB)
0xC0000000  ├──────────────────────┤
            │                      │
            │    Stack             │  ↓ (grows down)
            │                      │
            ├──────────────────────┤
            │                      │
            │    Memory Mapping    │  (shared libs, mmap)
            │                      │
            ├──────────────────────┤
            │                      │
            │    Heap              │  ↑ (grows up)
            │                      │
            ├──────────────────────┤
            │    BSS               │  (uninitialized data)
            ├──────────────────────┤
            │    Data              │  (initialized data)
            ├──────────────────────┤
            │    Text              │  (program code)
0x08048000  └──────────────────────┘
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int global_initialized = 42;     // Data segment
int global_uninitialized;        // BSS segment
const int global_const = 100;    // Read-only data segment

void print_memory_layout() {
    int stack_var = 10;          // Stack
    static int static_var = 20;  // Data segment
    int *heap_var = malloc(sizeof(int));  // Heap
    *heap_var = 30;
    
    printf("Memory Layout:\n");
    printf("Text (code):      %p\n", (void*)print_memory_layout);
    printf("Data (init):      %p\n", (void*)&global_initialized);
    printf("BSS (uninit):     %p\n", (void*)&global_uninitialized);
    printf("Heap:             %p\n", (void*)heap_var);
    printf("Stack:            %p\n", (void*)&stack_var);
    printf("Static:           %p\n", (void*)&static_var);
    
    free(heap_var);
}

int main() {
    print_memory_layout();
    return 0;
}
```

### 3.2 Memory Allocators Deep Dive

**How malloc() Works (Simplified):**
```
Free List Structure:

┌─────────────────────────────────────────┐
│  Allocated Block                        │
│  ┌──────────┬────────────────────────┐  │
│  │ Size|1   │   User Data            │  │
│  └──────────┴────────────────────────┘  │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  Free Block                             │
│  ┌──────────┬────┬──────┬──────────┐    │
│  │ Size|0   │Next│Prev  │  Unused  │    │
│  └──────────┴────┴──────┴──────────┘    │
└─────────────────────────────────────────┘

Last bit of size field indicates allocation status
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

// Simple memory allocator implementation
#define POOL_SIZE 4096

typedef struct Block {
    size_t size;
    int is_free;
    struct Block *next;
} Block;

static char memory_pool[POOL_SIZE];
static Block *free_list = NULL;
static int initialized = 0;

void init_allocator() {
    free_list = (Block*)memory_pool;
    free_list->size = POOL_SIZE - sizeof(Block);
    free_list->is_free = 1;
    free_list->next = NULL;
    initialized = 1;
}

void* my_malloc(size_t size) {
    if (!initialized) init_allocator();
    
    Block *current = free_list;
    
    // First fit strategy
    while (current != NULL) {
        if (current->is_free && current->size >= size) {
            // Split block if remainder is large enough
            if (current->size >= size + sizeof(Block) + 1) {
                Block *new_block = (Block*)((char*)current + sizeof(Block) + size);
                new_block->size = current->size - size - sizeof(Block);
                new_block->is_free = 1;
                new_block->next = current->next;
                
                current->size = size;
                current->next = new_block;
            }
            
            current->is_free = 0;
            return (void*)(current + 1);  // Return pointer after header
        }
        current = current->next;
    }
    
    return NULL;  // No suitable block found
}

void my_free(void *ptr) {
    if (ptr == NULL) return;
    
    Block *block = (Block*)ptr - 1;
    block->is_free = 1;
    
    // Coalesce with next block if free
    if (block->next && block->next->is_free) {
        block->size += sizeof(Block) + block->next->size;
        block->next = block->next->next;
    }
}

void print_memory_state() {
    Block *current = free_list;
    int block_num = 0;
    
    printf("\nMemory Pool State:\n");
    while (current != NULL) {
        printf("Block %d: Size=%zu, Free=%s\n", 
               block_num++, current->size, 
               current->is_free ? "Yes" : "No");
        current = current->next;
    }
}

// Memory fragmentation demonstration
void demonstrate_fragmentation() {
    printf("\n=== Fragmentation Demo ===\n");
    
    void *p1 = my_malloc(100);
    void *p2 = my_malloc(100);
    void *p3 = my_malloc(100);
    void *p4 = my_malloc(100);
    
    print_memory_state();
    
    // Free alternating blocks
    my_free(p2);
    my_free(p4);
    
    printf("\nAfter freeing p2 and p4:\n");
    print_memory_state();
    
    // Try to allocate 250 bytes - will fail due to fragmentation
    void *p5 = my_malloc(250);
    printf("\nAllocation of 250 bytes: %s\n", 
           p5 ? "Success" : "Failed (fragmented)");
    
    my_free(p1);
    my_free(p3);
}

int main() {
    demonstrate_fragmentation();
    return 0;
}
```

---

## 4. Data Structures & Algorithms

### 4.1 Linked Lists

**Singly Linked List:**
```
┌──────┬────┐    ┌──────┬────┐    ┌──────┬────┐
│ Data │Next│───→│ Data │Next│───→│ Data │NULL│
└──────┴────┘    └──────┴────┘    └──────┴────┘
   Head                               Tail
```

**Example:**
```c
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

LinkedList* create_list() {
    LinkedList *list = malloc(sizeof(LinkedList));
    list->head = NULL;
    list->size = 0;
    return list;
}

void insert_front(LinkedList *list, int data) {
    Node *new_node = malloc(sizeof(Node));
    new_node->data = data;
    new_node->next = list->head;
    list->head = new_node;
    list->size++;
}

void insert_end(LinkedList *list, int data) {
    Node *new_node = malloc(sizeof(Node));
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

int delete_node(LinkedList *list, int data) {
    if (list->head == NULL) return 0;
    
    // If head needs to be deleted
    if (list->head->data == data) {
        Node *temp = list->head;
        list->head = list->head->next;
        free(temp);
        list->size--;
        return 1;
    }
    
    Node *current = list->head;
    while (current->next != NULL) {
        if (current->next->data == data) {
            Node *temp = current->next;
            current->next = current->next->next;
            free(temp);
            list->size--;
            return 1;
        }
        current = current->next;
    }
    return 0;
}

void reverse_list(LinkedList *list) {
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

// Detect cycle using Floyd's algorithm (Tortoise and Hare)
int has_cycle(LinkedList *list) {
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

// Find middle element
Node* find_middle(LinkedList *list) {
    if (list->head == NULL) return NULL;
    
    Node *slow = list->head;
    Node *fast = list->head;
    
    while (fast->next != NULL && fast->next->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}

void print_list(LinkedList *list) {
    Node *current = list->head;
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}

void free_list(LinkedList *list) {
    Node *current = list->head;
    while (current != NULL) {
        Node *temp = current;
        current = current->next;
        free(temp);
    }
    free(list);
}

int main() {
    LinkedList *list = create_list();
    
    insert_end(list, 10);
    insert_end(list, 20);
    insert_end(list, 30);
    insert_front(list, 5);
    
    printf("Original list: ");
    print_list(list);
    
    Node *middle = find_middle(list);
    printf("Middle element: %d\n", middle->data);
    
    reverse_list(list);
    printf("Reversed list: ");
    print_list(list);
    
    delete_node(list, 20);
    printf("After deleting 20: ");
    print_list(list);
    
    free_list(list);
    return 0;
}
```

### 4.2 Stacks and Queues

**Stack (LIFO - Last In First Out):**
```
      Push(3)          Pop()
         ↓              ↑
    ┌────────┐     ┌────────┐
    │   3    │     │   2    │ ← Top
    ├────────┤     ├────────┤
    │   2    │     │   1    │
    ├────────┤     └────────┘
    │   1    │
    └────────┘
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_SIZE 100

// Array-based stack
typedef struct {
    int items[MAX_SIZE];
    int top;
} Stack;

Stack* create_stack() {
    Stack *s = malloc(sizeof(Stack));
    s->top = -1;
    return s;
}

int is_empty(Stack *s) {
    return s->top == -1;
}

int is_full(Stack *s) {
    return s->top == MAX_SIZE - 1;
}

void push(Stack *s, int item) {
    if (is_full(s)) {
        printf("Stack overflow\n");
        return;
    }
    s->items[++(s->top)] = item;
}

int pop(Stack *s) {
    if (is_empty(s)) {
        printf("Stack underflow\n");
        return -1;
    }
    return s->items[(s->top)--];
}

int peek(Stack *s) {
    if (is_empty(s)) return -1;
    return s->items[s->top];
}

// Balanced parentheses checker
int is_balanced(const char *expr) {
    Stack *s = create_stack();
    
    for (int i = 0; expr[i] != '\0'; i++) {
        char ch = expr[i];
        
        if (ch == '(' || ch == '[' || ch == '{') {
            push(s, ch);
        } else if (ch == ')' || ch == ']' || ch == '}') {
            if (is_empty(s)) {
                free(s);
                return 0;
            }
            
            char top = pop(s);
            if ((ch == ')' && top != '(') ||
                (ch == ']' && top != '[') ||
                (ch == '}' && top != '{')) {
                free(s);
                return 0;
            }
        }
    }
    
    int result = is_empty(s);
    free(s);
    return result;
}

// Queue (FIFO - First In First Out)
typedef struct {
    int items[MAX_SIZE];
    int front, rear;
} Queue;

Queue* create_queue() {
    Queue *q = malloc(sizeof(Queue));
    q->front = -1;
    q->rear = -1;
    return q;
}

int queue_is_empty(Queue *q) {
    return q->front == -1;
}

int queue_is_full(Queue *q) {
    return (q->rear + 1) % MAX_SIZE == q->front;
}

void enqueue(Queue *q, int item) {
    if (queue_is_full(q)) {
        printf("Queue is full\n");
        return;
    }
    
    if (queue_is_empty(q)) {
        q->front = 0;
    }
    q->rear = (q->rear + 1) % MAX_SIZE;
    q->items[q->rear] = item;
}

int dequeue(Queue *q) {
    if (queue_is_empty(q)) {
        printf("Queue is empty\n");
        return -1;
    }
    
    int item = q->items[q->front];
    if (q->front == q->rear) {
        q->front = q->rear = -1;
    } else {
        q->front = (q->front + 1) % MAX_SIZE;
    }
    return item;
}

int main() {
    // Stack demo
    Stack *s = create_stack();
    push(s, 10);
    push(s, 20);
    push(s, 30);
    printf("Popped: %d\n", pop(s));
    printf("Top: %d\n", peek(s));
    free(s);
    
    // Balanced parentheses
    printf("({[]}) is balanced: %s\n", 
           is_balanced("({[]})") ? "Yes" : "No");
    printf("({[}]) is balanced: %s\n", 
           is_balanced("({[}])") ? "Yes" : "No");
    
    // Queue demo
    Queue *q = create_queue();
    enqueue(q, 1);
    enqueue(q, 2);
    enqueue(q, 3);
    printf("Dequeued: %d\n", dequeue(q));
    printf("Dequeued: %d\n", dequeue(q));
    free(q);
    
    return 0;
}
```

---

## CONTINUED IN PART 2

This is Part 1 of the Complete C Programming Master Roadmap. It covers:
- Foundation & Basics
- Advanced C Concepts
- Memory Management
- Data Structures (Linked Lists, Stacks, Queues)

**Part 2 will include:**
- Trees (Binary Trees, BST, AVL, Red-Black)
- Hash Tables
- Graphs and Algorithms
- Sorting & Searching
- Systems Programming
- Linux Internals
- And much more!

Save this file and continue with Part 2 for the complete roadmap.