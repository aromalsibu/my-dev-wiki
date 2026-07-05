# StatefulWidget

Understand widgets that have mutable state and can rebuild dynamically.

---

# What is it?

A StatefulWidget is a widget that describes part of the user interface that can change over time. Unlike StatelessWidget, a StatefulWidget has a separate State object that holds mutable data and can trigger rebuilds when that data changes. This allows the UI to respond to user interactions, animations, network responses, and other dynamic events.

---

# Why does it exist?

StatefulWidget exists to:

- Handle mutable state in Flutter applications
- Update the UI in response to user interactions
- Manage animations and transitions
- Handle form inputs and validation
- React to network responses and data changes
- Manage complex widget lifecycle
- Enable dynamic and interactive UIs

---

# StatefulWidget Structure

> **StatefulWidget** has two parts: the widget and its state.

```dart
// StatefulWidget structure
class MyStatefulWidget extends StatefulWidget {
  // 1. Widget configuration (immutable)
  final String title;
  final Color color;

  const MyStatefulWidget({
    super.key,
    required this.title,
    this.color = Colors.blue,
  });

  // 2. createState() creates the State object
  @override
  State<MyStatefulWidget> createState() => _MyStatefulWidgetState();
}

// 3. State class (mutable)
class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  // State variables (mutable)
  int _counter = 0;
  bool _isActive = false;

  // Methods that modify state
  void _incrementCounter() {
    setState(() {
      _counter++; // Triggers rebuild
    });
  }

  void _toggleActive() {
    setState(() {
      _isActive = !_isActive;
    });
  }

  // 4. build() method
  @override
  Widget build(BuildContext context) {
    // Access widget properties: widget.title
    // Access state: _counter, _isActive
    return Container(
      color: widget.color,
      child: Column(
        children: [
          Text('Title: ${widget.title}'),
          Text('Counter: $_counter'),
          Text('Active: $_isActive'),
          ElevatedButton(
            onPressed: _incrementCounter,
            child: const Text('Increment'),
          ),
          ElevatedButton(
            onPressed: _toggleActive,
            child: const Text('Toggle'),
          ),
        ],
      ),
    );
  }
}

// Relationship:
// MyStatefulWidget (immutable configuration)
//     ↕
// _MyStatefulWidgetState (mutable state)
//     ↕
// Widget tree (rebuilt when state changes)
```

What's happening here?
- Widget holds immutable configuration
- State holds mutable data
- setState() triggers rebuilds
- Access widget properties via widget.
- State persists across rebuilds

---

# State Lifecycle

> **StatefulWidget has a rich lifecycle** with multiple stages.

```dart
// State lifecycle demonstration
class LifecycleDemo extends StatefulWidget {
  const LifecycleDemo({super.key, required this.initialValue});

  final String initialValue;

  @override
  State<LifecycleDemo> createState() => _LifecycleDemoState();
}

class _LifecycleDemoState extends State<LifecycleDemo> {
  String _value = '';
  
  // 1. Constructor
  _LifecycleDemoState() {
    print('1. State constructor');
  }

  // 2. initState() - Called once when state is created
  @override
  void initState() {
    super.initState();
    print('2. initState()');
    _value = widget.initialValue;
    // Initialize data, start animations, subscribe to streams
  }

  // 3. didChangeDependencies() - Called when dependencies change
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print('3. didChangeDependencies()');
    // Called after initState
    // Called when inherited widget changes
    // Called when dependencies change
  }

  // 4. build() - Called multiple times
  @override
  Widget build(BuildContext context) {
    print('4. build()');
    return Text(_value);
  }

  // 5. setState() - Triggers rebuild (called manually)
  void _updateValue(String newValue) {
    setState(() {
      print('5. setState()');
      _value = newValue;
    });
  }

  // 6. didUpdateWidget() - Called when widget updates
  @override
  void didUpdateWidget(covariant LifecycleDemo oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('6. didUpdateWidget()');
    // Called when parent rebuilds with new configuration
    // Compare old and new widget properties
    if (widget.initialValue != oldWidget.initialValue) {
      _value = widget.initialValue;
    }
  }

  // 7. deactivate() - Called when removed from tree
  @override
  void deactivate() {
    super.deactivate();
    print('7. deactivate()');
    // Called when widget is removed
    // Can be reinserted
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

  // 9. mounted property - Check if widget is in tree
  void _checkMounted() {
    if (mounted) {
      print('Widget is in tree');
    }
  }
}

// Lifecycle flow:
// 1. Constructor
// 2. initState() (once)
// 3. didChangeDependencies() (once)
// 4. build() (first time)
// 5. setState() (multiple times)
// 6. build() (repeated)
// 7. didUpdateWidget() (when parent changes)
// 8. deactivate() (removed from tree)
// 9. dispose() (permanently removed)
```

What's happening here?
- initState: one-time initialization
- setState: manual rebuild trigger
- didUpdateWidget: handle parent changes
- dispose: cleanup resources
- mounted: check if widget is active

---

# setState Method

> **setState()** is how you trigger UI updates.

```dart
// setState examples
class SetStateExample extends StatefulWidget {
  const SetStateExample({super.key});

  @override
  State<SetStateExample> createState() => _SetStateExampleState();
}

class _SetStateExampleState extends State<SetStateExample> {
  int _counter = 0;
  String _text = 'Hello';
  bool _isVisible = true;

  // 1. Basic setState
  void _incrementCounter() {
    setState(() {
      _counter++; // Update state
    });
    // build() is called automatically
  }

  // 2. Multiple updates in one setState
  void _updateMultiple() {
    setState(() {
      _counter++;
      _text = 'Updated';
      _isVisible = !_isVisible;
    });
    // Single rebuild for all updates
  }

  // 3. setState with heavy computation
  void _heavyUpdate() {
    // Heavy computation outside setState
    final result = _heavyComputation();
    
    // Only the UI update in setState
    setState(() {
      _counter = result;
    });
  }

  int _heavyComputation() {
    // Simulate heavy work
    return 42;
  }

  // 4. setState called from callbacks
  void _onButtonPressed() {
    setState(() {
      _counter++;
    });
  }

  // 5. setState with condition
  void _conditionalUpdate() {
    if (_counter < 10) {
      setState(() {
        _counter++;
      });
    }
  }

  // 6. setState after async operations
  Future<void> _asyncUpdate() async {
    // Async operation
    await Future.delayed(const Duration(seconds: 1));
    
    // Update state after async
    if (mounted) {
      setState(() {
        _counter++;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Counter: $_counter'),
        Text('Text: $_text'),
        if (_isVisible) Text('Visible'),
        ElevatedButton(
          onPressed: _incrementCounter,
          child: const Text('Increment'),
        ),
        ElevatedButton(
          onPressed: _updateMultiple,
          child: const Text('Update Multiple'),
        ),
        ElevatedButton(
          onPressed: _asyncUpdate,
          child: const Text('Async Update'),
        ),
      ],
    );
  }
}

// setState rules:
// 1. Only call inside State class
// 2. Never call after dispose()
// 3. Keep updates minimal
// 4. Don't call sync in build()
// 5. Check mounted for async
```

What's happening here?
- setState marks widget as dirty
- UI rebuilds after setState
- Multiple updates can be batched
- Heavy work outside setState
- Check mounted for async

---

# Accessing Widget Properties

> **Access widget properties** using the `widget` property.

```dart
// Accessing widget configuration
class ConfigAccessWidget extends StatefulWidget {
  const ConfigAccessWidget({
    super.key,
    required this.title,
    required this.initialValue,
    this.maxCount = 10,
  });

  final String title;
  final int initialValue;
  final int maxCount;

  @override
  State<ConfigAccessWidget> createState() => _ConfigAccessWidgetState();
}

class _ConfigAccessWidgetState extends State<ConfigAccessWidget> {
  int _count = 0;

  @override
  void initState() {
    super.initState();
    // Access widget properties in initState
    _count = widget.initialValue;
  }

  void _increment() {
    setState(() {
      // Check against widget property
      if (_count < widget.maxCount) {
        _count++;
      }
    });
  }

  @override
  void didUpdateWidget(covariant ConfigAccessWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    // Handle widget property changes
    if (widget.initialValue != oldWidget.initialValue) {
      setState(() {
        _count = widget.initialValue;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Access title from widget
        Text('Title: ${widget.title}'),
        Text('Count: $_count'),
        Text('Max: ${widget.maxCount}'),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

What's happening here?
- widget provides access to configuration
- Available in all lifecycle methods
- Use in initState for initialization
- Use in didUpdateWidget for changes
- Properties are immutable

---

# State Management Patterns

> **Common patterns** for managing state in StatefulWidget.

```dart
// 1. Single source of truth
class SingleSourceState extends StatefulWidget {
  const SingleSourceState({super.key});

  @override
  State<SingleSourceState> createState() => _SingleSourceStateState();
}

class _SingleSourceStateState extends State<SingleSourceState> {
  // Single source of truth for all data
  class AppState {
    final int count;
    final String name;
    final bool isActive;
    
    const AppState({
      required this.count,
      required this.name,
      required this.isActive,
    });
  }

  AppState _state = const AppState(
    count: 0,
    name: 'User',
    isActive: false,
  );

  void _updateState() {
    setState(() {
      // Update state atomically
      _state = AppState(
        count: _state.count + 1,
        name: _state.name,
        isActive: !_state.isActive,
      );
    });
  }

  @override
  Widget build(BuildContext context) {
    return Text('Count: ${_state.count}');
  }
}

// 2. Controller pattern
class ControllerState extends StatefulWidget {
  const ControllerState({super.key});

  @override
  State<ControllerState> createState() => _ControllerStateState();
}

class _ControllerStateState extends State<ControllerState> {
  late TextEditingController _controller;
  
  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
    _controller.addListener(_onTextChanged);
  }

  void _onTextChanged() {
    setState(() {
      // Update UI on text change
    });
  }

  @override
  void dispose() {
    _controller.removeListener(_onTextChanged);
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
    );
  }
}

// 3. State lifting
class LiftedState extends StatefulWidget {
  const LiftedState({super.key});

  @override
  State<LiftedState> createState() => _LiftedStateState();
}

class _LiftedStateState extends State<LiftedState> {
  int _sharedCounter = 0;

  void _incrementShared() {
    setState(() {
      _sharedCounter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // State shared between children
        CounterWidget(
          count: _sharedCounter,
          onIncrement: _incrementShared,
        ),
        AnotherCounterWidget(
          count: _sharedCounter,
          onIncrement: _incrementShared,
        ),
      ],
    );
  }
}

class CounterWidget extends StatelessWidget {
  const CounterWidget({
    super.key,
    required this.count,
    required this.onIncrement,
  });

  final int count;
  final VoidCallback onIncrement;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: onIncrement,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

What's happening here?
- Single source of truth
- Controller pattern for forms
- State lifting for sharing
- Different patterns for different needs

---

# Real-World Examples

> **Common use cases** for StatefulWidget.

```dart
// 1. Form with validation
class FormWidget extends StatefulWidget {
  const FormWidget({super.key});

  @override
  State<FormWidget> createState() => _FormWidgetState();
}

class _FormWidgetState extends State<FormWidget> {
  final _formKey = GlobalKey<FormState>();
  String _name = '';
  String _email = '';
  bool _isValid = false;

  void _validateAndSubmit() {
    if (_formKey.currentState!.validate()) {
      _formKey.currentState!.save();
      setState(() {
        _isValid = true;
      });
      // Submit data
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            decoration: const InputDecoration(labelText: 'Name'),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Please enter your name';
              }
              return null;
            },
            onSaved: (value) => _name = value!,
          ),
          TextFormField(
            decoration: const InputDecoration(labelText: 'Email'),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Please enter your email';
              }
              if (!value.contains('@')) {
                return 'Please enter a valid email';
              }
              return null;
            },
            onSaved: (value) => _email = value!,
          ),
          if (_isValid)
            const Text('Form is valid!', style: TextStyle(color: Colors.green)),
          ElevatedButton(
            onPressed: _validateAndSubmit,
            child: const Text('Submit'),
          ),
        ],
      ),
    );
  }
}

// 2. Timer with state
class TimerWidget extends StatefulWidget {
  const TimerWidget({super.key});

  @override
  State<TimerWidget> createState() => _TimerWidgetState();
}

class _TimerWidgetState extends State<TimerWidget> {
  int _seconds = 0;
  bool _isRunning = false;
  Timer? _timer;

  void _startTimer() {
    setState(() {
      _isRunning = true;
    });
    _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
      setState(() {
        _seconds++;
      });
    });
  }

  void _stopTimer() {
    _timer?.cancel();
    setState(() {
      _isRunning = false;
    });
  }

  void _resetTimer() {
    _timer?.cancel();
    setState(() {
      _seconds = 0;
      _isRunning = false;
    });
  }

  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          '${_seconds ~/ 60}:${(_seconds % 60).toString().padLeft(2, '0')}',
          style: const TextStyle(fontSize: 48),
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            if (!_isRunning)
              ElevatedButton(
                onPressed: _startTimer,
                child: const Text('Start'),
              ),
            if (_isRunning)
              ElevatedButton(
                onPressed: _stopTimer,
                child: const Text('Stop'),
              ),
            ElevatedButton(
              onPressed: _resetTimer,
              child: const Text('Reset'),
            ),
          ],
        ),
      ],
    );
  }
}

// 3. Toggle widget
class ToggleWidget extends StatefulWidget {
  const ToggleWidget({super.key});

  @override
  State<ToggleWidget> createState() => _ToggleWidgetState();
}

class _ToggleWidgetState extends State<ToggleWidget> {
  bool _isOn = false;

  void _toggle() {
    setState(() {
      _isOn = !_isOn;
    });
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _toggle,
      child: Container(
        width: 60,
        height: 30,
        decoration: BoxDecoration(
          color: _isOn ? Colors.green : Colors.grey,
          borderRadius: BorderRadius.circular(15),
        ),
        child: Align(
          alignment: _isOn ? Alignment.centerRight : Alignment.centerLeft,
          child: Container(
            width: 26,
            height: 26,
            margin: const EdgeInsets.all(2),
            decoration: const BoxDecoration(
              color: Colors.white,
              shape: BoxShape.circle,
            ),
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Form with validation
- Timer with start/stop/reset
- Toggle switch with animation
- Real-world interactive widgets

---

# Best Practices

## Keep State Minimal

```dart
// Good - Minimal state
class MinimalState extends StatefulWidget {
  const MinimalState({super.key});

  @override
  State<MinimalState> createState() => _MinimalStateState();
}

class _MinimalStateState extends State<MinimalState> {
  int _counter = 0; // Only what's needed

  @override
  Widget build(BuildContext context) {
    return Text('Counter: $_counter');
  }
}

// Bad - Too much state
class TooMuchState extends StatefulWidget {
  const TooMuchState({super.key});

  @override
  State<TooMuchState> createState() => _TooMuchStateState();
}

class _TooMuchStateState extends State<TooMuchState> {
  int _counter = 0;
  String _temp1 = '';
  String _temp2 = '';
  bool _flag1 = false;
  bool _flag2 = false;
  // Too many state variables
}
```

## Dispose Resources

```dart
// Good - Proper disposal
@override
void initState() {
  super.initState();
  _subscription = stream.listen(_handleData);
  _controller = AnimationController(vsync: this);
}

@override
void dispose() {
  _subscription.cancel();
  _controller.dispose();
  super.dispose();
}
```

## Use setState Correctly

```dart
// Good - Proper setState
void _update() {
  // Heavy work outside
  final result = _calculate();
  
  setState(() {
    _value = result;
  });
}

// Bad - Heavy work in setState
void _update() {
  setState(() {
    _value = _calculate(); // Heavy work inside
  });
}
```

---

# Common Mistakes

## Not Checking Mounted

Wrong:
```dart
Future<void> _asyncUpdate() async {
  await someAsync();
  setState(() {
    _counter++;
  });
}
```

Correct:
```dart
Future<void> _asyncUpdate() async {
  await someAsync();
  if (mounted) {
    setState(() {
      _counter++;
    });
  }
}
```

## Forgetting to Dispose

Wrong:
```dart
class _MyState extends State<MyWidget> {
  late StreamSubscription _subscription;
  
  @override
  void initState() {
    super.initState();
    _subscription = stream.listen((data) {
      setState(() {});
    });
  }
  // No dispose - memory leak!
}
```

Correct:
```dart
class _MyState extends State<MyWidget> {
  late StreamSubscription _subscription;
  
  @override
  void initState() {
    super.initState();
    _subscription = stream.listen((data) {
      if (mounted) {
        setState(() {});
      }
    });
  }
  
  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}
```

---

# Summary

StatefulWidget provides mutable state and dynamic UI updates. It has a rich lifecycle with methods for initialization, updates, and cleanup. Use setState to trigger rebuilds, access widget properties via `widget`, and always dispose resources properly.

---

# Next Steps

- [BuildContext](buildcontext.md)
- [Keys](keys.md)
- [Widget Lifecycle](widget-lifecycle.md)

---

# Did You Know?

- StatefulWidget has 9 lifecycle methods
- State objects persist across rebuilds
- setState triggers a rebuild of the entire subtree
- mounted helps prevent errors in async operations
- State can be lifted to parent widgets
- StatefulWidgets are used in most interactive UIs
- AnimationController should be disposed
- The State object is created by createState()