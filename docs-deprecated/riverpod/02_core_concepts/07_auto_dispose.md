# Auto Dispose

`autoDispose` is a provider modifier that automatically disposes a provider when it is no longer being used.

---

## What is it?

By default, Riverpod keeps a provider alive even after no widgets are listening to it.

This is useful for:

- Shared repositories
- Application-wide services
- Long-lived state

However, some providers are only needed temporarily.

Examples include:

- A screen-specific API request
- A search result
- A temporary form state
- A stream subscription

Keeping these providers alive wastes memory and system resources.

Applying the `.autoDispose` modifier tells Riverpod:

> "Dispose this provider when nothing is using it anymore."

---

## Why does it exist?

Imagine navigating between screens.

```text
Home Screen
      │
      ▼
User Profile
      │
Reads userProvider
      │
Back to Home
```

Without `autoDispose`:

```text
Provider
      │
Still Alive
      │
Still Cached
      │
Still Using Memory
```

Even though the screen is gone, the provider remains in memory.

With `autoDispose`:

```text
Leave Screen
      │
No Consumers
      │
Dispose Provider
      │
Free Memory
```

Benefits include:

- Lower memory usage
- Automatic resource cleanup
- Closing streams and sockets
- Cancelling subscriptions
- Better performance

---

## Syntax

### Basic Usage

```dart
final counterProvider = Provider.autoDispose<int>((ref) {
  return 0;
});
```

Explanation:

- `.autoDispose` changes the provider's lifecycle.
- The provider is disposed when no listeners remain.

---

### FutureProvider with autoDispose

```dart
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- The fetched data is discarded when unused.
- Returning to the screen creates a fresh provider.

---

### StreamProvider with autoDispose

```dart
final chatProvider = StreamProvider.autoDispose<Message>((ref) {
  return chatRepository.messages();
});
```

Explanation:

- The stream subscription is automatically cancelled.
- Prevents unnecessary background work.

---

### Cleaning Up Resources

```dart
final socketProvider = Provider.autoDispose((ref) {
  final socket = SocketConnection();

  ref.onDispose(() {
    socket.close();
  });

  return socket;
});
```

Explanation:

- `ref.onDispose()` performs cleanup.
- The socket closes automatically when the provider is disposed.

---

## Mental Model

Without `autoDispose`:

```text
Read Provider
      │
      ▼
Create
      │
      ▼
Cache
      │
      ▼
No Consumers
      │
      ▼
Still Alive
```

With `autoDispose`:

```text
Read Provider
      │
      ▼
Create
      │
      ▼
Cache
      │
      ▼
No Consumers
      │
      ▼
Dispose
```

`autoDispose` allows Riverpod to reclaim resources automatically.

---

## What Triggers Disposal?

An `autoDispose` provider is disposed when:

- No widgets are watching it.
- No other providers depend on it.
- No active listeners remain.

Once disposed:

- Cached values are removed.
- `ref.onDispose()` callbacks execute.
- Resources are released.

The next read creates a new instance.

---

## Examples

### Temporary Screen State

```dart
final searchProvider = FutureProvider.autoDispose<List<Product>>((ref) async {
  return repository.searchProducts();
});
```

Explanation:

- Search results exist only while the screen is active.
- Leaving the screen disposes the provider.

---

### Stream Subscription

```dart
final notificationsProvider =
    StreamProvider.autoDispose<Notification>((ref) {
  return repository.notifications();
});
```

Explanation:

- The stream subscription ends automatically.
- Prevents listening when no UI needs updates.

---

### Form Controller

```dart
final formProvider = Provider.autoDispose((ref) {
  final controller = TextEditingController();

  ref.onDispose(controller.dispose);

  return controller;
});
```

Explanation:

- The controller is disposed automatically.
- Prevents memory leaks.

---

## Auto Dispose vs Regular Providers

### Regular Provider

```text
Read
 │
 ▼
Create
 │
 ▼
Cache
 │
 ▼
No Consumers
 │
 ▼
Still Cached
```

The provider remains alive until:

- The container is disposed.
- It is invalidated.
- A dependency changes.

---

### Auto Dispose Provider

```text
Read
 │
 ▼
Create
 │
 ▼
Cache
 │
 ▼
No Consumers
 │
 ▼
Dispose
```

The provider is recreated on the next read.

---

## When to Use

Use `autoDispose` for:

- Screen-specific state
- API requests
- Search results
- Temporary forms
- Streams
- Timers
- Socket connections
- Temporary controllers
- Short-lived business logic

---

## When NOT to Use

Avoid `autoDispose` for:

- Authentication state
- Application settings
- Theme providers
- Shared repositories
- Dependency injection
- Global application state

These providers should usually remain alive for the application's lifetime.

---

## Best Practices

- Use `autoDispose` for temporary state.
- Combine it with `ref.onDispose()` to clean up resources.
- Avoid using it for global providers.
- Keep provider recreation inexpensive when using `autoDispose`.
- Consider `ref.keepAlive()` if an `autoDispose` provider should remain alive under certain conditions.

---

## Common Mistakes

### 1. Using autoDispose for Global State

❌ Wrong

```dart
final authProvider = Provider.autoDispose((ref) {
  return AuthService();
});
```

Why it's wrong:

- Authentication should persist.
- The provider may be recreated unnecessarily.

✔ Correct

```dart
final authProvider = Provider((ref) {
  return AuthService();
});
```

---

### 2. Forgetting Cleanup

❌ Wrong

```dart
final streamProvider = StreamProvider.autoDispose((ref) {
  return repository.messages();
});
```

Why it's wrong:

- If additional resources are created, they may remain open.

✔ Correct

```dart
final streamProvider = StreamProvider.autoDispose((ref) {
  final socket = SocketConnection();

  ref.onDispose(socket.close);

  return socket.messages();
});
```

Dispose any manually created resources.

---

### 3. Expecting Cached Data After Disposal

❌ Wrong

```text
Open Screen
      │
Fetch Data
      │
Leave Screen
      │
Return
      │
Expect Cached Data
```

Why it's wrong:

- `autoDispose` removes the cached value.
- Returning recreates the provider.

✔ Correct

If the data should persist across navigation, use a regular provider or keep the provider alive.

---

## Related APIs

- Provider.autoDispose
- FutureProvider.autoDispose
- StreamProvider.autoDispose
- NotifierProvider.autoDispose
- AsyncNotifierProvider.autoDispose
- ref.keepAlive()
- ref.onDispose()
- Provider Lifecycle
- Caching

---

## Summary

`autoDispose` changes a provider's lifecycle so it is automatically disposed when no longer in use. This helps reduce memory usage, clean up resources, and prevent unnecessary background work. It is ideal for temporary, screen-specific state but should generally be avoided for long-lived application state such as authentication or shared services.