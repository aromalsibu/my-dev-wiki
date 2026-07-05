# Minimizing Rebuilds

Reduce unnecessary widget rebuilds by watching only the state that actually changes.

---

# What is it?

Minimizing rebuilds is the practice of ensuring that only the widgets affected by a state change are rebuilt.

Whenever a provider's value changes, every widget watching that provider rebuilds. While Flutter rebuilds are generally inexpensive, rebuilding large widget trees unnecessarily can reduce performance and make your UI less responsive.

Riverpod provides several techniques to keep rebuilds as small as possible.

---

# Why does it exist?

If every state change rebuilt the entire UI, applications would become inefficient as they grow.

For example:

- Updating a counter should not rebuild the entire home screen.
- Changing a user's avatar should not rebuild unrelated settings widgets.
- Updating a loading indicator should not rebuild a complete page.

Riverpod encourages fine-grained reactivity so that each widget listens only to the state it actually needs.

---

# Syntax

## Watch only where data is needed

```dart
class UserName extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);

    return Text(user.name);
  }
}
```

Explanation:

- `ref.watch()` subscribes this widget to `userProvider`.
- Only `UserName` rebuilds when the provider changes.

---

## Split large widgets

```dart
class ProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        UserAvatar(),
        UserName(),
        UserStatistics(),
      ],
    );
  }
}
```

Explanation:

- Each child widget can watch its own provider.
- Updating one section does not rebuild the others.

---

## Watch different providers

```dart
final user = ref.watch(userProvider);
final notifications = ref.watch(notificationProvider);
```

Explanation:

- Widgets rebuild only when one of their watched providers changes.
- Keep providers focused on a single responsibility.

---

# Mental Model

Think of every provider as its own notification channel.

```text
User Provider
      │
      ▼
 UserName Widget

Notification Provider
      │
      ▼
NotificationBadge

Theme Provider
      │
      ▼
ThemeSwitcher
```

When one provider updates, only the widgets listening to that provider rebuild.

Small listeners produce small rebuilds.

---

# Examples

## Simple Example

```dart
final counter = ref.watch(counterProvider);

return Text('$counter');
```

Explanation:

- Only this widget rebuilds when the counter changes.

---

## Real-World Example

```dart
class Dashboard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        UserHeader(),
        NotificationsPanel(),
        WeatherCard(),
        NewsFeed(),
      ],
    );
  }
}
```

Explanation:

- Each section watches different providers.
- Updating weather does not rebuild the news feed.
- Updating notifications does not rebuild the user header.

---

# When to Use

Use rebuild optimization when:

- Building large screens.
- Working with frequently changing state.
- Creating dashboards.
- Building lists with many items.
- Optimizing animations and live updates.
- Improving application performance.

---

# When NOT to Use

Do not optimize prematurely.

Avoid making your widget tree overly complex just to reduce a few rebuilds.

For small applications, readability is usually more important than micro-optimizations.

Only optimize when rebuilds become measurable or affect user experience.

---

# Best Practices

- Keep widgets small.
- Watch providers as close as possible to where data is used.
- Create focused providers with a single responsibility.
- Split large pages into reusable widgets.
- Use immutable state.
- Use `select()` when only a single property is needed.
- Prefer multiple small providers over one massive provider.

---

# Common Mistakes

## Watching state too high in the widget tree

Wrong:

```dart
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);

    return EntireApplication(user);
  }
}
```

Explanation:

- Every change rebuilds the entire application subtree.

Correct:

```dart
class UserName extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);

    return Text(user.name);
  }
}
```

Explanation:

- Only the widget displaying the user's name rebuilds.

---

## Creating huge providers

Wrong:

```dart
final appProvider = Provider<AppState>((ref) {
  return AppState(...);
});
```

Explanation:

- Any change may trigger rebuilds across unrelated widgets.

Correct:

```dart
final userProvider = Provider<User>(...);

final settingsProvider = Provider<Settings>(...);

final notificationProvider = Provider<Notifications>(...);
```

Explanation:

- Separate providers reduce unnecessary rebuilds.

---

## Ignoring `select()`

Wrong:

```dart
final user = ref.watch(userProvider);

return Text(user.name);
```

Explanation:

- Changes to any field of `User` rebuild the widget.

Correct:

```dart
final name = ref.watch(
  userProvider.select((user) => user.name),
);

return Text(name);
```

Explanation:

- The widget rebuilds only when `name` changes.

---

# Related APIs

- Provider
- NotifierProvider
- AsyncNotifierProvider
- ConsumerWidget
- WidgetRef
- `ref.watch()`
- `select()`
- `.family`
- `.autoDispose`

---

# Summary

Minimizing rebuilds is about making widgets react only to the state they actually use.

Build small widgets, create focused providers, watch state close to where it is displayed, and use techniques like `select()` to avoid unnecessary updates. These practices help Riverpod applications remain responsive and scalable as they grow.