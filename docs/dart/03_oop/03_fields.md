# Fields

Understand how to define and use fields (instance variables) in Dart classes.

---

# What is it?

Fields are variables that hold data within a class. They represent the state of an object and can be instance fields (unique to each object), static fields (shared across all instances), or class-level constants.

---

# Why does it exist?

Fields exist to:

- Store object state and data
- Represent the properties of a class
- Enable encapsulation of data
- Provide access to object attributes
- Support data persistence and manipulation

---

# Instance Fields

## Basic Instance Fields

```dart
class Person {
  // Instance fields (each object has its own copy)
  String name;
  int age;
  bool isActive;
  
  // Constructor
  Person(this.name, this.age, this.isActive);
  
  // Another way
  Person.withDefaults(String name) 
      : name = name,
        age = 0,
        isActive = true;
}

// Usage
var person1 = Person('Alice', 25, true);
var person2 = Person('Bob', 30, false);

// Each instance has its own values
print(person1.name); // Alice
print(person2.name); // Bob
```

---

## Final Fields

```dart
class User {
  // Final fields (set once, cannot be changed)
  final String id;
  final String name;
  final DateTime createdAt;
  
  // Optional final field
  final String? email;
  
  User(this.id, this.name, this.email)
      : createdAt = DateTime.now();
  
  // Named constructor
  User.fromEmail(String email, String name)
      : id = email.hashCode.toString(),
        name = name,
        email = email,
        createdAt = DateTime.now();
}

// Usage
var user = User('123', 'Alice', 'alice@example.com');
print(user.id); // 123
print(user.name); // Alice

// Cannot modify final fields
// user.name = 'Bob'; // Error: Cannot assign to final
```

---

## Late Fields

```dart
class DatabaseService {
  // Late fields (initialized later)
  late String _connectionString;
  late String _databaseName;
  
  // Initialized in constructor
  DatabaseService(String connectionString) 
      : _connectionString = connectionString;
  
  // Late initialization with method
  void initialize(String dbName) {
    _databaseName = dbName;
  }
  
  // Late final (assigned once)
  late final String apiKey;
  
  void setApiKey(String key) {
    apiKey = key; // Can only be called once
    // apiKey = 'newkey'; // Error: Already assigned
  }
  
  void connect() {
    // Late fields must be initialized before use
    print('Connecting to $_databaseName with $_connectionString');
  }
}

// Usage
var db = DatabaseService('mysql://localhost');
db.initialize('my_db');
db.setApiKey('secret123');
db.connect(); // Connecting to my_db with mysql://localhost
```

---

# Static Fields

## Class-Level Variables

```dart
class AppConfig {
  // Static fields (shared across all instances)
  static String appName = 'My App';
  static String version = '1.0.0';
  static bool isDebug = false;
  
  // Static final
  static final DateTime startTime = DateTime.now();
  
  // Static constant
  static const int maxRetries = 3;
  static const String defaultLocale = 'en_US';
  
  // Private static field
  static int _instanceCount = 0;
  
  // Static getter
  static int get instanceCount => _instanceCount;
  
  // Static setter
  static set debugMode(bool value) {
    isDebug = value;
  }
  
  AppConfig() {
    _instanceCount++;
  }
}

// Usage (no instance needed)
print(AppConfig.appName); // My App
print(AppConfig.version); // 1.0.0
print(AppConfig.maxRetries); // 3

AppConfig.debugMode = true;
print(AppConfig.isDebug); // true

// Instance count
var config1 = AppConfig();
var config2 = AppConfig();
print(AppConfig.instanceCount); // 2
```

---

# Private Fields

## Encapsulation with Private Fields

```dart
class BankAccount {
  // Private fields (starts with _)
  String _accountNumber;
  double _balance = 0;
  String _accountType;
  
  // Public fields
  final String accountHolder;
  
  BankAccount(this.accountHolder, this._accountNumber, this._accountType);
  
  // Public getters
  String get accountNumber => _accountNumber;
  double get balance => _balance;
  String get accountType => _accountType;
  
  // Public methods to modify private data
  void deposit(double amount) {
    if (amount > 0) {
      _balance += amount;
      _logTransaction('deposit', amount);
    }
  }
  
  bool withdraw(double amount) {
    if (amount > 0 && amount <= _balance) {
      _balance -= amount;
      _logTransaction('withdraw', amount);
      return true;
    }
    return false;
  }
  
  void changeAccountNumber(String newNumber) {
    if (_validateAccountNumber(newNumber)) {
      _accountNumber = newNumber;
    }
  }
  
  // Private method
  void _logTransaction(String type, double amount) {
    print('$type of \$$amount for $_accountNumber');
  }
  
  // Private validation
  bool _validateAccountNumber(String number) {
    return number.length == 10 && RegExp(r'^\d+$').hasMatch(number);
  }
}

// Usage
var account = BankAccount('Alice', '1234567890', 'checking');
account.deposit(1000);
account.withdraw(500);

// Cannot access private fields
// print(account._balance); // Error: Private field
// account._logTransaction(); // Error: Private method

// Can access through getters
print(account.balance); // 500
```

---

# Computed Fields (Getters)

## Calculated Properties

```dart
class Rectangle {
  // Private fields
  double _width;
  double _height;
  
  Rectangle(this._width, this._height);
  
  // Computed getters (no backing field)
  double get area => _width * _height;
  double get perimeter => 2 * (_width + _height);
  double get diagonal => math.sqrt(_width * _width + _height * _height);
  
  // Boolean getter
  bool get isSquare => _width == _height;
  
  // String getter
  String get description => 'Rectangle(${_width}x$_height)';
  
  // Setters with validation
  set width(double value) {
    if (value > 0) {
      _width = value;
    }
  }
  
  set height(double value) {
    if (value > 0) {
      _height = value;
    }
  }
  
  // Computed setter
  set size(double value) {
    _width = value;
    _height = value;
  }
}

// Usage
var rect = Rectangle(10, 20);
print(rect.area); // 200
print(rect.perimeter); // 60
print(rect.isSquare); // false
print(rect.description); // Rectangle(10x20)

rect.size = 15;
print(rect.isSquare); // true
print(rect.description); // Rectangle(15x15)
```

---

# Static vs Instance Fields

## When to Use Each

```dart
class Vehicle {
  // Instance fields (unique to each object)
  final String model;
  final String licensePlate;
  String color;
  
  // Static fields (shared across all instances)
  static int _totalVehicles = 0;
  static const int maxSpeed = 200;
  static String manufacturer = 'Generic Motors';
  
  // Static method to get total
  static int get totalVehicles => _totalVehicles;
  
  Vehicle(this.model, this.licensePlate, this.color) {
    _totalVehicles++;
  }
  
  // Instance method uses both
  String get description => '$model by $manufacturer (${color})';
  
  // Static method utility
  static bool isValidLicensePlate(String plate) {
    return plate.length == 7 && plate.contains('-');
  }
}

// Usage
var car1 = Vehicle('Sedan', 'ABC-123', 'red');
var car2 = Vehicle('SUV', 'XYZ-789', 'blue');

print(car1.model); // Sedan (instance)
print(Vehicle.manufacturer); // Generic Motors (static)
print(Vehicle.maxSpeed); // 200 (static constant)
print(Vehicle.totalVehicles); // 2 (static)
print(Vehicle.isValidLicensePlate('ABC-123')); // true (static)
```

---

# Initialization Order

## Field Initialization Sequence

```dart
class InitializationDemo {
  // Fields initialized in declaration order
  int a = 10;
  int b = 20;
  
  // Can reference previously declared fields
  int c = a + b; // 30
  
  // Static fields initialized first
  static int staticField = 100;
  
  // Late fields
  late int lateField;
  
  // Constructor initializer list
  InitializationDemo(int value)
      : lateField = value,
        a = value * 2 {
    // Constructor body runs last
    print('Constructor body');
  }
  
  // Static initializer
  static int staticComputed = _computeStatic();
  
  static int _computeStatic() {
    print('Static initialization');
    return 42;
  }
}

// Usage
var demo = InitializationDemo(5);
print(demo.a); // 10 (or 10 from initializer)
print(demo.c); // 30
print(demo.lateField); // 5
print(InitializationDemo.staticField); // 100
```

---

# Best Practices

## Use Private Fields for Encapsulation

```dart
// Good: Encapsulated with private fields
class Person {
  String _name;
  int _age;
  
  Person(this._name, this._age);
  
  String get name => _name;
  int get age => _age;
  
  void celebrateBirthday() {
    _age++;
  }
  
  set name(String value) {
    if (value.isNotEmpty) {
      _name = value;
    }
  }
}

// Bad: Exposed fields
class BadPerson {
  String name;
  int age;
  BadPerson(this.name, this.age);
  // Anyone can modify without validation
}
```

---

## Use Final When Possible

```dart
// Good: Immutable fields
class ImmutableUser {
  final String id;
  final String name;
  final DateTime createdAt;
  
  ImmutableUser(this.id, this.name, this.createdAt);
}

// Bad: Mutable when not needed
class MutableUser {
  String id;
  String name;
  DateTime createdAt;
  MutableUser(this.id, this.name, this.createdAt);
}
```

---

## Use Static for Shared Data

```dart
// Good: Shared configuration
class AppSettings {
  static String theme = 'light';
  static String language = 'en';
  static bool notifications = true;
  
  static void setTheme(String theme) {
    AppSettings.theme = theme;
  }
}

// Bad: Duplicated data
class UserSettings {
  String theme = 'light';
  String language = 'en';
}
```

---

# Common Mistakes

## Using Static for Instance Data

Wrong:

```dart
class Person {
  static String name; // Wrong! Name should be instance-specific
  Person(this.name); // Error: Can't use this with static
}
```

Correct:

```dart
class Person {
  String name; // Instance field
  Person(this.name);
}
```

---

## Late Field Not Initialized

Wrong:

```dart
class Service {
  late String _config;
  
  void useService() {
    print(_config); // Error: Late field not initialized
  }
}
```

Correct:

```dart
class Service {
  late String _config;
  
  void initialize(String config) {
    _config = config; // Must be called before use
  }
  
  void useService() {
    print(_config); // Safe if initialized
  }
}
```

---

# Summary

Fields store the state of objects and classes. Understanding the different types (instance, static, final, late, private) and when to use them is crucial for effective object-oriented programming in Dart.

---

# Next Steps

Now that you understand fields, continue to:

- [Methods](methods.md)
- [Getters & Setters](getters-setters.md)
- [Inheritance](inheritance.md)
- [Mixins](mixins.md)

---

# Did You Know?

- Fields are initialized in declaration order
- Final fields must be initialized in constructor
- Late fields are initialized lazily
- Private fields are only accessible within the class
- Static fields are initialized when the class is loaded
- Getters can compute values without backing fields
- Setter parameters can be validated before assignment