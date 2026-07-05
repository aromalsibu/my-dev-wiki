# Watching Queries

Watching queries allows you to receive a stream of query results that automatically updates whenever the underlying data changes.

---

# What is it?

Normally, a query is executed once using methods like `get()` or `getSingle()`. The returned data never changes unless you execute the query again.

A watched query behaves differently.

Instead of returning a single result, it returns a **Stream** that emits new results whenever Drift detects changes to the tables used by the query.

This is one of Drift's most powerful features for building reactive applications.

---

# Why does it exist?

Imagine a todo application.

A user adds a new task.

```text
Todo List

✓ Buy milk
✓ Study Drift
```

Another task is inserted.

```text
Todo List

✓ Buy milk
✓ Study Drift
✓ Walk the dog
```

Without watching the query:

* You would need to manually reload the list.

With a watched query:

* Drift automatically reruns the query.
* The stream emits the updated list.
* Your UI updates automatically.

---

# Syntax

## Watching All Rows

```dart
final taskStream = select(tasks).watch();
```

Explanation:

* `watch()` executes the query and returns a `Stream<List<Task>>`.
* Whenever the `tasks` table changes, a new list is emitted.

---

## Watching a Filtered Query

```dart
final completedTasks = (select(tasks)
      ..where((t) => t.completed.equals(true)))
    .watch();
```

Explanation:

* The stream only emits completed tasks.
* Drift automatically reapplies the filter whenever data changes.

---

# Mental Model

Think of a watched query as subscribing to a live feed.

```text
Database
    │
Insert
Update
Delete
    │
    ▼
Drift detects changes
    │
    ▼
Query runs again
    │
    ▼
New Stream Event
    │
    ▼
UI Rebuilds
```

Instead of asking the database for updates, the database notifies your application.

---

# Examples

## Simple Example

Watch all users.

```dart
final usersStream = select(users).watch();
```

Explanation:

* Every insert, update, or delete on the `users` table produces a new list.
* Widgets listening to the stream rebuild automatically.

---

## Real-World Example

Display a live list of pending tasks.

```dart
Stream<List<Task>> watchPendingTasks() {
  return (select(tasks)
        ..where((t) => t.completed.equals(false))
        ..orderBy([
          (t) => OrderingTerm.asc(t.title),
        ]))
      .watch();
}
```

Using the stream in Flutter:

```dart
StreamBuilder<List<Task>>(
  stream: database.watchPendingTasks(),
  builder: (context, snapshot) {
    final tasks = snapshot.data ?? [];

    return ListView.builder(
      itemCount: tasks.length,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text(tasks[index].title),
        );
      },
    );
  },
);
```

Explanation:

* `watch()` creates a reactive query.
* Whenever the `tasks` table changes, the stream emits an updated list.
* `StreamBuilder` rebuilds automatically with the latest data.

---

# When to Use

Use watched queries when:

* Displaying live lists.
* Building dashboards.
* Showing chat messages.
* Displaying notifications.
* Creating reactive Flutter UIs.
* Using Riverpod or Bloc with streams.

---

# When NOT to Use

Avoid watched queries when:

* You only need the data once.
* You're exporting data.
* You're running one-time reports.
* Automatic updates aren't necessary.

In these cases, `get()` is usually more appropriate.

---

# Best Practices

* Use `watch()` for UI that should stay in sync with the database.
* Apply filters before calling `watch()`.
* Prefer watching only the data you need.
* Cancel unused stream subscriptions.
* Return streams from DAOs instead of widgets.

---

# Common Mistakes

## Using `get()` when live updates are needed

**Wrong**

```dart
final tasks = await select(tasks).get();
```

Explanation:

* The data is loaded only once.
* Future database changes won't be reflected automatically.

**Correct**

```dart
final tasks = select(tasks).watch();
```

Explanation:

* The query stays active and emits updates whenever the data changes.

---

## Watching an entire table unnecessarily

**Wrong**

```dart
select(tasks).watch();
```

When the UI only displays incomplete tasks.

Explanation:

* Every change to the table triggers the stream.
* More data is processed than necessary.

**Correct**

```dart
(select(tasks)
      ..where((t) => t.completed.equals(false)))
    .watch();
```

Explanation:

* Only the required rows are observed and emitted.

---

## Creating a new stream repeatedly

**Wrong**

```dart
@override
Widget build(BuildContext context) {
  final stream = select(tasks).watch();
}
```

Explanation:

* A new stream is created every time the widget rebuilds.

**Correct**

Create the stream in a DAO, repository, or state management layer and reuse it.

---

# Related APIs

* Stream Updates
* Reactive Queries
* Watching Multiple Tables
* Stream Performance
* Select

---

# Summary

`watch()` turns a Drift query into a reactive stream that automatically emits updated results whenever the underlying tables change. It eliminates the need for manual refreshes, making it ideal for Flutter applications that display live data such as task lists, chats, notifications, and dashboards.
