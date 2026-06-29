# StreamNotifier

`StreamNotifier<T>` is Riverpod’s base class for creating and controlling streams with embedded business logic, exposed through a `StreamNotifierProvider`.

---

## What is it?

`StreamNotifier` is used to manage **stream-based state with full control over emission logic**.

It allows you to:

- Create streams from scratch
- Transform or control emitted values
- Expose methods that influence the stream
- Access dependencies via `ref`
- Handle lifecycle-aware streaming

Unlike `StreamProvider`, which only consumes an existing stream, `StreamNotifier` **owns the stream and its behavior**.

---

## Why does it exist?

In real applications, streams are rarely just passive data sources.

You often need to:

- Start/stop streams manually
- Apply filters or transformations
- Reconnect on failure
- Trigger events (send messages, refresh data)
- Combine multiple async sources

With `StreamProvider`, logic becomes external and scattered.

`StreamNotifier` solves this by centralizing:

- Stream creation
- Business logic
- Event control
- Dependency usage

It turns streaming into a **managed state system**, not just a data subscription.

---

## Syntax

### Creating a StreamNotifier

```dart
class CounterNotifier extends StreamNotifier<int> {
  @override
  Stream<int> build() async* {
    var i = 0;

    while (true) {
      await Future.delayed(const Duration(seconds: 1));
      yield i++;
    }
  }
}
```

Explanation:

- `build()` defines the stream source.
- `yield` emits values over time.
- The stream is automatically managed by Riverpod.

---

### Connecting with StreamNotifierProvider

```dart
final counterProvider =
    StreamNotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);
```

Explanation:

- Exposes the notifier.
- State is consumed as `AsyncValue<int>`.

---

### Reading State

```dart
final value = ref.watch(counterProvider);
```

Explanation:

- Returns `AsyncValue<int>`.
- Updates on every stream emission.

---

### Calling Methods

```dart
ref.read(counterProvider.notifier).restart();
```

Explanation:

- Accesses notifier instance.
- Allows controlling stream behavior.

---

## Mental Model

Think of `StreamNotifier` as a **stream engine with controls**.

```text
            Widget
               │
               ▼
  StreamNotifierProvider
               │
               ▼
        StreamNotifier
        ┌──────┴──────┐
        ▼             ▼
  Business Logic     Stream
                       │
                       ▼
                 AsyncValue
                       │
     ┌─────────┬─────────┬─────────┐
     ▼         ▼         ▼
 Loading     Data     Error
```

Unlike passive streams, this one is actively controlled.

---

## Examples

### Timer Stream

```dart
class TimerNotifier extends StreamNotifier<int> {
  @override
  Stream<int> build() async* {
    int seconds = 0;

    while (true) {
      await Future.delayed(const Duration(seconds: 1));
      yield seconds++;
    }
  }
}
```

Explanation:

- Emits a value every second.
- Automatically updates UI.

---

### Chat Messages

```dart
class ChatNotifier extends StreamNotifier<Message> {
  @override
  Stream<Message> build() {
    return chatRepository.messageStream();
  }

  Future<void> sendMessage(String text) async {
    await chatRepository.send(text);
  }
}
```

Explanation:

- Streams incoming messages.
- Provides method to send messages.

---

### Live Notifications

```dart
class NotificationNotifier
    extends StreamNotifier<Notification> {
  @override
  Stream<Notification> build() {
    return notificationService.stream();
  }

  void markAllAsRead() {
    notificationService.clear();
  }
}
```

Explanation:

- Streams notifications.
- Exposes actions for managing them.

---

## State Flow

```text
build() called
      │
      ▼
 AsyncLoading
      │
      ▼
 First event emitted
      │
      ▼
 AsyncData
      │
      ▼
 New event arrives
      │
      ▼
 AsyncData updated
      │
      ▼
 Error occurs
      │
      ▼
 AsyncError
```

The stream continuously updates state until disposed.

---

## When to Use

Use `StreamNotifier` when:

- You need a controlled stream
- You must expose actions (send, restart, reset)
- Stream depends on business logic
- You combine multiple streams
- You manage real-time systems (chat, notifications, tracking)

---

## When NOT to Use

Avoid `StreamNotifier` when:

- You only consume an existing stream → use `StreamProvider`
- You only need a one-time async operation → use `FutureProvider`
- You only need mutable async state → use `AsyncNotifier`
- You only need simple sync state → use `Notifier`

---

## Best Practices

- Prefer `StreamProvider` if no control logic is needed
- Keep stream creation inside `build()`
- Use methods for user-triggered events
- Avoid manual subscriptions
- Ensure streams are properly closed if needed
- Keep emitted models immutable

---

## Common Mistakes

### 1. Using StreamNotifier for Simple Streams

❌ Wrong

```dart
class Messages extends StreamNotifier<Message> {
  @override
  Stream<Message> build() {
    return repository.messages();
  }
}
```

Why it's wrong:

- No additional logic or control is needed.
- Adds unnecessary complexity.

✔ Correct

```dart
final messagesProvider =
    StreamProvider((ref) {
  return repository.messages();
});
```

---

### 2. Doing Work in Widgets Instead of Notifier

❌ Wrong

```dart
socket.send(message);
```

Why it's wrong:

- Business logic leaks into UI layer.

✔ Correct

```dart
ref.read(chatProvider.notifier).sendMessage(message);
```

---

### 3. Mixing Stream and State Logic

❌ Wrong

```dart
state = AsyncData(stream.value);
```

Why it's wrong:

- Streams should not be manually converted into state.

✔ Correct

Let `StreamNotifier` handle emissions automatically.

---

## Related APIs

- StreamNotifierProvider
- StreamProvider
- AsyncNotifier
- Notifier
- AsyncValue
- ref.watch()
- ref.read()

---

## Summary

`StreamNotifier` is Riverpod’s advanced tool for managing controlled streams with business logic. It allows you to create, manage, and interact with streams while keeping state reactive, centralized, and lifecycle-aware. It is ideal for real-time systems where streams must be both consumed and controlled.