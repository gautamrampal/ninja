# PHP Source Code Mastery from C Programming Perspective

## Complete Guide to Studying, Modifying, and Contributing to PHP Codebase

---

## Table of Contents

1. [Introduction](#introduction)
2. [PHP Architecture Overview](#php-architecture-overview)
3. [Setting Up Development Environment](#setting-up-development-environment)
4. [PHP Codebase Structure](#php-codebase-structure)
5. [Zend Engine Deep Dive](#zend-engine-deep-dive)
6. [Memory Management in PHP](#memory-management-in-php)
7. [Building PHP Extensions](#building-php-extensions)
8. [Creating Custom Functions](#creating-custom-functions)
9. [Creating Custom Modules](#creating-custom-modules)
10. [PHP Internals API Reference](#php-internals-api-reference)
11. [Advanced Topics](#advanced-topics)
12. [Contributing to PHP Core](#contributing-to-php-core)
13. [Debugging and Testing](#debugging-and-testing)
14. [Best Practices](#best-practices)

---

## 1. Introduction

### Why Study PHP Internals from C Perspective?

PHP is written primarily in C, making it essential for developers who want to:
- Understand how PHP works under the hood
- Build high-performance extensions
- Contribute to PHP core development
- Create custom modules for specific needs
- Optimize PHP applications at the lowest level

### Prerequisites

- Strong C programming knowledge
- Understanding of memory management
- Familiarity with PHP language
- Unix/Linux system programming basics
- Understanding of compilers and build systems

---

## 2. PHP Architecture Overview

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────┐
│         PHP Application Layer           │
│    (PHP Scripts, User Code)            │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│           SAPI Layer                    │
│  (CLI, Apache, FPM, CGI, etc.)        │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│         Zend Engine Core                │
│  (Compiler, Executor, Memory Manager)  │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│        PHP Extensions Layer             │
│  (Standard, MySQLi, PDO, Custom, etc.) │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│        Operating System Layer           │
│    (System Calls, Libraries)           │
└─────────────────────────────────────────┘
```

### 2.2 Key Components

**SAPI (Server API)**
- Interface between web servers and PHP
- Examples: mod_php, PHP-FPM, CLI
- Handles request/response lifecycle

**Zend Engine**
- Core execution engine
- Compiles PHP to opcodes
- Executes bytecode
- Manages memory and resources

**Extensions**
- Modular functionality
- Can be compiled statically or loaded dynamically
- Provide additional functions and classes

---

## 3. Setting Up Development Environment

### 3.1 Getting PHP Source Code

```bash
# Clone PHP repository
git clone https://github.com/php/php-src.git
cd php-src

# Checkout a specific version (e.g., PHP 8.2)
git checkout PHP-8.2
```

### 3.2 Build Requirements

**Linux/Unix:**
```bash
# Ubuntu/Debian
sudo apt-get install build-essential autoconf automake libtool \
    bison re2c libxml2-dev libsqlite3-dev

# CentOS/RHEL
sudo yum groupinstall "Development Tools"
sudo yum install autoconf automake libtool bison re2c libxml2-devel \
    sqlite-devel
```

### 3.3 Building PHP from Source

```bash
# Generate configure script
./buildconf

# Configure with debug symbols
./configure --enable-debug \
            --enable-maintainer-zts \
            --prefix=/usr/local/php-dev \
            --with-config-file-path=/usr/local/php-dev/etc \
            --enable-mbstring \
            --with-openssl \
            --with-zlib

# Compile
make -j$(nproc)

# Install
sudo make install

# Verify installation
/usr/local/php-dev/bin/php -v
```

### 3.4 Debug Build Configuration

```bash
# Full debug build with additional checks
./configure --enable-debug \
            --enable-maintainer-zts \
            --disable-opcache-jit \
            --enable-address-sanitizer \
            --enable-undefined-sanitizer \
            --prefix=/usr/local/php-debug
```

---

## 4. PHP Codebase Structure

### 4.1 Directory Layout

```
php-src/
├── Zend/              # Zend Engine source code
│   ├── zend.h        # Main header
│   ├── zend_API.h    # API definitions
│   ├── zend_compile.c # Compiler
│   ├── zend_execute.c # Executor
│   ├── zend_vm_def.h  # VM definitions
│   └── zend_alloc.c   # Memory allocator
├── main/              # PHP core functionality
│   ├── php.h         # Main PHP header
│   ├── php_main.c    # Entry points
│   ├── SAPI.c        # SAPI abstraction
│   └── php_ini.c     # INI parsing
├── ext/               # Standard extensions
│   ├── standard/     # Standard PHP functions
│   ├── mysqli/       # MySQL improved extension
│   ├── pdo/          # PDO abstraction
│   └── json/         # JSON extension
├── sapi/              # SAPI implementations
│   ├── cli/          # Command-line interface
│   ├── fpm/          # FastCGI Process Manager
│   └── apache2handler/ # Apache module
├── TSRM/              # Thread Safe Resource Manager
└── tests/             # Test suite
```

### 4.2 Key Header Files

**zend.h** - Core Zend definitions
```c
#ifndef ZEND_H
#define ZEND_H

// Basic type definitions
typedef unsigned char zend_bool;
typedef intptr_t zend_intptr_t;
typedef uintptr_t zend_uintptr_t;

// Core macros
#define ZEND_API
#define SUCCESS 0
#define FAILURE -1

#endif
```

**php.h** - Main PHP header
```c
#ifndef PHP_H
#define PHP_H

#include "zend.h"
#include "zend_API.h"

// PHP version macros
#define PHP_VERSION "8.2.0"
#define PHP_MAJOR_VERSION 8
#define PHP_MINOR_VERSION 2

// Module entry structure
typedef struct _zend_module_entry zend_module_entry;

#endif
```

---

## 5. Zend Engine Deep Dive

### 5.1 Zend Value (zval) Structure

The `zval` is the fundamental data structure in PHP that represents any PHP value.

```c
// Simplified zval structure (PHP 7+)
typedef struct _zval_struct {
    zend_value value;           // Actual value storage
    union {
        struct {
            zend_uchar type;    // Variable type
            zend_uchar type_flags;
            union {
                uint16_t extra;
            } u;
        } v;
        uint32_t type_info;
    } u1;
    union {
        uint32_t next;          // For hash collision chaining
        uint32_t cache_slot;
        uint32_t opline_num;
        uint32_t lineno;
        uint32_t num_args;
        uint32_t fe_pos;
        uint32_t fe_iter_idx;
        uint32_t access_flags;
        uint32_t property_guard;
        uint32_t constant_flags;
        uint32_t extra;
    } u2;
} zval;

// Value union
typedef union _zend_value {
    zend_long lval;             // Long (integer)
    double dval;                // Double (float)
    zend_refcounted *counted;   // Reference counted types
    zend_string *str;           // String
    zend_array *arr;            // Array
    zend_object *obj;           // Object
    zend_resource *res;         // Resource
    zend_reference *ref;        // Reference
    zend_ast_ref *ast;          // Abstract syntax tree
    zval *zv;                   // Pointer to another zval
    void *ptr;                  // Generic pointer
    zend_class_entry *ce;       // Class entry
    zend_function *func;        // Function
    struct {
        uint32_t w1;
        uint32_t w2;
    } ww;
} zend_value;
```

### 5.2 Type System

```c
// PHP data types
#define IS_UNDEF     0
#define IS_NULL      1
#define IS_FALSE     2
#define IS_TRUE      3
#define IS_LONG      4
#define IS_DOUBLE    5
#define IS_STRING    6
#define IS_ARRAY     7
#define IS_OBJECT    8
#define IS_RESOURCE  9
#define IS_REFERENCE 10

// Type checking macros
#define Z_TYPE(zval)         (zval).u1.v.type
#define Z_TYPE_P(zval_p)     Z_TYPE(*(zval_p))

// Example: Type checking
void check_zval_type(zval *val) {
    switch (Z_TYPE_P(val)) {
        case IS_NULL:
            printf("NULL\n");
            break;
        case IS_LONG:
            printf("Integer: %ld\n", Z_LVAL_P(val));
            break;
        case IS_DOUBLE:
            printf("Float: %f\n", Z_DVAL_P(val));
            break;
        case IS_STRING:
            printf("String: %s\n", ZSTR_VAL(Z_STR_P(val)));
            break;
        case IS_ARRAY:
            printf("Array\n");
            break;
        case IS_OBJECT:
            printf("Object\n");
            break;
        default:
            printf("Other type\n");
    }
}
```

### 5.3 Working with zval

```c
// Creating and manipulating zvals

// Example 1: Creating a long (integer)
zval my_long;
ZVAL_LONG(&my_long, 42);

// Example 2: Creating a string
zval my_string;
ZVAL_STRING(&my_string, "Hello, PHP!");

// Example 3: Creating an array
zval my_array;
array_init(&my_array);
add_index_string(&my_array, 0, "first");
add_index_long(&my_array, 1, 42);
add_assoc_string(&my_array, "key", "value");

// Example 4: Creating a double
zval my_double;
ZVAL_DOUBLE(&my_double, 3.14159);

// Example 5: Setting to NULL
zval my_null;
ZVAL_NULL(&my_null);

// Example 6: Boolean values
zval my_true, my_false;
ZVAL_BOOL(&my_true, 1);
ZVAL_BOOL(&my_false, 0);
```

### 5.4 Reference Counting

PHP uses reference counting for memory management:

```c
// Reference counting macros
#define Z_REFCOUNT_P(zval_p)     zval_refcount_p(zval_p)
#define Z_ADDREF_P(zval_p)       zval_addref_p(zval_p)
#define Z_DELREF_P(zval_p)       zval_delref_p(zval_p)

// Example: Reference counting in action
void reference_counting_example() {
    zval my_string;
    ZVAL_STRING(&my_string, "Reference counted");
    
    // Initial refcount is 1
    printf("Refcount: %d\n", Z_REFCOUNT(my_string));
    
    // Add reference
    Z_ADDREF(my_string);
    printf("After addref: %d\n", Z_REFCOUNT(my_string));
    
    // Remove reference
    Z_DELREF(my_string);
    printf("After delref: %d\n", Z_REFCOUNT(my_string));
    
    // Clean up
    zval_ptr_dtor(&my_string);
}
```

### 5.5 String Handling

PHP 7+ uses `zend_string` for string management:

```c
// zend_string structure
struct _zend_string {
    zend_refcounted_h gc;      // Garbage collection header
    zend_ulong h;              // Hash value
    size_t len;                // String length
    char val[1];               // Flexible array member
};

// Creating strings
zend_string *str1 = zend_string_init("Hello", sizeof("Hello")-1, 0);
zend_string *str2 = zend_string_alloc(100, 0);  // Allocate 100 bytes

// String operations
zend_string *result = zend_string_concat2(
    ZSTR_VAL(str1), ZSTR_LEN(str1),
    " World", sizeof(" World")-1
);

// String comparison
if (zend_string_equals_literal(str1, "Hello")) {
    printf("Strings match!\n");
}

// Hashing
zend_ulong hash = zend_string_hash_val(str1);

// Release string
zend_string_release(str1);
```

### 5.6 Array (HashTable) Implementation

PHP arrays are implemented as hash tables:

```c
// HashTable structure (simplified)
typedef struct _zend_array {
    zend_refcounted_h gc;
    union {
        struct {
            zend_uchar flags;
            zend_uchar nApplyCount;
            zend_uchar nIteratorsCount;
            zend_uchar _unused;
        } v;
        uint32_t flags;
    } u;
    uint32_t nTableMask;
    Bucket *arData;
    uint32_t nNumUsed;
    uint32_t nNumOfElements;
    uint32_t nTableSize;
    uint32_t nInternalPointer;
    zend_long nNextFreeElement;
    dtor_func_t pDestructor;
} zend_array, HashTable;

// Bucket structure (array element)
typedef struct _Bucket {
    zval val;                  // The value
    zend_ulong h;              // Hash value for numeric keys
    zend_string *key;          // String key (NULL for numeric)
} Bucket;

// Working with arrays
void array_operations() {
    // Create array
    zval arr;
    array_init(&arr);
    HashTable *ht = Z_ARRVAL(arr);
    
    // Add elements
    zval val;
    ZVAL_LONG(&val, 42);
    zend_hash_index_add(ht, 0, &val);
    
    ZVAL_STRING(&val, "value");
    zend_hash_str_add(ht, "key", sizeof("key")-1, &val);
    
    // Retrieve elements
    zval *found = zend_hash_index_find(ht, 0);
    if (found) {
        printf("Found: %ld\n", Z_LVAL_P(found));
    }
    
    // Iterate array
    zend_string *key;
    zend_ulong num_key;
    zval *value;
    ZEND_HASH_FOREACH_KEY_VAL(ht, num_key, key, value) {
        if (key) {
            printf("Key: %s\n", ZSTR_VAL(key));
        } else {
            printf("Index: %lu\n", num_key);
        }
    } ZEND_HASH_FOREACH_END();
    
    // Clean up
    zval_ptr_dtor(&arr);
}
```

---

## 6. Memory Management in PHP

### 6.1 Zend Memory Manager

PHP has its own memory allocator optimized for web request lifecycle:

```c
// Memory allocation functions
void *emalloc(size_t size);                    // Request memory
void *ecalloc(size_t nmemb, size_t size);      // Zeroed memory
void *erealloc(void *ptr, size_t size);        // Resize
void efree(void *ptr);                         // Free memory
char *estrdup(const char *s);                  // Duplicate string
char *estrndup(const char *s, size_t len);     // Duplicate n bytes

// Persistent memory (survives request)
void *pemalloc(size_t size, int persistent);
void *pecalloc(size_t nmemb, size_t size, int persistent);
void *perealloc(void *ptr, size_t size, int persistent);
void pefree(void *ptr, int persistent);
char *pestrdup(const char *s, int persistent);

// Example: Memory management
void memory_example() {
    // Request memory
    char *buffer = emalloc(1024);
    strcpy(buffer, "This is freed automatically at request end");
    
    // Persistent memory (must be freed manually)
    char *persistent_buf = pemalloc(1024, 1);
    strcpy(persistent_buf, "This persists between requests");
    pefree(persistent_buf, 1);  // Must free
    
    // Request memory is auto-freed, but good practice:
    efree(buffer);
}
```

### 6.2 Memory Pools and Request Lifecycle

```c
// Memory lifecycle hooks
void php_request_startup_hook() {
    // Called at start of each request
    // Fresh memory pool allocated
}

void php_request_shutdown_hook() {
    // Called at end of each request
    // All request memory (emalloc) freed automatically
}

// Example: Extension with lifecycle management
typedef struct {
    char *persistent_data;
    char *request_data;
} my_extension_globals;

PHP_MINIT_FUNCTION(my_extension) {
    // Module initialization (once per process)
    MY_G(persistent_data) = pemalloc(1024, 1);
    return SUCCESS;
}

PHP_RINIT_FUNCTION(my_extension) {
    // Request initialization
    MY_G(request_data) = emalloc(1024);
    return SUCCESS;
}

PHP_RSHUTDOWN_FUNCTION(my_extension) {
    // Request shutdown
    // request_data freed automatically
    return SUCCESS;
}

PHP_MSHUTDOWN_FUNCTION(my_extension) {
    // Module shutdown (once per process)
    pefree(MY_G(persistent_data), 1);
    return SUCCESS;
}
```

### 6.3 Garbage Collection

PHP uses a cycle-collecting garbage collector:

```c
// Garbage collection API
void gc_collect_cycles(void);
int gc_enable(int enabled);
int gc_enabled(void);

// Making objects GC-aware
typedef struct _my_object {
    zend_object std;
    // Your data
    zval *data;
} my_object;

// Get buffer for cycle detection
static zend_object_handlers my_object_handlers;

void my_get_gc(zend_object *object, zval **table, int *n) {
    my_object *intern = (my_object*)object;
    
    *table = intern->data;
    *n = 1;
}

void init_my_object_handlers() {
    memcpy(&my_object_handlers, 
           zend_get_std_object_handlers(), 
           sizeof(zend_object_handlers));
    my_object_handlers.get_gc = my_get_gc;
}
```

---

## 7. Building PHP Extensions

### 7.1 Extension Skeleton

```bash
# Generate extension skeleton
cd php-src/ext
./ext_skel --ext myextension
cd myextension
```

### 7.2 Basic Extension Structure

**config.m4** - Build configuration
```m4
PHP_ARG_ENABLE(myextension, whether to enable myextension support,
[  --enable-myextension           Enable myextension support])

if test "$PHP_MYEXTENSION" != "no"; then
  PHP_NEW_EXTENSION(myextension, myextension.c, $ext_shared)
fi
```

**php_myextension.h** - Header file
```c
#ifndef PHP_MYEXTENSION_H
#define PHP_MYEXTENSION_H

extern zend_module_entry myextension_module_entry;
#define phpext_myextension_ptr &myextension_module_entry

#define PHP_MYEXTENSION_VERSION "1.0.0"

#ifdef PHP_WIN32
#    define PHP_MYEXTENSION_API __declspec(dllexport)
#elif defined(__GNUC__) && __GNUC__ >= 4
#    define PHP_MYEXTENSION_API __attribute__ ((visibility("default")))
#else
#    define PHP_MYEXTENSION_API
#endif

#ifdef ZTS
#include "TSRM.h"
#endif

// Function declarations
PHP_MINIT_FUNCTION(myextension);
PHP_MSHUTDOWN_FUNCTION(myextension);
PHP_RINIT_FUNCTION(myextension);
PHP_RSHUTDOWN_FUNCTION(myextension);
PHP_MINFO_FUNCTION(myextension);

PHP_FUNCTION(myextension_hello);
PHP_FUNCTION(myextension_add);

// Global structure
ZEND_BEGIN_MODULE_GLOBALS(myextension)
    zend_long counter;
    char *greeting;
ZEND_END_MODULE_GLOBALS(myextension)

#ifdef ZTS
#define MYEXT_G(v) TSRMG(myextension_globals_id, zend_myextension_globals *, v)
#else
#define MYEXT_G(v) (myextension_globals.v)
#endif

#endif    /* PHP_MYEXTENSION_H */
```

**myextension.c** - Implementation
```c
#ifdef HAVE_CONFIG_H
#include "config.h"
#endif

#include "php.h"
#include "php_ini.h"
#include "ext/standard/info.h"
#include "php_myextension.h"

// Global variables
ZEND_DECLARE_MODULE_GLOBALS(myextension)

// Function argument information
ZEND_BEGIN_ARG_INFO_EX(arginfo_myextension_hello, 0, 0, 0)
ZEND_END_ARG_INFO()

ZEND_BEGIN_ARG_INFO_EX(arginfo_myextension_add, 0, 0, 2)
    ZEND_ARG_INFO(0, a)
    ZEND_ARG_INFO(0, b)
ZEND_END_ARG_INFO()

// Function entries
const zend_function_entry myextension_functions[] = {
    PHP_FE(myextension_hello, arginfo_myextension_hello)
    PHP_FE(myextension_add, arginfo_myextension_add)
    PHP_FE_END
};

// Module entry
zend_module_entry myextension_module_entry = {
    STANDARD_MODULE_HEADER,
    "myextension",
    myextension_functions,
    PHP_MINIT(myextension),
    PHP_MSHUTDOWN(myextension),
    PHP_RINIT(myextension),
    PHP_RSHUTDOWN(myextension),
    PHP_MINFO(myextension),
    PHP_MYEXTENSION_VERSION,
    STANDARD_MODULE_PROPERTIES
};

#ifdef COMPILE_DL_MYEXTENSION
#ifdef ZTS
ZEND_TSRMLS_CACHE_DEFINE()
#endif
ZEND_GET_MODULE(myextension)
#endif

// INI entries
PHP_INI_BEGIN()
    STD_PHP_INI_ENTRY("myextension.greeting", "Hello", 
                      PHP_INI_ALL, OnUpdateString, greeting, 
                      zend_myextension_globals, myextension_globals)
PHP_INI_END()

// Lifecycle functions
PHP_MINIT_FUNCTION(myextension)
{
    REGISTER_INI_ENTRIES();
    MYEXT_G(counter) = 0;
    return SUCCESS;
}

PHP_MSHUTDOWN_FUNCTION(myextension)
{
    UNREGISTER_INI_ENTRIES();
    return SUCCESS;
}

PHP_RINIT_FUNCTION(myextension)
{
#if defined(COMPILE_DL_MYEXTENSION) && defined(ZTS)
    ZEND_TSRMLS_CACHE_UPDATE();
#endif
    return SUCCESS;
}

PHP_RSHUTDOWN_FUNCTION(myextension)
{
    return SUCCESS;
}

PHP_MINFO_FUNCTION(myextension)
{
    php_info_print_table_start();
    php_info_print_table_header(2, "myextension support", "enabled");
    php_info_print_table_row(2, "Version", PHP_MYEXTENSION_VERSION);
    php_info_print_table_end();
    
    DISPLAY_INI_ENTRIES();
}

// Function implementations
PHP_FUNCTION(myextension_hello)
{
    MYEXT_G(counter)++;
    php_printf("%s World! (Called %ld times)\n", 
               MYEXT_G(greeting), MYEXT_G(counter));
}

PHP_FUNCTION(myextension_add)
{
    zend_long a, b;
    
    ZEND_PARSE_PARAMETERS_START(2, 2)
        Z_PARAM_LONG(a)
        Z_PARAM_LONG(b)
    ZEND_PARSE_PARAMETERS_END();
    
    RETURN_LONG(a + b);
}
```

### 7.3 Building the Extension

```bash
# In the extension directory
phpize
./configure
make
make test

# Install
sudo make install

# Enable in php.ini
echo "extension=myextension.so" | sudo tee -a /usr/local/php-dev/etc/php.ini

# Verify
php -m | grep myextension
```

### 7.4 Testing the Extension

```php
<?php
// test.php
myextension_hello();  // Hello World! (Called 1 times)
myextension_hello();  // Hello World! (Called 2 times)

$result = myextension_add(5, 7);
echo "5 + 7 = $result\n";  // 5 + 7 = 12
?>
```

---

## 8. Creating Custom Functions

### 8.1 Parameter Parsing

```c
// Modern parameter parsing (PHP 7+)
PHP_FUNCTION(my_function)
{
    zend_string *str;
    zend_long num;
    zend_bool flag = 0;
    zval *array = NULL;
    
    ZEND_PARSE_PARAMETERS_START(2, 4)
        Z_PARAM_STR(str)              // Required string
        Z_PARAM_LONG(num)             // Required long
        Z_PARAM_OPTIONAL              // Optional parameters follow
        Z_PARAM_BOOL(flag)            // Optional bool
        Z_PARAM_ARRAY(array)          // Optional array
    ZEND_PARSE_PARAMETERS_END();
    
    php_printf("String: %s, Number: %ld, Flag: %d\n",
               ZSTR_VAL(str), num, flag);
}

// Legacy parameter parsing
PHP_FUNCTION(my_legacy_function)
{
    char *str;
    size_t str_len;
    zend_long num;
    
    if (zend_parse_parameters(ZEND_NUM_ARGS(), "sl", 
                              &str, &str_len, &num) == FAILURE) {
        return;
    }
    
    php_printf("String: %s (len=%zu), Number: %ld\n", 
               str, str_len, num);
}
```

### 8.2 Return Values

```c
// Return macros
PHP_FUNCTION(return_examples)
{
    // Return long
    RETURN_LONG(42);
    
    // Return double
    RETURN_DOUBLE(3.14);
    
    // Return string
    RETURN_STRING("Hello");
    
    // Return string with length
    RETURN_STRINGL("Binary\0Safe", 11);
    
    // Return bool
    RETURN_BOOL(1);
    RETURN_TRUE;
    RETURN_FALSE;
    
    // Return null
    RETURN_NULL();
    
    // Return array
    array_init(return_value);
    add_index_string(return_value, 0, "first");
    add_next_index_long(return_value, 42);
    return;  // Don't use RETURN_* with return_value
    
    // Return empty array
    RETURN_EMPTY_ARRAY();
    
    // Return resource
    zend_resource *res = zend_register_resource(data, le_myres);
    RETURN_RES(res);
}
```

### 8.3 Advanced Function Examples

**Example 1: String Manipulation**
```c
PHP_FUNCTION(my_str_reverse)
{
    zend_string *input;
    zend_string *output;
    size_t i;
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_STR(input)
    ZEND_PARSE_PARAMETERS_END();
    
    // Allocate output string
    output = zend_string_alloc(ZSTR_LEN(input), 0);
    
    // Reverse
    for (i = 0; i < ZSTR_LEN(input); i++) {
        ZSTR_VAL(output)[i] = ZSTR_VAL(input)[ZSTR_LEN(input) - 1 - i];
    }
    
    ZSTR_VAL(output)[ZSTR_LEN(input)] = '\0';
    
    RETURN_STR(output);
}
```

**Example 2: Array Processing**
```c
PHP_FUNCTION(my_array_sum)
{
    zval *array;
    zval *entry;
    double sum = 0.0;
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_ARRAY(array)
    ZEND_PARSE_PARAMETERS_END();
    
    ZEND_HASH_FOREACH_VAL(Z_ARRVAL_P(array), entry) {
        convert_to_double(entry);
        sum += Z_DVAL_P(entry);
    } ZEND_HASH_FOREACH_END();
    
    RETURN_DOUBLE(sum);
}
```

**Example 3: Callback Function**
```c
PHP_FUNCTION(my_callback)
{
    zval *callback;
    zval retval;
    zval args[2];
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_ZVAL(callback)
    ZEND_PARSE_PARAMETERS_END();
    
    if (!zend_is_callable(callback, 0, NULL)) {
        php_error_docref(NULL, E_WARNING, "Parameter must be callable");
        RETURN_FALSE;
    }
    
    // Prepare arguments
    ZVAL_STRING(&args[0], "Hello");
    ZVAL_LONG(&args[1], 42);
    
    // Call the callback
    if (call_user_function(EG(function_table), NULL, callback, 
                          &retval, 2, args) == SUCCESS) {
        // Copy return value
        ZVAL_COPY(return_value, &retval);
        zval_ptr_dtor(&retval);
    } else {
        RETURN_FALSE;
    }
    
    // Clean up
    zval_ptr_dtor(&args[0]);
}
```

**Example 4: File Operations**
```c
PHP_FUNCTION(my_file_read)
{
    zend_string *filename;
    FILE *fp;
    zend_string *contents;
    size_t filesize;
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_STR(filename)
    ZEND_PARSE_PARAMETERS_END();
    
    // Open file
    fp = fopen(ZSTR_VAL(filename), "rb");
    if (!fp) {
        php_error_docref(NULL, E_WARNING, 
                        "Failed to open file: %s", ZSTR_VAL(filename));
        RETURN_FALSE;
    }
    
    // Get file size
    fseek(fp, 0, SEEK_END);
    filesize = ftell(fp);
    rewind(fp);
    
    // Allocate and read
    contents = zend_string_alloc(filesize, 0);
    if (fread(ZSTR_VAL(contents), 1, filesize, fp) != filesize) {
        fclose(fp);
        zend_string_release(contents);
        RETURN_FALSE;
    }
    
    ZSTR_VAL(contents)[filesize] = '\0';
    fclose(fp);
    
    RETURN_STR(contents);
}
```

### 8.4 Function Registration

```c
// Argument info with type hints
ZEND_BEGIN_ARG_WITH_RETURN_TYPE_INFO_EX(arginfo_my_typed_func, 0, 2, IS_LONG, 0)
    ZEND_ARG_TYPE_INFO(0, a, IS_LONG, 0)
    ZEND_ARG_TYPE_INFO(0, b, IS_LONG, 0)
ZEND_END_ARG_INFO()

// Nullable return
ZEND_BEGIN_ARG_WITH_RETURN_TYPE_INFO_EX(arginfo_nullable, 0, 1, IS_STRING, 1)
    ZEND_ARG_TYPE_INFO(0, input, IS_STRING, 0)
ZEND_END_ARG_INFO()

// Variadic function
ZEND_BEGIN_ARG_INFO_EX(arginfo_variadic, 0, 0, 0)
    ZEND_ARG_VARIADIC_INFO(0, args)
ZEND_END_ARG_INFO()

const zend_function_entry my_functions[] = {
    PHP_FE(my_typed_func, arginfo_my_typed_func)
    PHP_FE(my_nullable, arginfo_nullable)
    PHP_FE(my_variadic, arginfo_variadic)
    PHP_FE_END
};
```

---

## 9. Creating Custom Modules

### 9.1 Complete Module Example: Database Wrapper

**php_mydb.h**
```c
#ifndef PHP_MYDB_H
#define PHP_MYDB_H

#define PHP_MYDB_VERSION "1.0.0"

extern zend_module_entry mydb_module_entry;
#define phpext_mydb_ptr &mydb_module_entry

// Resource type
typedef struct {
    void *connection;
    int is_connected;
} php_mydb_connection;

// Lifecycle functions
PHP_MINIT_FUNCTION(mydb);
PHP_MSHUTDOWN_FUNCTION(mydb);
PHP_RINIT_FUNCTION(mydb);
PHP_RSHUTDOWN_FUNCTION(mydb);
PHP_MINFO_FUNCTION(mydb);

// API functions
PHP_FUNCTION(mydb_connect);
PHP_FUNCTION(mydb_query);
PHP_FUNCTION(mydb_fetch);
PHP_FUNCTION(mydb_close);

// Resource destructor
static void php_mydb_connection_dtor(zend_resource *rsrc);

#endif
```

**mydb.c**
```c
#include "php.h"
#include "php_mydb.h"

static int le_mydb_connection;

ZEND_BEGIN_ARG_INFO_EX(arginfo_mydb_connect, 0, 0, 2)
    ZEND_ARG_TYPE_INFO(0, host, IS_STRING, 0)
    ZEND_ARG_TYPE_INFO(0, username, IS_STRING, 0)
    ZEND_ARG_TYPE_INFO(0, password, IS_STRING, 0)
ZEND_END_ARG_INFO()

ZEND_BEGIN_ARG_INFO_EX(arginfo_mydb_query, 0, 0, 2)
    ZEND_ARG_INFO(0, connection)
    ZEND_ARG_TYPE_INFO(0, query, IS_STRING, 0)
ZEND_END_ARG_INFO()

const zend_function_entry mydb_functions[] = {
    PHP_FE(mydb_connect, arginfo_mydb_connect)
    PHP_FE(mydb_query, arginfo_mydb_query)
    PHP_FE(mydb_fetch, NULL)
    PHP_FE(mydb_close, NULL)
    PHP_FE_END
};

zend_module_entry mydb_module_entry = {
    STANDARD_MODULE_HEADER,
    "mydb",
    mydb_functions,
    PHP_MINIT(mydb),
    PHP_MSHUTDOWN(mydb),
    NULL,
    NULL,
    PHP_MINFO(mydb),
    PHP_MYDB_VERSION,
    STANDARD_MODULE_PROPERTIES
};

#ifdef COMPILE_DL_MYDB
ZEND_GET_MODULE(mydb)
#endif

static void php_mydb_connection_dtor(zend_resource *rsrc)
{
    php_mydb_connection *conn = (php_mydb_connection *)rsrc->ptr;
    
    if (conn->is_connected) {
        // Close actual database connection
        // my_real_db_close(conn->connection);
        conn->is_connected = 0;
    }
    
    efree(conn);
}

PHP_MINIT_FUNCTION(mydb)
{
    le_mydb_connection = zend_register_list_destructors_ex(
        php_mydb_connection_dtor, NULL, 
        "MyDB Connection", module_number
    );
    
    return SUCCESS;
}

PHP_MSHUTDOWN_FUNCTION(mydb)
{
    return SUCCESS;
}

PHP_MINFO_FUNCTION(mydb)
{
    php_info_print_table_start();
    php_info_print_table_header(2, "MyDB Support", "enabled");
    php_info_print_table_row(2, "Version", PHP_MYDB_VERSION);
    php_info_print_table_end();
}

PHP_FUNCTION(mydb_connect)
{
    zend_string *host, *username, *password;
    php_mydb_connection *conn;
    
    ZEND_PARSE_PARAMETERS_START(3, 3)
        Z_PARAM_STR(host)
        Z_PARAM_STR(username)
        Z_PARAM_STR(password)
    ZEND_PARSE_PARAMETERS_END();
    
    conn = emalloc(sizeof(php_mydb_connection));
    
    // Simulate connection
    // conn->connection = my_real_db_connect(host, username, password);
    conn->is_connected = 1;
    
    RETURN_RES(zend_register_resource(conn, le_mydb_connection));
}

PHP_FUNCTION(mydb_query)
{
    zval *zconn;
    zend_string *query;
    php_mydb_connection *conn;
    
    ZEND_PARSE_PARAMETERS_START(2, 2)
        Z_PARAM_RESOURCE(zconn)
        Z_PARAM_STR(query)
    ZEND_PARSE_PARAMETERS_END();
    
    conn = (php_mydb_connection *)zend_fetch_resource(
        Z_RES_P(zconn), "MyDB Connection", le_mydb_connection
    );
    
    if (!conn->is_connected) {
        php_error_docref(NULL, E_WARNING, "Connection is closed");
        RETURN_FALSE;
    }
    
    // Execute query
    // result = my_real_db_query(conn->connection, ZSTR_VAL(query));
    
    RETURN_TRUE;
}

PHP_FUNCTION(mydb_close)
{
    zval *zconn;
    php_mydb_connection *conn;
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_RESOURCE(zconn)
    ZEND_PARSE_PARAMETERS_END();
    
    conn = (php_mydb_connection *)zend_fetch_resource(
        Z_RES_P(zconn), "MyDB Connection", le_mydb_connection
    );
    
    if (conn->is_connected) {
        // my_real_db_close(conn->connection);
        conn->is_connected = 0;
    }
    
    zend_list_close(Z_RES_P(zconn));
    RETURN_TRUE;
}
```

### 9.2 Object-Oriented Extension

**Creating a class in C:**

```c
// Class entry
zend_class_entry *myclass_ce;

// Object structure
typedef struct {
    zend_long value;
    zend_string *name;
    zend_object std;
} myclass_object;

// Get object from zend_object
static inline myclass_object *myclass_from_obj(zend_object *obj) {
    return (myclass_object *)((char *)(obj) - 
           XtOffsetOf(myclass_object, std));
}

#define Z_MYCLASS_P(zv) myclass_from_obj(Z_OBJ_P(zv))

// Object handlers
static zend_object_handlers myclass_object_handlers;

// Constructor
PHP_METHOD(MyClass, __construct)
{
    zend_string *name = NULL;
    zend_long value = 0;
    
    ZEND_PARSE_PARAMETERS_START(0, 2)
        Z_PARAM_OPTIONAL
        Z_PARAM_STR(name)
        Z_PARAM_LONG(value)
    ZEND_PARSE_PARAMETERS_END();
    
    myclass_object *intern = Z_MYCLASS_P(getThis());
    
    if (name) {
        intern->name = zend_string_copy(name);
    }
    intern->value = value;
}

// Getter method
PHP_METHOD(MyClass, getValue)
{
    ZEND_PARSE_PARAMETERS_NONE();
    
    myclass_object *intern = Z_MYCLASS_P(getThis());
    RETURN_LONG(intern->value);
}

// Setter method
PHP_METHOD(MyClass, setValue)
{
    zend_long value;
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_LONG(value)
    ZEND_PARSE_PARAMETERS_END();
    
    myclass_object *intern = Z_MYCLASS_P(getThis());
    intern->value = value;
}

// Create object handler
static zend_object *myclass_create_object(zend_class_entry *ce)
{
    myclass_object *intern = ecalloc(1, 
        sizeof(myclass_object) + zend_object_properties_size(ce));
    
    zend_object_std_init(&intern->std, ce);
    object_properties_init(&intern->std, ce);
    
    intern->std.handlers = &myclass_object_handlers;
    intern->value = 0;
    intern->name = NULL;
    
    return &intern->std;
}

// Free object handler
static void myclass_free_object(zend_object *object)
{
    myclass_object *intern = myclass_from_obj(object);
    
    if (intern->name) {
        zend_string_release(intern->name);
    }
    
    zend_object_std_dtor(&intern->std);
}

// Method entries
const zend_function_entry myclass_methods[] = {
    PHP_ME(MyClass, __construct, NULL, ZEND_ACC_PUBLIC)
    PHP_ME(MyClass, getValue, NULL, ZEND_ACC_PUBLIC)
    PHP_ME(MyClass, setValue, NULL, ZEND_ACC_PUBLIC)
    PHP_FE_END
};

// Register class
PHP_MINIT_FUNCTION(myclass)
{
    zend_class_entry ce;
    
    INIT_CLASS_ENTRY(ce, "MyClass", myclass_methods);
    myclass_ce = zend_register_internal_class(&ce);
    myclass_ce->create_object = myclass_create_object;
    
    memcpy(&myclass_object_handlers, 
           zend_get_std_object_handlers(), 
           sizeof(zend_object_handlers));
    myclass_object_handlers.free_obj = myclass_free_object;
    myclass_object_handlers.offset = XtOffsetOf(myclass_object, std);
    
    return SUCCESS;
}
```

**Using the class in PHP:**
```php
<?php
$obj = new MyClass("Test", 42);
echo $obj->getValue();  // 42
$obj->setValue(100);
echo $obj->getValue();  // 100
?>
```

---

## 10. PHP Internals API Reference

### 10.1 Core Macros

```c
// Success/Failure
#define SUCCESS 0
#define FAILURE -1

// Type operations
#define Z_TYPE(zval)         (zval).u1.v.type
#define Z_TYPE_P(zval_p)     Z_TYPE(*(zval_p))
#define Z_TYPE_PP(zval_pp)   Z_TYPE(**( zval_pp))

// Value access
#define Z_LVAL(zval)         (zval).value.lval
#define Z_LVAL_P(zval_p)     Z_LVAL(*(zval_p))
#define Z_DVAL(zval)         (zval).value.dval
#define Z_DVAL_P(zval_p)     Z_DVAL(*(zval_p))
#define Z_STR(zval)          (zval).value.str
#define Z_STR_P(zval_p)      Z_STR(*(zval_p))
#define Z_ARR(zval)          (zval).value.arr
#define Z_ARR_P(zval_p)      Z_ARR(*(zval_p))
#define Z_OBJ(zval)          (zval).value.obj
#define Z_OBJ_P(zval_p)      Z_OBJ(*(zval_p))
#define Z_RES(zval)          (zval).value.res
#define Z_RES_P(zval_p)      Z_RES(*(zval_p))

// String access
#define Z_STRVAL(zval)       ZSTR_VAL(Z_STR(zval))
#define Z_STRVAL_P(zval_p)   Z_STRVAL(*(zval_p))
#define Z_STRLEN(zval)       ZSTR_LEN(Z_STR(zval))
#define Z_STRLEN_P(zval_p)   Z_STRLEN(*(zval_p))

// Array operations
#define Z_ARRVAL(zval)       Z_ARR(zval)
#define Z_ARRVAL_P(zval_p)   Z_ARRVAL(*(zval_p))

// Initializers
#define ZVAL_NULL(z)              Z_TYPE_INFO_P(z) = IS_NULL
#define ZVAL_FALSE(z)             Z_TYPE_INFO_P(z) = IS_FALSE
#define ZVAL_TRUE(z)              Z_TYPE_INFO_P(z) = IS_TRUE
#define ZVAL_BOOL(z, b)           Z_TYPE_INFO_P(z) = (b) ? IS_TRUE : IS_FALSE
#define ZVAL_LONG(z, l)           do { \
        zval *__z = (z); \
        Z_LVAL_P(__z) = l; \
        Z_TYPE_INFO_P(__z) = IS_LONG; \
    } while (0)
#define ZVAL_DOUBLE(z, d)         do { \
        zval *__z = (z); \
        Z_DVAL_P(__z) = d; \
        Z_TYPE_INFO_P(__z) = IS_DOUBLE; \
    } while (0)
#define ZVAL_STRING(z, s)         do { \
        zval *__z = (z); \
        Z_STR_P(__z) = zend_string_init(s, strlen(s), 0); \
        Z_TYPE_INFO_P(__z) = IS_STRING; \
    } while (0)

// Reference counting
#define Z_REFCOUNT(zval)          zval_refcount_p(&(zval))
#define Z_REFCOUNT_P(zval_p)      Z_REFCOUNT(*(zval_p))
#define Z_ADDREF(zval)            zval_addref_p(&(zval))
#define Z_ADDREF_P(zval_p)        Z_ADDREF(*(zval_p))
#define Z_DELREF(zval)            zval_delref_p(&(zval))
#define Z_DELREF_P(zval_p)        Z_DELREF(*(zval_p))
```

### 10.2 Array Functions

```c
// Array initialization
int array_init(zval *arg);
int array_init_size(zval *arg, uint32_t size);

// Adding elements
int add_index_long(zval *arg, zend_ulong idx, zend_long n);
int add_index_double(zval *arg, zend_ulong idx, double d);
int add_index_string(zval *arg, zend_ulong idx, const char *str);
int add_index_stringl(zval *arg, zend_ulong idx, const char *str, size_t len);
int add_index_zval(zval *arg, zend_ulong idx, zval *value);

int add_next_index_long(zval *arg, zend_long n);
int add_next_index_string(zval *arg, const char *str);
int add_next_index_zval(zval *arg, zval *value);

int add_assoc_long(zval *arg, const char *key, zend_long n);
int add_assoc_string(zval *arg, const char *key, const char *str);
int add_assoc_zval(zval *arg, const char *key, zval *value);

// HashTable operations
int zend_hash_add(HashTable *ht, zend_string *key, zval *pData);
int zend_hash_update(HashTable *ht, zend_string *key, zval *pData);
zval *zend_hash_find(const HashTable *ht, zend_string *key);
int zend_hash_del(HashTable *ht, zend_string *key);
int zend_hash_exists(const HashTable *ht, zend_string *key);

// Iteration
#define ZEND_HASH_FOREACH_VAL(ht, _val) \
    ZEND_HASH_FOREACH(ht, 0); \
    _val = _z;

#define ZEND_HASH_FOREACH_KEY_VAL(ht, _h, _key, _val) \
    ZEND_HASH_FOREACH(ht, 0); \
    _h = _p->h; \
    _key = _p->key; \
    _val = _z;

#define ZEND_HASH_FOREACH_END() \
    } ZEND_HASH_FOREACH_END_DEL();
```

### 10.3 String Functions

```c
// String creation
zend_string *zend_string_init(const char *str, size_t len, int persistent);
zend_string *zend_string_alloc(size_t len, int persistent);
zend_string *zend_string_copy(zend_string *s);
zend_string *zend_string_dup(zend_string *s, int persistent);

// String operations
zend_string *zend_string_concat2(const char *str1, size_t str1_len,
                                  const char *str2, size_t str2_len);
int zend_string_equals(zend_string *s1, zend_string *s2);
int zend_string_equals_literal(zend_string *str, const char *literal);
zend_ulong zend_string_hash_val(zend_string *s);

// String release
void zend_string_release(zend_string *s);
void zend_string_free(zend_string *s);
```

### 10.4 Error Handling

```c
// Error reporting
void php_error_docref(const char *docref, int type, const char *format, ...);
void zend_error(int type, const char *format, ...);

// Error types
#define E_ERROR             (1<<0L)
#define E_WARNING           (1<<1L)
#define E_PARSE             (1<<2L)
#define E_NOTICE            (1<<3L)
#define E_CORE_ERROR        (1<<4L)
#define E_CORE_WARNING      (1<<5L)
#define E_COMPILE_ERROR     (1<<6L)
#define E_COMPILE_WARNING   (1<<7L)
#define E_USER_ERROR        (1<<8L)
#define E_USER_WARNING      (1<<9L)
#define E_USER_NOTICE       (1<<10L)
#define E_DEPRECATED        (1<<13L)
#define E_USER_DEPRECATED   (1<<14L)

// Example usage
if (some_error_condition) {
    php_error_docref(NULL, E_WARNING, "Something went wrong: %s", error_msg);
    RETURN_FALSE;
}
```

---

## 11. Advanced Topics

### 11.1 Opcode Generation and Optimization

```c
// Understanding opcodes
typedef struct _zend_op {
    const void *handler;
    znode_op op1;
    znode_op op2;
    znode_op result;
    uint32_t extended_value;
    uint32_t lineno;
    zend_uchar opcode;
    zend_uchar op1_type;
    zend_uchar op2_type;
    zend_uchar result_type;
} zend_op;

// Opcode types
#define ZEND_NOP                     0
#define ZEND_ADD                     1
#define ZEND_SUB                     2
#define ZEND_MUL                     3
#define ZEND_DIV                     4
#define ZEND_MOD                     5
#define ZEND_ECHO                    40
#define ZEND_ASSIGN                  38
#define ZEND_RETURN                  62

// Viewing opcodes
// Use: php -d opcache.opt_debug_level=0x10000 script.php
```

### 11.2 JIT Compilation (PHP 8+)

```c
// JIT configuration in php.ini
opcache.enable=1
opcache.jit_buffer_size=100M
opcache.jit=tracing  // or: function, on

// JIT modes:
// - 0: Disabled
// - tracing: Trace-based JIT
// - function: Function-based JIT
// - on: Default (tracing)

// Checking JIT status
PHP_FUNCTION(check_jit_status)
{
    array_init(return_value);
    
    zval *jit_enabled = zend_get_constant_str(
        "ZEND_JIT_ENABLED", sizeof("ZEND_JIT_ENABLED")-1
    );
    
    if (jit_enabled) {
        add_assoc_bool(return_value, "jit_enabled", Z_TYPE_P(jit_enabled) == IS_TRUE);
    }
}
```

### 11.3 Custom Stream Wrappers

```c
// Stream wrapper structure
typedef struct _php_stream_wrapper_ops {
    php_stream *(*stream_opener)(php_stream_wrapper *wrapper,
                                  const char *path, const char *mode,
                                  int options, zend_string **opened_path,
                                  php_stream_context *context);
    int (*stream_closer)(php_stream_wrapper *wrapper, php_stream *stream);
    int (*stream_stat)(php_stream_wrapper *wrapper, php_stream *stream,
                       php_stream_statbuf *ssb);
    int (*url_stat)(php_stream_wrapper *wrapper, const char *url,
                    int flags, php_stream_statbuf *ssb,
                    php_stream_context *context);
    php_stream *(*dir_opener)(php_stream_wrapper *wrapper,
                              const char *path, const char *mode,
                              int options, zend_string **opened_path,
                              php_stream_context *context);
    const char *label;
    int (*unlink)(php_stream_wrapper *wrapper, const char *url,
                  int options, php_stream_context *context);
    int (*rename)(php_stream_wrapper *wrapper, const char *url_from,
                  const char *url_to, int options,
                  php_stream_context *context);
    int (*stream_mkdir)(php_stream_wrapper *wrapper, const char *url,
                        int mode, int options, php_stream_context *context);
    int (*stream_rmdir)(php_stream_wrapper *wrapper, const char *url,
                        int options, php_stream_context *context);
    int (*stream_metadata)(php_stream_wrapper *wrapper, const char *url,
                          int option, void *value,
                          php_stream_context *context);
} php_stream_wrapper_ops;

// Example: Memory stream wrapper
static php_stream *memory_stream_opener(
    php_stream_wrapper *wrapper, const char *path,
    const char *mode, int options, zend_string **opened_path,
    php_stream_context *context)
{
    php_stream *stream;
    
    stream = php_stream_alloc(&memory_stream_ops, NULL, 0, mode);
    if (stream == NULL) {
        return NULL;
    }
    
    return stream;
}

static php_stream_wrapper_ops memory_stream_wops = {
    memory_stream_opener,
    NULL, /* close */
    NULL, /* stat */
    NULL, /* url_stat */
    NULL, /* dir_opener */
    "memory stream",
    NULL, /* unlink */
    NULL, /* rename */
    NULL, /* mkdir */
    NULL, /* rmdir */
    NULL  /* metadata */
};

php_stream_wrapper memory_stream_wrapper = {
    &memory_stream_wops,
    NULL,
    0
};

// Register wrapper
PHP_MINIT_FUNCTION(memory_stream)
{
    return php_register_url_stream_wrapper("memory", 
                                           &memory_stream_wrapper);
}
```

### 11.4 Persistent Resources

```c
// Persistent connections example
typedef struct {
    void *connection;
    time_t last_used;
    int in_use;
} persistent_connection;

static HashTable persistent_connections;

PHP_MINIT_FUNCTION(persistent_example)
{
    // Initialize persistent hash table
    zend_hash_init(&persistent_connections, 0, NULL,
                   persistent_connection_dtor, 1);  // 1 = persistent
    return SUCCESS;
}

PHP_MSHUTDOWN_FUNCTION(persistent_example)
{
    zend_hash_destroy(&persistent_connections);
    return SUCCESS;
}

PHP_FUNCTION(get_persistent_connection)
{
    zend_string *host;
    persistent_connection *pconn;
    zval *found;
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_STR(host)
    ZEND_PARSE_PARAMETERS_END();
    
    // Look for existing connection
    found = zend_hash_find(&persistent_connections, host);
    if (found) {
        pconn = (persistent_connection *)Z_PTR_P(found);
        if (!pconn->in_use) {
            pconn->in_use = 1;
            pconn->last_used = time(NULL);
            // Return existing connection
            RETURN_TRUE;
        }
    }
    
    // Create new persistent connection
    pconn = pemalloc(sizeof(persistent_connection), 1);
    // pconn->connection = create_real_connection(ZSTR_VAL(host));
    pconn->in_use = 1;
    pconn->last_used = time(NULL);
    
    zval tmp;
    ZVAL_PTR(&tmp, pconn);
    zend_hash_add(&persistent_connections, host, &tmp);
    
    RETURN_TRUE;
}
```

### 11.5 Thread Safety (ZTS)

```c
// Thread-safe resource manager
#ifdef ZTS
#define MYEXT_G(v) TSRMG(myext_globals_id, zend_myext_globals *, v)
extern int myext_globals_id;
#else
#define MYEXT_G(v) (myext_globals.v)
extern zend_myext_globals myext_globals;
#endif

// Declaring globals
ZEND_BEGIN_MODULE_GLOBALS(myext)
    zend_long counter;
    HashTable *cache;
ZEND_END_MODULE_GLOBALS(myext)

// In .c file
#ifdef ZTS
int myext_globals_id;
#else
zend_myext_globals myext_globals;
#endif

// Initialize globals
static void php_myext_init_globals(zend_myext_globals *myext_globals)
{
    myext_globals->counter = 0;
    myext_globals->cache = NULL;
}

PHP_MINIT_FUNCTION(myext)
{
#ifdef ZTS
    ts_allocate_id(&myext_globals_id,
                   sizeof(zend_myext_globals),
                   (ts_allocate_ctor) php_myext_init_globals,
                   NULL);
#else
    php_myext_init_globals(&myext_globals);
#endif
    return SUCCESS;
}
```

---

## 12. Contributing to PHP Core

### 12.1 Development Workflow

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/php-src.git
cd php-src

# Add upstream
git remote add upstream https://github.com/php/php-src.git

# Create feature branch
git checkout -b fix-issue-12345

# Make changes and commit
git add .
git commit -m "Fix issue #12345: Description of fix"

# Push and create PR
git push origin fix-issue-12345
```

### 12.2 Coding Standards

```c
// PHP coding standards

// 1. Indentation: Use tabs (4 spaces width)
void my_function(int param)
{
    if (param > 0) {
        // Code here
    }
}

// 2. Braces: K&R style
if (condition) {
    // code
} else {
    // code
}

// 3. Naming conventions
// - Functions: lowercase with underscores
void php_my_function(void);

// - Macros: UPPERCASE with underscores
#define MY_CONSTANT 42

// - Types: lowercase with underscores
typedef struct _my_struct {
    int field;
} my_struct;

// 4. Comments: C-style for multi-line
/*
 * This is a multi-line comment
 * explaining complex logic
 */

// C++-style for single line
// This is a simple comment

// 5. Function declarations
PHPAPI void php_my_function(
    const char *param1,
    size_t param1_len,
    zend_long param2
);

// 6. Error messages
php_error_docref(NULL, E_WARNING, 
                "Invalid parameter: expected %s, got %s",
                expected_type, actual_type);
```

### 12.3 Writing Tests

**PHPT Test Format:**
```
--TEST--
Test myextension_add function
--SKIPIF--
<?php if (!extension_loaded("myextension")) print "skip"; ?>
--FILE--
<?php
$result = myextension_add(5, 7);
var_dump($result);

$result = myextension_add(-10, 10);
var_dump($result);

$result = myextension_add(0, 0);
var_dump($result);
?>
--EXPECT--
int(12)
int(0)
int(0)
```

**Running tests:**
```bash
# Run all tests
make test

# Run specific test
php run-tests.php ext/myextension/tests/

# Run single test file
php run-tests.php ext/myextension/tests/add_function.phpt

# Generate test file
php run-tests.php --generate-diff ext/myextension/tests/
```

### 12.4 Documentation

```c
// Function documentation (for PHP manual)
/* {{{ proto int myextension_add(int a, int b)
   Add two integers and return the result */
PHP_FUNCTION(myextension_add)
{
    zend_long a, b;
    
    ZEND_PARSE_PARAMETERS_START(2, 2)
        Z_PARAM_LONG(a)
        Z_PARAM_LONG(b)
    ZEND_PARSE_PARAMETERS_END();
    
    RETURN_LONG(a + b);
}
/* }}} */
```

### 12.5 RFC Process

For significant changes:

1. **Discuss on internals mailing list**
   - php-internals@lists.php.net
   - Present idea and gather feedback

2. **Create RFC**
   - Write detailed proposal on wiki.php.net
   - Include motivation, implementation details, examples

3. **Implementation**
   - Create working patch/PR
   - Include tests and documentation

4. **Voting**
   - RFC must be open for at least 2 weeks before vote
   - Voting period: minimum 2 weeks
   - Requires 2/3 majority for language changes

---

## 13. Debugging and Testing

### 13.1 Debugging Tools

**GDB with PHP:**
```bash
# Compile with debug symbols
./configure --enable-debug

# Run with GDB
gdb --args php script.php

# Common GDB commands for PHP
(gdb) break zend_execute
(gdb) break php_my_function
(gdb) print *executor_globals.current_execute_data
(gdb) print (char*)Z_STRVAL_P(var)
```

**Useful GDB Macros (.gdbinit):**
```gdb
define zbacktrace
    set $ex = executor_globals.current_execute_data
    while $ex
        printf "%s() %s:%d\n", \
            $ex->func->common.function_name->val, \
            $ex->func->op_array.filename->val, \
            $ex->opline->lineno
        set $ex = $ex->prev_execute_data
    end
end

define printzv
    set $zv = (zval *)$arg0
    if $zv->u1.v.type == 1
        printf "NULL\n"
    end
    if $zv->u1.v.type == 4
        printf "long: %ld\n", $zv->value.lval
    end
    if $zv->u1.v.type == 5
        printf "double: %f\n", $zv->value.dval
    end
    if $zv->u1.v.type == 6
        printf "string: %s\n", $zv->value.str->val
    end
end
```

### 13.2 Valgrind

```bash
# Memory leak detection
valgrind --leak-check=full \
         --show-leak-kinds=all \
         --track-origins=yes \
         php script.php

# For extensions
valgrind --leak-check=full \
         php -dextension=myextension.so \
         test_script.php
```

### 13.3 AddressSanitizer

```bash
# Build with ASAN
./configure --enable-debug \
            --enable-address-sanitizer \
            --disable-opcache-jit

make

# Run
ASAN_OPTIONS=detect_leaks=1 php script.php
```

### 13.4 Logging and Tracing

```c
// Debug logging
#ifdef PHP_DEBUG
    php_printf("DEBUG: variable value = %ld\n", value);
#endif

// Custom logging function
void my_debug_log(const char *format, ...)
{
    va_list args;
    FILE *fp;
    
    fp = fopen("/tmp/php_debug.log", "a");
    if (!fp) return;
    
    va_start(args, format);
    vfprintf(fp, format, args);
    va_end(args);
    
    fprintf(fp, "\n");
    fclose(fp);
}

// Usage
my_debug_log("Function called with params: %s, %ld", str_param, int_param);
```

### 13.5 Performance Profiling

**Using XDebug:**
```bash
# Install xdebug
pecl install xdebug

# php.ini
zend_extension=xdebug.so
xdebug.mode=profile
xdebug.output_dir=/tmp/xdebug
xdebug.profiler_output_name=cachegrind.out.%p

# Analyze with KCachegrind
kcachegrind /tmp/xdebug/cachegrind.out.12345
```

**Using perf (Linux):**
```bash
# Record
perf record -g php script.php

# Report
perf report

# Annotate specific function
perf annotate php_my_function
```

---

## 14. Best Practices

### 14.1 Memory Management

```c
// ✅ Good: Use emalloc for request-bound memory
char *buffer = emalloc(1024);
// ... use buffer ...
// No need to free (auto-freed at request end)

// ✅ Better: Free explicitly for large allocations
char *large_buffer = emalloc(1024 * 1024);
// ... use large_buffer ...
efree(large_buffer);  // Don't wait for request end

// ✅ Good: Use pemalloc for persistent memory
char *persistent = pemalloc(1024, 1);
// ... use persistent across requests ...
pefree(persistent, 1);  // Must free manually

// ❌ Bad: Memory leak
char *leak = emalloc(1024);
leak = emalloc(1024);  // Previous allocation leaked

// ✅ Good: Reuse or free first
char *buffer = emalloc(1024);
// ... use buffer ...
efree(buffer);
buffer = emalloc(2048);
```

### 14.2 Error Handling

```c
// ✅ Good: Check return values
zval *found = zend_hash_find(ht, key);
if (!found) {
    php_error_docref(NULL, E_WARNING, "Key not found");
    RETURN_FALSE;
}

// ✅ Good: Validate parameters
PHP_FUNCTION(divide)
{
    double a, b;
    
    ZEND_PARSE_PARAMETERS_START(2, 2)
        Z_PARAM_DOUBLE(a)
        Z_PARAM_DOUBLE(b)
    ZEND_PARSE_PARAMETERS_END();
    
    if (b == 0.0) {
        php_error_docref(NULL, E_WARNING, "Division by zero");
        RETURN_FALSE;
    }
    
    RETURN_DOUBLE(a / b);
}

// ❌ Bad: No error checking
void *ptr = emalloc(size);
memcpy(ptr, data, size);  // What if emalloc failed?
```

### 14.3 Thread Safety

```c
// ✅ Good: Use globals properly
PHP_FUNCTION(increment_counter)
{
    MYEXT_G(counter)++;
    RETURN_LONG(MYEXT_G(counter));
}

// ❌ Bad: Global variable (not thread-safe)
static long counter = 0;

PHP_FUNCTION(bad_increment)
{
    counter++;  // Race condition in ZTS
    RETURN_LONG(counter);
}

// ✅ Good: Request-local storage
ZEND_BEGIN_MODULE_GLOBALS(myext)
    HashTable *request_cache;
ZEND_END_MODULE_GLOBALS(myext)

PHP_RINIT_FUNCTION(myext)
{
    ALLOC_HASHTABLE(MYEXT_G(request_cache));
    zend_hash_init(MYEXT_G(request_cache), 0, NULL, ZVAL_PTR_DTOR, 0);
    return SUCCESS;
}
```

### 14.4 API Usage

```c
// ✅ Good: Use correct API version
#if PHP_VERSION_ID >= 80000
    // PHP 8.0+ code
    zend_string *result = zend_string_concat3(
        ZSTR_VAL(str1), ZSTR_LEN(str1),
        " ",  1,
        ZSTR_VAL(str2), ZSTR_LEN(str2)
    );
#else
    // PHP 7.x code
    zend_string *result = zend_string_concat2(
        ZSTR_VAL(str1), ZSTR_LEN(str1),
        ZSTR_VAL(str2), ZSTR_LEN(str2)
    );
#endif

// ✅ Good: Check extension availability
#ifdef HAVE_JSON
    // Use JSON API
#else
    php_error_docref(NULL, E_WARNING, "JSON extension required");
#endif
```

### 14.5 Performance Optimization

```c
// ✅ Good: Cache expensive lookups
static zend_class_entry *cached_ce = NULL;

void get_my_class_entry(void)
{
    if (!cached_ce) {
        cached_ce = zend_lookup_class(
            zend_string_init("MyClass", sizeof("MyClass")-1, 0)
        );
    }
    return cached_ce;
}

// ✅ Good: Minimize allocations
// Instead of multiple small allocations:
char *a = emalloc(10);
char *b = emalloc(10);
char *c = emalloc(10);

// Use single allocation:
char *buffer = emalloc(30);
char *a = buffer;
char *b = buffer + 10;
char *c = buffer + 20;

// ✅ Good: Use appropriate data structures
// For small fixed sets: array
// For large dynamic sets: HashTable
// For ordered data: linked list or array

// ❌ Bad: Unnecessary conversions
convert_to_string(val);
convert_to_long(val);  // Lost string data

// ✅ Good: Check type first
if (Z_TYPE_P(val) != IS_LONG) {
    convert_to_long(val);
}
```

### 14.6 Security Considerations

```c
// ✅ Good: Validate input lengths
PHP_FUNCTION(process_data)
{
    zend_string *data;
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_STR(data)
    ZEND_PARSE_PARAMETERS_END();
    
    if (ZSTR_LEN(data) > MAX_ALLOWED_SIZE) {
        php_error_docref(NULL, E_WARNING, "Data too large");
        RETURN_FALSE;
    }
    
    // Process data...
}

// ✅ Good: Avoid buffer overflows
char buffer[256];
snprintf(buffer, sizeof(buffer), "Value: %s", user_input);

// ❌ Bad: Potential buffer overflow
char buffer[256];
sprintf(buffer, "Value: %s", user_input);

// ✅ Good: Sanitize file paths
PHP_FUNCTION(read_file)
{
    zend_string *filename;
    char resolved_path[MAXPATHLEN];
    
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_STR(filename)
    ZEND_PARSE_PARAMETERS_END();
    
    // Resolve and validate path
    if (!VCWD_REALPATH(ZSTR_VAL(filename), resolved_path)) {
        php_error_docref(NULL, E_WARNING, "Invalid path");
        RETURN_FALSE;
    }
    
    // Check if path is within allowed directory
    if (strncmp(resolved_path, "/allowed/path/", 14) != 0) {
        php_error_docref(NULL, E_WARNING, "Access denied");
        RETURN_FALSE;
    }
    
    // Safe to proceed...
}
```

---

## Conclusion

This comprehensive guide covers PHP internals from a C programmer's perspective. Key takeaways:

1. **PHP is C-based**: Understanding C is essential for PHP internals
2. **Zend Engine**: Core execution engine handling compilation and execution
3. **Memory Management**: Use emalloc/efree for request memory, pemalloc/pefree for persistent
4. **Extensions**: Modular way to extend PHP functionality
5. **Thread Safety**: Always consider ZTS when writing extensions
6. **Testing**: Write comprehensive tests for all functionality
7. **Documentation**: Document your code for others
8. **Performance**: Profile and optimize critical paths
9. **Security**: Always validate input and handle errors properly
10. **Community**: Engage with PHP internals community

### Resources

- **Official Documentation**: https://www.php.net/manual/en/internals2.php
- **PHP Internals Book**: https://www.phpinternalsbook.com/
- **Source Code**: https://github.com/php/php-src
- **Mailing List**: internals@lists.php.net
- **RFC Process**: https://wiki.php.net/rfc

### Next Steps

1. Set up a development environment
2. Build PHP from source
3. Create a simple extension
4. Study existing extensions (ext/standard, ext/json)
5. Contribute to PHP core or create your own extensions
6. Join the PHP internals community

Happy hacking!
