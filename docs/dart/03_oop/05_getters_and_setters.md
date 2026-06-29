# Getters & Setters

Understand how to use getters and setters to control access to class fields in Dart.

---

# What is it?

Getters and setters are special methods that provide controlled access to class fields. Getters allow you to retrieve values (possibly computed), and setters allow you to modify values with validation or side effects.

---

# Why does it exist?

Getters and setters exist to:

- Encapsulate field access
- Add validation when setting values
- Compute values on demand
- Provide read-only or write-only access
- Hide implementation details
- Maintain class invariants

---

# Basic Getters

## Simple Getters

```dart
class Person {
  String _name;
  int _age;
  
  Person(this._name, this._age);
  
  // Basic getter
  String get name => _name;
  
  // Getter with logic
  String get greeting => 'Hello, I am $_name';
  
  // Boolean getter
  bool get isAdult => _age >= 18;
  
  // Computed getter
  String get displayName => '$_name ($_age)';
}

// Usage
var person = Person('Alice', 25);
print(person.name); // Alice
print(person.greeting); // Hello, I am Alice
print(person.isAdult); // true
print(person.displayName); // Alice (25)
```

---

## Computed Getters

```dart
class Rectangle {
  double _width;
  double _height;
  
  Rectangle(this._width, this._height);
  
  // Computed getters (no backing field)
  double get area => _width * _height;
  double get perimeter => 2 * (_width + _height);
  double get diagonal => sqrt(_width * _width + _height * _height);
  
  // Boolean getter
  bool get isSquare => _width == _height;
  
  // String getter
  String get description => 'Rectangle(${_width}x${_height})';
  
  // Getter with calculation
  String get dimensions => '${_width.toStringAsFixed(1)} x ${_height.toStringAsFixed(1)}';
}

// Usage
var rect = Rectangle(10, 20);
print(rect.area); // 200
print(rect.perimeter); // 60
print(rect.isSquare); // false
print(rect.description); // Rectangle(10x20)
```

---

# Basic Setters

## Simple Setters

```dart
class Temperature {
  double _celsius;
  
  Temperature(this._celsius);
  
  // Getter
  double get celsius => _celsius;
  double get fahrenheit => (_celsius * 9 / 5) + 32;
  
  // Setter with validation
  set celsius(double value) {
    if (value < -273.15) {
      throw ArgumentError('Temperature cannot be below absolute zero');
    }
    _celsius = value;
  }
  
  // Setter with calculation
  set fahrenheit(double value) {
    _celsius = (value - 32) * 5 / 9;
  }
}

// Usage
var temp = Temperature(25);
print(temp.celsius); // 25
print(temp.fahrenheit); // 77

temp.celsius = 30;
print(temp.fahrenheit); // 86

temp.fahrenheit = 100;
print(temp.celsius); // 37.777...

// Throws error
// temp.celsius = -300; // Error: Below absolute zero
```

---

## Setters with Validation

```dart
class User {
  String _email;
  String _password;
  int _age;
  
  User(this._email, this._password, this._age);
  
  // Getter for email (read-only)
  String get email => _email;
  
  // Setter with validation
  set email(String value) {
    if (!value.contains('@')) {
      throw ArgumentError('Invalid email format');
    }
    _email = value;
  }
  
  // Password setter with strength check
  set password(String value) {
    if (value.length < 8) {
      throw ArgumentError('Password must be at least 8 characters');
    }
    if (!RegExp(r'[A-Z]').hasMatch(value)) {
      throw ArgumentError('Password must contain uppercase letter');
    }
    if (!RegExp(r'[a-z]').hasMatch(value)) {
      throw ArgumentError('Password must contain lowercase letter');
    }
    if (!RegExp(r'\d').hasMatch(value)) {
      throw ArgumentError('Password must contain a number');
    }
    _password = value;
  }
  
  // Age setter with range validation
  set age(int value) {
    if (value < 0 || value > 150) {
      throw ArgumentError('Invalid age range');
    }
    _age = value;
  }
  
  // Read-only getters
  String get password => '******'; // Never return actual password
  int get age => _age;
  
  bool get isAdult => _age >= 18;
}

// Usage
var user = User('user@example.com', 'Password123', 25);
user.email = 'new@example.com';
print(user.email); // new@example.com

// Throws errors
// user.password = 'short'; // Error: Too short
// user.age = -5; // Error: Invalid age
```

---

# Advanced Getters

## Getters with Logic

```dart
class BankAccount {
  String _accountNumber;
  double _balance;
  List<Transaction> _transactions = [];
  
  BankAccount(this._accountNumber, this._balance);
  
  // Getter with calculation
  double get balance {
    // Balance is just the current value
    return _balance;
  }
  
  // Getter with aggregation
  double get totalDeposits {
    return _transactions
        .where((t) => t.type == TransactionType.deposit)
        .fold(0, (sum, t) => sum + t.amount);
  }
  
  double get totalWithdrawals {
    return _transactions
        .where((t) => t.type == TransactionType.withdraw)
        .fold(0, (sum, t) => sum + t.amount);
  }
  
  // Getter with condition
  bool get hasTransactions => _transactions.isNotEmpty;
  
  // Getter returning list
  List<Transaction> get recentTransactions {
    return _transactions.take(10).toList();
  }
  
  // Getter with JSON format
  Map<String, dynamic> get toJson => {
    'accountNumber': _accountNumber,
    'balance': _balance,
    'transactionCount': _transactions.length,
  };
}

enum TransactionType { deposit, withdraw }

class Transaction {
  final TransactionType type;
  final double amount;
  final DateTime date;
  
  Transaction(this.type, this.amount, this.date);
}

// Usage
var account = BankAccount('123456', 1000);
print(account.balance); // 1000
print(account.hasTransactions); // false
```

---

# Read-only and Write-only

## Controlled Access

```dart
class Configuration {
  String _apiKey;
  bool _debugMode;
  static const int MAX_RETRIES = 3;
  
  Configuration(this._apiKey, this._debugMode);
  
  // Read-only getter
  String get apiKey => _apiKey;
  bool get debugMode => _debugMode;
  int get maxRetries => MAX_RETRIES;
  
  // Write-only setter (no getter)
  set apiKey(String value) {
    if (value.isEmpty) {
      throw ArgumentError('API key cannot be empty');
    }
    _apiKey = value;
  }
  
  // Getter with transformation
  String get maskedApiKey {
    if (_apiKey.length <= 4) return '****';
    return '${_apiKey.substring(0, 4)}...';
  }
  
  // Setter with side effects
  set debugMode(bool value) {
    _debugMode = value;
    if (value) {
      print('Debug mode enabled');
    }
  }
}

// Usage
var config = Configuration('abcdef123456', false);
print(config.maskedApiKey); // abcd...
print(config.apiKey); // abcdef123456 (read-only)

config.apiKey = 'newkey789'; // Write-only
// print(config.apiKey); // Still read-only, shows original
```

---

# Getters and Setters vs Public Fields

## Comparison

```dart
// Using public fields (less control)
class PublicUser {
  String name;
  int age;
  
  PublicUser(this.name, this.age);
}

// Using getters and setters (more control)
class PrivateUser {
  String _name;
  int _age;
  
  PrivateUser(this._name, this._age);
  
  // Getters
  String get name => _name;
  int get age => _age;
  
  // Setters with validation
  set name(String value) {
    if (value.isEmpty) {
      throw ArgumentError('Name cannot be empty');
    }
    _name = value;
  }
  
  set age(int value) {
    if (value < 0 || value > 150) {
      throw ArgumentError('Invalid age');
    }
    _age = value;
  }
}

// Usage
var pub = PublicUser('Alice', 25);
pub.name = ''; // No validation!
pub.age = -5; // No validation!

var priv = PrivateUser('Alice', 25);
// priv.name = ''; // Throws error
// priv.age = -5; // Throws error
```

---

# Best Practices

## Use Getters for Computed Values

```dart
// Good: Getters for computed values
class Circle {
  double radius;
  
  Circle(this.radius);
  
  double get area => pi * radius * radius;
  double get circumference => 2 * pi * radius;
  double get diameter => 2 * radius;
}

// Bad: Methods for simple computations
class BadCircle {
  double radius;
  
  BadCircle(this.radius);
  
  double calculateArea() => pi * radius * radius;
  double calculateCircumference() => 2 * pi * radius;
}
```

---

## Use Setters for Validation

```dart
// Good: Validation in setters
class Product {
  String _name;
  double _price;
  
  Product(this._name, this._price);
  
  set name(String value) {
    if (value.isEmpty) {
      throw ArgumentError('Product name cannot be empty');
    }
    _name = value;
  }
  
  set price(double value) {
    if (value < 0) {
      throw ArgumentError('Price cannot be negative');
    }
    _price = value;
  }
}

// Bad: No validation
class BadProduct {
  String name;
  double price;
  
  BadProduct(this.name, this.price);
  // Anyone can set invalid values
}
```

---

## Use Getters for Read-only Access

```dart
// Good: Read-only through getter
class Singleton {
  static final Singleton _instance = Singleton._internal();
  int _counter = 0;
  
  factory Singleton() => _instance;
  Singleton._internal();
  
  int get counter => _counter;
  void increment() => _counter++;
}

// Bad: Public mutable field
class BadSingleton {
  static final BadSingleton _instance = BadSingleton._internal();
  int counter = 0;
  
  factory BadSingleton() => _instance;
  BadSingleton._internal();
  // Anyone can modify counter directly
}
```

---

# Common Mistakes

## Not Using Private Fields

Wrong:

```dart
class Person {
  String name; // Public
  int age; // Public
  
  set name(String value) { // Setter exists but field is also public
    this.name = value; // Can still be accessed directly
  }
}
```

Correct:

```dart
class Person {
  String _name;
  int _age;
  
  Person(this._name, this._age);
  
  String get name => _name;
  set name(String value) => _name = value;
}
```

---

## Exposing Internal State

Wrong:

```dart
class Configuration {
  List<String> _settings = [];
  
  List<String> get settings => _settings; // Exposes internal list
  // Caller can mutate the list!
}
```

Correct:

```dart
class Configuration {
  List<String> _settings = [];
  
  List<String> get settings => List.unmodifiable(_settings);
  // Returns immutable copy
}
```

---

# Summary

Getters and setters provide controlled access to class fields, enabling validation, computed properties, and encapsulation. They're essential for maintaining class integrity and hiding implementation details.

---

# Next Steps

Now that you understand getters and setters, continue to:

- [Inheritance](inheritance.md)
- [Mixins](mixins.md)
- [Abstract Classes](abstract-classes.md)
- [Interfaces](interfaces.md)

---

# Did You Know?

- Getters and setters don't create new fields
- Computed getters have no backing field
- Setters can have validation logic
- You can have a setter without a getter
- Getters can return immutable copies
- Dart's getters are called like properties
- Setters use the assignment syntax (`obj.value = x`)