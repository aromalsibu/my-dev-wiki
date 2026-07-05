# Stream Performance

**Stream performance** is about writing efficient reactive queries that minimize unnecessary work while keeping your UI synchronized with the database.

---

# What is it?

Reactive queries are one of Drift's most powerful features, but every watched query has a cost.

Whenever a relevant table changes, Drift:

1. Detects the change.
2. Re-executes the query.
3. Compares and emits the new result.
4. Notifies all stream listeners.

For small applications, this overhead is negligible. However, as your application grows and watches dozens of queries across many tables, poorly designed streams can lead to unnecessary database work and UI rebuilds.

Writing performant streams ensures your app remains responsive even as the database grows.

---

# Why does it exist?

Imagine a dashboard with 20 widgets, each watching the entire `orders` table.

Whenever a single order changes:

* Every query runs again.
* Every widget rebuilds.
* The database performs more work than necessary.

In contrast, if each widget watches only the data it needs, only the affected queries are re-executed.

The goal isn't to avoid reactive queries—it's to make them as focused as possible.

---

# Syntax

Watch only active users.

```dart
final activeUsers = (select(users)
      ..where((u) => u.isActive.equals(true)))
    .watch();
```

Explanation:

* The query watches only active users instead of the entire table.
* Smaller result sets generally mean less work for both the database and the UI.

---

Limit the number of rows.

```dart
final latestMessages = (select(messages)
      ..orderBy([
        (m) => OrderingTerm.desc(m.createdAt),
      ])
      ..limit(20))
    .watch();
```

Explanation:

* Only the latest 20 messages are returned.
* Avoids loading and rebuilding hundreds or thousands of rows.

---

Watch only a single row.

```dart
final profile = (select(users)
      ..where((u) => u.id.equals(userId)))
    .watchSingle();
```

Explanation:

* `watchSingle()` is ideal when you expect exactly one row.
* It avoids working with a list when only one object is needed.

---

# Mental Model

Think of every watched query as a worker.

```text
Database Change
       │
       ▼
 Watched Query
       │
       ▼
 Database Work
       │
       ▼
 Stream Event
       │
       ▼
 UI Rebuild
```

Now imagine watching the entire database.

```text
Database Change
       │
       ├─────────────┐
       ▼             ▼
Query A         Query B
       ▼             ▼
Query C         Query D
       ▼             ▼
Many UI rebuilds
```

The more queries you watch—and the broader they are—the more work Drift performs.

Efficient queries reduce unnecessary processing.

---

# Examples

## Simple Example

Instead of watching every task:

```dart
select(tasks).watch();
```

Explanation:

* Every task is loaded, even if the screen only needs pending tasks.

A better approach:

```dart
(select(tasks)
      ..where((t) => t.completed.equals(false)))
    .watch();
```

Explanation:

* The stream returns only pending tasks.
* Less data is processed and displayed.

---

## Real-World Example

Suppose you're building a messaging app.

The home screen shows only the latest conversation from each chat.

```dart
Stream<List<Message>> watchRecentMessages() {
  return (select(messages)
        ..orderBy([
          (m) => OrderingTerm.desc(m.sentAt),
        ])
        ..limit(30))
      .watch();
}
```

Explanation:

* Only recent messages are watched.
* The UI doesn't need to process the user's entire message history.

If the user opens a conversation, create a separate stream for that conversation instead of watching every message in the database.

---

# When to Use

Optimize stream performance when:

* Your app has many reactive screens.
* Large tables are frequently updated.
* Queries return hundreds or thousands of rows.
* Multiple widgets listen to database streams.
* You notice excessive UI rebuilds or slow database operations.

---

# When NOT to Use

Don't over-optimize prematurely.

For small applications with a handful of streams, simple reactive queries are usually sufficient.

Measure performance first before introducing complex optimizations.

---

# Best Practices

* Watch only the data your UI needs.
* Filter results whenever possible.
* Use `limit()` for large datasets.
* Prefer `watchSingle()` for single-row queries.
* Dispose of stream subscriptions when they're no longer needed.
* Avoid creating duplicate streams for the same data.
* Let SQL perform filtering and sorting instead of doing it in Dart.

---

# Common Mistakes

## Watching an Entire Table

Wrong:

```dart
select(messages).watch();
```

Explanation:

* Loads every message, even if only recent ones are displayed.

Correct:

```dart
(select(messages)
      ..orderBy([
        (m) => OrderingTerm.desc(m.sentAt),
      ])
      ..limit(20))
    .watch();
```

Explanation:

* Only the required data is watched.

---

## Filtering After Receiving Data

Wrong:

```dart
final tasks = await select(tasks).watch();

tasks.where((task) => !task.completed);
```

Explanation:

* The database sends every row before filtering.

Correct:

```dart
(select(tasks)
      ..where((t) => t.completed.equals(false)))
    .watch();
```

Explanation:

* SQLite filters the data before it's returned.

---

## Creating Duplicate Streams

Wrong:

```dart
WidgetA -> select(users).watch()

WidgetB -> select(users).watch()

WidgetC -> select(users).watch()
```

Explanation:

* Multiple identical streams perform the same query repeatedly.

Correct:

* Share a single stream through your repository, DAO, or state management solution whenever appropriate.

---

# Related APIs

* `watch()`
* `watchSingle()`
* `limit()`
* `orderBy()`
* `where()`
* Reactive Queries
* Watching Queries

---

# Summary

Reactive queries are efficient by design, but their performance depends on how they're used. Watch only the data your UI needs, filter and limit results in SQL, avoid duplicate streams, and use specialized APIs like `watchSingle()` when appropriate. Well-designed streams keep your application responsive while fully leveraging Drift's reactive capabilities.
