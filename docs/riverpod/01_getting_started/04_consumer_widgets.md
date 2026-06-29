# Consumer Widgets

Consumer widgets are Flutter widgets that can read and react to Riverpod providers.

---

## What is it?

A Consumer Widget is a widget that has access to Riverpod providers through a `WidgetRef`.

Unlike a normal Flutter widget, a consumer widget can:

- Read providers
- Watch providers for changes
- Listen to provider updates
- Interact with Riverpod's state management system

Riverpod provides three consumer widgets:

- `Consumer`
- `ConsumerWidget`
- `ConsumerStatefulWidget`

These widgets bridge the gap between the **UI** and the **provider graph**.

---

## Why does it exist?

A normal Flutter widget has no knowledge of Riverpod.

For example:

```dart
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Cannot access providers here
  }
}
```

There is no `ref`, so you cannot read or watch providers.

Consumer widgets solve this by injecting a `WidgetRef`, allowing widgets to interact with Riverpod.

Benefits include:

- Automatic UI updates
- Simple provider access
- Fine-grained rebuilds
- Separation of UI and business logic

---

## Syntax

### ConsumerWidget

The most commonly used consumer widget.

```dart
class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final greeting = ref.watch(greetingProvider);

    return Text(greeting);
  }
}
```

Explanation:

- `ConsumerWidget` is a stateless widget with access to `WidgetRef`.
- `ref.watch()` subscribes to the provider.
- The widget rebuilds whenever the provider changes.

---

### Consumer

Useful when only a small portion of the widget tree needs provider access.

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text('Static Text'),

        Consumer(
          builder: (context, ref, child) {
            final greeting = ref.watch(greetingProvider);

            return Text(greeting);
          },
        ),
      ],
    );
  }
}
```

Explanation:

- `Consumer` provides a local `WidgetRef`.
- Only the `Consumer` widget rebuilds.
- The rest of the widget tree remains unchanged.

---

### ConsumerStatefulWidget

Use when you need both Riverpod and StatefulWidget features.

```dart
class HomePage extends ConsumerStatefulWidget {
  const HomePage({super.key});

  @override
  ConsumerState<HomePage> createState() => _HomePageState();
}

class _HomePageState extends ConsumerState<HomePage> {
  @override
  Widget build(BuildContext context) {
    final greeting = ref.watch(greetingProvider);

    return Text(greeting);
  }
}
```

Explanation:

- `ConsumerStatefulWidget` combines `StatefulWidget` with Riverpod.
- `ref` is available directly inside `ConsumerState`.
- Useful when using controllers, animations, or lifecycle methods.

---

## Mental Model

Think of a consumer widget as a **subscriber**.

```text
          Provider
              │
      Value changes
              │
              ▼
      Consumer Widget
              │
        Rebuild UI
```

A consumer widget watches providers.

Whenever a watched provider changes, Riverpod rebuilds only the affected consumer.

---

## Examples

### Displaying a Value

```dart
final appNameProvider = Provider((ref) => 'Riverpod App');

class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Text(ref.watch(appNameProvider));
  }
}
```

Explanation:

- The widget watches `appNameProvider`.
- The displayed text updates automatically if the provider changes.

---

### Watching Multiple Providers

```dart
final firstNameProvider = Provider((ref) => 'John');
final lastNameProvider = Provider((ref) => 'Doe');

class UserName extends ConsumerWidget {
  const UserName({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final firstName = ref.watch(firstNameProvider);
    final lastName = ref.watch(lastNameProvider);

    return Text('$firstName $lastName');
  }
}
```

Explanation:

- The widget subscribes to two providers.
- It rebuilds when either provider changes.

---

### Rebuilding Only Part of the UI

```dart
Column(
  children: [
    const Header(),

    Consumer(
      builder: (context, ref, child) {
        final count = ref.watch(counterProvider);

        return Text('$count');
      },
    ),

    const Footer(),
  ],
)
```

Explanation:

- Only the `Consumer` rebuilds.
- `Header` and `Footer` remain untouched.

---

## When to Use

### Use `ConsumerWidget` when:

- The widget is stateless.
- The entire widget depends on providers.
- Most Riverpod UI falls into this category.

---

### Use `Consumer` when:

- Only a small part of the UI needs provider access.
- You want to reduce rebuilds.
- You're inside an existing widget that shouldn't become a `ConsumerWidget`.

---

### Use `ConsumerStatefulWidget` when:

- You need `initState()`
- You need `dispose()`
- You use `AnimationController`
- You use `TextEditingController`
- You combine local state with Riverpod state

---

## When NOT to Use

Do not use consumer widgets when:

- The widget never accesses providers.
- The widget is purely presentational.
- No reactive state is required.

Use:

- `StatelessWidget`
- `StatefulWidget`

instead.

---

## Best Practices

- Prefer `ConsumerWidget` whenever possible.
- Use `Consumer` to rebuild only small UI sections.
- Use `ConsumerStatefulWidget` only when local widget state is required.
- Keep business logic inside providers, not widgets.
- Watch only the providers needed by the widget.

---

## Common Mistakes

### 1. Using StatelessWidget with `ref`

❌ Wrong

```dart
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    ref.watch(counterProvider);
  }
}
```

Why it's wrong:

- `StatelessWidget` has no `WidgetRef`.

✔ Correct

```dart
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Text('$count');
  }
}
```

---

### 2. Making Every Widget a ConsumerWidget

❌ Wrong

```text
App
 ├── ConsumerWidget
 ├── ConsumerWidget
 ├── ConsumerWidget
 ├── ConsumerWidget
```

Why it's wrong:

- Many widgets may not need provider access.
- Increases unnecessary complexity.

✔ Correct

Only widgets that use providers should be consumer widgets.

---

### 3. Watching Providers Too High in the Widget Tree

❌ Wrong

```dart
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(),
      body: LargeWidgetTree(count),
    );
  }
}
```

Why it's wrong:

- The entire page rebuilds when `count` changes.

✔ Correct

```dart
Scaffold(
  appBar: AppBar(),
  body: Consumer(
    builder: (context, ref, child) {
      final count = ref.watch(counterProvider);

      return Text('$count');
    },
  ),
)
```

This limits rebuilds to the smallest possible widget subtree.

---

## Related APIs

- Consumer
- ConsumerWidget
- ConsumerStatefulWidget
- ConsumerState
- WidgetRef
- ref.watch()
- ref.read()
- ref.listen()

---

## Summary

Consumer widgets connect Flutter widgets to Riverpod's provider system. They provide access to `WidgetRef`, allowing widgets to read, watch, and listen to providers. Use `ConsumerWidget` for most stateless UI, `Consumer` for localized rebuilds, and `ConsumerStatefulWidget` when widget lifecycle methods or local state are needed.