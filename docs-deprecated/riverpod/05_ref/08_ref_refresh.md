# ref.refresh()

`ref.refresh()` immediately invalidates a provider, recreates it, and returns its new value in a single operation.

---

## What is it?

`ref.refresh()` is used to **force a provider to recompute immediately**.

Unlike `ref.invalidate()`, which only marks a provider as invalid, `ref.refresh()`:

1. Invalidates the provider.
2. Immediately recreates it.
3. Returns the newly computed value.

Think of it as a shortcut for:

```dart
ref.invalidate(provider);
ref.read(provider);
```

except it happens atomically and is handled internally by Riverpod.

---

## Why does it exist?

Sometimes waiting until the next access isn't desirable.

For example:

- A user performs pull-to-refresh.
- Data must be fetched immediately.
- A retry button is pressed.
- Cached information should be replaced right now.

Using `ref.invalidate()` would only clear the provider's state.

`ref.refresh()` ensures the provider is recreated immediately, making it ideal for user-initiated refresh operations.

---

## Syntax

### Refreshing a Provider

```dart
final user = ref.refresh(userProvider);
```

Explanation:

- Discards the current provider state.
- Rebuilds the provider immediately.
- Returns the newly computed value.

---

### Refreshing an Async Provider

```dart
final user = ref.refresh(userProvider);
```

Explanation:

- Returns a new `AsyncValue<User>`.
- The provider begins loading immediately.

---

### Pull to Refresh

```dart
RefreshIndicator(
  onRefresh: () async {
    ref.refresh(productsProvider);
  },
  child: ProductList(),
)
```

Explanation:

- User pulls down.
- Products provider is recreated immediately.

---

## Return Value

Unlike `ref.invalidate()`, `ref.refresh()` returns the provider's new value.

Examples:

| Provider Type | Return Value |
|---------------|--------------|
| `Provider<int>` | `int` |
| `Provider<String>` | `String` |
| `NotifierProvider<Counter, int>` | `int` |
| `FutureProvider<User>` | `AsyncValue<User>` |
| `AsyncNotifierProvider<UserNotifier, User>` | `AsyncValue<User>` |
| `StreamProvider<Message>` | `AsyncValue<Message>` |

The returned value is the same type that `ref.watch()` or `ref.read()` would return.

---

## Execution Flow

```text
Provider has state
        │
        ▼
ref.refresh(provider)
        │
        ▼
Current state discarded
        │
        ▼
build() executes immediately
        │
        ▼
New state created
        │
        ▼
New value returned
```

Unlike `invalidate()`, recomputation happens immediately.

---

## Mental Model

Think of `ref.refresh()` as **restarting a provider instantly**.

```text
Current Provider
        │
        ▼
 refresh()
        │
        ▼
Old State Removed
        │
        ▼
New Provider Created
        │
        ▼
Fresh Value Returned
```

Instead of waiting for the next access, the provider is recreated immediately.

---

## Examples

### Refresh Cached Data

```dart
ref.refresh(productsProvider);
```

Explanation:

- Removes cached products.
- Fetches fresh products immediately.

---

### Retry After an Error

```dart
ElevatedButton(
  onPressed: () {
    ref.refresh(userProvider);
  },
  child: const Text('Retry'),
)
```

Explanation:

- Recreates the provider.
- Starts the request again.

---

### Pull-to-Refresh

```dart
RefreshIndicator(
  onRefresh: () async {
    ref.refresh(newsProvider);
  },
  child: const NewsList(),
)
```

Explanation:

- User requests fresh data.
- Provider begins loading immediately.

---

### Real-World Example

```dart
Future<void> refreshProfile() async {
  ref.refresh(profileProvider);
}
```

Explanation:

- Clears cached profile.
- Immediately requests the latest profile from the server.

---

## When to Use

Use `ref.refresh()` when:

- Implementing pull-to-refresh.
- Retrying failed API requests.
- Refreshing cached data immediately.
- User explicitly requests fresh data.
- Immediate recomputation is required.

---

## When NOT to Use

Avoid `ref.refresh()` when:

- Updating a small piece of state.
- Waiting until the next access is acceptable.
- The provider doesn't need to restart.

Instead:

- Update `state`.
- Use `ref.invalidate()` if immediate recomputation isn't necessary.

---

## Best Practices

- Use `refresh()` for user-initiated refresh actions.
- Keep refresh logic inside notifiers or UI events.
- Avoid repeatedly refreshing providers in short intervals.
- Prefer updating `state` for simple state changes.
- Refresh only the providers that actually need new data.

---

## Common Mistakes

### 1. Using refresh() for Normal State Updates

❌ Wrong

```dart
void increment() {
  ref.refresh(counterProvider);
}
```

Why it's wrong:

- The provider is recreated unnecessarily.

✔ Correct

```dart
void increment() {
  state++;
}
```

---

### 2. Using invalidate() When Immediate Data Is Needed

❌ Wrong

```dart
ref.invalidate(productsProvider);
```

Why it's wrong:

- The provider won't rebuild until it is accessed again.

✔ Correct

```dart
ref.refresh(productsProvider);
```

---

### 3. Refreshing Multiple Providers Unnecessarily

❌ Wrong

```dart
ref.refresh(userProvider);
ref.refresh(cartProvider);
ref.refresh(settingsProvider);
ref.refresh(productsProvider);
```

Why it's wrong:

- Causes unnecessary recomputation.
- May result in multiple network requests.

✔ Correct

Refresh only the providers that need updated data.

---

### 4. Expecting refresh() to Update Other Providers

❌ Wrong

```dart
ref.refresh(userProvider);

// Expecting cartProvider to refresh too
```

Why it's wrong:

- Only the specified provider is refreshed.

✔ Correct

Refresh each provider explicitly or design provider dependencies so dependent providers rebuild automatically.

---

## ref.refresh() vs ref.invalidate()

| Feature | `ref.refresh()` | `ref.invalidate()` |
|----------|-----------------|--------------------|
| Clears provider state | ✅ | ✅ |
| Immediately recreates provider | ✅ | ❌ |
| Returns new provider value | ✅ | ❌ |
| Waits until next access | ❌ | ✅ |
| Best for pull-to-refresh | ✅ | ❌ |

---

## Related APIs

- `ref.invalidate()`
- `ref.invalidateSelf()`
- `ref.watch()`
- `ref.read()`
- `build()`

---

## Summary

`ref.refresh()` immediately discards a provider's current state, recreates it, and returns the new value. It is the preferred API for pull-to-refresh, retry actions, and any scenario where fresh state is required immediately. Unlike `ref.invalidate()`, which waits until the next access, `ref.refresh()` performs the entire refresh process in a single operation.