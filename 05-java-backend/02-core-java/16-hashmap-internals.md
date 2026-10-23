# HashMap Deep Dive

## 1. What is it?
`HashMap` is a hash table based implementation of the `Map` interface in Java. It stores data in (Key, Value) pairs.

## 2. Why do we need it?
*Problem:* Finding an element in a List takes $O(N)$. We needed a data structure that provides $O(1)$ average time complexity for `put` and `get` operations.
*Solution:* Hashing.
*(Note: See [Hash Tables](../../01-dsa/08-hash-tables.md) in the DSA section for the fundamental algorithmic theory. Below focuses on Java internals).*

## 3. How does it work? (Internal Working)
1. **hash():** Java doesn't just use `key.hashCode()`. It uses a custom `hash()` function: `(h = key.hashCode()) ^ (h >>> 16)`. 
   *Why?* To mitigate poor hash functions by blending higher bits with lower bits, reducing collisions.
2. **Bucket Selection:** `index = (n - 1) & hash`, where `n` is the table capacity (always a power of 2).
3. **Collision:** If two keys land in the same bucket, they form a Linked List.
4. **Treeification (Java 8+):** If a bucket's linked list exceeds 8 nodes (and table capacity $\ge$ 64), it converts into a Red-Black Tree, improving worst-case search from $O(N)$ to $O(\log N)$.

## 4. Why is it designed this way?
- **Power of 2 Capacity:** Allows replacing expensive modulo `%` operations with fast bitwise AND `&`.
- **Treeification:** Protects against Hash DoS attacks where attackers deliberately feed keys that produce the same hash code, degrading server performance.

## 5. Common Mistakes
- **Mutable Keys:** If a key's fields are modified after it's added to the map, its `hashCode` changes. You will never be able to retrieve it! Always use immutable objects (like `String`) as keys.
- **Forgetting to override hashCode() when overriding equals():** Breaks the contract. Two "equal" objects will hash to different buckets.

## 6. Interview Questions

### SDE-1
**Q: What is the difference between HashMap and ConcurrentHashMap?**
**Short Answer:** HashMap is not thread-safe. ConcurrentHashMap is thread-safe and highly concurrent.

### SDE-2
**Q: What exactly happens when a HashMap resizes?**
**Detailed Answer:** When `size > capacity * loadFactor` (default 0.75), capacity doubles. A new array is allocated. Every entry is rehashed. However, because capacity is always a power of 2, Java 8 optimizes this: elements either stay at their current index `i`, or move to `i + oldCapacity`. No full re-calculation of the hash is needed.

**Q: Why does ConcurrentHashMap not allow null keys or values, while HashMap does?**
**Detailed Answer:** To avoid the *ambiguity problem* in concurrent environments. In HashMap, if `get(key)` returns `null`, it could mean the key isn't there, or the key is mapped to `null`. You can check with `containsKey(key)`. In a concurrent environment, the map could change between `get()` and `containsKey()`, leading to race conditions.

---

[← Previous: List, Set, Map Deep Dive](15-list-set-map.md) | [Next: JVM Memory & GC →](17-jvm-memory-and-gc.md)
