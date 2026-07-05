# Splitting Providers

Break large providers into smaller, focused providers to improve readability, maintainability, and performance.

---

# What is it?

Splitting providers is the practice of creating multiple providers that each manage a single piece of state or responsibility instead of combining unrelated state into one large provider.

Rather than having one provider manage everything, Riverpod encourages composing your application from many small providers.

This follows the Single Responsibility Principle (SRP) and enables fine-grained reactivity.

---

# Why does it exist?

Large providers often become difficult to maintain because they manage unrelated pieces of state.

For example, an application provider that manages:

- User information
- Theme settings
- Notifications
- Shopping cart
- Authentication
- Localization

creates unnecessary coupling.

Whenever one part changes, widgets watching the provider may rebuild even if they only use another part of the state.

Splitting providers keeps state independent, making the application easier to scale and optimize.

---

# Syntax

## Large provider (not recommended)

```dart
final appProvider = NotifierProvider<AppNotifier, AppState>(
  AppNotifier.new,
);
```

Explanation:

- `AppState` contains many unrelated pieces of state.
- Changes to one property may affect many consumers.

---

## Split into focused providers

```dart
final userProvider =
    NotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );

final themeProvider =
    NotifierProvider<ThemeNotifier, ThemeMode>(
      ThemeNotifier.new,
    );

final cartProvider =
    NotifierProvider<CartNotifier, Cart>(
      CartNotifier.new,
    );
```

Explanation:

- Each provider manages a single responsibility.
- Widgets subscribe only to the state they need.

---

## Compose providers together

```dart
final greetingProvider = Provider<String>((ref) {
  final user = ref.watch(userProvider);

  return 'Hello ${user.name}';
});
```

Explanation:

- Providers can depend on other providers.
- Complex state can be built through composition instead of one large provider.

---

# Mental Model

Instead of one large provider:

```text
AppProvider
│
├── User
├── Theme
├── Cart
├── Notifications
└── Settings
```

Prefer multiple focused providers:

```text
UserProvider
ThemeProvider
CartProvider
NotificationProvider
SettingsProvider
        │
        ▼
Small independent rebuilds
```

Think of providers as LEGO bricks. Small pieces can be combined in many ways, while one giant block is difficult to reuse.

---

# Examples

## Simple Example

```dart
final firstNameProvider = Provider<String>((ref) {
  return 'John';
});

final lastNameProvider = Provider<String>((ref) {
  return 'Doe';
});
```

Explanation:

- Each provider exposes one value.
- They can be combined wherever needed.

---

## Real-World Example

```dart
final authProvider =
    NotifierProvider<AuthNotifier, AuthState>(
      AuthNotifier.new,
    );

final profileProvider =
    FutureProvider<Profile>((ref) async {
      final auth = ref.watch(authProvider);

      return fetchProfile(auth.userId);
    });
```

Explanation:

- `profileProvider` depends on the authenticated user.
- Authentication and profile loading remain separate concerns.

---

# When to Use

Split providers when:

- A provider manages unrelated data.
- Multiple widgets use different parts of the state.
- Different state changes occur at different frequencies.
- Features are independent.
- Building medium or large applications.
- Creating reusable business logic.

---

# When NOT to Use

Avoid splitting providers when:

- The state always changes together.
- The values are tightly coupled.
- Creating additional providers adds unnecessary complexity.
- The application is very small.

Don't create separate providers for every tiny value if they are never used independently.

---

# Best Practices

- Give each provider one responsibility.
- Organize providers by feature instead of file type.
- Compose providers instead of creating massive state objects.
- Keep provider dependencies clear and simple.
- Use derived providers for computed values.
- Prefer immutable state models.

---

# Common Mistakes

## Creating one global provider

Wrong:

```dart
class AppState {
  final User user;
  final Cart cart;
  final ThemeMode theme;
  final Settings settings;
}
```

Explanation:

- Unrelated state becomes tightly coupled.
- Changes may trigger unnecessary rebuilds.

Correct:

```dart
final userProvider = NotifierProvider<UserNotifier, User>(
  UserNotifier.new,
);

final cartProvider = NotifierProvider<CartNotifier, Cart>(
  CartNotifier.new,
);

final themeProvider =
    NotifierProvider<ThemeNotifier, ThemeMode>(
      ThemeNotifier.new,
    );
```

Explanation:

- Each provider manages one concern.
- Widgets rebuild independently.

---

## Duplicating state

Wrong:

```dart
final profileProvider = Provider<User>((ref) {
  return User(...);
});

final authProvider = Provider<User>((ref) {
  return User(...);
});
```

Explanation:

- Two providers manage the same data.
- Keeping them synchronized becomes difficult.

Correct:

```dart
final authProvider =
    NotifierProvider<AuthNotifier, AuthState>(
      AuthNotifier.new,
    );

final profileProvider = Provider<User>((ref) {
  return ref.watch(authProvider).user;
});
```

Explanation:

- One provider owns the state.
- Other providers derive values from it.

---

## Making providers depend on everything

Wrong:

```dart
final dashboardProvider = Provider((ref) {
  ref.watch(userProvider);
  ref.watch(cartProvider);
  ref.watch(themeProvider);
  ref.watch(settingsProvider);
  ref.watch(notificationProvider);

  // ...
});
```

Explanation:

- The provider rebuilds whenever any dependency changes.
- It becomes difficult to optimize.

Correct:

```dart
final greetingProvider = Provider<String>((ref) {
  final user = ref.watch(userProvider);

  return 'Hello ${user.name}';
});
```

Explanation:

- Watch only the dependencies required to produce the result.

---

# Related APIs

- Provider
- NotifierProvider
- AsyncNotifierProvider
- FutureProvider
- StreamProvider
- Provider Families
- `select()`
- `ref.watch()`

---

# Summary

Splitting providers means dividing application state into small, focused providers instead of one large provider. This improves readability, reduces unnecessary rebuilds, simplifies testing, and makes Riverpod applications easier to maintain and scale.