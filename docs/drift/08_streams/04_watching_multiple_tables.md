# Watching Multiple Tables

**Watching multiple tables** allows a single reactive query to automatically update when changes occur in any of the tables it depends on.

---

# What is it?

Real-world applications often need data from more than one table. For example:

* Users and their posts
* Orders and customers
* Tasks and categories
* Products and reviews

When a query joins multiple tables, Drift automatically tracks all of them. If any of the involved tables change in a way that affects the query result, the query is re-executed and the stream emits the updated data.

You don't need to manually specify which tables to watch—Drift determines this from the query itself.

---

# Why does it exist?

Imagine a screen showing each task along with its category name.

The query depends on both the `tasks` table and the `categories` table.

Without automatic tracking, you would need to manually refresh the UI whenever either table changed.

Drift solves this by monitoring every table used by the query.

This means:

* Updating a task refreshes the stream.
* Renaming a category refreshes the stream.
* Deleting a category refreshes the stream if it affects the query.

The UI always stays synchronized with the database.

---

# Syntax

```dart
final stream = (select(tasks)
      ..join([
        innerJoin(
          categories,
          categories.id.equalsExp(tasks.categoryId),
        ),
      ]))
    .watch();
```

Explanation:

* The query joins the `tasks` and `categories` tables.
* Drift watches both tables automatically.
* Changes to either table may trigger a new stream event.

---

# Mental Model

Think of Drift building a dependency graph.

```text
Tasks Table ─────┐
                 │
                 ▼
           Joined Query
                 ▲
                 │
Categories Table ┘
                 │
                 ▼
              Stream
                 │
                 ▼
             Flutter UI
```

Whenever either table changes:

```text
Tasks updated
        │
        ▼
Query runs again

OR

Categories updated
        │
        ▼
Query runs again
```

You don't subscribe to individual tables—only to the query.

---

# Examples

## Simple Example

Watch tasks together with their category.

```dart
Stream<List<TypedResult>> watchTasksWithCategories() {
  return (select(tasks)
        ..join([
          innerJoin(
            categories,
            categories.id.equalsExp(tasks.categoryId),
          ),
        ]))
      .watch();
}
```

Explanation:

* The stream emits joined results.
* Updates to either table automatically refresh the query.

---

Rename a category.

```dart
await (update(categories)
      ..where((c) => c.id.equals(1)))
    .write(
  const CategoriesCompanion(
    name: Value('Work'),
  ),
);
```

Explanation:

* Even though the `tasks` table didn't change, the stream emits updated results because the joined data changed.

---

## Real-World Example

A shopping app displays products along with their brand names.

```dart
Stream<List<TypedResult>> watchProducts() {
  return (select(products)
        ..join([
          innerJoin(
            brands,
            brands.id.equalsExp(products.brandId),
          ),
        ]))
      .watch();
}
```

Explanation:

* The query depends on both `products` and `brands`.
* Changing a product or updating a brand name refreshes the stream.

---

Update a brand.

```dart
await (update(brands)
      ..where((b) => b.id.equals(brandId)))
    .write(
  BrandsCompanion(
    name: Value('OpenAI Hardware'),
  ),
);
```

Explanation:

* Product rows remain the same.
* The joined result changes because the brand name changed.
* Drift automatically emits the updated data.

---

# When to Use

Use queries that watch multiple tables when:

* Displaying joined data
* Showing relational information
* Building dashboards
* Displaying parent-child relationships
* Combining data from several tables
* Building reports that depend on multiple entities

---

# When NOT to Use

Avoid joining multiple tables when:

* Data comes from only one table.
* A simple query is sufficient.
* The joined data isn't needed.
* Performance is better served by separate queries.

Don't join tables just because they are related—join them only when the UI needs data from both.

---

# Best Practices

* Join only the tables required for the query.
* Filter data as early as possible.
* Prefer SQL joins over manually combining results in Dart.
* Keep joined queries focused and readable.
* Let Drift automatically manage table dependencies.

---

# Common Mistakes

## Manually Refreshing After Updating Another Table

Wrong:

```dart
await updateCategory();

await reloadTasks();
```

Explanation:

* A watched joined query refreshes automatically.

Correct:

```dart
await updateCategory();
```

Explanation:

* Drift detects the change and emits updated results.

---

## Running Multiple Queries Instead of a Join

Wrong:

```dart
final tasks = await select(tasks).get();
final categories = await select(categories).get();
```

Explanation:

* Multiple queries require manual mapping and synchronization.

Correct:

```dart
(select(tasks)
      ..join([
        innerJoin(
          categories,
          categories.id.equalsExp(tasks.categoryId),
        ),
      ]))
    .watch();
```

Explanation:

* Drift performs the join and keeps the combined result reactive.

---

## Assuming Only the Primary Table Is Watched

Wrong assumption:

> "Only changes to `tasks` will refresh this query."

Explanation:

* Drift tracks every table referenced by the query.

Correct understanding:

* Any relevant change in `tasks` or `categories` causes the query to be evaluated again.

---

# Related APIs

* `join()`
* `innerJoin()`
* `leftOuterJoin()`
* `watch()`
* `TypedResult`
* Reactive Queries
* Stream Updates

---

# Summary

When a reactive query involves multiple tables, Drift automatically watches all of them. Any relevant change to any referenced table causes the query to run again and emit updated results. This makes it easy to build reactive UIs that display relational data without writing manual synchronization logic.
