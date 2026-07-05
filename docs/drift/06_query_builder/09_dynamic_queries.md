# Dynamic Queries

Dynamic queries allow you to build SQL queries at runtime based on conditions, user input, or application state.

---

# What is it?

Not every query is known in advance.

For example, a search screen may allow users to:

* Search by name.
* Filter by category.
* Set a price range.
* Choose a sort order.

Since these options can change, the query must be built dynamically.

Drift's query builder makes it easy to add filters, ordering, and limits only when they're needed.

---

# Why does it exist?

Imagine an e-commerce application.

Sometimes users search by:

```text
Category only
```

Other times:

```text
Category
+
Price
```

Or perhaps:

```text
Category
+
Price
+
Rating
+
Availability
```

Writing separate queries for every possible combination quickly becomes unmanageable.

Dynamic queries let you build a single query that adapts to the available filters.

---

# Syntax

## Building a Query Incrementally

```dart
final query = select(products);

if (category != null) {
  query.where((p) => p.category.equals(category));
}

if (maxPrice != null) {
  query.where(
    (p) => p.price.isSmallerOrEqualValue(maxPrice),
  );
}

final results = await query.get();
```

Explanation:

* The query starts with `select(products)`.
* Conditions are added only when needed.
* `get()` executes the final query.

---

## Dynamic Ordering

```dart
final query = select(products);

if (sortByPrice) {
  query.orderBy([
    (p) => OrderingTerm.asc(p.price),
  ]);
} else {
  query.orderBy([
    (p) => OrderingTerm.asc(p.name),
  ]);
}

final results = await query.get();
```

Explanation:

* The ordering depends on runtime conditions.
* Only one ordering strategy is applied.

---

# Mental Model

Think of a dynamic query as building a sandwich.

```text
Start

Bread

↓

Add filters?

✓ Category

↓

✓ Price

↓

✓ Rating

↓

Serve Query
```

Each optional ingredient adds another piece to the final query.

---

# Examples

## Simple Example

Search for active users.

```dart
final query = select(users);

if (showActiveOnly) {
  query.where(
    (u) => u.isActive.equals(true),
  );
}

final usersList = await query.get();
```

Explanation:

* The filter is added only when required.
* Otherwise, all users are returned.

---

## Real-World Example

Product search.

```dart
final query = select(products);

if (category != null) {
  query.where(
    (p) => p.category.equals(category),
  );
}

if (minPrice != null) {
  query.where(
    (p) => p.price.isBiggerOrEqualValue(minPrice),
  );
}

query.orderBy([
  (p) => OrderingTerm.asc(p.name),
]);

query.limit(20);

final productsList = await query.get();
```

Explanation:

* The query adapts to the selected filters.
* Ordering and limiting are applied before execution.
* The same query builder supports many search combinations.

---

# When to Use

Use dynamic queries when:

* Building search screens.
* Creating filter panels.
* Supporting optional parameters.
* Implementing pagination.
* Building reusable repository methods.

---

# When NOT to Use

Avoid dynamic queries when:

* The query structure never changes.
* A fixed query is simpler and easier to understand.

For static queries, writing the conditions directly is often more readable.

---

# Best Practices

* Start with a base query and extend it.
* Add filters only when needed.
* Keep dynamic logic readable.
* Reuse query-building code where possible.
* Prefer Drift's query builder over manually constructing SQL strings.

---

# Common Mistakes

## Creating separate queries for every case

**Wrong**

```text
getProducts()

getProductsByCategory()

getProductsByPrice()

getProductsByCategoryAndPrice()

getProductsByCategoryAndPriceAndRating()
```

Explanation:

* The number of methods grows rapidly as filters increase.

**Correct**

Build one query and add conditions dynamically.

---

## Building raw SQL strings

**Wrong**

```dart
var sql = 'SELECT * FROM products';

if (category != null) {
  sql += " WHERE category = '$category'";
}
```

Explanation:

* Raw SQL is harder to maintain.
* It can introduce SQL injection risks if values aren't parameterized.

**Correct**

Use Drift's query builder and expressions to construct the query safely.

---

## Forgetting to execute the query

**Wrong**

```dart
final query = select(products);

// Add filters...
```

Explanation:

* Creating a query does not execute it.

**Correct**

```dart
final results = await query.get();
```

Explanation:

* `get()` executes the completed query and returns the results.

---

# Related APIs

* Select
* Filtering
* Ordering
* Limiting
* Expressions
* Custom SQL Expressions

---

# Summary

Dynamic queries allow you to build flexible SQL queries at runtime by adding filters, ordering, limits, and other clauses only when needed. They help avoid duplicate query code, simplify complex search features, and let you take full advantage of Drift's type-safe query builder without resorting to manually constructed SQL strings.
