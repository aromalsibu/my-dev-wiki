# Extensions

Understand how to add functionality to existing types in Dart.

---

# What is it?

Extensions allow you to add new methods, getters, setters, and operators to existing types without modifying their source code. This powerful feature enables you to extend classes, enums, and even primitive types with custom functionality.

---

# Why does it exist?

Extensions exist to:

- Add functionality to types you don't own
- Keep code organized and modular
- Create more expressive APIs
- Add utility methods to existing classes
- Avoid utility classes (e.g., StringUtils)
- Provide domain-specific extensions

---

# Basic Extensions

## Simple Extension

```dart
// Extension on String
extension StringExtension on String {
  // Add a method
  String reverse() {
    return split('').reversed.join('');
  }
  
  // Add a getter
  bool get isPalindrome {
    return this == reverse();
  }
  
  // Add a setter (rare)
  // Not possible directly with simple extensions
}

// Usage
String text = 'hello';
print(text.reverse()); // 'olleh'
print('radar'.isPalindrome); // true
print('hello'.isPalindrome); // false
```

---

## Extension with Parameters

```dart
extension NumberExtension on int {
  // Method with parameters
  int add(int other) => this + other;
  int multiply(int other) => this * other;
  
  // Multiple parameters
  int calculate(int a, int b) => this * a + b;
  
  // Generic method
  T apply<T>(T Function(int) transform) => transform(this);
}

// Usage
int num = 5;
print(num.add(3)); // 8
print(num.multiply(2)); // 10
print(num.calculate(2, 3)); // 13
print(num.apply((n) => n * n)); // 25
```

---

# Extension Properties

## Getters and Setters

```dart
extension DateTimeExtension on DateTime {
  // Getter
  String get shortDate {
    return '${year}-${month.toString().padLeft(2, '0')}-${day.toString().padLeft(2, '0')}';
  }
  
  String get shortTime {
    return '${hour.toString().padLeft(2, '0')}:${minute.toString().padLeft(2, '0')}';
  }
  
  // Computed property
  String get formatted {
    return '$shortDate $shortTime';
  }
  
  // Boolean getter
  bool get isWeekend {
    return weekday == DateTime.saturday || weekday == DateTime.sunday;
  }
  
  bool get isWorkday => !isWeekend;
}

// Usage
final now = DateTime.now();
print(now.shortDate); // 2026-06-29
print(now.shortTime); // 14:30
print(now.formatted); // 2026-06-29 14:30
print(now.isWeekend); // false
```

---

## Operators in Extensions

```dart
extension DurationExtension on Duration {
  // Operator overload
  Duration operator +(Duration other) {
    return Duration(milliseconds: inMilliseconds + other.inMilliseconds);
  }
  
  Duration operator *(int multiplier) {
    return Duration(milliseconds: inMilliseconds * multiplier);
  }
  
  // Comparison
  bool operator <(Duration other) {
    return inMilliseconds < other.inMilliseconds;
  }
  
  bool operator >(Duration other) {
    return inMilliseconds > other.inMilliseconds;
  }
}

// Usage
Duration d1 = Duration(seconds: 5);
Duration d2 = Duration(seconds: 3);
print(d1 + d2); // 8 seconds
print(d1 * 2); // 10 seconds
print(d1 > d2); // true
```

---

# Generic Extensions

## Extending Generic Types

```dart
// Extension on List<T>
extension ListExtension<T> on List<T> {
  // Get second element safely
  T? get second {
    if (length >= 2) return this[1];
    return null;
  }
  
  // Get last safely
  T? get lastOrNull => isEmpty ? null : last;
  
  // Safe access with default
  T elementAtOrElse(int index, T defaultValue) {
    if (index >= 0 && index < length) {
      return this[index];
    }
    return defaultValue;
  }
  
  // Swap elements
  void swap(int index1, int index2) {
    if (index1 == index2) return;
    final temp = this[index1];
    this[index1] = this[index2];
    this[index2] = temp;
  }
}

// Usage
List<int> numbers = [1, 2, 3, 4, 5];
print(numbers.second); // 2
print(numbers.elementAtOrElse(10, 0)); // 0
numbers.swap(0, 4);
print(numbers); // [5, 2, 3, 4, 1]
```

---

## Extending Map

```dart
extension MapExtension<K, V> on Map<K, V> {
  // Safe get with default
  V getOrElse(K key, V defaultValue) {
    return this[key] ?? defaultValue;
  }
  
  // Get all keys with values
  String toPrettyString() {
    return entries.map((e) => '${e.key}: ${e.value}').join(', ');
  }
  
  // Filter map
  Map<K, V> filter(bool Function(K key, V value) predicate) {
    return Map.fromEntries(
      entries.where((entry) => predicate(entry.key, entry.value))
    );
  }
}

// Usage
Map<String, int> scores = {'Alice': 95, 'Bob': 87};
print(scores.getOrElse('Charlie', 0)); // 0
print(scores.toPrettyString()); // Alice: 95, Bob: 87
print(scores.filter((k, v) => v > 90)); // {Alice: 95}
```

---

# Extension on Nullable Types

## Nullable Extensions

```dart
extension StringNullableExtension on String? {
  // Safe methods for nullable strings
  bool get isNullOrEmpty => this == null || this!.isEmpty;
  bool get isNullOrBlank => this == null || this!.trim().isEmpty;
  
  // Default value
  String orDefault(String defaultValue) => this ?? defaultValue;
  
  // Transform if not null
  String? map(String Function(String) transform) {
    if (this == null) return null;
    return transform(this!);
  }
}

// Usage
String? maybeName = null;
print(maybeName.isNullOrEmpty); // true
print(maybeName.orDefault('Guest')); // Guest

String? name = 'Alice';
print(name.map((s) => s.toUpperCase())); // ALICE
```

---

## Nullable Numeric Extensions

```dart
extension IntNullableExtension on int? {
  bool get isNull => this == null;
  bool get isNotNull => this != null;
  
  // Safe operations
  int orZero() => this ?? 0;
  int orDefault(int defaultValue) => this ?? defaultValue;
  
  // Chain operations
  int? add(int other) {
    if (this == null) return null;
    return this! + other;
  }
  
  int? multiply(int other) {
    if (this == null) return null;
    return this! * other;
  }
}

// Usage
int? maybeNumber = 5;
print(maybeNumber.isNotNull); // true
print(maybeNumber.orZero()); // 5
print(maybeNumber.add(3)); // 8

int? nullNumber = null;
print(nullNumber.orZero()); // 0
print(nullNumber.add(3)); // null
```

---

# Extension on Specific Types

## Type-Specific Extensions

```dart
// Extension on List<int>
extension IntListExtension on List<int> {
  // Stats
  int get sum => reduce((a, b) => a + b);
  int get product => reduce((a, b) => a * b);
  
  // Stats with default
  int? get average => isEmpty ? null : sum ~/ length;
  
  // Filter
  List<int> get even => where((n) => n % 2 == 0).toList();
  List<int> get odd => where((n) => n % 2 != 0).toList();
}

// Extension on List<String>
extension StringListExtension on List<String> {
  // Join with custom formatting
  String get commaSeparated => join(', ');
  String get newlineSeparated => join('\n');
  
  // Transform
  List<String> get upperCase => map((s) => s.toUpperCase()).toList();
  List<String> get lowerCase => map((s) => s.toLowerCase()).toList();
  
  // Filter
  List<String> get nonEmpty => where((s) => s.isNotEmpty).toList();
}

// Usage
List<int> numbers = [1, 2, 3, 4, 5];
print(numbers.sum); // 15
print(numbers.even); // [2, 4]

List<String> names = ['Alice', 'Bob', 'Charlie'];
print(names.commaSeparated); // Alice, Bob, Charlie
print(names.upperCase); // [ALICE, BOB, CHARLIE]
```

---

# Extension with Type Constraints

## Constrained Extensions

```dart
// Extension only for Iterable of num
extension NumericIterableExtension<T extends num> on Iterable<T> {
  // Works only with numeric types
  T get sum {
    if (isEmpty) return 0 as T;
    return reduce((a, b) => (a + b) as T);
  }
  
  T? get average {
    if (isEmpty) return null;
    return sum / length as T;
  }
}

// Extension only for Iterable of Comparable
extension ComparableIterableExtension<T extends Comparable<T>> on Iterable<T> {
  T get max {
    if (isEmpty) throw StateError('Cannot find max of empty iterable');
    return reduce((a, b) => a.compareTo(b) > 0 ? a : b);
  }
  
  T get min {
    if (isEmpty) throw StateError('Cannot find min of empty iterable');
    return reduce((a, b) => a.compareTo(b) < 0 ? a : b);
  }
  
  Iterable<T> sorted([bool ascending = true]) {
    final list = toList();
    list.sort();
    if (!ascending) list = list.reversed.toList();
    return list;
  }
}

// Usage
var numbers = [1, 2, 3, 4, 5];
print(numbers.sum); // 15
print(numbers.average); // 3
print(numbers.max); // 5

var names = ['Alice', 'Bob', 'Charlie'];
print(names.max); // 'Charlie' (alphabetically)
```

---

# Extensions on Classes

## Extending Your Own Classes

```dart
class Person {
  final String name;
  final int age;
  
  Person(this.name, this.age);
}

// Extension on Person
extension PersonExtension on Person {
  String get greeting => 'Hello, I am $name';
  
  bool get isAdult => age >= 18;
  bool get isSenior => age >= 65;
  
  String get ageGroup {
    if (age < 18) return 'Minor';
    if (age < 65) return 'Adult';
    return 'Senior';
  }
  
  Person copyWith({String? name, int? age}) {
    return Person(
      name ?? this.name,
      age ?? this.age,
    );
  }
}

// Usage
var person = Person('Alice', 25);
print(person.greeting); // Hello, I am Alice
print(person.isAdult); // true
print(person.ageGroup); // Adult
var updated = person.copyWith(age: 30);
```

---

# Extension Conflicts

## Handling Conflicts

```dart
// Extension 1
extension StringExtension on String {
  String reverse() => split('').reversed.join('');
  int get lengthDouble => length * 2;
}

// Extension 2
extension StringExtension2 on String {
  String reverse() => 'Reversed: ${split('').reversed.join('')}';
  int get lengthTriple => length * 3;
}

// Usage with conflicts
String text = 'hello';

// Explicit extension usage
print(StringExtension(text).reverse()); // 'olleh'
print(StringExtension2(text).reverse()); // 'Reversed: olleh'

// Unambiguous names are fine
print(text.lengthDouble); // 10
print(text.lengthTriple); // 15

// Ambiguous method needs explicit extension name
```

---

# Best Practices

## Keep Extensions Focused

```dart
// Good: Focused extension
extension DateFormatter on DateTime {
  String get yyyyMMdd => '${year}-${month.toString().padLeft(2, '0')}-${day.toString().padLeft(2, '0')}';
  String get ddMMyyyy => '${day.toString().padLeft(2, '0')}/${month.toString().padLeft(2, '0')}/$year';
}

// Bad: Too many unrelated methods
extension DateTimeUtils on DateTime {
  String get yyyyMMdd => // ...
  int get daysInMonth => // ...
  bool get isWeekend => // ...
  DateTime get nextMonday => // ...
  String get timeZone => // ...
  // Too many responsibilities!
}
```

---

## Use Descriptive Names

```dart
// Good: Descriptive extension name
extension FileExtension on String {
  bool get isImage => ['jpg', 'jpeg', 'png', 'gif', 'webp'].contains(extension);
  String get extension => split('.').last.toLowerCase();
}

// Bad: Vague extension name
extension Str on String {
  bool get isImg => // ...
}
```

---

# Common Mistakes

## Extending Too Broadly

Wrong:

```dart
// Too broad for Object
extension ObjectExtension on Object {
  String toJson() => // ...
}
// Would add to ALL objects
```

Correct:

```dart
// Specific type
extension StringExtension on String {
  String toJson() => '"$this"';
}
// Only for strings
```

---

## Name Collisions

Wrong:

```dart
// Both extensions define the same method
extension A on String {
  String format() => 'A: $this';
}

extension B on String {
  String format() => 'B: $this';
}
// Usage: text.format() // Ambiguous!
```

Correct:

```dart
// Use different names or explicit extension
extension AFormatting on String {
  String formatA() => 'A: $this';
}

extension BFormatting on String {
  String formatB() => 'B: $this';
}
```

---

# Summary

Extensions allow you to add functionality to existing types in a clean and organized way. They're great for adding utility methods, creating domain-specific APIs, and extending types you don't own.

---

# Next Steps

Now that you understand extensions, continue to:

- [Extension Types](extension-types.md)
- [Typedefs](typedefs.md)
- [Classes](classes.md)
- [Mixins](mixins.md)

---

# Did You Know?

- Extensions were introduced in Dart 2.6
- Extensions can be generic and constrained
- Extension names can be used to resolve conflicts
- Extensions can add operators to existing types
- You can extend nullable types
- Extensions work with Dart's type system
- Extensions are resolved at compile time