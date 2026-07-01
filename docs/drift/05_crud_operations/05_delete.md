## Delete

**Removing records from your Drift database**

---

# What is it?

**Delete** operations permanently remove records from your database tables. In Drift, you use the `delete()` function with conditions to specify which records to remove. You can delete single records, groups of records, or all records in a table.

> **Think of Delete like "shredding documents"** – you find the right files, permanently destroy them, and they're gone forever (unless you have a backup).

```dart
// 👇 Basic delete
await (delete(users)..where((u) => u.id.equals(1))).go();

// Generated SQL:
// DELETE FROM users WHERE id = 1
```

> **What's happening here?**
> - **`delete(users)`** – Target the users table
> - **`where()`** – Specify which records to delete
> - **`go()`** – Execute the delete operation
> - **Permanent** – Data is permanently removed

---

# Why does it exist?

- **Remove Data** – Delete unwanted records
- **Cleanup** – Remove old or inactive data
- **Data Privacy** – Comply with GDPR/CCPA requests
- **Space Management** – Free up database space
- **Data Correction** – Remove incorrect entries
- **Cascade Deletes** – Remove related records automatically

---

# Basic Delete Operations

> **Simple delete operations**

## Delete Single Record

```dart
// 👇 Delete a user by ID
await (delete(users)..where((u) => u.id.equals(1))).go();

// Generated SQL:
// DELETE FROM users WHERE id = 1
```

## Delete Multiple Records

```dart
// 👇 Delete all inactive users
await (delete(users)..where((u) => u.isActive.equals(false))).go();

// 👇 Delete users under 18
await (delete(users)..where((u) => u.age < const Variable(18))).go();

// 👇 Delete with multiple conditions
await (delete(users)
  ..where((u) => 
    u.isActive.equals(false) &
    u.createdAt < const Variable(DateTime(2023, 1, 1))
  ))
  .go();
```

---

## Delete All Records

```dart
// 👇 Delete all users (use with caution!)
await delete(users).go();

// Generated SQL:
// DELETE FROM users
```

## Delete with Limit

```dart
// 👇 Delete only first 10 inactive users
await (delete(users)
  ..where((u) => u.isActive.equals(false))
  ..limit(10))
  .go();
```

---

# Delete with Returning

> **Getting deleted data back**

```dart
// 👇 Delete and return deleted records
final deletedUsers = await (delete(users)..where((u) => u.id.equals(1)))
  .returning();

print('Deleted user: ${deletedUsers.first.name}');

// 👇 Delete and return specific fields
final results = await (delete(users)..where((u) => u.id.equals(1)))
  .returning(returning: (u) => [u.id, u.name]);

print('Deleted ID: ${results.first[0]}, Name: ${results.first[1]}');
```

---

# Delete with Cascade

> **Deleting related data automatically**

```dart
// lib/database/tables/posts.dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 ON DELETE CASCADE
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)();
}

// 👇 When user is deleted, their posts are automatically deleted
await (delete(users)..where((u) => u.id.equals(1))).go();

// Generated SQL:
// DELETE FROM users WHERE id = 1
// (Posts with user_id = 1 are automatically deleted by cascade)
```

---

# Soft Delete

> **Marking records as deleted without removing them**

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  
  // 👇 Soft delete fields
  BoolColumn get isDeleted => boolean()
    .withDefault(const Constant(false))
    .named('is_deleted')();
  
  DateTimeColumn get deletedAt => dateTime()
    .nullable()
    .named('deleted_at')();
}

// lib/database/database.dart - Soft delete methods
class AppDatabase extends _$AppDatabase {
  // 👇 Soft delete a user
  Future<void> softDeleteUser(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        isDeleted: const Value(true),
        deletedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Restore a soft-deleted user
  Future<void> restoreUser(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        isDeleted: const Value(false),
        deletedAt: const Value(null),
      ));
  }
  
  // 👇 Hard delete (permanent)
  Future<void> hardDeleteUser(int userId) async {
    await (delete(users)..where((u) => u.id.equals(userId))).go();
  }
  
  // 👇 Get active users (not soft-deleted)
  Future<List<User>> getActiveUsers() async {
    return await (select(users)
      ..where((u) => u.isDeleted.equals(false)))
      .get();
  }
}
```

---

# Delete with Relations

> **Deleting related data manually**

```dart
// 👇 Delete user and their posts
Future<void> deleteUserWithPosts(int userId) async {
  await transaction(() async {
    // 1️⃣ Delete user's posts
    await (delete(posts)..where((p) => p.userId.equals(userId))).go();
    
    // 2️⃣ Delete user
    await (delete(users)..where((u) => u.id.equals(userId))).go();
  });
}

// 👇 Delete order with items and restore stock
Future<void> deleteOrderWithItems(int orderId) async {
  await transaction(() async {
    // 1️⃣ Get order items
    final items = await (select(orderItems)
      ..where((i) => i.orderId.equals(orderId)))
      .get();
    
    // 2️⃣ Restore stock
    for (final item in items) {
      final product = await (select(products)
        ..where((p) => p.id.equals(item.productId)))
        .getSingle();
      
      await (update(products)..where((p) => p.id.equals(product.id)))
        .write(ProductsCompanion(
          stock: Value(product.stock + item.quantity),
          updatedAt: Value(DateTime.now()),
        ));
    }
    
    // 3️⃣ Delete order items
    await (delete(orderItems)..where((i) => i.orderId.equals(orderId))).go();
    
    // 4️⃣ Delete order
    await (delete(orders)..where((o) => o.id.equals(orderId))).go();
  });
}
```

---

# Real-World Example

> **Complete e-commerce delete system**

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // ==================== USER DELETES ====================
  
  // 👇 Soft delete user
  Future<void> softDeleteUser(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        isDeleted: const Value(true),
        deletedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Restore user
  Future<void> restoreUser(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        isDeleted: const Value(false),
        deletedAt: const Value(null),
      ));
  }
  
  // 👇 Hard delete user with validation
  Future<void> hardDeleteUser(int userId) async {
    // Check if user has orders
    final orderCount = await (select(orders)
      ..where((o) => o.userId.equals(userId)))
      .count();
    
    if (orderCount > 0) {
      throw Exception('Cannot delete user with existing orders');
    }
    
    await (delete(users)..where((u) => u.id.equals(userId))).go();
  }
  
  // 👇 Delete user with returning
  Future<User?> deleteUserReturning(int userId) async {
    final results = await (delete(users)..where((u) => u.id.equals(userId)))
      .returning();
    return results.isNotEmpty ? results.first : null;
  }
  
  // 👇 Bulk soft delete
  Future<void> bulkSoftDeleteUsers(List<int> userIds) async {
    await into(users).batch((batch) {
      for (final id in userIds) {
        batch.update(
          users,
          UsersCompanion(
            isDeleted: const Value(true),
            deletedAt: Value(DateTime.now()),
          ),
          (u) => u.id.equals(id),
        );
      }
    });
  }

  // ==================== PRODUCT DELETES ====================
  
  // 👇 Soft delete product
  Future<void> softDeleteProduct(int productId) async {
    await (update(products)..where((p) => p.id.equals(productId)))
      .write(ProductsCompanion(
        isActive: const Value(false),
        updatedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Hard delete product with validation
  Future<void> hardDeleteProduct(int productId) async {
    // Check if product is in any order
    final inOrders = await (select(orderItems)
      ..where((i) => i.productId.equals(productId)))
      .count() > 0;
    
    if (inOrders) {
      throw Exception('Cannot delete product that has been ordered');
    }
    
    await (delete(products)..where((p) => p.id.equals(productId))).go();
  }
  
  // 👇 Delete out-of-stock products
  Future<int> deleteOutOfStockProducts() async {
    final count = await (delete(products)
      ..where((p) => p.stock.equals(0))
      ..where((p) => p.isActive.equals(false)))
      .go();
    return count;
  }

  // ==================== ORDER DELETES ====================
  
  // 👇 Cancel and delete order
  Future<void> deleteOrderWithRestoration(int orderId) async {
    await transaction(() async {
      // 1️⃣ Get order items
      final items = await (select(orderItems)
        ..where((i) => i.orderId.equals(orderId)))
        .get();
      
      // 2️⃣ Restore stock
      for (final item in items) {
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();
        
        await (update(products)..where((p) => p.id.equals(product.id)))
          .write(ProductsCompanion(
            stock: Value(product.stock + item.quantity),
            updatedAt: Value(DateTime.now()),
          ));
      }
      
      // 3️⃣ Delete order items
      await (delete(orderItems)..where((i) => i.orderId.equals(orderId))).go();
      
      // 4️⃣ Delete order
      await (delete(orders)..where((o) => o.id.equals(orderId))).go();
    });
  }
  
  // 👇 Delete old completed orders
  Future<int> deleteOldOrders(Duration olderThan) async {
    final cutoffDate = DateTime.now().subtract(olderThan);
    
    final count = await (delete(orders)
      ..where((o) => 
        o.status.equals('delivered') |
        o.status.equals('cancelled') |
        o.status.equals('refunded')
      )
      ..where((o) => o.orderDate < const Variable(cutoffDate)))
      .go();
    
    return count;
  }

  // ==================== DATA CLEANUP ====================
  
  // 👇 Delete all soft-deleted records
  Future<int> cleanupSoftDeletedUsers() async {
    final count = await (delete(users)
      ..where((u) => u.isDeleted.equals(true))
      ..where((u) => u.deletedAt < const Variable(
        DateTime.now().subtract(Duration(days: 30))
      )))
      .go();
    return count;
  }
  
  // 👇 Delete inactive users
  Future<int> deleteInactiveUsers() async {
    final count = await (delete(users)
      ..where((u) => u.isActive.equals(false))
      ..where((u) => u.isVerified.equals(false))
      ..where((u) => u.createdAt < const Variable(
        DateTime.now().subtract(Duration(days: 365))
      )))
      .go();
    return count;
  }
  
  // 👇 Delete orphaned data
  Future<void> deleteOrphanedData() async {
    await transaction(() async {
      // Delete order items without orders
      await customDelete(
        'DELETE FROM order_items WHERE order_id NOT IN (SELECT id FROM orders)'
      ).go();
      
      // Delete posts without users
      await customDelete(
        'DELETE FROM posts WHERE user_id NOT IN (SELECT id FROM users)'
      ).go();
    });
  }
}
```

```dart
// lib/ui/pages/admin_page.dart
class AdminPage extends StatefulWidget {
  final AppDatabase db;
  
  const AdminPage({required this.db});
  
  @override
  _AdminPageState createState() => _AdminPageState();
}

class _AdminPageState extends State<AdminPage> {
  bool _isDeleting = false;
  String? _resultMessage;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Admin Panel')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // User management
            _buildSection('User Management', [
              _buildActionButton(
                'Soft Delete User',
                _softDeleteUser,
                Colors.orange,
              ),
              _buildActionButton(
                'Hard Delete User',
                _hardDeleteUser,
                Colors.red,
              ),
              _buildActionButton(
                'Cleanup Soft Deleted Users',
                _cleanupSoftDeleted,
                Colors.purple,
              ),
            ]),
            SizedBox(height: 16),
            
            // Product management
            _buildSection('Product Management', [
              _buildActionButton(
                'Delete Out of Stock Products',
                _deleteOutOfStock,
                Colors.red,
              ),
              _buildActionButton(
                'Archive Old Products',
                _archiveOldProducts,
                Colors.orange,
              ),
            ]),
            SizedBox(height: 16),
            
            // Order management
            _buildSection('Order Management', [
              _buildActionButton(
                'Delete Old Orders (1 year+)',
                _deleteOldOrders,
                Colors.red,
              ),
              _buildActionButton(
                'Cancel Pending Orders (30 days+)',
                _cancelPendingOrders,
                Colors.orange,
              ),
            ]),
            SizedBox(height: 16),
            
            // Data cleanup
            _buildSection('Data Cleanup', [
              _buildActionButton(
                'Delete Orphaned Data',
                _deleteOrphanedData,
                Colors.purple,
              ),
            ]),
            
            if (_resultMessage != null) ...[
              SizedBox(height: 16),
              Container(
                padding: EdgeInsets.all(12),
                color: _isDeleting ? Colors.blue : Colors.green[50],
                child: Text(
                  _resultMessage!,
                  style: TextStyle(
                    color: _isDeleting ? Colors.white : Colors.green,
                  ),
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }
  
  Widget _buildSection(String title, List<Widget> children) {
    return Container(
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey[300]!),
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Padding(
            padding: EdgeInsets.all(12),
            child: Text(
              title,
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          ...children,
        ],
      ),
    );
  }
  
  Widget _buildActionButton(String label, VoidCallback onPressed, Color color) {
    return Padding(
      padding: EdgeInsets.symmetric(horizontal: 12, vertical: 4),
      child: ElevatedButton(
        onPressed: _isDeleting ? null : onPressed,
        style: ElevatedButton.styleFrom(
          backgroundColor: color,
          minimumSize: Size(double.infinity, 40),
        ),
        child: _isDeleting
            ? CircularProgressIndicator(color: Colors.white)
            : Text(label, style: TextStyle(color: Colors.white)),
      ),
    );
  }
  
  Future<void> _softDeleteUser() async {
    setState(() {
      _isDeleting = true;
      _resultMessage = 'Soft deleting user...';
    });
    
    try {
      await widget.db.softDeleteUser(1); // Example user ID
      setState(() {
        _resultMessage = '✅ User soft deleted successfully';
        _isDeleting = false;
      });
    } catch (e) {
      setState(() {
        _resultMessage = '❌ Error: $e';
        _isDeleting = false;
      });
    }
  }
  
  Future<void> _hardDeleteUser() async {
    setState(() {
      _isDeleting = true;
      _resultMessage = 'Hard deleting user...';
    });
    
    try {
      final deletedUser = await widget.db.deleteUserReturning(1);
      setState(() {
        _resultMessage = '✅ User deleted: ${deletedUser?.name ?? 'Unknown'}';
        _isDeleting = false;
      });
    } catch (e) {
      setState(() {
        _resultMessage = '❌ Error: $e';
        _isDeleting = false;
      });
    }
  }
  
  Future<void> _cleanupSoftDeleted() async {
    setState(() {
      _isDeleting = true;
      _resultMessage = 'Cleaning up soft-deleted records...';
    });
    
    try {
      final count = await widget.db.cleanupSoftDeletedUsers();
      setState(() {
        _resultMessage = '✅ Removed $count soft-deleted users (older than 30 days)';
        _isDeleting = false;
      });
    } catch (e) {
      setState(() {
        _resultMessage = '❌ Error: $e';
        _isDeleting = false;
      });
    }
  }
  
  Future<void> _deleteOutOfStock() async {
    setState(() {
      _isDeleting = true;
      _resultMessage = 'Deleting out-of-stock products...';
    });
    
    try {
      final count = await widget.db.deleteOutOfStockProducts();
      setState(() {
        _resultMessage = '✅ Deleted $count out-of-stock products';
        _isDeleting = false;
      });
    } catch (e) {
      setState(() {
        _resultMessage = '❌ Error: $e';
        _isDeleting = false;
      });
    }
  }
  
  Future<void> _archiveOldProducts() async {
    setState(() {
      _isDeleting = true;
      _resultMessage = 'Archiving old products...';
    });
    
    try {
      // Archive products older than 2 years
      await widget.db.bulkUpdateProducts([
        ProductUpdate(
          id: 0, // Example
          isActive: false,
        ),
      ]);
      setState(() {
        _resultMessage = '✅ Products archived';
        _isDeleting = false;
      });
    } catch (e) {
      setState(() {
        _resultMessage = '❌ Error: $e';
        _isDeleting = false;
      });
    }
  }
  
  Future<void> _deleteOldOrders() async {
    setState(() {
      _isDeleting = true;
      _resultMessage = 'Deleting old orders...';
    });
    
    try {
      final count = await widget.db.deleteOldOrders(Duration(days: 365));
      setState(() {
        _resultMessage = '✅ Deleted $count orders older than 1 year';
        _isDeleting = false;
      });
    } catch (e) {
      setState(() {
        _resultMessage = '❌ Error: $e';
        _isDeleting = false;
      });
    }
  }
  
  Future<void> _cancelPendingOrders() async {
    setState(() {
      _isDeleting = true;
      _resultMessage = 'Cancelling pending orders...';
    });
    
    try {
      // Cancel pending orders older than 30 days
      await widget.db.updateOrderStatusBulk(
        OrderStatusUpdate(
          orderId: 0,
          newStatus: 'cancelled',
        ),
      );
      setState(() {
        _resultMessage = '✅ Pending orders cancelled';
        _isDeleting = false;
      });
    } catch (e) {
      setState(() {
        _resultMessage = '❌ Error: $e';
        _isDeleting = false;
      });
    }
  }
  
  Future<void> _deleteOrphanedData() async {
    setState(() {
      _isDeleting = true;
      _resultMessage = 'Deleting orphaned data...';
    });
    
    try {
      await widget.db.deleteOrphanedData();
      setState(() {
        _resultMessage = '✅ Orphaned data removed';
        _isDeleting = false;
      });
    } catch (e) {
      setState(() {
        _resultMessage = '❌ Error: $e';
        _isDeleting = false;
      });
    }
  }
}
```

---

# Best Practices

- **Always use WHERE clause** – Avoid accidental deletions
- **Use soft delete** – Instead of hard delete when possible
- **Test delete operations** – In development first
- **Use transactions** – For multi-table deletions
- **Add validation** – Check if deletion is allowed
- **Use cascade delete** – For related data automatically
- **Backup before bulk delete** – Important data
- **Log delete operations** – Track what was deleted
- **Use returning** – When you need deleted data

---

# Common Mistakes

## Mistake 1: Deleting without WHERE clause

Wrong:
```dart
// 🚫 Deletes ALL records!
await delete(users).go();
```

Correct:
```dart
// ✅ Always specify condition
await (delete(users)..where((u) => u.id.equals(1))).go();
```

## Mistake 2: Not checking dependencies

Wrong:
```dart
// 🚫 Deletes user but leaves orphaned posts
await (delete(users)..where((u) => u.id.equals(1))).go();
```

Correct:
```dart
// ✅ Handle related data
await transaction(() async {
  await (delete(posts)..where((p) => p.userId.equals(1))).go();
  await (delete(users)..where((u) => u.id.equals(1))).go();
});
```

## Mistake 3: Not using transactions

Wrong:
```dart
// 🚫 Partial deletion if one fails
await (delete(orders)..where((o) => o.id.equals(1))).go();
await (delete(orderItems)..where((i) => i.orderId.equals(1))).go();
```

Correct:
```dart
// ✅ Atomic operation
await transaction(() async {
  await (delete(orders)..where((o) => o.id.equals(1))).go();
  await (delete(orderItems)..where((i) => i.orderId.equals(1))).go();
});
```

---

# Summary

| Method | Use Case | Returns |
|--------|----------|---------|
| `go()` | Basic delete | Number of rows |
| `returning()` | Delete with data | Deleted records |
| `batch.delete` | Multiple deletes | Batch results |
| `customDelete` | Complex SQL | Number of rows |

---

# Next Steps

Now you understand delete operations, let's dive deeper:

- [Upsert](link) – Insert or update
- [Transactions](link) – Transaction management
- [Batch Operations](link) – Advanced batch patterns

---

# Did You Know?

- **Delete operations are permanent** – No undo without backup

- **`returning()` requires SQLite 3.35+** – Returns deleted rows

- **Soft delete is safer** – Records can be restored

- **Cascade deletes are automatic** – With foreign key constraints

- **Deletes can be batched** – For better performance

- **Transactions can rollback deletes** – If an error occurs

- **SQLite doesn't free space immediately** – Run VACUUM to shrink

- **Deletes are type-safe** – Compile-time checking

---
