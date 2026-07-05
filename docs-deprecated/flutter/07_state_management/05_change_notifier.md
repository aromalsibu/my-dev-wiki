# ChangeNotifier

Understand how to manage complex state with ChangeNotifier in Flutter.

---

# What is it?

ChangeNotifier is a class that provides change notification functionality. It allows you to create observable objects that can notify listeners when their state changes. ChangeNotifier is the foundation of many state management solutions in Flutter, including Provider, and is ideal for managing complex business logic and application state.

---

# Why does it exist?

ChangeNotifier exists to:

- Manage complex application state
- Notify listeners when state changes
- Support reactive UI updates
- Provide a foundation for state management
- Handle business logic and data models
- Enable separation of concerns
- Support multiple listeners

---

# Basic ChangeNotifier

> **Creating and using** ChangeNotifier.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

/// A simple counter model that extends ChangeNotifier
/// This class manages counter state and notifies listeners of changes
class CounterModel extends ChangeNotifier {
  // Private state variable
  int _count = 0;

  // Getter to expose the count
  int get count => _count;

  // Method to increment the counter
  void increment() {
    _count++;
    // Notify all listeners that the state has changed
    // This triggers UI rebuilds
    notifyListeners();
  }

  // Method to decrement the counter
  void decrement() {
    _count--;
    notifyListeners();
  }

  // Method to reset the counter to zero
  void reset() {
    _count = 0;
    notifyListeners();
  }
}

/// Main application widget
class ChangeNotifierExample extends StatelessWidget {
  const ChangeNotifierExample({super.key});

  @override
  Widget build(BuildContext context) {
    // Provide the CounterModel to the widget tree using ChangeNotifierProvider
    // This makes the model accessible to all descendant widgets
    return ChangeNotifierProvider(
      create: (_) => CounterModel(),
      child: MaterialApp(
        home: const CounterScreen(),
      ),
    );
  }
}

/// Screen that displays and interacts with the counter
class CounterScreen extends StatelessWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Access the CounterModel using Provider.of
    // This widget will rebuild when the counter changes
    final counter = Provider.of<CounterModel>(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('ChangeNotifier Counter'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Display the current count
            // The Text widget will rebuild when count changes
            Text(
              'Count: ${counter.count}',
              style: const TextStyle(fontSize: 24),
            ),
            const SizedBox(height: 16),
            
            // Row of control buttons
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
                const SizedBox(width: 8),
                
                // Reset button
                ElevatedButton(
                  onPressed: counter.reset,
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
```

What's happening here?
- CounterModel extends ChangeNotifier to manage state
- notifyListeners() triggers UI updates
- Provider.of<CounterModel>(context) accesses the model
- Widgets rebuild automatically when state changes
- Business logic is separated from UI

---

# ChangeNotifier with Complex State

> **Managing complex** state and logic.

```dart
/// Todo item model representing a single todo
class Todo {
  final int id;
  final String title;
  bool isCompleted;

  Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });

  // Method to toggle completion status
  void toggle() {
    isCompleted = !isCompleted;
  }

  // Returns a copy of the todo with updated properties
  Todo copyWith({
    int? id,
    String? title,
    bool? isCompleted,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }
}

/// TodoListModel manages a list of todos with ChangeNotifier
class TodoListModel extends ChangeNotifier {
  // Private list of todos
  List<Todo> _todos = [];
  int _nextId = 1;
  String _filter = 'all'; // all, active, completed

  // Getter for the filtered todo list
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

  // Getter for the current filter
  String get filter => _filter;

  // Getter for total number of todos
  int get totalCount => _todos.length;

  // Getter for number of active todos
  int get activeCount => _todos.where((todo) => !todo.isCompleted).length;

  // Getter for number of completed todos
  int get completedCount => _todos.where((todo) => todo.isCompleted).length;

  // Method to add a new todo
  void addTodo(String title) {
    // Don't add empty todos
    if (title.trim().isEmpty) return;

    // Create new todo with a unique id
    final todo = Todo(
      id: _nextId++,
      title: title.trim(),
    );
    
    // Add to the list
    _todos.add(todo);
    
    // Notify listeners that the list has changed
    notifyListeners();
  }

  // Method to toggle a todo's completion status
  void toggleTodo(int id) {
    // Find the todo by id
    final index = _todos.indexWhere((todo) => todo.id == id);
    if (index == -1) return;

    // Toggle the todo's completion status
    _todos[index].toggle();
    
    // Notify listeners of the change
    notifyListeners();
  }

  // Method to delete a todo
  void deleteTodo(int id) {
    // Remove the todo from the list
    _todos.removeWhere((todo) => todo.id == id);
    
    // Notify listeners of the change
    notifyListeners();
  }

  // Method to clear all completed todos
  void clearCompleted() {
    _todos.removeWhere((todo) => todo.isCompleted);
    notifyListeners();
  }

  // Method to change the filter
  void setFilter(String filter) {
    _filter = filter;
    notifyListeners();
  }
}

/// Main application with TodoListModel
class TodoApp extends StatelessWidget {
  const TodoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => TodoListModel(),
      child: MaterialApp(
        title: 'Todo App',
        home: const TodoScreen(),
      ),
    );
  }
}

/// Todo screen with filter controls
class TodoScreen extends StatelessWidget {
  const TodoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Get the TodoListModel instance
    final todoModel = Provider.of<TodoListModel>(context);
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todos'),
        // Show clear completed button if there are completed todos
        actions: [
          if (todoModel.completedCount > 0)
            TextButton(
              onPressed: todoModel.clearCompleted,
              child: const Text(
                'Clear Completed',
                style: TextStyle(color: Colors.white),
              ),
            ),
        ],
      ),
      body: Column(
        children: [
          // Input field to add new todos
          _buildAddTodoInput(context),
          
          // Filter chips
          _buildFilterChips(context),
          
          // Display todo statistics
          _buildStats(context),
          
          // Todo list
          Expanded(
            child: _buildTodoList(context),
          ),
        ],
      ),
    );
  }

  /// Builds the add todo input field
  Widget _buildAddTodoInput(BuildContext context) {
    final todoModel = Provider.of<TodoListModel>(context, listen: false);
    
    return Container(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: [
          // Text input field
          Expanded(
            child: TextField(
              decoration: const InputDecoration(
                hintText: 'Add a new todo...',
                border: OutlineInputBorder(),
              ),
              // Add todo when user presses Enter
              onSubmitted: (value) => todoModel.addTodo(value),
            ),
          ),
          const SizedBox(width: 8),
          
          // Add button
          ElevatedButton(
            onPressed: () {
              // Access the TextField's controller to get its value
              // In a real app, you'd use a TextEditingController
            },
            child: const Text('Add'),
          ),
        ],
      ),
    );
  }

  /// Builds filter chips for filtering todos
  Widget _buildFilterChips(BuildContext context) {
    final todoModel = Provider.of<TodoListModel>(context);
    
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          _buildFilterChip('All', 'all', todoModel),
          _buildFilterChip('Active', 'active', todoModel),
          _buildFilterChip('Completed', 'completed', todoModel),
        ],
      ),
    );
  }

  /// Helper method to build a single filter chip
  Widget _buildFilterChip(String label, String value, TodoListModel model) {
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

  /// Builds statistics display
  Widget _buildStats(BuildContext context) {
    final todoModel = Provider.of<TodoListModel>(context);
    
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text('Total: ${todoModel.totalCount}'),
          Text('Active: ${todoModel.activeCount}'),
          Text('Completed: ${todoModel.completedCount}'),
        ],
      ),
    );
  }

  /// Builds the list of todos
  Widget _buildTodoList(BuildContext context) {
    final todoModel = Provider.of<TodoListModel>(context);
    final todos = todoModel.todos;

    // Show empty state if no todos
    if (todos.isEmpty) {
      return const Center(
        child: Text('No todos yet. Add one above!'),
      );
    }

    // Build the list of todos
    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (context, index) {
        final todo = todos[index];
        
        return ListTile(
          // Checkbox to toggle completion
          leading: Checkbox(
            value: todo.isCompleted,
            onChanged: (_) => todoModel.toggleTodo(todo.id),
          ),
          
          // Todo title with strikethrough for completed items
          title: Text(
            todo.title,
            style: TextStyle(
              decoration: todo.isCompleted
                  ? TextDecoration.lineThrough
                  : null,
              color: todo.isCompleted ? Colors.grey : null,
            ),
          ),
          
          // Delete button
          trailing: IconButton(
            icon: const Icon(Icons.delete_outline),
            onPressed: () => todoModel.deleteTodo(todo.id),
          ),
          
          // Tap the list tile to toggle completion
          onTap: () => todoModel.toggleTodo(todo.id),
        );
      },
    );
  }
}
```

What's happening here?
- TodoListModel manages complex list state
- Methods encapsulate business logic
- notifyListeners triggers rebuilds
- Computed properties for derived data
- Filtering and statistics

---

# ChangeNotifier with Async Operations

> **Handling async** operations with ChangeNotifier.

```dart
/// User model for async operations
class User {
  final String name;
  final String email;
  final String avatar;

  const User({
    required this.name,
    required this.email,
    required this.avatar,
  });
}

/// AsyncUserModel manages user data with async operations
class AsyncUserModel extends ChangeNotifier {
  // State variables
  User? _user;
  bool _isLoading = false;
  String? _error;

  // Getters
  User? get user => _user;
  bool get isLoading => _isLoading;
  String? get error => _error;

  // Method to load user data asynchronously
  Future<void> loadUser() async {
    // Don't load if already loading
    if (_isLoading) return;

    // Set loading state
    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      // Simulate network request
      await Future.delayed(const Duration(seconds: 2));

      // Create user data
      _user = const User(
        name: 'John Doe',
        email: 'john@example.com',
        avatar: 'https://example.com/avatar.jpg',
      );
      
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      // Handle error
      _error = e.toString();
      _isLoading = false;
      notifyListeners();
    }
  }

  // Method to update user data
  Future<void> updateUser(String name, String email) async {
    if (_isLoading) return;

    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      await Future.delayed(const Duration(seconds: 1));

      // Update user with new data
      if (_user != null) {
        _user = User(
          name: name,
          email: email,
          avatar: _user!.avatar,
        );
      }
      
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      _isLoading = false;
      notifyListeners();
    }
  }
}

/// Screen for async user operations
class AsyncUserScreen extends StatelessWidget {
  const AsyncUserScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final userModel = Provider.of<AsyncUserModel>(context);
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Async User'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Loading indicator
            if (userModel.isLoading)
              const Center(child: CircularProgressIndicator()),
            
            // Error display
            if (userModel.error != null)
              Container(
                padding: const EdgeInsets.all(16),
                color: Colors.red[50],
                child: Row(
                  children: [
                    const Icon(Icons.error, color: Colors.red),
                    const SizedBox(width: 8),
                    Expanded(
                      child: Text(
                        userModel.error!,
                        style: const TextStyle(color: Colors.red),
                      ),
                    ),
                    TextButton(
                      onPressed: userModel.loadUser,
                      child: const Text('Retry'),
                    ),
                  ],
                ),
              ),
            
            // User display
            if (userModel.user != null && !userModel.isLoading)
              Card(
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    children: [
                      // Avatar placeholder
                      Container(
                        width: 100,
                        height: 100,
                        decoration: BoxDecoration(
                          color: Colors.blue,
                          borderRadius: BorderRadius.circular(50),
                        ),
                        child: const Icon(
                          Icons.person,
                          size: 50,
                          color: Colors.white,
                        ),
                      ),
                      const SizedBox(height: 16),
                      Text(
                        userModel.user!.name,
                        style: const TextStyle(
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      Text(
                        userModel.user!.email,
                        style: const TextStyle(fontSize: 16),
                      ),
                    ],
                  ),
                ),
              ),
            
            const Spacer(),
            
            // Action buttons
            Row(
              children: [
                // Load user button
                Expanded(
                  child: ElevatedButton(
                    onPressed: userModel.loadUser,
                    child: const Text('Load User'),
                  ),
                ),
                const SizedBox(width: 8),
                
                // Update user button
                Expanded(
                  child: ElevatedButton(
                    onPressed: userModel.user != null
                        ? () {
                            // In a real app, show a dialog to edit
                            userModel.updateUser(
                              'Jane Doe',
                              'jane@example.com',
                            );
                          }
                        : null,
                    child: const Text('Update User'),
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
```

What's happening here?
- Async operations with loading states
- Error handling and retry
- notifyListeners after async completion
- UI reflects loading, error, and data states

---

# ChangeNotifier with Multiple Models

> **Combining multiple** ChangeNotifiers.

```dart
/// Authentication model
class AuthModel extends ChangeNotifier {
  bool _isAuthenticated = false;
  String? _username;

  bool get isAuthenticated => _isAuthenticated;
  String? get username => _username;

  void login(String username) {
    _isAuthenticated = true;
    _username = username;
    notifyListeners();
  }

  void logout() {
    _isAuthenticated = false;
    _username = null;
    notifyListeners();
  }
}

/// Settings model
class SettingsModel extends ChangeNotifier {
  bool _isDarkMode = false;
  String _language = 'en';

  bool get isDarkMode => _isDarkMode;
  String get language => _language;

  void toggleTheme() {
    _isDarkMode = !_isDarkMode;
    notifyListeners();
  }

  void setLanguage(String language) {
    _language = language;
    notifyListeners();
  }
}

/// Cart model
class CartModel extends ChangeNotifier {
  List<String> _items = [];

  List<String> get items => _items;
  int get itemCount => _items.length;

  void addItem(String item) {
    _items.add(item);
    notifyListeners();
  }

  void removeItem(String item) {
    _items.remove(item);
    notifyListeners();
  }

  void clearCart() {
    _items.clear();
    notifyListeners();
  }
}

/// App with multiple providers
class MultiModelApp extends StatelessWidget {
  const MultiModelApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        // Provide all models at the root level
        ChangeNotifierProvider(create: (_) => AuthModel()),
        ChangeNotifierProvider(create: (_) => SettingsModel()),
        ChangeNotifierProvider(create: (_) => CartModel()),
      ],
      child: MaterialApp(
        home: const MultiModelScreen(),
      ),
    );
  }
}

/// Screen using multiple models
class MultiModelScreen extends StatelessWidget {
  const MultiModelScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Access all three models
    final auth = Provider.of<AuthModel>(context);
    final settings = Provider.of<SettingsModel>(context);
    final cart = Provider.of<CartModel>(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Multi Models'),
        actions: [
          // Cart counter in app bar
          IconButton(
            icon: Badge(
              label: Text('${cart.itemCount}'),
              child: const Icon(Icons.shopping_cart),
            ),
            onPressed: () {},
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Auth section
            Card(
              child: ListTile(
                title: Text(auth.isAuthenticated 
                    ? 'Logged in as ${auth.username}' 
                    : 'Not logged in'),
                trailing: ElevatedButton(
                  onPressed: auth.isAuthenticated
                      ? auth.logout
                      : () => auth.login('user'),
                  child: Text(auth.isAuthenticated ? 'Logout' : 'Login'),
                ),
              ),
            ),
            
            // Settings section
            Card(
              child: Column(
                children: [
                  SwitchListTile(
                    title: const Text('Dark Mode'),
                    value: settings.isDarkMode,
                    onChanged: (_) => settings.toggleTheme(),
                  ),
                  ListTile(
                    title: const Text('Language'),
                    trailing: DropdownButton<String>(
                      value: settings.language,
                      items: const [
                        DropdownMenuItem(value: 'en', child: Text('English')),
                        DropdownMenuItem(value: 'es', child: Text('Spanish')),
                      ],
                      onChanged: (value) {
                        if (value != null) {
                          settings.setLanguage(value);
                        }
                      },
                    ),
                  ),
                ],
              ),
            ),
            
            // Cart section
            Expanded(
              child: Card(
                child: Column(
                  children: [
                    Padding(
                      padding: const EdgeInsets.all(8),
                      child: Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text('Cart (${cart.itemCount} items)'),
                          Row(
                            children: [
                              IconButton(
                                icon: const Icon(Icons.add),
                                onPressed: () {
                                  cart.addItem('Item ${cart.itemCount + 1}');
                                },
                              ),
                              IconButton(
                                icon: const Icon(Icons.clear),
                                onPressed: cart.clearCart,
                              ),
                            ],
                          ),
                        ],
                      ),
                    ),
                    Expanded(
                      child: ListView.builder(
                        itemCount: cart.items.length,
                        itemBuilder: (context, index) {
                          return ListTile(
                            title: Text(cart.items[index]),
                            trailing: IconButton(
                              icon: const Icon(Icons.remove),
                              onPressed: () {
                                cart.removeItem(cart.items[index]);
                              },
                            ),
                          );
                        },
                      ),
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
- Multiple ChangeNotifiers for different domains
- MultiProvider combines providers
- Each model manages its own state
- Models are independent and focused

---

# Best Practices

## Keep Models Focused

```dart
// Good - Focused model
class UserModel extends ChangeNotifier {
  // Only user-related state and logic
}

// Bad - God model
class AppModel extends ChangeNotifier {
  // Everything in one model
}
```

## Use notifyListeners Correctly

```dart
// Good - Notify after changes
void update() {
  _value = newValue;
  notifyListeners();
}

// Bad - Notify without changes
void update() {
  if (_value != newValue) {
    _value = newValue;
    notifyListeners();
  }
}
```

## Separate UI and Logic

```dart
// Good - Logic in model
class CounterModel extends ChangeNotifier {
  void increment() {
    _count++;
    notifyListeners();
  }
}

// Bad - Logic in UI
ElevatedButton(
  onPressed: () {
    // Logic in UI - hard to test
    counter.value++;
    // No notification
  },
)
```

---

# Common Mistakes

## Not Calling notifyListeners

Wrong:
```dart
void update() {
  _value = newValue;
  // Missing notifyListeners()
}
```

Correct:
```dart
void update() {
  _value = newValue;
  notifyListeners();
}
```

## Creating Models in Build

Wrong:
```dart
@override
Widget build(BuildContext context) {
  final model = CounterModel(); // Created every rebuild
  return Text('${model.count}');
}
```

Correct:
```dart
// Create once and provide
ChangeNotifierProvider(
  create: (_) => CounterModel(),
  child: ...
)
```

---

# Summary

ChangeNotifier provides reactive state management with change notification. Use ChangeNotifier with Provider for complex app state, async operations, and multiple models. Keep models focused, call notifyListeners after changes, and separate business logic from UI.

---

# Next Steps

- [Listenable](listenable.md)
- [Restoration](restoration.md)
- [State Management Overview](state-management-overview.md)

---

# Did You Know?

- ChangeNotifier extends Listenable
- notifyListeners triggers UI rebuilds
- Provider integrates with ChangeNotifier
- Multiple listeners can subscribe
- Models can be combined with MultiProvider
- Async operations work with ChangeNotifier
- ChangeNotifier supports dispose
- ChangeNotifier is lightweight and efficient