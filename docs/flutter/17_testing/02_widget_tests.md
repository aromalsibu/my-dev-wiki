# Widget Tests

Understand how to test Flutter widgets to ensure they render and behave correctly.

---

# What is it?

Widget Tests (also called Component Tests) are tests that verify the behavior and appearance of individual widgets. They test widgets in isolation, simulating user interactions and verifying that the widget renders correctly and responds to user input as expected. Widget tests are faster than integration tests but more comprehensive than unit tests.

---

# Why does it exist?

Widget Tests exist to:

- Verify widget rendering
- Test user interactions
- Validate widget behavior
- Ensure UI consistency
- Catch visual regressions
- Test state changes
- Verify widget responsiveness

---

# Basic Widget Tests

> **Writing** your first widget tests.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

/// 1. Simple counter widget to test
class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;

  void _increment() {
    setState(() {
      _counter++;
    });
  }

  void _decrement() {
    setState(() {
      _counter--;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Display counter value
            Text(
              '$_counter',
              key: const Key('counter_value'),
              style: const TextStyle(fontSize: 24),
            ),
            const SizedBox(height: 16),
            // 2. Increment button
            ElevatedButton(
              key: const Key('increment_button'),
              onPressed: _increment,
              child: const Text('Increment'),
            ),
            const SizedBox(height: 8),
            // 3. Decrement button
            ElevatedButton(
              key: const Key('decrement_button'),
              onPressed: _decrement,
              child: const Text('Decrement'),
            ),
            // 4. Reset button
            ElevatedButton(
              key: const Key('reset_button'),
              onPressed: () {
                setState(() {
                  _counter = 0;
                });
              },
              child: const Text('Reset'),
            ),
          ],
        ),
      ),
    );
  }
}

/// 2. Widget tests
void main() {
  // Group tests for the CounterWidget
  group('CounterWidget Tests', () {
    // 3. Test widget initialization
    testWidgets('should display initial counter value of 0', (WidgetTester tester) async {
      // Act: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: CounterWidget(),
        ),
      );

      // Assert: Verify counter displays 0
      expect(find.text('0'), findsOneWidget);
      expect(find.text('1'), findsNothing);
    });

    // 4. Test increment button
    testWidgets('should increment counter when increment button is pressed', (WidgetTester tester) async {
      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: CounterWidget(),
        ),
      );

      // Act: Find and tap the increment button
      final incrementButton = find.byKey(const Key('increment_button'));
      expect(incrementButton, findsOneWidget);
      
      await tester.tap(incrementButton);
      await tester.pump(); // Rebuild the widget

      // Assert: Verify counter increased to 1
      expect(find.text('1'), findsOneWidget);
      expect(find.text('0'), findsNothing);
    });

    // 5. Test decrement button
    testWidgets('should decrement counter when decrement button is pressed', (WidgetTester tester) async {
      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: CounterWidget(),
        ),
      );

      // Act: Tap decrement button
      final decrementButton = find.byKey(const Key('decrement_button'));
      await tester.tap(decrementButton);
      await tester.pump();

      // Assert: Verify counter decreased to -1
      expect(find.text('-1'), findsOneWidget);
    });

    // 6. Test reset button
    testWidgets('should reset counter to 0 when reset button is pressed', (WidgetTester tester) async {
      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: CounterWidget(),
        ),
      );

      // Act: First increment to change the value
      final incrementButton = find.byKey(const Key('increment_button'));
      await tester.tap(incrementButton);
      await tester.pump();

      // Verify counter is now 1
      expect(find.text('1'), findsOneWidget);

      // Act: Tap reset button
      final resetButton = find.byKey(const Key('reset_button'));
      await tester.tap(resetButton);
      await tester.pump();

      // Assert: Verify counter reset to 0
      expect(find.text('0'), findsOneWidget);
      expect(find.text('1'), findsNothing);
    });

    // 7. Test multiple interactions
    testWidgets('should handle multiple interactions correctly', (WidgetTester tester) async {
      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: CounterWidget(),
        ),
      );

      // Act: Increment twice
      final incrementButton = find.byKey(const Key('increment_button'));
      await tester.tap(incrementButton);
      await tester.pump();
      await tester.tap(incrementButton);
      await tester.pump();

      // Assert: Counter should be 2
      expect(find.text('2'), findsOneWidget);

      // Act: Decrement once
      final decrementButton = find.byKey(const Key('decrement_button'));
      await tester.tap(decrementButton);
      await tester.pump();

      // Assert: Counter should be 1
      expect(find.text('1'), findsOneWidget);
    });
  });
}
```

What's happening here?
- testWidgets for widget testing
- pump() rebuilds the widget
- find finds widgets in the tree
- tap simulates user taps
- Keys identify widgets for testing

---

# Testing Forms

> **Testing** form widgets and validation.

```dart
/// 1. Login form widget
class LoginForm extends StatefulWidget {
  const LoginForm({super.key});

  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  String _message = '';

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      setState(() {
        _message = 'Login successful!';
      });
    } else {
      setState(() {
        _message = 'Please fix errors';
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Email field
              TextFormField(
                key: const Key('email_field'),
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Email is required';
                  }
                  if (!value.contains('@')) {
                    return 'Invalid email';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              // Password field
              TextFormField(
                key: const Key('password_field'),
                controller: _passwordController,
                obscureText: true,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Password is required';
                  }
                  if (value.length < 6) {
                    return 'Password must be at least 6 characters';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              // Submit button
              ElevatedButton(
                key: const Key('submit_button'),
                onPressed: _submitForm,
                child: const Text('Login'),
              ),
              // Message display
              if (_message.isNotEmpty)
                Padding(
                  padding: const EdgeInsets.only(top: 16),
                  child: Text(
                    _message,
                    key: const Key('message_text'),
                    style: TextStyle(
                      color: _message.contains('success') ? Colors.green : Colors.red,
                    ),
                  ),
                ),
            ],
          ),
        ),
      ),
    );
  }
}

/// 2. Form widget tests
void main() {
  group('LoginForm Tests', () {
    // 1. Test initial state
    testWidgets('should display form fields', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: LoginForm(),
        ),
      );

      // Verify form fields are present
      expect(find.byKey(const Key('email_field')), findsOneWidget);
      expect(find.byKey(const Key('password_field')), findsOneWidget);
      expect(find.byKey(const Key('submit_button')), findsOneWidget);
    });

    // 2. Test validation - empty fields
    testWidgets('should show validation errors when fields are empty', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: LoginForm(),
        ),
      );

      // Tap submit without entering any data
      final submitButton = find.byKey(const Key('submit_button'));
      await tester.tap(submitButton);
      await tester.pump();

      // Verify validation messages
      expect(find.text('Email is required'), findsOneWidget);
      expect(find.text('Password is required'), findsOneWidget);
    });

    // 3. Test validation - invalid email
    testWidgets('should show invalid email error', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: LoginForm(),
        ),
      );

      // Enter invalid email
      final emailField = find.byKey(const Key('email_field'));
      await tester.enterText(emailField, 'invalid-email');
      await tester.pump();

      // Tap submit
      final submitButton = find.byKey(const Key('submit_button'));
      await tester.tap(submitButton);
      await tester.pump();

      // Verify invalid email error
      expect(find.text('Invalid email'), findsOneWidget);
    });

    // 4. Test validation - short password
    testWidgets('should show password length error', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: LoginForm(),
        ),
      );

      // Enter valid email and short password
      final emailField = find.byKey(const Key('email_field'));
      await tester.enterText(emailField, 'test@example.com');
      
      final passwordField = find.byKey(const Key('password_field'));
      await tester.enterText(passwordField, '123');
      await tester.pump();

      // Tap submit
      final submitButton = find.byKey(const Key('submit_button'));
      await tester.tap(submitButton);
      await tester.pump();

      // Verify password error
      expect(find.text('Password must be at least 6 characters'), findsOneWidget);
    });

    // 5. Test successful login
    testWidgets('should show success message when form is valid', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: LoginForm(),
        ),
      );

      // Enter valid credentials
      final emailField = find.byKey(const Key('email_field'));
      await tester.enterText(emailField, 'test@example.com');
      
      final passwordField = find.byKey(const Key('password_field'));
      await tester.enterText(passwordField, 'password123');
      await tester.pump();

      // Tap submit
      final submitButton = find.byKey(const Key('submit_button'));
      await tester.tap(submitButton);
      await tester.pump();

      // Verify success message
      expect(find.text('Login successful!'), findsOneWidget);
      expect(find.byKey(const Key('message_text')), findsOneWidget);
    });
  });
}
```

What's happening here?
- Testing form fields
- enterText simulates typing
- Validation error checking
- Success message verification

---

# Testing Lists

> **Testing** list and scrolling widgets.

```dart
/// 1. List widget
class UserList extends StatelessWidget {
  const UserList({super.key, required this.users});

  final List<String> users;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('User List'),
      ),
      body: ListView.builder(
        key: const Key('user_list'),
        itemCount: users.length,
        itemBuilder: (context, index) {
          return ListTile(
            key: Key('user_item_$index'),
            title: Text(users[index]),
            leading: CircleAvatar(
              child: Text('${index + 1}'),
            ),
            trailing: const Icon(Icons.arrow_forward),
          );
        },
      ),
    );
  }
}

/// 2. List widget tests
void main() {
  group('UserList Tests', () {
    final testUsers = ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'];

    // 1. Test list rendering
    testWidgets('should display all users in the list', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: UserList(users: testUsers),
        ),
      );

      // Verify all users are displayed
      for (final user in testUsers) {
        expect(find.text(user), findsOneWidget);
      }
    });

    // 2. Test list item count
    testWidgets('should display correct number of items', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: UserList(users: testUsers),
        ),
      );

      // Find all list items
      final items = find.byType(ListTile);
      expect(items, findsNWidgets(testUsers.length));
    });

    // 3. Test list with keys
    testWidgets('should have keys for each item', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: UserList(users: testUsers),
        ),
      );

      // Verify each item has a key
      for (int i = 0; i < testUsers.length; i++) {
        expect(find.byKey(Key('user_item_$i')), findsOneWidget);
      }
    });

    // 4. Test empty list
    testWidgets('should handle empty list gracefully', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: UserList(users: []),
        ),
      );

      // Verify no items are displayed
      final items = find.byType(ListTile);
      expect(items, findsNothing);
    });
  });
}
```

What's happening here?
- Testing list rendering
- Item count verification
- Keyed item testing
- Empty list handling

---

# Testing Interactions

> **Testing** user interactions.

```dart
/// 1. Interactive widget
class TodoWidget extends StatefulWidget {
  const TodoWidget({super.key});

  @override
  State<TodoWidget> createState() => _TodoWidgetState();
}

class _TodoWidgetState extends State<TodoWidget> {
  final List<String> _todos = [];
  final TextEditingController _controller = TextEditingController();

  void _addTodo() {
    if (_controller.text.isNotEmpty) {
      setState(() {
        _todos.add(_controller.text);
        _controller.clear();
      });
    }
  }

  void _removeTodo(int index) {
    setState(() {
      _todos.removeAt(index);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo List'),
      ),
      body: Column(
        children: [
          // Input row
          Padding(
            padding: const EdgeInsets.all(8),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    key: const Key('todo_input'),
                    controller: _controller,
                    decoration: const InputDecoration(
                      hintText: 'Enter todo',
                    ),
                  ),
                ),
                ElevatedButton(
                  key: const Key('add_button'),
                  onPressed: _addTodo,
                  child: const Text('Add'),
                ),
              ],
            ),
          ),
          // Todo list
          Expanded(
            child: ListView.builder(
              key: const Key('todo_list'),
              itemCount: _todos.length,
              itemBuilder: (context, index) {
                return ListTile(
                  key: Key('todo_item_$index'),
                  title: Text(_todos[index]),
                  trailing: IconButton(
                    key: Key('delete_button_$index'),
                    icon: const Icon(Icons.delete),
                    onPressed: () => _removeTodo(index),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

/// 2. Interaction tests
void main() {
  group('TodoWidget Tests', () {
    // 1. Test adding a todo
    testWidgets('should add a todo when add button is pressed', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: TodoWidget(),
        ),
      );

      // Enter text in input field
      final inputField = find.byKey(const Key('todo_input'));
      await tester.enterText(inputField, 'Buy groceries');
      await tester.pump();

      // Tap add button
      final addButton = find.byKey(const Key('add_button'));
      await tester.tap(addButton);
      await tester.pump();

      // Verify todo was added
      expect(find.text('Buy groceries'), findsOneWidget);
    });

    // 2. Test adding multiple todos
    testWidgets('should add multiple todos', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: TodoWidget(),
        ),
      );

      // Add first todo
      await tester.enterText(find.byKey(const Key('todo_input')), 'Todo 1');
      await tester.tap(find.byKey(const Key('add_button')));
      await tester.pump();

      // Add second todo
      await tester.enterText(find.byKey(const Key('todo_input')), 'Todo 2');
      await tester.tap(find.byKey(const Key('add_button')));
      await tester.pump();

      // Verify both todos are present
      expect(find.text('Todo 1'), findsOneWidget);
      expect(find.text('Todo 2'), findsOneWidget);
    });

    // 3. Test deleting a todo
    testWidgets('should delete a todo when delete button is pressed', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: TodoWidget(),
        ),
      );

      // Add a todo first
      await tester.enterText(find.byKey(const Key('todo_input')), 'Todo to delete');
      await tester.tap(find.byKey(const Key('add_button')));
      await tester.pump();

      // Verify todo exists
      expect(find.text('Todo to delete'), findsOneWidget);

      // Delete the todo
      final deleteButton = find.byKey(const Key('delete_button_0'));
      await tester.tap(deleteButton);
      await tester.pump();

      // Verify todo is gone
      expect(find.text('Todo to delete'), findsNothing);
    });

    // 4. Test empty input
    testWidgets('should not add empty todo', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: TodoWidget(),
        ),
      );

      // Tap add without entering text
      final addButton = find.byKey(const Key('add_button'));
      await tester.tap(addButton);
      await tester.pump();

      // Verify no todos added
      final todoList = find.byType(ListTile);
      expect(todoList, findsNothing);
    });
  });
}
```

What's happening here?
- Testing input fields
- Adding and removing items
- Interaction testing
- Edge case handling

---

# Best Practices

## Use Keys for Testing

```dart
// Good - Use keys for test identification
Widget(
  key: const Key('my_widget'),
  // ...
)

// Bad - Relying on text only
find.text('Some text')
```

## Test User Interactions

```dart
// Good - Test interactions
await tester.tap(button);
await tester.pump();
expect(...);

// Bad - Only testing initial state
expect(find.text('0'), findsOneWidget);
```

## Use Appropriate Finders

```dart
// Good - Specific finders
find.byType(MyWidget)
find.byKey(const Key('button'))
find.text('Hello')

// Bad - Generic finders
find.byType(Widget) // Too broad
```

---

# Common Mistakes

## Not Using pump

Wrong:
```dart
// Missing pump - state not updated
await tester.tap(button);
expect(find.text('1'), findsOneWidget);
```

Correct:
```dart
// With pump - state updated
await tester.tap(button);
await tester.pump();
expect(find.text('1'), findsOneWidget);
```

## Not Using MaterialApp

Wrong:
```dart
// Missing MaterialApp - material widgets may not work
await tester.pumpWidget(MyWidget());
```

Correct:
```dart
// With MaterialApp - material widgets work
await tester.pumpWidget(
  MaterialApp(
    home: MyWidget(),
  ),
);
```

---

# Summary

Widget Tests verify widget rendering and behavior. Use testWidgets for widget testing, pump to rebuild widgets, find to locate widgets, and tap to simulate interactions. Test user interactions, form validation, and list rendering. Widget tests are essential for UI reliability.

---

# Next Steps

- [Integration Tests](integration-tests.md)
- [Golden Tests](golden-tests.md)
- [Test Utilities](test-utilities.md)

---

# Did You Know?

- Widget tests run in a test environment
- pump rebuilds widgets
- find locates widgets in the tree
- tap simulates user taps
- enterText simulates typing
- Keys help identify widgets
- Widget tests are fast
- Widget tests catch UI bugs early