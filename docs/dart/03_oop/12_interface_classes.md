# Interface Classes

Understand how to use interface classes to define strict contracts in Dart.

---

# What is it?

Interface classes are a feature introduced in Dart 3 that enforce strict interface contracts. A class marked with the `interface` modifier can be implemented (not extended), ensuring that the contract is followed without inheritance.

---

# Why does it exist?

Interface classes exist to:

- Define pure contracts without implementation
- Enforce interface implementation
- Prevent accidental extension
- Enable better abstraction
- Support multiple implementations
- Maintain clean separation

---

# Basic Interface Classes

## Creating an Interface

```dart
// Interface class - can only be implemented, not extended
interface class Logger {
  // No implementation needed
  void log(String message);
  void error(String message);
  void warning(String message);
  void info(String message);
  
  // Can have static methods
  static Logger getConsole() => ConsoleLogger();
  static Logger getFile() => FileLogger();
}

// Correct: Implementing interface
class ConsoleLogger implements Logger {
  @override
  void log(String message) {
    print('LOG: $message');
  }
  
  @override
  void error(String message) {
    print('ERROR: $message');
  }
  
  @override
  void warning(String message) {
    print('WARNING: $message');
  }
  
  @override
  void info(String message) {
    print('INFO: $message');
  }
}

// Another implementation
class FileLogger implements Logger {
  @override
  void log(String message) {
    print('Writing to file: $message');
  }
  
  @override
  void error(String message) {
    print('Writing error to file: $message');
  }
  
  @override
  void warning(String message) {
    print('Writing warning to file: $message');
  }
  
  @override
  void info(String message) {
    print('Writing info to file: $message');
  }
}

// Usage
void processLogger(Logger logger) {
  logger.info('Application started');
  logger.log('Processing data');
  logger.warning('Low memory');
  logger.error('Something went wrong');
}

var consoleLogger = Logger.getConsole();
var fileLogger = Logger.getFile();

processLogger(consoleLogger);
processLogger(fileLogger);
```

---

## Interface with Properties

```dart
interface class Repository {
  // Properties
  String get name;
  bool get isConnected;
  
  // Methods
  Future<void> connect();
  Future<void> disconnect();
  Future<List<Map<String, dynamic>>> findAll();
  Future<Map<String, dynamic>?> findById(String id);
  Future<void> save(Map<String, dynamic> data);
  Future<void> delete(String id);
}

// Implementation
class MySQLRepository implements Repository {
  final String _name = 'MySQL';
  bool _isConnected = false;
  
  @override
  String get name => _name;
  
  @override
  bool get isConnected => _isConnected;
  
  @override
  Future<void> connect() async {
    print('Connecting to MySQL...');
    _isConnected = true;
  }
  
  @override
  Future<void> disconnect() async {
    print('Disconnecting from MySQL...');
    _isConnected = false;
  }
  
  @override
  Future<List<Map<String, dynamic>>> findAll() async {
    print('Finding all from MySQL');
    return [];
  }
  
  @override
  Future<Map<String, dynamic>?> findById(String id) async {
    print('Finding by id $id from MySQL');
    return {'id': id, 'data': 'sample'};
  }
  
  @override
  Future<void> save(Map<String, dynamic> data) async {
    print('Saving to MySQL: $data');
  }
  
  @override
  Future<void> delete(String id) async {
    print('Deleting from MySQL: $id');
  }
}

// Another implementation
class InMemoryRepository implements Repository {
  final Map<String, Map<String, dynamic>> _data = {};
  bool _isConnected = true;
  
  @override
  String get name => 'InMemory';
  
  @override
  bool get isConnected => _isConnected;
  
  @override
  Future<void> connect() async {
    print('Connecting to InMemory...');
    _isConnected = true;
  }
  
  @override
  Future<void> disconnect() async {
    print('Disconnecting from InMemory...');
    _isConnected = false;
  }
  
  @override
  Future<List<Map<String, dynamic>>> findAll() async {
    print('Finding all from InMemory');
    return _data.values.toList();
  }
  
  @override
  Future<Map<String, dynamic>?> findById(String id) async {
    print('Finding by id $id from InMemory');
    return _data[id];
  }
  
  @override
  Future<void> save(Map<String, dynamic> data) async {
    print('Saving to InMemory: $data');
    _data[data['id']] = data;
  }
  
  @override
  Future<void> delete(String id) async {
    print('Deleting from InMemory: $id');
    _data.remove(id);
  }
}

// Usage
void processRepository(Repository repo) async {
  print('Repository: ${repo.name}');
  print('Connected: ${repo.isConnected}');
  
  await repo.connect();
  await repo.save({'id': '1', 'name': 'Alice'});
  var items = await repo.findAll();
  print('Items: $items');
  await repo.disconnect();
}
```

---

# Interface Classes with Default Implementation

## Providing Defaults

```dart
interface class Validator {
  // Interface with default implementation
  bool isValid(String value) {
    return value.isNotEmpty;
  }
  
  // Other validation methods
  bool isValidEmail(String email) {
    return email.contains('@') && email.contains('.');
  }
  
  bool isValidPhone(String phone) {
    return RegExp(r'^\d{10}$').hasMatch(phone);
  }
  
  bool isValidPassword(String password) {
    return password.length >= 8;
  }
}

// Implementation using defaults
class UserValidator implements Validator {
  @override
  bool isValid(String value) {
    // Override default behavior
    return value.isNotEmpty && value.length >= 3;
  }
  
  // Can use default implementations for other methods
  // Or override them
  @override
  bool isValidPassword(String password) {
    // Custom password validation
    return password.length >= 10 && 
           RegExp(r'[A-Z]').hasMatch(password) &&
           RegExp(r'\d').hasMatch(password);
  }
}

// Usage
var validator = UserValidator();
print(validator.isValid('Al')); // false (too short)
print(validator.isValid('Alice')); // true
print(validator.isValidEmail('test@example.com')); // true (from interface)
print(validator.isValidPhone('1234567890')); // true (from interface)
print(validator.isValidPassword('Password123')); // true (custom)
```

---

# Interface Classes vs Abstract Classes

## Comparison

```dart
// Interface (pure contract)
interface class Drawable {
  void draw();
  void erase();
  void resize(double factor);
  
  // Can have default implementation
  String get description => 'Drawable object';
}

// Abstract class (with implementation)
abstract class Shape {
  // Abstract method
  double get area;
  
  // Concrete method
  void printArea() {
    print('Area: $area');
  }
  
  // Abstract method
  void draw();
}

// Implementing interface and extending abstract class
class Circle implements Drawable, Shape {
  double radius;
  
  Circle(this.radius);
  
  // From Drawable
  @override
  void draw() {
    print('Drawing circle');
  }
  
  @override
  void erase() {
    print('Erasing circle');
  }
  
  @override
  void resize(double factor) {
    radius *= factor;
    print('Resized to $radius');
  }
  
  // From Shape
  @override
  double get area => math.pi * radius * radius;
  
  @override
  String get description => 'Circle with radius $radius';
}

// Usage
var circle = Circle(5);
circle.draw(); // Drawing circle
circle.printArea(); // Area: 78.53981633974483
circle.resize(2); // Resized to 10.0
circle.erase(); // Erasing circle
```

---

# Best Practices

## Use for API Contracts

```dart
// Good: Interface for API contract
interface class PaymentGateway {
  Future<bool> processPayment(double amount, String currency);
  Future<bool> refund(String transactionId);
  String get gatewayName;
  bool get isAvailable;
}

class StripeGateway implements PaymentGateway {
  @override
  String get gatewayName => 'Stripe';
  
  @override
  bool get isAvailable => true;
  
  @override
  Future<bool> processPayment(double amount, String currency) async {
    print('Processing Stripe payment: $amount $currency');
    return true;
  }
  
  @override
  Future<bool> refund(String transactionId) async {
    print('Refunding Stripe transaction: $transactionId');
    return true;
  }
}

class PayPalGateway implements PaymentGateway {
  @override
  String get gatewayName => 'PayPal';
  
  @override
  bool get isAvailable => true;
  
  @override
  Future<bool> processPayment(double amount, String currency) async {
    print('Processing PayPal payment: $amount $currency');
    return true;
  }
  
  @override
  Future<bool> refund(String transactionId) async {
    print('Refunding PayPal transaction: $transactionId');
    return true;
  }
}
```

---

## Keep Interfaces Focused

```dart
// Good: Focused interfaces
interface class Logger {
  void log(String message);
  void error(String message);
  void warning(String message);
}

interface class Metrics {
  void track(String metric, double value);
  void increment(String counter);
}

// Bad: Monolithic interface
interface class Everything {
  void log(String message);
  void track(String metric, double value);
  void save(String data);
  void validate(String input);
  // Too many responsibilities
}
```

---

# Common Mistakes

## Extending Interface Class

Wrong:

```dart
interface class Logger {
  void log(String message);
}

class CustomLogger extends Logger {
  // Error: Cannot extend interface class
}
```

Correct:

```dart
interface class Logger {
  void log(String message);
}

class CustomLogger implements Logger {
  @override
  void log(String message) {
    // Implementation
  }
}
```

---

## Missing Implementation

Wrong:

```dart
interface class Validator {
  bool isValid(String value);
}

class UserValidator implements Validator {
  // Missing isValid implementation
}
// Error: Missing implementation
```

Correct:

```dart
interface class Validator {
  bool isValid(String value);
}

class UserValidator implements Validator {
  @override
  bool isValid(String value) {
    return value.isNotEmpty;
  }
}
```

---

# Summary

Interface classes provide pure contracts that enforce implementation without inheritance. They're ideal for defining APIs, creating service contracts, and enabling polymorphism through implementation.

---

# Next Steps

Now that you understand interface classes, continue to:

- [Final Classes](final-classes.md)
- [Factory Constructors](factory-constructors.md)
- [Callable Classes](callable-classes.md)
- [Object Lifecycle](object-lifecycle.md)

---

# Did You Know?

- Interface classes were introduced in Dart 3
- They can only be implemented, not extended
- Interface classes can have default implementations
- They support multiple interface implementation
- Interface classes are pure contracts
- They're useful for dependency injection
- Interface classes enable better testing through mocks