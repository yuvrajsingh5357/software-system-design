# JPA Relationships and Fetching Strategies

## 1. Defining Relationships
Databases use Foreign Keys. JPA uses Object References.
- `@OneToOne`: E.g., User and UserProfile.
- `@ManyToOne` / `@OneToMany`: E.g., Department and Employee.
- `@ManyToMany`: E.g., Student and Course (requires a join table).

## 2. Bidirectional Relationships and "mappedBy"
In a `@OneToMany` relationship (e.g., Post -> Comment), both sides can map the relationship.
- However, relational databases only have one Foreign Key (on the `comment` table).
- To tell Hibernate that the `Comment` entity is the "owner" of the relationship (the one that actually controls the Foreign Key column), the `Post` entity must use `mappedBy = "post"`.
- If you don't use `mappedBy`, Hibernate will create an unnecessary third Join Table!

## 3. Fetching Strategies: LAZY vs EAGER
When you fetch a `Post` from the DB, should Hibernate immediately fetch all its `Comments`?
- **EAGER:** Fetches the associations immediately using a SQL JOIN (or a subsequent SELECT).
- **LAZY:** Creates a **Proxy** object for the association. The database is only hit when you actually call `.getComments().size()`.

**Best Practice:** Always use `FetchType.LAZY` for all associations (even `@ManyToOne` which is EAGER by default). EAGER fetching loads massive object graphs unnecessarily and destroys performance.

## 4. The N+1 Select Problem
*(See 05-jpa-hibernate/n-plus-one.md for full details).*
This occurs when you fetch N parent entities and then lazily access their children in a loop, resulting in N+1 total queries.
**Solution:** Use `JOIN FETCH` in JPQL or `@EntityGraph` to fetch parents and children in a single SQL query.

## 5. Cascade and OrphanRemoval
- **Cascade:** If you delete a `Post`, you probably want to delete all its `Comments`. `cascade = CascadeType.ALL` propagates entity state transitions from parent to child.
- **OrphanRemoval:** If you remove a `Comment` from the `Post.comments` list, `orphanRemoval = true` tells Hibernate to permanently delete that `Comment` from the database.

## 6. Interview Questions
### Q. What is a Hibernate Proxy?
**Short Answer:** A proxy is a dynamically generated subclass created by Hibernate to support Lazy Loading. When you request a lazy association, Hibernate returns a proxy object containing only the primary key. When you access any other method on that proxy, the proxy intercepts the call, hits the database to fetch the real data, and initializes the object.

### Q. What causes a LazyInitializationException?
**Short Answer:** It occurs when you try to access a lazily-loaded association of an entity that is in a `Detached` state. This usually happens when you return an entity from the Service layer (where the `@Transactional` boundary and Persistence Context close) to the Controller layer, and the Controller tries to call `.getComments()`.

---

[← Previous: JPA & Hibernate Fundamentals (Persistence Context)](01-jpa-hibernate-fundamentals.md) | [Next: The N+1 Select Problem →](03-n-plus-one-problem.md)
