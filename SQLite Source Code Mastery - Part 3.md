# SQLite Source Code Mastery Tutorial - PART 3
## B-Tree, Pager, Transactions & Query Optimization

---

## Table of Contents - Part 3
1. [B-Tree Deep Dive](#1-b-tree-deep-dive)
2. [Pager Layer](#2-pager-layer)
3. [Transaction Management](#3-transaction-management)
4. [Locking and Concurrency](#4-locking-and-concurrency)
5. [Write-Ahead Logging (WAL)](#5-write-ahead-logging)
6. [Query Optimization](#6-query-optimization)

---

## 1. B-Tree Deep Dive

### 1.1 B-Tree Page Structure

**Page Header Format:**

```c
// Page types
#define PGTYPE_INTKEY_LEAF   5   // B-tree leaf page (table)
#define PGTYPE_INTKEY_INT    2   // B-tree interior page (table)
#define PGTYPE_ZERODATA_LEAF 10  // B-tree leaf page (index)
#define PGTYPE_ZERODATA_INT  13  // B-tree interior page (index)

// Interior page header (12 bytes)
typedef struct IntPageHdr {
  u8 pageType;        // Page type (2 or 5)
  u16 firstFreeblock; // Offset to first freeblock
  u16 nCell;          // Number of cells on page
  u16 cellStart;      // Start of cell content area
  u8 nFrag;           // Number of fragmented free bytes
  u32 rightPtr;       // Right child pointer (interior only)
} IntPageHdr;

// Leaf page header (8 bytes)
typedef struct LeafPageHdr {
  u8 pageType;        // Page type (5 or 10)
  u16 firstFreeblock; // Offset to first freeblock
  u16 nCell;          // Number of cells on page
  u16 cellStart;      // Start of cell content area
  u8 nFrag;           // Number of fragmented free bytes
} LeafPageHdr;
```

**Page Layout Visualization:**

```
┌──────────────────────────────────────────────────────────┐
│ Page Header (8 or 12 bytes)                             │
│  - Page type                                             │
│  - First freeblock offset                               │
│  - Number of cells                                       │
│  - Cell content area start                              │
│  - Fragmented bytes count                               │
│  - Right child pointer (interior only)                  │
├──────────────────────────────────────────────────────────┤
│ Cell Pointer Array (2 bytes × number of cells)          │
│  - Each pointer is 2 bytes                              │
│  - Points to cell content location                       │
│  - Array grows downward                                  │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ Unallocated Space                                        │
│  - Available for new cells                              │
│                                                          │
├──────────────────────────────────────────────────────────┤
│ Cell Content Area (grows upward from end)                │
│  - Interior cell: [child_ptr(4)] [key(varint)]          │
│  - Table leaf:    [payload_size(varint)] [rowid(varint)]│
│                   [payload] [overflow_ptr(4)?]           │
│  - Index leaf:    [payload_size(varint)] [payload]       │
│                   [overflow_ptr(4)?]                     │
└──────────────────────────────────────────────────────────┘

Page Size: Typically 4096 bytes (configurable)
Max cells per page: ~(page_size - header) / (min_cell_size + 2)
```

### 1.2 B-Tree Cell Formats

**Interior Page Cell (Table):**

```c
// Interior cell format:
// [4-byte child pointer][varint key]

typedef struct BtreeCellInt {
  u32 childPtr;      // Child page number
  i64 key;           // Integer key (rowid)
} BtreeCellInt;

// Encoding example:
// Child page = 100, Key = 1000
// Bytes: 00 00 00 64 E8 07
//        └─child─┘ └key┘
```

**Leaf Page Cell (Table):**

```c
// Leaf cell format:
// [varint payload_size][varint rowid][payload][overflow?]

typedef struct BtreeCellLeaf {
  u64 nPayload;      // Payload size
  i64 rowid;         // Row ID
  u8 *payload;       // Actual data
  u32 ovflPtr;       // Overflow page (if needed)
} BtreeCellLeaf;

// Example:
// Payload size = 100, Rowid = 42, Data = "Hello..."
// Bytes: 64 2A [100 bytes of data]
//        │  │
//        │  └─ Rowid (varint 42)
//        └─ Payload size (varint 100)
```

**Overflow Pages:**

```
When cell payload exceeds max local payload:

Main Cell:
┌────────────────────────────────────┐
│ Payload size (varint)              │
│ Rowid (varint)                     │
│ Local payload (max_local bytes)    │
│ Overflow page number (4 bytes)     │
└────────────────────────────────────┘
         │
         ▼
Overflow Page 1:
┌────────────────────────────────────┐
│ Next overflow page (4 bytes)       │
│ Overflow payload data              │
└────────────────────────────────────┘
         │
         ▼
Overflow Page 2:
┌────────────────────────────────────┐
│ Next overflow = 0 (4 bytes)        │
│ Remaining payload data             │
└────────────────────────────────────┘
```

### 1.3 B-Tree Operations

**Search/Lookup (btree.c):**

```c
int sqlite3BtreeMovetoUnpacked(
  BtCursor *pCur,           // Cursor
  UnpackedRecord *pIdxKey,  // Key to search for
  i64 intKey,               // Integer key (for tables)
  int bias,                 // Bias (-1/0/+1)
  int *pRes                 // Result: <0, 0, or >0
){
  MemPage *pPage = pCur->pPage;
  int i, nCell;
  
  while(1){
    nCell = pPage->nCell;
    
    // Binary search in current page
    int lwr = 0;
    int upr = nCell - 1;
    int idx = -1;
    
    while( lwr <= upr ){
      int mid = (lwr + upr) / 2;
      
      // Get cell key
      i64 cellKey;
      getCellKey(pPage, mid, &cellKey);
      
      // Compare
      int c = (intKey < cellKey) ? -1 : 
              (intKey > cellKey) ? +1 : 0;
      
      if( c == 0 ){
        // Found exact match
        idx = mid;
        break;
      }else if( c < 0 ){
        upr = mid - 1;
      }else{
        lwr = mid + 1;
      }
    }
    
    // Check if we need to go to child page
    if( pPage->isLeaf ){
      // Leaf page - search complete
      pCur->ix = (idx >= 0) ? idx : lwr;
      *pRes = (idx >= 0) ? 0 : ((lwr >= nCell) ? 1 : -1);
      return SQLITE_OK;
    }else{
      // Interior page - follow child pointer
      u32 childPgno;
      if( idx >= 0 ){
        childPgno = getChildPtr(pPage, idx);
      }else if( lwr >= nCell ){
        childPgno = pPage->rightChild;
      }else{
        childPgno = getChildPtr(pPage, lwr);
      }
      
      // Load child page and continue search
      rc = getAndInitPage(pCur->pBt, childPgno, &pPage);
      if( rc ) return rc;
      pCur->pPage = pPage;
    }
  }
}
```

**Insert Operation:**

```c
int sqlite3BtreeInsert(
  BtCursor *pCur,           // Cursor pointing to insert position
  const void *pKey,         // Key data
  i64 nKey,                 // Key size
  const void *pData,        // Data payload
  int nData,                // Data size
  int nZero,                // Extra zeros
  int appendBias,           // Hint: likely appending
  int seekResult            // Result of prior seek
){
  MemPage *pPage = pCur->pPage;
  Btree *pBt = pCur->pBt;
  int rc;
  
  // Check if page has room for new cell
  int cellSize = computeCellSize(nKey, nData);
  
  if( pPage->nFree < cellSize + 2 ){
    // Page is full - need to split
    rc = balance(pCur);
    if( rc ) return rc;
    pPage = pCur->pPage;
  }
  
  // Allocate space for cell
  int idx = pCur->ix;
  u8 *pCell = allocateSpace(pPage, cellSize);
  
  // Write cell data
  int n = putVarint(pCell, nKey);       // Write key size
  n += putVarint(pCell + n, pData);     // Write data
  memcpy(pCell + n, pData, nData);      // Copy payload
  
  // Insert cell pointer
  insertCellPointer(pPage, idx, pCell - pPage->aData);
  
  // Update page header
  pPage->nCell++;
  pPage->nFree -= cellSize + 2;
  
  return SQLITE_OK;
}
```

**Page Split (balance_nonroot):**

```c
static int balance_nonroot(
  MemPage *pParent,          // Parent page
  int iParentIdx,            // Index of child in parent
  u8 *aOvflSpace,            // Space for overflow cell
  int isRoot                 // True if pParent is root
){
  BtShared *pBt = pParent->pBt;
  MemPage *apOld[NB];        // Original pages
  MemPage *apNew[NB+2];      // New pages after split
  int nOld;                  // Number of old pages
  int nNew;                  // Number of new pages
  
  // Load sibling pages
  nOld = getSiblings(pParent, iParentIdx, apOld);
  
  // Calculate total cells and space
  int nCell = 0;
  int szTotal = 0;
  for(int i=0; i<nOld; i++){
    nCell += apOld[i]->nCell;
    szTotal += apOld[i]->nFree;
  }
  
  // Determine number of pages needed
  int szPerPage = (pBt->usableSize - 12) / (nCell + 1);
  nNew = (szTotal + szPerPage - 1) / szPerPage;
  
  // Allocate new pages
  for(int i=0; i<nNew; i++){
    apNew[i] = allocatePage(pBt);
  }
  
  // Distribute cells across new pages
  int iCell = 0;
  int iNew = 0;
  
  for(int iOld=0; iOld<nOld; iOld++){
    MemPage *pOld = apOld[iOld];
    
    for(int i=0; i<pOld->nCell; i++){
      // Check if current new page is full
      if( apNew[iNew]->nFree < cellSize ){
        iNew++;
      }
      
      // Copy cell to new page
      copyCell(pOld, i, apNew[iNew]);
      iCell++;
    }
  }
  
  // Update parent pointers
  for(int i=0; i<nNew; i++){
    insertChildPointer(pParent, iParentIdx + i, 
                       apNew[i]->pgno);
  }
  
  // Release old pages
  for(int i=0; i<nOld; i++){
    releasePage(apOld[i]);
  }
  
  return SQLITE_OK;
}
```

**Visual Example of B-Tree Split:**

```
Before Insert (Page Full):
┌────────────────────────────┐
│ [10] [20] [30] [40] [50]   │
└────────────────────────────┘

Insert 25:
┌────────────────────────────┐
│ [10] [20] [25] [30] [40] [50] ← Overflow!
└────────────────────────────┘

After Split:
        Parent
      ┌────────┐
      │  [30]  │
      └───┬─┬──┘
          │ │
    ┌─────┘ └─────┐
    │              │
┌───────────┐  ┌───────────┐
│[10][20][25]│  │[30][40][50]│
└───────────┘  └───────────┘
  New Left       New Right
```

---

## 2. Pager Layer

### 2.1 Pager Structure

```c
struct Pager {
  sqlite3_vfs *pVfs;       // OS interface
  sqlite3_file *fd;        // Database file handle
  sqlite3_file *jfd;       // Journal file handle
  
  // Page cache
  PCache *pPCache;         // Page cache object
  PgHdr *pDirty;           // List of dirty pages
  PgHdr *pAll;             // List of all pages
  
  // File information
  Pgno dbSize;             // Size of database in pages
  Pgno dbOrigSize;         // Original size before transaction
  i64 journalOff;          // Current offset in journal
  
  // Transaction state
  u8 eState;               // Pager state
  u8 eLock;                // Current lock level
  u8 journalMode;          // Journal mode (DELETE/PERSIST/WAL)
  
  // Flags
  u8 changeCountDone;      // Change counter incremented
  u8 setMaster;            // True if master journal written
  u8 doNotSpill;           // Don't spill cache
  u8 subjInMemory;         // Sub-journal in memory
  
  // Statistics
  int nHit;                // Cache hits
  int nMiss;               // Cache misses
  int nWrite;              // Pages written
};

// Pager states
#define PAGER_OPEN           0  // No transaction
#define PAGER_READER         1  // Read transaction
#define PAGER_WRITER_LOCKED  2  // Reserved lock held
#define PAGER_WRITER_CACHEMOD 3 // Cache modified
#define PAGER_WRITER_DBMOD   4  // Database modified
#define PAGER_WRITER_FINISHED 5 // Transaction committing
#define PAGER_ERROR          6  // Error state
```

### 2.2 Page Cache (PCache)

```c
// Page header
struct PgHdr {
  sqlite3_pcache_page *pPage;  // Cache page
  void *pData;                 // Page data
  void *pExtra;                // Extra data
  PgHdr *pDirty;              // Next dirty page
  Pager *pPager;              // Owning pager
  Pgno pgno;                  // Page number
  
  // Reference counting
  u16 flags;                  // Page flags
  i16 nRef;                   // Reference count
  
  // Cache management
  PgHdr *pDirtyNext;          // Next in dirty list
  PgHdr *pDirtyPrev;          // Previous in dirty list
};

// Page flags
#define PGHDR_DIRTY        0x002   // Page is dirty
#define PGHDR_NEED_SYNC    0x004   // Fsync needed
#define PGHDR_NEED_READ    0x008   // Content is invalid
#define PGHDR_DONT_WRITE   0x010   // Don't write to disk
#define PGHDR_MMAP         0x020   // Memory-mapped page
```

**Page Fetch Operation:**

```c
int sqlite3PagerGet(
  Pager *pPager,       // Pager
  Pgno pgno,           // Page number
  DbPage **ppPage,     // OUT: Page handle
  int flags            // Flags
){
  PgHdr *pPg;
  
  // Check cache first
  pPg = sqlite3PcacheFetch(pPager->pPCache, pgno, 
                           flags & PAGER_GET_NOCONTENT);
  
  if( pPg ){
    // Cache hit
    pPager->nHit++;
  }else{
    // Cache miss - read from disk
    pPager->nMiss++;
    
    pPg = sqlite3PcacheFetchFinish(pPager->pPCache, pgno, 
                                    pPager->pTmpSpace);
    
    if( (flags & PAGER_GET_NOCONTENT) == 0 ){
      // Read page from disk
      i64 offset = (pgno - 1) * pPager->pageSize;
      int rc = sqlite3OsRead(pPager->fd, pPg->pData, 
                             pPager->pageSize, offset);
      
      if( rc != SQLITE_OK ){
        sqlite3PcacheRelease(pPg);
        return rc;
      }
    }
  }
  
  // Increment reference count
  pPg->nRef++;
  *ppPage = pPg;
  
  return SQLITE_OK;
}
```

**Cache Spill (write dirty pages):**

```c
static int pagerStress(void *p, PgHdr *pPg){
  Pager *pPager = (Pager*)p;
  
  // Can't spill during certain operations
  if( pPager->doNotSpill ){
    return SQLITE_OK;
  }
  
  // Write page to journal if in transaction
  if( pPager->eState >= PAGER_WRITER_CACHEMOD ){
    if( !subjRequiresPage(pPg) ){
      rc = subjournalPage(pPg);
      if( rc ) return rc;
    }
  }
  
  // Write page to database file
  if( pPg->flags & PGHDR_DIRTY ){
    rc = pager_write_pagelist(pPager, pPg);
    if( rc ) return rc;
  }
  
  // Mark page as clean
  pPg->flags &= ~PGHDR_DIRTY;
  
  return SQLITE_OK;
}
```

---

## 3. Transaction Management

### 3.1 Rollback Journal

**Journal File Format:**

```
┌──────────────────────────────────────┐
│ Journal Header (512 bytes)           │
│  - Magic number                      │
│  - Page count                        │
│  - Random nonce                      │
│  - Initial database size             │
│  - Sector size                       │
│  - Page size                         │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ Page Record 1                        │
│  - Page number (4 bytes)             │
│  - Original page data                │
│  - Checksum (4 bytes)                │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ Page Record 2                        │
│  - Page number (4 bytes)             │
│  - Original page data                │
│  - Checksum (4 bytes)                │
└──────────────────────────────────────┘
        ... more page records ...
┌──────────────────────────────────────┐
│ Master Journal Pointer (optional)    │
│  - Path to master journal            │
│  - For multi-database transactions   │
└──────────────────────────────────────┘
```

**Begin Transaction:**

```c
static int pager_begin(Pager *pPager, int exFlag){
  // Get read lock
  int rc = pagerLockDb(pPager, SHARED_LOCK);
  if( rc ) return rc;
  
  if( exFlag ){
    // Exclusive transaction - get reserved lock
    rc = pagerLockDb(pPager, RESERVED_LOCK);
    if( rc ) return rc;
    
    // Open journal file
    rc = pager_open_journal(pPager);
    if( rc ) return rc;
    
    // Write journal header
    rc = writeJournalHdr(pPager);
    if( rc ) return rc;
    
    pPager->eState = PAGER_WRITER_LOCKED;
  }else{
    pPager->eState = PAGER_READER;
  }
  
  return SQLITE_OK;
}
```

**Write Page to Journal:**

```c
static int pager_write(PgHdr *pPg){
  Pager *pPager = pPg->pPager;
  
  // Skip if already in journal
  if( subjRequiresPage(pPg) ){
    return SQLITE_OK;
  }
  
  // Write page number
  put32bits(pPager->jfd, pPg->pgno);
  
  // Write original page data
  rc = sqlite3OsWrite(pPager->jfd, pPg->pData, 
                      pPager->pageSize,
                      pPager->journalOff + 4);
  
  // Write checksum
  u32 cksum = pager_cksum(pPager, (u8*)pPg->pData);
  put32bits(pPager->jfd + pPager->pageSize + 4, cksum);
  
  pPager->journalOff += pPager->pageSize + 8;
  
  // Add to subjection list
  subjournalPageAdded(pPg);
  
  return SQLITE_OK;
}
```

**Commit Transaction:**

```c
int sqlite3PagerCommitPhaseOne(
  Pager *pPager,
  const char *zMaster,  // Master journal name
  int noSync            // True to skip sync
){
  // Ensure all dirty pages are in journal
  rc = pager_write_pagelist(pPager, pPager->pDirty);
  if( rc ) return rc;
  
  // Write master journal pointer
  if( zMaster ){
    rc = writeMasterJournal(pPager, zMaster);
    if( rc ) return rc;
  }
  
  // Sync journal file
  if( !noSync ){
    rc = sqlite3OsSync(pPager->jfd, pPager->syncFlags);
    if( rc ) return rc;
  }
  
  // Write all dirty pages to database
  rc = pager_write_pagelist(pPager, pPager->pDirty);
  if( rc ) return rc;
  
  // Sync database file
  if( !noSync ){
    rc = sqlite3OsSync(pPager->fd, pPager->syncFlags);
    if( rc ) return rc;
  }
  
  pPager->eState = PAGER_WRITER_FINISHED;
  
  return SQLITE_OK;
}

int sqlite3PagerCommitPhaseTwo(Pager *pPager){
  // Delete journal file
  if( pPager->journalMode == PAGER_JOURNALMODE_DELETE ){
    rc = sqlite3OsDelete(pPager->pVfs, pPager->zJournal, 0);
  }else if( pPager->journalMode == PAGER_JOURNALMODE_TRUNCATE ){
    rc = sqlite3OsTruncate(pPager->jfd, 0);
  }
  
  // Release locks
  pager_unlock(pPager);
  
  pPager->eState = PAGER_OPEN;
  
  return SQLITE_OK;
}
```

**Rollback Transaction:**

```c
int sqlite3PagerRollback(Pager *pPager){
  if( pPager->eState == PAGER_OPEN ){
    return SQLITE_OK;  // No transaction
  }
  
  // Playback journal
  if( pPager->journalOpen ){
    rc = pager_playback(pPager, 0);
    if( rc ) return rc;
  }
  
  // Release all dirty pages
  sqlite3PcacheClearSyncFlags(pPager->pPCache);
  sqlite3PcacheCleanAll(pPager->pPCache);
  
  // Delete journal
  if( pPager->journalMode == PAGER_JOURNALMODE_DELETE ){
    sqlite3OsDelete(pPager->pVfs, pPager->zJournal, 0);
  }
  
  // Release locks
  pager_unlock(pPager);
  
  pPager->eState = PAGER_OPEN;
  
  return SQLITE_OK;
}
```

---

## CONTINUED IN PART 4

This is Part 3 of the SQLite Source Code Mastery Tutorial. It covers:
- B-Tree implementation (page structure, operations, splitting)
- Pager layer (caching, page management)
- Transaction management (rollback journal)

**Part 4 will cover:**
- Locking mechanisms and concurrency
- Write-Ahead Logging (WAL) mode
- Query optimization and the query planner
- Virtual tables and extensions
- Performance tuning

Save this file and continue with Part 4!