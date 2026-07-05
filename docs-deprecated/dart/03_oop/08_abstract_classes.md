# Abstract Classes

Understand how to create abstract classes to define contracts and partial implementations in Dart.

---

# What is it?

Abstract classes are classes that cannot be instantiated directly. They serve as blueprints for other classes, defining a contract of methods that subclasses must implement. They can contain both abstract methods (without implementation) and concrete methods (with implementation).

---

# Why does it exist?

Abstract classes exist to:

- Define contracts for subclasses
- Provide partial implementations
- Enable polymorphism
- Enforce implementation of specific methods
- Share common code across subclasses
- Establish clear interfaces

---

# Basic Abstract Classes

## Simple Abstract Class

```dart
// Abstract class (cannot be instantiated)
abstract class Shape {
  // Abstract method (must be implemented)
  double get area;
  double get perimeter;
  
  // Concrete method (optional override)
  String get description => 'Shape with area $area and perimeter $perimeter';
  
  // Abstract method with parameters
  void scale(double factor);
}

// Concrete subclass
class Circle extends Shape {
  double radius;
  
  Circle(this.radius);
  
  @override
  double get area => math.pi * radius * radius;
  
  @override
  double get perimeter => 2 * math.pi * radius;
  
  @override
  void scale(double factor) {
    radius *= factor;
  }
  
  @override
  String get description => 'Circle with radius $radius';
}

// Another concrete subclass
class Rectangle extends Shape {
  double width;
  double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
  
  @override
  void scale(double factor) {
    width *= factor;
    height *= factor;
  }
  
  @override
  String get description => 'Rectangle $width x $height';
}

// Usage
var circle = Circle(5);
var rectangle = Rectangle(10, 20);

print(circle.area); // 78.54
print(circle.description); // Circle with radius 5

print(rectangle.area); // 200
print(rectangle.description); // Rectangle 10 x 20

// Cannot instantiate Shape
// var shape = Shape(); // Error: Abstract class cannot be instantiated
```

---

# Abstract Classes with Implementation

## Partial Implementation

```dart
abstract class Database {
  // Abstract methods (must be implemented)
  Future<void> connect();
  Future<void> disconnect();
  Future<List<Map<String, dynamic>>> query(String sql);
  
  // Concrete method with implementation
  Future<void> executeTransaction(List<String> queries) async {
    try {
      await connect();
      for (var query in queries) {
        await query(query);
      }
      await disconnect();
    } catch (e) {
      print('Transaction failed: $e');
      await disconnect();
    }
  }
  
  // Helper method with default implementation
  String sanitizeInput(String input) {
    return input.replaceAll("'", "\\'");
  }
}

class MySQLDatabase extends Database {
  String connectionString;
  
  MySQLDatabase(this.connectionString);
  
  @override
  Future<void> connect() async {
    print('Connecting to MySQL: $connectionString');
  }
  
  @override
  Future<void> disconnect() async {
    print('Disconnecting from MySQL');
  }
  
  @override
  Future<List<Map<String, dynamic>>> query(String sql) async {
    print('Executing MySQL query: $sql');
    return []; // Mock result
  }
  
  // Override concrete method if needed
  @override
  Future<void> executeTransaction(List<String> queries) async {
    print('Starting MySQL transaction...');
    await super.executeTransaction(queries);
    print('MySQL transaction completed');
  }
}

class PostgreSQLDatabase extends Database {
  String connectionString;
  
  PostgreSQLDatabase(this.connectionString);
  
  @override
  Future<void> connect() async {
    print('Connecting to PostgreSQL: $connectionString');
  }
  
  @override
  Future<void> disconnect() async {
    print('Disconnecting from PostgreSQL');
  }
  
  @override
  Future<List<Map<String, dynamic>>> query(String sql) async {
    print('Executing PostgreSQL query: $sql');
    return []; // Mock result
  }
}

// Usage
var mysql = MySQLDatabase('mysql://localhost:3306');
await mysql.connect();
await mysql.query('SELECT * FROM users');
await mysql.executeTransaction(['INSERT INTO users VALUES (1)']);

var postgres = PostgreSQLDatabase('postgres://localhost:5432');
await postgres.connect();
await postgres.query('SELECT * FROM products');
```

---

# Abstract Methods

## Defining Contracts

```dart
abstract class PaymentProcessor {
  // Abstract method - must be implemented
  Future<bool> processPayment(double amount, String currency);
  
  // Abstract method with parameters
  Future<bool> refundPayment(String transactionId);
  
  // Abstract getter
  String get providerName;
  
  // Abstract setter
  set apiKey(String key);
}

class StripeProcessor extends PaymentProcessor {
  String _apiKey = '';
  String get providerName => 'Stripe';
  
  @override
  set apiKey(String key) {
    _apiKey = key;
    print('Stripe API key set');
  }
  
  @override
  Future<bool> processPayment(double amount, String currency) async {
    print('Processing Stripe payment: $amount $currency');
    // Stripe-specific logic
    return true;
  }
  
  @override
  Future<bool> refundPayment(String transactionId) async {
    print('Refunding Stripe payment: $transactionId');
    // Stripe-specific refund logic
    return true;
  }
}

class PayPalProcessor extends PaymentProcessor {
  String _apiKey = '';
  String get providerName => 'PayPal';
  
  @override
  set apiKey(String key) {
    _apiKey = key;
    print('PayPal API key set');
  }
  
  @override
  Future<bool> processPayment(double amount, String currency) async {
    print('Processing PayPal payment: $amount $currency');
    // PayPal-specific logic
    return true;
  }
  
  @override
  Future<bool> refundPayment(String transactionId) async {
    print('Refunding PayPal payment: $transactionId');
    // PayPal-specific refund logic
    return true;
  }
}

// Usage
Future<void> processPayment(PaymentProcessor processor) async {
  processor.apiKey = 'test_key';
  var result = await processor.processPayment(100, 'USD');
  print('Payment ${result ? "successful" : "failed"}');
}

var stripe = StripeProcessor();
var paypal = PayPalProcessor();

await processPayment(stripe); // Stripe payment
await processPayment(paypal); // PayPal payment
```

---

# Abstract Class with Factory

## Factory Pattern with Abstract Class

```dart
abstract class Logger {
  // Abstract methods
  void log(String message);
  void error(String message);
  void warning(String message);
  void info(String message);
  
  // Factory method
  factory Logger(String type) {
    switch (type.toLowerCase()) {
      case 'console':
        return ConsoleLogger();
      case 'file':
        return FileLogger();
      case 'json':
        return JsonLogger();
      default:
        throw ArgumentError('Unknown logger type: $type');
    }
  }
}

class ConsoleLogger implements Logger {
  @override
  void log(String message) => print('LOG: $message');
  
  @override
  void error(String message) => print('ERROR: $message');
  
  @override
  void warning(String message) => print('WARNING: $message');
  
  @override
  void info(String message) => print('INFO: $message');
}

class FileLogger implements Logger {
  @override
  void log(String message) => print('Writing to file: $message');
  
  @override
  void error(String message) => print('Writing error to file: $message');
  
  @override
  void warning(String message) => print('Writing warning to file: $message');
  
  @override
  void info(String message) => print('Writing info to file: $message');
}

class JsonLogger implements Logger {
  @override
  void log(String message) => print('{"level": "log", "message": "$message"}');
  
  @override
  void error(String message) => print('{"level": "error", "message": "$message"}');
  
  @override
  void warning(String message) => print('{"level": "warning", "message": "$message"}');
  
  @override
  void info(String message) => print('{"level": "info", "message": "$message"}');
}

// Usage
var consoleLogger = Logger('console');
var fileLogger = Logger('file');
var jsonLogger = Logger('json');

consoleLogger.info('Application started');
fileLogger.error('File not found');
jsonLogger.warning('Disk space low');
```

---

# Abstract Class vs Interface

## Comparison

```dart
// Abstract class with implementation
abstract class Animal {
  // Abstract method
  void speak();
  
  // Concrete method (shared implementation)
  void breathe() {
    print('Breathing...');
  }
  
  // Method with default behavior
  void sleep() {
    print('Sleeping...');
  }
}

class Dog extends Animal {
  @override
  void speak() {
    print('Woof!');
  }
  
  // Can override concrete methods
  @override
  void sleep() {
    print('Dog sleeping...');
  }
}

// Interface (implicit via class)
class Flyable {
  void fly() {
    // Interface can have implementation
    print('Flying...');
  }
}

class Bird implements Flyable {
  @override
  void fly() {
    print('Bird flying...');
  }
}

// Using abstract class
var dog = Dog();
dog.speak(); // Woof!
dog.breathe(); // Breathing... (inherited)
dog.sleep(); // Dog sleeping... (overridden)

// Using interface
var bird = Bird();
bird.fly(); // Bird flying...
```

---

# Best Practices

## Use Abstract Classes for Contracts

```dart
// Good: Abstract class for contract
abstract class DataRepository {
  Future<List<T>> fetchAll<T>();
  Future<T> fetchById<T>(String id);
  Future<void> save<T>(T item);
  Future<void> delete(String id);
}

// Implementations
class UserRepository extends DataRepository {
  @override
  Future<List<User>> fetchAll<User>() async {
    // Fetch users
  }
  
  @override
  Future<User> fetchById<User>(String id) async {
    // Fetch user by id
  }
  
  @override
  Future<void> save<User>(User item) async {
    // Save user
  }
  
  @override
  Future<void> delete(String id) async {
    // Delete user
  }
}
```

---

## Provide Default Implementations

```dart
// Good: Default implementations in abstract class
abstract class Cache {
  // Abstract methods (must be implemented)
  Future<void> set(String key, dynamic value);
  Future<dynamic> get(String key);
  Future<void> delete(String key);
  
  // Concrete methods (optional override)
  Future<void> clear() async {
    // Default clear implementation
    print('Clearing cache...');
  }
  
  Future<bool> exists(String key) async {
    return await get(key) != null;
  }
  
  // Helper methods
  String generateKey(String prefix, String id) {
    return '$prefix:$id';
  }
}
```

---

# Common Mistakes

## Instantiating Abstract Class

Wrong:

```dart
abstract class Shape {
  void draw();
}

var shape = Shape(); // Error: Cannot instantiate abstract class
```

Correct:

```dart
class Circle extends Shape {
  @override
  void draw() => print('Drawing circle');
}

var circle = Circle();
circle.draw();
```

---

## Forgetting to Implement Methods

Wrong:

```dart
abstract class Animal {
  void speak();
}

class Dog extends Animal {
  // Missing implementation of speak()
}
// Error: Dog is missing implementation of speak()
```

Correct:

```dart
class Dog extends Animal {
  @override
  void speak() => print('Woof!');
}
```

---

# Summary

Abstract classes provide a powerful way to define contracts and partial implementations. They enable polymorphism, enforce method implementation, and promote code reuse while maintaining flexibility.

---

# Next Steps

Now that you understand abstract classes, continue to:

- [Interfaces](interfaces.md)
- [Sealed Classes](sealed-classes.md)
- [Factory Constructors](factory-constructors.md)
- [Callable Classes](callable-classes.md)

---

# Did You Know?

- Abstract classes cannot be instantiated
- Abstract methods have no implementation
- Concrete methods can have implementation
- Abstract classes can have constructors
- All methods in interfaces are implicitly abstract
- Abstract classes can implement multiple interfaces
- Dart uses implicit interfaces (every class is an interface)