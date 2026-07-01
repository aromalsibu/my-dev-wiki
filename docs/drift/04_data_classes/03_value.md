## Value<T>

**Mastering Drift's Value wrapper for optional fields**

---

# What is it?

**Value<T>** is a wrapper class in Drift that represents the state of a field in a companion. It has three possible states: present (with a value), absent (use default or skip), and explicitly null. This allows Drift to distinguish between "skip this field" and "set this field to NULL."

> **Think of Value<T> like a "checkbox with three states"** – checked (set value), unchecked (skip), or crossed out (set NULL). Each state tells Drift exactly what to do with that field.

```dart
// 👇 Three states of Value<T>

// 1️⃣ Present - Set this value
Value('John')     // Value present
Value(25)         // Value present
Value(true)       // Value present

// 2️⃣ Absent - Skip this field
const Value.absent()  // Don't update/use default

// 3️⃣ Explicit NULL - Set to NULL
const Value(null)     // Set to NULL (only for nullable columns)

// Usage in companion
UsersCompanion(
  name: Value('John'),           // Set to 'John'
  age: const Value.absent(),     // Skip (keep existing)
  email: const Value(null),      // Set to NULL
)
```

> **What's happening here?**
> - **Present** – Field will be set to the value
> - **Absent** – Field will be skipped (update) or default used (insert)
> - **Null** – Field will be set to NULL
> - **Type Safety** – Value<T> is generic and type-checked

---

# Why does it exist?

- **Partial Updates** – Skip fields you don't want to change
- **Default Handling** – Use defaults when fields are absent
- **NULL Distinction** – Distinguish between "skip" and "set NULL"
- **Type Safety** – Compile-time checking of values
- **IDE Support** – Autocomplete for Value operations
- **Immutability** – Value is immutable and safe

---

# The Three States

> **Understanding Value<T> states**

## State 1: Present

```dart
// 👇 Present state - field has a value
Value<String>('John')     // Non-null value
Value<int?>(25)           // Non-null value for nullable type
Value<bool>(true)         // Non-null value

// Usage
UsersCompanion(
  name: Value('John'),   // Set name to 'John'
  age: Value(25),         // Set age to 25
)

// Generated SQL for insert:
// INSERT INTO users (name, age) VALUES ('John', 25)

// Generated SQL for update:
// UPDATE users SET name = 'John', age = 25 WHERE id = 1
```

## State 2: Absent

```dart
// 👇 Absent state - skip the field
const Value.absent()      // Works for any type

// Usage
UsersCompanion(
  name: Value('John'),     // Set name
  age: const Value.absent(), // Skip age (keep existing)
)

// Generated SQL for insert:
// INSERT INTO users (name) VALUES ('John')
// (age uses default or is omitted if nullable)

// Generated SQL for update:
// UPDATE users SET name = 'John' WHERE id = 1
// (age remains unchanged)
```

## State 3: Explicit NULL

```dart
// 👇 Explicit NULL state - set to NULL
const Value(null)         // Works for nullable types only

// Usage
UsersCompanion(
  name: Value('John'),     // Set name
  age: const Value(null),  // Set age to NULL
)

// Generated SQL for insert:
// INSERT INTO users (name, age) VALUES ('John', NULL)

// Generated SQL for update:
// UPDATE users SET name = 'John', age = NULL WHERE id = 1
```

---

# Value in Inserts

> **How Value works when inserting new records**

## Required Fields

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();          // 👈 Required
  TextColumn get email => text()();         // 👈 Required
  IntColumn get age => integer().nullable()(); // 👈 Optional
  BoolColumn get isActive => boolean()      // 👈 Required (has default)
    .withDefault(const Constant(true))();
}

// 👇 Insert with required fields
await into(users).insert(
  UsersCompanion.insert(
    name: 'John',        // ✅ Required - must provide
    email: 'john@example.com', // ✅ Required - must provide
    // age: Value.absent() - ✅ Uses default (NULL)
    // isActive: Value.absent() - ✅ Uses default (true)
  ),
);
```

## Optional Fields

```dart
// 👇 Insert with all options
await into(users).insert(
  UsersCompanion.insert(
    name: 'John',
    email: 'john@example.com',
    age: Value(25),        // ✅ Set to 25
    // isActive: Value.absent() - ✅ Uses default
  ),
);

// 👇 Insert with explicit NULL
await into(users).insert(
  UsersCompanion.insert(
    name: 'John',
    email: 'john@example.com',
    age: const Value(null), // ✅ Set to NULL
  ),
);

// 👇 Insert with default value
await into(users).insert(
  UsersCompanion.insert(
    name: 'John',
    email: 'john@example.com',
    isActive: Value(false), // ✅ Override default
  ),
);
```

---

# Value in Updates

> **How Value works when updating existing records**

## Partial Updates

```dart
// 👇 Update only name
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),   // ✅ Update name
    age: const Value.absent(),     // ✅ Skip age
    isActive: const Value.absent(), // ✅ Skip isActive
  ));

// Generated SQL:
// UPDATE users SET name = 'John Updated' WHERE id = 1
// (age and isActive unchanged)
```

## Multiple Field Updates

```dart
// 👇 Update multiple fields
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    age: Value(26),
    isActive: const Value.absent(), // Skip
  ));
```

## Setting NULL

```dart
// 👇 Set age to NULL
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    age: const Value(null), // ✅ Set to NULL
  ));
```

## Clearing Optional Fields

```dart
// 👇 Reset multiple fields to NULL
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    fullName: const Value(null),
    bio: const Value(null),
    avatarUrl: const Value(null),
  ));
```

---

# Value Type Checking

> **Ensuring type safety with Value<T>**

## Generics with Value

```dart
// 👇 Value types match column types
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean()();
  DateTimeColumn get createdAt => dateTime()();
}

// ✅ Correct types
UsersCompanion(
  name: Value('John'),          // String
  age: Value(25),               // int
  isActive: Value(true),        // bool
  createdAt: Value(DateTime.now()), // DateTime
)

// ❌ Wrong types - compilation errors
UsersCompanion(
  name: Value(123),             // Error: Expected String
  age: Value('25'),             // Error: Expected int
  isActive: Value('true'),      // Error: Expected bool
)
```

## Nullable Types

```dart
// 👇 Nullable column
class Users extends Table {
  IntColumn get age => integer().nullable()(); // int?
}

// ✅ Valid values for nullable
UsersCompanion(
  age: const Value.absent(),   // Skip
  age: Value(25),              // Set to 25
  age: const Value(null),      // Set to NULL
)

// ❌ Invalid - non-nullable column
class Users extends Table {
  IntColumn get age => integer()(); // int (NOT NULL)
}

UsersCompanion(
  age: const Value(null),      // ❌ Error: Can't set NULL
)
```

---

# Value Helper Methods

> **Useful methods for working with Value<T>**

## Value Absence Check

```dart
// 👇 Check if value is present
final companion = UsersCompanion(
  name: Value('John'),
  age: const Value.absent(),
);

if (companion.name.present) {
  print('Name will be updated');
}

if (!companion.age.present) {
  print('Age will not be updated');
}
```

## Value Access

```dart
// 👇 Access value safely
final companion = UsersCompanion(
  name: Value('John'),
  age: const Value.absent(),
);

// ✅ Access with .value (throws if absent)
try {
  final name = companion.name.value; // 'John'
  final age = companion.age.value;   // Throws StateError
} catch (e) {
  print('Field is absent');
}

// ✅ Access with .valueOrNull
final name = companion.name.valueOrNull; // 'John'
final age = companion.age.valueOrNull;   // null (absent)

// ✅ Access with .orDefault
final name = companion.name.orDefault('Anonymous'); // 'John'
final age = companion.age.orDefault(18);            // 18 (default)
```

## Value Utility Methods

```dart
// 👇 Using Value in conditional logic
void updateUserAge(int userId, int? newAge) {
  final companion = UsersCompanion(
    age: newAge != null 
        ? Value(newAge) 
        : const Value.absent(),
  );
  
  // Update only if age is present
  if (companion.age.present) {
    update(users).write(companion);
  }
}

// 👇 Using Value with null safety
void updateUserFields(int userId, {
  String? name,
  int? age,
  bool? isActive,
}) {
  final companion = UsersCompanion(
    name: name != null ? Value(name) : const Value.absent(),
    age: age != null ? Value(age) : const Value.absent(),
    isActive: isActive != null ? Value(isActive) : const Value.absent(),
  );
  
  await (update(users)..where((u) => u.id.equals(userId)))
    .write(companion);
}
```

---

# Real-World Example

> **Complete Value<T> usage in e-commerce**

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable()();
  TextColumn get phoneNumber => text().nullable()();
  TextColumn get bio => text().nullable()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// lib/database/database.dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Using Value in user creation
  Future<int> createUser({
    required String username,
    required String email,
    String? fullName,
    String? phoneNumber,
    String? bio,
    int? age,
    bool isVerified = false,
  }) async {
    return await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        fullName: Value(fullName),
        phoneNumber: Value(phoneNumber),
        bio: Value(bio),
        age: Value(age),
        isVerified: Value(isVerified),
        // isActive: default (true)
        // createdAt: default (current time)
      ),
    );
  }

  // 👇 Using Value in user update
  Future<void> updateUser(int userId, {
    String? username,
    String? email,
    String? fullName,
    String? phoneNumber,
    String? bio,
    int? age,
    bool? isActive,
    bool? isVerified,
  }) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        username: username != null ? Value(username) : const Value.absent(),
        email: email != null ? Value(email) : const Value.absent(),
        fullName: fullName != null ? Value(fullName) : const Value.absent(),
        phoneNumber: phoneNumber != null ? Value(phoneNumber) : const Value.absent(),
        bio: bio != null ? Value(bio) : const Value.absent(),
        age: age != null ? Value(age) : const Value.absent(),
        isActive: isActive != null ? Value(isActive) : const Value.absent(),
        isVerified: isVerified != null ? Value(isVerified) : const Value.absent(),
        updatedAt: Value(DateTime.now()),
      ));
  }

  // 👇 Using Value to clear optional fields
  Future<void> clearUserProfile(int userId, {
    bool clearFullName = false,
    bool clearPhoneNumber = false,
    bool clearBio = false,
  }) async {
    final companion = UsersCompanion(
      fullName: clearFullName ? const Value(null) : const Value.absent(),
      phoneNumber: clearPhoneNumber ? const Value(null) : const Value.absent(),
      bio: clearBio ? const Value(null) : const Value.absent(),
      updatedAt: Value(DateTime.now()),
    );
    
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(companion);
  }

  // 👇 Using Value with conditional logic
  Future<void> updateUserProfile(int userId, {
    String? fullName,
    String? phoneNumber,
    String? bio,
  }) async {
    final companion = UsersCompanion(
      fullName: fullName != null 
          ? Value(fullName) 
          : (fullName == '' ? const Value(null) : const Value.absent()),
      phoneNumber: phoneNumber != null 
          ? Value(phoneNumber) 
          : (phoneNumber == '' ? const Value(null) : const Value.absent()),
      bio: bio != null 
          ? Value(bio) 
          : (bio == '' ? const Value(null) : const Value.absent()),
      updatedAt: Value(DateTime.now()),
    );
    
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(companion);
  }

  // 👇 Using Value in bulk operations
  Future<void> bulkUpdateUsers(List<UserUpdate> updates) async {
    await transaction(() async {
      for (final update in updates) {
        await (update(users)..where((u) => u.id.equals(update.id)))
          .write(UsersCompanion(
            username: update.username != null ? Value(update.username!) : const Value.absent(),
            email: update.email != null ? Value(update.email!) : const Value.absent(),
            isActive: update.isActive != null ? Value(update.isActive!) : const Value.absent(),
            updatedAt: Value(DateTime.now()),
          ));
      }
    });
  }

  // 👇 Using Value for safer updates
  Future<void> safeUpdateUser(int userId, {
    String? username,
    String? email,
    int? age,
  }) async {
    // Validate before creating companion
    if (username != null && username.length < 3) {
      throw Exception('Username must be at least 3 characters');
    }
    
    if (email != null && !email.contains('@')) {
      throw Exception('Invalid email format');
    }
    
    if (age != null && (age < 0 || age > 150)) {
      throw Exception('Age must be between 0 and 150');
    }
    
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        username: username != null ? Value(username) : const Value.absent(),
        email: email != null ? Value(email) : const Value.absent(),
        age: age != null ? Value(age) : const Value.absent(),
        updatedAt: Value(DateTime.now()),
      ));
  }

  // 👇 Using Value with null safety
  Future<void> updateUserWithNullSafety(int userId, {
    String? fullName,  // Use '' to clear, null to keep
    String? phoneNumber,
  }) async {
    final companion = UsersCompanion(
      fullName: fullName != null 
          ? Value(fullName.isEmpty ? null : fullName)
          : const Value.absent(),
      phoneNumber: phoneNumber != null 
          ? Value(phoneNumber.isEmpty ? null : phoneNumber)
          : const Value.absent(),
      updatedAt: Value(DateTime.now()),
    );
    
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(companion);
  }

  // 👇 Using Value for partial update from form data
  Future<void> updateUserFromForm(int userId, Map<String, dynamic> formData) async {
    final companion = UsersCompanion(
      fullName: formData.containsKey('fullName') 
          ? Value(formData['fullName'] as String?) 
          : const Value.absent(),
      phoneNumber: formData.containsKey('phoneNumber') 
          ? Value(formData['phoneNumber'] as String?) 
          : const Value.absent(),
      bio: formData.containsKey('bio') 
          ? Value(formData['bio'] as String?) 
          : const Value.absent(),
      age: formData.containsKey('age') 
          ? Value(formData['age'] as int?) 
          : const Value.absent(),
      isActive: formData.containsKey('isActive') 
          ? Value(formData['isActive'] as bool) 
          : const Value.absent(),
      updatedAt: Value(DateTime.now()),
    );
    
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(companion);
  }
}

class UserUpdate {
  final int id;
  final String? username;
  final String? email;
  final bool? isActive;
  
  UserUpdate({
    required this.id,
    this.username,
    this.email,
    this.isActive,
  });
}
```

```dart
// lib/ui/user_form.dart
class UserForm extends StatefulWidget {
  final AppDatabase db;
  final int? userId;
  
  const UserForm({required this.db, this.userId});
  
  @override
  _UserFormState createState() => _UserFormState();
}

class _UserFormState extends State<UserForm> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _nameController;
  late TextEditingController _emailController;
  late TextEditingController _ageController;
  late bool _isActive;
  
  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController();
    _emailController = TextEditingController();
    _ageController = TextEditingController();
    _isActive = true;
    
    if (widget.userId != null) {
      _loadUser();
    }
  }
  
  Future<void> _loadUser() async {
    final user = await (widget.db.select(widget.db.users)
      ..where((u) => u.id.equals(widget.userId!)))
      .getSingle();
    
    setState(() {
      _nameController.text = user.username;
      _emailController.text = user.email;
      _ageController.text = user.age?.toString() ?? '';
      _isActive = user.isActive;
    });
  }
  
  Future<void> _saveUser() async {
    if (!_formKey.currentState!.validate()) return;
    
    final age = int.tryParse(_ageController.text);
    
    if (widget.userId == null) {
      // Create new user
      await widget.db.createUser(
        username: _nameController.text,
        email: _emailController.text,
        age: age,
        isVerified: false,
      );
    } else {
      // Update existing user
      await widget.db.updateUser(
        widget.userId!,
        username: _nameController.text,
        email: _emailController.text,
        age: age,
        isActive: _isActive,
      );
    }
    
    Navigator.pop(context);
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.userId == null ? 'Create User' : 'Edit User'),
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _nameController,
                decoration: InputDecoration(labelText: 'Username'),
                validator: (value) {
                  if (value?.isEmpty ?? true) return 'Required';
                  if (value!.length < 3) return 'Must be at least 3 characters';
                  return null;
                },
              ),
              TextFormField(
                controller: _emailController,
                decoration: InputDecoration(labelText: 'Email'),
                validator: (value) {
                  if (value?.isEmpty ?? true) return 'Required';
                  if (!value!.contains('@')) return 'Invalid email';
                  return null;
                },
              ),
              TextFormField(
                controller: _ageController,
                decoration: InputDecoration(labelText: 'Age'),
                keyboardType: TextInputType.number,
                validator: (value) {
                  if (value?.isEmpty ?? true) return null; // Optional
                  final age = int.tryParse(value!);
                  if (age == null) return 'Must be a number';
                  if (age < 0 || age > 150) return 'Invalid age';
                  return null;
                },
              ),
              SwitchListTile(
                title: Text('Active'),
                value: _isActive,
                onChanged: (value) => setState(() => _isActive = value),
              ),
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: _saveUser,
                child: Text('Save'),
              ),
            ],
          ),
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _ageController.dispose();
    super.dispose();
  }
}
```

---

# Best Practices

- **Use `Value.absent()` for skips** – In updates
- **Use `Value(null)` for NULL** – For nullable columns
- **Use `Value(value)` for setting** – For non-null values
- **Check `.present` before accessing** – Avoid errors
- **Use `.valueOrNull` for safety** – Get value or null
- **Use `.orDefault` for defaults** – Fallback values
- **Validate before using** – Check business rules
- **Use in companions only** – Not in data classes
- **Document Value usage** – Explain field behavior

---

# Common Mistakes

## Mistake 1: Using `Value.absent()` with required fields in insert

Wrong:
```dart
// 🚫 Required field can't be absent in insert
await into(users).insert(
  UsersCompanion.insert(
    name: const Value.absent(), // ❌ Required
  ),
);
```

Correct:
```dart
// ✅ Provide value for required fields
await into(users).insert(
  UsersCompanion.insert(
    name: 'John', // Required field
  ),
);
```

## Mistake 2: Using `Value(null)` on non-nullable column

Wrong:
```dart
// 🚫 Non-nullable column can't be null
UsersCompanion(
  isActive: const Value(null), // ❌ isActive is NOT NULL
)
```

Correct:
```dart
// ✅ Non-nullable columns must have values
UsersCompanion(
  isActive: Value(true),
)
```

## Mistake 3: Not checking if value is present

Wrong:
```dart
// 🚫 May throw StateError
final name = companion.name.value; // Absent throws
```

Correct:
```dart
// ✅ Check before accessing
if (companion.name.present) {
  final name = companion.name.value;
}
// Or use .valueOrNull
final name = companion.name.valueOrNull;
```

---

# Summary

| State | Syntax | Insert | Update |
|-------|--------|--------|--------|
| **Present** | `Value(value)` | Set value | Set value |
| **Absent** | `Value.absent()` | Use default | Skip |
| **Null** | `Value(null)` | Set NULL | Set NULL |

---

# Next Steps

Now you understand Value<T>, let's dive deeper:

- [Insertable](link) – Custom insert logic
- [CRUD Operations](link) – Complete CRUD guide
- [Custom Data Classes](link) – Advanced data classes

---

# Did You Know?

- **Value.absent() is a const constructor** – Can be used as const

- **Value(null) is also const** – For nullable types

- **Value<T> is immutable** – Cannot be changed after creation

- **Value.absent() means "skip"** – Not "set to default"

- **Value(null) means "set to NULL"** – Different from absent

- **Generics enforce type safety** – Value<String> vs Value<int>

- **Value has `.present` getter** – Check if value exists

- **Value has `.valueOrNull`** – Safe access without exceptions

---

