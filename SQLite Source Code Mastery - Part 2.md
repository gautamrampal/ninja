# SQLite Source Code Mastery Tutorial - PART 2
## Parser, VDBE Engine & B-Tree Deep Dive

---

## Table of Contents - Part 2
1. [SQL Parser Implementation](#1-sql-parser-implementation)
2. [Code Generation](#2-code-generation)
3. [VDBE Execution Engine](#3-vdbe-execution-engine)
4. [B-Tree Implementation](#4-b-tree-implementation)
5. [Pager and Cache Management](#5-pager-and-cache-management)
6. [Transaction Management](#6-transaction-management)

---

## 1. SQL Parser Implementation

### 1.1 Tokenizer (tokenize.c)

**Token Types:**

```c
// From parse.h (generated from parse.y)
#define TK_SEMI                1
#define TK_EXPLAIN             2
#define TK_QUERY               3
#define TK_PLAN                4
#define TK_SELECT              5
#define TK_FROM                6
#define TK_WHERE               7
#define TK_AND                 8
#define TK_OR                  9
#define TK_NOT                10
// ... 150+ token types
```

**Tokenization Process:**

```c
// Token structure
typedef struct Token {
  const char *z;     // Pointer to token text
  unsigned int n;    // Token length
} Token;

// Main tokenization function
int sqlite3GetToken(const unsigned char *z, int *tokenType){
  int i, c;
  switch( *z ){
    case ' ': case '\t': case '\n': case '\f': case '\r': {
      // Whitespace
      for(i=1; isspace(z[i]); i++){}
      *tokenType = TK_SPACE;
      return i;
    }
    case '-': {
      if( z[1]=='-' ){
        // Comment: -- to end of line
        for(i=2; z[i] && z[i]!='\n'; i++){}
        *tokenType = TK_SPACE;
        return i;
      }
      *tokenType = TK_MINUS;
      return 1;
    }
    case '(': {
      *tokenType = TK_LP;
      return 1;
    }
    case ')': {
      *tokenType = TK_RP;
      return 1;
    }
    case ';': {
      *tokenType = TK_SEMI;
      return 1;
    }
    case '+': {
      *tokenType = TK_PLUS;
      return 1;
    }
    case '*': {
      *tokenType = TK_STAR;
      return 1;
    }
    // ... many more cases
    case 'A': case 'B': case 'C': // ... case 'Z':
    case 'a': case 'b': case 'c': // ... case 'z': {
      // Identifier or keyword
      for(i=1; (c=z[i])!=0 && (isalnum(c) || c=='_'); i++){}
      *tokenType = keywordCode((char*)z, i);
      return i;
    }
    case '0': case '1': case '2': // ... case '9': {
      // Numeric literal
      *tokenType = TK_INTEGER;
      for(i=1; isdigit(z[i]); i++){}
      if( z[i]=='.' ){
        i++;
        while( isdigit(z[i]) ){ i++; }
        *tokenType = TK_FLOAT;
      }
      if( z[i]=='e' || z[i]=='E' ){
        i++;
        if( z[i]=='+' || z[i]=='-' ) i++;
        while( isdigit(z[i]) ){ i++; }
        *tokenType = TK_FLOAT;
      }
      return i;
    }
    case '\'': case '"': {
      // String literal
      int delim = z[0];
      for(i=1; z[i]; i++){
        if( z[i]==delim ){
          if( z[i+1]==delim ){
            i++; // Escaped delimiter
          }else{
            break;
          }
        }
      }
      if( z[i] ) i++;
      *tokenType = TK_STRING;
      return i;
    }
  }
  *tokenType = TK_ILLEGAL;
  return 1;
}
```

**Example Tokenization:**

```c
// SQL: SELECT name FROM users WHERE id = 5
// Tokens generated:
Token tokens[] = {
  {"SELECT", 6},  // TK_SELECT
  {" ", 1},       // TK_SPACE (ignored)
  {"name", 4},    // TK_ID
  {" ", 1},       // TK_SPACE
  {"FROM", 4},    // TK_FROM
  {" ", 1},       // TK_SPACE
  {"users", 5},   // TK_ID
  {" ", 1},       // TK_SPACE
  {"WHERE", 5},   // TK_WHERE
  {" ", 1},       // TK_SPACE
  {"id", 2},      // TK_ID
  {" ", 1},       // TK_SPACE
  {"=", 1},       // TK_EQ
  {" ", 1},       // TK_SPACE
  {"5", 1}        // TK_INTEGER
};
```

### 1.2 Parser (parse.y - Lemon Parser)

**Parse Tree Structure:**

```c
// Expression node
struct Expr {
  u8 op;                 // Operation (TK_* constant)
  u8 op2;                // Secondary operation
  u32 flags;             // Various flags
  Expr *pLeft;           // Left operand
  Expr *pRight;          // Right operand
  union {
    ExprList *pList;     // Function arguments
    Select *pSelect;     // Subquery
  } x;
  Token token;           // Token value
  int iColumn;           // Column number
  i16 iAgg;              // Aggregate info
  i16 iRightJoinTable;   // Right join table
};

// SELECT statement structure
struct Select {
  ExprList *pEList;      // SELECT clause (result columns)
  SrcList *pSrc;         // FROM clause (tables)
  Expr *pWhere;          // WHERE clause
  ExprList *pGroupBy;    // GROUP BY clause
  Expr *pHaving;         // HAVING clause
  ExprList *pOrderBy;    // ORDER BY clause
  Select *pPrior;        // Prior SELECT in compound
  Expr *pLimit;          // LIMIT clause
  Expr *pOffset;         // OFFSET clause
  u32 selFlags;          // Various flags
  int iLimit, iOffset;   // Memory registers
};
```

**Grammar Rules (parse.y):**

```yacc
// Simplified grammar examples

// SELECT statement
cmd ::= select(S). {
  sqlite3Select(pParse, S);
}

select ::= SELECT distinct(D) selcollist(L) from(F) 
           where_opt(W) groupby_opt(G) 
           having_opt(H) orderby_opt(O) limit_opt(Q). {
  Select *pSelect = sqlite3SelectNew(pParse, L, F, W, G, H, O, D, Q);
}

// FROM clause
from ::= FROM seltablist(L). {
  $$ = L;
}

// WHERE clause
where_opt ::= WHERE expr(E). {
  $$ = E;
}

// Expression
expr ::= expr(L) AND expr(R). {
  $$ = sqlite3ExprAnd(pParse->db, L, R);
}

expr ::= expr(L) EQ expr(R). {
  $$ = sqlite3PExpr(pParse, TK_EQ, L, R);
}

expr ::= ID(X). {
  $$ = sqlite3ExprAlloc(pParse->db, TK_ID, &X, 1);
}

expr ::= INTEGER(X). {
  $$ = sqlite3ExprAlloc(pParse->db, TK_INTEGER, &X, 1);
}
```

**Parse Tree Example:**

```
SQL: SELECT name FROM users WHERE age > 25

Parse Tree:
Select
├── pEList (SELECT clause)
│   └── Expr(name)
├── pSrc (FROM clause)
│   └── SrcList(users)
└── pWhere (WHERE clause)
    └── Expr(TK_GT)
        ├── pLeft: Expr(age)
        └── pRight: Expr(25)
```

---

## 2. Code Generation

### 2.1 From Parse Tree to Bytecode

**Code Generator (select.c, insert.c, etc.):**

```c
// Generate code for SELECT
int sqlite3Select(
  Parse *pParse,         // Parser context
  Select *p,             // SELECT statement to code
  SelectDest *pDest      // How to dispose of results
){
  // 1. Allocate cursors for each table
  int iTab = pParse->nTab++;
  
  // 2. Generate code to open table
  sqlite3VdbeAddOp4(v, OP_OpenRead, iTab, pTab->tnum, iDb,
                    (char*)pTab, P4_TABLE);
  
  // 3. Generate code for WHERE clause
  int iLabel = sqlite3VdbeMakeLabel(v);
  sqlite3VdbeAddOp2(v, OP_Rewind, iTab, iLabel);
  
  // 4. Loop through records
  int iLoopStart = sqlite3VdbeCurrentAddr(v);
  if( pWhere ){
    // Generate WHERE condition
    int iRegWhere = sqlite3ExprCodeTemp(pParse, pWhere, &iReg);
    sqlite3VdbeAddOp3(v, OP_IfNot, iRegWhere, iLabel, 1);
  }
  
  // 5. Generate code for result columns
  for(int i=0; i<pEList->nExpr; i++){
    Expr *pExpr = pEList->a[i].pExpr;
    int iReg = sqlite3ExprCodeTarget(pParse, pExpr, iResult+i);
  }
  
  // 6. Output row
  sqlite3VdbeAddOp2(v, OP_ResultRow, iResult, nColumn);
  
  // 7. Next record
  sqlite3VdbeAddOp2(v, OP_Next, iTab, iLoopStart);
  
  // 8. Close cursor
  sqlite3VdbeResolveLabel(v, iLabel);
  sqlite3VdbeAddOp1(v, OP_Close, iTab);
  
  return SQLITE_OK;
}
```

**Expression Code Generation:**

```c
// Generate code for an expression
int sqlite3ExprCodeTarget(
  Parse *pParse,     // Parser context
  Expr *pExpr,       // Expression to code
  int target         // Target register
){
  Vdbe *v = pParse->pVdbe;
  int op = pExpr->op;
  
  switch(op){
    case TK_INTEGER: {
      // Load integer constant
      i64 value = sqlite3Atoi64(pExpr->token.z, NULL, 0);
      sqlite3VdbeAddOp2(v, OP_Integer, (int)value, target);
      break;
    }
    case TK_STRING: {
      // Load string constant
      sqlite3VdbeAddOp4(v, OP_String8, 0, target, 0, 
                        pExpr->token.z, pExpr->token.n);
      break;
    }
    case TK_COLUMN: {
      // Load column value
      sqlite3VdbeAddOp3(v, OP_Column, pExpr->iTable, 
                        pExpr->iColumn, target);
      break;
    }
    case TK_EQ:
    case TK_NE:
    case TK_LT:
    case TK_LE:
    case TK_GT:
    case TK_GE: {
      // Comparison operators
      int r1 = sqlite3ExprCodeTemp(pParse, pExpr->pLeft, &regFree1);
      int r2 = sqlite3ExprCodeTemp(pParse, pExpr->pRight, &regFree2);
      codeCompare(pParse, pExpr->pLeft, pExpr->pRight, op,
                  r1, r2, target, SQLITE_STOREP2);
      break;
    }
    case TK_AND: {
      // Logical AND
      int r1 = sqlite3ExprCodeTemp(pParse, pExpr->pLeft, &regFree1);
      int d2 = sqlite3VdbeMakeLabel(v);
      sqlite3ExprIfFalse(pParse, pExpr->pLeft, d2, SQLITE_JUMPIFNULL);
      int r2 = sqlite3ExprCodeTemp(pParse, pExpr->pRight, &regFree2);
      sqlite3VdbeAddOp3(v, OP_And, r1, r2, target);
      sqlite3VdbeResolveLabel(v, d2);
      break;
    }
    case TK_FUNCTION: {
      // Function call
      ExprList *pArgs = pExpr->x.pList;
      int nArgs = pArgs ? pArgs->nExpr : 0;
      
      // Allocate registers for arguments
      int r1 = sqlite3GetTempRange(pParse, nArgs);
      
      // Code each argument
      for(int i=0; i<nArgs; i++){
        sqlite3ExprCode(pParse, pArgs->a[i].pExpr, r1+i);
      }
      
      // Call function
      sqlite3VdbeAddOp4(v, OP_Function, 0, r1, target,
                        (char*)pExpr->u.zToken, P4_DYNAMIC);
      break;
    }
  }
  
  return target;
}
```

### 2.2 VDBE Opcodes Reference

**Common Opcodes:**

```c
// Movement and control
OP_Init         // Initialize program
OP_Goto         // Unconditional jump
OP_Gosub        // Call subroutine
OP_Return       // Return from subroutine
OP_Halt         // Stop execution
OP_Integer      // Load integer into register
OP_String       // Load string into register
OP_Null         // Load NULL into register

// Cursor operations
OP_OpenRead     // Open cursor for reading
OP_OpenWrite    // Open cursor for writing
OP_Close        // Close cursor
OP_Rewind       // Move cursor to first entry
OP_Next         // Move cursor to next entry
OP_Prev         // Move cursor to previous entry
OP_Seek         // Seek to specific rowid
OP_SeekGE       // Seek >= key
OP_SeekGT       // Seek > key
OP_SeekLE       // Seek <= key
OP_SeekLT       // Seek < key

// Data access
OP_Column       // Read column from cursor
OP_Rowid        // Get rowid from cursor
OP_MakeRecord   // Create record from registers
OP_Insert       // Insert record
OP_Delete       // Delete record

// Arithmetic
OP_Add          // Add two values
OP_Subtract     // Subtract
OP_Multiply     // Multiply
OP_Divide       // Divide
OP_Remainder    // Modulo

// Comparison
OP_Eq           // Equal
OP_Ne           // Not equal
OP_Lt           // Less than
OP_Le           // Less than or equal
OP_Gt           // Greater than
OP_Ge           // Greater than or equal

// Conditional jumps
OP_If           // Jump if true
OP_IfNot        // Jump if false
OP_IfNull       // Jump if NULL
OP_NotNull      // Jump if not NULL

// Results
OP_ResultRow    // Output current row
OP_AggStep      // Aggregate function step
OP_AggFinal     // Aggregate function final

// Transactions
OP_Transaction  // Begin transaction
OP_Commit       // Commit transaction
OP_Rollback     // Rollback transaction
```

**Complete Example - SELECT with WHERE:**

```c
// SQL: SELECT name, age FROM users WHERE age >= 18 AND active = 1

// Generated VDBE program:
Addr  Opcode         P1    P2    P3    P4             P5  Comment
----  -------------  ----  ----  ----  -------------  --  -------
0     Init           0     15    0                    00  Start
1     OpenRead       0     2     0     3              00  root=2 iDb=0; users
2     Rewind         0     13    0                    00  
3     Column         0     2     1                    00  r[1]=users.age
4     Integer        18    2     0                    00  r[2]=18
5     Ge             2     12    1     (BINARY)       51  if r[2]>=r[1] goto 12
6     Column         0     3     3                    00  r[3]=users.active
7     Integer        1     4     0                    00  r[4]=1
8     Ne             4     12    3     (BINARY)       51  if r[4]!=r[3] goto 12
9     Column         0     1     5                    00  r[5]=users.name
10    Column         0     2     6                    00  r[6]=users.age
11    ResultRow      5     2     0                    00  output=r[5..6]
12    Next           0     3     0                    01  
13    Close          0     0     0                    00  
14    Halt           0     0     0                    00  
15    Transaction    0     0     1     0              01  usesStmtJournal=0
16    Goto           0     1     0                    00
```

---

## 3. VDBE Execution Engine

### 3.1 VDBE Core (vdbe.c)

**Main Execution Loop:**

```c
int sqlite3VdbeExec(Vdbe *p){
  Op *aOp = p->aOp;              // Opcode array
  Op *pOp;                       // Current opcode
  int pc = p->pc;                // Program counter
  Mem *aMem = p->aMem;           // Memory registers
  Mem *pIn1, *pIn2, *pIn3;       // Input registers
  Mem *pOut;                     // Output register
  int rc = SQLITE_OK;
  
  // Main execution loop
  for(; pc>=0 && pc<p->nOp; pc++){
    pOp = &aOp[pc];
    
    // Opcode dispatch
    switch(pOp->opcode){
      
      case OP_Goto: {
        // Unconditional jump
        pc = pOp->p2 - 1;
        break;
      }
      
      case OP_Integer: {
        // Load integer into register
        pOut = &aMem[pOp->p2];
        pOut->flags = MEM_Int;
        pOut->u.i = pOp->p1;
        break;
      }
      
      case OP_String8: {
        // Load string into register
        pOut = &aMem[pOp->p2];
        pOut->flags = MEM_Str|MEM_Static|MEM_Term;
        pOut->z = pOp->p4.z;
        pOut->n = pOp->p1;
        break;
      }
      
      case OP_OpenRead:
      case OP_OpenWrite: {
        // Open cursor on B-tree
        int p1 = pOp->p1;         // Cursor number
        int p2 = pOp->p2;         // Root page
        Btree *pBt = db->aDb[pOp->p3].pBt;
        
        VdbeCursor *pCur = allocateCursor(p, p1, p2, CURTYPE_BTREE);
        pCur->uc.pCursor = sqlite3BtreeCursor(pBt, p2,
                                               (pOp->opcode==OP_OpenWrite),
                                               pOp->p4.p);
        break;
      }
      
      case OP_Rewind: {
        // Move cursor to first entry
        VdbeCursor *pC = p->apCsr[pOp->p1];
        BtCursor *pCrsr = pC->uc.pCursor;
        
        rc = sqlite3BtreeFirst(pCrsr, &res);
        if( rc ) goto abort_due_to_error;
        
        if( res ){
          // Table is empty
          pc = pOp->p2 - 1;
        }
        break;
      }
      
      case OP_Column: {
        // Read column from cursor
        VdbeCursor *pC = p->apCsr[pOp->p1];
        u32 *aOffset = pC->aOffset;
        i64 payloadSize;
        u32 offset = aOffset[pOp->p2];
        
        // Get column data
        pOut = &aMem[pOp->p3];
        memset(pOut, 0, sizeof(Mem));
        
        sqlite3VdbeSerialGet(
          pC->pPayload + offset,
          pC->aType[pOp->p2],
          pOut
        );
        break;
      }
      
      case OP_Eq:
      case OP_Ne:
      case OP_Lt:
      case OP_Le:
      case OP_Gt:
      case OP_Ge: {
        // Comparison
        pIn1 = &aMem[pOp->p1];
        pIn3 = &aMem[pOp->p3];
        
        int res = sqlite3MemCompare(pIn3, pIn1, pOp->p4.pColl);
        
        switch(pOp->opcode){
          case OP_Eq:    res = res==0;     break;
          case OP_Ne:    res = res!=0;     break;
          case OP_Lt:    res = res<0;      break;
          case OP_Le:    res = res<=0;     break;
          case OP_Gt:    res = res>0;      break;
          case OP_Ge:    res = res>=0;     break;
        }
        
        if( res ){
          pc = pOp->p2 - 1;
        }
        break;
      }
      
      case OP_Add:
      case OP_Subtract:
      case OP_Multiply:
      case OP_Divide: {
        // Arithmetic
        pIn1 = &aMem[pOp->p1];
        pIn2 = &aMem[pOp->p2];
        pOut = &aMem[pOp->p3];
        
        if((pIn1->flags | pIn2->flags) & MEM_Real){
          // Floating point
          double a = sqlite3VdbeRealValue(pIn1);
          double b = sqlite3VdbeRealValue(pIn2);
          
          switch(pOp->opcode){
            case OP_Add:      b += a;       break;
            case OP_Subtract: b -= a;       break;
            case OP_Multiply: b *= a;       break;
            case OP_Divide:   b /= a;       break;
          }
          
          pOut->u.r = b;
          pOut->flags = MEM_Real;
        }else{
          // Integer
          i64 a = sqlite3VdbeIntValue(pIn1);
          i64 b = sqlite3VdbeIntValue(pIn2);
          
          switch(pOp->opcode){
            case OP_Add:      b += a;       break;
            case OP_Subtract: b -= a;       break;
            case OP_Multiply: b *= a;       break;
            case OP_Divide:   b /= a;       break;
          }
          
          pOut->u.i = b;
          pOut->flags = MEM_Int;
        }
        break;
      }
      
      case OP_ResultRow: {
        // Output result row
        Mem *pMem = &aMem[pOp->p1];
        int n = pOp->p2;
        
        // Call result callback
        if( p->xCallback ){
          rc = p->xCallback(p->pCallbackArg, n, 
                            p->azResColumn, p->aColName);
        }
        
        // Store in result set
        p->pResultSet = pMem;
        p->nResColumn = n;
        
        // Pause execution
        p->pc = pc + 1;
        return SQLITE_ROW;
      }
      
      case OP_Next: {
        // Move to next record
        VdbeCursor *pC = p->apCsr[pOp->p1];
        BtCursor *pCrsr = pC->uc.pCursor;
        
        rc = sqlite3BtreeNext(pCrsr, &res);
        if( rc ) goto abort_due_to_error;
        
        if( res==0 ){
          // More records available
          pc = pOp->p2 - 1;
        }
        break;
      }
      
      case OP_Close: {
        // Close cursor
        closeCursor(p->apCsr[pOp->p1]);
        break;
      }
      
      case OP_Halt: {
        // Stop execution
        if( pOp->p1 ){
          p->rc = pOp->p1;
        }
        goto vdbe_return;
      }
      
      default: {
        // Unknown opcode
        sqlite3SetString(&p->zErrMsg, db, 
                         "unknown opcode %d", pOp->opcode);
        rc = SQLITE_CORRUPT_BKPT;
        goto abort_due_to_error;
      }
    }
  }
  
vdbe_return:
  p->pc = pc;
  return rc;
  
abort_due_to_error:
  p->rc = rc;
  return rc;
}
```

### 3.2 Memory Cells (Mem structure)

```c
// Memory cell - can hold any SQL value
struct Mem {
  union {
    i64 i;              // Integer value
    double r;           // Real (float) value
    FuncDef *pDef;      // Function definition
    RowSet *pRowSet;    // Row set
  } u;
  
  char *z;              // String or BLOB value
  int n;                // String/BLOB length
  
  u16 flags;            // Type flags
  u8 enc;               // Text encoding
  u8 eSubtype;          // Subtype
  
  // Memory management
  void (*xDel)(void*);  // Destructor
  Mem *pScopyFrom;      // Shallow copy source
  void *pFiller;        // Padding
};

// Type flags
#define MEM_Null      0x0001   // NULL value
#define MEM_Str       0x0002   // String value
#define MEM_Int       0x0004   // Integer value
#define MEM_Real      0x0008   // Real value
#define MEM_Blob      0x0010   // BLOB value
#define MEM_Static    0x1000   // Static string
#define MEM_Ephem     0x2000   // Ephemeral string
#define MEM_Term      0x4000   // String is null-terminated
```

---

## CONTINUED IN PART 3

This is Part 2 of the SQLite Source Code Mastery Tutorial. It covers:
- SQL Parser and tokenizer implementation
- Code generation from parse tree to VDBE bytecode
- VDBE execution engine internals
- Memory cell management

**Part 3 will cover:**
- B-Tree implementation in depth
- Page management and cache
- Transaction and locking mechanisms
- WAL (Write-Ahead Logging) mode
- Query optimization

Save this file and continue with Part 3!