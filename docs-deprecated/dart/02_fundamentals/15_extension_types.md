# Extension Types

Understand how to create zero-cost wrappers around existing types in Dart.

---

# What is it?

Extension Types are a powerful feature introduced in Dart 3 that allow you to create a compile-time wrapper around an existing type. Unlike regular extensions that add methods, extension types create a completely new type that behaves like the underlying type but with a different interface.

---

# Why does it exist?

Extension Types exist to:

- Create lightweight wrappers with zero runtime cost
- Add type safety without runtime overhead
- Define domain-specific types
- Hide implementation details
- Provide alternative APIs for existing types
- Prevent accidental mixing of semantically different values

---

# Basic Extension Types

## Simple Extension Type

```dart
// Extension type wrapping String
extension type Email(String value) {
  // Validation
  bool get isValid => value.contains('@') && value.contains('.');
  
  // Getters
  String get domain => value.split('@')[1];
  String get username => value.split('@')[0];
}

// Usage
Email email = Email('user@example.com');
print(email.isValid); // true
print(email.domain); // example.com
print(email.username); // user

// Underlying value access
String rawEmail = email.value; // 'user@example.com'
```

---

## Extension Type with Multiple Fields

```dart
// Extension type wrapping multiple primitive types
extension type Person((String name, int age) data) {
  String get name => data.$1;
  int get age => data.$2;
  
  bool get isAdult => age >= 18;
  
  String get greeting => 'Hello, I am $name, $age years old';
}

// Usage
var person = Person(('Alice', 25));
print(person.name); // Alice
print(person.age); // 25
print(person.isAdult); // true
print(person.greeting); // Hello, I am Alice, 25 years old

// Access underlying data
var raw = person.data; // ('Alice', 25)
```

---

# Extension Types vs Regular Extensions

## Extension Type (Compile-time Wrapper)

```dart
// Extension Type - Creates a new type
extension type Id(String value) {
  bool get isValid => value.isNotEmpty && RegExp(r'^[A-Z0-9]+$').hasMatch(value);
}

extension type UserId(String value) {
  bool get isValid => value.isNotEmpty && value.length >= 3;
}

void processId(Id id) {
  if (id.isValid) {
    print('Valid ID: ${id.value}');
  }
}

void processUserId(UserId userId) {
  if (userId.isValid) {
    print('Valid User ID: ${userId.value}');
  }
}

// Usage
Id id = Id('ABC123');
UserId userId = UserId('abc123');

processId(id);        // Valid ID: ABC123
processUserId(userId); // Valid User ID: abc123

// Type safety - prevents mixing
// processId(userId); // Error! Cannot assign UserId to Id
```

---

# Domain-Specific Types

## Creating Type-Safe Values

```dart
// Extension types for domain values
extension type ProductCode(String value) {
  // Validation logic
  bool get isValid => RegExp(r'^[A-Z]{2}\d{4}$').hasMatch(value);
  
  // Parsing
  String get category => value.substring(0, 2);
  String get number => value.substring(2);
  
  // Named constructors
  ProductCode.fromCategory(String category, String number) :
    this('${category.toUpperCase()}$number');
}

extension type Price(double value) {
  // Operations
  Price operator +(Price other) => Price(value + other.value);
  Price operator -(Price other) => Price(value - other.value);
  bool operator >(Price other) => value > other.value;
  bool operator <(Price other) => value < other.value;
  
  // Formatting
  String get formatted => '\$${value.toStringAsFixed(2)}';
  
  // Validation
  bool get isValid => value >= 0;
}

// Usage
var product = ProductCode('AB1234');
print(product.isValid); // true
print(product.category); // AB
print(product.number); // 1234

var price1 = Price(19.99);
var price2 = Price(9.99);
print((price1 + price2).formatted); // $29.98
print(price1 > price2); // true
```

---

# Wrapping Collections

## Extension Types with Collections

```dart
// Extension type wrapping a List
extension type UserList(List<String> users) {
  // Delegate methods
  int get length => users.length;
  bool get isEmpty => users.isEmpty;
  bool get isNotEmpty => users.isNotEmpty;
  
  // Custom operations
  bool contains(String user) => users.contains(user);
  String? get first => users.isNotEmpty ? users.first : null;
  String? get last => users.isNotEmpty ? users.last : null;
  
  // Modifying methods
  void add(String user) => users.add(user);
  void remove(String user) => users.remove(user);
  
  // Filtering
  List<String> get sorted => [...users]..sort();
  
  // Representation
  String get joined => users.join(', ');
}

// Usage
var list = UserList(['Alice', 'Bob', 'Charlie']);
print(list.length); // 3
print(list.joined); // Alice, Bob, Charlie
list.add('David');
print(list.joined); // Alice, Bob, Charlie, David
```

---

## Extension Type with Map

```dart
extension type Settings(Map<String, dynamic> config) {
  // Safe getters
  String getString(String key, [String defaultValue = '']) =>
      config[key] as String? ?? defaultValue;
  
  int getInt(String key, [int defaultValue = 0]) =>
      config[key] as int? ?? defaultValue;
  
  bool getBool(String key, [bool defaultValue = false]) =>
      config[key] as bool? ?? defaultValue;
  
  // Setters
  void setString(String key, String value) => config[key] = value;
  void setInt(String key, int value) => config[key] = value;
  void setBool(String key, bool value) => config[key] = value;
  
  // Check
  bool containsKey(String key) => config.containsKey(key);
}

// Usage
var settings = Settings({
  'host': 'localhost',
  'port': 8080,
  'debug': true,
});

print(settings.getString('host')); // localhost
print(settings.getInt('port')); // 8080
print(settings.getBool('debug')); // true
print(settings.getString('unknown', 'default')); // default
```

---

# Type Safety with Extension Types

## Preventing Unit Confusion

```dart
// Extension types for different units
extension type Meters(double value) {
  // Convert to kilometers
  Kilometers get toKilometers => Kilometers(value / 1000);
  Centimeters get toCentimeters => Centimeters(value * 100);
  
  // Operations
  Meters operator +(Meters other) => Meters(value + other.value);
  Meters operator -(Meters other) => Meters(value - other.value);
  bool operator >(Meters other) => value > other.value;
  
  String get formatted => '${value}m';
}

extension type Kilometers(double value) {
  Meters get toMeters => Meters(value * 1000);
  Centimeters get toCentimeters => Centimeters(value * 100000);
  
  String get formatted => '${value}km';
}

extension type Centimeters(double value) {
  Meters get toMeters => Meters(value / 100);
  Kilometers get toKilometers => Kilometers(value / 100000);
  
  String get formatted => '${value}cm';
}

// Usage - Type safe!
Meters length1 = Meters(1000);
Kilometers length2 = length1.toKilometers; // 1km
Centimeters length3 = length2.toCentimeters; // 100000cm

// Cannot mix units directly
// Kilometers invalid = length1; // Error! Different types
// Centimeters invalid2 = length2; // Error! Different types

print(length1.formatted); // 1000m
print(length2.formatted); // 1km
print(length3.formatted); // 100000cm
```

---

## Currency Types

```dart
extension type USD(double value) {
  USD operator +(USD other) => USD(value + other.value);
  USD operator -(USD other) => USD(value - other.value);
  
  Euro get toEuro => Euro(value * 0.85);
  GBP get toGBP => GBP(value * 0.72);
  
  String get formatted => '\$${value.toStringAsFixed(2)}';
}

extension type Euro(double value) {
  USD get toUSD => USD(value / 0.85);
  GBP get toGBP => GBP(value * 0.85);
  
  String get formatted => '€${value.toStringAsFixed(2)}';
}

extension type GBP(double value) {
  USD get toUSD => USD(value / 0.72);
  Euro get toEuro => Euro(value / 0.85);
  
  String get formatted => '£${value.toStringAsFixed(2)}';
}

// Usage
var dollars = USD(100);
var euros = dollars.toEuro; // €85.00
var pounds = euros.toGBP; // £72.25

print(dollars.formatted); // $100.00
print(euros.formatted); // €85.00
print(pounds.formatted); // £72.25
```

---

# Best Practices

## Use for Domain Modeling

```dart
// Good: Domain-specific types
extension type Email(String value) {
  bool get isValid => value.contains('@') && value.contains('.');
}

extension type PhoneNumber(String value) {
  bool get isValid => RegExp(r'^\d{10}$').hasMatch(value);
}

extension type ZipCode(String value) {
  bool get isValid => RegExp(r'^\d{5}(-\d{4})?$').hasMatch(value);
}

// Usage with type safety
void sendEmail(Email email) {
  if (email.isValid) {
    print('Sending to ${email.value}');
  }
}

// sendEmail(PhoneNumber('1234567890')); // Error! Type mismatch
```

---

## Use for API Boundaries

```dart
// Extension types for API data
extension type UserId(String value) {
  bool get isValid => value.isNotEmpty;
}

extension type ApiKey(String value) {
  bool get isValid => value.length >= 32;
}

class ApiClient {
  String? _apiKey;
  
  void setApiKey(ApiKey key) {
    if (key.isValid) {
      _apiKey = key.value;
    }
  }
  
  Future<void> getUser(UserId userId) async {
    if (!userId.isValid) {
      throw ArgumentError('Invalid user ID');
    }
    // Fetch user with userId.value
  }
}
```

---

# Common Mistakes

## Unnecessary Extension Types

Wrong:

```dart
// Overkill for simple cases
extension type SimpleString(String value) {
  // No extra behavior
}
```

Correct:

```dart
// Regular type alias if no behavior needed
typedef SimpleString = String;

// Or extension type with actual value
extension type ValidatedString(String value) {
  bool get isValid => value.isNotEmpty;
}
```

---

## Exposing Too Much

Wrong:

```dart
extension type User((String name, int age) data) {
  String get name => data.$1;
  int get age => data.$2;
  // Exposing internal structure - Bad
}
```

Correct:

```dart
extension type User((String name, int age) data) {
  String get name => data.$1;
  int get age => data.$2;
  
  // Expose only what's needed
  bool get isAdult => age >= 18;
  String get greeting => 'Hello, $name';
}
```

---

# Summary

Extension Types provide zero-cost wrappers that create new types with specific interfaces. They're ideal for domain modeling, adding type safety, and preventing accidental mixing of semantically different values.

---

# Next Steps

Now that you understand extension types, continue to:

- [Typedefs](typedefs.md)
- [Classes](classes.md)
- [Constructors](constructors.md)
- [Mixins](mixins.md)

---

# Did You Know?

- Extension types were introduced in Dart 3
- They have zero runtime cost (compile-time only)
- Extension types wrap existing types
- They can have methods, getters, and operators
- Extension types provide type safety
- They're great for domain modeling
- Extension types are different from regular extensions