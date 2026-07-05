# ref

`ref` is the Riverpod handle that provides access to providers, dependencies, and lifecycle utilities inside Notifiers, AsyncNotifiers, StreamNotifiers, and widgets.

---

## What is it?

`ref` is a **context object for Riverpod** that allows you to:

- Read providers
- Watch providers reactively
- Listen to changes
- Access other notifiers
- Manage lifecycle hooks
- Invalidate or refresh providers
- Inject dependencies

It is the **central gateway of Riverpod’s dependency system**.

---

## Why does it exist?

Without `ref`, notifiers would be isolated and unable to interact with:

- Other providers
- External services
- Dependency graphs
- Reactive updates

Before Riverpod’s design, state management often required:

- Manual dependency injection
- Global singletons
- Tight coupling between services

`ref` solves this by:

- Making dependencies explicit
- Enabling reactive composition
- Allowing safe cross-provider communication
- Supporting lifecycle-aware logic

It turns state management into a **graph of reactive dependencies instead of isolated objects**.

---

## Syntax

### Inside a Notifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    final repo = ref.watch(userRepositoryProvider);
    return repo.fetchUser();
  }
}
```

Explanation:

- `ref.watch()` listens to provider changes.
- Dependencies are automatically tracked.

---

### Inside a Widget

```dart
class UserView extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    return Text(user.name);
  }
}
```

Explanation:

- `ref.watch()` rebuilds widget on change.
- Provides reactive UI updates.

---

### Reading Without Listening

```dart
final repo = ref.read(userRepositoryProvider);
```

Explanation:

- Accesses value once.
- Does not trigger rebuilds.

---

### Listening to Changes

```dart
ref.listen(userProvider, (previous, next) {
  print(next);
});
```

Explanation:

- Reacts to state changes.
- Used for side effects (snackbars, navigation).

---

## Mental Model

Think of `ref` as the **control panel of the dependency graph**.

```text
            Provider Graph
                  │
                  ▼
               ref
      ┌──────────┼──────────┐
      ▼          ▼          ▼
   watch       read       listen
      │          │          │
      ▼          ▼          ▼
 reactive    one-time   side effects
 updates     access
```

It connects everything in Riverpod together.

---

## Core APIs

### 1. ref.watch()

```dart
final user = ref.watch(userProvider);
```

Explanation:

- Subscribes to provider changes.
- Rebuilds UI or notifier when value changes.

---

### 2. ref.read()

```dart
final repo = ref.read(userRepositoryProvider);
```

Explanation:

- Gets current value once.
- Does not rebuild on changes.

---

### 3. ref.listen()

```dart
ref.listen(counterProvider, (prev, next) {
  print(next);
});
```

Explanation:

- Runs side effects on state change.
- Does not rebuild UI.

---

### 4. ref.invalidate()

```dart
ref.invalidate(userProvider);
```

Explanation:

- Forces provider to recompute.
- Clears cached state.

---

### 5. ref.refresh()

```dart
final user = ref.refresh(userProvider);
```

Explanation:

- Invalidates and immediately recomputes provider.

---

### 6. ref.onDispose()

```dart
ref.onDispose(() {
  print("Disposed");
});
```

Explanation:

- Runs cleanup logic when provider is destroyed.

---

## Examples

### Dependency Injection

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    final repo = ref.watch(userRepositoryProvider);
    return repo.fetchUser();
  }
}
```

Explanation:

- Injects repository dependency.
- Automatically updates if dependency changes.

---

### Refreshing Data

```dart
Future<void> refresh() async {
  ref.invalidate(userProvider);
}
```

Explanation:

- Forces fresh data fetch.

---

### Listening for Side Effects

```dart
ref.listen(authProvider, (prev, next) {
  if (next.isLoggedIn) {
    print("Navigate to home");
  }
});
```

Explanation:

- Handles navigation or UI effects.

---

## When to Use

Use `ref` when you need to:

- Access other providers
- Listen to state changes
- Inject dependencies
- Trigger refresh/invalidate
- Perform side effects
- Manage lifecycle hooks

---

## When NOT to Use

Avoid using `ref` for:

- Direct UI rendering logic (use `watch` only in widgets)
- Heavy computations inside build without caching
- Business logic inside UI widgets when it belongs in Notifier methods

---

## Best Practices

- Prefer `watch()` for reactive dependencies
- Use `read()` for one-time access
- Keep side effects inside `listen()`
- Avoid overusing `invalidate()` unless necessary
- Structure dependencies clearly in `build()`

---

## Common Mistakes

### 1. Using read instead of watch

❌ Wrong

```dart
final user = ref.read(userProvider);
```

Why it's wrong:

- UI will NOT update on changes.

✔ Correct

```dart
final user = ref.watch(userProvider);
```

---

### 2. Using watch for side effects

❌ Wrong

```dart
final user = ref.watch(userProvider);
print(user);
```

Why it's wrong:

- Causes side effects during rebuilds.

✔ Correct

```dart
ref.listen(userProvider, (prev, next) {
  print(next);
});
```

---

### 3. Overusing invalidate

❌ Wrong

```dart
ref.invalidate(allProviders);
```

Why it's wrong:

- Breaks performance and cache efficiency.

✔ Correct

Invalidate only targeted providers.

---

## Related APIs

- ref.watch()
- ref.read()
- ref.listen()
- ref.invalidate()
- ref.refresh()
- ref.onDispose()
- Notifier
- AsyncNotifier
- StreamNotifier

---

## Summary

`ref` is the central dependency and lifecycle controller in Riverpod. It allows providers and widgets to interact with the dependency graph, enabling reactive updates, dependency injection, and lifecycle management. It is the backbone of Riverpod’s architecture.