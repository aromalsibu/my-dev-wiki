# WidgetRef

The object that provides access to Riverpod providers inside Flutter widgets.

---

# What is it?

`WidgetRef` is the core object that connects Flutter widgets to Riverpod.

It is passed into:

- `ConsumerWidget`
- `ConsumerState`
- `Consumer`

It allows widgets to **read, watch, and listen to providers**.

Without `WidgetRef`, a widget cannot interact with Riverpod.

---

# Why does it exist?

Flutter widgets do not have built-in access to external state systems.

Riverpod introduces `WidgetRef` to:

- Bridge Flutter UI and Riverpod container
- Provide a unified API for provider access
- Enable reactive updates (`watch`)
- Support one-time reads (`read`)
- Enable event listening (`listen`)

Before `WidgetRef`, provider access was tightly coupled with widget wrappers and less structured patterns.

---

# Syntax

## In ConsumerWidget

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(myProvider);

    return Text('$value');
  }
}
```

Explanation:

- `ref` is injected into `build`
- Used to access providers directly
- Automatically rebuilds when watched providers change

---

## In ConsumerState

```dart
class _MyState extends ConsumerState<MyWidget> {
  @override
  Widget build(BuildContext context) {
    final value = ref.watch(myProvider);

    return Text('$value');
  }
}
```

Explanation:

- `ref` is available inside state class
- Same API as ConsumerWidget
- Works with lifecycle methods

---

## ref.watch()

```dart
final count = ref.watch(counterProvider);
```

Explanation:

- Subscribes to provider
- Rebuilds widget when value changes

---

## ref.read()

```dart
final value = ref.read(counterProvider);
```

Explanation:

- Gets value once
- Does NOT rebuild widget
- Used for actions or one-time access

---

## ref.listen()

```dart
ref.listen(counterProvider, (previous, next) {
  print('Counter changed: $next');
});
```

Explanation:

- Listens to changes without rebuilding UI
- Used for side effects (snackbars, navigation, logging)

---

# Mental Model

Think of `WidgetRef` as the **control panel for Riverpod inside a widget**.

```text
Widget
  │
  ▼
WidgetRef
  ├── watch → reactive UI updates
  ├── read  → one-time access
  └── listen → side effects
```

It is the only gateway between UI and providers.

---

# Examples

## Simple Usage

```dart
final messageProvider = Provider<String>((ref) {
  return 'Hello Riverpod';
});
```

```dart
class MessageView extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final message = ref.watch(messageProvider);

    return Text(message);
  }
}
```

Explanation:

- Widget reacts automatically to provider changes
- Clean separation of UI and state

---

## Real-World Example

```dart
final authProvider = NotifierProvider<AuthNotifier, AuthState>(
  AuthNotifier.new,
);
```

```dart
class LoginButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isLoading = ref.watch(authProvider).isLoading;

    return ElevatedButton(
      onPressed: isLoading
          ? null
          : () {
              ref.read(authProvider.notifier).login();
            },
      child: Text(isLoading ? 'Loading...' : 'Login'),
    );
  }
}
```

Explanation:

- `watch` for UI state
- `read` for actions
- Clear separation of concerns

---

## Side Effect Example

```dart
ref.listen(authProvider, (prev, next) {
  if (next.isAuthenticated) {
    Navigator.pushNamed(context, '/home');
  }
});
```

Explanation:

- No UI rebuild
- Used for navigation logic
- Reacts only to state changes

---

# When to Use

Use `WidgetRef` when:

- You need to access providers in UI
- You want reactive updates (`watch`)
- You need to trigger actions (`read`)
- You need side effects (`listen`)
- You are inside any Riverpod widget

---

# When NOT to Use

Avoid using `WidgetRef` when:

- You are outside Flutter widget tree
- You are inside pure Dart services
- You are writing business logic (use providers instead)
- You are not interacting with UI

---

# Best Practices

- Use `watch` only inside build methods
- Use `read` for actions and events
- Use `listen` for side effects only
- Avoid heavy logic inside widgets
- Keep providers responsible for business logic
- Prefer `ConsumerWidget` over `Consumer` when possible

---

# Common Mistakes

## Using watch for One-Time Actions

**Wrong**

```dart
ref.watch(counterProvider);
```

Inside button callback or init logic.

**Correct**

```dart
ref.read(counterProvider);
```

---

## Using read in Build for Reactive UI

**Wrong**

```dart
final value = ref.read(counterProvider);
```

UI will NOT update.

**Correct**

```dart
final value = ref.watch(counterProvider);
```

---

## Misusing listen for UI Rendering

**Wrong**

```dart
ref.listen(provider, (_, next) {
  return Text('$next');
});
```

`listen` is not for UI.

**Correct**

Use `watch` for UI, `listen` for side effects.

---

# Related APIs

- `ConsumerWidget`
- `ConsumerStatefulWidget`
- `Consumer`
- `ref.watch()`
- `ref.read()`
- `ref.listen()`
- `Provider`
- `AsyncValue`

---

# Summary

`WidgetRef` is the central interface between Flutter widgets and Riverpod. It enables widgets to react to provider changes, trigger actions, and handle side effects through `watch`, `read`, and `listen`, forming the core interaction layer of Riverpod’s UI integration.