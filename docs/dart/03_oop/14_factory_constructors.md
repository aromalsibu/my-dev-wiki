# Factory Constructors

Understand how to use factory constructors for flexible object creation in Dart.

---

# What is it?

Factory constructors are special constructors that don't always create a new instance of the class. Instead, they can return existing instances, return subtypes, or perform complex initialization logic before creating an object.

---

# Why does it exist?

Factory constructors exist to:

- Return existing instances (caching)
- Create instances from different data sources
- Return subtypes of the class
- Perform complex initialization
- Implement singleton pattern
- Control object creation

---

# Basic Factory Constructors

## Simple Factory Constructor

```dart
class User {
  final String name;
  final int age;
  
  // Private constructor
  User._internal(this.name, this.age);
  
  // Factory constructor
  factory User(String name, int age) {
    // Validation logic
    if (name.isEmpty) {
      throw ArgumentError('Name cannot be empty');
    }
    if (age < 0 || age > 150) {
      throw ArgumentError('Invalid age');
    }
    
    // Create and return instance
    return User._internal(name, age);
  }
  
  // Factory with default values
  factory User.guest() {
    return User._internal('Guest', 0);
  }
}

// Usage
var user1 = User('Alice', 25);
var user2 = User.guest();

print(user1.name); // Alice
print(user2.name); // Guest
```

---

## Factory with Caching

```dart
class Logger {
  final String name;
  
  // Cache of instances
  static final Map<String, Logger> _cache = {};
  
  // Private constructor
  Logger._internal(this.name);
  
  // Factory constructor with caching
  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }
  
  void log(String message) {
    print('[$name] $message');
  }
}

// Usage
var logger1 = Logger('App');
var logger2 = Logger('App');
var logger3 = Logger('Network');

print(identical(logger1, logger2)); // true (cached)
print(identical(logger1, logger3)); // false (different)

logger1.log('Starting app'); // [App] Starting app
logger3.log('Connecting...'); // [Network] Connecting...
```

---

# Factory with Different Types

## Returning Subtypes

```dart
abstract class Shape {
  double get area;
  double get perimeter;
}

class Circle extends Shape {
  final double radius;
  
  Circle(this.radius);
  
  @override
  double get area => math.pi * radius * radius;
  
  @override
  double get perimeter => 2 * math.pi * radius;
}

class Rectangle extends Shape {
  final double width;
  final double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
}

class Square extends Rectangle {
  Square(double side) : super(side, side);
}

class ShapeFactory {
  // Factory constructor
  factory ShapeFactory.create(String type, Map<String, double> params) {
    switch (type.toLowerCase()) {
      case 'circle':
        return Circle(params['radius']!);
      case 'rectangle':
        return Rectangle(params['width']!, params['height']!);
      case 'square':
        return Square(params['side']!);
      default:
        throw ArgumentError('Unknown shape type: $type');
    }
  }
}

// Usage
var circle = ShapeFactory.create('circle', {'radius': 5});
var rectangle = ShapeFactory.create('rectangle', {'width': 10, 'height': 20});
var square = ShapeFactory.create('square', {'side': 15});

print(circle.area); // 78.54
print(rectangle.area); // 200
print(square.area); // 225
```

---

# Factory with JSON

## Creating from JSON

```dart
class Product {
  final String id;
  final String name;
  final double price;
  final String category;
  final bool inStock;
  
  // Private constructor
  Product._internal({
    required this.id,
    required this.name,
    required this.price,
    required this.category,
    this.inStock = true,
  });
  
  // Factory from JSON
  factory Product.fromJson(Map<String, dynamic> json) {
    return Product._internal(
      id: json['id'] as String,
      name: json['name'] as String,
      price: (json['price'] as num).toDouble(),
      category: json['category'] as String,
      inStock: json['inStock'] as bool? ?? true,
    );
  }
  
  // Factory with default
  factory Product.basic(String name, double price) {
    return Product._internal(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      name: name,
      price: price,
      category: 'General',
      inStock: true,
    );
  }
  
  // To JSON
  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'price': price,
    'category': category,
    'inStock': inStock,
  };
}

// Usage
var product1 = Product.fromJson({
  'id': 'P001',
  'name': 'Laptop',
  'price': 999.99,
  'category': 'Electronics',
  'inStock': true,
});

var product2 = Product.basic('Mouse', 29.99);

print(product1.name); // Laptop
print(product2.price); // 29.99
```

---

# Factory Singleton Pattern

## Singleton with Factory

```dart
class DatabaseConnection {
  static DatabaseConnection? _instance;
  
  // Private constructor
  DatabaseConnection._internal() {
    print('Creating new database connection');
  }
  
  // Factory constructor
  factory DatabaseConnection() {
    _instance ??= DatabaseConnection._internal();
    return _instance!;
  }
  
  // Instance methods
  void query(String sql) {
    print('Executing: $sql');
  }
  
  void disconnect() {
    print('Disconnecting');
    _instance = null;
  }
}

// Usage
var db1 = DatabaseConnection(); // Creating new database connection
var db2 = DatabaseConnection(); // Returns existing instance

print(identical(db1, db2)); // true

db1.query('SELECT * FROM users');
db2.query('SELECT * FROM products');

db1.disconnect(); // Disconnecting
var db3 = DatabaseConnection(); // Creating new database connection
print(identical(db1, db3)); // false
```

---

# Factory with Complex Validation

## Advanced Factory Constructors

```dart
class Email {
  final String value;
  
  // Private constructor
  Email._(this.value);
  
  // Factory with validation
  factory Email(String value) {
    if (!_isValid(value)) {
      throw FormatException('Invalid email: $value');
    }
    return Email._(value);
  }
  
  // Factory with specific domains
  factory Email.company(String value, String domain) {
    if (!_isValid(value)) {
      throw FormatException('Invalid email: $value');
    }
    if (!value.endsWith('@$domain')) {
      throw FormatException('Email must be from domain: $domain');
    }
    return Email._(value);
  }
  
  // Factory for test emails
  factory Email.test() {
    return Email._('test@example.com');
  }
  
  // Validation helper
  static bool _isValid(String email) {
    return email.contains('@') && email.contains('.');
  }
  
  String get domain => value.split('@')[1];
  String get username => value.split('@')[0];
  
  @override
  String toString() => value;
}

// Usage
var email1 = Email('user@example.com');
var email2 = Email.company('alice@company.com', 'company.com');
var email3 = Email.test();

print(email1); // user@example.com
print(email2); // alice@company.com
print(email3); // test@example.com

// Throws errors
// var invalid = Email('invalid'); // Error: Invalid email
// var wrongDomain = Email.company('user@gmail.com', 'company.com'); // Error
```

---

# Factory with Dependency Injection

## Flexible Object Creation

```dart
abstract class Service {
  void execute();
}

class RealService implements Service {
  @override
  void execute() {
    print('Real service executing...');
  }
}

class MockService implements Service {
  @override
  void execute() {
    print('Mock service executing...');
  }
}

class ServiceProvider {
  static bool _useMock = false;
  
  // Set mode
  static void useMock(bool use) {
    _useMock = use;
  }
  
  // Factory constructor
  factory ServiceProvider.create() {
    if (_useMock) {
      return MockService() as ServiceProvider;
    } else {
      return RealService() as ServiceProvider;
    }
  }
}

// Usage
ServiceProvider.useMock(true);
var mock = ServiceProvider.create();
mock.execute(); // Mock service executing...

ServiceProvider.useMock(false);
var real = ServiceProvider.create();
real.execute(); // Real service executing...
```

---

# Best Practices

## Use Factory for Complex Creation

```dart
// Good: Factory for complex creation
class Configuration {
  final Map<String, dynamic> _config;
  
  Configuration._(this._config);
  
  factory Configuration.fromFile(String path) {
    var content = File(path).readAsStringSync();
    var map = jsonDecode(content);
    return Configuration._(map);
  }
  
  factory Configuration.fromEnv() {
    var map = {
      'host': Platform.environment['HOST'] ?? 'localhost',
      'port': int.parse(Platform.environment['PORT'] ?? '8080'),
    };
    return Configuration._(map);
  }
  
  factory Configuration.defaults() {
    return Configuration._({'host': 'localhost', 'port': 8080});
  }
}
```

---

## Cache Expensive Objects

```dart
// Good: Caching expensive objects
class ImageLoader {
  static final Map<String, Image> _cache = {};
  
  factory ImageLoader.load(String url) {
    return _cache.putIfAbsent(
      url,
      () => ImageLoader._loadFromNetwork(url),
    );
  }
  
  ImageLoader._loadFromNetwork(String url) {
    // Network request
    print('Loading image from: $url');
    // Return image
  }
}
```

---

# Common Mistakes

## Using Factory for Simple Cases

Wrong:

```dart
class User {
  final String name;
  
  // Unnecessary factory
  factory User(String name) => User._(name);
  User._(this.name);
}
```

Correct:

```dart
class User {
  final String name;
  
  // Simple constructor
  User(this.name);
}
```

---

## Returning Null

Wrong:

```dart
factory User.findById(String id) {
  var user = _findUser(id);
  if (user == null) {
    return null; // Factory shouldn't return null
  }
  return User._(user);
}
```

Correct:

```dart
factory User.findById(String id) {
  var user = _findUser(id);
  if (user == null) {
    throw ArgumentError('User not found: $id');
  }
  return User._(user);
}

// Or return a default
factory User.findById(String id) {
  var user = _findUser(id);
  return user != null ? User._(user) : User._default();
}
```

---

# Summary

Factory constructors provide flexible object creation patterns. They enable caching, subtype returning, complex validation, and singleton implementation while maintaining clean APIs.

---

# Next Steps

Now that you understand factory constructors, continue to:

- [Callable Classes](callable-classes.md)
- [Object Lifecycle](object-lifecycle.md)
- [Generic Classes](generic-classes.md)
- [Generic Functions](generic-functions.md)

---

# Did You Know?

- Factory constructors don't use `this`
- They can return subtypes
- Factory constructors can be named
- They're useful for dependency injection
- Factory constructors can implement caching
- They work with private constructors
- Factory constructors can be used for validation