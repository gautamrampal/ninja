# SQLite Source Code Mastery Tutorial - PART 4
## Locking, WAL Mode & Query Optimization

---

## Table of Contents - Part 4
1. [Locking and Concurrency](#1-locking-and-concurrency)
2. [Write-Ahead Logging (WAL)](#2-write-ahead-logging-wal)
3. [Query Planner and Optimization](#3-query-planner-and-optimization)
4. [ANALYZE and Statistics](#4-analyze-and-statistics)
5. [Virtual Tables](#5-virtual-tables)
6. [Performance Tuning Tips](#6-performance-tuning-tips)

---

## 1. Locking and Concurrency

### 1.1 Lock States

**SQLite uses 5 lock levels:**

```
Lock Hierarchy (from least to most restrictive):

UNLOCKED (0)
   │
   ├─ No locks held
   └─ Database can be opened
   
SHARED (1)
   │
   ├─ Reading allowed
   ├─ Multiple readers possible
   └─ No writing allowed
   
RESERVED (2)
   │
   ├─ Preparing to write
   ├─ Other readers allowed
   └─ No other writers allowed
   
PENDING (3)
   │
   ├─ Waiting for readers to clear
   ├─ No new readers allowed
   └─ Transitional state
   
EXCLUSIVE (4)
   │
   ├─ Writing to database
   ├─ No readers allowed
   └─ No other writers allowed
```

**Lock State Transitions:**

```c
// Lock levels
#define NO_LOCK         0
#define SHARED_LOCK     1
#define RESERVED_LOCK   2
#define PENDING_LOCK    3
#define EXCLUSIVE_LOCK  4

// Valid transitions:
// UNLOCKED → SHARED → RESERVED → PENDING → EXCLUSIVE
// EXCLUSIVE → UNLOCKED (commit/rollback)
```

### 1.2 Locking Implementation (os_unix.c)

**Unix File Locking:**

```c
// Lock structure
typedef struct unixFile {
  sqlite3_io_methods const *pMethod;  // IO methods
  sqlite3_vfs *pVfs;                  // VFS that created this
  unixInodeInfo *pInode;              // Inode info
  int h;                              // File descriptor
  unsigned char eFileLock;            // Current lock level
  unsigned short int ctrlFlags;       // Flags
  int lastErrno;                      // Last errno value
  void *lockingContext;               // Locking context
} unixFile;

// Acquire lock
static int unixLock(sqlite3_file *id, int eFileLock){
  unixFile *pFile = (unixFile*)id;
  int rc = SQLITE_OK;
  
  // Get current lock level
  int eLock = pFile->eFileLock;
  
  // Check if upgrade needed
  if( eLock >= eFileLock ){
    return SQLITE_OK;  // Already have this lock or better
  }
  
  // Lock upgrade path
  if( eLock < SHARED_LOCK ){
    // UNLOCKED → SHARED
    rc = unixReadLock(pFile);
  }
  
  if( rc == SQLITE_OK && eFileLock > SHARED_LOCK ){
    // SHARED → RESERVED
    rc = unixWriteLock(pFile, 0);
  }
  
  if( rc == SQLITE_OK && eFileLock == EXCLUSIVE_LOCK ){
    // RESERVED → EXCLUSIVE
    rc = unixWriteLock(pFile, 1);
  }
  
  if( rc == SQLITE_OK ){
    pFile->eFileLock = eFileLock;
  }
  
  return rc;
}

// fcntl-based locking (POSIX)
static int fcntlLock(unixFile *pFile, int eFileLock){
  struct flock lock;
  
  memset(&lock, 0, sizeof(lock));
  lock.l_whence = SEEK_SET;
  lock.l_start = SHARED_FIRST;  // Lock byte offset
  lock.l_len = 1;                // Lock 1 byte
  
  if( eFileLock == SHARED_LOCK ){
    lock.l_type = F_RDLCK;  // Read lock
  }else{
    lock.l_type = F_WRLCK;  // Write lock
  }
  
  // Try to acquire lock (non-blocking)
  if( fcntl(pFile->h, F_SETLK, &lock) == -1 ){
    return SQLITE_BUSY;
  }
  
  return SQLITE_OK;
}

// Release locks
static int unixUnlock(sqlite3_file *id, int eFileLock){
  unixFile *pFile = (unixFile*)id;
  
  if( pFile->eFileLock <= eFileLock ){
    return SQLITE_OK;
  }
  
  // Release locks in reverse order
  if( pFile->eFileLock > SHARED_LOCK ){
    unlockWriteLock(pFile);
  }
  
  if( eFileLock == NO_LOCK ){
    unlockReadLock(pFile);
  }
  
  pFile->eFileLock = eFileLock;
  return SQLITE_OK;
}
```

**Lock Byte Layout:**

```
Database file lock bytes (starting at byte 1073741824):

Byte Offset   Purpose
───────────────────────────────────────
1073741824    PENDING_BYTE
1073741825    RESERVED_BYTE
1073741826+   SHARED_FIRST (510 bytes)

Lock Types:
- SHARED: Any byte in SHARED range
- RESERVED: RESERVED_BYTE
- PENDING: PENDING_BYTE
- EXCLUSIVE: All bytes locked
```

### 1.3 Concurrent Access Example

**Scenario: Multiple Readers + One Writer**

```c
// Process 1 - Reader
sqlite3 *db1;
sqlite3_open("test.db", &db1);

// Acquire SHARED lock
sqlite3_exec(db1, "BEGIN", 0, 0, 0);
sqlite3_exec(db1, "SELECT * FROM users", callback, 0, 0);
// Holds SHARED lock while reading

// Process 2 - Another Reader (concurrent)
sqlite3 *db2;
sqlite3_open("test.db", &db2);

// Can acquire SHARED lock (multiple readers OK)
sqlite3_exec(db2, "BEGIN", 0, 0, 0);
sqlite3_exec(db2, "SELECT * FROM users", callback, 0, 0);

// Process 3 - Writer (blocked)
sqlite3 *db3;
sqlite3_open("test.db", &db3);

sqlite3_exec(db3, "BEGIN", 0, 0, 0);
// Acquires RESERVED lock (readers can continue)

sqlite3_exec(db3, "INSERT INTO users VALUES (1, 'Alice')", 0, 0, 0);
// Still at RESERVED (readers finishing)

sqlite3_exec(db3, "COMMIT", 0, 0, 0);
// Tries PENDING → EXCLUSIVE
// Waits for db1 and db2 to release SHARED locks
// Then acquires EXCLUSIVE and writes
```

**Deadlock Prevention:**

```
SQLite prevents deadlocks through lock hierarchy:

1. Locks can only be upgraded (never downgraded during transaction)
2. PENDING lock prevents new readers (breaking circular wait)
3. Timeout on busy (sqlite3_busy_timeout)
4. Single writer at a time

No deadlock possible because:
- Only one process can hold RESERVED
- RESERVED holder will eventually get EXCLUSIVE
- New readers blocked by PENDING
```

---

## 2. Write-Ahead Logging (WAL)

### 2.1 WAL Mode Overview

**WAL vs Rollback Journal:**

```
Rollback Journal Mode:
┌─────────┐
│Database │ ← Original data
└─────────┘
     │
     ├──────────────┐
     │              │
┌─────────┐    ┌─────────┐
│ Journal │    │Database │
│  (old)  │    │  (new)  │
└─────────┘    └─────────┘

WAL Mode:
┌─────────┐     ┌─────────┐
│Database │     │   WAL   │
│  (old)  │     │  (new)  │
└─────────┘     └─────────┘
                      │
                      └─→ Checkpoint merges
```

**Advantages of WAL:**
- Better concurrency (readers don't block writers)
- Faster (no sync after every transaction)
- More graceful handling of interruptions

**WAL File Format:**

```
┌──────────────────────────────────────┐
│ WAL Header (32 bytes)                │
│  - Magic number (0x377f0682/0x377f0683)│
│  - File format version               │
│  - Page size                         │
│  - Checkpoint sequence               │
│  - Salt-1, Salt-2 (random)           │
│  - Checksum-1, Checksum-2            │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ Frame 1                              │
│  - Page number (4 bytes)             │
│  - Commit marker (4 bytes)           │
│  - Salt-1, Salt-2 (from header)      │
│  - Checksum-1, Checksum-2            │
│  - Page data (page_size bytes)       │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ Frame 2                              │
│  ...                                 │
└──────────────────────────────────────┘
        ... more frames ...
```

### 2.2 WAL Implementation (wal.c)

**WAL Structure:**

```c
struct Wal {
  sqlite3_vfs *pVfs;          // VFS interface
  sqlite3_file *pDbFd;        // Database file
  sqlite3_file *pWalFd;       // WAL file
  
  // Memory-mapped WAL
  volatile u32 *aWiData;      // WAL index in shared memory
  u32 szPage;                 // Page size
  
  // Read marks
  i64 hdr;                    // WAL header
  u32 minFrame;               // Minimum frame number
  u32 maxFrame;               // Maximum frame number
  
  // Checkpoint info
  u8 lockError;               // Lock error occurred
  u8 ckptLock;                // Checkpoint lock held
  u8 readLock;                // Read lock level
  u8 syncFlags;               // Sync flags
  
  // Callback
  int (*xCallback)(void *, sqlite3 *, const char *, int);
  void *pCallbackArg;
};

// WAL index structure (shared memory)
typedef struct WalIndexHdr {
  u32 iVersion;               // Index version
  u32 unused;                 // Unused padding
  u32 iChange;                // Incremented on each transaction
  u8 isInit;                  // 1 when initialized
  u8 bigEndCksum;             // Checksum byte order
  u16 szPage;                 // Page size
  u32 mxFrame;                // Max frame index
  u32 nPage;                  // Pages in database
  u32 aFrameCksum[2];         // Frame checksum
  u32 aSalt[2];               // Random salt
  u32 aCksum[2];              // Header checksum
} WalIndexHdr;
```

**Write to WAL:**

```c
int sqlite3WalFrames(
  Wal *pWal,                  // WAL handle
  int szPage,                 // Page size
  PgHdr *pList,               // List of dirty pages
  Pgno truncate,              // Truncate database to this size
  int isCommit,               // True for commit
  int syncFlags               // Sync flags
){
  PgHdr *p;
  int nFrame = 0;
  u32 salt[2];
  
  // Get current salt values
  memcpy(salt, pWal->hdr.aSalt, sizeof(salt));
  
  // Write each page as a frame
  for(p = pList; p; p = p->pDirty){
    nFrame++;
    
    // Frame header
    u8 aFrame[WAL_FRAME_HDRSIZE];
    walEncodeFrame(pWal, p->pgno, truncate, p->pData, aFrame);
    
    // Write frame header
    rc = sqlite3OsWrite(pWal->pWalFd, aFrame, sizeof(aFrame),
                        offset);
    offset += sizeof(aFrame);
    
    // Write frame data
    rc = sqlite3OsWrite(pWal->pWalFd, p->pData, szPage,
                        offset);
    offset += szPage;
    
    // Update WAL index
    walIndexAppend(pWal, nFrame, p->pgno);
  }
  
  // If commit, set commit marker
  if( isCommit ){
    walSetCommitMarker(pWal, nFrame);
  }
  
  // Sync WAL file
  if( syncFlags ){
    rc = sqlite3OsSync(pWal->pWalFd, syncFlags);
  }
  
  return SQLITE_OK;
}
```

**Read from WAL:**

```c
int sqlite3WalFindFrame(
  Wal *pWal,           // WAL handle
  Pgno pgno,           // Page number to find
  u32 *piRead          // OUT: Frame number
){
  u32 iRead = 0;
  u32 iLast = pWal->hdr.mxFrame;
  
  // Search WAL index for most recent frame
  // containing this page
  
  // Binary search in hash table
  WalHashLoc loc;
  if( walHashGet(pWal, walFramePage(iLast), &loc) ){
    // Found in hash - use hash to locate frame
    u32 iFrame = walFramePgno(pWal, loc.iSlot);
    if( iFrame == pgno ){
      iRead = loc.iSlot;
    }
  }
  
  if( iRead == 0 ){
    // Linear search from end
    for(u32 i = iLast; i >= 1; i--){
      if( walFramePage(pWal, i) == pgno ){
        iRead = i;
        break;
      }
    }
  }
  
  *piRead = iRead;
  return SQLITE_OK;
}
```

**Checkpoint Operation:**

```c
int sqlite3WalCheckpoint(
  Wal *pWal,                  // WAL handle
  int eMode,                  // Checkpoint mode
  int (*xBusy)(void*),        // Busy callback
  void *pBusyArg,             // Busy callback arg
  int sync_flags,             // Sync flags
  u8 *zBuf                    // Temporary buffer
){
  int rc = SQLITE_OK;
  u32 nBackfill = 0;
  
  // Determine how many frames to checkpoint
  u32 mxFrame = pWal->hdr.mxFrame;
  
  // For each frame in WAL
  for(u32 i = 1; i <= mxFrame; i++){
    Pgno pgno = walFramePage(pWal, i);
    
    // Read frame from WAL
    rc = walReadFrame(pWal, i, zBuf);
    if( rc ) break;
    
    // Write to database file
    i64 offset = (pgno - 1) * pWal->szPage;
    rc = sqlite3OsWrite(pWal->pDbFd, zBuf, 
                        pWal->szPage, offset);
    if( rc ) break;
    
    nBackfill = i;
  }
  
  // Sync database file
  if( sync_flags ){
    rc = sqlite3OsSync(pWal->pDbFd, sync_flags);
  }
  
  // Update checkpoint marker in WAL
  walRestartHdr(pWal, nBackfill);
  
  // If full checkpoint, truncate WAL
  if( eMode == CHECKPOINT_TRUNCATE ){
    rc = sqlite3OsTruncate(pWal->pWalFd, 0);
  }
  
  return rc;
}
```

**WAL Checkpoint Modes:**

```c
// Checkpoint modes
#define CHECKPOINT_PASSIVE  0  // Checkpoint what you can
#define CHECKPOINT_FULL     1  // Wait for readers, full checkpoint
#define CHECKPOINT_RESTART  2  // Full + restart WAL
#define CHECKPOINT_TRUNCATE 3  // Restart + truncate WAL to zero

// Usage:
sqlite3_wal_checkpoint_v2(db, NULL, SQLITE_CHECKPOINT_FULL,
                          &nLog, &nCkpt);
```

---

## 3. Query Planner and Optimization

### 3.1 Query Planning Process

**Planning Stages:**

```
SQL Query
    ↓
┌─────────────────────┐
│   Parse Tree        │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│   Semantic Check    │  (name resolution, type checking)
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│   Query Optimizer   │
│  - Rewrite rules    │
│  - Index selection  │
│  - Join order       │
│  - Cost estimation  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│   Code Generator    │
└──────────┬──────────┘
           ↓
     VDBE Bytecode
```

### 3.2 WHERE Clause Analysis (whereInt.h)

**WhereClause Structure:**

```c
struct WhereClause {
  Parse *pParse;              // Parser context
  WhereMaskSet *pMaskSet;     // Bitmask of tables
  WhereClause *pOuter;        // Outer WHERE clause
  u8 op;                      // Split operator (AND/OR)
  int nTerm;                  // Number of terms
  int nSlot;                  // Allocated slots
  WhereTerm *a;               // Array of terms
};

struct WhereTerm {
  Expr *pExpr;                // Expression for this term
  WhereClause *pWC;           // Containing WHERE clause
  LogEst truthProb;           // Probability of being true
  u16 eOperator;              // Operator (WO_EQ, WO_LT, etc.)
  u16 wtFlags;                // Flags
  u8 nChild;                  // Number of children
  u8 iParent;                 // Parent term index
  int iField;                 // Column number
  Bitmask prereqRight;        // Right-hand prerequisites
  Bitmask prereqAll;          // All prerequisites
};

// Operators
#define WO_EQ     0x0001    // term = expr
#define WO_LT     0x0002    // term < expr
#define WO_LE     0x0004    // term <= expr
#define WO_GT     0x0008    // term > expr
#define WO_GE     0x0010    // term >= expr
#define WO_MATCH  0x0020    // term MATCH expr
#define WO_ISNULL 0x0040    // term IS NULL
#define WO_IN     0x0080    // term IN (...)
```

**Index Selection:**

```c
WhereCost sqlite3WhereQueryCost(
  WhereInfo *pWInfo,
  WhereTerm *pTerm,
  Index *pIdx
){
  WhereCost cost;
  
  // Estimate rows scanned
  cost.rRun = 1.0;  // Start with 1 row
  
  // Adjust for each usable term
  for(int i = 0; i < pIdx->nColumn; i++){
    if( pTerm[i].eOperator & WO_EQ ){
      // Equality: divide by 10
      cost.rRun /= 10.0;
    }else if( pTerm[i].eOperator & (WO_LT|WO_LE|WO_GT|WO_GE) ){
      // Range: divide by 3
      cost.rRun /= 3.0;
    }
  }
  
  // Multiply by table size
  cost.rRun *= pIdx->aiRowEst[0];
  
  // Add setup cost
  cost.rSetup = 20.0;  // Cost to open cursor
  
  return cost;
}
```

### 3.3 Join Order Optimization

**Cost-Based Join Selection:**

```c
static void bestJoinOrder(
  WhereInfo *pWInfo,       // WHERE info
  WhereCost *pCost         // OUT: Best cost
){
  int nTab = pWInfo->nLevel;
  int nPermute = 1;
  
  // Calculate factorial for permutations
  for(int i = 1; i <= nTab; i++){
    nPermute *= i;
  }
  
  // If too many permutations, use greedy algorithm
  if( nPermute > 10000 ){
    greedyJoinOrder(pWInfo, pCost);
    return;
  }
  
  // Try all permutations
  WhereCost bestCost;
  bestCost.rRun = 1e100;  // Very high initial cost
  
  int aOrder[20];  // Join order
  for(int i = 0; i < nTab; i++) aOrder[i] = i;
  
  do {
    WhereCost cost;
    estimateJoinCost(pWInfo, aOrder, &cost);
    
    if( cost.rRun < bestCost.rRun ){
      bestCost = cost;
      memcpy(pWInfo->aiRowEst, aOrder, nTab * sizeof(int));
    }
  } while( nextPermutation(aOrder, nTab) );
  
  *pCost = bestCost;
}

static void estimateJoinCost(
  WhereInfo *pWInfo,
  int *aOrder,
  WhereCost *pCost
){
  double cost = 0.0;
  double rows = 1.0;
  
  // Nested loop join cost
  for(int i = 0; i < pWInfo->nLevel; i++){
    int iTab = aOrder[i];
    SrcList_item *pTab = &pWInfo->pTabList->a[iTab];
    
    // Table scan cost
    double tableCost = pTab->pTab->nRowEst;
    
    // Check for usable index
    Index *pBest = NULL;
    double bestIdxCost = tableCost;
    
    for(Index *pIdx = pTab->pTab->pIndex; pIdx; pIdx = pIdx->pNext){
      double idxCost = estimateIndexCost(pWInfo, pIdx, iTab);
      if( idxCost < bestIdxCost ){
        bestIdxCost = idxCost;
        pBest = pIdx;
      }
    }
    
    // Add to total cost (rows * inner_cost)
    cost += rows * bestIdxCost;
    rows *= bestIdxCost;  // Update row estimate
  }
  
  pCost->rRun = cost;
}
```

---

## CONTINUED IN PART 5

This is Part 4 of the SQLite Source Code Mastery Tutorial. It covers:
- Locking mechanisms and concurrency control
- Write-Ahead Logging (WAL) implementation
- Query planner basics and cost estimation

**Part 5 (Final) will cover:**
- ANALYZE and statistics collection
- Virtual tables implementation
- Extensions and custom functions
- Performance tuning and best practices
- Reading SQLite source code effectively

Save this file and continue with Part 5!