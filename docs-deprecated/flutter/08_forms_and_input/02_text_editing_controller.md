# TextEditingController

Understand how to control and manage text input programmatically using TextEditingController.

---

# What is it?

TextEditingController is a controller class that manages the state of a TextField or TextFormField. It provides programmatic control over the text content, selection, and composition, allowing you to read, update, and manipulate text input. It is essential for form handling, real-time validation, and controlling text fields programmatically.

---

# Why does it exist?

TextEditingController exists to:

- Manage text field state programmatically
- Read and modify text content
- Control text selection and cursor position
- Listen to text changes
- Clear or set text values
- Handle form data and validation
- Enable real-time text processing

---

# Basic TextEditingController

> **Creating and using** a TextEditingController.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic TextEditingController example
class BasicControllerExample extends StatefulWidget {
  const BasicControllerExample({super.key});

  @override
  State<BasicControllerExample> createState() => _BasicControllerExampleState();
}

class _BasicControllerExampleState extends State<BasicControllerExample> {
  // 1. Create a TextEditingController instance
  // This controller manages the text field's state
  final TextEditingController _controller = TextEditingController();

  @override
  void initState() {
    super.initState();
    
    // 2. Add a listener to the controller
    // This listener is called whenever the text changes
    // It's useful for real-time validation or updating other UI elements
    _controller.addListener(() {
      // Access the current text using _controller.text
      print('Current text: ${_controller.text}');
      
      // You can perform any action here when text changes
      // Example: update a character counter, validate input, etc.
    });
  }

  @override
  void dispose() {
    // 3. Always dispose the controller when the widget is removed
    // This prevents memory leaks
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('TextEditingController'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            // 4. Use the controller with a TextField
            // The controller manages this TextField's state
            TextField(
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'Enter text',
                hintText: 'Type something...',
                border: OutlineInputBorder(),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 5. Display the current text
            // We can access the text at any time using _controller.text
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'Current Text:',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    // Access the text property to get the current value
                    _controller.text.isEmpty ? '(empty)' : _controller.text,
                    style: const TextStyle(fontSize: 16),
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 6. Programmatic control buttons
            // These demonstrate how to manipulate the controller
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Set text programmatically
                ElevatedButton(
                  onPressed: () {
                    // Set the text using the text setter
                    // This updates the text field and triggers listeners
                    _controller.text = 'Hello World!';
                  },
                  child: const Text('Set Text'),
                ),
                const SizedBox(width: 8),
                
                // Clear the text
                ElevatedButton(
                  onPressed: () {
                    // Clear the text using the clear() method
                    // This is equivalent to setting text to empty string
                    _controller.clear();
                  },
                  child: const Text('Clear'),
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
- TextEditingController manages text state
- addListener listens for text changes
- controller.text accesses current text
- controller.clear() clears the text
- dispose prevents memory leaks

---

# Controller Methods and Properties

> **Common methods** and properties of TextEditingController.

```dart
/// Demonstration of controller methods
class ControllerMethodsExample extends StatefulWidget {
  const ControllerMethodsExample({super.key});

  @override
  State<ControllerMethodsExample> createState() => _ControllerMethodsExampleState();
}

class _ControllerMethodsExampleState extends State<ControllerMethodsExample> {
  // Create the controller
  final TextEditingController _controller = TextEditingController(
    // Optional: Set initial text when creating the controller
    // text: 'Initial text here',
  );

  // Track selection data
  String _selectionInfo = 'No selection';

  @override
  void initState() {
    super.initState();
    // Add listener to update selection info
    _controller.addListener(_updateSelectionInfo);
  }

  // Update selection information when text changes
  void _updateSelectionInfo() {
    // Get the current selection from the controller
    final selection = _controller.selection;
    
    // Check if there is a valid selection
    if (selection.isValid) {
      // Get the selected text using the selection range
      final selectedText = _controller.text.substring(
        selection.start,
        selection.end,
      );
      
      setState(() {
        _selectionInfo = 'Selected: "$selectedText" '
                        '(from ${selection.start} to ${selection.end})';
      });
    } else {
      setState(() {
        _selectionInfo = 'No selection';
      });
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Controller Methods'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // TextField with the controller
            TextField(
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'Edit this text',
                hintText: 'Type something...',
                border: OutlineInputBorder(),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // Display selection info
            Container(
              padding: const EdgeInsets.all(12),
              color: Colors.blue[50],
              child: Text(
                _selectionInfo,
                style: const TextStyle(fontSize: 14),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // Various controller operations
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                // 1. Get text
                _buildActionButton(
                  'Get Text',
                  () {
                    // Access the current text
                    final text = _controller.text;
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text('Text: "$text"')),
                    );
                  },
                ),
                
                // 2. Set text
                _buildActionButton(
                  'Set Text',
                  () {
                    // Set new text
                    _controller.text = 'New text set at ${DateTime.now().toLocal()}';
                  },
                ),
                
                // 3. Clear text
                _buildActionButton(
                  'Clear',
                  () {
                    // Clear the text field
                    _controller.clear();
                  },
                ),
                
                // 4. Select all text
                _buildActionButton(
                  'Select All',
                  () {
                    // Select all text in the field
                    _controller.selection = TextSelection(
                      baseOffset: 0,
                      extentOffset: _controller.text.length,
                    );
                  },
                ),
                
                // 5. Get selection
                _buildActionButton(
                  'Get Selection',
                  () {
                    // Get current selection
                    final selection = _controller.selection;
                    final text = _controller.text;
                    
                    // Show selection info
                    if (selection.isValid) {
                      final selected = text.substring(selection.start, selection.end);
                      ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(content: Text('Selected: "$selected"')),
                      );
                    } else {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(content: Text('No text selected')),
                      );
                    }
                  },
                ),
                
                // 6. Set cursor position
                _buildActionButton(
                  'Cursor at End',
                  () {
                    // Move cursor to the end of the text
                    _controller.selection = TextSelection.fromPosition(
                      TextPosition(offset: _controller.text.length),
                    );
                  },
                ),
                
                // 7. Replace text
                _buildActionButton(
                  'Replace Text',
                  () {
                    // Replace a portion of text
                    final text = _controller.text;
                    if (text.isNotEmpty) {
                      // Replace the first word with "Hello"
                      final firstSpace = text.indexOf(' ');
                      final start = 0;
                      final end = firstSpace == -1 ? text.length : firstSpace;
                      
                      // Create new text with replacement
                      final newText = 'Hello${text.substring(end)}';
                      _controller.text = newText;
                    }
                  },
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  // Helper to build action buttons
  Widget _buildActionButton(String label, VoidCallback onPressed) {
    return ElevatedButton(
      onPressed: onPressed,
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.blue[50],
        foregroundColor: Colors.blue[900],
      ),
      child: Text(label),
    );
  }
}
```

What's happening here?
- controller.text: Get or set text
- controller.clear(): Clear text
- controller.selection: Get/set selection
- controller.addListener: Listen to changes
- TextSelection manages cursor and selection

---

# Controller with TextEditingController Listeners

> **Using listeners** to react to text changes.

```dart
/// Advanced listener usage with TextEditingController
class ListenerExample extends StatefulWidget {
  const ListenerExample({super.key});

  @override
  State<ListenerExample> createState() => _ListenerExampleState();
}

class _ListenerExampleState extends State<ListenerExample> {
  // Create the controller
  final TextEditingController _controller = TextEditingController();
  
  // State variables for tracking changes
  int _characterCount = 0;
  bool _isValid = false;
  String _lastChange = 'None';

  @override
  void initState() {
    super.initState();
    
    // Add a listener to the controller
    // This listener is called every time the text changes
    _controller.addListener(() {
      // Update state based on text changes
      setState(() {
        // Get the current text
        final text = _controller.text;
        
        // Update character count
        _characterCount = text.length;
        
        // Validate the text (example: at least 3 characters)
        _isValid = text.length >= 3;
        
        // Track last change time
        _lastChange = DateTime.now().toLocal().toString();
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Controller Listeners'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Text field with real-time feedback
            TextField(
              controller: _controller,
              decoration: InputDecoration(
                labelText: 'Type something...',
                hintText: 'Minimum 3 characters',
                border: const OutlineInputBorder(),
                // Show character counter
                helperText: '$_characterCount characters',
                // Show validation state with colors
                helperStyle: TextStyle(
                  color: _isValid ? Colors.green : Colors.grey,
                ),
                // Show error if invalid
                errorText: _characterCount > 0 && !_isValid 
                    ? 'Minimum 3 characters required' 
                    : null,
              ),
            ),
            
            const SizedBox(height: 16),
            
            // Real-time feedback
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: _isValid ? Colors.green[50] : Colors.grey[50],
                borderRadius: BorderRadius.circular(8),
                border: Border.all(
                  color: _isValid ? Colors.green : Colors.grey,
                ),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    children: [
                      // Status indicator
                      Icon(
                        _isValid ? Icons.check_circle : Icons.info,
                        color: _isValid ? Colors.green : Colors.grey,
                      ),
                      const SizedBox(width: 8),
                      Text(
                        _isValid ? 'Valid Input' : 'Invalid Input',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          color: _isValid ? Colors.green : Colors.grey,
                        ),
                      ),
                      const Spacer(),
                      Text(
                        '$_characterCount chars',
                        style: const TextStyle(fontSize: 12),
                      ),
                    ],
                  ),
                  const SizedBox(height: 8),
                  Text(
                    'Text: "${_controller.text.isEmpty ? 'empty' : _controller.text}"',
                    style: const TextStyle(fontSize: 14),
                  ),
                  Text(
                    'Last changed: $_lastChange',
                    style: const TextStyle(fontSize: 12, color: Colors.grey),
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // Control buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Add text
                ElevatedButton.icon(
                  onPressed: () {
                    // Append text to the current text
                    final current = _controller.text;
                    _controller.text = current + ' appended';
                  },
                  icon: const Icon(Icons.add),
                  label: const Text('Append'),
                ),
                const SizedBox(width: 8),
                // Clear text
                ElevatedButton.icon(
                  onPressed: _controller.clear,
                  icon: const Icon(Icons.clear),
                  label: const Text('Clear'),
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
- addListener triggers on every change
- Real-time validation and feedback
- Character counting
- Status indicators

---

# Controller with Focus Management

> **Managing focus** with TextEditingController.

```dart
/// Focus management with TextEditingController
class FocusControllerExample extends StatefulWidget {
  const FocusControllerExample({super.key});

  @override
  State<FocusControllerExample> createState() => _FocusControllerExampleState();
}

class _FocusControllerExampleState extends State<FocusControllerExample> {
  // Controllers for each field
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  
  // Focus nodes for each field
  // FocusNode manages focus state for a widget
  final FocusNode _nameFocus = FocusNode();
  final FocusNode _emailFocus = FocusNode();
  final FocusNode _passwordFocus = FocusNode();

  @override
  void initState() {
    super.initState();
    
    // Add listeners to handle focus events
    // This is useful for focusing the next field when done
    _nameFocus.addListener(() {
      // When name field loses focus, move to email field
      if (!_nameFocus.hasFocus) {
        // Request focus on the email field
        // This moves the cursor to the email field
        _emailFocus.requestFocus();
      }
    });
  }

  @override
  void dispose() {
    // Dispose all controllers and focus nodes
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    _nameFocus.dispose();
    _emailFocus.dispose();
    _passwordFocus.dispose();
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
            // 1. Name field with focus management
            TextField(
              controller: _nameController,
              focusNode: _nameFocus,
              decoration: const InputDecoration(
                labelText: 'Name',
                hintText: 'Enter your name',
                border: OutlineInputBorder(),
              ),
              // Called when user presses "Next" on keyboard
              // This moves focus to the email field
              onEditingComplete: () {
                // Request focus on email field
                _emailFocus.requestFocus();
              },
            ),
            const SizedBox(height: 16),
            
            // 2. Email field
            TextField(
              controller: _emailController,
              focusNode: _emailFocus,
              decoration: const InputDecoration(
                labelText: 'Email',
                hintText: 'Enter your email',
                border: OutlineInputBorder(),
              ),
              keyboardType: TextInputType.emailAddress,
              // Move to password field when next is pressed
              onEditingComplete: () {
                _passwordFocus.requestFocus();
              },
            ),
            const SizedBox(height: 16),
            
            // 3. Password field
            TextField(
              controller: _passwordController,
              focusNode: _passwordFocus,
              decoration: const InputDecoration(
                labelText: 'Password',
                hintText: 'Enter your password',
                border: OutlineInputBorder(),
              ),
              obscureText: true,
              // Callback when user submits (presses done)
              // This typically performs the form submission
              onEditingComplete: () {
                // Submit the form
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(
                    content: Text('Form submitted!'),
                    backgroundColor: Colors.green,
                  ),
                );
                // Dismiss keyboard
                _passwordFocus.unfocus();
              },
            ),
            
            const SizedBox(height: 24),
            
            // Focus control buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Focus name field
                ElevatedButton(
                  onPressed: () {
                    // Request focus on name field
                    _nameFocus.requestFocus();
                  },
                  child: const Text('Focus Name'),
                ),
                const SizedBox(width: 8),
                // Focus email field
                ElevatedButton(
                  onPressed: () {
                    _emailFocus.requestFocus();
                  },
                  child: const Text('Focus Email'),
                ),
                const SizedBox(width: 8),
                // Unfocus all (dismiss keyboard)
                ElevatedButton(
                  onPressed: () {
                    // Unfocus all fields (hides keyboard)
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
- FocusNode manages field focus
- requestFocus() focuses a field
- unfocus() dismisses keyboard
- onEditingComplete handles submit

---

# Real-World Examples

> **Common patterns** with TextEditingController.

```dart
/// 1. Real-time search with controller
class RealTimeSearchExample extends StatefulWidget {
  const RealTimeSearchExample({super.key});

  @override
  State<RealTimeSearchExample> createState() => _RealTimeSearchExampleState();
}

class _RealTimeSearchExampleState extends State<RealTimeSearchExample> {
  // Controller for search field
  final TextEditingController _searchController = TextEditingController();
  
  // Timer for debouncing
  Timer? _debounceTimer;
  
  // List to store search results
  List<String> _results = [];
  
  // Sample data
  final List<String> _allItems = [
    'Apple', 'Banana', 'Orange', 'Grape', 'Watermelon',
    'Strawberry', 'Pineapple', 'Mango', 'Peach', 'Pear',
    'Cherry', 'Lemon', 'Lime', 'Berry', 'Melon',
  ];

  @override
  void initState() {
    super.initState();
    // Add listener for real-time search
    _searchController.addListener(_onSearchChanged);
  }

  @override
  void dispose() {
    _debounceTimer?.cancel();
    _searchController.dispose();
    super.dispose();
  }

  // Called when search text changes
  void _onSearchChanged() {
    // Cancel any pending debounce
    _debounceTimer?.cancel();
    
    // Debounce search to avoid heavy operations
    _debounceTimer = Timer(const Duration(milliseconds: 300), () {
      final query = _searchController.text;
      
      setState(() {
        if (query.isEmpty) {
          _results = [];
        } else {
          // Filter items based on query
          _results = _allItems
              .where((item) => item
                  .toLowerCase()
                  .contains(query.toLowerCase()))
              .toList();
        }
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Real-Time Search'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Search field with clear button
            TextField(
              controller: _searchController,
              decoration: InputDecoration(
                hintText: 'Search fruits...',
                prefixIcon: const Icon(Icons.search),
                suffixIcon: _searchController.text.isNotEmpty
                    ? IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          // Clear the search field
                          _searchController.clear();
                        },
                      )
                    : null,
                border: const OutlineInputBorder(
                  borderRadius: BorderRadius.all(Radius.circular(12)),
                ),
                filled: true,
                fillColor: Colors.grey[50],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // Show results
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
                          // Highlight matching text
                          subtitle: Text(
                            'Contains: ${_searchController.text}',
                            style: const TextStyle(
                              fontSize: 12,
                              color: Colors.grey,
                            ),
                          ),
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

/// 2. Auto-resizing text field
class AutoResizeTextExample extends StatefulWidget {
  const AutoResizeTextExample({super.key});

  @override
  State<AutoResizeTextExample> createState() => _AutoResizeTextExampleState();
}

class _AutoResizeTextExampleState extends State<AutoResizeTextExample> {
  final TextEditingController _controller = TextEditingController();
  final int _maxLines = 10;

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Auto-Resize TextField'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Auto-resizing text field
            TextField(
              controller: _controller,
              maxLines: _maxLines,
              minLines: 1,
              decoration: const InputDecoration(
                labelText: 'Write something...',
                hintText: 'This field grows as you type',
                border: OutlineInputBorder(),
                helperText: 'Press Enter to add new lines',
              ),
              // Called when user presses Enter
              onSubmitted: (value) {
                // This doesn't close the field; it just adds a new line
              },
            ),
            
            const SizedBox(height: 16),
            
            // Character and line count
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        'Statistics',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 14,
                        ),
                      ),
                      const SizedBox(height: 4),
                      Text('Characters: ${_controller.text.length}'),
                      Text('Words: ${_controller.text.split(RegExp(r'\s+')).where((w) => w.isNotEmpty).length}'),
                    ],
                  ),
                  Column(
                    crossAxisAlignment: CrossAxisAlignment.end,
                    children: [
                      const Text(
                        'Lines',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 14,
                        ),
                      ),
                      const SizedBox(height: 4),
                      Text('${_controller.text.split('\n').length}'),
                      Text('Max: $_maxLines'),
                    ],
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
- Real-time search with debouncing
- Auto-resizing text field
- Character and word counting
- Clear button in search field

---

# Best Practices

## Always Dispose Controllers

```dart
// Good - Dispose in dispose()
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final TextEditingController _controller = TextEditingController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

## Use Listeners for Real-Time Updates

```dart
// Good - Listen for changes
@override
void initState() {
  super.initState();
  _controller.addListener(() {
    // React to changes
  });
}
```

## Clear Controllers When Needed

```dart
// Good - Clear text
_controller.clear();

// Or
_controller.text = '';
```

---

# Common Mistakes

## Not Disposing Controller

Wrong:
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final TextEditingController _controller = TextEditingController();
  // No dispose - memory leak
}
```

Correct:
```dart
class _MyWidgetState extends State<MyWidget> {
  final TextEditingController _controller = TextEditingController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

## Not Using TextEditingController for Custom Text

Wrong:
```dart
// Setting initial value without controller
TextField(
  initialValue: 'Initial text', // Works but less control
)
```

Correct:
```dart
// Using controller for full control
final _controller = TextEditingController(text: 'Initial text');
TextField(controller: _controller)
```

---

# Summary

TextEditingController provides programmatic control over TextField and TextFormField. Use it to read, write, and manage text content, selection, and focus. Always dispose controllers to prevent memory leaks. Use listeners for real-time updates and validation.

---

# Next Steps

- [Form](form.md)
- [Form Validation](form-validation.md)
- [Focus](focus.md)

---

# Did You Know?

- TextEditingController must be disposed
- controller.text gets/sets text
- controller.clear() clears text
- controller.selection manages cursor
- addListener reacts to changes
- FocusNode manages keyboard focus
- Controllers can have initial text
- TextEditingController is not thread-safe