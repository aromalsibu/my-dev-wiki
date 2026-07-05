# Comments & Documentation

Understand how to write effective comments and documentation in Dart.

---

# What is it?

Comments and documentation are textual annotations in your code that explain what the code does, how it works, and how to use it. Dart supports several types of comments, including single-line, multi-line, and documentation comments that can be processed by tools like `dartdoc` to generate API documentation.

---

# Why does it exist?

Comments and documentation exist to:

- Explain complex code logic
- Document API usage
- Generate API documentation automatically
- Help other developers understand your code
- Provide context and reasoning
- Make code more maintainable

---

# Comment Types

## Single-Line Comments

```dart
// This is a single-line comment
// It explains the code below
var name = 'Alice'; // Inline comment explaining variable

// Comments can also document complex logic
// This loop calculates the sum of even numbers
var sum = 0;
for (var i = 0; i < 10; i++) {
  if (i % 2 == 0) { // Check if even
    sum += i; // Add to sum
  }
}
```

---

## Multi-Line Comments

```dart
/*
 * This is a multi-line comment.
 * It spans multiple lines.
 * Useful for longer explanations.
 */
void complexFunction() {
  /*
    Inside function, can be used for
    temporary code comments or explanations
  */
}
```

---

## Documentation Comments

```dart
/// This is a documentation comment.
/// It's used to document classes, methods, and variables.
/// Tools like `dartdoc` process these comments.
class MyClass {
  /// A documented field
  final String name;
  
  /// A documented constructor
  /// [name] - The name to use
  MyClass(this.name);
  
  /// A documented method
  /// Returns a greeting with the [name]
  String greet() => 'Hello, $name';
}

/// Documentation with Markdown
/// 
/// This class represents a user with a [name] and [age].
/// 
/// Example:
/// ```dart
/// var user = User('Alice', 25);
/// print(user.greeting);
/// ```
class User {
  final String name;
  final int age;
  
  User(this.name, this.age);
  
  /// Gets a greeting message.
  /// 
  /// Returns a string like "Hello, Alice".
  String get greeting => 'Hello, $name';
}
```

---

# Documentation Syntax

## Documenting Parameters

```dart
/// Calculates the area of a rectangle.
/// 
/// [width] - The width of the rectangle
/// [height] - The height of the rectangle
/// 
/// Returns the area as a double.
double calculateArea(double width, double height) {
  return width * height;
}

/// Creates a new user.
/// 
/// [name] - User's full name (required)
/// [age] - User's age (optional, defaults to 0)
/// [email] - User's email (optional)
User createUser({
  required String name,
  int age = 0,
  String? email,
}) {
  // Implementation
}
```

---

## Documenting Returns

```dart
/// Processes the data and returns a formatted string.
/// 
/// Returns:
/// - A formatted string containing the processed data
/// - Throws [FormatException] if data is invalid
/// 
/// Example:
/// ```dart
/// var result = processData(data);
/// print(result); // "Processed: data"
/// ```
String processData(String data) {
  if (data.isEmpty) {
    throw FormatException('Data cannot be empty');
  }
  return 'Processed: $data';
}
```

---

## Documenting Exceptions

```dart
/// Divides two numbers.
/// 
/// [dividend] - The number to divide
/// [divisor] - The number to divide by
/// 
/// Returns the division result as a double.
/// 
/// Throws [IntegerDivisionByZeroException] if [divisor] is zero.
/// 
/// Example:
/// ```dart
/// var result = divide(10, 2); // 5.0
/// ```
double divide(double dividend, double divisor) {
  if (divisor == 0) {
    throw IntegerDivisionByZeroException();
  }
  return dividend / divisor;
}
```

---

# Documentation Tags

## Common Tags

```dart
/// A class representing a person.
/// 
/// ```dart
/// var person = Person('Alice', 25);
/// ```
/// 
/// @see [User] for user-specific functionality
/// @since 1.0.0
/// @author Alice Developer
/// @deprecated Use [User] instead
class Person {
  final String name;
  final int age;
  
  /// Creates a new [Person].
  /// 
  /// @param name The person's name
  /// @param age The person's age
  Person(this.name, this.age);
  
  /// Gets the person's greeting.
  /// 
  /// @return A greeting string
  String get greeting => 'Hello, $name';
}
```

---

## References

```dart
/// A user with [name] and [email].
/// 
/// See also:
/// - [UserService] for managing users
/// - [Authentication] for login functionality
/// - [Profile] for user profile management
class User {
  /// The user's full name.
  final String name;
  
  /// The user's email address.
  /// 
  /// Should be unique across the system.
  final String email;
  
  User({required this.name, required this.email});
}
```

---

# Markdown in Documentation

## Formatting

```dart
/// # User Management
/// 
/// This class handles user-related operations.
/// 
/// ## Features
/// 
/// - Create new users
/// - Update user profiles
/// - Delete users
/// - Find users by email
/// 
/// ## Example
/// 
/// ```dart
/// var userService = UserService();
/// var user = userService.createUser(
///   name: 'Alice',
///   email: 'alice@example.com',
/// );
/// ```
/// 
/// ## Notes
/// 
/// > **Important:** User emails must be unique.
/// 
/// > **Warning:** This is not thread-safe.
class UserService {
  // Implementation
}
```

---

## Lists and Tables

```dart
/// # User Roles
/// 
/// ## Available Roles
/// 
/// - `admin`: Full system access
/// - `moderator`: Content moderation only
/// - `user`: Basic user access
/// - `guest`: Read-only access
/// 
/// ## Permission Matrix
/// 
/// | Role     | Read | Write | Delete | Admin |
/// |----------|------|-------|--------|-------|
/// | admin    | ✅   | ✅    | ✅     | ✅    |
/// | moderator| ✅   | ✅    | ❌     | ❌    |
/// | user     | ✅   | ✅    | ❌     | ❌    |
/// | guest    | ✅   | ❌    | ❌     | ❌    |
enum UserRole { admin, moderator, user, guest }
```

---

# Best Practices

## Comment Why, Not What

```dart
// Bad: Commenting what the code does
// Increment counter by 1
counter++;

// Good: Commenting why
// Retry counter for connection attempts
retryCount++;

// Bad: Obvious comment
// Calculate total
total = price * quantity;

// Good: Non-obvious comment
// Apply bulk discount for orders over 10 items
total = price * quantity;
if (quantity > 10) total *= 0.9;
```

---

## Keep Comments Up-to-Date

```dart
// Bad: Outdated comment
// This function used to sort by name, but now sorts by age
void sortUsers() {
  // Sorts by age (updated)
  users.sort((a, b) => a.age.compareTo(b.age));
}

// Good: Accurate comment
/// Sorts users by age in ascending order.
void sortUsers() {
  users.sort((a, b) => a.age.compareTo(b.age));
}
```

---

## Use Documentation for Public APIs

```dart
// Good: Documented public API
/// Fetches user data from the server.
/// 
/// [userId] - The ID of the user to fetch
/// 
/// Returns a [Future] containing the [User] object.
/// 
/// Throws [NetworkException] if the request fails.
Future<User> fetchUser(String userId) async {
  // Implementation
}

// Bad: Undocumented public API
Future<User> fetchUser(String userId) async {
  // Implementation
}
```

---

## Use TODOs for Future Work

```dart
// TODO: Add error handling
// TODO(Alice): Implement caching
// TODO: Optimize this algorithm (performance issue)
// TODO: Remove this after v2.0

void processData() {
  // Implementation
  // TODO: Log processing time
}

// With tracking
// TODO(Ticket-123): Add validation
// TODO: Fix this in next sprint
// TODO(bugfix): Handle null case
```

---

# Common Documentation Patterns

## Class Documentation

```dart
/// A user representation in the system.
/// 
/// This class holds all user-related data and provides
/// methods for user operations.
/// 
/// Example:
/// ```dart
/// var user = User(
///   id: '123',
///   name: 'Alice',
///   email: 'alice@example.com',
/// );
/// print(user.greeting); // Hello, Alice
/// ```
class User {
  final String id;
  final String name;
  final String email;
  
  User({required this.id, required this.name, required this.email});
  
  /// Returns a greeting message.
  String get greeting => 'Hello, $name';
}
```

---

## Method Documentation

```dart
/// Validates the user's email address.
/// 
/// This method checks if the email format is valid and
/// if the domain exists.
/// 
/// Returns:
/// - `true` if the email is valid
/// - `false` otherwise
/// 
/// Example:
/// ```dart
/// var isValid = validateEmail('user@example.com');
/// print(isValid); // true
/// ```
bool validateEmail(String email) {
  return email.contains('@') && email.contains('.');
}
```

---

# Summary

Comments and documentation are essential for creating maintainable code. Use single-line, multi-line, and documentation comments appropriately, and document your public APIs thoroughly.

---

# Next Steps

Now that you understand comments and documentation, you've completed the fundamentals section! Continue to:

- [Classes](classes.md) - Object-Oriented Programming
- [Constructors](constructors.md)
- [Fields](fields.md)
- [Methods](methods.md)

---

# Did You Know?

- Dart's `dartdoc` tool generates HTML documentation
- Documentation comments support Markdown
- `///` is preferred over `/** */` in Dart
- Documentation is the first thing other developers see
- Good documentation can reduce support tickets
- Documentation helps with code reviews
- Tools can generate documentation from code