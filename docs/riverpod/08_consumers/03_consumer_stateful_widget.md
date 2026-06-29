# ConsumerStatefulWidget

A StatefulWidget that provides direct access to Riverpod through `ConsumerState` and `WidgetRef`.

---

# What is it?

`ConsumerStatefulWidget` is Riverpod’s version of Flutter’s `StatefulWidget`.

It combines:

- Flutter’s stateful lifecycle (`initState`, `dispose`, etc.)
- Riverpod’s reactive provider system via `ref`

This allows you to use both **local mutable state** and **global reactive state** in the same widget.

---

# Why does it exist?

`ConsumerWidget` is great for stateless reactive UI, but it cannot handle:

- lifecycle hooks (`initState`, `dispose`)
- controllers (TextEditingController, AnimationController)
- subscriptions
- imperative logic tied to widget lifecycle

Before `ConsumerStatefulWidget`, you had to manually mix Riverpod with `StatefulWidget` and pass around `ref` using `Consumer`.

This created boilerplate and inconsistency.

`ConsumerStatefulWidget` solves this by:

- giving direct access to `ref` inside state class
- integrating Riverpod into Flutter lifecycle
- reducing boilerplate
- keeping reactive + imperative logic together cleanly

---

# Syntax

## Basic Usage

```dart
class CounterPage extends ConsumerStatefulWidget {
  const CounterPage({super.key});

  @override
  ConsumerState<CounterPage> createState() => _CounterPageState();
}
```

Explanation:

- Extends `ConsumerStatefulWidget` instead of `StatefulWidget`
- Creates a `ConsumerState` instead of `State`

---

## State Class with ref

```dart
class _CounterPageState extends ConsumerState<CounterPage> {
  @override
  void initState() {
    super.initState();
    ref.read(counterProvider);
  }

  @override
  Widget build(BuildContext context) {
    final count = ref.watch(counterProvider);

    return Text('$count');
  }
}
```

Explanation:

- `ref` is available in lifecycle methods and build
- `ref.watch()` triggers rebuilds
- `ref.read()` is used for one-time access

---

## Using Controllers

```dart
class SearchPage extends ConsumerStatefulWidget {
  const SearchPage({super.key});

  @override
  ConsumerState<SearchPage> createState() => _SearchPageState();
}

class _SearchPageState extends ConsumerState<SearchPage> {
  final controller = TextEditingController();

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final results = ref.watch(searchProvider(controller.text));

    return Column(
      children: [
        TextField(controller: controller),
        Expanded(child: Text(results.toString())),
      ],
    );
  }
}
```

Explanation:

- Local controller is managed via lifecycle methods
- Riverpod reacts to input changes via provider
- Combines imperative and reactive logic

---

# Mental Model

Think of `ConsumerStatefulWidget` as:

> A StatefulWidget with built-in access to Riverpod.

Without Riverpod:

```text
StatefulWidget → local state only
```

With Riverpod:

```text
StatefulWidget + ref → local + global reactive state
```

It bridges Flutter lifecycle and Riverpod state system.

---

# Examples

## Simple Counter

```dart
class CounterPage extends ConsumerStatefulWidget {
  const CounterPage({super.key});

  @override
  ConsumerState<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends ConsumerState<CounterPage> {
  @override
  Widget build(BuildContext context) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Center(
        child: Text('Count: $count'),
      ),
    );
  }
}
```

Explanation:

- Uses Riverpod state inside StatefulWidget
- Rebuilds automatically when counter changes

---

## Real-World Example

```dart
class ProfilePage extends ConsumerStatefulWidget {
  const ProfilePage({super.key});

  @override
  ConsumerState<ProfilePage> createState() => _ProfilePageState();
}

class _ProfilePageState extends ConsumerState<ProfilePage> {
  @override
  void initState() {
    super.initState();
    ref.read(profileProvider.notifier).loadProfile();
  }

  @override
  Widget build(BuildContext context) {
    final profile = ref.watch(profileProvider);

    return profile.when(
      data: (user) => Text(user.name),
      loading: () => const CircularProgressIndicator(),
      error: (e, _) => Text('Error: $e'),
    );
  }
}
```

Explanation:

- `initState` triggers initial data load
- `build` reacts to provider updates
- Clean separation of lifecycle and UI logic

---

## Form Handling Example

```dart
class LoginPage extends ConsumerStatefulWidget {
  const LoginPage({super.key});

  @override
  ConsumerState<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends ConsumerState<LoginPage> {
  final emailController = TextEditingController();

  @override
  void dispose() {
    emailController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final isLoading = ref.watch(authProvider).isLoading;

    return Column(
      children: [
        TextField(controller: emailController),
        ElevatedButton(
          onPressed: isLoading
              ? null
              : () {
                  ref
                      .read(authProvider.notifier)
                      .login(emailController.text);
                },
          child: Text(isLoading ? 'Loading...' : 'Login'),
        ),
      ],
    );
  }
}
```

Explanation:

- Combines controller + Riverpod state
- Uses notifier for actions
- Handles UI state cleanly

---

# When to Use

Use `ConsumerStatefulWidget` when:

- You need controllers (TextEditingController, AnimationController)
- You rely on lifecycle methods (initState, dispose)
- You need to trigger side effects
- You combine local + global state
- You handle forms, animations, or subscriptions

---

# When NOT to Use

Avoid `ConsumerStatefulWidget` when:

- No local state is needed → use `ConsumerWidget`
- You only display provider data
- Widget is purely declarative
- No lifecycle logic is required

---

# Best Practices

- Use `ref.read()` in lifecycle methods, not `watch()`
- Dispose controllers properly in `dispose()`
- Keep `build()` focused on UI only
- Move business logic into providers
- Avoid heavy logic inside state class
- Prefer `ConsumerWidget` unless StatefulWidget is necessary

---

# Common Mistakes

## Using ref.watch in initState

**Wrong**

```dart
@override
void initState() {
  ref.watch(counterProvider);
  super.initState();
}
```

**Why it's wrong:**
`watch` cannot be used outside build lifecycle.

**Correct**

```dart
ref.read(counterProvider);
```

---

## Putting Logic Inside Build

**Wrong**

```dart
build(...) {
  ref.read(provider).fetchData();
}
```

Causes repeated side effects on rebuild.

**Correct**

Use `initState` or provider logic instead.

---

## Not Disposing Controllers

**Wrong**

```dart
final controller = TextEditingController();
```

No cleanup leads to memory leaks.

**Correct**

```dart
@override
void dispose() {
  controller.dispose();
  super.dispose();
}
```

---

# Related APIs

- `ConsumerWidget`
- `Consumer`
- `WidgetRef`
- `ref.watch()`
- `ref.read()`
- `NotifierProvider`
- `AsyncValue`

---

# Summary

`ConsumerStatefulWidget` is a StatefulWidget integrated with Riverpod. It allows you to combine Flutter lifecycle methods with reactive provider access, making it ideal for forms, controllers, and side-effect-heavy UI components.