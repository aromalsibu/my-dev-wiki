# What is Ref?

`Ref` is Riverpod’s internal access handle that allows providers and widgets to interact with the dependency graph, read other providers, and manage lifecycle-aware behavior.

---

## What is it?

`ref` (short for *reference*) is the object that connects a provider or widget to Riverpod’s system.

It is available inside:

- `Notifier`
- `AsyncNotifier`
- `StreamNotifier`
- Provider `build()` methods
- `ConsumerWidget` / `ConsumerState`

It gives access to:
- other providers
- lifecycle hooks
- dependency tracking
- state invalidation
- reactive updates

---

## Why does it exist?

Without `ref`, providers would be isolated and unable to:

- communicate with each other
- react to dependency changes
- fetch other providers safely
- manage lifecycle events

Before Riverpod, state management often relied on:
- global singletons
- manual dependency injection
- tightly coupled services

`ref` solves this by making the entire app a **reactive dependency graph**.

Each provider can safely depend on others without tight coupling.

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

- `ref.watch()` listens to dependency changes
- if `userRepositoryProvider` changes, this provider rebuilds

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

- `ref.watch()` makes the widget reactive
- UI rebuilds automatically on state change

---

### Reading Without Listening

```dart
final repo = ref.read(userRepositoryProvider);
```

Explanation:

- gets value once
- does NOT rebuild on changes

---

## Mental Model

Think of `ref` as the **bridge between providers and the dependency graph**.

```text
        Riverpod Graph
              │
              ▼
             ref
   ┌──────────┼──────────┐
   ▼          ▼          ▼
 watch       read      listen
   │          │          │
 reactive   one-time   side-effects
```

It controls how providers interact with everything else.

---

## Core APIs

### 1. ref.watch()

```dart
final user = ref.watch(userProvider);
```

- reactive dependency
- rebuilds when value changes

---

### 2. ref.read()

```dart
final repo = ref.read(repoProvider);
```

- one-time access
- no rebuilds

---

### 3. ref.listen()

```dart
ref.listen(userProvider, (prev, next) {
  print(next);
});
```

Explanation:

- used for side effects
- does NOT rebuild UI

---

### 4. ref.invalidate()

```dart
ref.invalidate(userProvider);
```

Explanation:

- forces provider to recompute
- clears cached state

---

### 5. ref.refresh()

```dart
final value = ref.refresh(userProvider);
```

Explanation:

- invalidates + recomputes immediately

---

### 6. ref.onDispose()

```dart
ref.onDispose(() {
  print("disposed");
});
```

Explanation:

- runs cleanup logic when provider is destroyed

---

## When to Use

Use `ref` when you need to:

- access other providers
- build dependencies between providers
- trigger refresh/invalidation
- listen for side effects
- inject services into notifiers
- manage lifecycle behavior

---

## When NOT to Use

Avoid misuse of `ref`:

- don’t use `read()` when you need reactive updates
- don’t put UI logic inside notifiers using `ref.listen`
- don’t overuse `invalidate()` (breaks caching benefits)

---

## Best Practices

- prefer `watch()` for reactive dependencies
- use `read()` for one-time operations
- use `listen()` only for side effects
- keep provider dependencies explicit in `build()`
- avoid unnecessary invalidations

---

## Common Mistakes

### 1. Using read instead of watch

❌ Wrong

```dart
final user = ref.read(userProvider);
```

- UI will NOT update

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
ref.invalidateAll();
```

- destroys caching system

✔ Correct

invalidate only specific providers

---

## Related APIs

- Notifier
- AsyncNotifier
- StreamNotifier
- ref.watch()
- ref.read()
- ref.listen()
- ref.invalidate()
- ref.refresh()

---

## Summary

`ref` is the core dependency access system in Riverpod. It allows providers and widgets to interact with the reactive graph, enabling dependency injection, state tracking, and lifecycle management. It is the foundation that connects all Riverpod components together.