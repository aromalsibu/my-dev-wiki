# Logging

Record provider lifecycle events and state changes for debugging and diagnostics.

---

# What is it?

Logging is the process of recording provider activity as your application runs.

Using Riverpod's `ProviderObserver`, you can log important events such as:

- Provider creation
- State updates
- Provider disposal
- Provider failures

Logging helps you understand how state flows through your application without modifying your business logic.

---

# Why does it exist?

As applications grow, understanding why a provider changed becomes increasingly difficult.

Without logging, you may struggle to answer questions like:

- Which provider changed?
- What was the previous value?
- What triggered this update?
- Was the provider disposed correctly?

Logging provides visibility into the application's state management and simplifies debugging.

---

# Syntax

## Basic Logging

```dart
class AppProviderObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    print(
      '${context.provider.name ?? context.provider.runtimeType}: '
      '$previousValue -> $newValue',
    );
  }
}
```

Explanation:

- `didUpdateProvider()` is called whenever a provider's value changes.
- Both the previous and new values are available.
- The provider name helps identify which provider changed.

---

## Registering the Observer

```dart
void main() {
  runApp(
    ProviderScope(
      observers: [
        AppProviderObserver(),
      ],
      child: const MyApp(),
    ),
  );
}
```

Explanation:

- Register the observer inside `ProviderScope`.
- Every provider within the scope is logged.

---

## Logging Provider Disposal

```dart
class AppProviderObserver extends ProviderObserver {
  @override
  void didDisposeProvider(
    ProviderObserverContext context,
  ) {
    print(
      'Disposed: ${context.provider.name ?? context.provider.runtimeType}',
    );
  }
}
```

Explanation:

- Logs when a provider is disposed.
- Useful for verifying `.autoDispose` behavior.

---

# Mental Model

Think of logging as an application event log.

```text
Provider Changes
        │
        ▼
 ProviderObserver
        │
        ▼
      Log Output
```

Instead of guessing what happened, you can inspect the recorded events.

---

# Examples

## Logging Counter Updates

```dart
class Logger extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    print(
      'Counter: $previousValue → $newValue',
    );
  }
}
```

Explanation:

- Every counter update is printed.
- Useful while developing reactive features.

---

## Logging User Authentication

```dart
class AuthLogger extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    if (context.provider.name == 'authProvider') {
      print('Authentication state changed.');
    }
  }
}
```

Explanation:

- Logs only authentication-related updates.
- Reduces unnecessary log output.

---

## Structured Logging

```dart
class Logger extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    debugPrint(
      '[${DateTime.now()}] '
      '${context.provider.name}: '
      '$newValue',
    );
  }
}
```

Explanation:

- Includes timestamps.
- Easier to trace application events.

---

# When to Use

Use logging when:

- Developing new features
- Investigating bugs
- Understanding provider lifecycle
- Monitoring state changes
- Verifying application behavior

---

# When NOT to Use

Avoid excessive logging when:

- Running production builds
- Logging sensitive information
- Monitoring every provider unnecessarily
- Logging large objects repeatedly

Too much logging can make useful information difficult to find.

---

# Best Practices

- Log meaningful events only.
- Use provider names whenever possible.
- Avoid logging passwords, tokens, or personal data.
- Prefer `debugPrint()` over `print()` for Flutter applications.
- Disable verbose logging in release builds.

---

# Common Mistakes

## Logging Sensitive Data

**Wrong**

```dart
print(user.password);
```

Sensitive information should never appear in logs.

**Correct**

```dart
print('User authenticated.');
```

Log the event, not confidential data.

---

## Logging Every Provider

**Wrong**

```dart
didUpdateProvider(...) {
  print(newValue);
}
```

Large applications generate excessive output.

**Correct**

Filter important providers or log only during development.

---

## Using Logging as Business Logic

**Wrong**

```dart
didUpdateProvider(...) {
  saveUser();
}
```

Logging should never modify application state.

**Correct**

Use logging only to observe provider events.

---

# Related APIs

- `ProviderObserver`
- `ProviderScope`
- `ref.watch()`
- `ref.listen()`
- `.autoDispose`

---

# Summary

Logging provides visibility into Riverpod's provider lifecycle and state changes. By using `ProviderObserver`, you can monitor provider activity, diagnose issues, and understand how state flows through your application while keeping business logic separate from diagnostics.