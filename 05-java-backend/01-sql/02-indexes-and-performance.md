# Database Indexes and Performance

## 1. What Is An Index?
An index is a data structure (typically a B-Tree) that improves the speed of data retrieval operations on a database table at the cost of additional storage space and slower writes (INSERT, UPDATE, DELETE).

## 2. Types of Indexes
- **Clustered Index:** Defines the physical sorting order of the data on the disk. A table can only have ONE clustered index (usually the Primary Key). The leaf nodes contain the actual row data.
- **Non-Clustered (Secondary) Index:** Does not sort the physical data. It creates a separate structure mapping the indexed column to the Primary Key. A lookup requires two steps: Find the PK in the secondary index -> Look up the row in the clustered index (Bookmark Lookup).

## 3. Composite Indexes and The Leftmost Prefix Rule
A composite index spans multiple columns, e.g., `INDEX(last_name, first_name)`.
- **Leftmost Prefix Rule:** The database can only use the index if the query filters on the columns from left to right. 
  - `WHERE last_name = 'Smith'` -> Uses Index.
  - `WHERE last_name = 'Smith' AND first_name = 'John'` -> Uses Index.
  - `WHERE first_name = 'John'` -> **Cannot** use index. Performs a full table scan.

## 4. Query Optimization and Execution Plans
- **EXPLAIN (or EXPLAIN ANALYZE):** A command used to see the execution plan of a query. It shows whether the DB is using an Index Scan, Index Seek, or a Full Table Scan.
- **Covering Index:** If all the columns requested in the `SELECT` clause are present in the secondary index itself, the database doesn't need to perform the second step (bookmark lookup) to the clustered index. This is extremely fast.

## 5. Common Performance Mistakes
- **Indexing every column:** Slows down write operations drastically because every index must be updated on every INSERT.
- **Functions on indexed columns:** `WHERE YEAR(created_at) = 2023` destroys the index usage. The DB must calculate the function for every row. Fix: `WHERE created_at >= '2023-01-01' AND created_at < '2024-01-01'`.
- **N+1 Queries:** *(See JPA section).*

## 6. Interview Questions
### Q. What happens if you index a boolean column like `is_active`?
**Short Answer:** It will likely result in poor performance (low cardinality). If 99% of users are active, searching for `is_active = true` via an index means doing an index lookup and then a table lookup 99% of the time, which is actually slower than just scanning the table directly. The query optimizer will likely ignore the index entirely.

---

[← Previous: SQL Fundamentals & Queries](01-sql-fundamentals-and-queries.md) | [Next: Transactions and Locks →](03-transactions-and-locks.md)
