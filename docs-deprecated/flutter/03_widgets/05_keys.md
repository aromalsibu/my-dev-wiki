# Keys

Understand how Keys identify and preserve widgets in the widget tree.

---

# What is it?

Keys are identifiers that Flutter uses to uniquely identify widgets in the widget tree. They help Flutter efficiently update the UI by distinguishing between widgets, especially when widgets are added, removed, or reordered. Keys preserve state and improve performance during widget tree updates.

---

# Why does it exist?

Keys exist to:

- Uniquely identify widgets in the widget tree
- Preserve state when widgets are moved or reordered
- Improve performance during widget updates
- Enable efficient list and grid rendering
- Support form validation and focus management
- Access widgets globally via GlobalKey
- Maintain scroll positions in lists

---

# When to Use Keys

> **Keys are needed** in specific scenarios.

```dart
// 1. Reordering widgets in a list
class ReorderList extends StatefulWidget {
  const ReorderList({super.key});

  @override
  State<ReorderList> createState() => _ReorderListState();
}

class _ReorderListState extends State<ReorderList> {
  List<String> _items = ['Apple', 'Banana', 'Orange'];

  void _reorder(int oldIndex, int newIndex) {
    setState(() {
      if (oldIndex < newIndex) {
        newIndex -= 1;
      }
      final item = _items.removeAt(oldIndex);
      _items.insert(newIndex, item);
    });
  }

  @override
  Widget build(BuildContext context) {
    return ReorderableListView(
      onReorder: _reorder,
      children: _items.map((item) {
        // Key preserves state when reordering
        return ListTile(
          key: ValueKey(item),
          title: Text(item),
        );
      }).toList(),
    );
  }
}

// 2. Adding/removing widgets
class AddRemoveList extends StatefulWidget {
  const AddRemoveList({super.key});

  @override
  State<AddRemoveList> createState() => _AddRemoveListState();
}

class _AddRemoveListState extends State<AddRemoveList> {
  List<Widget> _widgets = [];

  void _addWidget() {
    setState(() {
      _widgets.add(
        Container(
          key: UniqueKey(), // Unique key for each widget
          color: Colors.blue,
          height: 50,
          margin: const EdgeInsets.all(4),
        ),
      );
    });
  }

  void _removeWidget() {
    setState(() {
      if (_widgets.isNotEmpty) {
        _widgets.removeLast();
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
              onPressed: _addWidget,
              child: const Text('Add'),
            ),
            ElevatedButton(
              onPressed: _removeWidget,
              child: const Text('Remove'),
            ),
          ],
        ),
        ..._widgets,
      ],
    );
  }
}

// 3. Preserving state across widget moves
class PreserveState extends StatefulWidget {
  const PreserveState({super.key});

  @override
  State<PreserveState> createState() => _PreserveStateState();
}

class _PreserveStateState extends State<PreserveState> {
  bool _showFirst = true;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Without key - state lost
        if (_showFirst)
          const CounterWithoutKey()
        else
          const CounterWithoutKey(),
        
        // With key - state preserved
        if (_showFirst)
          const CounterWithKey(key: ValueKey('counter'))
        else
          const CounterWithKey(key: ValueKey('counter')),
        
        ElevatedButton(
          onPressed: () {
            setState(() {
              _showFirst = !_showFirst;
            });
          },
          child: Text(_showFirst ? 'Show Second' : 'Show First'),
        ),
      ],
    );
  }
}

class CounterWithKey extends StatefulWidget {
  const CounterWithKey({super.key});

  @override
  State<CounterWithKey> createState() => _CounterWithKeyState();
}

class _CounterWithKeyState extends State<CounterWithKey> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Text('With Key: $_count'),
        ElevatedButton(
          onPressed: () {
            setState(() => _count++);
          },
          child: const Text('+'),
        ),
      ],
    );
  }
}
```

What's happening here?
- Keys preserve state during reorder
- Unique keys for dynamic widgets
- Keys preserve state when widget type changes
- Keys enable efficient updates

---

# Key Types

> **Different key types** serve different purposes.

```dart
// 1. ValueKey - Uses value for identity
class ValueKeyExample extends StatelessWidget {
  const ValueKeyExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Uses the string value as identity
        Text(
          'Item 1',
          key: const ValueKey('item_1'),
        ),
        Text(
          'Item 2',
          key: const ValueKey('item_2'),
        ),
        // Uses integer value
        Text(
          'Item 3',
          key: ValueKey(3),
        ),
      ],
    );
  }
}

// 2. ObjectKey - Uses object identity
class Person {
  final String id;
  final String name;
  
  const Person(this.id, this.name);
  
  @override
  bool operator ==(Object other) {
    return other is Person && other.id == id;
  }
  
  @override
  int get hashCode => id.hashCode;
}

class ObjectKeyExample extends StatelessWidget {
  const ObjectKeyExample({super.key});

  @override
  Widget build(BuildContext context) {
    final person = Person('1', 'Alice');
    
    return Column(
      children: [
        // Uses object identity
        Text(
          person.name,
          key: ObjectKey(person),
        ),
      ],
    );
  }
}

// 3. UniqueKey - Always unique
class UniqueKeyExample extends StatelessWidget {
  const UniqueKeyExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Each widget gets unique key
        Container(
          key: UniqueKey(),
          color: Colors.red,
          height: 50,
        ),
        Container(
          key: UniqueKey(),
          color: Colors.blue,
          height: 50,
        ),
      ],
    );
  }
}

// 4. GlobalKey - Accessible anywhere
class GlobalKeyExample extends StatefulWidget {
  const GlobalKeyExample({super.key});

  @override
  State<GlobalKeyExample> createState() => _GlobalKeyExampleState();
}

class _GlobalKeyExampleState extends State<GlobalKeyExample> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  final GlobalKey<ScaffoldState> _scaffoldKey = GlobalKey<ScaffoldState>();

  void _validateForm() {
    // Access form state using GlobalKey
    if (_formKey.currentState?.validate() ?? false) {
      _formKey.currentState?.save();
      // Form is valid
    }
  }

  void _showSnackBar() {
    // Access scaffold state using GlobalKey
    _scaffoldKey.currentState?.showSnackBar(
      const SnackBar(content: Text('Hello')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      key: _scaffoldKey,
      body: Form(
        key: _formKey,
        child: Column(
          children: [
            TextFormField(
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return 'Required';
                }
                return null;
              },
            ),
            ElevatedButton(
              onPressed: _validateForm,
              child: const Text('Validate'),
            ),
            ElevatedButton(
              onPressed: _showSnackBar,
              child: const Text('Show SnackBar'),
            ),
          ],
        ),
      ),
    );
  }
}

// 5. PageStorageKey - Preserves scroll position
class PageStorageKeyExample extends StatefulWidget {
  const PageStorageKeyExample({super.key});

  @override
  State<PageStorageKeyExample> createState() => _PageStorageKeyExampleState();
}

class _PageStorageKeyExampleState extends State<PageStorageKeyExample> {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      // Preserves scroll position
      key: const PageStorageKey('my_list'),
      itemCount: 100,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text('Item $index'),
        );
      },
    );
  }
}
```

What's happening here?
- ValueKey: identifies by value
- ObjectKey: identifies by object
- UniqueKey: always unique
- GlobalKey: accessible anywhere
- PageStorageKey: preserves scroll

---

# Key Comparison

> **Choosing the right key** for your use case.

```dart
// Comparison of key types
class KeyComparison extends StatelessWidget {
  const KeyComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. ValueKey - Best for lists with unique IDs
        const Text(
          'ValueKey',
          key: ValueKey('text_1'),
        ),
        
        // 2. ObjectKey - Best for custom objects
        const Text(
          'ObjectKey',
          key: ObjectKey('text_2'),
        ),
        
        // 3. UniqueKey - Best for dynamic content
        const Text(
          'UniqueKey',
          key: UniqueKey(),
        ),
        
        // 4. GlobalKey - Best for form access
        // final GlobalKey _key = GlobalKey();
        // Text('GlobalKey', key: _key),
        
        // 5. PageStorageKey - Best for scroll positions
        // ListView(key: PageStorageKey('list')),
      ],
    );
  }
}

// When to use each key:
// ValueKey:
// ✓ Items with unique IDs
// ✓ Database IDs
// ✓ Email addresses
// ✓ Usernames

// ObjectKey:
// ✓ Complex objects
// ✓ Custom classes
// ✓ Objects with equality

// UniqueKey:
// ✓ Dynamic widgets
// ✓ Widgets without natural identity
// ✓ Animations

// GlobalKey:
// ✓ Form validation
// ✓ Accessing widget state
// ✓ Managing focus
// ✓ Testing

// PageStorageKey:
// ✓ List scroll positions
// ✓ Grid scroll positions
// ✓ Tab views
```

What's happening here?
- ValueKey for IDs
- ObjectKey for objects
- UniqueKey for dynamic content
- GlobalKey for global access
- PageStorageKey for scroll

---

# Keys in Lists

> **Keys are essential** for efficient list updates.

```dart
// List with keys
class KeyedList extends StatefulWidget {
  const KeyedList({super.key});

  @override
  State<KeyedList> createState() => _KeyedListState();
}

class _KeyedListState extends State<KeyedList> {
  List<String> _items = ['A', 'B', 'C', 'D'];

  void _addItem() {
    setState(() {
      _items.insert(2, 'X');
    });
  }

  void _removeItem() {
    setState(() {
      if (_items.isNotEmpty) {
        _items.removeAt(1);
      }
    });
  }

  void _reorderItem() {
    setState(() {
      if (_items.length > 2) {
        final temp = _items[1];
        _items[1] = _items[2];
        _items[2] = temp;
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
            ElevatedButton(
              onPressed: _reorderItem,
              child: const Text('Reorder'),
            ),
          ],
        ),
        // With keys - efficient updates
        ..._items.map((item) {
          return Container(
            key: ValueKey(item), // Unique key
            color: Colors.blue,
            padding: const EdgeInsets.all(8),
            margin: const EdgeInsets.all(4),
            child: Text(item),
          );
        }).toList(),
        
        const Divider(),
        
        // Without keys - inefficient updates
        ..._items.map((item) {
          return Container(
            // No key
            color: Colors.green,
            padding: const EdgeInsets.all(8),
            margin: const EdgeInsets.all(4),
            child: Text(item),
          );
        }).toList(),
      ],
    );
  }
}

// ListView.builder with keys
class ListBuilderWithKeys extends StatelessWidget {
  const ListBuilderWithKeys({
    super.key,
    required this.items,
  });

  final List<String> items;

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: items.length,
      itemBuilder: (context, index) {
        return ListTile(
          // Key based on unique identifier
          key: ValueKey(items[index]),
          title: Text(items[index]),
        );
      },
    );
  }
}
```

What's happening here?
- Keys enable efficient list updates
- Without keys, all widgets rebuild
- With keys, only changed widgets rebuild
- Keys preserve state in lists
- Use ValueKey for list items

---

# GlobalKey in Detail

> **GlobalKey provides** global access to widgets.

```dart
// GlobalKey usage
class GlobalKeyDetailed extends StatefulWidget {
  const GlobalKeyDetailed({super.key});

  @override
  State<GlobalKeyDetailed> createState() => _GlobalKeyDetailedState();
}

class _GlobalKeyDetailedState extends State<GlobalKeyDetailed> {
  // 1. Accessing state
  final GlobalKey<ScaffoldState> scaffoldKey = GlobalKey<ScaffoldState>();
  final GlobalKey<FormState> formKey = GlobalKey<FormState>();
  
  // 2. Accessing any widget
  final GlobalKey<State<StatefulWidget>> widgetKey = GlobalKey();
  
  // 3. Accessing specific widget type
  final GlobalKey<_CustomWidgetState> customKey = GlobalKey();

  void _useGlobalKeys() {
    // Access scaffold methods
    scaffoldKey.currentState?.showSnackBar(
      const SnackBar(content: Text('Hello')),
    );
    
    // Access form methods
    if (formKey.currentState?.validate() ?? false) {
      formKey.currentState?.save();
    }
    
    // Access widget state
    final widgetState = customKey.currentState;
    widgetState?.doSomething();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      key: scaffoldKey,
      body: Form(
        key: formKey,
        child: Column(
          children: [
            const Text('GlobalKey Example'),
            
            // Custom widget with GlobalKey
            CustomWidget(
              key: customKey,
            ),
            
            ElevatedButton(
              onPressed: _useGlobalKeys,
              child: const Text('Use GlobalKeys'),
            ),
          ],
        ),
      ),
    );
  }
}

class CustomWidget extends StatefulWidget {
  const CustomWidget({super.key});

  @override
  State<CustomWidget> createState() => _CustomWidgetState();
}

class _CustomWidgetState extends State<CustomWidget> {
  String _text = 'Initial';

  void doSomething() {
    setState(() {
      _text = 'Updated!';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Text(_text);
  }
}

// GlobalKey rules:
// 1. GlobalKey must be unique
// 2. GlobalKey should not be rebuilt
// 3. GlobalKey persists across rebuilds
// 4. GlobalKey can find widget anywhere
// 5. GlobalKey works with StatefulWidget only
```

What's happening here?
- GlobalKey accesses scaffold
- GlobalKey validates forms
- GlobalKey accesses custom state
- GlobalKey enables global access
- GlobalKey persists across rebuilds

---

# Best Practices

## Use Keys Only When Needed

```dart
// Good - Keys only when necessary
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) {
      return ListTile(
        // Use key for dynamic lists
        key: ValueKey(items[index].id),
        title: Text(items[index].name),
      );
    },
  );
}

// Bad - Unnecessary keys
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      // No key needed for static widgets
      const Text('Hello', key: UniqueKey()), // Unnecessary
      const Text('World', key: UniqueKey()), // Unnecessary
    ],
  );
}
```

## Use Appropriate Key Types

```dart
// Good - ValueKey for unique IDs
@override
Widget build(BuildContext context) {
  return items.map((item) {
    return Text(
      item.name,
      key: ValueKey(item.id),
    );
  }).toList();
}

// Bad - UniqueKey for everything
@override
Widget build(BuildContext context) {
  return items.map((item) {
    return Text(
      item.name,
      key: UniqueKey(), // Causes unnecessary rebuilds
    );
  }).toList();
}
```

## Store GlobalKeys Properly

```dart
// Good - Store in State
class _MyWidgetState extends State<MyWidget> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  // ✓ Stored in State, persists
}

// Bad - Store in build
@override
Widget build(BuildContext context) {
  final GlobalKey<FormState> formKey = GlobalKey<FormState>();
  // ❌ Created every rebuild
}
```

---

# Common Mistakes

## Not Using Keys in Lists

Wrong:
```dart
// No keys
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text(items[index]),
    );
  },
)
```

Correct:
```dart
// With keys
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(
      key: ValueKey(items[index]),
      title: Text(items[index]),
    );
  },
)
```

## Using Same Key Multiple Times

Wrong:
```dart
// Duplicate keys
Column(
  children: [
    Text('A', key: const ValueKey('same')),
    Text('B', key: const ValueKey('same')), // Duplicate key
  ],
)
```

Correct:
```dart
// Unique keys
Column(
  children: [
    Text('A', key: const ValueKey('a')),
    Text('B', key: const ValueKey('b')),
  ],
)
```

---

# Summary

Keys identify widgets and help Flutter efficiently update the UI. Use keys for dynamic lists, reordering, and preserving state. Choose the appropriate key type (ValueKey, ObjectKey, UniqueKey, GlobalKey, PageStorageKey) based on your use case.

---

# Next Steps

- [Widget Lifecycle](widget-lifecycle.md)
- [Widget Composition](widget-composition.md)
- [Widget Catalog](widget-catalog.md)

---

# Did You Know?

- Keys are optional for static widgets
- Keys improve performance in lists
- GlobalKey accesses widgets globally
- PageStorageKey preserves scroll position
- UniqueKey is always unique
- ValueKey uses value equality
- ObjectKey uses object identity
- Keys should be unique in their scope