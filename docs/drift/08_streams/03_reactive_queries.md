# Reactive Queries

**Reactive queries** are queries that automatically emit updated results whenever the underlying database data changes.

---

# What is it?

A reactive query is a query that stays "alive" after it's executed. Instead of returning data once, it continuously watches the tables involved and emits new results whenever the query output changes.

In Drift, any query created with `watch()` becomes reactive.

For example, if you're displaying a list of products, you don't need to manually reload the list after inserting, updating, or deleting a product. The query automatically re-runs, and the UI receives the latest data.

---

# Why does it exist?

Traditional database queries return a snapshot of the data at a single point in time.

```text
Database → Query → Result
```

If the data changes later, your application has to fetch everything again.

Reactive queries eliminate this problem by continuously observing the database.

```text
Database
    │
    ▼
Reactive Query
    │
    ▼
Stream
    │
    ▼
UI
```

Whenever relevant data changes, Drift automatically:

* Detects the change
* Re-executes the query
* Emits the new result
* Keeps your UI synchronized

This makes building real-time and reactive applications much simpler.

---

# Syntax

```dart
final usersStream = select(users).watch();
```

Explanation:

* `watch()` creates a reactive query.
* The returned stream emits new results whenever the query result changes.

---

```dart
final userStream = (select(users)
      ..where((u) => u.id.equals(1)))
    .watchSingle();
```

Explanation:

* `watchSingle()` watches a query expected to return exactly one row.
* The stream emits a new user whenever that row changes.

---

```dart
final taskCount = (selectOnly(tasks)
      ..addColumns([tasks.id.count()]))
    .watchSingle();
```

Explanation:

* Aggregate queries can also be reactive.
* The count updates automatically whenever tasks are inserted or deleted.

---

# Mental Model

Think of a reactive query as a live subscription to your database.

```
Flutter Widget
       │
       ▼
 Reactive Query
       │
       ▼
    Database
```

Whenever the database changes:

```
Insert
Update
Delete
     │
     ▼
 Drift detects change
     │
     ▼
 Query executes again
     │
     ▼
 New data emitted
     │
     ▼
 UI rebuilds
```

Unlike one-time queries, reactive queries never become "stale" while the stream is active.

---

# Examples

## Simple Example

Watch all users.

```dart
Stream<List<User>> watchUsers() {
  return select(users).watch();
}
```

Explanation:

* The stream emits a new list whenever the `users` table changes.

---

Display the stream in Flutter.

```dart
StreamBuilder<List<User>>(
  stream: database.watchUsers(),
  builder: (context, snapshot) {
    final users = snapshot.data ?? [];

    return ListView.builder(
      itemCount: users.length,
      itemBuilder: (_, index) {
        return ListTile(
          title: Text(users[index].name),
        );
      },
    );
  },
);
```

Explanation:

* `StreamBuilder` rebuilds automatically whenever the query emits updated data.

---

## Real-World Example

Suppose you're building a shopping app that displays only items currently in the cart.

```dart
Stream<List<CartItem>> watchCartItems() {
  return (select(cartItems)
        ..where((t) => t.quantity.isBiggerThanValue(0)))
      .watch();
}
```

Explanation:

* Only items with a quantity greater than zero are returned.
* Drift watches the query and automatically updates it whenever cart data changes.

---

Increase the quantity of an item.

```dart
await (update(cartItems)
      ..where((t) => t.id.equals(itemId)))
    .write(
  CartItemsCompanion(
    quantity: Value(newQuantity),
  ),
);
```

Explanation:

* Drift detects the update.
* The query executes again.
* The shopping cart UI refreshes automatically.

---

# When to Use

Use reactive queries when:

* Displaying lists in Flutter
* Showing dashboards
* Building chat applications
* Displaying shopping carts
* Monitoring notifications
* Showing statistics that change frequently
* Building live data screens

---

# When NOT to Use

Reactive queries are unnecessary when:

* Loading data only once
* Exporting reports
* Running background jobs
* Performing database migrations
* Fetching configuration during app startup

In these cases, use `get()` instead of `watch()`.

---

# Best Practices

* Use reactive queries for UI state.
* Keep watched queries focused and efficient.
* Apply filters to avoid unnecessary work.
* Watch only the data your screen actually needs.
* Dispose stream subscriptions when they are no longer required.
* Prefer `watchSingle()` when expecting a single row.

---

# Common Mistakes

## Using `get()` for Live Data

Wrong:

```dart
final users = await select(users).get();
```

Explanation:

* This returns only a single snapshot.

Correct:

```dart
final users = select(users).watch();
```

Explanation:

* The query stays reactive and continues emitting updates.

---

## Watching More Data Than Necessary

Wrong:

```dart
select(tasks).watch();
```

Explanation:

* Every change to the table causes the entire query to re-run.

Correct:

```dart
(select(tasks)
      ..where((t) => t.completed.equals(false)))
    .watch();
```

Explanation:

* Watch only the subset your UI actually displays.

---

## Reloading Data After Every Change

Wrong:

```dart
await addTask();

await loadTasks();
```

Explanation:

* Manual reloads duplicate work already handled by Drift.

Correct:

```dart
await addTask();
```

Explanation:

* The reactive query automatically emits the updated task list.

---

# Related APIs

* `watch()`
* `watchSingle()`
* `watchSingleOrNull()`
* `get()`
* `StreamBuilder`
* Watching Queries
* Stream Updates

---

# Summary

Reactive queries are one of Drift's core features. Instead of returning data once, they continuously observe the database and automatically emit updated results whenever the underlying data changes. They eliminate manual refresh logic, keep Flutter UIs synchronized with the database, and make building responsive, real-time applications much easier.
