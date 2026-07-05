# Provider vs StateProvider

Choose between `Provider` and `StateProvider` based on whether your value is immutable or needs to change over time.

---

`Provider` exposes read-only values, while `StateProvider` exposes mutable state that can be updated by the application.

---

# What is it?

Both `Provider` and `StateProvider` expose values through Riverpod, but they serve different purposes.

- **Provider** exposes a computed or immutable value.
- **StateProvider** exposes a mutable value whose state can change.

The choice depends on whether the value should ever be modified after it is created.

---

# Why does it exist?

Applications contain both:

- Static or derived values
- Mutable application state

Using a single provider type for both scenarios would either introduce unnecessary complexity or make simple state difficult to manage.

Riverpod provides two separate provider types so you can choose the simplest solution for your use case.

---

# Syntax

## Provider

```dart
final appNameProvider = Provider<String>((ref) {
  return 'RiverShop';
});
```

Explanation:

- Computes or returns a value.
- Consumers cannot modify the value.

---

## StateProvider

```dart
final counterProvider =
    StateProvider<int>((ref) => 0);
```

Explanation:

- Stores mutable state.
- The value can be updated through its notifier.

---

## Updating a StateProvider

```dart
ref.read(counterProvider.notifier).state++;
```

Explanation:

- The provider's state changes.
- Widgets watching the provider rebuild automatically.

---

# Mental Model

```text
Provider
──────────────
Read-only
        │
        ▼
 Immutable Value

StateProvider
──────────────
Read + Write
        │
        ▼
 Mutable State
```

If the value never changes, use `Provider`.

If the value changes over time, use `StateProvider`.

---

# Examples

## Simple Example

### Provider

```dart
final apiUrlProvider = Provider<String>((ref) {
  return 'https://api.example.com';
});
```

Explanation:

- The API URL is constant.
- No updates are required.

---

### StateProvider

```dart
final selectedTabProvider =
    StateProvider<int>((ref) => 0);
```

Explanation:

- The selected tab changes as the user navigates.

---

## Real-World Example

```dart
final themeNameProvider =
    Provider<String>((ref) {
  final theme = ref.watch(themeProvider);

  return theme.name;
});
```

Explanation:

- `themeNameProvider` derives data from another provider.
- It does not own mutable state.

---

```dart
final searchQueryProvider =
    StateProvider<String>((ref) => '');
```

Explanation:

- The search query changes whenever the user types.
- The provider owns mutable UI state.

---

# Feature Comparison

| Feature | Provider | StateProvider |
|----------|----------|---------------|
| Mutable state | ❌ No | ✅ Yes |
| Read-only values | ✅ Yes | ⚠️ Possible but unnecessary |
| Derived values | ✅ Excellent | ❌ Not intended |
| Business logic | ❌ No | ❌ Very limited |
| Best for simple UI state | ❌ | ✅ |
| Supports direct state updates | ❌ | ✅ |
| Recommended for complex state | ❌ | ❌ Use `Notifier` |

---

# When to Use

Use **Provider** when:

- Exposing constants
- Creating services
- Injecting dependencies
- Computing derived values
- Reading repositories
- Sharing immutable objects

Use **StateProvider** when:

- Managing counters
- Toggle switches
- Selected tabs
- Search text
- Filter values
- Temporary UI state

---

# When NOT to Use

Avoid **Provider** when:

- The value needs to change.

Avoid **StateProvider** when:

- Business logic is required.
- Validation is needed.
- Multiple operations modify the state.

In those cases, prefer `Notifier`.

---

# Best Practices

- Default to `Provider` for immutable values.
- Use `StateProvider` only for simple mutable state.
- Keep business logic out of `StateProvider`.
- Use `Notifier` as state complexity grows.
- Keep providers focused on a single responsibility.

---

# Common Mistakes

## Using StateProvider for constants

Wrong:

```dart
final apiUrlProvider =
    StateProvider<String>(
      (ref) => 'https://api.example.com',
    );
```

Explanation:

- The value never changes.
- Mutable state is unnecessary.

Correct:

```dart
final apiUrlProvider =
    Provider<String>(
      (ref) => 'https://api.example.com',
    );
```

Explanation:

- `Provider` better communicates that the value is immutable.

---

## Using Provider for mutable state

Wrong:

```dart
final counterProvider =
    Provider<int>((ref) => 0);
```

Explanation:

- The counter cannot be updated.

Correct:

```dart
final counterProvider =
    StateProvider<int>((ref) => 0);
```

Explanation:

- `StateProvider` supports mutable state.

---

## Using StateProvider for business logic

Wrong:

```dart
ref.read(counterProvider.notifier).state++;
```

Explanation:

- As business rules grow, the UI becomes responsible for state changes.

Correct:

```dart
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Explanation:

- `Notifier` centralizes state changes and business logic.

---

# Related APIs

- Provider
- StateProvider
- Notifier
- NotifierProvider
- `ref.watch()`
- `ref.read()`

---

# Summary

`Provider` is the right choice for immutable values, dependency injection, and derived state. `StateProvider` is designed for simple mutable values such as UI state. When state management becomes more complex or requires business logic, migrate to `Notifier` for a more scalable solution.