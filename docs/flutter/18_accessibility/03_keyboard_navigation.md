# Keyboard Navigation

Understand how to implement keyboard navigation in Flutter applications for accessibility and desktop/web support.

---

# What is it?

Keyboard Navigation allows users to interact with your Flutter application using a keyboard instead of a touch screen or mouse. This is essential for accessibility, enabling users with motor impairments to navigate your app, and is also required for desktop and web applications where keyboard input is the primary interaction method.

---

# Why does it exist?

Keyboard Navigation exists to:

- Support users with motor impairments
- Enable desktop and web applications
- Provide alternative navigation method
- Improve accessibility compliance
- Support keyboard shortcuts
- Enhance power user experience
- Enable focus-based navigation

---

# Basic Keyboard Navigation

> **Implementing** keyboard navigation.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic keyboard navigation example
class BasicKeyboardNavigationExample extends StatefulWidget {
  const BasicKeyboardNavigationExample({super.key});

  @override
  State<BasicKeyboardNavigationExample> createState() => _BasicKeyboardNavigationExampleState();
}

class _BasicKeyboardNavigationExampleState extends State<BasicKeyboardNavigationExample> {
  // Focus nodes for navigation
  final FocusNode _focusNode1 = FocusNode();
  final FocusNode _focusNode2 = FocusNode();
  final FocusNode _focusNode3 = FocusNode();

  @override
  void dispose() {
    _focusNode1.dispose();
    _focusNode2.dispose();
    _focusNode3.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Keyboard Navigation'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Focusable widget with keyboard navigation
            Focus(
              focusNode: _focusNode1,
              child: Container(
                padding: const EdgeInsets.all(16),
                color: _focusNode1.hasFocus ? Colors.blue : Colors.grey[300],
                child: const Text(
                  'Press Tab to navigate',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            // 2. Focusable button
            Focus(
              focusNode: _focusNode2,
              child: ElevatedButton(
                onPressed: () {
                  print('Button pressed');
                },
                child: const Text('Button 1'),
              ),
            ),
            const SizedBox(height: 16),
            
            // 3. Focusable text field
            Focus(
              focusNode: _focusNode3,
              child: TextField(
                decoration: const InputDecoration(
                  labelText: 'Text Input',
                  border: OutlineInputBorder(),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 4. Keyboard shortcut info
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Text(
                'Tab: Navigate between elements\n'
                'Enter: Activate focused element\n'
                'Space: Toggle focused element',
                style: TextStyle(fontSize: 14),
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
- Focus manages keyboard focus
- Tab key navigates between elements
- Enter activates focused elements
- Visual focus indicators

---

# Focus Traversal

> **Controlling** focus navigation order.

```dart
/// Focus traversal example
class FocusTraversalExample extends StatefulWidget {
  const FocusTraversalExample({super.key});

  @override
  State<FocusTraversalExample> createState() => _FocusTraversalExampleState();
}

class _FocusTraversalExampleState extends State<FocusTraversalExample> {
  // Focus nodes for custom traversal
  final FocusNode _field1Focus = FocusNode();
  final FocusNode _field2Focus = FocusNode();
  final FocusNode _field3Focus = FocusNode();
  final FocusNode _buttonFocus = FocusNode();

  @override
  void dispose() {
    _field1Focus.dispose();
    _field2Focus.dispose();
    _field3Focus.dispose();
    _buttonFocus.dispose();
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
            // 1. Custom traversal order using FocusTraversalGroup
            FocusTraversalGroup(
              policy: OrderedTraversalPolicy(),
              child: Column(
                children: [
                  // Field 1
                  Focus(
                    focusNode: _field1Focus,
                    child: TextField(
                      decoration: const InputDecoration(
                        labelText: 'Field 1',
                        border: OutlineInputBorder(),
                      ),
                      onSubmitted: (_) {
                        // Move to next field
                        _field2Focus.requestFocus();
                      },
                    ),
                  ),
                  const SizedBox(height: 16),
                  
                  // Field 2
                  Focus(
                    focusNode: _field2Focus,
                    child: TextField(
                      decoration: const InputDecoration(
                        labelText: 'Field 2',
                        border: OutlineInputBorder(),
                      ),
                      onSubmitted: (_) {
                        // Move to next field
                        _field3Focus.requestFocus();
                      },
                    ),
                  ),
                  const SizedBox(height: 16),
                  
                  // Field 3
                  Focus(
                    focusNode: _field3Focus,
                    child: TextField(
                      decoration: const InputDecoration(
                        labelText: 'Field 3',
                        border: OutlineInputBorder(),
                      ),
                      onSubmitted: (_) {
                        // Move to button
                        _buttonFocus.requestFocus();
                      },
                    ),
                  ),
                  const SizedBox(height: 16),
                  
                  // Button
                  Focus(
                    focusNode: _buttonFocus,
                    child: ElevatedButton(
                      onPressed: () {
                        print('Form submitted');
                      },
                      child: const Text('Submit'),
                    ),
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 2. Focus order display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Focus Order:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 4),
                  Text('Field 1 → Field 2 → Field 3 → Submit'),
                  const SizedBox(height: 8),
                  const Text(
                    'Press Tab to navigate',
                    style: TextStyle(fontSize: 12, color: Colors.grey),
                  ),
                ],
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
- FocusTraversalGroup controls navigation order
- OrderedTraversalPolicy sets linear order
- requestFocus moves focus programmatically
- Tab key follows the custom order

---

# Keyboard Shortcuts

> **Implementing** keyboard shortcuts.

```dart
/// Keyboard shortcuts example
class KeyboardShortcutsExample extends StatefulWidget {
  const KeyboardShortcutsExample({super.key});

  @override
  State<KeyboardShortcutsExample> createState() => _KeyboardShortcutsExampleState();
}

class _KeyboardShortcutsExampleState extends State<KeyboardShortcutsExample> {
  String _lastAction = 'None';
  String _shortcutMessage = 'Press a key combination';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Keyboard Shortcuts'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. CallbackShortcuts for keyboard shortcuts
            CallbackShortcuts(
              bindings: {
                // Ctrl+S to save
                const SingleActivator(LogicalKeyboardKey.keyS, control: true): () {
                  setState(() {
                    _lastAction = 'Save';
                    _shortcutMessage = 'Ctrl+S pressed - Save action';
                  });
                },
                // Ctrl+Z to undo
                const SingleActivator(LogicalKeyboardKey.keyZ, control: true): () {
                  setState(() {
                    _lastAction = 'Undo';
                    _shortcutMessage = 'Ctrl+Z pressed - Undo action';
                  });
                },
                // Ctrl+Shift+Z to redo
                const SingleActivator(
                  LogicalKeyboardKey.keyZ,
                  control: true,
                  shift: true,
                ): () {
                  setState(() {
                    _lastAction = 'Redo';
                    _shortcutMessage = 'Ctrl+Shift+Z pressed - Redo action';
                  });
                },
                // Escape to cancel
                const SingleActivator(LogicalKeyboardKey.escape): () {
                  setState(() {
                    _lastAction = 'Cancel';
                    _shortcutMessage = 'Escape pressed - Cancel action';
                  });
                },
                // Enter to confirm
                const SingleActivator(LogicalKeyboardKey.enter): () {
                  setState(() {
                    _lastAction = 'Confirm';
                    _shortcutMessage = 'Enter pressed - Confirm action';
                  });
                },
                // F1 for help
                const SingleActivator(LogicalKeyboardKey.f1): () {
                  setState(() {
                    _lastAction = 'Help';
                    _shortcutMessage = 'F1 pressed - Help action';
                  });
                },
              },
              child: Focus(
                autofocus: true,
                child: Container(
                  padding: const EdgeInsets.all(24),
                  decoration: BoxDecoration(
                    color: Colors.grey[100],
                    borderRadius: BorderRadius.circular(8),
                    border: Border.all(color: Colors.grey[300]!),
                  ),
                  child: Column(
                    children: [
                      const Text(
                        'Focus here and try shortcuts',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 18,
                        ),
                      ),
                      const SizedBox(height: 16),
                      Container(
                        padding: const EdgeInsets.all(16),
                        decoration: BoxDecoration(
                          color: Colors.white,
                          borderRadius: BorderRadius.circular(8),
                        ),
                        child: Column(
                          children: [
                            Text(
                              'Last Action: $_lastAction',
                              style: const TextStyle(fontSize: 16),
                            ),
                            const SizedBox(height: 8),
                            Text(
                              _shortcutMessage,
                              style: const TextStyle(
                                fontSize: 16,
                                fontWeight: FontWeight.bold,
                                color: Colors.blue,
                              ),
                            ),
                          ],
                        ),
                      ),
                      const SizedBox(height: 16),
                      _buildShortcutInfo(),
                    ],
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildShortcutInfo() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(8),
      ),
      child: const Column(
        children: [
          Text(
            'Available Shortcuts:',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 8),
          Text('Ctrl+S: Save'),
          Text('Ctrl+Z: Undo'),
          Text('Ctrl+Shift+Z: Redo'),
          Text('Escape: Cancel'),
          Text('Enter: Confirm'),
          Text('F1: Help'),
        ],
      ),
    );
  }
}
```

What's happening here?
- CallbackShortcuts for keyboard shortcuts
- SingleActivator defines key combinations
- Multiple shortcut bindings
- Real-time feedback

---

# Focusable Widgets

> **Creating custom** focusable widgets.

```dart
/// Custom focusable widget
class FocusableButton extends StatefulWidget {
  const FocusableButton({
    super.key,
    required this.onPressed,
    required this.child,
    this.autofocus = false,
  });

  final VoidCallback onPressed;
  final Widget child;
  final bool autofocus;

  @override
  State<FocusableButton> createState() => _FocusableButtonState();
}

class _FocusableButtonState extends State<FocusableButton> {
  final FocusNode _focusNode = FocusNode();
  bool _isHovered = false;

  @override
  void initState() {
    super.initState();
    if (widget.autofocus) {
      _focusNode.requestFocus();
    }
    _focusNode.addListener(() {
      setState(() {});
    });
  }

  @override
  void dispose() {
    _focusNode.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: widget.onPressed,
      child: Focus(
        focusNode: _focusNode,
        child: MouseRegion(
          onEnter: (_) => setState(() => _isHovered = true),
          onExit: (_) => setState(() => _isHovered = false),
          child: Container(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
            decoration: BoxDecoration(
              color: _getBackgroundColor(),
              borderRadius: BorderRadius.circular(8),
              border: Border.all(
                color: _focusNode.hasFocus ? Colors.blue : Colors.transparent,
                width: 2,
              ),
              boxShadow: [
                if (_focusNode.hasFocus)
                  BoxShadow(
                    color: Colors.blue.withOpacity(0.3),
                    blurRadius: 8,
                  ),
              ],
            ),
            child: DefaultTextStyle(
              style: TextStyle(
                color: _getTextColor(),
                fontWeight: FontWeight.bold,
              ),
              child: widget.child,
            ),
          ),
        ),
      ),
    );
  }

  Color _getBackgroundColor() {
    if (_focusNode.hasFocus) return Colors.blue;
    if (_isHovered) return Colors.blue[100]!;
    return Colors.grey[200]!;
  }

  Color _getTextColor() {
    if (_focusNode.hasFocus) return Colors.white;
    return Colors.black;
  }
}

/// Using focusable widgets
class FocusableWidgetsExample extends StatelessWidget {
  const FocusableWidgetsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Focusable Widgets'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            const Text(
              'Try navigating with Tab key',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            FocusableButton(
              onPressed: () => print('Button 1 pressed'),
              child: const Text('Button 1'),
            ),
            const SizedBox(height: 8),
            FocusableButton(
              onPressed: () => print('Button 2 pressed'),
              child: const Text('Button 2'),
            ),
            const SizedBox(height: 8),
            FocusableButton(
              onPressed: () => print('Button 3 pressed'),
              child: const Text('Button 3'),
            ),
            const SizedBox(height: 24),
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Text(
                'Use Tab to navigate between buttons\n'
                'Enter or Space to activate focused button',
                style: TextStyle(fontSize: 14),
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
- Custom focusable button
- Visual focus indicators
- Mouse hover support
- Keyboard activation

---

# Real-World Examples

> **Common patterns** with keyboard navigation.

```dart
/// 1. Accessible form with keyboard navigation
class AccessibleFormKeyboard extends StatefulWidget {
  const AccessibleFormKeyboard({super.key});

  @override
  State<AccessibleFormKeyboard> createState() => _AccessibleFormKeyboardState();
}

class _AccessibleFormKeyboardState extends State<AccessibleFormKeyboard> {
  // Focus nodes for form fields
  final FocusNode _nameFocus = FocusNode();
  final FocusNode _emailFocus = FocusNode();
  final FocusNode _passwordFocus = FocusNode();
  final FocusNode _submitFocus = FocusNode();

  // Controllers
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  void dispose() {
    _nameFocus.dispose();
    _emailFocus.dispose();
    _passwordFocus.dispose();
    _submitFocus.dispose();
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Accessible Form'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: FocusTraversalGroup(
          child: Column(
            children: [
              // 1. Name field
              Focus(
                focusNode: _nameFocus,
                child: TextField(
                  controller: _nameController,
                  decoration: const InputDecoration(
                    labelText: 'Name',
                    hintText: 'Enter your name',
                    border: OutlineInputBorder(),
                  ),
                  onSubmitted: (_) {
                    _emailFocus.requestFocus();
                  },
                ),
              ),
              const SizedBox(height: 16),
              
              // 2. Email field
              Focus(
                focusNode: _emailFocus,
                child: TextField(
                  controller: _emailController,
                  decoration: const InputDecoration(
                    labelText: 'Email',
                    hintText: 'Enter your email',
                    border: OutlineInputBorder(),
                  ),
                  keyboardType: TextInputType.emailAddress,
                  onSubmitted: (_) {
                    _passwordFocus.requestFocus();
                  },
                ),
              ),
              const SizedBox(height: 16),
              
              // 3. Password field
              Focus(
                focusNode: _passwordFocus,
                child: TextField(
                  controller: _passwordController,
                  decoration: const InputDecoration(
                    labelText: 'Password',
                    hintText: 'Enter your password',
                    border: OutlineInputBorder(),
                  ),
                  obscureText: true,
                  onSubmitted: (_) {
                    _submitFocus.requestFocus();
                  },
                ),
              ),
              const SizedBox(height: 16),
              
              // 4. Submit button
              Focus(
                focusNode: _submitFocus,
                child: ElevatedButton(
                  onPressed: () {
                    print('Form submitted');
                  },
                  child: const Text('Submit'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

/// 2. Keyboard shortcut service
class KeyboardShortcutService {
  static void registerShortcuts(BuildContext context) {
    // Register global shortcuts
    final shortcuts = <LogicalKeySet, Intent>{
      LogicalKeySet(LogicalKeyboardKey.alt, LogicalKeyboardKey.keyS): const SaveIntent(),
      LogicalKeySet(LogicalKeyboardKey.alt, LogicalKeyboardKey.keyZ): const UndoIntent(),
      LogicalKeySet(LogicalKeyboardKey.alt, LogicalKeyboardKey.keyShift, LogicalKeyboardKey.keyZ): const RedoIntent(),
      LogicalKeySet(LogicalKeyboardKey.escape): const CancelIntent(),
    };

    // Add to widget tree using Shortcuts
    // This would be in the app's main widget
  }
}

class SaveIntent extends Intent {
  const SaveIntent();
}

class UndoIntent extends Intent {
  const UndoIntent();
}

class RedoIntent extends Intent {
  const RedoIntent();
}

class CancelIntent extends Intent {
  const CancelIntent();
}
```

What's happening here?
- Accessible form with keyboard navigation
- Focus traversal between fields
- Keyboard shortcut service
- Intent-based shortcuts

---

# Best Practices

## Provide Visual Focus Indicators

```dart
// Good - Clear focus indicator
Focus(
  child: Container(
    decoration: BoxDecoration(
      border: Border.all(
        color: focusNode.hasFocus ? Colors.blue : Colors.transparent,
        width: 2,
      ),
    ),
    child: child,
  ),
)

// Bad - No focus indicator
Focus(
  child: child,
)
```

## Use Logical Focus Order

```dart
// Good - Logical order
FocusTraversalGroup(
  child: Column(
    children: [
      Field1(), // First
      Field2(), // Second
      Field3(), // Third
    ],
  ),
)
```

## Support Keyboard Shortcuts

```dart
// Good - Common shortcuts
CallbackShortcuts(
  bindings: {
    const SingleActivator(LogicalKeyboardKey.keyS, control: true): () => save(),
    const SingleActivator(LogicalKeyboardKey.keyZ, control: true): () => undo(),
    const SingleActivator(LogicalKeyboardKey.escape): () => cancel(),
  },
)
```

---

# Common Mistakes

## No Keyboard Support

Wrong:
```dart
// Only touch support
GestureDetector(
  onTap: () {},
  child: Container(),
)
```

Correct:
```dart
// Keyboard support
Focus(
  child: GestureDetector(
    onTap: () {},
    child: Container(),
  ),
)
```

## No Focus Order

Wrong:
```dart
// Random focus order
Column(
  children: [
    Field2(),
    Field1(),
    Field3(),
  ],
)
```

Correct:
```dart
// Logical focus order
Column(
  children: [
    Field1(),
    Field2(),
    Field3(),
  ],
)
```

---

# Summary

Keyboard Navigation enables users to interact with apps using a keyboard. Use Focus for focus management, FocusTraversalGroup for navigation order, CallbackShortcuts for keyboard shortcuts, and provide visual focus indicators. Keyboard navigation is essential for accessibility and desktop/web apps.

---

# Next Steps

- [Accessibility Best Practices](accessibility-best-practices.md)
- [Semantics](semantics.md)
- [Screen Readers](screen-readers.md)

---

# Did You Know?

- Keyboard navigation supports motor impairments
- Tab key navigates between elements
- Enter activates focused elements
- Space toggles focused elements
- Keyboard shortcuts improve productivity
- Focus indicates current element
- FocusTraversalGroup controls order
- Accessibility is legally required