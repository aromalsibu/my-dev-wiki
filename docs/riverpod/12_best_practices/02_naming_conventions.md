# Naming Conventions

Use clear, consistent names for providers, notifiers, and related classes to make your Riverpod code easy to read and maintain.

---

# What is it?

Naming conventions are a set of guidelines for naming providers, notifiers, models, repositories, and other components.

Riverpod does not require a specific naming style, but following consistent conventions makes your codebase easier to navigate and understand.

Good names should describe **what something is**, not **how it works**.

---

# Why does it exist?

As an application grows, inconsistent names make code difficult to understand.

Poor naming can lead to:

- Duplicate providers
- Confusing responsibilities
- Difficult code navigation
- Inconsistent APIs
- Longer onboarding for new developers

A consistent naming strategy makes the entire project predictable.

---

# Syntax

## Name providers with the `Provider` suffix

```dart
final userProvider =
    NotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );
```

Explanation:

- The provider variable ends with `Provider`.
- This immediately identifies it as a Riverpod provider.

---

## Name notifiers with the `Notifier` suffix

```dart
class UserNotifier extends Notifier<User> {
  @override
  User build() {
    return const User();
  }
}
```

Explanation:

- Classes extending `Notifier` should end with `Notifier`.
- The name reflects the class's responsibility.

---

## Name providers after the data they expose

```dart
final themeProvider =
    NotifierProvider<ThemeNotifier, ThemeMode>(
      ThemeNotifier.new,
    );
```

Explanation:

- The provider name describes the exposed state.
- Consumers immediately know what the provider returns.

---

# Mental Model

Keep names predictable.

```text
User
    │
    ├── UserNotifier
    ├── userProvider
    ├── UserRepository
    └── UserService
```

Every related component shares the same root name.

---

# Examples

## Simple Example

```dart
final counterProvider =
    NotifierProvider<CounterNotifier, int>(
      CounterNotifier.new,
    );
```

Explanation:

- `counterProvider` exposes counter state.
- `CounterNotifier` manages that state.

---

## Real-World Example

```dart
final authenticationProvider =
    AsyncNotifierProvider<
        AuthenticationNotifier,
        User?>(
      AuthenticationNotifier.new,
    );
```

Explanation:

- The provider and notifier describe the same feature.
- Naming remains consistent across the codebase.

---

# When to Use

Follow naming conventions for:

- Providers
- Notifiers
- AsyncNotifiers
- StreamNotifiers
- Repositories
- Services
- Models
- Feature folders

Consistency is more important than the exact convention you choose.

---

# When NOT to Use

Avoid abbreviations unless they are universally understood.

For example:

- `authProvider` is widely accepted.
- `usrProv` is unclear.

Also avoid names that describe implementation details instead of purpose.

---

# Best Practices

- End provider variables with `Provider`.
- End notifier classes with `Notifier`.
- Use PascalCase for classes.
- Use camelCase for provider variables.
- Name providers after the state they expose.
- Keep names descriptive but concise.
- Use consistent terminology throughout the project.

---

# Common Mistakes

## Omitting the `Provider` suffix

Wrong:

```dart
final user =
    NotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );
```

Explanation:

- It is not immediately obvious that `user` is a provider.

Correct:

```dart
final userProvider =
    NotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );
```

Explanation:

- The suffix clearly identifies the variable as a provider.

---

## Using generic names

Wrong:

```dart
final dataProvider = Provider(...);
```

Explanation:

- The name gives no indication of the provider's purpose.

Correct:

```dart
final userProfileProvider = Provider(...);
```

Explanation:

- The provider name describes the data it exposes.

---

## Inconsistent terminology

Wrong:

```dart
final userProvider = ...;

class AccountNotifier extends Notifier<User> {
}
```

Explanation:

- "User" and "Account" refer to the same concept but use different names.
- This inconsistency can confuse developers.

Correct:

```dart
final userProvider = ...;

class UserNotifier extends Notifier<User> {
}
```

Explanation:

- Related components share the same naming pattern.

---

# Related APIs

- Provider
- Notifier
- AsyncNotifier
- StreamNotifier
- Provider Families

---

# Summary

Consistent naming conventions make Riverpod applications easier to understand, navigate, and maintain. Use descriptive names, suffix providers with `Provider`, suffix notifier classes with `Notifier`, and keep related components aligned around the same terminology.