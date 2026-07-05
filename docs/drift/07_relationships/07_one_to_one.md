# One-to-One

A one-to-one relationship allows one row in a table to be associated with exactly one row in another table.

---

# What is it?

In a one-to-one relationship:

* One record has only one related record.
* The related record belongs to only one parent.

Unlike a one-to-many relationship, neither side can have multiple associated rows.

This is typically implemented using a **foreign key** with a **unique constraint**.

---

# Why does it exist?

Sometimes it doesn't make sense to store all information in a single table.

For example:

* A user has one profile.
* An employee has one ID card.
* A product has one inventory record.

Splitting related data into separate tables keeps the database organized while maintaining a one-to-one relationship.

---

# Syntax

## Defining the Relationship

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();
}

class UserProfiles extends Table {
  IntColumn get id => integer().autoIncrement()();

  IntColumn get userId =>
      integer()
          .references(Users, #id)
          .unique()();

  TextColumn get bio => text().nullable()();

  TextColumn get avatarUrl => text().nullable()();
}
```

Explanation:

* `userId` references the `Users` table.
* `.unique()` ensures that each user can have only one profile.
* Together, the foreign key and unique constraint create a one-to-one relationship.

---

## Querying Related Data

```dart
final query = select(users).join([
  leftOuterJoin(
    userProfiles,
    userProfiles.userId.equalsExp(users.id),
  ),
]);

final rows = await query.get();
```

Explanation:

* `leftOuterJoin()` returns every user.
* Profile information is included when it exists.

---

# Mental Model

Think of two records connected by a single link.

```text
User

Alice
 │
 ▼
Profile

Bio
Avatar
```

Each user has one profile, and each profile belongs to one user.

---

# Examples

## Simple Example

A passport belongs to one person.

```text
Person
   │
   ▼
Passport
```

A passport cannot belong to multiple people, and a person has only one passport.

---

## Real-World Example

Load every user along with their profile.

```dart
final query = select(users).join([
  leftOuterJoin(
    userProfiles,
    userProfiles.userId.equalsExp(users.id),
  ),
]);

final rows = await query.get();

for (final row in rows) {
  final user = row.readTable(users);
  final profile = row.readTableOrNull(userProfiles);

  print('User: ${user.name}');
  print('Bio: ${profile?.bio ?? "No profile yet"}');
}
```

Explanation:

* `leftOuterJoin()` ensures every user is returned.
* `readTableOrNull()` safely handles users without profiles.
* Each result contains one user and, if available, one profile.

---

# When to Use

Use a one-to-one relationship for:

* Users and profiles.
* Employees and ID cards.
* Products and inventory records.
* Accounts and account settings.
* Customers and loyalty cards.

---

# When NOT to Use

Avoid a one-to-one relationship when:

* A parent can have multiple children (use one-to-many).
* Both tables can reference multiple rows (use many-to-many).
* The related data always exists and is always accessed together—in that case, a single table may be simpler.

---

# Best Practices

* Use a foreign key with a unique constraint.
* Join related tables instead of duplicating data.
* Use `leftOuterJoin()` if the related record is optional.
* Index foreign key columns.
* Split data only when it improves organization or access patterns.

---

# Common Mistakes

## Forgetting the unique constraint

**Wrong**

```dart
IntColumn get userId =>
    integer().references(Users, #id)();
```

Explanation:

* Multiple profiles could reference the same user.
* The relationship becomes one-to-many instead of one-to-one.

**Correct**

```dart
IntColumn get userId =>
    integer()
        .references(Users, #id)
        .unique()();
```

Explanation:

* `.unique()` guarantees only one profile per user.

---

## Duplicating user information

**Wrong**

```text
UserProfiles

User Name
User Email
Bio
Avatar
```

Explanation:

* User information is duplicated across tables.
* Updating the user's name requires updating multiple records.

**Correct**

Store only the `userId` in the profile table and retrieve user details through a join.

---

## Using an inner join for optional relationships

**Wrong**

```dart
innerJoin(
  userProfiles,
  userProfiles.userId.equalsExp(users.id),
)
```

Explanation:

* Users without profiles are excluded.

**Correct**

```dart
leftOuterJoin(
  userProfiles,
  userProfiles.userId.equalsExp(users.id),
)
```

Explanation:

* Every user is returned, even if they haven't created a profile yet.

---

# Related APIs

* One-to-Many
* Many-to-Many
* Foreign Keys
* Inner Join
* Left Join
* Mapping Joined Results

---

# Summary

A one-to-one relationship connects exactly one record in one table with exactly one record in another. In Drift, it is implemented using a foreign key combined with a unique constraint. This pattern is ideal for separating optional or specialized data—such as user profiles or account settings—while preserving data integrity and keeping the database normalized.
