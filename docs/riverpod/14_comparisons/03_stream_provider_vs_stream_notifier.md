# StreamProvider vs StreamNotifier

Choose between `StreamProvider` and `StreamNotifier` based on whether your stream is read-only or needs to be controlled and updated by your application.

---

`StreamProvider` listens to an existing stream, while `StreamNotifier` creates and manages a stream as part of your application's business logic.

---

# What is it?

Both providers expose asynchronous stream data through an `AsyncValue<T>`, but they have different responsibilities.

- **StreamProvider** subscribes to an existing stream and forwards its events.
- **StreamNotifier** owns the stream and provides methods to control or modify it.

Think of them as:

- **StreamProvider = Listen**
- **StreamNotifier = Manage**

---

# Why does it exist?

Many applications only need to consume a stream.

Examples include:

- Firebase authentication
- Firestore collections
- WebSocket messages
- Device sensors

Other applications need to actively manage the stream:

- Sending chat messages
- Filtering live data
- Reconnecting after failures
- Combining multiple event sources
- Controlling subscriptions

`StreamNotifier` provides a place for this business logic instead of pushing it into widgets.

---

# Syntax

## StreamProvider

```dart
final messagesProvider =
    StreamProvider<List<Message>>((ref) {
  return repository.messages();
});
```

Explanation:

- Listens to an existing stream.
- Emits new values as they arrive.
- Does not expose methods for modifying the stream.

---

## StreamNotifier

```dart
class ChatNotifier
    extends StreamNotifier<List<Message>> {
  @override
  Stream<List<Message>> build() {
    return repository.messages();
  }

  Future<void> sendMessage(String text) async {
    await repository.send(text);
  }
}

final chatProvider =
    StreamNotifierProvider<
        ChatNotifier,
        List<Message>>(
      ChatNotifier.new,
    );
```

Explanation:

- `build()` returns the stream to expose.
- Additional methods manage stream-related business logic.

---

# Mental Model

```text
StreamProvider
──────────────

External Stream
       │
       ▼
Riverpod
       │
       ▼
UI
```

```text
StreamNotifier
──────────────

Business Logic
       │
       ▼
Creates / Controls Stream
       │
       ▼
UI
```

`StreamProvider` consumes a stream, while `StreamNotifier` owns the stream's behavior.

---

# Examples

## Simple Example

### StreamProvider

```dart
final clockProvider =
    StreamProvider<DateTime>((ref) {
  return Stream.periodic(
    const Duration(seconds: 1),
    (_) => DateTime.now(),
  );
});
```

Explanation:

- The provider exposes a read-only stream of time updates.
- No additional behavior is required.

---

### StreamNotifier

```dart
class NotificationNotifier
    extends StreamNotifier<Notification> {
  @override
  Stream<Notification> build() {
    return notificationService.stream;
  }

  Future<void> markAsRead(
    Notification notification,
  ) async {
    await repository.markAsRead(notification.id);
  }
}
```

Explanation:

- The notifier exposes the stream.
- It also provides actions related to streamed data.

---

## Real-World Example

### StreamProvider

```dart
final authProvider =
    StreamProvider<User?>((ref) {
  return authRepository.authStateChanges();
});
```

Explanation:

- Simply forwards authentication state changes.
- No custom business logic is required.

---

### StreamNotifier

```dart
class ChatRoomNotifier
    extends StreamNotifier<List<Message>> {
  @override
  Stream<List<Message>> build() {
    return repository.watchMessages();
  }

  Future<void> deleteMessage(
    String id,
  ) async {
    await repository.deleteMessage(id);
  }

  Future<void> sendMessage(
    String text,
  ) async {
    await repository.sendMessage(text);
  }
}
```

Explanation:

- The notifier manages both the live stream and chat operations.
- Widgets remain focused on displaying messages.

---

# Feature Comparison

| Feature | StreamProvider | StreamNotifier |
|----------|----------------|----------------|
| Listen to streams | ✅ | ✅ |
| Returns `AsyncValue` | ✅ | ✅ |
| Business logic | ❌ | ✅ |
| Stream control | ❌ | ✅ |
| Send/update operations | ❌ | ✅ |
| Best for external streams | ✅ | ⚠️ Possible |
| Best for live application state | ❌ | ✅ |

---

# When to Use

Use **StreamProvider** when:

- Listening to Firebase streams
- Listening to Firestore collections
- Displaying sensor data
- Watching WebSocket events
- No additional stream logic is required

Use **StreamNotifier** when:

- Sending chat messages
- Managing subscriptions
- Reconnecting streams
- Filtering live events
- Combining multiple streams
- Adding business logic around streamed data

---

# When NOT to Use

Avoid **StreamProvider** when:

- The stream needs actions such as send, delete, or reconnect.
- Business logic belongs with the stream.

Avoid **StreamNotifier** when:

- The provider simply forwards an existing stream.
- No additional behavior is required.

Choose the simplest solution that meets your needs.

---

# Best Practices

- Use `StreamProvider` for passive stream consumption.
- Use `StreamNotifier` for active stream management.
- Keep stream-related business logic inside the notifier.
- Let widgets observe providers rather than manipulate streams directly.
- Avoid duplicating stream subscriptions.

---

# Common Mistakes

## Using StreamProvider for interactive features

Wrong:

```dart
final chatProvider =
    StreamProvider<List<Message>>(...);
```

Explanation:

- There is no place to send or delete messages.
- Business logic often ends up in the UI.

Correct:

```dart
final chatProvider =
    StreamNotifierProvider<
        ChatNotifier,
        List<Message>>(
      ChatNotifier.new,
    );
```

Explanation:

- The notifier owns both the stream and related actions.

---

## Performing repository actions from widgets

Wrong:

```dart
await repository.sendMessage(text);
```

Explanation:

- The widget becomes responsible for business logic.

Correct:

```dart
await ref
    .read(chatProvider.notifier)
    .sendMessage(text);
```

Explanation:

- Widgets request actions from the notifier.
- Business logic stays centralized.

---

## Creating multiple subscriptions

Wrong:

```dart
repository.watchMessages();

repository.watchMessages();
```

Explanation:

- Multiple subscriptions may waste resources and produce duplicate work.

Correct:

```dart
final messages = ref.watch(chatProvider);
```

Explanation:

- Let Riverpod manage the stream subscription.

---

# Related APIs

- StreamProvider
- StreamNotifier
- StreamNotifierProvider
- AsyncValue
- `ref.watch()`
- `ref.read()`

---

# Summary

`StreamProvider` is the ideal choice for consuming existing streams with no additional logic, while `StreamNotifier` is designed for live data that requires business logic, user actions, or stream management. Use `StreamProvider` to listen and `StreamNotifier` to manage.