# ref.onDispose()

`ref.onDispose()` registers a callback that is executed immediately before a provider is permanently disposed.

---

## What is it?

`ref.onDispose()` is a lifecycle API used to **clean up resources** when a provider is destroyed.

When Riverpod disposes a provider, it automatically executes every callback registered with `ref.onDispose()`.

This is commonly used to release resources such as:

- `StreamController`
- `StreamSubscription`
- `Timer`
- Database connections
- File handles
- Socket connections
- Animation controllers (outside widgets)

Think of it as the provider equivalent of Flutter's `dispose()` method.

---

## Why does it exist?

Providers often create resources that continue running even after the provider is no longer needed.

For example:

- A `Timer` keeps ticking.
- A `StreamSubscription` continues listening.
- A socket remains connected.
- A database connection stays open.

Without cleanup, these resources can cause:

- Memory leaks
- Background work
- Unexpected callbacks
- Wasted battery and CPU usage

`ref.onDispose()` ensures resources are cleaned up automatically when the provider's lifecycle ends.

---

## Availability

✅ Available in **all provider types**.

It can be used inside:

- `Provider`
- `FutureProvider`
- `StreamProvider`
- `Notifier`
- `AsyncNotifier`
- `StreamNotifier`

It is especially useful with `.autoDispose` providers, where disposal happens more frequently.

---

## Syntax

### Closing a StreamController

```dart
final provider = Provider((ref) {
  final controller = StreamController<int>();

  ref.onDispose(() {
    controller.close();
  });

  return controller;
});
```

Explanation:

- A `StreamController` is created.
- When the provider is disposed, the controller is closed automatically.

---

### Cancelling a Timer

```dart
final timer = Timer.periodic(
  const Duration(seconds: 1),
  (_) {},
);

ref.onDispose(() {
  timer.cancel();
});
```

Explanation:

- Prevents the timer from continuing after disposal.

---

### Cancelling a StreamSubscription

```dart
final subscription = stream.listen(print);

ref.onDispose(() {
  subscription.cancel();
});
```

Explanation:

- Stops listening to the stream.
- Prevents memory leaks.

---

## Return Value

`ref.onDispose()` returns `void`.

It simply registers one or more cleanup callbacks.

You can register multiple callbacks.

```dart
ref.onDispose(() {
  timer.cancel();
});

ref.onDispose(() {
  controller.close();
});
```

Riverpod executes all registered callbacks during disposal.

---

## Execution Flow

```text
Provider Created
       │
       ▼
Resource Allocated
       │
       ▼
ref.onDispose(callback)
       │
       ▼
Provider Disposed
       │
       ▼
Callback Executes
       │
       ▼
Resources Released
```

Cleanup happens exactly once, immediately before the provider is destroyed.

---

## Mental Model

Think of `ref.onDispose()` as the provider's **cleanup checklist**.

```text
Provider Starts
      │
      ▼
Create Resources
      │
      ▼
Register Cleanup
      │
      ▼
Provider Ends
      │
      ▼
Cleanup Runs Automatically
```

You're telling Riverpod:

> "Before destroying me, clean up these resources."

---

## Examples

### Close a StreamController

```dart
final controller = StreamController<String>();

ref.onDispose(() {
  controller.close();
});
```

Explanation:

- Prevents resource leaks.
- Frees the controller correctly.

---

### Cancel a Timer

```dart
final timer = Timer.periodic(
  const Duration(seconds: 1),
  (_) {},
);

ref.onDispose(timer.cancel);
```

Explanation:

- Stops periodic execution.
- Saves CPU and battery.

---

### Dispose a WebSocket

```dart
final socket = WebSocket.connect(url);

ref.onDispose(() {
  socket.close();
});
```

Explanation:

- Closes the network connection.
- Prevents background activity.

---

### Real-World Example

```dart
final chatProvider =
    StreamProvider.autoDispose<Message>((ref) {
  final socket = ChatSocket();

  ref.onDispose(() {
    socket.disconnect();
  });

  return socket.messages;
});
```

Explanation:

- Connects when the provider starts.
- Disconnects automatically when disposed.

---

## When to Use

Use `ref.onDispose()` when:

- Closing streams.
- Cancelling timers.
- Releasing database connections.
- Disconnecting sockets.
- Cleaning up external resources.
- Preventing memory leaks.

---

## When NOT to Use

Avoid `ref.onDispose()` when:

- No cleanup is required.
- Updating provider state.
- Triggering UI changes.
- Performing business logic unrelated to disposal.

It should only be used for **resource cleanup**.

---

## Best Practices

- Register cleanup immediately after creating a resource.
- Keep disposal callbacks small and focused.
- Register separate callbacks for unrelated resources.
- Always clean up external resources.
- Prefer automatic cleanup over manual cleanup.

---

## Common Mistakes

### 1. Forgetting to Close a StreamController

❌ Wrong

```dart
final controller = StreamController<int>();

return controller;
```

Why it's wrong:

- The controller remains in memory.
- Can cause resource leaks.

✔ Correct

```dart
final controller = StreamController<int>();

ref.onDispose(() {
  controller.close();
});
```

---

### 2. Forgetting to Cancel a Timer

❌ Wrong

```dart
Timer.periodic(
  const Duration(seconds: 1),
  (_) {},
);
```

Why it's wrong:

- Timer continues after the provider is gone.

✔ Correct

```dart
final timer = Timer.periodic(
  const Duration(seconds: 1),
  (_) {},
);

ref.onDispose(timer.cancel);
```

---

### 3. Performing Business Logic During Disposal

❌ Wrong

```dart
ref.onDispose(() {
  repository.saveUser();
});
```

Why it's wrong:

- Disposal should clean up resources, not perform application logic.
- The callback should execute quickly.

✔ Correct

Use `onDispose()` only for cleanup.

---

### 4. Assuming onDispose() Runs Frequently

❌ Wrong

```dart
ref.onDispose(() {
  print('Cleaning...');
});
```

Expecting it to run every rebuild.

Why it's wrong:

- It runs **only when the provider is actually disposed**, not on rebuilds.

✔ Correct

Use `build()` or state updates for logic that should run during recomputation.

---

## ref.onDispose() vs Flutter's dispose()

| Feature | `ref.onDispose()` | `State.dispose()` |
|----------|-------------------|-------------------|
| Belongs to | Provider | Widget |
| Cleans provider resources | ✅ | ❌ |
| Cleans widget resources | ❌ | ✅ |
| Triggered by provider lifecycle | ✅ | ❌ |
| Triggered by widget lifecycle | ❌ | ✅ |

---

## Related APIs

- `ref.onCancel()`
- `ref.onResume()`
- `ref.keepAlive()`
- `.autoDispose`
- `Ref Lifecycle`

---

## Summary

`ref.onDispose()` registers cleanup callbacks that execute when a provider is permanently disposed. It is the preferred way to release external resources such as timers, stream subscriptions, controllers, sockets, and database connections. Proper use of `ref.onDispose()` helps prevent memory leaks and ensures providers clean up after themselves automatically.