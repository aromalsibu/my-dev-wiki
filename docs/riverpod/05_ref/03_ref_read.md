# ref.read()

`ref.read()` reads the current value of a provider without subscribing to future changes or triggering automatic rebuilds.

---

## What is it?

`ref.read()` is used to **access a provider once**.

Unlike `ref.watch()`, it **does not create a dependency** between the caller and the provider.

This means:

- The current value is returned immediately.
- Future state changes are ignored.
- The caller will not rebuild when the provider updates.

`ref.read()` is ideal for one-time operations where reactive updates are unnecessary.

---

## Why does it exist?

Not every interaction with a provider needs to be reactive.

For example:

- Calling a notifier method
- Reading a configuration value once
- Accessing a repository
- Triggering an action from a button

If `ref.watch()` were used in these situations, unnecessary dependencies and rebuilds would occur.

`ref.read()` solves this by providing **one-time access** without affecting Riverpod's dependency graph.

---

## Syntax

### Reading a Provider

```dart
final counter = ref.read(counterProvider);
```

Explanation:

- Returns the current value.
- Does not subscribe to future updates.

---

### Reading a Notifier

```dart
final notifier = ref.read(counterProvider.notifier);

notifier.increment();
```

Explanation:

- Accesses the notifier instance.
- Used to call methods that modify state.

---

### Reading Inside a Button

```dart
ElevatedButton(
  onPressed: () {
    ref.read(counterProvider.notifier).increment();
  },
  child: const Text('Increment'),
)
```

Explanation:

- The button performs an action.
- No widget rebuild is needed.

---

## Return Value

`ref.read()` returns the provider's current value.

Examples:

| Provider Type | Return Value |
|---------------|--------------|
| `Provider<String>` | `String` |
| `NotifierProvider<Counter, int>` | `int` |
| `FutureProvider<User>` | `AsyncValue<User>` |
| `StreamProvider<Message>` | `AsyncValue<Message>` |

When reading a notifier:

| Expression | Return Type |
|------------|-------------|
| `ref.read(counterProvider.notifier)` | `Counter` |
| `ref.read(userProvider.notifier)` | `UserNotifier` |
| `ref.read(chatProvider.notifier)` | `ChatNotifier` |

---

## Execution Flow

```text
Caller
   │
   ▼
ref.read(provider)
   │
   ▼
Riverpod reads current value
   │
   ▼
Value returned
   │
   ▼
No dependency registered
   │
   ▼
Future provider changes ignored
```

Unlike `watch()`, no relationship is created between the caller and the provider.

---

## Mental Model

Think of `ref.read()` as **taking a snapshot**.

```text
Provider
   │
   ▼
Current Value
   │
   ▼
ref.read()
   │
   ▼
Snapshot Returned

Future Updates
      │
      ✖ Ignored
```

It gives you the provider's value **at this moment only**.

---

## Examples

### Simple Example

```dart
final count = ref.read(counterProvider);

print(count);
```

Explanation:

- Reads the current counter value.
- Future updates are ignored.

---

### Calling a Notifier Method

```dart
ref.read(counterProvider.notifier).increment();
```

Explanation:

- Reads the notifier.
- Executes the `increment()` method.
- Does not rebuild the caller.

---

### Reading a Repository

```dart
final repository = ref.read(userRepositoryProvider);

await repository.fetchUsers();
```

Explanation:

- Reads a dependency once.
- Ideal for services and repositories.

---

### Real-World Example

```dart
ElevatedButton(
  onPressed: () async {
    await ref
        .read(authProvider.notifier)
        .login(email, password);
  },
  child: const Text('Login'),
)
```

Explanation:

- User interaction triggers business logic.
- The button itself does not need to rebuild.

---

## When to Use

Use `ref.read()` when:

- Calling notifier methods
- Triggering actions from buttons
- Reading repositories or services
- Performing one-time reads
- Reactive updates are unnecessary

---

## When NOT to Use

Avoid `ref.read()` when:

- Displaying state in the UI
- Building reactive providers
- Automatic updates are expected

In these cases, use:

- `ref.watch()`

---

## Best Practices

- Use `read()` for actions, not rendering.
- Read notifiers when calling methods.
- Keep business logic inside notifiers.
- Use `watch()` whenever the UI should stay synchronized.
- Avoid replacing `watch()` with `read()` for performance unless rebuilds are truly unnecessary.

---

## Common Mistakes

### 1. Using `read()` to Display State

❌ Wrong

```dart
final counter = ref.read(counterProvider);

Text('$counter');
```

Why it's wrong:

- The UI will never update when the counter changes.

✔ Correct

```dart
final counter = ref.watch(counterProvider);

Text('$counter');
```

---

### 2. Using `watch()` to Call Methods

❌ Wrong

```dart
ref.watch(counterProvider.notifier).increment();
```

Why it's wrong:

- Registers an unnecessary dependency.
- The caller may rebuild without benefit.

✔ Correct

```dart
ref.read(counterProvider.notifier).increment();
```

---

### 3. Expecting `read()` to React to Changes

❌ Wrong

```dart
final user = ref.read(userProvider);

// Expecting UI to update automatically
```

Why it's wrong:

- `read()` is a one-time snapshot.
- Future updates are ignored.

✔ Correct

```dart
final user = ref.watch(userProvider);
```

---

### 4. Reading Repeatedly During Build

❌ Wrong

```dart
final user = ref.read(userProvider);
final cart = ref.read(cartProvider);
final settings = ref.read(settingsProvider);
```

Why it's wrong:

- If these values are displayed, they become stale after updates.

✔ Correct

Use `watch()` for values that affect the UI and `read()` only for one-time access.

---

## ref.read() vs ref.watch()

| Feature | `ref.read()` | `ref.watch()` |
|---------|--------------|---------------|
| Returns current value | ✅ | ✅ |
| Registers dependency | ❌ | ✅ |
| Rebuilds automatically | ❌ | ✅ |
| Best for UI rendering | ❌ | ✅ |
| Best for actions | ✅ | ❌ |

---

## Related APIs

- `ref.watch()`
- `ref.listen()`
- `Notifier`
- `NotifierProvider`
- `AsyncNotifier`
- `WidgetRef`

---

## Summary

`ref.read()` provides one-time access to a provider without creating a reactive dependency. It is the preferred API for triggering actions, calling notifier methods, and reading services or repositories when automatic rebuilds are not required. Whenever you only need the current value and don't want future updates, `ref.read()` is the right choice.