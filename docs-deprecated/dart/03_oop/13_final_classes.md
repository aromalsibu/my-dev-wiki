# Final Classes

Understand how to use final classes to prevent subclassing in Dart.

---

# What is it?

Final classes are a feature introduced in Dart 3 that prevent a class from being extended or implemented. A class marked with the `final` modifier cannot be used as a superclass or interface, ensuring the class's implementation remains unchanged.

---

# Why does it exist?

Final classes exist to:

- Prevent subclassing
- Preserve class behavior
- Ensure type safety
- Control API evolution
- Protect class invariants
- Enable compiler optimizations

---

# Basic Final Classes

## Creating a Final Class

```dart
// Final class - cannot be extended or implemented
final class User {
  final String id;
  final String name;
  final String email;
  
  User({
    required this.id,
    required this.name,
    required this.email,
  });
  
  // Methods
  String get displayName => '$name ($email)';
  
  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'email': email,
  };
  
  // Factory constructor
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
    );
  }
  
  @override
  String toString() => 'User(id: $id, name: $name)';
}

// Cannot extend final class
// class AdminUser extends User { // Error: Cannot extend final class
//   AdminUser(super.id, super.name, super.email);
// }

// Cannot implement final class
// class MockUser implements User { // Error: Cannot implement final class
//   @override
//   String get id => '';
// }

// Usage
var user = User(
  id: '1',
  name: 'Alice',
  email: 'alice@example.com',
);

print(user.displayName); // Alice (alice@example.com)
print(user.toJson()); // {id: 1, name: Alice, email: alice@example.com}
```

---

## Final Class with Private Constructor

```dart
final class Singleton {
  static final Singleton _instance = Singleton._internal();
  
  // Private constructor
  Singleton._internal();
  
  // Factory constructor returns the single instance
  factory Singleton() => _instance;
  
  void doSomething() {
    print('Singleton doing something');
  }
}

// Usage - Only one instance
var s1 = Singleton();
var s2 = Singleton();
print(identical(s1, s2)); // true
s1.doSomething(); // Singleton doing something
```

---

# Final Classes vs Sealed Classes

## Comparison

```dart
// Final class - Cannot be extended
final class FinalClass {
  void doSomething() => print('Final class');
}

// Sealed class - Can be extended in same file
sealed class SealedClass {
  void doSomething();
}

// Can extend sealed class
class SubClass extends SealedClass {
  @override
  void doSomething() {
    print('Subclass');
  }
}

// Can implement sealed class
class AnotherClass implements SealedClass {
  @override
  void doSomething() {
    print('Another implementation');
  }
}

// Cannot extend final class
// class ErrorClass extends FinalClass { // Error!
// }

// Cannot implement final class
// class ErrorImpl implements FinalClass { // Error!
// }
```

---

# Final Class Use Cases

## Value Objects

```dart
// Final class for value objects
final class Money {
  final double amount;
  final String currency;
  
  const Money(this.amount, this.currency);
  
  // Immutable operations
  Money add(Money other) {
    if (currency != other.currency) {
      throw ArgumentError('Cannot add different currencies');
    }
    return Money(amount + other.amount, currency);
  }
  
  Money subtract(Money other) {
    if (currency != other.currency) {
      throw ArgumentError('Cannot subtract different currencies');
    }
    return Money(amount - other.amount, currency);
  }
  
  Money multiply(double factor) {
    return Money(amount * factor, currency);
  }
  
  Money get absolute => Money(amount.abs(), currency);
  
  bool get isPositive => amount > 0;
  bool get isNegative => amount < 0;
  bool get isZero => amount == 0;
  
  @override
  String toString() => '$amount $currency';
  
  @override
  bool operator ==(Object other) =>
      other is Money && amount == other.amount && currency == other.currency;
  
  @override
  int get hashCode => amount.hashCode ^ currency.hashCode;
}

// Usage
var price1 = Money(19.99, 'USD');
var price2 = Money(9.99, 'USD');

var total = price1.add(price2);
var discounted = total.multiply(0.9);

print(total); // 29.98 USD
print(discounted); // 26.982 USD
```

---

## Domain Entities

```dart
final class Product {
  final String id;
  final String name;
  final Money price;
  final int stock;
  
  const Product({
    required this.id,
    required this.name,
    required this.price,
    this.stock = 0,
  });
  
  // Product methods
  bool get isInStock => stock > 0;
  bool get isLowStock => stock > 0 && stock < 5;
  bool get isOutOfStock => stock == 0;
  
  Product updateStock(int newStock) {
    return Product(
      id: id,
      name: name,
      price: price,
      stock: newStock,
    );
  }
  
  Product applyDiscount(double percentage) {
    if (percentage < 0 || percentage > 100) {
      throw ArgumentError('Invalid discount percentage');
    }
    return Product(
      id: id,
      name: name,
      price: price.multiply(1 - percentage / 100),
      stock: stock,
    );
  }
  
  @override
  String toString() => 'Product($name, ${price.toString()}, stock: $stock)';
}

// Usage
var product = Product(
  id: 'P001',
  name: 'Laptop',
  price: Money(999.99, 'USD'),
  stock: 10,
);

print(product.isInStock); // true
print(product.price); // 999.99 USD

var updated = product.updateStock(8);
var discounted = product.applyDiscount(10);

print(updated.stock); // 8
print(discounted.price); // 899.991 USD
```

---

# Final Classes with Static Members

## Utility Classes

```dart
final class MathUtils {
  // Private constructor prevents instantiation
  MathUtils._();
  
  // Static constants
  static const double pi = 3.14159;
  static const double e = 2.71828;
  
  // Static methods
  static double add(double a, double b) => a + b;
  static double subtract(double a, double b) => a - b;
  static double multiply(double a, double b) => a * b;
  static double divide(double a, double b) {
    if (b == 0) throw ArgumentError('Cannot divide by zero');
    return a / b;
  }
  
  static int factorial(int n) {
    if (n < 0) throw ArgumentError('Cannot factorial negative');
    return n <= 1 ? 1 : n * factorial(n - 1);
  }
  
  static List<int> fibonacci(int n) {
    if (n <= 0) return [];
    if (n == 1) return [0];
    if (n == 2) return [0, 1];
    
    final result = [0, 1];
    for (var i = 2; i < n; i++) {
      result.add(result[i - 1] + result[i - 2]);
    }
    return result;
  }
}

// Usage
print(MathUtils.pi); // 3.14159
print(MathUtils.add(5, 3)); // 8
print(MathUtils.factorial(5)); // 120
print(MathUtils.fibonacci(10)); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Cannot instantiate
// var math = MathUtils(); // Error: Private constructor
```

---

# Best Practices

## Use Final for Value Objects

```dart
// Good: Value object as final class
final class Email {
  final String value;
  
  const Email(this.value) {
    if (!value.contains('@') || !value.contains('.')) {
      throw ArgumentError('Invalid email: $value');
    }
  }
  
  String get domain => value.split('@')[1];
  String get username => value.split('@')[0];
  
  @override
  String toString() => value;
  
  @override
  bool operator ==(Object other) =>
      other is Email && value == other.value;
  
  @override
  int get hashCode => value.hashCode;
}
```

---

## Use Final for Security-Sensitive Classes

```dart
// Good: Security-sensitive class
final class Password {
  final String _value;
  
  Password(this._value) {
    if (_value.length < 8) {
      throw ArgumentError('Password must be at least 8 characters');
    }
  }
  
  bool verify(String input) {
    // Secure comparison
    return _value == input;
  }
  
  String get masked => '*' * _value.length;
  
  // Never expose actual password
  @override
  String toString() => 'Password(${masked})';
}
```

---

# Common Mistakes

## Using Final for Everything

Wrong:

```dart
// Unnecessarily final
final class UtilityClass {
  static void doSomething() {}
  // This class doesn't need to be final
}
```

Correct:

```dart
// Use final only when needed
class UtilityClass {
  static void doSomething() {}
  // Can be extended/implemented if useful
}

// Or use final if extension should be prevented
final class SecurityManager {
  // Security-sensitive class
}
```

---

## Forgetting Factory Constructors

Wrong:

```dart
final class Config {
  static final Config _instance = Config._internal();
  Config._internal();
  // Missing factory constructor
}
```

Correct:

```dart
final class Config {
  static final Config _instance = Config._internal();
  Config._internal();
  
  factory Config() => _instance; // Allows instance access
}
```

---

# Summary

Final classes prevent extension and implementation, preserving the integrity of the class. They're ideal for value objects, security-sensitive classes, and ensuring type safety.

---

# Next Steps

Now that you understand final classes, continue to:

- [Factory Constructors](factory-constructors.md)
- [Callable Classes](callable-classes.md)
- [Object Lifecycle](object-lifecycle.md)
- [Generic Classes](generic-classes.md)

---

# Did You Know?

- Final classes were introduced in Dart 3
- They cannot be extended or implemented
- Final classes can have factory constructors
- They're useful for value objects
- Final classes can have static members
- They provide stronger type safety
- Final classes are used in security-sensitive code