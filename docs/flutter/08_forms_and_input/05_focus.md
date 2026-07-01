# Focus

Understand how to manage keyboard focus in Flutter applications.

---

# What is it?

Focus is the system that manages which widget is currently receiving keyboard input in a Flutter application. When a TextField or other input widget has focus, it means the user can type into it. Focus management allows you to control where the keyboard appears, navigate between fields, and handle focus changes programmatically.

---

# Why does it exist?

Focus exists to:

- Manage keyboard input focus
- Enable keyboard navigation
- Control input fields programmatically
- Handle focus transitions
- Support accessibility
- Improve user experience
- Manage form navigation

---

# Basic Focus Management

> **Managing focus** with FocusNode.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic focus management example
class BasicFocusExample extends StatefulWidget {
  const BasicFocusExample({super.key});

  @override
  State<BasicFocusExample> createState() => _BasicFocusExampleState();
}

class _BasicFocusExampleState extends State<BasicFocusExample> {
  // 1. Create FocusNodes for each field
  // FocusNode tracks focus state for a specific widget
  final FocusNode _nameFocus = FocusNode();
  final FocusNode _emailFocus = FocusNode();
  final FocusNode _passwordFocus = FocusNode();

  // Controllers for fields
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  void initState() {
    super.initState();
    
    // 2. Add listeners to focus nodes
    // This is useful for reacting to focus changes
    _nameFocus.addListener(() {
      // Check if this field gained focus
      if (_nameFocus.hasFocus) {
        print('Name field focused');
      } else {
        print('Name field unfocused');
      }
    });
  }

  @override
  void dispose() {
    // 3. Always dispose focus nodes
    // This prevents memory leaks
    _nameFocus.dispose();
    _emailFocus.dispose();
    _passwordFocus.dispose();
    
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Focus Management'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 4. Name field with focus node
            // The focus node controls focus for this field
            TextField(
              controller: _nameController,
              focusNode: _nameFocus,
              decoration: const InputDecoration(
                labelText: 'Name',
                hintText: 'Enter your name',
                border: OutlineInputBorder(),
              ),
              // 5. Handle submit action
              // Called when user presses "Next" or "Done"
              onSubmitted: (value) {
                // Move focus to next field
                // This shows how to programmatically move focus
                _emailFocus.requestFocus();
              },
            ),
            const SizedBox(height: 16),
            
            // 6. Email field
            TextField(
              controller: _emailController,
              focusNode: _emailFocus,
              decoration: const InputDecoration(
                labelText: 'Email',
                hintText: 'Enter your email',
                border: OutlineInputBorder(),
              ),
              keyboardType: TextInputType.emailAddress,
              onSubmitted: (value) {
                // Move focus to password field
                _passwordFocus.requestFocus();
              },
            ),
            const SizedBox(height: 16),
            
            // 7. Password field
            TextField(
              controller: _passwordController,
              focusNode: _passwordFocus,
              decoration: const InputDecoration(
                labelText: 'Password',
                hintText: 'Enter your password',
                border: OutlineInputBorder(),
              ),
              obscureText: true,
              onSubmitted: (value) {
                // Submit the form
                // Unfocus to dismiss keyboard
                _passwordFocus.unfocus();
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(
                    content: Text('Form submitted!'),
                    backgroundColor: Colors.green,
                  ),
                );
              },
            ),
            
            const SizedBox(height: 24),
            
            // 8. Focus control buttons
            // These demonstrate programmatic focus control
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Request focus on name field
                ElevatedButton(
                  onPressed: () {
                    // requestFocus() moves focus to the field
                    _nameFocus.requestFocus();
                  },
                  child: const Text('Focus Name'),
                ),
                const SizedBox(width: 8),
                
                // Request focus on email field
                ElevatedButton(
                  onPressed: () {
                    _emailFocus.requestFocus();
                  },
                  child: const Text('Focus Email'),
                ),
                const SizedBox(width: 8),
                
                // Unfocus all fields (dismiss keyboard)
                ElevatedButton(
                  onPressed: () {
                    // unfocus() removes focus from all fields
                    // This dismisses the keyboard
                    FocusScope.of(context).unfocus();
                  },
                  child: const Text('Dismiss Keyboard'),
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
- FocusNode manages focus for a widget
- requestFocus() moves focus to a field
- unfocus() dismisses the keyboard
- FocusScope manages focus scope
- onSubmitted handles action

---

# FocusScope

> **Managing groups** of focus nodes.

```dart
/// FocusScope example
class FocusScopeExample extends StatefulWidget {
  const FocusScopeExample({super.key});

  @override
  State<FocusScopeExample> createState() => _FocusScopeExampleState();
}

class _FocusScopeExampleState extends State<FocusScopeExample> {
  // Focus nodes for each field
  final FocusNode _field1Focus = FocusNode();
  final FocusNode _field2Focus = FocusNode();
  final FocusNode _field3Focus = FocusNode();
  
  // FocusScope node for the form
  // This manages focus for all fields in the scope
  final FocusScopeNode _formScope = FocusScopeNode();

  @override
  void dispose() {
    _field1Focus.dispose();
    _field2Focus.dispose();
    _field3Focus.dispose();
    _formScope.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('FocusScope Example'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: FocusScope(
          // 1. Assign the scope node to the FocusScope
          // This groups all child focus nodes
          node: _formScope,
          
          // 2. Child widgets
          child: Column(
            children: [
              // Field 1
              TextField(
                focusNode: _field1Focus,
                decoration: const InputDecoration(
                  labelText: 'Field 1',
                  border: OutlineInputBorder(),
                ),
                onSubmitted: (_) {
                  // 3. Move focus to next field in scope
                  // This respects the focus order
                  _formScope.nextFocus();
                },
              ),
              const SizedBox(height: 16),
              
              // Field 2
              TextField(
                focusNode: _field2Focus,
                decoration: const InputDecoration(
                  labelText: 'Field 2',
                  border: OutlineInputBorder(),
                ),
                onSubmitted: (_) {
                  // Move to next field
                  _formScope.nextFocus();
                },
              ),
              const SizedBox(height: 16),
              
              // Field 3
              TextField(
                focusNode: _field3Focus,
                decoration: const InputDecoration(
                  labelText: 'Field 3',
                  border: OutlineInputBorder(),
                ),
                onSubmitted: (_) {
                  // If last field, unfocus
                  _formScope.unfocus();
                },
              ),
              
              const SizedBox(height: 24),
              
              // FocusScope control buttons
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  // Focus first field
                  ElevatedButton(
                    onPressed: () {
                      _formScope.requestFocus(_field1Focus);
                    },
                    child: const Text('First Field'),
                  ),
                  const SizedBox(width: 8),
                  
                  // Focus next field
                  ElevatedButton(
                    onPressed: () {
                      // Moves focus to the next field in scope
                      _formScope.nextFocus();
                    },
                    child: const Text('Next Field'),
                  ),
                  const SizedBox(width: 8),
                  
                  // Unfocus all
                  ElevatedButton(
                    onPressed: () {
                      // Removes focus from all fields in scope
                      _formScope.unfocus();
                    },
                    child: const Text('Unfocus'),
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
- FocusScope groups focus nodes
- nextFocus() moves to next field
- requestFocus() focuses a specific field
- unfocus() removes focus from all

---

# Focus Listeners

> **Reacting to focus** changes.

```dart
/// Focus listeners example
class FocusListenersExample extends StatefulWidget {
  const FocusListenersExample({super.key});

  @override
  State<FocusListenersExample> createState() => _FocusListenersExampleState();
}

class _FocusListenersExampleState extends State<FocusListenersExample> {
  // Focus nodes
  final FocusNode _field1Focus = FocusNode();
  final FocusNode _field2Focus = FocusNode();
  
  // State for focus tracking
  bool _field1HasFocus = false;
  bool _field2HasFocus = false;

  @override
  void initState() {
    super.initState();
    
    // 1. Add listeners to focus nodes
    // These listeners are called when focus changes
    _field1Focus.addListener(() {
      // Update state when focus changes
      setState(() {
        _field1HasFocus = _field1Focus.hasFocus;
      });
    });
    
    _field2Focus.addListener(() {
      setState(() {
        _field2HasFocus = _field2Focus.hasFocus;
      });
    });
  }

  @override
  void dispose() {
    _field1Focus.dispose();
    _field2Focus.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Focus Listeners'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. Field 1 with focus listener
            Container(
              decoration: BoxDecoration(
                // Highlight when focused
                border: Border.all(
                  color: _field1HasFocus ? Colors.blue : Colors.transparent,
                  width: 2,
                ),
                borderRadius: BorderRadius.circular(8),
              ),
              child: TextField(
                focusNode: _field1Focus,
                decoration: const InputDecoration(
                  labelText: 'Field 1',
                  hintText: 'Click to focus',
                  border: OutlineInputBorder(),
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            // 3. Field 2 with focus listener
            Container(
              decoration: BoxDecoration(
                border: Border.all(
                  color: _field2HasFocus ? Colors.blue : Colors.transparent,
                  width: 2,
                ),
                borderRadius: BorderRadius.circular(8),
              ),
              child: TextField(
                focusNode: _field2Focus,
                decoration: const InputDecoration(
                  labelText: 'Field 2',
                  hintText: 'Click to focus',
                  border: OutlineInputBorder(),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 4. Focus status display
            Card(
              color: Colors.blue[50],
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    Text(
                      'Field 1: ${_field1HasFocus ? "Focused ✓" : "Not Focused"}',
                      style: TextStyle(
                        color: _field1HasFocus ? Colors.blue : Colors.grey,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'Field 2: ${_field2HasFocus ? "Focused ✓" : "Not Focused"}',
                      style: TextStyle(
                        color: _field2HasFocus ? Colors.blue : Colors.grey,
                        fontWeight: FontWeight.bold,
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
- addListener tracks focus changes
- hasFocus checks current focus state
- Visual feedback on focus
- Real-time focus status display

---

# Real-World Examples

> **Common focus** management patterns.

```dart
/// 1. Login form with focus management
class LoginFormFocus extends StatefulWidget {
  const LoginFormFocus({super.key});

  @override
  State<LoginFormFocus> createState() => _LoginFormFocusState();
}

class _LoginFormFocusState extends State<LoginFormFocus> {
  // Focus nodes for form fields
  final FocusNode _usernameFocus = FocusNode();
  final FocusNode _passwordFocus = FocusNode();
  
  // Form controllers
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  
  // Form key
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Track focus states for styling
  bool _usernameHasFocus = false;
  bool _passwordHasFocus = false;

  @override
  void initState() {
    super.initState();
    
    // Add focus listeners
    _usernameFocus.addListener(() {
      setState(() {
        _usernameHasFocus = _usernameFocus.hasFocus;
      });
    });
    
    _passwordFocus.addListener(() {
      setState(() {
        _passwordHasFocus = _passwordFocus.hasFocus;
      });
    });
  }

  @override
  void dispose() {
    _usernameFocus.dispose();
    _passwordFocus.dispose();
    _usernameController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      // Form is valid
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Login successful!'),
          backgroundColor: Colors.green,
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Login Form'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Form(
          key: _formKey,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Username field
              TextFormField(
                controller: _usernameController,
                focusNode: _usernameFocus,
                decoration: InputDecoration(
                  labelText: 'Username',
                  hintText: 'Enter your username',
                  prefixIcon: const Icon(Icons.person),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                  // Change border color when focused
                  focusedBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                    borderSide: BorderSide(
                      color: _usernameHasFocus ? Colors.blue : Colors.grey,
                      width: 2,
                    ),
                  ),
                ),
                // Move to password field on submit
                onFieldSubmitted: (_) {
                  _passwordFocus.requestFocus();
                },
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your username';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // Password field
              TextFormField(
                controller: _passwordController,
                focusNode: _passwordFocus,
                obscureText: true,
                decoration: InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter your password',
                  prefixIcon: const Icon(Icons.lock),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                  focusedBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                    borderSide: BorderSide(
                      color: _passwordHasFocus ? Colors.blue : Colors.grey,
                      width: 2,
                    ),
                  ),
                ),
                onFieldSubmitted: (_) {
                  // Submit form when done is pressed
                  _submitForm();
                },
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your password';
                  }
                  if (value.length < 6) {
                    return 'Password must be at least 6 characters';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              
              // Login button
              SizedBox(
                width: double.infinity,
                height: 50,
                child: ElevatedButton(
                  onPressed: _submitForm,
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.blue,
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(12),
                    ),
                  ),
                  child: const Text(
                    'Login',
                    style: TextStyle(fontSize: 16),
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

/// 2. Autofocus and focus traversal
class FocusTraversalExample extends StatefulWidget {
  const FocusTraversalExample({super.key});

  @override
  State<FocusTraversalExample> createState() => _FocusTraversalExampleState();
}

class _FocusTraversalExampleState extends State<FocusTraversalExample> {
  // Focus nodes for a grid of fields
  final List<List<FocusNode>> _focusNodes = [];
  final List<List<TextEditingController>> _controllers = [];

  @override
  void initState() {
    super.initState();
    
    // Initialize focus nodes and controllers for a 3x3 grid
    for (int i = 0; i < 3; i++) {
      final rowNodes = <FocusNode>[];
      final rowControllers = <TextEditingController>[];
      for (int j = 0; j < 3; j++) {
        rowNodes.add(FocusNode());
        rowControllers.add(TextEditingController());
      }
      _focusNodes.add(rowNodes);
      _controllers.add(rowControllers);
    }
  }

  @override
  void dispose() {
    // Dispose all focus nodes and controllers
    for (final row in _focusNodes) {
      for (final node in row) {
        node.dispose();
      }
    }
    for (final row in _controllers) {
      for (final controller in row) {
        controller.dispose();
      }
    }
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Focus Traversal'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Grid of fields
            ...List.generate(3, (row) {
              return Row(
                children: List.generate(3, (col) {
                  return Expanded(
                    child: Padding(
                      padding: const EdgeInsets.all(4),
                      child: TextField(
                        controller: _controllers[row][col],
                        focusNode: _focusNodes[row][col],
                        decoration: InputDecoration(
                          labelText: '$row,$col',
                          border: const OutlineInputBorder(),
                        ),
                        // Move focus to next field in row
                        onEditingComplete: () {
                          // Move to next field in same row
                          if (col < 2) {
                            _focusNodes[row][col + 1].requestFocus();
                          } else if (row < 2) {
                            // Move to first field in next row
                            _focusNodes[row + 1][0].requestFocus();
                          } else {
                            // Last field - unfocus
                            _focusNodes[row][col].unfocus();
                          }
                        },
                      ),
                    ),
                  );
                }),
              );
            }),
            
            const SizedBox(height: 24),
            
            // Navigation controls
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Focus first field
                ElevatedButton(
                  onPressed: () {
                    _focusNodes[0][0].requestFocus();
                  },
                  child: const Text('First Field'),
                ),
                const SizedBox(width: 8),
                
                // Focus last field
                ElevatedButton(
                  onPressed: () {
                    _focusNodes[2][2].requestFocus();
                  },
                  child: const Text('Last Field'),
                ),
                const SizedBox(width: 8),
                
                // Unfocus all
                ElevatedButton(
                  onPressed: () {
                    FocusScope.of(context).unfocus();
                  },
                  child: const Text('Unfocus'),
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
- Login form with focus navigation
- Visual focus indicators
- Grid focus traversal
- Autofocus and focus management

---

# Best Practices

## Dispose Focus Nodes

```dart
// Good - Dispose in dispose()
@override
void dispose() {
  _focusNode.dispose();
  super.dispose();
}

// Bad - Not disposing
@override
void dispose() {
  // Missing focus node dispose
  super.dispose();
}
```

## Use FocusScope for Groups

```dart
// Good - Group fields in FocusScope
FocusScope(
  node: _formScope,
  child: Column(
    children: formFields,
  ),
)

// Bad - Individual focus management
// Each field manages its own focus independently
```

## Handle Focus Changes

```dart
// Good - React to focus changes
_focusNode.addListener(() {
  if (_focusNode.hasFocus) {
    // Handle focus gained
  } else {
    // Handle focus lost
  }
});
```

---

# Common Mistakes

## Not Disposing Focus Nodes

Wrong:
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final FocusNode _focusNode = FocusNode();
  // No dispose - memory leak
}
```

Correct:
```dart
class _MyWidgetState extends State<MyWidget> {
  final FocusNode _focusNode = FocusNode();
  
  @override
  void dispose() {
    _focusNode.dispose();
    super.dispose();
  }
}
```

## Not Using requestFocus

Wrong:
```dart
// Manually managing focus
_focusNode.hasFocus = true; // Doesn't work
```

Correct:
```dart
// Using requestFocus
_focusNode.requestFocus();
```

---

# Summary

Focus management controls keyboard input focus in Flutter applications. Use FocusNode for individual fields, FocusScope for groups, and focus listeners for reacting to focus changes. Always dispose focus nodes to prevent memory leaks.

---

# Next Steps

- [Keyboard Handling](keyboard-handling.md)
- [Input Formatters](input-formatters.md)
- [Forms & Input Overview](forms-input-overview.md)

---

# Did You Know?

- FocusNode must be disposed
- FocusScope groups focus nodes
- requestFocus() moves focus
- nextFocus() moves to next field
- Focus listeners track changes
- Autofocus sets initial focus
- Focus determines keyboard visibility
- Focus enables keyboard navigation