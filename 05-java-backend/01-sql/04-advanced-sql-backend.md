# Backend SQL Patterns: Pagination and Databases

## 1. Pagination: Offset vs Keyset

### Offset Pagination (The Bad Way)
`SELECT * FROM users ORDER BY id LIMIT 50 OFFSET 10000;`
- **How it works:** The database fetches 10050 rows, throws away the first 10000, and returns the last 50.
- **Problem:** Performance degrades linearly as the offset grows. Very slow for deep pages.

### Keyset Pagination (The Good Way - "Cursor" Pagination)
`SELECT * FROM users WHERE id > 10000 ORDER BY id LIMIT 50;`
- **How it works:** The database uses the index on `id` to jump directly to ID 10000 and fetches the next 50 rows.
- **Benefit:** Constant time $O(1)$ regardless of how deep the page is. 
- **Drawback:** You cannot jump to a specific page number (e.g., "Page 45"). You can only go "Next" or "Previous".

## 2. MySQL vs PostgreSQL (Backend Perspective)
Both are incredible open-source RDBMS, but they have different philosophies.

- **MySQL (InnoDB):**
  - Uses Index-Organized Tables (The clustered index IS the table).
  - Repeatable Read is the default isolation level.
  - Historically focused on extreme read-speed for web apps.
- **PostgreSQL:**
  - Uses Heap Tables (Data is stored in an unordered heap; primary key is a secondary index pointing to the heap).
  - Read Committed is the default isolation level.
  - Known for strict SQL compliance, advanced data types (JSONB, Arrays), and complex analytical query performance.

## 3. Interview Questions
### Q. How do you handle pagination for an API with millions of records?
**Short Answer:** I would use Keyset (Cursor) Pagination rather than Offset Pagination. Offset pagination requires the database to scan and discard rows, becoming extremely slow for high page numbers. Keyset pagination uses the index of the sort column (like a unique ID or timestamp) to jump directly to the next set of rows, ensuring $O(1)$ performance.

---

[← Previous: Transactions and Locks](03-transactions-and-locks.md) | [Next: Java Fundamentals & Platform Independence →](../02-core-java/01-java-fundamentals.md)
