# Assertions

Understand how to use assertions for debugging and validation in Dart.

---

# What is it?

Assertions are a debugging feature that allows you to validate assumptions about your code. They check if a condition is true, and if not, they throw an `AssertionError`. Assertions are only active in debug mode and are disabled in production builds.

---

# Why does it exist?

Assertions exist to:

- Validate assumptions during development
- Catch bugs early
- Document expected conditions
- Provide safety checks in debug mode
- Enable defensive programming
- Help with debugging

---

# Basic Assertions

## Using assert()

> **The `assert()` function** checks a condition and throws an `AssertionError` if the condition is false.

```dart
void main() {
  // Basic assertion
  int age = 25;
  assert(age > 0); // Passes - no output
  
  // Assert with message
  int temperature = -5;
  assert(temperature >= 0, 'Temperature cannot be negative');
  
  // Assert with computation
  var list = [1, 2, 3];
  assert(list.isNotEmpty);
  
  print('Program continues');
}

// Output:
// Uncaught Error: Assertion failed: "Temperature cannot be negative"
// (in debug mode only)
```

> **What's happening here?**
> - `assert()` checks a boolean condition
> - If false, it throws `AssertionError`
> - Optional message provides context
> - Assertions are active only in debug mode
> - In production, assertions are ignored

---

## Assert with Custom Message

> **You can provide** a custom message to help debug assertion failures.

```dart
void processUser(String name, int age, String email) {
  // Validate with messages
  assert(name.isNotEmpty, 'Name cannot be empty');
  assert(age >= 0 && age <= 150, 'Age must be between 0 and 150');
  assert(email.contains('@'), 'Email must contain @');
  assert(email.contains('.'), 'Email must contain .');
  
  print('User processed: $name, $age, $email');
}

void main() {
  try {
    processUser('', -5, 'invalid');
  } catch (e) {
    print('Caught: $e');
  }
}

// Output:
// Caught: Assertion failed: "Name cannot be empty"
```

---

# Assertion Use Cases

## Validating Inputs

> **Assertions are useful** for validating function inputs during development.

```dart
class Calculator {
  int divide(int a, int b) {
    // Assert can check conditions
    assert(b != 0, 'Cannot divide by zero');
    
    // Multiple assertions
    assert(a >= 0, 'Dividend must be positive');
    assert(b > 0, 'Divisor must be positive');
    
    return a ~/ b;
  }
  
  double sqrt(double value) {
    assert(value >= 0, 'Cannot sqrt negative number');
    return math.sqrt(value);
  }
  
  int factorial(int n) {
    assert(n >= 0, 'Cannot factorial negative number');
    assert(n <= 20, 'Value too large for factorial');
    
    if (n <= 1) return 1;
    return n * factorial(n - 1);
  }
}

void main() {
  var calc = Calculator();
  
  try {
    calc.divide(10, 0);
  } catch (e) {
    print('Error: $e');
  }
  
  try {
    calc.factorial(-5);
  } catch (e) {
    print('Error: $e');
  }
}

// Output:
// Error: Assertion failed: Cannot divide by zero
// Error: Assertion failed: Cannot factorial negative number
```

> **Key insights:**
> - Assertions validate function inputs
> - They catch programming errors early
> - Multiple assertions can be used
> - Clear messages help debugging

---

## Documenting Assumptions

> **Assertions document** assumptions and invariants in your code.

```dart
class Date {
  final int year;
  final int month;
  final int day;
  
  Date(this.year, this.month, this.day) {
    // Documenting assumptions
    assert(year > 0, 'Year must be positive');
    assert(month >= 1 && month <= 12, 'Month must be between 1 and 12');
    assert(day >= 1 && day <= 31, 'Day must be between 1 and 31');
  }
  
  bool isLeapYear() {
    // Documenting logic assumptions
    assert(year > 0, 'Year must be valid');
    
    return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
  }
  
  int daysInMonth() {
    assert(month >= 1 && month <= 12, 'Month must be valid');
    
    switch (month) {
      case 1: case 3: case 5: case 7: case 8: case 10: case 12:
        return 31;
      case 4: case 6: case 9: case 11:
        return 30;
      case 2:
        return isLeapYear() ? 29 : 28;
      default:
        // This should never happen
        assert(false, 'Invalid month in daysInMonth');
        return 0;
    }
  }
}
```

---

# Assertions vs Exceptions

## When to Use What

> **Assertions are for bugs**, exceptions are for runtime conditions.

```dart
class BankAccount {
  double _balance = 0;
  
  // Assertion - checks programming error
  void deposit(double amount) {
    assert(amount >= 0, 'Amount must be non-negative');
    _balance += amount;
  }
  
  // Exception - handles runtime condition
  void withdraw(double amount) {
    if (amount <= 0) {
      throw ArgumentError('Amount must be positive');
    }
    if (amount > _balance) {
      throw Exception('Insufficient funds');
    }
    _balance -= amount;
  }
}

void main() {
  var account = BankAccount();
  
  // Assertion failure (bug)
  // assert(account._balance >= 0, 'Balance should never be negative');
  
  // Exception (runtime condition)
  try {
    account.withdraw(100);
  } catch (e) {
    print('Caught: $e');
  }
}
```

> **Key insights:**
> - Assertions find bugs during development
> - Exceptions handle runtime conditions
> - Assertions are removed in production
> - Exceptions remain in production

---

# Assertions in Production

## Why Assertions are Disabled

> **Assertions are disabled** in production to improve performance.

```dart
// In debug mode
void debugExample() {
  assert(1 == 2, 'This assertion fails in debug');
  print('Debug mode: assertion would have failed');
}

// Production behavior
void productionExample() {
  // Assertions are removed
  // Equivalent to:
  // (1 == 2) // No effect
  // print('Production: assertions are ignored');
  print('Production: assertions are ignored');
}

void main() {
  debugExample();
  productionExample();
}

// Debug output:
// Uncaught Error: Assertion failed: This assertion fails in debug
//
// Production output (if assertions disabled):
// Production: assertions are ignored
```

---

# Real-World Example

## Validation Pipeline

> **Using assertions** in a validation pipeline.

```dart
class Order {
  final String id;
  final String customerId;
  final List<OrderItem> items;
  final DateTime createdAt;
  
  Order(this.id, this.customerId, this.items, this.createdAt) {
    // Validate constructor inputs
    assert(id.isNotEmpty, 'Order ID cannot be empty');
    assert(customerId.isNotEmpty, 'Customer ID cannot be empty');
    assert(items.isNotEmpty, 'Order must have at least one item');
    assert(createdAt.isBefore(DateTime.now()), 'Order cannot be in the future');
  }
  
  double calculateTotal() {
    // Assert internal state
    assert(items.isNotEmpty, 'Cannot calculate total for empty order');
    
    var total = 0.0;
    for (var item in items) {
      total += item.price * item.quantity;
    }
    
    // Assertion for sanity check
    assert(total > 0, 'Order total must be positive');
    return total;
  }
}

class OrderItem {
  final String productId;
  final double price;
  final int quantity;
  
  OrderItem(this.productId, this.price, this.quantity) {
    assert(productId.isNotEmpty, 'Product ID cannot be empty');
    assert(price > 0, 'Price must be positive');
    assert(quantity > 0, 'Quantity must be positive');
  }
}

void main() {
  try {
    var order = Order(
      '',
      'customer123',
      [],
      DateTime.now(),
    );
  } catch (e) {
    print('Order creation failed: $e');
  }
}
```

---

# Best Practices

## Use Assertions for Internal State

```dart
// Good: Validating internal state
class Counter {
  int _count = 0;
  
  void increment() {
    assert(_count >= 0, 'Count should never be negative');
    _count++;
  }
  
  void decrement() {
    assert(_count > 0, 'Cannot decrement below zero');
    _count--;
  }
}
```

---

## Use Assertions for Function Assumptions

```dart
// Good: Documenting assumptions
int getFirstElement(List<int> list) {
  assert(list.isNotEmpty, 'List must not be empty');
  return list[0];
}
```

---

# Common Mistakes

## Using Assertions for Runtime Errors

Wrong:
```dart
void process(String data) {
  // Should be exception, not assertion
  assert(data != null, 'Data cannot be null');
  // Null check should be an exception
}
```

Correct:
```dart
void process(String data) {
  if (data.isEmpty) {
    throw ArgumentError('Data cannot be empty');
  }
}
```

---

## Relying on Assertions for Security

Wrong:
```dart
void sensitiveOperation(User user) {
  // Assertions can be disabled in production!
  assert(user.isAdmin, 'User must be admin');
}
```

Correct:
```dart
void sensitiveOperation(User user) {
  if (!user.isAdmin) {
    throw UnauthorizedException('Admin access required');
  }
}
```

---

# Summary

Assertions are powerful debugging tools that validate assumptions during development. Use them for catching bugs, not for handling runtime errors.

---

# Next Steps

Now that you understand assertions, you've completed the Error Handling section! Continue to:

- [Libraries](libraries.md)
- [Imports](imports.md)
- [Exports](exports.md)
- [Core Libraries](core-libraries.md)

---

# Did You Know?

- Assertions are disabled in production
- Use `assert()` with a condition and optional message
- Assertions help catch bugs early
- They're for debugging, not validation
- Assertions can be used anywhere
- `assert()` throws `AssertionError`
- Use exceptions for runtime error handling

---

