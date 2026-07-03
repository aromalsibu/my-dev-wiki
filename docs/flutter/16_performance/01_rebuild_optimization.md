# Rebuild Optimization

Understand how to optimize rebuilds in Flutter for better performance.

---

# What is it?

Rebuild Optimization is the practice of minimizing unnecessary widget rebuilds in Flutter applications. When state changes, Flutter rebuilds widgets to reflect the new state. However, rebuilding too much can cause performance issues, especially in large applications. Optimizing rebuilds ensures your app runs smoothly and efficiently.

---

# Why does it exist?

Rebuild Optimization exists to:

- Improve application performance
- Reduce unnecessary widget rebuilds
- Minimize CPU and GPU usage
- Prevent jank and frame drops
- Optimize memory usage
- Enhance user experience
- Support complex UI structures

---

# Understanding Rebuilds

> **When and why** widgets rebuild.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Understanding rebuilds example
class RebuildUnderstanding extends StatefulWidget {
  const RebuildUnderstanding({super.key});

  @override
  State<RebuildUnderstanding> createState() => _RebuildUnderstandingState();
}

class _RebuildUnderstandingState extends State<RebuildUnderstanding> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('🔵 Parent rebuilt');
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Understanding Rebuilds'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. This widget rebuilds every time (bad)
            const Text(
              'This widget rebuilds every time',
              style: TextStyle(color: Colors.grey, fontSize: 12),
            ),
            
            // 2. This widget rebuilds because of setState
            Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 24),
            ),
            
            const SizedBox(height: 16),
            
            // 3. Child widgets also rebuild
            _buildChildWidgets(),
            
            const SizedBox(height: 16),
            
            // 4. Button triggers rebuild
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _counter++;
                });
                print('🟢 Button pressed - rebuild triggered');
              },
              child: const Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildChildWidgets() {
    print('🟡 Building child widgets');
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Container(
          padding: const EdgeInsets.all(8),
          color: Colors.blue[100],
          child: const Text('Child 1'),
        ),
        const SizedBox(width: 8),
        Container(
          padding: const EdgeInsets.all(8),
          color: Colors.green[100],
          child: const Text('Child 2'),
        ),
      ],
    );
  }
}

/// What triggers rebuilds:
/// 1. setState() called
/// 2. Parent widget rebuilds
/// 3. Inherited widget changes
/// 4. Animation changes
/// 5. Hot reload
/// 6. Device orientation change
```

What's happening here?
- setState triggers rebuilds
- Parent rebuilds cause child rebuilds
- All widgets in the subtree rebuild
- Unnecessary rebuilds affect performance

---

# const Widgets

> **Using const** to prevent rebuilds.

```dart
/// Const widgets optimization
class ConstWidgetOptimization extends StatefulWidget {
  const ConstWidgetOptimization({super.key});

  @override
  State<ConstWidgetOptimization> createState() => _ConstWidgetOptimizationState();
}

class _ConstWidgetOptimizationState extends State<ConstWidgetOptimization> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('🔵 Parent rebuilt');
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Const Optimization'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Non-const widget - rebuilds every time
            print('🟡 Building non-const widget');
            const Text('Non-const Widget', style: TextStyle(fontSize: 16)),
            
            // 2. Const widget - does NOT rebuild
            const Text(
              'Const Widget (Does NOT rebuild)',
              style: TextStyle(fontSize: 16, color: Colors.green),
            ),
            
            const SizedBox(height: 16),
            
            // 3. Non-const container
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.blue[100],
              child: const Text('Non-const Container'),
            ),
            
            const SizedBox(height: 8),
            
            // 4. Const container
            const SizedBox.shrink(),
            
            const SizedBox(height: 16),
            
            Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 24),
            ),
            
            const SizedBox(height: 16),
            
            // 5. Const widget with parameters (can't be const)
            // This still rebuilds
            CustomNonConstWidget(counter: _counter),
            
            const SizedBox(height: 8),
            
            // 6. Const widget
            const CustomConstWidget(),
            
            const SizedBox(height: 16),
            
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
      ),
    );
  }
}

/// Custom non-const widget
class CustomNonConstWidget extends StatelessWidget {
  const CustomNonConstWidget({super.key, required this.counter});

  final int counter;

  @override
  Widget build(BuildContext context) {
    print('🟠 CustomNonConstWidget rebuilt');
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.orange[100],
      child: Text('Counter: $counter (non-const)'),
    );
  }
}

/// Custom const widget
class CustomConstWidget extends StatelessWidget {
  const CustomConstWidget({super.key});

  @override
  Widget build(BuildContext context) {
    print('🟢 CustomConstWidget built (only once)');
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.green[100],
      child: const Text('Const Widget (cached)'),
    );
  }
}
```

What's happening here?
- const widgets are built once and cached
- Non-const widgets rebuild with parent
- const widgets improve performance
- Use const whenever possible

---

# Widget Splitting

> **Splitting widgets** to isolate rebuilds.

```dart
/// Widget splitting optimization
class WidgetSplittingExample extends StatefulWidget {
  const WidgetSplittingExample({super.key});

  @override
  State<WidgetSplittingExample> createState() => _WidgetSplittingExampleState();
}

class _WidgetSplittingExampleState extends State<WidgetSplittingExample> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('🟢 Parent build');
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Widget Splitting'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Bad: Everything in one widget
            // Everything rebuilds on counter change
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.red[100],
              child: Column(
                children: [
                  const Text(
                    'Bad: Everything rebuilds',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  Text('Counter: $_counter'),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 2. Good: Split into smaller widgets
            const Text(
              'Good: Only counter rebuilds',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            
            // Static widgets don't rebuild
            const _StaticWidget(),
            const _StaticWidget2(),
            
            // Dynamic widget that rebuilds
            _CounterDisplay(counter: _counter),
            
            const _StaticWidget3(),
            
            const SizedBox(height: 16),
            
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
      ),
    );
  }
}

/// Static widgets (do not rebuild)
class _StaticWidget extends StatelessWidget {
  const _StaticWidget();

  @override
  Widget build(BuildContext context) {
    print('🟡 _StaticWidget built (cached)');
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.green[100],
      child: const Text('Static Widget 1'),
    );
  }
}

class _StaticWidget2 extends StatelessWidget {
  const _StaticWidget2();

  @override
  Widget build(BuildContext context) {
    print('🟡 _StaticWidget2 built (cached)');
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.blue[100],
      child: const Text('Static Widget 2'),
    );
  }
}

class _StaticWidget3 extends StatelessWidget {
  const _StaticWidget3();

  @override
  Widget build(BuildContext context) {
    print('🟡 _StaticWidget3 built (cached)');
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.purple[100],
      child: const Text('Static Widget 3'),
    );
  }
}

/// Dynamic widget (rebuilds)
class _CounterDisplay extends StatelessWidget {
  const _CounterDisplay({required this.counter});

  final int counter;

  @override
  Widget build(BuildContext context) {
    print('🔴 _CounterDisplay rebuilt');
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.orange[100],
      child: Text('Counter: $counter'),
    );
  }
}
```

What's happening here?
- Split widgets into smaller pieces
- Static widgets don't rebuild
- Only dynamic widgets rebuild
- Better performance

---

# Keys Optimization

> **Using keys** to optimize rebuilds.

```dart
/// Keys optimization example
class KeysOptimizationExample extends StatefulWidget {
  const KeysOptimizationExample({super.key});

  @override
  State<KeysOptimizationExample> createState() => _KeysOptimizationExampleState();
}

class _KeysOptimizationExampleState extends State<KeysOptimizationExample> {
  List<String> _items = ['Item 1', 'Item 2', 'Item 3'];

  void _addItem() {
    setState(() {
      _items.insert(1, 'Item ${_items.length + 1}');
    });
  }

  void _removeItem() {
    setState(() {
      if (_items.isNotEmpty) {
        _items.removeAt(1);
      }
    });
  }

  void _shuffleItems() {
    setState(() {
      _items.shuffle();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Keys Optimization'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Control buttons
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _addItem,
                    child: const Text('Add'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _removeItem,
                    child: const Text('Remove'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _shuffleItems,
                    child: const Text('Shuffle'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 16),
            
            const Text(
              'Without Keys (Inefficient)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 4),
            _buildListWithoutKeys(),
            
            const SizedBox(height: 16),
            
            const Text(
              'With Keys (Efficient)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 4),
            _buildListWithKeys(),
          ],
        ),
      ),
    );
  }

  Widget _buildListWithoutKeys() {
    return Container(
      height: 150,
      color: Colors.grey[100],
      child: ListView.builder(
        itemCount: _items.length,
        itemBuilder: (context, index) {
          // No key - Flutter uses index
          print('Building without key: ${_items[index]}');
          return Container(
            padding: const EdgeInsets.all(8),
            margin: const EdgeInsets.all(2),
            color: Colors.blue[100 * (index % 9 + 1)],
            child: Text(_items[index]),
          );
        },
      ),
    );
  }

  Widget _buildListWithKeys() {
    return Container(
      height: 150,
      color: Colors.grey[100],
      child: ListView.builder(
        itemCount: _items.length,
        itemBuilder: (context, index) {
          final item = _items[index];
          // Using ValueKey for stable identity
          print('Building with key: $item');
          return Container(
            key: ValueKey(item), // Unique key
            padding: const EdgeInsets.all(8),
            margin: const EdgeInsets.all(2),
            color: Colors.green[100 * (index % 9 + 1)],
            child: Text(item),
          );
        },
      ),
    );
  }
}
```

What's happening here?
- Keys help Flutter identify widgets
- Without keys, all widgets rebuild
- With keys, only changed widgets rebuild
- Better performance for dynamic lists

---

# Real-World Examples

> **Common optimization** patterns.

```dart
/// 1. Optimized form with ValueListenableBuilder
class OptimizedForm extends StatelessWidget {
  const OptimizedForm({super.key});

  @override
  Widget build(BuildContext context) {
    final name = ValueNotifier<String>('');
    final email = ValueNotifier<String>('');

    return Scaffold(
      appBar: AppBar(
        title: const Text('Optimized Form'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Only these widgets rebuild
            ValueListenableBuilder<String>(
              valueListenable: name,
              builder: (context, value, child) {
                return TextField(
                  onChanged: (value) => name.value = value,
                  decoration: InputDecoration(
                    labelText: 'Name',
                    border: const OutlineInputBorder(),
                    errorText: value.isEmpty ? 'Required' : null,
                  ),
                );
              },
            ),
            const SizedBox(height: 8),
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
            
            // Submit button (rebuilds on validation changes)
            ValueListenableBuilder<bool>(
              valueListenable: ValueNotifier<bool>(
                name.value.isNotEmpty && email.value.contains('@'),
              ),
              builder: (context, isValid, child) {
                // Re-evaluate validation
                final valid = name.value.isNotEmpty && email.value.contains('@');
                return SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: valid
                        ? () {
                            ScaffoldMessenger.of(context).showSnackBar(
                              const SnackBar(
                                content: Text('Form submitted!'),
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

/// 2. Optimized list with const items
class OptimizedList extends StatefulWidget {
  const OptimizedList({super.key});

  @override
  State<OptimizedList> createState() => _OptimizedListState();
}

class _OptimizedListState extends State<OptimizedList> {
  final List<String> _items = List.generate(100, (index) => 'Item $index');

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Optimized List'),
      ),
      body: ListView.builder(
        itemCount: _items.length,
        itemBuilder: (context, index) {
          final item = _items[index];
          // Use const widgets where possible
          return _OptimizedListItem(
            key: ValueKey(item),
            title: item,
          );
        },
      ),
    );
  }
}

/// Optimized list item with const constructor
class _OptimizedListItem extends StatelessWidget {
  const _OptimizedListItem({
    super.key,
    required this.title,
  });

  final String title;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(title),
      leading: const Icon(Icons.star),
      trailing: const Icon(Icons.arrow_forward),
    );
  }
}
```

What's happening here?
- ValueListenableBuilder for efficient form
- Const widgets for static parts
- ValueKey for list items
- Optimized list with const constructors

---

# Best Practices

## Use const Whenever Possible

```dart
// Good - const widget
const Text('Static Text')

// Bad - Non-const
Text('Static Text')
```

## Split Large Widgets

```dart
// Good - Split into small widgets
class ParentWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const StaticHeader(),
        DynamicContent(),
        const StaticFooter(),
      ],
    );
  }
}
```

## Use Keys for Lists

```dart
// Good - Keys for dynamic lists
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(
      key: ValueKey(items[index].id),
      title: Text(items[index].name),
    );
  },
)
```

---

# Common Mistakes

## Not Using const

Wrong:
```dart
// Rebuilds every time
Text('Hello World')
```

Correct:
```dart
// Cached, doesn't rebuild
const Text('Hello World')
```

## Not Splitting Widgets

Wrong:
```dart
// Everything rebuilds
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      StaticHeader(),
      DynamicContent(), // Rebuilds
      StaticFooter(), // Also rebuilds
    ],
  );
}
```

Correct:
```dart
// Only dynamic part rebuilds
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      const StaticHeader(),
      DynamicContent(), // Only this rebuilds
      const StaticFooter(),
    ],
  );
}
```

---

# Summary

Rebuild Optimization improves performance by minimizing unnecessary widget rebuilds. Use const widgets, split large widgets, use keys for lists, and leverage ValueListenableBuilder. Optimize rebuilds to create smooth, performant applications.

---

# Next Steps

- [RepaintBoundary](repaintboundary.md)
- [Image Optimization](image-optimization.md)
- [DevTools](devtools.md)

---

# Did You Know?

- const widgets are built once and cached
- Widget splitting isolates rebuilds
- Keys help Flutter identify widgets
- ValueListenableBuilder limits rebuilds
- SetState rebuilds the entire widget subtree
- Performance improves with fewer rebuilds
- Flutter DevTools shows rebuilds
- Optimization prevents jank