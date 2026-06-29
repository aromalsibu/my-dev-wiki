# Typedefs

Understand how to create type aliases in Dart.

---

# What is it?

Typedefs (type aliases) allow you to create alternative names for existing types. They make your code more readable by giving meaningful names to complex type signatures, especially for function types, generics, and long type declarations.

---

# Why does it exist?

Typedefs exist to:

- Create meaningful names for complex types
- Reduce repetitive type annotations
- Improve code readability and maintainability
- Simplify function type declarations
- Make code more self-documenting
- Enable easier refactoring

---

# Basic Typedefs

## Simple Type Alias

```dart
// Simple typedef
typedef Name = String;
typedef Age = int;
typedef Email = String;
typedef PhoneNumber = String;

// Usage
void registerUser(Name name, Age age, Email email) {
  print('Registering $name ($age) with email $email');
}

// More readable than:
void registerUser(String name, int age, String email) {
  print('Registering $name ($age) with email $email');
}
```

---

## Generic Typedefs

```dart
// Generic typedef
typedef Pair<T> = (T, T);
typedef Triple<T> = (T, T, T);
typedef Result<T> = (bool, T?);

// Usage
Pair<int> coordinates = (10, 20);
Triple<String> names = ('Alice', 'Bob', 'Charlie');
Result<double> calculation = (true, 3.14);

print('Coordinates: ${coordinates.$1}, ${coordinates.$2}');
print('Names: ${names.$1}, ${names.$2}, ${names.$3}');
print('Result: ${calculation.$1}, ${calculation.$2}');
```

---

## Collection Typedefs

```dart
// Collection aliases
typedef UserList = List<String>;
typedef ScoreMap = Map<String, int>;
typedef StringSet = Set<String>;

// Usage
UserList users = ['Alice', 'Bob', 'Charlie'];
ScoreMap scores = {'Alice': 95, 'Bob': 87, 'Charlie': 92};
StringSet uniqueNames = {'Alice', 'Bob', 'Charlie'};

void printUsers(UserList list) {
  for (var user in list) {
    print(user);
  }
}

void printScores(ScoreMap map) {
  map.forEach((name, score) {
    print('$name: $score');
  });
}
```

---

# Function Typedefs

## Basic Function Types

```dart
// Function typedef
typedef IntOperation = int Function(int a, int b);
typedef StringCallback = String Function(String value);
typedef VoidCallback = void Function();

// Usage
IntOperation add = (a, b) => a + b;
IntOperation multiply = (a, b) => a * b;

StringCallback toUpper = (s) => s.toUpperCase();
VoidCallback printHello = () => print('Hello');

print(add(3, 5)); // 8
print(multiply(3, 5)); // 15
print(toUpper('hello')); // HELLO
printHello(); // Hello
```

---

## Function Types with Parameters

```dart
// Complex function typedefs
typedef AsyncCallback<T> = Future<T> Function();
typedef ErrorHandler = void Function(String error, StackTrace? stackTrace);
typedef DataTransformer<T, R> = R Function(T data);
typedef Predicate<T> = bool Function(T item);

// Usage
AsyncCallback<String> fetchData = () async {
  await Future.delayed(Duration(seconds: 1));
  return 'Data loaded';
};

ErrorHandler handleError = (error, stack) {
  print('Error: $error');
  if (stack != null) print('Stack: $stack');
};

DataTransformer<String, int> stringLength = (s) => s.length;
Predicate<int> isEven = (n) => n % 2 == 0;

// Use them
fetchData().then(print); // Data loaded
handleError('Something went wrong', null);
print(stringLength('Hello')); // 5
print(isEven(4)); // true
```

---

# Advanced Typedefs

## Typedefs with Generics

```dart
// Generic typedefs
typedef Operation<T> = T Function(T a, T b);
typedef Converter<T, R> = R Function(T input);
typedef Parser<T> = T Function(String input);

// Usage
Operation<int> addInts = (a, b) => a + b;
Operation<double> addDoubles = (a, b) => a + b;
Operation<String> concatenate = (a, b) => a + b;

Converter<String, int> length = (s) => s.length;
Parser<int> parseInt = (s) => int.parse(s);

print(addInts(3, 5)); // 8
print(addDoubles(3.14, 2.86)); // 6.0
print(concatenate('Hello', ' World')); // Hello World
print(length('Hello')); // 5
print(parseInt('42')); // 42
```

---

## Complex Generic Typedefs

```dart
// Complex type aliases
typedef Result<T> = ({bool success, T? data, String? error});
typedef AsyncResult<T> = Future<Result<T>>;
typedef StreamProcessor<T> = Stream<T> Function(Stream<T> input);
typedef Callback<T> = void Function(T value, {int index});

// Usage
Result<String> getData() {
  return (success: true, data: 'Hello', error: null);
}

Result<int> parseNumber(String input) {
  try {
    var value = int.parse(input);
    return (success: true, data: value, error: null);
  } catch (e) {
    return (success: false, data: null, error: e.toString());
  }
}

// Processing
var result = getData();
if (result.success) {
  print(result.data); // Hello
}

var numberResult = parseNumber('123');
if (numberResult.success) {
  print(numberResult.data); // 123
}
```

---

# Typedefs with Extensions

## Combining Typedefs and Extensions

```dart
// Typedef with extension
typedef ValidatedString = String;

extension ValidatedStringExtension on ValidatedString {
  bool get isValid => isNotEmpty;
  String get sanitized => trim();
  bool get isEmail => contains('@') && contains('.');
}

// Usage
ValidatedString name = 'Alice';
print(name.isValid); // true
print(name.sanitized); // 'Alice'
print(name.isEmail); // false

ValidatedString email = 'user@example.com';
print(email.isEmail); // true
```

---

# Typedefs in Classes

## Using Typedefs with Classes

```dart
typedef Validator<T> = bool Function(T value);
typedef Formatter<T> = String Function(T value);

class FormField<T> {
  final T value;
  final Validator<T> validator;
  final Formatter<T> formatter;
  
  FormField({required this.value, required this.validator, required this.formatter});
  
  bool get isValid => validator(value);
  String get formatted => formatter(value);
}

// Usage
var field = FormField<String>(
  value: 'Alice',
  validator: (v) => v.isNotEmpty,
  formatter: (v) => v.toUpperCase(),
);

print(field.isValid); // true
print(field.formatted); // ALICE
```

---

# Typedefs for Callbacks

## Event Handlers

```dart
// Event handler typedefs
typedef EventListener<T> = void Function(T event);
typedef EventFilter<T> = bool Function(T event);
typedef EventTransformer<T> = T Function(T event);

class EventEmitter<T> {
  final List<EventListener<T>> _listeners = [];
  final List<EventFilter<T>> _filters = [];
  
  void addListener(EventListener<T> listener) {
    _listeners.add(listener);
  }
  
  void addFilter(EventFilter<T> filter) {
    _filters.add(filter);
  }
  
  void emit(T event) {
    // Apply filters
    for (var filter in _filters) {
      if (!filter(event)) return;
    }
    
    // Notify listeners
    for (var listener in _listeners) {
      listener(event);
    }
  }
}

// Usage
var emitter = EventEmitter<String>();
emitter.addListener((event) => print('Received: $event'));
emitter.addFilter((event) => event.isNotEmpty);

emitter.emit('Hello'); // Received: Hello
emitter.emit(''); // No output (filtered)
```

---

# Typedefs for State Management

## State Management Patterns

```dart
// State management typedefs
typedef StateReducer<T> = T Function(T state, dynamic action);
typedef StateSelector<T, R> = R Function(T state);
typedef SideEffect<T> = void Function(T state);

class Store<T> {
  final T state;
  final StateReducer<T> reducer;
  
  Store(this.state, this.reducer);
  
  T dispatch(dynamic action) {
    return reducer(state, action);
  }
  
  R select<R>(StateSelector<T, R> selector) {
    return selector(state);
  }
}

// Usage
class CounterState {
  final int count;
  CounterState(this.count);
}

CounterState counterReducer(CounterState state, dynamic action) {
  if (action == 'increment') {
    return CounterState(state.count + 1);
  }
  if (action == 'decrement') {
    return CounterState(state.count - 1);
  }
  return state;
}

var store = Store(CounterState(0), counterReducer);
print(store.select((state) => state.count)); // 0
store.dispatch('increment');
var newState = store.dispatch('increment'); // But state is immutable
print(newState.count); // 2
```

---

# Best Practices

## Use Descriptive Names

```dart
// Good: Descriptive typedef names
typedef UserId = String;
typedef ProductCode = String;
typedef EmailAddress = String;
typedef PhoneNumber = String;

// Bad: Vague names
typedef S = String;
typedef N = int;
typedef L = List;
```

---

## Use for Complex Function Types

```dart
// Good: Clear function typedef
typedef AsyncDataFetcher<T> = Future<T> Function();
typedef ErrorHandler = void Function(Exception error);
typedef DataTransformer<T, R> = R Function(T input);

// Bad: Complex inline types
Future<String> Function() fetchData = () async => 'data';
void Function(Exception) handleError = (e) => print(e);
```

---

## Use for API Boundaries

```dart
// Good: Typedefs for API contracts
typedef ApiResponse<T> = ({bool success, T? data, String? error});
typedef ApiCallback<T> = void Function(ApiResponse<T> response);
typedef ApiEndpoint<T> = Future<ApiResponse<T>> Function();

class ApiClient {
  Future<ApiResponse<User>> getUser(String id) async {
    try {
      // Fetch user
      return (success: true, data: user, error: null);
    } catch (e) {
      return (success: false, data: null, error: e.toString());
    }
  }
}
```

---

# Common Mistakes

## Unnecessary Typedefs

Wrong:

```dart
// Unnecessary alias
typedef IntAlias = int;
typedef StringAlias = String;
```

Correct:

```dart
// Use only when meaningful
typedef UserId = String;
typedef Age = int;
```

---

## Overcomplicating Typedefs

Wrong:

```dart
// Overly complex
typedef ProcessResult<T1, T2, T3, R> = R Function(T1, T2, T3);

// Better: Use class or record
typedef ProcessResult = ({int code, String message, dynamic data});
```

---

## Not Using Typedefs for Repeated Types

Wrong:

```dart
// Repeating complex type
Future<void> process1(Future<String> Function() fetcher) { /* ... */ }
Future<void> process2(Future<String> Function() fetcher) { /* ... */ }
```

Correct:

```dart
// Single typedef
typedef DataFetcher = Future<String> Function();

Future<void> process1(DataFetcher fetcher) { /* ... */ }
Future<void> process2(DataFetcher fetcher) { /* ... */ }
```

---

# Summary

Typedefs make code more readable by giving meaningful names to complex types. They're especially useful for function types, generics, and API contracts.

---

# Next Steps

Now that you understand typedefs, continue to:

- [Metadata (Annotations)](metadata.md)
- [Comments & Documentation](comments-documentation.md)
- [Classes](classes.md)
- [Constructors](constructors.md)

---

# Did You Know?

- Typedefs are compile-time only (no runtime overhead)
- They can be generic with type parameters
- Typedefs work with any type (not just functions)
- Function typedefs are called "function type aliases"
- Typedefs can be exported from libraries
- They're resolved at compile time
- Typedefs can be nested and combined