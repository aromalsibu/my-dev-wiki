## Insertable

**Customizing insert operations in Drift**

---

# What is it?

**Insertable** is an interface in Drift that allows you to define custom logic for how objects are inserted into your database. Instead of only using generated Companions, you can implement the `Insertable` interface to create custom insertable objects that handle complex transformations, validations, or data preparation before insertion.

> **Think of Insertable like a "custom form processor"** – instead of just filling in a form (Companion), you can pre-process, validate, and transform the data before it's submitted to the database.

```dart
// 👇 Implementing Insertable interface
class UserRegistration implements Insertable<User> {
  final String username;
  final String email;
  final String password;
  
  UserRegistration({
    required this.username,
    required this.email,
    required this.password,
  });
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    return {
      'username': Variable(username),
      'email': Variable(email),
      'password_hash': Variable(hashPassword(password)),
      'created_at': Variable(DateTime.now()),
      'is_active': Variable(true),
    };
  }
}

// Usage - insert custom object
await into(users).insert(
  UserRegistration(
    username: 'john_doe',
    email: 'john@example.com',
    password: 'secure_password',
  ),
);
```

> **What's happening here?**
> - **Insertable interface** – Defines how to convert to columns
> - **Custom logic** – Transform, validate, prepare data
> - **Flexibility** – Not limited to Companion fields
> - **Type Safety** – Still fully type-checked

---

# Why does it exist?

- **Custom Validation** – Validate data before insertion
- **Data Transformation** – Hash passwords, format dates
- **Complex Logic** – Handle calculated fields
- **Third-party Integration** – Convert external data formats
- **Business Rules** – Enforce business logic at insertion
- **Code Organization** – Separate insertion logic from database

---

# Basic Insertable Implementation

> **Creating custom Insertable objects**

## Simple Implementation

```dart
// 👇 Custom Insertable for user registration
class UserRegistration implements Insertable<User> {
  final String username;
  final String email;
  final String password;
  final bool isAdmin;
  
  UserRegistration({
    required this.username,
    required this.email,
    required this.password,
    this.isAdmin = false,
  });
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    return {
      'username': Variable(username),
      'email': Variable(email),
      'password_hash': Variable(_hashPassword(password)),
      'is_admin': Variable(isAdmin ? 1 : 0),
      'created_at': Variable(DateTime.now()),
      'is_active': Variable(1),
      'is_verified': Variable(0),
    };
  }
  
  String _hashPassword(String password) {
    // Simulate password hashing
    return 'hashed_$password';
  }
}

// Usage
await into(users).insert(
  UserRegistration(
    username: 'john_doe',
    email: 'john@example.com',
    password: 'secure_password',
    isAdmin: true,
  ),
);
```

---

## Insertable with Validation

> **Validating data before insertion**

```dart
class ValidatedUser implements Insertable<User> {
  final String username;
  final String email;
  final int? age;
  final bool isActive;
  
  ValidatedUser({
    required this.username,
    required this.email,
    this.age,
    this.isActive = true,
  }) {
    // 👇 Validate in constructor
    if (username.length < 3) {
      throw ArgumentError('Username must be at least 3 characters');
    }
    
    if (!email.contains('@')) {
      throw ArgumentError('Invalid email format');
    }
    
    if (age != null && (age! < 0 || age! > 150)) {
      throw ArgumentError('Age must be between 0 and 150');
    }
  }
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    return {
      'username': Variable(username),
      'email': Variable(email),
      'age': age != null ? Variable(age!) : Variable(null),
      'is_active': Variable(isActive ? 1 : 0),
      'created_at': Variable(DateTime.now()),
    };
  }
}

// Usage with validation
try {
  await into(users).insert(
    ValidatedUser(
      username: 'jo', // ❌ Throws ArgumentError
      email: 'john@example.com',
    ),
  );
} catch (e) {
  print('Validation failed: $e');
}
```

---

# Advanced Insertable Patterns

> **Complex Insertable implementations**

## Pattern 1: Form Data Insertable

```dart
// 👇 Insertable from form data
class UserFormData implements Insertable<User> {
  final Map<String, dynamic> data;
  
  UserFormData(this.data);
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    final columns = <String, Expression>{};
    
    // Process each field with validation
    if (data.containsKey('username')) {
      final username = data['username'] as String;
      if (username.length < 3) {
        throw ArgumentError('Username too short');
      }
      columns['username'] = Variable(username);
    }
    
    if (data.containsKey('email')) {
      final email = data['email'] as String;
      if (!email.contains('@')) {
        throw ArgumentError('Invalid email');
      }
      columns['email'] = Variable(email);
    }
    
    if (data.containsKey('age')) {
      final age = data['age'] as int?;
      if (age != null && (age < 0 || age > 150)) {
        throw ArgumentError('Invalid age');
      }
      columns['age'] = Variable(age);
    }
    
    if (data.containsKey('isActive')) {
      columns['is_active'] = Variable(data['isActive'] as bool ? 1 : 0);
    }
    
    // 👇 Add timestamps automatically
    columns['created_at'] = Variable(DateTime.now());
    columns['updated_at'] = Variable(DateTime.now());
    
    return columns;
  }
}

// Usage
final formData = UserFormData({
  'username': 'john_doe',
  'email': 'john@example.com',
  'age': 25,
  'isActive': true,
});

await into(users).insert(formData);
```

---

## Pattern 2: API Response Insertable

```dart
// 👇 Insertable from API response
class ApiUser implements Insertable<User> {
  final Map<String, dynamic> apiResponse;
  
  ApiUser(this.apiResponse);
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    // API fields: id, login, email, profile, created_at
    return {
      'username': Variable(apiResponse['login'] as String),
      'email': Variable(apiResponse['email'] as String),
      'full_name': Variable(apiResponse['profile']['name'] as String?),
      'age': Variable(apiResponse['profile']['age'] as int?),
      'is_active': Variable(apiResponse['active'] as bool ? 1 : 0),
      'created_at': Variable(DateTime.parse(apiResponse['created_at'])),
    };
  }
}

// Usage
final apiResponse = await http.get('https://api.example.com/users/1');
final user = ApiUser(jsonDecode(apiResponse.body));
await into(users).insert(user);
```

---

## Pattern 3: Bulk Insert with Insertable

```dart
// 👇 Collection of Insertable objects
class UserList implements Insertable<User> {
  final List<Map<String, dynamic>> users;
  
  UserList(this.users);
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    // For bulk inserts, we need to handle multiple rows
    // This is simplified - bulk inserts use insertAll with Insertable
    return {};
  }
  
  // 👇 Get list of Insertable users
  List<UserInsertable> get insertables {
    return users.map((data) => UserInsertable(data)).toList();
  }
}

class UserInsertable implements Insertable<User> {
  final Map<String, dynamic> data;
  
  UserInsertable(this.data);
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    return {
      'username': Variable(data['username'] as String),
      'email': Variable(data['email'] as String),
      'created_at': Variable(DateTime.now()),
    };
  }
}

// Usage - bulk insert
final userList = UserList([
  {'username': 'user1', 'email': 'user1@example.com'},
  {'username': 'user2', 'email': 'user2@example.com'},
  {'username': 'user3', 'email': 'user3@example.com'},
]);

await into(users).insertAll(userList.insertables);
```

---

## Pattern 4: Transactional Insertable

```dart
// 👇 Insertable with transaction logic
class OrderWithItems implements Insertable<Order> {
  final int userId;
  final List<OrderItem> items;
  final String shippingAddress;
  
  OrderWithItems({
    required this.userId,
    required this.items,
    required this.shippingAddress,
  });
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    // This insertable creates an order with items in one transaction
    return {
      'user_id': Variable(userId),
      'order_number': Variable('ORD-${DateTime.now().millisecondsSinceEpoch}'),
      'total': Variable(_calculateTotal()),
      'shipping_address': Variable(shippingAddress),
      'status': Variable('pending'),
      'order_date': Variable(DateTime.now()),
    };
  }
  
  double _calculateTotal() {
    return items.fold(0.0, (sum, item) => sum + item.price * item.quantity);
  }
  
  // 👇 Custom insert method
  Future<int> insert(AppDatabase db) async {
    return await db.transaction(() async {
      // 1️⃣ Insert order
      final orderId = await db.into(db.orders).insert(this);
      
      // 2️⃣ Insert items
      for (final item in items) {
        await db.into(db.orderItems).insert(
          OrderItemInsertable(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            price: item.price,
          ),
        );
      }
      
      return orderId;
    });
  }
}

class OrderItem {
  final int productId;
  final int quantity;
  final double price;
  
  OrderItem({
    required this.productId,
    required this.quantity,
    required this.price,
  });
}

class OrderItemInsertable implements Insertable<OrderItem> {
  final int orderId;
  final int productId;
  final int quantity;
  final double price;
  
  OrderItemInsertable({
    required this.orderId,
    required this.productId,
    required this.quantity,
    required this.price,
  });
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    return {
      'order_id': Variable(orderId),
      'product_id': Variable(productId),
      'quantity': Variable(quantity),
      'unit_price': Variable(price),
      'subtotal': Variable(price * quantity),
    };
  }
}

// Usage
final order = OrderWithItems(
  userId: 1,
  items: [
    OrderItem(productId: 1, quantity: 2, price: 19.99),
    OrderItem(productId: 2, quantity: 1, price: 29.99),
  ],
  shippingAddress: '123 Main St, New York, NY',
);

final orderId = await order.insert(db);
```

---

# Insertable with Companions

> **Combining Insertable with Companions**

```dart
// 👇 Extending companion for custom logic
class CustomUserCompanion extends UsersCompanion {
  const CustomUserCompanion({
    super.id,
    super.name,
    super.email,
    super.age,
    super.isActive,
    super.createdAt,
  });
  
  // 👇 Custom factory constructor
  factory CustomUserCompanion.fromRegistration({
    required String name,
    required String email,
    String? password,
    int? age,
  }) {
    return CustomUserCompanion(
      name: Value(name),
      email: Value(email),
      age: Value(age),
      isActive: const Value(true),
      createdAt: Value(DateTime.now()),
    );
  }
  
  // 👇 Custom validation
  void validate() {
    if (name.value != null && name.value!.length < 2) {
      throw ArgumentError('Name too short');
    }
    if (email.value != null && !email.value!.contains('@')) {
      throw ArgumentError('Invalid email');
    }
    if (age.value != null && (age.value! < 0 || age.value! > 150)) {
      throw ArgumentError('Invalid age');
    }
  }
}

// Usage
final companion = CustomUserCompanion.fromRegistration(
  name: 'John Doe',
  email: 'john@example.com',
  age: 25,
);

companion.validate(); // Validate before insert
await into(users).insert(companion);
```

---

# Real-World Example

> **Complete e-commerce insertable system**

```dart
// lib/database/insertables/user_insertable.dart
import 'package:drift/drift.dart';
import 'package:bcrypt/bcrypt.dart';
import '../database.dart';

class UserInsertable implements Insertable<User> {
  final String username;
  final String email;
  final String password;
  final String? fullName;
  final int? age;
  final bool isAdmin;
  
  UserInsertable({
    required this.username,
    required this.email,
    required this.password,
    this.fullName,
    this.age,
    this.isAdmin = false,
  }) {
    // 👇 Validation in constructor
    if (username.length < 3) {
      throw ArgumentError('Username must be at least 3 characters');
    }
    if (!email.contains('@')) {
      throw ArgumentError('Invalid email format');
    }
    if (password.length < 8) {
      throw ArgumentError('Password must be at least 8 characters');
    }
    if (age != null && (age! < 0 || age! > 150)) {
      throw ArgumentError('Age must be between 0 and 150');
    }
  }
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    // 👇 Hash password
    final hashedPassword = BCrypt.hashpw(password, BCrypt.gensalt());
    
    return {
      'username': Variable(username),
      'email': Variable(email),
      'password_hash': Variable(hashedPassword),
      'full_name': Variable(fullName),
      'age': Variable(age),
      'is_admin': Variable(isAdmin ? 1 : 0),
      'is_active': Variable(1),
      'is_verified': Variable(0),
      'created_at': Variable(DateTime.now()),
      'updated_at': Value(null), // Let database handle
    };
  }
}

// lib/database/insertables/product_insertable.dart
class ProductInsertable implements Insertable<Product> {
  final String sku;
  final String name;
  final String? description;
  final double price;
  final int stock;
  final List<String> tags;
  
  ProductInsertable({
    required this.sku,
    required this.name,
    this.description,
    required this.price,
    this.stock = 0,
    this.tags = const [],
  }) {
    if (sku.length < 6) {
      throw ArgumentError('SKU must be at least 6 characters');
    }
    if (price < 0) {
      throw ArgumentError('Price cannot be negative');
    }
    if (stock < 0) {
      throw ArgumentError('Stock cannot be negative');
    }
  }
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    return {
      'sku': Variable(sku),
      'name': Variable(name),
      'description': Variable(description),
      'price': Variable(price),
      'stock': Variable(stock),
      'tags': Variable(tags.join(',')),
      'is_active': Variable(1),
      'created_at': Variable(DateTime.now()),
      'updated_at': Variable(DateTime.now()),
    };
  }
}

// lib/database/insertables/order_insertable.dart
class OrderInsertable implements Insertable<Order> {
  final int userId;
  final List<OrderItemInput> items;
  final String shippingAddress;
  final String billingAddress;
  final String paymentMethod;
  
  OrderInsertable({
    required this.userId,
    required this.items,
    required this.shippingAddress,
    required this.billingAddress,
    required this.paymentMethod,
  }) {
    if (items.isEmpty) {
      throw ArgumentError('Order must have at least one item');
    }
    if (shippingAddress.isEmpty) {
      throw ArgumentError('Shipping address required');
    }
  }
  
  double get total {
    return items.fold(
      0.0, 
      (sum, item) => sum + item.price * item.quantity,
    );
  }
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    return {
      'user_id': Variable(userId),
      'order_number': Variable('ORD-${DateTime.now().millisecondsSinceEpoch}'),
      'total': Variable(total),
      'shipping_address': Variable(shippingAddress),
      'billing_address': Variable(billingAddress),
      'payment_method': Variable(paymentMethod),
      'status': Variable('pending'),
      'payment_status': Variable('unpaid'),
      'order_date': Variable(DateTime.now()),
      'created_at': Variable(DateTime.now()),
    };
  }
  
  // 👇 Custom insert with transaction
  Future<int> insert(AppDatabase db) async {
    return await db.transaction(() async {
      // 1️⃣ Create order
      final orderId = await db.into(db.orders).insert(this);
      
      // 2️⃣ Add order items
      for (final item in items) {
        await db.into(db.orderItems).insert(
          OrderItemInsertable(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            price: item.price,
          ),
        );
        
        // 3️⃣ Update product stock
        final product = await db.getProduct(item.productId);
        await db.updateProductStock(
          item.productId, 
          product.stock - item.quantity,
        );
      }
      
      return orderId;
    });
  }
}

class OrderItemInput {
  final int productId;
  final int quantity;
  final double price;
  
  OrderItemInput({
    required this.productId,
    required this.quantity,
    required this.price,
  });
}

class OrderItemInsertable implements Insertable<OrderItem> {
  final int orderId;
  final int productId;
  final int quantity;
  final double price;
  
  OrderItemInsertable({
    required this.orderId,
    required this.productId,
    required this.quantity,
    required this.price,
  }) {
    if (quantity <= 0) {
      throw ArgumentError('Quantity must be positive');
    }
    if (price < 0) {
      throw ArgumentError('Price cannot be negative');
    }
  }
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    final subtotal = price * quantity;
    return {
      'order_id': Variable(orderId),
      'product_id': Variable(productId),
      'quantity': Variable(quantity),
      'unit_price': Variable(price),
      'subtotal': Variable(subtotal),
      'discount': Variable(0.0),
      'total': Variable(subtotal),
      'created_at': Variable(DateTime.now()),
    };
  }
}

// lib/database/database.dart
@DriftDatabase(tables: [Users, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Methods using insertables
  Future<int> registerUser({
    required String username,
    required String email,
    required String password,
    String? fullName,
    int? age,
  }) async {
    final user = UserInsertable(
      username: username,
      email: email,
      password: password,
      fullName: fullName,
      age: age,
    );
    
    return await into(users).insert(user);
  }

  Future<int> createProduct({
    required String sku,
    required String name,
    required double price,
    String? description,
    int stock = 0,
    List<String> tags = const [],
  }) async {
    final product = ProductInsertable(
      sku: sku,
      name: name,
      description: description,
      price: price,
      stock: stock,
      tags: tags,
    );
    
    return await into(products).insert(product);
  }

  Future<int> createOrder({
    required int userId,
    required List<CartItem> cartItems,
    required String shippingAddress,
    required String billingAddress,
    required String paymentMethod,
  }) async {
    // Convert cart items to order items
    final items = await Future.wait(
      cartItems.map((item) async {
        final product = await getProduct(item.productId);
        return OrderItemInput(
          productId: item.productId,
          quantity: item.quantity,
          price: product.price,
        );
      }).toList(),
    );
    
    final order = OrderInsertable(
      userId: userId,
      items: items,
      shippingAddress: shippingAddress,
      billingAddress: billingAddress,
      paymentMethod: paymentMethod,
    );
    
    return await order.insert(this);
  }

  Future<Product> getProduct(int id) async {
    final product = await (select(products)
      ..where((p) => p.id.equals(id)))
      .getSingleOrNull();
    if (product == null) throw Exception('Product not found');
    return product;
  }

  Future<void> updateProductStock(int productId, int newStock) async {
    await (update(products)..where((p) => p.id.equals(productId)))
      .write(ProductsCompanion(
        stock: Value(newStock),
        updatedAt: Value(DateTime.now()),
      ));
  }
}

class CartItem {
  final int productId;
  final int quantity;
  
  CartItem({required this.productId, required this.quantity});
}
```

---

# Best Practices

- **Validate in constructor** – Catch errors early
- **Use Insertable for complex logic** – Not for simple inserts
- **Keep Insertable focused** – One type per use case
- **Use transactions** – For multi-table operations
- **Handle errors gracefully** – Try-catch insert operations
- **Document Insertable purpose** – Explain what it does
- **Test Insertable logic** – Verify transformations
- **Use in API integrations** – Convert external data

---

# Common Mistakes

## Mistake 1: Not handling null values

Wrong:
```dart
// 🚫 Null values cause errors
@override
Map<String, Expression> toColumns(bool nullToAbsent) {
  return {
    'age': Variable(age), // If age is null, error
  };
}
```

Correct:
```dart
// ✅ Handle null values
@override
Map<String, Expression> toColumns(bool nullToAbsent) {
  return {
    'age': age != null ? Variable(age!) : Variable(null),
  };
}
```

## Mistake 2: Not validating data

Wrong:
```dart
// 🚫 Invalid data inserted
class UserInsertable implements Insertable<User> {
  final String username; // Could be empty
}
```

Correct:
```dart
// ✅ Validate in constructor
class UserInsertable implements Insertable<User> {
  final String username;
  
  UserInsertable({required this.username}) {
    if (username.isEmpty) throw ArgumentError('Username required');
  }
}
```

## Mistake 3: Not using transactions

Wrong:
```dart
// 🚫 Partial insert if one fails
await into(orders).insert(order);
await into(orderItems).insertAll(items); // If fails, order orphaned
```

Correct:
```dart
// ✅ Use transaction
await transaction(() async {
  await into(orders).insert(order);
  await into(orderItems).insertAll(items);
});
```

---

# Summary

| Feature | Purpose | Example |
|---------|---------|---------|
| **Insertable** | Custom insert logic | `implements Insertable<T>` |
| **Validation** | Ensure data quality | Constructor validation |
| **Transformation** | Modify data before insert | Hash passwords |
| **Transaction** | Atomic operations | Multi-table inserts |
| **Flexibility** | Handle any data source | API, forms, imports |

---

# Next Steps

Now you understand Insertable, let's dive deeper:

- [CRUD Operations](link) – Complete CRUD guide
- [Batch Insert](link) – Bulk insert operations
- [Custom Data Classes](link) – Advanced data classes

---

# Did You Know?

- **Insertable can transform data** – Hash, format, calculate

- **Insertable works with any data source** – API, forms, files

- **Insertable can validate** – Before database insertion

- **Insertable can be used in transactions** – Atomic operations

- **Insertable is type-safe** – `Insertable<T>` ensures type

- **Insertable works with `insertAll`** – Bulk operations

- **Insertable can extend Companion** – Combine with generated code

- **Insertable is reusable** – Across different tables

---

