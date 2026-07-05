# Local State

Understand how to manage local state in Flutter applications.

---

# What is it?

Local state refers to data that is specific to a single widget and does not need to be shared with other parts of the application. It is typically managed within a StatefulWidget using setState(). Local state is ideal for simple UI state like toggles, form inputs, animations, and any data that only affects a small part of the UI.

---

# Why does it exist?

Local state exists to:

- Manage widget-specific data
- Handle simple UI state
- Enable local updates without affecting the whole app
- Keep state management simple and contained
- Improve performance by limiting rebuild scope
- Support interactive UI elements
- Maintain component encapsulation

---

# Basic Local State

> **Managing state** within a single widget.

```dart
// Basic local state with StatefulWidget
class LocalStateExample extends StatefulWidget {
  const LocalStateExample({super.key});

  @override
  State<LocalStateExample> createState() => _LocalStateExampleState();
}

class _LocalStateExampleState extends State<LocalStateExample> {
  // Local state variables
  int _counter = 0;
  String _text = 'Hello';
  bool _isToggled = false;

  // Methods that update state
  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  void _updateText(String newText) {
    setState(() {
      _text = newText;
    });
  }

  void _toggleSwitch() {
    setState(() {
      _isToggled = !_isToggled;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Local State'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Counter
            Card(
              child: ListTile(
                title: const Text('Counter'),
                subtitle: Text('Value: $_counter'),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      icon: const Icon(Icons.remove),
                      onPressed: () {
                        setState(() {
                          _counter--;
                        });
                      },
                    ),
                    IconButton(
                      icon: const Icon(Icons.add),
                      onPressed: _incrementCounter,
                    ),
                  ],
                ),
              ),
            ),
            
            // Text input
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text('Text Input'),
                    const SizedBox(height: 8),
                    TextField(
                      onChanged: _updateText,
                      decoration: const InputDecoration(
                        border: OutlineInputBorder(),
                        hintText: 'Enter text',
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text('Current text: $_text'),
                  ],
                ),
              ),
            ),
            
            // Toggle
            Card(
              child: SwitchListTile(
                title: const Text('Toggle'),
                subtitle: Text(_isToggled ? 'ON' : 'OFF'),
                value: _isToggled,
                onChanged: (value) {
                  setState(() {
                    _isToggled = value;
                  });
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

What's happening here?
- State variables in State class
- setState() triggers rebuild
- Each interaction updates local state
- UI reflects state changes

---

# Form State

> **Managing form** inputs and validation.

```dart
// Form state management
class FormStateExample extends StatefulWidget {
  const FormStateExample({super.key});

  @override
  State<FormStateExample> createState() => _FormStateExampleState();
}

class _FormStateExampleState extends State<FormStateExample> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  // Form state
  String _name = '';
  String _email = '';
  String _password = '';
  bool _isSubmitting = false;
  String? _submissionResult;

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      setState(() {
        _isSubmitting = true;
        _submissionResult = null;
      });

      // Simulate submission
      Future.delayed(const Duration(seconds: 2), () {
        setState(() {
          _isSubmitting = false;
          _submissionResult = 'Form submitted successfully!';
        });
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Form State'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // Name field
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Name',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your name';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() {
                    _name = value;
                  });
                },
              ),
              const SizedBox(height: 16),
              
              // Email field
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your email';
                  }
                  if (!value.contains('@')) {
                    return 'Please enter a valid email';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() {
                    _email = value;
                  });
                },
              ),
              const SizedBox(height: 16),
              
              // Password field
              TextFormField(
                controller: _passwordController,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  border: OutlineInputBorder(),
                ),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter a password';
                  }
                  if (value.length < 6) {
                    return 'Password must be at least 6 characters';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() {
                    _password = value;
                  });
                },
              ),
              const SizedBox(height: 24),
              
              // Submit button with loading state
              _isSubmitting
                  ? const CircularProgressIndicator()
                  : ElevatedButton(
                      onPressed: _submitForm,
                      style: ElevatedButton.styleFrom(
                        minimumSize: const Size(double.infinity, 50),
                      ),
                      child: const Text('Submit'),
                    ),
              
              if (_submissionResult != null) ...[
                const SizedBox(height: 16),
                Text(
                  _submissionResult!,
                  style: TextStyle(
                    color: Colors.green[700],
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
              
              const SizedBox(height: 16),
              Text('Name: $_name'),
              Text('Email: $_email'),
              Text('Password: ${'*' * _password.length}'),
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- TextEditingController for form fields
- Form validation with validator
- Loading state during submission
- Form data displayed in UI

---

# Animation State

> **Managing animation** state locally.

```dart
// Animation state management
class AnimationStateExample extends StatefulWidget {
  const AnimationStateExample({super.key});

  @override
  State<AnimationStateExample> createState() => _AnimationStateExampleState();
}

class _AnimationStateExampleState extends State<AnimationStateExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;
  bool _isExpanded = false;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 500),
    );
    
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _toggleExpansion() {
    setState(() {
      _isExpanded = !_isExpanded;
    });
    
    if (_isExpanded) {
      _controller.forward();
    } else {
      _controller.reverse();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Animation State'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Animated container
            AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                return Container(
                  width: 200 * (1 + _animation.value * 0.5),
                  height: 200 * (1 + _animation.value * 0.5),
                  decoration: BoxDecoration(
                    color: Colors.blue,
                    borderRadius: BorderRadius.circular(8),
                    boxShadow: [
                      BoxShadow(
                        color: Colors.black.withOpacity(0.2),
                        blurRadius: 10 * _animation.value,
                        spreadRadius: 5 * _animation.value,
                      ),
                    ],
                  ),
                  child: Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Icon(
                          Icons.star,
                          size: 50 + 30 * _animation.value,
                          color: Colors.white,
                        ),
                        const SizedBox(height: 8),
                        Text(
                          _isExpanded ? 'Expanded!' : 'Tap to expand',
                          style: const TextStyle(
                            color: Colors.white,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),
            
            const SizedBox(height: 24),
            
            // Toggle button
            ElevatedButton(
              onPressed: _toggleExpansion,
              child: Text(_isExpanded ? 'Collapse' : 'Expand'),
            ),
            
            const SizedBox(height: 16),
            
            Text('State: ${_isExpanded ? 'Expanded' : 'Collapsed'}'),
            Text('Animation: ${_animation.value.toStringAsFixed(2)}'),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- AnimationController manages animation
- Animation values drive UI changes
- State tracks expansion status
- AnimatedBuilder rebuilds on animation

---

# Complex Local State

> **Managing complex** local state.

```dart
// Complex local state management
class TodoListStateExample extends StatefulWidget {
  const TodoListStateExample({super.key});

  @override
  State<TodoListStateExample> createState() => _TodoListStateExampleState();
}

class _TodoListStateExampleState extends State<TodoListStateExample> {
  // Complex state
  List<Todo> _todos = [];
  String _newTodoText = '';
  int _nextId = 1;
  String _filter = 'all';

  // Filtered todos
  List<Todo> get _filteredTodos {
    switch (_filter) {
      case 'active':
        return _todos.where((todo) => !todo.isCompleted).toList();
      case 'completed':
        return _todos.where((todo) => todo.isCompleted).toList();
      default:
        return _todos;
    }
  }

  void _addTodo() {
    if (_newTodoText.trim().isEmpty) return;
    
    setState(() {
      _todos.add(Todo(
        id: _nextId++,
        title: _newTodoText.trim(),
        isCompleted: false,
      ));
      _newTodoText = '';
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

  void _clearCompleted() {
    setState(() {
      _todos.removeWhere((todo) => todo.isCompleted);
    });
  }

  int get _activeCount => _todos.where((todo) => !todo.isCompleted).length;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo List'),
        actions: [
          if (_todos.any((todo) => todo.isCompleted))
            TextButton(
              onPressed: _clearCompleted,
              child: const Text(
                'Clear Completed',
                style: TextStyle(color: Colors.white),
              ),
            ),
        ],
      ),
      body: Column(
        children: [
          // Add todo input
          Container(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    decoration: const InputDecoration(
                      hintText: 'Add a new todo...',
                      border: OutlineInputBorder(),
                    ),
                    onChanged: (value) {
                      setState(() {
                        _newTodoText = value;
                      });
                    },
                    onSubmitted: (_) => _addTodo(),
                  ),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _addTodo,
                  child: const Text('Add'),
                ),
              ],
            ),
          ),
          
          // Filter tabs
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              _buildFilterChip('All', 'all'),
              _buildFilterChip('Active', 'active'),
              _buildFilterChip('Completed', 'completed'),
            ],
          ),
          
          // Stats
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text('Total: ${_todos.length}'),
                Text('Active: $_activeCount'),
              ],
            ),
          ),
          
          // Todo list
          Expanded(
            child: ListView.builder(
              itemCount: _filteredTodos.length,
              itemBuilder: (context, index) {
                final todo = _filteredTodos[index];
                return ListTile(
                  leading: Checkbox(
                    value: todo.isCompleted,
                    onChanged: (_) => _toggleTodo(todo.id),
                  ),
                  title: Text(
                    todo.title,
                    style: TextStyle(
                      decoration: todo.isCompleted
                          ? TextDecoration.lineThrough
                          : null,
                      color: todo.isCompleted ? Colors.grey : null,
                    ),
                  ),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete_outline),
                    onPressed: () => _deleteTodo(todo.id),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildFilterChip(String label, String value) {
    return Padding(
      padding: const EdgeInsets.all(4),
      child: ActionChip(
        label: Text(label),
        onPressed: () {
          setState(() {
            _filter = value;
          });
        },
        backgroundColor: _filter == value ? Colors.blue : Colors.grey[200],
        labelStyle: TextStyle(
          color: _filter == value ? Colors.white : Colors.black,
        ),
      ),
    );
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
```

What's happening here?
- Complex state management
- Multiple state variables
- Computed properties
- State operations with setState

---

# Best Practices

## Keep State Minimal

```dart
// Good - Minimal state
class GoodWidget extends StatefulWidget {
  @override
  State<GoodWidget> createState() => _GoodWidgetState();
}

class _GoodWidgetState extends State<GoodWidget> {
  int _counter = 0; // Only what's needed
  
  @override
  Widget build(BuildContext context) {
    return Text('$_counter');
  }
}

// Bad - Too much state
class BadWidget extends StatefulWidget {
  @override
  State<BadWidget> createState() => _BadWidgetState();
}

class _BadWidgetState extends State<BadWidget> {
  int _counter = 0;
  String _temp = '';
  bool _flag1 = false;
  bool _flag2 = false;
  // Unnecessary state
}
```

## Use setState Efficiently

```dart
// Good - Only update what changed
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      CounterWidget(counter: _counter),
      const StaticWidget(), // Won't rebuild
    ],
  );
}

// Bad - Rebuilding everything
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      CounterWidget(counter: _counter),
      StaticWidget(), // Rebuilds unnecessarily
    ],
  );
}
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
    return Text('Hello'); // Rebuilds unnecessarily
  }
}
```

---

# Common Mistakes

## Mutating State Without setState

Wrong:
```dart
// No setState - UI won't update
void _increment() {
  _counter++;
}
```

Correct:
```dart
// With setState
void _increment() {
  setState(() {
    _counter++;
  });
}
```

## Heavy Computation in setState

Wrong:
```dart
// Heavy work inside setState
void _update() {
  setState(() {
    _data = heavyComputation(); // Slow
  });
}
```

Correct:
```dart
// Work outside setState
void _update() {
  final result = heavyComputation();
  setState(() {
    _data = result;
  });
}
```

---

# Summary

Local state manages data within a single widget using StatefulWidget and setState(). It's ideal for simple UI state, form inputs, animations, and component-specific data. Keep state minimal, use setState efficiently, and consider lifting state up when sharing is needed.

---

# Next Steps

- [InheritedWidget](inheritedwidget.md)
- [InheritedModel](inheritedmodel.md)
- [ValueNotifier](valuenotifier.md)

---

# Did You Know?

- Local state is managed in StatefulWidget
- setState triggers rebuild
- Local state is not shared with other widgets
- State is encapsulated in the widget
- State persists across rebuilds
- Local state is good for UI-specific data
- State can be lifted to parent widgets
- Use const widgets for performance