# Lifecycle APIs

A quick reference for Riverpod APIs related to provider creation, disposal, caching, and lifecycle management.

---

# What is it?

Lifecycle APIs let you control how providers are created, kept alive, invalidated, paused, resumed, and disposed.

Most applications only use a few lifecycle APIs, but understanding them helps you build efficient and predictable state management.

---

# Why does it exist?

Riverpod automatically manages provider lifecycles, but there are times when you need more control.

Common scenarios include:

- Cleaning up resources
- Preventing unnecessary network requests
- Keeping expensive providers alive
- Refreshing cached data
- Responding to listeners being added or removed

Lifecycle APIs provide these capabilities.

---

# Lifecycle API Reference

| API | Purpose |
|------|---------|
| `ref.onDispose()` | Run cleanup before a provider is disposed |
| `ref.onCancel()` | Called when the last listener is removed |
| `ref.onResume()` | Called when a listener is added again |
| `ref.keepAlive()` | Prevent an auto-disposed provider from being disposed |
| `ref.invalidate()` | Mark a provider for recomputation |
| `ref.invalidateSelf()` | Invalidate the current provider |
| `ref.refresh()` | Recreate a provider immediately |
| `.autoDispose` | Dispose providers when no longer used |

---

# Cleanup

## `ref.onDispose()`

```dart
ref.onDispose(() {
  controller.dispose();
});
```

Explanation:

- Runs once before the provider is destroyed.
- Ideal for closing controllers, streams, and subscriptions.

---

# Auto Dispose

```dart
final userProvider =
    FutureProvider.autoDispose<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- Automatically disposes the provider when it is no longer listened to.
- Frees memory and releases resources.

---

# Keep a Provider Alive

```dart
final link = ref.keepAlive();
```

Explanation:

- Prevents an `.autoDispose` provider from being disposed.
- Returns a `KeepAliveLink` that can later be closed if needed.

---

# Detect Listener Removal

## `ref.onCancel()`

```dart
ref.onCancel(() {
  print('No active listeners');
});
```

Explanation:

- Invoked when the last listener stops watching the provider.
- Useful for pausing work or releasing temporary resources.

---

# Detect Listener Return

## `ref.onResume()`

```dart
ref.onResume(() {
  print('Listener attached');
});
```

Explanation:

- Called when a new listener starts watching again after cancellation.
- Useful for resuming paused work.

---

# Invalidate a Provider

```dart
ref.invalidate(userProvider);
```

Explanation:

- Marks the provider as stale.
- It will rebuild the next time it is accessed.

---

# Invalidate the Current Provider

```dart
ref.invalidateSelf();
```

Explanation:

- Marks the current provider for recomputation.
- Commonly used inside notifiers after state-changing operations.

---

# Refresh Immediately

```dart
ref.refresh(userProvider);
```

Explanation:

- Disposes and recreates the provider immediately.
- Returns the newly computed value.

---

# Lifecycle Flow

```text
Created
   │
   ▼
First Listener
   │
   ▼
Active
   │
   ├── refresh()
   ├── invalidate()
   ├── invalidateSelf()
   │
   ▼
No Listeners
   │
   ├── onCancel()
   │
   ▼
Disposed (.autoDispose)
   │
   └── onDispose()
```

If another listener appears before disposal:

```text
No Listeners
     │
     ▼
onResume()
     │
     ▼
Active Again
```

---

# Common Usage Patterns

## Dispose a controller

```dart
final provider = Provider((ref) {
  final controller = StreamController<int>();

  ref.onDispose(controller.close);

  return controller;
});
```

Explanation:

- Ensures the controller is always closed when the provider is disposed.

---

## Keep expensive data cached

```dart
final provider =
    FutureProvider.autoDispose((ref) async {
  ref.keepAlive();

  return repository.loadData();
});
```

Explanation:

- Prevents expensive data from being discarded immediately after listeners disappear.

---

## Refresh stale data

```dart
ref.invalidate(profileProvider);
```

Explanation:

- The next read fetches fresh data without forcing an immediate rebuild.

---

# Choosing the Right API

| Situation | API |
|-----------|-----|
| Cleanup resources | `ref.onDispose()` |
| Pause work when unused | `ref.onCancel()` |
| Resume paused work | `ref.onResume()` |
| Cache auto-disposed providers | `ref.keepAlive()` |
| Refresh later | `ref.invalidate()` |
| Refresh current provider | `ref.invalidateSelf()` |
| Refresh immediately | `ref.refresh()` |
| Automatically free resources | `.autoDispose` |

---

# Best Practices

- Always dispose of controllers, streams, and subscriptions.
- Use `.autoDispose` for short-lived providers.
- Use `keepAlive()` only when caching is beneficial.
- Prefer `invalidate()` when immediate recomputation isn't required.
- Use `refresh()` when you need fresh data immediately.
- Keep lifecycle callbacks lightweight.

---

# Related APIs

- `ref.onDispose()`
- `ref.onCancel()`
- `ref.onResume()`
- `ref.keepAlive()`
- `ref.invalidate()`
- `ref.invalidateSelf()`
- `ref.refresh()`
- `.autoDispose`

---

# Summary

Riverpod's lifecycle APIs provide fine-grained control over provider creation, caching, invalidation, and disposal. The most commonly used APIs are `onDispose()` for cleanup, `.autoDispose` for automatic resource management, `keepAlive()` for caching, `invalidate()` for deferred recomputation, and `refresh()` for immediate recomputation.