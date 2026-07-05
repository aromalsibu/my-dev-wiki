# Cross Join

A cross join combines every row from one table with every row from another table.

---

# What is it?

A cross join returns the **Cartesian product** of two tables.

This means every row in the first table is paired with every row in the second table.

Unlike inner joins and left joins, a cross join does **not** require a join condition.

---

# Why does it exist?

Sometimes you need every possible combination of two datasets.

For example:

* Every product with every available color.
* Every employee with every work shift.
* Every size with every material.
* Generating schedules or combinations.

A cross join automatically produces these combinations.

---

# Syntax

## Performing a Cross Join

```dart
final query = select(products).join([
  crossJoin(colors),
]);

final results = await query.get();
```

Explanation:

* `crossJoin()` combines every product with every color.
* No join condition is required.

---

## Reading Joined Rows

```dart
for (final row in results) {
  final product = row.readTable(products);
  final color = row.readTable(colors);

  print('${product.name} - ${color.name}');
}
```

Explanation:

* Each result contains one product and one color.
* Every possible pairing is returned.

---

# Mental Model

Think of creating every possible outfit.

```text
Products          Colors

Shirt             Red
Shoes             Blue

↓

Results

Shirt + Red
Shirt + Blue
Shoes + Red
Shoes + Blue
```

Each item from the first table is matched with every item from the second table.

---

# Examples

## Simple Example

Generate every size for every product.

```dart
final query = select(products).join([
  crossJoin(sizes),
]);

final rows = await query.get();
```

Explanation:

* Every product is paired with every size.
* The result contains all possible combinations.

---

## Real-World Example

Generate employee shift assignments.

```dart
final query = select(employees).join([
  crossJoin(shifts),
]);

final rows = await query.get();
```

Explanation:

* Every employee is paired with every available shift.
* Useful when generating scheduling options.

---

# When to Use

Use a cross join when you need to:

* Generate every possible combination.
* Create scheduling permutations.
* Build product variation matrices.
* Produce combinational datasets.
* Perform analytical queries involving Cartesian products.

---

# When NOT to Use

Avoid a cross join when:

* Tables are related through foreign keys.
* You only want matching records.
* The resulting dataset would become excessively large.

For related tables, use an inner join or left join instead.

---

# Best Practices

* Use cross joins only when all combinations are genuinely required.
* Be aware that the result size grows rapidly as table sizes increase.
* Apply filtering after the join if appropriate.
* Avoid cross joins on large tables unless necessary.
* Estimate the expected number of rows before executing the query.

---

# Common Mistakes

## Using a cross join instead of an inner join

**Wrong**

```dart
crossJoin(users)
```

When `orders` should be matched to their owners.

Explanation:

* Every order is paired with every user.
* Most of the results are meaningless.

**Correct**

```dart
innerJoin(
  users,
  users.id.equalsExp(orders.userId),
)
```

Explanation:

* Only related rows are returned.

---

## Underestimating the result size

**Wrong**

Cross joining:

* 10,000 products
* 5,000 colors

Explanation:

* The query produces **50 million** rows.
* This can severely impact performance.

**Correct**

Estimate the number of combinations before using a cross join and add filters where possible.

---

## Assuming a cross join finds relationships

**Wrong**

Expecting matching foreign keys automatically.

Explanation:

* A cross join has no join condition.
* It simply combines every row with every other row.

**Correct**

Use inner or left joins when relationships should determine the results.

---

# Related APIs

* Inner Join
* Left Join
* Self Join
* Mapping Joined Results
* Filtering

---

# Summary

A cross join creates the Cartesian product of two tables by pairing every row from one table with every row from the other. It is useful for generating combinations and analytical datasets but should be used carefully, as the number of returned rows grows quickly with the size of the joined tables.
