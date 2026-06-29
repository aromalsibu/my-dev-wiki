# ref.watch()

`ref.watch()` subscribes to a provider and automatically rebuilds the caller whenever the provider's state changes.

---

## What is it?

`ref.watch()` is the primary way to **consume a provider reactively** in Riverpod.

When you call `ref.watch()`, Riverpod:

- Reads the provider's current value.
- Registers the caller as a dependency.
- Automatically rebuilds the caller when the provider's state changes.

The caller can be:

- A `ConsumerWidget`
- A `ConsumerState`
- Another provider
- A `Notifier`
- An `AsyncNotifier`
- A `StreamNotifier`

Unlike `ref.read()`, `ref.watch()` creates a **reactive relationship** between the caller and the provider.

---

## Why does it exist?

Modern applications are reactive.

When application state changes, the UI should update automatically without manually refreshing widgets.

Without `ref.watch()`, developers would need to:

- manually subscribe to state
- manually notify widgets
- manually manage listeners
- manually rebuild the UI

`ref.watch()` removes all of this complexity.

It allows Riverpod to build a **dependency graph**, where providers and widgets automatically react to changes in the providers they depend on.

---

## Syntax

### Watching a Provider

```dart
final counter = ref.watch(counterProvider);
```

Explanation:

- Reads the current value of `counterProvider`.
- Registers the caller as a listener.
- Rebuilds the caller whenever `counterProvider` changes.

---

### Watching Inside a ConsumerWidget

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

- The widget automatically rebuilds whenever `counterProvider` updates.
- No manual listeners are required.

---

### Watching Another Provider

```dart
final greetingProvider = Provider<String>((ref) {
  final user = ref.watch(userProvider);

  return 'Hello ${user.name}';
});
```

Explanation:

- `greetingProvider` depends on `userProvider`.
- If `userProvider` changes, `greetingProvider` automatically recomputes.

---

## Return Value

`ref.watch()` returns the current value exposed by the provider.

Examples:

| Provider Type | Return Value |
|---------------|--------------|
| `Provider<String>` | `String` |
| `Provider<int>` | `int` |
| `NotifierProvider<Counter, int>` | `int` |
| `FutureProvider<User>` | `AsyncValue<User>` |
| `AsyncNotifierProvider<UserNotifier, User>` | `AsyncValue<User>` |
| `StreamProvider<Message>` | `AsyncValue<Message>` |
| `StreamNotifierProvider<ChatNotifier, Message>` | `AsyncValue<Message>` |

The return type always matches what the provider exposes.

---

## Execution Flow

```text
Widget/Provider starts
          │
          ▼
 ref.watch(counterProvider)
          │
          ▼
Riverpod reads current state
          │
          ▼
Registers dependency
          │
          ▼
Provider state changes
          │
          ▼
Riverpod marks dependent as dirty
          │
          ▼
Widget/Provider rebuilds
```

This dependency tracking happens automatically.

---

## Mental Model

Think of `ref.watch()` as **subscribing to live updates**.

```text
 Provider
    │
    ▼
 Current State
    │
    ▼
 ref.watch()
    │
    ▼
 Dependency Registered
    │
    ▼
 Future State Changes
    │
    ▼
 Automatic Rebuild
```

If you use `watch()`, you're telling Riverpod:

> "Keep me updated whenever this provider changes."

---

## Examples

### Simple Example

```dart
final counter = ref.watch(counterProvider);

Text('$counter');
```

Explanation:

- Displays the current counter value.
- Updates automatically when the counter changes.

---

### Watching an AsyncNotifier

```dart
final user = ref.watch(userProvider);

return user.when(
  data: (user) => Text(user.name),
  loading: () => const CircularProgressIndicator(),
  error: (error, stack) => Text(error.toString()),
);
```

Explanation:

- Watches an `AsyncNotifierProvider`.
- UI reacts to loading, success, and error states automatically.

---

### Provider Depending on Another Provider

```dart
final taxProvider = Provider<double>((ref) {
  final price = ref.watch(priceProvider);

  return price * 0.18;
});
```

Explanation:

- `taxProvider` automatically recalculates when `priceProvider` changes.
- No manual synchronization is required.

---

### Real-World Example

```dart
final filteredProductsProvider = Provider<List<Product>>((ref) {
  final products = ref.watch(productsProvider);
  final category = ref.watch(selectedCategoryProvider);

  return products
      .where((product) => product.category == category)
      .toList();
});
```

Explanation:

- Depends on two providers.
- Changes to either provider trigger recomputation.
- Keeps filtering logic out of the UI.

---

## When to Use

Use `ref.watch()` when:

- Your UI should update automatically.
- A provider depends on another provider.
- You want reactive state management.
- You are displaying application state.
- Automatic rebuilds are expected.

---

## When NOT to Use

Avoid `ref.watch()` when:

- You only need the value once.
- You're calling a notifier method.
- You're handling side effects like navigation or snackbars.
- Rebuilding would be unnecessary.

In those cases, use:

- `ref.read()`
- `ref.listen()`

---

## Best Practices

- Prefer `watch()` for displaying state.
- Keep watched providers focused on a single responsibility.
- Watch only the providers you actually need.
- Use `select()` when only part of the state is required.
- Avoid watching providers inside loops or unnecessary computations.

---

## Common Mistakes

### 1. Using `read()` Instead of `watch()`

❌ Wrong

```dart
final counter = ref.read(counterProvider);
```

Why it's wrong:

- The widget won't rebuild when the counter changes.

✔ Correct

```dart
final counter = ref.watch(counterProvider);
```

---

### 2. Watching a Notifier Instead of Its State

❌ Wrong

```dart
final notifier = ref.watch(counterProvider.notifier);
```

Why it's wrong:

- Watching the notifier does **not** rebuild when its state changes.

✔ Correct

```dart
final counter = ref.watch(counterProvider);
```

Watch the provider's state, not the notifier, unless you specifically need the notifier instance.

---

### 3. Using `watch()` for Side Effects

❌ Wrong

```dart
final auth = ref.watch(authProvider);

if (auth.isLoggedIn) {
  Navigator.pushNamed(context, '/home');
}
```

Why it's wrong:

- Navigation is a side effect.
- It may execute multiple times due to rebuilds.

✔ Correct

```dart
ref.listen(authProvider, (previous, next) {
  if (next.isLoggedIn) {
    Navigator.pushNamed(context, '/home');
  }
});
```

---

### 4. Watching More Providers Than Necessary

❌ Wrong

```dart
final user = ref.watch(userProvider);
final settings = ref.watch(settingsProvider);
final cart = ref.watch(cartProvider);
```

Why it's wrong:

- Every watched provider can trigger a rebuild.
- Watching unnecessary providers reduces performance.

✔ Correct

Only watch providers whose values are actually needed.

---

## Related APIs

- `ref.read()`
- `ref.listen()`
- `ref.select()`
- `ConsumerWidget`
- `WidgetRef`
- `NotifierProvider`
- `AsyncNotifierProvider`

---

## Summary

`ref.watch()` is Riverpod's primary reactive API. It reads a provider's current value, registers the caller as a dependency, and automatically rebuilds the caller whenever that provider's state changes. It is the foundation of Riverpod's reactive dependency system and should be the default choice whenever your UI or provider needs to stay synchronized with changing state.