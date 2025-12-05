# MongoDB Complete Mastery Tutorial
## From Beginner to Advanced - Comprehensive Guide

---

## Table of Contents

### Part 1: Beginner Level
1. [Introduction to MongoDB](#1-introduction-to-mongodb)
2. [Installation and Setup](#2-installation-and-setup)
3. [MongoDB Fundamentals](#3-mongodb-fundamentals)
4. [Basic CRUD Operations](#4-basic-crud-operations)
5. [Data Types and Schema Design Basics](#5-data-types-and-schema-design-basics)
6. [Query Basics](#6-query-basics)

### Part 2: Intermediate Level
7. [Advanced Queries and Operators](#7-advanced-queries-and-operators)
8. [Indexing Fundamentals](#8-indexing-fundamentals)
9. [Aggregation Framework](#9-aggregation-framework)
10. [Data Modeling Strategies](#10-data-modeling-strategies)
11. [Transactions](#11-transactions)
12. [Replication Basics](#12-replication-basics)

### Part 3: Advanced Level
13. [Sharding and Horizontal Scaling](#13-sharding-and-horizontal-scaling)
14. [Advanced Indexing Strategies](#14-advanced-indexing-strategies)
15. [Performance Optimization](#15-performance-optimization)
16. [Advanced Aggregation Techniques](#16-advanced-aggregation-techniques)
17. [Security and Authentication](#17-security-and-authentication)
18. [Backup and Recovery](#18-backup-and-recovery)
19. [Change Streams and Real-time Data](#19-change-streams-and-real-time-data)
20. [MongoDB Architecture Deep Dive](#20-mongodb-architecture-deep-dive)

---

# Part 1: Beginner Level

## 1. Introduction to MongoDB

### 1.1 What is MongoDB?

MongoDB is a **NoSQL, document-oriented database** that stores data in flexible, JSON-like documents called BSON (Binary JSON).

**Key Characteristics:**
- Schema-less (flexible schema)
- Horizontally scalable
- High performance
- Rich query language
- Document-based storage

### 1.2 SQL vs NoSQL Comparison

```
SQL (Relational)          |  NoSQL (MongoDB)
========================  |  ========================
Database                  |  Database
Table                     |  Collection
Row                       |  Document
Column                    |  Field
Table Join                |  Embedded Documents/References
Primary Key               |  _id Field
```

### 1.3 When to Use MongoDB?

**Use MongoDB when:**
- Schema is likely to change frequently
- You need horizontal scaling
- You have hierarchical data structures
- You need high write throughput
- Real-time analytics required

**Avoid MongoDB when:**
- Complex transactions across multiple documents are critical
- You need strong ACID compliance (use SQL)
- Data is highly relational with many joins

### 1.4 CAP Theorem and MongoDB

MongoDB prioritizes:
- **Consistency** (C): Configurable
- **Availability** (A): High availability through replication
- **Partition Tolerance** (P): Handles network partitions

MongoDB is **CP or AP** depending on configuration.

---

## 2. Installation and Setup

### 2.1 Installing MongoDB

#### Windows Installation
```powershell
# Download MongoDB Community Server from mongodb.com
# Install using MSI installer
# Add to PATH: C:\Program Files\MongoDB\Server\7.0\bin
```

#### macOS Installation
```bash
# Using Homebrew
brew tap mongodb/brew
brew install mongodb-community@7.0

# Start MongoDB
brew services start mongodb-community@7.0
```

#### Linux Installation (Ubuntu)
```bash
# Import public key
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -

# Create list file
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Install MongoDB
sudo apt-get update
sudo apt-get install -y mongodb-org

# Start MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod
```

### 2.2 MongoDB Tools

**Essential Tools:**
1. **mongod** - Database server
2. **mongosh** - MongoDB Shell (interactive)
3. **mongoimport/mongoexport** - Data import/export
4. **mongodump/mongorestore** - Backup utilities
5. **MongoDB Compass** - GUI tool

### 2.3 Starting MongoDB

```bash
# Start MongoDB server
mongod --dbpath /data/db

# Connect to MongoDB shell
mongosh

# Connect to specific database
mongosh "mongodb://localhost:27017/myDatabase"
```

### 2.4 Basic Configuration

**mongod.conf example:**
```yaml
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true

systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true

net:
  port: 27017
  bindIp: 127.0.0.1

security:
  authorization: enabled
```

---

## 3. MongoDB Fundamentals

### 3.1 Database Architecture

```
MongoDB Instance
├── Database 1
│   ├── Collection 1
│   │   ├── Document 1
│   │   ├── Document 2
│   │   └── Document 3
│   └── Collection 2
├── Database 2
└── Database 3
```

### 3.2 Documents

Documents are BSON objects (similar to JSON):

```javascript
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "John Doe",
  "age": 30,
  "email": "john@example.com",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zip": "10001"
  },
  "hobbies": ["reading", "gaming", "coding"],
  "created_at": ISODate("2024-01-15T10:30:00Z")
}
```

### 3.3 Collections

Collections are groups of documents:

```javascript
// Collections don't enforce schema
// But documents in same collection typically have similar structure

users collection:
{ "_id": 1, "name": "Alice", "age": 25 }
{ "_id": 2, "name": "Bob", "age": 30, "email": "bob@test.com" } // Different fields OK
```

### 3.4 The _id Field

```javascript
// Auto-generated ObjectId (default)
_id: ObjectId("507f1f77bcf86cd799439011")

// Custom _id
_id: "user_12345"
_id: 12345
_id: { "userId": 123, "timestamp": Date.now() }

// ObjectId structure (12 bytes):
// 4-byte timestamp | 5-byte random value | 3-byte counter
```

### 3.5 Database Operations

```javascript
// Show all databases
show dbs

// Switch to/create database
use myDatabase

// Show current database
db

// Show collections
show collections

// Drop database
db.dropDatabase()

// Create collection explicitly
db.createCollection("users")

// Collection with options
db.createCollection("logs", {
  capped: true,
  size: 5242880,  // 5MB
  max: 5000
})
```

---

## 4. Basic CRUD Operations

### 4.1 Create (Insert) Operations

#### Insert One Document
```javascript
// Insert single document
db.users.insertOne({
  name: "Alice Johnson",
  age: 28,
  email: "alice@example.com",
  status: "active",
  created_at: new Date()
})

// Response:
{
  acknowledged: true,
  insertedId: ObjectId("...")
}
```

#### Insert Multiple Documents
```javascript
// Insert many documents
db.users.insertMany([
  {
    name: "Bob Smith",
    age: 35,
    email: "bob@example.com",
    status: "active"
  },
  {
    name: "Carol White",
    age: 42,
    email: "carol@example.com",
    status: "inactive"
  },
  {
    name: "David Brown",
    age: 29,
    email: "david@example.com",
    status: "active"
  }
])

// Response:
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("..."),
    '1': ObjectId("..."),
    '2': ObjectId("...")
  }
}
```

#### Insert Options
```javascript
// Ordered insert (default: true)
db.users.insertMany([...], { ordered: false })

// With write concern
db.users.insertOne({...}, { 
  writeConcern: { w: "majority", wtimeout: 5000 } 
})
```

### 4.2 Read (Query) Operations

#### Find All Documents
```javascript
// Find all
db.users.find()

// Find all with pretty print
db.users.find().pretty()

// Find all as array
db.users.find().toArray()
```

#### Find with Filter
```javascript
// Find by field
db.users.find({ name: "Alice Johnson" })

// Find by multiple fields (AND)
db.users.find({ 
  status: "active", 
  age: 28 
})

// Find with comparison operators
db.users.find({ age: { $gt: 30 } })  // age > 30
db.users.find({ age: { $gte: 30 } }) // age >= 30
db.users.find({ age: { $lt: 30 } })  // age < 30
db.users.find({ age: { $lte: 30 } }) // age <= 30
db.users.find({ age: { $ne: 30 } })  // age != 30
```

#### Find One Document
```javascript
// Find first matching document
db.users.findOne({ email: "alice@example.com" })

// Find by _id
db.users.findOne({ _id: ObjectId("507f1f77bcf86cd799439011") })
```

#### Projection (Select Fields)
```javascript
// Include specific fields (1 = include, 0 = exclude)
db.users.find({}, { name: 1, email: 1 })
// Returns: { _id: ..., name: "...", email: "..." }

// Exclude _id
db.users.find({}, { name: 1, email: 1, _id: 0 })
// Returns: { name: "...", email: "..." }

// Exclude specific fields
db.users.find({}, { password: 0, ssn: 0 })
```

#### Cursor Methods
```javascript
// Limit results
db.users.find().limit(5)

// Skip documents
db.users.find().skip(10)

// Sort (1 = ascending, -1 = descending)
db.users.find().sort({ age: -1 })

// Chain methods
db.users.find({ status: "active" })
  .sort({ age: -1 })
  .limit(10)
  .skip(20)

// Count documents
db.users.find({ status: "active" }).count()
db.users.countDocuments({ status: "active" })
```

### 4.3 Update Operations

#### Update One Document
```javascript
// Update single document
db.users.updateOne(
  { name: "Alice Johnson" },  // filter
  { $set: { age: 29 } }        // update
)

// Response:
{
  acknowledged: true,
  matchedCount: 1,
  modifiedCount: 1
}
```

#### Update Multiple Documents
```javascript
// Update all matching documents
db.users.updateMany(
  { status: "inactive" },
  { $set: { status: "archived" } }
)
```

#### Update Operators
```javascript
// $set - Set field value
db.users.updateOne(
  { _id: 1 },
  { $set: { email: "newemail@example.com" } }
)

// $unset - Remove field
db.users.updateOne(
  { _id: 1 },
  { $unset: { temporaryField: "" } }
)

// $inc - Increment numeric field
db.users.updateOne(
  { _id: 1 },
  { $inc: { age: 1 } }  // Increase age by 1
)

// $mul - Multiply
db.products.updateOne(
  { _id: 1 },
  { $mul: { price: 1.1 } }  // Increase price by 10%
)

// $rename - Rename field
db.users.updateOne(
  { _id: 1 },
  { $rename: { "name": "fullName" } }
)

// $min - Update if new value is less
db.users.updateOne(
  { _id: 1 },
  { $min: { age: 25 } }  // Set to 25 only if current > 25
)

// $max - Update if new value is greater
db.users.updateOne(
  { _id: 1 },
  { $max: { age: 50 } }  // Set to 50 only if current < 50
)

// $currentDate - Set to current date
db.users.updateOne(
  { _id: 1 },
  { $currentDate: { lastModified: true } }
)
```

#### Array Update Operators
```javascript
// $push - Add element to array
db.users.updateOne(
  { _id: 1 },
  { $push: { hobbies: "photography" } }
)

// $push with $each (multiple elements)
db.users.updateOne(
  { _id: 1 },
  { $push: { hobbies: { $each: ["swimming", "cooking"] } } }
)

// $addToSet - Add if not exists
db.users.updateOne(
  { _id: 1 },
  { $addToSet: { hobbies: "reading" } }
)

// $pop - Remove first/last element
db.users.updateOne(
  { _id: 1 },
  { $pop: { hobbies: 1 } }  // 1 = last, -1 = first
)

// $pull - Remove matching elements
db.users.updateOne(
  { _id: 1 },
  { $pull: { hobbies: "gaming" } }
)

// $pullAll - Remove multiple values
db.users.updateOne(
  { _id: 1 },
  { $pullAll: { hobbies: ["gaming", "reading"] } }
)
```

#### Update with Upsert
```javascript
// Insert if not exists, update if exists
db.users.updateOne(
  { email: "newuser@example.com" },
  { $set: { name: "New User", age: 25 } },
  { upsert: true }
)
```

#### Replace One
```javascript
// Replace entire document (except _id)
db.users.replaceOne(
  { _id: 1 },
  {
    name: "Alice Johnson",
    age: 30,
    email: "alice.new@example.com",
    status: "active"
  }
)
```

#### Find and Modify
```javascript
// Find and update (returns old document by default)
db.users.findOneAndUpdate(
  { name: "Alice Johnson" },
  { $inc: { age: 1 } },
  { returnNewDocument: true }  // Return updated document
)

// Find and replace
db.users.findOneAndReplace(
  { _id: 1 },
  { name: "Alice", age: 30 },
  { returnNewDocument: true }
)

// Find and delete
db.users.findOneAndDelete(
  { status: "archived" }
)
```

### 4.4 Delete Operations

#### Delete One Document
```javascript
// Delete first matching document
db.users.deleteOne({ name: "Alice Johnson" })

// Response:
{
  acknowledged: true,
  deletedCount: 1
}
```

#### Delete Multiple Documents
```javascript
// Delete all matching documents
db.users.deleteMany({ status: "archived" })

// Delete all documents in collection
db.users.deleteMany({})
```

#### Delete with Options
```javascript
// With write concern
db.users.deleteOne(
  { _id: 1 },
  { writeConcern: { w: "majority" } }
)
```

---

## 5. Data Types and Schema Design Basics

### 5.1 BSON Data Types

```javascript
// String
{ name: "Alice Johnson" }

// Number (Integer)
{ age: 30 }
{ count: NumberInt(42) }
{ bigNumber: NumberLong("9223372036854775807") }

// Number (Decimal)
{ price: 99.99 }
{ precise: NumberDecimal("123.456789") }

// Boolean
{ isActive: true }

// Date
{ created_at: new Date() }
{ birthday: ISODate("1995-05-15T00:00:00Z") }

// Timestamp (internal use)
{ ts: Timestamp(1638360000, 1) }

// Object/Embedded Document
{ 
  address: { 
    street: "123 Main St", 
    city: "NYC" 
  } 
}

// Array
{ hobbies: ["reading", "gaming", "coding"] }

// Array of Objects
{ 
  orders: [
    { id: 1, total: 99.99 },
    { id: 2, total: 149.99 }
  ]
}

// ObjectId
{ _id: ObjectId("507f1f77bcf86cd799439011") }

// Binary Data
{ photo: BinData(0, "base64encodeddata...") }

// Null
{ middleName: null }

// Regular Expression
{ pattern: /^test/i }

// JavaScript Code
{ code: function() { return "Hello"; } }

// MinKey / MaxKey
{ min: MinKey, max: MaxKey }
```

### 5.2 Schema Design Principles

#### Embedding vs Referencing

**Embedding (Denormalization)**
```javascript
// Good for: One-to-Few, Data accessed together

// User with embedded address
{
  _id: 1,
  name: "Alice",
  email: "alice@example.com",
  address: {
    street: "123 Main St",
    city: "New York",
    state: "NY",
    zip: "10001"
  },
  orders: [
    { orderId: 1001, total: 99.99, date: ISODate("2024-01-15") },
    { orderId: 1002, total: 149.99, date: ISODate("2024-01-20") }
  ]
}

// Advantages:
// - Single query to retrieve all data
// - Better performance for reads
// - Atomic updates

// Disadvantages:
// - Document size can grow (16MB limit)
// - Data duplication
// - Complex updates if embedded data changes
```

**Referencing (Normalization)**
```javascript
// Good for: One-to-Many, Many-to-Many, Large documents

// Users collection
{
  _id: ObjectId("user1"),
  name: "Alice",
  email: "alice@example.com"
}

// Orders collection
{
  _id: ObjectId("order1"),
  userId: ObjectId("user1"),  // Reference
  total: 99.99,
  items: [...],
  date: ISODate("2024-01-15")
}

// Advantages:
// - No data duplication
// - Smaller documents
// - Easier to update referenced data

// Disadvantages:
// - Multiple queries or $lookup needed
// - No atomicity across documents
```

### 5.3 Schema Design Patterns

#### Pattern 1: One-to-One (Embedding)
```javascript
// User with profile
{
  _id: 1,
  username: "alice",
  email: "alice@example.com",
  profile: {
    firstName: "Alice",
    lastName: "Johnson",
    bio: "Software Engineer",
    avatar: "url/to/avatar.jpg"
  }
}
```

#### Pattern 2: One-to-Few (Embedding)
```javascript
// Product with reviews (limited)
{
  _id: 1,
  name: "Laptop",
  price: 999.99,
  reviews: [
    { user: "John", rating: 5, comment: "Great!" },
    { user: "Jane", rating: 4, comment: "Good value" }
  ]
}
```

#### Pattern 3: One-to-Many (Referencing)
```javascript
// Blog posts and comments

// Posts collection
{
  _id: ObjectId("post1"),
  title: "MongoDB Guide",
  content: "...",
  author: "Alice"
}

// Comments collection
{
  _id: ObjectId("comment1"),
  postId: ObjectId("post1"),
  user: "Bob",
  text: "Great article!",
  date: ISODate("2024-01-15")
}
```

#### Pattern 4: One-to-Squillions (Referencing with Parent Reference)
```javascript
// Log system - millions of logs per server

// Servers collection
{
  _id: "server1",
  hostname: "web-server-01",
  ip: "192.168.1.100"
}

// Logs collection (parent reference)
{
  _id: ObjectId("log1"),
  serverId: "server1",  // Reference to parent
  level: "ERROR",
  message: "Connection timeout",
  timestamp: ISODate("2024-01-15T10:30:00Z")
}
```

#### Pattern 5: Many-to-Many (Array of References)
```javascript
// Students and Courses

// Students collection
{
  _id: ObjectId("student1"),
  name: "Alice",
  courseIds: [
    ObjectId("course1"),
    ObjectId("course2"),
    ObjectId("course3")
  ]
}

// Courses collection
{
  _id: ObjectId("course1"),
  name: "Database Design",
  studentIds: [
    ObjectId("student1"),
    ObjectId("student2")
  ]
}
```

### 5.4 Anti-Patterns to Avoid

```javascript
// ❌ AVOID: Massive arrays
{
  _id: 1,
  logs: [/* thousands of log entries */]  // BAD: Unbounded growth
}

// ✅ BETTER: Use separate collection
// Logs collection with reference
{ serverId: 1, message: "...", timestamp: ... }

// ❌ AVOID: Bloated documents
{
  _id: 1,
  /* hundreds of fields */  // BAD: Too many fields
}

// ✅ BETTER: Split into logical sub-documents

// ❌ AVOID: Unnecessary indexes
db.collection.createIndex({ field1: 1, field2: 1, field3: 1, ... })  // BAD

// ✅ BETTER: Index based on query patterns
```

---

## 6. Query Basics

### 6.1 Comparison Operators

```javascript
// $eq - Equal to
db.users.find({ age: { $eq: 30 } })
db.users.find({ age: 30 })  // Shorthand

// $ne - Not equal
db.users.find({ status: { $ne: "inactive" } })

// $gt - Greater than
db.products.find({ price: { $gt: 100 } })

// $gte - Greater than or equal
db.products.find({ price: { $gte: 100 } })

// $lt - Less than
db.users.find({ age: { $lt: 30 } })

// $lte - Less than or equal
db.users.find({ age: { $lte: 30 } })

// $in - Match any value in array
db.users.find({ status: { $in: ["active", "pending"] } })

// $nin - Not in array
db.users.find({ status: { $nin: ["banned", "deleted"] } })
```

### 6.2 Logical Operators

```javascript
// $and - All conditions must be true
db.users.find({
  $and: [
    { age: { $gte: 25 } },
    { age: { $lte: 35 } },
    { status: "active" }
  ]
})

// Implicit AND (shorthand)
db.users.find({ age: { $gte: 25, $lte: 35 }, status: "active" })

// $or - At least one condition must be true
db.users.find({
  $or: [
    { age: { $lt: 25 } },
    { age: { $gt: 65 } }
  ]
})

// $nor - None of the conditions should be true
db.users.find({
  $nor: [
    { status: "banned" },
    { status: "deleted" }
  ]
})

// $not - Negates condition
db.users.find({
  age: { $not: { $gte: 30 } }
})

// Complex logical queries
db.users.find({
  $and: [
    {
      $or: [
        { age: { $lt: 25 } },
        { age: { $gt: 65 } }
      ]
    },
    { status: "active" }
  ]
})
```

### 6.3 Element Operators

```javascript
// $exists - Field exists
db.users.find({ email: { $exists: true } })
db.users.find({ middleName: { $exists: false } })

// $type - Field type
db.users.find({ age: { $type: "number" } })
db.users.find({ age: { $type: "string" } })

// Type aliases
db.collection.find({ field: { $type: "double" } })      // 1
db.collection.find({ field: { $type: "string" } })      // 2
db.collection.find({ field: { $type: "object" } })      // 3
db.collection.find({ field: { $type: "array" } })       // 4
db.collection.find({ field: { $type: "objectId" } })    // 7
db.collection.find({ field: { $type: "bool" } })        // 8
db.collection.find({ field: { $type: "date" } })        // 9
db.collection.find({ field: { $type: "null" } })        // 10
```

### 6.4 Array Query Operators

```javascript
// Sample data
db.users.insertMany([
  { name: "Alice", hobbies: ["reading", "gaming", "coding"] },
  { name: "Bob", hobbies: ["sports", "gaming"] },
  { name: "Carol", hobbies: ["reading", "cooking", "travel"] }
])

// Query array contains value
db.users.find({ hobbies: "gaming" })

// $all - Array contains all specified values
db.users.find({ hobbies: { $all: ["reading", "coding"] } })

// $elemMatch - At least one array element matches all conditions
db.products.find({
  reviews: {
    $elemMatch: {
      rating: { $gte: 4 },
      verified: true
    }
  }
})

// $size - Array has specific length
db.users.find({ hobbies: { $size: 3 } })

// Array index access
db.users.find({ "hobbies.0": "reading" })  // First element is "reading"
```

### 6.5 Regex and Text Search

```javascript
// Regular expressions
db.users.find({ name: /^A/ })              // Starts with 'A'
db.users.find({ name: /son$/ })            // Ends with 'son'
db.users.find({ name: /alice/i })          // Case-insensitive

// $regex operator
db.users.find({ 
  name: { 
    $regex: "^A", 
    $options: "i" 
  } 
})

// Text search (requires text index)
db.articles.createIndex({ content: "text", title: "text" })

// Search for text
db.articles.find({ $text: { $search: "mongodb tutorial" } })

// Search with phrases
db.articles.find({ $text: { $search: "\"exact phrase\"" } })

// Exclude words
db.articles.find({ $text: { $search: "mongodb -sql" } })

// Text score
db.articles.find(
  { $text: { $search: "mongodb" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } })
```

### 6.6 Querying Nested Documents

```javascript
// Sample data
{
  _id: 1,
  name: "Alice",
  address: {
    street: "123 Main St",
    city: "New York",
    state: "NY",
    coordinates: {
      lat: 40.7128,
      lng: -74.0060
    }
  }
}

// Dot notation for nested fields
db.users.find({ "address.city": "New York" })
db.users.find({ "address.coordinates.lat": { $gt: 40 } })

// Exact match on embedded document (order matters)
db.users.find({ 
  address: { 
    street: "123 Main St", 
    city: "New York", 
    state: "NY" 
  } 
})

// Partial match (use dot notation instead)
db.users.find({ 
  "address.city": "New York",
  "address.state": "NY"
})
```

### 6.7 Projection Advanced

```javascript
// Include specific fields
db.users.find(
  {},
  { name: 1, email: 1 }
)

// Exclude specific fields
db.users.find(
  {},
  { password: 0, ssn: 0 }
)

// Array element projection
db.users.find(
  {},
  { name: 1, "hobbies": { $slice: 2 } }  // First 2 hobbies
)

// $slice with skip
db.users.find(
  {},
  { name: 1, "hobbies": { $slice: [1, 2] } }  // Skip 1, return 2
)

// $elemMatch in projection
db.students.find(
  {},
  {
    name: 1,
    grades: { $elemMatch: { score: { $gte: 90 } } }
  }
)

// $ positional operator (first matching element)
db.students.find(
  { "grades.score": { $gte: 90 } },
  { "name": 1, "grades.$": 1 }
)
```

### 6.8 Query Performance Tips

```javascript
// Use indexes for frequently queried fields
db.users.createIndex({ email: 1 })

// Covered queries (query + projection uses only index)
db.users.find(
  { email: "alice@example.com" },
  { email: 1, _id: 0 }
)

// Explain query plan
db.users.find({ email: "alice@example.com" }).explain("executionStats")

// Limit results early
db.users.find().limit(10)  // GOOD
db.users.find().toArray().slice(0, 10)  // BAD

// Use projection to reduce data transfer
db.users.find({}, { name: 1, email: 1 })  // GOOD
db.users.find()  // BAD if you only need name and email
```

---

# Part 2: Intermediate Level

## 7. Advanced Queries and Operators

### 7.1 Evaluation Operators

```javascript
// $expr - Use aggregation expressions in query
db.monthlyBudget.find({
  $expr: { $gt: ["$spent", "$budget"] }
})

// Compare two fields
db.products.find({
  $expr: { $gt: ["$salePrice", "$costPrice"] }
})

// $jsonSchema - Validate document structure
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "email", "age"],
      properties: {
        name: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        email: {
          bsonType: "string",
          pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
          description: "must be a valid email"
        },
        age: {
          bsonType: "int",
          minimum: 0,
          maximum: 150,
          description: "must be an integer between 0 and 150"
        }
      }
    }
  }
})

// $mod - Modulo operation
db.inventory.find({ qty: { $mod: [5, 0] } })  // qty divisible by 5

// $where - JavaScript expression (SLOW - avoid if possible)
db.users.find({
  $where: function() {
    return this.age > 25 && this.status === 'active';
  }
})

// $regex with advanced patterns
db.products.find({
  name: {
    $regex: /^(laptop|computer|pc)/i,
    $options: "i"
  }
})
```

### 7.2 Geospatial Queries

```javascript
// Create geospatial index
db.places.createIndex({ location: "2dsphere" })

// Sample data (GeoJSON format)
db.places.insertMany([
  {
    name: "Central Park",
    location: {
      type: "Point",
      coordinates: [-73.9654, 40.7829]  // [longitude, latitude]
    }
  },
  {
    name: "Times Square",
    location: {
      type: "Point",
      coordinates: [-73.9851, 40.7580]
    }
  }
])

// $near - Find places near a point
db.places.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-73.9667, 40.7831]
      },
      $maxDistance: 1000  // meters
    }
  }
})

// $geoWithin - Find within polygon
db.places.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [-74.0, 40.7],
          [-73.9, 40.7],
          [-73.9, 40.8],
          [-74.0, 40.8],
          [-74.0, 40.7]
        ]]
      }
    }
  }
})

// $geoIntersects - Find intersecting geometries
db.places.find({
  location: {
    $geoIntersects: {
      $geometry: {
        type: "LineString",
        coordinates: [
          [-73.9667, 40.78],
          [-73.9667, 40.79]
        ]
      }
    }
  }
})

// Legacy coordinate pairs (2d index)
db.places.createIndex({ location: "2d" })
db.places.find({ location: { $near: [-73.9667, 40.7831] } })
```

### 7.3 Bitwise Operators

```javascript
// $bitsAllClear - All bits clear
db.inventory.find({ permissions: { $bitsAllClear: [1, 5] } })

// $bitsAllSet - All bits set
db.inventory.find({ permissions: { $bitsAllSet: [1, 5] } })

// $bitsAnyClear - Any bit clear
db.inventory.find({ permissions: { $bitsAnyClear: [1, 5] } })

// $bitsAnySet - Any bit set
db.inventory.find({ permissions: { $bitsAnySet: [1, 5] } })
```

### 7.4 Update Operators Advanced

```javascript
// $setOnInsert - Set value only on insert (upsert)
db.users.updateOne(
  { email: "new@example.com" },
  {
    $set: { lastLogin: new Date() },
    $setOnInsert: { created: new Date(), status: "active" }
  },
  { upsert: true }
)

// Array update with position
db.students.updateOne(
  { _id: 1 },
  { $set: { "grades.2": 95 } }  // Update third element
)

// $ positional operator - Update first match
db.students.updateOne(
  { _id: 1, "grades.subject": "Math" },
  { $set: { "grades.$.score": 95 } }
)

// $[] - Update all array elements
db.students.updateOne(
  { _id: 1 },
  { $inc: { "grades.$[].score": 5 } }  // Add 5 to all grades
)

// $[<identifier>] - Update filtered array elements
db.students.updateOne(
  { _id: 1 },
  { $inc: { "grades.$[elem].score": 10 } },
  { arrayFilters: [{ "elem.score": { $gte: 80 } }] }
)

// Complex array update
db.students.updateOne(
  { _id: 1 },
  {
    $push: {
      grades: {
        $each: [
          { subject: "Physics", score: 88 },
          { subject: "Chemistry", score: 92 }
        ],
        $sort: { score: -1 },
        $slice: 5  // Keep only top 5
      }
    }
  }
)
```

### 7.5 Bulk Write Operations

```javascript
// Ordered bulk write (stops on first error)
db.users.bulkWrite([
  {
    insertOne: {
      document: { name: "Alice", age: 25 }
    }
  },
  {
    updateOne: {
      filter: { name: "Bob" },
      update: { $set: { age: 30 } }
    }
  },
  {
    updateMany: {
      filter: { status: "inactive" },
      update: { $set: { status: "archived" } }
    }
  },
  {
    replaceOne: {
      filter: { name: "Carol" },
      replacement: { name: "Carol", age: 35, status: "active" }
    }
  },
  {
    deleteOne: {
      filter: { status: "deleted" }
    }
  },
  {
    deleteMany: {
      filter: { created: { $lt: new Date("2020-01-01") } }
    }
  }
], { ordered: true })

// Unordered bulk write (continues on error, better performance)
db.users.bulkWrite([...], { ordered: false })

// Response
{
  acknowledged: true,
  insertedCount: 1,
  insertedIds: { '0': ObjectId("...") },
  matchedCount: 2,
  modifiedCount: 2,
  deletedCount: 3,
  upsertedCount: 0,
  upsertedIds: {}
}
```

---

## 8. Indexing Fundamentals

### 8.1 Index Types

#### Single Field Index
```javascript
// Create ascending index
db.users.createIndex({ email: 1 })

// Create descending index
db.users.createIndex({ age: -1 })

// Create index with options
db.users.createIndex(
  { email: 1 },
  { 
    unique: true,
    name: "email_unique_idx"
  }
)
```

#### Compound Index
```javascript
// Multiple fields index
db.users.createIndex({ status: 1, age: -1 })

// Query patterns that can use this index:
// ✅ { status: "active" }
// ✅ { status: "active", age: { $gt: 25 } }
// ❌ { age: { $gt: 25 } }  // Doesn't use index (not prefix)

// Index prefix rule
db.users.createIndex({ a: 1, b: 1, c: 1 })
// Can support queries on:
// - {a}
// - {a, b}
// - {a, b, c}
// But NOT {b}, {c}, or {b, c}
```

#### Multikey Index (Arrays)
```javascript
// Automatically created for array fields
db.users.createIndex({ hobbies: 1 })

// Sample query
db.users.find({ hobbies: "gaming" })  // Uses multikey index

// Compound multikey index (at most one array field)
db.users.createIndex({ hobbies: 1, status: 1 })  // ✅ OK
db.users.createIndex({ hobbies: 1, tags: 1 })    // ❌ Error if both are arrays
```

#### Text Index
```javascript
// Create text index
db.articles.createIndex({ title: "text", content: "text" })

// Only one text index per collection
db.articles.createIndex({ "$**": "text" })  // All string fields

// Weighted text index
db.articles.createIndex(
  { title: "text", content: "text" },
  { weights: { title: 10, content: 5 } }
)

// Query text index
db.articles.find({ $text: { $search: "mongodb tutorial" } })
```

#### Geospatial Indexes
```javascript
// 2dsphere index (GeoJSON)
db.places.createIndex({ location: "2dsphere" })

// 2d index (legacy)
db.places.createIndex({ location: "2d" })
```

#### Hashed Index
```javascript
// For hash-based sharding
db.users.createIndex({ _id: "hashed" })

// Query must be equality match
db.users.find({ _id: "user123" })  // Uses hashed index
db.users.find({ _id: { $gt: "user100" } })  // Doesn't use hashed index
```

### 8.2 Index Properties

```javascript
// Unique index
db.users.createIndex({ email: 1 }, { unique: true })

// Partial index (index subset of documents)
db.orders.createIndex(
  { customerId: 1, orderDate: -1 },
  { 
    partialFilterExpression: { 
      status: "active" 
    } 
  }
)

// Sparse index (only documents with the field)
db.users.createIndex(
  { phone: 1 },
  { sparse: true }
)

// TTL index (auto-delete documents)
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }  // Delete after 1 hour
)

// Case-insensitive index
db.users.createIndex(
  { email: 1 },
  { 
    collation: { 
      locale: 'en', 
      strength: 2 
    } 
  }
)

// Background index creation (deprecated in 4.2+)
db.users.createIndex({ field: 1 }, { background: true })

// Hidden index (not used by query planner)
db.users.createIndex({ field: 1 }, { hidden: true })
```

### 8.3 Index Management

```javascript
// List all indexes
db.users.getIndexes()

// Drop index
db.users.dropIndex("email_1")
db.users.dropIndex({ email: 1 })

// Drop all indexes (except _id)
db.users.dropIndexes()

// Rebuild indexes
db.users.reIndex()

// Index statistics
db.users.stats()
db.users.aggregate([{ $indexStats: {} }])

// Hide/unhide index
db.users.hideIndex("email_1")
db.users.unhideIndex("email_1")
```

### 8.4 Query Optimization with Indexes

```javascript
// Explain query execution
db.users.find({ email: "alice@example.com" }).explain("executionStats")

// Execution stats to check:
{
  executionStats: {
    executionSuccess: true,
    nReturned: 1,              // Documents returned
    executionTimeMillis: 0,    // Execution time
    totalKeysExamined: 1,      // Index keys scanned
    totalDocsExamined: 1,      // Documents scanned
    executionStages: {
      stage: "IXSCAN",         // IXSCAN = index scan, COLLSCAN = collection scan
      indexName: "email_1"
    }
  }
}

// Covered query (uses only index)
db.users.createIndex({ email: 1, name: 1 })
db.users.find(
  { email: "alice@example.com" },
  { email: 1, name: 1, _id: 0 }
).explain()
// stage: "IXSCAN", no FETCH stage

// Force index usage (rarely needed)
db.users.find({ email: "alice@example.com" }).hint({ email: 1 })

// Natural order (no index)
db.users.find().hint({ $natural: 1 })
```

### 8.5 Index Best Practices

```javascript
// ✅ DO: Create indexes for frequent queries
db.users.createIndex({ status: 1, lastLogin: -1 })

// ✅ DO: Use ESR rule (Equality, Sort, Range)
db.orders.createIndex({ 
  status: 1,      // Equality
  orderDate: -1,  // Sort
  total: 1        // Range
})

// ✅ DO: Use covered queries
db.users.find(
  { email: "test@example.com" },
  { email: 1, name: 1, _id: 0 }
)

// ❌ AVOID: Too many indexes
// Each index slows down writes and uses memory

// ❌ AVOID: Unused indexes
// Monitor and remove unused indexes

// ❌ AVOID: Redundant indexes
db.users.createIndex({ email: 1 })
db.users.createIndex({ email: 1, name: 1 })
// Second index can handle queries on {email}, first is redundant

// ✅ DO: Use projection to reduce data transfer
db.users.find({}, { name: 1, email: 1 })

// ✅ DO: Monitor index usage
db.users.aggregate([
  { $indexStats: {} }
])
```

---

## 9. Aggregation Framework

### 9.1 Aggregation Pipeline Basics

```javascript
// Pipeline structure
db.collection.aggregate([
  { stage1 },
  { stage2 },
  { stage3 }
])

// Sample data
db.orders.insertMany([
  { 
    _id: 1, 
    customer: "Alice", 
    items: [
      { product: "Laptop", price: 999, qty: 1 },
      { product: "Mouse", price: 25, qty: 2 }
    ],
    date: ISODate("2024-01-15")
  },
  { 
    _id: 2, 
    customer: "Bob", 
    items: [
      { product: "Keyboard", price: 75, qty: 1 }
    ],
    date: ISODate("2024-01-16")
  }
])
```

### 9.2 Common Pipeline Stages

#### $match - Filter Documents
```javascript
// Filter early in pipeline
db.orders.aggregate([
  {
    $match: {
      date: { $gte: ISODate("2024-01-01") },
      "items.price": { $gt: 50 }
    }
  }
])

// Use indexes (place early in pipeline)
db.orders.aggregate([
  { $match: { status: "completed" } },  // Uses index
  // ... other stages
])
```

#### $project - Shape Documents
```javascript
// Include/exclude fields
db.orders.aggregate([
  {
    $project: {
      customer: 1,
      total: 1,
      _id: 0
    }
  }
])

// Computed fields
db.orders.aggregate([
  {
    $project: {
      customer: 1,
      itemCount: { $size: "$items" },
      total: {
        $reduce: {
          input: "$items",
          initialValue: 0,
          in: { $add: ["$$value", { $multiply: ["$$this.price", "$$this.qty"] }] }
        }
      }
    }
  }
])

// Rename fields
db.orders.aggregate([
  {
    $project: {
      customerName: "$customer",
      orderDate: "$date"
    }
  }
])
```

#### $group - Group and Aggregate
```javascript
// Group by field
db.orders.aggregate([
  {
    $group: {
      _id: "$customer",
      totalOrders: { $sum: 1 },
      avgOrderValue: { $avg: "$total" }
    }
  }
])

// Group by multiple fields
db.sales.aggregate([
  {
    $group: {
      _id: {
        year: { $year: "$date" },
        month: { $month: "$date" }
      },
      totalSales: { $sum: "$amount" },
      count: { $sum: 1 }
    }
  }
])

// Group all documents
db.orders.aggregate([
  {
    $group: {
      _id: null,
      total: { $sum: "$amount" },
      avg: { $avg: "$amount" },
      max: { $max: "$amount" },
      min: { $min: "$amount" }
    }
  }
])

// Accumulator operators
db.products.aggregate([
  {
    $group: {
      _id: "$category",
      products: { $push: "$name" },              // Array of all values
      firstProduct: { $first: "$name" },         // First value
      lastProduct: { $last: "$name" },           // Last value
      uniqueTags: { $addToSet: "$tags" },        // Array of unique values
      stats: {
        $accumulator: {
          init: function() { return { sum: 0, count: 0 }; },
          accumulate: function(state, value) {
            return { sum: state.sum + value, count: state.count + 1 };
          },
          accumulateArgs: ["$price"],
          merge: function(state1, state2) {
            return {
              sum: state1.sum + state2.sum,
              count: state1.count + state2.count
            };
          },
          finalize: function(state) {
            return state.sum / state.count;
          },
          lang: "js"
        }
      }
    }
  }
])
```

#### $sort - Sort Documents
```javascript
// Sort by field
db.orders.aggregate([
  { $sort: { date: -1 } }  // Descending
])

// Sort by multiple fields
db.orders.aggregate([
  { $sort: { status: 1, date: -1 } }
])

// Sort after grouping
db.orders.aggregate([
  { $group: { _id: "$customer", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } }
])
```

#### $limit and $skip
```javascript
// Pagination
db.orders.aggregate([
  { $sort: { date: -1 } },
  { $skip: 20 },
  { $limit: 10 }
])

// Top N
db.products.aggregate([
  { $sort: { sales: -1 } },
  { $limit: 10 }
])
```

#### $unwind - Deconstruct Arrays
```javascript
// Sample data
{
  _id: 1,
  customer: "Alice",
  items: ["A", "B", "C"]
}

// Unwind array
db.orders.aggregate([
  { $unwind: "$items" }
])

// Result:
{ _id: 1, customer: "Alice", items: "A" }
{ _id: 1, customer: "Alice", items: "B" }
{ _id: 1, customer: "Alice", items: "C" }

// Unwind with index
db.orders.aggregate([
  { 
    $unwind: { 
      path: "$items",
      includeArrayIndex: "itemIndex"
    } 
  }
])

// Preserve empty arrays
db.orders.aggregate([
  { 
    $unwind: { 
      path: "$items",
      preserveNullAndEmptyArrays: true
    } 
  }
])
```

#### $lookup - Join Collections
```javascript
// Left outer join
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  }
])

// Unwind after lookup
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customer"
    }
  },
  { $unwind: "$customer" }
])

// Advanced lookup with pipeline
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      let: { order_items: "$items" },
      pipeline: [
        {
          $match: {
            $expr: {
              $in: ["$_id", "$$order_items.productId"]
            }
          }
        },
        { $project: { name: 1, price: 1 } }
      ],
      as: "productDetails"
    }
  }
])
```

#### $addFields - Add Computed Fields
```javascript
// Add new fields
db.orders.aggregate([
  {
    $addFields: {
      totalItems: { $size: "$items" },
      year: { $year: "$date" }
    }
  }
])

// Modify existing fields
db.users.aggregate([
  {
    $addFields: {
      fullName: { $concat: ["$firstName", " ", "$lastName"] }
    }
  }
])
```

#### $replaceRoot - Replace Document
```javascript
// Promote embedded document
db.orders.aggregate([
  { $unwind: "$items" },
  { $replaceRoot: { newRoot: "$items" } }
])

// Merge embedded document
db.users.aggregate([
  {
    $replaceRoot: {
      newRoot: { $mergeObjects: ["$profile", "$$ROOT"] }
    }
  },
  { $project: { profile: 0 } }
])
```

#### $facet - Multiple Pipelines
```javascript
// Run multiple aggregations
db.products.aggregate([
  {
    $facet: {
      categoryCounts: [
        { $group: { _id: "$category", count: { $sum: 1 } } }
      ],
      priceStats: [
        {
          $group: {
            _id: null,
            avgPrice: { $avg: "$price" },
            maxPrice: { $max: "$price" },
            minPrice: { $min: "$price" }
          }
        }
      ],
      topProducts: [
        { $sort: { sales: -1 } },
        { $limit: 5 },
        { $project: { name: 1, sales: 1 } }
      ]
    }
  }
])
```

#### $bucket - Group by Ranges
```javascript
// Group by price ranges
db.products.aggregate([
  {
    $bucket: {
      groupBy: "$price",
      boundaries: [0, 50, 100, 200, 500],
      default: "Other",
      output: {
        count: { $sum: 1 },
        products: { $push: "$name" }
      }
    }
  }
])

// Auto bucket
db.products.aggregate([
  {
    $bucketAuto: {
      groupBy: "$price",
      buckets: 5,
      output: {
        count: { $sum: 1 },
        avgPrice: { $avg: "$price" }
      }
    }
  }
])
```

### 9.3 Aggregation Expressions

```javascript
// Arithmetic
db.orders.aggregate([
  {
    $project: {
      total: { $multiply: ["$price", "$quantity"] },
      discount: { $subtract: ["$price", "$salePrice"] },
      tax: { $multiply: ["$total", 0.1] }
    }
  }
])

// String operations
db.users.aggregate([
  {
    $project: {
      upperName: { $toUpper: "$name" },
      lowerName: { $toLower: "$name" },
      initials: { $substrCP: ["$name", 0, 1] },
      fullName: { $concat: ["$firstName", " ", "$lastName"] }
    }
  }
])

// Date operations
db.orders.aggregate([
  {
    $project: {
      year: { $year: "$date" },
      month: { $month: "$date" },
      dayOfWeek: { $dayOfWeek: "$date" },
      formattedDate: { $dateToString: { format: "%Y-%m-%d", date: "$date" } }
    }
  }
])

// Conditional
db.products.aggregate([
  {
    $project: {
      name: 1,
      priceCategory: {
        $cond: {
          if: { $gte: ["$price", 100] },
          then: "Premium",
          else: "Standard"
        }
      },
      status: {
        $switch: {
          branches: [
            { case: { $eq: ["$stock", 0] }, then: "Out of Stock" },
            { case: { $lte: ["$stock", 10] }, then: "Low Stock" },
            { case: { $gt: ["$stock", 100] }, then: "In Stock" }
          ],
          default: "Available"
        }
      }
    }
  }
])

// Array operations
db.users.aggregate([
  {
    $project: {
      name: 1,
      hobbyCount: { $size: "$hobbies" },
      firstHobby: { $arrayElemAt: ["$hobbies", 0] },
      hasGaming: { $in: ["gaming", "$hobbies"] },
      topHobbies: { $slice: ["$hobbies", 3] }
    }
  }
])
```

### 9.4 Real-World Aggregation Examples

#### Example 1: Sales Report
```javascript
db.sales.aggregate([
  // Filter by date range
  {
    $match: {
      date: {
        $gte: ISODate("2024-01-01"),
        $lt: ISODate("2024-02-01")
      }
    }
  },
  // Group by product and calculate totals
  {
    $group: {
      _id: "$productId",
      totalQuantity: { $sum: "$quantity" },
      totalRevenue: { $sum: { $multiply: ["$price", "$quantity"] } },
      averagePrice: { $avg: "$price" },
      orderCount: { $sum: 1 }
    }
  },
  // Lookup product details
  {
    $lookup: {
      from: "products",
      localField: "_id",
      foreignField: "_id",
      as: "product"
    }
  },
  { $unwind: "$product" },
  // Shape final output
  {
    $project: {
      _id: 0,
      productName: "$product.name",
      category: "$product.category",
      totalQuantity: 1,
      totalRevenue: 1,
      averagePrice: 1,
      orderCount: 1
    }
  },
  // Sort by revenue
  { $sort: { totalRevenue: -1 } }
])
```

#### Example 2: User Activity Analytics
```javascript
db.sessions.aggregate([
  // Unwind pages visited
  { $unwind: "$pagesVisited" },
  // Group by user and date
  {
    $group: {
      _id: {
        userId: "$userId",
        date: {
          $dateToString: { format: "%Y-%m-%d", date: "$timestamp" }
        }
      },
      sessionCount: { $sum: 1 },
      totalDuration: { $sum: "$duration" },
      uniquePages: { $addToSet: "$pagesVisited" }
    }
  },
  // Calculate metrics
  {
    $project: {
      userId: "$_id.userId",
      date: "$_id.date",
      sessionCount: 1,
      avgDuration: { $divide: ["$totalDuration", "$sessionCount"] },
      pageCount: { $size: "$uniquePages" }
    }
  }
])
```

#### Example 3: E-commerce Recommendations
```javascript
// Find products frequently bought together
db.orders.aggregate([
  { $unwind: "$items" },
  {
    $lookup: {
      from: "orders",
      let: { orderId: "$_id" },
      pipeline: [
        { $match: { $expr: { $eq: ["$_id", "$$orderId"] } } },
        { $unwind: "$items" }
      ],
      as: "relatedItems"
    }
  },
  { $unwind: "$relatedItems" },
  {
    $match: {
      $expr: { $ne: ["$items.productId", "$relatedItems.items.productId"] }
    }
  },
  {
    $group: {
      _id: {
        product1: "$items.productId",
        product2: "$relatedItems.items.productId"
      },
      frequency: { $sum: 1 }
    }
  },
  { $sort: { frequency: -1 } },
  { $limit: 100 }
])
```

---

## 10. Data Modeling Strategies

### 10.1 Embedded Documents vs References

#### When to Embed
```javascript
// One-to-One: User and Profile
{
  _id: ObjectId("user1"),
  username: "alice",
  email: "alice@example.com",
  profile: {
    firstName: "Alice",
    lastName: "Johnson",
    bio: "Software Engineer",
    avatar: "url/to/avatar.jpg",
    socialLinks: {
      twitter: "@alice",
      linkedin: "alice-johnson"
    }
  }
}

// One-to-Few: Blog Post and Comments (limited)
{
  _id: ObjectId("post1"),
  title: "MongoDB Best Practices",
  content: "...",
  author: "Alice",
  comments: [
    {
      user: "Bob",
      text: "Great article!",
      date: ISODate("2024-01-15")
    },
    {
      user: "Carol",
      text: "Very helpful",
      date: ISODate("2024-01-16")
    }
  ]
}
```

#### When to Reference
```javascript
// One-to-Many: Blog Post and Many Comments
// Posts collection
{
  _id: ObjectId("post1"),
  title: "MongoDB Guide",
  content: "...",
  author: "Alice"
}

// Comments collection
{
  _id: ObjectId("comment1"),
  postId: ObjectId("post1"),
  user: "Bob",
  text: "Great article!",
  date: ISODate("2024-01-15")
}
```

### 10.2 Schema Design Patterns

#### Pattern 1: Attribute Pattern
```javascript
// Problem: Products with varying attributes
// Bad approach: Sparse schema
{
  _id: 1,
  name: "Laptop",
  screenSize: "15 inch",
  ram: "16GB",
  storage: "512GB SSD"
}
{
  _id: 2,
  name: "Book",
  author: "John Doe",
  pages: 300,
  isbn: "123-456"
}

// Good approach: Attribute pattern
{
  _id: 1,
  name: "Laptop",
  category: "Electronics",
  attributes: [
    { key: "screenSize", value: "15 inch" },
    { key: "ram", value: "16GB" },
    { key: "storage", value: "512GB SSD" }
  ]
}

// Create index on attributes
db.products.createIndex({ "attributes.key": 1, "attributes.value": 1 })

// Query
db.products.find({
  "attributes": { $elemMatch: { key: "ram", value: "16GB" } }
})
```

#### Pattern 2: Extended Reference Pattern
```javascript
// Embed frequently accessed data
// Orders collection
{
  _id: ObjectId("order1"),
  customerId: ObjectId("customer1"),
  // Embed essential customer info
  customerInfo: {
    name: "Alice Johnson",
    email: "alice@example.com"
  },
  items: [...],
  total: 299.99
}

// Full customer data in separate collection
// Customers collection
{
  _id: ObjectId("customer1"),
  name: "Alice Johnson",
  email: "alice@example.com",
  phone: "555-1234",
  address: {...},
  preferences: {...}
}
```

#### Pattern 3: Subset Pattern
```javascript
// Problem: Product with thousands of reviews
// Solution: Store recent/top reviews in product

// Products collection
{
  _id: ObjectId("product1"),
  name: "Laptop",
  price: 999.99,
  // Only recent/top reviews
  recentReviews: [
    { user: "Alice", rating: 5, date: ISODate("2024-01-20") },
    { user: "Bob", rating: 4, date: ISODate("2024-01-19") }
  ],
  reviewCount: 1523,
  avgRating: 4.5
}

// All reviews in separate collection
// Reviews collection
{
  _id: ObjectId("review1"),
  productId: ObjectId("product1"),
  user: "Alice",
  rating: 5,
  comment: "Excellent product!",
  date: ISODate("2024-01-20")
}
```

#### Pattern 4: Computed Pattern
```javascript
// Pre-calculate expensive computations
{
  _id: ObjectId("product1"),
  name: "Laptop",
  reviews: [...],  // Array of reviews
  // Computed fields
  stats: {
    totalReviews: 1523,
    avgRating: 4.5,
    ratingDistribution: {
      5: 980,
      4: 400,
      3: 100,
      2: 30,
      1: 13
    },
    lastUpdated: ISODate("2024-01-20")
  }
}

// Update computed fields when new review added
db.products.updateOne(
  { _id: ObjectId("product1") },
  {
    $push: { reviews: newReview },
    $inc: { 
      "stats.totalReviews": 1,
      [`stats.ratingDistribution.${rating}`]: 1
    },
    $set: { "stats.lastUpdated": new Date() }
  }
)
```

#### Pattern 5: Bucket Pattern
```javascript
// Problem: Time-series data (IoT sensors)
// Solution: Group documents by time buckets

// Instead of one document per reading:
{
  sensorId: "sensor1",
  timestamp: ISODate("2024-01-15T10:00:00Z"),
  temperature: 22.5
}

// Use buckets (one document per hour):
{
  sensorId: "sensor1",
  date: ISODate("2024-01-15T10:00:00Z"),
  measurements: [
    { time: "10:00:00", temperature: 22.5 },
    { time: "10:00:30", temperature: 22.6 },
    { time: "10:01:00", temperature: 22.7 },
    // ... up to 120 measurements (1 per 30 seconds)
  ],
  count: 120,
  avgTemp: 22.8,
  maxTemp: 23.5,
  minTemp: 22.1
}
```

#### Pattern 6: Schema Versioning Pattern
```javascript
// Handle schema evolution
{
  _id: ObjectId("doc1"),
  schemaVersion: 2,
  name: "Alice",
  email: "alice@example.com",
  // Version 2 fields
  preferences: {
    newsletter: true,
    notifications: false
  }
}

// Application code handles multiple versions
function getUser(doc) {
  switch(doc.schemaVersion) {
    case 1:
      return migrateV1toV2(doc);
    case 2:
      return doc;
    default:
      return migrateToLatest(doc);
  }
}
```

### 10.3 Polymorphic Pattern
```javascript
// Single collection for different entity types
// Events collection
{
  _id: ObjectId("event1"),
  type: "page_view",
  userId: ObjectId("user1"),
  timestamp: ISODate("2024-01-15T10:00:00Z"),
  // Page view specific fields
  url: "/products/laptop",
  referrer: "/home"
}
{
  _id: ObjectId("event2"),
  type: "purchase",
  userId: ObjectId("user1"),
  timestamp: ISODate("2024-01-15T10:30:00Z"),
  // Purchase specific fields
  orderId: ObjectId("order1"),
  total: 999.99,
  items: [...]
}

// Query by type
db.events.find({ type: "purchase" })

// Create compound index
db.events.createIndex({ type: 1, timestamp: -1 })
```

### 10.4 Design for Query Patterns
```javascript
// Design schema based on how data will be queried

// Example: Social media posts
// Query 1: Get user's posts
// Query 2: Get posts by hashtag
// Query 3: Get user's feed (posts from followed users)

// Posts collection
{
  _id: ObjectId("post1"),
  userId: ObjectId("user1"),
  username: "alice",  // Denormalized for performance
  content: "Learning MongoDB! #mongodb #database",
  hashtags: ["mongodb", "database"],  // Extracted for easy querying
  timestamp: ISODate("2024-01-15T10:00:00Z"),
  likes: 42,
  comments: 5
}

// Indexes for query patterns
db.posts.createIndex({ userId: 1, timestamp: -1 })  // Query 1
db.posts.createIndex({ hashtags: 1, timestamp: -1 }) // Query 2

// Feed collection (materialized view)
{
  userId: ObjectId("user2"),  // User viewing the feed
  posts: [
    {
      postId: ObjectId("post1"),
      author: "alice",
      content: "Learning MongoDB!",
      timestamp: ISODate("2024-01-15T10:00:00Z")
    },
    // ... more posts from followed users
  ],
  lastUpdated: ISODate("2024-01-15T11:00:00Z")
}
```

---

## 11. Transactions

### 11.1 ACID Transactions in MongoDB

MongoDB supports multi-document ACID transactions:
- **Atomicity**: All or nothing
- **Consistency**: Data integrity maintained
- **Isolation**: Transactions isolated from each other
- **Durability**: Committed data persists

### 11.2 Single Document Transactions
```javascript
// Single document operations are always atomic
db.accounts.updateOne(
  { _id: "account1" },
  {
    $inc: { balance: -100 },
    $push: { 
      transactions: { 
        type: "withdraw", 
        amount: 100, 
        date: new Date() 
      } 
    }
  }
)
// Both balance update and transaction log happen atomically
```

### 11.3 Multi-Document Transactions
```javascript
// Start a session
const session = db.getMongo().startSession();

try {
  // Start transaction
  session.startTransaction();
  
  const accountsCol = session.getDatabase("bank").getCollection("accounts");
  
  // Debit from account A
  accountsCol.updateOne(
    { _id: "accountA" },
    { $inc: { balance: -100 } },
    { session }
  );
  
  // Credit to account B
  accountsCol.updateOne(
    { _id: "accountB" },
    { $inc: { balance: 100 } },
    { session }
  );
  
  // Log transaction
  session.getDatabase("bank").getCollection("transfers").insertOne(
    {
      from: "accountA",
      to: "accountB",
      amount: 100,
      timestamp: new Date()
    },
    { session }
  );
  
  // Commit transaction
  session.commitTransaction();
  print("Transaction committed successfully");
  
} catch (error) {
  // Abort transaction on error
  session.abortTransaction();
  print("Transaction aborted: " + error);
} finally {
  session.endSession();
}
```

### 11.4 Transaction Options
```javascript
// Transaction with options
session.startTransaction({
  readConcern: { level: "snapshot" },
  writeConcern: { w: "majority" },
  readPreference: "primary",
  maxCommitTimeMS: 30000
});
```

### 11.5 Transactions in Application Code (Node.js)
```javascript
const { MongoClient } = require('mongodb');

async function transferMoney(fromAccount, toAccount, amount) {
  const client = new MongoClient('mongodb://localhost:27017');
  
  try {
    await client.connect();
    const session = client.startSession();
    
    const transactionOptions = {
      readPreference: 'primary',
      readConcern: { level: 'local' },
      writeConcern: { w: 'majority' }
    };
    
    try {
      await session.withTransaction(async () => {
        const accounts = client.db('bank').collection('accounts');
        
        // Check balance
        const fromDoc = await accounts.findOne(
          { _id: fromAccount },
          { session }
        );
        
        if (fromDoc.balance < amount) {
          throw new Error('Insufficient funds');
        }
        
        // Debit
        await accounts.updateOne(
          { _id: fromAccount },
          { $inc: { balance: -amount } },
          { session }
        );
        
        // Credit
        await accounts.updateOne(
          { _id: toAccount },
          { $inc: { balance: amount } },
          { session }
        );
        
      }, transactionOptions);
      
      console.log('Transfer successful');
    } finally {
      await session.endSession();
    }
  } finally {
    await client.close();
  }
}
```

### 11.6 Transaction Best Practices
```javascript
// ✅ DO: Keep transactions short
// ❌ AVOID: Long-running transactions

// ✅ DO: Use appropriate read/write concerns
session.startTransaction({
  readConcern: { level: "snapshot" },
  writeConcern: { w: "majority" }
});

// ✅ DO: Handle errors and retry logic
const maxRetries = 3;
let attempt = 0;

while (attempt < maxRetries) {
  try {
    await session.withTransaction(async () => {
      // Transaction operations
    });
    break;  // Success
  } catch (error) {
    attempt++;
    if (attempt >= maxRetries) throw error;
  }
}

// ❌ AVOID: Large transactions (> 16MB)
// ❌ AVOID: Transactions on unbounded queries
```

---

## 12. Replication Basics

### 12.1 Replica Set Architecture
```
Primary Node
    ↓ (writes)
    ↓ (replicates to)
    ↓
    ├─→ Secondary Node 1
    ├─→ Secondary Node 2
    └─→ Arbiter (voting only, no data)
```

### 12.2 Setting Up Replica Set
```bash
# Start three mongod instances
mongod --replSet rs0 --port 27017 --dbpath /data/rs0-1
mongod --replSet rs0 --port 27018 --dbpath /data/rs0-2
mongod --replSet rs0 --port 27019 --dbpath /data/rs0-3
```

```javascript
// Connect to one instance and initiate replica set
mongosh --port 27017

// Initiate replica set
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" },
    { _id: 2, host: "localhost:27019" }
  ]
})

// Check status
rs.status()

// Check configuration
rs.conf()
```

### 12.3 Replica Set Operations
```javascript
// Add member
rs.add("localhost:27020")

// Remove member
rs.remove("localhost:27020")

// Add arbiter
rs.addArb("localhost:27021")

// Step down primary (force election)
rs.stepDown()

// Check if current node is primary
db.isMaster()

// Get replica set status
rs.status()
```

### 12.4 Read Preference
```javascript
// Read from primary only (default)
db.collection.find().readPref("primary")

// Read from primary preferred
db.collection.find().readPref("primaryPreferred")

// Read from secondary
db.collection.find().readPref("secondary")

// Read from secondary preferred
db.collection.find().readPref("secondaryPreferred")

// Read from nearest (lowest latency)
db.collection.find().readPref("nearest")

// In connection string
mongodb://localhost:27017,localhost:27018,localhost:27019/?replicaSet=rs0&readPreference=secondaryPreferred
```

### 12.5 Write Concern
```javascript
// Wait for acknowledgment from primary only
db.collection.insertOne(
  { name: "Alice" },
  { writeConcern: { w: 1 } }
)

// Wait for majority
db.collection.insertOne(
  { name: "Bob" },
  { writeConcern: { w: "majority" } }
)

// Wait for specific number of replicas
db.collection.insertOne(
  { name: "Carol" },
  { writeConcern: { w: 2, wtimeout: 5000 } }
)

// Journal acknowledgment
db.collection.insertOne(
  { name: "David" },
  { writeConcern: { w: 1, j: true } }
)
```

### 12.6 Read Concern
```javascript
// Local (default) - returns latest data
db.collection.find().readConcern("local")

// Available - returns available data (may be rolled back)
db.collection.find().readConcern("available")

// Majority - returns majority-committed data
db.collection.find().readConcern("majority")

// Linearizable - reads reflect all writes
db.collection.find().readConcern("linearizable")

// Snapshot - consistent snapshot (transactions)
db.collection.find().readConcern("snapshot")
```

---

# Part 3: Advanced Level

## 13. Sharding and Horizontal Scaling

### 13.1 Sharding Architecture
```
Application
    ↓
Mongos (Query Router)
    ↓
    ├─→ Shard 1 (Replica Set)
    ├─→ Shard 2 (Replica Set)
    └─→ Shard 3 (Replica Set)
    
Config Servers (Replica Set)
  - Stores metadata and configuration
```

### 13.2 Shard Key Selection
```javascript
// Good shard keys:
// 1. High cardinality (many unique values)
// 2. Even distribution
// 3. Matches query patterns

// Example 1: userId (good for even distribution)
sh.shardCollection("mydb.users", { userId: 1 })

// Example 2: Compound shard key
sh.shardCollection("mydb.orders", { customerId: 1, orderDate: 1 })

// Example 3: Hashed shard key (even distribution)
sh.shardCollection("mydb.logs", { _id: "hashed" })

// ❌ BAD shard keys:
// - Monotonically increasing (_id, timestamp)
// - Low cardinality (status, country)
// - Non-queried fields
```

### 13.3 Setting Up Sharded Cluster
```bash
# 1. Start config servers (3 replicas)
mongod --configsvr --replSet configReplSet --port 27019 --dbpath /data/configdb1
mongod --configsvr --replSet configReplSet --port 27020 --dbpath /data/configdb2
mongod --configsvr --replSet configReplSet --port 27021 --dbpath /data/configdb3

# Initiate config server replica set
mongosh --port 27019
rs.initiate({
  _id: "configReplSet",
  configsvr: true,
  members: [
    { _id: 0, host: "localhost:27019" },
    { _id: 1, host: "localhost:27020" },
    { _id: 2, host: "localhost:27021" }
  ]
})

# 2. Start shard servers (replica sets)
# Shard 1
mongod --shardsvr --replSet shard1 --port 27022 --dbpath /data/shard1-1
mongod --shardsvr --replSet shard1 --port 27023 --dbpath /data/shard1-2

# Shard 2
mongod --shardsvr --replSet shard2 --port 27024 --dbpath /data/shard2-1
mongod --shardsvr --replSet shard2 --port 27025 --dbpath /data/shard2-2

# 3. Start mongos (query router)
mongos --configdb configReplSet/localhost:27019,localhost:27020,localhost:27021 --port 27017
```

```javascript
// Connect to mongos and add shards
mongosh --port 27017

// Add shards
sh.addShard("shard1/localhost:27022,localhost:27023")
sh.addShard("shard2/localhost:27024,localhost:27025")

// Enable sharding on database
sh.enableSharding("mydb")

// Shard collection
sh.shardCollection("mydb.users", { userId: 1 })

// Check shard status
sh.status()
```

### 13.4 Chunk Management
```javascript
// View chunks
use config
db.chunks.find({ ns: "mydb.users" }).pretty()

// Split chunk manually
sh.splitAt("mydb.users", { userId: 50000 })

// Move chunk
sh.moveChunk("mydb.users", { userId: 50000 }, "shard2")

// Balance chunks
sh.startBalancer()
sh.stopBalancer()
sh.getBalancerState()

// Set balancer window
use config
db.settings.updateOne(
  { _id: "balancer" },
  { $set: { activeWindow: { start: "01:00", stop: "05:00" } } },
  { upsert: true }
)
```

### 13.5 Zone Sharding (Tag-Aware Sharding)
```javascript
// Add tags to shards
sh.addShardTag("shard1", "US-EAST")
sh.addShardTag("shard2", "US-WEST")

// Define zone ranges
sh.addTagRange(
  "mydb.users",
  { zipCode: "00000" },
  { zipCode: "50000" },
  "US-EAST"
)

sh.addTagRange(
  "mydb.users",
  { zipCode: "50000" },
  { zipCode: "99999" },
  "US-WEST"
)
```

### 13.6 Querying Sharded Collections
```javascript
// Targeted query (includes shard key)
db.users.find({ userId: 12345 })  // Routes to specific shard

// Broadcast query (no shard key)
db.users.find({ email: "alice@example.com" })  // Queries all shards

// Explain sharded query
db.users.find({ userId: 12345 }).explain()
```

---

## 14. Advanced Indexing Strategies

### 14.1 Index Intersection
```javascript
// MongoDB can use multiple indexes
db.users.createIndex({ status: 1 })
db.users.createIndex({ age: 1 })

// Query can use both indexes
db.users.find({ status: "active", age: { $gt: 25 } })

// Check with explain
db.users.find({ status: "active", age: { $gt: 25 } }).explain()
```

### 14.2 Wildcard Indexes
```javascript
// Index all fields
db.products.createIndex({ "$**": 1 })

// Index all fields under a path
db.products.createIndex({ "attributes.$**": 1 })

// With wildcard projection
db.products.createIndex(
  { "$**": 1 },
  { 
    wildcardProjection: { 
      "attributes.color": 1, 
      "attributes.size": 1 
    } 
  }
)
```

### 14.3 Collation and Case-Insensitive Indexes
```javascript
// Create case-insensitive index
db.users.createIndex(
  { email: 1 },
  { 
    collation: { 
      locale: 'en', 
      strength: 2 
    } 
  }
)

// Query with same collation
db.users.find({ email: "ALICE@EXAMPLE.COM" })
  .collation({ locale: 'en', strength: 2 })
```

### 14.4 Index Build Options
```javascript
// Build index in foreground (default, faster)
db.collection.createIndex({ field: 1 })

// Rolling index build (no downtime)
// 1. Build on secondaries
// 2. Step down primary
// 3. Build on old primary

// Unique partial index
db.users.createIndex(
  { email: 1 },
  { 
    unique: true,
    partialFilterExpression: { 
      email: { $exists: true } 
    } 
  }
)
```

### 14.5 Index Maintenance
```javascript
// Get index sizes
db.collection.stats().indexSizes

// Rebuild fragmented indexes
db.collection.reIndex()

// Compact collection (reclaim space)
db.runCommand({ compact: 'collection' })

// Validate indexes
db.collection.validate({ full: true })
```

---

## 15. Performance Optimization

### 15.1 Query Optimization
```javascript
// Use explain() to analyze queries
db.orders.find({ status: "completed" }).explain("executionStats")

// Key metrics:
{
  executionStats: {
    executionTimeMillis: 5,
    totalKeysExamined: 100,
    totalDocsExamined: 100,
    nReturned: 100,
    executionStages: {
      stage: "IXSCAN",  // COLLSCAN is bad, IXSCAN is good
      // ...
    }
  }
}

// Optimize:
// 1. totalKeysExamined ≈ nReturned (good index selectivity)
// 2. totalDocsExamined ≈ nReturned (covered query or good index)
// 3. executionTimeMillis < acceptable threshold
```

### 15.2 Aggregation Optimization
```javascript
// ✅ GOOD: Filter early
db.orders.aggregate([
  { $match: { status: "completed" } },  // Early filter
  { $lookup: { from: "customers", ... } },
  { $group: { ... } }
])

// ❌ BAD: Filter late
db.orders.aggregate([
  { $lookup: { from: "customers", ... } },
  { $group: { ... } },
  { $match: { status: "completed" } }  // Late filter
])

// ✅ GOOD: Project early to reduce data size
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $project: { customerId: 1, total: 1 } },  // Reduce fields
  { $lookup: { ... } }
])

// Use $limit early
db.orders.aggregate([
  { $match: { ... } },
  { $sort: { date: -1 } },
  { $limit: 100 },  // Limit before expensive operations
  { $lookup: { ... } }
])

// Use allowDiskUse for large aggregations
db.orders.aggregate(
  [...],
  { allowDiskUse: true }
)
```

### 15.3 Connection Pooling
```javascript
// Node.js example
const { MongoClient } = require('mongodb');

const client = new MongoClient('mongodb://localhost:27017', {
  maxPoolSize: 50,
  minPoolSize: 10,
  maxIdleTimeMS: 30000,
  waitQueueTimeoutMS: 5000
});

await client.connect();
```

### 15.4 Document Design for Performance
```javascript
// ✅ GOOD: Embed frequently accessed data
{
  _id: 1,
  userId: 123,
  userName: "Alice",  // Denormalized
  orderItems: [...]    // Embedded
}

// ❌ BAD: Unbounded arrays
{
  _id: 1,
  logs: [/* thousands of items */]  // Document growth
}

// ✅ GOOD: Keep documents small
// Average document size < 1KB is ideal

// Use projection to fetch only needed fields
db.users.find({}, { name: 1, email: 1 })
```

### 15.5 Monitoring and Profiling
```javascript
// Enable profiling
db.setProfilingLevel(2)  // 0=off, 1=slow queries, 2=all queries

// Set slow query threshold
db.setProfilingLevel(1, { slowms: 100 })

// View profiled queries
db.system.profile.find().sort({ ts: -1 }).limit(10)

// Database statistics
db.stats()

// Collection statistics
db.collection.stats()

// Current operations
db.currentOp()

// Server status
db.serverStatus()

// Kill slow operation
db.killOp(opid)
```

### 15.6 Caching Strategies
```javascript
// Application-level caching (Redis example)
const redis = require('redis');
const client = redis.createClient();

async function getUser(userId) {
  // Check cache first
  const cached = await client.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Query MongoDB
  const user = await db.users.findOne({ _id: userId });
  
  // Cache result
  await client.setEx(
    `user:${userId}`,
    3600,  // TTL in seconds
    JSON.stringify(user)
  );
  
  return user;
}

// Invalidate cache on update
async function updateUser(userId, updates) {
  await db.users.updateOne({ _id: userId }, { $set: updates });
  await client.del(`user:${userId}`);
}
```

---

## 16. Advanced Aggregation Techniques

### 16.1 Window Functions
```javascript
// Cumulative sum
db.sales.aggregate([
  { $sort: { date: 1 } },
  {
    $setWindowFields: {
      sortBy: { date: 1 },
      output: {
        cumulativeTotal: {
          $sum: "$amount",
          window: {
            documents: ["unbounded", "current"]
          }
        }
      }
    }
  }
])

// Moving average
db.sales.aggregate([
  { $sort: { date: 1 } },
  {
    $setWindowFields: {
      sortBy: { date: 1 },
      output: {
        movingAvg: {
          $avg: "$amount",
          window: {
            documents: [-6, 0]  // Last 7 days including current
          }
        }
      }
    }
  }
])

// Rank
db.students.aggregate([
  {
    $setWindowFields: {
      sortBy: { score: -1 },
      output: {
        rank: { $rank: {} }
      }
    }
  }
])
```

### 16.2 GraphQL-style Queries with $facet
```javascript
db.products.aggregate([
  {
    $facet: {
      // Pagination
      data: [
        { $sort: { name: 1 } },
        { $skip: 0 },
        { $limit: 20 }
      ],
      // Total count
      total: [
        { $count: "count" }
      ],
      // Categories
      categories: [
        { $group: { _id: "$category", count: { $sum: 1 } } },
        { $sort: { count: -1 } }
      ],
      // Price ranges
      priceRanges: [
        {
          $bucket: {
            groupBy: "$price",
            boundaries: [0, 50, 100, 200, 500],
            default: "Other"
          }
        }
      ]
    }
  }
])
```

### 16.3 Complex Data Transformations
```javascript
// Pivot data
db.sales.aggregate([
  {
    $group: {
      _id: { 
        product: "$product",
        month: { $month: "$date" }
      },
      total: { $sum: "$amount" }
    }
  },
  {
    $group: {
      _id: "$_id.product",
      sales: {
        $push: {
          month: "$_id.month",
          total: "$total"
        }
      }
    }
  },
  {
    $project: {
      product: "$_id",
      jan: {
        $arrayElemAt: [
          "$sales.total",
          { $indexOfArray: ["$sales.month", 1] }
        ]
      },
      feb: {
        $arrayElemAt: [
          "$sales.total",
          { $indexOfArray: ["$sales.month", 2] }
        ]
      }
      // ... more months
    }
  }
])
```

### 16.4 Recursive Aggregation (Tree Structures)
```javascript
// Employee hierarchy
db.employees.aggregate([
  {
    $graphLookup: {
      from: "employees",
      startWith: "$_id",
      connectFromField: "_id",
      connectToField: "managerId",
      as: "subordinates",
      maxDepth: 5,
      depthField: "level"
    }
  }
])
```

---

## 17. Security and Authentication

### 17.1 Enable Authentication
```javascript
// Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: "securePassword123",
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" }
  ]
})

// Restart mongod with --auth
mongod --auth --port 27017 --dbpath /data/db

// Connect with authentication
mongosh -u admin -p securePassword123 --authenticationDatabase admin
```

### 17.2 User Management
```javascript
// Create database user
use myapp
db.createUser({
  user: "appUser",
  pwd: "appPassword",
  roles: [
    { role: "readWrite", db: "myapp" }
  ]
})

// Create read-only user
db.createUser({
  user: "analyst",
  pwd: "analystPassword",
  roles: [
    { role: "read", db: "myapp" }
  ]
})

// Custom role
db.createRole({
  role: "customRole",
  privileges: [
    {
      resource: { db: "myapp", collection: "users" },
      actions: ["find", "update"]
    },
    {
      resource: { db: "myapp", collection: "logs" },
      actions: ["find"]
    }
  ],
  roles: []
})

// Grant role to user
db.grantRolesToUser("appUser", ["customRole"])

// Revoke role
db.revokeRolesFromUser("appUser", ["customRole"])

// Update user password
db.changeUserPassword("appUser", "newPassword")

// Drop user
db.dropUser("appUser")
```

### 17.3 Built-in Roles
```javascript
// Database Roles:
// - read: Read data from all non-system collections
// - readWrite: Read and write data
// - dbAdmin: Database administration
// - dbOwner: Database owner
// - userAdmin: Create and modify users

// Cluster Roles:
// - clusterAdmin: Full cluster administration
// - clusterManager: Manage cluster
// - clusterMonitor: Read-only cluster monitoring
// - hostManager: Monitor and manage servers

// Backup Roles:
// - backup: Backup database
// - restore: Restore database

// All-Database Roles:
// - readAnyDatabase
// - readWriteAnyDatabase
// - userAdminAnyDatabase
// - dbAdminAnyDatabase

// Superuser:
// - root: Full access
```

### 17.4 Encryption
```bash
# Encryption at rest
mongod --enableEncryption \
  --encryptionKeyFile /path/to/keyfile \
  --encryptionCipherMode AES256-CBC

# TLS/SSL
mongod --tlsMode requireTLS \
  --tlsCertificateKeyFile /path/to/cert.pem \
  --tlsCAFile /path/to/ca.pem

# Connect with TLS
mongosh --tls \
  --tlsCertificateKeyFile /path/to/client-cert.pem \
  --tlsCAFile /path/to/ca.pem \
  --host mongodb.example.com
```

### 17.5 Field-Level Encryption
```javascript
const { ClientEncryption } = require('mongodb-client-encryption');

const client = new MongoClient(uri);
const encryption = new ClientEncryption(client, {
  keyVaultNamespace: 'encryption.__keyVault',
  kmsProviders: {
    local: {
      key: Buffer.from(localMasterKey, 'base64')
    }
  }
});

// Create data key
const dataKeyId = await encryption.createDataKey('local');

// Encrypt field
const encryptedSSN = await encryption.encrypt(
  '123-45-6789',
  {
    algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic',
    keyId: dataKeyId
  }
);

// Insert encrypted data
await db.users.insertOne({
  name: 'Alice',
  ssn: encryptedSSN
});

// Automatic decryption with configured client
const user = await db.users.findOne({ name: 'Alice' });
console.log(user.ssn);  // Automatically decrypted
```

### 17.6 Auditing
```javascript
// Enable auditing
mongod --auditDestination file \
  --auditFormat JSON \
  --auditPath /var/log/mongodb/audit.json

// Audit filter (config file)
auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/audit.json
  filter: '{ "atype": { "$in": ["authenticate", "createUser", "dropUser"] } }'
```

---

## 18. Backup and Recovery

### 18.1 Backup Methods
```bash
# 1. mongodump (logical backup)
mongodump --uri="mongodb://localhost:27017" --out=/backup/$(date +%Y%m%d)

# Backup specific database
mongodump --db=myapp --out=/backup/myapp

# Backup specific collection
mongodump --db=myapp --collection=users --out=/backup/users

# Compressed backup
mongodump --gzip --archive=/backup/backup.gz

# With authentication
mongodump --username=admin --password=pass --authenticationDatabase=admin --out=/backup
```

```bash
# 2. mongorestore
mongorestore /backup/20240115

# Restore specific database
mongorestore --db=myapp /backup/myapp

# Drop existing data before restore
mongorestore --drop /backup/20240115

# Restore from archive
mongorestore --gzip --archive=/backup/backup.gz
```

```bash
# 3. File system snapshots (best for replica sets)
# Stop writes
db.fsyncLock()

# Create snapshot (example with LVM)
lvcreate --size 10G --snapshot --name mongodb-snap /dev/vg0/mongodb

# Resume writes
db.fsyncUnlock()

# Mount and copy snapshot
mount /dev/vg0/mongodb-snap /mnt/snapshot
cp -r /mnt/snapshot /backup
umount /mnt/snapshot
lvremove /dev/vg0/mongodb-snap
```

### 18.2 Point-in-Time Recovery
```bash
# Enable oplog for replica sets
# Oplog is automatically available

# Backup with oplog
mongodump --oplog --out=/backup/pit

# Restore to specific point in time
mongorestore --oplogReplay \
  --oplogLimit 1234567890:1 \
  /backup/pit
```

### 18.3 Cloud Backups (MongoDB Atlas)
```javascript
// Automated backups in Atlas
// - Continuous backups (oplog-based)
// - Snapshot backups
// - Cross-region backups
// - Configurable retention policies

// Manual snapshot via API
const axios = require('axios');

await axios.post(
  'https://cloud.mongodb.com/api/atlas/v1.0/groups/{GROUP-ID}/clusters/{CLUSTER-NAME}/backup/snapshots',
  {
    description: 'Manual backup before migration',
    retentionDays: 7
  },
  {
    auth: {
      username: publicKey,
      password: privateKey
    }
  }
);
```

### 18.4 Disaster Recovery Plan
```javascript
// 1. Regular automated backups
// 2. Store backups in different location/region
// 3. Test recovery procedures regularly
// 4. Document recovery procedures
// 5. Monitor backup success/failure

// Example backup script
#!/bin/bash
BACKUP_DIR="/backup/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Create backup
mongodump --out="$BACKUP_DIR/$DATE"

# Compress
tar -czf "$BACKUP_DIR/$DATE.tar.gz" "$BACKUP_DIR/$DATE"
rm -rf "$BACKUP_DIR/$DATE"

# Upload to S3
aws s3 cp "$BACKUP_DIR/$DATE.tar.gz" s3://my-backup-bucket/mongodb/

# Remove old backups
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
```

---

## 19. Change Streams and Real-time Data

### 19.1 Change Streams Basics
```javascript
// Watch entire collection
const changeStream = db.orders.watch();

changeStream.on('change', (change) => {
  console.log('Change detected:', change);
});

// Change document structure:
{
  _id: { /* Resume token */ },
  operationType: 'insert',  // insert, update, replace, delete
  clusterTime: Timestamp(1, 1638360000),
  ns: { db: 'mydb', coll: 'orders' },
  documentKey: { _id: ObjectId('...') },
  fullDocument: { /* Complete document */ },
  updateDescription: {
    updatedFields: { status: 'shipped' },
    removedFields: []
  }
}
```

### 19.2 Change Stream Filters
```javascript
// Filter by operation type
const pipeline = [
  { $match: { operationType: 'insert' } }
];
const changeStream = db.orders.watch(pipeline);

// Filter by field values
const pipeline = [
  {
    $match: {
      operationType: 'update',
      'updateDescription.updatedFields.status': 'shipped'
    }
  }
];

// Complex filters
const pipeline = [
  {
    $match: {
      $or: [
        { operationType: 'insert' },
        {
          operationType: 'update',
          'updateDescription.updatedFields.total': { $gte: 1000 }
        }
      ]
    }
  }
];
```

### 19.3 Resume Token and Resumability
```javascript
// Save resume token
let resumeToken;
const changeStream = db.orders.watch();

changeStream.on('change', (change) => {
  resumeToken = change._id;
  // Process change
});

// Resume from saved token
const newChangeStream = db.orders.watch([], {
  resumeAfter: resumeToken
});

// Start from specific time
const changeStream = db.orders.watch([], {
  startAtOperationTime: Timestamp(1, 1638360000)
});
```

### 19.4 Real-time Application Example (Node.js)
```javascript
const { MongoClient } = require('mongodb');
const express = require('express');
const WebSocket = require('ws');

const app = express();
const wss = new WebSocket.Server({ port: 8080 });

const client = await MongoClient.connect('mongodb://localhost:27017');
const db = client.db('myapp');

// Watch for changes
const changeStream = db.collection('orders').watch([
  { $match: { operationType: { $in: ['insert', 'update'] } } }
]);

// Broadcast changes to WebSocket clients
changeStream.on('change', (change) => {
  const message = JSON.stringify({
    type: change.operationType,
    data: change.fullDocument
  });
  
  wss.clients.forEach((client) => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(message);
    }
  });
});

// Handle new WebSocket connections
wss.on('connection', (ws) => {
  console.log('Client connected');
  
  ws.on('close', () => {
    console.log('Client disconnected');
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### 19.5 Change Streams for Caching
```javascript
const Redis = require('redis');
const redisClient = Redis.createClient();

// Invalidate cache on changes
const changeStream = db.users.watch();

changeStream.on('change', async (change) => {
  const userId = change.documentKey._id.toString();
  
  switch (change.operationType) {
    case 'update':
    case 'replace':
      // Update cache
      await redisClient.setEx(
        `user:${userId}`,
        3600,
        JSON.stringify(change.fullDocument)
      );
      break;
      
    case 'delete':
      // Remove from cache
      await redisClient.del(`user:${userId}`);
      break;
  }
});
```

---

## 20. MongoDB Architecture Deep Dive

### 20.1 Storage Engine (WiredTiger)
```javascript
// WiredTiger configuration
storage:
  engine: wiredTiger
  wiredTiger:
    engineConfig:
      cacheSizeGB: 4
      journalCompressor: snappy
      directoryForIndexes: true
    collectionConfig:
      blockCompressor: snappy
    indexConfig:
      prefixCompression: true
```

### 20.2 Memory Management
```javascript
// Memory usage:
// - WiredTiger cache: 50% of (RAM - 1GB) or 256MB, whichever is larger
// - MongoDB process: Additional overhead
// - OS file system cache: Remaining RAM

// Monitor memory usage
db.serverStatus().wiredTiger.cache

// Set cache size
mongod --wiredTigerCacheSizeGB 4
```

### 20.3 Journal and Durability
```javascript
// Journal commits every 100ms by default
storage:
  journal:
    enabled: true
    commitIntervalMs: 100

// Write concern with journal
db.collection.insertOne(
  { data: "important" },
  { writeConcern: { w: 1, j: true } }
)
```

### 20.4 Compression
```javascript
// Block compression (storage)
// - snappy (default, balanced)
// - zlib (higher compression, slower)
// - zstd (best compression)

// Create collection with compression
db.createCollection("logs", {
  storageEngine: {
    wiredTiger: {
      configString: "block_compressor=zstd"
    }
  }
})

// Index compression
db.collection.createIndex(
  { field: 1 },
  { storageEngine: { wiredTiger: { configString: "prefix_compression=true" } } }
)
```

### 20.5 Connection Pooling and Concurrency
```javascript
// Connection limits
net:
  maxIncomingConnections: 65536

// Tickets (concurrent operations)
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 4
    concurrentTransactions:
      read: 128
      write: 128

// Monitor connections
db.serverStatus().connections
```

### 20.6 Diagnostic Tools
```javascript
// Database profiler
db.setProfilingLevel(1, { slowms: 100 })
db.system.profile.find().sort({ ts: -1 }).limit(10)

// Explain plan
db.collection.find({ field: value }).explain("executionStats")

// Index statistics
db.collection.aggregate([{ $indexStats: {} }])

// Collection statistics
db.collection.stats()

// Database statistics
db.stats()

// Server statistics
db.serverStatus()

// Current operations
db.currentOp({
  "active": true,
  "secs_running": { $gt: 10 }
})

// Log verbosity
db.setLogLevel(2, "query")

// Validate collection
db.collection.validate({ full: true })
```

---

## 21. MongoDB Best Practices Summary

### 21.1 Schema Design
```javascript
// ✅ DO:
// - Embed data that is accessed together
// - Use references for large or independently accessed data
// - Design for your query patterns
// - Keep documents under 16MB
// - Use schema versioning for evolving schemas

// ❌ AVOID:
// - Massive arrays (unbounded growth)
// - Deeply nested documents (>100 levels)
// - Redundant indexes
// - High cardinality shard keys with monotonic increase
```

### 21.2 Indexing
```javascript
// ✅ DO:
// - Create indexes for frequent queries
// - Use compound indexes wisely (ESR rule)
// - Use covered queries when possible
// - Monitor index usage with $indexStats
// - Remove unused indexes

// ❌ AVOID:
// - Over-indexing (each index slows writes)
// - Indexes on fields with low cardinality
// - Redundant indexes
```

### 21.3 Query Optimization
```javascript
// ✅ DO:
// - Use projection to limit returned fields
// - Filter early in aggregation pipelines
// - Use $limit to reduce processed documents
// - Analyze queries with explain()
// - Use indexes effectively

// ❌ AVOID:
// - Full collection scans
// - Large in-memory sorts
// - Querying without indexes
// - Fetching entire documents when only few fields needed
```

### 21.4 Application Development
```javascript
// ✅ DO:
// - Use connection pooling
// - Handle errors and timeouts
// - Use appropriate write/read concerns
// - Validate data before inserting
// - Use transactions when needed

// ❌ AVOID:
// - Opening new connection for each operation
// - Ignoring errors
// - Large transactions (>1000 documents)
// - Unbounded queries without limits
```

### 21.5 Production Deployment
```javascript
// ✅ DO:
// - Use replica sets (minimum 3 nodes)
// - Enable authentication and encryption
// - Regular backups with testing
// - Monitor performance metrics
// - Set up alerts for critical metrics
// - Use sharding for horizontal scaling
// - Keep MongoDB updated

// ❌ AVOID:
// - Running single node in production
// - Default security settings
// - Untested backups
// - Ignoring performance warnings
```

---

## 22. MongoDB with Programming Languages

### 22.1 Node.js Driver
```javascript
const { MongoClient } = require('mongodb');

// Connection
const client = new MongoClient('mongodb://localhost:27017', {
  maxPoolSize: 50,
  serverSelectionTimeoutMS: 5000
});

await client.connect();
const db = client.db('myapp');
const collection = db.collection('users');

// CRUD operations
await collection.insertOne({ name: 'Alice', age: 25 });
await collection.find({ age: { $gt: 20 } }).toArray();
await collection.updateOne({ name: 'Alice' }, { $set: { age: 26 } });
await collection.deleteOne({ name: 'Alice' });

// Transactions
const session = client.startSession();
try {
  await session.withTransaction(async () => {
    await collection.insertOne({ name: 'Bob' }, { session });
    await collection.updateOne({ name: 'Alice' }, { $inc: { age: 1 } }, { session });
  });
} finally {
  await session.endSession();
}

await client.close();
```

### 22.2 Python (PyMongo)
```python
from pymongo import MongoClient
from bson import ObjectId

# Connection
client = MongoClient('mongodb://localhost:27017', maxPoolSize=50)
db = client['myapp']
collection = db['users']

# CRUD operations
collection.insert_one({'name': 'Alice', 'age': 25})
users = collection.find({'age': {'$gt': 20}})
collection.update_one({'name': 'Alice'}, {'$set': {'age': 26}})
collection.delete_one({'name': 'Alice'})

# Aggregation
pipeline = [
    {'$match': {'status': 'active'}},
    {'$group': {'_id': '$city', 'count': {'$sum': 1}}},
    {'$sort': {'count': -1}}
]
results = list(collection.aggregate(pipeline))

# Transactions
with client.start_session() as session:
    with session.start_transaction():
        collection.insert_one({'name': 'Bob'}, session=session)
        collection.update_one({'name': 'Alice'}, {'$inc': {'age': 1}}, session=session)

client.close()
```

### 22.3 Java Driver
```java
import com.mongodb.client.*;
import org.bson.Document;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;

// Connection
MongoClient mongoClient = MongoClients.create("mongodb://localhost:27017");
MongoDatabase database = mongoClient.getDatabase("myapp");
MongoCollection<Document> collection = database.getCollection("users");

// CRUD operations
Document user = new Document("name", "Alice").append("age", 25);
collection.insertOne(user);

FindIterable<Document> users = collection.find(Filters.gt("age", 20));

collection.updateOne(
    Filters.eq("name", "Alice"),
    Updates.set("age", 26)
);

collection.deleteOne(Filters.eq("name", "Alice"));

// Transactions
ClientSession session = mongoClient.startSession();
try {
    session.startTransaction();
    collection.insertOne(session, new Document("name", "Bob"));
    collection.updateOne(session, 
        Filters.eq("name", "Alice"),
        Updates.inc("age", 1)
    );
    session.commitTransaction();
} catch (Exception e) {
    session.abortTransaction();
} finally {
    session.close();
}

mongoClient.close();
```

### 22.4 Go Driver
```go
package main

import (
    "context"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
    "go.mongodb.org/mongo-driver/bson"
)

func main() {
    client, err := mongo.Connect(context.TODO(), options.Client().ApplyURI("mongodb://localhost:27017"))
    if err != nil {
        panic(err)
    }
    defer client.Disconnect(context.TODO())
    
    collection := client.Database("myapp").Collection("users")
    
    // Insert
    user := bson.M{"name": "Alice", "age": 25}
    collection.InsertOne(context.TODO(), user)
    
    // Find
    cursor, _ := collection.Find(context.TODO(), bson.M{"age": bson.M{"$gt": 20}})
    defer cursor.Close(context.TODO())
    
    // Update
    collection.UpdateOne(
        context.TODO(),
        bson.M{"name": "Alice"},
        bson.M{"$set": bson.M{"age": 26}},
    )
    
    // Delete
    collection.DeleteOne(context.TODO(), bson.M{"name": "Alice"})
}
```

---

## 23. Common Use Cases and Patterns

### 23.1 E-commerce Platform
```javascript
// Products
{
  _id: ObjectId("product1"),
  sku: "LAP-001",
  name: "Gaming Laptop",
  category: "Electronics",
  price: 999.99,
  inventory: {
    quantity: 50,
    warehouse: "US-EAST"
  },
  specifications: {
    ram: "16GB",
    storage: "512GB SSD",
    processor: "Intel i7"
  },
  reviews: [
    { user: "user1", rating: 5, comment: "Excellent!" }
  ],
  avgRating: 4.8,
  totalReviews: 234
}

// Orders
{
  _id: ObjectId("order1"),
  userId: ObjectId("user1"),
  status: "processing",
  items: [
    {
      productId: ObjectId("product1"),
      sku: "LAP-001",
      name: "Gaming Laptop",
      price: 999.99,
      quantity: 1
    }
  ],
  subtotal: 999.99,
  tax: 79.99,
  shipping: 19.99,
  total: 1099.97,
  shippingAddress: { /*...*/ },
  createdAt: ISODate("2024-01-15"),
  updatedAt: ISODate("2024-01-15")
}

// Decrease inventory when order placed
db.products.updateOne(
  { _id: ObjectId("product1"), "inventory.quantity": { $gte: 1 } },
  { $inc: { "inventory.quantity": -1 } }
)
```

### 23.2 Social Media Application
```javascript
// Users
{
  _id: ObjectId("user1"),
  username: "alice",
  email: "alice@example.com",
  profile: {
    displayName: "Alice Johnson",
    bio: "Software Engineer",
    avatar: "url/to/avatar.jpg"
  },
  followers: [ObjectId("user2"), ObjectId("user3")],
  following: [ObjectId("user2")],
  stats: {
    followerCount: 2,
    followingCount: 1,
    postCount: 42
  },
  createdAt: ISODate("2024-01-01")
}

// Posts
{
  _id: ObjectId("post1"),
  userId: ObjectId("user1"),
  username: "alice",  // Denormalized
  content: "Learning MongoDB!",
  hashtags: ["mongodb", "database"],
  media: [
    { type: "image", url: "url/to/image.jpg" }
  ],
  likes: {
    count: 42,
    users: [ObjectId("user2"), ObjectId("user3")]
  },
  comments: [
    {
      userId: ObjectId("user2"),
      username: "bob",
      text: "Great post!",
      createdAt: ISODate("2024-01-15T10:30:00Z")
    }
  ],
  createdAt: ISODate("2024-01-15T10:00:00Z")
}

// Get user feed (posts from followed users)
db.posts.find({
  userId: { $in: user.following }
}).sort({ createdAt: -1 }).limit(20)
```

### 23.3 IoT and Time-Series Data
```javascript
// Sensor data with bucketing pattern
{
  sensorId: "temp-sensor-01",
  location: "warehouse-A",
  date: ISODate("2024-01-15T10:00:00Z"),
  measurements: [
    { timestamp: ISODate("2024-01-15T10:00:00Z"), value: 22.5 },
    { timestamp: ISODate("2024-01-15T10:00:30Z"), value: 22.6 },
    { timestamp: ISODate("2024-01-15T10:01:00Z"), value: 22.7 }
  ],
  count: 120,
  stats: {
    min: 22.1,
    max: 23.5,
    avg: 22.8
  }
}

// Time-series collections (MongoDB 5.0+)
db.createCollection("weather", {
  timeseries: {
    timeField: "timestamp",
    metaField: "metadata",
    granularity: "hours"
  }
})

db.weather.insertOne({
  timestamp: ISODate("2024-01-15T10:00:00Z"),
  metadata: { sensorId: "sensor-01", location: "NYC" },
  temperature: 22.5,
  humidity: 65
})
```

### 23.4 Content Management System
```javascript
// Articles
{
  _id: ObjectId("article1"),
  slug: "mongodb-best-practices",
  title: "MongoDB Best Practices",
  content: "Full article content...",
  excerpt: "Learn MongoDB best practices...",
  author: {
    id: ObjectId("user1"),
    name: "Alice Johnson",
    avatar: "url/to/avatar.jpg"
  },
  category: "Databases",
  tags: ["mongodb", "nosql", "database"],
  status: "published",
  publishedAt: ISODate("2024-01-15"),
  updatedAt: ISODate("2024-01-16"),
  seo: {
    metaDescription: "...",
    keywords: ["mongodb", "nosql"]
  },
  stats: {
    views: 1234,
    likes: 56,
    shares: 12
  },
  comments: [
    {
      userId: ObjectId("user2"),
      name: "Bob",
      text: "Great article!",
      createdAt: ISODate("2024-01-15T12:00:00Z")
    }
  ]
}
```

---

## 24. Troubleshooting Guide

### 24.1 Common Issues and Solutions
```javascript
// Issue: Slow queries
// Solution 1: Add indexes
db.collection.createIndex({ field: 1 })

// Solution 2: Use explain() to analyze
db.collection.find({ field: value }).explain("executionStats")

// Solution 3: Limit results
db.collection.find().limit(100)

// Issue: High memory usage
// Solution 1: Reduce cache size
mongod --wiredTigerCacheSizeGB 2

// Solution 2: Add indexes to reduce collection scans
// Solution 3: Use projection to reduce data transfer

// Issue: Connection timeouts
// Solution 1: Increase pool size
const client = new MongoClient(uri, { maxPoolSize: 100 })

// Solution 2: Check network connectivity
// Solution 3: Increase timeout settings

// Issue: Write conflicts in transactions
// Solution: Implement retry logic
while (retries < maxRetries) {
  try {
    await session.withTransaction(async () => {
      // Transaction operations
    });
    break;
  } catch (error) {
    if (error.hasErrorLabel('TransientTransactionError')) {
      retries++;
      continue;
    }
    throw error;
  }
}
```

### 24.2 Performance Tuning Checklist
```javascript
// 1. Schema design
// - Embed or reference based on access patterns
// - Avoid unbounded arrays
// - Keep documents under 16MB

// 2. Indexing
// - Create indexes for frequent queries
// - Remove unused indexes
// - Use covered queries

// 3. Query optimization
// - Use projection
// - Limit results
// - Filter early in aggregations

// 4. Hardware
// - Use SSDs
// - Adequate RAM (more = better)
// - Fast network

// 5. Configuration
// - Appropriate cache size
// - Connection pool settings
// - Write concern balance
```

---

## Conclusion

This comprehensive MongoDB tutorial covers:

**Beginner Level:**
- MongoDB fundamentals and installation
- Basic CRUD operations
- Data types and schema design basics
- Query fundamentals

**Intermediate Level:**
- Advanced queries and operators
- Indexing strategies
- Aggregation framework
- Data modeling patterns
- Transactions
- Replication

**Advanced Level:**
- Sharding and horizontal scaling
- Advanced indexing
- Performance optimization
- Advanced aggregation
- Security and authentication
- Backup and recovery
- Change streams
- Architecture deep dive

### Next Steps:
1. Practice with real-world projects
2. Explore MongoDB Atlas (cloud offering)
3. Learn MongoDB Ops Manager for production deployments
4. Study MongoDB's official documentation
5. Join MongoDB community forums
6. Get MongoDB certification

### Resources:
- Official Documentation: https://docs.mongodb.com
- MongoDB University: https://university.mongodb.com
- Community Forums: https://www.mongodb.com/community/forums
- Stack Overflow: Tag [mongodb]

---

**Happy Learning! 🚀**