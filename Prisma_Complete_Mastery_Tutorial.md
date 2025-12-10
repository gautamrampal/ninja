# Prisma Complete Mastery Tutorial - From Zero to Ninja

> A comprehensive guide transforming complete beginners into Prisma experts through structured, linear progression with extensive code examples covering Node.js integration with MySQL, PostgreSQL, MongoDB, CRUD operations, and REST API development.

---

## Table of Contents

- [Phase 1: Prisma Fundamentals](#phase-1-prisma-fundamentals)
- [Phase 2: Database Setup (MySQL, PostgreSQL, MongoDB)](#phase-2-database-setup-mysql-postgresql-mongodb)
- [Phase 3: Prisma Schema Deep Dive](#phase-3-prisma-schema-deep-dive)
- [Phase 4: CRUD Operations Mastery](#phase-4-crud-operations-mastery)
- [Phase 5: Advanced Queries & Relationships](#phase-5-advanced-queries--relationships)
- [Phase 6: REST API Development](#phase-6-rest-api-development)
- [Phase 7: Migrations & Schema Management](#phase-7-migrations--schema-management)
- [Phase 8: Performance & Optimization](#phase-8-performance--optimization)
- [Phase 9: Production & Best Practices](#phase-9-production--best-practices)

---

## Phase 1: Prisma Fundamentals

### Module 1.1: Understanding Prisma

#### What is Prisma?

Prisma is a next-generation ORM (Object-Relational Mapping) tool for Node.js and TypeScript. It provides a type-safe database client that makes database access easy and error-free.

**Why Use Prisma?**

```javascript
// Traditional approach (without Prisma)
const mysql = require('mysql2/promise');
const connection = await mysql.createConnection({
  host: 'localhost',
  user: 'root',
  database: 'mydb'
});

// Raw SQL query - prone to errors, no type safety
const [rows] = await connection.execute(
  'SELECT * FROM users WHERE age > ? AND status = ?',
  [18, 'active']
);
// Result: rows is any[] - no type information

// Prisma approach - type-safe, auto-completion, error prevention
const users = await prisma.user.findMany({
  where: {
    age: { gt: 18 },
    status: 'active'
  }
});
// Result: users is User[] - fully typed with IntelliSense
```

**Key Benefits:**
1. **Type Safety** - Catch errors at compile time, not runtime
2. **Auto-completion** - IntelliSense for all database operations
3. **Database Agnostic** - Switch databases without rewriting queries
4. **Migration System** - Version control for your database schema
5. **Query Optimization** - Automatically optimizes queries

#### Prisma Architecture

Prisma consists of three main components:

```
┌─────────────────────────────────────────┐
│         Your Application                │
│    (Node.js/TypeScript/JavaScript)      │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│        Prisma Client                    │
│  (Auto-generated, Type-safe)            │
│  - findMany(), create(), update()...    │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│        Prisma Engine                    │
│  (Query Engine - Rust based)            │
│  - Translates to SQL                    │
│  - Handles connections                  │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│          Database                       │
│  MySQL | PostgreSQL | MongoDB           │
│  SQLite | SQL Server | CockroachDB      │
└─────────────────────────────────────────┘
```

**Prisma Schema (schema.prisma):**
- Single source of truth for your database structure
- Defines models, relations, and database connection
- Written in Prisma Schema Language (PSL)

### Module 1.2: Environment Setup

#### Prerequisites

```bash
# Check Node.js version (requires Node.js 14.17.0 or higher)
node --version

# Check npm version
npm --version

# If not installed, download from nodejs.org
```

#### Creating a New Project

```bash
# Create a new directory for your project
mkdir prisma-mastery
cd prisma-mastery

# Initialize a new Node.js project
npm init -y

# Install TypeScript and related dependencies (recommended)
npm install typescript ts-node @types/node --save-dev

# Initialize TypeScript configuration
npx tsc --init

# Install Prisma CLI as a development dependency
npm install prisma --save-dev

# Install Prisma Client (for database operations)
npm install @prisma/client

# Initialize Prisma in your project
npx prisma init
```

**What happens after `npx prisma init`?**

```
prisma-mastery/
├── node_modules/
├── prisma/
│   └── schema.prisma      # Your database schema file (created)
├── .env                   # Environment variables (created)
├── .gitignore            # Updated to exclude .env
├── package.json
└── tsconfig.json
```

**Understanding `.env` file:**

```bash
# .env file - stores sensitive configuration
# This file should NEVER be committed to version control

# Database connection URL
# Format: DATABASE_URL="provider://USER:PASSWORD@HOST:PORT/DATABASE"
DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
```

**Understanding `schema.prisma`:**

```prisma
// prisma/schema.prisma

// Generator: Defines what Prisma Client generates
generator client {
  provider = "prisma-client-js"  // Generates JavaScript/TypeScript client
}

// Datasource: Defines database connection
datasource db {
  provider = "postgresql"  // Database type (postgresql, mysql, mongodb, sqlite, etc.)
  url      = env("DATABASE_URL")  // Reads from .env file
}

// Models will be defined here (we'll add these later)
```

#### Project Structure Best Practices

```
prisma-mastery/
├── prisma/
│   ├── schema.prisma          # Database schema
│   ├── migrations/            # Migration history (created later)
│   └── seed.ts               # Seed data script (optional)
├── src/
│   ├── index.ts              # Main application entry
│   ├── prisma.ts             # Prisma client instance
│   ├── models/               # Business logic/models
│   ├── routes/               # API routes
│   └── utils/                # Utility functions
├── .env                       # Environment variables
├── .gitignore
├── package.json
└── tsconfig.json
```

### Module 1.3: First Prisma Model

#### Creating Your First Model

A model represents a table in your database. Let's create a simple User model.

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// User model - represents a "users" table
model User {
  id        Int      @id @default(autoincrement())  // Primary key, auto-increment
  email     String   @unique                         // Unique constraint
  name      String?                                  // Optional field (nullable)
  age       Int      @default(0)                     // Default value
  isActive  Boolean  @default(true)                  // Boolean with default
  createdAt DateTime @default(now())                 // Timestamp, defaults to now
  updatedAt DateTime @updatedAt                      // Auto-updates on change
}
```

**Understanding Field Attributes:**

```prisma
model User {
  // @id - Marks field as primary key
  id Int @id @default(autoincrement())
  
  // @unique - Ensures values are unique across all records
  email String @unique
  
  // ? - Makes field optional (nullable)
  name String?
  
  // @default() - Sets default value when not provided
  age Int @default(0)
  
  // @updatedAt - Automatically updates to current time on record update
  updatedAt DateTime @updatedAt
  
  // @map() - Maps to different database column name
  firstName String @map("first_name")
  
  // @@map() - Maps to different database table name
  @@map("users")
}
```

#### Running Your First Migration

Migrations create/update your database schema based on your Prisma schema.

```bash
# Create and apply migration (for PostgreSQL, MySQL)
npx prisma migrate dev --name init

# This command does three things:
# 1. Creates SQL migration file in prisma/migrations/
# 2. Applies migration to database
# 3. Generates Prisma Client
```

**What happens:**

```
prisma/migrations/
└── 20231210120000_init/
    └── migration.sql
```

**Generated SQL (migration.sql):**

```sql
-- CreateTable
CREATE TABLE "User" (
    "id" SERIAL NOT NULL,
    "email" TEXT NOT NULL,
    "name" TEXT,
    "age" INTEGER NOT NULL DEFAULT 0,
    "isActive" BOOLEAN NOT NULL DEFAULT true,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "User_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "User_email_key" ON "User"("email");
```

#### Using Prisma Client

Create a Prisma Client instance to interact with your database.

```typescript
// src/prisma.ts - Prisma Client singleton

import { PrismaClient } from '@prisma/client';

// Create a single instance of Prisma Client
// This prevents creating multiple connections
const prisma = new PrismaClient({
  log: ['query', 'info', 'warn', 'error'],  // Enable logging
});

export default prisma;
```

**Your First CRUD Operation:**

```typescript
// src/index.ts - First database operation

import prisma from './prisma';

async function main() {
  // CREATE - Insert a new user
  const newUser = await prisma.user.create({
    data: {
      email: 'alice@example.com',
      name: 'Alice Johnson',
      age: 28,
    },
  });
  
  console.log('Created user:', newUser);
  // Output: Created user: { id: 1, email: 'alice@example.com', name: 'Alice Johnson', age: 28, isActive: true, createdAt: 2023-12-10T..., updatedAt: 2023-12-10T... }
  
  // READ - Fetch all users
  const allUsers = await prisma.user.findMany();
  console.log('All users:', allUsers);
  
  // READ - Fetch single user by unique field
  const user = await prisma.user.findUnique({
    where: { email: 'alice@example.com' },
  });
  console.log('Found user:', user);
  
  // UPDATE - Modify user
  const updatedUser = await prisma.user.update({
    where: { email: 'alice@example.com' },
    data: { age: 29 },
  });
  console.log('Updated user:', updatedUser);
  
  // DELETE - Remove user
  const deletedUser = await prisma.user.delete({
    where: { email: 'alice@example.com' },
  });
  console.log('Deleted user:', deletedUser);
}

// Execute main function
main()
  .catch((e) => {
    console.error('Error:', e);
    process.exit(1);
  })
  .finally(async () => {
    // Disconnect Prisma Client
    await prisma.$disconnect();
  });
```

**Running Your Code:**

```bash
# Compile and run TypeScript
npx ts-node src/index.ts

# Or add script to package.json
# "scripts": {
#   "dev": "ts-node src/index.ts"
# }
npm run dev
```

### Module 1.4: Prisma Studio

Prisma Studio is a visual database browser that comes with Prisma.

```bash
# Launch Prisma Studio
npx prisma studio

# Opens in browser at http://localhost:5555
```

**Prisma Studio Features:**
- View all tables and records
- Add, edit, delete records visually
- Filter and search data
- Great for development and debugging

---

## Phase 2: Database Setup (MySQL, PostgreSQL, MongoDB)

### Module 2.1: PostgreSQL Setup

#### Installing PostgreSQL

**Windows:**
```bash
# Download installer from postgresql.org
# Or use Chocolatey
choco install postgresql

# Start PostgreSQL service
net start postgresql-x64-14
```

**macOS:**
```bash
# Using Homebrew
brew install postgresql@14
brew services start postgresql@14
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

#### Creating PostgreSQL Database

```bash
# Access PostgreSQL shell
psql -U postgres

# Inside psql shell:
# Create database
CREATE DATABASE prisma_tutorial;

# Create user with password
CREATE USER prisma_user WITH PASSWORD 'secure_password123';

# Grant privileges
GRANT ALL PRIVILEGES ON DATABASE prisma_tutorial TO prisma_user;

# Exit psql
\q
```

#### Configuring Prisma for PostgreSQL

```bash
# .env file
DATABASE_URL="postgresql://prisma_user:secure_password123@localhost:5432/prisma_tutorial?schema=public"
```

```prisma
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  age       Int
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**Run Migration:**

```bash
npx prisma migrate dev --name init_postgresql
npx prisma generate
```

### Module 2.2: MySQL Setup

#### Installing MySQL

**Windows:**
```bash
# Download installer from mysql.com
# Or use Chocolatey
choco install mysql

# Start MySQL service
net start MySQL80
```

**macOS:**
```bash
# Using Homebrew
brew install mysql
brew services start mysql
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install mysql-server
sudo systemctl start mysql
sudo systemctl enable mysql

# Secure installation
sudo mysql_secure_installation
```

#### Creating MySQL Database

```bash
# Access MySQL shell
mysql -u root -p

# Inside MySQL shell:
# Create database
CREATE DATABASE prisma_tutorial;

# Create user with password
CREATE USER 'prisma_user'@'localhost' IDENTIFIED BY 'secure_password123';

# Grant privileges
GRANT ALL PRIVILEGES ON prisma_tutorial.* TO 'prisma_user'@'localhost';
FLUSH PRIVILEGES;

# Exit MySQL
exit;
```

#### Configuring Prisma for MySQL

```bash
# .env file
DATABASE_URL="mysql://prisma_user:secure_password123@localhost:3306/prisma_tutorial"
```

```prisma
// prisma/schema.prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  age       Int
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**Run Migration:**

```bash
npx prisma migrate dev --name init_mysql
npx prisma generate
```

**MySQL-specific Data Types:**

```prisma
model Product {
  id          Int      @id @default(autoincrement())
  name        String   @db.VarChar(255)           // Variable character
  description String   @db.Text                   // Long text
  price       Decimal  @db.Decimal(10, 2)         // 10 digits, 2 decimal places
  stock       Int      @db.UnsignedInt            // Unsigned integer
  weight      Float    @db.Float                  // Float
  isActive    Boolean  @db.TinyInt                // Boolean as TinyInt
  metadata    Json     @db.Json                   // JSON field
  createdAt   DateTime @default(now()) @db.Timestamp(0)
}
```

### Module 2.3: MongoDB Setup

MongoDB setup with Prisma is different because MongoDB is a NoSQL database.

#### Installing MongoDB

**Windows:**
```bash
# Download installer from mongodb.com
# Or use Chocolatey
choco install mongodb

# Start MongoDB service
net start MongoDB
```

**macOS:**
```bash
# Using Homebrew
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

**Linux (Ubuntu/Debian):**
```bash
# Import MongoDB public key
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

# Create list file
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

# Install MongoDB
sudo apt-get update
sudo apt-get install -y mongodb-org

# Start MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod
```

#### Configuring Prisma for MongoDB

```bash
# .env file
# Local MongoDB
DATABASE_URL="mongodb://localhost:27017/prisma_tutorial"

# MongoDB Atlas (cloud)
DATABASE_URL="mongodb+srv://username:password@cluster0.mongodb.net/prisma_tutorial?retryWrites=true&w=majority"
```

```prisma
// prisma/schema.prisma
datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

// MongoDB uses String @id @default(auto()) @map("_id") @db.ObjectId
model User {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  email     String   @unique
  name      String?
  age       Int
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**Key Differences with MongoDB:**

1. **No Migrations** - MongoDB is schema-less, so `prisma migrate` doesn't work
2. **Use `db push`** instead:

```bash
# Push schema to MongoDB (no migration files)
npx prisma db push

# Generate Prisma Client
npx prisma generate
```

3. **ObjectId for IDs:**

```prisma
model Post {
  id       String @id @default(auto()) @map("_id") @db.ObjectId
  title    String
  authorId String @db.ObjectId
  
  // Relation to User
  author   User   @relation(fields: [authorId], references: [id])
}

model User {
  id    String @id @default(auto()) @map("_id") @db.ObjectId
  name  String
  posts Post[]
}
```

**MongoDB-specific Types:**

```prisma
model Product {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  name        String
  description String
  price       Float
  tags        String[]                      // Array of strings
  metadata    Json                          // Embedded JSON
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

### Module 2.4: Multi-Database Project Structure

For projects using multiple databases:

```
prisma-mastery/
├── prisma/
│   ├── schema.prisma         # Main schema (e.g., PostgreSQL)
│   ├── schema-mysql.prisma   # MySQL schema
│   └── schema-mongo.prisma   # MongoDB schema
├── src/
│   ├── prisma-pg.ts          # PostgreSQL client
│   ├── prisma-mysql.ts       # MySQL client
│   └── prisma-mongo.ts       # MongoDB client
└── .env
```

**Configure Multiple Schemas:**

```bash
# .env
POSTGRES_URL="postgresql://user:pass@localhost:5432/db"
MYSQL_URL="mysql://user:pass@localhost:3306/db"
MONGO_URL="mongodb://localhost:27017/db"
```

```prisma
// prisma/schema-mysql.prisma
datasource db {
  provider = "mysql"
  url      = env("MYSQL_URL")
}

generator client {
  provider = "prisma-client-js"
  output   = "../node_modules/.prisma/client-mysql"  // Custom output
}

model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
}
```

**Generate Clients:**

```bash
# Generate PostgreSQL client
npx prisma generate --schema=./prisma/schema.prisma

# Generate MySQL client
npx prisma generate --schema=./prisma/schema-mysql.prisma

# Generate MongoDB client
npx prisma generate --schema=./prisma/schema-mongo.prisma
```

**Using Multiple Clients:**

```typescript
// src/prisma-mysql.ts
import { PrismaClient as MySQLClient } from '.prisma/client-mysql';
export const mysqlClient = new MySQLClient();

// src/prisma-pg.ts
import { PrismaClient as PGClient } from '@prisma/client';
export const pgClient = new PGClient();

// src/index.ts
import { mysqlClient } from './prisma-mysql';
import { pgClient } from './prisma-pg';

async function main() {
  // Use MySQL
  const mysqlUser = await mysqlClient.user.create({
    data: { email: 'mysql@example.com', name: 'MySQL User' }
  });
  
  // Use PostgreSQL
  const pgUser = await pgClient.user.create({
    data: { email: 'pg@example.com', name: 'PG User' }
  });
}
```

---

## Phase 3: Prisma Schema Deep Dive

### Module 3.1: Data Types & Field Attributes

#### Scalar Types

Prisma supports various scalar types that map to database-specific types.

```prisma
model DataTypesExample {
  id          Int      @id @default(autoincrement())
  
  // String types
  name        String                          // Variable length string
  description String   @db.Text              // Long text (MySQL/PostgreSQL)
  code        String   @db.VarChar(10)       // Fixed max length
  
  // Number types
  age         Int                             // 32-bit integer
  bigNumber   BigInt                          // 64-bit integer
  price       Float                           // Floating point
  exactPrice  Decimal                         // Precise decimal
  
  // Boolean
  isActive    Boolean  @default(true)
  
  // DateTime
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  scheduledAt DateTime?                       // Optional datetime
  
  // JSON
  metadata    Json                            // JSON data
  
  // Bytes (binary data)
  avatar      Bytes?                          // Binary data
  
  // Enum (defined separately)
  role        Role     @default(USER)
  
  // Arrays (PostgreSQL, MongoDB)
  tags        String[]                        // Array of strings
}

// Enum definition
enum Role {
  USER
  ADMIN
  MODERATOR
}
```

#### Field Attributes

```prisma
model User {
  // @id - Primary key
  id Int @id @default(autoincrement())
  
  // @unique - Unique constraint
  email     String @unique
  username  String @unique
  
  // @default() - Default value
  status    String @default("active")
  createdAt DateTime @default(now())
  
  // @updatedAt - Auto-update timestamp
  updatedAt DateTime @updatedAt
  
  // @map() - Custom database column name
  firstName String @map("first_name")
  lastName  String @map("last_name")
  
  // @db - Database-specific type
  bio       String @db.Text
  balance   Decimal @db.Decimal(10, 2)
  
  // @ignore - Exclude from Prisma Client
  internal  String @ignore
  
  // @relation - Define relationship
  posts     Post[]
  profile   Profile?
  
  // @@map() - Custom table name (model-level)
  @@map("users")
  
  // @@unique() - Composite unique constraint
  @@unique([firstName, lastName])
  
  // @@index() - Database index
  @@index([email, status])
  
  // @@id() - Composite primary key
  // @@id([field1, field2])
}
```

### Module 3.2: Relationships

#### One-to-One Relationship

A User has one Profile, and a Profile belongs to one User.

```prisma
model User {
  id      Int      @id @default(autoincrement())
  email   String   @unique
  name    String
  profile Profile?  // One-to-one (optional)
}

model Profile {
  id     Int    @id @default(autoincrement())
  bio    String
  avatar String?
  
  // Foreign key field
  userId Int    @unique
  
  // Relation field
  user   User   @relation(fields: [userId], references: [id])
}
```

**Usage:**

```typescript
// Create user with profile
const userWithProfile = await prisma.user.create({
  data: {
    email: 'john@example.com',
    name: 'John Doe',
    profile: {
      create: {
        bio: 'Software developer',
        avatar: 'avatar.jpg'
      }
    }
  },
  include: {
    profile: true  // Include profile in response
  }
});

// Update profile
const updatedUser = await prisma.user.update({
  where: { id: 1 },
  data: {
    profile: {
      update: {
        bio: 'Senior software developer'
      }
    }
  }
});
```

#### One-to-Many Relationship

A User can have many Posts, but a Post belongs to one User.

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
  posts Post[]  // One-to-many
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  
  // Foreign key field
  authorId  Int
  
  // Relation field
  author    User     @relation(fields: [authorId], references: [id])
}
```

**Usage:**

```typescript
// Create user with multiple posts
const userWithPosts = await prisma.user.create({
  data: {
    email: 'author@example.com',
    name: 'Jane Author',
    posts: {
      create: [
        { title: 'First Post', content: 'Hello World' },
        { title: 'Second Post', content: 'Learning Prisma' }
      ]
    }
  },
  include: {
    posts: true
  }
});

// Find user with all posts
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: {
      where: { published: true },  // Filter posts
      orderBy: { createdAt: 'desc' }
    }
  }
});

// Create post for existing user
const post = await prisma.post.create({
  data: {
    title: 'Third Post',
    content: 'Advanced Prisma',
    author: {
      connect: { id: 1 }  // Connect to existing user
    }
  }
});
```

#### Many-to-Many Relationship

Users can have many Roles, and Roles can belong to many Users.

**Implicit Many-to-Many (Prisma manages join table):**

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
  roles Role[]  // Many-to-many
}

model Role {
  id    Int    @id @default(autoincrement())
  name  String @unique
  users User[]  // Many-to-many
}

// Prisma automatically creates "_RoleToUser" join table
```

**Usage:**

```typescript
// Create user with roles
const user = await prisma.user.create({
  data: {
    email: 'admin@example.com',
    name: 'Admin User',
    roles: {
      create: [
        { name: 'ADMIN' },
        { name: 'MODERATOR' }
      ]
    }
  },
  include: {
    roles: true
  }
});

// Add role to existing user
const updatedUser = await prisma.user.update({
  where: { id: 1 },
  data: {
    roles: {
      connect: { id: 2 }  // Connect existing role
    }
  }
});

// Remove role from user
await prisma.user.update({
  where: { id: 1 },
  data: {
    roles: {
      disconnect: { id: 2 }
    }
  }
});
```

**Explicit Many-to-Many (Custom join table with extra fields):**

```prisma
model User {
  id              Int              @id @default(autoincrement())
  email           String           @unique
  name            String
  userRoles       UserRole[]       // Join table relation
}

model Role {
  id              Int              @id @default(autoincrement())
  name            String           @unique
  userRoles       UserRole[]       // Join table relation
}

model UserRole {
  id          Int      @id @default(autoincrement())
  assignedAt  DateTime @default(now())
  assignedBy  String
  
  userId      Int
  roleId      Int
  
  user        User     @relation(fields: [userId], references: [id])
  role        Role     @relation(fields: [roleId], references: [id])
  
  @@unique([userId, roleId])  // Prevent duplicate assignments
}
```

**Usage:**

```typescript
// Assign role to user with metadata
const assignment = await prisma.userRole.create({
  data: {
    assignedBy: 'admin@example.com',
    user: { connect: { id: 1 } },
    role: { connect: { id: 2 } }
  }
});

// Get user with roles and metadata
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    userRoles: {
      include: {
        role: true  // Include role details
      }
    }
  }
});
```

#### Self-Relation

A User can follow other Users.

```prisma
model User {
  id         Int    @id @default(autoincrement())
  email      String @unique
  name       String
  
  // Users this user follows
  following  User[] @relation("UserFollows")
  
  // Users following this user
  followers  User[] @relation("UserFollows")
}
```

**Usage:**

```typescript
// User 1 follows User 2
await prisma.user.update({
  where: { id: 1 },
  data: {
    following: {
      connect: { id: 2 }
    }
  }
});

// Get user with followers and following
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    followers: true,
    following: true
  }
});
```

### Module 3.3: Advanced Schema Features

#### Composite Types (MongoDB)

```prisma
// Available for MongoDB only
model User {
  id      String  @id @default(auto()) @map("_id") @db.ObjectId
  email   String  @unique
  name    String
  address Address // Embedded document
}

type Address {
  street  String
  city    String
  state   String
  zipCode String
  country String
}
```

**Usage:**

```typescript
const user = await prisma.user.create({
  data: {
    email: 'john@example.com',
    name: 'John Doe',
    address: {
      street: '123 Main St',
      city: 'New York',
      state: 'NY',
      zipCode: '10001',
      country: 'USA'
    }
  }
});
```

#### Enums

```prisma
enum UserRole {
  USER
  ADMIN
  MODERATOR
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
}

model User {
  id    Int      @id @default(autoincrement())
  email String   @unique
  role  UserRole @default(USER)
}

model Order {
  id     Int         @id @default(autoincrement())
  status OrderStatus @default(PENDING)
}
```

**Usage:**

```typescript
import { UserRole, OrderStatus } from '@prisma/client';

// Create user with role
const user = await prisma.user.create({
  data: {
    email: 'admin@example.com',
    role: UserRole.ADMIN
  }
});

// Filter by enum
const admins = await prisma.user.findMany({
  where: {
    role: UserRole.ADMIN
  }
});

// Update order status
const order = await prisma.order.update({
  where: { id: 1 },
  data: {
    status: OrderStatus.SHIPPED
  }
});
```

#### Cascading Deletes & Referential Actions

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  authorId Int
  
  // Cascading delete: when user is deleted, delete all their posts
  author   User   @relation(fields: [authorId], references: [id], onDelete: Cascade)
}

model Comment {
  id     Int  @id @default(autoincrement())
  text   String
  postId Int
  
  // Set null: when post is deleted, set postId to null
  post   Post @relation(fields: [postId], references: [id], onDelete: SetNull)
}

model Like {
  id     Int  @id @default(autoincrement())
  userId Int
  postId Int
  
  // Restrict: prevent deletion if related records exist
  user   User @relation(fields: [userId], references: [id], onDelete: Restrict)
  post   Post @relation(fields: [postId], references: [id], onDelete: Restrict)
}
```

**Referential Actions:**
- `Cascade` - Delete related records
- `SetNull` - Set foreign key to null
- `Restrict` - Prevent deletion
- `NoAction` - Database default behavior
- `SetDefault` - Set to default value

#### Indexes for Performance

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  firstName String
  lastName  String
  age       Int
  city      String
  createdAt DateTime @default(now())
  
  // Single field index
  @@index([email])
  
  // Composite index (multiple fields)
  @@index([firstName, lastName])
  
  // Index with sorting
  @@index([createdAt(sort: Desc)])
  
  // Unique composite constraint
  @@unique([firstName, lastName, age])
  
  // Full-text index (MySQL)
  @@index([firstName, lastName], type: Fulltext)
}
```

#### Views (PostgreSQL, MySQL)

```prisma
// Define a database view
view ActiveUsers {
  id        Int      @unique
  email     String
  name      String
  postCount Int
}
```

**Create view in migration:**

```sql
-- In migration file
CREATE VIEW "ActiveUsers" AS
SELECT u.id, u.email, u.name, COUNT(p.id) as "postCount"
FROM "User" u
LEFT JOIN "Post" p ON p."authorId" = u.id
WHERE u."isActive" = true
GROUP BY u.id;
```

---

## Phase 4: CRUD Operations Mastery

### Module 4.1: Create Operations

#### Basic Create

```typescript
// Create single record
const user = await prisma.user.create({
  data: {
    email: 'john@example.com',
    name: 'John Doe',
    age: 30
  }
});

console.log('Created user:', user);
// Output: { id: 1, email: 'john@example.com', name: 'John Doe', age: 30, ... }
```

#### Create with Relations

```typescript
// Create user with nested profile
const userWithProfile = await prisma.user.create({
  data: {
    email: 'jane@example.com',
    name: 'Jane Smith',
    profile: {
      create: {
        bio: 'Developer & Designer',
        avatar: 'jane-avatar.jpg'
      }
    }
  },
  include: {
    profile: true  // Include created profile in response
  }
});

// Create post for existing user
const post = await prisma.post.create({
  data: {
    title: 'Getting Started with Prisma',
    content: 'Prisma is an amazing ORM...',
    author: {
      connect: { id: 1 }  // Connect to user with id: 1
    }
  }
});

// Create user with multiple posts
const authorWithPosts = await prisma.user.create({
  data: {
    email: 'author@example.com',
    name: 'Author Name',
    posts: {
      create: [
        { title: 'Post 1', content: 'Content 1' },
        { title: 'Post 2', content: 'Content 2' },
        { title: 'Post 3', content: 'Content 3' }
      ]
    }
  },
  include: {
    posts: true
  }
});
```

#### Create Many

```typescript
// Create multiple records at once
const users = await prisma.user.createMany({
  data: [
    { email: 'user1@example.com', name: 'User 1', age: 25 },
    { email: 'user2@example.com', name: 'User 2', age: 30 },
    { email: 'user3@example.com', name: 'User 3', age: 35 }
  ],
  skipDuplicates: true  // Skip records that would cause unique constraint violations
});

console.log(`Created ${users.count} users`);
// Output: Created 3 users
```

#### Create or Connect

```typescript
// Create if doesn't exist, or connect if exists
const post = await prisma.post.create({
  data: {
    title: 'My Post',
    content: 'Post content',
    author: {
      connectOrCreate: {
        where: { email: 'author@example.com' },
        create: {
          email: 'author@example.com',
          name: 'New Author'
        }
      }
    }
  }
});
```

### Module 4.2: Read Operations

#### Find Unique

```typescript
// Find by unique field
const user = await prisma.user.findUnique({
  where: { email: 'john@example.com' }
});

// Find by primary key
const userById = await prisma.user.findUnique({
  where: { id: 1 }
});

// Find by composite unique constraint
const assignment = await prisma.userRole.findUnique({
  where: {
    userId_roleId: {
      userId: 1,
      roleId: 2
    }
  }
});
```

#### Find First

```typescript
// Find first matching record
const firstActiveUser = await prisma.user.findFirst({
  where: { isActive: true },
  orderBy: { createdAt: 'desc' }
});

// Find first or throw error
const user = await prisma.user.findFirstOrThrow({
  where: { email: 'admin@example.com' }
});
```

#### Find Many

```typescript
// Find all records
const allUsers = await prisma.user.findMany();

// Find with filters
const activeUsers = await prisma.user.findMany({
  where: {
    isActive: true,
    age: { gte: 18 }  // Greater than or equal
  }
});

// Find with pagination
const users = await prisma.user.findMany({
  skip: 0,      // Offset
  take: 10,     // Limit
  orderBy: {
    createdAt: 'desc'
  }
});

// Find with multiple conditions
const users = await prisma.user.findMany({
  where: {
    AND: [
      { age: { gte: 18 } },
      { age: { lte: 65 } },
      { isActive: true }
    ]
  }
});

// OR conditions
const users = await prisma.user.findMany({
  where: {
    OR: [
      { email: { contains: 'gmail.com' } },
      { email: { contains: 'yahoo.com' } }
    ]
  }
});

// NOT conditions
const users = await prisma.user.findMany({
  where: {
    NOT: {
      email: { contains: 'spam.com' }
    }
  }
});
```

#### Filter Operators

```typescript
// String filters
const users = await prisma.user.findMany({
  where: {
    name: { equals: 'John' },              // Exact match
    email: { contains: 'example' },        // Contains substring
    name: { startsWith: 'J' },             // Starts with
    email: { endsWith: '.com' },           // Ends with
    name: { in: ['John', 'Jane', 'Bob'] }, // In list
    email: { not: { contains: 'spam' } }   // Not contains
  }
});

// Number filters
const products = await prisma.product.findMany({
  where: {
    price: { gt: 100 },      // Greater than
    stock: { gte: 10 },      // Greater than or equal
    discount: { lt: 50 },    // Less than
    rating: { lte: 5 },      // Less than or equal
    quantity: { not: 0 }     // Not equal
  }
});

// Date filters
const recentPosts = await prisma.post.findMany({
  where: {
    createdAt: {
      gte: new Date('2023-01-01'),
      lte: new Date('2023-12-31')
    }
  }
});

// Array filters (PostgreSQL, MongoDB)
const posts = await prisma.post.findMany({
  where: {
    tags: { has: 'javascript' },              // Has element
    tags: { hasEvery: ['node', 'prisma'] },   // Has all elements
    tags: { hasSome: ['react', 'vue'] },      // Has at least one
    tags: { isEmpty: false }                  // Not empty
  }
});

// JSON filters
const users = await prisma.user.findMany({
  where: {
    metadata: {
      path: ['preferences', 'theme'],
      equals: 'dark'
    }
  }
});
```

#### Select Specific Fields

```typescript
// Select only specific fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    name: true
    // password field not included
  }
});

// Include related data
const users = await prisma.user.findMany({
  include: {
    posts: true,
    profile: true
  }
});

// Nested select
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    posts: {
      select: {
        title: true,
        createdAt: true
      },
      where: { published: true }
    }
  }
});
```

#### Count Records

```typescript
// Count all records
const totalUsers = await prisma.user.count();

// Count with filter
const activeUserCount = await prisma.user.count({
  where: { isActive: true }
});

// Count with grouping
const postCountByAuthor = await prisma.post.groupBy({
  by: ['authorId'],
  _count: {
    id: true
  }
});
```

### Module 4.3: Update Operations

#### Update Single Record

```typescript
// Update by unique field
const user = await prisma.user.update({
  where: { email: 'john@example.com' },
  data: {
    name: 'John Updated',
    age: 31
  }
});

// Update with increment
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    views: { increment: 1 }  // views = views + 1
  }
});

// Update with multiply/divide
const product = await prisma.product.update({
  where: { id: 1 },
  data: {
    price: { multiply: 1.1 }  // Increase price by 10%
  }
});
```

#### Update Nested Relations

```typescript
// Update user and their profile
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    name: 'Updated Name',
    profile: {
      update: {
        bio: 'Updated bio'
      }
    }
  }
});

// Update or create nested relation
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    profile: {
      upsert: {
        create: { bio: 'New bio' },
        update: { bio: 'Updated bio' }
      }
    }
  }
});

// Disconnect relation
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    profile: {
      disconnect: true
    }
  }
});

// Delete nested relation
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    profile: {
      delete: true
    }
  }
});
```

#### Update Many

```typescript
// Update multiple records
const result = await prisma.user.updateMany({
  where: {
    age: { lt: 18 }
  },
  data: {
    isActive: false
  }
});

console.log(`Updated ${result.count} users`);

// Update all records
const result = await prisma.product.updateMany({
  data: {
    discount: 10
  }
});
```

#### Upsert (Update or Insert)

```typescript
// Update if exists, create if doesn't exist
const user = await prisma.user.upsert({
  where: { email: 'john@example.com' },
  update: {
    name: 'John Updated',
    age: 31
  },
  create: {
    email: 'john@example.com',
    name: 'John New',
    age: 30
  }
});
```

### Module 4.4: Delete Operations

#### Delete Single Record

```typescript
// Delete by unique field
const deletedUser = await prisma.user.delete({
  where: { email: 'john@example.com' }
});

// Delete by ID
const deletedUser = await prisma.user.delete({
  where: { id: 1 }
});
```

#### Delete Many

```typescript
// Delete multiple records
const result = await prisma.user.deleteMany({
  where: {
    isActive: false,
    createdAt: {
      lt: new Date('2020-01-01')
    }
  }
});

console.log(`Deleted ${result.count} users`);

// Delete all records (use with caution!)
const result = await prisma.user.deleteMany({});
```

#### Cascading Deletes

```typescript
// If onDelete: Cascade is set in schema
// Deleting user will automatically delete all related posts
const deletedUser = await prisma.user.delete({
  where: { id: 1 }
});
// All posts by this user are also deleted

// Manual cascading delete
await prisma.post.deleteMany({
  where: { authorId: 1 }
});
await prisma.user.delete({
  where: { id: 1 }
});
```

### Module 4.5: Advanced Query Patterns

#### Aggregations

```typescript
// Aggregate functions
const aggregations = await prisma.user.aggregate({
  _count: {
    id: true  // Count users
  },
  _avg: {
    age: true  // Average age
  },
  _sum: {
    age: true  // Sum of ages
  },
  _min: {
    age: true  // Minimum age
  },
  _max: {
    age: true  // Maximum age
  }
});

console.log(aggregations);
// Output: { _count: { id: 100 }, _avg: { age: 32.5 }, _sum: { age: 3250 }, _min: { age: 18 }, _max: { age: 75 } }
```

#### Group By

```typescript
// Group by single field
const usersByAge = await prisma.user.groupBy({
  by: ['age'],
  _count: {
    id: true
  },
  orderBy: {
    age: 'asc'
  }
});

// Group by multiple fields
const postStats = await prisma.post.groupBy({
  by: ['authorId', 'published'],
  _count: {
    id: true
  },
  _avg: {
    views: true
  },
  having: {
    views: {
      _avg: {
        gt: 100  // Only groups with average views > 100
      }
    }
  }
});
```

#### Distinct

```typescript
// Get distinct values
const distinctAges = await prisma.user.findMany({
  distinct: ['age'],
  select: {
    age: true
  }
});

// Distinct on multiple fields
const distinctUsers = await prisma.user.findMany({
  distinct: ['city', 'country']
});
```

#### Pagination

```typescript
// Offset-based pagination
async function getUsersPage(page: number, pageSize: number) {
  const skip = (page - 1) * pageSize;
  
  const [users, total] = await Promise.all([
    prisma.user.findMany({
      skip,
      take: pageSize,
      orderBy: { createdAt: 'desc' }
    }),
    prisma.user.count()
  ]);
  
  return {
    data: users,
    total,
    page,
    pageSize,
    totalPages: Math.ceil(total / pageSize)
  };
}

// Usage
const result = await getUsersPage(2, 10);  // Page 2, 10 items per page

// Cursor-based pagination (better for large datasets)
async function getUsersCursor(cursor: number | undefined, pageSize: number) {
  const users = await prisma.user.findMany({
    take: pageSize,
    skip: cursor ? 1 : 0,  // Skip the cursor
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { id: 'asc' }
  });
  
  return {
    data: users,
    nextCursor: users.length === pageSize ? users[users.length - 1].id : undefined
  };
}

// Usage
let cursor;
const page1 = await getUsersCursor(undefined, 10);
cursor = page1.nextCursor;
const page2 = await getUsersCursor(cursor, 10);
```

---

## Phase 5: Advanced Queries & Relationships

### Module 5.1: Complex Filtering

#### Nested Filters

```typescript
// Find users who have published posts
const users = await prisma.user.findMany({
  where: {
    posts: {
      some: {
        published: true
      }
    }
  }
});

// Find users who have ALL posts published
const users = await prisma.user.findMany({
  where: {
    posts: {
      every: {
        published: true
      }
    }
  }
});

// Find users who have NO published posts
const users = await prisma.user.findMany({
  where: {
    posts: {
      none: {
        published: true
      }
    }
  }
});

// Complex nested conditions
const users = await prisma.user.findMany({
  where: {
    posts: {
      some: {
        AND: [
          { published: true },
          { views: { gt: 1000 } },
          {
            comments: {
              some: {
                text: { contains: 'great' }
              }
            }
          }
        ]
      }
    }
  }
});
```

#### Relation Count Filtering

```typescript
// Find users with more than 5 posts
const activeAuthors = await prisma.user.findMany({
  where: {
    posts: {
      some: {}
    }
  },
  include: {
    _count: {
      select: {
        posts: true
      }
    }
  }
});

// Filter results in application code
const prolificAuthors = activeAuthors.filter(user => user._count.posts > 5);
```

### Module 5.2: Transactions

#### Sequential Transactions

```typescript
// All operations succeed or all fail
const transfer = await prisma.$transaction(async (tx) => {
  // Deduct from sender
  const sender = await tx.account.update({
    where: { id: 1 },
    data: {
      balance: { decrement: 100 }
    }
  });
  
  // Check if sender has enough balance
  if (sender.balance < 0) {
    throw new Error('Insufficient funds');
  }
  
  // Add to receiver
  const receiver = await tx.account.update({
    where: { id: 2 },
    data: {
      balance: { increment: 100 }
    }
  });
  
  // Create transaction record
  const transaction = await tx.transaction.create({
    data: {
      from: 1,
      to: 2,
      amount: 100
    }
  });
  
  return { sender, receiver, transaction };
});
```

#### Batch Transactions

```typescript
// Execute multiple operations as a single transaction
const [deletedPosts, deletedComments, deletedUser] = await prisma.$transaction([
  prisma.post.deleteMany({ where: { authorId: 1 } }),
  prisma.comment.deleteMany({ where: { userId: 1 } }),
  prisma.user.delete({ where: { id: 1 } })
]);
```

#### Transaction Isolation Levels (PostgreSQL)

```typescript
const result = await prisma.$transaction(
  async (tx) => {
    // Your transaction operations
    const user = await tx.user.findUnique({ where: { id: 1 } });
    await tx.user.update({
      where: { id: 1 },
      data: { balance: user.balance + 100 }
    });
  },
  {
    isolationLevel: 'Serializable', // Highest isolation
    maxWait: 5000,    // Maximum wait time in ms
    timeout: 10000,   // Maximum execution time in ms
  }
);
```

### Module 5.3: Raw Queries

#### Raw SQL Queries

```typescript
// Execute raw SQL query
const users = await prisma.$queryRaw`
  SELECT * FROM "User" WHERE age > ${18}
`;

// Raw query with multiple parameters
const email = 'john@example.com';
const minAge = 18;
const users = await prisma.$queryRaw`
  SELECT * FROM "User" 
  WHERE email = ${email} AND age > ${minAge}
`;

// Unsafe raw query (use only with trusted input)
const tableName = 'User';
const users = await prisma.$queryRawUnsafe(
  `SELECT * FROM "${tableName}" WHERE age > $1`,
  18
);
```

#### Execute Raw SQL

```typescript
// Execute non-query SQL (INSERT, UPDATE, DELETE)
const result = await prisma.$executeRaw`
  UPDATE "User" SET "isActive" = false WHERE age < ${18}
`;
console.log(`Updated ${result} rows`);

// Execute multiple statements
await prisma.$executeRaw`
  CREATE INDEX idx_user_email ON "User"(email);
`;
```

### Module 5.4: Middleware

#### Query Middleware

```typescript
// Add timestamp logging
prisma.$use(async (params, next) => {
  const before = Date.now();
  const result = await next(params);
  const after = Date.now();
  
  console.log(`Query ${params.model}.${params.action} took ${after - before}ms`);
  return result;
});

// Soft delete middleware
prisma.$use(async (params, next) => {
  if (params.model === 'User') {
    if (params.action === 'delete') {
      // Change delete to update
      params.action = 'update';
      params.args['data'] = { deletedAt: new Date() };
    }
    if (params.action === 'deleteMany') {
      params.action = 'updateMany';
      if (params.args.data !== undefined) {
        params.args.data['deletedAt'] = new Date();
      } else {
        params.args['data'] = { deletedAt: new Date() };
      }
    }
  }
  return next(params);
});

// Auto-filter deleted records
prisma.$use(async (params, next) => {
  if (params.model === 'User') {
    if (params.action === 'findUnique' || params.action === 'findFirst') {
      params.action = 'findFirst';
      params.args.where['deletedAt'] = null;
    }
    if (params.action === 'findMany') {
      if (params.args.where) {
        if (params.args.where.deletedAt === undefined) {
          params.args.where['deletedAt'] = null;
        }
      } else {
        params.args['where'] = { deletedAt: null };
      }
    }
  }
  return next(params);
});
```

### Module 5.5: Query Optimization

#### Select Only Needed Fields

```typescript
// Bad: Fetches all fields including large text fields
const posts = await prisma.post.findMany();

// Good: Select only needed fields
const posts = await prisma.post.findMany({
  select: {
    id: true,
    title: true,
    createdAt: true,
    author: {
      select: {
        id: true,
        name: true
      }
    }
  }
});
```

#### Batch Operations

```typescript
// Bad: Multiple database calls
for (const email of emails) {
  await prisma.user.create({
    data: { email, name: 'User' }
  });
}

// Good: Single batch operation
await prisma.user.createMany({
  data: emails.map(email => ({ email, name: 'User' }))
});
```

#### Connection Pooling

```typescript
// Configure connection pool in schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  
  // Connection pool settings
  relationMode = "prisma"
}

// Or in DATABASE_URL
// postgresql://user:pass@localhost:5432/db?connection_limit=10&pool_timeout=20
```

---

## Phase 6: REST API Development

### Module 6.1: Express.js Setup

#### Project Setup

```bash
# Install Express and dependencies
npm install express
npm install --save-dev @types/express

# Install additional middleware
npm install cors helmet morgan
npm install dotenv

# For request validation
npm install joi
npm install --save-dev @types/joi
```

#### Basic Express Server

```typescript
// src/server.ts - Express server setup

import express, { Express, Request, Response, NextFunction } from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import dotenv from 'dotenv';

// Load environment variables
dotenv.config();

// Create Express app
const app: Express = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(helmet());                          // Security headers
app.use(cors());                            // Enable CORS
app.use(morgan('dev'));                     // Request logging
app.use(express.json());                    // Parse JSON bodies
app.use(express.urlencoded({ extended: true }));  // Parse URL-encoded bodies

// Health check endpoint
app.get('/health', (req: Request, res: Response) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// 404 handler
app.use((req: Request, res: Response) => {
  res.status(404).json({ error: 'Route not found' });
});

// Error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});

export default app;
```

### Module 6.2: User CRUD API

#### User Routes

```typescript
// src/routes/user.routes.ts

import { Router } from 'express';
import * as userController from '../controllers/user.controller';
import { validateUser, validateUserId } from '../middlewares/validation';

const router = Router();

// User routes
router.get('/users', userController.getAllUsers);           // GET /api/users
router.get('/users/:id', validateUserId, userController.getUserById);  // GET /api/users/:id
router.post('/users', validateUser, userController.createUser);        // POST /api/users
router.put('/users/:id', validateUserId, validateUser, userController.updateUser);  // PUT /api/users/:id
router.delete('/users/:id', validateUserId, userController.deleteUser);  // DELETE /api/users/:id

export default router;
```

#### User Controller

```typescript
// src/controllers/user.controller.ts

import { Request, Response } from 'express';
import prisma from '../prisma';

// GET /api/users - Get all users
export const getAllUsers = async (req: Request, res: Response) => {
  try {
    const { page = 1, limit = 10, search, sortBy = 'createdAt', order = 'desc' } = req.query;
    
    // Build where clause
    const where: any = {};
    if (search) {
      where.OR = [
        { name: { contains: search as string, mode: 'insensitive' } },
        { email: { contains: search as string, mode: 'insensitive' } }
      ];
    }
    
    // Calculate pagination
    const skip = (Number(page) - 1) * Number(limit);
    const take = Number(limit);
    
    // Execute query with count
    const [users, total] = await Promise.all([
      prisma.user.findMany({
        where,
        skip,
        take,
        orderBy: { [sortBy as string]: order },
        select: {
          id: true,
          email: true,
          name: true,
          age: true,
          isActive: true,
          createdAt: true,
          updatedAt: true
        }
      }),
      prisma.user.count({ where })
    ]);
    
    // Return paginated response
    res.json({
      data: users,
      pagination: {
        total,
        page: Number(page),
        limit: Number(limit),
        totalPages: Math.ceil(total / Number(limit))
      }
    });
  } catch (error) {
    console.error('Error fetching users:', error);
    res.status(500).json({ error: 'Failed to fetch users' });
  }
};

// GET /api/users/:id - Get user by ID
export const getUserById = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    
    const user = await prisma.user.findUnique({
      where: { id: Number(id) },
      include: {
        posts: {
          select: {
            id: true,
            title: true,
            published: true,
            createdAt: true
          },
          orderBy: { createdAt: 'desc' },
          take: 10  // Limit to 10 most recent posts
        },
        _count: {
          select: {
            posts: true
          }
        }
      }
    });
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    console.error('Error fetching user:', error);
    res.status(500).json({ error: 'Failed to fetch user' });
  }
};

// POST /api/users - Create new user
export const createUser = async (req: Request, res: Response) => {
  try {
    const { email, name, age } = req.body;
    
    // Check if user already exists
    const existingUser = await prisma.user.findUnique({
      where: { email }
    });
    
    if (existingUser) {
      return res.status(400).json({ error: 'User with this email already exists' });
    }
    
    // Create user
    const user = await prisma.user.create({
      data: {
        email,
        name,
        age
      }
    });
    
    res.status(201).json(user);
  } catch (error) {
    console.error('Error creating user:', error);
    res.status(500).json({ error: 'Failed to create user' });
  }
};

// PUT /api/users/:id - Update user
export const updateUser = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const { email, name, age, isActive } = req.body;
    
    // Check if user exists
    const existingUser = await prisma.user.findUnique({
      where: { id: Number(id) }
    });
    
    if (!existingUser) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    // Update user
    const user = await prisma.user.update({
      where: { id: Number(id) },
      data: {
        ...(email && { email }),
        ...(name && { name }),
        ...(age !== undefined && { age }),
        ...(isActive !== undefined && { isActive })
      }
    });
    
    res.json(user);
  } catch (error) {
    console.error('Error updating user:', error);
    res.status(500).json({ error: 'Failed to update user' });
  }
};

// DELETE /api/users/:id - Delete user
export const deleteUser = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    
    // Check if user exists
    const existingUser = await prisma.user.findUnique({
      where: { id: Number(id) }
    });
    
    if (!existingUser) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    // Delete user
    await prisma.user.delete({
      where: { id: Number(id) }
    });
    
    res.json({ message: 'User deleted successfully' });
  } catch (error) {
    console.error('Error deleting user:', error);
    res.status(500).json({ error: 'Failed to delete user' });
  }
};
```

#### Validation Middleware

```typescript
// src/middlewares/validation.ts

import { Request, Response, NextFunction } from 'express';
import Joi from 'joi';

// User validation schema
const userSchema = Joi.object({
  email: Joi.string().email().required(),
  name: Joi.string().min(2).max(100).required(),
  age: Joi.number().integer().min(0).max(150).required(),
  isActive: Joi.boolean().optional()
});

// Validate user data
export const validateUser = (req: Request, res: Response, next: NextFunction) => {
  const { error } = userSchema.validate(req.body);
  
  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }
  
  next();
};

// Validate user ID parameter
export const validateUserId = (req: Request, res: Response, next: NextFunction) => {
  const { id } = req.params;
  
  if (!id || isNaN(Number(id))) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  
  next();
};
```

#### Integrating Routes

```typescript
// src/server.ts - Updated with routes

import express from 'express';
import userRoutes from './routes/user.routes';

const app = express();

// ... middleware setup ...

// API routes
app.use('/api', userRoutes);

// ... error handlers ...

export default app;
```

### Module 6.3: Post CRUD API

#### Post Schema Update

```prisma
// prisma/schema.prisma - Add Post model

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  age       Int
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  posts     Post[]   // One-to-many relation
}

model Post {
  id          Int      @id @default(autoincrement())
  title       String
  content     String   @db.Text
  published   Boolean  @default(false)
  views       Int      @default(0)
  tags        String[]  // Array field (PostgreSQL)
  authorId    Int
  author      User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([authorId])
  @@index([published])
}
```

```bash
# Run migration
npx prisma migrate dev --name add_post_model
```

#### Post Controller

```typescript
// src/controllers/post.controller.ts

import { Request, Response } from 'express';
import prisma from '../prisma';

// GET /api/posts - Get all posts
export const getAllPosts = async (req: Request, res: Response) => {
  try {
    const { 
      page = 1, 
      limit = 10, 
      search, 
      published, 
      authorId,
      sortBy = 'createdAt',
      order = 'desc'
    } = req.query;
    
    // Build where clause
    const where: any = {};
    
    if (search) {
      where.OR = [
        { title: { contains: search as string, mode: 'insensitive' } },
        { content: { contains: search as string, mode: 'insensitive' } }
      ];
    }
    
    if (published !== undefined) {
      where.published = published === 'true';
    }
    
    if (authorId) {
      where.authorId = Number(authorId);
    }
    
    // Pagination
    const skip = (Number(page) - 1) * Number(limit);
    const take = Number(limit);
    
    // Query with author details
    const [posts, total] = await Promise.all([
      prisma.post.findMany({
        where,
        skip,
        take,
        orderBy: { [sortBy as string]: order },
        include: {
          author: {
            select: {
              id: true,
              name: true,
              email: true
            }
          }
        }
      }),
      prisma.post.count({ where })
    ]);
    
    res.json({
      data: posts,
      pagination: {
        total,
        page: Number(page),
        limit: Number(limit),
        totalPages: Math.ceil(total / Number(limit))
      }
    });
  } catch (error) {
    console.error('Error fetching posts:', error);
    res.status(500).json({ error: 'Failed to fetch posts' });
  }
};

// GET /api/posts/:id - Get post by ID
export const getPostById = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    
    // Increment view count
    const post = await prisma.post.update({
      where: { id: Number(id) },
      data: { views: { increment: 1 } },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            email: true
          }
        }
      }
    });
    
    if (!post) {
      return res.status(404).json({ error: 'Post not found' });
    }
    
    res.json(post);
  } catch (error) {
    console.error('Error fetching post:', error);
    res.status(500).json({ error: 'Failed to fetch post' });
  }
};

// POST /api/posts - Create new post
export const createPost = async (req: Request, res: Response) => {
  try {
    const { title, content, published, tags, authorId } = req.body;
    
    // Verify author exists
    const author = await prisma.user.findUnique({
      where: { id: authorId }
    });
    
    if (!author) {
      return res.status(404).json({ error: 'Author not found' });
    }
    
    // Create post
    const post = await prisma.post.create({
      data: {
        title,
        content,
        published: published || false,
        tags: tags || [],
        authorId
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            email: true
          }
        }
      }
    });
    
    res.status(201).json(post);
  } catch (error) {
    console.error('Error creating post:', error);
    res.status(500).json({ error: 'Failed to create post' });
  }
};

// PUT /api/posts/:id - Update post
export const updatePost = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const { title, content, published, tags } = req.body;
    
    // Check if post exists
    const existingPost = await prisma.post.findUnique({
      where: { id: Number(id) }
    });
    
    if (!existingPost) {
      return res.status(404).json({ error: 'Post not found' });
    }
    
    // Update post
    const post = await prisma.post.update({
      where: { id: Number(id) },
      data: {
        ...(title && { title }),
        ...(content && { content }),
        ...(published !== undefined && { published }),
        ...(tags && { tags })
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            email: true
          }
        }
      }
    });
    
    res.json(post);
  } catch (error) {
    console.error('Error updating post:', error);
    res.status(500).json({ error: 'Failed to update post' });
  }
};

// DELETE /api/posts/:id - Delete post
export const deletePost = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    
    // Check if post exists
    const existingPost = await prisma.post.findUnique({
      where: { id: Number(id) }
    });
    
    if (!existingPost) {
      return res.status(404).json({ error: 'Post not found' });
    }
    
    // Delete post
    await prisma.post.delete({
      where: { id: Number(id) }
    });
    
    res.json({ message: 'Post deleted successfully' });
  } catch (error) {
    console.error('Error deleting post:', error);
    res.status(500).json({ error: 'Failed to delete post' });
  }
};

// POST /api/posts/:id/publish - Publish post
export const publishPost = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    
    const post = await prisma.post.update({
      where: { id: Number(id) },
      data: { published: true }
    });
    
    res.json(post);
  } catch (error) {
    console.error('Error publishing post:', error);
    res.status(500).json({ error: 'Failed to publish post' });
  }
};
```

#### Post Routes

```typescript
// src/routes/post.routes.ts

import { Router } from 'express';
import * as postController from '../controllers/post.controller';

const router = Router();

router.get('/posts', postController.getAllPosts);
router.get('/posts/:id', postController.getPostById);
router.post('/posts', postController.createPost);
router.put('/posts/:id', postController.updatePost);
router.delete('/posts/:id', postController.deletePost);
router.post('/posts/:id/publish', postController.publishPost);

export default router;
```

### Module 6.4: Multi-Database REST API

#### MySQL-specific API

```typescript
// src/controllers/product.controller.ts (MySQL)

import { Request, Response } from 'express';
import { mysqlClient } from '../prisma-mysql';

// Product model in MySQL
export const getAllProducts = async (req: Request, res: Response) => {
  try {
    const products = await mysqlClient.product.findMany({
      orderBy: { createdAt: 'desc' }
    });
    
    res.json(products);
  } catch (error) {
    console.error('Error fetching products:', error);
    res.status(500).json({ error: 'Failed to fetch products' });
  }
};

export const createProduct = async (req: Request, res: Response) => {
  try {
    const { name, description, price, stock } = req.body;
    
    const product = await mysqlClient.product.create({
      data: {
        name,
        description,
        price,
        stock
      }
    });
    
    res.status(201).json(product);
  } catch (error) {
    console.error('Error creating product:', error);
    res.status(500).json({ error: 'Failed to create product' });
  }
};
```

#### MongoDB-specific API

```typescript
// src/controllers/article.controller.ts (MongoDB)

import { Request, Response } from 'express';
import { mongoClient } from '../prisma-mongo';

// Article model in MongoDB
export const getAllArticles = async (req: Request, res: Response) => {
  try {
    const articles = await mongoClient.article.findMany({
      orderBy: { createdAt: 'desc' }
    });
    
    res.json(articles);
  } catch (error) {
    console.error('Error fetching articles:', error);
    res.status(500).json({ error: 'Failed to fetch articles' });
  }
};

export const createArticle = async (req: Request, res: Response) => {
  try {
    const { title, content, tags, metadata } = req.body;
    
    const article = await mongoClient.article.create({
      data: {
        title,
        content,
        tags,
        metadata
      }
    });
    
    res.status(201).json(article);
  } catch (error) {
    console.error('Error creating article:', error);
    res.status(500).json({ error: 'Failed to create article' });
  }
};
```

### Module 6.5: Error Handling & Best Practices

#### Custom Error Classes

```typescript
// src/utils/errors.ts

export class AppError extends Error {
  statusCode: number;
  isOperational: boolean;
  
  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(message: string = 'Resource not found') {
    super(message, 404);
  }
}

export class ValidationError extends AppError {
  constructor(message: string = 'Validation failed') {
    super(message, 400);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message: string = 'Unauthorized') {
    super(message, 401);
  }
}
```

#### Async Error Handler

```typescript
// src/utils/asyncHandler.ts

import { Request, Response, NextFunction } from 'express';

type AsyncFunction = (
  req: Request,
  res: Response,
  next: NextFunction
) => Promise<any>;

export const asyncHandler = (fn: AsyncFunction) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};
```

#### Global Error Handler

```typescript
// src/middlewares/errorHandler.ts

import { Request, Response, NextFunction } from 'express';
import { AppError } from '../utils/errors';
import { Prisma } from '@prisma/client';

export const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  // Prisma errors
  if (err instanceof Prisma.PrismaClientKnownRequestError) {
    // Unique constraint violation
    if (err.code === 'P2002') {
      return res.status(400).json({
        error: 'A record with this value already exists'
      });
    }
    
    // Record not found
    if (err.code === 'P2025') {
      return res.status(404).json({
        error: 'Record not found'
      });
    }
  }
  
  // Prisma validation error
  if (err instanceof Prisma.PrismaClientValidationError) {
    return res.status(400).json({
      error: 'Invalid data provided'
    });
  }
  
  // Custom app errors
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: err.message
    });
  }
  
  // Unknown errors
  console.error('Unexpected error:', err);
  res.status(500).json({
    error: 'An unexpected error occurred'
  });
};
```

#### Updated Controller with Error Handling

```typescript
// src/controllers/user.controller.ts - Improved version

import { Request, Response, NextFunction } from 'express';
import prisma from '../prisma';
import { asyncHandler } from '../utils/asyncHandler';
import { NotFoundError, ValidationError } from '../utils/errors';

export const getUserById = asyncHandler(async (req: Request, res: Response) => {
  const { id } = req.params;
  
  const user = await prisma.user.findUnique({
    where: { id: Number(id) },
    include: {
      posts: {
        take: 10,
        orderBy: { createdAt: 'desc' }
      }
    }
  });
  
  if (!user) {
    throw new NotFoundError('User not found');
  }
  
  res.json(user);
});

export const createUser = asyncHandler(async (req: Request, res: Response) => {
  const { email, name, age } = req.body;
  
  // Additional validation
  if (age < 0 || age > 150) {
    throw new ValidationError('Age must be between 0 and 150');
  }
  
  const user = await prisma.user.create({
    data: { email, name, age }
  });
  
  res.status(201).json(user);
});
```

---

## Phase 7: Migrations & Schema Management

### Module 7.1: Migration Basics

#### Creating Migrations

```bash
# Create a new migration (dev mode)
npx prisma migrate dev --name add_user_profile

# Create migration without applying (for review)
npx prisma migrate dev --create-only

# Apply pending migrations
npx prisma migrate deploy

# Reset database (WARNING: deletes all data)
npx prisma migrate reset
```

#### Migration Files

Migrations are stored in `prisma/migrations/` directory:

```
prisma/migrations/
├── 20231210120000_init/
│   └── migration.sql
├── 20231211083000_add_user_profile/
│   └── migration.sql
└── migration_lock.toml
```

**Example Migration SQL:**

```sql
-- CreateTable
CREATE TABLE "Profile" (
    "id" SERIAL NOT NULL,
    "bio" TEXT NOT NULL,
    "avatar" TEXT,
    "userId" INTEGER NOT NULL,

    CONSTRAINT "Profile_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "Profile_userId_key" ON "Profile"("userId");

-- AddForeignKey
ALTER TABLE "Profile" ADD CONSTRAINT "Profile_userId_fkey" 
    FOREIGN KEY ("userId") REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE;
```

### Module 7.2: Schema Changes

#### Adding Fields

```prisma
// Before
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
}

// After - Add new fields
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  phone     String?  // Optional field
  verified  Boolean  @default(false)  // With default
  bio       String   @default("")  // Non-nullable with default
  createdAt DateTime @default(now())
}
```

```bash
npx prisma migrate dev --name add_user_fields
```

#### Renaming Fields

```prisma
// Rename field using @map
model User {
  id        Int    @id @default(autoincrement())
  email     String @unique
  firstName String @map("name")  // Keeps DB column as "name"
}
```

Or manually edit migration:

```sql
-- Rename column
ALTER TABLE "User" RENAME COLUMN "name" TO "firstName";
```

#### Changing Field Types

```prisma
// Change age from Int to String
model User {
  id   Int    @id @default(autoincrement())
  age  String  // Changed from Int
}
```

**Migration will look like:**

```sql
-- AlterTable
ALTER TABLE "User" ALTER COLUMN "age" SET DATA TYPE TEXT;
```

**For data preservation:**

```sql
-- Custom migration with data conversion
ALTER TABLE "User" ADD COLUMN "age_new" TEXT;
UPDATE "User" SET "age_new" = CAST("age" AS TEXT);
ALTER TABLE "User" DROP COLUMN "age";
ALTER TABLE "User" RENAME COLUMN "age_new" TO "age";
```

### Module 7.3: Advanced Migration Techniques

#### Custom SQL in Migrations

```bash
# Create migration file only
npx prisma migrate dev --create-only --name add_custom_index
```

Edit generated migration:

```sql
-- CreateIndex
CREATE INDEX CONCURRENTLY "idx_user_email_name" 
    ON "User"("email", "name");

-- Add custom function
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW."updatedAt" = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Add trigger
CREATE TRIGGER update_user_updated_at BEFORE UPDATE ON "User"
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

Then apply:

```bash
npx prisma migrate dev
```

#### Data Migration

```bash
# Create migration for schema change
npx prisma migrate dev --create-only --name add_full_name
```

Edit migration to include data transformation:

```sql
-- Add new column
ALTER TABLE "User" ADD COLUMN "fullName" TEXT;

-- Migrate data
UPDATE "User" SET "fullName" = "firstName" || ' ' || "lastName";

-- Make column required
ALTER TABLE "User" ALTER COLUMN "fullName" SET NOT NULL;

-- Optionally drop old columns
-- ALTER TABLE "User" DROP COLUMN "firstName";
-- ALTER TABLE "User" DROP COLUMN "lastName";
```

### Module 7.4: Database Push (Development)

#### Using db push

```bash
# Push schema to database without creating migration
npx prisma db push

# Skip generating Prisma Client
npx prisma db push --skip-generate

# Accept data loss warnings
npx prisma db push --accept-data-loss
```

**When to use `db push`:**
- Rapid prototyping
- MongoDB (no migrations support)
- Schema exploration
- Development environments

**When to use migrations:**
- Production environments
- Team collaboration
- Version control of schema changes
- Rollback capability

### Module 7.5: Production Migration Strategy

#### Deployment Workflow

```bash
# 1. In development: Create and test migration
npx prisma migrate dev --name add_feature

# 2. Commit migration files to version control
git add prisma/migrations
git commit -m "Add feature migration"

# 3. In production: Apply migrations
npx prisma migrate deploy

# 4. Generate Prisma Client
npx prisma generate
```

#### Zero-Downtime Migrations

**Adding Required Field:**

Step 1 - Make field optional:
```prisma
model User {
  newField String?  // Optional first
}
```

Step 2 - Deploy and update app to write to new field

Step 3 - Backfill existing data:
```sql
UPDATE "User" SET "newField" = 'default' WHERE "newField" IS NULL;
```

Step 4 - Make field required:
```prisma
model User {
  newField String  // Now required
}
```

**Renaming Field:**

Step 1 - Add new field:
```prisma
model User {
  oldField String
  newField String?
}
```

Step 2 - Update app to write to both fields

Step 3 - Backfill data:
```sql
UPDATE "User" SET "newField" = "oldField" WHERE "newField" IS NULL;
```

Step 4 - Update app to read from new field

Step 5 - Remove old field:
```prisma
model User {
  newField String
}
```

---

## Phase 8: Performance & Optimization

### Module 8.1: Query Optimization

#### N+1 Problem

**Bad - N+1 Queries:**

```typescript
// This makes 1 query for users + N queries for each user's posts
const users = await prisma.user.findMany();

for (const user of users) {
  const posts = await prisma.post.findMany({
    where: { authorId: user.id }
  });
  console.log(`${user.name} has ${posts.length} posts`);
}
// Total: 1 + N queries
```

**Good - Single Query with Include:**

```typescript
// This makes 1 query using JOIN
const users = await prisma.user.findMany({
  include: {
    posts: true
  }
});

for (const user of users) {
  console.log(`${user.name} has ${user.posts.length} posts`);
}
// Total: 1 query
```

#### Select Only Needed Fields

```typescript
// Bad - Fetches all fields including large TEXT fields
const posts = await prisma.post.findMany();

// Good - Select only needed fields
const posts = await prisma.post.findMany({
  select: {
    id: true,
    title: true,
    createdAt: true,
    author: {
      select: {
        name: true
      }
    }
  }
});
```

#### Pagination Best Practices

```typescript
// Offset pagination (simple but slow for large offsets)
const posts = await prisma.post.findMany({
  skip: 1000,  // Slow for large values
  take: 10,
  orderBy: { id: 'asc' }
});

// Cursor pagination (fast for large datasets)
const posts = await prisma.post.findMany({
  take: 10,
  skip: 1,
  cursor: {
    id: lastPostId
  },
  orderBy: { id: 'asc' }
});
```

### Module 8.2: Database Indexes

#### Adding Indexes

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique  // Automatic index
  firstName String
  lastName  String
  age       Int
  city      String
  country   String
  createdAt DateTime @default(now())
  
  // Single field index for frequent queries
  @@index([email])
  
  // Composite index for common filter combinations
  @@index([city, country])
  
  // Index for sorting
  @@index([createdAt(sort: Desc)])
  
  // Covering index (includes additional fields)
  @@index([lastName, firstName])
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  published Boolean
  authorId  Int
  createdAt DateTime @default(now())
  
  // Foreign key indexes
  @@index([authorId])
  
  // Filtered index
  @@index([published, createdAt])
}
```

#### Index Strategy

```typescript
// Query that benefits from index
const users = await prisma.user.findMany({
  where: {
    city: 'New York',
    country: 'USA'
  },
  orderBy: {
    lastName: 'asc'
  }
});
// Uses @@index([city, country]) and @@index([lastName, firstName])
```

### Module 8.3: Connection Pooling

#### Configure Connection Pool

```bash
# .env - PostgreSQL with connection pool settings
DATABASE_URL="postgresql://user:password@localhost:5432/mydb?connection_limit=10&pool_timeout=20"

# MySQL
DATABASE_URL="mysql://user:password@localhost:3306/mydb?connection_limit=10&pool_timeout=20"
```

#### Prisma Client Configuration

```typescript
// src/prisma.ts - Production configuration

import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
    errorFormat: 'minimal',
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;

export default prisma;
```

### Module 8.4: Query Analysis

#### Enable Query Logging

```typescript
const prisma = new PrismaClient({
  log: [
    {
      emit: 'event',
      level: 'query',
    },
  ],
});

prisma.$on('query', (e) => {
  console.log('Query: ' + e.query);
  console.log('Params: ' + e.params);
  console.log('Duration: ' + e.duration + 'ms');
});
```

#### Using EXPLAIN

```typescript
// Run EXPLAIN for query analysis (PostgreSQL)
const result = await prisma.$queryRaw`
  EXPLAIN ANALYZE
  SELECT * FROM "User" WHERE "city" = 'New York' AND "age" > 25;
`;

console.log(result);
```

### Module 8.5: Caching Strategies

#### Redis Caching

```bash
npm install redis
npm install --save-dev @types/node-redis
```

```typescript
// src/cache.ts - Redis cache implementation

import { createClient } from 'redis';

const redisClient = createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379'
});

redisClient.on('error', (err) => console.error('Redis error:', err));

await redisClient.connect();

export default redisClient;
```

```typescript
// src/controllers/user.controller.ts - With caching

import redisClient from '../cache';
import prisma from '../prisma';

export const getUserById = async (req: Request, res: Response) => {
  const { id } = req.params;
  const cacheKey = `user:${id}`;
  
  try {
    // Try cache first
    const cached = await redisClient.get(cacheKey);
    if (cached) {
      return res.json(JSON.parse(cached));
    }
    
    // Query database
    const user = await prisma.user.findUnique({
      where: { id: Number(id) },
      include: { posts: true }
    });
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    // Cache for 5 minutes
    await redisClient.setEx(cacheKey, 300, JSON.stringify(user));
    
    res.json(user);
  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
};

// Invalidate cache on update
export const updateUser = async (req: Request, res: Response) => {
  const { id } = req.params;
  
  const user = await prisma.user.update({
    where: { id: Number(id) },
    data: req.body
  });
  
  // Invalidate cache
  await redisClient.del(`user:${id}`);
  
  res.json(user);
};
```

---

## Phase 9: Production & Best Practices

### Module 9.1: Environment Configuration

#### Multiple Environment Setup

```bash
# .env.development
DATABASE_URL="postgresql://dev:dev@localhost:5432/myapp_dev"
NODE_ENV="development"
PORT=3000
LOG_LEVEL="debug"

# .env.test
DATABASE_URL="postgresql://test:test@localhost:5432/myapp_test"
NODE_ENV="test"

# .env.production
DATABASE_URL="postgresql://prod:secure_password@prod-db.example.com:5432/myapp_prod"
NODE_ENV="production"
PORT=8080
LOG_LEVEL="error"
```

#### Configuration Management

```typescript
// src/config/database.ts

import { PrismaClient } from '@prisma/client';

const prismaClientSingleton = () => {
  return new PrismaClient({
    log:
      process.env.NODE_ENV === 'development'
        ? ['query', 'error', 'warn']
        : ['error'],
  });
};

type PrismaClientSingleton = ReturnType<typeof prismaClientSingleton>;

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClientSingleton | undefined;
};

const prisma = globalForPrisma.prisma ?? prismaClientSingleton();

export default prisma;

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

### Module 9.2: Database Seeding

#### Seed Script

```typescript
// prisma/seed.ts

import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  console.log('Starting seed...');
  
  // Clear existing data
  await prisma.post.deleteMany();
  await prisma.user.deleteMany();
  
  // Create users
  const alice = await prisma.user.create({
    data: {
      email: 'alice@example.com',
      name: 'Alice',
      age: 28,
      posts: {
        create: [
          {
            title: 'Getting Started with Prisma',
            content: 'Prisma is an amazing ORM...',
            published: true,
            tags: ['prisma', 'database', 'nodejs']
          },
          {
            title: 'Advanced Prisma Techniques',
            content: 'Let\'s dive deep into Prisma...',
            published: true,
            tags: ['prisma', 'advanced']
          }
        ]
      }
    },
  });
  
  const bob = await prisma.user.create({
    data: {
      email: 'bob@example.com',
      name: 'Bob',
      age: 35,
      posts: {
        create: [
          {
            title: 'Node.js Best Practices',
            content: 'Here are some best practices...',
            published: true,
            tags: ['nodejs', 'javascript']
          }
        ]
      }
    },
  });
  
  console.log('Seed completed:', { alice, bob });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

#### Configure Seed in package.json

```json
{
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  },
  "scripts": {
    "seed": "ts-node prisma/seed.ts"
  }
}
```

```bash
# Run seed
npx prisma db seed

# Or
npm run seed
```

### Module 9.3: Testing

#### Test Database Setup

```typescript
// tests/setup.ts

import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL_TEST
    }
  }
});

export async function setupTestDatabase() {
  // Reset database
  execSync('npx prisma migrate reset --force --skip-seed', {
    env: {
      ...process.env,
      DATABASE_URL: process.env.DATABASE_URL_TEST
    }
  });
  
  // Run migrations
  execSync('npx prisma migrate deploy', {
    env: {
      ...process.env,
      DATABASE_URL: process.env.DATABASE_URL_TEST
    }
  });
}

export async function teardownTestDatabase() {
  await prisma.$disconnect();
}

export { prisma };
```

#### Integration Tests

```bash
npm install --save-dev jest @types/jest ts-jest supertest @types/supertest
```

```typescript
// tests/user.test.ts

import request from 'supertest';
import app from '../src/server';
import { prisma, setupTestDatabase, teardownTestDatabase } from './setup';

beforeAll(async () => {
  await setupTestDatabase();
});

afterAll(async () => {
  await teardownTestDatabase();
});

beforeEach(async () => {
  // Clear data before each test
  await prisma.post.deleteMany();
  await prisma.user.deleteMany();
});

describe('User API', () => {
  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'Test User',
        age: 25
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      expect(response.body).toMatchObject(userData);
      expect(response.body.id).toBeDefined();
    });
    
    it('should reject duplicate email', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'Test User',
        age: 25
      };
      
      // Create first user
      await prisma.user.create({ data: userData });
      
      // Try to create duplicate
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400);
      
      expect(response.body.error).toBeDefined();
    });
  });
  
  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      const user = await prisma.user.create({
        data: {
          email: 'test@example.com',
          name: 'Test User',
          age: 25
        }
      });
      
      const response = await request(app)
        .get(`/api/users/${user.id}`)
        .expect(200);
      
      expect(response.body.id).toBe(user.id);
      expect(response.body.email).toBe(user.email);
    });
    
    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/users/99999')
        .expect(404);
    });
  });
  
  describe('PUT /api/users/:id', () => {
    it('should update user', async () => {
      const user = await prisma.user.create({
        data: {
          email: 'test@example.com',
          name: 'Test User',
          age: 25
        }
      });
      
      const response = await request(app)
        .put(`/api/users/${user.id}`)
        .send({ name: 'Updated Name' })
        .expect(200);
      
      expect(response.body.name).toBe('Updated Name');
    });
  });
  
  describe('DELETE /api/users/:id', () => {
    it('should delete user', async () => {
      const user = await prisma.user.create({
        data: {
          email: 'test@example.com',
          name: 'Test User',
          age: 25
        }
      });
      
      await request(app)
        .delete(`/api/users/${user.id}`)
        .expect(200);
      
      const deletedUser = await prisma.user.findUnique({
        where: { id: user.id }
      });
      
      expect(deletedUser).toBeNull();
    });
  });
});
```

### Module 9.4: Security Best Practices

#### SQL Injection Prevention

```typescript
// SAFE - Prisma automatically prevents SQL injection
const email = req.query.email; // Even if malicious
const user = await prisma.user.findUnique({
  where: { email }
});

// UNSAFE - Raw queries with user input
const email = req.query.email;
const users = await prisma.$queryRawUnsafe(
  `SELECT * FROM User WHERE email = '${email}'`  // VULNERABLE!
);

// SAFE - Raw queries with parameterization
const email = req.query.email;
const users = await prisma.$queryRaw`
  SELECT * FROM User WHERE email = ${email}
`;
```

#### Input Validation

```typescript
import Joi from 'joi';

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  name: Joi.string().min(2).max(100).required(),
  age: Joi.number().integer().min(0).max(150).required(),
});

export const validateUser = (req: Request, res: Response, next: NextFunction) => {
  const { error } = userSchema.validate(req.body);
  
  if (error) {
    return res.status(400).json({
      error: error.details[0].message
    });
  }
  
  next();
};
```

#### Rate Limiting

```bash
npm install express-rate-limit
```

```typescript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later'
});

app.use('/api/', limiter);
```

#### Environment Variables Security

```typescript
// NEVER commit .env to version control
// Add to .gitignore
.env
.env.local
.env.production

// Use environment variable validation
import dotenv from 'dotenv';
import Joi from 'joi';

dotenv.config();

const envSchema = Joi.object({
  DATABASE_URL: Joi.string().required(),
  NODE_ENV: Joi.string().valid('development', 'production', 'test').required(),
  PORT: Joi.number().default(3000),
  JWT_SECRET: Joi.string().required()
}).unknown();

const { error, value: envVars } = envSchema.validate(process.env);

if (error) {
  throw new Error(`Config validation error: ${error.message}`);
}

export const config = {
  database: envVars.DATABASE_URL,
  nodeEnv: envVars.NODE_ENV,
  port: envVars.PORT,
  jwtSecret: envVars.JWT_SECRET
};
```

### Module 9.5: Deployment

#### Docker Setup

```dockerfile
# Dockerfile

FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY prisma ./prisma/

# Install dependencies
RUN npm ci --only=production

# Generate Prisma Client
RUN npx prisma generate

# Copy application code
COPY . .

# Build TypeScript
RUN npm run build

# Expose port
EXPOSE 3000

# Run migrations and start app
CMD ["sh", "-c", "npx prisma migrate deploy && npm start"]
```

```yaml
# docker-compose.yml

version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: prisma
      POSTGRES_PASSWORD: prisma
      POSTGRES_DB: myapp
    ports:
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  app:
    build: .
    ports:
      - '3000:3000'
    environment:
      DATABASE_URL: postgresql://prisma:prisma@postgres:5432/myapp
      NODE_ENV: production
    depends_on:
      - postgres

volumes:
  postgres_data:
```

```bash
# Build and run
docker-compose up --build
```

#### Production Checklist

```markdown
## Pre-Deployment Checklist

### Database
- [ ] Run all migrations in staging
- [ ] Test rollback procedures
- [ ] Configure connection pooling
- [ ] Set up database backups
- [ ] Create read replicas if needed

### Security
- [ ] Validate all environment variables
- [ ] Use strong database passwords
- [ ] Enable SSL for database connections
- [ ] Implement rate limiting
- [ ] Add request validation
- [ ] Configure CORS properly
- [ ] Enable helmet middleware

### Performance
- [ ] Add database indexes
- [ ] Implement caching strategy
- [ ] Configure connection pool size
- [ ] Enable query logging (temporarily)
- [ ] Set up monitoring

### Code
- [ ] Remove console.logs
- [ ] Add proper error handling
- [ ] Implement graceful shutdown
- [ ] Add health check endpoints
- [ ] Configure logging service

### Testing
- [ ] Run integration tests
- [ ] Perform load testing
- [ ] Test migration rollback
- [ ] Verify backup restoration
```

#### Graceful Shutdown

```typescript
// src/server.ts - Production server with graceful shutdown

import express from 'express';
import prisma from './prisma';

const app = express();
const PORT = process.env.PORT || 3000;

// ... middleware and routes ...

const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, closing server gracefully');
  
  server.close(async () => {
    console.log('HTTP server closed');
    
    // Disconnect Prisma
    await prisma.$disconnect();
    console.log('Database connections closed');
    
    process.exit(0);
  });
  
  // Force shutdown after 10 seconds
  setTimeout(() => {
    console.error('Forced shutdown after timeout');
    process.exit(1);
  }, 10000);
});

process.on('SIGINT', async () => {
  console.log('SIGINT received, closing server gracefully');
  server.close(async () => {
    await prisma.$disconnect();
    process.exit(0);
  });
});
```

---

## Appendix: Complete Example Projects

### Project 1: Blog API (PostgreSQL)

#### Complete Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String
  password  String    // Hashed
  bio       String?   @db.Text
  avatar    String?
  role      Role      @default(USER)
  isActive  Boolean   @default(true)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  
  posts     Post[]
  comments  Comment[]
  likes     Like[]
  
  @@index([email])
  @@map("users")
}

model Post {
  id          Int       @id @default(autoincrement())
  title       String
  slug        String    @unique
  content     String    @db.Text
  excerpt     String?
  coverImage  String?
  published   Boolean   @default(false)
  publishedAt DateTime?
  views       Int       @default(0)
  authorId    Int
  categoryId  Int?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  author      User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  category    Category? @relation(fields: [categoryId], references: [id], onDelete: SetNull)
  comments    Comment[]
  likes       Like[]
  tags        PostTag[]
  
  @@index([authorId])
  @@index([categoryId])
  @@index([published, publishedAt])
  @@index([slug])
  @@map("posts")
}

model Category {
  id          Int      @id @default(autoincrement())
  name        String   @unique
  slug        String   @unique
  description String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  posts       Post[]
  
  @@map("categories")
}

model Tag {
  id        Int       @id @default(autoincrement())
  name      String    @unique
  slug      String    @unique
  createdAt DateTime  @default(now())
  
  posts     PostTag[]
  
  @@map("tags")
}

model PostTag {
  postId    Int
  tagId     Int
  createdAt DateTime @default(now())
  
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  tag       Tag      @relation(fields: [tagId], references: [id], onDelete: Cascade)
  
  @@id([postId, tagId])
  @@map("post_tags")
}

model Comment {
  id        Int      @id @default(autoincrement())
  content   String   @db.Text
  postId    Int
  userId    Int
  parentId  Int?     // For nested comments
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id], onDelete: Cascade)
  replies   Comment[] @relation("CommentReplies")
  
  @@index([postId])
  @@index([userId])
  @@map("comments")
}

model Like {
  id        Int      @id @default(autoincrement())
  userId    Int
  postId    Int
  createdAt DateTime @default(now())
  
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  
  @@unique([userId, postId])
  @@map("likes")
}

enum Role {
  USER
  ADMIN
  MODERATOR
}
```

### Project 2: E-Commerce API (MySQL)

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("MYSQL_URL")
}

model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String
  phone     String?
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  
  addresses Address[]
  orders    Order[]
  cart      CartItem[]
  reviews   Review[]
  
  @@map("users")
}

model Product {
  id          Int       @id @default(autoincrement())
  name        String
  slug        String    @unique
  description String    @db.Text
  price       Decimal   @db.Decimal(10, 2)
  comparePrice Decimal? @db.Decimal(10, 2)
  stock       Int       @default(0)
  sku         String    @unique
  images      Json      // Array of image URLs
  categoryId  Int
  isActive    Boolean   @default(true)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  category    Category  @relation(fields: [categoryId], references: [id])
  orderItems  OrderItem[]
  cartItems   CartItem[]
  reviews     Review[]
  
  @@index([categoryId])
  @@index([slug])
  @@map("products")
}

model Category {
  id          Int       @id @default(autoincrement())
  name        String    @unique
  slug        String    @unique
  description String?   @db.Text
  parentId    Int?
  createdAt   DateTime  @default(now())
  
  parent      Category?  @relation("CategoryTree", fields: [parentId], references: [id], onDelete: Cascade)
  children    Category[] @relation("CategoryTree")
  products    Product[]
  
  @@map("categories")
}

model Address {
  id         Int     @id @default(autoincrement())
  userId     Int
  street     String
  city       String
  state      String
  zipCode    String
  country    String
  isDefault  Boolean @default(false)
  
  user       User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  orders     Order[]
  
  @@index([userId])
  @@map("addresses")
}

model Order {
  id              Int         @id @default(autoincrement())
  orderNumber     String      @unique
  userId          Int
  status          OrderStatus @default(PENDING)
  subtotal        Decimal     @db.Decimal(10, 2)
  tax             Decimal     @db.Decimal(10, 2)
  shipping        Decimal     @db.Decimal(10, 2)
  total           Decimal     @db.Decimal(10, 2)
  shippingAddressId Int
  createdAt       DateTime    @default(now())
  updatedAt       DateTime    @updatedAt
  
  user            User        @relation(fields: [userId], references: [id])
  shippingAddress Address     @relation(fields: [shippingAddressId], references: [id])
  items           OrderItem[]
  
  @@index([userId])
  @@index([status])
  @@map("orders")
}

model OrderItem {
  id        Int     @id @default(autoincrement())
  orderId   Int
  productId Int
  quantity  Int
  price     Decimal @db.Decimal(10, 2)
  
  order     Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product   Product @relation(fields: [productId], references: [id])
  
  @@index([orderId])
  @@map("order_items")
}

model CartItem {
  id        Int      @id @default(autoincrement())
  userId    Int
  productId Int
  quantity  Int      @default(1)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  
  @@unique([userId, productId])
  @@map("cart_items")
}

model Review {
  id        Int      @id @default(autoincrement())
  userId    Int
  productId Int
  rating    Int      // 1-5
  title     String?
  comment   String?  @db.Text
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  
  @@unique([userId, productId])
  @@index([productId])
  @@map("reviews")
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
}
```

### Project 3: Social Media API (MongoDB)

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongodb"
  url      = env("MONGO_URL")
}

model User {
  id            String    @id @default(auto()) @map("_id") @db.ObjectId
  username      String    @unique
  email         String    @unique
  name          String
  bio           String?
  avatar        String?
  coverPhoto    String?
  verified      Boolean   @default(false)
  followers     String[]  @db.ObjectId
  following     String[]  @db.ObjectId
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  posts         Post[]
  comments      Comment[]
  likes         Like[]
  
  @@map("users")
}

model Post {
  id          String    @id @default(auto()) @map("_id") @db.ObjectId
  content     String
  images      String[]  // Array of image URLs
  authorId    String    @db.ObjectId
  likes       Int       @default(0)
  shares      Int       @default(0)
  visibility  Visibility @default(PUBLIC)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  author      User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments    Comment[]
  likeRecords Like[]
  
  @@map("posts")
}

model Comment {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  content   String
  postId    String   @db.ObjectId
  userId    String   @db.ObjectId
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@map("comments")
}

model Like {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  postId    String   @db.ObjectId
  userId    String   @db.ObjectId
  createdAt DateTime @default(now())
  
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([postId, userId])
  @@map("likes")
}

enum Visibility {
  PUBLIC
  FRIENDS
  PRIVATE
}
```

---

## Conclusion

Congratulations! You've completed the Prisma Complete Mastery Tutorial. You now have comprehensive knowledge of:

✅ **Prisma Fundamentals** - Understanding ORM, schema design, and Prisma architecture
✅ **Multi-Database Support** - Working with PostgreSQL, MySQL, and MongoDB
✅ **CRUD Operations** - Mastering create, read, update, and delete operations
✅ **Advanced Queries** - Complex filtering, relations, transactions, and aggregations
✅ **REST API Development** - Building production-ready APIs with Express.js
✅ **Migrations** - Managing schema changes and database versioning
✅ **Performance Optimization** - Indexes, connection pooling, caching, and query optimization
✅ **Production Deployment** - Testing, security, Docker, and best practices

### Next Steps

1. **Build Real Projects** - Apply your knowledge to build actual applications
2. **Explore Advanced Topics**:
   - GraphQL with Prisma
   - Real-time subscriptions
   - Multi-tenancy
   - Database sharding
3. **Join the Community**:
   - [Prisma GitHub](https://github.com/prisma/prisma)
   - [Prisma Discord](https://pris.ly/discord)
   - [Prisma Documentation](https://www.prisma.io/docs)

### Resources

- **Official Docs**: https://www.prisma.io/docs
- **Prisma Examples**: https://github.com/prisma/prisma-examples
- **Prisma Blog**: https://www.prisma.io/blog
- **YouTube Channel**: https://www.youtube.com/c/PrismaData

---

**Happy Coding with Prisma! 🚀**