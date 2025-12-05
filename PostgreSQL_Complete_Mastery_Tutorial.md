# PostgreSQL Complete Mastery Tutorial
## From Beginner to Source Code Deep Dive

---

## Table of Contents

### Part 1: Beginner Level
1. [Introduction to PostgreSQL](#1-introduction-to-postgresql)
2. [Installation and Setup](#2-installation-and-setup)
3. [Basic SQL Operations](#3-basic-sql-operations)
4. [Data Types](#4-data-types)
5. [Constraints](#5-constraints)
6. [Basic Queries](#6-basic-queries)

### Part 2: Intermediate Level
7. [Advanced Queries](#7-advanced-queries)
8. [Joins and Relationships](#8-joins-and-relationships)
9. [Indexes](#9-indexes)
10. [Views and Materialized Views](#10-views-and-materialized-views)
11. [Transactions and ACID](#11-transactions-and-acid)
12. [Stored Procedures and Functions](#12-stored-procedures-and-functions)

### Part 3: Advanced Level
13. [Query Optimization](#13-query-optimization)
14. [Advanced Indexing Strategies](#14-advanced-indexing-strategies)
15. [Partitioning](#15-partitioning)
16. [Replication](#16-replication)
17. [Performance Tuning](#17-performance-tuning)
18. [Advanced Features](#18-advanced-features)

### Part 4: PostgreSQL Internals & Source Code
19. [Architecture Overview](#19-architecture-overview)
20. [Process Model](#20-process-model)
21. [Memory Management](#21-memory-management)
22. [Storage Engine](#22-storage-engine)
23. [Query Processing Pipeline](#23-query-processing-pipeline)
24. [Source Code Deep Dive](#24-source-code-deep-dive)

---

## Part 1: Beginner Level

### 1. Introduction to PostgreSQL

#### What is PostgreSQL?
// PostgreSQL is an advanced, open-source object-relational database management system (ORDBMS)
// It emphasizes extensibility and SQL compliance
// Originally developed at UC Berkeley, first released in 1996

#### Key Features:
- **ACID Compliance**: Ensures reliable transactions (Atomicity, Consistency, Isolation, Durability)
- **MVCC**: Multi-Version Concurrency Control for high concurrency without read locks
- **Extensibility**: Custom data types, operators, functions, and even procedural languages
- **Standards Compliance**: Highly SQL:2016 compliant with many advanced features
- **Advanced Features**: JSON/JSONB, full-text search, GIS support via PostGIS extension

#### PostgreSQL vs Other RDBMS:
```sql
-- PostgreSQL supports advanced features out of the box
-- Example: Array data types (not available in MySQL without workarounds)
CREATE TABLE products (
    id SERIAL PRIMARY KEY,           // SERIAL is auto-incrementing integer
    name VARCHAR(100),
    tags TEXT[]                      // Array column stores multiple text values
);

-- Insert data with arrays using ARRAY constructor
INSERT INTO products (name, tags) 
VALUES ('Laptop', ARRAY['electronics', 'computers', 'portable']);

-- Query arrays using ANY operator to check if value exists in array
SELECT * FROM products WHERE 'electronics' = ANY(tags);

-- Alternative: use contains operator @> to check if array contains subarray
SELECT * FROM products WHERE tags @> ARRAY['computers'];
```

#### Use Cases:
- Web applications requiring complex queries and data integrity
- Data warehousing and analytics with complex aggregations
- Geospatial applications (PostGIS extension provides GIS functionality)
- Financial systems requiring strict ACID compliance
- Applications needing JSON document storage with SQL querying capabilities

---

### 2. Installation and Setup

#### Linux Installation (Ubuntu/Debian):
```bash
# Update package list to get latest repository information
sudo apt update

# Install PostgreSQL and additional contributed modules
sudo apt install postgresql postgresql-contrib

# Check PostgreSQL service status
sudo systemctl status postgresql

# Start PostgreSQL service if not running
sudo systemctl start postgresql

# Enable auto-start on system boot
sudo systemctl enable postgresql

# Check installed version
psql --version
```

#### Windows Installation:
```powershell
# Download installer from postgresql.org
# Run the installer (installs PostgreSQL server, pgAdmin, command line tools)
# Default installation creates 'postgres' superuser account

# Connect using psql command line tool
# -U specifies username, postgres is default superuser
psql -U postgres

# If connection fails, check if PostgreSQL service is running
# Open Services (services.msc) and start postgresql-x64-{version}
```

#### macOS Installation:
```bash
# Using Homebrew package manager (install from brew.sh if needed)
brew install postgresql

# Start PostgreSQL service using brew services
brew services start postgresql

# Or start manually for current session
pg_ctl -D /usr/local/var/postgres start

# Connect to default postgres database
psql postgres
```

#### Initial Configuration:
```sql
-- Connect as postgres superuser (Linux/Mac: use sudo -u postgres psql)
sudo -u postgres psql

-- Create a new user with password authentication
-- WITH PASSWORD creates encrypted password
CREATE USER myuser WITH PASSWORD 'mypassword';

-- Create a database with specific owner
-- Owner has all privileges on database
CREATE DATABASE mydb OWNER myuser;

-- Grant all privileges on database to user (if not owner)
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;

-- List all databases with \l meta-command
\l

-- Connect to the newly created database
\c mydb

-- List all users and their roles
\du

-- Create user with additional attributes
CREATE USER admin_user WITH 
    PASSWORD 'secure_password' 
    CREATEDB              // Can create databases
    CREATEROLE            // Can create roles
    LOGIN                 // Can login
    SUPERUSER;            // Has all privileges

-- Exit psql
\q
```

#### Configuration Files:
```bash
# Main configuration file: postgresql.conf
# Location varies by OS:
# Linux: /etc/postgresql/{version}/main/postgresql.conf
# Mac: /usr/local/var/postgres/postgresql.conf
# Windows: C:\Program Files\PostgreSQL\{version}\data\postgresql.conf

# Key settings to configure:
# max_connections = 100           # Maximum concurrent connections
# shared_buffers = 128MB          # Memory for caching data (25% of RAM recommended)
# work_mem = 4MB                  # Memory for sorting and hash operations
# maintenance_work_mem = 64MB     # Memory for VACUUM, CREATE INDEX
# effective_cache_size = 4GB      # Estimate of OS cache (50-75% of RAM)

# Client authentication file: pg_hba.conf
# Controls who can connect from where and how they authenticate
# Format: TYPE  DATABASE  USER  ADDRESS  METHOD
# Example:
# local   all       all              peer
# host    all       all   127.0.0.1/32   md5
# host    all       all   ::1/128        md5

# Reload configuration without restart:
SELECT pg_reload_conf();

# View current settings:
SHOW all;
SHOW shared_buffers;
```

---

### 3. Basic SQL Operations

#### CREATE Operations:
```sql
-- Create a database with specific encoding and locale
CREATE DATABASE company 
    ENCODING 'UTF8'              // Character encoding
    LC_COLLATE 'en_US.UTF-8'     // Sort order
    LC_CTYPE 'en_US.UTF-8'       // Character classification
    TEMPLATE template0;          // Base template

-- Connect to database
\c company

-- Create a simple table with various constraints
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,                    // SERIAL: auto-incrementing integer, PRIMARY KEY: unique identifier
    first_name VARCHAR(50) NOT NULL,          // VARCHAR(n): variable-length string with max length, NOT NULL: required field
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,                // UNIQUE: no duplicate values allowed
    hire_date DATE DEFAULT CURRENT_DATE,      // DEFAULT: value if not specified
    salary NUMERIC(10, 2)                     // NUMERIC(precision, scale): exact decimal numbers
);

-- Create table with multiple constraints and documentation
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL UNIQUE,   // Combination of NOT NULL and UNIQUE
    location VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,    // TIMESTAMP: date and time
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create junction table with composite primary key and foreign keys
CREATE TABLE employee_departments (
    emp_id INTEGER REFERENCES employees(id) ON DELETE CASCADE,      // Foreign key with cascade delete
    dept_id INTEGER REFERENCES departments(dept_id) ON DELETE CASCADE,
    joined_date DATE DEFAULT CURRENT_DATE,
    is_primary BOOLEAN DEFAULT TRUE,
    PRIMARY KEY (emp_id, dept_id)             // Composite primary key: both columns together must be unique
);

-- Create table with CHECK constraints
CREATE TABLE salary_history (
    id SERIAL PRIMARY KEY,
    employee_id INTEGER REFERENCES employees(id),
    old_salary NUMERIC(10, 2),
    new_salary NUMERIC(10, 2),
    change_date DATE DEFAULT CURRENT_DATE,
    CHECK (new_salary >= 0),                  // Ensure salary is non-negative
    CHECK (new_salary != old_salary)          // Ensure there's actually a change
);
```

#### INSERT Operations:
```sql
-- Insert single row with explicit column names (recommended)
INSERT INTO departments (dept_name, location)
VALUES ('Engineering', 'San Francisco');

-- Insert multiple rows in one statement (efficient for bulk inserts)
INSERT INTO departments (dept_name, location)
VALUES 
    ('Marketing', 'New York'),
    ('Sales', 'Chicago'),
    ('HR', 'Boston'),
    ('Finance', 'Boston');

-- Insert with RETURNING clause (PostgreSQL-specific feature)
-- Returns specified columns of inserted rows
INSERT INTO employees (first_name, last_name, email, salary)
VALUES ('John', 'Doe', 'john.doe@company.com', 75000.00)
RETURNING id, first_name, last_name;        // Returns: id, first_name, last_name of inserted row

-- Insert from SELECT query (copy data from another table or query result)
INSERT INTO employees (first_name, last_name, email, salary)
SELECT first_name, last_name, email, salary
FROM temp_employees
WHERE status = 'approved';

-- Insert with DEFAULT keyword for columns with default values
INSERT INTO employees (first_name, last_name, email, salary, hire_date)
VALUES ('Jane', 'Smith', 'jane.smith@company.com', 70000, DEFAULT);

-- Insert with only required columns (others use DEFAULT or NULL)
INSERT INTO employees (first_name, last_name, email)
VALUES ('Bob', 'Johnson', 'bob.johnson@company.com');
// hire_date will use DEFAULT CURRENT_DATE, salary will be NULL

-- Insert with ON CONFLICT (UPSERT - insert or update)
INSERT INTO employees (id, first_name, last_name, email, salary)
VALUES (1, 'John', 'Doe', 'john.doe@company.com', 80000)
ON CONFLICT (id)                           // If id already exists
DO UPDATE SET salary = EXCLUDED.salary;    // Update salary to new value

-- ON CONFLICT DO NOTHING (ignore duplicates)
INSERT INTO employees (email, first_name, last_name)
VALUES ('duplicate@company.com', 'Test', 'User')
ON CONFLICT (email) DO NOTHING;
```

#### SELECT Operations:
```sql
-- Select all columns from table (use sparingly in production)
SELECT * FROM employees;

-- Select specific columns (better performance, clearer intent)
SELECT first_name, last_name, salary FROM employees;

-- Select with WHERE clause for filtering
SELECT * FROM employees WHERE salary > 60000;

-- Multiple conditions with AND (both must be true)
SELECT * FROM employees 
WHERE salary > 60000 AND hire_date > '2023-01-01';

-- Multiple conditions with OR (at least one must be true)
SELECT * FROM employees 
WHERE department = 'Engineering' OR department = 'Marketing';

-- IN operator for multiple value matching (cleaner than multiple ORs)
SELECT * FROM employees 
WHERE department IN ('Engineering', 'Marketing', 'Sales');

-- NOT IN to exclude values
SELECT * FROM employees 
WHERE department NOT IN ('HR', 'Finance');

-- LIKE operator for pattern matching
// % matches any sequence of characters, _ matches single character
SELECT * FROM employees WHERE email LIKE '%@company.com';     // Ends with @company.com
SELECT * FROM employees WHERE first_name LIKE 'J%';           // Starts with J
SELECT * FROM employees WHERE last_name LIKE '%son';          // Ends with son
SELECT * FROM employees WHERE first_name LIKE '_ohn';         // Four chars, last three are 'ohn'

-- ILIKE for case-insensitive pattern matching (PostgreSQL-specific)
SELECT * FROM employees WHERE email ILIKE '%COMPANY%';

-- BETWEEN for range queries (inclusive on both ends)
SELECT * FROM employees 
WHERE salary BETWEEN 50000 AND 80000;

SELECT * FROM employees 
WHERE hire_date BETWEEN '2023-01-01' AND '2023-12-31';

-- IS NULL and IS NOT NULL (cannot use = or != with NULL)
SELECT * FROM employees WHERE middle_name IS NULL;
SELECT * FROM employees WHERE middle_name IS NOT NULL;

-- ORDER BY for sorting results
SELECT * FROM employees ORDER BY salary DESC;           // Descending order
SELECT * FROM employees ORDER BY salary ASC;            // Ascending (default)
SELECT * FROM employees ORDER BY last_name, first_name; // Sort by multiple columns

-- LIMIT and OFFSET for pagination
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 10;           // First 10 rows
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 10 OFFSET 20; // Rows 21-30

-- DISTINCT to remove duplicates
SELECT DISTINCT department FROM employees;

-- DISTINCT with multiple columns (combination must be unique)
SELECT DISTINCT department, location FROM employees;
```

#### UPDATE Operations:
```sql
-- Update single row by primary key
UPDATE employees 
SET salary = 80000.00 
WHERE id = 1;                              // Always use WHERE to avoid updating all rows!

-- Update multiple columns at once
UPDATE employees 
SET 
    salary = salary * 1.1,                 // Increase salary by 10%
    email = 'new.email@company.com',
    updated_at = CURRENT_TIMESTAMP
WHERE id = 1;

-- Update using calculated values
UPDATE employees 
SET salary = salary * 1.05                 // 5% raise
WHERE department = 'Engineering' AND salary < 100000;

-- Update with subquery (set salary to department average)
UPDATE employees e1
SET salary = (
    SELECT AVG(salary) 
    FROM employees e2 
    WHERE e2.department = e1.department
)
WHERE salary < 50000;

-- Update with FROM clause (join with another table)
UPDATE employees e
SET department = d.new_dept_name
FROM department_changes d
WHERE e.department = d.old_dept_name;

-- Update with RETURNING clause (see what was changed)
UPDATE employees 
SET salary = salary * 1.05 
WHERE department = 'Engineering'
RETURNING id, first_name, last_name, salary;    // Returns affected rows

-- Update all rows (dangerous - use with caution!)
UPDATE employees SET last_updated = CURRENT_TIMESTAMP;
```

#### DELETE Operations:
```sql
-- Delete specific row by primary key
DELETE FROM employees WHERE id = 1;

-- Delete with condition
DELETE FROM employees WHERE hire_date < '2020-01-01';

-- Delete with multiple conditions
DELETE FROM employees 
WHERE salary < 40000 AND hire_date < '2021-01-01';

-- Delete with subquery
DELETE FROM employees 
WHERE id IN (
    SELECT emp_id 
    FROM employee_departments 
    WHERE dept_id = 5
);

-- Delete with RETURNING clause (see what was deleted)
DELETE FROM employees 
WHERE salary < 40000
RETURNING id, first_name, last_name, salary;

-- Delete with EXISTS subquery
DELETE FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM employee_departments ed
    WHERE ed.emp_id = e.id
);

-- TRUNCATE TABLE - faster than DELETE for removing all rows
// Removes all rows, resets SERIAL sequences, cannot be rolled back in some cases
TRUNCATE TABLE employees;

-- TRUNCATE with CASCADE (also truncates dependent tables)
TRUNCATE TABLE employees CASCADE;

-- TRUNCATE with RESTART IDENTITY (reset auto-increment counters)
TRUNCATE TABLE employees RESTART IDENTITY;
```

---

### 4. Data Types

#### Numeric Types:
```sql
CREATE TABLE numeric_examples (
    -- Integer types (exact numbers)
    small_num SMALLINT,              // -32,768 to +32,767 (2 bytes)
    regular_num INTEGER,             // -2,147,483,648 to +2,147,483,647 (4 bytes)
    big_num BIGINT,                  // Very large integers (8 bytes)
    
    -- Auto-incrementing integers (use for IDs)
    id SERIAL,                       // Auto-incrementing INTEGER
    big_id BIGSERIAL,                // Auto-incrementing BIGINT
    small_id SMALLSERIAL,            // Auto-incrementing SMALLINT
    
    -- Exact decimal numbers (for money, measurements requiring precision)
    price NUMERIC(10, 2),            // 10 total digits, 2 after decimal point
    precise_value DECIMAL(15, 5),    // DECIMAL is alias for NUMERIC
    
    -- Floating-point numbers (approximate, for scientific calculations)
    measurement REAL,                // 6 decimal digits precision (4 bytes)
    scientific_value DOUBLE PRECISION // 15 decimal digits precision (8 bytes)
);

-- Examples of numeric operations
INSERT INTO numeric_examples (small_num, regular_num, big_num, price, measurement)
VALUES (100, 50000, 9999999999, 19.99, 3.14159);

-- Mathematical operations
SELECT 
    price * 1.1 AS price_with_tax,          // Multiplication
    price / 2 AS half_price,                 // Division
    price + 5 AS price_plus_shipping,        // Addition
    MOD(regular_num, 10) AS last_digit,      // Modulo
    POWER(small_num, 2) AS squared,          // Exponentiation
    SQRT(regular_num) AS square_root,        // Square root
    ROUND(price, 1) AS rounded_price,        // Rounding
    CEIL(price) AS ceiling,                  // Round up
    FLOOR(price) AS floor_value              // Round down
FROM numeric_examples;
```

#### Character Types:
```sql
CREATE TABLE string_examples (
    -- Variable-length with limit (most common, efficient)
    name VARCHAR(100),               // Up to 100 characters, stores only actual length
    
    -- Fixed-length, blank-padded (use for fixed codes)
    country_code CHAR(2),            // Always 2 characters, padded with spaces
    product_code CHAR(10),
    
    -- Unlimited variable-length (for long text)
    description TEXT,                // No length limit, good for articles, comments
    
    -- Case-insensitive text (requires extension)
    email CITEXT                     // Automatically handles case-insensitive comparisons
);

-- Enable citext extension for case-insensitive text
CREATE EXTENSION IF NOT EXISTS citext;

-- String operations
INSERT INTO string_examples (name, country_code, description, email)
VALUES ('John Doe', 'US', 'This is a long description text...', 'JOHN@EXAMPLE.COM');

-- String functions
SELECT 
    LENGTH(name) AS name_length,                      // Character count
    UPPER(name) AS uppercase,                          // Convert to uppercase
    LOWER(name) AS lowercase,                          // Convert to lowercase
    SUBSTRING(name FROM 1 FOR 4) AS first_four,        // Extract substring
    POSITION('Doe' IN name) AS position,               // Find substring position
    CONCAT(name, ' - ', country_code) AS combined,     // Concatenate strings
    name || ' (' || country_code || ')' AS concat_op,  // Concatenation operator
    TRIM(name) AS trimmed,                             // Remove leading/trailing spaces
    REPLACE(name, 'John', 'Jane') AS replaced,         // Replace substring
    SPLIT_PART(name, ' ', 1) AS first_word             // Split and get part
FROM string_examples;

-- Case-insensitive query with CITEXT
SELECT * FROM string_examples WHERE email = 'john@example.com';  // Matches 'JOHN@EXAMPLE.COM'
```

#### Date/Time Types:
```sql
CREATE TABLE datetime_examples (
    -- Date only (no time component)
    birth_date DATE,                 // Format: YYYY-MM-DD
    
    -- Time only (no date component)
    meeting_time TIME,               // Format: HH:MI:SS
    meeting_time_tz TIME WITH TIME ZONE,  // Time with timezone
    
    -- Date and time without timezone
    created_at TIMESTAMP,            // Format: YYYY-MM-DD HH:MI:SS
    
    -- Date and time with timezone (RECOMMENDED for most use cases)
    updated_at TIMESTAMPTZ,          // Stores in UTC, converts to session timezone
    
    -- Time interval (duration)
    duration INTERVAL                // Represents time spans like '2 hours', '3 days'
);

-- Insert datetime values
INSERT INTO datetime_examples (
    birth_date, 
    meeting_time, 
    created_at, 
    updated_at, 
    duration
)
VALUES (
    '1990-05-15',
    '14:30:00',
    '2024-01-15 10:30:00',
    '2024-01-15 10:30:00-08',       // -08 is timezone offset
    '2 hours 30 minutes'
);

-- Date/Time functions and operations
SELECT 
    CURRENT_DATE,                                      // Today's date
    CURRENT_TIME,                                      // Current time with timezone
    CURRENT_TIMESTAMP,                                 // Current date and time
    NOW(),                                             // Same as CURRENT_TIMESTAMP
    TIMEOFDAY(),                                       // Current date/time as string
    
    -- Extract parts of date/time
    EXTRACT(YEAR FROM birth_date) AS birth_year,
    EXTRACT(MONTH FROM birth_date) AS birth_month,
    EXTRACT(DAY FROM birth_date) AS birth_day,
    EXTRACT(DOW FROM birth_date) AS day_of_week,      // 0=Sunday, 6=Saturday
    
    -- Date arithmetic
    birth_date + INTERVAL '1 year' AS next_birthday,
    birth_date - INTERVAL '6 months' AS six_months_before,
    NOW() - created_at AS time_since_creation,
    
    -- Age calculation
    AGE(CURRENT_DATE, birth_date) AS age,
    DATE_PART('year', AGE(CURRENT_DATE, birth_date)) AS years_old,
    
    -- Date formatting
    TO_CHAR(created_at, 'YYYY-MM-DD') AS formatted_date,
    TO_CHAR(created_at, 'Mon DD, YYYY') AS readable_date,
    TO_CHAR(created_at, 'HH24:MI:SS') AS time_24h,
    
    -- Date truncation (round down to unit)
    DATE_TRUNC('month', created_at) AS month_start,
    DATE_TRUNC('year', created_at) AS year_start
FROM datetime_examples;
```

#### Boolean Type:
```sql
CREATE TABLE boolean_examples (
    id SERIAL PRIMARY KEY,
    is_active BOOLEAN DEFAULT TRUE,         // Default value is TRUE
    is_verified BOOLEAN,                    // Can be TRUE, FALSE, or NULL
    has_access BOOLEAN NOT NULL DEFAULT FALSE
);

-- Insert boolean values (multiple formats accepted)
INSERT INTO boolean_examples (is_active, is_verified)
VALUES (TRUE, FALSE);                      // Standard SQL boolean literals

INSERT INTO boolean_examples (is_active, is_verified)
VALUES ('yes', 'no');                      // 'yes'/'no' converted to boolean

INSERT INTO boolean_examples (is_active, is_verified)
VALUES ('1', '0');                         // '1'/'0' also work

INSERT INTO boolean_examples (is_active, is_verified)
VALUES ('t', 'f');                         // 't'/'f' single character format

-- Boolean queries
SELECT * FROM boolean_examples WHERE is_active = TRUE;
SELECT * FROM boolean_examples WHERE is_active;           // Same as = TRUE
SELECT * FROM boolean_examples WHERE NOT is_verified;     // Same as = FALSE
SELECT * FROM boolean_examples WHERE is_verified IS NULL;

-- Boolean operations
SELECT 
    is_active AND is_verified AS both_true,    // Logical AND
    is_active OR is_verified AS at_least_one,   // Logical OR
    NOT is_active AS negated,                   // Logical NOT
    is_active = is_verified AS are_equal
FROM boolean_examples;
```

#### Array Types:
```sql
CREATE TABLE array_examples (
    id SERIAL PRIMARY KEY,
    tags TEXT[],                     // One-dimensional text array
    numbers INTEGER[],               // One-dimensional integer array
    matrix INTEGER[][],              // Multi-dimensional array (2D in this case)
    bounded_array INTEGER[5]         // Array with size constraint (not enforced)
);

-- Insert arrays using ARRAY constructor
INSERT INTO array_examples (tags, numbers, matrix)
VALUES (
    ARRAY['postgresql', 'database', 'tutorial'],
    ARRAY[1, 2, 3, 4, 5],
    ARRAY[[1,2,3], [4,5,6]]         // 2D array
);

-- Insert using curly brace syntax
INSERT INTO array_examples (tags, numbers)
VALUES (
    '{"tag1", "tag2", "tag3"}',     // Alternative array literal syntax
    '{10, 20, 30, 40}'
);

-- Array queries
// ANY: check if value exists in array
SELECT * FROM array_examples WHERE 'postgresql' = ANY(tags);

// @>: contains operator (left array contains right array)
SELECT * FROM array_examples WHERE tags @> ARRAY['database'];

// <@: is contained by
SELECT * FROM array_examples WHERE ARRAY['database'] <@ tags;

// &&: overlap operator (arrays have common elements)
SELECT * FROM array_examples WHERE tags && ARRAY['tutorial', 'guide'];

-- Array indexing (1-based in PostgreSQL!)
SELECT 
    tags[1] AS first_tag,            // Access first element
    tags[2:3] AS second_and_third,   // Slice array
    matrix[1][2] AS element_1_2      // 2D array access
FROM array_examples;

-- Array functions
SELECT 
    array_length(tags, 1) AS tag_count,             // Length of dimension 1
    array_append(tags, 'new_tag') AS with_new,      // Add element to end
    array_prepend('first', tags) AS with_first,     // Add element to start
    array_remove(tags, 'database') AS filtered,     // Remove all occurrences
    array_cat(tags, ARRAY['extra']) AS concatenated, // Concatenate arrays
    array_position(tags, 'database') AS db_position, // Find element position
    unnest(tags) AS individual_tag                   // Expand array to rows
FROM array_examples;

-- Aggregate arrays from rows
SELECT 
    array_agg(first_name ORDER BY first_name) AS all_names
FROM employees;
```

#### JSON and JSONB Types:
```sql
CREATE TABLE json_examples (
    id SERIAL PRIMARY KEY,
    data JSON,                       // JSON: text storage, preserves formatting and order
    metadata JSONB                   // JSONB: binary storage, faster queries, supports indexing
);

-- Insert JSON data
INSERT INTO json_examples (data, metadata)
VALUES (
    '{"name": "John", "age": 30, "city": "NYC"}',
    '{"tags": ["developer", "postgresql"], "score": 95, "active": true}'
);

-- Insert with json_build_object function
INSERT INTO json_examples (data, metadata)
VALUES (
    json_build_object('name', 'Jane', 'age', 25),
    jsonb_build_object('tags', ARRAY['designer'], 'score', 88)
);

-- JSON query operators
SELECT 
    data->>'name' AS name_text,              // ->> returns text (extracts as text)
    data->'age' AS age_json,                  // -> returns JSON (keeps as JSON)
    (data->>'age')::INTEGER AS age_int,       // Cast extracted text to integer
    
    metadata->'tags'->0 AS first_tag,         // Navigate nested JSON (get first array element)
    metadata->'tags'->>1 AS second_tag_text,  // Get second tag as text
    
    metadata @> '{"score": 95}' AS has_score_95,        // @> contains operator
    metadata ? 'tags' AS has_tags_key,                  // ? key exists operator
    metadata ?| ARRAY['tags', 'rating'] AS has_any,     // ?| any key exists
    metadata ?& ARRAY['tags', 'score'] AS has_all       // ?& all keys exist
FROM json_examples;

-- JSON functions
SELECT 
    jsonb_array_length(metadata->'tags') AS tag_count,          // Array length
    jsonb_object_keys(metadata) AS keys,                         // All object keys
    jsonb_each(metadata) AS key_value_pairs,                     // Expand to key-value rows
    jsonb_array_elements(metadata->'tags') AS tag_element,       // Expand array to rows
    jsonb_array_elements_text(metadata->'tags') AS tag_text,     // Expand array to text rows
    
    -- Build JSON
    jsonb_build_array('a', 'b', 'c') AS json_array,
    jsonb_build_object('key', 'value', 'num', 123) AS json_object,
    
    -- Modify JSON
    jsonb_set(metadata, '{score}', '100') AS updated_score,      // Update value
    metadata || '{"new_field": "value"}' AS merged,               // Merge objects
    metadata - 'score' AS without_score,                          // Remove key
    metadata #- '{tags,0}' AS remove_first_tag                    // Remove by path
FROM json_examples;

-- Create GIN index on JSONB for fast queries
CREATE INDEX idx_metadata_gin ON json_examples USING GIN (metadata);

-- Efficient query using GIN index
SELECT * FROM json_examples 
WHERE metadata @> '{"tags": ["postgresql"]}';

-- Path-based queries
SELECT * FROM json_examples 
WHERE metadata @@ '$.score > 90';            // JSONPath query
```

#### UUID Type:
```sql
-- Enable UUID extension for generation functions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE uuid_examples (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),    // Auto-generate UUID v4
    user_id UUID NOT NULL,
    name VARCHAR(100),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Insert with auto-generated UUID
INSERT INTO uuid_examples (user_id, name) 
VALUES (uuid_generate_v4(), 'Test Record');

-- Insert with specific UUID (useful for testing, migrations)
INSERT INTO uuid_examples (id, user_id, name) 
VALUES (
    'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',
    '550e8400-e29b-41d4-a716-446655440000',
    'Custom UUID'
);

-- UUID generation functions
SELECT 
    uuid_generate_v1() AS uuid_v1,          // Time-based UUID (includes MAC address)
    uuid_generate_v1mc() AS uuid_v1mc,      // Time-based with random MAC
    uuid_generate_v4() AS uuid_v4,          // Random UUID (most common)
    uuid_nil() AS nil_uuid;                 // All zeros UUID

-- Advantages of UUID:
// 1. Globally unique across tables and databases
// 2. No central coordination needed
// 3. Hard to guess/enumerate
// 4. Can be generated client-side
// Disadvantages:
// 1. Larger storage (16 bytes vs 4/8 for integers)
// 2. Slower indexing than sequential integers
// 3. Less human-readable
```

#### Other Useful Types:
```sql
CREATE TABLE other_types_examples (
    -- Network address types
    ip_address INET,                 // IPv4 or IPv6 address with subnet
    ip_only CIDR,                    // IPv4 or IPv6 network
    mac_address MACADDR,             // MAC address
    
    -- Geometric types
    point_location POINT,            // Point in 2D space (x, y)
    line_segment LINE,               // Infinite line
    box_area BOX,                    // Rectangle
    circle_region CIRCLE,            // Circle (center point and radius)
    
    -- Range types
    int_range INT4RANGE,             // Range of integers
    date_range DATERANGE,            // Range of dates
    timestamp_range TSTZRANGE,       // Range of timestamps
    
    -- Money type
    price MONEY,                     // Currency amount (locale-specific)
    
    -- Bit string types
    bits BIT(8),                     // Fixed-length bit string
    varying_bits VARBIT              // Variable-length bit string
);

-- Examples of using special types
INSERT INTO other_types_examples (
    ip_address,
    point_location,
    date_range,
    price
) VALUES (
    '192.168.1.1/24',
    POINT(10.5, 20.3),
    '[2024-01-01, 2024-12-31]',     // Inclusive range
    '$19.99'
);

-- Query examples
SELECT 
    ip_address << '192.168.0.0/16' AS in_subnet,     // Check if IP in subnet
    point_location <-> POINT(0, 0) AS distance,      // Distance between points
    date_range @> '2024-06-15'::DATE AS contains_date, // Range contains value
    upper(date_range) AS range_end                    // Upper bound of range
FROM other_types_examples;
```

---

### 5. Constraints

#### PRIMARY KEY Constraint:
```sql
-- Single column primary key (most common)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,           // Automatically creates unique index
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL
);

-- Composite primary key (multiple columns together form unique identifier)
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER NOT NULL,
    price NUMERIC(10, 2),
    PRIMARY KEY (order_id, product_id)    // Both order_id AND product_id together must be unique
);

-- Named primary key constraint (useful for later reference)
CREATE TABLE products (
    id SERIAL,
    sku VARCHAR(50) NOT NULL,
    name VARCHAR(100),
    CONSTRAINT pk_products PRIMARY KEY (id)
);

-- Add primary key to existing table
ALTER TABLE existing_table ADD PRIMARY KEY (id);

-- Add named primary key
ALTER TABLE existing_table 
ADD CONSTRAINT pk_existing_table PRIMARY KEY (id);

-- Drop primary key
ALTER TABLE products DROP CONSTRAINT pk_products;

-- Primary key characteristics:
// 1. NOT NULL automatically applied
// 2. UNIQUE automatically enforced
// 3. Creates index automatically
// 4. Only one primary key per table
// 5. Used as default target for foreign keys
```

#### FOREIGN KEY Constraint:
```sql
-- Basic foreign key (references primary key of another table)
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),    // Links to users table
    order_date DATE DEFAULT CURRENT_DATE,
    total NUMERIC(10, 2)
);

-- Foreign key with referential integrity actions
CREATE TABLE order_details (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id) 
        ON DELETE CASCADE                    // Delete details when order deleted
        ON UPDATE CASCADE,                   // Update details when order id changes
    product_id INTEGER REFERENCES products(id) 
        ON DELETE RESTRICT,                  // Prevent deletion if referenced
    quantity INTEGER NOT NULL,
    unit_price NUMERIC(10, 2)
);

-- Referential actions explained:
// CASCADE: Automatically delete/update dependent rows
// RESTRICT: Prevent delete/update if dependent rows exist (default)
// SET NULL: Set foreign key to NULL when parent deleted
// SET DEFAULT: Set foreign key to DEFAULT value
// NO ACTION: Similar to RESTRICT, but check can be deferred

-- Example of SET NULL and SET DEFAULT
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    manager_id INTEGER REFERENCES employees(id) 
        ON DELETE SET NULL,                  // If manager deleted, set to NULL
    department_id INTEGER DEFAULT 1 REFERENCES departments(id)
        ON DELETE SET DEFAULT                // If dept deleted, set to default dept
);

-- Named foreign key constraints
CREATE TABLE reviews (
    id SERIAL PRIMARY KEY,
    product_id INTEGER,
    user_id INTEGER,
    rating INTEGER CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    CONSTRAINT fk_review_product FOREIGN KEY (product_id) 
        REFERENCES products(id) ON DELETE CASCADE,
    CONSTRAINT fk_review_user FOREIGN KEY (user_id) 
        REFERENCES users(id) ON DELETE CASCADE
);

-- Add foreign key to existing table
ALTER TABLE existing_table 
ADD CONSTRAINT fk_user_id 
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

-- Drop foreign key
ALTER TABLE reviews DROP CONSTRAINT fk_review_product;

-- Check foreign key constraints
SELECT 
    tc.table_name, 
    kcu.column_name,
    ccu.table_name AS foreign_table_name,
    ccu.column_name AS foreign_column_name 
FROM information_schema.table_constraints AS tc 
JOIN information_schema.key_column_usage AS kcu
    ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage AS ccu
    ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY';
```

#### UNIQUE Constraint:
```sql
-- Single column unique constraint
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE,               // No duplicate emails allowed
    username VARCHAR(50) UNIQUE
);

-- Composite unique constraint (combination must be unique)
CREATE TABLE enrollments (
    id SERIAL PRIMARY KEY,
    student_id INTEGER,
    course_id INTEGER,
    semester VARCHAR(20),
    UNIQUE (student_id, course_id, semester)  // Same student can't enroll in same course twice per semester
);

-- Named unique constraint
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50),
    upc VARCHAR(12),
    CONSTRAINT uq_product_sku UNIQUE (sku),
    CONSTRAINT uq_product_upc UNIQUE (upc)
);

-- Add unique constraint to existing table
ALTER TABLE existing_table ADD UNIQUE (email);

ALTER TABLE existing_table 
ADD CONSTRAINT uq_email UNIQUE (email);

-- Drop unique constraint
ALTER TABLE products DROP CONSTRAINT uq_product_sku;

-- Partial unique index (unique only where condition is true)
// Useful when you want NULL values but unique non-NULL values
CREATE UNIQUE INDEX idx_unique_active_email 
ON users (email) 
WHERE is_active = TRUE;                      // Email unique only for active users

-- Case-insensitive unique constraint
CREATE UNIQUE INDEX idx_unique_lower_username 
ON users (LOWER(username));                  // Username unique regardless of case

-- NULL handling in UNIQUE constraints:
// - Multiple NULL values are allowed (NULL != NULL in SQL)
// - Only non-NULL values must be unique
// - Use partial index to enforce "unique or NULL" semantics
```

#### CHECK Constraint:
```sql
-- Simple CHECK constraint (validates single column)
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2) CHECK (price > 0),                    // Price must be positive
    discount_percent INTEGER CHECK (discount_percent BETWEEN 0 AND 100),
    stock_quantity INTEGER CHECK (stock_quantity >= 0)
);

-- Multi-column CHECK constraint (validates relationship between columns)
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    birth_date DATE NOT NULL,
    hire_date DATE NOT NULL,
    termination_date DATE,
    salary NUMERIC(10, 2) NOT NULL,
    CHECK (hire_date > birth_date),                           // Can't hire before birth
    CHECK (termination_date IS NULL OR termination_date >= hire_date), // Can't terminate before hire
    CHECK (salary > 0)
);

-- Named CHECK constraints (recommended for easier maintenance)
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    order_date DATE NOT NULL,
    ship_date DATE,
    delivery_date DATE,
    status VARCHAR(20) NOT NULL,
    total_amount NUMERIC(12, 2),
    
    CONSTRAINT chk_ship_after_order 
        CHECK (ship_date IS NULL OR ship_date >= order_date),
    
    CONSTRAINT chk_delivery_after_ship 
        CHECK (delivery_date IS NULL OR delivery_date >= ship_date),
    
    CONSTRAINT chk_valid_status 
        CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    
    CONSTRAINT chk_positive_amount 
        CHECK (total_amount >= 0)
);

-- Complex CHECK constraints with logic
CREATE TABLE bank_accounts (
    id SERIAL PRIMARY KEY,
    account_type VARCHAR(20) NOT NULL,
    balance NUMERIC(15, 2) NOT NULL,
    minimum_balance NUMERIC(15, 2) DEFAULT 0,
    overdraft_limit NUMERIC(15, 2) DEFAULT 0,
    
    CONSTRAINT chk_account_type 
        CHECK (account_type IN ('savings', 'checking', 'business')),
    
    -- Different rules for different account types
    CONSTRAINT chk_balance_rules CHECK (
        (account_type = 'savings' AND balance >= minimum_balance) OR
        (account_type = 'checking' AND balance >= -overdraft_limit) OR
        (account_type = 'business')
    )
);

-- CHECK constraint with expressions
CREATE TABLE time_tracking (
    id SERIAL PRIMARY KEY,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    break_minutes INTEGER DEFAULT 0,
    
    CONSTRAINT chk_end_after_start 
        CHECK (end_time > start_time),
    
    CONSTRAINT chk_reasonable_break 
        CHECK (break_minutes >= 0 AND break_minutes <= EXTRACT(EPOCH FROM (end_time - start_time))/60)
);

-- Add CHECK constraint to existing table
ALTER TABLE products 
ADD CONSTRAINT chk_price_positive CHECK (price > 0);

-- Drop CHECK constraint
ALTER TABLE products DROP CONSTRAINT chk_price_positive;

-- Disable/Enable constraint (useful for bulk loading)
ALTER TABLE products DISABLE TRIGGER ALL;  // Disables constraints temporarily
-- Bulk load data
ALTER TABLE products ENABLE TRIGGER ALL;

-- CHECK constraint limitations:
// 1. Cannot reference other tables (use triggers for that)
// 2. Cannot use subqueries
// 3. Cannot use non-immutable functions (like CURRENT_DATE in some cases)
```

#### NOT NULL Constraint:
```sql
-- Define NOT NULL during table creation
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,         // Required field
    last_name VARCHAR(50) NOT NULL,          // Required field
    email VARCHAR(100) NOT NULL,             // Required field
    phone VARCHAR(20),                       // Optional field (allows NULL)
    address TEXT
);

-- NOT NULL with DEFAULT value
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date DATE NOT NULL DEFAULT CURRENT_DATE,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    notes TEXT                               // Optional
);

-- Add NOT NULL to existing column (ensure no NULL values exist first!)
// First update NULL values
UPDATE customers SET phone = 'Unknown' WHERE phone IS NULL;
// Then add constraint
ALTER TABLE customers ALTER COLUMN phone SET NOT NULL;

-- Remove NOT NULL constraint
ALTER TABLE customers ALTER COLUMN phone DROP NOT NULL;

-- NOT NULL on multiple columns
CREATE TABLE bookings (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    room_id INTEGER NOT NULL,
    check_in DATE NOT NULL,
    check_out DATE NOT NULL,
    num_guests INTEGER NOT NULL DEFAULT 1
);

-- Difference between NULL and empty string:
INSERT INTO customers (first_name, last_name, email, phone)
VALUES ('John', 'Doe', 'john@example.com', NULL);    // phone is NULL (unknown)

INSERT INTO customers (first_name, last_name, email, phone)
VALUES ('Jane', 'Smith', 'jane@example.com', '');    // phone is empty string (known to be empty)

-- Query differences
SELECT * FROM customers WHERE phone IS NULL;         // Finds NULL values
SELECT * FROM customers WHERE phone = '';            // Finds empty strings
SELECT * FROM customers WHERE phone IS NULL OR phone = '';  // Finds both
```

#### DEFAULT Constraint:
```sql
-- Various DEFAULT examples
CREATE TABLE log_entries (
    id SERIAL PRIMARY KEY,
    message TEXT NOT NULL,
    severity VARCHAR(20) DEFAULT 'INFO',              // Static default value
    created_at TIMESTAMPTZ DEFAULT NOW(),             // Function-based default
    created_by VARCHAR(50) DEFAULT CURRENT_USER,      // System function
    is_processed BOOLEAN DEFAULT FALSE,
    retry_count INTEGER DEFAULT 0,
    processed_at TIMESTAMPTZ,                         // NULL by default (no DEFAULT clause)
    metadata JSONB DEFAULT '{}'::JSONB                // Default empty JSON object
);

-- Default with expressions
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    action VARCHAR(20) NOT NULL,
    old_value JSONB,
    new_value JSONB,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    changed_by VARCHAR(50) DEFAULT CURRENT_USER,
    session_id VARCHAR(100) DEFAULT 'unknown'
);

-- Insert using default values
INSERT INTO log_entries (message) 
VALUES ('Application started');                      // Other columns use DEFAULT values

INSERT INTO log_entries (message, severity) 
VALUES ('Error occurred', 'ERROR');                  // Override specific DEFAULT

INSERT INTO log_entries DEFAULT VALUES;              // Use DEFAULT for all columns (fails if NOT NULL without DEFAULT)

-- Explicitly request DEFAULT value
INSERT INTO log_entries (message, created_at) 
VALUES ('Test message', DEFAULT);                    // Use DEFAULT for created_at

-- Add DEFAULT to existing column
ALTER TABLE log_entries 
ALTER COLUMN severity SET DEFAULT 'WARNING';

-- Change DEFAULT value
ALTER TABLE log_entries 
ALTER COLUMN severity SET DEFAULT 'DEBUG';

-- Remove DEFAULT constraint
ALTER TABLE log_entries 
ALTER COLUMN severity DROP DEFAULT;

-- DEFAULT vs NOT NULL:
// - DEFAULT provides value when INSERT omits column
// - NOT NULL prevents NULL values
// - Common pattern: NOT NULL + DEFAULT ensures column always has a value

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,                   // Required, no default
    email VARCHAR(100) NOT NULL,                     // Required, no default
    is_active BOOLEAN NOT NULL DEFAULT TRUE,         // Required with default
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),   // Required with default
    bio TEXT                                         // Optional, defaults to NULL
);
```

#### EXCLUSION Constraint:
```sql
-- Enable btree_gist extension for exclusion constraints
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Prevent overlapping date ranges (e.g., room bookings)
CREATE TABLE room_bookings (
    id SERIAL PRIMARY KEY,
    room_id INTEGER NOT NULL,
    guest_name VARCHAR(100) NOT NULL,
    check_in DATE NOT NULL,
    check_out DATE NOT NULL,
    
    -- Exclusion constraint: same room can't have overlapping bookings
    EXCLUDE USING GIST (
        room_id WITH =,                              // Same room AND
        daterange(check_in, check_out, '[]') WITH && // Overlapping date ranges
    )
);

-- This succeeds (no overlap)
INSERT INTO room_bookings (room_id, guest_name, check_in, check_out)
VALUES (101, 'John Doe', '2024-01-01', '2024-01-05');

INSERT INTO room_bookings (room_id, guest_name, check_in, check_out)
VALUES (101, 'Jane Smith', '2024-01-06', '2024-01-10');  // OK: starts after first ends

-- This fails (overlap detected)
-- INSERT INTO room_bookings (room_id, guest_name, check_in, check_out)
-- VALUES (101, 'Bob Wilson', '2024-01-03', '2024-01-07');  // ERROR: overlaps with first

-- Employee scheduling (prevent double-booking employees)
CREATE TABLE employee_schedule (
    id SERIAL PRIMARY KEY,
    employee_id INTEGER NOT NULL,
    shift_start TIMESTAMP NOT NULL,
    shift_end TIMESTAMP NOT NULL,
    
    EXCLUDE USING GIST (
        employee_id WITH =,
        tstzrange(shift_start, shift_end) WITH &&
    )
);

-- Meeting room reservations with overlap prevention
CREATE TABLE meeting_rooms (
    id SERIAL PRIMARY KEY,
    room_number INTEGER NOT NULL,
    meeting_start TIMESTAMP NOT NULL,
    meeting_end TIMESTAMP NOT NULL,
    organizer VARCHAR(100),
    
    -- Named exclusion constraint
    CONSTRAINT no_room_overlap EXCLUDE USING GIST (
        room_number WITH =,
        tstzrange(meeting_start, meeting_end) WITH &&
    )
);

-- Exclusion with multiple conditions
CREATE TABLE parking_spots (
    id SERIAL PRIMARY KEY,
    spot_number INTEGER NOT NULL,
    license_plate VARCHAR(20),
    reserved_from TIMESTAMP NOT NULL,
    reserved_until TIMESTAMP NOT NULL,
    
    -- Same spot can't be reserved by different cars at same time
    EXCLUDE USING GIST (
        spot_number WITH =,
        tstzrange(reserved_from, reserved_until) WITH &&
    ) WHERE (license_plate IS NOT NULL)              // Only when license plate specified
);

-- View exclusion constraints
SELECT 
    conname AS constraint_name,
    conrelid::regclass AS table_name
FROM pg_constraint 
WHERE contype = 'x';  // 'x' = exclusion constraint
```

---

### 6. Basic Queries

#### SELECT with WHERE Clause:
```sql
-- Comparison operators
SELECT * FROM employees WHERE salary > 70000;        // Greater than
SELECT * FROM employees WHERE salary >= 70000;       // Greater than or equal
SELECT * FROM employees WHERE salary < 70000;        // Less than
SELECT * FROM employees WHERE salary <= 70000;       // Less than or equal
SELECT * FROM employees WHERE salary = 70000;        // Equal
SELECT * FROM employees WHERE salary != 70000;       // Not equal
SELECT * FROM employees WHERE salary <> 70000;       // Not equal (SQL standard)

-- Logical operators (AND, OR, NOT)
SELECT * FROM employees 
WHERE salary > 60000 AND department = 'Engineering';  // Both conditions must be true

SELECT * FROM employees 
WHERE department = 'Sales' OR department = 'Marketing';  // At least one condition must be true

SELECT * FROM employees 
WHERE NOT (salary < 50000);                          // Negates condition

-- Complex logical combinations
SELECT * FROM employees 
WHERE (department = 'Engineering' AND salary > 80000) 
   OR (department = 'Sales' AND salary > 70000);

-- IN operator (value matches any in list)
SELECT * FROM employees 
WHERE department IN ('Engineering', 'Sales', 'Marketing');

SELECT * FROM employees 
WHERE id IN (1, 2, 3, 5, 8, 13);                     // Fibonacci IDs

-- NOT IN operator
SELECT * FROM employees 
WHERE department NOT IN ('HR', 'Admin');

-- BETWEEN operator (inclusive range)
SELECT * FROM employees 
WHERE salary BETWEEN 50000 AND 80000;                // Includes 50000 and 80000

SELECT * FROM employees 
WHERE hire_date BETWEEN '2023-01-01' AND '2023-12-31';

-- NOT BETWEEN
SELECT * FROM employees 
WHERE salary NOT BETWEEN 40000 AND 60000;

-- LIKE operator (pattern matching)
// % matches zero or more characters
// _ matches exactly one character
SELECT * FROM employees WHERE first_name LIKE 'J%';       // Starts with 'J'
SELECT * FROM employees WHERE last_name LIKE '%son';      // Ends with 'son'
SELECT * FROM employees WHERE email LIKE '%@gmail.com';   // Contains '@gmail.com'
SELECT * FROM employees WHERE first_name LIKE '_ohn';     // Second char 'o', ends with 'hn'
SELECT * FROM employees WHERE first_name LIKE 'J_n%';     // Starts with 'J', third char 'n'

-- ILIKE operator (case-insensitive LIKE, PostgreSQL-specific)
SELECT * FROM employees WHERE first_name ILIKE 'john%';   // Matches 'John', 'JOHN', 'john'

-- NOT LIKE
SELECT * FROM employees WHERE email NOT LIKE '%@company.com';

-- IS NULL and IS NOT NULL (cannot use = or != with NULL!)
SELECT * FROM employees WHERE middle_name IS NULL;
SELECT * FROM employees WHERE middle_name IS NOT NULL;
SELECT * FROM employees WHERE bonus IS NULL OR bonus = 0;

-- Regular expressions (PostgreSQL-specific)
SELECT * FROM employees WHERE email ~ '^[a-z]+@[a-z]+\.(com|org)$';  // ~ is regex match
SELECT * FROM employees WHERE email ~* 'GMAIL';                      // ~* is case-insensitive
SELECT * FROM employees WHERE email !~ 'hotmail';                    // !~ is negated match

-- Combining multiple conditions
SELECT * FROM employees 
WHERE department IN ('Engineering', 'Sales') 
  AND salary BETWEEN 60000 AND 100000
  AND hire_date >= '2022-01-01'
  AND email LIKE '%@company.com'
  AND manager_id IS NOT NULL;
```

#### Aggregate Functions:
```sql
-- COUNT: count rows or non-NULL values
SELECT COUNT(*) FROM employees;                      // Count all rows (includes NULLs)
SELECT COUNT(middle_name) FROM employees;            // Count non-NULL middle names
SELECT COUNT(DISTINCT department) FROM employees;    // Count unique departments
SELECT COUNT(DISTINCT department, location) FROM employees;  // Count unique combinations

-- SUM: sum of numeric values (ignores NULL)
SELECT SUM(salary) AS total_payroll FROM employees;
SELECT SUM(bonus) FROM employees;                    // NULLs ignored in sum

-- AVG: average value (ignores NULL)
SELECT AVG(salary) AS average_salary FROM employees;
SELECT ROUND(AVG(salary), 2) AS avg_salary_rounded FROM employees;

-- MIN and MAX: minimum and maximum values
SELECT MIN(salary) AS min_salary FROM employees;
SELECT MAX(salary) AS max_salary FROM employees;
SELECT MIN(hire_date) AS first_hire, MAX(hire_date) AS latest_hire FROM employees;

-- Multiple aggregates in one query
SELECT 
    COUNT(*) AS total_employees,
    COUNT(DISTINCT department) AS num_departments,
    SUM(salary) AS total_payroll,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    STDDEV(salary) AS salary_stddev,                 // Standard deviation
    VARIANCE(salary) AS salary_variance              // Variance
FROM employees;

-- STRING_AGG: concatenate strings with separator
SELECT STRING_AGG(first_name, ', ' ORDER BY first_name) AS all_names 
FROM employees;

SELECT 
    department,
    STRING_AGG(first_name || ' ' || last_name, '; ' ORDER BY hire_date) AS employees_list
FROM employees
GROUP BY department;

-- ARRAY_AGG: aggregate values into array
SELECT ARRAY_AGG(first_name ORDER BY first_name) AS name_array 
FROM employees;

SELECT 
    department,
    ARRAY_AGG(salary ORDER BY salary DESC) AS salary_list
FROM employees
GROUP BY department;

-- JSON_AGG: aggregate rows into JSON array
SELECT JSON_AGG(
    JSON_BUILD_OBJECT(
        'name', first_name || ' ' || last_name,
        'salary', salary,
        'department', department
    ) ORDER BY salary DESC
) AS employees_json
FROM employees;

-- JSONB_OBJECT_AGG: create JSON object from key-value pairs
SELECT JSONB_OBJECT_AGG(
    id, 
    first_name || ' ' || last_name
) AS employees_map
FROM employees;

-- Boolean aggregates
SELECT 
    BOOL_AND(is_active) AS all_active,               // TRUE if all are TRUE
    BOOL_OR(is_verified) AS any_verified             // TRUE if any is TRUE
FROM users;
```

#### GROUP BY Clause:
```sql
-- Basic GROUP BY (aggregate by category)
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;

-- GROUP BY with multiple aggregates
SELECT 
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;

-- GROUP BY multiple columns
SELECT 
    department,
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    COUNT(*) AS count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department, EXTRACT(YEAR FROM hire_date)
ORDER BY department, hire_year;

-- GROUP BY with HAVING (filter groups after aggregation)
// WHERE filters rows before grouping
// HAVING filters groups after aggregation
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE is_active = TRUE                               // Filter rows first
GROUP BY department
HAVING AVG(salary) > 70000                           // Filter groups
ORDER BY avg_salary DESC;

-- Complex HAVING clause
SELECT department, COUNT(*) AS emp_count, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 5 AND AVG(salary) > 65000          // Multiple conditions on aggregates
ORDER BY emp_count DESC;

-- GROUP BY with WHERE and HAVING
SELECT 
    department,
    COUNT(*) AS active_count,
    AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= '2020-01-01'                      // Only recent hires
GROUP BY department
HAVING COUNT(*) >= 3                                 // Departments with at least 3 recent hires
ORDER BY avg_salary DESC;

-- GROUP BY ALL columns except aggregated (use expressions)
SELECT 
    department,
    CASE 
        WHEN salary < 60000 THEN 'Low'
        WHEN salary < 90000 THEN 'Medium'
        ELSE 'High'
    END AS salary_range,
    COUNT(*) AS count
FROM employees
GROUP BY department, salary_range                    // Group by expression result
ORDER BY department, salary_range;

-- GROUP BY with ROLLUP (PostgreSQL 9.5+)
// Creates subtotals and grand total
SELECT 
    department,
    job_title,
    COUNT(*) AS count,
    SUM(salary) AS total_salary
FROM employees
GROUP BY ROLLUP(department, job_title)
ORDER BY department NULLS FIRST, job_title NULLS FIRST;

-- GROUP BY with CUBE (all possible combinations)
SELECT 
    department,
    location,
    COUNT(*) AS count
FROM employees
GROUP BY CUBE(department, location);

-- GROUPING SETS (specific grouping combinations)
SELECT 
    department,
    job_title,
    COUNT(*) AS count
FROM employees
GROUP BY GROUPING SETS (
    (department),              // Group by department
    (job_title),               // Group by job_title
    (department, job_title),   // Group by both
    ()                         // Grand total
)
ORDER BY department NULLS LAST, job_title NULLS LAST;
```

#### ORDER BY Clause:
```sql
-- Single column ordering
SELECT * FROM employees ORDER BY salary ASC;         // Ascending (default, smallest to largest)
SELECT * FROM employees ORDER BY salary DESC;        // Descending (largest to smallest)
SELECT * FROM employees ORDER BY salary;             // ASC is default

-- Multiple column ordering (priority left to right)
SELECT * FROM employees 
ORDER BY department ASC, salary DESC;                // First by dept (A-Z), then by salary (high to low)

SELECT * FROM employees 
ORDER BY last_name, first_name;                      // Alphabetical by last name, then first name

-- ORDER BY with column position (not recommended, but possible)
SELECT first_name, last_name, salary 
FROM employees 
ORDER BY 3 DESC, 2;                                  // Order by 3rd column (salary), then 2nd (last_name)

-- ORDER BY with expressions
SELECT first_name, last_name, salary
FROM employees
ORDER BY LENGTH(first_name) DESC, last_name;         // Longest names first

SELECT * FROM employees
ORDER BY salary * 12;                                // Order by annual salary

SELECT * FROM employees
ORDER BY LOWER(last_name);                           // Case-insensitive alphabetical

-- ORDER BY with CASE expression
SELECT * FROM employees
ORDER BY 
    CASE department
        WHEN 'Engineering' THEN 1
        WHEN 'Sales' THEN 2
        WHEN 'Marketing' THEN 3
        ELSE 4
    END,
    salary DESC;

-- ORDER BY with NULL handling
SELECT * FROM employees ORDER BY middle_name NULLS FIRST;   // NULLs at the beginning
SELECT * FROM employees ORDER BY middle_name NULLS LAST;    // NULLs at the end (default for ASC)
SELECT * FROM employees ORDER BY bonus DESC NULLS LAST;     // High bonuses first, NULLs at end

-- ORDER BY with aggregate functions (requires GROUP BY)
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;                            // Departments with highest avg salary first

-- ORDER BY with computed columns
SELECT 
    first_name,
    last_name,
    salary,
    salary * 12 AS annual_salary
FROM employees
ORDER BY annual_salary DESC;                         // Can reference alias from SELECT

-- Random order
SELECT * FROM employees ORDER BY RANDOM();           // Different order each time
SELECT * FROM employees ORDER BY RANDOM() LIMIT 10;  // Random sample of 10
```

#### LIMIT and OFFSET:
```sql
-- LIMIT: restrict number of rows returned
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;       // Top 10 highest paid

-- OFFSET: skip rows from beginning
SELECT * FROM employees ORDER BY salary DESC LIMIT 10 OFFSET 10;  // Rows 11-20

-- Pagination pattern (page_number starts from 1)
// Page 1 (rows 1-20)
SELECT * FROM employees ORDER BY id LIMIT 20 OFFSET 0;

// Page 2 (rows 21-40)
SELECT * FROM employees ORDER BY id LIMIT 20 OFFSET 20;

// Page 3 (rows 41-60)
SELECT * FROM employees ORDER BY id LIMIT 20 OFFSET 40;

// Generic formula: OFFSET = (page_number - 1) * page_size

-- LIMIT without ORDER BY (non-deterministic results)
SELECT * FROM employees LIMIT 5;                     // Which 5? Unpredictable!

-- LIMIT 0 (check query validity without returning data)
SELECT * FROM employees LIMIT 0;                     // Returns column structure only

-- FETCH FIRST (SQL standard alternative to LIMIT)
SELECT * FROM employees 
ORDER BY salary DESC 
FETCH FIRST 10 ROWS ONLY;

-- OFFSET with FETCH
SELECT * FROM employees 
ORDER BY salary DESC 
OFFSET 10 ROWS 
FETCH FIRST 10 ROWS ONLY;

-- FETCH FIRST with percentage (PostgreSQL extension)
SELECT * FROM employees 
ORDER BY salary DESC 
FETCH FIRST 10 PERCENT ROWS ONLY;                     // Top 10% by salary

-- WITH TIES: include ties for last row
SELECT * FROM employees 
ORDER BY salary DESC 
FETCH FIRST 5 ROWS WITH TIES;                        // If 5th and 6th have same salary, include both

-- Performance considerations:
// LIMIT is faster than counting all rows
// Use indexes on ORDER BY columns for better performance
// OFFSET becomes slow with large values (consider cursor-based pagination)
```

#### DISTINCT:
```sql
-- Remove duplicate rows from result set
SELECT DISTINCT department FROM employees;

-- DISTINCT on multiple columns (combination must be unique)
SELECT DISTINCT department, location FROM employees;

-- DISTINCT with aggregate functions
SELECT COUNT(DISTINCT department) AS dept_count FROM employees;
SELECT COUNT(DISTINCT location) AS location_count FROM employees;

-- DISTINCT ON (PostgreSQL-specific feature)
// Returns first row of each group
SELECT DISTINCT ON (department) 
    department, 
    first_name, 
    last_name, 
    salary
FROM employees
ORDER BY department, salary DESC;                    // Highest paid employee per department

-- Multiple DISTINCT ON columns
SELECT DISTINCT ON (department, location) 
    department,
    location,
    first_name,
    salary
FROM employees
ORDER BY department, location, salary DESC;          // Highest paid per dept-location combo

-- DISTINCT vs GROUP BY:
// DISTINCT: removes duplicates
SELECT DISTINCT department FROM employees;

// GROUP BY: aggregates
SELECT department FROM employees GROUP BY department;
// Both return same result, but GROUP BY allows aggregates

SELECT department, COUNT(*) FROM employees GROUP BY department;  // Can add aggregates

-- DISTINCT with NULL
SELECT DISTINCT middle_name FROM employees;          // NULL is treated as distinct value
```

#### Subqueries:
```sql
-- Subquery in WHERE clause (scalar subquery returns single value)
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);  // Employees earning above average

-- Subquery with IN (subquery returns multiple rows)
SELECT * FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'NYC'
);

-- Subquery with NOT IN
SELECT * FROM departments
WHERE id NOT IN (
    SELECT DISTINCT department_id FROM employees WHERE department_id IS NOT NULL
);                                                   // Departments with no employees

-- Subquery in FROM clause (derived table / inline view)
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_averages                                   // Must alias derived tables
WHERE avg_sal > 70000;

-- Correlated subquery (references outer query)
// Runs once per row in outer query
SELECT 
    e1.first_name, 
    e1.last_name, 
    e1.salary, 
    e1.department
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department = e1.department              // References outer query's department
);

-- EXISTS subquery (tests existence, returns boolean)
// More efficient than IN for large datasets
SELECT * FROM departments d
WHERE EXISTS (
    SELECT 1                                         // Actual columns don't matter
    FROM employees e
    WHERE e.department_id = d.id AND e.salary > 100000
);                                                   // Departments with at least one high earner

-- NOT EXISTS (find non-matching rows)
SELECT * FROM departments d
WHERE NOT EXISTS (
    SELECT 1 FROM employees e
    WHERE e.department_id = d.id
);                                                   // Departments with no employees

-- Subquery in SELECT clause (must return single value)
SELECT 
    first_name,
    last_name,
    salary,
    (
        SELECT AVG(salary) 
        FROM employees e2 
        WHERE e2.department = e1.department
    ) AS dept_avg_salary,
    salary - (
        SELECT AVG(salary) 
        FROM employees e2 
        WHERE e2.department = e1.department
    ) AS diff_from_avg
FROM employees e1;

-- ANY / ALL with subquery
SELECT * FROM employees
WHERE salary > ALL (                                 // Greater than ALL values in subquery
    SELECT salary FROM employees WHERE department = 'HR'
);                                                   // Higher paid than any HR employee

SELECT * FROM employees
WHERE salary > ANY (                                 // Greater than ANY value in subquery
    SELECT salary FROM employees WHERE department = 'HR'
);                                                   // Higher paid than at least one HR employee

-- Subquery with multiple columns
SELECT * FROM employees
WHERE (department, salary) IN (
    SELECT department, MAX(salary)
    FROM employees
    GROUP BY department
);                                                   // Highest paid employee(s) per department
```

#### CASE Expressions:
```sql
-- Simple CASE (equality comparison)
SELECT 
    first_name,
    last_name,
    salary,
    CASE department
        WHEN 'Engineering' THEN 'Tech'
        WHEN 'Sales' THEN 'Business'
        WHEN 'Marketing' THEN 'Business'
        WHEN 'HR' THEN 'Support'
        ELSE 'Other'
    END AS department_category
FROM employees;

-- Searched CASE (conditional logic)
SELECT 
    first_name,
    last_name,
    salary,
    CASE
        WHEN salary < 50000 THEN 'Entry Level'
        WHEN salary BETWEEN 50000 AND 80000 THEN 'Mid Level'
        WHEN salary BETWEEN 80001 AND 120000 THEN 'Senior Level'
        WHEN salary > 120000 THEN 'Executive Level'
        ELSE 'Unknown'
    END AS salary_range,
    CASE
        WHEN salary < 50000 THEN salary * 1.10       // 10% raise
        WHEN salary < 80000 THEN salary * 1.07       // 7% raise
        ELSE salary * 1.05                           // 5% raise
    END AS new_salary
FROM employees;

-- CASE in aggregate functions (conditional aggregation)
SELECT 
    department,
    COUNT(*) AS total_employees,
    COUNT(CASE WHEN salary > 80000 THEN 1 END) AS high_earners,
    COUNT(CASE WHEN salary <= 80000 THEN 1 END) AS regular_earners,
    SUM(CASE WHEN is_manager THEN 1 ELSE 0 END) AS manager_count,
    AVG(CASE WHEN department = 'Engineering' THEN salary END) AS eng_avg_salary
FROM employees
GROUP BY department;

-- CASE for pivoting data
SELECT 
    first_name,
    last_name,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 1 THEN total ELSE 0 END) AS jan_sales,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 2 THEN total ELSE 0 END) AS feb_sales,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 3 THEN total ELSE 0 END) AS mar_sales
FROM sales_people sp
JOIN orders o ON sp.id = o.salesperson_id
GROUP BY first_name, last_name;

-- CASE in WHERE clause
SELECT * FROM employees
WHERE 
    CASE 
        WHEN department = 'Engineering' THEN salary > 70000
        WHEN department = 'Sales' THEN salary > 60000
        ELSE salary > 50000
    END;

-- CASE in ORDER BY (custom sorting)
SELECT * FROM employees
ORDER BY 
    CASE department
        WHEN 'Executive' THEN 1
        WHEN 'Engineering' THEN 2
        WHEN 'Sales' THEN 3
        WHEN 'Marketing' THEN 4
        ELSE 5
    END,
    salary DESC;

-- CASE in UPDATE
UPDATE employees
SET salary = 
    CASE 
        WHEN performance_rating = 5 THEN salary * 1.15
        WHEN performance_rating = 4 THEN salary * 1.10
        WHEN performance_rating = 3 THEN salary * 1.05
        ELSE salary
    END;

-- Nested CASE expressions
SELECT 
    first_name,
    salary,
    CASE
        WHEN department = 'Engineering' THEN
            CASE
                WHEN salary > 100000 THEN 'Senior Engineer'
                WHEN salary > 70000 THEN 'Mid-Level Engineer'
                ELSE 'Junior Engineer'
            END
        WHEN department = 'Sales' THEN
            CASE
                WHEN salary > 90000 THEN 'Sales Director'
                ELSE 'Sales Rep'
            END
        ELSE 'Other'
    END AS job_level
FROM employees;

-- COALESCE as simpler alternative for NULL handling
// COALESCE returns first non-NULL value
SELECT 
    first_name,
    COALESCE(middle_name, '') AS middle_name,        // Empty string if NULL
    last_name,
    COALESCE(phone, email, 'No contact') AS contact  // First available contact method
FROM employees;

-- NULLIF returns NULL if two values are equal
SELECT 
    first_name,
    salary,
    bonus,
    salary / NULLIF(bonus, 0) AS salary_bonus_ratio  // Avoid division by zero
FROM employees;
```

---

## Part 2: Intermediate Level

### 7. Advanced Queries

#### Window Functions:
```sql
-- ROW_NUMBER(): assigns unique sequential number to each row
SELECT 
    first_name,
    last_name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS salary_rank  // 1, 2, 3, 4...
FROM employees;

-- RANK(): like ROW_NUMBER but gaps after ties
SELECT 
    first_name,
    last_name,
    department,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank,              // 1, 2, 2, 4...
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank   // 1, 2, 2, 3...
FROM employees;

-- PARTITION BY: separate window for each group
SELECT 
    first_name,
    last_name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank_with_gaps
FROM employees;
// Ranks reset for each department

-- Running total with SUM() window function
SELECT 
    first_name,
    last_name,
    hire_date,
    salary,
    SUM(salary) OVER (ORDER BY hire_date) AS running_total,
    SUM(salary) OVER (PARTITION BY department ORDER BY hire_date) AS dept_running_total
FROM employees;

-- Moving average (rolling average)
SELECT 
    first_name,
    hire_date,
    salary,
    AVG(salary) OVER (
        ORDER BY hire_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW     // Window: 2 rows before + current row
    ) AS moving_avg_3,
    AVG(salary) OVER (
        ORDER BY hire_date 
        ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
    ) AS moving_avg_5
FROM employees;

-- Window frame clauses:
// ROWS: physical row offsets
// RANGE: logical value ranges
// GROUPS: peer groups

-- LAG() and LEAD(): access previous/next rows
SELECT 
    first_name,
    hire_date,
    salary,
    LAG(salary, 1) OVER (ORDER BY hire_date) AS prev_hire_salary,    // Previous row
    LEAD(salary, 1) OVER (ORDER BY hire_date) AS next_hire_salary,   // Next row
    salary - LAG(salary, 1) OVER (ORDER BY hire_date) AS salary_diff,
    LAG(salary, 1, 0) OVER (ORDER BY hire_date) AS prev_salary_default  // Default 0 if no previous
FROM employees;

-- FIRST_VALUE() and LAST_VALUE()
SELECT 
    first_name,
    department,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS highest_in_dept,
    LAST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING  // Important for LAST_VALUE!
    ) AS lowest_in_dept,
    salary - FIRST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary DESC) AS diff_from_highest
FROM employees;

-- NTH_VALUE(): get nth value in window
SELECT 
    first_name,
    department,
    salary,
    NTH_VALUE(salary, 2) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_highest_salary
FROM employees;

-- NTILE(): divide rows into N buckets
SELECT 
    first_name,
    salary,
    NTILE(4) OVER (ORDER BY salary) AS salary_quartile,      // Divide into 4 equal groups
    NTILE(10) OVER (ORDER BY salary) AS salary_decile,       // Divide into 10 groups
    NTILE(100) OVER (ORDER BY salary) AS salary_percentile   // Divide into 100 groups
FROM employees;

-- PERCENT_RANK(): relative rank (0 to 1)
SELECT 
    first_name,
    salary,
    PERCENT_RANK() OVER (ORDER BY salary) AS percent_rank,   // 0 = lowest, 1 = highest
    ROUND(PERCENT_RANK() OVER (ORDER BY salary)::NUMERIC * 100, 2) AS percentile
FROM employees;

-- CUME_DIST(): cumulative distribution (0 to 1)
SELECT 
    first_name,
    salary,
    CUME_DIST() OVER (ORDER BY salary) AS cumulative_dist,   // Fraction of rows <= current row
    ROUND(CUME_DIST() OVER (ORDER BY salary)::NUMERIC * 100, 2) AS cumulative_pct
FROM employees;

-- Named window specification (reuse window definition)
SELECT 
    first_name,
    department,
    salary,
    ROW_NUMBER() OVER w AS row_num,
    RANK() OVER w AS rank,
    AVG(salary) OVER w AS dept_avg
FROM employees
WINDOW w AS (PARTITION BY department ORDER BY salary DESC);

-- Complex window function example: gap and island problem
// Find consecutive sequences
SELECT 
    id,
    hire_date,
    id - ROW_NUMBER() OVER (ORDER BY id) AS island_id       // Same value = consecutive IDs
FROM employees
ORDER BY id;
```

#### Common Table Expressions (CTEs):
```sql
-- Basic CTE (WITH clause)
// Makes complex queries more readable
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 80000
)
SELECT department, COUNT(*) AS count, AVG(salary) AS avg_salary
FROM high_earners
GROUP BY department;

-- Multiple CTEs (comma-separated)
WITH 
engineering_emp AS (
    SELECT * FROM employees WHERE department = 'Engineering'
),
high_salary_eng AS (
    SELECT * FROM engineering_emp WHERE salary > 90000
),
avg_stats AS (
    SELECT AVG(salary) AS avg_sal FROM high_salary_eng
)
SELECT e.*, a.avg_sal
FROM high_salary_eng e
CROSS JOIN avg_stats a;

-- CTE with complex calculations
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        SUM(total) AS total_sales,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
),
sales_with_growth AS (
    SELECT 
        month,
        total_sales,
        order_count,
        LAG(total_sales) OVER (ORDER BY month) AS prev_month_sales,
        total_sales - LAG(total_sales) OVER (ORDER BY month) AS sales_growth,
        ROUND(
            (total_sales - LAG(total_sales) OVER (ORDER BY month)) / 
            LAG(total_sales) OVER (ORDER BY month) * 100, 
            2
        ) AS growth_pct
    FROM monthly_sales
)
SELECT * FROM sales_with_growth
WHERE growth_pct IS NOT NULL
ORDER BY month;

-- Recursive CTE: organizational hierarchy
WITH RECURSIVE employee_hierarchy AS (
    -- Base case (anchor member): top-level managers
    SELECT 
        id, 
        first_name, 
        last_name, 
        manager_id, 
        1 AS level,
        first_name || ' ' || last_name AS path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case (recursive member): employees reporting to managers
    SELECT 
        e.id, 
        e.first_name, 
        e.last_name, 
        e.manager_id, 
        eh.level + 1,
        eh.path || ' -> ' || e.first_name || ' ' || e.last_name
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT 
    REPEAT('  ', level - 1) || first_name || ' ' || last_name AS org_chart,
    level,
    path
FROM employee_hierarchy 
ORDER BY path;

-- Recursive CTE: number series generation
WITH RECURSIVE number_series AS (
    SELECT 1 AS n                                            // Base case
    UNION ALL
    SELECT n + 1 FROM number_series WHERE n < 100            // Recursive case
)
SELECT * FROM number_series;

-- Recursive CTE: date series generation
WITH RECURSIVE date_series AS (
    SELECT DATE '2024-01-01' AS date
    UNION ALL
    SELECT date + INTERVAL '1 day'
    FROM date_series
    WHERE date < DATE '2024-12-31'
)
SELECT date, TO_CHAR(date, 'Day') AS day_name
FROM date_series;

-- Recursive CTE: bill of materials (BOM)
WITH RECURSIVE parts_explosion AS (
    -- Top-level products
    SELECT 
        part_id,
        part_name,
        1 AS quantity,
        1 AS level
    FROM parts
    WHERE part_id = 100                                      // Start with specific product
    
    UNION ALL
    
    -- Sub-components
    SELECT 
        p.part_id,
        p.part_name,
        pe.quantity * bom.quantity,                          // Cumulative quantity
        pe.level + 1
    FROM parts p
    INNER JOIN bill_of_materials bom ON p.part_id = bom.component_id
    INNER JOIN parts_explosion pe ON bom.part_id = pe.part_id
)
SELECT 
    REPEAT('  ', level - 1) || part_name AS indented_name,
    quantity,
    level
FROM parts_explosion;

-- CTE for data modification (writeable CTE)
WITH deleted_employees AS (
    DELETE FROM employees
    WHERE hire_date < '2020-01-01'
    RETURNING *                                              // Return deleted rows
)
INSERT INTO employees_archive
SELECT * FROM deleted_employees;

-- CTE with INSERT
WITH new_employees AS (
    INSERT INTO employees (first_name, last_name, email, salary)
    VALUES ('John', 'Doe', 'john@example.com', 75000)
    RETURNING *
)
SELECT id, first_name, last_name FROM new_employees;

-- MATERIALIZED CTE (evaluate once, use multiple times)
// Default behavior since PostgreSQL 12
WITH expensive_query AS MATERIALIZED (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT e.*, eq.avg_salary
FROM employees e
JOIN expensive_query eq ON e.department = eq.department;

-- NOT MATERIALIZED CTE (inline the CTE)
WITH simple_filter AS NOT MATERIALIZED (
    SELECT * FROM employees WHERE salary > 50000
)
SELECT * FROM simple_filter WHERE department = 'Engineering';
```