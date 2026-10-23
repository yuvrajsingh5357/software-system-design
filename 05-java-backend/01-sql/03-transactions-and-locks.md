# Transactions, Isolation Levels, and Locks

## 1. What Is A Transaction?
A transaction is a sequence of database operations treated as a single, atomic unit of work. Either all operations succeed, or none do.

## 2. ACID Properties
- **Atomicity:** "All or nothing." If one step fails, the entire transaction rolls back.
- **Consistency:** The transaction must leave the DB in a valid state, respecting all constraints and foreign keys.
- **Isolation:** Concurrent transactions should not interfere with each other.
- **Durability:** Once committed, the changes are permanent, even if the DB crashes immediately after.

## 3. Concurrency Problems
- **Dirty Read:** Reading uncommitted changes from another transaction.
- **Non-Repeatable Read:** Reading a row, another transaction updates it, and reading it again yields a different value.
- **Phantom Read:** Reading a set of rows matching a condition, another transaction inserts a new row matching the condition, and reading again yields a different set of rows.

## 4. Isolation Levels (Trade-off: Consistency vs Performance)
1. **Read Uncommitted:** Highest performance, lowest consistency. Allows Dirty Reads.
2. **Read Committed:** Prevents Dirty Reads. (Default in PostgreSQL, SQL Server).
3. **Repeatable Read:** Prevents Dirty and Non-Repeatable Reads. (Default in MySQL/InnoDB).
4. **Serializable:** Highest consistency, lowest performance. Transactions execute as if they were sequential. Prevents all concurrency problems.

## 5. Locking Mechanisms
- **Pessimistic Locking:** "Lock the row now because I expect contention." `SELECT * FROM orders WHERE id=1 FOR UPDATE;`. No other transaction can read/update this row until the current transaction commits. Can cause deadlocks.
- **Optimistic Locking:** "Don't lock the row, just check if it changed before saving." Uses a `version` column. When saving: `UPDATE orders SET status='PAID', version=2 WHERE id=1 AND version=1`. If rows updated = 0, someone else modified it. Throw an exception.

## 6. Deadlocks
Occurs when Tx A holds Lock 1 and waits for Lock 2, while Tx B holds Lock 2 and waits for Lock 1.
*Prevention:* Always acquire locks in the same consistent order across the application. Keep transactions short.

## 7. Interview Questions
### Q. How does Spring implement Transactions?
**Short Answer:** Via AOP Proxies. When you annotate a method with `@Transactional`, Spring wraps the class in a proxy. When the method is called, the proxy intercepts the call, opens a DB connection, starts a transaction, executes the method, and then commits (or rolls back if a RuntimeException is thrown).

### Q. What happens if a private method is annotated with @Transactional?
**Short Answer:** It will be completely ignored. Spring's default AOP uses dynamic proxies, which can only intercept public method calls coming from outside the proxy instance.

---

[← Previous: Indexes and Performance](02-indexes-and-performance.md) | [Next: Advanced SQL Patterns (Pagination) →](04-advanced-sql-backend.md)
