# Stream Updates

**Stream Updates** are the automatic notifications Drift sends to query streams whenever relevant database changes occur.

---

# What is it?

One of Drift's most powerful features is its ability to keep query results synchronized with the database. When you watch a query, Drift continuously monitors the tables involved. If data affecting the query changes, Drift automatically re-executes the query and emits the updated result through the stream.

You don't manually notify listeners or refresh data. Drift handles update detection for you.

For example, if your app watches all users and a new user is inserted, the stream automatically emits the updated list without additional code.

---

# Why does it exist?

Without automatic stream updates, applications typically need to:

- Reload data after every insert
- Refresh the UI manually
- Track which widgets should rebuild
- Handle synchronization logic themselves

This quickly becomes error-prone as applications grow.

Drift solves this by automatically detecting changes to tables used by watched queries and re-running only the affected queries.

This makes building reactive applications much simpler.

---

# Syntax

```dart
final usersStream = select(users).watch();
```

Explanation:

- `watch()` creates a stream that automatically updates when the query result changes.

---

```dart
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    age: 24,
  ),
);
```

Explanation:

- Inserting into the `users` table triggers every stream watching queries that depend on `users`.

---

```dart
await (update(users)..where((u) => u.id.equals(1))).write(
  UsersCompanion(
    age: Value(25),
  ),
);
```

Explanation:

- Updating matching rows causes affected query streams to emit fresh results automatically.

---

```dart
await (delete(users)..where((u) => u.id.equals(1))).go();
```

Explanation:

- Deleting rows also triggers updates for streams watching the affected table.

---

# Mental Model

Think of Drift as maintaining a dependency graph between queries and tables.

```
Users Table
     │
     │
     ▼
select(users).watch()
     │
     ▼
 Stream
     │
     ▼
 Flutter UI
```

Whenever the **Users** table changes:

```
Insert
Update
Delete
      │
      ▼
Drift detects change
      │
      ▼
Query runs again
      │
      ▼
New result emitted
      │
      ▼
UI rebuilds automatically
```

You only write the query once. Drift handles keeping it up to date.

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

- Every modification to the `users` table produces a new list of users.

---

Insert a new user.

```dart
await into(users).insert(
  UsersCompanion.insert(
    name: 'John',
    age: 30,
  ),
);
```

Explanation:

- The stream automatically emits the updated list containing John.

---

## Real-World Example

A task management app displays pending tasks.

```dart
Stream<List<Task>> watchPendingTasks() {
  return (select(tasks)
        ..where((t) => t.completed.equals(false)))
      .watch();
}
```

Explanation:

- The stream watches only pending tasks.
- Drift re-executes the filtered query whenever the `tasks` table changes.

---

Mark a task as completed.

```dart
await (update(tasks)
      ..where((t) => t.id.equals(taskId)))
    .write(
  const TasksCompanion(
    completed: Value(true),
  ),
);
```

Explanation:

- The updated task no longer matches the filter.
- Drift reruns the query.
- The completed task disappears from the emitted list automatically.
- The UI updates without calling setState or manually refreshing data.

---

# When to Use

Use stream updates when:

- Building reactive Flutter UIs
- Displaying lists that should stay synchronized
- Showing dashboards with live data
- Monitoring frequently changing tables
- Eliminating manual refresh logic

---

# When NOT to Use

Avoid relying on stream updates when:

- You only need data once
- The screen doesn't require live updates
- Performing one-time background processing
- Exporting reports or generating snapshots

In these situations, prefer `get()` instead of `watch()`.

---

# Best Practices

- Watch only the queries your UI actually needs.
- Keep watched queries as specific as possible.
- Use filters to reduce unnecessary updates.
- Cancel subscriptions when they are no longer needed.
- Let Drift manage updates instead of implementing manual refresh logic.

---

# Common Mistakes

## Watching an Entire Table Unnecessarily

Wrong:

```dart
select(users).watch();
```

Explanation:

- Every change to the table causes the full query to rerun, even if only a subset is needed.

Correct:

```dart
(select(users)
      ..where((u) => u.active.equals(true)))
    .watch();
```

Explanation:

- Watch only the data required by the UI.

---

## Refreshing Data Manually

Wrong:

```dart
await insertUser();

setState(() {
  loadUsers();
});
```

Explanation:

- Manual refresh defeats the purpose of reactive queries.

Correct:

```dart
await insertUser();
```

Explanation:

- The watched stream automatically emits the updated data.

---

## Expecting Updates from Non-Watched Queries

Wrong:

```dart
final users = await select(users).get();
```

Explanation:

- `get()` returns a snapshot and never updates automatically.

Correct:

```dart
final users = select(users).watch();
```

Explanation:

- `watch()` provides continuous updates whenever data changes.

---

# Related APIs

- `watch()`
- `get()`
- `watchSingle()`
- `watchSingleOrNull()`
- `StreamBuilder`
- Reactive Queries
- Watching Multiple Tables

---

# Summary

Stream updates are the mechanism that makes Drift reactive. Whenever data changes in tables used by a watched query, Drift automatically detects the change, re-executes the query, and emits fresh results through the stream. This eliminates manual refresh logic and keeps Flutter UIs synchronized with the database with minimal code.