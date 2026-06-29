# ref.invalidate()

`ref.invalidate()` marks a provider as invalid, causing Riverpod to discard its current state so that it will be recomputed the next time it is accessed.

---

## What is it?

`ref.invalidate()` is used to **reset a provider's current state**.

When a provider is invalidated:

- Its cached state is discarded.
- Its lifecycle is restarted.
- It is marked for recomputation.
- The next read or watch creates fresh state.

Think of it as telling Riverpod:

> "This provider's current state is no longer valid. Recreate it."

Unlike directly modifying state, `invalidate()` **rebuilds the entire provider**.

---

## Why does it exist?

Sometimes updating the current state isn't enough.

For example:

- A user logs out.
- Cached API data becomes stale.
- Application settings change.
- A dependency has fundamentally changed.
- You need to force a complete reload.

Instead of manually resetting every field, `ref.invalidate()` recreates the provider from the beginning by running its `build()` method again.

This keeps state management simple and predictable.

---

## Syntax

### Invalidating a Provider

```dart
ref.invalidate(userProvider);
```

Explanation:

- Marks `userProvider` as invalid.
- The provider will rebuild the next time it is accessed.

---

### Invalidating Inside a Notifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  Future<void> logout() async {
    await authRepository.logout();

    ref.invalidate(userProvider);
  }
}
```

Explanation:

- Logs the user out.
- Clears the existing user state.

---

### Invalidating Another Provider

```dart
ref.invalidate(cartProvider);
```

Explanation:

- Removes the current cart state.
- A new cart will be created when accessed again.

---

## Return Value

`ref.invalidate()` returns `void`.

Its purpose is to notify Riverpod that a provider's state should no longer be used.

---

## Execution Flow

```text
Provider has cached state
          │
          ▼
ref.invalidate(provider)
          │
          ▼
Riverpod discards cached state
          │
          ▼
Provider marked as invalid
          │
          ▼
Next watch/read
          │
          ▼
build() executes again
          │
          ▼
Fresh state created
```

Notice that `invalidate()` **does not immediately return a new value**.

It simply clears the existing provider.

---

## Mental Model

Think of `ref.invalidate()` as **clearing a cache**.

```text
Provider
     │
 Cached State
     │
     ▼
invalidate()
     │
     ▼
Cache Removed
     │
     ▼
Next Access
     │
     ▼
Fresh State
```

You're not updating the provider.

You're **throwing it away** so Riverpod creates a new one.

---

## Examples

### Refresh Cached Data

```dart
ref.invalidate(productsProvider);
```

Explanation:

- Removes cached products.
- Next access fetches fresh data.

---

### Logout

```dart
Future<void> logout() async {
  await authRepository.logout();

  ref.invalidate(userProvider);
}
```

Explanation:

- Clears authenticated user data.
- Forces a new authentication state.

---

### Clear Search Results

```dart
ref.invalidate(searchProvider);
```

Explanation:

- Removes previous search state.
- New searches start from a clean state.

---

### Real-World Example

```dart
Future<void> updateProfile(User user) async {
  await repository.update(user);

  ref.invalidate(userProvider);
}
```

Explanation:

- Updates the user on the server.
- Invalidates the cached profile.
- Next read fetches the latest data.

---

## When to Use

Use `ref.invalidate()` when:

- Cached data is outdated.
- The provider should restart completely.
- User logs out.
- Application settings change.
- Dependencies have fundamentally changed.

---

## When NOT to Use

Avoid `ref.invalidate()` when:

- You only need to update one field.
- State can be modified directly.
- Rebuilding the provider is unnecessary.

Instead:

- Update `state`
- Call notifier methods
- Use immutable state updates

---

## Best Practices

- Invalidate only the providers that need refreshing.
- Prefer updating `state` for small changes.
- Use invalidation for cache resets, not routine updates.
- Keep invalidation logic inside notifiers when possible.
- Avoid invalidating providers unnecessarily, as it causes recomputation.

---

## Common Mistakes

### 1. Using invalidate() for Simple State Updates

❌ Wrong

```dart
void increment() {
  ref.invalidate(counterProvider);
}
```

Why it's wrong:

- Recreates the entire provider.
- A simple state update is sufficient.

✔ Correct

```dart
void increment() {
  state++;
}
```

---

### 2. Expecting Immediate Data

❌ Wrong

```dart
ref.invalidate(userProvider);

final user = ref.read(userProvider);
```

Why it's wrong:

- `invalidate()` only clears the provider.
- Reading immediately may trigger recomputation, but invalidation itself does not return new data.

✔ Correct

Allow Riverpod to recreate the provider naturally, or use `ref.refresh()` if an immediate recomputation is required.

---

### 3. Invalidating Too Many Providers

❌ Wrong

```dart
ref.invalidate(userProvider);
ref.invalidate(cartProvider);
ref.invalidate(settingsProvider);
ref.invalidate(productsProvider);
```

Why it's wrong:

- Causes unnecessary recomputation.
- Can negatively affect performance.

✔ Correct

Invalidate only the providers whose state is actually stale.

---

### 4. Using invalidate() Instead of refresh()

❌ Wrong

```dart
ref.invalidate(productsProvider);

// Immediately expecting fresh products
```

Why it's wrong:

- `invalidate()` waits until the provider is accessed again.

✔ Correct

```dart
ref.refresh(productsProvider);
```

Use `refresh()` when you need an immediate recomputation.

---

## ref.invalidate() vs ref.refresh()

| Feature | `ref.invalidate()` | `ref.refresh()` |
|----------|--------------------|-----------------|
| Clears provider state | ✅ | ✅ |
| Immediately recreates provider | ❌ | ✅ |
| Waits until next access | ✅ | ❌ |
| Returns provider value | ❌ | ✅ |
| Best for cache invalidation | ✅ | ⚠️ Sometimes |

---

## Related APIs

- `ref.refresh()`
- `ref.invalidateSelf()`
- `ref.watch()`
- `ref.read()`
- `build()`

---

## Summary

`ref.invalidate()` tells Riverpod that a provider's current state is no longer valid. It discards the provider's cached state and causes the provider to be recreated the next time it is accessed. It is the preferred API for clearing stale caches, resetting providers after major application events, and forcing a clean provider lifecycle without immediately recomputing the value.