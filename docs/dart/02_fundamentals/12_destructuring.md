# Destructuring

Understand how to extract data from complex structures in Dart.

---

# What is it?

Destructuring is the process of breaking down complex data structures (like records, lists, maps, and objects) into individual variables. It allows you to extract multiple values from a data structure in a single, concise statement.

---

# Why does it exist?

Destructuring exists to:

- Extract multiple values in one go
- Make code more concise and readable
- Simplify working with complex data
- Reduce boilerplate access code
- Enable pattern matching
- Improve code clarity

---

# Destructuring Records

## Positional Records

```dart
// Basic record destructuring
var (x, y) = (10, 20);
print('x: $x, y: $y'); // x: 10, y: 20

// With type annotations
var (String name, int age) = ('Alice', 25);
print('$name is $age years old');

// Partial destructuring (ignore some fields)
var (_, age) = ('Alice', 25);
print('Age: $age');

// Destructuring into existing variables
var existingX = 0;
var existingY = 0;
(existingX, existingY) = (5, 10);
print('$existingX, $existingY'); // 5, 10
```

---

## Named Records

```dart
// Named record destructuring
var (:name, :age) = (name: 'Alice', age: 25);
print('Name: $name, Age: $age');

// With different variable names
var (:name as userName, :age as userAge) = (name: 'Alice', age: 25);
print('$userName is $userAge');

// Partial destructuring
var (:name, :age, :) = (name: 'Alice', age: 25, city: 'NYC');
print('Name: $name, Age: $age');

// Destructuring with types
var ({String name, int age}) = (name: 'Alice', age: 25);
```

---

## Mixed Records

```dart
// Mixed positional and named
var (id, :name, :age) = (1, name: 'Alice', age: 25);
print('ID: $id, Name: $name, Age: $age');

// Destructuring mixed records
var (int id, {String name, int age}) = (1, name: 'Alice', age: 25);
print('$id: $name is $age');
```

---

# Destructuring Lists

## Basic List Destructuring

```dart
// Simple list destructuring
var [a, b, c] = [1, 2, 3];
print('$a, $b, $c'); // 1, 2, 3

// With type annotations
var [String first, String second] = ['Alice', 'Bob'];
print('$first and $second');

// Ignoring elements
var [first, _, third] = [1, 2, 3];
print('$first, $third'); // 1, 3

// Partial destructuring
var [x, y] = [1, 2, 3, 4, 5]; // Error! Wrong length
// Use rest pattern for variable length
```

---

## List with Rest Pattern

```dart
// Capture remaining elements
var [first, ...rest] = [1, 2, 3, 4, 5];
print('First: $first, Rest: $rest'); // First: 1, Rest: [2, 3, 4, 5]

// Rest at different positions
var [...rest, last] = [1, 2, 3, 4, 5];
print('Rest: $rest, Last: $last');

var [first, ...middle, last] = [1, 2, 3, 4, 5];
print('$first, $middle, $last'); // 1, [2, 3, 4], 5

// Rest with type
var [String first, ...List<String> rest] = ['a', 'b', 'c'];
print('$first, $rest'); // a, [b, c]
```

---

## Nested List Destructuring

```dart
// Nested lists
var [[x, y], [z, w]] = [[1, 2], [3, 4]];
print('($x,$y) - ($z,$w)');

// Mixed nesting
var [first, [second, third]] = [1, [2, 3]];
print('$first, $second, $third');

// Deep nesting
var [[[a, b], [c, d]], [e, f]] = [
  [
    [1, 2],
    [3, 4]
  ],
  [5, 6]
];
print('$a,$b,$c,$d - $e,$f');

// List with nested rest
var [first, [second, ...rest]] = [1, [2, 3, 4, 5]];
print('$first, $second, $rest');
```

---

# Destructuring Maps

## Basic Map Destructuring

```dart
// Destructure map entries
var {'name': name, 'age': age} = {'name': 'Alice', 'age': 25};
print('$name is $age years old');

// With type annotations
var {String name, int age} = {'name': 'Alice', 'age': 25};
print('$name: $age');

// Ignoring keys
var {'name': name, 'age': _} = {'name': 'Alice', 'age': 25};
print('Name: $name');

// Partial destructuring
var {'name': name, 'city': city} = {
  'name': 'Alice',
  'age': 25,
  'city': 'NYC'
};
print('$name lives in $city'); // Alice lives in NYC
```

---

## Map with Conditions

```dart
// Conditional destructuring
var data = {'type': 'user', 'name': 'Alice', 'age': 25};

if (data case {'type': 'user', 'name': String name, 'age': int age}) {
  print('User: $name, $age');
}

// With nested maps
var complex = {
  'user': {
    'name': 'Alice',
    'age': 25,
    'address': {
      'city': 'NYC',
      'zip': '10001'
    }
  }
};

if (complex case {
  'user': {
    'name': String name,
    'address': {
      'city': String city,
      'zip': String zip
    }
  }
}) {
  print('$name lives in $city ($zip)');
}
```

---

# Destructuring Objects

## Basic Object Destructuring

```dart
class Point {
  final int x;
  final int y;
  
  Point(this.x, this.y);
}

class Person {
  final String name;
  final int age;
  final String? city;
  
  Person({required this.name, required this.age, this.city});
}

// Destructure object
var Point(x: x, y: y) = Point(10, 20);
print('x: $x, y: $y');

// With type annotations
var Person(name: String name, age: int age) = Person(name: 'Alice', age: 25);
print('$name is $age');

// With optional fields
var Person(name: String name, age: int age, city: String? city) = 
    Person(name: 'Alice', age: 25, city: 'NYC');
print('$name lives in ${city ?? "unknown"}');
```

---

## Nested Object Destructuring

```dart
class Address {
  final String street;
  final String city;
  
  Address(this.street, this.city);
}

class User {
  final String name;
  final Address address;
  
  User(this.name, this.address);
}

// Nested destructuring
var User(
  name: String name,
  address: Address(
    street: String street,
    city: String city
  )
) = User('Alice', Address('123 Main St', 'NYC'));

print('$name lives at $street in $city');

// With type inference
var User(name: name, address: Address(street: street, city: city)) = 
    User('Alice', Address('123 Main St', 'NYC'));
print('$name, $street, $city');
```

---

## Destructuring with Custom Getters

```dart
class Rectangle {
  final int width;
  final int height;
  
  Rectangle(this.width, this.height);
  
  // Custom getters for destructuring
  int get area => width * height;
  int get perimeter => 2 * (width + height);
}

// Destructure custom properties
var Rectangle(width: w, height: h, area: a, perimeter: p) = 
    Rectangle(10, 20);
print('$w x $h = $a area, $p perimeter');
```

---

# Destructuring in Functions

## Function Parameters

```dart
// Destructure in function parameters
void printPerson((String name, int age) person) {
  var (name, age) = person;
  print('$name is $age');
}

// Destructure directly in parameter
void printPoint((int x, int y) point) {
  var (x, y) = point;
  print('($x, $y)');
}

// Named record parameters
void displayUser({required (String name, int age) user}) {
  var (name, age) = user;
  print('$name: $age');
}
```

---

## Return Values

```dart
// Function returning destructured values
(String, int) getUserInfo() {
  return ('Alice', 25);
}

// Destructure on call
void main() {
  var (name, age) = getUserInfo();
  print('$name, $age');
  
  // Or with existing variables
  String existingName;
  int existingAge;
  (existingName, existingAge) = getUserInfo();
}
```

---

# Destructuring in Loops

## Iterating Collections

```dart
// List of records
List<(String, int)> users = [
  ('Alice', 25),
  ('Bob', 30),
  ('Charlie', 35),
];

// Destructure in for loop
for (var (name, age) in users) {
  print('$name is $age years old');
}

// With index
for (var i = 0; i < users.length; i++) {
  var (name, age) = users[i];
  print('$i: $name is $age');
}

// Map entries
Map<String, int> scores = {'Alice': 95, 'Bob': 87};

for (var MapEntry(:key, :value) in scores.entries) {
  print('$key scored $value');
}

// With type annotation
for (var (String name, int age) in users) {
  print('$name: $age');
}
```

---

# Destructuring in Conditional Statements

## If with Destructuring

```dart
// Destructure in if statement
if (getUser() case (String name, int age)) {
  print('User: $name, $age');
}

// With guards
if (getData() case (String name, int age) when age >= 18) {
  print('Adult: $name');
}

// Map destructuring in if
if (getConfig() case {
  'host': String host,
  'port': int port
}) {
  print('$host:$port');
}
```

---

## Switch with Destructuring

```dart
// Switch with destructuring
void process(Object value) {
  switch (value) {
    case (int x, int y):
      print('Point: ($x, $y)');
    case [String first, String second]:
      print('Two strings: $first, $second');
    case {'name': String name, 'age': int age}:
      print('$name is $age');
    default:
      print('Unknown');
  }
}
```

---

# Best Practices

## Use Destructuring for Clarity

```dart
// Without destructuring (verbose)
var user = getUser();
String name = user.$1;
int age = user.$2;
String email = user.$3;
print('$name ($age) - $email');

// With destructuring (concise)
var (name, age, email) = getUser();
print('$name ($age) - $email');

// Even better with named records
var (:name, :age, :email) = getUser();
print('$name ($age) - $email');
```

---

## Keep It Readable

```dart
// Good: Simple destructuring
var (x, y) = getCoordinates();

// Good: Named destructuring
var (:firstName, :lastName) = getPerson();

// Bad: Overly nested destructuring
var (((a, b), (c, d)), ((e, f), (g, h))) = getComplexData();

// Better: Break it down
var (matrix1, matrix2) = getComplexData();
var ((a, b), (c, d)) = matrix1;
var ((e, f), (g, h)) = matrix2;
```

---

# Common Mistakes

## Wrong Pattern Type

Wrong:

```dart
var (x, y) = {'x': 1, 'y': 2}; // Error: Can't destructure map as record
```

Correct:

```dart
var {'x': x, 'y': y} = {'x': 1, 'y': 2};
```

---

## Missing Fields

Wrong:

```dart
var [first, second] = [1, 2, 3]; // Error: Wrong length
```

Correct:

```dart
var [first, second, third] = [1, 2, 3];
var [first, ...rest] = [1, 2, 3];
```

---

## Incorrect Field Names

Wrong:

```dart
var (:name, :age, :city) = (name: 'Alice', age: 25);
// Error: city doesn't exist
```

Correct:

```dart
var (:name, :age) = (name: 'Alice', age: 25);
```

---

# Summary

Destructuring provides a powerful and concise way to extract data from complex structures. It works with records, lists, maps, objects, and can be used in variable declarations, function parameters, and loops.

---

# Next Steps

Now that you understand destructuring, continue to:

- [Enums](enums.md)
- [Extensions](extensions.md)
- [Classes](classes.md)
- [Constructors](constructors.md)

---

# Did You Know?

- Destructuring works with any data structure
- The rest pattern (`...`) captures remaining elements
- You can ignore values with the wildcard `_`
- Destructuring works with nested structures
- It's commonly used with pattern matching
- You can destructure custom getters
- Destructuring can be used in function parameters