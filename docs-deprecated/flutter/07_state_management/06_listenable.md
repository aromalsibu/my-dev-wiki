# Listenable

Understand how to create custom observable objects with Listenable in Flutter.

---

# What is it?

Listenable is an abstract class that provides the foundation for change notification in Flutter. It represents an object that can have listeners which are notified when the object changes. Both ChangeNotifier and ValueNotifier extend Listenable. Understanding Listenable gives you the ability to create custom observable objects with fine-grained control over change notifications.

---

# Why does it exist?

Listenable exists to:

- Provide a foundation for reactive programming
- Enable custom observable objects
- Support change notification patterns
- Integrate with Flutter's widget system
- Allow efficient UI updates
- Create custom state management solutions
- Support listener-based architectures

---

# Listenable Basics

> **Understanding** Listenable and its usage.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// A custom Listenable implementation
/// This demonstrates how to create a custom observable object
class CounterListenable extends Listenable {
  // Private list of listeners
  // This tracks all the listeners that have been added
  final List<VoidCallback> _listeners = [];

  // Private state
  int _count = 0;

  // Getter for the count
  int get count => _count;

  // Method to increment the counter
  void increment() {
    _count++;
    // Notify all listeners about the change
    _notifyListeners();
  }

  // Method to decrement the counter
  void decrement() {
    _count--;
    _notifyListeners();
  }

  // Method to add a listener
  // This is required by the Listenable interface
  @override
  void addListener(VoidCallback listener) {
    _listeners.add(listener);
  }

  // Method to remove a listener
  // This is required by the Listenable interface
  @override
  void removeListener(VoidCallback listener) {
    _listeners.remove(listener);
  }

  // Helper method to notify all listeners
  void _notifyListeners() {
    // Notify each listener by calling their callback
    for (final listener in _listeners) {
      listener();
    }
  }
}

/// Application demonstrating custom Listenable usage
class ListenableExample extends StatefulWidget {
  const ListenableExample({super.key});

  @override
  State<ListenableExample> createState() => _ListenableExampleState();
}

class _ListenableExampleState extends State<ListenableExample> {
  // Create an instance of our custom Listenable
  final CounterListenable _counter = CounterListenable();

  @override
  void initState() {
    super.initState();
    
    // Add a listener to the counter
    // This listener will be called whenever the counter changes
    _counter.addListener(() {
      // This callback will be executed when _counter notifies listeners
      // We could perform side effects here
      print('Counter changed to: ${_counter.count}');
    });
  }

  @override
  void dispose() {
    // Clean up resources
    // Remove all listeners to prevent memory leaks
    // For this example, we'd need to track the listener reference
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Listenable Counter'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Use AnimatedBuilder to listen to the counter
            // AnimatedBuilder will rebuild whenever the counter changes
            AnimatedBuilder(
              animation: _counter,
              builder: (context, child) {
                return Text(
                  'Count: ${_counter.count}',
                  style: const TextStyle(fontSize: 24),
                );
              },
            ),
            const SizedBox(height: 16),
            
            // Control buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _counter.decrement,
                  child: const Text('-'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _counter.increment,
                  child: const Text('+'),
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
- CounterListenable implements the Listenable interface
- addListener and removeListener manage listeners
- _notifyListeners triggers all listeners
- AnimatedBuilder listens to Listenable objects
- Custom observable objects can be created

---

# Listenable vs ChangeNotifier

> **Comparing** Listenable with ChangeNotifier.

```dart
/// Custom Listenable implementation
/// More control over listener management
class CustomListenable extends Listenable {
  final List<VoidCallback> _listeners = [];
  int _value = 0;

  int get value => _value;

  void setValue(int newValue) {
    if (_value != newValue) {
      _value = newValue;
      _notifyListeners();
    }
  }

  @override
  void addListener(VoidCallback listener) {
    _listeners.add(listener);
  }

  @override
  void removeListener(VoidCallback listener) {
    _listeners.remove(listener);
  }

  void _notifyListeners() {
    for (final listener in _listeners) {
      listener();
    }
  }
}

/// Comparison of Listenable and ChangeNotifier
class ListenableComparison extends StatelessWidget {
  const ListenableComparison({super.key});

  @override
  Widget build(BuildContext context) {
    // Custom Listenable
    final customListenable = CustomListenable();
    
    // ChangeNotifier (pre-built)
    final changeNotifier = ChangeNotifier()..addListener(() {});
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Listenable Comparison'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Listenable implementation
            _buildListenableSection(customListenable),
            
            const SizedBox(height: 16),
            const Divider(),
            const SizedBox(height: 16),
            
            // ChangeNotifier implementation
            _buildChangeNotifierSection(changeNotifier),
            
            // Advantages of each approach
            const SizedBox(height: 24),
            const Card(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'Key Differences:',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    SizedBox(height: 8),
                    Text('• Listenable: Full control, manual listener management'),
                    Text('• ChangeNotifier: Built-in notifyListeners(), easier to use'),
                    Text('• Both: Support AnimatedBuilder and ListenableBuilder'),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildListenableSection(CustomListenable listenable) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            const Text(
              'Custom Listenable',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            AnimatedBuilder(
              animation: listenable,
              builder: (context, child) {
                return Text('Value: ${listenable.value}');
              },
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () => listenable.setValue(listenable.value - 1),
                  child: const Text('-'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () => listenable.setValue(listenable.value + 1),
                  child: const Text('+'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildChangeNotifierSection(ChangeNotifier notifier) {
    // Using a ValueNotifier as an example of ChangeNotifier
    final valueNotifier = ValueNotifier<int>(0);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            const Text(
              'ChangeNotifier (ValueNotifier)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            ValueListenableBuilder<int>(
              valueListenable: valueNotifier,
              builder: (context, value, child) {
                return Text('Value: $value');
              },
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () => valueNotifier.value--,
                  child: const Text('-'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () => valueNotifier.value++,
                  child: const Text('+'),
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
- CustomListenable: Manual listener management
- ChangeNotifier: Built-in notification system
- Both work with AnimatedBuilder
- Choose based on control vs convenience

---

# Advanced Listenable Usage

> **Advanced patterns** with Listenable.

```dart
/// Combined Listenable that listens to multiple sources
/// This demonstrates how to create complex observable objects
class CompositeListenable extends Listenable {
  final List<Listenable> _children;
  final List<VoidCallback> _listeners = [];
  bool _isListening = false;

  CompositeListenable(this._children) {
    _startListening();
  }

  void _startListening() {
    if (_isListening) return;
    _isListening = true;
    
    // Add a listener to each child
    for (final child in _children) {
      child.addListener(_notifyListeners);
    }
  }

  @override
  void addListener(VoidCallback listener) {
    _listeners.add(listener);
    if (!_isListening) {
      _startListening();
    }
  }

  @override
  void removeListener(VoidCallback listener) {
    _listeners.remove(listener);
    
    // If no listeners, stop listening to children
    if (_listeners.isEmpty && _isListening) {
      _stopListening();
    }
  }

  void _stopListening() {
    if (!_isListening) return;
    _isListening = false;
    
    for (final child in _children) {
      child.removeListener(_notifyListeners);
    }
  }

  void _notifyListeners() {
    for (final listener in _listeners) {
      listener();
    }
  }
}

/// Advanced Listenable usage example
class AdvancedListenableExample extends StatefulWidget {
  const AdvancedListenableExample({super.key});

  @override
  State<AdvancedListenableExample> createState() => _AdvancedListenableExampleState();
}

class _AdvancedListenableExampleState extends State<AdvancedListenableExample> {
  // Create two independent Listenables
  final ValueNotifier<int> _counter1 = ValueNotifier<int>(0);
  final ValueNotifier<int> _counter2 = ValueNotifier<int>(0);
  
  // Create a composite that combines both
  late CompositeListenable _combined;

  @override
  void initState() {
    super.initState();
    _combined = CompositeListenable([_counter1, _counter2]);
  }

  @override
  void dispose() {
    _combined.dispose();
    _counter1.dispose();
    _counter2.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Advanced Listenable'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // First counter
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('Counter 1:'),
                    Row(
                      children: [
                        IconButton(
                          icon: const Icon(Icons.remove),
                          onPressed: () => _counter1.value--,
                        ),
                        AnimatedBuilder(
                          animation: _counter1,
                          builder: (context, child) {
                            return Text('${_counter1.value}');
                          },
                        ),
                        IconButton(
                          icon: const Icon(Icons.add),
                          onPressed: () => _counter1.value++,
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            // Second counter
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('Counter 2:'),
                    Row(
                      children: [
                        IconButton(
                          icon: const Icon(Icons.remove),
                          onPressed: () => _counter2.value--,
                        ),
                        AnimatedBuilder(
                          animation: _counter2,
                          builder: (context, child) {
                            return Text('${_counter2.value}');
                          },
                        ),
                        IconButton(
                          icon: const Icon(Icons.add),
                          onPressed: () => _counter2.value++,
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            // Combined total (updates when either changes)
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text(
                      'Combined Total:',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    AnimatedBuilder(
                      animation: _combined,
                      builder: (context, child) {
                        return Text(
                          '${_counter1.value + _counter2.value}',
                          style: const TextStyle(
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        );
                      },
                    ),
                  ],
                ),
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
- CompositeListenable combines multiple Listenables
- Starts/stops listening based on listeners
- Efficient resource management
- Complex observable hierarchies

---

# Real-World Examples

> **Common patterns** with Listenable.

```dart
/// Form validation with Listenable
class FormValidationListenable extends Listenable {
  final List<ValueNotifier<bool>> _validators = [];
  final List<VoidCallback> _listeners = [];
  bool _isValid = false;

  bool get isValid => _isValid;

  void addValidator(ValueNotifier<bool> validator) {
    _validators.add(validator);
    validator.addListener(_validate);
    _validate(); // Initial validation
  }

  void _validate() {
    bool newValid = true;
    for (final validator in _validators) {
      if (!validator.value) {
        newValid = false;
        break;
      }
    }
    
    if (_isValid != newValid) {
      _isValid = newValid;
      _notifyListeners();
    }
  }

  @override
  void addListener(VoidCallback listener) {
    _listeners.add(listener);
  }

  @override
  void removeListener(VoidCallback listener) {
    _listeners.remove(listener);
  }

  void _notifyListeners() {
    for (final listener in _listeners) {
      listener();
    }
  }

  void dispose() {
    for (final validator in _validators) {
      validator.removeListener(_validate);
    }
  }
}

/// Listenable with debouncing
class DebouncedListenable extends Listenable {
  final Listenable _source;
  final Duration delay;
  final List<VoidCallback> _listeners = [];
  Timer? _timer;

  DebouncedListenable(this._source, this.delay) {
    _source.addListener(_onSourceChanged);
  }

  void _onSourceChanged() {
    // Cancel previous timer
    _timer?.cancel();
    // Wait for the delay before notifying
    _timer = Timer(delay, _notifyListeners);
  }

  @override
  void addListener(VoidCallback listener) {
    _listeners.add(listener);
  }

  @override
  void removeListener(VoidCallback listener) {
    _listeners.remove(listener);
  }

  void _notifyListeners() {
    for (final listener in _listeners) {
      listener();
    }
  }

  void dispose() {
    _source.removeListener(_onSourceChanged);
    _timer?.cancel();
  }
}
```

What's happening here?
- Form validation with multiple validators
- Debounced notifications for performance
- Custom listener management
- Real-world observable patterns

---

# Best Practices

## Implement Listener Management

```dart
// Good - Proper listener management
class MyListenable extends Listenable {
  final List<VoidCallback> _listeners = [];
  
  @override
  void addListener(VoidCallback listener) {
    _listeners.add(listener);
  }
  
  @override
  void removeListener(VoidCallback listener) {
    _listeners.remove(listener);
  }
}
```

## Notify Efficiently

```dart
// Good - Only notify when needed
void update(value) {
  if (_value != value) {
    _value = value;
    _notifyListeners();
  }
}

// Bad - Notify even without changes
void update(value) {
  _value = value;
  _notifyListeners();
}
```

## Clean Up Resources

```dart
// Good - Dispose properly
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  late MyListenable _listenable;
  
  @override
  void initState() {
    super.initState();
    _listenable = MyListenable();
    // Add listeners
  }
  
  @override
  void dispose() {
    // Remove listeners and dispose
    _listenable.dispose();
    super.dispose();
  }
}
```

---

# Common Mistakes

## Forgetting to Remove Listeners

Wrong:
```dart
void initState() {
  super.initState();
  listenable.addListener(_handleChange);
  // No removal in dispose
}
```

Correct:
```dart
void initState() {
  super.initState();
  listenable.addListener(_handleChange);
}

void dispose() {
  listenable.removeListener(_handleChange);
  super.dispose();
}
```

## Not Implementing RemoveListener

Wrong:
```dart
class MyListenable extends Listenable {
  @override
  void addListener(VoidCallback listener) {}
  // Missing removeListener
}
```

Correct:
```dart
class MyListenable extends Listenable {
  final List<VoidCallback> _listeners = [];
  
  @override
  void addListener(VoidCallback listener) {
    _listeners.add(listener);
  }
  
  @override
  void removeListener(VoidCallback listener) {
    _listeners.remove(listener);
  }
}
```

---

# Summary

Listenable provides the foundation for reactive programming in Flutter. It enables custom observable objects with listener-based change notification. Use Listenable for custom reactive solutions, advanced patterns, and when you need full control over notification behavior.

---

# Next Steps

- [Restoration](restoration.md)
- [State Management Overview](state-management-overview.md)
- [Provider](provider.md)

---

# Did You Know?

- Listenable is the base class for ChangeNotifier
- AnimatedBuilder works with any Listenable
- Listenable supports multiple listeners
- Listeners can be added and removed
- Listenable enables custom observable objects
- Listenable is lightweight
- Listenable can be combined
- Listenable supports efficient UI updates