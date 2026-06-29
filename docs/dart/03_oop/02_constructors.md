# Constructors

Understand how to create and use constructors in Dart classes.

---

# What is it?

Constructors are special methods that are called when creating instances of a class. They initialize the object's state, set up fields, and can perform any necessary setup operations. Dart provides various types of constructors to handle different initialization scenarios.

---

# Why does it exist?

Constructors exist to:

- Initialize object state when created
- Set up fields with initial values
- Perform validation and setup logic
- Control how objects are created
- Enable different creation patterns
- Support dependency injection

---

# Generative Constructors

## Basic Constructor

```dart
class Person {
  String name;
  int age;
  
  // Generative constructor
  Person(this.name, this.age);
  
  // Constructor with validation
  Person.withValidation(this.name, this.age) {
    if (age < 0) {
      throw ArgumentError('Age cannot be negative');
    }
    if (name.isEmpty) {
      throw ArgumentError('Name cannot be empty');
    }
  }
}

// Usage
var person1 = Person('Alice', 25);
var person2 = Person.withValidation('Bob', 30);
```

---

## Constructor with Initializer List

```dart
class User {
  final String id;
  final String name;
  final DateTime createdAt;
  
  // Constructor with initializer list
  User(String id, String name)
      : id = id,
        name = name,
        createdAt = DateTime.now() {
    // Body runs after initialization
    if (id.isEmpty) {
      throw ArgumentError('ID cannot be empty');
    }
  }
  
  // Multiple initializers
  User.fromEmail(String email, String name)
      : id = email.hashCode.toString(),
        name = name,
        createdAt = DateTime.now() {
    if (!email.contains('@')) {
      throw ArgumentError('Invalid email');
    }
  }
}

// Usage
var user1 = User('123', 'Alice');
var user2 = User.fromEmail('alice@example.com', 'Alice');
```

---

# Named Constructors

## Multiple Constructors

```dart
class Rectangle {
  double width;
  double height;
  
  // Default constructor
  Rectangle(this.width, this.height);
  
  // Named constructor - square
  Rectangle.square(double size)
      : width = size,
        height = size;
  
  // Named constructor - from points
  Rectangle.fromPoints(double x1, double y1, double x2, double y2)
      : width = x2 - x1,
        height = y2 - y1 {
    if (width <= 0 || height <= 0) {
      throw ArgumentError('Invalid points');
    }
  }
  
  // Named constructor - from string
  Rectangle.fromString(String value) : this(0, 0) {
    var parts = value.split('x');
    if (parts.length == 2) {
      width = double.parse(parts[0]);
      height = double.parse(parts[1]);
    }
  }
  
  double get area => width * height;
}

// Usage
var rect1 = Rectangle(10, 20);
var rect2 = Rectangle.square(15);
var rect3 = Rectangle.fromPoints(0, 0, 10, 20);
var rect4 = Rectangle.fromString('10x20');

print(rect1.area); // 200
print(rect2.area); // 225
print(rect3.area); // 200
print(rect4.area); // 200
```

---

# Redirecting Constructors

## Constructor Redirection

```dart
class Point {
  final double x;
  final double y;
  
  // Main constructor
  Point(this.x, this.y);
  
  // Redirecting constructor to main
  Point.zero() : this(0, 0);
  
  // Redirecting to another named constructor
  Point.fromPolar(double radius, double angle)
      : this(radius * math.cos(angle), radius * math.sin(angle));
  
  // Multiple redirects
  Point.origin() : this.zero();
  
  @override
  String toString() => 'Point($x, $y)';
}

// Usage
var p1 = Point(3, 4);
var p2 = Point.zero(); // (0, 0)
var p3 = Point.origin(); // (0, 0)
var p4 = Point.fromPolar(5, 3.14); // (-4.99, 0.05)
```

---

# Factory Constructors

## Creating Factory Constructors

```dart
class Logger {
  final String name;
  static final Map<String, Logger> _cache = {};
  
  // Factory constructor (may return existing instance)
  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }
  
  // Private constructor
  Logger._internal(this.name);
  
  void log(String message) {
    print('[$name] $message');
  }
}

// Usage - Always returns same instance for same name
var logger1 = Logger('App');
var logger2 = Logger('App');
print(identical(logger1, logger2)); // true

var logger3 = Logger('Network');
print(identical(logger1, logger3)); // false

logger1.log('Starting app');
logger3.log('Sending request');
```

---

## Factory Constructor with Validation

```dart
class Email {
  final String value;
  
  // Factory constructor with validation
  factory Email(String value) {
    if (!value.contains('@')) {
      throw ArgumentError('Invalid email: $value');
    }
    return Email._internal(value);
  }
  
  // Private constructor
  Email._internal(this.value);
  
  String get domain => value.split('@')[1];
  String get username => value.split('@')[0];
  
  @override
  String toString() => value;
}

// Usage
var email = Email('user@example.com');
print(email.username); // user
print(email.domain); // example.com

// This throws an error
// var invalid = Email('invalid-email');
```

---

## Factory Constructor from JSON

```dart
class User {
  final String id;
  final String name;
  final String email;
  final DateTime createdAt;
  
  // Private constructor
  User._internal({
    required this.id,
    required this.name,
    required this.email,
    required this.createdAt,
  });
  
  // Factory constructor from JSON
  factory User.fromJson(Map<String, dynamic> json) {
    return User._internal(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      createdAt: DateTime.parse(json['createdAt'] as String),
    );
  }
  
  // Factory with default values
  factory User.create(String name, String email) {
    return User._internal(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      name: name,
      email: email,
      createdAt: DateTime.now(),
    );
  }
  
  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'email': email,
    'createdAt': createdAt.toIso8601String(),
  };
}

// Usage
var user = User.fromJson({
  'id': '123',
  'name': 'Alice',
  'email': 'alice@example.com',
  'createdAt': '2024-01-01T00:00:00.000Z',
});

var newUser = User.create('Bob', 'bob@example.com');
print(newUser.name); // Bob
```

---

# Constant Constructors

## Creating Constant Objects

```dart
class ImmutablePoint {
  final double x;
  final double y;
  
  // Constant constructor
  const ImmutablePoint(this.x, this.y);
  
  // Constant named constructor
  const ImmutablePoint.zero() : this(0, 0);
  
  // Regular constructor (can't be const)
  ImmutablePoint.fromPolar(double radius, double angle)
      : this(radius * math.cos(angle), radius * math.sin(angle));
}

// Usage - All instances are compile-time constants
const p1 = ImmutablePoint(3, 4);
const p2 = ImmutablePoint.zero();

// Equality check
const p3 = ImmutablePoint(3, 4);
print(identical(p1, p3)); // true (same constant instance)

// Non-constant factory
var p4 = ImmutablePoint.fromPolar(5, 1.5);
print(p4); // Not a constant
```

---

# Private Constructors

## Singleton Pattern

```dart
class Singleton {
  // Private constructor
  Singleton._internal();
  
  // Static instance
  static final Singleton _instance = Singleton._internal();
  
  // Factory constructor returns the instance
  factory Singleton() {
    return _instance;
  }
  
  // Static getter
  static Singleton get instance => _instance;
  
  void doSomething() {
    print('Singleton doing something');
  }
}

// Usage - Only one instance
var singleton1 = Singleton();
var singleton2 = Singleton();
print(identical(singleton1, singleton2)); // true

var singleton3 = Singleton.instance;
print(identical(singleton1, singleton3)); // true

singleton1.doSomething();
```

---

## Utility Class (All Static Members)

```dart
class MathUtils {
  // Private constructor prevents instantiation
  MathUtils._();
  
  static int add(int a, int b) => a + b;
  static int subtract(int a, int b) => a - b;
  static int multiply(int a, int b) => a * b;
  static double divide(int a, int b) => a / b;
  
  static const double pi = 3.14159;
  static const double e = 2.71828;
}

// Usage - No instance needed
print(MathUtils.add(5, 3)); // 8
print(MathUtils.pi); // 3.14159

// Error: Can't instantiate
// var utils = MathUtils(); // Error: Private constructor
```

---

# Best Practices

## Use Factory Constructors for Caching

```dart
// Good: Caching with factory
class DatabaseConnection {
  static final Map<String, DatabaseConnection> _connections = {};
  final String url;
  
  factory DatabaseConnection(String url) {
    return _connections.putIfAbsent(url, () => DatabaseConnection._internal(url));
  }
  
  DatabaseConnection._internal(this.url);
}

// Bad: Creating new connections each time
class BadDatabaseConnection {
  final String url;
  BadDatabaseConnection(this.url);
}
```

---

## Validate in Constructors

```dart
// Good: Validation in constructor
class PositiveNumber {
  final int value;
  
  PositiveNumber(this.value) {
    if (value <= 0) {
      throw ArgumentError('Value must be positive');
    }
  }
}

// Bad: No validation
class UnvalidatedNumber {
  int value;
  UnvalidatedNumber(this.value);
}
```

---

## Use Initializer List for Final Fields

```dart
// Good: Using initializer list
class Product {
  final String id;
  final String name;
  final double price;
  
  Product(String id, String name, double price)
      : id = id,
        name = name,
        price = price {
    if (price < 0) {
      throw ArgumentError('Price cannot be negative');
    }
  }
}

// Bad: Assigning final in body
class BadProduct {
  final String id;
  final String name;
  
  BadProduct(String id, String name) {
    this.id = id; // Error! Can't assign to final in body
    this.name = name; // Error! Can't assign to final in body
  }
}
```

---

# Common Mistakes

## Forgetting to Initialize Final Fields

Wrong:

```dart
class User {
  final String id;
  final String name;
  
  User(this.name) {
    // id not initialized!
  }
}
```

Correct:

```dart
class User {
  final String id;
  final String name;
  
  User(this.name, this.id);
  // Or
  User(this.name) : id = DateTime.now().millisecondsSinceEpoch.toString();
}
```

---

## Using const with Non-constant Fields

Wrong:

```dart
class Point {
  final double x;
  final double y;
  
  const Point(this.x, this.y); // Error: Can't use const with double
}
```

Correct:

```dart
class Point {
  final double x;
  final double y;
  
  const Point(this.x, this.y); // Works with const if values are constants
}

const p1 = Point(1.5, 2.5); // Works
```

---

# Summary

Constructors are essential for creating and initializing objects. Dart provides various constructor types for different use cases, including generative, named, redirecting, factory, and constant constructors.

---

# Next Steps

Now that you understand constructors, continue to:

- [Fields](fields.md)
- [Methods](methods.md)
- [Getters & Setters](getters-setters.md)
- [Inheritance](inheritance.md)

---

# Did You Know?

- Dart supports multiple named constructors
- Factory constructors can return existing instances
- Const constructors create compile-time constants
- Redirecting constructors call another constructor
- Initializer lists run before the constructor body
- Private constructors prevent instantiation
- Factory constructors don't use `this`