# Auto Dispose Strategy

Use automatic disposal to release resources when providers are no longer needed while keeping important state alive when appropriate.

---

# What is it?

An auto dispose strategy is the approach you use to determine which providers should be automatically disposed and which should remain alive.

Riverpod's `.autoDispose` modifier automatically destroys a provider after it no longer has any listeners. This helps prevent memory leaks and frees resources that are no longer in use.

Choosing where to use `.autoDispose` is an important performance consideration.

---

# Why does it exist?

Without automatic disposal, providers remain in memory even after the UI that uses them has disappeared.

This can lead to:

- Unnecessary memory usage.
- Active network subscriptions.
- Running timers.
- Open stream connections.
- Cached data that is never used again.

Auto disposal ensures resources are cleaned up automatically while allowing long-lived application state to remain available when needed.

---

# Syntax

## Automatically dispose a provider

```dart
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  return fetchUser();
});
```

Explanation:

- `.autoDispose` tells Riverpod to dispose the provider when it is no longer being watched.
- The provider is recreated the next time it is accessed.

---

## Keep an auto disposed provider alive

```dart
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  ref.keepAlive();

  return fetchUser();
});
```

Explanation:

- `ref.keepAlive()` prevents automatic disposal.
- Useful when the result should remain cached after loading.

---

## Clean up resources

```dart
final timerProvider = Provider.autoDispose<Timer>((ref) {
  final timer = Timer.periodic(
    const Duration(seconds: 1),
    (_) {},
  );

  ref.onDispose(() {
    timer.cancel();
  });

  return timer;
});
```

Explanation:

- `ref.onDispose()` runs when the provider is destroyed.
- Use it to release resources such as timers, streams, or controllers.

---

# Mental Model

Without auto dispose:

```text
Open screen
      │
      ▼
Provider created
      │
      ▼
Close screen
      │
      ▼
Provider remains alive
```

With auto dispose:

```text
Open screen
      │
      ▼
Provider created
      │
      ▼
Close screen
      │
      ▼
Provider disposed
      │
      ▼
Resources released
```

Think of `.autoDispose` as automatically cleaning up after a provider is no longer needed.

---

# Examples

## Simple Example

```dart
final postsProvider =
    FutureProvider.autoDispose<List<Post>>((ref) async {
  return fetchPosts();
});
```

Explanation:

- The provider is disposed after the screen stops using it.
- Opening the screen again creates a new provider instance.

---

## Real-World Example

```dart
final chatProvider =
    StreamProvider.autoDispose<Message>((ref) {
  final stream = chatRepository.messages();

  ref.onDispose(() {
    chatRepository.disconnect();
  });

  return stream;
});
```

Explanation:

- The chat connection exists only while the screen is active.
- Leaving the screen automatically disconnects the stream.

---

# When to Use

Use `.autoDispose` for:

- Screen-specific state.
- Network requests.
- Temporary forms.
- Search results.
- Streams.
- Timers.
- Animation-related state.
- Short-lived providers.

---

# When NOT to Use

Avoid `.autoDispose` for:

- Authentication state.
- Current user information.
- Application configuration.
- Theme settings.
- Dependency injection providers.
- Shared repositories.
- Global application state.

These values are typically needed throughout the application's lifetime.

---

# Best Practices

- Default to `.autoDispose` for feature-specific providers.
- Keep global state alive only when necessary.
- Always clean up external resources using `ref.onDispose()`.
- Cache expensive results with `ref.keepAlive()` when appropriate.
- Test provider lifecycle behavior during navigation.

---

# Common Mistakes

## Using `.autoDispose` for global state

Wrong:

```dart
final authProvider =
    NotifierProvider.autoDispose<AuthNotifier, AuthState>(
      AuthNotifier.new,
    );
```

Explanation:

- Authentication state may be destroyed when no widget is listening.
- Users may need to reinitialize state unnecessarily.

Correct:

```dart
final authProvider =
    NotifierProvider<AuthNotifier, AuthState>(
      AuthNotifier.new,
    );
```

Explanation:

- Global application state should usually remain alive.

---

## Forgetting to clean up resources

Wrong:

```dart
final streamProvider =
    StreamProvider.autoDispose((ref) {
  return repository.messages();
});
```

Explanation:

- External resources may continue running if they require manual cleanup.

Correct:

```dart
final streamProvider =
    StreamProvider.autoDispose((ref) {
  ref.onDispose(() {
    repository.disconnect();
  });

  return repository.messages();
});
```

Explanation:

- Cleanup code runs automatically when the provider is disposed.

---

## Using `keepAlive()` everywhere

Wrong:

```dart
ref.keepAlive();
```

Explanation:

- Preventing disposal for every provider defeats the purpose of `.autoDispose`.
- Memory usage can grow unnecessarily.

Correct:

```dart
if (cacheShouldBePreserved) {
  ref.keepAlive();
}
```

Explanation:

- Keep providers alive only when there is a clear performance or user experience benefit.

---

# Related APIs

- `.autoDispose`
- `ref.keepAlive()`
- `ref.onDispose()`
- `ref.onCancel()`
- `ref.onResume()`
- FutureProvider
- StreamProvider
- AsyncNotifierProvider

---

# Summary

An auto dispose strategy is about deciding which providers should have short lifetimes and which should persist for the lifetime of the application. Use `.autoDispose` for temporary state and resources, combine it with `ref.onDispose()` for cleanup, and use `ref.keepAlive()` only when retaining cached state provides a measurable benefit.