# Keyboard Handling

Understand how to handle keyboard input and behavior in Flutter applications.

---

# What is it?

Keyboard handling refers to the management of keyboard input, appearance, dismissal, and behavior in Flutter applications. It covers everything from showing and hiding the keyboard, handling keyboard events, managing keyboard types, and controlling how the keyboard interacts with your UI. Proper keyboard handling is essential for creating a smooth user experience, especially on mobile devices.

---

# Why does it exist?

Keyboard handling exists to:

- Control keyboard appearance and behavior
- Handle keyboard input events
- Manage keyboard dismissal
- Support different input types
- Handle keyboard shortcuts (desktop)
- Manage keyboard focus
- Provide keyboard navigation

---

# Keyboard Types

> **Different keyboard** types for different inputs.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

/// Keyboard types example
class KeyboardTypesExample extends StatefulWidget {
  const KeyboardTypesExample({super.key});

  @override
  State<KeyboardTypesExample> createState() => _KeyboardTypesExampleState();
}

class _KeyboardTypesExampleState extends State<KeyboardTypesExample> {
  // Controllers for each field
  final TextEditingController _textController = TextEditingController();
  final TextEditingController _numberController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _phoneController = TextEditingController();
  final TextEditingController _urlController = TextEditingController();
  final TextEditingController _datetimeController = TextEditingController();

  @override
  void dispose() {
    _textController.dispose();
    _numberController.dispose();
    _emailController.dispose();
    _phoneController.dispose();
    _urlController.dispose();
    _datetimeController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Keyboard Types'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Text input (default)
            TextField(
              controller: _textController,
              decoration: const InputDecoration(
                labelText: 'Text Input',
                hintText: 'Regular text keyboard',
                border: OutlineInputBorder(),
              ),
              // Default keyboard - shows standard keyboard
              keyboardType: TextInputType.text,
            ),
            const SizedBox(height: 16),
            
            // 2. Number input
            // Shows numeric keyboard with digits
            TextField(
              controller: _numberController,
              decoration: const InputDecoration(
                labelText: 'Number Input',
                hintText: 'Numeric keyboard',
                border: OutlineInputBorder(),
              ),
              // Shows number keyboard with digits
              keyboardType: TextInputType.number,
            ),
            const SizedBox(height: 16),
            
            // 3. Email input
            // Shows email-specific keyboard with @ and .com
            TextField(
              controller: _emailController,
              decoration: const InputDecoration(
                labelText: 'Email Input',
                hintText: 'Email keyboard with @',
                border: OutlineInputBorder(),
              ),
              keyboardType: TextInputType.emailAddress,
              // Disable autocorrect for emails
              autocorrect: false,
            ),
            const SizedBox(height: 16),
            
            // 4. Phone input
            // Shows phone number keyboard
            TextField(
              controller: _phoneController,
              decoration: const InputDecoration(
                labelText: 'Phone Input',
                hintText: 'Phone number keyboard',
                border: OutlineInputBorder(),
              ),
              keyboardType: TextInputType.phone,
            ),
            const SizedBox(height: 16),
            
            // 5. URL input
            // Shows URL-specific keyboard with . and /
            TextField(
              controller: _urlController,
              decoration: const InputDecoration(
                labelText: 'URL Input',
                hintText: 'URL keyboard with / and .',
                border: OutlineInputBorder(),
              ),
              keyboardType: TextInputType.url,
              autocorrect: false,
            ),
            const SizedBox(height: 16),
            
            // 6. DateTime input
            TextField(
              controller: _datetimeController,
              decoration: const InputDecoration(
                labelText: 'Date/Time Input',
                hintText: 'Date/Time keyboard',
                border: OutlineInputBorder(),
              ),
              keyboardType: TextInputType.datetime,
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Text: Standard keyboard
- Number: Numeric keypad
- Email: Email-specific keyboard
- Phone: Phone number keyboard
- URL: URL-specific keyboard
- DateTime: DateTime keyboard

---

# Keyboard Actions

> **Controlling keyboard** actions and buttons.

```dart
/// Keyboard actions example
class KeyboardActionsExample extends StatefulWidget {
  const KeyboardActionsExample({super.key});

  @override
  State<KeyboardActionsExample> createState() => _KeyboardActionsExampleState();
}

class _KeyboardActionsExampleState extends State<KeyboardActionsExample> {
  // Controllers for form fields
  final TextEditingController _field1Controller = TextEditingController();
  final TextEditingController _field2Controller = TextEditingController();
  final TextEditingController _field3Controller = TextEditingController();
  final TextEditingController _field4Controller = TextEditingController();

  // Focus nodes for navigation
  final FocusNode _field1Focus = FocusNode();
  final FocusNode _field2Focus = FocusNode();
  final FocusNode _field3Focus = FocusNode();
  final FocusNode _field4Focus = FocusNode();

  @override
  void dispose() {
    _field1Controller.dispose();
    _field2Controller.dispose();
    _field3Controller.dispose();
    _field4Controller.dispose();
    _field1Focus.dispose();
    _field2Focus.dispose();
    _field3Focus.dispose();
    _field4Focus.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Keyboard Actions'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Text input with Done button
            // textInputAction controls the action button on the keyboard
            TextField(
              controller: _field1Controller,
              focusNode: _field1Focus,
              decoration: const InputDecoration(
                labelText: 'Field 1 - Done',
                hintText: 'Press Done to submit',
                border: OutlineInputBorder(),
              ),
              // Done button: submits the form
              textInputAction: TextInputAction.done,
              onSubmitted: (value) {
                // Called when Done is pressed
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(
                    content: Text('Submitted!'),
                    backgroundColor: Colors.green,
                  ),
                );
                // Dismiss keyboard
                _field1Focus.unfocus();
              },
            ),
            const SizedBox(height: 16),
            
            // 2. Text input with Next button
            TextField(
              controller: _field2Controller,
              focusNode: _field2Focus,
              decoration: const InputDecoration(
                labelText: 'Field 2 - Next',
                hintText: 'Press Next to go to next field',
                border: OutlineInputBorder(),
              ),
              // Next button: moves to next field
              textInputAction: TextInputAction.next,
              onSubmitted: (value) {
                // Move focus to next field
                _field3Focus.requestFocus();
              },
            ),
            const SizedBox(height: 16),
            
            // 3. Text input with Continue button
            TextField(
              controller: _field3Controller,
              focusNode: _field3Focus,
              decoration: const InputDecoration(
                labelText: 'Field 3 - Continue',
                hintText: 'Press Continue to continue',
                border: OutlineInputBorder(),
              ),
              textInputAction: TextInputAction.continueAction,
              onSubmitted: (value) {
                // Move to next field
                _field4Focus.requestFocus();
              },
            ),
            const SizedBox(height: 16),
            
            // 4. Text input with Send button
            TextField(
              controller: _field4Controller,
              focusNode: _field4Focus,
              decoration: const InputDecoration(
                labelText: 'Field 4 - Send',
                hintText: 'Press Send to send message',
                border: OutlineInputBorder(),
              ),
              textInputAction: TextInputAction.send,
              onSubmitted: (value) {
                // Send the message
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text('Sending: $value'),
                    backgroundColor: Colors.blue,
                  ),
                );
                _field4Focus.unfocus();
              },
            ),
            
            const SizedBox(height: 24),
            
            // Info about keyboard actions
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
                    'Keyboard Actions:',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  SizedBox(height: 8),
                  Text('• Done: Submit/Complete form'),
                  Text('• Next: Move to next field'),
                  Text('• Continue: Continue action'),
                  Text('• Send: Send message'),
                  Text('• Search: Perform search'),
                  Text('• Go: Navigate'),
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
- textInputAction controls keyboard button
- Done: Submits form
- Next: Moves to next field
- Continue: Continues action
- Send: Sends message

---

# Keyboard Dismissal

> **Showing and hiding** the keyboard.

```dart
/// Keyboard dismissal example
class KeyboardDismissalExample extends StatefulWidget {
  const KeyboardDismissalExample({super.key});

  @override
  State<KeyboardDismissalExample> createState() => _KeyboardDismissalExampleState();
}

class _KeyboardDismissalExampleState extends State<KeyboardDismissalExample> {
  // Controllers for fields
  final TextEditingController _controller1 = TextEditingController();
  final TextEditingController _controller2 = TextEditingController();
  
  // Focus nodes
  final FocusNode _focusNode1 = FocusNode();
  final FocusNode _focusNode2 = FocusNode();

  @override
  void dispose() {
    _controller1.dispose();
    _controller2.dispose();
    _focusNode1.dispose();
    _focusNode2.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Keyboard Dismissal'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Dismiss on submit
            TextField(
              controller: _controller1,
              focusNode: _focusNode1,
              decoration: const InputDecoration(
                labelText: 'Dismiss on Submit',
                hintText: 'Press Done to dismiss',
                border: OutlineInputBorder(),
              ),
              textInputAction: TextInputAction.done,
              onSubmitted: (value) {
                // 1. Unfocus to dismiss keyboard
                _focusNode1.unfocus();
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(
                    content: Text('Keyboard dismissed'),
                    duration: Duration(milliseconds: 500),
                  ),
                );
              },
            ),
            const SizedBox(height: 16),
            
            // 2. Tap outside to dismiss
            // GestureDetector to detect taps outside text fields
            GestureDetector(
              onTap: () {
                // 2. Unfocus all fields when tapping outside
                // This dismisses the keyboard
                FocusScope.of(context).unfocus();
              },
              child: Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.grey[100],
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Column(
                  children: [
                    TextField(
                      controller: _controller2,
                      focusNode: _focusNode2,
                      decoration: const InputDecoration(
                        labelText: 'Tap outside to dismiss',
                        hintText: 'Tap on the background',
                        border: OutlineInputBorder(),
                      ),
                    ),
                    const SizedBox(height: 16),
                    const Text(
                      'Tap anywhere outside the text field to dismiss keyboard',
                      style: TextStyle(fontSize: 14, color: Colors.grey),
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 3. Manual dismiss buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Dismiss keyboard
                ElevatedButton(
                  onPressed: () {
                    // 3. Use FocusScope to dismiss all
                    FocusScope.of(context).unfocus();
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Keyboard dismissed'),
                        duration: Duration(milliseconds: 500),
                      ),
                    );
                  },
                  child: const Text('Dismiss Keyboard'),
                ),
                const SizedBox(width: 8),
                // Focus field 1
                ElevatedButton(
                  onPressed: () {
                    _focusNode1.requestFocus();
                  },
                  child: const Text('Focus Field 1'),
                ),
              ],
            ),
            
            const SizedBox(height: 16),
            
            // 4. Resize view when keyboard appears
            // Using SingleChildScrollView automatically handles this
            const Text(
              'Note: Using SingleChildScrollView helps with keyboard resizing',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- unfocus() dismisses keyboard
- FocusScope.unfocus() dismisses all
- GestureDetector detects outside taps
- SingleChildScrollView handles resizing

---

# Keyboard Shortcuts

> **Handling keyboard** shortcuts (desktop/web).

```dart
/// Keyboard shortcuts example
class KeyboardShortcutsExample extends StatefulWidget {
  const KeyboardShortcutsExample({super.key});

  @override
  State<KeyboardShortcutsExample> createState() => _KeyboardShortcutsExampleState();
}

class _KeyboardShortcutsExampleState extends State<KeyboardShortcutsExample> {
  // Controllers for text fields
  final TextEditingController _controller = TextEditingController();
  String _shortcutMessage = 'Press a shortcut key...';

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

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
            // 1. SingleShortcut widget for keyboard shortcuts
            // This detects specific key combinations
            CallbackShortcuts(
              bindings: {
                // Ctrl+S to save
                const SingleActivator(LogicalKeyboardKey.keyS, control: true): () {
                  setState(() {
                    _shortcutMessage = 'Ctrl+S pressed - Save action';
                  });
                },
                // Ctrl+Z to undo
                const SingleActivator(LogicalKeyboardKey.keyZ, control: true): () {
                  setState(() {
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
                    _shortcutMessage = 'Ctrl+Shift+Z pressed - Redo action';
                  });
                },
                // Escape to cancel
                const SingleActivator(LogicalKeyboardKey.escape): () {
                  setState(() {
                    _shortcutMessage = 'Escape pressed - Cancel action';
                  });
                },
              },
              child: Focus(
                autofocus: true,
                child: Container(
                  padding: const EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: Colors.blue[50],
                    borderRadius: BorderRadius.circular(8),
                    border: Border.all(color: Colors.blue[200]!),
                  ),
                  child: Column(
                    children: [
                      const Text(
                        'Focus here and try shortcuts:',
                        style: TextStyle(fontWeight: FontWeight.bold),
                      ),
                      const SizedBox(height: 8),
                      Text(
                        _shortcutMessage,
                        style: const TextStyle(fontSize: 16),
                      ),
                      const SizedBox(height: 8),
                      const Text(
                        'Ctrl+S, Ctrl+Z, Ctrl+Shift+Z, Escape',
                        style: TextStyle(fontSize: 12, color: Colors.grey),
                      ),
                    ],
                  ),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 2. Text field with keyboard shortcuts
            TextField(
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'Try shortcuts in text field',
                hintText: 'Type something...',
                border: OutlineInputBorder(),
                helperText: 'Ctrl+A to select all, Ctrl+C to copy',
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 3. Shortcut info
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Common Shortcuts:',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  SizedBox(height: 8),
                  Text('• Ctrl+A: Select all'),
                  Text('• Ctrl+C: Copy'),
                  Text('• Ctrl+V: Paste'),
                  Text('• Ctrl+X: Cut'),
                  Text('• Ctrl+Z: Undo'),
                  Text('• Ctrl+S: Save'),
                  Text('• Escape: Cancel'),
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
- CallbackShortcuts for key bindings
- SingleActivator defines key combinations
- Focus required for shortcut detection
- Desktop/web keyboard shortcuts

---

# Real-World Examples

> **Common keyboard** handling patterns.

```dart
/// 1. Search field with keyboard handling
class SearchFieldWithKeyboard extends StatefulWidget {
  const SearchFieldWithKeyboard({super.key});

  @override
  State<SearchFieldWithKeyboard> createState() => _SearchFieldWithKeyboardState();
}

class _SearchFieldWithKeyboardState extends State<SearchFieldWithKeyboard> {
  final TextEditingController _searchController = TextEditingController();
  final FocusNode _searchFocus = FocusNode();
  List<String> _results = [];
  final List<String> _allItems = [
    'Apple', 'Banana', 'Orange', 'Grape', 'Watermelon',
    'Strawberry', 'Pineapple', 'Mango', 'Peach', 'Pear',
  ];

  @override
  void dispose() {
    _searchController.dispose();
    _searchFocus.dispose();
    super.dispose();
  }

  void _performSearch(String query) {
    setState(() {
      if (query.isEmpty) {
        _results = [];
      } else {
        _results = _allItems
            .where((item) => item
                .toLowerCase()
                .contains(query.toLowerCase()))
            .toList();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Search with Keyboard'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Search field with keyboard handling
            TextField(
              controller: _searchController,
              focusNode: _searchFocus,
              decoration: InputDecoration(
                hintText: 'Search...',
                prefixIcon: const Icon(Icons.search),
                // Clear button
                suffixIcon: _searchController.text.isNotEmpty
                    ? IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          _searchController.clear();
                          _performSearch('');
                        },
                      )
                    : null,
                border: const OutlineInputBorder(
                  borderRadius: BorderRadius.all(Radius.circular(12)),
                ),
                filled: true,
                fillColor: Colors.grey[50],
              ),
              // Search action on keyboard
              textInputAction: TextInputAction.search,
              // Trigger search when submitted
              onSubmitted: _performSearch,
              // Real-time search as user types
              onChanged: _performSearch,
            ),
            
            const SizedBox(height: 16),
            
            // Show results or empty state
            Expanded(
              child: _results.isEmpty
                  ? Center(
                      child: Text(
                        _searchController.text.isEmpty
                            ? 'Start typing to search'
                            : 'No results found',
                        style: TextStyle(
                          color: Colors.grey[600],
                          fontSize: 16,
                        ),
                      ),
                    )
                  : ListView.builder(
                      itemCount: _results.length,
                      itemBuilder: (context, index) {
                        return ListTile(
                          leading: const Icon(Icons.search),
                          title: Text(_results[index]),
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

/// 2. Form with keyboard navigation
class KeyboardNavigationForm extends StatefulWidget {
  const KeyboardNavigationForm({super.key});

  @override
  State<KeyboardNavigationForm> createState() => _KeyboardNavigationFormState();
}

class _KeyboardNavigationFormState extends State<KeyboardNavigationForm> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Controllers
  final TextEditingController _firstNameController = TextEditingController();
  final TextEditingController _lastNameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  
  // Focus nodes for navigation
  final FocusNode _firstNameFocus = FocusNode();
  final FocusNode _lastNameFocus = FocusNode();
  final FocusNode _emailFocus = FocusNode();
  final FocusNode _passwordFocus = FocusNode();

  @override
  void dispose() {
    _firstNameController.dispose();
    _lastNameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    _firstNameFocus.dispose();
    _lastNameFocus.dispose();
    _emailFocus.dispose();
    _passwordFocus.dispose();
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
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // First name
              TextFormField(
                controller: _firstNameController,
                focusNode: _firstNameFocus,
                decoration: const InputDecoration(
                  labelText: 'First Name',
                  hintText: 'Enter first name',
                  border: OutlineInputBorder(),
                ),
                // Next button to move to last name
                textInputAction: TextInputAction.next,
                onFieldSubmitted: (_) {
                  _lastNameFocus.requestFocus();
                },
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'First name required';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // Last name
              TextFormField(
                controller: _lastNameController,
                focusNode: _lastNameFocus,
                decoration: const InputDecoration(
                  labelText: 'Last Name',
                  hintText: 'Enter last name',
                  border: OutlineInputBorder(),
                ),
                textInputAction: TextInputAction.next,
                onFieldSubmitted: (_) {
                  _emailFocus.requestFocus();
                },
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Last name required';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // Email
              TextFormField(
                controller: _emailController,
                focusNode: _emailFocus,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  hintText: 'Enter email',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.emailAddress,
                textInputAction: TextInputAction.next,
                onFieldSubmitted: (_) {
                  _passwordFocus.requestFocus();
                },
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Email required';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // Password
              TextFormField(
                controller: _passwordController,
                focusNode: _passwordFocus,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter password',
                  border: OutlineInputBorder(),
                ),
                obscureText: true,
                // Done button to submit
                textInputAction: TextInputAction.done,
                onFieldSubmitted: (_) {
                  // Submit form when done is pressed
                  if (_formKey.currentState!.validate()) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Form submitted!'),
                        backgroundColor: Colors.green,
                      ),
                    );
                    // Dismiss keyboard
                    FocusScope.of(context).unfocus();
                  }
                },
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Password required';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              
              // Submit button
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: () {
                    if (_formKey.currentState!.validate()) {
                      // Form is valid
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('Form submitted!'),
                          backgroundColor: Colors.green,
                        ),
                      );
                      FocusScope.of(context).unfocus();
                    }
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
```

What's happening here?
- Search with keyboard actions
- Form with keyboard navigation
- Next/Done buttons for form fields
- Keyboard focus traversal

---

# Best Practices

## Use Appropriate Keyboard Type

```dart
// Good - Correct keyboard type
TextField(
  keyboardType: TextInputType.emailAddress,
  decoration: InputDecoration(labelText: 'Email'),
)

// Bad - Wrong keyboard type
TextField(
  keyboardType: TextInputType.text,
  decoration: InputDecoration(labelText: 'Email'),
)
```

## Handle Keyboard Dismissal

```dart
// Good - Dismiss keyboard when done
onFieldSubmitted: (value) {
  FocusScope.of(context).unfocus();
}

// Bad - Keep keyboard open
onFieldSubmitted: (value) {
  // Keyboard stays open
}
```

## Use textInputAction for Navigation

```dart
// Good - Proper navigation
TextField(
  textInputAction: TextInputAction.next,
  onFieldSubmitted: (_) => nextFocus.requestFocus(),
)

// Bad - No navigation
TextField(
  // No textInputAction
  // User has to tap the next field manually
)
```

---

# Common Mistakes

## Not Dismissing Keyboard

Wrong:
```dart
// Keyboard stays open after submit
onSubmitted: (value) {
  submitForm();
}
```

Correct:
```dart
// Dismiss keyboard after submit
onSubmitted: (value) {
  submitForm();
  FocusScope.of(context).unfocus();
}
```

## Wrong Keyboard Type

Wrong:
```dart
// Shows text keyboard for phone number
TextField(
  keyboardType: TextInputType.text,
  decoration: InputDecoration(labelText: 'Phone'),
)
```

Correct:
```dart
// Shows phone keyboard
TextField(
  keyboardType: TextInputType.phone,
  decoration: InputDecoration(labelText: 'Phone'),
)
```

---

# Summary

Keyboard handling controls keyboard appearance, behavior, and dismissal. Use keyboardType for appropriate keyboards, textInputAction for button actions, and unfocus() for dismissal. Proper keyboard handling improves user experience, especially on mobile devices.

---

# Next Steps

- [Input Formatters](input-formatters.md)
- [Focus](focus.md)
- [Forms & Input Overview](forms-input-overview.md)

---

# Did You Know?

- keyboardType controls keyboard appearance
- textInputAction controls keyboard button
- unfocus() dismisses keyboard
- FocusScope manages focus groups
- CallbackShortcuts handle keyboard shortcuts
- SingleChildScrollView handles keyboard resizing
- GestureDetector can detect outside taps
- Keyboard shortcuts work on desktop/web