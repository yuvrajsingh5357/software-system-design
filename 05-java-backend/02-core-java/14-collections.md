# Java Collections Framework

## 1. What Is It?
The Java Collections Framework is a unified architecture for storing, manipulating, and retrieving groups of objects. It provides interfaces (like `List`, `Set`, `Map`) and concrete implementations (like `ArrayList`, `HashSet`, `HashMap`).

## 2. Collection vs Collections
- **`Collection`:** An **interface** that defines the basic structure and methods (like `add`, `remove`, `size`) for a group of elements. Interfaces like `List`, `Set`, and `Queue` extend `Collection`. (`Map` is part of the framework but does not extend `Collection`).
- **`Collections`:** A **utility class** consisting entirely of static methods that operate on or return collections (like `Collections.sort()`, `Collections.reverse()`, `Collections.synchronizedList()`).

## 3. Fail-Fast vs Fail-Safe Iterators
When iterating over a collection, what happens if another thread (or even the same thread outside the iterator) modifies the collection (adds or removes an element)?

- **Fail-Fast Iterators:**
  - They immediately throw a `ConcurrentModificationException`.
  - They work by reading the original collection directly and checking a `modCount` variable.
  - Examples: Iterators for `ArrayList`, `HashMap`, `HashSet`.
  - Memory: Highly efficient (no extra memory needed).

- **Fail-Safe Iterators:**
  - They do **not** throw an exception. They allow concurrent modification.
  - They work by creating a **clone/copy** of the underlying data structure and iterating over that snapshot.
  - Examples: Iterators for `ConcurrentHashMap`, `CopyOnWriteArrayList`.
  - Memory: Requires extra memory and processing to clone the collection.

## 4. Comparable vs Comparator (Sorting)
To sort custom objects (like a `User` class) in a collection, Java needs to know *how* to compare them.

- **`Comparable` (Natural Ordering):**
  - The class itself implements `Comparable<T>` and overrides the `compareTo(Object o)` method.
  - Modifies the source class.
  - "The object compares **itself** with another object."
  - Provides exactly one default sorting logic (e.g., sorting Users by ID).
  
- **`Comparator` (Custom Ordering):**
  - A separate class implements `Comparator<T>` and overrides `compare(Object o1, Object o2)`.
  - Does not modify the source class.
  - "A **third object** compares two other objects."
  - Provides flexibility. You can create multiple comparators (e.g., sort Users by Name, sort by Age, sort by Registration Date) and pass them to `Collections.sort(list, myComparator)`.

## 5. Interview Questions

### Q. What is the difference between Fail-Fast and Fail-Safe iterators?
#### Short Interview Answer
A fail-fast iterator operates directly on the original collection and immediately throws a `ConcurrentModificationException` if the collection is structurally modified during iteration. A fail-safe iterator operates on a clone or snapshot of the collection, so it does not throw an exception if the original collection is modified, but it requires extra memory overhead.

### Q. When would you choose Comparator over Comparable?
#### Short Interview Answer
I would use `Comparable` to define the default, natural sorting order for a class (like sorting an `Employee` by employee ID). I would use `Comparator` when I need to define multiple different sorting strategies (like sorting by name, then by salary), or when I need to sort a class that I do not own and cannot modify to implement `Comparable`.

---

[← Previous: Exception Handling](13-exceptions.md) | [Next: List, Set, Map Deep Dive →](15-list-set-map.md)
