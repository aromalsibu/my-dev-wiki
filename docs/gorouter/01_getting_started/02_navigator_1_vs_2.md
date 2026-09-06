# Navigator 1.0 vs Navigator 2.0

## Overview

Flutter provides **two distinct navigation systems**:

| | **Navigator 1.0** | **Navigator 2.0** |
|---|---|---|
| **Paradigm** | Imperative | Declarative |
| **Approach** | Push/pop commands | State-driven page list |
| **Control** | You tell the navigator *what to do* | You tell the navigator *what to display* |
| **URL Sync** | Manual or impossible | Automatic |
| **Complexity** | Simple | Powerful but verbose |

> **Terminology Note**: The Flutter team originally called this "Navigator 2.0," but it is now officially referred to as the **Router API**. However, "Navigator 2.0" remains the commonly used term in the community and is the name you'll encounter in most articles and discussions.

**Both systems are valid and still used in production**. The key is choosing the right one for your app's scale and requirements.

---

## Why It Exists

### The Limitations of Navigator 1.0

Navigator 1.0 is **imperative**—you explicitly tell the navigator to push or pop routes. While simple and effective for basic apps, it struggles with modern application needs:

**1. Not Extensible**
The standard Navigator provides a fixed set of methods. You cannot add custom behavior or modify existing navigation patterns.

**2. No URL Synchronization**
On the web, the browser URL doesn't automatically reflect the current screen. This breaks bookmarks, the back button, and SEO.

**3. Deep Linking is Painful**
Consider a deep link like `/user/{friendId}/post/{postId}`. With Navigator 1.0, you must manually construct the entire navigation stack—pushing the profile, then the post—and handle edge cases like what happens when the user presses back.

**4. Navigation State is Implicit**
The navigation stack is a side effect of push/pop commands, not a declarative description of what screens should exist. This makes it hard to:
- Restore navigation state after app restart
- Test navigation logic
- Reason about the complete app state from a single source of truth

**5. Complex Flows Become Fragile**
Conditional navigation (authentication checks, onboarding flows) requires duplicating logic across screens.

### The Promise of Navigator 2.0

Navigator 2.0 introduces a **declarative** approach where navigation is a function of **app state**. Instead of issuing commands, you describe what screens should exist, and Flutter updates the UI accordingly.

This aligns with Flutter's core **declarative UI philosophy**—just as your UI is a function of state, so is your navigation stack.

---

## Syntax / Basic Usage

### Navigator 1.0: Imperative Push/Pop

```dart
// 1. Define routes in MaterialApp
MaterialApp(
  routes: {
    '/': (context) => const HomeScreen(),
    '/details': (context) => const DetailsScreen(),
  },
);

// 2. Navigate imperatively
// Push a new screen onto the stack
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => const DetailsScreen()),
);

// Or use named routes
Navigator.pushNamed(context, '/details');

// 3. Pop to return
Navigator.pop(context);
```

### Navigator 2.0: Declarative Page List

```dart
// 1. Navigation is driven by state
class MyApp extends StatefulWidget {
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  bool _showDetails = false;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Navigator(
        // The pages list describes what should be displayed
        pages: [
          const MaterialPage(child: HomeScreen()),
          if (_showDetails) const MaterialPage(child: DetailsScreen()),
        ],
        onPopPage: (route, result) {
          // Handle back navigation by updating state
          if (!route.didPop(result)) return false;
          setState(() {
            _showDetails = false;
          });
          return true;
        },
      ),
    );
  }
}
```

### Code Breakdown (Navigator 2.0 Example)

| Line/Block | Explanation |
|------------|-------------|
| `bool _showDetails` | State variable that controls whether the details screen is visible. |
| `Navigator(pages: [...])` | The `Navigator` widget now takes a `pages` list instead of using push/pop commands. |
| `if (_showDetails) const MaterialPage(...)` | The page list is built declaratively—when `_showDetails` is `true`, the details page appears in the stack. |
| `onPopPage` | Callback triggered when the user presses the back button. You must update state to reflect the new stack. |
| `setState(() { _showDetails = false; })` | Updating state causes the `pages` list to rebuild, removing the details page. |

> **Note**: This example is intentionally minimal to illustrate the core concept. In practice, you'd use `Router` and `RouterDelegate` for full control, which involves significantly more boilerplate.

---

## Mental Model

### The Stack vs. The Blueprint

**Navigator 1.0 is like a stack of physical cards:**

```
   ┌─────────┐
   │ Details │  ← You push a card on top
   ├─────────┤
   │  Home   │  ← You pop to remove the top card
   └─────────┘
```

You tell the system *what action* to take (push or pop), and it mutates the stack.

**Navigator 2.0 is like a blueprint you hand to a builder:**

```
   ┌─────────────────────────────────┐
   │  "Build this stack of screens"  │
   │  ┌─────────┐                    │
   │  │ Details │  ← Part of the     │
   │  ├─────────┤     blueprint      │
   │  │  Home   │                    │
   │  └─────────┘                    │
   └─────────────────────────────────┘
                │
                ▼
         ┌───────────┐
         │  Builder  │  ← Reconciles the
         │ (Flutter) │     blueprint with
         └───────────┘     what's displayed
```

You describe the desired stack (the "blueprint"), and Flutter figures out what changed and animates the transition.

---

## Core Concepts

### Navigator 1.0

| Concept | Description |
|---------|-------------|
| **Navigator** | A widget that manages a stack of `Route` objects. |
| **Route** | An object representing a screen, typically `MaterialPageRoute` or `CupertinoPageRoute`. |
| **Push** | Add a new route to the top of the stack. |
| **Pop** | Remove the top route from the stack. |
| **Named Routes** | Routes defined with string names in `MaterialApp.routes`. |

### Navigator 2.0

| Concept | Description |
|---------|-------------|
| **Router** | The widget that provides the foundation for Navigator 2.0 navigation. |
| **RouterDelegate** | Controls the `Navigator`'s `pages` list based on app state. |
| **RouteInformationParser** | Parses incoming URLs (from the browser or deep links) into a custom data type. |
| **Page** | An immutable object describing a screen in the navigation stack. |
| **Declarative** | You describe *what* to display; the framework handles *how* to get there. |

---

## How It Works

### Navigator 1.0 Workflow

```
User taps button
       │
       ▼
Navigator.push(context, route)
       │
       ▼
Route is added to stack
       │
       ▼
New screen animates in
       │
       ▼
Stack: [Home, Details]
```

The navigation stack is mutated imperatively. The UI is a **side effect** of the push/pop commands.

### Navigator 2.0 Workflow

```
App state changes
       │
       ▼
pages list is rebuilt
       │
       ▼
Navigator compares old vs new pages
       │
       ▼
Flutter animates the difference
       │
       ▼
UI reflects the new state
```

The UI is a **direct function** of the `pages` list. When state changes, the pages list changes, and the navigator reconciles the difference.

---

## Comparison Table

| Aspect | Navigator 1.0 | Navigator 2.0 |
|--------|---------------|---------------|
| **Paradigm** | Imperative | Declarative |
| **Navigation Control** | Push/pop commands | State-driven page list |
| **URL Sync (Web)** | Manual or not supported | Automatic |
| **Deep Linking** | Difficult, requires manual stack construction | Built-in support |
| **Conditional Routes** | Logic scattered across screens | Centralized in state |
| **State Restoration** | Manual | Built-in |
| **Nested Navigation** | Complex | Supported via `ShellRoute` |
| **Boilerplate** | Minimal | Significant (unless using GoRouter) |
| **Learning Curve** | Low | Steep |
| **Best For** | Small to medium apps, mobile-only | Large apps, web, deep linking |

---

## When to Use

### Use Navigator 1.0 When:

- You're building a **small to medium app** with a simple navigation flow.
- You **don't need deep linking**.
- You **don't support Flutter Web** (or don't need URL synchronization).
- You have **few conditional routes**.
- You want **minimal setup** and a low learning curve.

Most mobile-only apps work perfectly well with Navigator 1.0.

### Use Navigator 2.0 When:

- You're building a **Flutter Web app** that needs browser URL synchronization.
- You need **deep linking** (app links, universal links).
- Your navigation **depends on app state** (authentication, user roles, feature flags).
- You have **complex navigation flows** with nested routes or tabs.
- You need **navigation state restoration** after app restart.

---

## When Not to Use Navigator 2.0 (Directly)

**Do not use raw Navigator 2.0 for most apps.** The raw API requires implementing `RouterDelegate` and `RouteInformationParser` manually, which involves **significant boilerplate**. The official Flutter team and community **recommend using `go_router`** instead—a package that sits on top of Navigator 2.0 and handles the boilerplate for you.

Use raw Navigator 2.0 directly only if:
- You have **very specific, custom navigation requirements** that GoRouter doesn't support.
- You're building a **routing library** for others to use.
- You're **contributing to the Flutter framework** itself.

For almost all real-world apps, **GoRouter is the recommended way to access Navigator 2.0's power**.

---

## Summary

- **Navigator 1.0** is **imperative** and **stack-based**—you push and pop routes. Simple, reliable, and perfect for many apps.
- **Navigator 2.0** is **declarative** and **state-driven**—you describe what screens should exist. Powerful, flexible, and essential for complex routing.
- **Navigator 2.0** solves real problems: URL sync, deep linking, conditional navigation, and state restoration.
- The raw Navigator 2.0 API is **verbose and complex**—use **GoRouter** instead.
- **Choose based on your app's needs**, not trends. Both systems are valid and used in production.
- For **most Flutter projects in 2024 and beyond, GoRouter is the best choice**.

---

