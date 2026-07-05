## Filtering

**Mastering WHERE clauses in Drift queries**

---

# What is it?

**Filtering** is the process of narrowing down query results using WHERE clauses. Drift provides a rich set of operators and methods to build complex filter conditions with type safety and IDE support. You can combine multiple conditions, use logical operators, and apply various comparison types to get exactly the data you need.

> **Think of Filtering like "sorting through a deck of cards"** – you pull out only the cards that meet your criteria (red cards, face cards, hearts, etc.) and ignore the rest.

```dart
// 👇 Basic filtering
final adultUsers = await (select(users)
  ..where((u) => u.age > const Variable(18)))
  .get();

// 👇 Multiple conditions (AND)
final activeAdults = await (select(users)
  ..where((u) => u.age > const Variable(18))
  ..where((u) => u.isActive.equals(true)))
  .get();

// 👇 Complex filtering with OR
final specialUsers = await (select(users)
  ..where((u) => 
    u.age < const Variable(13) |
    u.age > const Variable(65)
  ))
  .get();
```

> **What's happening here?**
> - **WHERE clauses** – Filter rows based on conditions
> - **Type-safe operators** – `>`, `<`, `=`, `!=`, `LIKE`, etc.
> - **Logical operators** – `&` (AND), `|` (OR)
> - **Null handling** – `isNull()`, `isNotNull()`
> - **Chainable** – Multiple `where()` calls combine with AND

---

# Why does it exist?

- **Data Precision** – Get exactly what you need
- **Performance** – Reduce data transfer and processing
- **User Experience** – Enable search and filter functionality
- **Business Logic** – Enforce business rules in queries
- **Data Analysis** – Analyze specific subsets of data
- **Type Safety** – Compile-time validation of conditions

---

# Basic Filtering

> **Simple WHERE clauses**

## Single Condition

```dart
// 👇 Equal to
final users = await (select(users)
  ..where((u) => u.name.equals('John')))
  .get();

// 👇 Greater than
final adults = await (select(users)
  ..where((u) => u.age > const Variable(18)))
  .get();

// 👇 Less than
final minors = await (select(users)
  ..where((u) => u.age < const Variable(18)))
  .get();

// 👇 Not equal
final nonAdmins = await (select(users)
  ..where((u) => u.isAdmin.equals(false)))
  .get();
```

## Multiple Conditions (AND)

```dart
// 👇 Chain where clauses
final activeAdults = await (select(users)
  ..where((u) => u.age > const Variable(18))
  ..where((u) => u.isActive.equals(true)))
  .get();

// 👇 Use & operator
final activeAdults2 = await (select(users)
  ..where((u) => 
    u.age > const Variable(18) &
    u.isActive.equals(true)
  ))
  .get();
```

---

# Comparison Operators

> **All comparison operators**

## Equality Operators

```dart
// 👇 Equals
..where((u) => u.name.equals('John'))

// 👇 Not equals
..where((u) => u.name.isNotValue('John'))

// 👇 Case-insensitive equals
..where((u) => u.name.equals('john', caseSensitive: false))
```

## Numeric Comparisons

```dart
// 👇 Greater than
..where((u) => u.age > const Variable(18))

// 👇 Greater than or equal
..where((u) => u.age.isBiggerOrEqualValue(18))

// 👇 Less than
..where((u) => u.age < const Variable(18))

// 👇 Less than or equal
..where((u) => u.age.isSmallerOrEqualValue(18))

// 👇 Between (inclusive)
..where((u) => u.age.isBetweenValues(18, 65))

// 👇 Between (exclusive)
..where((u) => u.age > const Variable(18) & u.age < const Variable(65))
```

---

# Logical Operators

> **Combining conditions with AND, OR, NOT**

## AND Operator

```dart
// 👇 Using & (AND)
..where((u) => 
  u.age > const Variable(18) &
  u.isActive.equals(true) &
  u.isVerified.equals(true)
)

// 👇 Multiple where clauses (all AND)
..where((u) => u.age > const Variable(18))
..where((u) => u.isActive.equals(true))
..where((u) => u.isVerified.equals(true))
```

## OR Operator

```dart
// 👇 Using | (OR)
..where((u) => 
  u.age < const Variable(13) |
  u.age > const Variable(65)
)

// 👇 Complex OR with multiple conditions
..where((u) => 
  u.status.equals('premium') |
  u.status.equals('vip') |
  u.isAdmin.equals(true)
)
```

## NOT Operator

```dart
// 👇 Using .not()
..where((u) => u.isActive.equals(true).not())

// 👇 NOT with numeric
..where((u) => u.age > const Variable(18).not())

// 👇 NOT IN
..where((u) => u.id.isNotIn([1, 2, 3]))

// 👇 IS NOT NULL
..where((u) => u.age.isNotNull())
```

## Complex Combinations

```dart
// 👇 (Age > 18 AND Active) OR Admin
..where((u) => 
  (u.age > const Variable(18) & u.isActive.equals(true)) |
  u.isAdmin.equals(true)
)

// 👇 NOT (Age < 18 OR Inactive)
..where((u) => 
  (u.age < const Variable(18) | u.isActive.equals(false)).not()
)

// 👇 With parentheses for clarity
..where((u) => 
  (
    u.age > const Variable(18) & 
    u.age < const Variable(65)
  ) &
  (
    u.isActive.equals(true) |
    u.isVerified.equals(true)
  )
)
```

---

# String Filtering

> **Advanced string comparisons**

## LIKE Patterns

```dart
// 👇 Contains pattern
..where((u) => u.name.like('%John%'))

// 👇 Starts with
..where((u) => u.name.like('John%'))

// 👇 Ends with
..where((u) => u.name.like('%Doe'))

// 👇 Exact pattern
..where((u) => u.name.like('John Doe'))

// 👇 Case-insensitive LIKE
..where((u) => u.name.like('%john%', caseSensitive: false))
```

## Helper Methods

```dart
// 👇 Contains (LIKE %value%)
..where((u) => u.name.contains('oh'))

// 👇 Starts with (LIKE value%)
..where((u) => u.name.startsWith('Jo'))

// 👇 Ends with (LIKE %value)
..where((u) => u.name.endsWith('hn'))

// 👇 Case-insensitive versions
..where((u) => u.name.contains('JOHN', caseSensitive: false))
..where((u) => u.name.startsWith('JO', caseSensitive: false))
..where((u) => u.name.endsWith('HN', caseSensitive: false))
```

## Pattern with Special Characters

```dart
// 👇 Escape special characters
..where((u) => u.name.like(r'%100%'))

// 👇 Multiple patterns
..where((u) => 
  u.name.like('%John%') |
  u.name.like('%Jane%')
)
```

---

# Null Checks

> **Filtering NULL values**

## IS NULL / IS NOT NULL

```dart
// 👇 IS NULL
final usersWithoutAge = await (select(users)
  ..where((u) => u.age.isNull()))
  .get();

// 👇 IS NOT NULL
final usersWithAge = await (select(users)
  ..where((u) => u.age.isNotNull()))
  .get();

// 👇 Combine with other conditions
final adultsWithAge = await (select(users)
  ..where((u) => u.age.isNotNull())
  ..where((u) => u.age > const Variable(18)))
  .get();
```

---

# IN and NOT IN

> **Filtering against lists**

## Basic IN

```dart
// 👇 IN list of values
final users = await (select(users)
  ..where((u) => u.id.isIn([1, 2, 3, 4, 5])))
  .get();

// 👇 IN list of strings
final users = await (select(users)
  ..where((u) => u.status.isIn(['active', 'premium'])))
  .get();

// 👇 NOT IN
final users = await (select(users)
  ..where((u) => u.id.isNotIn([1, 2, 3])))
  .get();

// 👇 IN from subquery (using custom SQL)
final usersWithOrders = await customSelect('''
  SELECT * FROM users 
  WHERE id IN (SELECT DISTINCT user_id FROM orders)
''').get();
```

---

# Dynamic Filtering

> **Building filters conditionally**

## Basic Dynamic Filter

```dart
Future<List<User>> filterUsers({
  String? name,
  int? minAge,
  int? maxAge,
  bool? isActive,
  bool? isVerified,
}) async {
  final query = select(users);
  
  // 👇 Add filters conditionally
  if (name != null && name.isNotEmpty) {
    query.where((u) => u.name.like('%$name%'));
  }
  
  if (minAge != null) {
    query.where((u) => u.age > const Variable(minAge));
  }
  
  if (maxAge != null) {
    query.where((u) => u.age < const Variable(maxAge));
  }
  
  if (isActive != null) {
    query.where((u) => u.isActive.equals(isActive));
  }
  
  if (isVerified != null) {
    query.where((u) => u.isVerified.equals(isVerified));
  }
  
  return await query.get();
}

// Usage
final activeUsers = await filterUsers(
  isActive: true,
  minAge: 18,
);

final verifiedAdults = await filterUsers(
  isVerified: true,
  minAge: 18,
  maxAge: 65,
);
```

---

# Real-World Example

> **Complete e-commerce filtering system**

```dart
// lib/database/filter_service.dart
import 'package:drift/drift.dart';

class FilterService {
  final AppDatabase db;
  
  FilterService(this.db);
  
  // ==================== USER FILTERS ====================
  
  // 👇 Advanced user filter
  Future<List<User>> filterUsers({
    String? search,
    int? minAge,
    int? maxAge,
    bool? isActive,
    bool? isVerified,
    bool? isAdmin,
    String? status,
    DateTime? fromDate,
    DateTime? toDate,
    List<int>? excludeIds,
  }) async {
    final query = db.select(db.users);
    
    // Search filter
    if (search != null && search.isNotEmpty) {
      query.where((u) => 
        u.name.like('%$search%') |
        u.email.like('%$search%')
      );
    }
    
    // Age range
    if (minAge != null) {
      query.where((u) => u.age > const Variable(minAge));
    }
    if (maxAge != null) {
      query.where((u) => u.age < const Variable(maxAge));
    }
    
    // Boolean filters
    if (isActive != null) {
      query.where((u) => u.isActive.equals(isActive));
    }
    if (isVerified != null) {
      query.where((u) => u.isVerified.equals(isVerified));
    }
    if (isAdmin != null) {
      query.where((u) => u.isAdmin.equals(isAdmin));
    }
    
    // Status filter
    if (status != null && status.isNotEmpty) {
      query.where((u) => u.status.equals(status));
    }
    
    // Date range
    if (fromDate != null) {
      query.where((u) => u.createdAt > const Variable(fromDate));
    }
    if (toDate != null) {
      query.where((u) => u.createdAt < const Variable(toDate));
    }
    
    // Exclude users
    if (excludeIds != null && excludeIds.isNotEmpty) {
      query.where((u) => u.id.isNotIn(excludeIds));
    }
    
    return await query.get();
  }
  
  // ==================== PRODUCT FILTERS ====================
  
  // 👇 Advanced product filter
  Future<List<Product>> filterProducts({
    String? search,
    String? category,
    double? minPrice,
    double? maxPrice,
    int? minStock,
    int? maxStock,
    bool? isActive,
    List<String>? tags,
    List<String>? excludeCategories,
    DateTime? fromDate,
    DateTime? toDate,
  }) async {
    final query = db.select(db.products);
    
    // Search
    if (search != null && search.isNotEmpty) {
      query.where((p) => 
        p.name.like('%$search%') |
        p.sku.like('%$search%') |
        p.description.like('%$search%')
      );
    }
    
    // Category
    if (category != null && category.isNotEmpty) {
      query.where((p) => p.category.equals(category));
    }
    
    // Price range
    if (minPrice != null) {
      query.where((p) => p.price > const Variable(minPrice));
    }
    if (maxPrice != null) {
      query.where((p) => p.price < const Variable(maxPrice));
    }
    
    // Stock range
    if (minStock != null) {
      query.where((p) => p.stock > const Variable(minStock));
    }
    if (maxStock != null) {
      query.where((p) => p.stock < const Variable(maxStock));
    }
    
    // Active status
    if (isActive != null) {
      query.where((p) => p.isActive.equals(isActive));
    }
    
    // Tags (using JSON or custom logic)
    if (tags != null && tags.isNotEmpty) {
      for (final tag in tags) {
        query.where((p) => p.tags.like('%$tag%'));
      }
    }
    
    // Exclude categories
    if (excludeCategories != null && excludeCategories.isNotEmpty) {
      query.where((p) => p.category.isNotIn(excludeCategories));
    }
    
    // Date range
    if (fromDate != null) {
      query.where((p) => p.createdAt > const Variable(fromDate));
    }
    if (toDate != null) {
      query.where((p) => p.createdAt < const Variable(toDate));
    }
    
    return await query.get();
  }
  
  // ==================== ORDER FILTERS ====================
  
  // 👇 Advanced order filter
  Future<List<Order>> filterOrders({
    int? userId,
    String? status,
    String? paymentStatus,
    String? paymentMethod,
    double? minTotal,
    double? maxTotal,
    DateTime? fromDate,
    DateTime? toDate,
    bool? isPaid,
    bool? isShipped,
    bool? isDelivered,
    List<String>? excludeStatuses,
  }) async {
    final query = db.select(db.orders);
    
    if (userId != null) {
      query.where((o) => o.userId.equals(userId));
    }
    
    if (status != null && status.isNotEmpty) {
      query.where((o) => o.status.equals(status));
    }
    
    if (paymentStatus != null && paymentStatus.isNotEmpty) {
      query.where((o) => o.paymentStatus.equals(paymentStatus));
    }
    
    if (paymentMethod != null && paymentMethod.isNotEmpty) {
      query.where((o) => o.paymentMethod.equals(paymentMethod));
    }
    
    if (minTotal != null) {
      query.where((o) => o.total > const Variable(minTotal));
    }
    if (maxTotal != null) {
      query.where((o) => o.total < const Variable(maxTotal));
    }
    
    if (fromDate != null) {
      query.where((o) => o.orderDate > const Variable(fromDate));
    }
    if (toDate != null) {
      query.where((o) => o.orderDate < const Variable(toDate));
    }
    
    if (isPaid != null) {
      query.where((o) => o.isPaid.equals(isPaid));
    }
    if (isShipped != null) {
      query.where((o) => o.isShipped.equals(isShipped));
    }
    if (isDelivered != null) {
      query.where((o) => o.isDelivered.equals(isDelivered));
    }
    
    if (excludeStatuses != null && excludeStatuses.isNotEmpty) {
      query.where((o) => o.status.isNotIn(excludeStatuses));
    }
    
    return await query.get();
  }
  
  // ==================== COMPLEX FILTERS ====================
  
  // 👇 Filter with multiple conditions and sorting
  Future<List<User>> advancedUserSearch({
    String? query,
    int? minAge,
    int? maxAge,
    bool? isActive,
    String? sortBy,
    bool ascending = true,
    int? limit,
  }) async {
    final queryBuilder = db.select(db.users);
    
    // Build WHERE clause
    if (query != null && query.isNotEmpty) {
      queryBuilder.where((u) => 
        u.name.like('%$query%') |
        u.email.like('%$query%')
      );
    }
    
    if (minAge != null) {
      queryBuilder.where((u) => u.age > const Variable(minAge));
    }
    if (maxAge != null) {
      queryBuilder.where((u) => u.age < const Variable(maxAge));
    }
    if (isActive != null) {
      queryBuilder.where((u) => u.isActive.equals(isActive));
    }
    
    // Apply sorting
    if (sortBy != null) {
      final mode = ascending ? OrderingMode.asc : OrderingMode.desc;
      switch (sortBy) {
        case 'name':
          queryBuilder.orderBy([(u) => OrderingTerm(expression: u.name, mode: mode)]);
          break;
        case 'age':
          queryBuilder.orderBy([(u) => OrderingTerm(expression: u.age, mode: mode)]);
          break;
        case 'createdAt':
          queryBuilder.orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: mode)]);
          break;
        default:
          queryBuilder.orderBy([(u) => OrderingTerm(expression: u.id, mode: mode)]);
      }
    }
    
    // Apply limit
    if (limit != null) {
      queryBuilder.limit(limit);
    }
    
    return await queryBuilder.get();
  }
  
  // ==================== FILTER WITH COUNT ====================
  
  // 👇 Filter and get count
  Future<FilterResult<User>> filterUsersWithCount({
    String? search,
    int? minAge,
    int? maxAge,
    bool? isActive,
    int? page,
    int? pageSize,
  }) async {
    final query = db.select(db.users);
    
    // Apply all filters
    if (search != null && search.isNotEmpty) {
      query.where((u) => u.name.like('%$search%'));
    }
    if (minAge != null) {
      query.where((u) => u.age > const Variable(minAge));
    }
    if (maxAge != null) {
      query.where((u) => u.age < const Variable(maxAge));
    }
    if (isActive != null) {
      query.where((u) => u.isActive.equals(isActive));
    }
    
    // Get total count
    final total = await query.count();
    
    // Get paginated results
    final items = await (query
      ..orderBy([(u) => OrderingTerm(expression: u.id)])
      ..limit(pageSize ?? 10, offset: (page ?? 0) * (pageSize ?? 10)))
      .get();
    
    return FilterResult(
      items: items,
      total: total,
      hasMore: (page ?? 0) * (pageSize ?? 10) + items.length < total,
    );
  }
}

// ==================== DATA CLASSES ====================

class FilterResult<T> {
  final List<T> items;
  final int total;
  final bool hasMore;
  
  FilterResult({
    required this.items,
    required this.total,
    this.hasMore = false,
  });
}
```

```dart
// lib/ui/pages/filter_page.dart
class FilterPage extends StatefulWidget {
  final FilterService filterService;
  
  const FilterPage({required this.filterService});
  
  @override
  _FilterPageState createState() => _FilterPageState();
}

class _FilterPageState extends State<FilterPage> {
  final _searchController = TextEditingController();
  int? _minAge;
  int? _maxAge;
  bool? _isActive;
  List<User> _users = [];
  bool _isLoading = false;
  bool _showFilters = false;
  
  Future<void> _applyFilters() async {
    setState(() => _isLoading = true);
    
    try {
      final users = await widget.filterService.filterUsers(
        search: _searchController.text,
        minAge: _minAge,
        maxAge: _maxAge,
        isActive: _isActive,
      );
      
      setState(() {
        _users = users;
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
      // Handle error
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Filter Users'),
        actions: [
          IconButton(
            icon: Icon(Icons.filter_list),
            onPressed: () => setState(() => _showFilters = !_showFilters),
          ),
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _searchController,
                    decoration: InputDecoration(
                      hintText: 'Search users...',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (_) => _applyFilters(),
                  ),
                ),
                SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _applyFilters,
                  child: Text('Search'),
                ),
              ],
            ),
          ),
          if (_showFilters) ...[
            Container(
              padding: EdgeInsets.all(16),
              color: Colors.grey[100],
              child: Column(
                children: [
                  Row(
                    children: [
                      Expanded(
                        child: TextFormField(
                          decoration: InputDecoration(labelText: 'Min Age'),
                          keyboardType: TextInputType.number,
                          onChanged: (v) => _minAge = int.tryParse(v),
                        ),
                      ),
                      SizedBox(width: 16),
                      Expanded(
                        child: TextFormField(
                          decoration: InputDecoration(labelText: 'Max Age'),
                          keyboardType: TextInputType.number,
                          onChanged: (v) => _maxAge = int.tryParse(v),
                        ),
                      ),
                    ],
                  ),
                  SwitchListTile(
                    title: Text('Active Users'),
                    value: _isActive ?? false,
                    onChanged: (value) {
                      setState(() {
                        _isActive = _isActive == null ? true : null;
                      });
                    },
                  ),
                ],
              ),
            ),
          ],
          if (_isLoading)
            Center(child: CircularProgressIndicator())
          else
            Expanded(
              child: ListView.builder(
                itemCount: _users.length,
                itemBuilder: (context, index) {
                  final user = _users[index];
                  return ListTile(
                    title: Text(user.name),
                    subtitle: Text('Age: ${user.age ?? "N/A"}'),
                    trailing: Icon(
                      user.isActive ? Icons.check_circle : Icons.cancel,
                      color: user.isActive ? Colors.green : Colors.red,
                    ),
                  );
                },
              ),
            ),
        ],
      ),
    );
  }
}
```

---

# Best Practices

- **Use indexes** – On frequently filtered columns
- **Combine conditions** – With AND/OR for complex filters
- **Use variables** – Prevent SQL injection
- **Use `.isNotNull()`** – For optional fields
- **Use `.isIn()`** – For multiple values
- **Test performance** – For complex filters
- **Use pagination** – With filtered results
- **Chain conditions** – For better readability

---

# Common Mistakes

## Mistake 1: Forgetting `const Variable()`

Wrong:
```dart
// 🚫 Error: int can't be used in Expression
..where((u) => u.age > 18)
```

Correct:
```dart
// ✅ Use Variable
..where((u) => u.age > const Variable(18))
```

## Mistake 2: Not using `.isNotNull()`

Wrong:
```dart
// 🚫 Won't work correctly for null
..where((u) => u.age > const Variable(18))
// This excludes null values automatically, but if you want to include them...
```

Correct:
```dart
// ✅ Handle null explicitly
..where((u) => u.age.isNotNull())
..where((u) => u.age > const Variable(18))
```

## Mistake 3: Using OR incorrectly

Wrong:
```dart
// 🚫 Both conditions must be met (AND)
..where((u) => u.age > const Variable(18))
..where((u) => u.isActive.equals(true))
```

Correct:
```dart
// ✅ Use | for OR
..where((u) => 
  u.age > const Variable(18) |
  u.isActive.equals(true)
)
```

---

# Summary

| Operator | Syntax | Purpose |
|----------|--------|---------|
| **Equals** | `.equals(value)` | Exact match |
| **Not Equals** | `.isNotValue(value)` | Exclude match |
| **Greater Than** | `>` or `.isBiggerOrEqualValue()` | Numeric comparison |
| **Like** | `.like(pattern)` | Pattern matching |
| **Contains** | `.contains(text)` | Substring |
| **In** | `.isIn(list)` | List match |
| **Null** | `.isNull()` / `.isNotNull()` | Null check |
| **AND** | `&` or chaining | Multiple conditions |
| **OR** | `\|` | Either condition |

---

# Next Steps

Now you understand filtering, let's dive deeper:

- [Ordering](link) – Sorting query results
- [Limiting](link) – Pagination and limiting
- [Distinct](link) – Getting unique values

---

# Did You Know?

- **Filters are applied at the database level** – More efficient

- **LIKE with wildcards is case-sensitive** – Use with caseSensitive: false

- **Multiple where() calls are ANDed** – Not OR

- **Indexes speed up filtered queries** – Dramatically

- **NULL values are excluded by default** – In comparisons

- **Variables prevent SQL injection** – Always use them

- **Complex filters can use parentheses** – For grouping

- **Filters can be dynamic** – Built conditionally

---
