# Project Structure

Organize providers, features, and shared code into a clear, scalable folder structure.

---

# What is it?

Project structure is the way you organize your application's files and directories.

Riverpod does not enforce any specific folder structure. However, a well-organized project makes it easier to locate providers, understand dependencies, and scale the application as it grows.

A good structure should group related code together and keep features isolated from one another.

---

# Why does it exist?

As applications grow, poor organization leads to:

- Difficult navigation
- Duplicate providers
- Circular dependencies
- Hard-to-maintain code
- Slower onboarding for new developers

A consistent project structure helps teams work more efficiently and keeps the codebase maintainable.

---

# Syntax

## Feature-first structure (Recommended)

```text
lib/
├── core/
│   ├── network/
│   ├── services/
│   ├── providers/
│   └── utils/
│
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   ├── presentation/
│   │   └── providers/
│   │
│   ├── profile/
│   └── cart/
│
└── main.dart
```

Explanation:

- `core/` contains shared functionality used across the application.
- `features/` groups everything related to a specific feature.
- Each feature owns its own providers.

---

## Keeping providers near the feature

```text
features/
└── cart/
    ├── data/
    ├── presentation/
    ├── providers/
    └── cart_notifier.dart
```

Explanation:

- Providers stay close to the code they support.
- Features become self-contained and easier to maintain.

---

# Mental Model

Instead of organizing by file type:

```text
providers/
models/
screens/
repositories/
```

Prefer organizing by feature:

```text
features/
├── auth/
├── cart/
├── profile/
└── settings/
```

Think of each feature as a small application with its own UI, business logic, and providers.

---

# Examples

## Simple Example

```text
features/
└── profile/
    ├── profile_page.dart
    ├── profile_provider.dart
    └── profile_repository.dart
```

Explanation:

- Everything related to the profile feature lives together.

---

## Real-World Example

```text
lib/
├── core/
│   ├── api/
│   ├── database/
│   ├── providers/
│   └── theme/
│
├── features/
│   ├── authentication/
│   ├── dashboard/
│   ├── orders/
│   ├── products/
│   └── settings/
│
└── shared/
    ├── widgets/
    └── extensions/
```

Explanation:

- Shared infrastructure lives in `core/`.
- Business features remain independent.
- Reusable UI components are placed in `shared/`.

---

# When to Use

Use a structured project layout when:

- Building medium or large applications.
- Working in a team.
- Maintaining long-term projects.
- Developing reusable features.
- Creating modular applications.

---

# When NOT to Use

A small prototype or proof of concept may not need a complex folder hierarchy.

For very small applications, a simpler structure can be easier to manage. As the project grows, you can gradually reorganize it into feature-based modules.

---

# Best Practices

- Prefer feature-first organization.
- Keep providers inside their owning feature.
- Separate shared code into `core/` or `shared/`.
- Keep files focused on a single responsibility.
- Avoid deeply nested folder structures.
- Use consistent naming conventions throughout the project.

---

# Common Mistakes

## Organizing only by file type

Wrong:

```text
providers/
models/
screens/
repositories/
services/
```

Explanation:

- Files from the same feature become scattered across the project.
- Navigating the codebase becomes more difficult.

Correct:

```text
features/
├── auth/
├── profile/
├── products/
└── orders/
```

Explanation:

- Each feature contains everything it needs.
- Related code stays together.

---

## Creating a global providers folder

Wrong:

```text
providers/
├── auth_provider.dart
├── cart_provider.dart
├── profile_provider.dart
└── settings_provider.dart
```

Explanation:

- Feature logic becomes centralized.
- Ownership of providers becomes unclear.

Correct:

```text
features/
├── auth/
│   └── providers/
├── cart/
│   └── providers/
└── profile/
    └── providers/
```

Explanation:

- Providers remain close to the feature that owns them.

---

## Deeply nesting folders

Wrong:

```text
features/
└── auth/
    └── src/
        └── internal/
            └── providers/
                └── implementations/
```

Explanation:

- Excessive nesting makes navigation harder.

Correct:

```text
features/
└── auth/
    ├── data/
    ├── presentation/
    └── providers/
```

Explanation:

- Keep the structure shallow and easy to navigate.

---

# Related APIs

- Provider
- NotifierProvider
- AsyncNotifierProvider
- ProviderScope
- ProviderContainer

---

# Summary

A good project structure groups related code by feature instead of by file type. Keeping providers close to the features they belong to improves readability, simplifies maintenance, reduces coupling, and makes Riverpod applications easier to scale.