# Accessibility Best Practices

Understand how to make Flutter applications accessible to all users.

---

# What is it?

Accessibility Best Practices are guidelines and techniques for making Flutter applications usable by people with disabilities. This includes supporting screen readers, keyboard navigation, color contrast, touch target sizes, and other accessibility features. Following these practices ensures your app is inclusive and compliant with accessibility standards.

---

# Why does it exist?

Accessibility Best Practices exist to:

- Make apps usable for everyone
- Support users with disabilities
- Comply with legal requirements
- Improve user experience for all
- Reach wider audience
- Follow WCAG guidelines
- Create inclusive applications

---

# Color Contrast

> **Ensuring** readable text and UI elements.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Color contrast example
class ColorContrastExample extends StatelessWidget {
  const ColorContrastExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Color Contrast'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Good contrast - readable
            const Text(
              'Good Contrast (Readable)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.blue[900],
              child: const Text(
                'This text has good contrast',
                style: TextStyle(color: Colors.white),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Poor contrast - hard to read
            const Text(
              'Poor Contrast (Hard to Read)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.blue[200],
              child: const Text(
                'This text has poor contrast',
                style: TextStyle(color: Colors.blue[100]),
              ),
            ),
            const SizedBox(height: 16),

            // 3. Contrast with large text
            const Text(
              'Large Text Contrast',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.grey[200],
              child: const Text(
                'Large text can have lower contrast',
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            const SizedBox(height: 16),

            // 4. Accessible color palette
            _buildAccessiblePalette(),
          ],
        ),
      ),
    );
  }

  Widget _buildAccessiblePalette() {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Text(
          'Accessible Color Palette',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 8),
        Wrap(
          spacing: 8,
          runSpacing: 8,
          children: [
            _buildColorChip(Colors.blue, 'Blue'),
            _buildColorChip(Colors.green, 'Green'),
            _buildColorChip(Colors.red, 'Red'),
            _buildColorChip(Colors.orange, 'Orange'),
            _buildColorChip(Colors.purple, 'Purple'),
            _buildColorChip(Colors.teal, 'Teal'),
          ],
        ),
        const SizedBox(height: 8),
        const Text(
          'Tip: Use contrast ratio of at least 4.5:1 for normal text',
          style: TextStyle(fontSize: 12, color: Colors.grey),
        ),
      ],
    );
  }

  Widget _buildColorChip(Color color, String label) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
      decoration: BoxDecoration(
        color: color,
        borderRadius: BorderRadius.circular(4),
      ),
      child: Text(
        label,
        style: const TextStyle(color: Colors.white),
      ),
    );
  }
}
```

What's happening here?
- Good contrast is essential for readability
- Poor contrast makes text hard to read
- Large text can have lower contrast
- Accessible color palettes help

---

# Touch Targets

> **Ensuring** touchable areas are large enough.

```dart
/// Touch targets example
class TouchTargetsExample extends StatelessWidget {
  const TouchTargetsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Touch Targets'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Too small - hard to tap
            const Text(
              'Too Small (Hard to Tap)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Row(
              children: [
                IconButton(
                  icon: const Icon(Icons.star, size: 20),
                  onPressed: () {},
                ),
                IconButton(
                  icon: const Icon(Icons.favorite, size: 20),
                  onPressed: () {},
                ),
                IconButton(
                  icon: const Icon(Icons.share, size: 20),
                  onPressed: () {},
                ),
              ],
            ),
            const SizedBox(height: 16),

            // 2. Good size - easy to tap
            const Text(
              'Good Size (Easy to Tap)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Row(
              children: [
                _buildLargeIconButton(Icons.star),
                _buildLargeIconButton(Icons.favorite),
                _buildLargeIconButton(Icons.share),
              ],
            ),
            const SizedBox(height: 16),

            // 3. Minimum touch target
            const Text(
              'Minimum Touch Target (44px)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.grey[200],
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  _buildMinTouchTarget('A'),
                  _buildMinTouchTarget('B'),
                  _buildMinTouchTarget('C'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildLargeIconButton(IconData icon) {
    return Container(
      width: 48,
      height: 48,
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(8),
      ),
      child: IconButton(
        icon: Icon(icon, color: Colors.white),
        onPressed: () {},
      ),
    );
  }

  Widget _buildMinTouchTarget(String label) {
    return Container(
      width: 44,
      height: 44,
      decoration: BoxDecoration(
        color: Colors.blue[100],
        borderRadius: BorderRadius.circular(4),
      ),
      child: Center(
        child: Text(
          label,
          style: const TextStyle(fontWeight: FontWeight.bold),
        ),
      ),
    );
  }
}
```

What's happening here?
- Minimum touch target: 44px
- Larger targets are easier to tap
- Consider motor impairments
- Provide adequate spacing

---

# Labeling

> **Providing** clear labels for screen readers.

```dart
/// Labeling example
class LabelingExample extends StatelessWidget {
  const LabelingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Labeling'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Unlabeled icon (bad)
            const Text(
              'Unlabeled (Bad)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            IconButton(
              icon: const Icon(Icons.settings),
              onPressed: () {},
            ),
            const SizedBox(height: 16),

            // 2. Labeled icon (good)
            const Text(
              'Labeled (Good)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              label: 'Settings',
              hint: 'Opens settings screen',
              child: IconButton(
                icon: const Icon(Icons.settings),
                onPressed: () {},
              ),
            ),
            const SizedBox(height: 16),

            // 3. Labeled button
            const Text(
              'Labeled Button',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              label: 'Submit form',
              child: ElevatedButton(
                onPressed: () {},
                child: const Text('Submit'),
              ),
            ),
            const SizedBox(height: 16),

            // 4. Labeled image
            const Text(
              'Labeled Image',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              label: 'App logo - Flutter dash',
              image: true,
              child: const Icon(
                Icons.flutter_dash,
                size: 80,
                color: Colors.blue,
              ),
            ),
            const SizedBox(height: 16),

            // 5. Form field labeling
            const Text(
              'Form Field Labeling',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Semantics(
              label: 'Email address input',
              textField: true,
              child: TextField(
                decoration: const InputDecoration(
                  labelText: 'Email',
                  hintText: 'Enter your email',
                  border: OutlineInputBorder(),
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
- All interactive elements need labels
- Semantics adds accessibility labels
- Label describes the element
- Hint explains interaction

---

# Semantic Structure

> **Creating** accessible semantic structure.

```dart
/// Semantic structure example
class SemanticStructureExample extends StatelessWidget {
  const SemanticStructureExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Semantic Structure'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Proper heading hierarchy
            Semantics(
              header: true,
              label: 'Main heading',
              child: const Text(
                'Main Heading',
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            const SizedBox(height: 8),

            Semantics(
              header: true,
              label: 'Sub heading',
              child: const Text(
                'Sub Heading',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            const SizedBox(height: 8),

            const Text(
              'Content text goes here. This is the main content.',
              style: TextStyle(fontSize: 16),
            ),
            const SizedBox(height: 16),

            // 2. List structure
            Semantics(
              label: 'Feature list',
              child: Column(
                children: [
                  _buildListItem('Feature 1', 'Description of feature 1'),
                  _buildListItem('Feature 2', 'Description of feature 2'),
                  _buildListItem('Feature 3', 'Description of feature 3'),
                ],
              ),
            ),
            const SizedBox(height: 16),

            // 3. Group related items
            MergeSemantics(
              child: Row(
                children: [
                  Semantics(
                    label: 'Rating: 4 stars',
                    child: const Icon(Icons.star, color: Colors.yellow),
                  ),
                  Semantics(
                    label: 'Rating: 4 stars',
                    child: const Icon(Icons.star, color: Colors.yellow),
                  ),
                  Semantics(
                    label: 'Rating: 4 stars',
                    child: const Icon(Icons.star, color: Colors.yellow),
                  ),
                  Semantics(
                    label: 'Rating: 4 stars',
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

  Widget _buildListItem(String title, String description) {
    return Semantics(
      label: '$title: $description',
      child: Container(
        padding: const EdgeInsets.all(8),
        margin: const EdgeInsets.only(bottom: 4),
        decoration: BoxDecoration(
          color: Colors.grey[100],
          borderRadius: BorderRadius.circular(4),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              title,
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
            Text(
              description,
              style: TextStyle(color: Colors.grey[600]),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Headers provide structure
- Lists organize content
- MergeSemantics groups related items
- Semantic labels describe content

---

# Testing Accessibility

> **Testing** accessibility features.

```dart
/// Accessibility testing example
class AccessibilityTestingExample extends StatelessWidget {
  const AccessibilityTestingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Accessibility Testing'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Widgets to test
            Semantics(
              label: 'Test button for accessibility',
              button: true,
              child: ElevatedButton(
                onPressed: () {},
                child: const Text('Test Button'),
              ),
            ),
            const SizedBox(height: 16),

            // 2. Accessibility test info
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Accessibility Testing Tips:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('• Use screen readers (TalkBack, VoiceOver)'),
                  Text('• Check keyboard navigation'),
                  Text('• Verify color contrast'),
                  Text('• Test with different font sizes'),
                  Text('• Use accessibility scanner tools'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// Accessibility test helper
class AccessibilityTestHelper {
  /// Test semantic label
  static void testLabel(
    WidgetTester tester,
    Finder finder,
    String expectedLabel,
  ) {
    final semantics = tester.getSemantics(finder);
    expect(semantics.label, contains(expectedLabel));
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
  static void testButtonRole(
    WidgetTester tester,
    Finder finder,
    bool expected,
  ) {
    final semantics = tester.getSemantics(finder);
    expect(semantics.hasAction(SemanticsAction.tap), expected);
  }

  /// Test if widget has correct role
  static void testRole(
    WidgetTester tester,
    Finder finder,
    SemanticsRole role,
  ) {
    final semantics = tester.getSemantics(finder);
    expect(semantics.role, role);
  }
}
```

What's happening here?
- Testing semantic labels
- Testing focusability
- Testing roles
- Accessibility testing tools

---

# Best Practices

## Provide Clear Labels

```dart
// Good - Clear, descriptive label
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

## Ensure Color Contrast

```dart
// Good - High contrast
Text(
  'Readable text',
  style: TextStyle(
    color: Colors.white,
    backgroundColor: Colors.blue[900],
  ),
)

// Bad - Low contrast
Text(
  'Hard to read text',
  style: TextStyle(
    color: Colors.blue[100],
    backgroundColor: Colors.blue[200],
  ),
)
```

## Provide Keyboard Navigation

```dart
// Good - Keyboard support
Focus(
  child: GestureDetector(
    onTap: () {},
    child: Container(),
  ),
)

// Bad - No keyboard support
GestureDetector(
  onTap: () {},
  child: Container(),
)
```

---

# Common Mistakes

## Missing Labels

Wrong:
```dart
// No label for screen readers
IconButton(
  icon: Icon(Icons.settings),
  onPressed: () {},
)
```

Correct:
```dart
// With label
Semantics(
  label: 'Settings',
  child: IconButton(
    icon: Icon(Icons.settings),
    onPressed: () {},
  ),
)
```

## Poor Contrast

Wrong:
```dart
// Low contrast text
Text(
  'Hard to read',
  style: TextStyle(
    color: Colors.grey[300],
    backgroundColor: Colors.white,
  ),
)
```

Correct:
```dart
// High contrast text
Text(
  'Easy to read',
  style: TextStyle(
    color: Colors.black,
    backgroundColor: Colors.white,
  ),
)
```

---

# Summary

Accessibility Best Practices ensure your Flutter app is usable by everyone. Provide clear labels, ensure color contrast, support keyboard navigation, use proper semantic structure, and test with accessibility tools. Accessibility is essential for creating inclusive applications.

---

# Next Steps

- [Semantics](semantics.md)
- [Screen Readers](screen-readers.md)
- [Keyboard Navigation](keyboard-navigation.md)

---

# Did You Know?

- Accessibility is a legal requirement
- WCAG guidelines define accessibility
- Screen readers need proper labels
- Color contrast improves readability
- Keyboard navigation supports motor impairments
- Touch targets should be at least 44px
- Accessibility benefits all users
- Inclusive design is good design