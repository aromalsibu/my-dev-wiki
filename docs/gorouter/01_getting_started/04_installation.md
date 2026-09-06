# Installation

## Overview

Installing GoRouter is a straightforward process that integrates the package into your Flutter project's dependency management system. Like any Flutter package, GoRouter is installed via **pub.dev**—the official package repository for Dart and Flutter—and becomes available throughout your project once the dependency is added and fetched.

> **Why This Matters**: Proper installation is the foundation of everything that follows. A correctly installed package ensures you have access to the latest features, bug fixes, and API compatibility. GoRouter is **maintained by the Flutter team**, so staying current with versions is particularly important for stability and security.

---

## Why Installation Is Different for GoRouter

Unlike many packages where installation is a purely mechanical step, GoRouter's installation sets the stage for a **fundamental architectural choice** in your app. By installing GoRouter, you're committing to:

1. **Declarative routing** over imperative navigation
2. **URL-based navigation** that works across all platforms
3. **Deep linking** support for mobile and web

The installation itself is simple, but the **implications** are significant—you're adopting a routing paradigm that will shape your app's architecture.

---

## Prerequisites

Before installing GoRouter, ensure your development environment meets these requirements:

| Requirement | Minimum Version | Why It Matters |
|-------------|-----------------|----------------|
| **Flutter SDK** | >=3.35.0 | GoRouter 17.x relies on newer Flutter APIs for navigation and state management |
| **Dart SDK** | ^3.9.0 | Required for null safety and modern Dart language features |
| **Platform Support** | Android, iOS, Web, macOS, Windows, Linux | GoRouter works across all Flutter-supported platforms |

> **Note**: As of May 2026, the latest stable version is **go_router 17.2.3**. Always check [pub.dev](https://pub.dev/packages/go_router) for the most current version.

---

## Installation Methods

### Method 1: Using `flutter pub add` (Recommended)

The simplest and most reliable way to install GoRouter is using Flutter's built-in package management command:

```bash
flutter pub add go_router
```

This command:
1. Adds the latest version of `go_router` to your `pubspec.yaml` dependencies
2. Automatically runs `flutter pub get` to fetch the package
3. Updates the `pubspec.lock` file with the resolved version

**Why this is the recommended approach**: The `flutter pub add` command ensures you get the **latest compatible version** of the package, handles dependency resolution automatically, and eliminates the risk of typos in version numbers.

### Method 2: Manual `pubspec.yaml` Addition

If you prefer explicit control over versioning, you can manually add GoRouter to your `pubspec.yaml` file:

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^17.2.3  # Use the latest stable version
```

After adding the dependency, run:

```bash
flutter pub get
```

**Version Notation Explained**:
- `^17.2.3` — Allows any version from 17.2.3 up to (but not including) 18.0.0
- `17.2.3` — Pins to exactly this version (not recommended)
- `any` — Uses the latest version (not recommended for production)

### Method 3: For Type-Safe Routing (Advanced)

If you plan to use GoRouter's **type-safe routing** features (powered by code generation), you'll also need development dependencies:

```yaml
dependencies:
  go_router: ^17.2.3

dev_dependencies:
  build_runner: ^2.4.0
  go_router_builder: ^17.2.3
  build_verify: ^3.0.0
```

Install these with:

```bash
flutter pub add go_router
flutter pub add --dev build_runner go_router_builder build_verify
```

> **Note**: Type-safe routing is an advanced feature. For most apps, the basic installation is sufficient to get started.

---

## Verification

After installation, verify that GoRouter is correctly installed by:

### 1. Check `pubspec.yaml`

Your `pubspec.yaml` should now include:

```yaml
dependencies:
  go_router: ^17.2.3  # or the version you installed
```

### 2. Import and Use

Create a simple GoRouter configuration to confirm everything works:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';  // ← The import statement

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  MyApp({super.key});

  // Create a basic router configuration
  final GoRouter _router = GoRouter(
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => const HomePage(),
      ),
    ],
  );

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,  // ← Use routerConfig, not home or routes
      title: 'GoRouter Demo',
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('GoRouter Works!')),
      body: const Center(child: Text('Installation successful')),
    );
  }
}
```

### 3. Run Your App

```bash
flutter run
```

If the app builds and displays the home page without errors, GoRouter is successfully installed.

---

## Common Installation Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **"Package not found"** | Typo in package name | Ensure `go_router` (with underscore) is spelled correctly |
| **Version conflict** | Incompatible with other packages | Run `flutter pub upgrade` or adjust version constraints |
| **"SDK version too low"** | Flutter or Dart version outdated | Upgrade Flutter: `flutter upgrade` |
| **Import not working** | Missing import statement | Add `import 'package:go_router/go_router.dart';` |
| **Build fails** | Corrupted cache | Run `flutter clean` then `flutter pub get` |

---

## Platform-Specific Setup (Optional)

While GoRouter itself doesn't require platform-specific configuration, certain features do:

### For Deep Linking on Android

If you plan to use deep linking, you'll need to configure your `AndroidManifest.xml`:

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="http" android:host="yourdomain.com" />
    <data android:scheme="https" />
</intent-filter>
```

### For Universal Links on iOS

For iOS universal links, configure **Associated Domains** in Xcode:

1. Open `ios/Runner.xcworkspace`
2. Select the **Runner** target
3. Go to **Signing & Capabilities**
4. Add **Associated Domains** capability
5. Add `applinks:yourdomain.com`

---

## What's Next?

With GoRouter successfully installed, you're ready to:
- **Define your first route** (coming in the next section)
- **Configure the router** with `initialLocation`, `redirect`, and `errorBuilder`
- **Build your route tree** with nested routes and shell routes

---

## Summary

- **Installation is simple**: Use `flutter pub add go_router`
- **Latest version**: As of May 2026, **v17.2.3** is the stable release
- **Platform support**: Works on Android, iOS, Web, macOS, Windows, and Linux
- **Optional advanced setup**: Type-safe routing requires `go_router_builder`
- **Deep linking**: Requires additional platform-specific configuration (Android/iOS)
- **Verify installation**: Import the package and create a basic router

---
