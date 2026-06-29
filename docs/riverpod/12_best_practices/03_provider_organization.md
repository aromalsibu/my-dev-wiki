# Provider Organization

Organize providers by feature and responsibility to create a scalable, maintainable Riverpod application.

---

# What is it?

Provider organization is the practice of arranging providers so they are easy to find, understand, and maintain.

As a project grows, the number of providers can quickly increase. Keeping them organized prevents duplication, reduces coupling, and makes dependencies easier to manage.

A good organization strategy ensures every provider has a clear owner and purpose.

---

# Why does it exist?

Without a clear organization strategy, projects often end up with:

- Hundreds of providers in one file
- Duplicate providers for the same data
- Circular dependencies
- Providers that are difficult to locate
- Unclear ownership of business logic

Proper organization keeps the codebase predictable and easier to extend.

---

# Syntax

## Keep providers inside their feature

```text
features/
└── products/
    ├── data/
    ├── presentation/
    ├── providers/
    │   ├── product_provider.dart
    │   ├── product_list_provider.dart
    │   └── cart_provider.dart
    └── widgets/
```

Explanation:

- Providers live alongside the feature they belong to.
- Related code remains together.

---

## Separate shared providers

```text
core/
└── providers/
    ├── api_provider.dart
    ├── dio_provider.dart
    └── shared_preferences_provider.dart
```

Explanation:

- Only application-wide dependencies belong in `core`.
- Feature-specific providers should stay within their respective features.

---

## Create derived providers

```dart
final fullNameProvider = Provider<String>((ref) {
  final user = ref.watch(userProvider);

  return '${user.firstName} ${user.lastName}';
});
```

Explanation:

- Derived providers compute values from existing providers.
- Avoid duplicating state across multiple providers.

---

# Mental Model

Poor organization:

```text
providers/
├── auth.dart
├── user.dart
├── cart.dart
├── orders.dart
├── settings.dart
├── dashboard.dart
├── notifications.dart
└── ...
```

Feature-based organization:

```text
features/
├── auth/
│   └── providers/
├── cart/
│   └── providers/
├── orders/
│   └── providers/
└── settings/
    └── providers/
```

Each feature owns its providers, making responsibilities clear and reducing coupling.

---

# Examples

## Simple Example

```text
features/
└── profile/
    ├── profile_provider.dart
    ├── profile_repository.dart
    └── profile_page.dart
```

Explanation:

- All profile-related code is grouped together.

---

## Real-World Example

```text
lib/
├── core/
│   ├── providers/
│   ├── network/
│   └── services/
│
├── features/
│   ├── authentication/
│   │   └── providers/
│   │
│   ├── products/
│   │   └── providers/
│   │
│   ├── orders/
│   │   └── providers/
│   │
│   └── settings/
│       └── providers/
│
└── shared/
    └── widgets/
```

Explanation:

- Global infrastructure is centralized.
- Business logic remains inside each feature.
- Features can evolve independently.

---

# When to Use

Organize providers carefully when:

- Building medium or large applications.
- Working with multiple developers.
- Creating reusable features.
- Splitting the application into modules.
- Maintaining long-lived projects.

---

# When NOT to Use

A small prototype with only a few providers may not need a complex folder structure.

Start simple, then reorganize as the application grows.

Avoid introducing unnecessary abstraction for small projects.

---

# Best Practices

- Organize providers by feature.
- Keep one responsibility per provider.
- Use derived providers instead of duplicating state.
- Place global dependencies in `core`.
- Keep provider files focused and small.
- Use consistent naming across all providers.
- Document provider dependencies when they become complex.

---

# Common Mistakes

## Putting every provider in one file

Wrong:

```text
providers.dart
```

Explanation:

- The file becomes difficult to navigate.
- Merge conflicts become more frequent in team environments.

Correct:

```text
features/
├── auth/providers/
├── cart/providers/
└── profile/providers/
```

Explanation:

- Providers remain close to the features they support.
- Files stay manageable.

---

## Duplicating state

Wrong:

```dart
final userProvider = Provider<User>(...);

final profileProvider = Provider<User>(...);
```

Explanation:

- Two providers own the same data.
- Keeping them synchronized becomes difficult.

Correct:

```dart
final userProvider = Provider<User>(...);

final profileProvider = Provider<Profile>((ref) {
  final user = ref.watch(userProvider);

  return user.profile;
});
```

Explanation:

- One provider owns the source of truth.
- Other providers derive values from it.

---

## Mixing global and feature-specific providers

Wrong:

```text
core/
└── providers/
    ├── auth_provider.dart
    ├── cart_provider.dart
    ├── order_provider.dart
    └── profile_provider.dart
```

Explanation:

- Feature logic leaks into the application's shared layer.

Correct:

```text
core/
└── providers/
    ├── dio_provider.dart
    └── logger_provider.dart

features/
├── auth/providers/
├── cart/providers/
└── orders/providers/
```

Explanation:

- Shared dependencies stay global.
- Business logic stays within its owning feature.

---

# Related APIs

- Provider
- NotifierProvider
- AsyncNotifierProvider
- Provider Families
- `ref.watch()`
- `ProviderScope`

---

# Summary

Well-organized providers make Riverpod applications easier to navigate, maintain, and scale. Group providers by feature, keep each provider focused on a single responsibility, use derived providers instead of duplicating state, and reserve global providers for shared infrastructure.