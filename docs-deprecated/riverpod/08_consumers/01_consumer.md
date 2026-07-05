# Consumer

A widget that rebuilds only a specific part of the widget tree using Riverpod providers.

---

# What is it?

`Consumer` is a Flutter widget provided by Riverpod that gives you access to a `ref` inside the widget tree.

It allows you to read providers **without converting the entire widget into a Riverpod-aware widget**.

Instead, it wraps only a portion of the UI and rebuilds that part when watched providers change.

---

# Why does it exist?

Without `Consumer`, you would need to make the entire widget a `ConsumerWidget` or `ConsumerStatefulWidget`.

That can be unnecessary when only a small part of the UI depends on Riverpod state.

`Consumer` solves this by:

- Limiting rebuild scope
- Avoiding full widget conversion
- Improving performance in large widget trees
- Allowing localized state access inside build methods

---

# Syntax

## Basic Usage

```dart
Consumer(
  builder: (context, ref, child) {
    final count = ref.watch(counterProvider);

    return Text('$count');
  },
);
```

Explanation:

- `builder` gives access to `ref`
- `ref.watch()` listens to provider changes
- Only this widget rebuilds when `counterProvider` changes

---

## With Existing Widget Tree

```dart
Column(
  children: [
    Text('Static Header'),

    Consumer(
      builder: (context, ref, child) {
        final user = ref.watch(userProvider);
        return Text(user.name);
      },
    ),
  ],
);
```

Explanation:

- Only the `Consumer` section rebuilds
- The rest of the widget tree stays unchanged

---

## Using child Optimization

```dart
Consumer(
  builder: (context, ref, child) {
    final count = ref.watch(counterProvider);

    return Column(
      children: [
        Text('$count'),
        child!,
      ],
    );
  },
  child: const Text('Static Footer'),
);
```

Explanation:

- `child` does not rebuild
- Useful for expensive static widgets
- Improves performance by avoiding unnecessary rebuilds

---

# Mental Model

Think of `Consumer` as a **localized subscription window**.

Without Consumer:

```text
Whole widget rebuilds
```

With Consumer:

```text
Only Consumer section rebuilds
```

It isolates reactive behavior inside a subtree.

---

# Examples

## Simple Counter Example

```dart
Consumer(
  builder: (context, ref, _) {
    final count = ref.watch(counterProvider);
    return Text('Count: $count');
  },
);
```

Explanation:

- Displays reactive counter value
- Rebuilds only when counter changes

---

## Real-World Example

```dart
Consumer(
  builder: (context, ref, _) {
    final user = ref.watch(userProvider);

    return ListTile(
      title: Text(user.name),
      subtitle: Text(user.email),
    );
  },
);
```

Explanation:

- Only this list tile updates when user changes
- Efficient for list-heavy UIs

---

## Combining with Buttons

```dart
Consumer(
  builder: (context, ref, _) {
    final isLoading = ref.watch(authProvider).isLoading;

    return ElevatedButton(
      onPressed: isLoading ? null : () {},
      child: Text(isLoading ? 'Loading...' : 'Login'),
    );
  },
);
```

Explanation:

- Button reacts to loading state
- Prevents unnecessary rebuilds of parent widget

---

# When to Use

Use `Consumer` when:

- Only a small part of UI depends on Riverpod
- You want to avoid converting full widget classes
- Working inside large widget trees
- You need quick provider access in a build method

---

# When NOT to Use

Avoid `Consumer` when:

- The entire widget depends on providers → use `ConsumerWidget`
- You are already inside a Riverpod widget class
- Overusing it leads to deeply nested builders
- Code readability becomes difficult due to nesting

---

# Best Practices

- Keep Consumer scope as small as possible
- Use `child` parameter for static widgets
- Prefer `ConsumerWidget` for full-page widgets
- Avoid deeply nested Consumers
- Use for localized state access only

---

# Common Mistakes

## Wrapping Entire Screen in Consumer

**Wrong**

```dart
Consumer(
  builder: (context, ref, _) {
    return Scaffold(
      body: ...
    );
  },
);
```

Unnecessary wrapper for full-page widgets.

**Correct**

Use `ConsumerWidget` instead.

---

## Not Using child Optimization

**Wrong**

```dart
Consumer(
  builder: (context, ref, _) {
    return Column(
      children: [
        Text('Static Header'),
        HeavyWidget(),
      ],
    );
  },
);
```

Static widgets rebuild unnecessarily.

**Correct**

```dart
Consumer(
  builder: (context, ref, child) {
    return Column(
      children: [
        child!,
        HeavyWidget(),
      ],
    );
  },
  child: Text('Static Header'),
);
```

---

## Overusing Consumer

**Wrong**

Wrapping every widget in Consumer.

**Correct**

Only use it where provider access is needed.

---

# Related APIs

- `ConsumerWidget`
- `ConsumerStatefulWidget`
- `WidgetRef`
- `ref.watch()`
- `Provider`
- `AsyncValue`

---

# Summary

`Consumer` is a lightweight Riverpod widget that allows selective rebuilding of UI based on provider changes. It provides scoped access to `ref`, making it useful for optimizing large widget trees while keeping Riverpod integration localized and efficient.