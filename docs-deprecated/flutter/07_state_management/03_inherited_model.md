Here's the enhanced version of your InheritedModel documentation with comprehensive explanations and properly commented code:

---

# InheritedModel

Understand how to share and subscribe to specific parts of data using InheritedModel.

---

## What is it?

InheritedModel is an advanced version of InheritedWidget that allows widgets to subscribe to specific aspects of data. Instead of rebuilding when any part of the data changes, InheritedModel enables fine-grained control, where widgets only rebuild when the specific data they depend on actually changes. This provides better performance for complex data models.

### Key Characteristics:

- **Fine-Grained Control**: Widgets subscribe to specific data aspects, not the entire model
- **Performance Optimized**: Only rebuilds widgets when their subscribed aspects change
- **Type-Safe**: Uses enums or strings to define data aspects
- **Flexible**: Can handle complex data models with multiple independent properties
- **Efficient**: Reduces unnecessary rebuilds in large widget trees
- **Reactive**: Automatically updates UI when subscribed data changes

### When to Use InheritedModel:

✅ **Perfect for:**
- Settings/preferences with multiple independent categories
- Shopping carts where different widgets need different data
- Async data with status and content separation
- Complex data models with multiple independent properties
- Performance-critical sections with frequent updates
- Any scenario where widgets depend on different parts of data

❌ **Not ideal for:**
- Simple state management (use ValueNotifier or InheritedWidget)
- Very small widget trees with minimal rebuilds
- Cases where all widgets depend on all data
- When you don't need fine-grained rebuild control

---

## Why does it exist?

InheritedModel exists to solve several important problems in Flutter development:

### 1. Enable Fine-Grained Rebuild Control
Allows widgets to rebuild only when the specific data they depend on changes, not when any part of the data changes.

### 2. Optimize Performance for Complex Data
By reducing unnecessary rebuilds, InheritedModel improves performance for complex data models.

### 3. Allow Widgets to Subscribe to Specific Data Aspects
Widgets can declare exactly which parts of the data they need, enabling precise dependency tracking.

### 4. Reduce Unnecessary Rebuilds
Prevents widgets from rebuilding when unrelated data changes, improving app performance.

### 5. Support Complex Data Models
Handles data with multiple independent properties that different widgets depend on.

### 6. Provide Efficient Data Sharing
Shares data down the widget tree while maintaining efficiency through aspect-based subscriptions.

### 7. Enable Partial Updates
Updates only the parts of the UI that actually need to change when data is updated.

---

## Basic InheritedModel

> **Creating and using** InheritedModel.

This example demonstrates the fundamental concepts of InheritedModel with a counter and name display that update independently.

### Code Example:

```dart
import 'package:flutter/material.dart';

// ============================================================
// 1. DEFINE ASPECTS
// ============================================================
// Aspects represent different parts of your data model
// Widgets can subscribe to specific aspects
enum DataAspect { 
  counter, // For counter-related widgets
  name,    // For name-related widgets
  both     // For widgets that need both
}

// ============================================================
// 2. INHERITEDMODEL IMPLEMENTATION
// ============================================================
/// An InheritedModel that manages counter and name data
/// Provides aspect-based subscriptions for fine-grained rebuild control
class AppModel extends InheritedModel<DataAspect> {
  const AppModel({
    super.key,
    required this.counter,
    required this.name,
    required this.updateCounter,
    required this.updateName,
    required super.child,
  });

  // ============================================================
  // 2a. DATA FIELDS
  // ============================================================
  // The actual data being managed
  final int counter;
  final String name;
  
  // Callbacks to update the data
  final VoidCallback updateCounter;
  final VoidCallback updateName;

  // ============================================================
  // 2b. STATIC ACCESSOR METHOD
  // ============================================================
  // Provides access to the model with optional aspect parameter
  static AppModel? of(BuildContext context, {DataAspect? aspect}) {
    return InheritedModel.inheritFrom<AppModel>(context, aspect: aspect);
  }

  // ============================================================
  // 2c. UPDATE SHOULD NOTIFY
  // ============================================================
  // Determines if any part of the data has changed
  // This is a coarse check - fine-grained control is in updateShouldNotifyDependent
  @override
  bool updateShouldNotify(AppModel oldWidget) {
    return counter != oldWidget.counter || name != oldWidget.name;
  }

  // ============================================================
  // 2d. UPDATE SHOULD NOTIFY DEPENDENT (CRITICAL)
  // ============================================================
  // This is the key method for fine-grained rebuild control
  // It checks if the specific aspects a widget depends on have actually changed
  @override
  bool updateShouldNotifyDependent(
    AppModel oldWidget,
    Set<DataAspect> dependencies, // The aspects this widget subscribes to
  ) {
    // Check each dependency and return true only if that specific data changed
    
    // If widget depends on counter and counter changed
    if (dependencies.contains(DataAspect.counter) && 
        counter != oldWidget.counter) {
      return true;
    }
    
    // If widget depends on name and name changed
    if (dependencies.contains(DataAspect.name) && 
        name != oldWidget.name) {
      return true;
    }
    
    // If widget depends on both and either changed
    if (dependencies.contains(DataAspect.both) && 
        (counter != oldWidget.counter || name != oldWidget.name)) {
      return true;
    }
    
    // No relevant changes - don't rebuild
    return false;
  }
}

// ============================================================
// 3. PARENT WIDGET (STATE MANAGER)
// ============================================================
/// StatefulWidget that manages the data and provides it via InheritedModel
class InheritedModelExample extends StatefulWidget {
  const InheritedModelExample({super.key});

  @override
  State<InheritedModelExample> createState() => _InheritedModelExampleState();
}

class _InheritedModelExampleState extends State<InheritedModelExample> {
  // ============================================================
  // 3a. STATE DATA
  // ============================================================
  int _counter = 0;
  String _name = 'John';

  // ============================================================
  // 3b. UPDATE METHODS
  // ============================================================
  void _updateCounter() {
    setState(() {
      _counter++;
    });
  }

  void _updateName() {
    setState(() {
      _name = _name == 'John' ? 'Jane' : 'John';
    });
  }

  @override
  Widget build(BuildContext context) {
    // ============================================================
    // 3c. PROVIDE DATA VIA INHERITEDMODEL
    // ============================================================
    // AppModel wraps the widget tree and provides data to all descendants
    return AppModel(
      counter: _counter,
      name: _name,
      updateCounter: _updateCounter,
      updateName: _updateName,
      child: MaterialApp(
        home: const ModelHomeScreen(),
      ),
    );
  }
}

// ============================================================
// 4. CONSUMER SCREEN
// ============================================================
/// Displays widgets that subscribe to different data aspects
class ModelHomeScreen extends StatelessWidget {
  const ModelHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('InheritedModel'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // ============================================================
            // 4a. COUNTER DISPLAY (Only counter aspect)
            // ============================================================
            // This widget only rebuilds when counter changes
            const CounterDisplay(),
            const SizedBox(height: 16),
            
            // ============================================================
            // 4b. NAME DISPLAY (Only name aspect)
            // ============================================================
            // This widget only rebuilds when name changes
            const NameDisplay(),
            const SizedBox(height: 16),
            
            // ============================================================
            // 4c. BOTH DISPLAY (Both aspects)
            // ============================================================
            // This widget rebuilds when either counter or name changes
            const BothDisplay(),
            const SizedBox(height: 24),
            
            // ============================================================
            // 4d. CONTROL BUTTONS
            // ============================================================
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      // Update only the counter
                      AppModel.of(context)?.updateCounter();
                    },
                    child: const Text('Update Counter'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      // Update only the name
                      AppModel.of(context)?.updateName();
                    },
                    child: const Text('Update Name'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

// ============================================================
// 5. COUNTER DISPLAY WIDGET
// ============================================================
/// Displays the counter value
/// Only subscribes to the counter aspect
class CounterDisplay extends StatelessWidget {
  const CounterDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    // ============================================================
    // 5a. SUBSCRIBE TO COUNTER ASPECT
    // ============================================================
    // This widget will only rebuild when counter changes
    // Because it subscribes specifically to DataAspect.counter
    final model = AppModel.of(context, aspect: DataAspect.counter);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            const Text(
              'Counter:',
              style: TextStyle(fontSize: 18),
            ),
            Text(
              '${model?.counter ?? 0}',
              style: const TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// ============================================================
// 6. NAME DISPLAY WIDGET
// ============================================================
/// Displays the name value
/// Only subscribes to the name aspect
class NameDisplay extends StatelessWidget {
  const NameDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    // ============================================================
    // 6a. SUBSCRIBE TO NAME ASPECT
    // ============================================================
    // This widget will only rebuild when name changes
    final model = AppModel.of(context, aspect: DataAspect.name);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            const Text(
              'Name:',
              style: TextStyle(fontSize: 18),
            ),
            Text(
              model?.name ?? 'Unknown',
              style: const TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// ============================================================
// 7. BOTH DISPLAY WIDGET
// ============================================================
/// Displays both counter and name
/// Subscribes to both aspects
class BothDisplay extends StatelessWidget {
  const BothDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    // ============================================================
    // 7a. SUBSCRIBE TO BOTH ASPECTS
    // ============================================================
    // This widget rebuilds when either counter or name changes
    final model = AppModel.of(context, aspect: DataAspect.both);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text('Counter:'),
                Text('${model?.counter ?? 0}'),
              ],
            ),
            const Divider(),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text('Name:'),
                Text(model?.name ?? 'Unknown'),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### What's happening here?

1. **DataAspect Enum**: Defines the different aspects of data that widgets can subscribe to.

2. **AppModel Class**: The InheritedModel implementation that manages counter and name data.

3. **updateShouldNotifyDependent**: The critical method that controls rebuilds based on aspect subscriptions.

4. **Aspect Subscriptions**: Widgets subscribe to specific aspects using the `aspect` parameter.

5. **Fine-Grained Rebuilds**: CounterDisplay only rebuilds when counter changes, NameDisplay only when name changes, BothDisplay when either changes.

### Key Points:
- DataAspect enum defines aspects
- updateShouldNotifyDependent controls rebuilds
- Each widget subscribes to specific aspects
- Widgets only rebuild when their aspect changes
- Provides performance optimization through fine-grained control

---

## InheritedModel with Lists

> **Managing lists** with InheritedModel.

This example demonstrates managing a todo list with InheritedModel, showing CRUD operations with aspect-based subscriptions.

### Code Example:

```dart
import 'package:flutter/material.dart';

// ============================================================
// 1. TODO ASPECTS
// ============================================================
// Defines different aspects of todo data
enum TodoAspect { 
  all,        // All todos
  completed,  // Completed todos only
  active      // Active todos only
}

// ============================================================
// 2. TODO MODEL CLASS
// ============================================================
/// Data model for a todo item
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

// ============================================================
// 3. TODO MODEL (INHERITEDMODEL)
// ============================================================
/// InheritedModel that manages todo list data
class TodoModel extends InheritedModel<TodoAspect> {
  const TodoModel({
    super.key,
    required this.todos,
    required this.addTodo,
    required this.toggleTodo,
    required this.deleteTodo,
    required super.child,
  });

  // ============================================================
  // 3a. DATA AND OPERATIONS
  // ============================================================
  final List<Todo> todos;
  final Function(String) addTodo;
  final Function(int) toggleTodo;
  final Function(int) deleteTodo;

  // ============================================================
  // 3b. STATIC ACCESSOR
  // ============================================================
  static TodoModel? of(BuildContext context, {TodoAspect? aspect}) {
    return InheritedModel.inheritFrom<TodoModel>(context, aspect: aspect);
  }

  // ============================================================
  // 3c. UPDATE SHOULD NOTIFY
  // ============================================================
  // Simple check - list has changed
  @override
  bool updateShouldNotify(TodoModel oldWidget) {
    return todos != oldWidget.todos;
  }

  // ============================================================
  // 3d. UPDATE SHOULD NOTIFY DEPENDENT
  // ============================================================
  // Different aspects can have different rebuild logic
  @override
  bool updateShouldNotifyDependent(
    TodoModel oldWidget,
    Set<TodoAspect> dependencies,
  ) {
    // In a real app, you could implement more specific logic
    // For simplicity, we rebuild all aspects when list changes
    return todos != oldWidget.todos;
  }
}

// ============================================================
// 4. PARENT WIDGET
// ============================================================
/// StatefulWidget that manages todo list state
class TodoListModelExample extends StatefulWidget {
  const TodoListModelExample({super.key});

  @override
  State<TodoListModelExample> createState() => _TodoListModelExampleState();
}

class _TodoListModelExampleState extends State<TodoListModelExample> {
  // ============================================================
  // 4a. STATE DATA
  // ============================================================
  List<Todo> _todos = [];
  int _nextId = 1;

  // ============================================================
  // 4b. CRUD OPERATIONS
  // ============================================================
  void _addTodo(String title) {
    setState(() {
      _todos.add(Todo(id: _nextId++, title: title, isCompleted: false));
    });
  }

  void _toggleTodo(int id) {
    setState(() {
      final todo = _todos.firstWhere((todo) => todo.id == id);
      todo.isCompleted = !todo.isCompleted;
    });
  }

  void _deleteTodo(int id) {
    setState(() {
      _todos.removeWhere((todo) => todo.id == id);
    });
  }

  @override
  Widget build(BuildContext context) {
    return TodoModel(
      todos: _todos,
      addTodo: _addTodo,
      toggleTodo: _toggleTodo,
      deleteTodo: _deleteTodo,
      child: MaterialApp(
        home: const TodoListScreen(),
      ),
    );
  }
}

// ============================================================
// 5. TODO LIST SCREEN
// ============================================================
/// Displays the todo list with filtering
class TodoListScreen extends StatelessWidget {
  const TodoListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final model = TodoModel.of(context);
    String _newTodoText = '';

    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo List (InheritedModel)'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // ============================================================
            // 5a. ADD TODO INPUT
            // ============================================================
            Row(
              children: [
                Expanded(
                  child: TextField(
                    onChanged: (value) => _newTodoText = value,
                    decoration: const InputDecoration(
                      hintText: 'Add new todo...',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (value) {
                      model?.addTodo(value);
                    },
                  ),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    if (_newTodoText.isNotEmpty) {
                      model?.addTodo(_newTodoText);
                    }
                  },
                  child: const Text('Add'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            
            // ============================================================
            // 5b. TODO LIST
            // ============================================================
            // Subscribes to 'all' aspect
            Expanded(
              child: ValueListenableBuilder(
                valueListenable: ValueNotifier(0),
                builder: (context, _, __) {
                  final todos = model?.todos ?? [];
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
                          onChanged: (_) => model?.toggleTodo(todo.id),
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
                          onPressed: () => model?.deleteTodo(todo.id),
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

### What's happening here?

1. **TodoAspect Enum**: Defines different aspects of todo data (all, completed, active).

2. **Todo Model**: InheritedModel that manages todo list and CRUD operations.

3. **State Management**: Parent widget manages the actual list state.

4. **CRUD Operations**: Add, toggle, and delete todos with setState.

5. **List Display**: Subscribes to 'all' aspect to display all todos.

### Key Points:
- List state in InheritedModel
- CRUD operations
- Aspect-based subscriptions
- Efficient list management

---

## InheritedModel with Async Data

> **Handling async** data with InheritedModel.

This example demonstrates managing asynchronous data with InheritedModel, including loading states and error handling.

### Code Example:

```dart
import 'package:flutter/material.dart';

// ============================================================
// 1. DATA STATUS ENUM
// ============================================================
// Defines the possible states of async data
enum DataStatus { 
  initial,   // Not yet loaded
  loading,   // Currently loading
  loaded,    // Successfully loaded
  error      // Error occurred
}

// ============================================================
// 2. ASYNC DATA MODEL (INHERITEDMODEL)
// ============================================================
/// InheritedModel that manages async data loading
class AsyncDataModel extends InheritedModel<String> {
  const AsyncDataModel({
    super.key,
    required this.data,
    required this.status,
    required this.error,
    required this.loadData,
    required super.child,
  });

  // ============================================================
  // 2a. DATA FIELDS
  // ============================================================
  final List<String> data;
  final DataStatus status;
  final String? error;
  final VoidCallback loadData;

  // ============================================================
  // 2b. STATIC ACCESSOR
  // ============================================================
  static AsyncDataModel? of(BuildContext context, {String? aspect}) {
    return InheritedModel.inheritFrom<AsyncDataModel>(context, aspect: aspect);
  }

  // ============================================================
  // 2c. UPDATE SHOULD NOTIFY
  // ============================================================
  @override
  bool updateShouldNotify(AsyncDataModel oldWidget) {
    return data != oldWidget.data || status != oldWidget.status;
  }

  // ============================================================
  // 2d. UPDATE SHOULD NOTIFY DEPENDENT
  // ============================================================
  // Different aspects for data and status
  @override
  bool updateShouldNotifyDependent(
    AsyncDataModel oldWidget,
    Set<String> dependencies,
  ) {
    // If widget depends on 'data' aspect
    if (dependencies.contains('data')) {
      return data != oldWidget.data;
    }
    
    // If widget depends on 'status' aspect
    if (dependencies.contains('status')) {
      return status != oldWidget.status;
    }
    
    // No relevant dependencies - don't rebuild
    return false;
  }
}

// ============================================================
// 3. PARENT WIDGET
// ============================================================
/// StatefulWidget that manages async data loading
class AsyncDataExample extends StatefulWidget {
  const AsyncDataExample({super.key});

  @override
  State<AsyncDataExample> createState() => _AsyncDataExampleState();
}

class _AsyncDataExampleState extends State<AsyncDataExample> {
  // ============================================================
  // 3a. STATE DATA
  // ============================================================
  List<String> _data = [];
  DataStatus _status = DataStatus.initial;
  String? _error;

  // ============================================================
  // 3b. ASYNC LOADING
  // ============================================================
  Future<void> _loadData() async {
    // Update to loading state
    setState(() {
      _status = DataStatus.loading;
      _error = null;
    });

    try {
      // Simulate network request
      await Future.delayed(const Duration(seconds: 2));
      
      // Simulate success
      _data = ['Item 1', 'Item 2', 'Item 3'];
      
      setState(() {
        _status = DataStatus.loaded;
      });
    } catch (e) {
      // Handle error
      setState(() {
        _status = DataStatus.error;
        _error = e.toString();
      });
    }
  }

  @override
  void initState() {
    super.initState();
    // Auto-load data when widget initializes
    _loadData();
  }

  @override
  Widget build(BuildContext context) {
    return AsyncDataModel(
      data: _data,
      status: _status,
      error: _error,
      loadData: _loadData,
      child: MaterialApp(
        home: const AsyncDataScreen(),
      ),
    );
  }
}

// ============================================================
// 4. ASYNC DATA SCREEN
// ============================================================
/// Displays async data with loading and error states
class AsyncDataScreen extends StatelessWidget {
  const AsyncDataScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Async Data (InheritedModel)'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // ============================================================
            // 4a. STATUS DISPLAY
            // ============================================================
            // Subscribes to 'status' aspect
            // Only rebuilds when status changes
            AsyncDataStatusDisplay(),
            
            const SizedBox(height: 16),
            
            // ============================================================
            // 4b. DATA DISPLAY
            // ============================================================
            // Subscribes to 'data' aspect
            // Only rebuilds when data changes
            Expanded(
              child: AsyncDataListDisplay(),
            ),
          ],
        ),
      ),
    );
  }
}

// ============================================================
// 5. STATUS DISPLAY
// ============================================================
/// Displays the current loading/error status
/// Subscribes to 'status' aspect only
class AsyncDataStatusDisplay extends StatelessWidget {
  const AsyncDataStatusDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    // ============================================================
    // 5a. SUBSCRIBE TO STATUS ASPECT
    // ============================================================
    // Only rebuilds when status changes
    final model = AsyncDataModel.of(context, aspect: 'status');
    
    if (model == null) return const SizedBox.shrink();

    // Show different UI based on status
    switch (model.status) {
      case DataStatus.initial:
        return const Card(
          child: Padding(
            padding: EdgeInsets.all(16),
            child: Text('Ready to load data'),
          ),
        );
        
      case DataStatus.loading:
        return const Card(
          child: Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                CircularProgressIndicator(),
                SizedBox(width: 16),
                Text('Loading data...'),
              ],
            ),
          ),
        );
        
      case DataStatus.loaded:
        return Card(
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(Icons.check_circle, color: Colors.green),
                const SizedBox(width: 8),
                Text('Loaded ${model.data.length} items'),
              ],
            ),
          ),
        );
        
      case DataStatus.error:
        return Card(
          color: Colors.red[50],
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              children: [
                Row(
                  children: [
                    Icon(Icons.error, color: Colors.red),
                    const SizedBox(width: 8),
                    const Text('Error loading data'),
                  ],
                ),
                if (model.error != null)
                  Padding(
                    padding: const EdgeInsets.only(top: 8),
                    child: Text(
                      model.error!,
                      style: TextStyle(color: Colors.red[700]),
                    ),
                  ),
                const SizedBox(height: 8),
                ElevatedButton(
                  onPressed: model.loadData,
                  child: const Text('Retry'),
                ),
              ],
            ),
          ),
        );
    }
  }
}

// ============================================================
// 6. DATA LIST DISPLAY
// ============================================================
/// Displays the loaded data list
/// Subscribes to 'data' aspect only
class AsyncDataListDisplay extends StatelessWidget {
  const AsyncDataListDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    // ============================================================
    // 6a. SUBSCRIBE TO DATA ASPECT
    // ============================================================
    // Only rebuilds when data changes
    final model = AsyncDataModel.of(context, aspect: 'data');
    
    if (model == null) return const SizedBox.shrink();

    if (model.data.isEmpty) {
      return const Center(child: Text('No data available'));
    }

    return ListView.builder(
      itemCount: model.data.length,
      itemBuilder: (context, index) {
        return Card(
          child: ListTile(
            leading: CircleAvatar(
              child: Text('${index + 1}'),
            ),
            title: Text(model.data[index]),
          ),
        );
      },
    );
  }
}
```

### What's happening here?

1. **DataStatus Enum**: Defines the possible states of async data (initial, loading, loaded, error).

2. **AsyncDataModel**: InheritedModel that manages async data with separate 'data' and 'status' aspects.

3. **Data Loading**: Simulates async data loading with error handling.

4. **Separate Subscriptions**: StatusDisplay subscribes to 'status', DataListDisplay subscribes to 'data'.

5. **Efficient Updates**: Status changes only rebuild status widgets, data changes only rebuild data widgets.

### Key Points:
- Async data loading with InheritedModel
- Separate data and status aspects
- Loading and error state management
- Aspect-based subscriptions for efficiency

---

## Real-World Examples

> **Common patterns** with InheritedModel.

This section demonstrates practical applications of InheritedModel in real-world scenarios.

### 1. Settings Provider with InheritedModel

```dart
import 'package:flutter/material.dart';

// ============================================================
// 1. SETTINGS ASPECTS
// ============================================================
// Defines different settings categories
enum SettingsAspect { 
  theme,         // Theme-related settings
  language,      // Language settings
  notifications  // Notification preferences
}

// ============================================================
// 2. SETTINGS MODEL (INHERITEDMODEL)
// ============================================================
/// InheritedModel that manages application settings
class SettingsModel extends InheritedModel<SettingsAspect> {
  const SettingsModel({
    super.key,
    required this.themeMode,
    required this.language,
    required this.notificationsEnabled,
    required this.updateTheme,
    required this.updateLanguage,
    required this.toggleNotifications,
    required super.child,
  });

  // ============================================================
  // 2a. SETTINGS DATA
  // ============================================================
  final ThemeMode themeMode;
  final String language;
  final bool notificationsEnabled;
  
  // Update callbacks
  final Function(ThemeMode) updateTheme;
  final Function(String) updateLanguage;
  final Function(bool) toggleNotifications;

  // ============================================================
  // 2b. STATIC ACCESSOR
  // ============================================================
  static SettingsModel? of(BuildContext context, {SettingsAspect? aspect}) {
    return InheritedModel.inheritFrom<SettingsModel>(context, aspect: aspect);
  }

  // ============================================================
  // 2c. UPDATE LOGIC
  // ============================================================
  @override
  bool updateShouldNotify(SettingsModel oldWidget) {
    return themeMode != oldWidget.themeMode ||
           language != oldWidget.language ||
           notificationsEnabled != oldWidget.notificationsEnabled;
  }

  @override
  bool updateShouldNotifyDependent(
    SettingsModel oldWidget,
    Set<SettingsAspect> dependencies,
  ) {
    // Theme-related widgets rebuild only when theme changes
    if (dependencies.contains(SettingsAspect.theme) && 
        themeMode != oldWidget.themeMode) {
      return true;
    }
    
    // Language-related widgets rebuild only when language changes
    if (dependencies.contains(SettingsAspect.language) && 
        language != oldWidget.language) {
      return true;
    }
    
    // Notification-related widgets rebuild only when notification setting changes
    if (dependencies.contains(SettingsAspect.notifications) && 
        notificationsEnabled != oldWidget.notificationsEnabled) {
      return true;
    }
    
    return false;
  }
}

// ============================================================
// 3. SETTINGS SCREEN
// ============================================================
/// Displays settings with aspect-based subscriptions
class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final model = SettingsModel.of(context);
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Settings'),
      ),
      body: ListView(
        children: [
          // ============================================================
          // 3a. THEME SETTINGS
          // ============================================================
          // Subscribes to theme aspect only
          ThemeSettingsTile(),
          
          // ============================================================
          // 3b. LANGUAGE SETTINGS
          // ============================================================
          // Subscribes to language aspect only
          LanguageSettingsTile(),
          
          // ============================================================
          // 3c. NOTIFICATION SETTINGS
          // ============================================================
          // Subscribes to notifications aspect only
          NotificationSettingsTile(),
        ],
      ),
    );
  }
}

/// Theme settings tile - only rebuilds when theme changes
class ThemeSettingsTile extends StatelessWidget {
  const ThemeSettingsTile({super.key});

  @override
  Widget build(BuildContext context) {
    final model = SettingsModel.of(context, aspect: SettingsAspect.theme);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Theme',
              style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              children: [
                const Text('Dark Mode'),
                const Spacer(),
                Switch(
                  value: model?.themeMode == ThemeMode.dark,
                  onChanged: (value) {
                    model?.updateTheme(
                      value ? ThemeMode.dark : ThemeMode.light,
                    );
                  },
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

/// Language settings tile - only rebuilds when language changes
class LanguageSettingsTile extends StatelessWidget {
  const LanguageSettingsTile({super.key});

  @override
  Widget build(BuildContext context) {
    final model = SettingsModel.of(context, aspect: SettingsAspect.language);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Language',
              style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              children: [
                Expanded(
                  child: DropdownButton<String>(
                    value: model?.language ?? 'en',
                    isExpanded: true,
                    items: const [
                      DropdownMenuItem(value: 'en', child: Text('English')),
                      DropdownMenuItem(value: 'es', child: Text('Spanish')),
                      DropdownMenuItem(value: 'fr', child: Text('French')),
                    ],
                    onChanged: (value) {
                      if (value != null) {
                        model?.updateLanguage(value);
                      }
                    },
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

/// Notification settings tile - only rebuilds when notification setting changes
class NotificationSettingsTile extends StatelessWidget {
  const NotificationSettingsTile({super.key});

  @override
  Widget build(BuildContext context) {
    final model = SettingsModel.of(context, aspect: SettingsAspect.notifications);
    
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Notifications',
              style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              children: [
                const Text('Enable Notifications'),
                const Spacer(),
                Switch(
                  value: model?.notificationsEnabled ?? false,
                  onChanged: (value) {
                    model?.toggleNotifications(value);
                  },
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

**What's happening here?**
- SettingsModel manages three independent settings categories
- Each settings tile subscribes to only its relevant aspect
- Theme changes only rebuild theme widgets
- Language changes only rebuild language widgets
- Notification changes only rebuild notification widgets

---

### 2. Shopping Cart Model

```dart
import 'package:flutter/material.dart';

// ============================================================
// 1. CART ASPECTS
// ============================================================
// Defines different aspects of cart data
enum CartAspect { 
  items,    // Item list
  total,    // Total price
  count     // Item count
}

// ============================================================
// 2. CART ITEM MODEL
// ============================================================
/// Data model for a cart item
class CartItem {
  final String id;
  final String name;
  final double price;
  int quantity;

  CartItem({
    required this.id,
    required this.name,
    required this.price,
    this.quantity = 1,
  });
}

// ============================================================
// 3. CART MODEL (INHERITEDMODEL)
// ============================================================
/// InheritedModel that manages shopping cart data
class CartModel extends InheritedModel<CartAspect> {
  const CartModel({
    super.key,
    required this.items,
    required this.addItem,
    required this.removeItem,
    required this.updateQuantity,
    required super.child,
  });

  // ============================================================
  // 3a. CART DATA
  // ============================================================
  final List<CartItem> items;
  final Function(CartItem) addItem;
  final Function(String) removeItem;
  final Function(String, int) updateQuantity;

  // ============================================================
  // 3b. COMPUTED PROPERTIES
  // ============================================================
  int get totalCount => items.fold(0, (sum, item) => sum + item.quantity);
  double get totalPrice => items.fold(
    0.0, 
    (sum, item) => sum + (item.price * item.quantity)
  );

  // ============================================================
  // 3c. STATIC ACCESSOR
  // ============================================================
  static CartModel? of(BuildContext context, {CartAspect? aspect}) {
    return InheritedModel.inheritFrom<CartModel>(context, aspect: aspect);
  }

  // ============================================================
  // 3d. UPDATE LOGIC
  // ============================================================
  @override
  bool updateShouldNotify(CartModel oldWidget) {
    return items != oldWidget.items;
  }

  @override
  bool updateShouldNotifyDependent(
    CartModel oldWidget,
    Set<CartAspect> dependencies,
  ) {
    // Items aspect - changes when list changes
    if (dependencies.contains(CartAspect.items)) {
      return items != oldWidget.items;
    }
    
    // Total price aspect - changes when total price changes
    if (dependencies.contains(CartAspect.total)) {
      return totalPrice != oldWidget.totalPrice;
    }
    
    // Count aspect - changes when item count changes
    if (dependencies.contains(CartAspect.count)) {
      return totalCount != oldWidget.totalCount;
    }
    
    return false;
  }
}

// ============================================================
// 4. CART SCREEN
// ============================================================
/// Displays cart with aspect-based subscriptions
class CartScreen extends StatelessWidget {
  const CartScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Shopping Cart'),
        actions: [
          // ============================================================
          // 4a. CART COUNT (Subscribes to 'count' aspect)
          // ============================================================
          CartCountBadge(),
        ],
      ),
      body: Column(
        children: [
          // ============================================================
          // 4b. CART ITEMS (Subscribes to 'items' aspect)
          // ============================================================
          Expanded(
            child: CartItemsList(),
          ),
          // ============================================================
          // 4c. CART TOTAL (Subscribes to 'total' aspect)
          // ============================================================
          CartTotalBar(),
        ],
      ),
    );
  }
}

/// Cart count badge - only rebuilds when count changes
class CartCountBadge extends StatelessWidget {
  const CartCountBadge({super.key});

  @override
  Widget build(BuildContext context) {
    final model = CartModel.of(context, aspect: CartAspect.count);
    final count = model?.totalCount ?? 0;
    
    return Container(
      padding: const EdgeInsets.all(8),
      child: Row(
        children: [
          const Icon(Icons.shopping_cart),
          const SizedBox(width: 4),
          Text(
            '$count',
            style: const TextStyle(color: Colors.white),
          ),
        ],
      ),
    );
  }
}

/// Cart items list - only rebuilds when items change
class CartItemsList extends StatelessWidget {
  const CartItemsList({super.key});

  @override
  Widget build(BuildContext context) {
    final model = CartModel.of(context, aspect: CartAspect.items);
    final items = model?.items ?? [];
    
    if (items.isEmpty) {
      return const Center(child: Text('Cart is empty'));
    }
    
    return ListView.builder(
      itemCount: items.length,
      itemBuilder: (context, index) {
        final item = items[index];
        return Card(
          child: ListTile(
            title: Text(item.name),
            subtitle: Text('Price: \$${item.price.toStringAsFixed(2)}'),
            trailing: Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                IconButton(
                  icon: const Icon(Icons.remove),
                  onPressed: () {
                    if (item.quantity > 1) {
                      model?.updateQuantity(item.id, item.quantity - 1);
                    } else {
                      model?.removeItem(item.id);
                    }
                  },
                ),
                Text('${item.quantity}'),
                IconButton(
                  icon: const Icon(Icons.add),
                  onPressed: () {
                    model?.updateQuantity(item.id, item.quantity + 1);
                  },
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}

/// Cart total bar - only rebuilds when total price changes
class CartTotalBar extends StatelessWidget {
  const CartTotalBar({super.key});

  @override
  Widget build(BuildContext context) {
    final model = CartModel.of(context, aspect: CartAspect.total);
    final total = model?.totalPrice ?? 0.0;
    
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: const BorderRadius.vertical(top: Radius.circular(16)),
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          const Text(
            'Total:',
            style: TextStyle(
              color: Colors.white,
              fontSize: 18,
              fontWeight: FontWeight.bold,
            ),
          ),
          Text(
            '\$${total.toStringAsFixed(2)}',
            style: const TextStyle(
              color: Colors.white,
              fontSize: 24,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }
}
```

**What's happening here?**
- CartModel manages items, total, and count separately
- Each UI component subscribes to only what it needs
- Count badge only rebuilds when count changes
- Items list only rebuilds when items change
- Total bar only rebuilds when total price changes

---

## Best Practices

### 1. Define Clear Aspects

```dart
// ✅ Good - Clear, specific aspects
enum DataAspect { 
  profile,    // User profile data
  settings,   // Settings data
  preferences // User preferences
}

// ❌ Bad - Ambiguous, overlapping aspects
enum DataAspect { 
  all,  // Too broad
  some, // Unclear
  data  // Generic
}
```

### 2. Use updateShouldNotifyDependent Correctly

```dart
// ✅ Good - Specific dependency checking
@override
bool updateShouldNotifyDependent(
  MyModel oldWidget,
  Set<MyAspect> dependencies,
) {
  // Only rebuild if the specific aspect changed
  if (dependencies.contains(MyAspect.counter)) {
    return counter != oldWidget.counter;
  }
  if (dependencies.contains(MyAspect.name)) {
    return name != oldWidget.name;
  }
  return false;
}

// ❌ Bad - No dependency checking
@override
bool updateShouldNotifyDependent(
  MyModel oldWidget,
  Set<MyAspect> dependencies,
) {
  // Rebuilds for any change regardless of dependencies
  return true;
}
```

### 3. Provide Aspect Parameter in Accessor

```dart
// ✅ Good - Aspect parameter available
static MyModel? of(BuildContext context, {MyAspect? aspect}) {
  return InheritedModel.inheritFrom<MyModel>(context, aspect: aspect);
}

// ❌ Bad - No aspect parameter
static MyModel? of(BuildContext context) {
  return context.dependOnInheritedWidgetOfExactType<MyModel>();
}
```

### 4. Keep Aspects Granular

```dart
// ✅ Good - Granular aspects
enum SettingsAspect { 
  theme,
  language,
  notifications,
  privacy
}

// ❌ Bad - Too broad
enum SettingsAspect { 
  all // One aspect for everything
}
```

### 5. Use const for Initial Values

```dart
// ✅ Good - const for better performance
const AppModel(
  counter: 0,
  name: 'Default',
  // ...
)

// ❌ Bad - Creating new objects unnecessarily
AppModel(
  counter: 0,
  name: 'Default',
  // ...
)
```

---

## Common Mistakes

### 1. Forgetting updateShouldNotifyDependent

```dart
// ❌ WRONG - No fine-grained control
@override
bool updateShouldNotifyDependent(
  MyModel oldWidget,
  Set<MyAspect> dependencies,
) {
  // Uses default implementation that ignores aspects
  return super.updateShouldNotifyDependent(oldWidget, dependencies);
}

// ✅ CORRECT - Custom control
@override
bool updateShouldNotifyDependent(
  MyModel oldWidget,
  Set<MyAspect> dependencies,
) {
  // Check each dependency specifically
  if (dependencies.contains(MyAspect.data)) {
    return data != oldWidget.data;
  }
  return false;
}
```

### 2. Not Using Aspect Parameter

```dart
// ❌ WRONG - No aspect subscription
final model = InheritedModel.inheritFrom<MyModel>(context);
// Widget rebuilds for ANY change

// ✅ CORRECT - With aspect subscription
final model = InheritedModel.inheritFrom<MyModel>(
  context,
  aspect: MyAspect.data,
);
// Widget only rebuilds when data aspect changes
```

### 3. Using Strings Instead of Enums

```dart
// ❌ WRONG - Using strings (error-prone)
static AsyncDataModel? of(BuildContext context, {String? aspect}) {
  return InheritedModel.inheritFrom<AsyncDataModel>(context, aspect: aspect);
}
// Usage: AsyncDataModel.of(context, aspect: 'data')

// ✅ CORRECT - Using enums (type-safe)
enum DataAspect { data, status, error }

static AsyncDataModel? of(BuildContext context, {DataAspect? aspect}) {
  return InheritedModel.inheritFrom<AsyncDataModel>(context, aspect: aspect);
}
// Usage: AsyncDataModel.of(context, aspect: DataAspect.data)
```

### 4. Returning Null Without Context

```dart
// ❌ WRONG - No error handling
static MyModel of(BuildContext context) {
  return InheritedModel.inheritFrom<MyModel>(context)!; // Could crash
}

// ✅ CORRECT - Safe access with null handling
static MyModel? of(BuildContext context) {
  return InheritedModel.inheritFrom<MyModel>(context);
}
// Usage: model?.doSomething()
```

---

## Summary

InheritedModel extends InheritedWidget with fine-grained rebuild control. Widgets can subscribe to specific aspects of data and only rebuild when those aspects change. This provides better performance for complex data models and is useful for settings, cart, and async data management.

### Key Takeaways:

✅ **Fine-Grained Control** - Widgets subscribe to specific data aspects
✅ **Performance Optimized** - Only rebuilds when subscribed data changes
✅ **Type-Safe** - Use enums for aspect definitions
✅ **Flexible** - Works with any data type or structure
✅ **Efficient** - Reduces unnecessary rebuilds
✅ **Reactive** - Automatic UI updates for subscribed data

### When to Use:

- ✅ Settings/preferences with independent categories
- ✅ Shopping carts with multiple metrics
- ✅ Async data with separate status and content
- ✅ Complex data models with independent properties
- ✅ Performance-critical sections with frequent updates

### When to Avoid:

- ❌ Simple state management (use ValueNotifier or InheritedWidget)
- ❌ Very small widget trees
- ❌ Cases where all widgets depend on all data
- ❌ When you don't need fine-grained rebuild control

---

## Next Steps

- [ValueNotifier](valuenotifier.md) - Simple reactive state management
- [ChangeNotifier](changenotifier.md) - More complex state management
- [Listenable](listenable.md) - Low-level notification system
- [Provider](provider.md) - Advanced state management pattern

---

## Did You Know?

- 💡 InheritedModel is an advanced version of InheritedWidget
- 💡 Aspects enable fine-grained rebuild control
- 💡 updateShouldNotifyDependent is the key method for controlling rebuilds
- 💡 Widgets subscribe to specific aspects using the aspect parameter
- 💡 InheritedModel improves performance for complex data models
- 💡 Useful for settings, cart, and async data management
- 💡 Supports multiple aspects per widget
- 💡 Reduces unnecessary rebuilds and improves app performance

---

**Done!** This enhanced version now has:
- ✅ Detailed explanations for every heading
- ✅ Proper code comments with section separators
- ✅ "What's happening here?" summaries
- ✅ Best practices and common mistakes sections
- ✅ Real-world examples with explanations
- ✅ Comprehensive educational content

Ready for the next file! 🚀