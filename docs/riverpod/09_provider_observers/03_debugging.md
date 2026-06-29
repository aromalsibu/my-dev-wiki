# Debugging

Use Riverpod's debugging tools to inspect provider state, lifecycle events, and application behavior.

---

# What is it?

Debugging is the process of identifying, understanding, and fixing issues related to providers and state management.

Riverpod includes several features that make debugging easier, such as:

- `ProviderObserver`
- Provider names
- `ProviderContainer`
- `ref.listen()`
- Flutter DevTools integration

Together, these tools help you understand how providers are created, updated, and disposed.

---

# Why does it exist?

State management bugs are often difficult to diagnose because they happen behind the scenes.

Common questions include:

- Why did this widget rebuild?
- Why is a provider recreated?
- Why isn't the UI updating?
- Why is data fetched multiple times?
- Why wasn't a provider disposed?

Riverpod's debugging tools provide insight into these questions without changing application behavior.

---

# Syntax

## Using ProviderObserver

```dart
class AppObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    debugPrint(
      '${context.provider.name}: '
      '$previousValue → $newValue',
    );
  }
}
```

Explanation:

- Logs every provider update.
- Helps identify unexpected state changes.

---

## Naming Providers

```dart
final counterProvider = Provider<int>(
  name: 'counterProvider',
  (ref) => 0,
);
```

Explanation:

- Assigning a name makes logs and debugging output easier to understand.
- Especially useful in large applications.

---

## Listening to State Changes

```dart
ref.listen(counterProvider, (previous, next) {
  debugPrint('Counter changed: $next');
});
```

Explanation:

- Observes provider changes without rebuilding the widget.
- Useful for debugging specific providers.

---

# Mental Model

Think of debugging as tracing the provider lifecycle.

```text
Provider Created
        │
        ▼
Provider Updated
        │
        ▼
Widget Rebuild
        │
        ▼
Provider Disposed
```

Debugging tools help you observe each step in this process.

---

# Examples

## Finding Unexpected Rebuilds

```dart
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    debugPrint('HomePage rebuilt');

    final user = ref.watch(userProvider);

    return Text(user.name);
  }
}
```

Explanation:

- Shows when the widget rebuilds.
- Helps determine if rebuilds occur more often than expected.

---

## Checking AutoDispose

```dart
class Observer extends ProviderObserver {
  @override
  void didDisposeProvider(
    ProviderObserverContext context,
  ) {
    debugPrint(
      '${context.provider.name} disposed',
    );
  }
}
```

Explanation:

- Confirms when providers are disposed.
- Useful when working with `.autoDispose`.

---

## Debugging Async Providers

```dart
final user = ref.watch(userProvider);

user.when(
  data: (_) => debugPrint('Loaded'),
  loading: () => debugPrint('Loading'),
  error: (e, _) => debugPrint('Error: $e'),
);
```

Explanation:

- Makes async state transitions visible.
- Helps identify loading or error issues.

---

# When to Use

Use debugging tools when:

- A provider behaves unexpectedly
- Widgets rebuild too often
- Async providers fail
- Providers are recreated unexpectedly
- Investigating lifecycle issues
- Diagnosing production bugs

---

# When NOT to Use

Avoid leaving debugging code in production when:

- It produces excessive logs
- It exposes sensitive information
- It impacts performance
- It makes code harder to maintain

Remove temporary debugging statements after resolving issues.

---

# Best Practices

- Give important providers descriptive names.
- Use `ProviderObserver` for application-wide diagnostics.
- Use `ref.listen()` to debug specific providers.
- Prefer `debugPrint()` over `print()`.
- Keep debugging code separate from business logic.

---

# Common Mistakes

## Forgetting to Name Providers

**Wrong**

```dart
final provider = Provider((ref) => User());
```

Logs may only show the provider type.

**Correct**

```dart
final userProvider = Provider<User>(
  name: 'userProvider',
  (ref) => User(),
);
```

Meaningful names simplify debugging.

---

## Leaving Debug Prints Everywhere

**Wrong**

```dart
debugPrint('Build');
debugPrint('State');
debugPrint('Update');
```

Excessive logging makes debugging harder.

**Correct**

Log only the information needed to investigate the issue.

---

## Confusing Logging with Debugging

**Wrong**

Assuming logs alone will identify every problem.

**Correct**

Use logging together with provider names, observers, lifecycle callbacks, and Flutter DevTools to understand application behavior.

---

# Related APIs

- `ProviderObserver`
- `ProviderScope`
- `ProviderContainer`
- `ref.watch()`
- `ref.listen()`
- `AsyncValue`

---

# Summary

Debugging in Riverpod involves observing provider lifecycle events, state changes, and widget rebuilds. By combining tools such as `ProviderObserver`, provider names, `ref.listen()`, and Flutter DevTools, you can quickly identify and resolve state management issues while keeping application logic clean.