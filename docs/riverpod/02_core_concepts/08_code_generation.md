# Code Generation

Code generation is a Riverpod feature that automatically generates provider boilerplate using annotations, resulting in cleaner, safer, and more maintainable code.

---

## What is it?

Traditionally, creating providers requires manually declaring them.

Example:

```dart
final counterProvider = NotifierProvider<Counter, int>(
  Counter.new,
);

class Counter extends Notifier<int> {
  @override
  int build() => 0;
}
```

With code generation, you only write the business logic.

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
}
```

Riverpod generates the provider automatically.

Instead of writing repetitive boilerplate, you focus on implementing the provider's behavior.

---

## Why does it exist?

As applications grow, manually creating providers becomes repetitive.

Without code generation, every provider requires:

- Declaring the provider variable
- Choosing the correct provider type
- Specifying generic types
- Keeping provider names in sync
- Writing additional boilerplate

Code generation solves these problems by:

- Reducing boilerplate
- Improving type safety
- Eliminating repetitive code
- Providing compile-time validation
- Supporting automatic provider naming
- Simplifying provider families

---

## Syntax

### Step 1: Add Dependencies

```yaml
dependencies:
  flutter_riverpod: ^2.x.x
  riverpod_annotation: ^2.x.x

dev_dependencies:
  riverpod_generator: ^2.x.x
  build_runner: ^2.x.x
```

Explanation:

- `riverpod_annotation` provides annotations such as `@riverpod`.
- `riverpod_generator` generates provider code.
- `build_runner` executes the generator.

---

### Step 2: Add a Part File

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';
```

Explanation:

- The `part` directive links the generated file.
- The `.g.dart` file is created automatically.

---

### Step 3: Annotate a Provider

```dart
@riverpod
String appName(AppNameRef ref) {
  return 'Riverpod';
}
```

Explanation:

- `@riverpod` marks the function for code generation.
- Riverpod generates the corresponding provider.
- `AppNameRef` is also generated.

---

### Step 4: Generate the Code

```bash
dart run build_runner build
```

Explanation:

- Scans your project for annotations.
- Generates all `.g.dart` files.

For continuous generation during development:

```bash
dart run build_runner watch
```

Explanation:

- Watches for file changes.
- Automatically regenerates code when needed.

---

## Mental Model

Without code generation:

```text
Write Provider
      │
Write Boilerplate
      │
Write Types
      │
Maintain Everything
```

With code generation:

```text
Write Logic
      │
      ▼
@riverpod
      │
      ▼
Generator
      │
      ▼
Creates Provider
```

You write the logic.

The generator writes the boilerplate.

---

## Generated Provider Names

Suppose you write:

```dart
@riverpod
String greeting(GreetingRef ref) {
  return 'Hello';
}
```

Riverpod generates:

```dart
final greetingProvider = Provider<String>(...);
```

You use it like any manually created provider.

```dart
ref.watch(greetingProvider);
```

---

## Examples

### Function Provider

```dart
@riverpod
String greeting(GreetingRef ref) {
  return 'Hello Riverpod';
}
```

Explanation:

- Generates a `Provider<String>`.
- No manual provider declaration is required.

---

### Notifier

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Explanation:

- Generates a `NotifierProvider`.
- `_$Counter` is generated automatically.
- Only business logic is written manually.

---

### AsyncNotifier

```dart
@riverpod
class User extends _$User {
  @override
  Future<UserModel> build() async {
    return repository.fetchUser();
  }
}
```

Explanation:

- Generates an `AsyncNotifierProvider`.
- Handles asynchronous state automatically.

---

### Provider Family

```dart
@riverpod
String userName(UserNameRef ref, int id) {
  return 'User $id';
}
```

Explanation:

- Generates a provider family automatically.
- No need to manually use the `.family` modifier.

---

## Generated Files

Suppose you have:

```text
lib/providers/user_provider.dart
```

Running the generator creates:

```text
lib/providers/user_provider.g.dart
```

The generated file contains:

- Provider declarations
- Generated reference types
- Hashes
- Internal implementation details

You should **never edit** the generated file manually.

---

## When to Use

Use code generation when:

- Building medium or large applications
- Using `Notifier`
- Using `AsyncNotifier`
- Creating provider families
- Reducing boilerplate
- Improving maintainability

It is the recommended approach for modern Riverpod projects.

---

## When NOT to Use

Manual providers may be sufficient when:

- Learning Riverpod
- Building very small applications
- Creating quick prototypes
- Avoiding build-time tooling

Even then, code generation remains a valid choice.

---

## Best Practices

- Use `@riverpod` for new providers.
- Commit generated `.g.dart` files to version control (unless your team's workflow differs).
- Never edit generated files manually.
- Run `build_runner watch` during development.
- Keep provider logic inside the annotated function or class, not in the generated code.

---

## Common Mistakes

### 1. Forgetting the Part File

❌ Wrong

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

@riverpod
String greeting(GreetingRef ref) {
  return 'Hello';
}
```

Why it's wrong:

- The generator cannot link the generated code.

✔ Correct

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'greeting_provider.g.dart';
```

---

### 2. Forgetting to Run the Generator

❌ Wrong

```dart
@riverpod
String greeting(GreetingRef ref) {
  return 'Hello';
}
```

Why it's wrong:

- The generated provider does not exist yet.
- Compilation fails.

✔ Correct

Run:

```bash
dart run build_runner build
```

or

```bash
dart run build_runner watch
```

---

### 3. Editing Generated Files

❌ Wrong

```dart
// greeting_provider.g.dart

final greetingProvider = ...
// Modified manually
```

Why it's wrong:

- Changes are overwritten the next time code generation runs.

✔ Correct

Only edit the original annotated file.

---

## Related APIs

- @riverpod
- Notifier
- AsyncNotifier
- StreamNotifier
- Provider Families
- build_runner
- riverpod_annotation
- riverpod_generator

---

## Summary

Code generation allows Riverpod to automatically generate providers from annotated functions and classes. By using `@riverpod`, you write only the business logic while Riverpod generates the provider declarations, reference types, and boilerplate. This approach reduces repetitive code, improves type safety, and is the recommended way to build modern Riverpod applications.