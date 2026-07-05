# .family

Creates multiple independent instances of the same provider based on an input parameter.

---

# What is it?

`.family` is a provider modifier that allows a provider to accept external parameters.

Instead of creating a separate provider for every possible value, `.family` lets you reuse the same provider definition with different inputs. Riverpod creates and manages a unique provider instance for each parameter.

This is useful whenever the data your provider returns depends on an external value.

---

# Why does it exist?

Without `.family`, you would need to create a new provider for every input.

For example, if you wanted to fetch users by their ID, you might end up writing multiple providers:

```dart
final user1Provider = FutureProvider((ref) => api.fetchUser(1));

final user2Provider = FutureProvider((ref) => api.fetchUser(2));

final user3Provider = FutureProvider((ref) => api.fetchUser(3));
```

This approach is repetitive and doesn't scale.

`.family` solves this by allowing the provider to receive a parameter.

---

# Syntax

## Basic Usage

```dart
final greetingProvider = Provider.family<String, String>((ref, name) {
  return 'Hello, $name!';
});
```

Explanation:

- `.family` allows the provider to accept a parameter.
- The second generic type (`String`) represents the parameter type.
- The callback receives the parameter after `ref`.

---

## Reading a Family Provider

```dart
final greeting = ref.watch(greetingProvider('John'));
```

Explanation:

- Calling the provider with a parameter creates (or reuses) the corresponding provider instance.
- Each parameter has its own independent state.

---

## With FutureProvider

```dart
final userProvider =
    FutureProvider.family<User, int>((ref, userId) async {
  return api.fetchUser(userId);
});
```

Explanation:

- `userId` is passed into the provider.
- Riverpod creates a separate provider instance for each user ID.

---

## With NotifierProvider

```dart
final counterProvider =
    NotifierProvider.family<CounterNotifier, int, String>(
      CounterNotifier.new,
    );
```

Explanation:

- The first generic type is the notifier.
- The second is the state type.
- The third is the parameter type.

---

# Mental Model

Think of `.family` as a factory that creates provider instances.

Without `.family`

```text
Provider
   │
   ▼
Single Instance
```

With `.family`

```text
userProvider(1)
        │
        ▼
 Provider Instance #1

userProvider(2)
        │
        ▼
 Provider Instance #2

userProvider(3)
        │
        ▼
 Provider Instance #3
```

Each parameter gets its own independent provider and state.

---

# Examples

## Simple Example

```dart
final squareProvider =
    Provider.family<int, int>((ref, number) {
  return number * number;
});
```

Reading it:

```dart
final result = ref.watch(squareProvider(5));
```

Explanation:

- The provider calculates the square of the supplied number.
- Calling it with a different number creates another provider instance.

---

## Real-World Example

```dart
final productProvider =
    FutureProvider.family<Product, String>((ref, productId) async {
  return repository.fetchProduct(productId);
});
```

Usage:

```dart
final product = ref.watch(productProvider('P001'));
```

Explanation:

- Each product ID has its own cached provider.
- Different product pages do not interfere with one another.

---

# When to Use

Use `.family` when:

- Fetching data by ID
- Loading user profiles
- Loading product details
- Searching with a query
- Filtering lists
- Working with route parameters
- Any provider that depends on external input

---

# When NOT to Use

Avoid `.family` when:

- The provider always returns the same value.
- No external parameter is required.
- The provider represents global application state.

A normal provider is simpler in these cases.

---

# Best Practices

- Keep family parameters immutable.
- Use simple parameter types whenever possible.
- Combine `.family` with `.autoDispose` for temporary data.
- Reuse one family provider instead of creating many similar providers.
- Pass only the data required to identify the provider instance.

---

# Common Mistakes

## Creating Multiple Providers Instead of Using Family

**Wrong**

```dart
final user1Provider = FutureProvider((ref) => api.fetchUser(1));

final user2Provider = FutureProvider((ref) => api.fetchUser(2));
```

This becomes difficult to maintain.

**Correct**

```dart
final userProvider =
    FutureProvider.family<User, int>((ref, id) async {
  return api.fetchUser(id);
});
```

One provider works for every user.

---

## Passing Mutable Objects

**Wrong**

```dart
final provider =
    Provider.family<User, List<String>>((ref, data) {
  ...
});
```

A mutable parameter can lead to unexpected behavior.

**Correct**

```dart
final provider =
    Provider.family<User, String>((ref, id) {
  ...
});
```

Prefer immutable values such as IDs, enums, or value objects.

---

## Forgetting That Each Parameter Has Its Own State

**Wrong**

```dart
ref.watch(userProvider(1));

ref.watch(userProvider(2));
```

Assuming both calls share the same state is incorrect.

**Correct**

Treat each parameter as a completely separate provider instance.

---

# Related APIs

- `.autoDispose`
- `.select`
- `Provider`
- `FutureProvider`
- `StreamProvider`
- `NotifierProvider`
- `AsyncNotifierProvider`

---

# Summary

`.family` allows a provider to receive parameters and creates an independent provider instance for each unique value. It eliminates the need for multiple nearly identical providers and is the recommended approach whenever provider logic depends on external input.