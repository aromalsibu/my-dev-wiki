# ConsumerWidget

A stateless widget that has built-in access to Riverpod providers through `WidgetRef`.

---

# What is it?

`ConsumerWidget` is a Riverpod replacement for Flutter’s `StatelessWidget`.

It gives you direct access to `WidgetRef` in the `build()` method, allowing you to read and watch providers without using a `Consumer` widget.

It is the most commonly used Riverpod widget for building reactive UI.

---

# Why does it exist?

Without `ConsumerWidget`, you would need to wrap parts of your UI in `Consumer` just to access providers.

This leads to:
- unnecessary nesting
- fragmented UI logic
- harder readability

`ConsumerWidget` solves this by making the entire widget Riverpod-aware in a clean and structured way.

It provides:

- Full provider access in `build()`
- Clean widget structure
- Automatic rebuilds on provider changes
- Better readability than nested `Consumer` widgets

---

# Syntax

## Basic Usage

```dart
class CounterView extends ConsumerWidget {
  const CounterView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Text('$count');
  }
}
```

Explanation:

- Extends `ConsumerWidget` instead of `StatelessWidget`
- `WidgetRef` gives access to Riverpod providers
- `ref.watch()` listens to provider changes and rebuilds UI

---

## Watching Multiple Providers

```dart
class UserProfileView extends ConsumerWidget {
  const UserProfileView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    final theme = ref.watch(themeProvider);

    return Column(
      children: [
        Text(user.name),
        Text('Theme: $theme'),
      ],
    );
  }
}
```

Explanation:

- Multiple providers can be watched in a single build
- Widget rebuilds if any watched provider changes

---

## Using AsyncValue

```dart
class ProfilePage extends ConsumerWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => const CircularProgressIndicator(),
      error: (e, _) => Text('Error: $e'),
    );
  }
}
```

Explanation:

- `AsyncValue.when()` handles all states
- UI reacts automatically to loading/data/error changes

---

# Mental Model

Think of `ConsumerWidget` as:

> A stateless widget with built-in reactive subscription support.

Without ConsumerWidget:

```text
StatelessWidget → no direct provider access
```

With ConsumerWidget:

```text
StatelessWidget + WidgetRef → reactive UI
```

It becomes a fully reactive stateless component.

---

# Examples

## Simple Counter

```dart
class CounterText extends ConsumerWidget {
  const CounterText({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Text('Count: $count');
  }
}
```

Explanation:

- Automatically rebuilds when counter changes
- No manual state management required

---

## Real-World Example

```dart
class UserCard extends ConsumerWidget {
  const UserCard({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);

    return Card(
      child: ListTile(
        title: Text(user.name),
        subtitle: Text(user.email),
      ),
    );
  }
}
```

Explanation:

- Clean UI component
- Automatically updates when user data changes

---

## Combining Multiple Providers

```dart
class DashboardView extends ConsumerWidget {
  const DashboardView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    final notifications = ref.watch(notificationProvider);

    return Column(
      children: [
        Text('Hello ${user.name}'),
        Text('Notifications: ${notifications.length}'),
      ],
    );
  }
}
```

Explanation:

- Combines multiple reactive sources
- UI updates when any dependency changes

---

# When to Use

Use `ConsumerWidget` when:

- Building full screens or pages
- Widget depends on one or more providers
- You want clean and readable reactive UI
- Stateless widget behavior is sufficient

---

# When NOT to Use

Avoid `ConsumerWidget` when:

- You need internal mutable state → use `ConsumerStatefulWidget`
- Only a small part of UI needs Riverpod → use `Consumer`
- You don’t need any providers at all → use `StatelessWidget`

---

# Best Practices

- Prefer `ConsumerWidget` over `Consumer` for full pages
- Keep build method clean and focused
- Avoid heavy logic inside `build()`
- Use separate widgets for complex UI sections
- Combine with `AsyncValue.when()` for async state handling

---

# Common Mistakes

## Putting Business Logic in Build

**Wrong**

```dart
build(...) {
  final result = ref.watch(provider).value * 2;
}
```

Heavy logic inside build method.

**Correct**

Move logic into provider or computed provider.

---

## Watching Too Many Providers in One Widget

**Wrong**

```dart
ref.watch(providerA);
ref.watch(providerB);
ref.watch(providerC);
```

Can lead to unnecessary rebuilds.

**Correct**

Split UI into smaller widgets if needed.

---

## Using ConsumerWidget for Everything

**Wrong**

Using it even when no providers are needed.

**Correct**

Use plain `StatelessWidget` when no Riverpod state is required.

---

# Related APIs

- `Consumer`
- `ConsumerStatefulWidget`
- `WidgetRef`
- `ref.watch()`
- `AsyncValue`
- `Provider`

---

# Summary

`ConsumerWidget` is a stateless Riverpod-aware widget that provides direct access to providers via `WidgetRef`. It simplifies reactive UI development by eliminating the need for `Consumer` wrappers and is ideal for building clean, full-screen reactive widgets.