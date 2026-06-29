# keepAlive

Prevents a provider from being disposed when it is no longer listened to.

---

# What is it?

`keepAlive` is a Riverpod mechanism that keeps an otherwise disposable provider alive even after it has no active listeners.

Normally, providers marked with `.autoDispose` are destroyed when unused. `keepAlive` overrides this behavior and tells Riverpod to retain the provider in memory.

It is typically used through `ref.keepAlive()` inside a provider.

---

# Why does it exist?

By default, Riverpod aggressively disposes providers to free memory.

However, in some cases, you may want to:

- Cache expensive computations
- Preserve state between navigations
- Avoid refetching data repeatedly
- Keep background data alive temporarily

Without `keepAlive`, providers would be recreated every time they are re-accessed.

---

# Syntax

## Basic Usage

```dart
final dataProvider = FutureProvider.autoDispose((ref) async {
  final link = ref.keepAlive();

  final data = await api.fetchData();

  return data;
});
```

Explanation:

- `ref.keepAlive()` prevents disposal of this provider.
- The provider remains alive even when no widgets are listening.
- Useful for caching async results.

---

## Using keepAlive with Disposal Control

```dart
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  final link = ref.keepAlive();

  final user = await api.fetchUser();

  link.close();

  return user;
});
```

Explanation:

- `link.close()` manually allows disposal again.
- You can dynamically control lifecycle behavior.

---

## Conditional KeepAlive

```dart
final configProvider = Provider.autoDispose((ref) {
  final link = ref.keepAlive();

  if (shouldCacheConfig) {
    return configService.load();
  }

  link.close();
  return configService.load();
});
```

Explanation:

- KeepAlive is applied conditionally.
- Useful when caching depends on runtime conditions.

---

# Mental Model

Think of `keepAlive` as a **lifecycle override switch**.

Without keepAlive:

```text
No listeners
   │
   ▼
Provider disposed
```

With keepAlive:

```text
No listeners
   │
   ▼
Provider stays in memory
```

Even when unused, the provider is preserved.

---

# Examples

## Simple Cache Example

```dart
final productsProvider = FutureProvider.autoDispose((ref) async {
  ref.keepAlive();

  return api.fetchProducts();
});
```

Explanation:

- Products are fetched once.
- They remain cached even after navigation.

---

## Real-World Example

```dart
final sessionProvider = Provider.autoDispose<Session>((ref) {
  final link = ref.keepAlive();

  final session = sessionManager.currentSession;

  return session;
});
```

Explanation:

- Session state persists even when UI is destroyed.
- Avoids repeated session reconstruction.

---

## Background Data Example

```dart
final feedProvider = StreamProvider.autoDispose((ref) {
  final link = ref.keepAlive();

  return api.streamFeed();
});
```

Explanation:

- Live feed continues running even when UI is not active.
- Useful for real-time systems.

---

# When to Use

Use `keepAlive` when:

- You want to cache expensive async results
- Data should persist across navigation
- Avoiding repeated network calls
- Keeping background streams alive
- Optimizing performance for frequently accessed data

---

# When NOT to Use

Avoid `keepAlive` when:

- Data should be temporary
- Memory usage must be minimal
- State should reset on navigation
- Provider represents screen-specific data
- Auto-disposal is desired behavior

Overusing `keepAlive` can lead to memory leaks or stale data.

---

# Best Practices

- Use only when caching is intentional
- Combine carefully with `.autoDispose`
- Always consider lifecycle explicitly
- Close `KeepAliveLink` when appropriate
- Avoid keeping large unused objects in memory
- Prefer simplicity unless caching is required

---

# Common Mistakes

## Forgetting That Provider Never Disposes

**Wrong**

```dart
final provider = FutureProvider.autoDispose((ref) async {
  ref.keepAlive();
  return fetchData();
});
```

Assuming it will still be disposed automatically.

**Correct**

Understand that `keepAlive` overrides auto-dispose behavior.

---

## Keeping Alive Unnecessarily

**Wrong**

```dart
ref.keepAlive();
return smallValue;
```

Using keepAlive for trivial or cheap operations.

**Correct**

Only use for expensive or reusable computations.

---

## Not Managing KeepAliveLink

**Wrong**

```dart
final link = ref.keepAlive();
// never closed
```

This permanently prevents disposal.

**Correct**

```dart
final link = ref.keepAlive();
link.close();
```

Close it when caching is no longer needed.

---

# Related APIs

- `.autoDispose`
- `ref.keepAlive()`
- `KeepAliveLink`
- `.family`
- `Provider`
- `FutureProvider`
- `StreamProvider`

---

# Summary

`keepAlive` prevents a provider from being disposed when it has no listeners. It is used to cache data, preserve state across navigation, and optimize performance, but must be used carefully to avoid unnecessary memory retention.