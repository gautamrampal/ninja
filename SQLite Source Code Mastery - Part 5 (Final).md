# SQLite Source Code Mastery Tutorial - PART 5 (FINAL)
## Statistics, Virtual Tables & Mastery

---

## Table of Contents - Part 5
1. [ANALYZE and Statistics](#1-analyze-and-statistics)
2. [Virtual Tables](#2-virtual-tables)
3. [Extensions and Custom Functions](#3-extensions-and-custom-functions)
4. [Performance Tuning](#4-performance-tuning)
5. [Reading SQLite Source Effectively](#5-reading-sqlite-source-effectively)
6. [Contributing to SQLite](#6-contributing-to-sqlite)
7. [Conclusion and Next Steps](#7-conclusion-and-next-steps)

---

## 1. ANALYZE and Statistics

### 1.1 Statistics Tables

**sqlite_stat1 - Index Statistics:**

```sql
CREATE TABLE sqlite_stat1(
  tbl TEXT,      -- Table name
  idx TEXT,      -- Index name (or NULL for table)
  stat TEXT      -- Statistics string
);

-- Example data:
-- tbl='users', idx='idx_users_age', stat='1000 10'
-- Meaning: 1000 rows total, average 10 rows per distinct age value
```

**sqlite_stat4 - Detailed Statistics:**

```sql
CREATE TABLE sqlite_stat4(
  tbl TEXT,      -- Table name
  idx TEXT,      -- Index name
  neq TEXT,      -- Number of rows with same key
  nlt TEXT,      -- Number of rows with lesser key
  ndlt TEXT,     -- Number of distinct keys less than
  sample BLOB    -- Sample row
);

-- Stores histogram samples for better estimates
```

### 1.2 ANALYZE Implementation

**Running ANALYZE:**

```c
// analyze.c
void sqlite3Analyze(Parse *pParse, Token *pName1, Token *pName2){
  sqlite3 *db = pParse->db;
  
  // For each table
  for(Table *pTab = db->pSchema->tblHash; pTab; pTab = pTab->pNext){
    
    // Count rows in table
    sqlite3VdbeAddOp2(v, OP_Count, iTabCur, regStat4+1);
    
    // For each index
    for(Index *pIdx = pTab->pIndex; pIdx; pIdx = pIdx->pNext){
      
      // Collect samples
      int nSample = 0;
      int nCol = pIdx->nColumn;
      
      // Open cursor on index
      sqlite3VdbeAddOp3(v, OP_OpenRead, iIdxCur, pIdx->tnum, iDb);
      
      // Sample rows (reservoir sampling)
      sqlite3VdbeAddOp2(v, OP_Rewind, iIdxCur, endOfLoop);
      
      int iLoop = sqlite3VdbeCurrentAddr(v);
      
      // Random sampling
      sqlite3VdbeAddOp2(v, OP_Integer, nSample, regTemp);
      sqlite3VdbeAddOp2(v, OP_Integer, SQLITE_STAT4_SAMPLES, regTemp+1);
      sqlite3VdbeAddOp3(v, OP_Random, regTemp+1, regTemp, 0);
      
      // If selected, store sample
      // ... sample storage code ...
      
      sqlite3VdbeAddOp2(v, OP_Next, iIdxCur, iLoop);
      sqlite3VdbeResolveLabel(v, endOfLoop);
      
      // Calculate statistics
      // stat string format: "N K1 K2 K3 ..."
      // N = total rows
      // Ki = average rows per distinct value in first i columns
      
      // Store in sqlite_stat1
      sqlite3VdbeAddOp4(v, OP_MakeRecord, regStat, 3, regRec,
                        "aaa", 0);
      sqlite3VdbeAddOp2(v, OP_NewRowid, iStatCur, regRowid);
      sqlite3VdbeAddOp3(v, OP_Insert, iStatCur, regRec, regRowid);
    }
  }
}
```

**Using Statistics in Query Planning:**

```c
// where.c
LogEst sqlite3WhereEstimateNumRows(
  WhereLoop *pLoop,
  Index *pIdx
){
  // Get statistics from sqlite_stat1
  sqlite3_stmt *pStmt;
  sqlite3_prepare(db, 
    "SELECT stat FROM sqlite_stat1 WHERE tbl=? AND idx=?",
    -1, &pStmt, 0);
  
  sqlite3_bind_text(pStmt, 1, pTab->zName, -1, SQLITE_STATIC);
  sqlite3_bind_text(pStmt, 2, pIdx->zName, -1, SQLITE_STATIC);
  
  if( sqlite3_step(pStmt) == SQLITE_ROW ){
    const char *zStat = (const char*)sqlite3_column_text(pStmt, 0);
    
    // Parse stat string: "N K1 K2 K3"
    // N = total rows, Ki = avg rows per distinct key
    LogEst nRow = sqlite3LogEst(atoi(zStat));
    
    // Adjust for WHERE terms
    for(int i = 0; i < nTerm; i++){
      if( pTerm[i].eOperator & WO_EQ ){
        // Equality reduces by factor of Ki
        nRow -= sqlite3LogEst(atoi(nextToken));
      }
    }
    
    return nRow;
  }
  
  // No stats - use default estimate
  return sqlite3LogEst(1000);
}
```

**Example with Statistics:**

```sql
-- Create table and index
CREATE TABLE users(id INTEGER PRIMARY KEY, age INTEGER, name TEXT);
CREATE INDEX idx_age ON users(age);

-- Insert test data
INSERT INTO users SELECT x, x % 100, 'User' || x FROM generate_series(1, 10000);

-- Without ANALYZE
EXPLAIN QUERY PLAN SELECT * FROM users WHERE age = 25;
-- Scan using idx_age (estimated rows: 100)

-- Run ANALYZE
ANALYZE;

-- After ANALYZE
EXPLAIN QUERY PLAN SELECT * FROM users WHERE age = 25;
-- Scan using idx_age (estimated rows: 100 - accurate!)

-- Check statistics
SELECT * FROM sqlite_stat1 WHERE tbl='users';
-- tbl='users', idx='idx_age', stat='10000 100'
-- Meaning: 10000 rows, avg 100 rows per distinct age
```

---

## 2. Virtual Tables

### 2.1 Virtual Table Interface

**Virtual Table Module:**

```c
// Virtual table module structure
typedef struct sqlite3_module {
  int iVersion;                     // Module version
  
  // Lifecycle
  int (*xCreate)(sqlite3*, void *pAux, int argc, const char **argv,
                 sqlite3_vtab **ppVTab, char **pzErr);
  int (*xConnect)(sqlite3*, void *pAux, int argc, const char **argv,
                  sqlite3_vtab **ppVTab, char **pzErr);
  int (*xBestIndex)(sqlite3_vtab *pVTab, sqlite3_index_info*);
  int (*xDisconnect)(sqlite3_vtab *pVTab);
  int (*xDestroy)(sqlite3_vtab *pVTab);
  
  // Data access
  int (*xOpen)(sqlite3_vtab *pVTab, sqlite3_vtab_cursor **ppCursor);
  int (*xClose)(sqlite3_vtab_cursor*);
  int (*xFilter)(sqlite3_vtab_cursor*, int idxNum, const char *idxStr,
                 int argc, sqlite3_value **argv);
  int (*xNext)(sqlite3_vtab_cursor*);
  int (*xEof)(sqlite3_vtab_cursor*);
  int (*xColumn)(sqlite3_vtab_cursor*, sqlite3_context*, int);
  int (*xRowid)(sqlite3_vtab_cursor*, sqlite3_int64 *pRowid);
  
  // Updates (optional)
  int (*xUpdate)(sqlite3_vtab*, int, sqlite3_value**, sqlite3_int64*);
  
  // Transactions (optional)
  int (*xBegin)(sqlite3_vtab *pVTab);
  int (*xSync)(sqlite3_vtab *pVTab);
  int (*xCommit)(sqlite3_vtab *pVTab);
  int (*xRollback)(sqlite3_vtab *pVTab);
  
  // More optional methods...
} sqlite3_module;
```

### 2.2 Example: CSV Virtual Table

```c
#include <sqlite3ext.h>
SQLITE_EXTENSION_INIT1

// Virtual table structure
typedef struct csv_vtab {
  sqlite3_vtab base;    // Base class
  char *zFilename;      // CSV filename
  int nCol;             // Number of columns
  char **azCol;         // Column names
} csv_vtab;

// Cursor structure
typedef struct csv_cursor {
  sqlite3_vtab_cursor base;  // Base class
  FILE *fp;                  // File pointer
  char *zLine;               // Current line
  int nLine;                 // Line number
  char **azVal;              // Column values
} csv_cursor;

// Create virtual table
static int csvCreate(
  sqlite3 *db,
  void *pAux,
  int argc, const char *const*argv,
  sqlite3_vtab **ppVtab,
  char **pzErr
){
  csv_vtab *pVtab;
  
  // argv[0] = module name
  // argv[1] = database name
  // argv[2] = table name
  // argv[3+] = arguments (filename, etc.)
  
  if( argc < 4 ){
    *pzErr = sqlite3_mprintf("no filename specified");
    return SQLITE_ERROR;
  }
  
  // Allocate virtual table
  pVtab = sqlite3_malloc(sizeof(csv_vtab));
  if( !pVtab ) return SQLITE_NOMEM;
  memset(pVtab, 0, sizeof(csv_vtab));
  
  pVtab->zFilename = sqlite3_mprintf("%s", argv[3]);
  
  // Read first line to get column names
  FILE *fp = fopen(pVtab->zFilename, "r");
  if( !fp ){
    *pzErr = sqlite3_mprintf("cannot open file: %s", argv[3]);
    sqlite3_free(pVtab);
    return SQLITE_ERROR;
  }
  
  char line[1024];
  if( fgets(line, sizeof(line), fp) ){
    // Parse CSV header
    pVtab->nCol = 1;
    for(char *p = line; *p; p++){
      if( *p == ',' ) pVtab->nCol++;
    }
    
    // Allocate column names
    pVtab->azCol = sqlite3_malloc(pVtab->nCol * sizeof(char*));
    
    char *token = strtok(line, ",\n");
    for(int i = 0; i < pVtab->nCol && token; i++){
      pVtab->azCol[i] = sqlite3_mprintf("%s", token);
      token = strtok(NULL, ",\n");
    }
  }
  fclose(fp);
  
  // Declare table schema
  char *zSql = sqlite3_mprintf("CREATE TABLE x(");
  for(int i = 0; i < pVtab->nCol; i++){
    zSql = sqlite3_mprintf("%z%s%s TEXT", zSql,
                           i > 0 ? ", " : "",
                           pVtab->azCol[i]);
  }
  zSql = sqlite3_mprintf("%z)", zSql);
  
  int rc = sqlite3_declare_vtab(db, zSql);
  sqlite3_free(zSql);
  
  *ppVtab = &pVtab->base;
  return rc;
}

// Open cursor
static int csvOpen(sqlite3_vtab *pVTab, sqlite3_vtab_cursor **ppCursor){
  csv_vtab *pVtab = (csv_vtab*)pVTab;
  csv_cursor *pCur;
  
  pCur = sqlite3_malloc(sizeof(csv_cursor));
  if( !pCur ) return SQLITE_NOMEM;
  memset(pCur, 0, sizeof(csv_cursor));
  
  pCur->fp = fopen(pVtab->zFilename, "r");
  if( !pCur->fp ){
    sqlite3_free(pCur);
    return SQLITE_ERROR;
  }
  
  // Skip header line
  char line[1024];
  fgets(line, sizeof(line), pCur->fp);
  
  pCur->azVal = sqlite3_malloc(pVtab->nCol * sizeof(char*));
  
  *ppCursor = &pCur->base;
  return SQLITE_OK;
}

// Close cursor
static int csvClose(sqlite3_vtab_cursor *cur){
  csv_cursor *pCur = (csv_cursor*)cur;
  
  if( pCur->fp ) fclose(pCur->fp);
  if( pCur->zLine ) sqlite3_free(pCur->zLine);
  if( pCur->azVal ){
    for(int i = 0; i < ((csv_vtab*)cur->pVtab)->nCol; i++){
      sqlite3_free(pCur->azVal[i]);
    }
    sqlite3_free(pCur->azVal);
  }
  sqlite3_free(pCur);
  
  return SQLITE_OK;
}

// Read next row
static int csvNext(sqlite3_vtab_cursor *cur){
  csv_cursor *pCur = (csv_cursor*)cur;
  csv_vtab *pVtab = (csv_vtab*)cur->pVtab;
  
  // Free previous line
  if( pCur->zLine ){
    for(int i = 0; i < pVtab->nCol; i++){
      sqlite3_free(pCur->azVal[i]);
    }
  }
  
  // Read next line
  char line[1024];
  if( !fgets(line, sizeof(line), pCur->fp) ){
    return SQLITE_OK;  // EOF
  }
  
  pCur->zLine = sqlite3_mprintf("%s", line);
  pCur->nLine++;
  
  // Parse CSV line
  char *token = strtok(line, ",\n");
  for(int i = 0; i < pVtab->nCol && token; i++){
    pCur->azVal[i] = sqlite3_mprintf("%s", token);
    token = strtok(NULL, ",\n");
  }
  
  return SQLITE_OK;
}

// Check if at EOF
static int csvEof(sqlite3_vtab_cursor *cur){
  csv_cursor *pCur = (csv_cursor*)cur;
  return pCur->zLine == NULL;
}

// Get column value
static int csvColumn(
  sqlite3_vtab_cursor *cur,
  sqlite3_context *ctx,
  int i
){
  csv_cursor *pCur = (csv_cursor*)cur;
  
  if( i >= 0 && i < ((csv_vtab*)cur->pVtab)->nCol ){
    sqlite3_result_text(ctx, pCur->azVal[i], -1, SQLITE_TRANSIENT);
  }
  
  return SQLITE_OK;
}

// Get rowid
static int csvRowid(sqlite3_vtab_cursor *cur, sqlite_int64 *pRowid){
  csv_cursor *pCur = (csv_cursor*)cur;
  *pRowid = pCur->nLine;
  return SQLITE_OK;
}

// Filter (start iteration)
static int csvFilter(
  sqlite3_vtab_cursor *cur,
  int idxNum, const char *idxStr,
  int argc, sqlite3_value **argv
){
  return csvNext(cur);
}

// Best index (query planning)
static int csvBestIndex(sqlite3_vtab *tab, sqlite3_index_info *pInfo){
  pInfo->estimatedCost = 1000000.0;  // Full scan
  return SQLITE_OK;
}

// Module definition
static sqlite3_module csvModule = {
  0,                    // iVersion
  csvCreate,            // xCreate
  csvCreate,            // xConnect
  csvBestIndex,         // xBestIndex
  NULL,                 // xDisconnect
  NULL,                 // xDestroy
  csvOpen,              // xOpen
  csvClose,             // xClose
  csvFilter,            // xFilter
  csvNext,              // xNext
  csvEof,               // xEof
  csvColumn,            // xColumn
  csvRowid,             // xRowid
  NULL,                 // xUpdate
  NULL,                 // xBegin
  NULL,                 // xSync
  NULL,                 // xCommit
  NULL,                 // xRollback
};

// Extension entry point
#ifdef _WIN32
__declspec(dllexport)
#endif
int sqlite3_csv_init(
  sqlite3 *db,
  char **pzErrMsg,
  const sqlite3_api_routines *pApi
){
  SQLITE_EXTENSION_INIT2(pApi);
  return sqlite3_create_module(db, "csv", &csvModule, 0);
}
```

**Using the CSV Virtual Table:**

```sql
-- Load extension
.load ./csv

-- Create virtual table
CREATE VIRTUAL TABLE users_csv USING csv('users.csv');

-- Query CSV file as table
SELECT * FROM users_csv WHERE age > 25;

-- Join with regular tables
SELECT u.name, o.order_date
FROM users_csv u
JOIN orders o ON u.id = o.user_id;
```

---

## 3. Extensions and Custom Functions

### 3.1 Custom Scalar Function

```c
// Example: ROT13 function
static void rot13Func(
  sqlite3_context *context,
  int argc,
  sqlite3_value **argv
){
  const unsigned char *zIn;
  unsigned char *zOut;
  int n;
  
  if( argc != 1 ) return;
  
  zIn = sqlite3_value_text(argv[0]);
  if( !zIn ) return;
  
  n = sqlite3_value_bytes(argv[0]);
  zOut = sqlite3_malloc(n + 1);
  
  for(int i = 0; i < n; i++){
    unsigned char c = zIn[i];
    if( c >= 'a' && c <= 'z' ){
      c = ((c - 'a' + 13) % 26) + 'a';
    }else if( c >= 'A' && c <= 'Z' ){
      c = ((c - 'A' + 13) % 26) + 'A';
    }
    zOut[i] = c;
  }
  zOut[n] = 0;
  
  sqlite3_result_text(context, (char*)zOut, n, sqlite3_free);
}

// Register function
int sqlite3_extension_init(
  sqlite3 *db,
  char **pzErrMsg,
  const sqlite3_api_routines *pApi
){
  SQLITE_EXTENSION_INIT2(pApi);
  
  return sqlite3_create_function(db, "rot13", 1,
           SQLITE_UTF8 | SQLITE_DETERMINISTIC,
           0, rot13Func, 0, 0);
}

// Usage:
// SELECT rot13('hello');  -- Returns: 'uryyb'
```

### 3.2 Custom Aggregate Function

```c
// Example: MEDIAN aggregate function
typedef struct MedianCtx {
  int nAlloc;
  int nUsed;
  double *aVal;
} MedianCtx;

static void medianStep(
  sqlite3_context *context,
  int argc,
  sqlite3_value **argv
){
  MedianCtx *p;
  double val;
  
  p = sqlite3_aggregate_context(context, sizeof(*p));
  if( !p ) return;
  
  val = sqlite3_value_double(argv[0]);
  
  if( p->nUsed >= p->nAlloc ){
    p->nAlloc = p->nAlloc ? p->nAlloc * 2 : 8;
    p->aVal = sqlite3_realloc(p->aVal, p->nAlloc * sizeof(double));
  }
  
  p->aVal[p->nUsed++] = val;
}

static int compareDouble(const void *a, const void *b){
  double da = *(double*)a;
  double db = *(double*)b;
  return (da < db) ? -1 : (da > db) ? 1 : 0;
}

static void medianFinalize(sqlite3_context *context){
  MedianCtx *p;
  double median;
  
  p = sqlite3_aggregate_context(context, 0);
  if( !p || p->nUsed == 0 ) return;
  
  qsort(p->aVal, p->nUsed, sizeof(double), compareDouble);
  
  if( p->nUsed % 2 == 0 ){
    median = (p->aVal[p->nUsed/2 - 1] + p->aVal[p->nUsed/2]) / 2.0;
  }else{
    median = p->aVal[p->nUsed/2];
  }
  
  sqlite3_result_double(context, median);
  sqlite3_free(p->aVal);
}

// Register aggregate
sqlite3_create_function(db, "median", 1,
  SQLITE_UTF8, 0, 0, medianStep, medianFinalize);

// Usage:
// SELECT median(salary) FROM employees;
```

---

## 4. Performance Tuning

### 4.1 Using EXPLAIN QUERY PLAN

```sql
-- Example query
EXPLAIN QUERY PLAN
SELECT u.name, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.age > 25
GROUP BY u.id;

-- Output shows execution plan:
-- SCAN TABLE users AS u
-- SEARCH TABLE orders AS o USING INDEX idx_user_id (user_id=?)
```

### 4.2 Index Design Tips

```sql
-- Bad: No index on WHERE clause
CREATE TABLE users(id INTEGER, name TEXT, age INTEGER);
SELECT * FROM users WHERE age = 25;  -- Full table scan

-- Good: Index on WHERE clause
CREATE INDEX idx_age ON users(age);
SELECT * FROM users WHERE age = 25;  -- Index scan

-- Better: Covering index (includes all needed columns)
CREATE INDEX idx_age_name ON users(age, name);
SELECT name FROM users WHERE age = 25;  -- Index-only scan

-- Best practices:
-- 1. Index columns used in WHERE, JOIN, ORDER BY
-- 2. Put equality columns before range columns
-- 3. Include frequently accessed columns at end
-- 4. Don't over-index (slows INSERTs)
```

### 4.3 Query Optimization Techniques

```sql
-- Use INDEXED BY to force index
SELECT * FROM users INDEXED BY idx_age WHERE age = 25;

-- Use NOT INDEXED to force table scan
SELECT * FROM users NOT INDEXED WHERE age = 25;

-- Subquery optimization
-- Bad: Correlated subquery
SELECT name FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- Good: Join instead
SELECT DISTINCT u.name
FROM users u
JOIN orders o ON u.id = o.user_id;

-- Use LIMIT for pagination
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

-- Batch inserts
BEGIN;
INSERT INTO users VALUES (1, 'Alice', 25);
INSERT INTO users VALUES (2, 'Bob', 30);
-- ... many more ...
COMMIT;
```

### 4.4 Performance PRAGMAs

```sql
-- Cache size (negative = KB, positive = pages)
PRAGMA cache_size = -8000;  -- 8MB cache

-- Memory-mapped I/O
PRAGMA mmap_size = 268435456;  -- 256MB

-- Synchronous mode
PRAGMA synchronous = NORMAL;  -- Balance safety/speed
-- FULL = safest, slowest
-- NORMAL = balanced
-- OFF = fastest, unsafe

-- Journal mode
PRAGMA journal_mode = WAL;  -- Write-Ahead Log

-- Temp store
PRAGMA temp_store = MEMORY;  -- Temp tables in memory

-- Auto-vacuum
PRAGMA auto_vacuum = INCREMENTAL;

-- Page size (must set before creating DB)
PRAGMA page_size = 4096;

-- Analyze automatically
PRAGMA optimize;  -- Run at app shutdown
```

---

## 5. Reading SQLite Source Effectively

### 5.1 Navigation Strategy

**Start Here:**

```
1. Documentation:
   - docs/fileformat.html (database file format)
   - docs/opcode.html (VDBE opcodes)
   - docs/arch.html (architecture)

2. Core Files:
   - main.c (API entry points)
   - vdbe.c (execution engine)
   - btree.c (B-tree implementation)
   - pager.c (page cache)

3. Subsystems:
   - parse.y (SQL parser grammar)
   - select.c (SELECT statement)
   - insert.c/update.c/delete.c (modifications)
   - where.c (WHERE clause optimization)
```

### 5.2 Code Reading Tips

**Understanding Code Patterns:**

```c
// SQLite coding conventions

// Error handling pattern
int rc;
rc = someFunction();
if( rc != SQLITE_OK ){
  goto error_exit;
}

error_exit:
  cleanup();
  return rc;

// Testcase macros (for coverage)
testcase( condition );

// Assert macros (disabled in release)
assert( invariant );

// Memory allocation
void *p = sqlite3DbMalloc(db, size);
if( p == 0 ) return SQLITE_NOMEM;

// Reference counting
void *p = refCountedObject();
sqlite3_ref(p);  // Increment
sqlite3_unref(p);  // Decrement

// Mutex locking
sqlite3_mutex_enter(mutex);
// ... critical section ...
sqlite3_mutex_leave(mutex);
```

### 5.3 Debugging SQLite

```bash
# Compile with debugging
gcc -g -DSQLITE_DEBUG -DSQLITE_ENABLE_EXPLAIN_COMMENTS \
    shell.c sqlite3.c -o sqlite3-debug

# Run with GDB
gdb ./sqlite3-debug
(gdb) break sqlite3VdbeExec
(gdb) run
sqlite> SELECT * FROM users;
(gdb) print *pOp
(gdb) continue

# Enable VDBE trace
sqlite> PRAGMA vdbe_trace = ON;
sqlite> SELECT * FROM users;
-- Shows each opcode executed

# Enable parser trace
gcc -DSQLITE_DEBUG -DYYTRACKMAXSTACKDEPTH=1000 ...
```

---

## 6. Contributing to SQLite

### 6.1 SQLite Development Process

**Important Notes:**
- SQLite uses Fossil, not Git
- Contributions require signing contributor agreement
- Code must pass extensive test suite
- Changes go through rigorous review

**Getting Started:**

```bash
# Clone with Fossil
fossil clone https://www.sqlite.org/src sqlite.fossil
mkdir sqlite-src
cd sqlite-src
fossil open ../sqlite.fossil

# Build and test
./configure
make
make test

# Test coverage is extensive (~100%)
# Full test suite takes hours to run
```

### 6.2 Test-Driven Development

```tcl
# SQLite test in TCL
# test/select1.test

do_test select1-1.1 {
  execsql {
    CREATE TABLE t1(a INTEGER, b INTEGER);
    INSERT INTO t1 VALUES(1, 2);
    INSERT INTO t1 VALUES(3, 4);
    SELECT * FROM t1;
  }
} {1 2 3 4}

do_test select1-1.2 {
  execsql {
    SELECT a, b FROM t1 WHERE a=1;
  }
} {1 2}

# Run tests
make test
```

### 6.3 Reporting Bugs

**Good Bug Report:**

```
Title: Incorrect result with complex JOIN and subquery

SQLite Version: 3.45.0
Platform: Linux x86_64

Steps to Reproduce:
1. Create tables with attached SQL
2. Run query: SELECT ...
3. Observe incorrect result

Expected Result: [describe expected]
Actual Result: [describe actual]

Schema:
CREATE TABLE t1(...);
...

Test Case:
[complete, minimal test case]

Bisect Result: Issue appeared in version 3.44.0
```

---

## 7. Conclusion and Next Steps

### 7.1 What You've Learned

**Comprehensive Coverage:**

✅ **Architecture** - Layered design from SQL to disk
✅ **Parser** - Tokenization and parse tree construction
✅ **Code Generation** - Converting SQL to VDBE bytecode
✅ **VDBE** - Virtual machine execution engine
✅ **B-Tree** - Page structure and tree operations
✅ **Pager** - Page cache and transaction management
✅ **Locking** - Multi-version concurrency control
✅ **WAL Mode** - Write-ahead logging for performance
✅ **Query Optimization** - Cost-based query planning
✅ **Extensions** - Virtual tables and custom functions

### 7.2 Recommended Projects

**Beginner:**
1. Write custom SQLite functions
2. Create a simple virtual table
3. Analyze query plans for optimization

**Intermediate:**
4. Implement a new aggregation function
5. Create an extension for JSON operations
6. Write a custom VFS (virtual file system)

**Advanced:**
7. Contribute a bug fix to SQLite
8. Implement a new index type
9. Add a new optimization to the query planner

### 7.3 Further Reading

**Essential Resources:**

```
Documentation:
- https://sqlite.org/docs.html
- https://sqlite.org/arch.html
- https://sqlite.org/fileformat.html
- https://sqlite.org/opcode.html

Books:
- "SQLite Database System" by Jay A. Kreibich
- "The Definitive Guide to SQLite" by Grant Allen

Source Code Study:
- Start with: main.c, vdbe.c, btree.c
- Read: parse.y for SQL grammar
- Study: where.c for optimization

Community:
- https://sqlite.org/forum/
- Fossil repository: https://sqlite.org/src
- Mailing list archives
```