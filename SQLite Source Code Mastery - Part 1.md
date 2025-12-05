# SQLite Source Code Mastery Tutorial
## Deep Dive into SQLite's Architecture and Implementation

---

## Table of Contents - Part 1
1. [Introduction to SQLite](#1-introduction-to-sqlite)
2. [Getting the Source Code](#2-getting-the-source-code)
3. [SQLite Architecture Overview](#3-sqlite-architecture-overview)
4. [Core Components](#4-core-components)
5. [Building SQLite from Source](#5-building-sqlite-from-source)
6. [The Amalgamation File](#6-the-amalgamation-file)

---

## 1. Introduction to SQLite

### What Makes SQLite Special?

SQLite is the most widely deployed database engine in the world:
- **Embedded**: No separate server process
- **Zero Configuration**: No setup or administration needed
- **Self-Contained**: Single file database
- **Transactional**: ACID compliant
- **Public Domain**: Free for any use

**Usage Statistics:**
```
- Used by billions of devices
- ~1 trillion active databases
- In every Android/iOS device
- In all major web browsers
- In macOS, Windows, Linux systems
```

### Why Study SQLite Source Code?

1. **Excellent Code Quality**: Clean, well-documented C code
2. **Complete System**: See how a full DBMS works
3. **Real-World Application**: Production-grade software
4. **Educational Value**: Learn data structures, algorithms, OS concepts
5. **Manageable Size**: ~150K lines vs millions in other databases

---

## 2. Getting the Source Code

### Download SQLite

```bash
# Official SQLite source
wget https://www.sqlite.org/2024/sqlite-amalgamation-3450000.zip
unzip sqlite-amalgamation-3450000.zip

# Or clone the fossil repository
fossil clone https://www.sqlite.org/src sqlite.fossil
mkdir sqlite
cd sqlite
fossil open ../sqlite.fossil

# Or use GitHub mirror
git clone https://github.com/sqlite/sqlite.git
```

### Directory Structure

```
sqlite/
├── src/           # Core source files
├── ext/           # Extensions
├── test/          # Test suite (extensive!)
├── tool/          # Build tools
├── doc/           # Documentation
└── amalgamation/  # Single-file distribution

Key Source Files:
src/
├── btree.c        # B-tree implementation
├── pager.c        # Page cache
├── os_unix.c      # OS interface (Unix)
├── os_win.c       # OS interface (Windows)
├── vdbe.c         # Virtual machine
├── parse.y        # SQL parser (yacc)
├── select.c       # SELECT statement
├── insert.c       # INSERT statement
├── update.c       # UPDATE statement
├── delete.c       # DELETE statement
└── main.c         # Main entry points
```

---

## 3. SQLite Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    SQL Interface                        │
│  sqlite3_open(), sqlite3_exec(), sqlite3_prepare()     │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│                  Tokenizer & Parser                     │
│  Input: SQL Text → Output: Parse Tree                  │
│  Files: tokenize.c, parse.y                            │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│                   Code Generator                        │
│  Parse Tree → Bytecode (VDBE opcodes)                  │
│  Files: select.c, insert.c, update.c, delete.c        │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│            Virtual Database Engine (VDBE)               │
│  Executes bytecode instructions                         │
│  Files: vdbe.c, vdbeaux.c, vdbeapi.c                   │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│                   B-Tree Layer                          │
│  Table & Index storage using B+ trees                  │
│  Files: btree.c                                        │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│                    Pager Layer                          │
│  Page cache, transaction, locking                       │
│  Files: pager.c                                        │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│                  OS Interface (VFS)                     │
│  File I/O, locking, memory allocation                  │
│  Files: os_unix.c, os_win.c, mem*.c                    │
└─────────────────────────────────────────────────────────┘
```

### Data Flow Example

**Query: `SELECT name FROM users WHERE age > 25`**

```
1. Tokenizer:
   SELECT → TK_SELECT
   name → TK_ID
   FROM → TK_FROM
   users → TK_ID
   WHERE → TK_WHERE
   age > 25 → TK_ID, TK_GT, TK_INTEGER

2. Parser:
   Creates parse tree (AST)
   
3. Code Generator:
   Generates VDBE bytecode:
   OpenRead 0 2 0    # Open cursor on table users
   Rewind 0 10       # Start at beginning
   Column 0 1        # Get age column
   Integer 25        # Push 25
   Gt                # age > 25?
   IfNot 8           # If false, skip
   Column 0 0        # Get name column
   ResultRow         # Output row
   Next 0 3          # Next record
   Close 0           # Close cursor
   Halt              # Done

4. VDBE:
   Executes bytecode, calls B-tree layer

5. B-tree:
   Reads pages, traverses tree structure

6. Pager:
   Manages page cache, reads from disk

7. OS Layer:
   Performs actual file I/O
```

---

## 4. Core Components

### 4.1 The SQLite Database Handle

**Structure: `sqlite3`** (main.c)

```c
struct sqlite3 {
  sqlite3_vfs *pVfs;           // OS interface
  Db *aDb;                     // Attached databases
  int nDb;                     // Number of databases
  u32 mDbFlags;                // Flags
  u64 flags;                   // Connection flags
  int errCode;                 // Last error code
  int errMask;                 // Error mask
  u16 dbOptFlags;              // Optimization flags
  u8 autoCommit;               // Auto-commit flag
  u8 temp_store;               // Temp storage location
  
  // Memory allocation
  sqlite3_mutex *mutex;        // Connection mutex
  sqlite3_mem_methods *mem;    // Memory allocator
  
  // Virtual table
  Hash aModule;                // Virtual table modules
  
  // Prepared statements
  Vdbe *pVdbe;                 // List of VDBEs
  
  // B-tree and pager
  int nChange;                 // Rows changed
  int nTotalChange;            // Total rows changed
  
  // Function/collation registration
  Hash aFunc;                  // User functions
  Hash aCollSeq;               // Collating sequences
  
  // Extensions
  void **aExtension;           // Loaded extensions
};
```

**Key Functions:**

```c
// Opening a database
int sqlite3_open(
  const char *filename,   // Database filename (UTF-8)
  sqlite3 **ppDb          // OUT: SQLite db handle
);

// Implementation walkthrough:
int sqlite3_open(const char *zFilename, sqlite3 **ppDb){
  // 1. Allocate sqlite3 structure
  sqlite3 *db = sqlite3MallocZero(sizeof(sqlite3));
  
  // 2. Initialize mutex
  db->mutex = sqlite3MutexAlloc(SQLITE_MUTEX_RECURSIVE);
  
  // 3. Set default values
  db->errMask = 0xff;
  db->nDb = 2;  // main + temp
  db->flags = SQLITE_ShortColNames | SQLITE_EnableTrigger;
  
  // 4. Register built-in functions
  sqlite3RegisterBuiltinFunctions(db);
  
  // 5. Register built-in collating sequences
  sqlite3RegisterDateTimeFunctions();
  
  // 6. Open the database file
  rc = sqlite3BtreeOpen(db->pVfs, zFilename, db, &db->aDb[0].pBt, 0, flags);
  
  // 7. Set page size and encoding
  sqlite3BtreeSetPageSize(db->aDb[0].pBt, SQLITE_DEFAULT_PAGE_SIZE, -1, 0);
  
  // 8. Return handle
  *ppDb = db;
  return SQLITE_OK;
}
```

### 4.2 Prepared Statements (VDBE)

**Structure: `Vdbe`** (vdbeInt.h)

```c
struct Vdbe {
  sqlite3 *db;              // Database connection
  Vdbe *pPrev, *pNext;      // Linked list of VDBEs
  
  // Program and execution
  Op *aOp;                  // Array of opcodes
  int nOp;                  // Number of opcodes
  int pc;                   // Program counter
  
  // Memory and registers
  Mem *aMem;                // Memory cells (registers)
  int nMem;                 // Number of memory cells
  
  // Cursors
  VdbeCursor **apCsr;       // Array of cursors
  int nCursor;              // Number of cursors
  
  // Results
  Mem *pResultSet;          // Result set
  int nResColumn;           // Number of result columns
  
  // Execution state
  u8 explain;               // Explain mode
  u8 expired;               // Statement expired
  u8 runOnlyOnce;           // Run only once flag
  
  // Performance
  u64 nFkConstraint;        // Foreign key constraints
  u64 nStmtDefCons;         // Statement deferred constraints
};
```

**VDBE Opcodes Example:**

```c
// Opcode structure
typedef struct VdbeOp {
  u8 opcode;          // Opcode number
  signed char p4type; // Parameter 4 type
  u16 p5;            // Parameter 5 (flags)
  int p1;            // Parameter 1
  int p2;            // Parameter 2
  int p3;            // Parameter 3
  union {
    int i;           // Integer value
    void *p;         // Pointer value
    char *z;         // String value
  } p4;              // Parameter 4
} VdbeOp;

// Example: SELECT * FROM users WHERE id = 5
VdbeOp ops[] = {
  {OP_Init,         0,  9, 0},         // Initialize
  {OP_OpenRead,     0,  2, 0},         // Open cursor on table users
  {OP_Integer,      5,  1, 0},         // Push 5 into register 1
  {OP_Rewind,       0,  8, 0},         // Go to first record
  {OP_Column,       0,  0, 2},         // Get id column into register 2
  {OP_Eq,           1,  7, 2},         // Compare id == 5
  {OP_Next,         0,  4, 0},         // Go to next record
  {OP_ResultRow,    2,  3, 0},         // Output result
  {OP_Close,        0,  0, 0},         // Close cursor
  {OP_Halt,         0,  0, 0}          // End program
};
```

**Preparing a Statement:**

```c
int sqlite3_prepare_v2(
  sqlite3 *db,            // Database handle
  const char *zSql,       // SQL statement (UTF-8)
  int nByte,              // Length of zSql in bytes
  sqlite3_stmt **ppStmt,  // OUT: Statement handle
  const char **pzTail     // OUT: Pointer to unused portion of zSql
);

// Implementation flow:
// 1. Tokenize SQL
// 2. Parse SQL into parse tree
// 3. Generate VDBE bytecode
// 4. Optimize bytecode
// 5. Return prepared statement
```

### 4.3 B-Tree Structure

**B-Tree Node Layout:**

```
Page Structure (default 4096 bytes):

┌────────────────────────────────────────┐
│  Page Header (8 or 12 bytes)          │
│  - Page type (leaf/interior)          │
│  - First freeblock                    │
│  - Number of cells                    │
│  - Cell content area offset           │
│  - Fragmented free bytes              │
├────────────────────────────────────────┤
│  Cell Pointer Array                    │
│  - 2 bytes per cell                   │
│  - Points to cell location            │
├────────────────────────────────────────┤
│  Unallocated Space                     │
├────────────────────────────────────────┤
│  Cell Content Area                     │
│  - Cells grow from end of page        │
│  - Each cell contains:                │
│    * Payload size (varint)            │
│    * Rowid (varint, for table)        │
│    * Payload data                     │
│    * Overflow page number (if needed) │
└────────────────────────────────────────┘
```

**B-Tree Operations:**

```c
// Search for a key
int sqlite3BtreeMovetoUnpacked(
  BtCursor *pCur,      // Cursor to use
  UnpackedRecord *pIdxKey,  // Key to search for
  i64 intKey,          // Integer key (for tables)
  int bias,            // Search bias
  int *pRes            // Search result
);

// Insert a new entry
int sqlite3BtreeInsert(
  BtCursor *pCur,           // Cursor pointing to insertion point
  const void *pKey,         // Key data
  i64 nKey,                 // Key length
  const void *pData,        // Data
  int nData,                // Data length
  int nZero,                // Extra zeros to append
  int appendBias,           // Append hint
  int seekResult            // Result of prior seek
);

// Delete an entry
int sqlite3BtreeDelete(BtCursor *pCur, u8 flags);
```

---

## 5. Building SQLite from Source

### 5.1 Simple Build

**Using Amalgamation (Easiest):**

```bash
# Download amalgamation
wget https://www.sqlite.org/2024/sqlite-amalgamation-3450000.zip
unzip sqlite-amalgamation-3450000.zip
cd sqlite-amalgamation-3450000

# Compile (basic)
gcc -o sqlite3 shell.c sqlite3.c -lpthread -ldl

# Compile with optimizations
gcc -o sqlite3 shell.c sqlite3.c \
    -lpthread -ldl \
    -O3 \
    -DSQLITE_ENABLE_FTS5 \
    -DSQLITE_ENABLE_JSON1 \
    -DSQLITE_ENABLE_RTREE

# Run
./sqlite3 test.db
```

### 5.2 Configure Build (Full Source)

```bash
# Clone repository
fossil clone https://www.sqlite.org/src sqlite.fossil
mkdir sqlite-src
cd sqlite-src
fossil open ../sqlite.fossil

# Configure
./configure --prefix=/usr/local

# Common configure options:
./configure \
    --enable-debug \
    --enable-fts5 \
    --enable-json1 \
    --enable-rtree \
    --enable-session

# Build
make

# Test (recommended - very extensive test suite)
make test

# Install
sudo make install
```

### 5.3 Custom Build Options

**Compile-Time Options (sqlite3.c):**

```c
// Performance
#define SQLITE_DEFAULT_CACHE_SIZE -2000  // 2MB cache
#define SQLITE_DEFAULT_PAGE_SIZE 4096
#define SQLITE_MAX_PAGE_SIZE 65536

// Features
#define SQLITE_ENABLE_FTS5          // Full-text search
#define SQLITE_ENABLE_JSON1         // JSON functions
#define SQLITE_ENABLE_RTREE         // R-Tree index
#define SQLITE_ENABLE_STAT4         // Better query planning
#define SQLITE_ENABLE_DBPAGE_VTAB   // Access raw pages

// Security
#define SQLITE_OMIT_LOAD_EXTENSION  // Disable extension loading
#define SQLITE_SECURE_DELETE        // Overwrite deleted data

// Debugging
#define SQLITE_DEBUG                // Enable debugging
#define SQLITE_ENABLE_EXPLAIN_COMMENTS  // Comments in EXPLAIN
```

**Example Custom Build:**

```bash
gcc -o sqlite3-custom shell.c sqlite3.c \
    -lpthread -ldl -lm \
    -O3 \
    -DSQLITE_ENABLE_FTS5 \
    -DSQLITE_ENABLE_JSON1 \
    -DSQLITE_ENABLE_RTREE \
    -DSQLITE_ENABLE_STAT4 \
    -DSQLITE_ENABLE_DBPAGE_VTAB \
    -DSQLITE_DEFAULT_CACHE_SIZE=-8000 \
    -DSQLITE_MAX_MMAP_SIZE=268435456
```

---

## 6. The Amalgamation File

### What is the Amalgamation?

The amalgamation is a single C file containing all SQLite source code:
- **sqlite3.c**: ~250,000 lines of C code
- **sqlite3.h**: Public header file
- **shell.c**: Command-line shell

**Advantages:**
- Easy to integrate into projects
- Single compilation unit (faster compilation)
- Better compiler optimizations
- Simpler distribution

**How It's Created:**

```bash
# From fossil repository
cd sqlite-src
./configure
make sqlite3.c

# The tool that creates it
tclsh tool/mksqlite3c.tcl
```

**Structure of sqlite3.c:**

```c
/************** Begin file sqliteInt.h ***************************************/
// Internal definitions

/************** Begin file os.h *********************************************/
// OS interface

/************** Begin file btree.h ******************************************/
// B-tree interface

// ... many more files ...

/************** Begin file main.c *******************************************/
// Main implementation

/************** Begin file pager.c ******************************************/
// Pager implementation

// ... and so on ...
```

---

## CONTINUED IN PART 2

This is Part 1 of the SQLite Source Code Mastery Tutorial. It covers:
- Introduction and architecture overview
- Getting and building the source code
- Core data structures (sqlite3, Vdbe, B-tree)
- Understanding the amalgamation

**Part 2 will cover:**
- Deep dive into the Parser and Code Generator
- VDBE execution engine internals
- B-Tree implementation details
- Pager and cache management
- Transaction and locking mechanisms

Save this file and continue with Part 2!