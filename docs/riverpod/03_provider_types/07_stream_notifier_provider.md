# StreamNotifierProvider

`StreamNotifierProvider` is a provider that exposes a `StreamNotifier`, allowing you to create, manage, and control a stream of values while encapsulating business logic.

---

## What is it?

`StreamNotifierProvider` is the recommended provider for **mutable streaming state** in Riverpod.

Unlike `StreamProvider`, which simply listens to an existing stream, `StreamNotifierProvider` lets you:

- Create streams
- Control emitted values
- Expose business logic
- React to user actions
- Access dependencies
- Manage stream lifecycle

It combines:

- Stream generation
- State management
- Business logic

into a single notifier.

---

## Why does it exist?

Consider a chat application.

With `StreamProvider`, you can easily listen to incoming messages.

```text
Server
   │
   ▼
StreamProvider
   │
   ▼
UI
```

But what if you also need to:

- Send messages
- Delete messages
- Filter messages
- Reconnect automatically
- Retry failed connections

`StreamProvider` has no place to store that business logic.

`StreamNotifierProvider` solves this by giving your stream its own controller class.

Benefits include:

- Centralized stream logic
- Better testability
- Dependency injection
- Lifecycle management
- Cleaner architecture

---

## Syntax

### Creating a StreamNotifier

```dart
class CounterNotifier extends StreamNotifier<int> {
  @override
  Stream<int> build() async* {
    for (var i = 0; i < 5; i++) {
      await Future.delayed(const Duration(seconds: 1));
      yield i;
    }
  }
}
```

Explanation:

- `build()` returns a `Stream<int>`.
- Every `yield` emits a new value.
- Consumers rebuild automatically.

---

### Creating a StreamNotifierProvider

```dart
final counterProvider =
    StreamNotifierProvider<
      CounterNotifier,
      int
    >(CounterNotifier.new);
```

Explanation:

- Exposes the notifier.
- Widgets watch an `AsyncValue<int>`.
- Methods are accessed through `.notifier`.

---

### Reading the State

```dart
final counter = ref.watch(counterProvider);
```

Explanation:

- Returns an `AsyncValue<int>`.
- Automatically updates for each emitted value.

---

### Calling Methods

```dart
ref.read(counterProvider.notifier).restart();
```

Explanation:

- `.notifier` returns the `StreamNotifier`.
- Methods control stream behavior.

---

## Mental Model

Think of `StreamNotifier` as a live event controller.

```text
             Widget
                │
                ▼
     StreamNotifierProvider
                │
                ▼
        StreamNotifier
        ┌───────┴────────┐
        ▼                ▼
 Business Logic        Stream
                           │
                    AsyncValue
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Loading           Data            Error
```

The notifier owns both the stream and the business logic that controls it.

---

## Examples

### Timer

```dart
class TimerNotifier extends StreamNotifier<int> {
  @override
  Stream<int> build() async* {
    var seconds = 0;

    while (true) {
      await Future.delayed(const Duration(seconds: 1));
      yield seconds++;
    }
  }
}
```

Explanation:

- Emits a new value every second.
- Widgets rebuild automatically.

---

### Chat Messages

```dart
class ChatNotifier extends StreamNotifier<Message> {
  @override
  Stream<Message> build() {
    return socket.messages();
  }

  Future<void> send(Message message) async {
    await socket.send(message);
  }
}
```

Explanation:

- The notifier streams incoming messages.
- It also exposes a method to send messages.

---

### Location Tracking

```dart
class LocationNotifier
    extends StreamNotifier<Location> {
  @override
  Stream<Location> build() {
    return locationService.locationStream();
  }

  Future<void> refresh() async {
    locationService.requestUpdate();
  }
}
```

Explanation:

- Streams location updates.
- Allows requesting a fresh location through a method.

---

## State Flow

```text
Provider Created
        │
        ▼
build()
        │
        ▼
 AsyncLoading
        │
        ▼
First Stream Event
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
Method Called
        │
        ▼
Business Logic
        │
        ▼
Future Stream Events
```

The provider continues emitting values until the stream ends or the provider is disposed.

---

## StreamNotifierProvider vs StreamProvider

| Feature | StreamProvider | StreamNotifierProvider |
|---------|----------------|-------------------------|
| Listen to existing streams | ✅ | ✅ |
| Create streams | Limited | ✅ |
| Expose methods | ❌ | ✅ |
| Business logic | ❌ | ✅ |
| Dependency injection | Limited | ✅ |
| Mutable stream behavior | ❌ | ✅ |

---

## When to Use

Use `StreamNotifierProvider` for:

- Chat systems
- Live dashboards
- WebSockets
- Real-time collaboration
- Sensor streams
- Location tracking
- Event-driven applications
- Streaming business logic

Use it when your stream needs more than simple consumption.

---

## When NOT to Use

Avoid `StreamNotifierProvider` when:

- You only need to listen to an existing stream.
- No business logic is required.
- The operation returns a single asynchronous value.
- The state is synchronous.

Instead, use:

| Scenario | Recommended Provider |
|----------|----------------------|
| Existing stream | `StreamProvider` |
| One-time async request | `FutureProvider` |
| Mutable async state | `AsyncNotifierProvider` |
| Mutable sync state | `NotifierProvider` |

---

## Best Practices

- Keep stream creation inside the notifier.
- Expose meaningful methods for stream-related operations.
- Keep widgets free of business logic.
- Use immutable models for emitted values.
- Use `autoDispose` for short-lived streams when appropriate.

---

## Common Mistakes

### 1. Using StreamProvider for Mutable Stream Logic

❌ Wrong

```dart
final chatProvider =
    StreamProvider<Message>((ref) {
  return socket.messages();
});

// Sending messages from widgets
socket.send(message);
```

Why it's wrong:

- Business logic is scattered outside the provider.
- The provider only consumes the stream.

✔ Correct

```dart
ref.read(chatProvider.notifier).send(message);
```

Keep stream operations inside the notifier.

---

### 2. Performing Business Logic in Widgets

❌ Wrong

```dart
onPressed: () {
  socket.send(message);
}
```

Why it's wrong:

- Widgets should not communicate directly with services.
- This makes testing and maintenance harder.

✔ Correct

```dart
onPressed: () {
  ref.read(chatProvider.notifier).send(message);
}
```

Delegate operations to the notifier.

---

### 3. Using StreamNotifierProvider for Read-Only Streams

❌ Wrong

```dart
class NotificationsNotifier
    extends StreamNotifier<Notification> {
  @override
  Stream<Notification> build() {
    return notificationService.stream;
  }
}
```

Why it's wrong:

- If no business logic or methods are needed, a `StreamProvider` is simpler.

✔ Correct

```dart
final notificationsProvider =
    StreamProvider<Notification>((ref) {
  return notificationService.stream;
});
```

Use the simplest provider that meets your needs.

---

## Recommendation

> **Use `StreamNotifierProvider` when your stream requires business logic or user interactions.**
>
> If you're only consuming an existing stream without modifying its behavior, prefer `StreamProvider`.

---

## Related APIs

- StreamNotifier
- StreamProvider
- AsyncNotifierProvider
- NotifierProvider
- AsyncValue
- ref.watch()
- ref.read()

---

## Summary

`StreamNotifierProvider` is Riverpod's provider for mutable streaming state. It combines stream creation, business logic, and state management inside a `StreamNotifier`, exposing values as an `AsyncValue` while allowing methods to control stream behavior. It is the preferred choice when your application needs to both consume and manage real-time data streams.