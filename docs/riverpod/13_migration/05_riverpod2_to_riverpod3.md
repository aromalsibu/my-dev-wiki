# Riverpod 2 → Riverpod 3

Migrate from Riverpod 2 to Riverpod 3 by adopting the new APIs, improved lifecycle behavior, and modern state management patterns.

---

# What is it?

Riverpod 3 is the next major version of Riverpod. It builds on the foundation of Riverpod 2 while improving performance, consistency, and developer experience.

Most applications can migrate incrementally because Riverpod 3 maintains compatibility with many existing APIs. However, it also encourages replacing older patterns with newer, more idiomatic ones.

---

# Why does it exist?

Riverpod 3 introduces improvements to:

- Provider lifecycle management
- Performance
- Code generation
- Modern notifier APIs
- Developer ergonomics
- API consistency

It also continues the transition away from older APIs such as `StateNotifierProvider` and `ChangeNotifierProvider` in favor of Riverpod's native notifier system.

---

# Migration Checklist

For most applications, migration consists of the following steps:

- Upgrade to the latest Riverpod packages.
- Read the official migration guide.
- Replace deprecated APIs.
- Prefer `Notifier`, `AsyncNotifier`, and `StreamNotifier`.
- Update generated code (if using code generation).
- Run tests after each migration step.
- Migrate feature by feature instead of rewriting the entire application.

---

# Syntax

## Upgrade package versions

```yaml
dependencies:
  flutter_riverpod: latest
  riverpod_annotation: latest

dev_dependencies:
  riverpod_generator: latest
  build_runner: latest
```

Explanation:

- Upgrade all Riverpod packages together.
- Keep generator packages compatible with the runtime package.

---

## Replace StateNotifier

Before:

```dart
final counterProvider =
    StateNotifierProvider<
        CounterNotifier,
        int>(
      (ref) => CounterNotifier(),
    );
```

Explanation:

- Uses the legacy `StateNotifier` API.

After:

```dart
final counterProvider =
    NotifierProvider<
        CounterNotifier,
        int>(
      CounterNotifier.new,
    );
```

Explanation:

- `NotifierProvider` is the preferred API for synchronous state.

---

## Replace FutureProvider when mutations are needed

Before:

```dart
final userProvider =
    FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- Suitable only for read-only asynchronous state.

After:

```dart
final userProvider =
    AsyncNotifierProvider<
        UserNotifier,
        User>(
      UserNotifier.new,
    );
```

Explanation:

- `AsyncNotifier` supports loading, refreshing, and updating state.

---

## Replace ChangeNotifierProvider

Before:

```dart
final settingsProvider =
    ChangeNotifierProvider<SettingsNotifier>(
      (ref) => SettingsNotifier(),
    );
```

Explanation:

- Relies on Flutter's `ChangeNotifier`.

After:

```dart
final settingsProvider =
    NotifierProvider<
        SettingsNotifier,
        Settings>(
      SettingsNotifier.new,
    );
```

Explanation:

- Prefer Riverpod's native notifier APIs for new code.

---

# Mental Model

Riverpod 2:

```text
Legacy APIs
     │
     ▼
StateNotifier
ChangeNotifier
```

Riverpod 3:

```text
Modern APIs
     │
     ▼
Notifier
AsyncNotifier
StreamNotifier
```

The goal is not just to upgrade versions, but to adopt the newer APIs that integrate more closely with Riverpod's architecture.

---

# Examples

## Simple Example

### Before

```dart
class CounterNotifier
    extends StateNotifier<int> {
  CounterNotifier() : super(0);
}
```

Explanation:

- State is initialized in the constructor.

---

### After

```dart
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;
}
```

Explanation:

- `build()` replaces constructor-based initialization.

---

## Real-World Example

### Before

```dart
class UserNotifier
    extends StateNotifier<User?> {
  UserNotifier() : super(null);
}
```

Explanation:

- Uses the legacy notifier pattern.

---

### After

```dart
class UserNotifier
    extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async {
    return repository.currentUser();
  }
}
```

Explanation:

- Initialization, dependencies, and asynchronous loading are handled in one place.

---

# When to Use

Migrate to Riverpod 3 when:

- Starting a new project.
- Maintaining an active application.
- Adopting modern Riverpod APIs.
- Improving testability and maintainability.
- Reducing boilerplate.

---

# When NOT to Use

A complete rewrite is rarely necessary.

For large applications:

- Upgrade package versions first.
- Keep working code running.
- Migrate feature by feature.
- Replace legacy APIs gradually.

Incremental migration is usually safer than a full rewrite.

---

# Best Practices

- Migrate one feature at a time.
- Prefer `Notifier`, `AsyncNotifier`, and `StreamNotifier`.
- Remove deprecated APIs gradually.
- Keep tests passing throughout the migration.
- Update generated code after changing annotated providers.
- Read release notes before upgrading across major versions.

---

# Common Mistakes

## Rewriting the entire project

Wrong:

```text
Delete everything
Start over
```

Explanation:

- Full rewrites are risky and time-consuming.
- They often introduce new bugs.

Correct:

```text
Upgrade packages
↓
Migrate one feature
↓
Run tests
↓
Repeat
```

Explanation:

- Incremental migration reduces risk and keeps the application functional.

---

## Mixing old and new patterns unnecessarily

Wrong:

```text
New Feature → StateNotifier

Another Feature → Notifier
```

Explanation:

- Continuing to build new features with legacy APIs creates inconsistency.

Correct:

```text
Existing Features → StateNotifier (until migrated)

New Features → Notifier
```

Explanation:

- Existing code can remain stable while new development adopts modern APIs.

---

## Ignoring generated code

Wrong:

```text
Modify annotated providers
Run application
```

Explanation:

- Generated files become outdated.

Correct:

```text
Modify providers
↓
Run build_runner
↓
Run application
```

Explanation:

- Keep generated code synchronized with source files.

---

# Related APIs

- Notifier
- AsyncNotifier
- StreamNotifier
- NotifierProvider
- AsyncNotifierProvider
- StreamNotifierProvider
- StateNotifier
- StateNotifierProvider
- ChangeNotifierProvider

---

# Summary

Migrating from Riverpod 2 to Riverpod 3 is primarily about adopting Riverpod's modern APIs rather than rewriting your application. Upgrade your packages, migrate incrementally, replace legacy notifiers with `Notifier`, `AsyncNotifier`, and `StreamNotifier`, and keep your codebase consistent as you transition to the latest version.