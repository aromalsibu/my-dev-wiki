# ValueListenableBuilder

Understand how to listen to changes in ValueNotifier and rebuild the UI accordingly.

---

# What is it?

ValueListenableBuilder is a widget that rebuilds itself when a ValueNotifier's value changes. It provides a simple and efficient way to listen to changes in a single value and update the UI reactively. ValueListenableBuilder is perfect for managing simple reactive state like counters, toggles, text fields, and other single-value states.

---

# Why does it exist?

ValueListenableBuilder exists to:

- Listen to ValueNotifier changes
- Rebuild UI on value changes
- Provide reactive UI updates
- Simplify state management
- Reduce boilerplate code
- Handle single-value state efficiently

---

# Basic ValueListenableBuilder

> **Creating** a basic ValueListenableBuilder.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic ValueListenableBuilder example
class BasicValueListenableBuilderExample extends StatelessWidget {
  const BasicValueListenableBuilderExample({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Create a ValueNotifier with initial value
    final counter = ValueNotifier<int>(0);

    return Scaffold(
      appBar: AppBar(
        title: const Text('ValueListenableBuilder'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 2. ValueListenableBuilder listens to changes
            ValueListenableBuilder<int>(
              // The ValueNotifier to listen to
              valueListenable: counter,
              
              // 3. Builder function called when value changes
              builder: (BuildContext context, int value, Widget? child) {
                return Column(
                  children: [
                    const Text(
                      'Counter:',
                      style: TextStyle(fontSize: 16),
                    ),
                    Text(
                      '$value',
                      style: const TextStyle(
                        fontSize: 48,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    // 4. Child widget (not rebuilt)
                    if (child != null) child,
                  ],
                );
              },
              // 5. Child widget that doesn't rebuild
              child: const Text(
                'This widget does not rebuild!',
                style: TextStyle(fontSize: 12, color: Colors.grey),
              ),
            ),
            const SizedBox(height: 16),
            
            // 6. Control buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () {
                    // Update the value (triggers rebuild)
                    counter.value--;
                  },
                  child: const Text('-'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    // Update the value (triggers rebuild)
                    counter.value++;
                  },
                  child: const Text('+'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    // Reset the value
                    counter.value = 0;
                  },
                  child: const Text('Reset'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- ValueNotifier holds the value
- ValueListenableBuilder listens to changes
- builder rebuilds on value changes
- child is cached and not rebuilt

---

# ValueListenableBuilder with Complex Types

> **Using ValueListenableBuilder** with complex data.

```dart
/// User data model
class User {
  final String name;
  final int age;
  final String email;

  User({
    required this.name,
    required this.age,
    required this.email,
  });

  // Copy with method for immutability
  User copyWith({
    String? name,
    int? age,
    String? email,
  }) {
    return User(
      name: name ?? this.name,
      age: age ?? this.age,
      email: email ?? this.email,
    );
  }
}

/// ValueListenableBuilder with complex types
class ComplexValueListenableBuilder extends StatelessWidget {
  const ComplexValueListenableBuilder({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. ValueNotifier with custom object
    final userNotifier = ValueNotifier<User>(
      const User(
        name: 'John Doe',
        age: 25,
        email: 'john@example.com',
      ),
    );

    return Scaffold(
      appBar: AppBar(
        title: const Text('Complex ValueListenableBuilder'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. Display user info with ValueListenableBuilder
            ValueListenableBuilder<User>(
              valueListenable: userNotifier,
              builder: (context, user, child) {
                return Card(
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        const Text(
                          'User Information',
                          style: TextStyle(
                            fontWeight: FontWeight.bold,
                            fontSize: 18,
                          ),
                        ),
                        const SizedBox(height: 8),
                        Text('Name: ${user.name}'),
                        Text('Age: ${user.age}'),
                        Text('Email: ${user.email}'),
                      ],
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 16),
            
            // 3. Control buttons
            Row(
              children: [
                // Update name
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      // Update using copyWith (immutable)
                      userNotifier.value = userNotifier.value.copyWith(
                        name: userNotifier.value.name == 'John Doe'
                            ? 'Jane Doe'
                            : 'John Doe',
                      );
                    },
                    child: const Text('Toggle Name'),
                  ),
                ),
                const SizedBox(width: 8),
                // Update age
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      userNotifier.value = userNotifier.value.copyWith(
                        age: userNotifier.value.age + 1,
                      );
                    },
                    child: const Text('Increase Age'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 8),
            // Reset user
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: () {
                  userNotifier.value = const User(
                    name: 'John Doe',
                    age: 25,
                    email: 'john@example.com',
                  );
                },
                child: const Text('Reset User'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- ValueNotifier with custom objects
- Immutable updates with copyWith
- Multiple fields updated independently
- Rebuilds only when value changes

---

# ValueListenableBuilder with Lists

> **Managing lists** with ValueListenableBuilder.

```dart
/// ValueListenableBuilder with lists
class ListValueListenableBuilder extends StatelessWidget {
  const ListValueListenableBuilder({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. ValueNotifier with a list
    final itemsNotifier = ValueNotifier<List<String>>([]);

    return Scaffold(
      appBar: AppBar(
        title: const Text('List ValueListenableBuilder'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. Input field and add button
            Row(
              children: [
                Expanded(
                  child: TextField(
                    decoration: const InputDecoration(
                      hintText: 'Add item...',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (value) {
                      if (value.isNotEmpty) {
                        // 3. Update list (create new list)
                        final newList = List<String>.from(itemsNotifier.value)
                          ..add(value);
                        itemsNotifier.value = newList;
                      }
                    },
                  ),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    // Add a sample item
                    final newList = List<String>.from(itemsNotifier.value)
                      ..add('Item ${itemsNotifier.value.length + 1}');
                    itemsNotifier.value = newList;
                  },
                  child: const Text('Add'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            
            // 4. Display list with ValueListenableBuilder
            Expanded(
              child: ValueListenableBuilder<List<String>>(
                valueListenable: itemsNotifier,
                builder: (context, items, child) {
                  if (items.isEmpty) {
                    return const Center(
                      child: Text(
                        'No items yet',
                        style: TextStyle(color: Colors.grey),
                      ),
                    );
                  }
                  
                  return ListView.builder(
                    itemCount: items.length,
                    itemBuilder: (context, index) {
                      final item = items[index];
                      return ListTile(
                        title: Text(item),
                        trailing: IconButton(
                          icon: const Icon(Icons.delete, color: Colors.red),
                          onPressed: () {
                            // 5. Remove item (create new list without it)
                            final newList = List<String>.from(items)
                              ..removeAt(index);
                            itemsNotifier.value = newList;
                          },
                        ),
                      );
                    },
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- ValueNotifier with List
- Immutable list updates
- Add and remove items
- Automatic UI updates

---

# ValueListenableBuilder vs setState

> **Comparing** ValueListenableBuilder with setState.

```dart
/// Comparison of ValueListenableBuilder and setState
class ComparisonExample extends StatelessWidget {
  const ComparisonExample({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Comparison'),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'setState'),
              Tab(text: 'ValueListenableBuilder'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            SetStateExample(),
            ValueListenableBuilderExample(),
          ],
        ),
      ),
    );
  }
}

/// 1. setState approach (rebuilds everything)
class SetStateExample extends StatefulWidget {
  const SetStateExample({super.key});

  @override
  State<SetStateExample> createState() => _SetStateExampleState();
}

class _SetStateExampleState extends State<SetStateExample> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    // Everything rebuilds when setState is called
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Text(
            'setState Example',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          Text(
            'Counter: $_counter',
            style: const TextStyle(fontSize: 24),
          ),
          // This text rebuilds even though it doesn't need to
          const Text(
            'This text rebuilds on every change',
            style: TextStyle(fontSize: 12, color: Colors.grey),
          ),
          ElevatedButton(
            onPressed: () {
              setState(() {
                _counter++;
              });
            },
            child: const Text('Increment'),
          ),
        ],
      ),
    );
  }
}

/// 2. ValueListenableBuilder approach (only rebuilds what's needed)
class ValueListenableBuilderExample extends StatelessWidget {
  const ValueListenableBuilderExample({super.key});

  @override
  Widget build(BuildContext context) {
    final counter = ValueNotifier<int>(0);

    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Text(
            'ValueListenableBuilder Example',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          // 1. Only this part rebuilds
          ValueListenableBuilder<int>(
            valueListenable: counter,
            builder: (context, value, child) {
              return Text(
                'Counter: $value',
                style: const TextStyle(fontSize: 24),
              );
            },
          ),
          // 2. This text does NOT rebuild
          const Text(
            'This text does NOT rebuild',
            style: TextStyle(fontSize: 12, color: Colors.green),
          ),
          // 3. Child widget doesn't rebuild
          ValueListenableBuilder<int>(
            valueListenable: counter,
            builder: (context, value, child) {
              return Column(
                children: [
                  if (child != null) child,
                  ElevatedButton(
                    onPressed: () {
                      counter.value++;
                    },
                    child: const Text('Increment'),
                  ),
                ],
              );
            },
            child: const Text(
              'Cached widget',
              style: TextStyle(color: Colors.blue),
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- setState: Rebuilds everything
- ValueListenableBuilder: Rebuilds only listening widgets
- ValueListenableBuilder is more efficient
- Child parameter optimizes performance

---

# Real-World Examples

> **Common patterns** with ValueListenableBuilder.

```dart
/// 1. Theme toggle with ValueListenableBuilder
class ThemeToggleExample extends StatelessWidget {
  const ThemeToggleExample({super.key});

  @override
  Widget build(BuildContext context) {
    final isDarkMode = ValueNotifier<bool>(false);

    return MaterialApp(
      theme: isDarkMode.value ? ThemeData.dark() : ThemeData.light(),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Theme Toggle'),
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ValueListenableBuilder<bool>(
                valueListenable: isDarkMode,
                builder: (context, isDark, child) {
                  return Text(
                    isDark ? 'Dark Mode' : 'Light Mode',
                    style: const TextStyle(fontSize: 24),
                  );
                },
              ),
              const SizedBox(height: 16),
              Switch(
                value: isDarkMode.value,
                onChanged: (value) {
                  isDarkMode.value = value;
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}

/// 2. Form validation with ValueListenableBuilder
class FormValidationExample extends StatelessWidget {
  const FormValidationExample({super.key});

  @override
  Widget build(BuildContext context) {
    final email = ValueNotifier<String>('');
    final password = ValueNotifier<String>('');
    final isValid = ValueNotifier<bool>(false);

    // 1. Validate form when fields change
    void validateForm() {
      isValid.value = email.value.isNotEmpty &&
          email.value.contains('@') &&
          password.value.length >= 6;
    }

    // 2. Add listeners to validate
    email.addListener(validateForm);
    password.addListener(validateForm);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Form Validation'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Email field
            ValueListenableBuilder<String>(
              valueListenable: email,
              builder: (context, value, child) {
                return TextField(
                  onChanged: (value) => email.value = value,
                  decoration: InputDecoration(
                    labelText: 'Email',
                    border: const OutlineInputBorder(),
                    errorText: value.isNotEmpty && !value.contains('@')
                        ? 'Invalid email'
                        : null,
                  ),
                );
              },
            ),
            const SizedBox(height: 16),
            
            // Password field
            ValueListenableBuilder<String>(
              valueListenable: password,
              builder: (context, value, child) {
                return TextField(
                  onChanged: (value) => password.value = value,
                  obscureText: true,
                  decoration: InputDecoration(
                    labelText: 'Password',
                    border: const OutlineInputBorder(),
                    errorText: value.isNotEmpty && value.length < 6
                        ? 'Min 6 characters'
                        : null,
                  ),
                );
              },
            ),
            const SizedBox(height: 16),
            
            // Submit button (enabled based on validation)
            ValueListenableBuilder<bool>(
              valueListenable: isValid,
              builder: (context, valid, child) {
                return SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: valid
                        ? () {
                            ScaffoldMessenger.of(context).showSnackBar(
                              const SnackBar(
                                content: Text('Form submitted!'),
                                backgroundColor: Colors.green,
                              ),
                            );
                          }
                        : null,
                    child: const Text('Submit'),
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Theme toggle with ValueNotifier
- Form validation with multiple notifiers
- Real-time validation feedback
- Submit button enabled state

---

# Best Practices

## Use Child Parameter for Performance

```dart
// Good - Child is cached
ValueListenableBuilder<int>(
  valueListenable: counter,
  child: const Text('Static text'),
  builder: (context, value, child) {
    return Column(
      children: [
        Text('Dynamic: $value'),
        if (child != null) child,
      ],
    );
  },
)
```

## Use Immutable Updates

```dart
// Good - Immutable update
final newList = List<String>.from(items.value)..add('new item');
items.value = newList;

// Bad - Mutating the list
items.value.add('new item'); // Won't trigger rebuild
```

## Dispose ValueNotifiers

```dart
// Good - Dispose in StatefulWidget
@override
void dispose() {
  _counter.dispose();
  super.dispose();
}
```

---

# Common Mistakes

## Mutating Values Directly

Wrong:
```dart
// Won't trigger rebuild
items.value.add('new item');
```

Correct:
```dart
// Assign a new value
final newList = List<String>.from(items.value)..add('new item');
items.value = newList;
```

## Forgetting to Dispose

Wrong:
```dart
// Memory leak
final counter = ValueNotifier<int>(0);
// No dispose
```

Correct:
```dart
// Dispose properly
@override
void dispose() {
  counter.dispose();
  super.dispose();
}
```

---

# Summary

ValueListenableBuilder provides efficient reactive UI updates for single-value state. Use ValueNotifier for state management, ValueListenableBuilder for listening to changes, and the child parameter for performance optimization. ValueListenableBuilder is simpler and more efficient than setState for single values.

---

# Next Steps

- [AnimatedBuilder](animatedbuilder.md)
- [Async Patterns](async-patterns.md)
- [Provider](../state-management/provider.md)

---

# Did You Know?

- ValueListenableBuilder listens to ValueNotifier
- Only the builder rebuilds, not the whole widget
- child parameter caches static widgets
- ValueNotifier can hold any type
- Multiple listeners can subscribe
- ValueNotifier must be disposed
- Immutable updates are required
- ValueListenableBuilder is more efficient than setState