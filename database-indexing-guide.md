# Database Indexing: How It Makes Your Database Faster

## Table of Contents
1. [What is a Database Index?](#what-is-a-database-index)
2. [The Library Analogy](#the-library-analogy)
3. [How Databases Search Without Indexes](#how-databases-search-without-indexes)
4. [How Indexes Speed Up Searches](#how-indexes-speed-up-searches)
5. [Types of Indexes](#types-of-indexes)
6. [How Indexes Work Internally](#how-indexes-work-internally)
7. [SQL Examples](#sql-examples)
8. [When to Use Indexes](#when-to-use-indexes)
9. [The Trade-offs](#the-trade-offs)
10. [Best Practices](#best-practices)

## What is a Database Index?

A database index is a data structure that improves the speed of data retrieval operations on a database table. Think of it as a special lookup table that the database search engine can use to speed up data retrieval. Simply put, an index is a pointer to data in a table.

### Key Concept
An index creates a separate data structure that holds:
- A sorted copy of selected column(s) data
- Pointers to the actual rows in the table

## The Library Analogy

Imagine you're in a massive library with millions of books, and you need to find all books written by "Stephen King."

### Without an Index (Card Catalog)
You would have to:
1. Start from the first bookshelf
2. Pick up each book one by one
3. Check if the author is Stephen King
4. Continue until you've checked every single book

**Time Required**: If checking one book takes 1 second, and there are 1 million books, it would take about 11.5 days of continuous searching!

### With an Index (Card Catalog)
The library has a card catalog (index) organized alphabetically by author:
1. Go directly to the "K" section
2. Find "King, Stephen"
3. See all the shelf locations listed
4. Walk directly to those specific shelves

**Time Required**: Maybe 2 minutes total!

## How Databases Search Without Indexes

When a database doesn't have an index, it performs what's called a **Full Table Scan** or **Sequential Scan**.

### The Process
```
Start → Read Row 1 → Check Condition → Not Match → 
Read Row 2 → Check Condition → Not Match →
Read Row 3 → Check Condition → Match! → Save Result →
Read Row 4 → Check Condition → Not Match →
... Continue until the last row
```

### Why It's Slow
- The database must read every single row
- It checks each row against your search condition
- Even if it finds matches early, it continues to the end (unless you use LIMIT)
- The time complexity is O(n) where n is the number of rows

### Real-World Impact
If you have a table with 10 million customer records and search for customers from "New York":
- **Without Index**: Database checks all 10 million rows
- **Time**: Could take 30+ seconds

## How Indexes Speed Up Searches

Indexes use sophisticated data structures (usually B-Trees or B+ Trees) that allow the database to find data without scanning every row.

### The Magic of Sorted Data
Indexes keep data sorted, which enables **binary search** instead of linear search.

#### Linear Search (No Index)
```
Looking for value: 42
Check: 15 → No
Check: 7 → No  
Check: 93 → No
Check: 42 → Found!
Checks needed: 4 (potentially all values)
```

#### Binary Search (With Index)
```
Sorted data: [7, 15, 42, 93]
Looking for value: 42

Step 1: Check middle (15 or 42) → Is 42 > 15? Yes, search right half
Step 2: Check middle of right half → Found 42!
Checks needed: 2 (maximum log₂n)
```

### Time Complexity Comparison
- **Without Index**: O(n) - Linear time
- **With Index**: O(log n) - Logarithmic time

For 1 million rows:
- Linear search: Up to 1,000,000 checks
- Binary search: Maximum 20 checks (log₂(1,000,000) ≈ 20)

## Types of Indexes

### 1. Primary Index (Clustered Index)
- **What it is**: The main index that determines the physical order of data in the table
- **Characteristics**: 
  - Only one per table
  - Usually created on the primary key
  - Data is physically sorted based on this index
- **Analogy**: Like organizing books on shelves by ISBN number

### 2. Secondary Index (Non-Clustered Index)
- **What it is**: Additional indexes that don't affect physical data order
- **Characteristics**:
  - Can have multiple per table
  - Contains pointers to the actual data
  - Like having multiple card catalogs (by author, by title, by subject)

### 3. Unique Index
- **What it is**: Ensures all values in the indexed column are unique
- **Use case**: Email addresses, usernames, social security numbers

### 4. Composite Index (Multi-Column Index)
- **What it is**: Index on multiple columns combined
- **Example**: Index on (last_name, first_name)
- **Benefit**: Speeds up queries that search by multiple columns

### 5. Full-Text Index
- **What it is**: Special index for text searching
- **Use case**: Searching within large text fields like article content

## How Indexes Work Internally

### B-Tree Structure (Most Common)

A B-Tree is like a family tree, but for data:

```
                    [50]
                   /    \
                 /        \
            [20, 30]    [70, 80]
           /   |   \    /   |   \
        [10] [25] [35][60] [75] [90]
```

#### How B-Tree Searching Works
Looking for value 25:
1. Start at root: Is 25 < 50? Yes, go left
2. At [20, 30]: Is 25 between 20 and 30? Yes
3. Go to middle child
4. Found [25]!

**Only 3 steps instead of potentially checking all values!**

### What Makes B-Trees Fast

1. **Balanced Structure**: All paths from root to leaves have similar length
2. **Multiple Values per Node**: Reduces tree height
3. **Sorted Within Nodes**: Enables binary search within each node
4. **Disk-Optimized**: Each node can be one disk page

### Index Storage

Indexes are stored separately from the table data:

```
Main Table (Employees):
+----+---------+------------+--------+
| ID | Name    | Department | Salary |
+----+---------+------------+--------+
| 1  | Alice   | IT         | 70000  |
| 2  | Bob     | HR         | 60000  |
| 3  | Charlie | IT         | 75000  |
+----+---------+------------+--------+

Index on Department:
+------------+-------------+
| Department | Row Pointer |
+------------+-------------+
| HR         | → Row 2     |
| IT         | → Row 1     |
| IT         | → Row 3     |
+------------+-------------+
```

## SQL Examples

### Creating a Table Without Indexes

```sql
-- Create a table for employees
CREATE TABLE employees (
    employee_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10, 2),
    hire_date DATE
);

-- Insert sample data (imagine 1 million rows)
INSERT INTO employees VALUES 
(1, 'John', 'Doe', 'john.doe@company.com', 'Engineering', 75000, '2020-01-15'),
(2, 'Jane', 'Smith', 'jane.smith@company.com', 'Marketing', 65000, '2019-03-22'),
-- ... many more rows
(1000000, 'Alice', 'Johnson', 'alice.j@company.com', 'Sales', 70000, '2021-07-01');
```

### Query Performance Without Index

```sql
-- This query will be SLOW without an index
SELECT * FROM employees 
WHERE last_name = 'Smith';

-- Database must check all 1 million rows
-- Execution time: ~5-10 seconds
```

### Creating Indexes

```sql
-- 1. Create a simple index on last_name
CREATE INDEX idx_lastname ON employees(last_name);

-- Now the same query is FAST
SELECT * FROM employees 
WHERE last_name = 'Smith';
-- Execution time: ~0.01 seconds

-- 2. Create a unique index on email
CREATE UNIQUE INDEX idx_email ON employees(email);

-- 3. Create a composite index for common query patterns
CREATE INDEX idx_dept_salary ON employees(department, salary);

-- This query benefits from the composite index
SELECT * FROM employees 
WHERE department = 'Engineering' 
AND salary > 70000;

-- 4. Create an index with included columns (covering index)
CREATE INDEX idx_lastname_with_email 
ON employees(last_name) 
INCLUDE (email, first_name);
```

### Checking Query Execution Plans

```sql
-- MySQL
EXPLAIN SELECT * FROM employees WHERE last_name = 'Smith';

-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM employees WHERE last_name = 'Smith';

-- SQL Server
SET SHOWPLAN_TEXT ON;
GO
SELECT * FROM employees WHERE last_name = 'Smith';
GO
```

### Example Output of EXPLAIN

```sql
-- Without Index:
+----+-------------+-----------+------+---------------+------+---------+------+--------+-------------+
| id | select_type | table     | type | possible_keys | key  | key_len | ref  | rows   | Extra       |
+----+-------------+-----------+------+---------------+------+---------+------+--------+-------------+
| 1  | SIMPLE      | employees | ALL  | NULL          | NULL | NULL    | NULL | 1000000| Using where |
+----+-------------+-----------+------+---------------+------+---------+------+--------+-------------+

-- With Index:
+----+-------------+-----------+------+---------------+--------------+---------+-------+------+-------+
| id | select_type | table     | type | possible_keys | key          | key_len | ref   | rows | Extra |
+----+-------------+-----------+------+---------------+--------------+---------+-------+------+-------+
| 1  | SIMPLE      | employees | ref  | idx_lastname  | idx_lastname | 52      | const | 5    | NULL  |
+----+-------------+-----------+------+---------------+--------------+---------+-------+------+-------+
```

Notice how "rows" dropped from 1,000,000 to just 5!

### Index Usage in Different Scenarios

```sql
-- Scenario 1: Equality Search (Perfect for indexes)
SELECT * FROM employees WHERE employee_id = 12345;
-- With index on employee_id: <0.001 seconds

-- Scenario 2: Range Search (Good for indexes)
SELECT * FROM employees 
WHERE hire_date BETWEEN '2020-01-01' AND '2020-12-31';
-- With index on hire_date: ~0.1 seconds

-- Scenario 3: Sorting (Indexes help!)
SELECT * FROM employees 
ORDER BY salary DESC 
LIMIT 10;
-- With index on salary: ~0.01 seconds
-- Without index: Database must sort all rows first

-- Scenario 4: Joining Tables (Critical for performance)
SELECT e.*, d.department_name 
FROM employees e
JOIN departments d ON e.department_id = d.id;
-- Indexes on both e.department_id and d.id make this fast

-- Scenario 5: Composite Index Usage
-- Index on (department, salary)
SELECT * FROM employees 
WHERE department = 'Engineering' AND salary > 70000;
-- Uses full composite index: Very fast

SELECT * FROM employees WHERE department = 'Engineering';
-- Still uses the index (leftmost part): Fast

SELECT * FROM employees WHERE salary > 70000;
-- Cannot use the composite index efficiently: Slower
```

## When to Use Indexes

### Perfect Candidates for Indexing

1. **Primary Keys**: Always indexed automatically
2. **Foreign Keys**: Columns used in JOIN operations
3. **Columns in WHERE Clauses**: Frequently searched columns
4. **Columns in ORDER BY**: Frequently sorted columns
5. **Columns in GROUP BY**: Grouping operations

### When NOT to Use Indexes

1. **Small Tables**: Tables with fewer than 1000 rows
2. **Frequently Updated Columns**: Indexes slow down writes
3. **Columns with Low Cardinality**: Like gender (only M/F values)
4. **Wide Columns**: Very long text fields

## The Trade-offs

### Benefits of Indexes

1. **Faster Searches**: Dramatic speed improvement for SELECT queries
2. **Faster Sorting**: ORDER BY operations are quicker
3. **Faster Joins**: JOIN operations are more efficient
4. **Unique Constraints**: Enforce data integrity

### Costs of Indexes

1. **Storage Space**: Each index requires additional disk space
   - Rule of thumb: An index can be 10-20% the size of the table

2. **Slower Writes**: INSERT, UPDATE, DELETE operations are slower
   - Each write must update the table AND all its indexes

3. **Maintenance Overhead**: Indexes need to be rebuilt periodically

### Real-World Example of Trade-offs

```sql
-- Table with 10 million rows, 5 GB in size

-- Without any indexes:
SELECT: 10 seconds (slow)
INSERT: 0.001 seconds (fast)
Storage: 5 GB

-- With 5 indexes:
SELECT: 0.01 seconds (fast!)
INSERT: 0.05 seconds (slower)
Storage: 6.5 GB (30% more)
```

## Best Practices

### 1. Index Selection Strategy

```sql
-- Analyze your queries first
-- Find the most frequent WHERE conditions

-- Good: Index on frequently searched column
CREATE INDEX idx_customer_email ON customers(email);

-- Better: Composite index for common query pattern
CREATE INDEX idx_customer_location ON customers(country, city);

-- Best: Covering index that includes all needed columns
CREATE INDEX idx_customer_search 
ON customers(email) 
INCLUDE (first_name, last_name, phone);
```

### 2. Monitor Index Usage

```sql
-- Check which indexes are being used (PostgreSQL)
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Find unused indexes (MySQL)
SELECT 
    table_schema,
    table_name,
    index_name,
    cardinality
FROM information_schema.statistics
WHERE table_schema = 'your_database'
ORDER BY cardinality DESC;
```

### 3. Index Maintenance

```sql
-- Rebuild fragmented indexes (SQL Server)
ALTER INDEX idx_lastname ON employees REBUILD;

-- Analyze table statistics (PostgreSQL)
ANALYZE employees;

-- Optimize table (MySQL)
OPTIMIZE TABLE employees;
```

### 4. Common Patterns

```sql
-- Pattern 1: Index for lookup tables
CREATE INDEX idx_status ON orders(status)
WHERE status IN ('pending', 'processing'); -- Partial index

-- Pattern 2: Index for date ranges
CREATE INDEX idx_created_at ON events(created_at DESC);

-- Pattern 3: Index for full-text search
CREATE FULLTEXT INDEX idx_content ON articles(title, body);

-- Pattern 4: Index for JSON data (PostgreSQL)
CREATE INDEX idx_metadata ON products((metadata->>'category'));
```

## Understanding Index Selectivity

**Selectivity** is how unique the values in a column are.

### High Selectivity (Good for Indexes)
- Unique IDs: 100% selective
- Email addresses: Nearly 100% selective
- Phone numbers: Very high selectivity

### Low Selectivity (Poor for Indexes)
- Boolean fields: Only 2 values (50% selective)
- Gender: Usually 2-3 values
- Status fields: Often just a few values

### Rule of Thumb
```
Selectivity = Number of Distinct Values / Total Number of Rows

Good for index: Selectivity > 0.1 (10%)
Poor for index: Selectivity < 0.01 (1%)
```

## Advanced Concepts

### Covering Indexes
A covering index includes all columns needed by a query, so the database never needs to access the actual table.

```sql
-- Query
SELECT first_name, last_name, email 
FROM employees 
WHERE department = 'Engineering';

-- Covering index for this query
CREATE INDEX idx_dept_covering 
ON employees(department) 
INCLUDE (first_name, last_name, email);

-- Now the query can be answered entirely from the index!
```

### Partial Indexes
Index only a subset of rows that match a condition.

```sql
-- Only index active users (saves space)
CREATE INDEX idx_active_users 
ON users(email) 
WHERE status = 'active';

-- This query uses the partial index
SELECT * FROM users 
WHERE email = 'john@example.com' 
AND status = 'active';
```

### Function-Based Indexes
Index the result of a function or expression.

```sql
-- Index on uppercase last name for case-insensitive searches
CREATE INDEX idx_upper_lastname 
ON employees(UPPER(last_name));

-- This query can use the function-based index
SELECT * FROM employees 
WHERE UPPER(last_name) = 'SMITH';
```

## Conclusion

Database indexes are like the table of contents, index, and bookmarks of a book all rolled into one. They trade storage space and write performance for dramatically faster read operations. Understanding how and when to use indexes is crucial for building performant database applications.

### Key Takeaways

1. **Indexes are separate data structures** that maintain sorted copies of column data with pointers to rows
2. **They change O(n) operations to O(log n)**, making searches exponentially faster as data grows
3. **Every index has a cost**: storage space and slower writes
4. **Choose indexes based on your query patterns**, not randomly
5. **Monitor and maintain your indexes** regularly
6. **One good composite index** can be better than multiple single-column indexes

### Final Tip
Start with indexes on primary keys and foreign keys, then add indexes based on your actual query performance needs. Use EXPLAIN plans to verify that your indexes are being used effectively, and remove indexes that aren't providing value.

Remember: The goal isn't to index everything, but to index smartly based on your application's specific needs.
source: https://youtu.be/3G293is403I?si=Jhy-FmtyCUxvku8s
