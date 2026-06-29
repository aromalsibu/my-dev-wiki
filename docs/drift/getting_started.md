# Getting Started

## What is Drift?

Drift is a reactive persistence library for Dart and Flutter built on top of SQLite.

It provides a type-safe, Dart-first API for defining database schemas, writing queries, and interacting with SQLite without manually writing most SQL.

Unlike using SQLite directly, Drift generates strongly typed Dart code from your schema, making database operations easier to write, safer, and less error-prone.

---

## Why Drift?

SQLite is a powerful embedded database, but working with it directly often involves:

- Writing raw SQL strings
- Manually converting rows into Dart objects
- Handling database updates yourself
- Managing schema changes manually

Drift solves these problems by providing:

- Type-safe queries
- Generated data classes
- Compile-time validation
- Reactive database streams
- Migration support
- Easy integration with Flutter

This allows you to focus on your application's data rather than low-level database management.

---

## How Drift Works

At a high level, Drift sits between your Flutter application and SQLite.

```text
Flutter App
      │
      ▼
Business Logic
      │
      ▼
Drift
      │
      ▼
SQLite
```

You define your database schema in Dart, Drift generates the required code, and your application interacts with the generated API instead of raw SQL.

...
