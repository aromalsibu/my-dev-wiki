# Test Utilities

Understand how to use test utilities to write more efficient and maintainable tests in Flutter.

---

# What is it?

Test Utilities are helper functions and classes that simplify writing tests by providing common functionality like creating test widgets, mocking data, and simulating user interactions. They reduce boilerplate code, make tests more readable, and improve test maintainability.

---

# Why does it exist?

Test Utilities exist to:

- Reduce test boilerplate
- Make tests more readable
- Provide common test functionality
- Simplify test setup and teardown
- Support test data creation
- Enable test customization
- Improve test maintenance

---

# Basic Test Utilities

> **Creating** test utilities.

```dart
// 1. test_utils/widget_test_utils.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

/// 1. Test widget wrapper
/// Wraps a widget with MaterialApp for testing
Widget wrapWithMaterialApp(Widget widget, {ThemeData? theme}) {
  return MaterialApp(
    theme: theme ?? ThemeData.light(),
    home: Scaffold(
      body: widget,
    ),
  );
}

/// 2. Test widget wrapper with custom theme
Widget wrapWithTheme(Widget widget, ThemeData theme) {
  return MaterialApp(
    theme: theme,
    home: Scaffold(
      body: widget,
    ),
  );
}

/// 3. Test widget wrapper with state management
Widget wrapWithProvider(Widget widget) {
  // In a real app, this would include your providers
  return MaterialApp(
    home: Scaffold(
      body: widget,
    ),
  );
}

/// 4. Common test data
class TestData {
  static const user = {
    'id': '1',
    'name': 'John Doe',
    'email': 'john@example.com',
    'age': 25,
  };

  static const users = [
    {'id': '1', 'name': 'John Doe', 'email': 'john@example.com', 'age': 25},
    {'id': '2', 'name': 'Jane Smith', 'email': 'jane@example.com', 'age': 30},
  ];
}
```

```dart
/// 5. Using test utilities
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/test_utils/widget_test_utils.dart';
import 'package:my_app/widgets/my_widget.dart';

void main() {
  group('MyWidget Tests', () {
    testWidgets('should display user data', (WidgetTester tester) async {
      // Arrange: Use utility to wrap widget
      await tester.pumpWidget(
        wrapWithMaterialApp(
          MyWidget(user: TestData.user),
        ),
      );

      await tester.pumpAndSettle();

      // Assert: Verify content
      expect(find.text('John Doe'), findsOneWidget);
      expect(find.text('john@example.com'), findsOneWidget);
    });
  });
}
```

What's happening here?
- Test utilities reduce boilerplate
- Consistent test setup
- Reusable test data
- Cleaner test code

---

# Finder Utilities

> **Custom finders** for testing.

```dart
/// test_utils/finder_utils.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

/// 1. Custom finder for widgets with specific text
Finder findWidgetWithText(String text) {
  return find.ancestor(
    of: find.text(text),
    matching: find.byType(Widget),
  );
}

/// 2. Finder for widgets with specific key
Finder findWidgetByKey(Key key) {
  return find.byKey(key);
}

/// 3. Finder for widgets with specific type
Finder findWidgetByType(Type type) {
  return find.byType(type);
}

/// 4. Finder for widgets with specific property
Finder findWidgetWithProperty(dynamic property) {
  return find.byWidgetPredicate(
    (widget) => widget.toString().contains(property.toString()),
  );
}

/// 5. Finder for widgets with specific padding
Finder findWidgetWithPadding(EdgeInsets padding) {
  return find.byWidgetPredicate(
    (widget) {
      if (widget is Padding) {
        return widget.padding == padding;
      }
      return false;
    },
  );
}

/// 6. Using custom finders
void main() {
  group('Custom Finders Tests', () {
    testWidgets('should find widget with text', (WidgetTester tester) async {
      // Arrange
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: Center(
              child: Text('Hello World'),
            ),
          ),
        ),
      );

      // Act: Use custom finder
      final widget = findWidgetWithText('Hello World');

      // Assert
      expect(widget, findsOneWidget);
    });

    testWidgets('should find widget with padding', (WidgetTester tester) async {
      // Arrange
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: Padding(
              padding: EdgeInsets.all(16),
              child: Text('Padded Text'),
            ),
          ),
        ),
      );

      // Act: Use custom finder
      final widget = findWidgetWithPadding(const EdgeInsets.all(16));

      // Assert
      expect(widget, findsOneWidget);
    });
  });
}
```

What's happening here?
- Custom finders for specific widgets
- Widget property matching
- Ancestor finder for complex trees
- Predicate-based finding

---

# Test Data Utilities

> **Creating** test data.

```dart
/// test_utils/data_utils.dart
import 'package:my_app/models/user.dart';

/// 1. Test data factory
class TestDataFactory {
  /// Create a test user
  static User createUser({
    String id = '1',
    String name = 'John Doe',
    String email = 'john@example.com',
    int age = 25,
  }) {
    return User(
      id: id,
      name: name,
      email: email,
      age: age,
    );
  }

  /// Create a list of test users
  static List<User> createUsers(int count) {
    return List.generate(count, (index) {
      return User(
        id: '${index + 1}',
        name: 'User ${index + 1}',
        email: 'user${index + 1}@example.com',
        age: 20 + index,
      );
    });
  }

  /// Create a test todo
  static Todo createTodo({
    String id = '1',
    String title = 'Test Todo',
    bool isCompleted = false,
  }) {
    return Todo(
      id: id,
      title: title,
      isCompleted: isCompleted,
    );
  }

  /// Create a list of test todos
  static List<Todo> createTodos(int count) {
    return List.generate(count, (index) {
      return Todo(
        id: '${index + 1}',
        title: 'Todo ${index + 1}',
        isCompleted: index % 2 == 0,
      );
    });
  }
}

/// 2. Test data with random values
class RandomTestData {
  static final Random _random = Random();

  static String randomString(int length) {
    const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    return String.fromCharCodes(
      Iterable.generate(
        length,
        (_) => chars.codeUnitAt(_random.nextInt(chars.length)),
      ),
    );
  }

  static int randomInt(int min, int max) {
    return min + _random.nextInt(max - min + 1);
  }

  static bool randomBool() {
    return _random.nextBool();
  }

  static String randomEmail() {
    return '${randomString(8)}@${randomString(5)}.com';
  }

  static User randomUser() {
    return User(
      id: '${randomInt(1, 1000)}',
      name: randomString(10),
      email: randomEmail(),
      age: randomInt(18, 80),
    );
  }
}

/// 3. Using test data utilities
void main() {
  group('Test Data Utilities Tests', () {
    test('should create test user', () {
      // Arrange
      final user = TestDataFactory.createUser();

      // Assert
      expect(user.id, '1');
      expect(user.name, 'John Doe');
      expect(user.email, 'john@example.com');
      expect(user.age, 25);
    });

    test('should create multiple test users', () {
      // Arrange
      final users = TestDataFactory.createUsers(3);

      // Assert
      expect(users.length, 3);
      expect(users[0].id, '1');
      expect(users[1].id, '2');
      expect(users[2].id, '3');
    });

    test('should create random test data', () {
      // Arrange
      final user = RandomTestData.randomUser();

      // Assert
      expect(user.id.isNotEmpty, true);
      expect(user.name.isNotEmpty, true);
      expect(user.email.contains('@'), true);
      expect(user.age >= 18 && user.age <= 80, true);
    });
  });
}
```

What's happening here?
- Test data factories
- Consistent test data
- Random data generation
- Reusable test objects

---

# Mock Utilities

> **Creating** mock utilities.

```dart
/// test_utils/mock_utils.dart
import 'package:mockito/mockito.dart';
import 'package:my_app/services/api_service.dart';
import 'package:my_app/services/auth_service.dart';

/// 1. Mock service factory
class MockServiceFactory {
  /// Create a mock ApiService
  static ApiService createMockApiService() {
    final mock = MockApiService();
    return mock;
  }

  /// Create a mock AuthService
  static AuthService createMockAuthService() {
    final mock = MockAuthService();
    return mock;
  }

  /// Setup mock API responses
  static void setupMockApiResponses(MockApiService mock) {
    // Setup default responses
    when(mock.fetchUser(any)).thenAnswer((_) async {
      return TestDataFactory.createUser();
    });

    when(mock.fetchUsers()).thenAnswer((_) async {
      return TestDataFactory.createUsers(5);
    });

    when(mock.createUser(any)).thenAnswer((_) async {
      return TestDataFactory.createUser(id: '3');
    });
  }

  /// Setup mock auth responses
  static void setupMockAuthResponses(MockAuthService mock) {
    when(mock.login(any, any)).thenAnswer((_) async {
      return true;
    });

    when(mock.logout()).thenAnswer((_) async {});
  }

  /// Create mock with error responses
  static void setupMockErrorResponses(MockApiService mock) {
    when(mock.fetchUser(any)).thenThrow(Exception('Network error'));
    when(mock.fetchUsers()).thenThrow(Exception('Network error'));
  }
}

/// 2. Mock classes
class MockApiService extends Mock implements ApiService {}
class MockAuthService extends Mock implements AuthService {}

/// 3. Using mock utilities
void main() {
  group('Mock Utilities Tests', () {
    late MockApiService mockApi;
    late AuthService authService;

    setUp(() {
      // Setup mock
      mockApi = MockServiceFactory.createMockApiService();
      MockServiceFactory.setupMockApiResponses(mockApi);
      authService = AuthService(apiService: mockApi);
    });

    test('should use mock responses', () async {
      // Act
      final user = await mockApi.fetchUser('1');

      // Assert
      expect(user.id, '1');
      expect(user.name, 'John Doe');
    });

    test('should handle mock errors', () async {
      // Arrange: Setup error response
      MockServiceFactory.setupMockErrorResponses(mockApi);

      // Act & Assert
      expect(
        () => mockApi.fetchUser('1'),
        throwsA(isA<Exception>()),
      );
    });
  });
}
```

What's happening here?
- Mock service factory
- Default mock responses
- Error mock setup
- Reusable mock utilities

---

# Real-World Examples

> **Common patterns** with test utilities.

```dart
/// 1. Complete test utility package
/// test_utils/test_utils.dart
library test_utils;

export 'finder_utils.dart';
export 'data_utils.dart';
export 'mock_utils.dart';
export 'widget_test_utils.dart';

/// 2. Test configuration
/// test/test_config.dart
import 'package:flutter_test/flutter_test.dart';

class TestConfig {
  /// Configure test environment
  static void configure() {
    // Set test environment variables
    // Configure test services
    // Initialize test data
  }

  /// Skip tests on specific platforms
  static bool skipOnPlatform(String platform) {
    // Implementation
    return false;
  }

  /// Run tests in specific mode
  static bool isDebugMode() {
    return true;
  }
}

/// 3. Using complete test utilities
void main() {
  // Configure tests
  TestConfig.configure();

  group('Complete Test Utility Usage', () {
    testWidgets('should test widget with utilities', (WidgetTester tester) async {
      // Arrange: Use data utility
      final user = TestDataFactory.createUser();

      // Arrange: Use widget utility
      await tester.pumpWidget(
        wrapWithMaterialApp(
          MyWidget(user: user),
        ),
      );

      await tester.pumpAndSettle();

      // Act: Use finder utility
      final nameWidget = findWidgetWithText(user.name);
      final emailWidget = findWidgetWithText(user.email);

      // Assert
      expect(nameWidget, findsOneWidget);
      expect(emailWidget, findsOneWidget);
    });

    testWidgets('should test with mocks', (WidgetTester tester) async {
      // Arrange: Setup mock
      final mockApi = MockServiceFactory.createMockApiService();
      MockServiceFactory.setupMockApiResponses(mockApi);
      final authService = AuthService(apiService: mockApi);

      // Act: Use mock
      final result = await authService.login('test@example.com', 'password');

      // Assert
      expect(result, true);
    });

    testWidgets('should test with random data', (WidgetTester tester) async {
      // Arrange: Create random data
      final users = List.generate(5, (_) => RandomTestData.randomUser());

      // Act: Test with random data
      for (final user in users) {
        await tester.pumpWidget(
          wrapWithMaterialApp(
            UserCard(user: user),
          ),
        );

        await tester.pumpAndSettle();

        // Assert: Verify each user is displayed correctly
        expect(find.text(user.name), findsOneWidget);
        expect(find.text(user.email), findsOneWidget);
      }
    });
  });
}
```

What's happening here?
- Complete test utility package
- Test configuration
- Combined utility usage
- Random data testing

---

# Best Practices

## Create Reusable Utilities

```dart
// Good - Reusable utilities
Widget wrapWithMaterialApp(Widget widget) { ... }
Finder findWidgetWithText(String text) { ... }

// Bad - Duplicated code
// Repeated MaterialApp wrapper in every test
```

## Use Consistent Naming

```dart
// Good - Consistent naming
wrapWithMaterialApp()
wrapWithTheme()
wrapWithProvider()

// Bad - Inconsistent naming
wrapWidget()
wrapApp()
wrap()
```

## Document Utilities

```dart
// Good - Documented utility
/// Wraps a widget with MaterialApp for testing.
/// 
/// Example:
/// ```
/// await tester.pumpWidget(
///   wrapWithMaterialApp(MyWidget()),
/// );
/// ```
Widget wrapWithMaterialApp(Widget widget) { ... }
```

---

# Common Mistakes

## Overcomplicating Utilities

Wrong:
```dart
// Too complex
Widget wrapWithEverything(Widget widget) {
  // Too many configurations
}
```

Correct:
```dart
// Simple and focused
Widget wrapWithMaterialApp(Widget widget) {
  // One responsibility
}
```

## Not Reusing Utilities

Wrong:
```dart
// Duplicated code
test('test 1', () {
  await tester.pumpWidget(
    MaterialApp(home: Scaffold(body: MyWidget())),
  );
});

test('test 2', () {
  await tester.pumpWidget(
    MaterialApp(home: Scaffold(body: MyWidget())),
  );
});
```

Correct:
```dart
// Reusable utility
test('test 1', () {
  await tester.pumpWidget(
    wrapWithMaterialApp(MyWidget()),
  );
});

test('test 2', () {
  await tester.pumpWidget(
    wrapWithMaterialApp(MyWidget()),
  );
});
```

---

# Summary

Test Utilities simplify testing by providing reusable functions, data, and mocks. Create utilities for widget wrapping, finders, test data, and mocks. Use consistent naming, document utilities, and avoid overcomplicating them. Good test utilities make tests more maintainable and efficient.

---

# Next Steps

- [Performance Testing](performance-testing.md)
- [CI/CD Integration](cicd-integration.md)
- [Testing Best Practices](testing-best-practices.md)

---

# Did You Know?

- Test utilities reduce boilerplate
- Custom finders simplify widget location
- Test data factories create consistent data
- Mock utilities simplify service testing
- Reusable utilities improve test maintenance
- Consistent naming helps organization
- Documented utilities are easier to use
- Test utilities are essential for large projects