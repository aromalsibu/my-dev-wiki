# Ref Lifecycle

`Ref` exists only for the lifetime of its associated provider or widget, managing dependencies, lifecycle events, and interactions with Riverpod during that period.

---

## What is the Ref Lifecycle?

Every provider and consumer receives its own `Ref` object.

The `Ref` is created when the provider (or consumer) is initialized and is destroyed when that provider (or consumer) is disposed.

During its lifetime, `Ref` is responsible for:

- Reading other providers
- Watching dependencies
- Listening to state changes
- Invalidating providers
- Registering lifecycle callbacks
- Managing provider relationships

Once the provider is disposed, its `Ref` is also destroyed.

---

## Why does it exist?

Riverpod needs a way to manage communication between providers.

Instead of allowing providers to directly reference each other, Riverpod gives each provider its own `Ref`.

This provides several benefits:

- Automatic dependency tracking
- Safe lifecycle management
- Automatic cleanup
- Controlled access to providers
- Better performance through selective recomputation

Without `Ref`, Riverpod couldn't determine:

- Which providers depend on each other
- When to rebuild providers
- When providers should be disposed
- How to manage caching

---

## Lifecycle Stages

A `Ref` typically goes through the following stages.

```text
Provider Created
       │
       ▼
Ref Created
       │
       ▼
build() Executes
       │
       ▼
Provider Active
       │
       ▼
Dependencies Change
       │
       ▼
Provider Rebuilds
       │
       ▼
Last Listener Removed (.autoDispose)
       │
       ▼
onCancel()
       │
       ▼
Listener Returns?
   │            │
 Yes            No
   │            │
   ▼            ▼
onResume()   onDispose()
                   │
                   ▼
             Ref Destroyed
```

---

## Stage 1 — Ref Creation

When Riverpod creates a provider, it also creates a corresponding `Ref`.

```dart
final counterProvider = Provider((ref) {
  return 0;
});
```

Explanation:

- A new `Ref` instance is created.
- The provider can now interact with Riverpod.

---

## Stage 2 — Dependency Registration

As the provider executes, it may watch other providers.

```dart
final greetingProvider = Provider((ref) {
  final user = ref.watch(userProvider);

  return 'Hello ${user.name}';
});
```

Explanation:

- `greetingProvider` depends on `userProvider`.
- Riverpod records this dependency.

---

## Stage 3 — Provider Rebuild

If a watched dependency changes, Riverpod rebuilds the provider.

```text
userProvider changes
        │
        ▼
greetingProvider rebuilds
        │
        ▼
Same Ref instance continues
```

The provider gets a new computed value while continuing its lifecycle.

---

## Stage 4 — Lifecycle Events

For `.autoDispose` providers, `Ref` exposes lifecycle callbacks.

```dart
ref.onCancel(() {
  print('No listeners');
});

ref.onResume(() {
  print('Listener returned');
});

ref.onDispose(() {
  print('Provider disposed');
});
```

Explanation:

- `onCancel()` runs when the last listener is removed.
- `onResume()` runs if a listener returns.
- `onDispose()` runs before the provider is destroyed.

---

## Stage 5 — Disposal

Eventually, the provider is destroyed.

```text
Provider Disposed
       │
       ▼
onDispose()
       │
       ▼
Ref Destroyed
```

After disposal:

- The provider no longer exists.
- The `Ref` becomes invalid.
- Lifecycle callbacks cannot be registered.
- Dependencies are removed.

---

## Ref Lifecycle in Different Provider Types

| Provider Type | Ref Lifetime |
|---------------|--------------|
| `Provider` | Until the provider is disposed |
| `Notifier` | Until the notifier is disposed |
| `FutureProvider` | While the provider exists |
| `StreamProvider` | While the provider exists |
| `.autoDispose` providers | Until automatically disposed |
| `ConsumerWidget` | During each widget build |
| `ConsumerStatefulWidget` | While the widget exists |

---

## Lifecycle and Dependency Graph

```text
userProvider
      │
      ▼
profileProvider
      │
      ▼
dashboardProvider
```

If `userProvider` changes:

```text
userProvider
      │
      ▼
profileProvider rebuilds
      │
      ▼
dashboardProvider rebuilds
```

`Ref` is responsible for maintaining this dependency graph.

---

## Lifecycle APIs

Throughout its lifetime, a `Ref` can use different APIs.

| API | Purpose |
|------|---------|
| `watch()` | Register dependencies |
| `read()` | Read once |
| `listen()` | Observe changes |
| `invalidate()` | Clear provider state |
| `refresh()` | Recreate provider immediately |
| `keepAlive()` | Prevent auto-disposal |
| `onCancel()` | Last listener removed |
| `onResume()` | Listener returns |
| `onDispose()` | Final cleanup |

---

## Best Practices

- Keep providers focused on a single responsibility.
- Register lifecycle callbacks immediately after creating resources.
- Prefer `watch()` for reactive dependencies.
- Use `read()` only for one-time access.
- Clean up external resources with `onDispose()`.
- Use `.autoDispose` for short-lived providers.
- Avoid performing heavy work inside lifecycle callbacks.

---

## Common Mistakes

### 1. Using Ref After Disposal

❌ Wrong

```dart
ref.onDispose(() {
  ref.read(userProvider);
});
```

Why it's wrong:

- The provider is being destroyed.
- Accessing other providers during disposal can lead to unexpected behavior.

✔ Correct

Use `onDispose()` only for cleanup.

---

### 2. Confusing Widget Lifecycle with Ref Lifecycle

❌ Wrong

Assuming:

```text
Widget disposed
      │
      ▼
Provider disposed
```

Why it's wrong:

- Providers can outlive widgets.
- Multiple widgets may share the same provider.

✔ Correct

Treat provider and widget lifecycles as separate concepts.

---

### 3. Forgetting AutoDispose Behavior

❌ Wrong

Expecting an `.autoDispose` provider to remain alive forever.

Why it's wrong:

- It is disposed when no listeners remain unless kept alive.

✔ Correct

Use:

```dart
ref.keepAlive();
```

when caching is required.

---

### 4. Using read() Instead of watch()

❌ Wrong

```dart
final user = ref.read(userProvider);
```

When reactive updates are expected.

Why it's wrong:

- The provider won't rebuild.

✔ Correct

```dart
final user = ref.watch(userProvider);
```

---

## Ref Lifecycle vs Widget Lifecycle

| Feature | Ref Lifecycle | Widget Lifecycle |
|----------|---------------|------------------|
| Managed by | Riverpod | Flutter |
| Starts when | Provider is created | Widget is inserted |
| Ends when | Provider is disposed | Widget is removed |
| Tracks dependencies | ✅ | ❌ |
| Registers provider callbacks | ✅ | ❌ |
| Handles UI rendering | ❌ | ✅ |

---

## Related APIs

- `Ref`
- `WidgetRef`
- `ref.watch()`
- `ref.read()`
- `ref.listen()`
- `ref.onDispose()`
- `ref.onCancel()`
- `ref.onResume()`
- `.autoDispose`

---

## Summary

The `Ref` lifecycle begins when a provider or consumer is created and ends when it is disposed. During this time, `Ref` manages provider dependencies, lifecycle callbacks, state invalidation, and communication with Riverpod. Understanding this lifecycle is essential for writing efficient providers, preventing resource leaks, and taking full advantage of Riverpod's automatic dependency tracking and lifecycle management.