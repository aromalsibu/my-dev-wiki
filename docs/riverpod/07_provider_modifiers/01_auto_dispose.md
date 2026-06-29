# .autoDispose

Automatically disposes a provider when it is no longer being used.

---

# What is it?

`.autoDispose` is a provider modifier that tells Riverpod to destroy a provider once nothing is listening to it anymore.

Normally, providers remain alive in memory even after no widgets are using them. Adding `.autoDispose` changes this behavior by allowing Riverpod to free the provider's state and resources when it becomes unused.

This is especially useful for temporary state such as screen-specific data, network requests, and short-lived objects.

---

# Why does it exist?

Without `.autoDispose`, providers stay alive for the lifetime of the `ProviderScope`.

This is useful for shared application state, but unnecessary for temporary state that is only needed while a screen is visible.

`.autoDispose` helps:

- Reduce memory usage
- Dispose unused resources automatically
- Prevent stale state from lingering
- Cancel work that is no longer needed

---

# Syntax

## Basic Usage

```dart
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  return fetchUser();
});
```

Explanation:

- `.autoDispose` changes the provider's lifecycle.
- The provider is destroyed when no listeners remain.
- The next time it is accessed, it is created again.

---

## With NotifierProvider

```dart
final counterProvider =
    NotifierProvider.autoDispose<CounterNotifier, int>(
      CounterNotifier.new,
    );
```

Explanation:

- The notifier is automatically disposed when unused.
- A new notifier instance is created when the provider is accessed again.

---

## With StreamProvider

```dart
final messagesProvider =
    StreamProvider.autoDispose<List<Message>>((ref) {
  return chatRepository.messages();
});
```

Explanation:

- The stream subscription is cancelled automatically.
- A new subscription starts when the provider is listened to again.

---

# Mental Model

Think of `.autoDispose` as an automatic cleanup system.

```text
Widget starts listening
        │
        ▼
 Provider is created
        │
        ▼
Provider stays alive
        │
        ▼
No listeners remain
        │
        ▼
Riverpod disposes provider
```

Without `.autoDispose`:

```text
Created
   │
   ▼
Lives for entire ProviderScope
```

With `.autoDispose`:

```text
Created
   │
   ▼
Used
   │
   ▼
Disposed automatically
```

---

# Examples

## Simple Example

```dart
final randomNumberProvider =
    Provider.autoDispose<int>((ref) {
  return Random().nextInt(100);
});
```

Explanation:

- Every time the provider is recreated, a new random number is generated.
- The previous value is discarded once unused.

---

## Real-World Example

```dart
final profileProvider =
    FutureProvider.autoDispose<User>((ref) async {
  return api.fetchProfile();
});
```

Explanation:

- The profile is loaded while the screen is active.
- Leaving the screen disposes the provider.
- Returning reloads fresh data.

---

# When to Use

Use `.autoDispose` when:

- State belongs to a single screen
- Fetching temporary API data
- Managing stream subscriptions
- Holding search results
- Managing form state
- Working with short-lived providers

---

# When NOT to Use

Avoid `.autoDispose` when:

- The state should be shared across multiple screens.
- The data should remain cached throughout the application.
- Recreating the provider is expensive and unnecessary.

Examples include:

- Authentication state
- Current user
- App configuration
- Theme settings

---

# Best Practices

- Use `.autoDispose` for screen-specific providers.
- Combine it with `ref.onDispose()` to clean up resources.
- Use `ref.keepAlive()` when you want selective caching.
- Don't use it for global application state.
- Let Riverpod manage disposal instead of manual cleanup whenever possible.

---

# Common Mistakes

## Forgetting That State Is Recreated

**Wrong**

```dart
final counterProvider =
    StateProvider.autoDispose<int>((ref) => 0);
```

Navigating away resets the counter unexpectedly.

**Correct**

```dart
final counterProvider =
    StateProvider<int>((ref) => 0);
```

Use a normal provider if the state should persist.

---

## Using autoDispose for Global State

**Wrong**

```dart
final authProvider =
    Provider.autoDispose<AuthService>((ref) {
  return AuthService();
});
```

The authentication service may be recreated unnecessarily.

**Correct**

```dart
final authProvider = Provider<AuthService>((ref) {
  return AuthService();
});
```

Global services should generally live for the application's lifetime.

---

## Assuming Resources Are Cleaned Up Automatically

**Wrong**

```dart
final timerProvider =
    Provider.autoDispose<Timer>((ref) {
  return Timer.periodic(
    const Duration(seconds: 1),
    (_) {},
  );
});
```

The timer continues running unless explicitly cancelled.

**Correct**

```dart
final timerProvider =
    Provider.autoDispose<Timer>((ref) {
  final timer = Timer.periodic(
    const Duration(seconds: 1),
    (_) {},
  );

  ref.onDispose(timer.cancel);

  return timer;
});
```

Use `ref.onDispose()` to clean up resources created inside the provider.

---

# Related APIs

- `.family`
- `keepAlive`
- `ref.keepAlive()`
- `ref.onDispose()`
- `Provider`
- `FutureProvider`
- `StreamProvider`
- `NotifierProvider`

---

# Summary

`.autoDispose` automatically destroys a provider when it is no longer being used. It helps reduce memory usage, clean up temporary state, and manage short-lived resources without manual lifecycle management. Use it for screen-specific or temporary providers, but avoid it for long-lived application state.