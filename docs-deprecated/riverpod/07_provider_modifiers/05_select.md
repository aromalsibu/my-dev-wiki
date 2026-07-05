# .select

Creates a derived subscription that only rebuilds when a specific part of a provider changes.

---

# What is it?

`.select` is a provider modifier that allows you to **listen to only a small part of a provider’s state** instead of the entire object.

Instead of rebuilding whenever any field in the state changes, `.select` ensures that Riverpod only triggers a rebuild when the selected value changes.

This is a performance optimization tool.

---

# Why does it exist?

Without `.select`, every change in a provider causes all listeners to rebuild — even if the change is irrelevant.

For example, if a user object changes its `name`, a widget that only cares about `id` would still rebuild unnecessarily.

`.select` solves this by allowing **fine-grained listening**.

It helps:

- Reduce unnecessary rebuilds
- Improve performance in large UIs
- Isolate state dependencies
- Make UI more efficient and predictable

---

# Syntax

## Basic Usage

```dart
final name = ref.watch(
  userProvider.select((user) => user.name),
);
```

Explanation:

- `select()` extracts only the `name` field from the user object.
- The widget rebuilds only when `name` changes.

---

## With Provider

```dart
final userNameProvider = Provider<String>((ref) {
  return ref.watch(
    userProvider.select((user) => user.name),
  );
});
```

Explanation:

- A derived provider listens only to `name`.
- Changes in other fields do not trigger rebuilds.

---

## With NotifierProvider

```dart
final countProvider = NotifierProvider<Counter, CounterState>(
  Counter.new,
);

final countValue = ref.watch(
  countProvider.select((state) => state.value),
);
```

Explanation:

- Only `state.value` is observed.
- Other parts of `CounterState` are ignored for rebuilds.

---

# Mental Model

Think of `.select` as a **lens or filter** over a provider.

Without `.select`

```text
Provider State:
{
  id: 1,
  name: "Alice",
  age: 25
}

Any change → rebuild
```

With `.select`

```text
Select only "name"
        │
        ▼
Rebuild only when "name" changes
```

It narrows the dependency scope.

---

# Examples

## Simple Example

```dart
final userName = ref.watch(
  userProvider.select((u) => u.name),
);
```

Explanation:

- Only the `name` field is observed.
- UI will not rebuild when other fields change.

---

## Real-World Example

```dart
final isLoggedIn = ref.watch(
  authProvider.select((state) => state.isAuthenticated),
);
```

Explanation:

- UI depends only on authentication status.
- Token refresh or user profile updates won't trigger rebuilds.

---

## List Optimization Example

```dart
final itemCount = ref.watch(
  cartProvider.select((cart) => cart.items.length),
);
```

Explanation:

- Only item count changes matter.
- Adding metadata to cart won't rebuild this widget.

---

# When to Use

Use `.select` when:

- You only need a small part of a large state object
- You want to reduce rebuilds in complex UIs
- Working with large immutable state models
- Optimizing performance-critical widgets
- Listening to derived values (like flags or counts)

---

# When NOT to Use

Avoid `.select` when:

- You need the entire state object
- State is already small and simple
- It makes code harder to read unnecessarily
- Multiple fields are required in the same widget

Overusing `.select` can reduce readability without meaningful performance gains.

---

# Best Practices

- Use `.select` for large state objects only
- Select primitive or simple derived values
- Keep selectors pure and side-effect free
- Prefer clarity over micro-optimization
- Combine with immutable state models for best results

---

# Common Mistakes

## Selecting Too Much State

**Wrong**

```dart
ref.watch(
  userProvider.select((u) => u),
);
```

This defeats the purpose of `.select`.

**Correct**

```dart
ref.watch(
  userProvider.select((u) => u.name),
);
```

Only select what you need.

---

## Using Select for Small Objects

**Wrong**

```dart
final count = ref.watch(
  counterProvider.select((c) => c.value),
);
```

If the state is already simple, `.select` adds unnecessary complexity.

**Correct**

```dart
final count = ref.watch(counterProvider);
```

---

## Selecting Non-Comparable Objects

**Wrong**

```dart
ref.watch(
  provider.select((state) => state.complexObject),
);
```

If the object does not implement proper equality, rebuilds may still occur.

**Correct**

Select primitive or well-defined immutable values.

---

# Related APIs

- `ref.watch()`
- `Provider`
- `StateNotifier`
- `Notifier`
- `AsyncValue`
- `.autoDispose`
- `.family`

---

# Summary

`.select` is a performance optimization tool that allows Riverpod to listen to only a specific part of a provider's state. It reduces unnecessary rebuilds by narrowing the dependency scope, making UI updates more efficient and predictable when working with large or complex state objects.