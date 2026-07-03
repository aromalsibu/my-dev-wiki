# Golden Tests

Understand how to use golden tests to detect visual regressions in Flutter applications.

---

# What is it?

Golden Tests are a type of widget test that compare the visual appearance of a widget against a previously captured "golden" image. If the widget's appearance changes, the test fails, alerting you to visual regressions. Golden tests are essential for maintaining UI consistency across your application.

---

# Why does it exist?

Golden Tests exist to:

- Detect visual regressions
- Ensure UI consistency
- Verify widget appearance
- Catch unintended UI changes
- Support design system compliance
- Test responsive layouts
- Validate styling and theming

---

# Setting Up Golden Tests

> **Configuring** golden tests in your project.

```yaml
# 1. Add dependencies in pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_goldens:
    sdk: flutter
```

```dart
// 2. Create golden test structure
// test/
//   golden/
//     widgets/
//       button_golden_test.dart
//       card_golden_test.dart
//     screens/
//       login_screen_golden_test.dart
//   goldens/
//     widgets/
//       button.png
//       button_dark.png
//     screens/
//       login_screen.png

/// 3. Basic golden test setup
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  // Enable golden tests
  testWidgets('Golden test example', (WidgetTester tester) async {
    // Arrange: Build the widget
    await tester.pumpWidget(
      const MaterialApp(
        home: Scaffold(
          body: Center(
            child: MyButton(),
          ),
        ),
      ),
    );

    // Act: Wait for rendering
    await tester.pumpAndSettle();

    // Assert: Compare with golden image
    await expectLater(
      find.byType(MyButton),
      matchesGoldenFile('goldens/my_button.png'),
    );
  });
}
```

What's happening here?
- flutter_goldens for golden testing
- matchesGoldenFile compares images
- Golden images stored in goldens folder
- Visual regression detection

---

# Basic Golden Tests

> **Writing** your first golden tests.

```dart
/// 1. Widgets for golden testing
class CustomButton extends StatelessWidget {
  const CustomButton({
    super.key,
    required this.label,
    this.isPrimary = true,
  });

  final String label;
  final bool isPrimary;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      decoration: BoxDecoration(
        color: isPrimary ? Colors.blue : Colors.grey[200],
        borderRadius: BorderRadius.circular(8),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 4,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Text(
        label,
        style: TextStyle(
          color: isPrimary ? Colors.white : Colors.black,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}

/// 2. Golden tests
void main() {
  // Group golden tests
  group('Golden Tests', () {
    // 1. Test primary button
    testWidgets('should match primary button golden', (WidgetTester tester) async {
      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: Center(
              child: CustomButton(
                label: 'Primary Button',
                isPrimary: true,
              ),
            ),
          ),
        ),
      );

      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        find.byType(CustomButton),
        matchesGoldenFile('goldens/primary_button.png'),
      );
    });

    // 2. Test secondary button
    testWidgets('should match secondary button golden', (WidgetTester tester) async {
      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: Center(
              child: CustomButton(
                label: 'Secondary Button',
                isPrimary: false,
              ),
            ),
          ),
        ),
      );

      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        find.byType(CustomButton),
        matchesGoldenFile('goldens/secondary_button.png'),
      );
    });

    // 3. Test different sizes
    testWidgets('should match different sizes', (WidgetTester tester) async {
      // Test small button
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: Center(
              child: SizedBox(
                width: 80,
                child: CustomButton(
                  label: 'Small',
                  isPrimary: true,
                ),
              ),
            ),
          ),
        ),
      );

      await tester.pumpAndSettle();

      await expectLater(
        find.byType(CustomButton),
        matchesGoldenFile('goldens/small_button.png'),
      );
    });

    // 4. Test with different themes
    testWidgets('should match dark theme golden', (WidgetTester tester) async {
      // Arrange: Build with dark theme
      await tester.pumpWidget(
        MaterialApp(
          theme: ThemeData.dark(),
          home: const Scaffold(
            body: Center(
              child: CustomButton(
                label: 'Dark Button',
                isPrimary: true,
              ),
            ),
          ),
        ),
      );

      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        find.byType(CustomButton),
        matchesGoldenFile('goldens/dark_button.png'),
      );
    });
  });
}
```

What's happening here?
- matchesGoldenFile compares images
- Multiple button states tested
- Different themes tested
- Visual regression detection

---

# Golden Tests with Different States

> **Testing** widget states.

```dart
/// 1. Interactive widget for golden testing
class InteractiveWidget extends StatefulWidget {
  const InteractiveWidget({super.key});

  @override
  State<InteractiveWidget> createState() => _InteractiveWidgetState();
}

class _InteractiveWidgetState extends State<InteractiveWidget> {
  bool _isActive = false;
  bool _isHovered = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        setState(() {
          _isActive = !_isActive;
        });
      },
      child: Container(
        width: 100,
        height: 100,
        decoration: BoxDecoration(
          color: _isActive ? Colors.green : Colors.blue,
          borderRadius: BorderRadius.circular(8),
          boxShadow: _isHovered
              ? [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.2),
                    blurRadius: 8,
                  ),
                ]
              : null,
        ),
        child: Center(
          child: Text(
            _isActive ? 'Active' : 'Inactive',
            style: const TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }
}

/// 2. Golden tests for different states
void main() {
  group('Widget States Golden Tests', () {
    // 1. Test inactive state
    testWidgets('should match inactive state', (WidgetTester tester) async {
      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: Center(
              child: InteractiveWidget(),
            ),
          ),
        ),
      );

      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        find.byType(InteractiveWidget),
        matchesGoldenFile('goldens/inactive_state.png'),
      );
    });

    // 2. Test active state
    testWidgets('should match active state', (WidgetTester tester) async {
      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: Center(
              child: InteractiveWidget(),
            ),
          ),
        ),
      );

      // Act: Tap to activate
      await tester.tap(find.byType(InteractiveWidget));
      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        find.byType(InteractiveWidget),
        matchesGoldenFile('goldens/active_state.png'),
      );
    });

    // 3. Test hover state (desktop only)
    testWidgets('should match hover state', (WidgetTester tester) async {
      // This test is only relevant for desktop/web
      // Skip on mobile platforms
      if (Platform.isAndroid || Platform.isIOS) {
        return;
      }

      // Arrange: Build the widget
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: Center(
              child: InteractiveWidget(),
            ),
          ),
        ),
      );

      // Act: Simulate hover
      final widget = find.byType(InteractiveWidget);
      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        widget,
        matchesGoldenFile('goldens/hover_state.png'),
      );
    });
  });
}
```

What's happening here?
- Testing different widget states
- Interactive state changes
- Hover state testing
- Platform-specific tests

---

# Golden Tests with Different Screen Sizes

> **Testing** responsive widgets.

```dart
/// 1. Responsive widget
class ResponsiveCard extends StatelessWidget {
  const ResponsiveCard({super.key});

  @override
  Widget build(BuildContext context) {
    final screenWidth = MediaQuery.of(context).size.width;
    final isDesktop = screenWidth > 800;
    final isTablet = screenWidth > 600 && screenWidth <= 800;

    return Container(
      padding: const EdgeInsets.all(16),
      child: Card(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Header
              Container(
                width: double.infinity,
                height: 150,
                color: Colors.blue[100],
                child: const Center(
                  child: Icon(Icons.image, size: 50, color: Colors.white),
                ),
              ),
              const SizedBox(height: 8),
              const Text(
                'Card Title',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              const Text(
                'Card description goes here. This is a sample description.',
                style: TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 8),
              // Layout changes based on screen size
              isDesktop
                  ? Row(
                      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                      children: const [
                        Expanded(
                          child: Text('Desktop Layout'),
                        ),
                        Text('More space available'),
                      ],
                    )
                  : isTablet
                      ? Row(
                          children: const [
                            Expanded(
                              child: Text('Tablet Layout'),
                            ),
                            Icon(Icons.arrow_forward),
                          ],
                        )
                      : const Row(
                          children: [
                            Expanded(
                              child: Text('Mobile Layout'),
                            ),
                            Icon(Icons.arrow_forward),
                          ],
                        ),
            ],
          ),
        ),
      ),
    );
  }
}

/// 2. Golden tests for responsive widget
void main() {
  group('Responsive Golden Tests', () {
    // 1. Test mobile size
    testWidgets('should match mobile layout', (WidgetTester tester) async {
      // Arrange: Set mobile screen size
      tester.view.physicalSize = const Size(400, 800);
      tester.view.devicePixelRatio = 1.0;

      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: ResponsiveCard(),
          ),
        ),
      );

      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        find.byType(ResponsiveCard),
        matchesGoldenFile('goldens/mobile_card.png'),
      );
    });

    // 2. Test tablet size
    testWidgets('should match tablet layout', (WidgetTester tester) async {
      // Arrange: Set tablet screen size
      tester.view.physicalSize = const Size(700, 1000);
      tester.view.devicePixelRatio = 1.0;

      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: ResponsiveCard(),
          ),
        ),
      );

      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        find.byType(ResponsiveCard),
        matchesGoldenFile('goldens/tablet_card.png'),
      );
    });

    // 3. Test desktop size
    testWidgets('should match desktop layout', (WidgetTester tester) async {
      // Arrange: Set desktop screen size
      tester.view.physicalSize = const Size(1200, 800);
      tester.view.devicePixelRatio = 1.0;

      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: ResponsiveCard(),
          ),
        ),
      );

      await tester.pumpAndSettle();

      // Assert: Compare with golden
      await expectLater(
        find.byType(ResponsiveCard),
        matchesGoldenFile('goldens/desktop_card.png'),
      );
    });
  });
}
```

What's happening here?
- Testing different screen sizes
- Responsive layout validation
- Mobile, tablet, desktop layouts
- Device pixel ratio handling

---

# Updating Golden Images

> **Managing** golden image updates.

```bash
# 1. Run golden tests
flutter test --update-goldens

# 2. Update specific golden files
flutter test test/golden/ --update-goldens

# 3. Update all golden files
flutter test --update-goldens --no-test

# 4. Review golden changes
# Golden files are stored in test/goldens/
# Review changes before committing

# 5. CI/CD integration
# Run golden tests in CI
# Fail if golden files don't match
```

```dart
/// 3. Golden test with skip option
void main() {
  group('Golden Tests with Options', () {
    // 1. Test with skip
    testWidgets('should match golden (skip if needed)', (WidgetTester tester) async {
      // Skip if golden file doesn't exist yet
      final hasGolden = await File('goldens/my_widget.png').exists();

      if (!hasGolden) {
        // Create initial golden
        await _createGolden(tester);
        return;
      }

      // Run normal golden test
      await tester.pumpWidget(const MyWidget());
      await tester.pumpAndSettle();

      await expectLater(
        find.byType(MyWidget),
        matchesGoldenFile('goldens/my_widget.png'),
      );
    });

    // 2. Test with custom threshold
    testWidgets('should match golden with custom threshold', (WidgetTester tester) async {
      await tester.pumpWidget(const MyWidget());
      await tester.pumpAndSettle();

      await expectLater(
        find.byType(MyWidget),
        matchesGoldenFile('goldens/my_widget.png', threshold: 0.01),
      );
    });
  });
}
```

What's happening here?
- Updating golden images
- Custom similarity threshold
- CI/CD integration
- Conditional golden testing

---

# Best Practices

## Keep Golden Files Organized

```dart
// Good - Organized golden files
goldens/
  widgets/
    button.png
    card.png
  screens/
    login.png
    home.png
  themes/
    light.png
    dark.png
```

## Use Consistent Naming

```dart
// Good - Consistent naming
matchesGoldenFile('goldens/widgets/button_primary.png')
matchesGoldenFile('goldens/widgets/button_secondary.png')

// Bad - Inconsistent naming
matchesGoldenFile('goldens/button1.png')
matchesGoldenFile('goldens/button2.png')
```

## Test Different States

```dart
// Good - Test all states
testWidgets('should match enabled state', () { ... });
testWidgets('should match disabled state', () { ... });
testWidgets('should match hovered state', () { ... });
```

---

# Common Mistakes

## Not Updating Golden Files

Wrong:
```dart
// Golden files not updated after UI changes
// Tests will fail unnecessarily
```

Correct:
```dart
// Update golden files after intentional UI changes
flutter test --update-goldens
```

## Platform-Specific Differences

Wrong:
```dart
// Golden files differ between platforms
// Tests fail on different platforms
```

Correct:
```dart
// Create platform-specific golden files
matchesGoldenFile('goldens/widgets/button_android.png')
matchesGoldenFile('goldens/widgets/button_ios.png')
```

---

# Summary

Golden Tests detect visual regressions by comparing widgets against golden images. Use matchesGoldenFile for comparison, organize golden files properly, and update them when UI changes intentionally. Golden tests are essential for maintaining UI consistency.

---

# Next Steps

- [Test Utilities](test-utilities.md)
- [Performance Testing](performance-testing.md)
- [CI/CD Integration](cicd-integration.md)

---

# Did You Know?

- Golden tests detect visual regressions
- matchesGoldenFile compares images
- Golden files are stored in test/goldens/
- Update golden files with --update-goldens
- Platform-specific golden files may be needed
- Custom threshold controls sensitivity
- Golden tests are visual snapshots
- Golden tests are part of CI/CD