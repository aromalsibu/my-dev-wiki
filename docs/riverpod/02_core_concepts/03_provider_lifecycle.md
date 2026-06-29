# Provider Lifecycle

The provider lifecycle describes how Riverpod creates, caches, updates, and disposes providers throughout an application's execution.

---

## What is it?

Every provider in Riverpod goes through a series of lifecycle stages.

These stages determine:

- When a provider is created
- When its value is cached
- When it is reused
- When it is recomputed
- When it is disposed

Unlike traditional dependency injection systems where objects often live for the entire application, Riverpod creates providers **only when they are needed** and disposes them when appropriate.

Understanding the lifecycle helps you write efficient providers and avoid memory leaks or unnecessary recomputations.

---

## Why does it exist?

Imagine an application with hundreds of providers.

If Riverpod eagerly created every provider at startup:

- Startup time would increase.
- Memory usage would grow.
- Many providers would never be used.

Instead, Riverpod manages provider lifecycles automatically.

Benefits include:

- Lazy initialization
- Automatic caching
- Reduced memory usage
- Automatic cleanup
- Efficient recomputation
- Better application performance

---

## Syntax

A provider lifecycle is mostly automatic, but Riverpod exposes lifecycle APIs to react to lifecycle events.

### Basic Provider

```dart
final userProvider = Provider((ref) {
  return User();
});
```

Explanation:

- The provider is **not** created immediately.
- It is created only when first accessed.
- Riverpod caches the created value.

---

### Registering a Dispose Callback

```dart
final socketProvider = Provider((ref) {
  final socket = SocketConnection();

  ref.onDispose(() {
    socket.close();
  });

  return socket;
});
```

Explanation:

- `ref.onDispose()` registers cleanup logic.
- The callback runs when the provider is disposed.
- Useful for closing resources such as sockets or streams.

---

### Invalidating a Provider

```dart
ref.invalidate(userProvider);
```

Explanation:

- Invalidates the cached provider.
- The current value is discarded.
- The provider is recreated the next time it is read.

---

## Mental Model

Think of every provider as moving through a lifecycle.

```text
           First Read
               │
               ▼
          Create Provider
               │
               ▼
         Cache the Value
               │
               ▼
        Consumers Read It
               │
        Dependency Changes
               │
               ▼
         Recompute Value
               │
       No Longer Needed
               │
               ▼
          Dispose Provider
```

Riverpod manages this entire process automatically.

---

## Lifecycle Stages

### 1. Uninitialized

```text
Provider Exists
      │
      ▼
Never Read
```

At this stage:

- No value exists.
- No object has been created.
- No memory is allocated.

---

### 2. Created

```text
First Read
      │
      ▼
Provider Executes
```

Riverpod executes the provider function.

For example:

```dart
final userProvider = Provider((ref) {
  print('Creating user');

  return User();
});
```

The provider is created only once unless it is invalidated or disposed.

---

### 3. Cached

```text
Provider Created
       │
       ▼
Value Stored
```

After creation:

- Riverpod stores the value.
- Future reads reuse the same instance.
- The provider does not execute again unless necessary.

---

### 4. Active

```text
Widgets
    │
    ▼
Watching Provider
```

When consumers watch a provider:

- Riverpod tracks dependencies.
- Changes trigger updates.
- The provider remains alive while needed.

---

### 5. Recomputed

A provider may execute again when:

- One of its dependencies changes.
- It is refreshed.
- It is invalidated.

Example:

```dart
final fullNameProvider = Provider((ref) {
  final firstName = ref.watch(firstNameProvider);

  return '$firstName Doe';
});
```

If `firstNameProvider` changes, `fullNameProvider` is recomputed automatically.

---

### 6. Disposed

```text
No Consumers
      │
      ▼
Dispose
```

Riverpod removes the provider when:

- An `autoDispose` provider is no longer used.
- The owning `ProviderContainer` is disposed.
- The provider is invalidated and replaced.

During disposal:

- `ref.onDispose()` callbacks execute.
- Resources are cleaned up.
- Cached values are removed.

---

## Examples

### Lazy Initialization

```dart
final repositoryProvider = Provider((ref) {
  print('Repository created');

  return UserRepository();
});
```

Explanation:

- Nothing happens until the provider is read.
- The message prints only on the first access.

---

### Automatic Caching

```dart
final repository = ref.watch(repositoryProvider);

final repository2 = ref.watch(repositoryProvider);
```

Explanation:

- The provider executes only once.
- Both variables reference the same cached instance.

---

### Recomputing a Provider

```dart
ref.invalidate(repositoryProvider);
```

Explanation:

- Removes the cached value.
- The provider is recreated on the next read.

---

### Cleaning Up Resources

```dart
final streamProvider = Provider((ref) {
  final controller = StreamController<int>();

  ref.onDispose(() {
    controller.close();
  });

  return controller;
});
```

Explanation:

- The stream controller is closed when the provider is disposed.
- Prevents resource leaks.

---

## When to Use

Understanding the provider lifecycle is important when:

- Managing expensive objects
- Creating repositories
- Working with streams
- Using sockets
- Managing database connections
- Implementing caching
- Cleaning up resources

---

## When NOT to Use

You usually do **not** need to manage the lifecycle manually.

Avoid:

- Manually caching provider values.
- Manually recreating providers.
- Manually disposing providers outside Riverpod.

Let Riverpod handle the lifecycle whenever possible.

---

## Best Practices

- Let providers be lazily created.
- Register cleanup using `ref.onDispose()`.
- Use `ref.invalidate()` when data becomes stale.
- Keep providers lightweight and focused.
- Avoid side effects during provider creation unless necessary.

---

## Common Mistakes

### 1. Assuming Providers Are Created at Startup

❌ Wrong

```dart
final repositoryProvider = Provider((ref) {
  print('Created');
  return Repository();
});
```

Why it's wrong:

- The provider is **not** created when the app starts.
- It is created only when first accessed.

✔ Correct

Understand that providers are lazily initialized.

---

### 2. Forgetting Cleanup

❌ Wrong

```dart
final socketProvider = Provider((ref) {
  return SocketConnection();
});
```

Why it's wrong:

- Open resources remain alive.
- Can cause memory leaks.

✔ Correct

```dart
final socketProvider = Provider((ref) {
  final socket = SocketConnection();

  ref.onDispose(socket.close);

  return socket;
});
```

---

### 3. Manually Caching Values

❌ Wrong

```dart
UserRepository? repository;

final repositoryProvider = Provider((ref) {
  repository ??= UserRepository();

  return repository!;
});
```

Why it's wrong:

- Riverpod already caches providers.
- Manual caching is unnecessary and error-prone.

✔ Correct

```dart
final repositoryProvider = Provider((ref) {
  return UserRepository();
});
```

Let Riverpod manage the cache.

---

## Related APIs

- Provider
- ProviderContainer
- ProviderScope
- ref.watch()
- ref.read()
- ref.invalidate()
- ref.refresh()
- ref.onDispose()
- Auto Dispose

---

## Summary

The provider lifecycle defines how Riverpod manages providers from creation to disposal. Providers are lazily created, automatically cached, recomputed when dependencies change, and disposed when no longer needed. By understanding this lifecycle, you can build efficient, scalable, and resource-friendly applications while letting Riverpod handle most lifecycle management automatically.