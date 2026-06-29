# StateProvider → Notifier

Migrate from `StateProvider` to `Notifier` when state management grows beyond simple value updates.

---

`StateProvider` is intended for simple, local state. As soon as your state requires validation, business logic, or multiple operations, `Notifier` becomes the better choice.

---

# What is it?

Both `StateProvider` and `Notifier` manage synchronous state, but they target different levels of complexity.

- **StateProvider** exposes a mutable value with minimal boilerplate.
- **Notifier** encapsulates state and the logic that modifies it.

Think of `StateProvider` as a simple state holder and `Notifier` as a complete state manager.

---

# Why does it exist?

`StateProvider` is great for simple values such as:

- Selected tab
- Checkbox state
- Search query
- Theme mode
- Filter selection

However, as requirements grow, state often needs:

- Validation
- Multiple update methods
- Derived state
- Business rules
- Coordination with repositories or services

Moving to `Notifier` keeps this logic out of the UI and makes it easier to maintain and test.

---

# Syntax

## StateProvider

```dart
final counterProvider = StateProvider<int>((ref) => 0);
```

Explanation:

- Stores a single mutable value.
- The UI updates the state directly.

---

## Updating a StateProvider

```dart
ref.read(counterProvider.notifier).state++;
```

Explanation:

- The widget modifies the provider's state.
- No dedicated business logic layer exists.

---

## Notifier

```dart
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}

final counterProvider =
    NotifierProvider<CounterNotifier, int>(
      CounterNotifier.new,
    );
```

Explanation:

- `build()` initializes the state.
- State changes are performed through methods.
- Business logic is centralized in the notifier.

---

# Mental Model

StateProvider:

```text
UI
 │
 ▼
Update state directly
```

Notifier:

```text
UI
 │
 ▼
Notifier
 │
 ▼
Update state
```

With `Notifier`, the UI asks the notifier to perform an action instead of modifying the state itself.

---

# Examples

## Simple Example

### Before

```dart
final countProvider = StateProvider<int>((ref) => 0);

ref.read(countProvider.notifier).state++;
```

Explanation:

- The widget changes the state directly.

---

### After

```dart
class CountNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}

ref.read(countProvider.notifier).increment();
```

Explanation:

- The widget calls a method instead of manipulating state.
- The notifier owns the update logic.

---

## Real-World Example

### Before

```dart
final quantityProvider =
    StateProvider<int>((ref) => 1);
```

```dart
ref.read(quantityProvider.notifier).state++;
```

Explanation:

- There is no validation.
- The UI can set invalid values.

---

### After

```dart
class QuantityNotifier extends Notifier<int> {
  @override
  int build() => 1;

  void increment() {
    state++;
  }

  void decrement() {
    if (state > 1) {
      state--;
    }
  }
}
```

Explanation:

- Validation is handled inside the notifier.
- Invalid state transitions are prevented.

---

# When to Use

Migrate to `Notifier` when:

- State requires business logic.
- Validation is needed.
- Multiple actions modify the state.
- The state is shared across multiple widgets.
- The feature is expected to grow.
- You want better testability.

---

# When NOT to Use

Continue using `StateProvider` for:

- Simple counters
- Toggle switches
- Selected indices
- Search text
- Temporary UI state
- Small prototypes

Don't replace every `StateProvider` automatically. If the state remains simple, `StateProvider` is still an excellent choice.

---

# Best Practices

- Use `StateProvider` only for simple mutable values.
- Move business logic into `Notifier`.
- Keep widgets free of state mutation logic.
- Expose meaningful methods instead of direct state updates.
- Prefer immutable state models as complexity increases.

---

# Common Mistakes

## Putting business logic in the UI

Wrong:

```dart
final notifier = ref.read(counterProvider.notifier);

if (notifier.state < 10) {
  notifier.state++;
}
```

Explanation:

- Validation is scattered across widgets.
- Multiple widgets may implement the same logic differently.

Correct:

```dart
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    if (state < 10) {
      state++;
    }
  }
}
```

Explanation:

- The notifier becomes the single source of truth for state changes.

---

## Continuing to modify `state` directly

Wrong:

```dart
ref.read(counterProvider.notifier).state++;
```

Explanation:

- The UI bypasses business logic.
- Future validation becomes harder to introduce.

Correct:

```dart
ref
    .read(counterProvider.notifier)
    .increment();
```

Explanation:

- Widgets request an action.
- The notifier decides how the state changes.

---

## Migrating too early

Wrong:

```dart
class ThemeNotifier extends Notifier<bool> {
  @override
  bool build() => false;
}
```

Explanation:

- If the state only stores a simple boolean with no additional logic, `Notifier` may add unnecessary complexity.

Correct:

```dart
final darkModeProvider =
    StateProvider<bool>((ref) => false);
```

Explanation:

- Keep simple state simple.
- Introduce a `Notifier` only when the feature outgrows `StateProvider`.

---

# Related APIs

- StateProvider
- Notifier
- NotifierProvider
- AsyncNotifier
- `ref.watch()`
- `ref.read()`

---

# Summary

`StateProvider` is ideal for simple, mutable values with minimal logic. As state becomes more complex, migrating to `Notifier` centralizes business logic, improves testability, and keeps widgets focused on presentation. Use `StateProvider` for simplicity and `Notifier` when your application's requirements demand a more structured approach.