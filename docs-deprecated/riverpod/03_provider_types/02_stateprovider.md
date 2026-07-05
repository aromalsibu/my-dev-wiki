# StateProvider

`StateProvider` is a provider that exposes a simple mutable value, making it easy to manage lightweight application state.

---

## What is it?

`StateProvider` is the simplest mutable provider in Riverpod.

Unlike `Provider`, which exposes an immutable value, `StateProvider` stores a value that can be updated over time.

It is designed for **simple state**, such as:

- Counters
- Booleans
- Selected index
- Search text
- Theme mode
- Filter values
- Form selections

Internally, a `StateProvider` stores its value inside a `StateController`.

---

## Why does it exist?

Many applications need small pieces of mutable state.

For example:

- Is dark mode enabled?
- Which tab is selected?
- What page is currently active?
- Is the checkbox checked?

Using a full `Notifier` for these simple values adds unnecessary boilerplate.

`StateProvider` offers:

- Minimal code
- Easy updates
- Automatic UI rebuilding
- Reactive state management

For more complex business logic, `NotifierProvider` is the recommended choice.

---

## Syntax

### Creating a StateProvider

```dart
final counterProvider = StateProvider<int>((ref) {
  return 0;
});
```

Explanation:

- `StateProvider<T>` manages mutable state of type `T`.
- The initial value is `0`.
- Riverpod stores the value in a `StateController<int>`.

---

### Reading the State

```dart
class CounterText extends ConsumerWidget {
  const CounterText({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Text('$count');
  }
}
```

Explanation:

- `ref.watch()` returns the current state value.
- The widget rebuilds whenever the state changes.

---

### Updating the State

```dart
ref.read(counterProvider.notifier).state++;
```

Explanation:

- `.notifier` returns the `StateController`.
- Updating `.state` changes the value.
- Riverpod notifies all listeners automatically.

---

### Setting a New Value

```dart
ref.read(counterProvider.notifier).state = 10;
```

Explanation:

- Replaces the current state.
- All watching widgets rebuild.

---

## Mental Model

Think of `StateProvider` as a small reactive box.

```text
          StateProvider
                │
                ▼
        StateController
                │
         state = Current Value
                │
                ▼
        Widgets Watch State
```

Changing the value updates every widget watching the provider.

---

## Examples

### Counter

```dart
final counterProvider = StateProvider<int>((ref) => 0);

FloatingActionButton(
  onPressed: () {
    ref.read(counterProvider.notifier).state++;
  },
)
```

Explanation:

- Increments the counter.
- The UI updates automatically.

---

### Theme Toggle

```dart
final darkModeProvider = StateProvider<bool>((ref) => false);
```

Explanation:

- Stores whether dark mode is enabled.
- Useful for simple application settings.

---

### Selected Tab

```dart
final selectedTabProvider = StateProvider<int>((ref) => 0);
```

Explanation:

- Tracks the selected tab index.
- Common in navigation bars and tab views.

---

### Search Query

```dart
final searchQueryProvider = StateProvider<String>((ref) => '');
```

Explanation:

- Stores the current search text.
- Can be updated as the user types.

---

## StateController

Every `StateProvider` exposes a `StateController`.

```dart
final controller = ref.read(counterProvider.notifier);
```

The controller provides access to the mutable state.

```dart
controller.state++;
```

or

```dart
controller.state = 100;
```

Most applications interact with the controller only to update the state.

---

## When to Use

Use `StateProvider` for:

- Counters
- Boolean flags
- Selected index
- Theme selection
- Search text
- Radio button selection
- Dropdown value
- Checkbox state
- Simple filters

These values typically require little or no business logic.

---

## When NOT to Use

Avoid `StateProvider` when:

- State contains business logic.
- Multiple values must be managed together.
- Validation is required.
- Side effects occur during updates.
- Asynchronous operations are involved.

Instead, use:

| Scenario | Recommended Provider |
|----------|----------------------|
| Complex state | `NotifierProvider` |
| Async state | `AsyncNotifierProvider` |
| Streams | `StreamProvider` |
| Read-only values | `Provider` |

---

## Best Practices

- Keep state simple.
- Store only one value per provider.
- Prefer immutable values where possible.
- Use `NotifierProvider` as state complexity grows.
- Avoid placing business logic directly in widgets.

---

## Common Mistakes

### 1. Using StateProvider for Complex State

❌ Wrong

```dart
final userProvider = StateProvider<User>((ref) {
  return User();
});
```

Why it's wrong:

- User management often involves validation, updates, and business logic.
- `StateProvider` becomes difficult to maintain.

✔ Correct

```dart
final userProvider =
    NotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );
```

---

### 2. Modifying Objects Instead of Replacing Them

❌ Wrong

```dart
ref.read(userProvider.notifier).state.name = 'John';
```

Why it's wrong:

- Riverpod cannot detect internal mutations reliably.
- Widgets may not rebuild as expected.

✔ Correct

```dart
ref.read(userProvider.notifier).state =
    user.copyWith(name: 'John');
```

Replace the entire object with a new immutable instance.

---

### 3. Reading Instead of Watching

❌ Wrong

```dart
final count = ref.read(counterProvider);
```

Why it's wrong:

- The widget will not rebuild when the counter changes.

✔ Correct

```dart
final count = ref.watch(counterProvider);
```

Use `watch()` for reactive UI updates.

---

## StateProvider vs Provider

| Feature | Provider | StateProvider |
|---------|----------|---------------|
| Mutable state | ❌ | ✅ |
| Read-only value | ✅ | ✅ |
| Dependency injection | ✅ | ❌ |
| Computed values | ✅ | Limited |
| Business logic | ❌ | ❌ |
| Automatic rebuilds | Yes (on recompute) | Yes (on state change) |

---

## Related APIs

- Provider
- StateController
- NotifierProvider
- AsyncNotifierProvider
- ref.watch()
- ref.read()
- state

---

## Summary

`StateProvider` is Riverpod's simplest mutable provider. It is ideal for lightweight state such as counters, booleans, selected indexes, and search text. It exposes a `StateController` that allows updating the current value through the `state` property. For complex state or business logic, `NotifierProvider` is the recommended alternative.