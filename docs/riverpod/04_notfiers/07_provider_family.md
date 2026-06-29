# Provider Families

`Family` is a Riverpod modifier that allows a provider to accept external parameters, enabling dynamic provider instances based on input values.

---

## What is it?

A `Family` lets you create **parameterized providers**.

Instead of having a single global provider instance, you can generate multiple instances based on input values like:

- IDs
- Query parameters
- Filters
- Configuration values

Each unique parameter creates a **separate provider instance with its own state**.

---

## Why does it exist?

In real applications, data is rarely global.

You often need:

- User-specific data (`userId`)
- Product details (`productId`)
- Paginated lists (`page`)
- Filtered queries (`category`)

Without families, developers would need:

- Multiple duplicated providers
- Manual state filtering
- Complex mapping logic

`Family` solves this by:

- Creating reusable parameterized providers
- Automatically caching per parameter
- Keeping logic clean and reusable
- Avoiding duplication

It enables **scalable, dynamic state management**.

---

## Syntax

### Notifier Family

```dart
final counterProvider =
    NotifierProvider.family<Counter, int, int>(
  Counter.new,
);
```

Explanation:

- First `int` → state type
- Second `int` → parameter type
- Each parameter creates a separate instance

---

### Using the Family

```dart
final counter = ref.watch(counterProvider(5));
```

Explanation:

- `5` is the parameter
- Each value creates a unique provider instance

---

### AsyncNotifier Family

```dart
final userProvider =
    AsyncNotifierProvider.family<UserNotifier, User, String>(
  UserNotifier.new,
);
```

Explanation:

- Parameter is `String` (userId)
- Each user has separate async state

---

### StreamNotifier Family

```dart
final chatProvider =
    StreamNotifierProvider.family<ChatNotifier, Message, String>(
  ChatNotifier.new,
);
```

Explanation:

- Each chat room ID gets its own stream instance

---

## Mental Model

Think of `Family` as a **provider factory**.

```text
          Family Provider
                 │
     ┌───────────┼───────────┐
     ▼           ▼           ▼
 user:1      user:2      user:3
     │           │           │
  instance    instance    instance
     │           │           │
   state       state       state
```

Each parameter produces an independent provider instance.

---

## Examples

### Fetch User by ID

```dart
final userProvider =
    AsyncNotifierProvider.family<UserNotifier, User, String>(
  UserNotifier.new,
);
```

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build(String userId) async {
    return repository.fetchUser(userId);
  }
}
```

Explanation:

- Each `userId` creates a separate fetch.
- State is isolated per user.

---

### Product Details

```dart
final productProvider =
    FutureProvider.family<Product, String>((ref, id) {
  return repository.getProduct(id);
});
```

Explanation:

- Each product ID loads its own data.
- No shared state conflicts.

---

### Pagination

```dart
final pageProvider =
    NotifierProvider.family<PageNotifier, List<Item>, int>(
  PageNotifier.new,
);
```

```dart
class PageNotifier extends Notifier<List<Item>> {
  @override
  List<Item> build(int page) {
    return repository.fetchPage(page);
  }
}
```

Explanation:

- Each page number has separate state.
- Supports infinite scroll patterns.

---

## When to Use

Use `Family` when:

- Data depends on an input parameter
- You need multiple independent instances
- You fetch data by ID
- You handle pagination or filters
- You want reusable provider logic

---

## When NOT to Use

Avoid `Family` when:

- Data is global or shared
- Parameter changes frequently in a way that should reset internal state
- You only need a single instance
- State is not parameter-dependent

---

## Best Practices

- Keep parameters simple and immutable
- Use IDs or primitives when possible
- Avoid passing large objects as parameters
- Ensure parameter equality is stable
- Combine with `autoDispose` for temporary instances

---

## Common Mistakes

### 1. Using Complex Objects as Parameters

❌ Wrong

```dart
ref.watch(userProvider(UserModel(...)));
```

Why it's wrong:

- Object equality may break caching.
- Causes unnecessary rebuilds.

✔ Correct

```dart
ref.watch(userProvider(user.id));
```

---

### 2. Forgetting Parameter in build()

❌ Wrong

```dart
Future<User> build() async {
  return repo.fetchUser(); // missing id
}
```

Why it's wrong:

- Provider is not parameter-aware.

✔ Correct

```dart
Future<User> build(String id) async {
  return repo.fetchUser(id);
}
```

---

### 3. Creating Too Many Families

❌ Wrong

```dart
family for every small state
```

Why it's wrong:

- Overcomplicates simple state management.

✔ Correct

Use families only for truly dynamic data.

---

## Related APIs

- NotifierProvider.family
- AsyncNotifierProvider.family
- StreamNotifierProvider.family
- FutureProvider.family
- ref.watch()
- autoDispose

---

## Summary

`Family` is a Riverpod feature that enables parameterized providers, allowing each input value to create an independent provider instance. It is essential for handling dynamic data such as user IDs, product details, pagination, and filtered queries while keeping state management clean and reusable.