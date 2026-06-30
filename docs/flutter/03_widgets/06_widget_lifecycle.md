# Widget Lifecycle

Understand the complete lifecycle of widgets in Flutter.

---

# What is it?

The widget lifecycle refers to the series of events that occur from the moment a widget is created until it is destroyed. Both StatelessWidget and StatefulWidget have lifecycles, but StatefulWidget has a much richer lifecycle with multiple stages. Understanding the lifecycle helps you manage resources, handle state changes, and optimize performance.

---

# Why does it exist?

The widget lifecycle exists to:

- Manage widget creation and destruction
- Initialize and clean up resources
- Handle state changes and updates
- Optimize performance
- Manage animations and controllers
- Handle dependencies and inherited widgets
- Enable proper resource management

---

# StatelessWidget Lifecycle

> **StatelessWidget has a simple lifecycle** with only two main stages.

```dart
// StatelessWidget lifecycle
class StatelessLifecycle extends StatelessWidget {
  const StatelessLifecycle({super.key, required this.message});

  final String message;

  // 1. Constructor - Widget is created
  // Called when widget is instantiated
  const StatelessLifecycle({super.key, required this.message}) {
    print('1. Constructor called');
  }

  @override
  Widget build(BuildContext context) {
    // 2. build() - Widget is built
    // Called after constructor
    // Called when parent rebuilds
    // Called when dependencies change
    print('2. build() called');
    
    return Text(message);
  }

  // Lifecycle summary:
  // 1. Constructor → Widget is created
  // 2. build() → Widget is built
  // 3. Rebuild → Called when parent rebuilds
  // 4. Removed → Widget is removed from tree
}

// Usage example
class ParentWidget extends StatefulWidget {
  const ParentWidget({super.key});

  @override
  State<ParentWidget> createState() => _ParentWidgetState();
}

class _ParentWidgetState extends State<ParentWidget> {
  String _message = 'Hello';
  bool _showChild = true;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Child rebuilds when parent rebuilds
        if (_showChild)
          StatelessLifecycle(message: _message),
        
        ElevatedButton(
          onPressed: () {
            setState(() {
              _message = 'Updated';
            });
          },
          child: const Text('Update'),
        ),
        ElevatedButton(
          onPressed: () {
            setState(() {
              _showChild = !_showChild;
            });
          },
          child: const Text('Toggle'),
        ),
      ],
    );
  }
}
```

What's happening here?
- Constructor creates widget
- build() builds UI
- Parent rebuilds trigger rebuild
- Widgets are removed when condition changes
- Simple and predictable

---

# StatefulWidget Lifecycle

> **StatefulWidget has a rich lifecycle** with multiple stages.

```dart
// Complete StatefulWidget lifecycle
class CompleteLifecycle extends StatefulWidget {
  const CompleteLifecycle({super.key, required this.initialValue});

  final String initialValue;

  @override
  State<CompleteLifecycle> createState() => _CompleteLifecycleState();
}

class _CompleteLifecycleState extends State<CompleteLifecycle> {
  String _value = '';
  
  // 1. Constructor (State object created)
  _CompleteLifecycleState() {
    print('1. State constructor');
  }

  // 2. initState() - Called once when State is created
  @override
  void initState() {
    super.initState();
    print('2. initState()');
    _value = widget.initialValue;
    // Initialize data
    // Start animations
    // Subscribe to streams
    // Set up controllers
  }

  // 3. didChangeDependencies() - Called when dependencies change
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print('3. didChangeDependencies()');
    // Called after initState
    // Called when inherited widget changes
    // Called when dependencies change
    // Safe to use context here
  }

  // 4. build() - Called multiple times
  @override
  Widget build(BuildContext context) {
    print('4. build()');
    return Text(_value);
  }

  // 5. setState() - Manually triggered rebuild
  void _updateValue(String newValue) {
    setState(() {
      print('5. setState() called');
      _value = newValue;
    });
  }

  // 6. didUpdateWidget() - Called when parent rebuilds
  @override
  void didUpdateWidget(covariant CompleteLifecycle oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('6. didUpdateWidget()');
    // Called when widget configuration changes
    // Compare old and new widget properties
    if (widget.initialValue != oldWidget.initialValue) {
      setState(() {
        _value = widget.initialValue;
      });
    }
  }

  // 7. deactivate() - Called when removed from tree
  @override
  void deactivate() {
    super.deactivate();
    print('7. deactivate()');
    // Called when widget is removed from tree
    // Can be reinserted
    // State is preserved
  }

  // 8. dispose() - Called when permanently removed
  @override
  void dispose() {
    print('8. dispose()');
    // Clean up resources
    // Cancel subscriptions
    // Dispose controllers
    super.dispose();
  }
}

// Lifecycle flow diagram:
// createState() → constructor → initState() → didChangeDependencies() → build()
//   ↓
// setState() → build() (repeated)
//   ↓
// didUpdateWidget() (when parent changes)
//   ↓
// deactivate() (removed from tree)
//   ↓
// dispose() (permanently removed)
```

What's happening here?
- initState: one-time initialization
- didChangeDependencies: handle dependency changes
- build: build UI (called multiple times)
- setState: trigger rebuild
- didUpdateWidget: handle parent changes
- deactivate: temporary removal
- dispose: permanent removal

---

# Lifecycle in Detail

> **Each lifecycle method** has a specific purpose.

```dart
// Detailed lifecycle with real-world usage
class DetailedLifecycle extends StatefulWidget {
  const DetailedLifecycle({super.key});

  @override
  State<DetailedLifecycle> createState() => _DetailedLifecycleState();
}

class _DetailedLifecycleState extends State<DetailedLifecycle>
    with SingleTickerProviderStateMixin {
  
  late AnimationController _controller;
  late StreamSubscription _subscription;
  String _data = 'Loading...';
  bool _isInitialized = false;

  // 1. initState - Initialize resources
  @override
  void initState() {
    super.initState();
    
    // Initialize animation controller
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );
    
    // Subscribe to stream
    _subscription = dataStream.listen((data) {
      if (mounted) {
        setState(() {
          _data = data;
        });
      }
    });
    
    // Start animation
    _controller.forward();
    
    _isInitialized = true;
  }

  // 2. didChangeDependencies - Handle inherited widget changes
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    
    // Access inherited widgets
    final theme = Theme.of(context);
    final mediaQuery = MediaQuery.of(context);
    
    // Update UI based on dependencies
    if (mediaQuery.size.width < 600) {
      // Mobile layout
    } else {
      // Tablet layout
    }
  }

  // 3. build - Build UI
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Data: $_data'),
        AnimatedBuilder(
          animation: _controller,
          builder: (context, child) {
            return Transform.rotate(
              angle: _controller.value * 2 * 3.14,
              child: const Icon(Icons.refresh),
            );
          },
        ),
        ElevatedButton(
          onPressed: () {
            setState(() {
              _data = 'Updated';
            });
          },
          child: const Text('Update'),
        ),
      ],
    );
  }

  // 4. didUpdateWidget - Handle widget changes
  @override
  void didUpdateWidget(covariant DetailedLifecycle oldWidget) {
    super.didUpdateWidget(oldWidget);
    // Called when parent changes widget configuration
    // Update state based on new widget properties
  }

  // 5. deactivate - Temporary removal
  @override
  void deactivate() {
    super.deactivate();
    // Widget is removed from tree
    // Might be reinserted
  }

  // 6. dispose - Clean up resources
  @override
  void dispose() {
    // Cancel stream subscription
    _subscription.cancel();
    
    // Dispose animation controller
    _controller.dispose();
    
    super.dispose();
  }
}
```

What's happening here?
- initState sets up resources
- didChangeDependencies handles inherited changes
- build creates UI
- didUpdateWidget handles configuration changes
- deactivate handles temporary removal
- dispose cleans up resources

---

# Lifecycle with InheritedWidget

> **InheritedWidgets affect** the lifecycle.

```dart
// InheritedWidget lifecycle interaction
class MyInheritedData extends InheritedWidget {
  const MyInheritedData({
    super.key,
    required this.data,
    required super.child,
  });

  final String data;

  static MyInheritedData? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<MyInheritedData>();
  }

  @override
  bool updateShouldNotify(MyInheritedData oldWidget) {
    return data != oldWidget.data;
  }
}

class InheritedLifecycle extends StatefulWidget {
  const InheritedLifecycle({super.key});

  @override
  State<InheritedLifecycle> createState() => _InheritedLifecycleState();
}

class _InheritedLifecycleState extends State<InheritedLifecycle> {
  @override
  void initState() {
    super.initState();
    // At this point, no inherited widgets can be accessed
    // because context is not fully available
    // ❌ Theme.of(context) - Won't work here
    // ✅ Use didChangeDependencies instead
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // ✓ Safe to access inherited widgets here
    final inheritedData = MyInheritedData.of(context);
    
    // This will be called whenever inherited data changes
    if (inheritedData != null) {
      // Update state based on inherited data
      setState(() {
        // Update UI
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    // ✓ Safe to access inherited widgets in build
    final inheritedData = MyInheritedData.of(context);
    final theme = Theme.of(context);
    
    return Text(inheritedData?.data ?? 'No data');
  }
}

// Lifecycle with inherited widgets:
// 1. State is created
// 2. initState() runs
// 3. didChangeDependencies() runs (can access inherited)
// 4. build() runs (can access inherited)
// 5. When inherited data changes:
//    - didChangeDependencies() runs again
//    - build() runs again
```

What's happening here?
- Inherited widgets not available in initState
- didChangeDependencies can access inherited
- build can access inherited
- Inherited changes trigger rebuild

---

# Lifecycle and State Management

> **Lifecycle methods** are crucial for state management.

```dart
// State management with lifecycle
class StateManagementLifecycle extends StatefulWidget {
  const StateManagementLifecycle({super.key});

  @override
  State<StateManagementLifecycle> createState() => _StateManagementLifecycleState();
}

class _StateManagementLifecycleState extends State<StateManagementLifecycle> {
  // 1. State variables
  List<String> _items = [];
  bool _isLoading = false;
  String? _error;

  // 2. initState - Load initial data
  @override
  void initState() {
    super.initState();
    _loadData();
  }

  // 3. Load data asynchronously
  Future<void> _loadData() async {
    setState(() {
      _isLoading = true;
      _error = null;
    });

    try {
      // Simulate network request
      await Future.delayed(const Duration(seconds: 2));
      
      final data = await _fetchData();
      
      if (mounted) {
        setState(() {
          _items = data;
          _isLoading = false;
        });
      }
    } catch (e) {
      if (mounted) {
        setState(() {
          _error = e.toString();
          _isLoading = false;
        });
      }
    }
  }

  Future<List<String>> _fetchData() async {
    // Simulated API call
    return ['Item 1', 'Item 2', 'Item 3'];
  }

  // 4. didUpdateWidget - Handle parent changes
  @override
  void didUpdateWidget(covariant StateManagementLifecycle oldWidget) {
    super.didUpdateWidget(oldWidget);
    // Parent might have changed something
    // Reload data if needed
  }

  // 5. dispose - Clean up
  @override
  void dispose() {
    // Cancel any pending operations
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_error != null) {
      return Center(
        child: Column(
          children: [
            Text('Error: $_error'),
            ElevatedButton(
              onPressed: _loadData,
              child: const Text('Retry'),
            ),
          ],
        ),
      );
    }

    return ListView.builder(
      itemCount: _items.length,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text(_items[index]),
        );
      },
    );
  }
}
```

What's happening here?
- initState loads initial data
- Async operations check mounted
- State updates with setState
- Error handling in state
- Cleanup in dispose

---

# Lifecycle Best Practices

## Initialize in initState

```dart
// Good - Initialize in initState
@override
void initState() {
  super.initState();
  _controller = AnimationController(vsync: this);
  _subscription = stream.listen(_handleData);
  _loadInitialData();
}

// Bad - Initialize in build
@override
Widget build(BuildContext context) {
  // ❌ Don't initialize in build
  // Controller recreated every build
}
```

## Clean Up in dispose

```dart
// Good - Clean up resources
@override
void dispose() {
  _controller.dispose();
  _subscription.cancel();
  _timer?.cancel();
  super.dispose();
}

// Bad - Missing dispose
@override
void dispose() {
  // ❌ Missing cleanup
  super.dispose();
}
```

## Check mounted for Async

```dart
// Good - Check mounted
Future<void> _asyncOperation() async {
  await someAsync();
  if (mounted) {
    setState(() {
      // Update UI
    });
  }
}

// Bad - No mounted check
Future<void> _asyncOperation() async {
  await someAsync();
  // ❌ Might try to update after dispose
  setState(() {
    // Update UI
  });
}
```

---

# Common Mistakes

## Using Context in initState

Wrong:
```dart
@override
void initState() {
  super.initState();
  // ❌ Don't use context in initState
  final theme = Theme.of(context);
}
```

Correct:
```dart
@override
void didChangeDependencies() {
  super.didChangeDependencies();
  // ✓ Use context in didChangeDependencies
  final theme = Theme.of(context);
}
```

## Forgetting to Call super

Wrong:
```dart
@override
void initState() {
  // ❌ Missing super.initState()
  // Causes bugs
  _loadData();
}

@override
void dispose() {
  // ❌ Missing super.dispose()
  // Causes memory leaks
  _cleanup();
}
```

Correct:
```dart
@override
void initState() {
  super.initState();
  _loadData();
}

@override
void dispose() {
  _cleanup();
  super.dispose();
}
```

---

# Summary

Widget lifecycle manages widget creation, updates, and destruction. StatelessWidget has a simple lifecycle with constructor and build. StatefulWidget has a rich lifecycle with initState, didChangeDependencies, build, didUpdateWidget, deactivate, and dispose. Understanding the lifecycle helps manage resources and build performant apps.

---

# Next Steps

- [Widget Composition](widget-composition.md)
- [Widget Catalog](widget-catalog.md)
- [Constraints](../layout/constraints.md)

---

# Did You Know?

- initState runs only once
- didChangeDependencies runs after initState
- build can run many times
- didUpdateWidget runs when parent changes
- deactivate runs when removed from tree
- dispose runs permanently removing
- mounted checks if widget is in tree
- Use mounted for async operations