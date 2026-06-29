# Patterns

Understand Dart's pattern matching system and how to use patterns effectively.

---

# What is it?

Patterns are a syntactic feature introduced in Dart 3 that allow you to match, destructure, and bind values from complex data structures. Patterns enable you to extract data from objects, collections, and records in a concise and expressive way.

---

# Why does it exist?

Patterns exist to:

- Destructure complex data structures
- Enable powerful pattern matching
- Make code more concise and readable
- Extract data from objects and collections
- Support switch statements with rich patterns
- Reduce boilerplate code

---

# Pattern Types

## Variable Pattern

```dart
// Variable pattern binds a value to a variable
var (x, y) = (10, 20);
print('x: $x, y: $y');

// Variable pattern with type
var (String name, int age) = ('Alice', 25);

// Variable pattern with wildcard
var (_, age) = ('Alice', 25);
print('Age: $age');
```

---

## Constant Pattern

```dart
// Matching against constants
void checkValue(dynamic value) {
  switch (value) {
    case 0:
      print('Zero');
      break;
    case 42:
      print('The answer');
      break;
    case 'hello':
      print('Greeting');
      break;
    default:
      print('Something else');
  }
}

// Constant pattern in variable assignment
var (x, y) = (10, 20);
```

---

## Wildcard Pattern

```dart
// Ignoring values with wildcard
var (_, y) = (10, 20); // Ignore first value

// Multiple wildcards
var (_, _, z) = (1, 2, 3);

// In list patterns
var [_, second, _] = [1, 2, 3];
print(second); // 2

// In map patterns
var {'name': _, 'age': age} = {'name': 'Alice', 'age': 25};
print(age); // 25
```

---

# Pattern Matching with Switch

## Switch with Patterns

```dart
void describe(dynamic value) {
  switch (value) {
    case 0:
      print('Zero');
      break;
    case 42:
      print('The answer');
      break;
    case 'hello':
      print('Greeting');
      break;
    case int n when n > 0:
      print('Positive integer: $n');
      break;
    case int n:
      print('Integer: $n');
      break;
    case String s:
      print('String: $s');
      break;
    default:
      print('Unknown');
  }
}
```

---

## Switch with Records

```dart
void processPoint(Object value) {
  switch (value) {
    case (0, 0):
      print('Origin');
      break;
    case (int x, 0):
      print('On X-axis: x = $x');
      break;
    case (0, int y):
      print('On Y-axis: y = $y');
      break;
    case (int x, int y) when x == y:
      print('On diagonal: ($x, $y)');
      break;
    case (int x, int y):
      print('Point: ($x, $y)');
      break;
    default:
      print('Invalid point');
  }
}
```

---

## Switch with Lists

```dart
void processList(Object value) {
  switch (value) {
    case []:
      print('Empty list');
      break;
    case [int x]:
      print('Single element: $x');
      break;
    case [int a, int b]:
      print('Two elements: $a, $b');
      break;
    case [int a, int b, int c] when a + b == c:
      print('Valid triangle: $a + $b = $c');
      break;
    case [int first, ...int rest]:
      print('First: $first, Rest: $rest');
      break;
    default:
      print('Unknown list');
  }
}
```

---

# List Patterns

## Basic List Patterns

```dart
// Destructuring lists
var [a, b, c] = [1, 2, 3];
print('$a, $b, $c'); // 1, 2, 3

// Ignoring elements
var [first, _, third] = [1, 2, 3];
print('$first, $third'); // 1, 3

// Matching specific values
var [0, x, 2] = [0, 5, 2];
print('x: $x'); // 5

// With type annotations
var [String name, int age] = ['Alice', 25];
print('$name is $age years old');
```

---

## List Patterns with Rest

```dart
// Rest pattern captures remaining elements
var [first, ...rest] = [1, 2, 3, 4, 5];
print('First: $first, Rest: $rest'); // First: 1, Rest: [2, 3, 4, 5]

// Rest at different positions
var [...rest, last] = [1, 2, 3, 4, 5];
print('Rest: $rest, Last: $last');

var [first, ...middle, last] = [1, 2, 3, 4, 5];
print('$first, $middle, $last');

// Rest with type
var [String first, ...List<String> rest] = ['a', 'b', 'c'];
```

---

## Nested List Patterns

```dart
// Nested list patterns
var [[x, y], [z, w]] = [[1, 2], [3, 4]];
print('($x,$y) - ($z,$w)');

// Mixed nesting
var [first, [second, third]] = [1, [2, 3]];
print('$first, $second, $third');

// Complex nesting
var [[a, b], c, [d, e]] = [[1, 2], 3, [4, 5]];
```

---

# Map Patterns

## Basic Map Patterns

```dart
// Destructuring maps
var {'name': name, 'age': age} = {'name': 'Alice', 'age': 25};
print('$name is $age years old');

// Ignoring keys
var {'name': name, 'age': _} = {'name': 'Alice', 'age': 25};
print('Name: $name');

// With type annotations
var {String name, int age} = {'name': 'Alice', 'age': 25};
print('$name is $age years old');
```

---

## Map Patterns with Conditions

```dart
void processMap(Object value) {
  switch (value) {
    case {'type': 'user', 'name': String name, 'age': int age}:
      print('User: $name, $age');
      break;
    case {'type': 'product', 'name': String name, 'price': int price}:
      print('Product: $name, \$$price');
      break;
    case {'error': String error}:
      print('Error: $error');
      break;
    default:
      print('Unknown map');
  }
}
```

---

# Object Patterns

## Destructuring Objects

```dart
// Define a class with proper naming
class Point {
  final int x;
  final int y;
  
  Point(this.x, this.y);
}

// Destructuring object
var Point(x: x, y: y) = Point(10, 20);
print('x: $x, y: $y');

// Nested object patterns
class Line {
  final Point start;
  final Point end;
  
  Line(this.start, this.end);
}

var Line(start: Point(x: x1, y: y1), end: Point(x: x2, y: y2)) = 
    Line(Point(1, 2), Point(3, 4));
print('($x1,$y1) to ($x2,$y2)');
```

---

## Object Patterns with Type

```dart
void processShape(Object value) {
  switch (value) {
    case Point(x: int x, y: int y):
      print('Point: ($x, $y)');
      break;
    case Circle(radius: int r):
      print('Circle: radius $r');
      break;
    case Rectangle(width: int w, height: int h):
      print('Rectangle: ${w}x${h}');
      break;
    default:
      print('Unknown shape');
  }
}
```

---

# Pattern Guards

## Using Guards with Patterns

```dart
void processNumber(Object value) {
  switch (value) {
    case int n when n < 0:
      print('Negative: $n');
      break;
    case int n when n == 0:
      print('Zero');
      break;
    case int n when n > 0 && n <= 10:
      print('Small positive: $n');
      break;
    case int n when n > 10:
      print('Large positive: $n');
      break;
    default:
      print('Not an integer');
  }
}

// Guard with multiple conditions
void processUser(Object value) {
  switch (value) {
    case User(name: String name, age: int age) when age >= 18:
      print('Adult: $name');
      break;
    case User(name: String name, age: int age) when age < 18:
      print('Minor: $name');
      break;
    default:
      print('Not a user');
  }
}
```

---

## Guards with Destructuring

```dart
// Guard with destructured values
void processPoint(Object value) {
  switch (value) {
    case (int x, int y) when x == y:
      print('On diagonal at ($x, $y)');
      break;
    case (int x, int y) when x > 0 && y > 0:
      print('First quadrant: ($x, $y)');
      break;
    case (int x, int y) when x < 0 && y > 0:
      print('Second quadrant: ($x, $y)');
      break;
    default:
      print('Other point');
  }
}
```

---

# Pattern Variables

## Binding Variables

```dart
// Extract variables from patterns
var (x, y) = (10, 20);
var [a, b, c] = [1, 2, 3];
var {'name': name, 'age': age} = {'name': 'Alice', 'age': 25};

// Pattern variables in switch
void describe(Object value) {
  switch (value) {
    case (int x, int y):
      print('Point: ($x, $y)');
      break;
    case [String s]:
      print('List with string: $s');
      break;
  }
}
```

---

## Variable Scopes

```dart
// Variables are scoped to their pattern match
void process(Object value) {
  switch (value) {
    case (int x, int y):
      print('x: $x, y: $y');
      break;
    case (String s, int n):
      print('$s: $n');
      break;
  }
  
  // Error: x, y, s, n not accessible here
  // print(x); // Error!
}
```

---

# Pattern Combinations

## OR Patterns

```dart
// OR pattern (multiple alternatives)
void processValue(Object value) {
  switch (value) {
    case 0 | 1 | 2:
      print('Small number');
      break;
    case 'red' | 'green' | 'blue':
      print('Primary color');
      break;
    case int n when n == 5 || n == 10:
      print('5 or 10');
      break;
    default:
      print('Other');
  }
}
```

---

## AND Patterns

```dart
// AND pattern (multiple conditions)
void process(Object value) {
  switch (value) {
    case (int x, int y) when x > 0 && y > 0:
      print('Positive point: ($x, $y)');
      break;
    case [int a, int b, int c] when a + b == c && c > 0:
      print('Valid triangle');
      break;
    default:
      print('Other');
  }
}
```

---

# Pattern in Expressions

## Pattern in If Statements

```dart
// Pattern matching in if
if (getPoint() case (int x, int y) when x > 0 && y > 0) {
  print('Positive point');
}

// Pattern matching with assignment
if (getUser() case User(name: String name, age: int age)) {
  print('User: $name, $age');
}

// Pattern matching with guard
if (getData() case (String name, int age) when age >= 18) {
  print('Adult: $name');
}
```

---

## Pattern in Variables

```dart
// Declare and destructure
var (x, y) = (10, 20);
final (name, age) = ('Alice', 25);

// Pattern in function parameters
void process((String, int) data) {
  var (name, age) = data;
  print('$name is $age');
}

void process2([String name = 'Guest', int age = 0]) {
  print('$name is $age');
}
```

---

# Best Practices

## Use Patterns for Clarity

```dart
// Without pattern (verbose)
var data = fetchUser();
String name = data['name'] as String;
int age = data['age'] as int;
print('$name is $age');

// With pattern (concise)
var {'name': name, 'age': age} = fetchUser();
print('$name is $age');
```

---

## Use Guards for Complex Logic

```dart
// Complex switch without guards
switch (value) {
  case int n:
    if (n > 0) {
      print('Positive');
    } else if (n < 0) {
      print('Negative');
    } else {
      print('Zero');
    }
    break;
}

// With guards (clearer)
switch (value) {
  case int n when n > 0:
    print('Positive');
    break;
  case int n when n < 0:
    print('Negative');
    break;
  case int n:
    print('Zero');
    break;
}
```

---

# Common Mistakes

## Wrong Pattern Type

Wrong:

```dart
var (x, y) = {'x': 1, 'y': 2}; // Error! Wrong pattern type
```

Correct:

```dart
var {'x': x, 'y': y} = {'x': 1, 'y': 2};
```

---

## Incomplete Destructuring

Wrong:

```dart
var [first, second] = [1, 2, 3]; // Error! Wrong number of elements
```

Correct:

```dart
var [first, second, third] = [1, 2, 3];
var [first, ...rest] = [1, 2, 3];
```

---

## Missing Cases

Wrong:

```dart
switch (value) {
  case 0:
    print('Zero');
  case 1:
    print('One');
  // Missing default
}
```

Correct:

```dart
switch (value) {
  case 0:
    print('Zero');
    break;
  case 1:
    print('One');
    break;
  default:
    print('Other');
}
```

---

# Summary

Patterns in Dart provide powerful ways to match, destructure, and extract data from complex structures. They enable concise and readable code, especially when combined with switch statements and guards.

---

# Next Steps

Now that you understand patterns, continue to:

- [Pattern Matching](pattern-matching.md)
- [Destructuring](destructuring.md)
- [Enums](enums.md)
- [Classes](classes.md)

---

# Did You Know?

- Patterns were introduced in Dart 3
- Patterns support destructuring lists, maps, and objects
- Pattern guards allow conditional matching
- Wildcard patterns ignore values
- OR patterns support multiple alternatives
- Patterns can be used in switch, if, and variable declarations
- Pattern variables are scoped to their match context