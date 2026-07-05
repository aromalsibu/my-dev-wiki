# .overrideWithValue

Replaces a provider with a fixed value instead of executing its original logic.

---

# What is it?

`.overrideWithValue` is a provider modifier that allows you to override a provider with a **constant prebuilt value**.

Unlike `.overrideWith`, which replaces the provider logic, `.overrideWithValue` directly injects an already created value into the provider system.

This makes it the simplest form of overriding.

---

# Why does it exist?

In many cases (especially testing and UI previews), you don’t need custom logic — you just want a fixed value.

Without `.overrideWithValue`, you would still need to wrap simple values inside a function using `.overrideWith`.

`.overrideWithValue` removes that unnecessary boilerplate.

It is designed for:

- Tests
- Mock UI states
- Static dependency injection
- Predefined data setups

---

# Syntax

## Basic Usage

```dart
final counterProvider = Provider<int>((ref) => 10);
```

Explanation:

- A simple provider returning an integer.

---

## Overriding with Value

```dart
ProviderScope(
  overrides: [
    counterProvider.overrideWithValue(99),
  ],
  child: const MyApp(),
);
```

Explanation:

- The provider is replaced with a fixed value.
- No function is executed.
- The value is used directly wherever the provider is read.

---

## With Complex Objects

```dart
final userProvider = Provider<User>((ref) {
  return User(id: 1, name: 'Default');
});
```

Override:

```dart
ProviderScope(
  overrides: [
    userProvider.overrideWithValue(
      User(id: 99, name: 'Mock User'),
    ),
  ],
  child: const MyApp(),
);
```

Explanation:

- A fully constructed object is injected.
- No recomputation or factory logic is used.

---

# Mental Model

Think of `.overrideWithValue` as **direct injection into the provider graph**.

Without override:

```text
Provider → executes logic → produces value
```

With overrideWithValue:

```text
Provider → skipped → injected value used directly
```

It completely bypasses provider creation logic.

---

# Examples

## Simple Example

```dart
final themeProvider = Provider<String>((ref) => 'Light');
```

Override:

```dart
ProviderScope(
  overrides: [
    themeProvider.overrideWithValue('Dark'),
  ],
  child: const MyApp(),
);
```

Explanation:

- The app behaves as if the theme is always "Dark".
- No provider logic is executed.

---

## Real-World Example

```dart
final authProvider = Provider<User?>((ref) {
  return null;
});
```

Test override:

```dart
ProviderScope(
  overrides: [
    authProvider.overrideWithValue(
      User(id: 1, name: 'Test User'),
    ),
  ],
  child: const MyApp(),
);
```

Explanation:

- Authentication state is mocked.
- Useful for bypassing login flows in tests.

---

## UI Preview Example

```dart
final productProvider = Provider<Product>((ref) {
  throw UnimplementedError();
});
```

Override:

```dart
ProviderScope(
  overrides: [
    productProvider.overrideWithValue(
      Product(id: 'P1', name: 'Preview Product'),
    ),
  ],
  child: const ProductCard(),
);
```

Explanation:

- Enables UI development without backend or API.

---

# When to Use

Use `.overrideWithValue` when:

- You already have a fully constructed object
- Writing widget tests
- Creating UI previews or storybook setups
- Injecting static mocks
- Avoiding async or computation logic

---

# When NOT to Use

Avoid `.overrideWithValue` when:

- You need to compute or fetch data dynamically
- The value depends on runtime logic
- You need different behavior per test scenario
- The provider requires setup or cleanup logic

In those cases, use `.overrideWith` instead.

---

# Best Practices

- Use for **simple and static overrides only**
- Prefer `.overrideWith` for anything involving logic
- Keep test overrides close to test setup code
- Avoid large object graphs unless necessary
- Use factory methods for complex mock creation

---

# Common Mistakes

## Using overrideWithValue for Dynamic Data

**Wrong**

```dart
counterProvider.overrideWithValue(Random().nextInt(100));
```

This makes the value unpredictable at test time.

**Correct**

```dart
counterProvider.overrideWithValue(42);
```

Keep values deterministic.

---

## Trying to Use Async Logic

**Wrong**

```dart
userProvider.overrideWithValue(fetchUser());
```

This returns a Future, not a value.

**Correct**

```dart
userProvider.overrideWithValue(User(id: 1, name: 'Mock'));
```

`.overrideWithValue` does not support async computation.

---

## Overusing It for Complex Providers

**Wrong**

```dart
ProviderScope(
  overrides: [
    complexService.overrideWithValue(ComplexServiceImpl(...)),
  ],
);
```

This bypasses lifecycle management unnecessarily.

**Correct**

Use `.overrideWith` when setup logic is required.

---

# Related APIs

- `.overrideWith`
- `ProviderScope`
- `.family`
- `.autoDispose`
- `ProviderContainer`

---

# Summary

`.overrideWithValue` replaces a provider with a fixed prebuilt value. It is the simplest form of overriding and is best suited for tests, UI previews, and static dependency injection. Unlike `.overrideWith`, it does not execute any logic — it directly injects the value into Riverpod’s dependency graph.