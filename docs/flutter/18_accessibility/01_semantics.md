# Semantics

Understand how to make Flutter applications accessible using the Semantics widget.

---

# What is it?

Semantics is a widget that attaches semantic information to the widget tree, making your app accessible to screen readers and other assistive technologies. It provides a way to describe the meaning and purpose of UI elements, enabling users with disabilities to understand and interact with your app.

---

# Why does it exist?

Semantics exists to:

- Make apps accessible to screen readers
- Provide meaningful descriptions of UI elements
- Support assistive technologies
- Enable keyboard navigation
- Improve app accessibility
- Comply with accessibility standards
- Create inclusive user experiences

---

# Basic Semantics

> **Adding** semantic information to widgets.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic Semantics example
class BasicSemanticsExample extends StatelessWidget {
  const BasicSemanticsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Semantics Example'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Widget without semantics (less accessible)
            const Text(
              'Widget without Semantics',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Container(
              width: 100,
              height: 100,
              color: Colors.blue,
              child: const Center(
                child: Text(
                  'Tap me',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Widget with semantics (accessible)
            const Text(
              'Widget with Semantics',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              // 1. Label describes the widget
              label: 'Blue square button, tap to perform action',
              // 2. Hint describes what happens
              hint: 'Opens a dialog when tapped',
              // 3. Button role indicates it's a button
              button: true,
              // 4. Custom action
              onTap: () {
                // This is read by screen readers
                print('Button tapped');
              },
              child: Container(
                width: 100,
                height: 100,
                color: Colors.green,
                child: const Center(
                  child: Text(
                    'Tap me',
                    style: TextStyle(color: Colors.white),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 16),

            // 3. Image with semantics
            const Text(
              'Image with Semantics',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              label: 'Flutter logo',
              image: true,
              child: const Icon(
                Icons.flutter_dash,
                size: 100,
                color: Colors.blue,
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
- Semantics adds accessibility information
- label describes the widget
- hint describes what happens on interaction
- button role indicates button behavior

---

# Semantic Properties

> **Using different** semantic properties.

```dart
/// Semantic properties example
class SemanticPropertiesExample extends StatelessWidget {
  const SemanticPropertiesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Semantic Properties'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Label and hint
            Semantics(
              label: 'Submit form button',
              hint: 'Submits the form data',
              child: ElevatedButton(
                onPressed: () {},
                child: const Text('Submit'),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Value and state
            Semantics(
              label: 'Volume slider',
              value: '50%',
              increasedValue: '60%',
              decreasedValue: '40%',
              child: Slider(
                value: 0.5,
                onChanged: (value) {},
              ),
            ),
            const SizedBox(height: 16),

            // 3. Toggle state
            Semantics(
              label: 'Dark mode switch',
              value: 'On',
              child: Switch(
                value: true,
                onChanged: (value) {},
              ),
            ),
            const SizedBox(height: 16),

            // 4. Progress indicator
            Semantics(
              label: 'Download progress',
              value: '75%',
              child: LinearProgressIndicator(
                value: 0.75,
              ),
            ),
            const SizedBox(height: 16),

            // 5. List item with semantics
            Semantics(
              label: 'Item 1: Apple',
              hint: 'Tap to select',
              selected: true,
              child: ListTile(
                title: const Text('Apple'),
                leading: const Icon(Icons.food),
                trailing: const Icon(Icons.check),
              ),
            ),
            Semantics(
              label: 'Item 2: Banana',
              hint: 'Tap to select',
              selected: false,
              child: ListTile(
                title: const Text('Banana'),
                leading: const Icon(Icons.food),
              ),
            ),
            const SizedBox(height: 16),

            // 6. Header
            Semantics(
              label: 'Section Header',
              header: true,
              child: Container(
                padding: const EdgeInsets.all(8),
                color: Colors.blue[100],
                child: const Text(
                  'Fruits Section',
                  style: TextStyle(
                    fontSize: 20,
                    fontWeight: FontWeight.bold,
                  ),
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
- label describes the element
- value provides current value
- header indicates section headers
- selected indicates selection state

---

# Exclude Semantics

> **Excluding** elements from accessibility.

```dart
/// Exclude Semantics example
class ExcludeSemanticsExample extends StatelessWidget {
  const ExcludeSemanticsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Exclude Semantics'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Widget with semantics
            const Text(
              'Widget with Semantics',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              label: 'Important text',
              child: Container(
                padding: const EdgeInsets.all(16),
                color: Colors.green[100],
                child: const Text('This text is accessible'),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Widget without semantics (excluded)
            const Text(
              'Widget without Semantics (Excluded)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            ExcludeSemantics(
              excluding: true,
              child: Container(
                padding: const EdgeInsets.all(16),
                color: Colors.red[100],
                child: const Text('This text is NOT accessible'),
              ),
            ),
            const SizedBox(height: 16),

            // 3. Partially excluded
            const Text(
              'Partially Excluded',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Column(
              children: [
                ExcludeSemantics(
                  excluding: true,
                  child: Container(
                    padding: const EdgeInsets.all(8),
                    color: Colors.grey[200],
                    child: const Text('This is excluded'),
                  ),
                ),
                const SizedBox(height: 8),
                Container(
                  padding: const EdgeInsets.all(8),
                  color: Colors.blue[100],
                  child: const Text('This is included'),
                ),
              ],
            ),
            const SizedBox(height: 16),

            // 4. Exclude children only
            const Text(
              'Exclude Children Only',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              label: 'Container with decorative children',
              child: Container(
                padding: const EdgeInsets.all(16),
                color: Colors.orange[100],
                child: Column(
                  children: [
                    ExcludeSemantics(
                      excluding: true,
                      child: const Icon(Icons.decorative, color: Colors.grey),
                    ),
                    const SizedBox(height: 8),
                    const Text('This text is included'),
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
- ExcludeSemantics removes from accessibility
- excluding: true excludes the child
- Can partially exclude children
- Useful for decorative elements

---

# Merge Semantics

> **Merging** semantic information.

```dart
/// Merge Semantics example
class MergeSemanticsExample extends StatelessWidget {
  const MergeSemanticsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Merge Semantics'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Without merging (separate elements)
            const Text(
              'Without Merging',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              label: 'First label',
              child: Container(
                padding: const EdgeInsets.all(8),
                color: Colors.blue[100],
                child: const Text('First element'),
              ),
            ),
            Semantics(
              label: 'Second label',
              child: Container(
                padding: const EdgeInsets.all(8),
                color: Colors.green[100],
                child: const Text('Second element'),
              ),
            ),
            const SizedBox(height: 16),

            // 2. With merging (single combined element)
            const Text(
              'With Merging',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            MergeSemantics(
              child: Row(
                children: [
                  Semantics(
                    label: 'Combined label',
                    child: Container(
                      padding: const EdgeInsets.all(8),
                      color: Colors.orange[100],
                      child: const Text('Element 1'),
                    ),
                  ),
                  Semantics(
                    label: 'Combined label',
                    child: Container(
                      padding: const EdgeInsets.all(8),
                      color: Colors.purple[100],
                      child: const Text('Element 2'),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 16),

            // 3. Practical example - Custom switch
            const Text(
              'Practical Example: Custom Switch',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            _CustomSwitch(),
          ],
        ),
      ),
    );
  }
}

/// Custom switch with merged semantics
class _CustomSwitch extends StatefulWidget {
  const _CustomSwitch();

  @override
  State<_CustomSwitch> createState() => _CustomSwitchState();
}

class _CustomSwitchState extends State<_CustomSwitch> {
  bool _isOn = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        setState(() {
          _isOn = !_isOn;
        });
      },
      child: MergeSemantics(
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            // Label
            Semantics(
              label: 'Dark mode',
              child: const Text('Dark Mode'),
            ),
            const SizedBox(width: 8),
            // Switch
            Semantics(
              label: 'Dark mode toggle',
              value: _isOn ? 'On' : 'Off',
              child: Container(
                width: 50,
                height: 30,
                decoration: BoxDecoration(
                  color: _isOn ? Colors.blue : Colors.grey,
                  borderRadius: BorderRadius.circular(15),
                ),
                child: AnimatedAlign(
                  duration: const Duration(milliseconds: 200),
                  alignment: _isOn ? Alignment.centerRight : Alignment.centerLeft,
                  child: Container(
                    width: 26,
                    height: 26,
                    margin: const EdgeInsets.all(2),
                    decoration: const BoxDecoration(
                      color: Colors.white,
                      shape: BoxShape.circle,
                    ),
                  ),
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
- MergeSemantics combines child semantics
- Creates a single semantic node
- Useful for grouping related elements
- Improves accessibility for custom widgets

---

# Real-World Examples

> **Common patterns** with Semantics.

```dart
/// 1. Accessible custom button
class AccessibleButton extends StatelessWidget {
  const AccessibleButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.icon,
    this.hint,
  });

  final String label;
  final VoidCallback onPressed;
  final IconData? icon;
  final String? hint;

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: label,
      hint: hint ?? 'Tap to activate',
      button: true,
      onTap: onPressed,
      child: GestureDetector(
        onTap: onPressed,
        child: Container(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
          decoration: BoxDecoration(
            color: Colors.blue,
            borderRadius: BorderRadius.circular(8),
          ),
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              if (icon != null) ...[
                Icon(icon, color: Colors.white),
                const SizedBox(width: 8),
              ],
              Text(
                label,
                style: const TextStyle(color: Colors.white),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

/// 2. Accessible list tile
class AccessibleListTile extends StatelessWidget {
  const AccessibleListTile({
    super.key,
    required this.title,
    required this.subtitle,
    required this.icon,
    required this.onTap,
  });

  final String title;
  final String subtitle;
  final IconData icon;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: title,
      hint: 'Tap to view details',
      child: ListTile(
        leading: Icon(icon),
        title: Text(title),
        subtitle: Text(subtitle),
        trailing: const Icon(Icons.arrow_forward_ios, size: 16),
        onTap: onTap,
      ),
    );
  }
}

/// 3. Accessible form field
class AccessibleFormField extends StatelessWidget {
  const AccessibleFormField({
    super.key,
    required this.label,
    required this.controller,
    this.validator,
    this.keyboardType = TextInputType.text,
    this.obscureText = false,
  });

  final String label;
  final TextEditingController controller;
  final String? Function(String?)? validator;
  final TextInputType keyboardType;
  final bool obscureText;

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: label,
      textField: true,
      child: TextFormField(
        controller: controller,
        decoration: InputDecoration(
          labelText: label,
          border: const OutlineInputBorder(),
        ),
        validator: validator,
        keyboardType: keyboardType,
        obscureText: obscureText,
      ),
    );
  }
}

/// 4. Accessibility test helper
class AccessibilityHelper {
  /// Check if a widget has semantics
  static bool hasSemantics(Widget widget) {
    // In a real test, you would check for semantics nodes
    return true;
  }

  /// Verify accessibility label
  static void verifyLabel(
    WidgetTester tester,
    String label,
    Finder finder,
  ) {
    // This would be used in tests
    final semantics = tester.getSemantics(finder);
    expect(semantics.label, contains(label));
  }

  /// Verify accessibility hint
  static void verifyHint(
    WidgetTester tester,
    String hint,
    Finder finder,
  ) {
    // This would be used in tests
    final semantics = tester.getSemantics(finder);
    expect(semantics.hint, contains(hint));
  }
}
```

What's happening here?
- Accessible custom button
- Accessible list tile
- Accessible form field
- Accessibility test helpers

---

# Best Practices

## Provide Meaningful Labels

```dart
// Good - Descriptive label
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

## Use Appropriate Roles

```dart
// Good - Correct role
Semantics(
  button: true,
  child: GestureDetector(...),
)

// Bad - Missing role
Semantics(
  child: GestureDetector(...),
)
```

## Test Accessibility

```dart
// Good - Test semantics
testWidgets('should have correct semantics', (tester) async {
  await tester.pumpWidget(const MyWidget());
  final semantics = tester.getSemantics(find.byType(MyWidget));
  expect(semantics.label, 'My Widget');
});
```

---

# Common Mistakes

## Missing Semantics

Wrong:
```dart
// No semantics
GestureDetector(
  onTap: () {},
  child: Container(),
)
```

Correct:
```dart
// With semantics
Semantics(
  label: 'Tap button',
  button: true,
  child: GestureDetector(
    onTap: () {},
    child: Container(),
  ),
)
```

## Overly Complex Labels

Wrong:
```dart
// Too verbose
Semantics(
  label: 'This is a button that you can press to submit the form',
  child: ElevatedButton(...),
)
```

Correct:
```dart
// Concise and clear
Semantics(
  label: 'Submit form',
  child: ElevatedButton(...),
)
```

---

# Summary

Semantics makes Flutter apps accessible to users with disabilities. Use Semantics to add labels, hints, and roles, ExcludeSemantics to hide decorative elements, and MergeSemantics to group related elements. Accessibility is essential for creating inclusive applications.

---

# Next Steps

- [Screen Readers](screen-readers.md)
- [Keyboard Navigation](keyboard-navigation.md)
- [Accessibility Best Practices](accessibility-best-practices.md)

---

# Did You Know?

- Semantics enables screen reader support
- label describes the element
- hint describes interaction result
- button role indicates button behavior
- ExcludeSemantics hides elements
- MergeSemantics combines elements
- Accessibility is a legal requirement
- Semantics improves user experience for all