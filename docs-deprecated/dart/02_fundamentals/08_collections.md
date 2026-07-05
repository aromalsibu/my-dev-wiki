# Collections

Understand Dart's collection types and how to work with them effectively.

---

# What is it?

Collections are data structures that store groups of objects. Dart provides three main collection types: List (ordered collection), Set (unique items), and Map (key-value pairs). Each collection type has specific characteristics and use cases.

---

# Why does it exist?

Collections exist to:

- Store and organize groups of related data
- Provide efficient access to elements
- Enable bulk operations on data
- Manage data relationships (key-value pairs)
- Support functional programming patterns
- Handle dynamic data sets

---

# Lists

## Creating Lists

```dart
// Empty list
List<String> emptyList = [];
var emptyList2 = <String>[];

// List with values
List<String> names = ['Alice', 'Bob', 'Charlie'];
var numbers = [1, 2, 3, 4, 5];

// List with type inference
var fruits = ['Apple', 'Banana', 'Orange']; // List<String>

// Fixed-length list
var fixedList = List<int>.filled(3, 0);
fixedList[0] = 1;
fixedList[1] = 2;
fixedList[2] = 3;
// fixedList.add(4); // Error: Cannot add to fixed-length list

// List with generated values
var generated = List.generate(5, (index) => index * 2);
// [0, 2, 4, 6, 8]

// List from constructor
var listFrom = List.from([1, 2, 3]); // Creates a new list
```

---

## List Properties

```dart
List<int> numbers = [1, 2, 3, 4, 5];

// Length
int length = numbers.length; // 5

// Check if empty
bool isEmpty = numbers.isEmpty; // false
bool isNotEmpty = numbers.isNotEmpty; // true

// First and last
int first = numbers.first; // 1
int last = numbers.last; // 5

// Single (only use if list has exactly one element)
// int single = numbers.single; // Error: More than one element

// Access by index
int second = numbers[1]; // 2
```

---

## List Operations

```dart
List<int> numbers = [1, 2, 3];

// Add elements
numbers.add(4); // [1, 2, 3, 4]
numbers.addAll([5, 6]); // [1, 2, 3, 4, 5, 6]

// Insert at position
numbers.insert(2, 99); // [1, 2, 99, 3, 4, 5, 6]
numbers.insertAll(3, [7, 8]); // Insert multiple

// Remove elements
numbers.remove(99); // Removes first occurrence
numbers.removeAt(0); // Remove by index
numbers.removeLast(); // Remove last element
numbers.removeRange(1, 3); // Remove indices 1 and 2

// Replace elements
numbers[0] = 100; // [100, ...]
numbers.replaceRange(1, 3, [10, 20]); // Replace range

// Clear all
numbers.clear(); // []
```

---

## List Iteration

```dart
List<String> names = ['Alice', 'Bob', 'Charlie'];

// For loop
for (var i = 0; i < names.length; i++) {
  print(names[i]);
}

// For-in loop
for (var name in names) {
  print(name);
}

// ForEach
names.forEach((name) => print(name));

// While loop
var index = 0;
while (index < names.length) {
  print(names[index]);
  index++;
}

// Map (transforms list)
var upperNames = names.map((name) => name.toUpperCase()).toList();

// Where (filters list)
var filtered = names.where((name) => name.startsWith('A')).toList();
```

---

## List Transformations

```dart
List<int> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Map - Transform each element
var doubled = numbers.map((n) => n * 2).toList(); // [2, 4, 6, 8, 10]

// Where - Filter elements
var evens = numbers.where((n) => n % 2 == 0).toList(); // [2, 4, 6, 8, 10]

// Reduce - Combine elements
var sum = numbers.reduce((a, b) => a + b); // 55

// Fold - Like reduce with initial value
var sumWithInitial = numbers.fold(10, (a, b) => a + b); // 65

// Skip and Take
var skipFirst3 = numbers.skip(3).toList(); // [4, 5, 6, 7, 8, 9, 10]
var takeFirst5 = numbers.take(5).toList(); // [1, 2, 3, 4, 5]

// Contains
bool hasThree = numbers.contains(3); // true

// Index of
int indexOfFive = numbers.indexOf(5); // 4

// Sublist
var sublist = numbers.sublist(2, 5); // [3, 4, 5]

// Sort
var unsorted = [3, 1, 4, 1, 5];
unsorted.sort(); // [1, 1, 3, 4, 5]
unsorted.sort((a, b) => b.compareTo(a)); // Descending [5, 4, 3, 1, 1]

// Reversed
var reversed = numbers.reversed.toList(); // [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

---

# Sets

## Creating Sets

```dart
// Empty set
Set<String> emptySet = {};
var emptySet2 = <String>{};

// Set with values
Set<String> fruits = {'Apple', 'Banana', 'Orange'};
var numbers = {1, 2, 3, 4, 5};

// Set from list (removes duplicates)
var fromList = Set.from([1, 2, 2, 3, 3, 4]);
// {1, 2, 3, 4}

// Set with type inference
var colors = {'Red', 'Green', 'Blue'}; // Set<String>
```

---

## Set Operations

```dart
Set<int> numbers = {1, 2, 3};

// Add elements
numbers.add(4);
numbers.addAll([5, 6]);

// Remove elements
numbers.remove(3);
numbers.removeAll([4, 5]);

// Check contains
bool hasTwo = numbers.contains(2);

// Clear
numbers.clear();

// Set operations
Set<int> set1 = {1, 2, 3, 4};
Set<int> set2 = {3, 4, 5, 6};

// Union (combined)
var union = set1.union(set2); // {1, 2, 3, 4, 5, 6}

// Intersection (common elements)
var intersection = set1.intersection(set2); // {3, 4}

// Difference (in set1 but not set2)
var difference = set1.difference(set2); // {1, 2}

// Symmetric difference (in either but not both)
var symmetricDiff = set1.difference(set2).union(set2.difference(set1));
// {1, 2, 5, 6}
```

---

# Maps

## Creating Maps

```dart
// Empty map
Map<String, int> emptyMap = {};
var emptyMap2 = <String, int>{};

// Map with values
Map<String, int> scores = {
  'Alice': 95,
  'Bob': 87,
  'Charlie': 92,
};

// Map with type inference
var fruits = {
  'Apple': 5,
  'Banana': 3,
  'Orange': 7,
}; // Map<String, int>

// Map from list
var fromList = Map.fromIterable(
  ['Alice', 'Bob', 'Charlie'],
  key: (item) => item,
  value: (item) => item.length,
);
// {'Alice': 5, 'Bob': 3, 'Charlie': 7}

// Map with generated keys/values
var generated = Map.fromIterables(
  ['a', 'b', 'c'],
  [1, 2, 3],
);
// {'a': 1, 'b': 2, 'c': 3}
```

---

## Map Operations

```dart
Map<String, int> scores = {'Alice': 95, 'Bob': 87};

// Add/Update entries
scores['Charlie'] = 92; // Add
scores['Bob'] = 90; // Update
scores.addAll({'David': 88, 'Eve': 94});

// Remove entries
scores.remove('Bob');
scores.removeWhere((key, value) => value < 90);

// Check contains
bool hasAlice = scores.containsKey('Alice');
bool hasScore95 = scores.containsValue(95);

// Get with default
int aliceScore = scores['Alice'] ?? 0;
int aliceScore2 = scores.putIfAbsent('Alice', () => 95);

// Clear
scores.clear();

// Update all values
scores.updateAll((key, value) => value + 5);
```

---

## Map Iteration

```dart
Map<String, int> scores = {'Alice': 95, 'Bob': 87, 'Charlie': 92};

// Keys and values
var keys = scores.keys; // Iterable<String>
var values = scores.values; // Iterable<int>

// For-each
scores.forEach((key, value) {
  print('$key: $value');
});

// For-in on entries
for (var entry in scores.entries) {
  print('${entry.key}: ${entry.value}');
}

// Map (transform)
var updated = scores.map((key, value) {
  return MapEntry(key.toUpperCase(), value + 10);
});

// Where
var filtered = scores.entries
    .where((entry) => entry.value > 90)
    .toList();
```

---

# Advanced Collection Operations

## Spread Operator

```dart
// List spread
var list1 = [1, 2, 3];
var list2 = [0, ...list1, 4, 5];
// [0, 1, 2, 3, 4, 5]

// Null-aware spread
List<int>? maybeList;
var list3 = [1, 2, ...?maybeList, 3];
// If maybeList is null, expands to nothing

// Spread with set
var set1 = {1, 2, 3};
var set2 = {0, ...set1, 4, 5};
// {0, 1, 2, 3, 4, 5}

// Spread with map
var map1 = {'a': 1, 'b': 2};
var map2 = {'c': 3, ...map1};
// {'c': 3, 'a': 1, 'b': 2}
```

---

## Collection If

```dart
bool includeExtra = true;
String? optionalItem = 'item';

// Conditional inclusion in list
var items = [
  'item1',
  'item2',
  if (includeExtra) 'item3',
  if (optionalItem != null) optionalItem,
];

// Conditional in set
var uniqueItems = {
  'item1',
  'item2',
  if (includeExtra) 'item3',
};

// Conditional in map
var config = {
  'host': 'localhost',
  'port': 8080,
  if (debugMode) 'debug': true,
};
```

---

## Collection For

```dart
// Generate list from another list
var original = [1, 2, 3];
var doubled = [
  for (var n in original) n * 2,
];
// [2, 4, 6]

// Nested for in list
var pairs = [
  for (var i in [1, 2, 3])
    for (var j in [4, 5, 6])
      '$i, $j',
];

// For in set
var set = {1, 2, 3};
var processed = {
  for (var n in set) n * 2,
};
// {2, 4, 6}

// For in map
var scores = {'a': 1, 'b': 2};
var doubledScores = {
  for (var entry in scores.entries)
    entry.key: entry.value * 2,
};
// {'a': 2, 'b': 4}
```

---

# Functional Collection Operations

## Map with Type Conversion

```dart
List<String> strings = ['1', '2', '3'];
var numbers = strings.map(int.parse).toList(); // [1, 2, 3]

List<Map<String, dynamic>> users = [
  {'name': 'Alice', 'age': 25},
  {'name': 'Bob', 'age': 30},
];

var names = users.map((user) => user['name'] as String).toList();
// ['Alice', 'Bob']
```

---

## Filtering with Where

```dart
List<int> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Filter even numbers
var evens = numbers.where((n) => n % 2 == 0).toList();

// Filter with index
var withIndex = numbers.whereIndexed((index, n) => index % 2 == 0);

// Filter and map combined
var result = numbers
    .where((n) => n > 5)
    .map((n) => n * 2)
    .toList();
```

---

## Aggregation

```dart
List<int> numbers = [1, 2, 3, 4, 5];

// Sum
var sum = numbers.reduce((a, b) => a + b); // 15

// Product
var product = numbers.reduce((a, b) => a * b); // 120

// Max
var max = numbers.reduce((a, b) => a > b ? a : b); // 5

// Min
var min = numbers.reduce((a, b) => a < b ? a : b); // 1

// Any
bool anyGreaterThanThree = numbers.any((n) => n > 3); // true

// Every
bool allGreaterThanZero = numbers.every((n) => n > 0); // true

// Fold (with initial value)
var sumWithInit = numbers.fold(10, (a, b) => a + b); // 25
```

---

# Best Practices

## Choose the Right Collection

```dart
// Use List for ordered collections
List<String> queue = ['first', 'second', 'third'];

// Use Set for unique items
Set<String> uniqueNames = {'Alice', 'Bob', 'Charlie'};

// Use Map for key-value pairs
Map<String, int> scores = {'Alice': 95, 'Bob': 87};
```

---

## Prefer Immutable When Possible

```dart
// Use final for collections that shouldn't change
final List<String> names = ['Alice', 'Bob'];
// names = ['Charlie']; // Error: Can't reassign final
// names.add('Charlie'); // Still works! (mutating collection)

// Truly immutable (using const)
const List<String> constantNames = ['Alice', 'Bob'];
// constantNames.add('Charlie'); // Error: Can't mutate const
```

---

## Use Collection Literals

```dart
// Good: Clean and readable
var list = ['a', 'b', 'c'];
var set = {'a', 'b', 'c'};
var map = {'a': 1, 'b': 2};

// Bad: Overly verbose
var list = List<String>.of(['a', 'b', 'c']);
```

---

# Common Mistakes

## Modifying During Iteration

Wrong:

```dart
List<int> numbers = [1, 2, 3, 4, 5];
for (var n in numbers) {
  if (n == 3) {
    numbers.remove(n); // Concurrent modification error
  }
}
```

Correct:

```dart
List<int> numbers = [1, 2, 3, 4, 5];
numbers.removeWhere((n) => n == 3);
// Or use a copy
var filtered = numbers.where((n) => n != 3).toList();
```

---

## Wrong Equality in Set

Wrong:

```dart
class Person {
  final String name;
  Person(this.name);
}
var people = {Person('Alice'), Person('Alice')}; // Set of 2 (different objects)
```

Correct:

```dart
class Person {
  final String name;
  Person(this.name);
  
  @override
  bool operator ==(Object other) => other is Person && name == other.name;
  
  @override
  int get hashCode => name.hashCode;
}
```

---

# Summary

Dart collections provide powerful and flexible ways to store, organize, and manipulate data. Understanding Lists, Sets, Maps, and their operations helps you write efficient and expressive code.

---

# Next Steps

Now that you understand collections, continue to:

- [Records](records.md)
- [Patterns](patterns.md)
- [Classes](classes.md)
- [Constructors](constructors.md)

---

# Did You Know?

- Dart collections are generic and type-safe
- Set and Map use `==` and `hashCode` for equality
- Collections support functional programming patterns
- The spread operator (`...`) works with all collection types
- Collection `if` and `for` are compile-time features
- Dart has specialized collections like `Queue` and `LinkedList`