# What is a Widget?

Understand the fundamental building block of Flutter applications.

---

# What is it?

A widget is the basic building block of a Flutter application's user interface. Everything you see on screen in a Flutter app is a widget - from buttons and text to layouts and animations. Widgets describe what the UI should look like given its current configuration and state.

---

# Why does it exist?

Widgets exist to:

- Provide a declarative way to build UI
- Create a consistent and composable UI system
- Enable hot reload and fast development
- Support reactive programming model
- Handle different screen sizes and platforms
- Provide a rich catalog of pre-built components
- Make UI development intuitive and efficient

---

# Widget Philosophy

> **Widgets are the heart** of Flutter's UI system.

```dart
// Everything is a widget
class EverythingIsWidget extends StatelessWidget {
  const EverythingIsWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // Even layout and styling are widgets
    return Container(          // Widget
      padding: const EdgeInsets.all(16), // Widget
      color: Colors.blue,     // Part of widget
      child: Row(             // Widget
        children: [
          Icon(Icons.star),   // Widget
          Text('Hello'),      // Widget
          ElevatedButton(     // Widget
            onPressed: () {}, // Callback
            child: Text('Press'), // Widget
          ),
        ],
      ),
    );
  }
}

// Widget characteristics:
// 1. Immutable - Once created, cannot change
// 2. Composable - Build complex UIs from simple widgets
// 3. Declarative - Describe what UI should look like
// 4. Reactive - Automatically update when state changes
// 5. Platform-aware - Adapt to different platforms
```

What's happening here?
- Every UI element is a widget
- Widgets are composed together
- Widgets describe the UI
- Widgets are immutable
- Widgets are the foundation of Flutter

---

# Widget Types

> **Widgets are categorized** by their behavior and purpose.

```dart
// 1. StatelessWidget - No mutable state
class MyStatelessWidget extends StatelessWidget {
  const MyStatelessWidget({super.key, required this.title});

  final String title; // Immutable property

  @override
  Widget build(BuildContext context) {
    // Called when widget is created or parent changes
    return Text(title);
  }
}

// 2. StatefulWidget - Has mutable state
class MyStatefulWidget extends StatefulWidget {
  const MyStatefulWidget({super.key});

  @override
  State<MyStatefulWidget> createState() => _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  int _counter = 0; // Mutable state

  void _increment() {
    setState(() {
      _counter++; // Triggers rebuild
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// 3. RenderObjectWidget - Handles rendering
class MyRenderWidget extends LeafRenderObjectWidget {
  const MyRenderWidget({super.key});

  @override
  RenderObject createRenderObject(BuildContext context) {
    return _MyRenderObject();
  }
}

// 4. InheritedWidget - Shares data down the tree
class MyInheritedWidget extends InheritedWidget {
  const MyInheritedWidget({
    super.key,
    required this.data,
    required super.child,
  });

  final String data;

  static MyInheritedWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<MyInheritedWidget>();
  }

  @override
  bool updateShouldNotify(MyInheritedWidget oldWidget) {
    return data != oldWidget.data;
  }
}

// 5. ProxyWidget - Wraps child widgets
class MyProxyWidget extends ProxyWidget {
  const MyProxyWidget({super.key, required super.child});
}
```

What's happening here?
- StatelessWidget has no mutable state
- StatefulWidget has mutable state
- RenderObjectWidget handles custom rendering
- InheritedWidget shares data
- ProxyWidget wraps other widgets

---

# Widget Composition

> **Composition** is how widgets are combined.

```dart
// Widget composition examples
class ProfileCard extends StatelessWidget {
  const ProfileCard({super.key});

  @override
  Widget build(BuildContext context) {
    // Composing widgets to create complex UI
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(8),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 8,
          ),
        ],
      ),
      child: Row(
        children: [
          // Compose avatar
          const CircleAvatar(
            radius: 30,
            backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
          ),
          const SizedBox(width: 16),
          
          // Compose user info
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'John Doe',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                Text(
                  'Flutter Developer',
                  style: TextStyle(
                    color: Colors.grey[600],
                    fontSize: 14,
                  ),
                ),
              ],
            ),
          ),
          
          // Compose action button
          IconButton(
            icon: const Icon(Icons.more_vert),
            onPressed: () {},
          ),
        ],
      ),
    );
  }
}

// Composing widgets in different ways:
class CompositionExample extends StatelessWidget {
  const CompositionExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Vertical composition
        const SizedBox(
          height: 10,
        ),
        
        // 2. Horizontal composition
        const Row(
          children: [
            Icon(Icons.star),
            Text('Star'),
          ],
        ),
        
        // 3. Nested composition
        Container(
          child: Row(
            children: [
              Container(
                child: const Text('Nested'),
              ),
            ],
          ),
        ),
        
        // 4. Conditional composition
        if (true)
          const Text('Conditional'),
        
        // 5. List composition
        ...['A', 'B', 'C'].map((letter) => Text(letter)),
      ],
    );
  }
}
```

What's happening here?
- Widgets are composed together
- Complex UIs from simple widgets
- Composition over inheritance
- Nested and conditional composition
- Reusable widget components

---

# Widget Immutability

> **Widgets are immutable** and cannot change.

```dart
// Widget immutability
class ImmutableWidget extends StatelessWidget {
  // 1. All properties must be final
  final String title;
  final int count;
  final VoidCallback? onPressed;

  // 2. Constructor with required parameters
  const ImmutableWidget({
    super.key,
    required this.title,
    this.count = 0,
    this.onPressed,
  });

  @override
  Widget build(BuildContext context) {
    // 3. Widgets never change
    // New widget created when properties change
    return Container(
      child: Column(
        children: [
          Text(title), // Uses immutable value
          Text('Count: $count'), // Uses immutable value
          if (onPressed != null)
            ElevatedButton(
              onPressed: onPressed, // Callback can be called
              child: const Text('Press'),
            ),
        ],
      ),
    );
  }
}

// Usage
class ImmutableExample extends StatefulWidget {
  const ImmutableExample({super.key});

  @override
  State<ImmutableExample> createState() => _ImmutableExampleState();
}

class _ImmutableExampleState extends State<ImmutableExample> {
  String _title = 'Hello';
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    // 4. New widget created on rebuild
    return ImmutableWidget(
      title: _title,
      count: _count,
      onPressed: () {
        setState(() {
          _count++;
        });
      },
    );
  }
}

// Why immutability?
// 1. Easier to reason about
// 2. Hot reload works
// 3. Performance optimization
// 4. State management simplicity
// 5. Predictable behavior
```

What's happening here?
- Widgets are immutable
- Properties must be final
- New widgets on changes
- State is stored elsewhere
- Immutability enables hot reload

---

# Widget Lifecycle (Stateless)

> **StatelessWidget lifecycle** is simple.

```dart
// StatelessWidget lifecycle
class StatelessLifecycleWidget extends StatelessWidget {
  const StatelessLifecycleWidget({super.key, required this.message});

  final String message;

  // 1. Constructor called
  const StatelessLifecycleWidget({super.key, required this.message}) {
    print('1. Constructor called');
  }

  @override
  Widget build(BuildContext context) {
    // 2. build() called
    print('2. build() called');
    
    // 3. Returns widget tree
    return Container(
      child: Text(message),
    );
  }

  // Lifecycle events:
  // 1. Constructor - Widget is created
  // 2. build() - Widget tree is built
  // 3. Rebuild - Called when parent rebuilds
  // 4. Dispose - Widget is removed

  // No mutable state
  // No state management methods
  // Pure UI representation
}

// When StatelessWidget rebuilds:
// 1. Parent widget rebuilds
// 2. Inherited widget changes
// 3. Hot reload
// 4. BuildContext changes
```

What's happening here?
- StatelessWidget has simple lifecycle
- Constructor creates widget
- build() builds UI
- No state to manage
- Pure and predictable

---

# Widget Lifecycle (Stateful)

> **StatefulWidget lifecycle** is more complex.

```dart
// StatefulWidget lifecycle
class StatefulLifecycleWidget extends StatefulWidget {
  const StatefulLifecycleWidget({super.key});

  @override
  State<StatefulLifecycleWidget> createState() => _StatefulLifecycleWidgetState();
}

class _StatefulLifecycleWidgetState extends State<StatefulLifecycleWidget> {
  int _counter = 0;

  // 1. initState() - Called once when widget is created
  @override
  void initState() {
    super.initState();
    print('1. initState()');
    // Initialize data
    // Start animations
    // Subscribe to streams
    _counter = 0;
  }

  // 2. didChangeDependencies() - Called when dependencies change
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print('2. didChangeDependencies()');
    // Called when inherited widget changes
    // Called after initState
    // Called when dependencies change
  }

  // 3. build() - Called multiple times
  @override
  Widget build(BuildContext context) {
    print('3. build() called');
    return Column(
      children: [
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('Increment'),
        ),
      ],
    );
  }

  void _increment() {
    // 4. setState() triggers rebuild
    setState(() {
      _counter++;
    });
  }

  // 5. didUpdateWidget() - Called when parent rebuilds
  @override
  void didUpdateWidget(covariant StatefulLifecycleWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('4. didUpdateWidget()');
    // Called when widget configuration changes
    // Compare old and new widget
  }

  // 6. deactivate() - Called when removed from tree
  @override
  void deactivate() {
    super.deactivate();
    print('5. deactivate()');
    // Called when widget is removed
    // Can be reinserted
  }

  // 7. dispose() - Called when permanently removed
  @override
  void dispose() {
    super.dispose();
    print('6. dispose()');
    // Clean up resources
    // Cancel subscriptions
    // Dispose controllers
  }
}

// Lifecycle summary:
// 1. createState() - Create state object
// 2. mounted = true
// 3. initState() - Initialize
// 4. didChangeDependencies() - Dependencies changed
// 5. build() - Build widget tree
// 6. didUpdateWidget() - Widget updated
// 7. setState() - Rebuild triggered
// 8. deactivate() - Removed from tree
// 9. dispose() - Permanently removed
```

What's happening here?
- StatefulWidget has complex lifecycle
- State persists across rebuilds
- initState initializes state
- setState triggers rebuild
- dispose cleans up resources

---

# Widget Keys

> **Keys** identify widgets in the tree.

```dart
// Widget keys
class KeysExample extends StatelessWidget {
  const KeysExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. ValueKey - Uses value for identity
        Text(
          'Value Key',
          key: const ValueKey('text_1'),
        ),
        
        // 2. ObjectKey - Uses object identity
        Text(
          'Object Key',
          key: ObjectKey('text_2'),
        ),
        
        // 3. UniqueKey - Always unique
        Text(
          'Unique Key',
          key: UniqueKey(),
        ),
        
        // 4. PageStorageKey - Preserves scroll position
        ListView.builder(
          key: const PageStorageKey('list_page'),
          itemCount: 100,
          itemBuilder: (context, index) {
            return ListTile(
              title: Text('Item $index'),
            );
          },
        ),
        
        // 5. GlobalKey - Access widget globally
        // GlobalKey<State> key = GlobalKey();
        // Container(key: key)
      ],
    );
  }
}

// When to use keys:
// 1. Reordering widgets in lists
// 2. Preserving state when moving widgets
// 3. Preserving scroll position
// 4. Accessing widget state globally
// 5. Form validation and focus management

// Key examples:
class KeyedListWidget extends StatefulWidget {
  const KeyedListWidget({super.key});

  @override
  State<KeyedListWidget> createState() => _KeyedListWidgetState();
}

class _KeyedListWidgetState extends State<KeyedListWidget> {
  List<String> _items = ['Item 1', 'Item 2', 'Item 3'];

  void _addItem() {
    setState(() {
      _items.add('Item ${_items.length + 1}');
    });
  }

  void _removeItem() {
    setState(() {
      if (_items.isNotEmpty) {
        _items.removeLast();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Row(
          children: [
            ElevatedButton(
              onPressed: _addItem,
              child: const Text('Add'),
            ),
            ElevatedButton(
              onPressed: _removeItem,
              child: const Text('Remove'),
            ),
          ],
        ),
        Column(
          children: _items.map((item) {
            // With key - Efficient update
            return Text(
              item,
              key: ValueKey(item),
            );
          }).toList(),
        ),
      ],
    );
  }
}
```

What's happening here?
- Keys identify widgets
- Different key types for different needs
- Keys improve performance
- Keys preserve state
- Keys enable global access

---

# Widget Catalog

> **Common widget categories** in Flutter.

```dart
// Widget catalog categories
class WidgetCatalog extends StatelessWidget {
  const WidgetCatalog({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Layout Widgets
        // Container, Row, Column, Stack, Expanded, Flexible
        
        // 2. Basic Widgets
        // Text, Container, Image, Icon, Button
        
        // 3. Input Widgets
        // TextField, Checkbox, Radio, Slider, Switch
        
        // 4. Scrolling Widgets
        // ListView, GridView, CustomScrollView, SingleChildScrollView
        
        // 5. Navigation Widgets
        // Navigator, Router, Route
        
        // 6. Material Widgets
        // Scaffold, AppBar, Card, Drawer, FloatingActionButton
        
        // 7. Cupertino Widgets
        // CupertinoApp, CupertinoNavigationBar, CupertinoButton
        
        // 8. Animation Widgets
        // AnimatedContainer, AnimatedOpacity, Hero
        
        // 9. Async Widgets
        // FutureBuilder, StreamBuilder, ValueListenableBuilder
        
        // 10. Accessibility Widgets
        // Semantics, ExcludeSemantics, MergeSemantics
      ],
    );
  }
}

// Widget classification by purpose:
// 1. Building blocks - Basic UI elements
// 2. Layout - Arranging other widgets
// 3. Styling - Visual appearance
// 4. Behavior - Interactivity
// 5. Platform - Material and Cupertino
// 6. Async - Handling async data
```

What's happening here?
- Widgets are categorized by purpose
- Different categories for different needs
- Hundreds of built-in widgets
- Community packages add more
- Choose right widget for the job

---

# Best Practices

## Keep Widgets Small and Focused

```dart
// Good - Small, focused widgets
class UserAvatar extends StatelessWidget {
  const UserAvatar({super.key, required this.imageUrl});

  final String imageUrl;

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      backgroundImage: NetworkImage(imageUrl),
      radius: 30,
    );
  }
}

class UserName extends StatelessWidget {
  const UserName({super.key, required this.name});

  final String name;

  @override
  Widget build(BuildContext context) {
    return Text(
      name,
      style: const TextStyle(
        fontSize: 18,
        fontWeight: FontWeight.bold,
      ),
    );
  }
}

// Bad - Large, unfocused widget
class LargeWidget extends StatelessWidget {
  const LargeWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // Everything in one widget
    return Container(
      child: Column(
        children: [
          // Avatar
          // Name
          // Email
          // Bio
          // Actions
          // All in one place
        ],
      ),
    );
  }
}
```

## Use const Widgets

```dart
// Good - Using const
@override
Widget build(BuildContext context) {
  return const Column(
    children: [
      Text('Hello'),
      Text('World'),
    ],
  );
}

// Bad - Not using const
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      Text('Hello'),
      Text('World'),
    ],
  );
}
```

## Choose Appropriate Widget Types

```dart
// Good - Using StatelessWidget for static UI
class StaticText extends StatelessWidget {
  const StaticText({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text('Hello World');
  }
}

// Good - Using StatefulWidget for interactive UI
class Counter extends StatefulWidget {
  const Counter({super.key});

  @override
  State<Counter> createState() => _CounterState();
}

// Bad - Using StatefulWidget for static UI
class BadWidget extends StatefulWidget {
  const BadWidget({super.key});

  @override
  State<BadWidget> createState() => _BadWidgetState();
}
```

---

# Common Mistakes

## Mutating Widgets

Wrong:
```dart
// Don't mutate widgets
var text = Text('Hello');
text = Text('World'); // Wrong approach
```

Correct:
```dart
// Use state to change content
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  String text = 'Hello';
  
  void changeText() {
    setState(() {
      text = 'World';
    });
  }
}
```

## Not Using Keys When Needed

Wrong:
```dart
// No keys for dynamic lists
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Text(items[index]); // May cause issues
  },
)
```

Correct:
```dart
// Use keys for dynamic lists
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Text(
      items[index],
      key: ValueKey(index),
    );
  },
)
```

---

# Summary

Widgets are the building blocks of Flutter UIs. They are immutable, composable, and declarative. Understanding widget types, composition, lifecycle, and best practices is essential for building Flutter applications effectively.

---

# Next Steps

- [StatelessWidget](statelesswidget.md)
- [StatefulWidget](statefulwidget.md)
- [BuildContext](buildcontext.md)

---

# Did You Know?

- There are over 100 built-in widgets
- Widgets are immutable by design
- StatelessWidgets are more performant
- StatefulWidgets have 9 lifecycle methods
- Keys optimize widget updates
- Everything in Flutter is a widget
- Widgets can be composed infinitely
- The widget tree can have thousands of nodes