# Operators

Understand Dart's comprehensive set of operators and how to use them effectively.

---

# What is it?

Operators are symbols that perform operations on one or more operands (values). Dart provides a rich set of operators for arithmetic, comparison, logical operations, bitwise operations, and more. Many operators can be overridden for custom types.

---

# Why does it exist?

Operators exist to:

- Perform mathematical calculations
- Compare values and make decisions
- Manipulate data at a fundamental level
- Provide concise syntax for common operations
- Enable expressive and readable code
- Work with both primitive types and custom objects

---

# Arithmetic Operators

## Basic Arithmetic

```dart
int a = 10;
int b = 3;

// Addition
int sum = a + b;           // 13

// Subtraction
int difference = a - b;    // 7

// Multiplication
int product = a * b;       // 30

// Division (double result)
double quotient = a / b;   // 3.3333333333333335

// Integer division (truncates)
int integerDivision = a ~/ b; // 3

// Modulo (remainder)
int remainder = a % b;     // 1

// Negation
int negative = -a;         // -10
```

---

## Increment and Decrement

```dart
int count = 0;

// Postfix increment (uses then increments)
print(count++); // 0
print(count);   // 1

// Prefix increment (increments then uses)
print(++count); // 2
print(count);   // 2

// Postfix decrement
print(count--); // 2
print(count);   // 1

// Prefix decrement
print(--count); // 0
print(count);   // 0
```

---

# Comparison Operators

## Equality and Relational

```dart
int a = 5;
int b = 10;

// Equality
bool isEqual = a == b;        // false
bool isNotEqual = a != b;     // true

// Greater/Less than
bool isGreater = a > b;       // false
bool isLess = a < b;          // true
bool isGreaterOrEqual = a >= b; // false
bool isLessOrEqual = a <= b;   // true

// Identity (same object)
var obj1 = Object();
var obj2 = obj1;
bool sameObject = obj1 == obj2; // true (same instance)
```

---

# Logical Operators

## Boolean Logic

```dart
bool a = true;
bool b = false;

// Logical AND (both true)
bool andResult = a && b;     // false

// Logical OR (at least one true)
bool orResult = a || b;      // true

// Logical NOT (negation)
bool notResult = !a;         // false
bool notResult2 = !b;        // true

// Compound conditions
bool complex = (a && b) || !a; // false
```

---

## Short-circuit Evaluation

```dart
// AND short-circuit (stops at first false)
bool result = false && expensiveOperation(); // expensiveOperation not called

// OR short-circuit (stops at first true)
bool result2 = true || expensiveOperation(); // expensiveOperation not called
```

---

# Assignment Operators

## Basic Assignment

```dart
// Simple assignment
var x = 5;

// Compound assignment
int y = 10;
y += 5;  // y = y + 5  (15)
y -= 3;  // y = y - 3  (12)
y *= 2;  // y = y * 2  (24)
y ~/= 3; // y = y ~/ 3 (8)
y %= 3;  // y = y % 3  (2)
y++;     // y = y + 1  (3)

// Null-aware assignment
String? name;
name ??= 'Guest'; // Assign if null
print(name); // 'Guest'
```

---

# Null-aware Operators

## Safe Access and Defaults

```dart
String? maybeName = null;

// Null-aware access (safe navigation)
int? length = maybeName?.length; // null (no error)

// Null coalescing (default value)
String displayName = maybeName ?? 'Guest'; // 'Guest'

// Null coalescing assignment
maybeName ??= 'Alice'; // Assigns 'Alice'

// Null-aware cascade
class Person {
  String? name;
  int? age;
}

Person? person = Person()
  ?..name = 'Alice'
  ..age = 25;
  
// Equivalent to:
// if (person != null) {
//   person.name = 'Alice';
//   person.age = 25;
// }
```

---

# Type Operators

## Type Checking and Casting

```dart
dynamic value = 'Hello';

// Type check (is)
bool isString = value is String;  // true
bool isInt = value is int;        // false

// Type check negation (is!)
bool notString = value is! String; // false

// Type cast (as)
String str = value as String;     // Safe cast

// Type cast with check
if (value is String) {
  // value is now treated as String
  print(value.length);
}
```

---

# Bitwise Operators

## Bit Manipulation

```dart
int a = 0b1010;  // 10 in decimal
int b = 0b1100;  // 12 in decimal

// Bitwise AND
int and = a & b;     // 0b1000 (8)

// Bitwise OR
int or = a | b;      // 0b1110 (14)

// Bitwise XOR (exclusive OR)
int xor = a ^ b;     // 0b0110 (6)

// Bitwise NOT (complement)
int not = ~a;        // ...11110101 (-11)

// Left shift (multiply by 2^)
int leftShift = a << 2; // 0b101000 (40)

// Right shift (divide by 2^)
int rightShift = a >> 1; // 0b0101 (5)
```

---

# Cascade Notation

## Chaining Operations

```dart
class Person {
  String name = '';
  int age = 0;
  
  void sayHello() {
    print('Hello, I am $name');
  }
}

// Without cascade
var person = Person();
person.name = 'Alice';
person.age = 25;
person.sayHello();

// With cascade (..)
var person2 = Person()
  ..name = 'Alice'
  ..age = 25
  ..sayHello();

// Null-aware cascade (?..)
Person? maybePerson;
maybePerson
  ?..name = 'Alice'
  ..age = 25
  ..sayHello();
```

---

# Spread Operators

## Expanding Collections

```dart
// Spread operator (...)
var list1 = [1, 2, 3];
var list2 = [0, ...list1, 4, 5];
// Result: [0, 1, 2, 3, 4, 5]

// Null-aware spread (...?)
List<int>? maybeList;
var list3 = [1, 2, ...?maybeList, 3];
// If maybeList is null, expands to nothing

// Spread with collections
var map1 = {'a': 1, 'b': 2};
var map2 = {'c': 3, ...map1};
// Result: {'c': 3, 'a': 1, 'b': 2}
```

---

# Collection Operators

## if and for in Collections

```dart
// Collection if
bool includeItem = true;
var items = [
  'item1',
  'item2',
  if (includeItem) 'item3', // Included only if true
];

// Collection for
var numbers = [1, 2, 3];
var doubled = [
  for (var n in numbers) n * 2, // [2, 4, 6]
];

// Combined
var list = [
  'a',
  if (condition) 'b',
  for (var i in [1, 2, 3]) i,
];
```

---

# Operator Precedence

## Precedence Table

```dart
// Highest precedence (evaluated first)
// ()  .  ?.  []  !  ~  - (unary)
// *  /  ~/  %
// +  -
// <<  >>
// &  ^  |
// <  <=  >  >=  as  is  is!
// ==  !=
// &&
// ||
// ??
// ?..  ..  ..
// =  ??=  +=  -=  etc.
// Lowest precedence (evaluated last)

// Example
int result = 2 + 3 * 4;     // 14 (multiplication first)
int result2 = (2 + 3) * 4;  // 20 (parentheses override)
```

---

# Overriding Operators

## Custom Operator Implementation

```dart
class Vector {
  final double x, y;
  
  Vector(this.x, this.y);
  
  // Override + operator
  Vector operator +(Vector other) {
    return Vector(x + other.x, y + other.y);
  }
  
  // Override - operator
  Vector operator -(Vector other) {
    return Vector(x - other.x, y - other.y);
  }
  
  // Override * operator (scalar multiplication)
  Vector operator *(double scalar) {
    return Vector(x * scalar, y * scalar);
  }
  
  // Override == operator
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    if (other is! Vector) return false;
    return x == other.x && y == other.y;
  }
  
  @override
  int get hashCode => x.hashCode ^ y.hashCode;
}

// Usage
var v1 = Vector(1, 2);
var v2 = Vector(3, 4);
var v3 = v1 + v2;      // Vector(4, 6)
var v4 = v1 * 2;       // Vector(2, 4)
```

---

## Overridable Operators

```dart
// All operators that can be overridden
class Point {
  final int x, y;
  
  Point(this.x, this.y);
  
  // Unary operators
  Point operator -() => Point(-x, -y);
  
  // Binary operators
  Point operator +(Point other) => Point(x + other.x, y + other.y);
  Point operator -(Point other) => Point(x - other.x, y - other.y);
  Point operator *(int scalar) => Point(x * scalar, y * scalar);
  Point operator /(double scalar) => Point(x / scalar, y / scalar);
  Point operator ~/(int scalar) => Point(x ~/ scalar, y ~/ scalar);
  Point operator %(int scalar) => Point(x % scalar, y % scalar);
  
  // Comparison operators
  bool operator ==(Object other) => 
    other is Point && x == other.x && y == other.y;
  
  int operator <(Point other) => x < other.x && y < other.y;
  
  // Index operators
  int operator [](int index) {
    if (index == 0) return x;
    if (index == 1) return y;
    throw RangeError('Index out of range');
  }
  
  void operator []=(int index, int value) {
    if (index == 0) x = value;
    else if (index == 1) y = value;
    else throw RangeError('Index out of range');
  }
}
```

---

# Best Practices

## Use Operators for Readability

```dart
// Good: Concise and readable
int result = a + b * c;

// Bad: Overly complex one-liners
int result = (a + b) * (c - d) / (e + f) % 2;

// Better: Break complex operations
int sum = a + b;
int difference = c - d;
int product = sum * difference;
int result = product ~/ (e + f);
```

---

## Null Safety with Operators

```dart
// Good: Safe null handling
String? name = getUserName();
String display = name ?? 'Guest';
int? length = name?.length;

// Bad: Unsafe null handling
String display = name!; // Danger!
int length = name.length; // Danger!
```

---

## Operator Overloading Guidelines

```dart
// Good: Intuitive operator overloading
class Money {
  final double amount;
  Money(this.amount);
  
  Money operator +(Money other) => Money(amount + other.amount);
  Money operator -(Money other) => Money(amount - other.amount);
  bool operator <(Money other) => amount < other.amount;
}

// Bad: Counter-intuitive overloading
class Money {
  final double amount;
  Money(this.amount);
  
  Money operator +(Money other) => Money(amount * other.amount); // Confusing!
}
```

---

# Common Mistakes

## Assignment vs Comparison

Wrong:

```dart
if (x = 5) { // Assignment instead of comparison
  // ...
}
```

Correct:

```dart
if (x == 5) { // Comparison
  // ...
}
```

---

## Short-circuit Gotchas

Wrong:

```dart
bool result = condition && functionWithSideEffects();
// functionWithSideEffects might not be called
```

Correct:

```dart
if (condition) {
  functionWithSideEffects();
}
```

---

## Increment/Decrement Confusion

Wrong:

```dart
var x = 5;
var y = x++; // y = 5, x = 6
var z = ++x; // z = 7, x = 7
// Hard to track values
```

Correct:

```dart
var x = 5;
var y = x;
x = x + 1; // Clear and explicit
```

---

# Summary

Dart's operators provide powerful and concise ways to manipulate data. Understanding operator precedence, null-aware operators, and proper usage patterns helps you write cleaner, safer, and more efficient code.

---

# Next Steps

Now that you understand operators, continue to:

- [Control Flow](control-flow.md)
- [Functions](functions.md)
- [Collections](collections.md)
- [Records](records.md)

---

# Did You Know?

- You can override most operators in Dart
- The null-aware operators (?. and ??) are unique to Dart
- Dart uses short-circuit evaluation for && and ||
- The cascade operator (..) is unique to Dart
- Operator precedence follows standard mathematical rules
- Some operators cannot be overridden (like && and ||)