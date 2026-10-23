# SQL Fundamentals, Constraints, and Queries

## 1. What Is It?
Structured Query Language (SQL) is the standard language for interacting with Relational Database Management Systems (RDBMS). For a backend engineer, writing efficient SQL is as important as writing efficient Java code.

## 2. Constraints
Constraints ensure data integrity at the database level.
- **Primary Key (PK):** Uniquely identifies a row. Indexed automatically. (Usually a BIGINT auto-increment or UUID).
- **Foreign Key (FK):** Ensures referential integrity between tables. If an Order references User ID 5, User 5 must exist.
- **Unique:** Ensures all values in a column are distinct. (Creates a unique index behind the scenes).
- **Not Null:** Ensures a column cannot have NULL values.

## 3. Joins
Joins combine rows from two or more tables based on a related column.
- **INNER JOIN:** Returns records that have matching values in both tables.
- **LEFT JOIN:** Returns all records from the left table, and the matched records from the right table (NULL if no match).
- **RIGHT JOIN:** Opposite of LEFT JOIN.
- **FULL OUTER JOIN:** Returns all records when there is a match in either left or right table.

*Backend Context:* Avoid joining too many tables (e.g., > 5) in a single OLTP query, as it can cause performance degradation and lock contention. In microservices, joins across service boundaries are impossible, requiring API composition or data replication.

## 4. Subqueries and CTEs (Common Table Expressions)
- **Subquery:** A query nested inside another query. Can be in the SELECT, FROM, or WHERE clause.
- **CTE:** Defined using the `WITH` keyword. It acts as a temporary result set that you can reference within another SELECT, INSERT, UPDATE, or DELETE statement.
*Why CTE over Subquery?* Readability and reusability. A CTE reads top-to-bottom, whereas a nested subquery reads inside-out.

## 5. Window Functions
Perform calculations across a set of table rows that are somehow related to the current row, WITHOUT collapsing the rows (unlike GROUP BY).
- Examples: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LEAD()`, `LAG()`.
- Syntax: `RANK() OVER (PARTITION BY department_id ORDER BY salary DESC)`
- *Use Case:* Finding the top 3 highest-paid employees in *each* department without writing complex self-joins.

## 6. Interview Questions
### Q. What is the difference between WHERE and HAVING?
**Short Answer:** `WHERE` filters rows *before* aggregation (GROUP BY) takes place. `HAVING` filters rows *after* the aggregation has been applied.

### Q. What is the difference between RANK() and DENSE_RANK()?
**Short Answer:** If two rows tie for 1st place, `RANK()` will assign them both rank 1, and the next row will be rank 3 (skipping 2). `DENSE_RANK()` will assign them both 1, and the next row will be 2 (no gaps).

---

[Next: Indexes and Performance →](02-indexes-and-performance.md)
