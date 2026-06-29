# StreamProvider

`StreamProvider` is a provider that exposes values from a `Stream` as an `AsyncValue`, making it ideal for continuous asynchronous data updates.

---

## What is it?

Unlike `FutureProvider`, which produces a single value, `StreamProvider` listens to a `Stream` and emits new values whenever they become available.

Each emitted value automatically updates every consumer watching the provider.

Common use cases include:

- Chat messages
- Live notifications
- Firebase Firestore
- WebSocket connections
- Sensor data
- Location updates
- Stock prices
- Real-time dashboards

Like `FutureProvider`, `StreamProvider` exposes its state through an `AsyncValue`.

---

## Why does it exist?

Managing streams manually requires:

- Creating subscriptions
- Handling loading state
- Handling errors
- Cancelling subscriptions
- Preventing memory leaks
- Updating the UI

`StreamProvider` automates all of these tasks.

It provides:

- Automatic stream subscriptions
- Automatic cancellation
- Loading, data, and error states
- Shared subscriptions
- Lifecycle management
- Caching of the latest emitted value

This allows you to focus on consuming the stream rather than managing it.

---

## Syntax

### Creating a StreamProvider

```dart
final counterProvider = StreamProvider<int>((ref) async* {
  for (var i = 0; i < 5; i++) {
    await Future.delayed(const Duration(seconds: 1));
    yield i;
  }
});
```

Explanation:

- `StreamProvider<T>` manages a `Stream<T>`.
- `yield` emits values to listeners.
- Each emitted value updates consumers automatically.

---

### Reading the Provider

```dart
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counter = ref.watch(counterProvider);

    return counter.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text(error.toString()),
      data: (value) => Text('$value'),
    );
  }
}
```

Explanation:

- `ref.watch()` returns an `AsyncValue<int>`.
- `when()` handles loading, data, and error states.

---

### Auto Dispose Stream

```dart
final messagesProvider =
    StreamProvider.autoDispose<Message>((ref) {
  return repository.messages();
});
```

Explanation:

- The stream subscription is cancelled automatically when unused.
- Ideal for screen-specific streams.

---

## Mental Model

Think of `StreamProvider` as a live pipeline.

```text
           Stream
              │
              ▼
      StreamProvider
              │
              ▼
        AsyncValue
     ┌──────┼──────┐
     ▼      ▼      ▼
Loading  Data  Error
              │
              ▼
     New Stream Event
              │
              ▼
        Updated Data
```

Every new stream event updates the provider and rebuilds dependent widgets.

---

## Examples

### Timer

```dart
final timerProvider = StreamProvider<int>((ref) async* {
  var count = 0;

  while (true) {
    await Future.delayed(const Duration(seconds: 1));
    yield count++;
  }
});
```

Explanation:

- Emits a new value every second.
- The UI rebuilds for each emission.

---

### Firebase Firestore

```dart
final usersProvider =
    StreamProvider<List<User>>((ref) {
  return firestore.usersStream();
});
```

Explanation:

- Listens for real-time database updates.
- Every change updates the UI automatically.

---

### WebSocket

```dart
final chatProvider =
    StreamProvider<Message>((ref) {
  return socket.messages();
});
```

Explanation:

- Receives messages from a WebSocket.
- New messages are delivered as stream events.

---

### Location Updates

```dart
final locationProvider =
    StreamProvider<Location>((ref) {
  return locationService.locationStream();
});
```

Explanation:

- Continuously emits location changes.
- Useful for navigation or tracking applications.

---

## AsyncValue States

A `StreamProvider` always returns an `AsyncValue`.

```text
Start Listening
       │
       ▼
 AsyncLoading
       │
       ▼
First Event
       │
       ▼
 AsyncData
       │
       ▼
More Events
       │
       ▼
 AsyncData
       │
       ▼
Stream Error
       │
       ▼
 AsyncError
```

Unlike `FutureProvider`, a `StreamProvider` may emit **many** data events during its lifetime.

---

## Caching Behavior

Riverpod caches the **latest emitted value**.

```text
Stream
   │
   ▼
1 → 2 → 3 → 4
         │
         ▼
 Cached Value = 4
```

New consumers immediately receive the latest cached value instead of waiting for the next event.

---

## StreamProvider vs FutureProvider

| Feature | FutureProvider | StreamProvider |
|---------|----------------|----------------|
| Single result | ✅ | ❌ |
| Multiple values | ❌ | ✅ |
| Continuous updates | ❌ | ✅ |
| Returns `AsyncValue` | ✅ | ✅ |
| Loading state | ✅ | ✅ |
| Error handling | ✅ | ✅ |
| Shared subscriptions | N/A | ✅ |

---

## When to Use

Use `StreamProvider` for:

- Firebase Firestore
- WebSockets
- Live notifications
- Chat applications
- GPS updates
- Sensor readings
- Stock prices
- Real-time analytics
- Periodic timers

---

## When NOT to Use

Avoid `StreamProvider` when:

- Only one asynchronous value is needed.
- The data should be modified by the provider.
- Complex business logic is required.
- You need methods that update the stream's state.

Instead, use:

| Scenario | Recommended Provider |
|----------|----------------------|
| One-time async request | `FutureProvider` |
| Mutable async state | `AsyncNotifierProvider` |
| Continuous mutable state | `StreamNotifierProvider` |

---

## Best Practices

- Use `autoDispose` for screen-specific streams.
- Keep stream creation inside repositories or services.
- Handle loading and error states with `AsyncValue`.
- Avoid creating multiple providers for the same stream.
- Let Riverpod manage subscriptions instead of subscribing manually.

---

## Common Mistakes

### 1. Using FutureProvider for Continuous Data

❌ Wrong

```dart
final chatProvider =
    FutureProvider<List<Message>>((ref) async {
  return repository.messages();
});
```

Why it's wrong:

- `FutureProvider` only returns a single result.
- It does not listen for future updates.

✔ Correct

```dart
final chatProvider =
    StreamProvider<List<Message>>((ref) {
  return repository.messages();
});
```

---

### 2. Manually Listening to the Stream

❌ Wrong

```dart
repository.messages().listen((message) {
  // Update UI manually
});
```

Why it's wrong:

- Manual subscriptions require manual cleanup.
- Easy to introduce memory leaks.

✔ Correct

```dart
final messages = ref.watch(chatProvider);
```

Let Riverpod manage the subscription lifecycle.

---

### 3. Using StreamProvider for Mutable Business Logic

❌ Wrong

```dart
final todosProvider =
    StreamProvider<List<Todo>>((ref) {
  return todoRepository.todos();
});

// Add, edit, or delete todos inside widgets
```

Why it's wrong:

- `StreamProvider` is designed to consume streams.
- It is not intended to manage mutations.

✔ Correct

Use `StreamNotifierProvider` when your provider needs to both emit stream updates and expose business logic.

---

## Recommendation

> **Use `StreamProvider` for consuming existing streams.**
>
> If your provider needs to create, control, or mutate streamed state through methods, prefer `StreamNotifierProvider`.

---

## Related APIs

- FutureProvider
- StreamNotifierProvider
- AsyncNotifierProvider
- AsyncValue
- Provider
- ref.watch()
- ref.refresh()
- ref.invalidate()

---

## Summary

`StreamProvider` is Riverpod's provider for continuous asynchronous data. It listens to a `Stream`, exposes values as an `AsyncValue`, automatically manages subscriptions, caches the latest emitted value, and updates consumers whenever new events arrive. It is the preferred choice for real-time data sources such as Firebase, WebSockets, notifications, and live streams.