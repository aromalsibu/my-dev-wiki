## Custom Data Classes

**Creating and using your own data classes with Drift**

---

# What is it?

**Custom Data Classes** are your own Dart classes that represent database rows, instead of using Drift's generated data classes. By using the `@UseRowClass` annotation, you can tell Drift to map query results to your own classes, allowing you to add custom business logic, computed properties, and methods while maintaining type safety.

> **Think of Custom Data Classes like "custom containers"** – instead of using the standard box (generated class), you can design your own container with special features, extra pockets, and custom labels.

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
  
  // 👇 Custom computed properties
  bool get isAdult => age != null && age! >= 18;
  String get displayName => name.isNotEmpty ? name : 'Anonymous';
  String get initials => name.split(' ').map((s) => s[0]).join().toUpperCase();
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
  Type get rowClass => AppUser; // 👈 Tell Drift to use your class
}

// Now queries return AppUser instead of User
Future<List<AppUser>> getUsers() async {
  return await select(users).get(); // 👈 Returns List<AppUser>
}
```

> **What's happening here?**
> - **@UseRowClass** – Tells Drift to use your custom class
> - **Business Logic** – Add methods and computed properties
> - **Type Safety** – Still fully typed and checked
> - **Flexibility** – Not limited to generated fields

---

# Why does it exist?

- **Business Logic** – Add methods and computed properties
- **Custom Formatting** – Control how data is displayed
- **Domain Models** – Separate database models from domain models
- **Validation Logic** – Add validation methods
- **Serialization** – Custom toJson/fromJson logic
- **Code Organization** – Keep database and domain logic separate

---

# Basic Custom Data Classes

> **Creating simple custom data classes**

## Standard Implementation

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
  
  @override
  Type get rowClass => AppUser;
}

// lib/database/models/app_user.dart
import 'package:drift/drift.dart';

@UseRowClass
class AppUser {
  final int id;
  final String username;
  final String email;
  final String? fullName;
  final int? age;
  final bool isActive;
  final bool isVerified;
  final DateTime createdAt;
  final DateTime? updatedAt;
  
  AppUser({
    required this.id,
    required this.username,
    required this.email,
    this.fullName,
    this.age,
    required this.isActive,
    required this.isVerified,
    required this.createdAt,
    this.updatedAt,
  });
  
  // 👇 Computed properties
  String get displayName => fullName ?? username;
  
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
  
  bool get isAdult => age != null && age! >= 18;
  bool get isTeenager => age != null && age! >= 13 && age! < 18;
  bool get isChild => age != null && age! < 13;
  
  String get ageGroup {
    if (age == null) return 'Unknown';
    if (isChild) return 'Child';
    if (isTeenager) return 'Teenager';
    return 'Adult';
  }
  
  bool get canLogin => isActive && isVerified;
  
  Duration get accountAge => DateTime.now().difference(createdAt);
  
  String get accountAgeText {
    final days = accountAge.inDays;
    if (days < 1) return 'Today';
    if (days < 7) return '$days days';
    if (days < 30) return '${days ~/ 7} weeks';
    if (days < 365) return '${days ~/ 30} months';
    return '${days ~/ 365} years';
  }
  
  // 👇 Copy with
  AppUser copyWith({
    int? id,
    String? username,
    String? email,
    String? fullName,
    int? age,
    bool? isActive,
    bool? isVerified,
    DateTime? createdAt,
    DateTime? updatedAt,
  }) {
    return AppUser(
      id: id ?? this.id,
      username: username ?? this.username,
      email: email ?? this.email,
      fullName: fullName ?? this.fullName,
      age: age ?? this.age,
      isActive: isActive ?? this.isActive,
      isVerified: isVerified ?? this.isVerified,
      createdAt: createdAt ?? this.createdAt,
      updatedAt: updatedAt ?? this.updatedAt,
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
    'createdAt': createdAt.toIso8601String(),
    'updatedAt': updatedAt?.toIso8601String(),
  };
  
  factory AppUser.fromJson(Map<String, dynamic> json) => AppUser(
    id: json['id'] as int,
    username: json['username'] as String,
    email: json['email'] as String,
    fullName: json['fullName'] as String?,
    age: json['age'] as int?,
    isActive: json['isActive'] as bool,
    isVerified: json['isVerified'] as bool,
    createdAt: DateTime.parse(json['createdAt'] as String),
    updatedAt: json['updatedAt'] != null
        ? DateTime.parse(json['updatedAt'] as String)
        : null,
  );
}
```

---

# Custom Data Classes with Relationships

> **Handling related data in custom classes**

## One-to-Many Relationship

```dart
// lib/database/tables/posts.dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  IntColumn get userId => integer().references(Users, #id)();
  BoolColumn get isPublished => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get publishedAt => dateTime().nullable()();
  
  @override
  Type get rowClass => AppPost;
}

// lib/database/models/app_post.dart
@UseRowClass
class AppPost {
  final int id;
  final String title;
  final String content;
  final int userId;
  final bool isPublished;
  final DateTime createdAt;
  final DateTime? publishedAt;
  
  // 👇 Related user (loaded separately)
  AppUser? user;
  
  AppPost({
    required this.id,
    required this.title,
    required this.content,
    required this.userId,
    required this.isPublished,
    required this.createdAt,
    this.publishedAt,
    this.user,
  });
  
  String get excerpt {
    if (content.length <= 150) return content;
    return '${content.substring(0, 150)}...';
  }
  
  bool get isDraft => !isPublished;
  
  String get status => isPublished ? 'Published' : 'Draft';
  
  AppPost withUser(AppUser user) {
    return AppPost(
      id: id,
      title: title,
      content: content,
      userId: userId,
      isPublished: isPublished,
      createdAt: createdAt,
      publishedAt: publishedAt,
      user: user,
    );
  }
}

// lib/database/database.dart - Custom query with related data
class AppDatabase extends _$AppDatabase {
  // 👇 Get posts with their authors
  Future<List<AppPost>> getPostsWithAuthors() async {
    final results = await customSelect('''
      SELECT p.*, u.* 
      FROM posts p
      INNER JOIN users u ON p.user_id = u.id
      WHERE p.is_published = 1
      ORDER BY p.created_at DESC
    ''').get();
    
    return results.map((row) {
      // Parse post
      final post = AppPost(
        id: row.data['id'] as int,
        title: row.data['title'] as String,
        content: row.data['content'] as String,
        userId: row.data['user_id'] as int,
        isPublished: row.data['is_published'] == 1,
        createdAt: DateTime.fromMillisecondsSinceEpoch(row.data['created_at'] as int),
        publishedAt: row.data['published_at'] != null
            ? DateTime.fromMillisecondsSinceEpoch(row.data['published_at'] as int)
            : null,
      );
      
      // Parse user
      final user = AppUser(
        id: row.data['users.id'] as int,
        username: row.data['users.username'] as String,
        email: row.data['users.email'] as String,
        fullName: row.data['users.full_name'] as String?,
        age: row.data['users.age'] as int?,
        isActive: row.data['users.is_active'] == 1,
        isVerified: row.data['users.is_verified'] == 1,
        createdAt: DateTime.fromMillisecondsSinceEpoch(row.data['users.created_at'] as int),
        updatedAt: row.data['users.updated_at'] != null
            ? DateTime.fromMillisecondsSinceEpoch(row.data['users.updated_at'] as int)
            : null,
      );
      
      return post.withUser(user);
    }).toList();
  }
}
```

---

# Custom Data Classes with Business Logic

> **Adding complex business logic to data classes**

```dart
// lib/database/models/app_order.dart
@UseRowClass
class AppOrder {
  final int id;
  final String orderNumber;
  final int userId;
  final double subtotal;
  final double tax;
  final double shippingCost;
  final double discount;
  final double total;
  final String status;
  final String shippingAddress;
  final DateTime orderDate;
  final DateTime? shippedDate;
  final DateTime? deliveredDate;
  final bool isPaid;
  final bool isShipped;
  final bool isDelivered;
  
  // 👇 Related items (loaded separately)
  final List<AppOrderItem>? items;
  
  AppOrder({
    required this.id,
    required this.orderNumber,
    required this.userId,
    required this.subtotal,
    required this.tax,
    required this.shippingCost,
    required this.discount,
    required this.total,
    required this.status,
    required this.shippingAddress,
    required this.orderDate,
    this.shippedDate,
    this.deliveredDate,
    required this.isPaid,
    required this.isShipped,
    required this.isDelivered,
    this.items,
  });
  
  // 👇 Computed properties
  double get subtotalWithTax => subtotal + tax;
  double get totalWithShipping => subtotalWithTax + shippingCost;
  double get finalTotal => totalWithShipping - discount;
  
  String get statusDisplay {
    switch (status) {
      case 'pending': return '⏳ Pending';
      case 'processing': return '🔄 Processing';
      case 'paid': return '💰 Paid';
      case 'shipped': return '📦 Shipped';
      case 'delivered': return '✅ Delivered';
      case 'cancelled': return '❌ Cancelled';
      default: return status;
    }
  }
  
  String get paymentStatusDisplay {
    return isPaid ? '✅ Paid' : '❌ Unpaid';
  }
  
  String get deliveryStatus {
    if (isDelivered) return 'Delivered';
    if (isShipped) return 'In Transit';
    return 'Processing';
  }
  
  bool get isPending => status == 'pending';
  bool get isProcessing => status == 'processing';
  bool get isCancelled => status == 'cancelled';
  bool get isComplete => status == 'delivered';
  
  bool get canBeCancelled => isPending || isProcessing;
  bool get canBeShipped => status == 'paid' || status == 'processing';
  
  // 👇 Date formatting
  String get orderDateFormatted {
    final now = DateTime.now();
    final diff = now.difference(orderDate);
    
    if (diff.inDays == 0) return 'Today';
    if (diff.inDays == 1) return 'Yesterday';
    if (diff.inDays < 7) return '${diff.inDays} days ago';
    return '${orderDate.month}/${orderDate.day}/${orderDate.year}';
  }
  
  String get shippingTime {
    if (shippedDate == null) return 'Not shipped yet';
    if (deliveredDate != null) {
      final diff = deliveredDate!.difference(shippedDate!);
      final days = diff.inDays;
      return days == 0 ? 'Same day' : '$days days';
    }
    final diff = DateTime.now().difference(shippedDate!);
    final days = diff.inDays;
    return 'In transit for $days days';
  }
  
  // 👇 Total calculations with items
  double get itemCount => items?.length ?? 0;
  double get totalItems {
    if (items == null) return 0;
    return items!.fold(0, (sum, item) => sum + item.quantity);
  }
  
  // 👇 Copy with
  AppOrder copyWith({
    int? id,
    String? orderNumber,
    int? userId,
    double? subtotal,
    double? tax,
    double? shippingCost,
    double? discount,
    double? total,
    String? status,
    String? shippingAddress,
    DateTime? orderDate,
    DateTime? shippedDate,
    DateTime? deliveredDate,
    bool? isPaid,
    bool? isShipped,
    bool? isDelivered,
    List<AppOrderItem>? items,
  }) {
    return AppOrder(
      id: id ?? this.id,
      orderNumber: orderNumber ?? this.orderNumber,
      userId: userId ?? this.userId,
      subtotal: subtotal ?? this.subtotal,
      tax: tax ?? this.tax,
      shippingCost: shippingCost ?? this.shippingCost,
      discount: discount ?? this.discount,
      total: total ?? this.total,
      status: status ?? this.status,
      shippingAddress: shippingAddress ?? this.shippingAddress,
      orderDate: orderDate ?? this.orderDate,
      shippedDate: shippedDate ?? this.shippedDate,
      deliveredDate: deliveredDate ?? this.deliveredDate,
      isPaid: isPaid ?? this.isPaid,
      isShipped: isShipped ?? this.isShipped,
      isDelivered: isDelivered ?? this.isDelivered,
      items: items ?? this.items,
    );
  }
}

// lib/database/models/app_order_item.dart
@UseRowClass
class AppOrderItem {
  final int orderId;
  final int productId;
  final int quantity;
  final double unitPrice;
  final double discountPercent;
  final double subtotal;
  final double discountAmount;
  final double total;
  
  // 👇 Related product (loaded separately)
  final AppProduct? product;
  
  AppOrderItem({
    required this.orderId,
    required this.productId,
    required this.quantity,
    required this.unitPrice,
    required this.discountPercent,
    required this.subtotal,
    required this.discountAmount,
    required this.total,
    this.product,
  });
  
  double get totalWithoutDiscount => subtotal;
  
  double get discountAmountDisplay => discountAmount;
  
  double get totalWithDiscount => total;
  
  double get pricePerUnitWithDiscount => unitPrice * (1 - discountPercent / 100);
  
  String get productName => product?.name ?? 'Product #$productId';
  String get productSku => product?.sku ?? '';
  
  AppOrderItem withProduct(AppProduct product) {
    return AppOrderItem(
      orderId: orderId,
      productId: productId,
      quantity: quantity,
      unitPrice: unitPrice,
      discountPercent: discountPercent,
      subtotal: subtotal,
      discountAmount: discountAmount,
      total: total,
      product: product,
    );
  }
}
```

---

# Custom Data Classes with Validation

> **Adding validation logic to data classes**

```dart
// lib/database/models/validated_user.dart
@UseRowClass
class ValidatedUser {
  final int id;
  final String username;
  final String email;
  final String? fullName;
  final int? age;
  
  ValidatedUser({
    required this.id,
    required this.username,
    required this.email,
    this.fullName,
    this.age,
  });
  
  // 👇 Validation methods
  bool get isValidUsername => username.length >= 3 && username.length <= 30;
  bool get isValidEmail => email.contains('@') && email.contains('.');
  bool get isValidAge => age == null || (age! >= 0 && age! <= 150);
  bool get isValidFullName => fullName == null || fullName!.length >= 2;
  
  bool get isValid => 
    isValidUsername && 
    isValidEmail && 
    isValidAge && 
    isValidFullName;
  
  List<String> get validationErrors {
    final errors = <String>[];
    
    if (!isValidUsername) {
      errors.add('Username must be 3-30 characters');
    }
    
    if (!isValidEmail) {
      errors.add('Invalid email format');
    }
    
    if (!isValidAge) {
      errors.add('Age must be between 0 and 150');
    }
    
    if (!isValidFullName) {
      errors.add('Full name must be at least 2 characters');
    }
    
    return errors;
  }
  
  String get validationErrorText {
    final errors = validationErrors;
    return errors.isEmpty ? '✅ Valid' : errors.join('\n');
  }
}

// Usage in UI
class UserValidationWidget extends StatelessWidget {
  final ValidatedUser user;
  
  const UserValidationWidget({required this.user});
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Username: ${user.username}'),
            Text('Email: ${user.email}'),
            Text('Age: ${user.age ?? "Not set"}'),
            SizedBox(height: 8),
            Container(
              padding: EdgeInsets.all(8),
              color: user.isValid ? Colors.green[50] : Colors.red[50],
              child: Text(
                user.validationErrorText,
                style: TextStyle(
                  color: user.isValid ? Colors.green : Colors.red,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

# Real-World Example

> **Complete e-commerce custom data classes**

```dart
// lib/database/models/ecommerce_models.dart
import 'package:drift/drift.dart';

// ==================== USER MODEL ====================
@UseRowClass
class AppUser {
  final int id;
  final String username;
  final String email;
  final String? fullName;
  final String? phoneNumber;
  final int? age;
  final bool isActive;
  final bool isVerified;
  final bool isAdmin;
  final DateTime createdAt;
  final DateTime? updatedAt;
  final DateTime? lastLogin;
  
  // 👇 Related data (lazy loaded)
  List<AppOrder>? orders;
  List<AppAddress>? addresses;
  
  AppUser({
    required this.id,
    required this.username,
    required this.email,
    this.fullName,
    this.phoneNumber,
    this.age,
    required this.isActive,
    required this.isVerified,
    required this.isAdmin,
    required this.createdAt,
    this.updatedAt,
    this.lastLogin,
    this.orders,
    this.addresses,
  });
  
  // 👇 Display properties
  String get displayName => fullName ?? username;
  String get initials => _getInitials();
  
  String _getInitials() {
    if (fullName != null && fullName!.isNotEmpty) {
      final parts = fullName!.split(' ');
      if (parts.length >= 2) {
        return '${parts.first[0]}${parts.last[0]}'.toUpperCase();
      }
      return fullName![0].toUpperCase();
    }
    return username[0].toUpperCase();
  }
  
  // 👇 Status properties
  bool get canLogin => isActive && isVerified;
  bool get isOnline => lastLogin != null &&
      DateTime.now().difference(lastLogin!).inMinutes < 5;
  
  String get status {
    if (!isActive) return 'Inactive';
    if (!isVerified) return 'Unverified';
    if (isOnline) return 'Online';
    return 'Offline';
  }
  
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
  
  // 👇 Stats
  int get totalOrders => orders?.length ?? 0;
  double get totalSpent => orders?.fold(0.0, (sum, o) => sum + o.total) ?? 0.0;
  
  double get averageOrderValue => totalOrders > 0 ? totalSpent / totalOrders : 0.0;
  
  int get completedOrders => orders?.where((o) => o.isComplete).length ?? 0;
  
  // 👇 Address methods
  AppAddress? get defaultAddress => addresses?.firstWhere(
    (a) => a.isDefault,
    orElse: () => addresses?.firstOrNull,
  );
  
  bool get hasDefaultAddress => defaultAddress != null;
  
  // 👇 Copy with
  AppUser copyWith({
    int? id,
    String? username,
    String? email,
    String? fullName,
    String? phoneNumber,
    int? age,
    bool? isActive,
    bool? isVerified,
    bool? isAdmin,
    DateTime? createdAt,
    DateTime? updatedAt,
    DateTime? lastLogin,
    List<AppOrder>? orders,
    List<AppAddress>? addresses,
  }) {
    return AppUser(
      id: id ?? this.id,
      username: username ?? this.username,
      email: email ?? this.email,
      fullName: fullName ?? this.fullName,
      phoneNumber: phoneNumber ?? this.phoneNumber,
      age: age ?? this.age,
      isActive: isActive ?? this.isActive,
      isVerified: isVerified ?? this.isVerified,
      isAdmin: isAdmin ?? this.isAdmin,
      createdAt: createdAt ?? this.createdAt,
      updatedAt: updatedAt ?? this.updatedAt,
      lastLogin: lastLogin ?? this.lastLogin,
      orders: orders ?? this.orders,
      addresses: addresses ?? this.addresses,
    );
  }
  
  AppUser withOrders(List<AppOrder> orders) => copyWith(orders: orders);
  AppUser withAddresses(List<AppAddress> addresses) => copyWith(addresses: addresses);
}

// ==================== ADDRESS MODEL ====================
@UseRowClass
class AppAddress {
  final int id;
  final int userId;
  final String street;
  final String city;
  final String state;
  final String zipCode;
  final String country;
  final bool isDefault;
  final DateTime createdAt;
  
  AppAddress({
    required this.id,
    required this.userId,
    required this.street,
    required this.city,
    required this.state,
    required this.zipCode,
    required this.country,
    required this.isDefault,
    required this.createdAt,
  });
  
  String get fullAddress => '$street, $city, $state $zipCode, $country';
  
  String get shortAddress => '$street, $city';
  
  bool get isValid => 
    street.isNotEmpty && 
    city.isNotEmpty && 
    state.isNotEmpty && 
    zipCode.isNotEmpty && 
    country.isNotEmpty;
  
  AppAddress copyWith({
    int? id,
    int? userId,
    String? street,
    String? city,
    String? state,
    String? zipCode,
    String? country,
    bool? isDefault,
    DateTime? createdAt,
  }) {
    return AppAddress(
      id: id ?? this.id,
      userId: userId ?? this.userId,
      street: street ?? this.street,
      city: city ?? this.city,
      state: state ?? this.state,
      zipCode: zipCode ?? this.zipCode,
      country: country ?? this.country,
      isDefault: isDefault ?? this.isDefault,
      createdAt: createdAt ?? this.createdAt,
    );
  }
}

// ==================== ORDER MODEL ====================
@UseRowClass
class AppOrder {
  final int id;
  final String orderNumber;
  final int userId;
  final double subtotal;
  final double tax;
  final double shippingCost;
  final double discount;
  final double total;
  final String status;
  final String shippingAddress;
  final String billingAddress;
  final String paymentMethod;
  final String paymentStatus;
  final DateTime orderDate;
  final DateTime? shippedDate;
  final DateTime? deliveredDate;
  final bool isPaid;
  final bool isShipped;
  final bool isDelivered;
  
  // 👇 Related data
  AppUser? user;
  List<AppOrderItem>? items;
  
  AppOrder({
    required this.id,
    required this.orderNumber,
    required this.userId,
    required this.subtotal,
    required this.tax,
    required this.shippingCost,
    required this.discount,
    required this.total,
    required this.status,
    required this.shippingAddress,
    required this.billingAddress,
    required this.paymentMethod,
    required this.paymentStatus,
    required this.orderDate,
    this.shippedDate,
    this.deliveredDate,
    required this.isPaid,
    required this.isShipped,
    required this.isDelivered,
    this.user,
    this.items,
  });
  
  // 👇 Status methods
  bool get isPending => status == 'pending';
  bool get isProcessing => status == 'processing';
  bool get isPaidStatus => status == 'paid';
  bool get isShippedStatus => status == 'shipped';
  bool get isDeliveredStatus => status == 'delivered';
  bool get isCancelledStatus => status == 'cancelled';
  bool get isComplete => isDeliveredStatus;
  bool get canBeCancelled => isPending || isProcessing;
  bool get canBeShipped => isPaidStatus || isProcessing;
  
  String get statusDisplay {
    switch (status) {
      case 'pending': return '⏳ Pending';
      case 'processing': return '🔄 Processing';
      case 'paid': return '💰 Paid';
      case 'shipped': return '📦 Shipped';
      case 'delivered': return '✅ Delivered';
      case 'cancelled': return '❌ Cancelled';
      default: return status;
    }
  }
  
  String get paymentStatusDisplay {
    switch (paymentStatus) {
      case 'unpaid': return '💳 Unpaid';
      case 'paid': return '✅ Paid';
      case 'refunded': return '↩️ Refunded';
      default: return paymentStatus;
    }
  }
  
  // 👇 Date formatting
  String get orderDateFormatted {
    final now = DateTime.now();
    final diff = now.difference(orderDate);
    
    if (diff.inDays == 0) return 'Today at ${_formatTime(orderDate)}';
    if (diff.inDays == 1) return 'Yesterday at ${_formatTime(orderDate)}';
    if (diff.inDays < 7) return '${diff.inDays} days ago';
    return _formatDate(orderDate);
  }
  
  String _formatDate(DateTime date) {
    return '${date.month}/${date.day}/${date.year}';
  }
  
  String _formatTime(DateTime date) {
    final hour = date.hour.toString().padLeft(2, '0');
    final minute = date.minute.toString().padLeft(2, '0');
    return '$hour:$minute';
  }
  
  // 👇 Totals
  int get itemCount => items?.length ?? 0;
  int get totalItems => items?.fold(0, (sum, i) => sum + i.quantity) ?? 0;
  
  double get subtotalWithTax => subtotal + tax;
  double get totalWithShipping => subtotalWithTax + shippingCost;
  double get finalTotal => totalWithShipping - discount;
  
  // 👇 Shipping
  String get shippingTime {
    if (shippedDate == null) return 'Not shipped yet';
    if (deliveredDate != null) {
      final diff = deliveredDate!.difference(shippedDate!);
      final days = diff.inDays;
      return days == 0 ? 'Same day' : '$days days';
    }
    final diff = DateTime.now().difference(shippedDate!);
    final days = diff.inDays;
    return 'In transit for $days days';
  }
  
  bool get isShippingLate {
    if (shippedDate == null || deliveredDate != null) return false;
    final days = DateTime.now().difference(shippedDate!).inDays;
    return days > 5;
  }
  
  // 👇 Copy with
  AppOrder copyWith({
    int? id,
    String? orderNumber,
    int? userId,
    double? subtotal,
    double? tax,
    double? shippingCost,
    double? discount,
    double? total,
    String? status,
    String? shippingAddress,
    String? billingAddress,
    String? paymentMethod,
    String? paymentStatus,
    DateTime? orderDate,
    DateTime? shippedDate,
    DateTime? deliveredDate,
    bool? isPaid,
    bool? isShipped,
    bool? isDelivered,
    AppUser? user,
    List<AppOrderItem>? items,
  }) {
    return AppOrder(
      id: id ?? this.id,
      orderNumber: orderNumber ?? this.orderNumber,
      userId: userId ?? this.userId,
      subtotal: subtotal ?? this.subtotal,
      tax: tax ?? this.tax,
      shippingCost: shippingCost ?? this.shippingCost,
      discount: discount ?? this.discount,
      total: total ?? this.total,
      status: status ?? this.status,
      shippingAddress: shippingAddress ?? this.shippingAddress,
      billingAddress: billingAddress ?? this.billingAddress,
      paymentMethod: paymentMethod ?? this.paymentMethod,
      paymentStatus: paymentStatus ?? this.paymentStatus,
      orderDate: orderDate ?? this.orderDate,
      shippedDate: shippedDate ?? this.shippedDate,
      deliveredDate: deliveredDate ?? this.deliveredDate,
      isPaid: isPaid ?? this.isPaid,
      isShipped: isShipped ?? this.isShipped,
      isDelivered: isDelivered ?? this.isDelivered,
      user: user ?? this.user,
      items: items ?? this.items,
    );
  }
  
  AppOrder withUser(AppUser user) => copyWith(user: user);
  AppOrder withItems(List<AppOrderItem> items) => copyWith(items: items);
}

// ==================== ORDER ITEM MODEL ====================
@UseRowClass
class AppOrderItem {
  final int orderId;
  final int productId;
  final int quantity;
  final double unitPrice;
  final double discountPercent;
  final double subtotal;
  final double discountAmount;
  final double total;
  final bool isShipped;
  final bool isDelivered;
  
  // 👇 Related data
  AppProduct? product;
  
  AppOrderItem({
    required this.orderId,
    required this.productId,
    required this.quantity,
    required this.unitPrice,
    required this.discountPercent,
    required this.subtotal,
    required this.discountAmount,
    required this.total,
    required this.isShipped,
    required this.isDelivered,
    this.product,
  });
  
  // 👇 Computed properties
  double get pricePerUnitWithDiscount => unitPrice * (1 - discountPercent / 100);
  
  double get totalWithoutDiscount => subtotal;
  double get totalWithDiscount => total;
  double get discountSaved => discountAmount;
  
  String get productName => product?.name ?? 'Product #$productId';
  String get productSku => product?.sku ?? '';
  
  bool get isShippedStatus => isShipped;
  bool get isDeliveredStatus => isDelivered;
  
  String get status {
    if (isDelivered) return 'Delivered';
    if (isShipped) return 'Shipped';
    return 'Pending';
  }
  
  // 👇 Copy with
  AppOrderItem copyWith({
    int? orderId,
    int? productId,
    int? quantity,
    double? unitPrice,
    double? discountPercent,
    double? subtotal,
    double? discountAmount,
    double? total,
    bool? isShipped,
    bool? isDelivered,
    AppProduct? product,
  }) {
    return AppOrderItem(
      orderId: orderId ?? this.orderId,
      productId: productId ?? this.productId,
      quantity: quantity ?? this.quantity,
      unitPrice: unitPrice ?? this.unitPrice,
      discountPercent: discountPercent ?? this.discountPercent,
      subtotal: subtotal ?? this.subtotal,
      discountAmount: discountAmount ?? this.discountAmount,
      total: total ?? this.total,
      isShipped: isShipped ?? this.isShipped,
      isDelivered: isDelivered ?? this.isDelivered,
      product: product ?? this.product,
    );
  }
  
  AppOrderItem withProduct(AppProduct product) => copyWith(product: product);
}

// lib/database/database.dart - Complete usage
@DriftDatabase(tables: [Users, Posts, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Get user with all related data
  Future<AppUser> getUserWithData(int userId) async {
    final user = await (select(users)..where((u) => u.id.equals(userId)))
        .getSingle();
    
    // Get orders
    final orders = await (select(orders)
      ..where((o) => o.userId.equals(userId)))
      .get();
    
    // Get addresses
    final addresses = await (select(addresses)
      ..where((a) => a.userId.equals(userId)))
      .get();
    
    return user.withOrders(orders).withAddresses(addresses);
  }

  // 👇 Get order with all related data
  Future<AppOrder> getOrderWithData(int orderId) async {
    final order = await (select(orders)
      ..where((o) => o.id.equals(orderId)))
      .getSingle();
    
    // Get items with products
    final items = await (select(orderItems)
      ..where((i) => i.orderId.equals(orderId)))
      .get();
    
    final itemWithProducts = <AppOrderItem>[];
    for (final item in items) {
      final product = await (select(products)
        ..where((p) => p.id.equals(item.productId)))
        .getSingle();
      itemWithProducts.add(item.withProduct(product));
    }
    
    // Get user
    final user = await (select(users)
      ..where((u) => u.id.equals(order.userId)))
      .getSingle();
    
    return order
        .withUser(user)
        .withItems(itemWithProducts);
  }
}

// lib/ui/widgets/order_summary.dart
class OrderSummaryWidget extends StatelessWidget {
  final AppOrder order;
  
  const OrderSummaryWidget({required this.order});
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.all(8),
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text(
                  'Order #${order.orderNumber}',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                Text(
                  order.statusDisplay,
                  style: TextStyle(
                    color: _getStatusColor(order.status),
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
            SizedBox(height: 8),
            Text('Placed: ${order.orderDateFormatted}'),
            Text('${order.itemCount} items, ${order.totalItems} total'),
            Text('\$${order.finalTotal.toStringAsFixed(2)}'),
            if (order.isShipped)
              Text('Shipping: ${order.shippingTime}'),
            if (order.isComplete)
              Text('Delivered ✅'),
          ],
        ),
      ),
    );
  }
  
  Color _getStatusColor(String status) {
    switch (status) {
      case 'pending': return Colors.orange;
      case 'processing': return Colors.blue;
      case 'paid': return Colors.green;
      case 'shipped': return Colors.purple;
      case 'delivered': return Colors.green;
      case 'cancelled': return Colors.red;
      default: return Colors.grey;
    }
  }
}
```

---

# Best Practices

- **Use `@UseRowClass`** – For custom data classes
- **Add business logic** – Methods and computed properties
- **Keep it focused** – One class per domain concept
- **Use copyWith** – For immutable updates
- **Add validation methods** – `isValid`, `validationErrors`
- **Add display helpers** – `formatDate`, `displayName`
- **Document properties** – Explain what they do
- **Keep data classes pure** – No database logic

---

# Common Mistakes

## Mistake 1: Forgetting `@UseRowClass`

Wrong:
```dart
// 🚫 Won't work
class AppUser { ... }
```

Correct:
```dart
// ✅ Must annotate
@UseRowClass
class AppUser { ... }
```

## Mistake 2: Not specifying rowClass in table

Wrong:
```dart
// 🚫 Returns generated User class
class Users extends Table {
  // Missing rowClass override
}
```

Correct:
```dart
// ✅ Specify custom class
class Users extends Table {
  @override
  Type get rowClass => AppUser;
}
```

## Mistake 3: Adding database logic to data classes

Wrong:
```dart
// 🚫 Database logic in data class
@UseRowClass
class AppUser {
  void save() { ... } // ❌ Not domain logic
}
```

Correct:
```dart
// ✅ Keep data classes pure
@UseRowClass
class AppUser {
  // Domain logic only
}
```

---

# Summary

| Feature | Purpose | Example |
|---------|---------|---------|
| **@UseRowClass** | Mark custom data class | `@UseRowClass` |
| **rowClass** | Tell Drift which class to use | `Type get rowClass => AppUser` |
| **Business Logic** | Domain logic in class | `isAdult`, `displayName` |
| **Validation** | Validate data | `isValid`, `validationErrors` |
| **Relationships** | Link related data | `withUser`, `withItems` |

---

# Next Steps

Now you understand custom data classes, let's dive deeper:

- [Companions](link) – Advanced companion usage
- [CRUD Operations](link) – Complete CRUD guide
- [Relationships](link) – Working with related data

---

# Did You Know?

- **Custom data classes are fully typed** – No casting needed

- **`@UseRowClass` works with any class** – Not just generated ones

- **You can have multiple custom classes** – For different views

- **Data classes can be immutable** – With final fields

- **Custom classes can have factories** – For complex creation

- **You can mix generated and custom** – In the same database

- **Custom classes support copyWith** – For immutability

- **Data classes can be serialized** – To JSON for APIs

---
