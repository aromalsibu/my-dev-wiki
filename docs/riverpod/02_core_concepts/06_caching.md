# Caching

Caching is Riverpod's mechanism for storing provider values so they can be reused instead of being recreated every time they are accessed.

---

## What is it?

When a provider is read for the first time, Riverpod executes the provider's build function and stores the resulting value.

Subsequent reads return the **cached value** instead of executing the provider again.

For example:

```text
First Read
     │
     ▼
Create Provider
     │
     ▼
Store Value
     │
     ▼
Future Reads
     │
     ▼
Return Cached Value
```

This behavior is automatic and applies to all provider types.

---

## Why does it exist?

Imagine a provider that creates an expensive object.

```dart
final repositoryProvider = Provider((ref) {
  print('Creating repository');

  return UserRepository();
});
```

Without caching:

```text
Read #1
Create Repository

Read #2
Create Repository

Read #3
Create Repository
```

A new repository would be created every time the provider is accessed.

With caching:

```text
Read #1
Create Repository

Read #2
Use Cached Repository

Read #3
Use Cached Repository
```

Benefits include:

- Better performance
- Less memory allocation
- Shared provider instances
- Reduced database queries
- Fewer API requests
- Efficient dependency reuse

---

## Syntax

Caching happens automatically.

### Basic Example

```dart
final repositoryProvider = Provider((ref) {
  print('Repository created');

  return UserRepository();
});
```

Explanation:

- The provider executes only on the first read.
- Riverpod caches the created `UserRepository`.
- Later reads reuse the same instance.

---

### Reading Multiple Times

```dart
final repo1 = ref.watch(repositoryProvider);
final repo2 = ref.watch(repositoryProvider);
```

Explanation:

- The provider executes only once.
- Both variables reference the same cached object.

---

### Cache Invalidation

```dart
ref.invalidate(repositoryProvider);
```

Explanation:

- Removes the cached value.
- The provider is recreated the next time it is accessed.

---

### Immediate Refresh

```dart
final repository = ref.refresh(repositoryProvider);
```

Explanation:

- Clears the cache.
- Recreates the provider immediately.
- Returns the new value.

---

## Mental Model

Think of Riverpod's cache as a storage room.

```text
              Provider
                  │
         First Access
                  │
                  ▼
          Create Value
                  │
                  ▼
         Store in Cache
                  │
         ┌────────┴────────┐
         ▼                 ▼
    Widget A          Widget B
         │                 │
         └──── Same Object ────┘
```

Instead of creating a new object every time, Riverpod reuses the cached value.

---

## How Caching Works

### Step 1: First Read

```text
Provider
    │
Not Cached
    │
Create Value
```

The provider executes.

---

### Step 2: Store Value

```text
Provider
    │
Cache Value
```

The result is stored in the `ProviderContainer`.

---

### Step 3: Future Reads

```text
Provider
    │
Cached?
    │
Yes
    │
Return Cached Value
```

No recomputation occurs.

---

### Step 4: Cache Removed

The cache is removed when:

- `ref.invalidate()` is called.
- `ref.refresh()` is called.
- An `autoDispose` provider is disposed.
- A dependency changes.
- The `ProviderContainer` is disposed.

---

## Examples

### Expensive Repository

```dart
final repositoryProvider = Provider((ref) {
  print('Repository created');

  return UserRepository();
});
```

Explanation:

- The repository is created only once.
- Every consumer receives the cached instance.

---

### Computed Provider

```dart
final firstNameProvider = Provider((ref) => 'John');
final lastNameProvider = Provider((ref) => 'Doe');

final fullNameProvider = Provider((ref) {
  return '${ref.watch(firstNameProvider)} ${ref.watch(lastNameProvider)}';
});
```

Explanation:

- `fullNameProvider` caches its computed value.
- It is recomputed only if one of its dependencies changes.

---

### FutureProvider

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- The asynchronous result is cached.
- Multiple widgets share the same request and result.

---

## What Triggers Cache Recreation?

A cached provider is recreated when:

- One of its dependencies changes.
- `ref.invalidate()` is called.
- `ref.refresh()` is called.
- An `autoDispose` provider is recreated.
- The provider is overridden.
- A new `ProviderContainer` is created.

---

## When to Use

Caching is automatic and beneficial for:

- API services
- Repositories
- Database connections
- Computed values
- Configuration objects
- Dependency injection
- Expensive calculations

---

## When NOT to Rely on Caching

Do not rely on cached values when:

- Data changes frequently.
- Fresh API data is always required.
- User-specific information changes.
- Cache expiration must be enforced.

In these cases, invalidate or refresh the provider as needed.

---

## Best Practices

- Let Riverpod manage caching automatically.
- Avoid manually caching provider values.
- Invalidate stale data instead of creating new providers.
- Keep providers pure so cached values remain predictable.
- Use `autoDispose` for temporary providers that shouldn't stay cached indefinitely.

---

## Common Mistakes

### 1. Manual Caching

❌ Wrong

```dart
UserRepository? repository;

final repositoryProvider = Provider((ref) {
  repository ??= UserRepository();

  return repository!;
});
```

Why it's wrong:

- Riverpod already caches provider values.
- Manual caching adds unnecessary complexity.

✔ Correct

```dart
final repositoryProvider = Provider((ref) {
  return UserRepository();
});
```

---

### 2. Assuming Providers Execute Every Read

❌ Wrong

```dart
final repository = ref.watch(repositoryProvider);
final repository2 = ref.watch(repositoryProvider);
```

Why it's wrong:

- The provider executes only once.
- Both variables reference the same cached instance.

✔ Correct

Understand that Riverpod automatically reuses cached values until the cache is invalidated.

---

### 3. Forgetting to Invalidate Stale Data

❌ Wrong

```dart
await repository.updateUser(user);
```

Why it's wrong:

- The provider still returns the old cached value.

✔ Correct

```dart
await repository.updateUser(user);

ref.invalidate(userProvider);
```

Invalidate the provider so it loads fresh data.

---

## Related APIs

- Provider
- ProviderContainer
- Provider Lifecycle
- ref.invalidate()
- ref.refresh()
- Auto Dispose
- FutureProvider
- StreamProvider

---

## Summary

Caching is a core feature of Riverpod that stores provider values after their first creation and reuses them for future reads. This improves performance, reduces unnecessary object creation, and ensures that consumers share the same provider instance. Riverpod automatically manages caching, recreating providers only when their dependencies change or when the cache is explicitly invalidated or refreshed.