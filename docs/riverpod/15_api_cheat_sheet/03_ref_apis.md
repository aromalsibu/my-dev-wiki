# Ref APIs

A quick reference for the `Ref` and `WidgetRef` methods used to read, observe, refresh, and manage providers.

---

# What is it?

`Ref` is Riverpod's interface for interacting with providers.

It allows you to:

- Read providers
- Watch providers
- Listen for changes
- Refresh state
- Invalidate providers
- Manage lifecycle callbacks

`WidgetRef` extends these capabilities for use inside Flutter widgets.

---

# Why does it exist?

Instead of exposing global provider access, Riverpod centralizes all provider interactions through `Ref`.

This makes dependencies:

- Explicit
- Testable
- Easy to trace
- Safe to manage

Nearly every Riverpod application uses `Ref` extensively.

---

# State Access APIs

| API | Purpose |
|------|---------|
| `ref.watch()` | Watch a provider and rebuild on changes |
| `ref.read()` | Read a provider once without listening |
| `ref.listen()` | Execute a callback when a provider changes |
| `ref.listenManual()` | Manually manage a listener's lifecycle |

---

## `ref.watch()`

```dart
final counter = ref.watch(counterProvider);
```

Explanation:

- Subscribes to the provider.
- Rebuilds when the provider emits a new value.

---

## `ref.read()`

```dart
final notifier =
    ref.read(counterProvider.notifier);
```

Explanation:

- Reads the provider once.
- Does not subscribe for future updates.
- Commonly used in callbacks and notifier methods.

---

## `ref.listen()`

```dart
ref.listen(userProvider, (previous, next) {
  // React to changes
});
```

Explanation:

- Executes the callback whenever the provider changes.
- Does not rebuild the widget.

---

## `ref.listenManual()`

```dart
final subscription =
    ref.listenManual(
      userProvider,
      (previous, next) {
        // Handle updates
      },
    );
```

Explanation:

- Creates a listener that can be manually closed.
- Useful when you need explicit control over the listener's lifetime.

---

# Refresh APIs

| API | Purpose |
|------|---------|
| `ref.refresh()` | Immediately recreate a provider |
| `ref.invalidate()` | Mark a provider for recomputation |
| `ref.invalidateSelf()` | Mark the current provider for recomputation |

---

## `ref.refresh()`

```dart
ref.refresh(userProvider);
```

Explanation:

- Disposes and recreates the provider immediately.
- Returns the provider's new value.

---

## `ref.invalidate()`

```dart
ref.invalidate(userProvider);
```

Explanation:

- Marks the provider as stale.
- It rebuilds the next time it is accessed.

---

## `ref.invalidateSelf()`

```dart
ref.invalidateSelf();
```

Explanation:

- Invalidates the current provider from within its own implementation.

---

# Lifecycle APIs

| API | Purpose |
|------|---------|
| `ref.keepAlive()` | Prevent auto-disposal |
| `ref.onDispose()` | Cleanup before disposal |
| `ref.onCancel()` | Called when the last listener is removed |
| `ref.onResume()` | Called when listeners return |
| `ref.exists()` | Check whether a provider has been initialized |

---

## `ref.keepAlive()`

```dart
final link = ref.keepAlive();
```

Explanation:

- Prevents an `.autoDispose` provider from being disposed while the link remains active.

---

## `ref.onDispose()`

```dart
ref.onDispose(() {
  controller.dispose();
});
```

Explanation:

- Runs cleanup code before the provider is destroyed.

---

## `ref.onCancel()`

```dart
ref.onCancel(() {
  print('No listeners');
});
```

Explanation:

- Triggered when the final listener is removed.

---

## `ref.onResume()`

```dart
ref.onResume(() {
  print('Listener returned');
});
```

Explanation:

- Triggered when a listener is attached again after cancellation.

---

## `ref.exists()`

```dart
final initialized =
    ref.exists(userProvider);
```

Explanation:

- Returns `true` if the provider has already been created.
- Useful for conditional logic involving provider initialization.

---

# Mental Model

```text
                Ref
                 │
      ┌──────────┼──────────┐
      │          │          │
      ▼          ▼          ▼
  Read State  Manage State  Lifecycle
      │          │          │
      ▼          ▼          ▼
 watch()   refresh()    onDispose()
 read()    invalidate() keepAlive()
 listen()  invalidateSelf()
```

---

# Choosing the Right API

| Situation | API |
|-----------|-----|
| Build UI from state | `ref.watch()` |
| Call notifier methods | `ref.read()` |
| Perform side effects | `ref.listen()` |
| Control listener lifetime | `ref.listenManual()` |
| Force immediate reload | `ref.refresh()` |
| Reload on next access | `ref.invalidate()` |
| Reload current provider | `ref.invalidateSelf()` |
| Prevent disposal | `ref.keepAlive()` |
| Dispose resources | `ref.onDispose()` |
| Detect listener removal | `ref.onCancel()` |
| Detect listener return | `ref.onResume()` |
| Check initialization | `ref.exists()` |

---

# Best Practices

- Use `watch()` inside widget build methods.
- Use `read()` inside event handlers and notifier methods.
- Use `listen()` for side effects such as navigation or snackbars.
- Prefer `invalidate()` over `refresh()` when immediate recomputation is unnecessary.
- Always register cleanup callbacks for disposable resources.
- Keep lifecycle callbacks lightweight.

---

# Related APIs

- `WidgetRef`
- `Ref`
- `ref.watch()`
- `ref.read()`
- `ref.listen()`
- `ref.listenManual()`
- `ref.refresh()`
- `ref.invalidate()`
- `ref.invalidateSelf()`
- `ref.keepAlive()`
- `ref.onDispose()`
- `ref.onCancel()`
- `ref.onResume()`
- `ref.exists()`

---

# Summary

`Ref` is the primary interface for interacting with Riverpod providers. Its most frequently used methods are `watch()` for reactive reads, `read()` for one-time access, `listen()` for side effects, `refresh()` and `invalidate()` for recomputation, and lifecycle methods such as `onDispose()` and `keepAlive()` for managing provider resources. Mastering these APIs enables efficient and predictable state management throughout a Riverpod application.