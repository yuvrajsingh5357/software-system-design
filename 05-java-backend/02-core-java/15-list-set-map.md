# List, Set, and Map Differences

*(Key points extracted from Java Collections deep dive)*

## 1. List vs Set

| Aspect | List (`ArrayList`, `LinkedList`) | Set (`HashSet`, `TreeSet`) |
|--------|----------------------------------|----------------------------|
| **Duplicates** | Allows duplicates. | Does **not** allow duplicates. |
| **Ordering** | Maintains insertion order. | No guaranteed order (`HashSet`), or naturally sorted (`TreeSet`). |
| **Null Elements**| Allows multiple nulls. | `HashSet` allows exactly one null. `TreeSet` throws exception. |
| **Access** | Index-based access (`get(index)`). | No index access. Focuses on value presence lookup (`contains()`). |
| **Use Case** | Ordered data logs, to-do lists. | Unique tags, IDs, user roles. |

## 2. HashSet vs LinkedHashSet vs TreeSet

| Aspect | `HashSet` | `LinkedHashSet` | `TreeSet` |
|--------|-----------|-----------------|-----------|
| **Internal Structure** | Hash Table | Hash Table + Linked List | Red-Black Tree |
| **Ordering** | Unpredictable / Random | Maintains insertion order | Sorted order (Natural or Comparator) |
| **Performance** | **Fastest** $O(1)$ for add/remove/lookup | Slightly slower than HashSet | **Slowest** $O(\log N)$ due to tree balancing |
| **Null Handling** | Allows 1 null | Allows 1 null | Does **not** allow null |
| **Use Case** | Fast uniqueness check | Uniqueness + order (e.g. recent items) | Uniqueness + sorted data (e.g. rankings) |

## 3. HashMap vs LinkedHashMap

| Aspect | `HashMap` | `LinkedHashMap` |
|--------|-----------|-----------------|
| **Internal Structure** | Hash Table | Hash Table + Linked List |
| **Ordering** | Unpredictable iteration order | Maintains insertion order (or access order) |
| **Performance** | Faster | Slightly slower |
| **Memory** | Lower memory usage | Higher memory (stores prev/next pointers) |
| **Use Case** | Unordered fast lookups | Order-sensitive tasks like LRU Caches |

## 4. HashMap vs ConcurrentHashMap

| Aspect | `HashMap` | `ConcurrentHashMap` |
|--------|-----------|---------------------|
| **Thread Safety** | **Not** thread-safe. | **Thread-safe** (safe concurrent access). |
| **Null Handling**| Allows 1 null key, multiple null values. | **Disallows** null keys and null values. |
| **Iterator** | Fail-fast | Fail-safe |
| **Locking** | None | Lock striping (locks only segments/buckets, not the whole map). |

## 5. CopyOnWriteArraySet vs HashSet

- **`HashSet`:** Fast, not thread-safe. Throws exceptions on concurrent modification.
- **`CopyOnWriteArraySet`:** Thread-safe. Backed by an array. On every write operation (`add`, `remove`), it creates a fresh copy of the entire underlying array. 
- **Use Case:** Extremely slow for writes. Only use for collections that are heavily read and very rarely modified (like an observer list or feature flag registry).

---

[← Previous: Collections Framework](14-collections.md) | [Next: HashMap Internals →](16-hashmap-internals.md)
