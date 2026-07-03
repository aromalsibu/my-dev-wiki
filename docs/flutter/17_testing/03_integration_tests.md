# Integration Tests

Understand how to write and run integration tests in Flutter to verify your app works correctly as a whole.

---

# What is it?

Integration Tests (also called End-to-End Tests) are tests that verify your entire application works correctly by simulating real user interactions. They run on a real device or emulator and test the app as a whole, including UI, business logic, and platform integration. Integration tests are the most comprehensive type of test but are also the slowest to run.

---

# Why does it exist?

Integration Tests exist to:

- Test the app as a whole
- Verify user flows work correctly
- Test platform integration
- Catch bugs that unit tests miss
- Ensure app works on real devices
- Validate UI and interactions
- Test app performance

---

# Setting Up Integration Tests

> **Configuring** integration tests in your project.

```yaml
# 1. Add integration_test dependency in pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter

# 2. Create test structure
# integration_test/
#   app_test.dart
#   login_test.dart
#   navigation_test.dart
```

```dart
// 3. integration_test/app_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  // 4. Initialize integration test
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('App Integration Tests', () {
    // 5. Test basic app launch
    testWidgets('should launch the app', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Assert: Verify app launched
      expect(find.text('My App'), findsOneWidget);
    });

    // 6. Test navigation flow
    testWidgets('should navigate between screens', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Navigate to detail screen
      final detailButton = find.byKey(const Key('go_to_detail'));
      await tester.tap(detailButton);
      await tester.pumpAndSettle();

      // Assert: Verify detail screen appears
      expect(find.text('Detail Screen'), findsOneWidget);
    });
  });
}
```

What's happening here?
- integration_test package for integration tests
- IntegrationTestWidgetsFlutterBinding initializes
- pumpAndSettle waits for animations
- Testing full app flows

---

# Testing User Flows

> **Testing complete** user journeys.

```dart
/// 1. Login flow test
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Login Flow Tests', () {
    // 1. Test successful login
    testWidgets('should login successfully with valid credentials', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Enter email
      final emailField = find.byKey(const Key('email_field'));
      await tester.enterText(emailField, 'test@example.com');

      // Act: Enter password
      final passwordField = find.byKey(const Key('password_field'));
      await tester.enterText(passwordField, 'password123');

      // Act: Tap login button
      final loginButton = find.byKey(const Key('login_button'));
      await tester.tap(loginButton);
      await tester.pumpAndSettle();

      // Assert: Verify logged in
      expect(find.text('Welcome, User!'), findsOneWidget);
      expect(find.text('Dashboard'), findsOneWidget);
    });

    // 2. Test failed login
    testWidgets('should show error with invalid credentials', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Enter invalid credentials
      await tester.enterText(find.byKey(const Key('email_field')), 'wrong@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), 'wrongpass');
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle();

      // Assert: Verify error message
      expect(find.text('Invalid credentials'), findsOneWidget);
      expect(find.text('Dashboard'), findsNothing);
    });

    // 3. Test logout
    testWidgets('should logout successfully', (WidgetTester tester) async {
      // Act: Launch and login
      app.main();
      await tester.pumpAndSettle();

      await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), 'password123');
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle();

      // Act: Logout
      final logoutButton = find.byKey(const Key('logout_button'));
      await tester.tap(logoutButton);
      await tester.pumpAndSettle();

      // Assert: Verify logged out
      expect(find.text('Login'), findsOneWidget);
      expect(find.text('Dashboard'), findsNothing);
    });
  });
}
```

What's happening here?
- Testing complete user flows
- Successful and failed login
- Logout flow testing
- Full app interaction

---

# Testing Navigation

> **Testing** screen navigation.

```dart
/// Navigation tests
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Navigation Tests', () {
    // 1. Test navigation with parameters
    testWidgets('should navigate with parameters', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Navigate to product detail
      final productCard = find.byKey(const Key('product_card_1'));
      await tester.tap(productCard);
      await tester.pumpAndSettle();

      // Assert: Verify product details
      expect(find.text('Product 1'), findsOneWidget);
      expect(find.text('\$99.99'), findsOneWidget);
      expect(find.text('Description'), findsOneWidget);
    });

    // 2. Test back navigation
    testWidgets('should navigate back correctly', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Navigate to detail
      await tester.tap(find.byKey(const Key('product_card_1')));
      await tester.pumpAndSettle();

      // Assert: Detail screen is shown
      expect(find.text('Product 1'), findsOneWidget);

      // Act: Press back
      final backButton = find.byKey(const Key('back_button'));
      await tester.tap(backButton);
      await tester.pumpAndSettle();

      // Assert: Back to home
      expect(find.text('Product 1'), findsNothing);
      expect(find.text('Home'), findsOneWidget);
    });

    // 3. Test deep navigation
    testWidgets('should navigate through multiple screens', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Navigate to profile
      await tester.tap(find.byKey(const Key('profile_tab')));
      await tester.pumpAndSettle();

      // Act: Navigate to settings
      await tester.tap(find.byKey(const Key('settings_button')));
      await tester.pumpAndSettle();

      // Act: Navigate to security
      await tester.tap(find.byKey(const Key('security_button')));
      await tester.pumpAndSettle();

      // Assert: Security screen is shown
      expect(find.text('Security Settings'), findsOneWidget);
    });
  });
}
```

What's happening here?
- Navigation testing
- Parameter passing
- Back navigation
- Deep navigation flows

---

# Testing Data Persistence

> **Testing** data storage and retrieval.

```dart
/// Data persistence tests
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Data Persistence Tests', () {
    // 1. Test saving data
    testWidgets('should save data locally', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Enter data
      await tester.enterText(
        find.byKey(const Key('data_input')),
        'Test data',
      );
      await tester.tap(find.byKey(const Key('save_button')));
      await tester.pumpAndSettle();

      // Act: Restart app
      await tester.restartAndRestore();

      // Assert: Data is still there
      expect(find.text('Test data'), findsOneWidget);
    });

    // 2. Test deleting data
    testWidgets('should delete data', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Add data
      await tester.enterText(
        find.byKey(const Key('data_input')),
        'Data to delete',
      );
      await tester.tap(find.byKey(const Key('save_button')));
      await tester.pumpAndSettle();

      // Act: Delete data
      final deleteButton = find.byKey(const Key('delete_button_0'));
      await tester.tap(deleteButton);
      await tester.pumpAndSettle();

      // Assert: Data is gone
      expect(find.text('Data to delete'), findsNothing);
    });

    // 3. Test multiple data items
    testWidgets('should handle multiple data items', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Add multiple items
      final dataInput = find.byKey(const Key('data_input'));
      final saveButton = find.byKey(const Key('save_button'));

      for (int i = 1; i <= 3; i++) {
        await tester.enterText(dataInput, 'Item $i');
        await tester.tap(saveButton);
        await tester.pumpAndSettle();
      }

      // Assert: All items are present
      expect(find.text('Item 1'), findsOneWidget);
      expect(find.text('Item 2'), findsOneWidget);
      expect(find.text('Item 3'), findsOneWidget);
    });
  });
}
```

What's happening here?
- Data persistence testing
- Save and delete operations
- App restart testing
- Multiple data items

---

# Performance Testing

> **Testing** app performance.

```dart
/// Performance tests
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Performance Tests', () {
    // 1. Test scrolling performance
    testWidgets('should scroll smoothly', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Scroll through list
      final listView = find.byKey(const Key('long_list'));
      await tester.fling(listView, const Offset(0, -500), 1000);
      await tester.pumpAndSettle();

      // Assert: Still responsive
      expect(find.text('Item 50'), findsOneWidget);
    });

    // 2. Test animation performance
    testWidgets('should animate smoothly', (WidgetTester tester) async {
      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Act: Trigger animation
      final animateButton = find.byKey(const Key('animate_button'));
      await tester.tap(animateButton);
      await tester.pumpAndSettle();

      // Assert: Animation completed
      expect(find.text('Animation Complete'), findsOneWidget);
    });

    // 3. Test app startup time
    testWidgets('should start quickly', (WidgetTester tester) async {
      final stopwatch = Stopwatch()..start();

      // Act: Launch the app
      app.main();
      await tester.pumpAndSettle();

      // Assert: Startup time is acceptable
      stopwatch.stop();
      expect(stopwatch.elapsedMilliseconds, lessThan(3000));
    });
  });
}
```

What's happening here?
- Scrolling performance
- Animation testing
- Startup time measurement
- Performance validation

---

# Real-World Examples

> **Common patterns** in integration testing.

```dart
/// 1. Shopping cart flow
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Shopping Cart Tests', () {
    testWidgets('should complete checkout flow', (WidgetTester tester) async {
      // 1. Launch app
      app.main();
      await tester.pumpAndSettle();

      // 2. Add item to cart
      await tester.tap(find.byKey(const Key('add_to_cart_1')));
      await tester.pumpAndSettle();

      // 3. Go to cart
      await tester.tap(find.byKey(const Key('cart_icon')));
      await tester.pumpAndSettle();

      // 4. Verify item in cart
      expect(find.text('Item 1'), findsOneWidget);

      // 5. Proceed to checkout
      await tester.tap(find.byKey(const Key('checkout_button')));
      await tester.pumpAndSettle();

      // 6. Enter shipping info
      await tester.enterText(
        find.byKey(const Key('address_input')),
        '123 Main St',
      );
      await tester.tap(find.byKey(const Key('continue_button')));
      await tester.pumpAndSettle();

      // 7. Enter payment info
      await tester.enterText(
        find.byKey(const Key('card_input')),
        '4111 1111 1111 1111',
      );
      await tester.tap(find.byKey(const Key('place_order_button')));
      await tester.pumpAndSettle();

      // 8. Verify order confirmation
      expect(find.text('Order Confirmed'), findsOneWidget);
    });
  });
}

/// 2. Multi-platform testing
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Multi-Platform Tests', () {
    testWidgets('should work on Android', (WidgetTester tester) async {
      // Platform-specific tests
      // Test Android-specific features
    });

    testWidgets('should work on iOS', (WidgetTester tester) async {
      // Platform-specific tests
      // Test iOS-specific features
    });

    testWidgets('should handle different screen sizes', (WidgetTester tester) async {
      // Test with different screen sizes
      tester.view.physicalSize = const Size(800, 600);
      await tester.pumpAndSettle();

      // Verify layout adapts
      expect(find.byType(AdaptiveLayout), findsOneWidget);
    });
  });
}
```

What's happening here?
- Complete checkout flow
- Multi-platform testing
- Different screen sizes
- Real-world scenarios

---

# Best Practices

## Test Critical User Flows

```dart
// Good - Test critical flows
testWidgets('should complete purchase flow', () {
  // Login -> Add to cart -> Checkout -> Confirm
});

// Bad - Testing trivial flows
testWidgets('should display logo', () {
  // Only tests logo presence
});
```

## Use Real Devices

```dart
// Good - Test on real devices
flutter test integration_test --device-id=emulator-5554

// Bad - Only testing on emulator
```

## Handle Async Operations

```dart
// Good - Wait for async operations
await tester.pumpAndSettle();

// Bad - Not waiting
await tester.pump();
// May miss async updates
```

---

# Common Mistakes

## Not Waiting for Animations

Wrong:
```dart
// Missing wait
await tester.tap(button);
expect(find.text('Loaded'), findsOneWidget);
```

Correct:
```dart
// Wait for animations
await tester.tap(button);
await tester.pumpAndSettle();
expect(find.text('Loaded'), findsOneWidget);
```

## Testing Implementation Details

Wrong:
```dart
// Tests internal state
expect(widget.state.isLoading, true);
```

Correct:
```dart
// Tests visible behavior
expect(find.text('Loading...'), findsOneWidget);
```

---

# Summary

Integration Tests verify your entire application works correctly by simulating real user interactions. Use integration_test package for integration tests, test complete user flows, and run tests on real devices. Integration tests catch bugs that unit tests miss and ensure your app works as expected.

---

# Next Steps

- [Golden Tests](golden-tests.md)
- [Test Utilities](test-utilities.md)
- [Performance Testing](performance-testing.md)

---

# Did You Know?

- Integration tests run on real devices
- pumpAndSettle waits for animations
- Integration tests are comprehensive
- Tests can be run on multiple devices
- Real user flows are tested
- Integration tests catch UI bugs
- Performance can be measured
- Integration tests are essential for quality