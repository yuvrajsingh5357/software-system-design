# JPA: The N+1 Select Problem

## 1. What is it?
A performance anti-pattern where an ORM executes 1 query to fetch a list of N parent entities, and then N additional queries to fetch their lazy-loaded associations.

## 2. Mental Model
You want a list of 100 Posts and their Authors.
Query 1: `SELECT * FROM Post` (returns 100 posts).
You loop through the posts: `post.getAuthor().getName()`.
Query 2-101: `SELECT * FROM Author WHERE id = ?`.
Total queries: 101. 

## 3. Why is this designed this way?
Associations are `LAZY` by default (or should be). If Spring fetched the Author automatically every time, it might fetch millions of rows you didn't need. 

## 4. Solutions (Implementation)
1. **Fetch Join (JPQL):**
   `@Query("SELECT p FROM Post p JOIN FETCH p.author")`
   This translates to a single SQL `INNER JOIN`, fetching both simultaneously.
2. **Entity Graphs:**
   `@EntityGraph(attributePaths = {"author"})` on the repository method.

## 5. Production Debugging
**Symptom:** High CPU on database, high latency on API.
**Fix:** Turn on `spring.jpa.show-sql=true` in development. Look for repetitive identical queries. Better yet, use a tool like `datasource-proxy` to assert query counts in integration tests.

## 6. Interview Questions
**Q: If I change @OneToMany(fetch = FetchType.EAGER), does it fix N+1?**
**Answer:** NO! EAGER fetching often *causes* N+1. If you use standard Spring Data `findAll()`, Hibernate executes the select for the parent, realizes it needs the children, and issues N subsequent selects. EAGER only works gracefully if you explicitly use a JOIN. Always use LAZY and explicitly FETCH JOIN when needed.

---

[← Previous: Relationships & Fetching Strategies](02-relationships-fetching.md) | [Next: HTTP & REST Fundamentals →](../06-rest-web/01-http-rest-fundamentals.md)
