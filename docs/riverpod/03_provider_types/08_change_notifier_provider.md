# ChangeNotifierProvider (Legacy)

> **Legacy Provider**
>
> `ChangeNotifierProvider` exists primarily to support migration from Flutter's `ChangeNotifier` architecture. For new Riverpod applications, prefer `NotifierProvider` or `AsyncNotifierProvider`.

---

## What is it?

`ChangeNotifierProvider` exposes a Flutter `ChangeNotifier` object through Riverpod.

It allows existing `ChangeNotifier`-based classes to be used without rewriting them immediately.

This provider is mainly intended for:

- Migrating existing Flutter applications
- Supporting third-party libraries that expose `ChangeNotifier`
- Incremental migration to modern Riverpod APIs

It is **not recommended** for new projects.

---

## Why does it exist?

Before Riverpod introduced `Notifier`, Flutter applications commonly used:

- `ChangeNotifier`
- `Provider` package
- `notifyListeners()`

Many existing applications still rely on this architecture.

Instead of forcing developers to rewrite everything at once, Riverpod provides `ChangeNotifierProvider` as a compatibility layer.

This allows gradual migration to the newer notifier-based APIs.

---

## Syntax

### Creating a ChangeNotifier

```dart
class CounterNotifier extends ChangeNotifier {
  int count = 0;

  void increment() {
    count++;
    notifyListeners();
  }
}
```

Explanation:

- `ChangeNotifier` stores mutable state.
- `notifyListeners()` rebuilds all listeners.

---

### Creating a ChangeNotifierProvider

```dart
final counterProvider =
    ChangeNotifierProvider<CounterNotifier>((ref) {
  return CounterNotifier();
});
```

Explanation:

- Exposes the `CounterNotifier`.
- Riverpod manages its lifecycle.

---

### Reading the Provider

```dart
class CounterText extends ConsumerWidget {
  const CounterText({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counter = ref.watch(counterProvider);

    return Text('${counter.count}');
  }
}
```

Explanation:

- `ref.watch()` returns the `CounterNotifier`.
- Widgets rebuild when `notifyListeners()` is called.

---

### Calling Methods

```dart
ref.read(counterProvider).increment();
```

Explanation:

- The provider exposes the notifier directly.
- Calling `increment()` updates the state and notifies listeners.

---

## Mental Model

`ChangeNotifierProvider` is simply a bridge between Riverpod and Flutter's `ChangeNotifier`.

```text
            Widget
               │
               ▼
ChangeNotifierProvider
               │
               ▼
      ChangeNotifier
               │
        notifyListeners()
               │
               ▼
        Widget Rebuild
```

Unlike `Notifier`, state changes are triggered manually using `notifyListeners()`.

---

## Examples

### Counter

```dart
class CounterNotifier extends ChangeNotifier {
  int count = 0;

  void increment() {
    count++;
    notifyListeners();
  }
}
```

Explanation:

- Every update requires `notifyListeners()`.
- Forgetting it prevents UI updates.

---

### Theme Manager

```dart
class ThemeNotifier extends ChangeNotifier {
  bool isDark = false;

  void toggle() {
    isDark = !isDark;
    notifyListeners();
  }
}
```

Explanation:

- The notifier manages mutable state.
- Widgets rebuild after each notification.

---

### Form Model

```dart
class LoginForm extends ChangeNotifier {
  String email = '';

  void updateEmail(String value) {
    email = value;
    notifyListeners();
  }
}
```

Explanation:

- Updates form state.
- Every field change requires a notification.

---

## How It Differs from NotifierProvider

With `ChangeNotifier`:

```dart
count++;
notifyListeners();
```

You are responsible for notifying listeners.

---

With `Notifier`:

```dart
state++;
```

Riverpod automatically notifies listeners.

No explicit notification is required.

---

## ChangeNotifierProvider vs NotifierProvider

| Feature | ChangeNotifierProvider | NotifierProvider |
|---------|------------------------|------------------|
| Recommended for new apps | ❌ | ✅ |
| Manual notifications | ✅ | ❌ |
| `state` property | ❌ | ✅ |
| Immutable updates | Optional | Recommended |
| Business logic | ✅ | ✅ |
| Automatic notifications | ❌ | ✅ |

---

## When to Use

Use `ChangeNotifierProvider` only when:

- Migrating an existing Flutter project
- Using a third-party package that exposes `ChangeNotifier`
- Gradually moving from the Provider package to Riverpod

---

## When NOT to Use

Avoid it in new applications.

Instead, use:

| Scenario | Recommended Provider |
|----------|----------------------|
| Mutable synchronous state | `NotifierProvider` |
| Mutable asynchronous state | `AsyncNotifierProvider` |
| Read-only value | `Provider` |
| Simple UI state | `StateProvider` |

---

## Best Practices

- Prefer `NotifierProvider` for new code.
- Keep `ChangeNotifierProvider` only as a migration tool.
- Migrate one feature at a time instead of rewriting the entire application.
- Replace mutable models with immutable state when migrating.
- Avoid introducing new `ChangeNotifier` classes into a modern Riverpod project.

---

## Common Mistakes

### 1. Forgetting `notifyListeners()`

❌ Wrong

```dart
void increment() {
  count++;
}
```

Why it's wrong:

- Widgets are never notified.
- The UI does not rebuild.

✔ Correct

```dart
void increment() {
  count++;
  notifyListeners();
}
```

---

### 2. Using ChangeNotifier in New Projects

❌ Wrong

```dart
final counterProvider =
    ChangeNotifierProvider((ref) {
  return CounterNotifier();
});
```

Why it's wrong:

- Modern Riverpod provides better alternatives.
- Requires manual notifications.
- Encourages mutable state.

✔ Correct

```dart
final counterProvider =
    NotifierProvider<Counter, int>(
      Counter.new,
    );
```

---

### 3. Mutating Large Objects

❌ Wrong

```dart
user.name = 'John';
notifyListeners();
```

Why it's wrong:

- Mutable state becomes difficult to track.
- Debugging unexpected changes is harder.

✔ Correct

Prefer immutable models and update them through a `Notifier`.

---

## Migration Example

### Before

```dart
class Counter extends ChangeNotifier {
  int count = 0;

  void increment() {
    count++;
    notifyListeners();
  }
}
```

---

### After

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Explanation:

- `state` replaces the mutable field.
- `notifyListeners()` is no longer needed.
- Riverpod automatically updates consumers.

---

## Recommendation

> **Avoid `ChangeNotifierProvider` in new applications.**
>
> It exists primarily for compatibility with existing Flutter codebases. For new development, `NotifierProvider` provides a cleaner, safer, and more idiomatic Riverpod experience.

---

## Related APIs

- NotifierProvider
- AsyncNotifierProvider
- StateNotifierProvider (Legacy)
- ChangeNotifier
- Provider

---

## Summary

`ChangeNotifierProvider` is a legacy provider that integrates Flutter's `ChangeNotifier` with Riverpod. It allows existing `ChangeNotifier`-based code to work within Riverpod while providing a migration path to modern APIs. For new applications, `NotifierProvider` is the recommended replacement because it eliminates manual notifications, encourages immutable state, and offers a cleaner architecture.