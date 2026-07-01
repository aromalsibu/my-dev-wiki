## Generated Data Classes

**Understanding Drift's generated data classes and companions**

---

# What is it?

**Generated Data Classes** are the typed Dart classes that Drift automatically creates from your table definitions. These include the data class itself (representing a row), the companion class (for inserts and updates), and various helper classes. They provide type-safe access to your database rows and eliminate manual mapping code.

> **Think of Generated Data Classes like "automatic formatters"** – they take raw database rows and convert them into beautiful, type-safe Dart objects that are easy to work with.

```dart
// 👇 Your table definition
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
}

// 👇 Drift generates these classes

// 1️⃣ Data class (represents a row)
class User {
  final int id;
  final String name;
  final String email;
  final int? age;
  final bool isActive;
  
  User({
    required this.id,
    required this.name,
    required this.email,
    this.age,
    required this.isActive,
  });
}

// 2️⃣ Companion class (for inserts/updates)
class UsersCompanion extends UpdateCompanion<User> {
  final Value<int> id;
  final Value<String> name;
  final Value<String> email;
  final Value<int?> age;
  final Value<bool> isActive;
  
  const UsersCompanion({
    this.id = const Value.absent(),
    this.name = const Value.absent(),
    this.email = const Value.absent(),
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
  });
  
  UsersCompanion.insert({
    this.id = const Value.absent(),
    required String name,
    required String email,
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
  }) : super.insert();
}

// 3️⃣ Table manager (provides query builder)
class $UsersTable extends TableManager {
  // ... query methods
}
```

> **What's happening here?**
> - **Data Class** – Typed representation of a database row
> - **Companion Class** – For inserts and updates with `Value<T>`
> - **Table Manager** – Query builder and CRUD operations
> - **Type Safety** – All operations are fully typed

---

# Why does it exist?

- **Type Safety** – No manual casting or mapping
- **Code Generation** – Zero boilerplate to write
- **IDE Support** – Autocomplete and refactoring
- **Error Prevention** – Catch errors at compile-time
- **Consistency** – Generated code follows patterns
- **Productivity** – Focus on business logic, not mapping

---

# Data Classes

> **The main generated class representing a row**

## Standard Data Class

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// Generated User class (simplified)
class User {
  final int id;
  final String name;
  final String email;
  final int? age;
  final bool isActive;
  final DateTime createdAt;
  
  User({
    required this.id,
    required this.name,
    required this.email,
    this.age,
    required this.isActive,
    required this.createdAt,
  });
  
  // 👇 Copy with modifications
  User copyWith({
    int? id,
    String? name,
    String? email,
    int? age,
    bool? isActive,
    DateTime? createdAt,
  }) {
    return User(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      age: age ?? this.age,
      isActive: isActive ?? this.isActive,
      createdAt: createdAt ?? this.createdAt,
    );
  }
  
  // 👇 To JSON
  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'email': email,
    'age': age,
    'isActive': isActive,
    'createdAt': createdAt.toIso8601String(),
  };
  
  // 👇 From JSON
  factory User.fromJson(Map<String, dynamic> json) => User(
    id: json['id'] as int,
    name: json['name'] as String,
    email: json['email'] as String,
    age: json['age'] as int?,
    isActive: json['isActive'] as bool,
    createdAt: DateTime.parse(json['createdAt'] as String),
  );
}
```

> **Key insights:**
> - **All fields are final** – Immutable by default
> - **Required fields** – Non-nullable for NOT NULL columns
> - **Optional fields** – Nullable for columns with `.nullable()`
> - **`copyWith` method** – Create modified copies
> - **`toJson`/`fromJson`** – Serialization support

---

## Data Class with Custom Row Class

> **Using your own data class with `@UseRowClass`**

```dart
// 👇 Your custom data class
@UseRowClass
class AppUser {
  final int id;
  final String name;
  final String email;
  final int? age;
  final bool isActive;
  final DateTime createdAt;
  
  AppUser({
    required this.id,
    required this.name,
    required this.email,
    this.age,
    required this.isActive,
    required this.createdAt,
  });
  
  // 👇 Optional: Custom mapping
  factory AppUser.fromRow(Row row) {
    return AppUser(
      id: row.read('id'),
      name: row.read('name'),
      email: row.read('email'),
      age: row.readOrNull('age'),
      isActive: row.read('is_active') == 1,
      createdAt: row.read('created_at'),
    );
  }
}

// 👇 Table definition with custom row class
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  
  @override
  Type get rowClass => AppUser; // 👈 Tell Drift to use AppUser
}

// Now queries return AppUser instead of User
Future<List<AppUser>> getUsers() async {
  return await select(users).get(); // 👈 Returns List<AppUser>
}
```

---

## Data Class with Custom Methods

```dart
// 👇 Custom data class with business logic
@UseRowClass
class AppUser {
  final int id;
  final String name;
  final String email;
  final int? age;
  final bool isActive;
  final DateTime createdAt;
  
  AppUser({
    required this.id,
    required this.name,
    required this.email,
    this.age,
    required this.isActive,
    required this.createdAt,
  });
  
  // 👇 Business logic methods
  bool get isAdult => age != null && age! >= 18;
  bool get isTeenager => age != null && age! >= 13 && age! < 18;
  bool get isChild => age != null && age! < 13;
  
  String get displayName => name.isNotEmpty ? name : email.split('@').first;
  
  String get initials {
    final parts = name.split(' ');
    if (parts.length >= 2) {
      return '${parts.first[0]}${parts.last[0]}'.toUpperCase();
    }
    return name.isNotEmpty ? name[0].toUpperCase() : '?';
  }
  
  // 👇 Computed properties
  Duration get accountAge => DateTime.now().difference(createdAt);
  String get accountAgeText {
    final days = accountAge.inDays;
    if (days < 30) return '${days} days';
    if (days < 365) return '${days ~/ 30} months';
    return '${days ~/ 365} years';
  }
  
  // 👇 Copy with modifications
  AppUser copyWith({
    int? id,
    String? name,
    String? email,
    int? age,
    bool? isActive,
    DateTime? createdAt,
  }) {
    return AppUser(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      age: age ?? this.age,
      isActive: isActive ?? this.isActive,
      createdAt: createdAt ?? this.createdAt,
    );
  }
}

// Usage in your app
final user = AppUser(
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  age: 25,
  isActive: true,
  createdAt: DateTime.now().subtract(Duration(days: 100)),
);

print(user.isAdult); // true
print(user.initials); // JD
print(user.accountAgeText); // 3 months
```

---

# Companion Classes

> **For inserts and updates with `Value<T>`**

## Standard Companion

```dart
// Generated UsersCompanion (simplified)
class UsersCompanion extends UpdateCompanion<User> {
  // 👇 All columns as Value<T>
  final Value<int> id;
  final Value<String> name;
  final Value<String> email;
  final Value<int?> age;
  final Value<bool> isActive;
  final Value<DateTime> createdAt;
  
  // 👇 Constructor with all absent
  const UsersCompanion({
    this.id = const Value.absent(),
    this.name = const Value.absent(),
    this.email = const Value.absent(),
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
    this.createdAt = const Value.absent(),
  });
  
  // 👇 Constructor for inserts
  UsersCompanion.insert({
    this.id = const Value.absent(),
    required String name,
    required String email,
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
    this.createdAt = const Value.absent(),
  }) : super.insert();
}
```

## Using Companions

```dart
// 1️⃣ Insert with required fields (uses defaults)
await into(users).insert(
  UsersCompanion.insert(
    name: 'John Doe',
    email: 'john@example.com',
    // isActive: uses default (true)
    // createdAt: uses default (current time)
  ),
);

// 2️⃣ Insert with all fields
await into(users).insert(
  UsersCompanion.insert(
    name: 'Jane Smith',
    email: 'jane@example.com',
    age: Value(28),
    isActive: Value(true),
    // createdAt: uses default
  ),
);

// 3️⃣ Insert with null for optional field
await into(users).insert(
  UsersCompanion.insert(
    name: 'Bob Wilson',
    email: 'bob@example.com',
    age: const Value(null), // 👈 Explicitly insert NULL
  ),
);

// 4️⃣ Update specific fields
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    // Other fields remain unchanged
  ));

// 5️⃣ Update with multiple fields
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    age: Value(26),
    isActive: Value(false),
  ));
```

---

# Value<T> Explained

> **Understanding Drift's Value wrapper**

## What is Value<T>?

```dart
// 👇 Value<T> has three states

// 1️⃣ Absent (use default)
const Value.absent() // 👈 Don't update this field

// 2️⃣ Present (explicit value)
Value('John') // 👈 Set to this value
Value(25)     // 👈 Set to this value
Value(true)   // 👈 Set to this value

// 3️⃣ Explicit null
const Value(null) // 👈 Set to NULL (only for nullable columns)
```

## Using Value<T>

```dart
class UsersCompanion extends UpdateCompanion<User> {
  final Value<String> name;
  final Value<int?> age;
  final Value<bool> isActive;
  
  const UsersCompanion({
    this.name = const Value.absent(),
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
  });
}

// 👇 Different use cases

// 1️⃣ Don't update (keep existing)
UsersCompanion(
  // name = absent
  age: const Value.absent(),
)

// 2️⃣ Update with value
UsersCompanion(
  name: Value('New Name'),
  age: const Value.absent(),
)

// 3️⃣ Set to NULL (only nullable columns)
UsersCompanion(
  age: const Value(null), // age is int? (nullable)
)

// 4️⃣ Update with null check
int? newAge = getAgeFromUser();
UsersCompanion(
  age: newAge != null ? Value(newAge) : const Value.absent(),
)

// 5️⃣ Conditional update
UsersCompanion(
  name: shouldUpdateName ? Value(newName) : const Value.absent(),
  age: shouldUpdateAge ? Value(newAge) : const Value.absent(),
)
```

---

# Real-World Example

> **Complete data classes for e-commerce application**

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable().named('full_name')();
  TextColumn get passwordHash => text().named('password_hash')();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false))();
  BoolColumn get isAdmin => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
  DateTimeColumn get lastLogin => dateTime().nullable()();
}

// lib/database/data_classes/app_user.dart
import 'package:drift/drift.dart';

@UseRowClass
class AppUser {
  final int id;
  final String username;
  final String email;
  final String? fullName;
  final String passwordHash;
  final int? age;
  final bool isActive;
  final bool isVerified;
  final bool isAdmin;
  final DateTime createdAt;
  final DateTime? updatedAt;
  final DateTime? lastLogin;

  AppUser({
    required this.id,
    required this.username,
    required this.email,
    this.fullName,
    required this.passwordHash,
    this.age,
    required this.isActive,
    required this.isVerified,
    required this.isAdmin,
    required this.createdAt,
    this.updatedAt,
    this.lastLogin,
  });

  // 👇 Display name
  String get displayName => fullName ?? username;

  // 👇 Initials
  String get initials {
    if (fullName != null && fullName!.isNotEmpty) {
      final parts = fullName!.split(' ');
      if (parts.length >= 2) {
        return '${parts.first[0]}${parts.last[0]}'.toUpperCase();
      }
      return fullName![0].toUpperCase();
    }
    return username[0].toUpperCase();
  }

  // 👇 Age checks
  bool get isAdult => age != null && age! >= 18;
  bool get isTeenager => age != null && age! >= 13 && age! < 18;
  bool get isChild => age != null && age! < 13;

  // 👇 Account age
  Duration get accountAge => DateTime.now().difference(createdAt);
  
  String get accountAgeText {
    final days = accountAge.inDays;
    if (days < 1) return 'Today';
    if (days < 7) return '$days days';
    if (days < 30) return '${days ~/ 7} weeks';
    if (days < 365) return '${days ~/ 30} months';
    return '${days ~/ 365} years';
  }

  // 👇 Status
  bool get isOnline => lastLogin != null && 
    DateTime.now().difference(lastLogin!).inMinutes < 5;

  bool get canLogin => isActive && isVerified;

  // 👇 Copy with
  AppUser copyWith({
    int? id,
    String? username,
    String? email,
    String? fullName,
    String? passwordHash,
    int? age,
    bool? isActive,
    bool? isVerified,
    bool? isAdmin,
    DateTime? createdAt,
    DateTime? updatedAt,
    DateTime? lastLogin,
  }) {
    return AppUser(
      id: id ?? this.id,
      username: username ?? this.username,
      email: email ?? this.email,
      fullName: fullName ?? this.fullName,
      passwordHash: passwordHash ?? this.passwordHash,
      age: age ?? this.age,
      isActive: isActive ?? this.isActive,
      isVerified: isVerified ?? this.isVerified,
      isAdmin: isAdmin ?? this.isAdmin,
      createdAt: createdAt ?? this.createdAt,
      updatedAt: updatedAt ?? this.updatedAt,
      lastLogin: lastLogin ?? this.lastLogin,
    );
  }

  // 👇 JSON
  Map<String, dynamic> toJson() => {
    'id': id,
    'username': username,
    'email': email,
    'fullName': fullName,
    'age': age,
    'isActive': isActive,
    'isVerified': isVerified,
    'isAdmin': isAdmin,
    'createdAt': createdAt.toIso8601String(),
    'updatedAt': updatedAt?.toIso8601String(),
    'lastLogin': lastLogin?.toIso8601String(),
  };

  factory AppUser.fromJson(Map<String, dynamic> json) => AppUser(
    id: json['id'] as int,
    username: json['username'] as String,
    email: json['email'] as String,
    fullName: json['fullName'] as String?,
    passwordHash: json['passwordHash'] as String,
    age: json['age'] as int?,
    isActive: json['isActive'] as bool,
    isVerified: json['isVerified'] as bool,
    isAdmin: json['isAdmin'] as bool,
    createdAt: DateTime.parse(json['createdAt'] as String),
    updatedAt: json['updatedAt'] != null 
        ? DateTime.parse(json['updatedAt'] as String) 
        : null,
    lastLogin: json['lastLogin'] != null 
        ? DateTime.parse(json['lastLogin'] as String) 
        : null,
  );
}

// lib/database/tables/products.dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  RealColumn get price => real().customConstraint('CHECK (price >= 0)')();
  IntColumn get stock => integer().withDefault(const Constant(0))();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// lib/database/data_classes/app_product.dart
import 'package:drift/drift.dart';

@UseRowClass
class AppProduct {
  final int id;
  final String sku;
  final String name;
  final String? description;
  final double price;
  final int stock;
  final bool isActive;
  final DateTime createdAt;
  final DateTime? updatedAt;

  AppProduct({
    required this.id,
    required this.sku,
    required this.name,
    this.description,
    required this.price,
    required this.stock,
    required this.isActive,
    required this.createdAt,
    this.updatedAt,
  });

  // 👇 Computed properties
  bool get isInStock => stock > 0;
  bool get isLowStock => stock > 0 && stock < 5;
  bool get isOutOfStock => stock == 0;
  
  String get stockStatus {
    if (isOutOfStock) return 'Out of Stock';
    if (isLowStock) return 'Low Stock';
    return 'In Stock';
  }

  String get priceFormatted => '\$${price.toStringAsFixed(2)}';
  
  String get skuShort => sku.length > 8 ? '...${sku.substring(sku.length - 5)}' : sku;

  // 👇 Business logic
  bool canOrder(int quantity) => stock >= quantity && isActive;

  AppProduct copyWith({
    int? id,
    String? sku,
    String? name,
    String? description,
    double? price,
    int? stock,
    bool? isActive,
    DateTime? createdAt,
    DateTime? updatedAt,
  }) {
    return AppProduct(
      id: id ?? this.id,
      sku: sku ?? this.sku,
      name: name ?? this.name,
      description: description ?? this.description,
      price: price ?? this.price,
      stock: stock ?? this.stock,
      isActive: isActive ?? this.isActive,
      createdAt: createdAt ?? this.createdAt,
      updatedAt: updatedAt ?? this.updatedAt,
    );
  }

  Map<String, dynamic> toJson() => {
    'id': id,
    'sku': sku,
    'name': name,
    'description': description,
    'price': price,
    'stock': stock,
    'isActive': isActive,
    'createdAt': createdAt.toIso8601String(),
    'updatedAt': updatedAt?.toIso8601String(),
  };

  factory AppProduct.fromJson(Map<String, dynamic> json) => AppProduct(
    id: json['id'] as int,
    sku: json['sku'] as String,
    name: json['name'] as String,
    description: json['description'] as String?,
    price: json['price'] as double,
    stock: json['stock'] as int,
    isActive: json['isActive'] as bool,
    createdAt: DateTime.parse(json['createdAt'] as String),
    updatedAt: json['updatedAt'] != null 
        ? DateTime.parse(json['updatedAt'] as String) 
        : null,
  );
}

// lib/database/database.dart - Usage
@DriftDatabase(tables: [Users, Products])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Using custom data classes
  Future<List<AppUser>> getActiveUsers() async {
    return await (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  Future<AppUser?> getUserById(int id) async {
    return await (select(users)..where((u) => u.id.equals(id)))
        .getSingleOrNull();
  }

  // 👇 Using companions
  Future<int> createUser({
    required String username,
    required String email,
    required String passwordHash,
    String? fullName,
    int? age,
  }) async {
    return await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        passwordHash: passwordHash,
        fullName: Value(fullName),
        age: Value(age),
        // isActive: true (default)
        // isVerified: false (default)
        // isAdmin: false (default)
      ),
    );
  }

  Future<void> updateUserProfile(int userId, {
    String? fullName,
    int? age,
    bool? isActive,
  }) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        fullName: fullName != null ? Value(fullName) : const Value.absent(),
        age: age != null ? Value(age) : const Value.absent(),
        isActive: isActive != null ? Value(isActive) : const Value.absent(),
        updatedAt: Value(DateTime.now()),
      ));
  }

  // 👇 Using data class methods
  Future<void> verifyUser(int userId) async {
    final user = await getUserById(userId);
    if (user != null && !user.isVerified) {
      await (update(users)..where((u) => u.id.equals(userId)))
        .write(UsersCompanion(
          isVerified: const Value(true),
          updatedAt: Value(DateTime.now()),
        ));
    }
  }

  // 👇 Using data class in UI
  String getUserDisplayName(int userId) {
    final user = await getUserById(userId);
    return user?.displayName ?? 'Unknown User';
  }

  bool canUserLogin(int userId) {
    final user = await getUserById(userId);
    return user?.canLogin ?? false;
  }
}
```

```dart
// lib/ui/user_profile.dart
class UserProfile extends StatelessWidget {
  final AppUser user;
  
  const UserProfile({required this.user});
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: CircleAvatar(
          child: Text(user.initials),
        ),
        title: Text(user.displayName),
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(user.email),
            Text('Age: ${user.age ?? 'Not specified'}'),
            Text('Account: ${user.accountAgeText}'),
            Text('Status: ${user.isActive ? 'Active' : 'Inactive'}'),
          ],
        ),
        trailing: Icon(
          user.isActive ? Icons.check_circle : Icons.cancel,
          color: user.isActive ? Colors.green : Colors.red,
        ),
      ),
    );
  }
}
```

---

# Best Practices

- **Use custom data classes** – For business logic methods
- **Add computed properties** – To data classes
- **Use `copyWith` method** – For immutability
- **Use `Value.absent()`** – To skip fields in updates
- **Use `Value(null)`** – To set nullable fields to NULL
- **Keep companions for inserts/updates** – Don't modify manually
- **Add helper methods** – For common operations
- **Document data classes** – Explain business logic
- **Test data class methods** – Verify logic works

---

# Common Mistakes

## Mistake 1: Trying to manually create companions

Wrong:
```dart
// 🚫 Manually creating companion (won't work)
class UsersCompanion extends UpdateCompanion<User> {
  final String name; // Wrong type
}
```

Correct:
```dart
// ✅ Let Drift generate companions
// Don't manually define them
```

## Mistake 2: Using Value incorrectly

Wrong:
```dart
// 🚫 Trying to use raw value
await update(users).write(UsersCompanion(
  name: 'John', // ❌ Should be Value
));
```

Correct:
```dart
// ✅ Use Value wrapper
await update(users).write(UsersCompanion(
  name: Value('John'),
));
```

## Mistake 3: Modifying generated data classes

Wrong:
```dart
// 🚫 Editing generated file
// users.g.dart
class User {
  // Adding custom method
  String get displayName => name; // Will be overwritten
}
```

Correct:
```dart
// ✅ Use custom data class
@UseRowClass
class AppUser {
  // Your custom code here
}
```

---

# Summary

| Class Type | Purpose | Key Feature |
|------------|---------|-------------|
| **Data Class** | Represents a row | Type-safe, immutable |
| **Companion** | Inserts/updates | `Value<T>` wrapper |
| **Table Manager** | Query builder | CRUD operations |
| **Custom Row Class** | Business logic | Custom methods |

---

# Next Steps

Now you understand generated data classes, let's dive deeper:

- [Companions](link) – Deep dive into companions
- [Value<T>](link) – Understanding Value wrapper
- [Insertable](link) – Custom insert logic

---

# Did You Know?

- **Data classes are fully typed** – No manual casting needed

- **`copyWith` creates new instances** – Immutability pattern

- **Companions use `Value<T>`** – For optional fields

- **`Value.absent()` means "skip"** – Not "set to null"

- **`Value(null)` means "set to NULL"** – For nullable columns

- **Custom data classes need `@UseRowClass`** – Annotation

- **Generated data classes are final** – Cannot be extended

- **Table managers are generated** – For each table

---

