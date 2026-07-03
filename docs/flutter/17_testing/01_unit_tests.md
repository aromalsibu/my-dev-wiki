# Unit Tests

Understand how to write and run unit tests in Flutter to ensure your code works correctly.

---

# What is it?

Unit Tests are automated tests that verify the behavior of individual units of code, such as functions, methods, or classes. They are the foundation of testing in Flutter and help ensure that your business logic, data models, and utility functions work correctly. Unit tests are fast, isolated, and run without any UI or platform dependencies.

---

# Why does it exist?

Unit Tests exist to:

- Verify individual code units work correctly
- Catch bugs early in development
- Prevent regressions when changing code
- Document expected behavior
- Improve code design and maintainability
- Enable refactoring with confidence
- Reduce debugging time

---

# Setting Up Unit Tests

> **Configuring** testing in your project.

```yaml
# 1. Add test dependencies in pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  test: ^1.24.0
  mockito: ^5.4.0
```

```dart
// 2. Create test folder structure
// lib/
//   models/
//     user.dart
//   services/
//     api_service.dart
// test/
//   models/
//     user_test.dart
//   services/
//     api_service_test.dart

/// 3. Example: User model
/// lib/models/user.dart
class User {
  final String id;
  final String name;
  final String email;
  final int age;

  User({
    required this.id,
    required this.name,
    required this.email,
    required this.age,
  });

  // Factory constructor to create User from JSON
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      age: json['age'] as int,
    );
  }

  // Convert User to JSON
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'age': age,
    };
  }

  // Create a copy with updated fields
  User copyWith({
    String? id,
    String? name,
    String? email,
    int? age,
  }) {
    return User(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      age: age ?? this.age,
    );
  }

  // Validation method
  bool isValid() {
    return id.isNotEmpty &&
        name.isNotEmpty &&
        email.contains('@') &&
        age >= 0;
  }

  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is User &&
        other.id == id &&
        other.name == name &&
        other.email == email &&
        other.age == age;
  }

  @override
  int get hashCode {
    return id.hashCode ^
        name.hashCode ^
        email.hashCode ^
        age.hashCode;
  }
}
```

What's happening here?
- test package for unit tests
- Organized test structure
- Models to test
- Testing utilities

---

# Writing Unit Tests

> **Creating** your first unit tests.

```dart
// test/models/user_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/models/user.dart';

/// 1. Basic test group
void main() {
  // Group tests by class or feature
  group('User Model Tests', () {
    // 2. Setup test data
    const testUser = User(
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
      age: 25,
    );

    // 3. Test initialization
    test('should create a User instance correctly', () {
      // Arrange
      const user = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      // Assert
      expect(user.id, '1');
      expect(user.name, 'John Doe');
      expect(user.email, 'john@example.com');
      expect(user.age, 25);
    });

    // 4. Test JSON serialization
    test('should convert from JSON correctly', () {
      // Arrange
      final json = {
        'id': '1',
        'name': 'John Doe',
        'email': 'john@example.com',
        'age': 25,
      };

      // Act
      final user = User.fromJson(json);

      // Assert
      expect(user.id, '1');
      expect(user.name, 'John Doe');
      expect(user.email, 'john@example.com');
      expect(user.age, 25);
    });

    // 5. Test JSON deserialization
    test('should convert to JSON correctly', () {
      // Arrange
      const user = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      // Act
      final json = user.toJson();

      // Assert
      expect(json['id'], '1');
      expect(json['name'], 'John Doe');
      expect(json['email'], 'john@example.com');
      expect(json['age'], 25);
    });

    // 6. Test copyWith method
    test('should create a copy with updated fields', () {
      // Arrange
      const user = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      // Act
      final updatedUser = user.copyWith(
        name: 'Jane Doe',
        age: 26,
      );

      // Assert
      expect(updatedUser.id, '1');
      expect(updatedUser.name, 'Jane Doe');
      expect(updatedUser.email, 'john@example.com');
      expect(updatedUser.age, 26);
    });

    // 7. Test validation
    test('should validate user correctly', () {
      // Arrange
      const validUser = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      const invalidUser = User(
        id: '',
        name: 'John Doe',
        email: 'invalid-email',
        age: -5,
      );

      // Assert
      expect(validUser.isValid(), true);
      expect(invalidUser.isValid(), false);
    });

    // 8. Test equality
    test('should compare users correctly', () {
      // Arrange
      const user1 = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      const user2 = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      const user3 = User(
        id: '2',
        name: 'Jane Doe',
        email: 'jane@example.com',
        age: 30,
      );

      // Assert
      expect(user1, user2); // Equal
      expect(user1, isNot(user3)); // Not equal
    });
  });

  // 9. Test with setup and teardown
  group('User Model with Setup', () {
    // Setup runs before each test
    setUp(() {
      print('Setting up test');
    });

    // Teardown runs after each test
    tearDown(() {
      print('Cleaning up after test');
    });

    test('should create user with default values', () {
      // Test implementation
    });
  });

  // 10. Parameterized tests
  group('Parameterized Tests', () {
    // Test multiple scenarios with different inputs
    test('should validate email correctly', () {
      const validEmails = ['john@example.com', 'jane@test.co', 'user@domain.org'];
      const invalidEmails = ['john', 'john@', '@example.com', 'john@example'];

      for (final email in validEmails) {
        expect(User(
          id: '1',
          name: 'Test',
          email: email,
          age: 25,
        ).isValid(), true);
      }

      for (final email in invalidEmails) {
        expect(User(
          id: '1',
          name: 'Test',
          email: email,
          age: 25,
        ).isValid(), false);
      }
    });
  });
}
```

What's happening here?
- group organizes related tests
- test defines individual tests
- setUp runs before each test
- tearDown runs after each test
- expect asserts expected values

---

# Testing Services

> **Testing** services and business logic.

```dart
// lib/services/api_service.dart

import 'dart:convert';
import 'package:http/http.dart' as http;

/// API service for making network requests
class ApiService {
  final http.Client client;
  final String baseUrl;

  ApiService({
    required this.client,
    required this.baseUrl,
  });

  /// Fetch user data from API
  Future<User> fetchUser(String id) async {
    try {
      final response = await client.get(
        Uri.parse('$baseUrl/users/$id'),
      );

      if (response.statusCode == 200) {
        final data = json.decode(response.body) as Map<String, dynamic>;
        return User.fromJson(data);
      } else {
        throw Exception('Failed to fetch user: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }

  /// Fetch multiple users
  Future<List<User>> fetchUsers() async {
    try {
      final response = await client.get(
        Uri.parse('$baseUrl/users'),
      );

      if (response.statusCode == 200) {
        final data = json.decode(response.body) as List<dynamic>;
        return data.map((json) => User.fromJson(json)).toList();
      } else {
        throw Exception('Failed to fetch users: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }

  /// Create a new user
  Future<User> createUser(User user) async {
    try {
      final response = await client.post(
        Uri.parse('$baseUrl/users'),
        headers: {'Content-Type': 'application/json'},
        body: json.encode(user.toJson()),
      );

      if (response.statusCode == 201) {
        final data = json.decode(response.body) as Map<String, dynamic>;
        return User.fromJson(data);
      } else {
        throw Exception('Failed to create user: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }
}
```

```dart
// test/services/api_service_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:mockito/mockito.dart';
import 'package:my_app/models/user.dart';
import 'package:my_app/services/api_service.dart';

// 1. Create mock classes
class MockClient extends Mock implements http.Client {}

/// ApiService unit tests
void main() {
  late ApiService apiService;
  late MockClient mockClient;

  // Setup before each test
  setUp(() {
    mockClient = MockClient();
    apiService = ApiService(
      client: mockClient,
      baseUrl: 'https://api.example.com',
    );
  });

  // 2. Test fetching a single user
  group('fetchUser', () {
    test('should return User when response is successful', () async {
      // Arrange
      const userId = '1';
      final mockResponse = {
        'id': '1',
        'name': 'John Doe',
        'email': 'john@example.com',
        'age': 25,
      };

      // Mock the HTTP client response
      when(mockClient.get(any)).thenAnswer((_) async {
        return http.Response(
          json.encode(mockResponse),
          200,
        );
      });

      // Act
      final user = await apiService.fetchUser(userId);

      // Assert
      expect(user.id, '1');
      expect(user.name, 'John Doe');
      expect(user.email, 'john@example.com');
      expect(user.age, 25);
    });

    test('should throw exception when response status is not 200', () async {
      // Arrange
      const userId = '1';

      // Mock error response
      when(mockClient.get(any)).thenAnswer((_) async {
        return http.Response(
          'Not Found',
          404,
        );
      });

      // Act & Assert
      expect(
        () => apiService.fetchUser(userId),
        throwsA(isA<Exception>()),
      );
    });

    test('should throw exception on network error', () async {
      // Arrange
      const userId = '1';

      // Mock network error
      when(mockClient.get(any)).thenThrow(Exception('Network error'));

      // Act & Assert
      expect(
        () => apiService.fetchUser(userId),
        throwsA(isA<Exception>()),
      );
    });
  });

  // 3. Test fetching multiple users
  group('fetchUsers', () {
    test('should return List<User> when response is successful', () async {
      // Arrange
      final mockResponse = [
        {
          'id': '1',
          'name': 'John Doe',
          'email': 'john@example.com',
          'age': 25,
        },
        {
          'id': '2',
          'name': 'Jane Doe',
          'email': 'jane@example.com',
          'age': 30,
        },
      ];

      when(mockClient.get(any)).thenAnswer((_) async {
        return http.Response(
          json.encode(mockResponse),
          200,
        );
      });

      // Act
      final users = await apiService.fetchUsers();

      // Assert
      expect(users.length, 2);
      expect(users[0].id, '1');
      expect(users[1].id, '2');
    });

    test('should throw exception on error response', () async {
      // Arrange
      when(mockClient.get(any)).thenAnswer((_) async {
        return http.Response(
          'Error',
          500,
        );
      });

      // Act & Assert
      expect(
        () => apiService.fetchUsers(),
        throwsA(isA<Exception>()),
      );
    });
  });

  // 4. Test creating a user
  group('createUser', () {
    test('should create and return User when successful', () async {
      // Arrange
      const user = User(
        id: '3',
        name: 'Bob Smith',
        email: 'bob@example.com',
        age: 35,
      );

      final mockResponse = {
        'id': '3',
        'name': 'Bob Smith',
        'email': 'bob@example.com',
        'age': 35,
      };

      when(mockClient.post(any, headers: anyNamed('headers'), body: anyNamed('body')))
          .thenAnswer((_) async {
        return http.Response(
          json.encode(mockResponse),
          201,
        );
      });

      // Act
      final createdUser = await apiService.createUser(user);

      // Assert
      expect(createdUser.id, '3');
      expect(createdUser.name, 'Bob Smith');
      expect(createdUser.email, 'bob@example.com');
      expect(createdUser.age, 35);
    });
  });
}
```

What's happening here?
- Mockito for mocking dependencies
- Testing success and error cases
- Network request mocking
- Exception testing

---

# Testing Async Code

> **Testing** asynchronous operations.

```dart
// lib/services/auth_service.dart

/// Authentication service
class AuthService {
  final ApiService apiService;
  User? _currentUser;

  AuthService({required this.apiService});

  User? get currentUser => _currentUser;

  /// Login with email and password
  Future<bool> login(String email, String password) async {
    try {
      // Simulate authentication
      // In a real app, this would call the API
      if (email.isEmpty || password.isEmpty) {
        return false;
      }

      // Fetch user from API
      final user = await apiService.fetchUser('1');
      
      // Simulate password verification
      if (password == 'password123') {
        _currentUser = user;
        return true;
      }

      return false;
    } catch (e) {
      print('Login error: $e');
      return false;
    }
  }

  /// Logout user
  void logout() {
    _currentUser = null;
  }

  /// Check if user is authenticated
  bool get isAuthenticated => _currentUser != null;
}
```

```dart
// test/services/auth_service_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:my_app/models/user.dart';
import 'package:my_app/services/api_service.dart';
import 'package:my_app/services/auth_service.dart';

/// Async test example
void main() {
  late AuthService authService;
  late ApiService mockApiService;

  setUp(() {
    mockApiService = MockApiService();
    authService = AuthService(apiService: mockApiService);
  });

  // 1. Test login with async
  group('login', () {
    test('should return true with valid credentials', () async {
      // Arrange
      const user = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      // Mock the API service to return a user
      when(mockApiService.fetchUser('1')).thenAnswer((_) async => user);

      // Act
      final result = await authService.login('john@example.com', 'password123');

      // Assert
      expect(result, true);
      expect(authService.currentUser, user);
    });

    test('should return false with invalid password', () async {
      // Arrange
      const user = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      when(mockApiService.fetchUser('1')).thenAnswer((_) async => user);

      // Act
      final result = await authService.login('john@example.com', 'wrongpassword');

      // Assert
      expect(result, false);
      expect(authService.currentUser, null);
    });

    test('should return false with empty credentials', () async {
      // Act
      final result = await authService.login('', '');

      // Assert
      expect(result, false);
      expect(authService.currentUser, null);
    });

    test('should return false on API error', () async {
      // Arrange
      when(mockApiService.fetchUser('1')).thenThrow(Exception('API error'));

      // Act
      final result = await authService.login('john@example.com', 'password123');

      // Assert
      expect(result, false);
      expect(authService.currentUser, null);
    });
  });

  // 2. Test logout
  group('logout', () {
    test('should clear current user', () async {
      // Arrange
      const user = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      when(mockApiService.fetchUser('1')).thenAnswer((_) async => user);
      await authService.login('john@example.com', 'password123');
      expect(authService.currentUser, isNotNull);

      // Act
      authService.logout();

      // Assert
      expect(authService.currentUser, null);
    });
  });

  // 3. Test authentication status
  group('isAuthenticated', () {
    test('should return true when user is logged in', () async {
      // Arrange
      const user = User(
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
      );

      when(mockApiService.fetchUser('1')).thenAnswer((_) async => user);
      await authService.login('john@example.com', 'password123');

      // Assert
      expect(authService.isAuthenticated, true);
    });

    test('should return false when user is logged out', () {
      // Assert
      expect(authService.isAuthenticated, false);
    });
  });
}

/// Mock ApiService for testing
class MockApiService extends Mock implements ApiService {}
```

What's happening here?
- Testing async methods with Future
- Mocking API calls
- Testing authentication flows
- State verification

---

# Test Coverage

> **Measuring** test coverage.

```bash
# 1. Run tests with coverage
flutter test --coverage

# 2. Generate coverage report
genhtml coverage/lcov.info -o coverage/html

# 3. Open coverage report
open coverage/html/index.html

# 4. Check coverage in terminal
flutter test --coverage && lcov --summary coverage/lcov.info

# 5. View coverage in VS Code
# Install Coverage Gutters extension
# Click "Coverage Gutters: Display Coverage"
```

What's happening here?
- Coverage reports show untested code
- Helps identify missing tests
- Improves test completeness
- Ensures code quality

---

# Real-World Examples

> **Common patterns** in unit testing.

```dart
/// 1. Testing with testWidgets (Widget tests)
class MyWidget extends StatelessWidget {
  const MyWidget({super.key, required this.data});

  final String data;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My Widget')),
      body: Center(
        child: Text(
          data,
          style: const TextStyle(fontSize: 24),
        ),
      ),
    );
  }
}

void main() {
  testWidgets('MyWidget displays data correctly', (WidgetTester tester) async {
    // Arrange
    const testData = 'Hello, World!';

    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: MyWidget(data: testData),
      ),
    );

    // Assert
    expect(find.text(testData), findsOneWidget);
    expect(find.text('Some other text'), findsNothing);
  });
}

/// 2. Testing with golden files
// test/widgets/my_widget_test.dart
void main() {
  testWidgets('MyWidget matches golden image', (WidgetTester tester) async {
    // Arrange
    const testData = 'Hello, World!';

    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: MyWidget(data: testData),
      ),
    );

    // Assert
    await expectLater(
      find.byType(MyWidget),
      matchesGoldenFile('goldens/my_widget.png'),
    );
  });
}

/// 3. Testing with test groups
void main() {
  group('User Model Tests', () {
    setUp(() {
      // Setup before each test
    });

    test('should create user', () {
      // Test creation
    });

    test('should validate user', () {
      // Test validation
    });

    group('JSON Tests', () {
      test('should serialize to JSON', () {
        // Test JSON serialization
      });

      test('should deserialize from JSON', () {
        // Test JSON deserialization
      });
    });
  });
}
```

What's happening here?
- Widget tests with testWidgets
- Golden file testing
- Grouped test organization

---

# Best Practices

## Test One Thing Per Test

```dart
// Good - One assertion per test
test('should return user when fetch succeeds', () {
  // Test one thing
});

test('should throw error when fetch fails', () {
  // Test another thing
});

// Bad - Multiple assertions in one test
test('should handle fetch', () {
  // Tests multiple things
});
```

## Use Meaningful Names

```dart
// Good - Descriptive test names
test('should calculate total correctly when tax is applied', () { ... });

// Bad - Vague test names
test('test1', () { ... });
```

## Keep Tests Independent

```dart
// Good - Independent tests
test('test A', () {
  // Independent setup
});

test('test B', () {
  // Independent setup
});

// Bad - Tests depend on each other
int sharedState = 0;
test('test 1', () {
  sharedState = 1;
});

test('test 2', () {
  // Depends on test 1
});
```

---

# Common Mistakes

## Not Testing Edge Cases

Wrong:
```dart
// Only tests happy path
test('should return user', () {
  // Tests successful case only
});
```

Correct:
```dart
// Tests all cases
test('should return user', () {
  // Happy path
});

test('should throw error when not found', () {
  // Error case
});

test('should handle empty input', () {
  // Edge case
});
```

## Not Using Mocks

Wrong:
```dart
// Uses real API in tests
test('should fetch user', () {
  // Makes real network request
});
```

Correct:
```dart
// Uses mock API in tests
test('should fetch user', () {
  // Uses mock API
});
```

---

# Summary

Unit Tests verify individual code units work correctly. Use test package for unit tests, Mockito for mocking dependencies, and testWidgets for widget tests. Write one assertion per test, test edge cases, and keep tests independent. Unit tests are essential for building reliable, maintainable Flutter applications.

---

# Next Steps

- [Widget Tests](widget-tests.md)
- [Integration Tests](integration-tests.md)
- [Golden Tests](golden-tests.md)

---

# Did You Know?

- Unit tests run without UI
- Mockito creates test doubles
- testWidgets tests widgets
- Coverage reports show untested code
- Golden tests compare visual appearance
- Tests should be independent
- One assertion per test is best practice
- Tests improve code quality