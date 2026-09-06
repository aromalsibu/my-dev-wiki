# Why GoRouter Exists

## Overview

**GoRouter** exists because Flutter's raw Navigator 2.0 API, while powerful, is **too complex for everyday use**. GoRouter is the **pragmatic wrapper** that makes declarative routing accessible to every Flutter developer.

The purpose of the `go_router` package is to **use declarative routes to reduce complexity**, regardless of the platform you're targeting (mobile, web, desktop), handle deep and dynamic linking from Android, iOS and the web, along with a number of other navigation-related scenarios, while still providing an **easy-to-use developer experience**.

In essence: **GoRouter gives you Navigator 2.0's power without Navigator 2.0's pain**.

---

## Why It Exists

### The Problem: Navigator 2.0 is Too Hard

Flutter's Navigator 2.0 (the Router API) solved real problems—URL synchronization, deep linking, and declarative navigation—but introduced a **new problem**: **developer complexity**.

To implement Navigator 2.0 correctly, you need to:

1. Implement a custom `RouterDelegate` that manages the `Navigator`'s page stack
2. Implement a `RouteInformationParser` that parses URLs into a custom data type
3. Wire everything together with `MaterialApp.router`
4. Handle edge cases like back navigation, state restoration, and URL updates
5. Keep everything in sync with your app's state

The result is **hundreds of lines of boilerplate** just to get basic routing working. This is the "complex structure" of Navigator 2.0 that GoRouter simplifies.

**Consider this**: The raw Navigator 2.0 API requires you to manage the navigation stack as a list of `Page` objects and manually handle `onPopPage` callbacks. Any mistake can break back navigation, state restoration, or deep linking.

### The Previous Limitation: No Middle Ground

Before GoRouter, Flutter developers faced a **binary choice**:

| Option | Pros | Cons |
|--------|------|------|
| **Navigator 1.0** | Simple, familiar, works | No URL sync, no deep linking, no declarative state |
| **Navigator 2.0 (raw)** | Powerful, declarative, full control | Extremely complex, verbose, error-prone |

There was **no middle ground**—no solution that gave you the benefits of Navigator 2.0 without the complexity. This gap is exactly what GoRouter fills.

> **Key Insight**: The Flutter team recognized that developers were avoiding Navigator 2.0 because of its complexity. GoRouter was created to **remove that barrier** and make declarative routing the default choice.

---

## How It Works: The Simplification

### What GoRouter Does For You

GoRouter handles **all the Navigator 2.0 boilerplate** behind the scenes:

```dart
// Without GoRouter: You write all of this yourself
class MyRouterDelegate extends RouterDelegate<MyRoutePath> {
  // 100+ lines of code managing pages, pop callbacks, and state
}

class MyRouteInformationParser extends RouteInformationParser<MyRoutePath> {
  // 50+ lines of code parsing URLs
}

// With GoRouter: You just define routes
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: (context, state) => const HomeScreen()),
    GoRoute(path: '/details', builder: (context, state) => const DetailsScreen()),
  ],
);
```

GoRouter internally provides:
- A fully functional `RouterDelegate`
- A complete `RouteInformationParser`
- URL parsing and matching logic
- Navigation stack management
- History synchronization
- Deep linking support

All of this is **abstracted away** so you can focus on **what matters**: defining your app's routes.

### The Declarative Promise

GoRouter embodies Flutter's **declarative philosophy**:

> **Your navigation is a function of your app's state, not a sequence of commands.**

Instead of imperatively pushing and popping screens:

```dart
// Imperative (Navigator 1.0)
Navigator.push(context, MaterialPageRoute(builder: (_) => DetailsScreen()));

// Declarative (GoRouter)
context.go('/details');
```

The URL becomes the **single source of truth** for navigation state. When the URL changes, the UI updates automatically.

---

## Core Benefits

### 1. URL-Based Navigation

GoRouter provides a **URL-based API** that works consistently across all platforms:

```dart
// Navigate using paths—just like a web router
context.go('/products/123');
context.go('/profile?tab=settings');
```

This works on **mobile, web, and desktop** with the same API.

### 2. Deep Linking Made Simple

Deep linking—opening your app from a URL—is **built-in**:

```dart
// Define a route with parameters
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id']!;
    return ProductScreen(id: id);
  },
)

// The URL /product/42 automatically opens the ProductScreen with id=42
```

No manual parsing. No stack construction. Just **define and forget**.

### 3. Declarative Route Configuration

Your entire route tree is defined in **one place**:

```dart
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: ...),
    GoRoute(
      path: '/auth',
      builder: ...,
      routes: [
        GoRoute(path: 'login', builder: ...),
        GoRoute(path: 'register', builder: ...),
      ],
    ),
  ],
);
```

This centralization makes routing **predictable, testable, and maintainable**.

### 4. Web Support Out of the Box

On the web, GoRouter **automatically synchronizes** the browser URL with your app's navigation state:

- The URL updates when you navigate
- The back button works correctly
- Page refreshes restore the correct state
- URLs are shareable and bookmarkable

### 5. Clean, Intuitive API

GoRouter provides a **developer-friendly API** that feels natural:

```dart
// Navigate
context.go('/settings');
context.push('/details/42');
context.pop();

// With parameters
context.go('/user/123?tab=posts');

// Named routes (type-safe alternative)
context.goNamed('profile', pathParameters: {'id': '123'});
```

---

## Mental Model

### The "Web Router for Flutter" Analogy

If you've used **React Router, Express, or Flask**, GoRouter will feel familiar:

```
┌─────────────────────────────────────────────────────────────┐
│                    Your App State                          │
│                                                             │
│  Current URL: /dashboard/profile/123?view=detailed        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    GoRouter Engine                         │
│                                                             │
│  1. Parse URL into segments                                │
│  2. Match against route tree                               │
│  3. Extract parameters (id=123, view=detailed)            │
│  4. Build the page stack                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Flutter UI                              │
│                                                             │
│  [Dashboard] → [Profile] (with id=123, detailed view)    │
└─────────────────────────────────────────────────────────────┘
```

### The "Declarative vs Imperative" Mindset

| Imperative (Navigator 1.0) | Declarative (GoRouter) |
|----------------------------|------------------------|
| "Push this screen" | "The URL is now /details" |
| "Pop the top screen" | "The URL is now /" |
| "Replace with this screen" | "The URL is now /settings" |

With GoRouter, you **change the URL** and let the router figure out **what to display**. This is the same mental model as the web.

---

## The Official Status

GoRouter is **maintained by the Flutter team** and is the **recommended solution** for declarative routing.

> **Note**: As of 2025, GoRouter has entered maintenance mode but **remains the most practical and widely adopted solution** for Flutter navigation. The Flutter community and the official team continue to support it as the primary routing solution for most apps.

The latest version (as of May 2026) is **v17.2.3**, demonstrating active maintenance and continued relevance.

---

## When to Use GoRouter

**Use GoRouter when:**
- You want **declarative, URL-based navigation**
- You need **deep linking** (app links, universal links)
- You're building for **Flutter Web** and need URL sync
- You have **complex navigation** (nested routes, tabs, authentication)
- You want **clean, maintainable routing code**
- You're starting a **new project** and want the best routing solution

**The verdict**: For **most Flutter projects in 2026, GoRouter is the best choice**.

---

## When Not to Use GoRouter

**Consider alternatives when:**
- Your app has **only 1-2 screens** with no navigation complexity
- You're **deeply invested** in another routing solution (like `auto_route`)
- You need **low-level control** that GoRouter doesn't expose

Even then, the raw Navigator 2.0 API is the only alternative if you need full control.

---

## Summary

- **GoRouter exists** because Navigator 2.0 is too complex for everyday use
- It **wraps Navigator 2.0** and handles all the boilerplate
- It provides a **URL-based, declarative API** that works on all platforms
- It **simplifies deep linking**, web support, and complex navigation
- It's **maintained by the Flutter team** and is the recommended solution
- It bridges the gap between **Navigator 1.0's simplicity** and **Navigator 2.0's power**

> **Bottom Line**: GoRouter is the **pragmatic choice**—it gives you everything you need from Navigator 2.0 without the complexity that made developers avoid it.

---

