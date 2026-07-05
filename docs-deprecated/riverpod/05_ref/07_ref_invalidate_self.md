# ref.invalidateSelf()

`ref.invalidateSelf()` marks the current provider as invalid, causing Riverpod to discard its state and recreate it the next time it is accessed.

---

## What is it?

`ref.invalidateSelf()` is a convenience method used **inside a provider or notifier** to invalidate **itself**.

Instead of specifying the provider to invalidate, Riverpod already knows which provider owns the current `ref`.

For example, instead of:

```dart
ref.invalidate(userProvider);
```

You can simply write:

```dart
ref.invalidateSelf();
```

when you're already inside `userProvider`.

This makes the code cleaner and avoids referencing the provider itself.

---

## Why does it exist?

A provider often knows when **its own state** has become invalid.

For example:

- A cache has expired.
- Data needs to be reloaded.
- Authentication state changes.
- A retry operation is requested.

Without `invalidateSelf()`, the provider would need to reference itself, which is unnecessary and sometimes impossible.

`ref.invalidateSelf()` provides a simple, self-contained way for a provider to restart its own lifecycle.

---

## Syntax

### Invalidating the Current Provider

```dart
ref.invalidateSelf();
```

Explanation:

- Marks the current provider as invalid.
- Riverpod discards its cached state.
- The provider will rebuild on the next access.

---

### Inside an AsyncNotifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  Future<void> retry() async {
    ref.invalidateSelf();
  }

  @override
  Future<User> build() {
    return repository.fetchUser();
  }
}
```

Explanation:

- Invalidates the current notifier.
- The next access runs `build()` again.

---

### Retry After an Error

```dart
Future<void> reload() async {
  ref.invalidateSelf();
}
```

Explanation:

- Clears the current state.
- Triggers a fresh computation when the provider is accessed again.

---

## Return Value

`ref.invalidateSelf()` returns `void`.

Its only purpose is to notify Riverpod that the current provider should be discarded.

---

## Execution Flow

```text
Current Provider
        │
        ▼
ref.invalidateSelf()
        │
        ▼
Riverpod removes current state
        │
        ▼
Provider marked as invalid
        │
        ▼
Next access
        │
        ▼
build() executes again
        │
        ▼
Fresh state created
```

Unlike updating `state`, the provider starts from the beginning of its lifecycle.

---

## Mental Model

Think of `ref.invalidateSelf()` as **restarting yourself**.

```text
Current Provider
       │
       ▼
invalidateSelf()
       │
       ▼
Destroy Current State
       │
       ▼
Run build() Again
       │
       ▼
Fresh Provider Instance
```

You're telling Riverpod:

> "Throw me away and create a new version of me."

---

## Examples

### Simple Retry

```dart
Future<void> retry() async {
  ref.invalidateSelf();
}
```

Explanation:

- Discards the current provider.
- The next read starts over.

---

### Refresh Cached Data

```dart
class ProductsNotifier extends AsyncNotifier<List<Product>> {
  Future<void> refreshProducts() async {
    ref.invalidateSelf();
  }

  @override
  Future<List<Product>> build() {
    return repository.fetchProducts();
  }
}
```

Explanation:

- Clears cached products.
- Next access fetches fresh data.

---

### After Clearing Local Cache

```dart
Future<void> clearCache() async {
  await cache.clear();

  ref.invalidateSelf();
}
```

Explanation:

- Removes local cache.
- Forces a clean provider rebuild.

---

### Real-World Example

```dart
Future<void> retryPayment() async {
  await paymentRepository.resetSession();

  ref.invalidateSelf();
}
```

Explanation:

- Resets the payment session.
- Restarts the provider with fresh state.

---

## When to Use

Use `ref.invalidateSelf()` when:

- A provider needs to restart itself.
- Cached data has become stale.
- A retry action is required.
- Internal state should be recreated.
- You want to rerun `build()` from inside the provider.

---

## When NOT to Use

Avoid `ref.invalidateSelf()` when:

- Only part of the state changes.
- A simple state update is enough.
- You need to invalidate a different provider.

Instead:

- Update `state`
- Use `ref.invalidate(otherProvider)` for other providers.

---

## Best Practices

- Prefer `invalidateSelf()` over `invalidate(currentProvider)` when inside the provider.
- Use it for retries and cache invalidation.
- Keep invalidation logic inside notifiers.
- Avoid repeatedly invalidating the provider in short intervals.
- Update `state` instead of invalidating when only small changes are needed.

---

## Common Mistakes

### 1. Using invalidateSelf() for Normal State Changes

❌ Wrong

```dart
void increment() {
  ref.invalidateSelf();
}
```

Why it's wrong:

- Recreates the entire provider.
- A simple state update is faster.

✔ Correct

```dart
void increment() {
  state++;
}
```

---

### 2. Invalidating Instead of Updating

❌ Wrong

```dart
Future<void> updateName(String name) async {
  ref.invalidateSelf();
}
```

Why it's wrong:

- The provider is recreated unnecessarily.

✔ Correct

```dart
state = state.copyWith(name: name);
```

---

### 3. Expecting Immediate Rebuild

❌ Wrong

```dart
ref.invalidateSelf();

// Expecting build() to run immediately
```

Why it's wrong:

- The provider rebuilds when it is accessed again.

✔ Correct

Use `ref.refresh()` from another provider or widget if an immediate recomputation is required.

---

### 4. Invalidating Another Provider

❌ Wrong

```dart
ref.invalidateSelf();
```

When the intention is to refresh another provider.

Why it's wrong:

- Only the current provider is invalidated.

✔ Correct

```dart
ref.invalidate(otherProvider);
```

---

## ref.invalidateSelf() vs ref.invalidate()

| Feature | `ref.invalidateSelf()` | `ref.invalidate()` |
|----------|------------------------|--------------------|
| Invalidates current provider | ✅ | ❌ |
| Invalidates any provider | ❌ | ✅ |
| Used inside the provider | ✅ | ✅ |
| Requires provider argument | ❌ | ✅ |

---

## Related APIs

- `ref.invalidate()`
- `ref.refresh()`
- `build()`
- `ref.watch()`
- `ref.read()`

---

## Summary

`ref.invalidateSelf()` allows a provider to invalidate its own state without referencing itself explicitly. It discards the current provider instance and causes Riverpod to recreate it the next time it is accessed. It is especially useful for retries, cache invalidation, and restarting a provider's lifecycle from within the provider itself.