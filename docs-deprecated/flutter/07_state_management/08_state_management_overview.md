# State Management Overview

Understand the different state management approaches in Flutter.

---

# What is it?

State management is the way you manage and share data across your Flutter application. It determines how data flows through your app, how widgets react to changes, and how different parts of your app communicate. Flutter offers multiple state management solutions, from simple built-in approaches to more complex third-party packages.

---

# Why does it exist?

State management exists to:

- Manage application data efficiently
- Keep UI in sync with data changes
- Share data between widgets
- Maintain separation of concerns
- Handle complex application logic
- Improve code maintainability
- Enable scalable app development

---

# State Management Approaches

> **Overview of different** state management solutions.

```dart
/// 1. setState - Local state management
/// Best for: Simple widgets, local UI state
class SetStateExample extends StatefulWidget {
  const SetStateExample({super.key});

  @override
  State<SetStateExample> createState() => _SetStateExampleState();
}

class _SetStateExampleState extends State<SetStateExample> {
  // Local state variable
  int _counter = 0;

  void _incrementCounter() {
    // Update state and rebuild the widget
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('setState Example'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Counter:',
              style: TextStyle(fontSize: 20),
            ),
            Text(
              '$_counter',
              style: const TextStyle(fontSize: 40, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            
            // Button to increment counter
            ElevatedButton(
              onPressed: _incrementCounter,
              child: const Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }
}

/// 2. Provider - Dependency Injection
/// Best for: Medium to large apps, sharing data between widgets
class ProviderExample extends StatelessWidget {
  const ProviderExample({super.key});

  @override
  Widget build(BuildContext context) {
    // Provide the CounterModel to the widget tree
    return ChangeNotifierProvider(
      create: (_) => CounterModel(),
      child: MaterialApp(
        home: const ProviderScreen(),
      ),
    );
  }
}

/// Counter model that extends ChangeNotifier
/// This class manages the counter state and notifies listeners
class CounterModel extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    // Notify listeners to rebuild UI
    notifyListeners();
  }

  void decrement() {
    _count--;
    notifyListeners();
  }
}

/// Screen that uses the CounterModel
class ProviderScreen extends StatelessWidget {
  const ProviderScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Access the CounterModel using Provider.of
    final counter = Provider.of<CounterModel>(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Provider Example'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Counter:',
              style: TextStyle(fontSize: 20),
            ),
            Text(
              '${counter.count}',
              style: const TextStyle(fontSize: 40, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Decrement button
                ElevatedButton(
                  onPressed: counter.decrement,
                  child: const Text('-'),
                ),
                const SizedBox(width: 8),
                // Increment button
                ElevatedButton(
                  onPressed: counter.increment,
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

/// 3. ValueNotifier - Simple Reactive State
/// Best for: Simple values, primitive types
class ValueNotifierExample extends StatelessWidget {
  const ValueNotifierExample({super.key});

  @override
  Widget build(BuildContext context) {
    // Create a ValueNotifier with initial value
    final counter = ValueNotifier<int>(0);

    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('ValueNotifier Example'),
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Listen to ValueNotifier changes
              ValueListenableBuilder<int>(
                valueListenable: counter,
                builder: (context, value, child) {
                  return Column(
                    children: [
                      const Text(
                        'Counter:',
                        style: TextStyle(fontSize: 20),
                      ),
                      Text(
                        '$value',
                        style: const TextStyle(
                          fontSize: 40,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ],
                  );
                },
              ),
              const SizedBox(height: 16),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  ElevatedButton(
                    // Decrement the value
                    onPressed: () => counter.value--,
                    child: const Text('-'),
                  ),
                  const SizedBox(width: 8),
                  ElevatedButton(
                    // Increment the value
                    onPressed: () => counter.value++,
                    child: const Text('+'),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}

/// 4. InheritedWidget - Data Sharing
/// Best for: App-wide data, themes, settings
class InheritedWidgetExample extends StatelessWidget {
  const InheritedWidgetExample({super.key});

  @override
  Widget build(BuildContext context) {
    // Provide data using CustomInheritedWidget
    return CustomInheritedWidget(
      data: 'Shared Data',
      child: MaterialApp(
        home: const InheritedScreen(),
      ),
    );
  }
}

/// Custom InheritedWidget that shares data down the tree
class CustomInheritedWidget extends InheritedWidget {
  const CustomInheritedWidget({
    super.key,
    required this.data,
    required super.child,
  });

  final String data;

  // Static method to access the data
  // This is how child widgets access the shared data
  static CustomInheritedWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<CustomInheritedWidget>();
  }

  @override
  bool updateShouldNotify(CustomInheritedWidget oldWidget) {
    // Only rebuild if data has changed
    return data != oldWidget.data;
  }
}

/// Screen that accesses data from InheritedWidget
class InheritedScreen extends StatelessWidget {
  const InheritedScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Get the data from the InheritedWidget
    final data = CustomInheritedWidget.of(context)?.data ?? 'No data';

    return Scaffold(
      appBar: AppBar(
        title: const Text('InheritedWidget Example'),
      ),
      body: Center(
        child: Text(
          'Data: $data',
          style: const TextStyle(fontSize: 24),
        ),
      ),
    );
  }
}
```

What's happening here?
- setState: Simple local state
- Provider: Dependency injection with ChangeNotifier
- ValueNotifier: Simple reactive values
- InheritedWidget: Data sharing down the tree

---

# State Management Comparison

> **Comparing different** approaches.

```dart
/// Comparison widget showing different approaches
class StateManagementComparison extends StatelessWidget {
  const StateManagementComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('State Management Comparison'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Comparison table
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'State Management Options:',
                      style: TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 16),
                    
                    _buildComparisonRow(
                      'setState',
                      'Simple local state',
                      'Single widget',
                      'Beginner',
                      'High',
                    ),
                    _buildComparisonRow(
                      'ValueNotifier',
                      'Simple reactive values',
                      'Single or few widgets',
                      'Beginner',
                      'High',
                    ),
                    _buildComparisonRow(
                      'InheritedWidget',
                      'Data sharing down tree',
                      'App-wide data',
                      'Intermediate',
                      'Medium',
                    ),
                    _buildComparisonRow(
                      'Provider',
                      'DI with ChangeNotifier',
                      'Medium to large apps',
                      'Intermediate',
                      'Medium',
                    ),
                    _buildComparisonRow(
                      'Riverpod',
                      'Modern reactive state',
                      'Complex applications',
                      'Intermediate',
                      'Low',
                    ),
                    _buildComparisonRow(
                      'BLoC',
                      'Business logic component',
                      'Large enterprise apps',
                      'Advanced',
                      'Low',
                    ),
                  ],
                ),
              ),
            ),
            
            const Spacer(),
            
            // Recommendation
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
                border: Border.all(color: Colors.blue[200]!),
              ),
              child: const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Recommendation:',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  SizedBox(height: 8),
                  Text('• Start with setState for simple widgets'),
                  Text('• Use Provider for medium to large apps'),
                  Text('• Consider Riverpod for new projects'),
                  Text('• Choose based on app complexity'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  /// Builds a row for the comparison table
  Widget _buildComparisonRow(
    String name,
    String description,
    String useCase,
    String difficulty,
    String boilerplate,
  ) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        children: [
          // Name
          SizedBox(
            width: 100,
            child: Text(
              name,
              style: const TextStyle(
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          // Description
          Expanded(
            flex: 2,
            child: Text(
              description,
              style: const TextStyle(fontSize: 12),
            ),
          ),
          // Use case
          Expanded(
            flex: 1,
            child: Text(
              useCase,
              style: const TextStyle(fontSize: 12),
            ),
          ),
          // Difficulty
          SizedBox(
            width: 80,
            child: Text(
              difficulty,
              style: TextStyle(
                fontSize: 12,
                color: _getDifficultyColor(difficulty),
              ),
            ),
          ),
          // Boilerplate
          SizedBox(
            width: 80,
            child: Text(
              boilerplate,
              style: TextStyle(
                fontSize: 12,
                color: _getBoilerplateColor(boilerplate),
              ),
            ),
          ),
        ],
      ),
    );
  }

  Color _getDifficultyColor(String difficulty) {
    switch (difficulty) {
      case 'Beginner':
        return Colors.green;
      case 'Intermediate':
        return Colors.orange;
      case 'Advanced':
        return Colors.red;
      default:
        return Colors.black;
    }
  }

  Color _getBoilerplateColor(String boilerplate) {
    switch (boilerplate) {
      case 'High':
        return Colors.green;
      case 'Medium':
        return Colors.orange;
      case 'Low':
        return Colors.red;
      default:
        return Colors.black;
    }
  }
}
```

What's happening here?
- Each approach has different use cases
- Trade-offs between simplicity and flexibility
- Choose based on app complexity
- Consider team experience

---

# State Management Decision Flow

> **How to choose** the right approach.

```dart
/// Decision flow widget
class StateManagementDecisionFlow extends StatelessWidget {
  const StateManagementDecisionFlow({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Decision Flow'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Decision tree
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'How to Choose:',
                      style: TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 16),
                    
                    _buildDecisionStep(
                      'Is the state local to one widget?',
                      'Use setState() - It\'s simple and built-in',
                      Colors.green,
                    ),
                    const SizedBox(height: 12),
                    
                    _buildDecisionStep(
                      'Is it a simple value (int, bool, String)?',
                      'Use ValueNotifier with ValueListenableBuilder',
                      Colors.blue,
                    ),
                    const SizedBox(height: 12),
                    
                    _buildDecisionStep(
                      'Do you need to share data between widgets?',
                      'Use Provider (medium apps) or Riverpod (new apps)',
                      Colors.orange,
                    ),
                    const SizedBox(height: 12),
                    
                    _buildDecisionStep(
                      'Is this a large, complex application?',
                      'Use Riverpod or BLoC for enterprise apps',
                      Colors.red,
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // Best practices
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.green[50],
                borderRadius: BorderRadius.circular(8),
                border: Border.all(color: Colors.green[200]!),
              ),
              child: const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Best Practices:',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  SizedBox(height: 8),
                  Text('• Keep state management consistent'),
                  Text('• Separate business logic from UI'),
                  Text('• Use immutable data when possible'),
                  Text('• Test state logic independently'),
                  Text('• Consider app complexity before choosing'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildDecisionStep(String question, String answer, Color color) {
    return Container(
      padding: const EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: color.withOpacity(0.1),
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: color.withOpacity(0.3)),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            children: [
              Icon(
                Icons.help_outline,
                color: color,
                size: 20,
              ),
              const SizedBox(width: 8),
              Expanded(
                child: Text(
                  question,
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ],
          ),
          const SizedBox(height: 4),
          Row(
            children: [
              Icon(
                Icons.arrow_forward,
                color: color,
                size: 16,
              ),
              const SizedBox(width: 8),
              Expanded(
                child: Text(
                  answer,
                  style: TextStyle(
                    color: color,
                    fontSize: 14,
                  ),
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Decision flow for choosing approach
- Consider state scope and complexity
- Match solution to needs
- Follow best practices

---

# Summary

Flutter offers multiple state management approaches for different needs. Choose based on app complexity, team experience, and specific requirements. Start simple with setState and ValueNotifier, scale up to Provider or Riverpod for larger apps.

---

# Next Steps

- [Provider](provider.md)
- [Riverpod](riverpod.md)
- [BLoC](bloc.md)

---

# Did You Know?

- setState is built into Flutter
- Provider is the most popular solution
- Riverpod is the modern alternative
- BLoC is great for complex business logic
- Each approach has trade-offs
- Choose based on app complexity
- Test state logic independently
- Keep state management consistent