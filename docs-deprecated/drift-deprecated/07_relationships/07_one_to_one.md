## One-to-One

**Implementing one-to-one relationships in Drift**

---

# What is it?

**One-to-One** is a relationship where a record in one table corresponds to exactly one record in another table. This is implemented using a foreign key with a unique constraint, ensuring that each record on the "many" side can only be linked to one record on the "one" side. One-to-one relationships are useful for splitting data into separate tables for organization, security, or performance reasons.

> **Think of One-to-One like "a person and their passport"** – each person has exactly one passport, and each passport belongs to exactly one person. They are separate entities but uniquely linked.

```dart
// 👇 One-to-One: A user has one profile
// Users table
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
}

// Profiles table (one-to-one with Users)
class Profiles extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Foreign key with unique constraint
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .unique() // 👈 Ensures one-to-one
    .named('user_id')();
  
  TextColumn get bio => text().nullable()();
  TextColumn get avatar => text().nullable()();
  TextColumn get phone => text().nullable()();
  DateTimeColumn get birthDate => dateTime().nullable()();
}

// Query: Get user with profile
final userId = 1;
final query = db.select(db.users).join([
  innerJoin(
    db.profiles,
    db.profiles.userId.equals(db.users.id),
  )
]);
query.where((u) => db.users.id.equals(userId));

final result = await query.getSingle();
final user = result.readTable(db.users);
final profile = result.readTable(db.profiles);
```

> **What's happening here?**
> - **Users table** – One side of the relationship
> - **Profiles table** – The related table
> - **Unique constraint** – `unique()` ensures one-to-one
> - **Foreign key** – Links to the Users table
> - **Query** – Inner join returns both records

---

# Why does it exist?

- **Data Separation** – Split large tables for organization
- **Security** – Separate sensitive data
- **Performance** – Optimize different access patterns
- **Optional Data** – Keep optional data separate
- **Schema Evolution** – Add fields without modifying main table
- **Data Integrity** – Enforce unique relationships

---

# Defining One-to-One

> **Creating the relationship in tables**

## Basic One-to-One

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get passwordHash => text().named('password_hash')();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/tables/profiles.dart (One-to-One)
class Profiles extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Foreign key with unique constraint
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .unique() // 👈 Ensures one-to-one
    .named('user_id')();
  
  TextColumn get fullName => text().nullable().named('full_name')();
  TextColumn get bio => text().nullable()();
  TextColumn get avatar => text().nullable()();
  TextColumn get phone => text().nullable()();
  DateTimeColumn get birthDate => dateTime().nullable().named('birth_date')();
  TextColumn get address => text().nullable()();
  TextColumn get city => text().nullable()();
  TextColumn get country => text().nullable()();
  
  @override
  List<Index> get indexes => [
    Index('idx_profiles_user', 'user_id'),
  ];
}
```

---

# Querying One-to-One

> **Retrieving related data**

## Get User with Profile (Inner Join)

```dart
// 👇 Get user with profile (user must have profile)
Future<UserWithProfile> getUserWithProfile(int userId) async {
  final query = db.select(db.users).join([
    innerJoin(
      db.profiles,
      db.profiles.userId.equals(db.users.id),
    )
  ]);
  
  query.where((u) => db.users.id.equals(userId));
  
  final result = await query.getSingle();
  final user = result.readTable(db.users);
  final profile = result.readTable(db.profiles);
  
  return UserWithProfile(
    user: user,
    profile: profile,
  );
}
```

## Get User with Profile (Left Join)

```dart
// 👇 Get user with optional profile
Future<UserWithOptionalProfile> getUserWithOptionalProfile(int userId) async {
  final query = db.select(db.users).join([
    leftJoin(
      db.profiles,
      db.profiles.userId.equals(db.users.id),
    )
  ]);
  
  query.where((u) => db.users.id.equals(userId));
  
  final result = await query.getSingle();
  final user = result.readTable(db.users);
  final profile = result.readTableOrNull(db.profiles);
  
  return UserWithOptionalProfile(
    user: user,
    profile: profile,
  );
}
```

## Get All Users with Profiles

```dart
// 👇 Get all users with their profiles
Future<List<UserWithProfile>> getAllUsersWithProfiles() async {
  final query = db.select(db.users).join([
    innerJoin(
      db.profiles,
      db.profiles.userId.equals(db.users.id),
    )
  ]);
  
  query.where((u) => db.users.isActive.equals(true));
  query.orderBy([(u) => OrderingTerm.asc(db.users.username)]);
  
  final results = await query.get();
  
  return results.map((row) {
    return UserWithProfile(
      user: row.readTable(db.users),
      profile: row.readTable(db.profiles),
    );
  }).toList();
}

// 👇 Get only users with profiles (left join filter)
Future<List<UserWithOptionalProfile>> getAllUsersWithOptionalProfiles() async {
  final query = db.select(db.users).join([
    leftJoin(
      db.profiles,
      db.profiles.userId.equals(db.users.id),
    )
  ]);
  
  query.where((u) => db.users.isActive.equals(true));
  query.orderBy([(u) => OrderingTerm.asc(db.users.username)]);
  
  final results = await query.get();
  
  return results.map((row) {
    return UserWithOptionalProfile(
      user: row.readTable(db.users),
      profile: row.readTableOrNull(db.profiles),
    );
  }).toList();
}
```

---

# Managing One-to-One

> **Creating, updating, and deleting related data**

## Create User with Profile

```dart
// 👇 Create user and profile together
Future<UserWithProfile> createUserWithProfile({
  required String username,
  required String email,
  required String password,
  required String fullName,
  String? bio,
}) async {
  return await db.transaction(() async {
    // 1️⃣ Create user
    final userId = await db.into(db.users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        passwordHash: _hashPassword(password),
      ),
    );
    
    // 2️⃣ Create profile
    await db.into(db.profiles).insert(
      ProfilesCompanion.insert(
        userId: userId,
        fullName: Value(fullName),
        bio: Value(bio),
      ),
    );
    
    // 3️⃣ Get complete user with profile
    return await getUserWithProfile(userId);
  });
}
```

## Update Profile

```dart
// 👇 Update user profile
Future<Profile> updateProfile({
  required int userId,
  String? fullName,
  String? bio,
  String? phone,
  String? address,
  String? city,
  String? country,
}) async {
  // Check if profile exists
  final existing = await (db.select(db.profiles)
    ..where((p) => p.userId.equals(userId)))
    .getSingleOrNull();
  
  if (existing == null) {
    // Create profile if doesn't exist
    final id = await db.into(db.profiles).insert(
      ProfilesCompanion.insert(
        userId: userId,
        fullName: fullName != null ? Value(fullName) : const Value.absent(),
        bio: bio != null ? Value(bio) : const Value.absent(),
        phone: phone != null ? Value(phone) : const Value.absent(),
        address: address != null ? Value(address) : const Value.absent(),
        city: city != null ? Value(city) : const Value.absent(),
        country: country != null ? Value(country) : const Value.absent(),
      ),
    );
    return await db.getProfile(id);
  }
  
  // Update existing profile
  await (db.update(db.profiles)..where((p) => p.userId.equals(userId)))
    .write(ProfilesCompanion(
      fullName: fullName != null ? Value(fullName) : const Value.absent(),
      bio: bio != null ? Value(bio) : const Value.absent(),
      phone: phone != null ? Value(phone) : const Value.absent(),
      address: address != null ? Value(address) : const Value.absent(),
      city: city != null ? Value(city) : const Value.absent(),
      country: country != null ? Value(country) : const Value.absent(),
    ));
  
  return await db.getProfile(existing.id);
}
```

## Delete User with Profile (Cascade)

```dart
// 👇 Delete user (profile auto-deleted via cascade)
Future<void> deleteUserWithProfile(int userId) async {
  await (db.delete(db.users)..where((u) => u.id.equals(userId))).go();
  // Profile is automatically deleted due to ON DELETE CASCADE
}
```

---

# Real-World Example

> **Complete e-commerce one-to-one system**

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get passwordHash => text().named('password_hash')();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false))();
  BoolColumn get isAdmin => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get lastLogin => dateTime().nullable().named('last_login')();
}

// lib/database/tables/user_profiles.dart
class UserProfiles extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .unique()
    .named('user_id')();
  
  TextColumn get fullName => text().nullable().named('full_name')();
  TextColumn get bio => text().nullable()();
  TextColumn get avatar => text().nullable()();
  TextColumn get phone => text().nullable()();
  DateTimeColumn get birthDate => dateTime().nullable().named('birth_date')();
  TextColumn get address => text().nullable()();
  TextColumn get city => text().nullable()();
  TextColumn get state => text().nullable()();
  TextColumn get country => text().nullable()();
  TextColumn get zipCode => text().nullable().named('zip_code')();
  TextColumn get website => text().nullable()();
  TextColumn get company => text().nullable()();
  TextColumn get position => text().nullable()();
  
  @override
  List<Index> get indexes => [
    Index('idx_user_profiles_user', 'user_id'),
  ];
}

// lib/database/tables/user_settings.dart
class UserSettings extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .unique()
    .named('user_id')();
  
  TextColumn get theme => text()
    .withDefault(const Constant('light'))
    .customConstraint("CHECK (theme IN ('light', 'dark', 'system'))")();
  
  TextColumn get language => text()
    .withDefault(const Constant('en'))
    .withLength(max: 2)();
  
  BoolColumn get notifications => boolean()
    .withDefault(const Constant(true))();
  
  BoolColumn get emailNotifications => boolean()
    .withDefault(const Constant(true))
    .named('email_notifications')();
  
  BoolColumn get pushNotifications => boolean()
    .withDefault(const Constant(true))
    .named('push_notifications')();
  
  @override
  List<Index> get indexes => [
    Index('idx_user_settings_user', 'user_id'),
  ];
}

// lib/database/one_to_one_service.dart
import 'package:drift/drift.dart';

class OneToOneService {
  final AppDatabase db;
  
  OneToOneService(this.db);

  // ==================== COMPLETE USER DATA ====================
  
  // 👇 Get complete user data
  Future<CompleteUserData> getCompleteUserData(int userId) async {
    final user = await db.getUser(userId);
    
    // Get profile (optional)
    final profile = await (db.select(db.userProfiles)
      ..where((p) => p.userId.equals(userId)))
      .getSingleOrNull();
    
    // Get settings (optional)
    final settings = await (db.select(db.userSettings)
      ..where((s) => s.userId.equals(userId)))
      .getSingleOrNull();
    
    return CompleteUserData(
      user: user,
      profile: profile,
      settings: settings,
    );
  }

  // ==================== USER PROFILE MANAGEMENT ====================
  
  // 👇 Create full user (with profile and settings)
  Future<CompleteUserData> createFullUser({
    required String username,
    required String email,
    required String password,
    String? fullName,
    String? bio,
    String? theme,
    String? language,
  }) async {
    return await db.transaction(() async {
      // 1️⃣ Create user
      final userId = await db.into(db.users).insert(
        UsersCompanion.insert(
          username: username,
          email: email,
          passwordHash: _hashPassword(password),
        ),
      );
      
      // 2️⃣ Create profile
      await db.into(db.userProfiles).insert(
        UserProfilesCompanion.insert(
          userId: userId,
          fullName: Value(fullName),
          bio: Value(bio),
        ),
      );
      
      // 3️⃣ Create settings
      await db.into(db.userSettings).insert(
        UserSettingsCompanion.insert(
          userId: userId,
          theme: theme != null ? Value(theme) : const Value.absent(),
          language: language != null ? Value(language) : const Value.absent(),
        ),
      );
      
      return await getCompleteUserData(userId);
    });
  }
  
  // 👇 Update user profile
  Future<UserProfiles> updateUserProfile({
    required int userId,
    String? fullName,
    String? bio,
    String? avatar,
    String? phone,
    DateTime? birthDate,
    String? address,
    String? city,
    String? state,
    String? country,
    String? zipCode,
    String? website,
    String? company,
    String? position,
  }) async {
    // Check if profile exists
    final existing = await (db.select(db.userProfiles)
      ..where((p) => p.userId.equals(userId)))
      .getSingleOrNull();
    
    final companion = UserProfilesCompanion(
      fullName: fullName != null ? Value(fullName) : const Value.absent(),
      bio: bio != null ? Value(bio) : const Value.absent(),
      avatar: avatar != null ? Value(avatar) : const Value.absent(),
      phone: phone != null ? Value(phone) : const Value.absent(),
      birthDate: birthDate != null ? Value(birthDate) : const Value.absent(),
      address: address != null ? Value(address) : const Value.absent(),
      city: city != null ? Value(city) : const Value.absent(),
      state: state != null ? Value(state) : const Value.absent(),
      country: country != null ? Value(country) : const Value.absent(),
      zipCode: zipCode != null ? Value(zipCode) : const Value.absent(),
      website: website != null ? Value(website) : const Value.absent(),
      company: company != null ? Value(company) : const Value.absent(),
      position: position != null ? Value(position) : const Value.absent(),
    );
    
    if (existing == null) {
      // Create profile
      final id = await db.into(db.userProfiles).insert(
        UserProfilesCompanion.insert(
          userId: userId,
          fullName: fullName != null ? Value(fullName) : const Value.absent(),
          bio: bio != null ? Value(bio) : const Value.absent(),
          avatar: avatar != null ? Value(avatar) : const Value.absent(),
          phone: phone != null ? Value(phone) : const Value.absent(),
          birthDate: birthDate != null ? Value(birthDate) : const Value.absent(),
          address: address != null ? Value(address) : const Value.absent(),
          city: city != null ? Value(city) : const Value.absent(),
          state: state != null ? Value(state) : const Value.absent(),
          country: country != null ? Value(country) : const Value.absent(),
          zipCode: zipCode != null ? Value(zipCode) : const Value.absent(),
          website: website != null ? Value(website) : const Value.absent(),
          company: company != null ? Value(company) : const Value.absent(),
          position: position != null ? Value(position) : const Value.absent(),
        ),
      );
      
      return await db.getUserProfile(id);
    }
    
    // Update existing
    await (db.update(db.userProfiles)..where((p) => p.userId.equals(userId)))
      .write(companion);
    
    return await db.getUserProfile(existing.id);
  }

  // ==================== USER SETTINGS MANAGEMENT ====================
  
  // 👇 Update user settings
  Future<UserSettings> updateUserSettings({
    required int userId,
    String? theme,
    String? language,
    bool? notifications,
    bool? emailNotifications,
    bool? pushNotifications,
  }) async {
    final existing = await (db.select(db.userSettings)
      ..where((s) => s.userId.equals(userId)))
      .getSingleOrNull();
    
    final companion = UserSettingsCompanion(
      theme: theme != null ? Value(theme) : const Value.absent(),
      language: language != null ? Value(language) : const Value.absent(),
      notifications: notifications != null 
          ? Value(notifications) 
          : const Value.absent(),
      emailNotifications: emailNotifications != null 
          ? Value(emailNotifications) 
          : const Value.absent(),
      pushNotifications: pushNotifications != null 
          ? Value(pushNotifications) 
          : const Value.absent(),
    );
    
    if (existing == null) {
      // Create settings
      final id = await db.into(db.userSettings).insert(
        UserSettingsCompanion.insert(
          userId: userId,
          theme: theme != null ? Value(theme) : const Value.absent(),
          language: language != null ? Value(language) : const Value.absent(),
          notifications: notifications != null 
              ? Value(notifications) 
              : const Value.absent(),
          emailNotifications: emailNotifications != null 
              ? Value(emailNotifications) 
              : const Value.absent(),
          pushNotifications: pushNotifications != null 
              ? Value(pushNotifications) 
              : const Value.absent(),
        ),
      );
      
      return await db.getUserSettings(id);
    }
    
    await (db.update(db.userSettings)..where((s) => s.userId.equals(userId)))
      .write(companion);
    
    return await db.getUserSettings(existing.id);
  }

  // ==================== COMPLETE USER QUERIES ====================
  
  // 👇 Get users with complete data
  Future<List<CompleteUserData>> getAllUsersWithCompleteData() async {
    final users = await db.select(db.users)
      .where((u) => u.isActive.equals(true))
      .get();
    
    final result = <CompleteUserData>[];
    
    for (final user in users) {
      final completeData = await getCompleteUserData(user.id);
      result.add(completeData);
    }
    
    return result;
  }
  
  // 👇 Search users with profile
  Future<List<UserWithProfile>> searchUsersWithProfile({
    String? query,
    String? city,
    String? country,
  }) async {
    final queryBuilder = db.select(db.users).join([
      innerJoin(
        db.userProfiles,
        db.userProfiles.userId.equals(db.users.id),
      )
    ]);
    
    if (query != null && query.isNotEmpty) {
      queryBuilder.where((u) => 
        db.users.username.like('%$query%') |
        db.userProfiles.fullName.like('%$query%')
      );
    }
    
    if (city != null && city.isNotEmpty) {
      queryBuilder.where((p) => db.userProfiles.city.equals(city));
    }
    
    if (country != null && country.isNotEmpty) {
      queryBuilder.where((p) => db.userProfiles.country.equals(country));
    }
    
    queryBuilder.where((u) => db.users.isActive.equals(true));
    queryBuilder.orderBy([(u) => OrderingTerm.asc(db.users.username)]);
    
    final results = await queryBuilder.get();
    
    return results.map((row) {
      return UserWithProfile(
        user: row.readTable(db.users),
        profile: row.readTable(db.userProfiles),
      );
    }).toList();
  }

  // ==================== USER STATISTICS ====================
  
  // 👇 Get user statistics
  Future<Map<String, dynamic>> getUserStats() async {
    final totalUsers = await db.select(db.users).count();
    
    final profilesCount = await db.select(db.userProfiles).count();
    final settingsCount = await db.select(db.userSettings).count();
    
    final activeUsers = await (db.select(db.users)
      ..where((u) => u.isActive.equals(true)))
      .count();
    
    final verifiedUsers = await (db.select(db.users)
      ..where((u) => u.isVerified.equals(true)))
      .count();
    
    return {
      'totalUsers': totalUsers,
      'activeUsers': activeUsers,
      'verifiedUsers': verifiedUsers,
      'profilesCompleted': profilesCount,
      'settingsConfigured': settingsCount,
      'completionRate': totalUsers > 0 
          ? (profilesCount / totalUsers) * 100 
          : 0.0,
    };
  }

  // ==================== HELPER METHODS ====================
  
  String _hashPassword(String password) {
    // Simulate password hashing
    return 'hashed_$password';
  }
}

// ==================== DATA CLASSES ====================

class UserWithProfile {
  final User user;
  final UserProfile profile;
  
  UserWithProfile({
    required this.user,
    required this.profile,
  });
}

class UserWithOptionalProfile {
  final User user;
  final UserProfile? profile;
  
  UserWithOptionalProfile({
    required this.user,
    this.profile,
  });
}

class CompleteUserData {
  final User user;
  final UserProfile? profile;
  final UserSettings? settings;
  
  CompleteUserData({
    required this.user,
    this.profile,
    this.settings,
  });
}
```

---

# One-to-One vs Other Relationships

| Feature | One-to-One | One-to-Many | Many-to-Many |
|---------|------------|-------------|--------------|
| **Cardinality** | 1:1 | 1:N | N:M |
| **Unique Constraint** | Required | Not required | Not required |
| **Foreign Key** | Unique | Non-unique | Junction table |
| **Use Case** | User-Profile | User-Posts | User-Roles |

---

# Best Practices

- **Use unique constraint** – Enforce one-to-one
- **Use foreign key** – Maintain integrity
- **Use cascade delete** – Clean up related data
- **Use left join** – For optional relationships
- **Use inner join** – For required relationships
- **Separate concerns** – Split large tables
- **Index foreign key** – For performance
- **Use transactions** – For multiple operations

---

# Common Mistakes

## Mistake 1: Forgetting unique constraint

Wrong:
```dart
// 🚫 Multiple profiles per user possible
IntColumn get userId => integer().references(Users, #id)();
```

Correct:
```dart
// ✅ One-to-one enforced
IntColumn get userId => integer()
  .references(Users, #id)
  .unique()();
```

## Mistake 2: Wrong join type

Wrong:
```dart
// 🚫 Only users with profiles
innerJoin(db.profiles, ...)
```

Correct:
```dart
// ✅ All users (with or without profiles)
leftJoin(db.profiles, ...)
```

## Mistake 3: Not handling optional data

Wrong:
```dart
// 🚫 Crashes if no profile
final profile = row.readTable(db.profiles);
print(profile.fullName);
```

Correct:
```dart
// ✅ Handle null
final profile = row.readTableOrNull(db.profiles);
if (profile != null) {
  print(profile.fullName);
}
```

---

# Summary

| Feature | Description | Key Point |
|---------|-------------|-----------|
| **Unique Constraint** | Enforces 1:1 | `unique()` |
| **Foreign Key** | Links tables | `references()` |
| **Cascade Delete** | Auto cleanup | `onDelete: KeyAction.cascade` |
| **Join Type** | Inner or Left | Required or optional |

---

# Next Steps

Now you understand one-to-one, let's dive deeper:

- [Mapping Joined Results](link) – Result mapping
- [Streams](link) – Reactive queries
- [Watching Queries](link) – Real-time updates

---

# Did You Know?

- **One-to-one is the least common relationship** – In databases

- **One-to-one can improve performance** – By splitting tables

- **One-to-one can improve security** – By separating sensitive data

- **One-to-one relationships are often optional** – One side may be null

- **One-to-one can be bidirectional** – Both tables reference each other

- **One-to-one is useful for** – User profiles, settings, preferences

- **One-to-one can be merged** – Into a single table

- **One-to-one is supported** – By unique foreign key constraint

---

