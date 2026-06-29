# Performance Monitoring

Monitor provider activity and widget rebuilds to identify performance bottlenecks in a Riverpod application.

---

# What is it?

Performance monitoring is the process of measuring how Riverpod providers and Flutter widgets behave while your application is running.

Rather than guessing why an application feels slow, performance monitoring helps you answer questions like:

- Which providers update frequently?
- Which widgets rebuild the most?
- Are providers being recreated unnecessarily?
- Is `autoDispose` working as expected?
- Are expensive computations running too often?

Riverpod provides several tools that make these issues easy to identify.

---

# Why does it exist?

A Flutter application may become slow because of unnecessary work, not because Riverpod itself is slow.

Common performance problems include:

- Unnecessary widget rebuilds
- Watching entire objects instead of specific fields
- Expensive providers recomputing repeatedly
- Missing caching
- Improper use of `autoDispose`

Monitoring helps you find these issues early and optimize only where necessary.

---

# Syntax

## Monitoring Provider Updates

```dart
class AppObserver extends ProviderObserver {
  @override
 void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    debugPrint(
      'Updated: ${context.provider.name ?? context.provider.runtimeType}',
    );
  }
}
```

Explanation:

- Logs every provider update.
- Helps identify providers that change frequently.

---

## Monitoring Widget Rebuilds

```dart
class CounterText extends ConsumerWidget {
  const CounterText({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    debugPrint('CounterText rebuilt');

    final count = ref.watch(counterProvider);

    return Text('$count');
  }
}
```

Explanation:

- Prints whenever the widget rebuilds.
- Useful for spotting unnecessary rebuilds.

---

## Using `select()` for Optimization

```dart
final username = ref.watch(
  userProvider.select((user) => user.name),
);
```

Explanation:

- Only listens to the `name` field.
- Reduces rebuilds when other properties change.

---

# Mental Model

Think of performance monitoring as measuring work done by the application.

```text
Provider Updates
        │
        ▼
 Widget Rebuilds
        │
        ▼
 Performance Analysis
        │
        ▼
 Optimization
```

The goal is not to eliminate rebuilds, but to eliminate **unnecessary** rebuilds.

---

# Examples

## Finding Frequently Updated Providers

```dart
class Observer extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    debugPrint(
      '${context.provider.name} updated',
    );
  }
}
```

Explanation:

- Reveals providers that update more often than expected.
- Helps identify excessive state changes.

---

## Detecting Rebuilds

```dart
class ProfilePage extends ConsumerWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    debugPrint('ProfilePage rebuilt');

    final user = ref.watch(userProvider);

    return Text(user.name);
  }
}
```

Explanation:

- Helps verify whether rebuild frequency matches expectations.

---

## Optimizing with Smaller Widgets

```dart
Column(
  children: const [
    Header(),
    UserProfile(),
    Footer(),
  ],
);
```

Explanation:

- Splitting large widgets limits rebuild scope.
- Only widgets that depend on changed providers rebuild.

---

# When to Use

Monitor performance when:

- The UI feels slow
- Widgets rebuild unexpectedly
- Providers update too frequently
- Network requests repeat unnecessarily
- Memory usage appears excessive
- Optimizing production applications

---

# When NOT to Use

Avoid premature optimization when:

- There is no measurable performance problem
- The application is still in early development
- Optimization makes the code significantly harder to read

Always measure before optimizing.

---

# Best Practices

- Measure performance before making changes.
- Use `ProviderObserver` during development.
- Use `select()` only when it provides measurable benefits.
- Keep widgets small and focused.
- Cache expensive computations when appropriate.
- Use `.autoDispose` for temporary state.
- Profile the application using Flutter DevTools before making major optimizations.

---

# Common Mistakes

## Optimizing Without Measuring

**Wrong**

Adding `select()` everywhere because it "might" improve performance.

**Correct**

Profile the application first, then optimize the actual bottlenecks.

---

## Watching Large Objects

**Wrong**

```dart
final user = ref.watch(userProvider);
```

When only the user's name is displayed.

**Correct**

```dart
final name = ref.watch(
  userProvider.select((u) => u.name),
);
```

Only listen to the required data.

---

## Building Large Reactive Widgets

**Wrong**

One `ConsumerWidget` containing an entire screen.

**Correct**

Split the UI into smaller widgets so rebuilds remain localized.

---

# Related APIs

- `ProviderObserver`
- `ConsumerWidget`
- `WidgetRef`
- `ref.watch()`
- `select()`
- `.autoDispose`
- `ProviderScope`

---

# Summary

Performance monitoring helps you understand how providers and widgets behave at runtime. By observing provider updates, tracking widget rebuilds, and using tools like `ProviderObserver`, `select()`, and Flutter DevTools, you can identify real performance bottlenecks and optimize your application without sacrificing readability.