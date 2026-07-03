# Architecture Patterns

Understand common architecture patterns used in Flutter applications for better organization, scalability, and maintainability.

---

# What is it?

Architecture Patterns are proven solutions to common problems in software design. They provide a structured approach to organizing your code, separating concerns, and managing complexity. In Flutter, architecture patterns help you structure your application in a way that is scalable, maintainable, and testable.

---

# Why does it exist?

Architecture Patterns exist to:

- Provide structured code organization
- Separate concerns effectively
- Improve code maintainability
- Enable scalability
- Support testing
- Guide team development
- Reduce complexity

---

# MVC Pattern

> **Model-View-Controller** architecture.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// 1. Model - Data and business logic
class CounterModel {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
  }

  void decrement() {
    _count--;
  }

  void reset() {
    _count = 0;
  }
}

/// 2. Controller - Mediates between Model and View
class CounterController {
  final CounterModel _model;

  CounterController(this._model);

  int get count => _model.count;

  void increment() {
    _model.increment();
  }

  void decrement() {
    _model.decrement();
  }

  void reset() {
    _model.reset();
  }
}

/// 3. View - UI presentation
class CounterView extends StatefulWidget {
  final CounterController controller;

  const CounterView({super.key, required this.controller});

  @override
  State<CounterView> createState() => _CounterViewState();
}

class _CounterViewState extends State<CounterView> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('MVC Counter'),
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
              '${widget.controller.count}',
              style: const TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      widget.controller.decrement();
                    });
                  },
                  child: const Text('-'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      widget.controller.increment();
                    });
                  },
                  child: const Text('+'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      widget.controller.reset();
                    });
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

/// 4. Usage
void main() {
  final model = CounterModel();
  final controller = CounterController(model);
  runApp(MaterialApp(
    home: CounterView(controller: controller),
  ));
}
```

What's happening here?
- Model: Data and business logic
- View: UI presentation
- Controller: Mediates between Model and View
- Clear separation of concerns

---

# MVVM Pattern

> **Model-View-ViewModel** architecture.

```dart
/// 1. Model - Data and business logic
class UserModel {
  final String id;
  final String name;
  final String email;

  UserModel({
    required this.id,
    required this.name,
    required this.email,
  });

  // Validation
  bool isValid() {
    return name.isNotEmpty && email.contains('@');
  }
}

/// 2. ViewModel - Exposes data and commands to View
class UserViewModel extends ChangeNotifier {
  UserModel? _user;
  bool _isLoading = false;
  String _error = '';

  UserModel? get user => _user;
  bool get isLoading => _isLoading;
  String get error => _error;
  bool get hasError => _error.isNotEmpty;

  // Commands
  Future<void> loadUser(String id) async {
    _setLoading(true);
    _clearError();

    try {
      // Simulate API call
      await Future.delayed(const Duration(seconds: 1));
      _user = UserModel(
        id: id,
        name: 'John Doe',
        email: 'john@example.com',
      );
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }

  Future<void> updateUser(String name, String email) async {
    if (_user == null) return;

    _setLoading(true);
    _clearError();

    try {
      // Simulate API call
      await Future.delayed(const Duration(seconds: 1));
      _user = UserModel(
        id: _user!.id,
        name: name,
        email: email,
      );
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }

  void _setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners();
  }

  void _setError(String error) {
    _error = error;
    notifyListeners();
  }

  void _clearError() {
    _error = '';
    notifyListeners();
  }
}

/// 3. View - UI presentation
class UserView extends StatelessWidget {
  const UserView({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => UserViewModel()..loadUser('1'),
      child: Scaffold(
        appBar: AppBar(
          title: const Text('MVVM User'),
        ),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Consumer<UserViewModel>(
            builder: (context, viewModel, _) {
              if (viewModel.isLoading) {
                return const Center(child: CircularProgressIndicator());
              }

              if (viewModel.hasError) {
                return Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Icon(Icons.error, color: Colors.red, size: 48),
                      const SizedBox(height: 16),
                      Text(viewModel.error),
                      const SizedBox(height: 16),
                      ElevatedButton(
                        onPressed: () => viewModel.loadUser('1'),
                        child: const Text('Retry'),
                      ),
                    ],
                  ),
                );
              }

              final user = viewModel.user;
              if (user == null) {
                return const Center(child: Text('No user data'));
              }

              return Column(
                children: [
                  _buildUserInfo(user),
                  const SizedBox(height: 16),
                  _buildUpdateButton(viewModel),
                ],
              );
            },
          ),
        ),
      ),
    );
  }

  Widget _buildUserInfo(UserModel user) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('ID: ${user.id}'),
            Text('Name: ${user.name}'),
            Text('Email: ${user.email}'),
          ],
        ),
      ),
    );
  }

  Widget _buildUpdateButton(UserViewModel viewModel) {
    return SizedBox(
      width: double.infinity,
      child: ElevatedButton(
        onPressed: () {
          viewModel.updateUser('Jane Doe', 'jane@example.com');
        },
        child: const Text('Update User'),
      ),
    );
  }
}

/// 4. Usage
void main() {
  runApp(const MaterialApp(
    home: UserView(),
  ));
}
```

What's happening here?
- Model: Data and logic
- ViewModel: Exposes data and commands
- View: UI presentation
- State management with Provider

---

# BLoC Pattern

> **Business Logic Component** architecture.

```dart
/// 1. Events
abstract class CounterEvent {}

class IncrementEvent extends CounterEvent {}

class DecrementEvent extends CounterEvent {}

class ResetEvent extends CounterEvent {}

/// 2. States
abstract class CounterState {
  final int count;

  CounterState(this.count);
}

class CounterInitial extends CounterState {
  CounterInitial() : super(0);
}

class CounterUpdated extends CounterState {
  CounterUpdated(super.count);
}

/// 3. BLoC - Business Logic Component
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterInitial()) {
    // Register event handlers
    on<IncrementEvent>((event, emit) {
      emit(CounterUpdated(state.count + 1));
    });

    on<DecrementEvent>((event, emit) {
      emit(CounterUpdated(state.count - 1));
    });

    on<ResetEvent>((event, emit) {
      emit(CounterInitial());
    });
  }
}

/// 4. View - UI presentation
class CounterBlocView extends StatelessWidget {
  const CounterBlocView({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterBloc(),
      child: Scaffold(
        appBar: AppBar(
          title: const Text('BLoC Counter'),
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Text(
                'Counter:',
                style: TextStyle(fontSize: 20),
              ),
              BlocBuilder<CounterBloc, CounterState>(
                builder: (context, state) {
                  return Text(
                    '${state.count}',
                    style: const TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
                  );
                },
              ),
              const SizedBox(height: 16),
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  ElevatedButton(
                    onPressed: () {
                      context.read<CounterBloc>().add(DecrementEvent());
                    },
                    child: const Text('-'),
                  ),
                  const SizedBox(width: 8),
                  ElevatedButton(
                    onPressed: () {
                      context.read<CounterBloc>().add(IncrementEvent());
                    },
                    child: const Text('+'),
                  ),
                  const SizedBox(width: 8),
                  ElevatedButton(
                    onPressed: () {
                      context.read<CounterBloc>().add(ResetEvent());
                    },
                    child: const Text('Reset'),
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
```

What's happening here?
- Events: Triggers for state changes
- States: Different UI states
- BLoC: Business logic and state management
- Event-driven architecture

---

# Provider Pattern

> **Provider** architecture.

```dart
/// 1. Model with ChangeNotifier
class TodoModel extends ChangeNotifier {
  List<Todo> _todos = [];
  String _filter = 'all';

  List<Todo> get todos {
    switch (_filter) {
      case 'active':
        return _todos.where((todo) => !todo.isCompleted).toList();
      case 'completed':
        return _todos.where((todo) => todo.isCompleted).toList();
      default:
        return _todos;
    }
  }

  String get filter => _filter;
  int get totalCount => _todos.length;
  int get activeCount => _todos.where((todo) => !todo.isCompleted).length;
  int get completedCount => _todos.where((todo) => todo.isCompleted).length;

  void addTodo(String title) {
    if (title.trim().isEmpty) return;
    _todos.add(Todo(
      id: DateTime.now().millisecondsSinceEpoch,
      title: title.trim(),
    ));
    notifyListeners();
  }

  void toggleTodo(int id) {
    final index = _todos.indexWhere((todo) => todo.id == id);
    if (index != -1) {
      _todos[index].isCompleted = !_todos[index].isCompleted;
      notifyListeners();
    }
  }

  void deleteTodo(int id) {
    _todos.removeWhere((todo) => todo.id == id);
    notifyListeners();
  }

  void setFilter(String filter) {
    _filter = filter;
    notifyListeners();
  }

  void clearCompleted() {
    _todos.removeWhere((todo) => todo.isCompleted);
    notifyListeners();
  }
}

class Todo {
  final int id;
  final String title;
  bool isCompleted;

  Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });
}

/// 2. View with Provider
class TodoView extends StatelessWidget {
  const TodoView({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => TodoModel(),
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Provider Todo'),
          actions: [
            Consumer<TodoModel>(
              builder: (context, model, _) {
                if (model.completedCount > 0) {
                  return TextButton(
                    onPressed: model.clearCompleted,
                    child: const Text(
                      'Clear Completed',
                      style: TextStyle(color: Colors.white),
                    ),
                  );
                }
                return const SizedBox.shrink();
              },
            ),
          ],
        ),
        body: Column(
          children: [
            _buildAddTodoInput(),
            _buildFilterChips(),
            _buildStats(),
            Expanded(
              child: _buildTodoList(),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildAddTodoInput() {
    return Container(
      padding: const EdgeInsets.all(8),
      child: Row(
        children: [
          Expanded(
            child: TextField(
              onSubmitted: (value) {
                context.read<TodoModel>().addTodo(value);
              },
              decoration: const InputDecoration(
                hintText: 'Add a todo...',
                border: OutlineInputBorder(),
              ),
            ),
          ),
          const SizedBox(width: 8),
          ElevatedButton(
            onPressed: () {
              // In a real app, you'd use a controller
            },
            child: const Text('Add'),
          ),
        ],
      ),
    );
  }

  Widget _buildFilterChips() {
    return Consumer<TodoModel>(
      builder: (context, model, _) {
        return Padding(
          padding: const EdgeInsets.symmetric(horizontal: 8),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              _buildFilterChip('All', 'all', model),
              _buildFilterChip('Active', 'active', model),
              _buildFilterChip('Completed', 'completed', model),
            ],
          ),
        );
      },
    );
  }

  Widget _buildFilterChip(String label, String value, TodoModel model) {
    final isSelected = model.filter == value;
    return Padding(
      padding: const EdgeInsets.all(4),
      child: ActionChip(
        label: Text(label),
        onPressed: () => model.setFilter(value),
        backgroundColor: isSelected ? Colors.blue : Colors.grey[200],
        labelStyle: TextStyle(
          color: isSelected ? Colors.white : Colors.black,
        ),
      ),
    );
  }

  Widget _buildStats() {
    return Consumer<TodoModel>(
      builder: (context, model, _) {
        return Padding(
          padding: const EdgeInsets.symmetric(horizontal: 16),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text('Total: ${model.totalCount}'),
              Text('Active: ${model.activeCount}'),
              Text('Completed: ${model.completedCount}'),
            ],
          ),
        );
      },
    );
  }

  Widget _buildTodoList() {
    return Consumer<TodoModel>(
      builder: (context, model, _) {
        final todos = model.todos;
        if (todos.isEmpty) {
          return const Center(child: Text('No todos'));
        }
        return ListView.builder(
          itemCount: todos.length,
          itemBuilder: (context, index) {
            final todo = todos[index];
            return ListTile(
              leading: Checkbox(
                value: todo.isCompleted,
                onChanged: (_) => model.toggleTodo(todo.id),
              ),
              title: Text(
                todo.title,
                style: TextStyle(
                  decoration: todo.isCompleted
                      ? TextDecoration.lineThrough
                      : null,
                ),
              ),
              trailing: IconButton(
                icon: const Icon(Icons.delete),
                onPressed: () => model.deleteTodo(todo.id),
              ),
            );
          },
        );
      },
    );
  }
}
```

What's happening here?
- Model with ChangeNotifier
- Provider for dependency injection
- Consumer for selective rebuilds
- Clean separation of UI and logic

---

# Choosing an Architecture

> **Comparing** architecture patterns.

```dart
/// Architecture Comparison
class ArchitectureComparison extends StatelessWidget {
  const ArchitectureComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Architecture Patterns'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            _buildComparisonTable(),
            const SizedBox(height: 16),
            _buildRecommendation(),
          ],
        ),
      ),
    );
  }

  Widget _buildComparisonTable() {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Pattern Comparison:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            _buildComparisonRow('Pattern', 'Complexity', 'Testability', 'Popularity'),
            const Divider(),
            _buildComparisonRow('MVC', 'Low', 'Medium', 'High'),
            _buildComparisonRow('MVVM', 'Medium', 'High', 'High'),
            _buildComparisonRow('BLoC', 'Medium', 'High', 'High'),
            _buildComparisonRow('Provider', 'Low', 'Medium', 'Very High'),
          ],
        ),
      ),
    );
  }

  Widget _buildComparisonRow(String name, String complexity, String testability, String popularity) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        children: [
          SizedBox(width: 80, child: Text(name, style: const TextStyle(fontWeight: FontWeight.bold))),
          SizedBox(width: 80, child: Text(complexity)),
          SizedBox(width: 80, child: Text(testability)),
          SizedBox(width: 80, child: Text(popularity)),
        ],
      ),
    );
  }

  Widget _buildRecommendation() {
    return Container(
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
            'Recommendations:',
            style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16),
          ),
          SizedBox(height: 8),
          Text('• Small apps: Provider pattern'),
          Text('• Medium apps: MVVM with Provider'),
          Text('• Large apps: BLoC or Clean Architecture'),
          Text('• Choose based on team size and app complexity'),
        ],
      ),
    );
  }
}
```

What's happening here?
- Comparison of architecture patterns
- Complexity, testability, popularity
- Recommendations for different app sizes
- Choosing the right pattern

---

# Best Practices

## Choose Based on App Size

```dart
// Good - Choose appropriate pattern
// Small app: Provider
// Medium app: MVVM
// Large app: BLoC or Clean Architecture

// Bad - Using complex pattern for simple app
// BLoC for a simple counter app (overkill)
```

## Keep Layers Separate

```dart
// Good - Clear separation
// Presentation layer: Widgets
// Business layer: BLoC/Provider
// Data layer: Repositories/Services

// Bad - Mixed concerns
// UI with business logic
// Data access in widgets
```

## Use Dependency Injection

```dart
// Good - DI
class MyWidget extends StatelessWidget {
  final AuthRepository auth;
  
  const MyWidget({required this.auth});
}

// Bad - Direct instantiation
class MyWidget extends StatelessWidget {
  final auth = AuthRepository(); // Not testable
}
```

---

# Common Mistakes

## Using Wrong Pattern

Wrong:
```dart
// Using BLoC for a simple counter
// Too much complexity for a simple app
```

Correct:
```dart
// Using Provider for a simple counter
// Simpler and more appropriate
```

## Mixing Patterns

Wrong:
```dart
// Mixing BLoC and Provider in the same feature
// Confusing and hard to maintain
```

Correct:
```dart
// Consistent pattern throughout the app
// One pattern per feature or app
```

---

# Summary

Architecture Patterns provide structured approaches to organizing your code. Choose MVC for simple apps, MVVM for medium apps with complex UI, BLoC for large apps requiring strict separation, and Provider for a balance of simplicity and power. The right architecture makes your code more maintainable and scalable.

---

# Next Steps

- [Project Structure](project-structure.md)
- [Folder Organization](folder-organization.md)
- [Separation of Concerns](separation-of-concerns.md)

---

# Did You Know?

- MVC is the oldest and most basic pattern
- MVVM was introduced by Microsoft
- BLoC stands for Business Logic Component
- Provider is built on InheritedWidget
- Clean Architecture has 3 layers
- Pattern choice depends on app size
- Consistent pattern is more important than pattern choice
- Patterns can be combined for specific needs