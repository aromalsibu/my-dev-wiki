## Migration Examples

**Real-world migration scenarios and solutions**

---

# What is it?

**Migration Examples** are practical, real-world scenarios that demonstrate how to handle common database schema changes. These examples cover everything from adding columns and creating tables to complex data transformations and handling destructive changes.

> **Think of Migration Examples like "cookbook recipes"** – instead of just learning the theory, you see exactly how to handle specific situations with step-by-step instructions and best practices.

---

# Why does it exist?

- **Practical Guidance** – See how to handle real scenarios
- **Common Patterns** – Learn from typical use cases
- **Problem Solving** – Find solutions to specific problems
- **Best Practices** – See the right way to do things
- **Learning** – Understand by example
- **Templates** – Use as starting points for your migrations

---

# Adding a Column

> **Adding a new column to an existing table**

## Simple Column Addition

```dart
// 👇 Add a new column
class _MigrationV1ToV2 {
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    // Add the new column
    await migrator.addColumn(db.users, db.users.lastLogin);
    
    // Set default value for existing rows
    await db.customUpdate('''
      UPDATE users 
      SET last_login = created_at 
      WHERE last_login IS NULL
    ''').go();
  }
}

// In your database class
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (migrator, from, to) async {
    if (from == 1 && to == 2) {
      await _MigrationV1ToV2.up(migrator, this);
    }
  },
);
```

## Adding a NOT NULL Column

```dart
// 👇 Add NOT NULL column with default
class _MigrationV2ToV3 {
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    // Add the column as nullable first
    await migrator.addColumn(db.users, db.users.status);
    
    // Set default values
    await db.customUpdate('''
      UPDATE users 
      SET status = 'active' 
      WHERE status IS NULL
    ''').go();
    
    // Now make it NOT NULL (using custom SQL)
    await db.customSelect('''
      CREATE TABLE users_new (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT NOT NULL,
        status TEXT NOT NULL DEFAULT 'active',
        created_at INTEGER NOT NULL
      );
      
      INSERT INTO users_new (id, name, email, status, created_at)
      SELECT id, name, email, status, created_at FROM users;
      
      DROP TABLE users;
      ALTER TABLE users_new RENAME TO users;
    ''').go();
  }
}
```

---

# Creating a New Table

> **Adding a brand new table to your schema**

## Simple Table Creation

```dart
// 👇 Create a new table
class _MigrationV3ToV4 {
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    // Create the new table
    await migrator.createTable(db.profiles);
    
    // Add foreign key to users
    await migrator.addForeignKey(
      db.profiles,
      db.profiles.userId,
      db.users,
      db.users.id,
    );
    
    // Add index
    await migrator.addIndex(
      db.profiles,
      'idx_profiles_user',
      [db.profiles.userId],
    );
    
    // Create initial profiles for existing users
    await db.customInsert('''
      INSERT INTO profiles (user_id, full_name, created_at)
      SELECT 
        id,
        name as full_name,
        datetime('now')
      FROM users
    ''').go();
  }
}
```

## Table with Joins

```dart
// 👇 Create a table with relationship to existing data
class _MigrationV4ToV5 {
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    // Create orders table
    await migrator.createTable(db.orders);
    
    // Create order_items table
    await migrator.createTable(db.orderItems);
    
    // Set up foreign keys
    await migrator.addForeignKey(
      db.orderItems,
      db.orderItems.orderId,
      db.orders,
      db.orders.id,
    );
    await migrator.addForeignKey(
      db.orderItems,
      db.orderItems.productId,
      db.products,
      db.products.id,
    );
    
    // Add indexes
    await migrator.addIndex(db.orders, 'idx_orders_user', [db.orders.userId]);
    await migrator.addIndex(db.orderItems, 'idx_order_items_order', [db.orderItems.orderId]);
  }
}
```

---

# Adding an Index

> **Creating indexes for performance**

## Basic Index Addition

```dart
// 👇 Add a new index
class _MigrationV5ToV6 {
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    await migrator.addIndex(
      db.users,
      'idx_users_email',
      [db.users.email],
    );
    
    await migrator.addIndex(
      db.orders,
      'idx_orders_status_date',
      [db.orders.status, db.orders.orderDate],
    );
  }
}
```

## Conditional Index

```dart
// 👇 Add a partial index
class _MigrationV6ToV7 {
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    await db.customSelect('''
      CREATE INDEX idx_orders_active 
      ON orders(order_date) 
      WHERE status = 'active'
    ''').go();
  }
}
```

---

# Renaming a Column

> **Changing column names**

```dart
// 👇 Rename a column (requires recreating table)
class _MigrationV7ToV8 {
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    // SQLite doesn't support RENAME COLUMN directly
    // We need to recreate the table
    await db.transaction(() async {
      // Create new table with renamed column
      await db.customSelect('''
        CREATE TABLE users_new (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          email TEXT NOT NULL,
          status TEXT NOT NULL,
          last_login INTEGER,
          member_since INTEGER NOT NULL  -- renamed from created_at
        )
      ''').go();
      
      // Copy data
      await db.customInsert('''
        INSERT INTO users_new (id, name, email, status, last_login, member_since)
        SELECT id, name, email, status, last_login, created_at
        FROM users
      ''').go();
      
      // Swap tables
      await db.customSelect('DROP TABLE users').go();
      await db.customSelect('ALTER TABLE users_new RENAME TO users').go();
    });
  }
}
```

---

# Changing Data Types

> **Modifying column data types**

```dart
// 👇 Change column data type
class _MigrationV8ToV9 {
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    await db.transaction(() async {
      // Create table with new data type
      await db.customSelect('''
        CREATE TABLE products_new (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          price REAL NOT NULL,  -- changed from INTEGER
          stock INTEGER NOT NULL,
          category TEXT NOT NULL
        )
      ''').go();
      
      // Convert data
      await db.customInsert('''
        INSERT INTO products_new (id, name, price, stock, category)
        SELECT id, name, CAST(price AS REAL), stock, category
        FROM products
      ''').go();
      
      // Swap tables
      await db.customSelect('DROP TABLE products').go();
      await db.customSelect('ALTER TABLE products_new RENAME TO products').go();
    });
  }
}
```

---

# Real-World Examples

> **Complete migration scenarios**

## Example 1: Adding User Profiles

```dart
// lib/database/migrations/migration_001_add_profiles.dart
class Migration001AddProfiles {
  static const version = 2;
  static const description = 'Add user profiles table';
  
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    print('📦 Running migration: $description');
    
    // 1️⃣ Create profiles table
    await migrator.createTable(db.profiles);
    
    // 2️⃣ Add foreign key
    await migrator.addForeignKey(
      db.profiles,
      db.profiles.userId,
      db.users,
      db.users.id,
    );
    
    // 3️⃣ Add index
    await migrator.addIndex(
      db.profiles,
      'idx_profiles_user',
      [db.profiles.userId],
    );
    
    // 4️⃣ Migrate data
    await db.customInsert('''
      INSERT INTO profiles (user_id, full_name, bio, created_at)
      SELECT 
        id,
        name,
        'Welcome to the app!',
        datetime('now')
      FROM users
      WHERE NOT EXISTS (SELECT 1 FROM profiles WHERE profiles.user_id = users.id)
    ''').go();
    
    print('✅ Migration complete');
  }
  
  static Future<void> down(Migrator migrator, AppDatabase db) async {
    print('⬇️ Rolling back: $description');
    await db.customSelect('DROP TABLE IF EXISTS profiles').go();
    print('✅ Rollback complete');
  }
}
```

---

## Example 2: Adding Soft Delete

```dart
// lib/database/migrations/migration_002_soft_delete.dart
class Migration002SoftDelete {
  static const version = 3;
  static const description = 'Add soft delete to users and products';
  
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    print('📦 Running migration: $description');
    
    // 1️⃣ Add soft delete columns to users
    await migrator.addColumn(db.users, db.users.isDeleted);
    await migrator.addColumn(db.users, db.users.deletedAt);
    
    // 2️⃣ Add soft delete columns to products
    await migrator.addColumn(db.products, db.products.isDeleted);
    await migrator.addColumn(db.products, db.products.deletedAt);
    
    // 3️⃣ Set defaults for existing records
    await db.customUpdate('''
      UPDATE users SET is_deleted = 0 WHERE is_deleted IS NULL
    ''').go();
    
    await db.customUpdate('''
      UPDATE products SET is_deleted = 0 WHERE is_deleted IS NULL
    ''').go();
    
    // 4️⃣ Add indexes
    await migrator.addIndex(
      db.users,
      'idx_users_deleted',
      [db.users.isDeleted],
    );
    await migrator.addIndex(
      db.products,
      'idx_products_deleted',
      [db.products.isDeleted],
    );
    
    print('✅ Migration complete');
  }
  
  static Future<void> down(Migrator migrator, AppDatabase db) async {
    print('⬇️ Rolling back: $description');
    
    // Recreating table to remove columns (simplified)
    await db.customSelect('''
      CREATE TABLE users_new AS 
      SELECT id, name, email, created_at 
      FROM users
    ''').go();
    
    await db.customSelect('DROP TABLE users').go();
    await db.customSelect('ALTER TABLE users_new RENAME TO users').go();
    
    await db.customSelect('''
      CREATE TABLE products_new AS 
      SELECT id, name, price, stock, category, created_at 
      FROM products
    ''').go();
    
    await db.customSelect('DROP TABLE products').go();
    await db.customSelect('ALTER TABLE products_new RENAME TO products').go();
    
    print('✅ Rollback complete');
  }
}
```

---

## Example 3: Complex Data Migration

```dart
// lib/database/migrations/migration_003_normalize_categories.dart
class Migration003NormalizeCategories {
  static const version = 4;
  static const description = 'Normalize categories into separate table';
  
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    print('📦 Running migration: $description');
    
    await db.transaction(() async {
      // 1️⃣ Create categories table
      await migrator.createTable(db.categories);
      
      // 2️⃣ Extract distinct categories
      await db.customInsert('''
        INSERT INTO categories (name, created_at)
        SELECT DISTINCT category, datetime('now')
        FROM products
        WHERE category IS NOT NULL AND category != ''
      ''').go();
      
      // 3️⃣ Add category_id to products
      await migrator.addColumn(db.products, db.products.categoryId);
      
      // 4️⃣ Link products to categories
      await db.customUpdate('''
        UPDATE products 
        SET category_id = (
          SELECT id FROM categories 
          WHERE categories.name = products.category
        )
      ''').go();
      
      // 5️⃣ Add foreign key
      await migrator.addForeignKey(
        db.products,
        db.products.categoryId,
        db.categories,
        db.categories.id,
      );
      
      // 6️⃣ Add index
      await migrator.addIndex(
        db.products,
        'idx_products_category',
        [db.products.categoryId],
      );
      
      // 7️⃣ Drop old category column
      // SQLite doesn't support DROP COLUMN directly
      // Recreate products without category column
      await db.customSelect('''
        CREATE TABLE products_new (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          sku TEXT NOT NULL UNIQUE,
          price REAL NOT NULL,
          stock INTEGER NOT NULL,
          category_id INTEGER,
          is_active INTEGER NOT NULL DEFAULT 1,
          created_at INTEGER NOT NULL,
          updated_at INTEGER,
          FOREIGN KEY (category_id) REFERENCES categories(id)
        )
      ''').go();
      
      await db.customInsert('''
        INSERT INTO products_new (id, name, sku, price, stock, category_id, is_active, created_at, updated_at)
        SELECT id, name, sku, price, stock, category_id, is_active, created_at, updated_at
        FROM products
      ''').go();
      
      await db.customSelect('DROP TABLE products').go();
      await db.customSelect('ALTER TABLE products_new RENAME TO products').go();
    });
    
    print('✅ Migration complete');
  }
  
  static Future<void> down(Migrator migrator, AppDatabase db) async {
    print('⬇️ Rolling back: $description');
    // Complex rollback... 
    // This would require recreating the original structure
    print('⚠️ Rollback requires restoring from backup');
  }
}
```

---

## Example 4: Performance Optimization

```dart
// lib/database/migrations/migration_004_performance_optimization.dart
class Migration004PerformanceOptimization {
  static const version = 5;
  static const description = 'Add performance indexes';
  
  static Future<void> up(Migrator migrator, AppDatabase db) async {
    print('📦 Running migration: $description');
    
    // 1️⃣ Users table indexes
    await migrator.addIndex(db.users, 'idx_users_email', [db.users.email]);
    await migrator.addIndex(db.users, 'idx_users_status', [db.users.status]);
    await migrator.addIndex(db.users, 'idx_users_created', [db.users.createdAt]);
    
    // 2️⃣ Orders table indexes
    await migrator.addIndex(db.orders, 'idx_orders_user', [db.orders.userId]);
    await migrator.addIndex(db.orders, 'idx_orders_status', [db.orders.status]);
    await migrator.addIndex(db.orders, 'idx_orders_date', [db.orders.orderDate]);
    
    // 3️⃣ Products table indexes
    await migrator.addIndex(db.products, 'idx_products_sku', [db.products.sku]);
    await migrator.addIndex(db.products, 'idx_products_price', [db.products.price]);
    await migrator.addIndex(db.products, 'idx_products_stock', [db.products.stock]);
    
    // 4️⃣ Composite indexes
    await migrator.addIndex(
      db.orders,
      'idx_orders_user_status',
      [db.orders.userId, db.orders.status],
    );
    
    await migrator.addIndex(
      db.products,
      'idx_products_category_price',
      [db.products.categoryId, db.products.price],
    );
    
    print('✅ Migration complete');
  }
}
```

---

# Migration Examples Summary

| Example | Use Case | Complexity |
|---------|----------|------------|
| **Add Column** | Simple addition | Low |
| **Create Table** | New table | Low |
| **Add Index** | Performance | Low |
| **Rename Column** | Schema cleanup | Medium |
| **Change Type** | Data type update | Medium |
| **Soft Delete** | Add soft delete | Medium |
| **Normalize Data** | Complex transformation | High |

---

# Best Practices

- **Test migrations** – On a copy of production data
- **Backup before migration** – Always
- **Use transactions** – For multiple operations
- **Validate after migration** – Verify success
- **Document migrations** – What and why
- **Provide rollback** – Plan for failures
- **Test performance** – Ensure no slowdowns
- **Monitor progress** – For long migrations

---

# Migration Checklist

| Item | Description | Importance |
|------|-------------|------------|
| **Backup** | Create before migration | Critical |
| **Test** | Test on staging | Critical |
| **Transaction** | Use for safety | High |
| **Validation** | Verify after | High |
| **Rollback** | Plan for failure | High |
| **Documentation** | Track changes | Medium |
| **Performance** | Test speed | Medium |

---

# Next Steps

Now you understand migration examples, let's dive deeper:

- [Destructive Migrations](link) – Handling destructive changes
- [Testing Migrations](link) – Migration testing strategies
- [Best Practices](link) – Migration best practices

---

# Did You Know?

- **Migrations can be complex** – Data transformation is common

- **Migrations should be idempotent** – Safe to run multiple times

- **Migrations should be tested** – With realistic data

- **Migrations can be versioned** – Like application code

- **Migrations should be reversible** – Plan rollbacks

- **Migrations can be automated** – With CI/CD

- **Migrations are critical** – For production apps

- **Migrations require planning** – Think ahead

---
