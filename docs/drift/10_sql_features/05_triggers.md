## Triggers

**Automating database actions with triggers in Drift**

---

# What is it?

**Triggers** are database objects that automatically execute a specified set of SQL statements when certain events occur on a table, such as INSERT, UPDATE, or DELETE. They're useful for maintaining data integrity, auditing changes, enforcing business rules, and automating complex operations. Drift supports creating and managing triggers through raw SQL.

> **Think of Triggers like "automated checklists"** – whenever something happens (like a new order being placed), a trigger automatically performs related actions (like updating inventory, logging the event, and sending notifications) without you having to remember to do them.

```dart
// 👇 Create a trigger to update inventory when an order is placed
await db.customSelect('''
  CREATE TRIGGER update_inventory_after_order
  AFTER INSERT ON order_items
  FOR EACH ROW
  BEGIN
    UPDATE products 
    SET stock = stock - NEW.quantity 
    WHERE id = NEW.product_id;
  END;
''').go();

// Now whenever an order item is inserted:
// INSERT INTO order_items (order_id, product_id, quantity) VALUES (1, 1, 5);
// The trigger automatically updates the product's stock!
```

> **What's happening here?**
> - **Event** – AFTER INSERT on order_items table
> - **Action** – UPDATE products stock
> - **NEW** – Refers to the newly inserted row
> - **Automatic** – Trigger runs automatically

---

# Why does it exist?

- **Data Integrity** – Automatically maintain data consistency
- **Audit Logging** – Track changes to important tables
- **Business Rules** – Enforce complex business logic
- **Automation** – Reduce manual application code
- **Performance** – Database-level operations
- **Consistency** – Ensure consistent behavior

---

# Creating Triggers

> **Different types of triggers**

## Basic Insert Trigger

```dart
// 👇 Trigger on INSERT
await db.customSelect('''
  CREATE TRIGGER log_user_insert
  AFTER INSERT ON users
  FOR EACH ROW
  BEGIN
    INSERT INTO user_audit (user_id, action, timestamp)
    VALUES (NEW.id, 'INSERT', datetime('now'));
  END;
''').go();

// Now every user insert is automatically logged
```

## Basic Update Trigger

```dart
// 👇 Trigger on UPDATE
await db.customSelect('''
  CREATE TRIGGER log_user_update
  AFTER UPDATE ON users
  FOR EACH ROW
  BEGIN
    INSERT INTO user_audit (user_id, action, timestamp)
    VALUES (OLD.id, 'UPDATE', datetime('now'));
  END;
''').go();

// Now every user update is automatically logged
```

## Basic Delete Trigger

```dart
// 👇 Trigger on DELETE
await db.customSelect('''
  CREATE TRIGGER log_user_delete
  AFTER DELETE ON users
  FOR EACH ROW
  BEGIN
    INSERT INTO user_audit (user_id, action, timestamp)
    VALUES (OLD.id, 'DELETE', datetime('now'));
  END;
''').go();

// Now every user deletion is automatically logged
```

---

# Advanced Triggers

> **Complex trigger patterns**

## Trigger with Conditions

```dart
// 👇 Conditional trigger
await db.customSelect('''
  CREATE TRIGGER update_stock_after_order
  AFTER INSERT ON order_items
  FOR EACH ROW
  WHEN NEW.quantity > 0
  BEGIN
    UPDATE products 
    SET stock = stock - NEW.quantity,
        updated_at = datetime('now')
    WHERE id = NEW.product_id;
    
    -- Log the change
    INSERT INTO inventory_log (product_id, quantity_change, timestamp)
    VALUES (NEW.product_id, -NEW.quantity, datetime('now'));
  END;
''').go();
```

## Trigger with Multiple Operations

```dart
// 👇 Trigger with multiple statements
await db.customSelect('''
  CREATE TRIGGER order_processing
  AFTER INSERT ON orders
  FOR EACH ROW
  BEGIN
    -- Update user stats
    UPDATE users 
    SET total_orders = total_orders + 1,
        last_order_date = datetime('now')
    WHERE id = NEW.user_id;
    
    -- Record in history
    INSERT INTO order_history (order_id, status, timestamp)
    VALUES (NEW.id, 'created', datetime('now'));
    
    -- Send notification (via application logic)
    -- Note: Can't send notifications from SQL, but can set a flag
    INSERT INTO notifications (user_id, type, message, timestamp)
    VALUES (NEW.user_id, 'order_created', 'Order placed', datetime('now'));
  END;
''').go();
```

## BEFORE vs AFTER Triggers

```dart
// 👇 BEFORE trigger (validate before insert)
await db.customSelect('''
  CREATE TRIGGER validate_user_age
  BEFORE INSERT ON users
  FOR EACH ROW
  WHEN NEW.age < 0 OR NEW.age > 150
  BEGIN
    SELECT RAISE(ABORT, 'Invalid age');
  END;
''').go();

// 👇 AFTER trigger (audit after insert)
await db.customSelect('''
  CREATE TRIGGER audit_user_insert
  AFTER INSERT ON users
  FOR EACH ROW
  BEGIN
    INSERT INTO user_audit (user_id, action, old_value, new_value, timestamp)
    VALUES (NEW.id, 'INSERT', NULL, 'user created', datetime('now'));
  END;
''').go();
```

---

# Real-World Example

> **Complete e-commerce trigger system**

```dart
// lib/database/trigger_service.dart
import 'package:drift/drift.dart';

class TriggerService {
  final AppDatabase db;
  
  TriggerService(this.db);

  // ==================== INITIALIZE TRIGGERS ====================
  
  Future<void> initializeTriggers() async {
    await _createUserTriggers();
    await _createOrderTriggers();
    await _createOrderItemTriggers();
    await _createProductTriggers();
    await _createInventoryTriggers();
  }

  // ==================== USER TRIGGERS ====================
  
  Future<void> _createUserTriggers() async {
    // 👇 Audit user creation
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS audit_user_insert
      AFTER INSERT ON users
      FOR EACH ROW
      BEGIN
        INSERT INTO user_audit (user_id, action, old_value, new_value, timestamp)
        VALUES (NEW.id, 'INSERT', NULL, 'user created', datetime('now'));
      END;
    ''').go();
    
    // 👇 Audit user updates
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS audit_user_update
      AFTER UPDATE ON users
      FOR EACH ROW
      BEGIN
        INSERT INTO user_audit (user_id, action, old_value, new_value, timestamp)
        VALUES (OLD.id, 'UPDATE', 'old data', 'new data', datetime('now'));
      END;
    ''').go();
    
    // 👇 Audit user deletion
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS audit_user_delete
      AFTER DELETE ON users
      FOR EACH ROW
      BEGIN
        INSERT INTO user_audit (user_id, action, old_value, new_value, timestamp)
        VALUES (OLD.id, 'DELETE', 'user deleted', NULL, datetime('now'));
      END;
    ''').go();
  }

  // ==================== ORDER TRIGGERS ====================
  
  Future<void> _createOrderTriggers() async {
    // 👇 Process new orders
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS process_new_order
      AFTER INSERT ON orders
      FOR EACH ROW
      WHEN NEW.total > 0
      BEGIN
        -- Update user stats
        UPDATE users 
        SET total_orders = total_orders + 1,
            total_spent = total_spent + NEW.total,
            last_order_date = datetime('now')
        WHERE id = NEW.user_id;
        
        -- Record order history
        INSERT INTO order_history (order_id, status, action, timestamp)
        VALUES (NEW.id, 'created', 'Order placed', datetime('now'));
        
        -- Create notification
        INSERT INTO notifications (user_id, type, message, timestamp)
        VALUES (NEW.user_id, 'order_created', 'Order #' || NEW.order_number || ' placed', datetime('now'));
      END;
    ''').go();
    
    // 👇 Handle order status changes
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS order_status_update
      AFTER UPDATE OF status ON orders
      FOR EACH ROW
      WHEN OLD.status != NEW.status
      BEGIN
        INSERT INTO order_history (order_id, status, action, timestamp)
        VALUES (NEW.id, NEW.status, 'Status changed from ' || OLD.status || ' to ' || NEW.status, datetime('now'));
        
        -- Handle specific status transitions
        UPDATE order_items 
        SET status = NEW.status 
        WHERE order_id = NEW.id;
      END;
    ''').go();
    
    // 👇 Order cancellation cleanup
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS cancel_order_cleanup
      AFTER UPDATE OF status ON orders
      FOR EACH ROW
      WHEN NEW.status = 'cancelled' AND OLD.status != 'cancelled'
      BEGIN
        -- Restore stock
        UPDATE products 
        SET stock = stock + (
          SELECT oi.quantity 
          FROM order_items oi 
          WHERE oi.order_id = NEW.id AND oi.product_id = products.id
        ),
        updated_at = datetime('now')
        WHERE id IN (
          SELECT product_id FROM order_items WHERE order_id = NEW.id
        );
        
        -- Refund notification
        INSERT INTO notifications (user_id, type, message, timestamp)
        VALUES (NEW.user_id, 'order_cancelled', 'Order #' || NEW.order_number || ' cancelled', datetime('now'));
      END;
    ''').go();
  }

  // ==================== ORDER ITEM TRIGGERS ====================
  
  Future<void> _createOrderItemTriggers() async {
    // 👇 Update inventory when items are ordered
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS update_inventory_after_item
      AFTER INSERT ON order_items
      FOR EACH ROW
      BEGIN
        -- Update product stock
        UPDATE products 
        SET stock = stock - NEW.quantity,
            updated_at = datetime('now')
        WHERE id = NEW.product_id;
        
        -- Log inventory change
        INSERT INTO inventory_log (product_id, change_type, quantity, timestamp)
        VALUES (NEW.product_id, 'order', -NEW.quantity, datetime('now'));
        
        -- Check if product is low on stock
        INSERT INTO inventory_alerts (product_id, alert_type, message, timestamp)
        SELECT 
          NEW.product_id, 
          'low_stock', 
          'Product is low on stock. Quantity: ' || (SELECT stock FROM products WHERE id = NEW.product_id),
          datetime('now')
        WHERE (SELECT stock FROM products WHERE id = NEW.product_id) < 10;
      END;
    ''').go();
    
    // 👇 Handle order item deletion
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS restore_stock_on_item_delete
      AFTER DELETE ON order_items
      FOR EACH ROW
      BEGIN
        UPDATE products 
        SET stock = stock + OLD.quantity,
            updated_at = datetime('now')
        WHERE id = OLD.product_id;
        
        INSERT INTO inventory_log (product_id, change_type, quantity, timestamp)
        VALUES (OLD.product_id, 'restore', OLD.quantity, datetime('now'));
      END;
    ''').go();
  }

  // ==================== PRODUCT TRIGGERS ====================
  
  Future<void> _createProductTriggers() async {
    // 👇 Product creation audit
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS product_created
      AFTER INSERT ON products
      FOR EACH ROW
      BEGIN
        INSERT INTO product_audit (product_id, action, old_value, new_value, timestamp)
        VALUES (NEW.id, 'INSERT', NULL, 'product created', datetime('now'));
      END;
    ''').go();
    
    // 👇 Product price change audit
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS product_price_audit
      AFTER UPDATE OF price ON products
      FOR EACH ROW
      WHEN OLD.price != NEW.price
      BEGIN
        INSERT INTO product_audit (product_id, action, old_value, new_value, timestamp)
        VALUES (NEW.id, 'PRICE_CHANGE', OLD.price, NEW.price, datetime('now'));
        
        -- Update order items with new price? 
        -- Better to keep historical price in order_items
      END;
    ''').go();
  }

  // ==================== INVENTORY TRIGGERS ====================
  
  Future<void> _createInventoryTriggers() async {
    // 👇 Low stock alert
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS low_stock_alert
      AFTER UPDATE OF stock ON products
      FOR EACH ROW
      WHEN NEW.stock < 10 AND NEW.stock > 0 AND OLD.stock >= 10
      BEGIN
        INSERT INTO inventory_alerts (product_id, alert_type, message, timestamp)
        VALUES (NEW.id, 'low_stock', 'Product is low on stock. Quantity: ' || NEW.stock, datetime('now'));
      END;
    ''').go();
    
    // 👇 Out of stock alert
    await db.customSelect('''
      CREATE TRIGGER IF NOT EXISTS out_of_stock_alert
      AFTER UPDATE OF stock ON products
      FOR EACH ROW
      WHEN NEW.stock = 0 AND OLD.stock > 0
      BEGIN
        INSERT INTO inventory_alerts (product_id, alert_type, message, timestamp)
        VALUES (NEW.id, 'out_of_stock', 'Product is out of stock!', datetime('now'));
      END;
    ''').go();
  }

  // ==================== DROP TRIGGERS ====================
  
  Future<void> dropAllTriggers() async {
    await db.customSelect('DROP TRIGGER IF EXISTS audit_user_insert').go();
    await db.customSelect('DROP TRIGGER IF EXISTS audit_user_update').go();
    await db.customSelect('DROP TRIGGER IF EXISTS audit_user_delete').go();
    await db.customSelect('DROP TRIGGER IF EXISTS process_new_order').go();
    await db.customSelect('DROP TRIGGER IF EXISTS order_status_update').go();
    await db.customSelect('DROP TRIGGER IF EXISTS cancel_order_cleanup').go();
    await db.customSelect('DROP TRIGGER IF EXISTS update_inventory_after_item').go();
    await db.customSelect('DROP TRIGGER IF EXISTS restore_stock_on_item_delete').go();
    await db.customSelect('DROP TRIGGER IF EXISTS product_created').go();
    await db.customSelect('DROP TRIGGER IF EXISTS product_price_audit').go();
    await db.customSelect('DROP TRIGGER IF EXISTS low_stock_alert').go();
    await db.customSelect('DROP TRIGGER IF EXISTS out_of_stock_alert').go();
  }

  // ==================== VIEW TRIGGER INFO ====================
  
  Future<List<TriggerInfo>> getTriggerInfo() async {
    final results = await db.customSelect('''
      SELECT 
        name,
        tbl_name as table_name,
        sql,
        type
      FROM sqlite_master 
      WHERE type = 'trigger'
      ORDER BY name
    ''').get();
    
    return results.map((row) {
      return TriggerInfo(
        name: row.data['name'] as String,
        tableName: row.data['table_name'] as String,
        sql: row.data['sql'] as String,
        type: row.data['type'] as String,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class TriggerInfo {
  final String name;
  final String tableName;
  final String sql;
  final String type;
  
  TriggerInfo({
    required this.name,
    required this.tableName,
    required this.sql,
    required this.type,
  });
}
```

---

# Trigger Best Practices

- **Use `IF NOT EXISTS`** – Prevent errors
- **Keep triggers simple** – Avoid complex logic
- **Document triggers** – Explain what they do
- **Test triggers thoroughly** – Verify behavior
- **Monitor trigger performance** – Can be slow
- **Use BEFORE triggers for validation** – Prevent invalid data
- **Use AFTER triggers for actions** – Audit, notifications
- **Drop triggers when no longer needed** – Clean up

---

# Trigger Use Cases

| Use Case | Trigger Type | Example |
|----------|-------------|---------|
| **Audit Logging** | AFTER INSERT/UPDATE/DELETE | Log all changes |
| **Data Validation** | BEFORE INSERT/UPDATE | Validate age range |
| **Stock Management** | AFTER INSERT/UPDATE/DELETE | Update inventory |
| **Cascading Actions** | AFTER DELETE | Clean up related data |
| **Notifications** | AFTER INSERT | Send notifications |
| **Summary Updates** | AFTER INSERT/UPDATE | Update user totals |

---

# Common Mistakes

## Mistake 1: Triggers causing recursion

Wrong:
```dart
// 🚫 Recursive trigger (updates product, which triggers again)
CREATE TRIGGER update_product
AFTER UPDATE ON products
BEGIN
  UPDATE products SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

Correct:
```dart
// ✅ Prevent recursion with conditions
CREATE TRIGGER update_product
AFTER UPDATE ON products
WHEN NEW.updated_at IS NULL OR OLD.updated_at != NEW.updated_at
BEGIN
  UPDATE products SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

## Mistake 2: Triggers with high complexity

Wrong:
```dart
// 🚫 Too complex for trigger
CREATE TRIGGER complex_process
AFTER INSERT ON orders
BEGIN
  -- 50+ lines of SQL
END;
```

Correct:
```dart
// ✅ Keep triggers simple
CREATE TRIGGER simple_process
AFTER INSERT ON orders
BEGIN
  -- Minimal SQL
  -- Application logic handles complexity
END;
```

## Mistake 3: Not handling BEFORE vs AFTER correctly

Wrong:
```dart
// 🚫 BEFORE trigger for logging
CREATE TRIGGER log_insert
BEFORE INSERT ON users
BEGIN
  INSERT INTO audit (action) VALUES ('insert');
END;
```

Correct:
```dart
// ✅ AFTER trigger for logging
CREATE TRIGGER log_insert
AFTER INSERT ON users
BEGIN
  INSERT INTO audit (action, timestamp) 
  VALUES ('insert', datetime('now'));
END;
```

---

# Summary

| Trigger Type | Purpose | Example |
|--------------|---------|---------|
| **BEFORE INSERT** | Validate data | Check constraints |
| **AFTER INSERT** | Audit/log | Record creation |
| **BEFORE UPDATE** | Validate changes | Check price changes |
| **AFTER UPDATE** | Audit/log | Record updates |
| **BEFORE DELETE** | Validate deletion | Check dependencies |
| **AFTER DELETE** | Cleanup | Remove related data |

---

# Next Steps

Now you understand triggers, let's dive deeper:

- [CTEs](link) – Common Table Expressions
- [Window Functions](link) – Advanced analytics
- [SQLite Functions](link) – SQLite function library

---

# Did You Know?

- **Triggers are atomic** – Run as part of the transaction

- **Triggers can be recursive** – But watch for loops

- **Triggers can be conditionally executed** – Using WHEN

- **Triggers are database-level** – Work even if application changes

- **Triggers are powerful** – For data integrity

- **Triggers are production-tested** – Used in large systems

- **Triggers can slow writes** – Additional operations

- **Triggers are supported in SQLite** – Full support

---

