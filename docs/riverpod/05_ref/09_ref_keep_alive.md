# ref.keepAlive()

`ref.keepAlive()` prevents an `.autoDispose` provider from being automatically disposed, allowing its state to remain alive even when no longer being listened to.

---

## What is it?

`ref.keepAlive()` is a lifecycle API that **overrides the default behavior of `.autoDispose` providers**.

Normally, an `.autoDispose` provider is destroyed when:

- No widgets are watching it.
- No other providers depend on it.

By calling `ref.keepAlive()`, you tell Riverpod:

> "Don't dispose this provider when the last listener is removed."

This allows the provider to preserve its state, avoiding unnecessary recomputation or repeated network requests.

---

## Why does it exist?

`.autoDispose` is excellent for freeing unused resources, but there are situations where disposing immediately is undesirable.

For example:

- A network request has already completed.
- Data should remain cached while navigating between screens.
- Expensive computations shouldn't run repeatedly.
- The provider's state should survive temporary UI changes.

`ref.keepAlive()` gives you the flexibility to **combine automatic disposal with selective caching**.

---

## Availability

✅ **Only available inside `.autoDispose` providers.**

It can be used in:

- `Provider.autoDispose`
- `FutureProvider.autoDispose`
- `StreamProvider.autoDispose`
- `NotifierProvider.autoDispose`
- `AsyncNotifierProvider.autoDispose`
- `StreamNotifierProvider.autoDispose`

It is **not available** in regular (non-autoDispose) providers because they are already kept alive by default.

---

## Syntax

### Keeping a Provider Alive

```dart
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  final user = await repository.fetchUser();

  ref.keepAlive();

  return user;
});
```

Explanation:

- The provider is `.autoDispose`.
- After successfully fetching the user, `keepAlive()` prevents disposal.
- Future reads reuse the cached value.

---

### Storing the KeepAlive Link

```dart
final link = ref.keepAlive();
```

Explanation:

- Returns a `KeepAliveLink`.
- The link can later be closed to allow disposal again.

---

### Allowing Disposal Later

```dart
final link = ref.keepAlive();

Timer(
  const Duration(minutes: 5),
  () => link.close(),
);
```

Explanation:

- Keeps the provider alive temporarily.
- After five minutes, disposal is allowed again.

---

## Return Value

`ref.keepAlive()` returns a `KeepAliveLink`.

The link provides manual control over the keep-alive behavior.

Common method:

| Method | Description |
|---------|-------------|
| `close()` | Releases the keep-alive request and allows auto-disposal again. |

If the returned link isn't needed, it can simply be ignored.

---

## Execution Flow

```text
.autoDispose Provider
        │
        ▼
Last Listener Removed
        │
        ▼
keepAlive() called?
      │
 ┌────┴────┐
 │         │
No        Yes
 │         │
 ▼         ▼
Dispose   Keep State
            │
            ▼
Future Reads Reuse Cache
```

---

## Mental Model

Think of `.autoDispose` providers as **automatic doors**.

Normally:

```text
Last Visitor Leaves
        │
        ▼
Door Closes
        │
        ▼
Provider Disposed
```

With `keepAlive()`:

```text
Last Visitor Leaves
        │
        ▼
keepAlive()
        │
        ▼
Door Stays Open
        │
        ▼
Provider Remains Alive
```

You're telling Riverpod:

> "Even if nobody is watching me right now, don't throw me away."

---

## Examples

### Cache API Response

```dart
final productsProvider =
    FutureProvider.autoDispose<List<Product>>((ref) async {
  final products = await repository.fetchProducts();

  ref.keepAlive();

  return products;
});
```

Explanation:

- Products are fetched once.
- Returning to the screen reuses cached data.

---

### Cache Only After Success

```dart
final profile = await repository.fetchProfile();

ref.keepAlive();

return profile;
```

Explanation:

- Failed requests are not cached.
- Only successful responses remain alive.

---

### Temporary Cache

```dart
final link = ref.keepAlive();

Future.delayed(
  const Duration(minutes: 10),
  () => link.close(),
);
```

Explanation:

- Keeps data alive for 10 minutes.
- Afterwards, normal auto-dispose behavior resumes.

---

### Real-World Example

```dart
final newsProvider =
    FutureProvider.autoDispose<List<Article>>((ref) async {
  final news = await repository.fetchLatestNews();

  ref.keepAlive();

  return news;
});
```

Explanation:

- User opens the News screen.
- Articles are downloaded once.
- Navigating away and back does not trigger another network request.

---

## When to Use

Use `ref.keepAlive()` when:

- Caching successful API responses.
- Preserving expensive computations.
- Preventing repeated network requests.
- Keeping state during temporary navigation.
- Reducing unnecessary provider recreation.

---

## When NOT to Use

Avoid `ref.keepAlive()` when:

- The state should always be discarded.
- Sensitive data should be removed immediately (e.g., authentication tokens).
- Memory usage is more important than performance.
- The provider should behave exactly like a normal `.autoDispose` provider.

---

## Best Practices

- Use `keepAlive()` only when caching provides a real benefit.
- Prefer caching successful results rather than failed requests.
- Store the `KeepAliveLink` if the cache should expire later.
- Don't disable auto-dispose everywhere—use it selectively.
- Combine `keepAlive()` with `.autoDispose` to balance memory usage and performance.

---

## Common Mistakes

### 1. Calling keepAlive() in a Non-autoDispose Provider

❌ Wrong

```dart
final counterProvider = Provider((ref) {
  ref.keepAlive();
  return 0;
});
```

Why it's wrong:

- Regular providers are already kept alive.
- `keepAlive()` is only available for `.autoDispose` providers.

✔ Correct

```dart
final counterProvider =
    Provider.autoDispose((ref) {
  ref.keepAlive();

  return 0;
});
```

---

### 2. Keeping Every Provider Alive

❌ Wrong

```dart
ref.keepAlive();
```

In every `.autoDispose` provider.

Why it's wrong:

- Defeats the purpose of `.autoDispose`.
- Increases memory usage unnecessarily.

✔ Correct

Keep alive only providers that benefit from caching.

---

### 3. Forgetting to Close the KeepAliveLink

❌ Wrong

```dart
final link = ref.keepAlive();
```

When temporary caching is intended.

Why it's wrong:

- The provider stays alive indefinitely.

✔ Correct

```dart
final link = ref.keepAlive();

Future.delayed(
  const Duration(minutes: 5),
  () => link.close(),
);
```

---

### 4. Using keepAlive() Instead of State Management

❌ Wrong

Using `keepAlive()` to avoid updating state correctly.

Why it's wrong:

- `keepAlive()` controls lifecycle, not state updates.

✔ Correct

Use `state` to update data and `keepAlive()` only to control disposal.

---

## ref.keepAlive() vs .keepAlive Modifier

| Feature | `ref.keepAlive()` | `keepAlive: true` (Code Generation) |
|----------|-------------------|-------------------------------------|
| Called at runtime | ✅ | ❌ |
| Conditional | ✅ | ❌ |
| Manual control | ✅ | ❌ |
| Returns `KeepAliveLink` | ✅ | ❌ |
| Best for dynamic caching | ✅ | ❌ |
| Best for always-on providers | ❌ | ✅ |

> **Note:** If you're using Riverpod code generation, `@Riverpod(keepAlive: true)` keeps the provider alive permanently, while `ref.keepAlive()` allows runtime decisions.

---

## Related APIs

- `.autoDispose`
- `ref.onDispose()`
- `ref.onCancel()`
- `ref.onResume()`
- `KeepAliveLink`

---

## Summary

`ref.keepAlive()` allows an `.autoDispose` provider to remain alive even after its last listener is removed. It is primarily used to cache successful results, avoid repeated work, and improve performance while still benefiting from `.autoDispose`. When greater control is needed, the returned `KeepAliveLink` can be used to determine exactly when the provider becomes disposable again.