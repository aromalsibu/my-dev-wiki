# select()

Listen to only a specific part of a provider's state to avoid unnecessary widget rebuilds.

---

# What is it?

`select()` is a Riverpod API that allows a widget or provider to watch only a derived value instead of the entire state.

Normally, `ref.watch(provider)` rebuilds whenever the provider's state changes. With `select()`, Riverpod compares only the selected value and rebuilds only if that value changes.

This is one of the most effective optimization techniques for reducing rebuilds in large applications.

---

# Why does it exist?

Many state objects contain multiple fields.

For example:

```dart
class User {
  final String name;
  final String email;
  final String avatarUrl;
  final bool isPremium;
}
```

If a widget only displays the user's name, it shouldn't rebuild when the email or avatar changes.

Without `select()`, every change to the `User` object causes the widget to rebuild.

`select()` solves this by subscribing only to the property that the widget actually uses.

---

# Syntax

## Watching a single property

```dart
final name = ref.watch(
  userProvider.select((user) => user.name),
);
```

Explanation:

- `select()` transforms the provider's state into the value you want to observe.
- The widget rebuilds only when `user.name` changes.

---

## Watching a boolean

```dart
final isPremium = ref.watch(
  userProvider.select((user) => user.isPremium),
);
```

Explanation:

- Only changes to `isPremium` trigger a rebuild.
- Other changes to the `User` object are ignored.

---

## Using multiple selections

```dart
final name = ref.watch(
  userProvider.select((user) => user.name),
);

final theme = ref.watch(
  settingsProvider.select((settings) => settings.themeMode),
);
```

Explanation:

- Each `select()` independently tracks its selected value.
- The widget rebuilds only when one of the selected values changes.

---

# Mental Model

Without `select()`:

```text
User
 ├── Name
 ├── Email
 ├── Avatar
 └── Premium

        │
        ▼

Entire widget rebuilds
```

With `select()`:

```text
User
 ├── Name  ◄──────── Watching
 ├── Email
 ├── Avatar
 └── Premium

Only rebuild when Name changes
```

Think of `select()` as subscribing to a single field instead of the entire object.

---

# Examples

## Simple Example

```dart
class UserName extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final name = ref.watch(
      userProvider.select((user) => user.name),
    );

    return Text(name);
  }
}
```

Explanation:

- The widget ignores changes to every property except `name`.

---

## Real-World Example

```dart
class PremiumBadge extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isPremium = ref.watch(
      userProvider.select((user) => user.isPremium),
    );

    if (!isPremium) {
      return const SizedBox.shrink();
    }

    return const Icon(Icons.star);
  }
}
```

Explanation:

- The badge rebuilds only when the premium status changes.
- Changes to the user's name or profile picture do not affect this widget.

---

# When to Use

Use `select()` when:

- Watching large state objects.
- Displaying only one property.
- Optimizing frequently rebuilt widgets.
- Building large lists.
- Creating dashboards.
- Working with immutable models.

---

# When NOT to Use

Avoid `select()` when:

- The widget needs the entire object.
- The provider exposes only a single value.
- The optimization adds unnecessary complexity.

Don't use `select()` everywhere by default. Use it where it provides a measurable benefit.

---

# Best Practices

- Keep selected values simple.
- Select immutable values.
- Use `select()` for frequently changing state.
- Prefer multiple small providers over deeply nested selections.
- Avoid expensive computations inside the selector.

---

# Common Mistakes

## Watching the whole object

Wrong:

```dart
final user = ref.watch(userProvider);

return Text(user.name);
```

Explanation:

- Any change to the `User` object rebuilds the widget.

Correct:

```dart
final name = ref.watch(
  userProvider.select((user) => user.name),
);

return Text(name);
```

Explanation:

- Only changes to `name` trigger a rebuild.

---

## Selecting computed values with expensive logic

Wrong:

```dart
final value = ref.watch(
  userProvider.select(
    (user) => heavyCalculation(user),
  ),
);
```

Explanation:

- The selector executes whenever Riverpod evaluates the provider.
- Expensive work should not be performed here.

Correct:

```dart
final score = ref.watch(
  userProvider.select((user) => user.score),
);
```

Explanation:

- Keep selectors lightweight.
- Move heavy computations into dedicated providers.

---

## Selecting mutable collections

Wrong:

```dart
final items = ref.watch(
  cartProvider.select((cart) => cart.items),
);
```

Explanation:

- Mutating the same list instance may prevent Riverpod from detecting changes correctly.

Correct:

```dart
final itemCount = ref.watch(
  cartProvider.select((cart) => cart.items.length),
);
```

Explanation:

- Prefer immutable collections or select simple immutable values.

---

# Related APIs

- `ref.watch()`
- Provider
- NotifierProvider
- AsyncNotifierProvider
- ConsumerWidget
- WidgetRef
- Provider Families
- Minimizing Rebuilds

---

# Summary

`select()` allows widgets to listen to only the part of a provider's state that they actually use. By selecting specific properties instead of entire objects, you can significantly reduce unnecessary rebuilds and improve the performance of large Riverpod applications.