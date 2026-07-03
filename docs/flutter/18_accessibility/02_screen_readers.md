# Screen Readers

Understand how to make Flutter applications compatible with screen readers for users with visual impairments.

---

# What is it?

Screen readers are assistive technologies that convert on-screen text and elements into speech or braille output. Flutter provides built-in support for screen readers through the Semantics widget, which enables blind and visually impaired users to navigate and interact with your app using audio feedback.

---

# Why does it exist?

Screen Readers exist to:

- Make apps accessible to blind users
- Support visually impaired users
- Convert UI to speech output
- Enable audio navigation
- Provide alternative interaction
- Ensure inclusive design
- Comply with accessibility standards

---

# Basic Screen Reader Support

> **Making widgets** screen reader compatible.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic screen reader example
class BasicScreenReaderExample extends StatelessWidget {
  const BasicScreenReaderExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Screen Reader Example'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Text with semantics
            Semantics(
              label: 'Welcome message',
              child: const Text(
                'Welcome to our app!',
                style: TextStyle(fontSize: 24),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Interactive button
            Semantics(
              label: 'Click me button',
              hint: 'Double tap to activate',
              button: true,
              child: ElevatedButton(
                onPressed: () {},
                child: const Text('Click Me'),
              ),
            ),
            const SizedBox(height: 16),

            // 3. Image with description
            Semantics(
              label: 'App logo',
              image: true,
              child: const Icon(
                Icons.flutter_dash,
                size: 100,
                color: Colors.blue,
              ),
            ),
            const SizedBox(height: 16),

            // 4. List with screen reader support
            Semantics(
              label: 'User list',
              child: Column(
                children: [
                  _buildListItem('John Doe', 'john@example.com'),
                  _buildListItem('Jane Smith', 'jane@example.com'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildListItem(String name, String email) {
    return Semantics(
      label: 'User: $name, email: $email',
      hint: 'Double tap to view user details',
      child: ListTile(
        title: Text(name),
        subtitle: Text(email),
        trailing: const Icon(Icons.arrow_forward),
        onTap: () {},
      ),
    );
  }
}
```

What's happening here?
- Semantics provides screen reader information
- label describes the element
- hint explains interaction
- Roles identify element types

---

# Screen Reader Properties

> **Using** screen reader specific properties.

```dart
/// Screen reader properties example
class ScreenReaderPropertiesExample extends StatelessWidget {
  const ScreenReaderPropertiesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Screen Reader Properties'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Live region - automatically announces changes
            Semantics(
              liveRegion: true,
              label: 'Status updates',
              child: Container(
                padding: const EdgeInsets.all(8),
                color: Colors.green[100],
                child: const Text('Item added to cart'),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Focusable element
            Semantics(
              focusable: true,
              label: 'Focusable button',
              child: ElevatedButton(
                onPressed: () {},
                child: const Text('Focus Me'),
              ),
            ),
            const SizedBox(height: 16),

            // 3. Scrolling announcement
            Semantics(
              label: 'Scrollable list',
              scrollable: true,
              child: Container(
                height: 100,
                color: Colors.grey[200],
                child: const Center(
                  child: Text('Scrollable area'),
                ),
              ),
            ),
            const SizedBox(height: 16),

            // 4. Sorting announcement
            Semantics(
              label: 'Items sorted by name',
              sortKey: const OrdinalSortKey(1),
              child: const Text('A - Z'),
            ),
            const SizedBox(height: 8),
            Semantics(
              label: 'Items sorted by date',
              sortKey: const OrdinalSortKey(2),
              child: const Text('Newest first'),
            ),
            const SizedBox(height: 16),

            // 5. Container with merged semantics
            MergeSemantics(
              child: Row(
                children: [
                  Semantics(
                    label: 'Star rating: 4',
                    child: const Icon(Icons.star, color: Colors.yellow),
                  ),
                  Semantics(
                    label: 'Star rating: 4',
                    child: const Icon(Icons.star, color: Colors.yellow),
                  ),
                  Semantics(
                    label: 'Star rating: 4',
                    child: const Icon(Icons.star, color: Colors.yellow),
                  ),
                  Semantics(
                    label: 'Star rating: 4',
                    child: const Icon(Icons.star, color: Colors.yellow),
                  ),
                  Semantics(
                    label: 'Empty star',
                    child: const Icon(Icons.star_border, color: Colors.grey),
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
- liveRegion announces changes
- focusable indicates keyboard focus
- scrollable identifies scrolling
- sortKey controls reading order

---

# Custom Screen Reader Announcements

> **Creating custom** announcements.

```dart
/// Custom screen reader announcements
class CustomAnnouncementsExample extends StatefulWidget {
  const CustomAnnouncementsExample({super.key});

  @override
  State<CustomAnnouncementsExample> createState() => _CustomAnnouncementsExampleState();
}

class _CustomAnnouncementsExampleState extends State<CustomAnnouncementsExample> {
  int _count = 0;
  String _status = 'Ready';

  // 1. Custom announcement method
  void _announce(String message) {
    setState(() {
      _status = message;
    });
    // Semantics announcements are made automatically
  }

  void _increment() {
    setState(() {
      _count++;
    });
    // Announce the new count
    _announce('Counter incremented to $_count');
  }

  void _decrement() {
    setState(() {
      _count--;
    });
    // Announce the new count
    _announce('Counter decremented to $_count');
  }

  void _reset() {
    setState(() {
      _count = 0;
    });
    _announce('Counter reset to zero');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Announcements'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Live region for announcements
            Semantics(
              liveRegion: true,
              label: _status,
              child: Container(
                padding: const EdgeInsets.all(8),
                color: Colors.blue[50],
                child: Text(
                  'Status: $_status',
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Counter with announcements
            Semantics(
              label: 'Counter value: $_count',
              child: Text(
                '$_count',
                style: const TextStyle(
                  fontSize: 48,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            const SizedBox(height: 16),

            // 3. Control buttons with announcements
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Semantics(
                  label: 'Decrement counter',
                  hint: 'Decreases counter by 1',
                  child: ElevatedButton(
                    onPressed: _decrement,
                    child: const Text('-'),
                  ),
                ),
                const SizedBox(width: 8),
                Semantics(
                  label: 'Increment counter',
                  hint: 'Increases counter by 1',
                  child: ElevatedButton(
                    onPressed: _increment,
                    child: const Text('+'),
                  ),
                ),
                const SizedBox(width: 8),
                Semantics(
                  label: 'Reset counter',
                  hint: 'Resets counter to zero',
                  child: ElevatedButton(
                    onPressed: _reset,
                    child: const Text('Reset'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 16),

            // 4. Multiple announcements
            Semantics(
              label: 'Multiple buttons with announcements',
              child: Wrap(
                spacing: 8,
                children: [
                  _buildAnnounceButton('Save', 'Data saved'),
                  _buildAnnounceButton('Delete', 'Item deleted'),
                  _buildAnnounceButton('Edit', 'Editing mode enabled'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildAnnounceButton(String label, String announcement) {
    return Semantics(
      label: label,
      hint: announcement,
      child: ElevatedButton(
        onPressed: () {
          _announce(announcement);
        },
        child: Text(label),
      ),
    );
  }
}
```

What's happening here?
- liveRegion announces status changes
- Custom announcements for interactions
- Semantic hints for actions
- Real-time feedback for screen readers

---

# Testing Screen Reader Support

> **Testing** accessibility.

```dart
/// Screen reader testing
class ScreenReaderTesting extends StatelessWidget {
  const ScreenReaderTesting({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Testing Screen Readers'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Testable widget
            Semantics(
              label: 'Test button',
              hint: 'This is a test button',
              button: true,
              child: ElevatedButton(
                onPressed: () {},
                child: const Text('Test Button'),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Test info
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
                    'Testing Tips:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('• Use TalkBack (Android) or VoiceOver (iOS)'),
                  Text('• Navigate with swipes'),
                  Text('• Listen to announcements'),
                  Text('• Check for missing labels'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// Screen reader test helper
class ScreenReaderTestHelper {
  /// Test if widget has correct semantics
  static void testSemantics(
    WidgetTester tester,
    Finder finder,
    String expectedLabel,
  ) {
    final semantics = tester.getSemantics(finder);
    expect(semantics.label, contains(expectedLabel));
  }

  /// Test if widget has hint
  static void testHint(
    WidgetTester tester,
    Finder finder,
    String expectedHint,
  ) {
    final semantics = tester.getSemantics(finder);
    expect(semantics.hint, contains(expectedHint));
  }

  /// Test if widget is focusable
  static void testFocusable(
    WidgetTester tester,
    Finder finder,
    bool expected,
  ) {
    final semantics = tester.getSemantics(finder);
    expect(semantics.isFocusable, expected);
  }

  /// Test if widget has button role
  static void testButton(
    WidgetTester tester,
    Finder finder,
    bool expected,
  ) {
    final semantics = tester.getSemantics(finder);
    expect(semantics.hasAction(SemanticsAction.tap), expected);
  }
}
```

What's happening here?
- Testable screen reader widgets
- Testing helper methods
- Semantics verification
- Accessibility testing

---

# Real-World Examples

> **Common patterns** with screen readers.

```dart
/// 1. Accessible form with screen reader support
class AccessibleForm extends StatefulWidget {
  const AccessibleForm({super.key});

  @override
  State<AccessibleForm> createState() => _AccessibleFormState();
}

class _AccessibleFormState extends State<AccessibleForm> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Accessible Form'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // Name field with screen reader support
              Semantics(
                label: 'Name input field',
                textField: true,
                child: TextFormField(
                  controller: _nameController,
                  decoration: const InputDecoration(
                    labelText: 'Name',
                    hintText: 'Enter your name',
                    border: OutlineInputBorder(),
                  ),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please enter your name';
                    }
                    return null;
                  },
                ),
              ),
              const SizedBox(height: 16),

              // Email field with screen reader support
              Semantics(
                label: 'Email input field',
                textField: true,
                child: TextFormField(
                  controller: _emailController,
                  decoration: const InputDecoration(
                    labelText: 'Email',
                    hintText: 'Enter your email',
                    border: OutlineInputBorder(),
                  ),
                  keyboardType: TextInputType.emailAddress,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please enter your email';
                    }
                    if (!value.contains('@')) {
                      return 'Please enter a valid email';
                    }
                    return null;
                  },
                ),
              ),
              const SizedBox(height: 16),

              // Submit button with screen reader support
              Semantics(
                label: 'Submit form',
                hint: 'Submits the form data',
                button: true,
                child: SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () {
                      if (_formKey.currentState!.validate()) {
                        ScaffoldMessenger.of(context).showSnackBar(
                          const SnackBar(
                            content: Text('Form submitted!'),
                            backgroundColor: Colors.green,
                          ),
                        );
                      }
                    },
                    child: const Text('Submit'),
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

/// 2. Screen reader announcement service
class ScreenReaderService {
  /// Announce a message to screen readers
  static void announce(BuildContext context, String message) {
    // This will be read by screen readers
    SemanticsService.announce(
      message,
      TextDirection.ltr,
    );
  }

  /// Announce an error
  static void announceError(BuildContext context, String error) {
    announce(context, 'Error: $error');
  }

  /// Announce a success
  static void announceSuccess(BuildContext context, String message) {
    announce(context, 'Success: $message');
  }

  /// Announce navigation
  static void announceNavigation(BuildContext context, String destination) {
    announce(context, 'Navigating to $destination');
  }
}
```

What's happening here?
- Accessible form with screen reader support
- Screen reader announcement service
- Form validation announcements
- Navigation announcements

---

# Best Practices

## Provide Clear Labels

```dart
// Good - Clear descriptive label
Semantics(
  label: 'Submit contact form',
  child: ElevatedButton(...),
)

// Bad - Vague label
Semantics(
  label: 'Button',
  child: ElevatedButton(...),
)
```

## Use Live Regions for Updates

```dart
// Good - Live region for updates
Semantics(
  liveRegion: true,
  label: statusMessage,
  child: Container(...),
)
```

## Provide Hints for Actions

```dart
// Good - Action hint
Semantics(
  hint: 'Double tap to delete item',
  child: IconButton(
    icon: Icon(Icons.delete),
    onPressed: () {},
  ),
)
```

---

# Common Mistakes

## No Screen Reader Support

Wrong:
```dart
// No semantics for screen readers
IconButton(
  icon: Icon(Icons.settings),
  onPressed: () {},
)
```

Correct:
```dart
// With screen reader support
Semantics(
  label: 'Settings',
  hint: 'Opens settings screen',
  child: IconButton(
    icon: Icon(Icons.settings),
    onPressed: () {},
  ),
)
```

## Missing Announcements

Wrong:
```dart
// No announcement for changes
void _update() {
  setState(() {});
}
```

Correct:
```dart
// Announce changes
void _update() {
  setState(() {});
  SemanticsService.announce('Data updated', TextDirection.ltr);
}
```

---

# Summary

Screen Readers make apps accessible to blind and visually impaired users. Use Semantics to provide labels, hints, and roles. Use live regions for announcements, test with screen readers, and follow accessibility best practices. Screen reader support is essential for inclusive app design.

---

# Next Steps

- [Keyboard Navigation](keyboard-navigation.md)
- [Accessibility Best Practices](accessibility-best-practices.md)
- [Semantics](semantics.md)

---

# Did You Know?

- Screen readers convert UI to speech
- Semantics provides accessibility info
- Live regions announce updates
- TalkBack is Android's screen reader
- VoiceOver is iOS's screen reader
- Screen readers support gestures
- Accessibility is legally required
- Good accessibility benefits all users