# Const Widgets

Understand how to use const widgets to optimize performance in Flutter.

---

# What is it?

Const widgets are widgets that are created with the `const` keyword, which tells Flutter that the widget is constant and will never change. When you use `const`, Flutter can create the widget once and reuse it whenever the same configuration is needed, instead of rebuilding it every time. This significantly improves performance in Flutter applications.

---

# Why does it exist?

Const widgets exist to:

- Improve performance by reducing rebuilds
- Reduce memory usage through widget caching
- Optimize rendering by reusing widgets
- Prevent unnecessary widget creation
- Enable compile-time optimizations
- Improve hot reload performance
- Reduce CPU usage during rebuilds

---

# Basic Const Usage

> **Using const** in widgets.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic const usage example
class BasicConstExample extends StatelessWidget {
  const BasicConstExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Const Widgets'), // const
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Non-const widget - recreated on every rebuild
            // This widget will be rebuilt every time
            Text('Non-const Text'), // No const keyword
            
            // 2. Const widget - created once and cached
            // This widget is created once and reused
            const Text('Const Text'), // With const keyword
            
            const SizedBox(height: 16),
            
            // 3. Const widget with const constructor
            const CustomConstWidget(),
            
            const SizedBox(height: 16),
            
            // 4. Non-const widget with non-const constructor
            CustomNonConstWidget(),
            
            const SizedBox(height: 16),
            
            // 5. Const widget with parameters
            const Text(
              'Const with parameters',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// Custom const widget
class CustomConstWidget extends StatelessWidget {
  const CustomConstWidget({super.key}); // const constructor

  @override
  Widget build(BuildContext context) {
    print('🔵 CustomConstWidget built (only once)');
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.green[100],
      child: const Text(
        'Const Widget',
        style: TextStyle(fontWeight: FontWeight.bold),
      ),
    );
  }
}

/// Custom non-const widget
class CustomNonConstWidget extends StatelessWidget {
  const CustomNonConstWidget({super.key}); // const constructor

  @override
  Widget build(BuildContext context) {
    print('🔴 CustomNonConstWidget rebuilt');
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.red[100],
      child: const Text(
        'Non-Const Widget',
        style: TextStyle(fontWeight: FontWeight.bold),
      ),
    );
  }
}
```

What's happening here?
- const widgets are created once and cached
- Non-const widgets are recreated on every rebuild
- const improves performance significantly
- Use const whenever possible

---

# Const vs Non-Const Performance

> **Comparing** const and non-const performance.

```dart
/// Const vs Non-Const performance comparison
class ConstPerformanceComparison extends StatefulWidget {
  const ConstPerformanceComparison({super.key});

  @override
  State<ConstPerformanceComparison> createState() => _ConstPerformanceComparisonState();
}

class _ConstPerformanceComparisonState extends State<ConstPerformanceComparison> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('🟢 Parent rebuilt (counter: $_counter)');
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Const vs Non-Const'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Non-const widgets - rebuild every time
            const Text(
              'Non-Const (Rebuild every time)',
              style: TextStyle(
                fontWeight: FontWeight.bold,
                color: Colors.red,
              ),
            ),
            _buildNonConstWidgets(),
            
            const SizedBox(height: 16),
            
            // 2. Const widgets - cached, don't rebuild
            const Text(
              'Const (Cached)',
              style: TextStyle(
                fontWeight: FontWeight.bold,
                color: Colors.green,
              ),
            ),
            _buildConstWidgets(),
            
            const SizedBox(height: 16),
            
            // 3. Counter display
            Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 24),
            ),
            
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

  Widget _buildNonConstWidgets() {
    print('🔴 Building non-const widgets');
    return Column(
      children: [
        Container(
          padding: const EdgeInsets.all(8),
          color: Colors.red[100],
          child: Text('Non-const 1'),
        ),
        Container(
          padding: const EdgeInsets.all(8),
          color: Colors.red[100],
          child: Text('Non-const 2'),
        ),
        Container(
          padding: const EdgeInsets.all(8),
          color: Colors.red[100],
          child: Text('Non-const 3'),
        ),
      ],
    );
  }

  Widget _buildConstWidgets() {
    print('🟢 Building const widgets (once)');
    return const Column(
      children: [
        SizedBox.shrink(),
        SizedBox.shrink(),
        SizedBox.shrink(),
      ],
    );
  }
}
```

What's happening here?
- Non-const widgets rebuild every time
- Const widgets are cached and reused
- Const widgets significantly improve performance
- Use const for static content

---

# Const in Lists

> **Using const** in list builders.

```dart
/// Const in lists example
class ConstInListsExample extends StatefulWidget {
  const ConstInListsExample({super.key});

  @override
  State<ConstInListsExample> createState() => _ConstInListsExampleState();
}

class _ConstInListsExampleState extends State<ConstInListsExample> {
  final List<String> _items = List.generate(100, (index) => 'Item $index');
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Const in Lists'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              setState(() {
                _counter++;
              });
            },
          ),
        ],
      ),
      body: Column(
        children: [
          // Counter display
          Container(
            padding: const EdgeInsets.all(8),
            color: Colors.blue[50],
            child: Text(
              'Build count: $_counter',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          // List with const items
          Expanded(
            child: ListView.builder(
              itemCount: _items.length,
              itemBuilder: (context, index) {
                // 1. Non-const item (rebuilds every time)
                // return _buildNonConstItem(index);
                
                // 2. Const item (cached)
                return _buildConstItem(index);
              },
            ),
          ),
        ],
      ),
    );
  }

  // Non-const item - rebuilds every time
  Widget _buildNonConstItem(int index) {
    print('🔴 Building non-const item: $index');
    return Container(
      padding: const EdgeInsets.all(8),
      margin: const EdgeInsets.all(2),
      color: Colors.blue[100 * (index % 9 + 1)],
      child: Text('Item $index (non-const)'),
    );
  }

  // Const item - cached
  Widget _buildConstItem(int index) {
    print('🟢 Building const item: $index (once)');
    return Container(
      padding: const EdgeInsets.all(8),
      margin: const EdgeInsets.all(2),
      color: Colors.green[100 * (index % 9 + 1)],
      child: Text('Item $index (const)'),
    );
  }
}
```

What's happening here?
- Const items are built once and cached
- Non-const items rebuild on every refresh
- Const improves list performance
- Use const for list items when possible

---

# Const with Children

> **Using const** with child widgets.

```dart
/// Const with children example
class ConstWithChildrenExample extends StatefulWidget {
  const ConstWithChildrenExample({super.key});

  @override
  State<ConstWithChildrenExample> createState() => _ConstWithChildrenExampleState();
}

class _ConstWithChildrenExampleState extends State<ConstWithChildrenExample> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Const with Children'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Non-const children - rebuild every time
            const Text(
              'Non-Const Children (Rebuild)',
              style: TextStyle(
                fontWeight: FontWeight.bold,
                color: Colors.red,
              ),
            ),
            _buildNonConstChildren(),
            
            const SizedBox(height: 16),
            
            // 2. Const children - cached
            const Text(
              'Const Children (Cached)',
              style: TextStyle(
                fontWeight: FontWeight.bold,
                color: Colors.green,
              ),
            ),
            _buildConstChildren(),
            
            const SizedBox(height: 16),
            
            Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 24),
            ),
            
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

  Widget _buildNonConstChildren() {
    return Column(
      children: [
        // Non-const child
        Container(
          padding: const EdgeInsets.all(8),
          color: Colors.red[100],
          child: const Text('Non-const Child 1'),
        ),
        // Another non-const child
        Container(
          padding: const EdgeInsets.all(8),
          color: Colors.red[100],
          child: const Text('Non-const Child 2'),
        ),
      ],
    );
  }

  Widget _buildConstChildren() {
    return const Column(
      children: [
        // Const child
        ConstChildWidget(text: 'Const Child 1'),
        // Another const child
        ConstChildWidget(text: 'Const Child 2'),
      ],
    );
  }
}

/// Const child widget
class ConstChildWidget extends StatelessWidget {
  const ConstChildWidget({
    super.key,
    required this.text,
  });

  final String text;

  @override
  Widget build(BuildContext context) {
    print('🟢 ConstChildWidget built (cached): $text');
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.green[100],
      child: Text(text),
    );
  }
}
```

What's happening here?
- Const children are built once and cached
- Non-const children rebuild on every change
- Const improves performance with many children
- Use const for child widgets when possible

---

# Real-World Examples

> **Common patterns** with const widgets.

```dart
/// 1. Const icon buttons
class ConstIconButtons extends StatelessWidget {
  const ConstIconButtons({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: const [
        // Const icons
        Icon(Icons.star, color: Colors.yellow, size: 30),
        SizedBox(width: 8),
        Icon(Icons.favorite, color: Colors.red, size: 30),
        SizedBox(width: 8),
        Icon(Icons.home, color: Colors.blue, size: 30),
        SizedBox(width: 8),
        Icon(Icons.person, color: Colors.green, size: 30),
      ],
    );
  }
}

/// 2. Const card
class ConstCard extends StatelessWidget {
  const ConstCard({
    super.key,
    required this.title,
    required this.subtitle,
  });

  final String title;
  final String subtitle;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      margin: const EdgeInsets.all(8),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(8),
        boxShadow: const [
          BoxShadow(
            color: Colors.black12,
            blurRadius: 8,
            offset: Offset(0, 2),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            title,
            style: const TextStyle(
              fontWeight: FontWeight.bold,
              fontSize: 18,
            ),
          ),
          const SizedBox(height: 4),
          Text(
            subtitle,
            style: const TextStyle(color: Colors.grey),
          ),
        ],
      ),
    );
  }
}

/// 3. Const app bar
class ConstAppBar extends StatelessWidget {
  const ConstAppBar({super.key, required this.title});

  final String title;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      color: Colors.blue,
      child: Row(
        children: [
          const Icon(Icons.menu, color: Colors.white),
          const SizedBox(width: 16),
          Expanded(
            child: Text(
              title,
              style: const TextStyle(
                color: Colors.white,
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          const Icon(Icons.search, color: Colors.white),
          const SizedBox(width: 8),
          const Icon(Icons.more_vert, color: Colors.white),
        ],
      ),
    );
  }
}

/// 4. Const loading indicator
class ConstLoadingIndicator extends StatelessWidget {
  const ConstLoadingIndicator({super.key});

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          CircularProgressIndicator(),
          SizedBox(height: 16),
          Text(
            'Loading...',
            style: TextStyle(
              color: Colors.grey,
              fontSize: 16,
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Const icons and spacers
- Const card with const decorations
- Const app bar with const icons
- Const loading indicator

---

# Best Practices

## Use const for Static Widgets

```dart
// Good - const for static widgets
const Text('Hello World')
const Icon(Icons.star)
const SizedBox(height: 16)

// Bad - Non-const for static widgets
Text('Hello World')
Icon(Icons.star)
SizedBox(height: 16)
```

## Use const Constructors

```dart
// Good - const constructor
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});
  
  @override
  Widget build(BuildContext context) {
    return const Text('Hello');
  }
}

// Bad - Missing const
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Text('Hello');
  }
}
```

## Use const for List Items

```dart
// Good - const list items
const [
  Text('Item 1'),
  Text('Item 2'),
  Text('Item 3'),
]

// Bad - Non-const list items
[
  Text('Item 1'),
  Text('Item 2'),
  Text('Item 3'),
]
```

---

# Common Mistakes

## Missing const Keyword

Wrong:
```dart
// No const - rebuilds every time
Text('Hello World')
```

Correct:
```dart
// With const - cached
const Text('Hello World')
```

## Using const with Non-const Values

Wrong:
```dart
// Can't use const with non-const value
const Text('Hello $name') // Error
```

Correct:
```dart
// Without const for dynamic values
Text('Hello $name')
```

---

# Summary

Const widgets improve performance by caching widgets and preventing unnecessary rebuilds. Use const for static widgets, const constructors, and list items. Const widgets are built once and reused, significantly improving performance in Flutter applications.

---

# Next Steps

- [Widget Splitting](widget-splitting.md)
- [Keys](keys.md)
- [RepaintBoundary](repaintboundary.md)

---

# Did You Know?

- Const widgets are built once and cached
- Non-const widgets rebuild on every change
- const improves performance significantly
- Use const for static content
- const constructors enable const usage
- const lists are more efficient
- const reduces memory usage
- const improves hot reload performance