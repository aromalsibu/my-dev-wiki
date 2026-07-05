# Provider vs GetX

Compare Riverpod and GetX to understand their different philosophies and decide which approach better suits your Flutter application.

---

Riverpod and GetX both simplify Flutter development, but they have very different goals and architectural styles.

---

# What is it?

Both Riverpod and GetX can manage application state, but GetX is a much broader framework.

**Riverpod** focuses on:

- State management
- Dependency injection
- Provider composition

**GetX** provides:

- State management
- Dependency injection
- Navigation
- Route management
- Internationalization
- Utilities

Riverpod follows a modular approach, while GetX aims to provide an all-in-one solution.

---

# Why does it exist?

Different teams have different priorities.

Some teams prefer:

- Explicit dependencies
- Compile-time safety
- Scalable architecture
- Separation of concerns

Others prefer:

- Rapid development
- Minimal code
- Global access to services
- Integrated routing and utilities

Riverpod and GetX are designed around these different priorities.

---

# Syntax

## Riverpod

```dart
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}

final counterProvider =
    NotifierProvider<CounterNotifier, int>(
      CounterNotifier.new,
    );
```

Explanation:

- Business logic lives in a notifier.
- Widgets interact using `ref.watch()` and `ref.read()`.

---

## GetX

```dart
class CounterController extends GetxController {
  final count = 0.obs;

  void increment() {
    count.value++;
  }
}
```

Explanation:

- `.obs` creates an observable value.
- Widgets react to changes using GetX widgets such as `Obx`.

---

# Mental Model

Riverpod:

```text
Widget
   │
   ▼
Provider
   │
   ▼
Notifier
   │
   ▼
State
```

GetX:

```text
Widget
   │
   ▼
Controller
   │
   ▼
Observable (.obs)
```

Riverpod revolves around providers, while GetX revolves around controllers and observable values.

---

# Examples

## Simple Example

### Riverpod

```dart
final count = ref.watch(counterProvider);
```

Explanation:

- Widgets subscribe to providers.

---

### GetX

```dart
Obx(() {
  return Text('${controller.count.value}');
});
```

Explanation:

- `Obx` rebuilds when the observable value changes.

---

## Real-World Example

### Riverpod

```dart
class LoginNotifier
    extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async => null;

  Future<void> login(
    String email,
    String password,
  ) async {
    state = await AsyncValue.guard(() async {
      return repository.login(email, password);
    });
  }
}
```

Explanation:

- Business logic stays inside the notifier.
- Asynchronous state is represented by `AsyncValue`.

---

### GetX

```text
Login Page
      │
      ▼
LoginController
      │
      ▼
Rx<User?>
```

Explanation:

- Controllers expose observable values.
- Widgets react to those observables.

---

# Feature Comparison

| Feature | Riverpod | GetX |
|----------|----------|------|
| State management | ✅ | ✅ |
| Dependency injection | ✅ | ✅ |
| Navigation | ❌ External package | ✅ Built in |
| Route management | ❌ | ✅ |
| Internationalization | ❌ | ✅ |
| Compile-time safety | ✅ Strong | ✅ Good |
| Global service locator | ❌ Encourages explicit providers | ✅ Common pattern |
| Boilerplate | Low | Very low |
| Modular architecture | ✅ | ⚠️ Depends on usage |

---

# When to Use

Choose **Riverpod** when you want:

- Scalable architecture
- Explicit dependencies
- Strong separation of concerns
- Feature-based organization
- Easy testing
- Fine-grained provider composition

Choose **GetX** when you want:

- Rapid prototyping
- Built-in navigation
- Integrated dependency injection
- A single package for multiple concerns
- Minimal setup

---

# When NOT to Use

Riverpod may not be ideal if your primary goal is to use a single package for state management, routing, and dependency injection.

GetX may not be the best choice if you want to keep navigation, dependency injection, and state management as independent concerns or if your team prefers explicit dependency graphs.

Ultimately, both libraries can be used successfully in production; the decision depends on your architectural preferences.

---

# Best Practices

### Riverpod

- Keep business logic inside notifiers.
- Compose providers instead of relying on global state.
- Use feature-based organization.
- Inject dependencies through providers.

### GetX

- Keep controllers focused on a single feature.
- Avoid turning controllers into large "god objects."
- Organize routes and services consistently.
- Dispose of resources appropriately.

---

# Common Mistakes

## Using global state everywhere

Wrong:

```text
Global Controller
        │
        ▼
Entire application
```

Explanation:

- Excessive global state increases coupling and makes features harder to isolate.

Correct:

```text
Feature
   │
Provider
   │
Notifier
```

Explanation:

- Scope state to the feature that owns it whenever possible.

---

## Mixing responsibilities

Wrong:

```text
Controller

Navigation
API
Authentication
Settings
Theme
```

Explanation:

- One controller becomes responsible for unrelated features.

Correct:

```text
AuthenticationNotifier

SettingsNotifier

ThemeNotifier
```

Explanation:

- Each class has a single responsibility.

---

## Putting business logic in widgets

Wrong:

```dart
onPressed: () async {
  await repository.login();
}
```

Explanation:

- Widgets become tightly coupled to the data layer.

Correct:

```dart
ref
    .read(authProvider.notifier)
    .login(email, password);
```

Explanation:

- Keep business logic inside providers or controllers, not in the UI.

---

# Related APIs

## Riverpod

- Provider
- Notifier
- AsyncNotifier
- ProviderScope

## GetX

- GetxController
- Obx
- GetBuilder
- Bindings

---

# Summary

Riverpod and GetX both provide effective state management, but they take different approaches. Riverpod focuses on explicit providers, dependency injection, and scalable architecture, while GetX offers an all-in-one solution with state management, routing, and dependency injection in a single package. Choose Riverpod for modular, provider-based applications and GetX when you prefer an integrated framework with minimal setup.