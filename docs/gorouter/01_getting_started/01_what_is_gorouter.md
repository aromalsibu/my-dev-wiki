# GoRouter

## Overview

**GoRouter** is a declarative routing package for Flutter that provides a robust, URL-based navigation system built on top of the **Navigator 2.0** API. It simplifies complex navigation scenarios—such as deep linking, nested routes, authentication flows, and web URL synchronization—by allowing you to define your app's navigation structure as a **route tree** using a simple, intuitive configuration.

**Purpose:**  
- Manage navigation declaratively, rather than imperatively pushing and popping pages.  
- Keep the browser URL (on web) and app state in sync automatically.  
- Handle deep links and app links with minimal boilerplate.  
- Provide a unified API for all platforms (mobile, web, desktop).  
- Enable advanced patterns like nested navigation, tabs, and route guards.

**Where It Fits:**  
GoRouter sits above Flutter's `Navigator` API, acting as a routing **orchestrator**. It consumes a route configuration—a tree of `GoRoute` objects—and maps URLs to widget screens. It integrates with Flutter's `MaterialApp.router` or `CupertinoApp.router` constructors, replacing the traditional `routes` parameter and imperative `Navigator.push()` calls.

---

## Why It Exists

### The Problem with Navigator 1.0

Flutter's original routing system (Navigator 1.0) is **imperative** and **stack-based**. You push and pop routes using `Navigator.push(context, MaterialPageRoute(...))`. While simple, this approach has severe limitations as apps grow:

- **No URL synchronization**: The browser address bar (on web) doesn't reflect the current screen, breaking bookmarks and the back button.
- **Difficult deep linking**: Handling deep links requires manually parsing URIs and pushing multiple routes in the correct order.
- **No declarative state**: Navigation is driven by code, not by state; you cannot easily represent the navigation stack as a data model.
- **Nested navigation complexity**: Managing multiple `Navigator` instances for tabs or sub-screens is cumbersome.
- **Hard to handle redirects**: Authentication checks must be repeated in every screen; no global redirect logic.

### The Promise of Navigator 2.0

Flutter introduced **Navigator 2.0** to address these issues by making navigation **declarative** and **state-driven**. Instead of pushing pages, you provide a `Page` list to a `Navigator` widget, and the framework reconciles changes. You can also provide a `Router` delegate that listens to platform navigation events (like browser back) and maps them to your app state.

However, **Navigator 2.0's raw API is complex**:

- You must implement `RouterDelegate` and `RouteInformationParser`.
- Managing the page stack manually involves deep understanding of `Page` objects and their keys.
- Handling the browser URL and history requires parsing and serializing `RouteInformation`.
- It's easy to make mistakes with state restoration and synchronization.

### Why GoRouter Was Introduced

GoRouter was created to **wrap and simplify Navigator 2.0**, providing:

- **Declarative route definitions** as a tree of `GoRoute` objects, much like a web router.
- **Automatic URL mapping**: Routes are defined with paths (`/user/:id`), and GoRouter matches the current URL to the corresponding screen.
- **State-driven navigation**: You navigate by setting a new location (URL), and GoRouter updates the page stack automatically.
- **Built-in support for redirects**: Define `redirect` logic on routes to guard access or reroute.
- **Nested navigation** with `ShellRoute` and `StatefulShellRoute` for tabs and sub-navigators.
- **Seamless web integration**: Synchronizes with the browser's URL, history, and refresh.
- **Simple API**: A single `GoRouter` object configures everything; use `context.go()` or `context.push()` to navigate, rather than cumbersome `Navigator` calls.

GoRouter effectively bridges the gap between the **intent** of Navigator 2.0 and the **practicality** of implementing it.

---

## Syntax / Basic Usage

Here is the smallest useful GoRouter setup:

```dart
// 1. Define your route configuration.
final GoRouter _router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/details/:id',
      builder: (context, state) {
        // Access the path parameter :id
        final id = state.pathParameters['id']!;
        return DetailsScreen(id: id);
      },
    ),
  ],
);

// 2. Use MaterialApp.router instead of MaterialApp.
void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router, // Provide the GoRouter instance
      title: 'GoRouter Demo',
    );
  }
}

// 3. Navigate using context extensions.
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Navigate to details with id=42
            context.go('/details/42');
          },
          child: Text('Go to Details'),
        ),
      ),
    );
  }
}
```

### Code Breakdown

| Line/Block | Explanation |
|------------|-------------|
| `GoRouter(routes: [...])` | Creates the router configuration. `routes` is a list of `GoRoute` objects defining the route tree. |
| `GoRoute(path: '/', builder: ...)` | Defines a route that matches the exact path `/`. The `builder` returns the widget to display. |
| `GoRoute(path: '/details/:id', builder: ...)` | Path parameters are declared with `:id`. They are extracted via `state.pathParameters`. |
| `state.pathParameters['id']` | Retrieves the value of the path parameter `id` from the current route state. |
| `MaterialApp.router(routerConfig: _router)` | Instead of `home` or `routes`, we use `routerConfig` to inject the `GoRouter` instance. |
| `context.go('/details/42')` | Navigates to the given path, replacing the entire history stack (like a web navigation). This is the primary navigation method. |
| `context.push('/details/42')` | Alternatively, you can push a new route without clearing the stack, like `Navigator.push`. |

---

## Mental Model

### The Web Routing Analogy

Think of GoRouter like a **web framework router** (Express, Flask, or React Router) but for a mobile/desktop Flutter app.

```
+----------------------------------------------------------+
|                       Browser URL                          |
|          https://example.com/user/123?tab=profile         |
+----------------------------------------------------------+
                             |
                             ▼
+----------------------------------------------------------+
|                      Routing Engine                        |
|  Path: /user/:id    Query: ?tab=profile                   |
|  Match route: /user/:id -> UserScreen                     |
|  Extract id=123, tab=profile                              |
+----------------------------------------------------------+
                             |
                             ▼
+----------------------------------------------------------+
|                     Screen Widget                         |
|         UserScreen(userId: 123, tab: 'profile')           |
+----------------------------------------------------------+
```

In GoRouter, the URL is the single source of truth for navigation state. When you call `context.go('/user/123?tab=profile')`, GoRouter:

1. Parses the URL into a `RouteMatch` list.
2. Matches the path against the route tree.
3. Builds the corresponding widget tree, injecting parameters.
4. Updates the `Navigator` pages to reflect the new stack.
5. On web, updates the browser URL; on mobile, this supports deep linking.

This mental model is **declarative**: you define what screens exist and under which paths, and GoRouter handles the rest.

---

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Route Tree** | A hierarchy of `GoRoute` objects. Each route can have child routes, enabling nested navigation. The tree mirrors the URL structure (e.g., `/parent/child`). |
| **Path Parameters** | Variables in the path prefixed with `:`, e.g., `/user/:id`. They are extracted from the URL and passed to the builder. |
| **Query Parameters** | Key-value pairs after `?`, e.g., `?tab=profile`. Accessible via `state.uri.queryParameters`. |
| **State** | `GoRouterState` holds information about the current navigation: the matched routes, path/query parameters, and the current location. |
| **Navigator Stack** | GoRouter manages a stack of `Page` objects. Navigating with `go` replaces the entire stack; `push` adds a new page; `pop` removes the top. |
| **Redirects** | A `redirect` function on a route or on the router that can intercept navigation and change the destination (e.g., for authentication). |
| **ShellRoute** | A special route that provides a persistent UI (like a bottom navigation bar) while its children are swapped inside it. |
| **StatefulShellRoute** | An extension of `ShellRoute` that preserves the state of each child navigator (e.g., each tab's navigation stack). |

---

## How It Works (Briefly)

1. **Initialization**: When `MaterialApp.router` builds, it obtains the `_router`'s `RouterDelegate` and `RouteInformationParser` from `routerConfig`. GoRouter provides these behind the scenes.

2. **Route Matching**: When a location (URL) changes (via `context.go`, browser back, or deep link), GoRouter parses the location into a list of segments and traverses the route tree to find the best match.

3. **Building Pages**: For each matched route, GoRouter invokes its `builder` (or `pageBuilder`) to produce a `Page` object. It respects child routes and nested structures, resulting in a list of pages that are set on the `Navigator`.

4. **Navigation Update**: The `Navigator` receives the new page list and animates transitions accordingly. The entire process is reactive: changing the URL updates the UI declaratively.

5. **History Management**: GoRouter uses Flutter's `Router` API to listen to system navigation events (back button, browser back) and syncs them with the location.

---

## When to Use GoRouter

✅ **Your app has multiple screens with complex navigation patterns** (nested routes, tabs, authentication).  
✅ **You need deep linking** (app links, universal links) or URL-based sharing.  
✅ **You're building for the web** and need browser URL synchronization.  
✅ **You prefer a declarative, state-driven approach** over imperative pushes.  
✅ **You want a unified navigation API across all platforms** (mobile, web, desktop).  

---

## When Not to Use GoRouter

❌ **Your app is extremely simple** (just a few screens with no deep linking). The standard `Navigator` might be simpler.  
❌ **You heavily rely on `Navigator.push` with custom transitions and complex interactions**—though GoRouter supports custom `PageBuilder`, it adds a layer of abstraction.  
❌ **You're in a very early prototyping phase** and want minimal setup; start with simple Navigator and migrate later.  
❌ **You're already deeply invested in `auto_route`** or another routing solution; migration may be costly.  

---

## Summary

- **GoRouter** is a declarative routing solution built on Navigator 2.0.
- It solves the pain points of imperative navigation: URL sync, deep linking, and state‑driven UI.
- Define your routes as a tree of `GoRoute`s and pass them to `MaterialApp.router`.
- Navigate using `context.go()`, `push()`, etc., and let GoRouter manage the page stack.
- Core features: path/query parameters, nested routes, redirects, and shell routes.
- Suitable for most non‑trivial Flutter apps, especially with web or deep‑link requirements.

---
