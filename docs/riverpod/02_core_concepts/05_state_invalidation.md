# State Invalidation

State invalidation is the process of marking a provider's cached value as stale, causing Riverpod to recreate it the next time it is accessed.

---

## What is it?

Riverpod caches provider values to avoid unnecessary recomputation. Sometimes, however, that cached value is no longer valid.

For example:

- A user logs out.
- Data on the server has changed.
- Settings are updated.
- A cache expires.

In these situations, you need a way to tell Riverpod:

> "This provider's current value is outdated. Throw it away and create a fresh one."

This process is called **state invalidation**.

Riverpod provides APIs such as:

- `ref.invalidate()`
- `ref.invalidateSelf()`

to invalidate providers.

---

## Why does it exist?

Without invalidation, providers would continue returning cached values forever.

Imagine a provider that fetches user information.

```text
First Read
     │
     ▼
Fetch User
     │
     ▼
Cache Result
     │
     ▼
User Updates Profile
     │
     ▼
Still Returns Old Data ❌
```

The application would display outdated information.

State invalidation solves this by clearing the cached value so Riverpod can compute a fresh one.

Benefits include:

- Fresh application data
- Explicit cache control
- Better synchronization with external systems
- Reduced unnecessary network requests

---

## Syntax

### Invalidating Another Provider

```dart
ref.invalidate(userProvider);
```

Explanation:

- Marks `userProvider` as stale.
- Removes its cached value.
- The provider is recreated the next time it is read or watched.

---

### Invalidating the Current Provider

```dart
ref.invalidateSelf();
```

Explanation:

- Invalidates the currently executing provider.
- Riverpod recreates it when needed.
- Useful for self-refreshing providers.

---

### Invalidating After an Update

```dart
Future<void> updateUser(User user) async {
  await repository.update(user);

  ref.invalidate(userProvider);
}
```

Explanation:

- The user is updated in the repository.
- The cached provider is cleared.
- The next read fetches the latest data.

---

## Mental Model

Think of a provider cache like a whiteboard.

```text
Provider
    │
    ▼
Cached Value
    │
    ▼
invalidate()
    │
    ▼
Erase Whiteboard
    │
    ▼
Next Read
    │
    ▼
Create New Value
```

Invalidation does **not** immediately recreate the provider.

It simply removes the current cached value.

The next access triggers a new computation.

---

## Invalidation vs Refresh

These two APIs are often confused.

### `invalidate()`

```text
Provider
    │
Remove Cache
    │
Wait...
    │
Next Read
    │
Create Again
```

The provider is recreated **later**, only when someone accesses it.

---

### `refresh()`

```text
Provider
    │
Remove Cache
    │
Immediately Create Again
```

The provider is recreated **immediately**, and the new value is returned.

---

## Examples

### Refreshing User Data

```dart
class UserNotifier extends Notifier<User> {
  @override
  User build() {
    return repository.getCurrentUser();
  }

  Future<void> refreshUser() async {
    ref.invalidateSelf();
  }
}
```

Explanation:

- The notifier invalidates itself.
- Riverpod recreates it when it is accessed again.

---

### Invalidating After Login

```dart
Future<void> login() async {
  await authRepository.login();

  ref.invalidate(userProvider);
}
```

Explanation:

- Authentication changes the current user.
- Invalidating the provider ensures the next read returns updated user information.

---

### Invalidating Multiple Providers

```dart
ref.invalidate(userProvider);
ref.invalidate(orderProvider);
ref.invalidate(cartProvider);
```

Explanation:

- Each provider's cached value is cleared.
- They are recreated independently when next accessed.

---

## Common Scenarios

State invalidation is useful when:

- A user logs in or out.
- Profile information changes.
- API data becomes outdated.
- Cache expiration occurs.
- Local database changes.
- Settings are updated.
- Language or theme changes.
- A dependency has changed.

---

## When to Use

Use state invalidation when:

- Cached data is no longer valid.
- External systems modify your data.
- You need to force a provider to fetch fresh information.
- Updating one provider should cause another to reload.

---

## When NOT to Use

Do not invalidate providers:

- To simply read the latest cached value.
- When no underlying data has changed.
- As a replacement for updating state inside a `Notifier`.

If you're modifying a notifier's own state, prefer:

```dart
state = newValue;
```

instead of invalidating the entire provider.

---

## Best Practices

- Invalidate only the providers that are affected.
- Prefer updating `state` directly inside notifiers when possible.
- Use invalidation for stale or externally changed data.
- Avoid invalidating providers unnecessarily.
- Understand the difference between `invalidate()` and `refresh()`.

---

## Common Mistakes

### 1. Assuming `invalidate()` Recreates Immediately

❌ Wrong

```dart
ref.invalidate(userProvider);
```

Why it's wrong:

- The provider is **not** recreated immediately.
- It waits until the next read or watch.

✔ Correct

Use `ref.refresh()` if you need an immediate recomputation.

---

### 2. Invalidating Everything

❌ Wrong

```dart
ref.invalidate(userProvider);
ref.invalidate(orderProvider);
ref.invalidate(settingsProvider);
ref.invalidate(cartProvider);
```

Why it's wrong:

- Causes unnecessary recomputation.
- Can impact performance.

✔ Correct

Invalidate only the providers affected by the change.

---

### 3. Invalidating Instead of Updating State

❌ Wrong

```dart
class Counter extends Notifier<int> {
  void increment() {
    ref.invalidateSelf();
  }
}
```

Why it's wrong:

- The provider is recreated instead of simply updating its state.
- Less efficient.

✔ Correct

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Update state directly when possible.

---

## Related APIs

- ref.invalidate()
- ref.invalidateSelf()
- ref.refresh()
- ref.watch()
- Provider Lifecycle
- Caching
- Auto Dispose

---

## Summary

State invalidation allows Riverpod to discard a provider's cached value when it becomes outdated. Instead of immediately recreating the provider, `ref.invalidate()` marks it as stale so it can be recomputed the next time it is accessed. This mechanism helps keep application data fresh while preserving Riverpod's efficient caching behavior.