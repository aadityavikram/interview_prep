# 1. Indexes in SQL, how will you create one and how does it work?

1. What is an Index in SQL?  
  a. An index in SQL is like the index in the back of a book. Instead of flipping through every page to find a word, you can jump directly to where it’s located.  
  b. Without an index: SQL scans the entire table (full table scan).  
  c. With an index: SQL uses a sorted data structure (usually a B-Tree or sometimes a Hash) to quickly locate rows.  
  d. Indexes improve read performance (SELECT queries) but can slow down writes (INSERT, UPDATE, DELETE) because the index also needs to be updated.  

2. How to Create an Index  
  a. Single-column index
```sql
CREATE INDEX idx_customer_name
ON Customers (CustomerName);
```

This will make queries filtering by CustomerName much faster, e.g.
```sql
SELECT * FROM Customers WHERE CustomerName = 'Aaditya';
```

  b. Composite (multi-column) index
```sql
CREATE INDEX idx_customer_city_country
ON Customers (City, Country);
```

Useful when queries often filter by City AND Country together.  
Order matters: This index will help queries filtering by (City) or (City, Country) but not by (Country) alone.  

  c. Unique index  
```sql
CREATE UNIQUE INDEX idx_unique_email
ON Users (Email);
```
Ensures all emails are unique, often used for constraints.  

  d. Index on expressions (some databases)  
```sql
CREATE INDEX idx_lower_email
ON Users (LOWER(Email));
```
Useful if queries often use functions:  
```sql
SELECT * FROM Users WHERE LOWER(Email) = 'test@gmail.com';
```

3. How It Works Internally  

  | Index | Feature |
  |-------|---------|
  | B-Tree Index (default in most RDBMS like MySQL, PostgreSQL, Oracle) | Stores data in a balanced tree structure. |
  | | Lookup, insert, and delete take O(log n) time. |
  | | Great for range queries (BETWEEN, <, >, ORDER BY). |
  | Hash Index (used in some cases, e.g., PostgreSQL) | Uses a hash table. |
  | | Very fast for equality lookups (=). |
  | | Not useful for range queries. |
  | Bitmap Index (Oracle, Data Warehousing) | Uses bitmaps for low-cardinality columns (e.g., Gender: Male/Female). |
  | | Very efficient for aggregations. |

4. Pros and Cons

  | Pros | Cons |
  |------|------|
  | Columns frequently used in WHERE, JOIN, ORDER BY, GROUP BY. | Columns with very few unique values (e.g., is_active with only 0/1). |
  | High-selectivity columns (many unique values, e.g., email, customer_id). | Tables that are mostly written to (heavy inserts/updates). |

<br />
<br />
<br />




# 2. Clustered vs Non-Clustered Indexes

1. Clustered Index  
  a. A clustered index determines the physical order of rows in a table (like the main arrangement of data).  
  c. b. Think of it like a dictionary: words are stored in alphabetical order, so the data itself is ordered.  
  d. Only one clustered index per table, because data can only be physically sorted one way.  
  e. In SQL Server, the Primary Key creates a clustered index by default (unless specified otherwise).  
  f. In MySQL (InnoDB engine), the Primary Key is always the clustered index.  
  - g. Best for columns that are:  
    - 1. Primary keys, unique identifiers  
    - 2. Frequently used in range queries (BETWEEN, <, >).  

```sql
CREATE TABLE my_table (
    id INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(100),
    PRIMARY KEY (id)
);
```

2. Non-Clustered Index  
  a. A non-clustered index is like a separate lookup table that has pointers to the actual table rows.  
  b. Think of it like a book index at the end of a textbook: the index has keywords + page numbers.  
  c. You can create multiple non-clustered indexes on a table.  
  d. Doesn’t change the physical order of the data.  
  - e. Best for:  
    - 1. Columns used often in WHERE, JOIN, ORDER BY, GROUP BY.  
    - 2. Queries that don’t align with the clustered index column.

```sql
CREATE INDEX index_name ON table_name (column1, column2, ...);
```

<br />
<br />
<br />



# 3. 
