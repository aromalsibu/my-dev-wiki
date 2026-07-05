# Running Your First App

Understand how to create and run your first Flutter application.

---

# What is it?

Running your first Flutter app involves creating a new Flutter project, understanding the default counter app, and launching it on an emulator or physical device. This is the first step in your Flutter development journey.

---

# Why does it exist?

Running your first app exists to:

- Verify your Flutter installation works correctly
- Understand the basic structure of a Flutter app
- Learn the development workflow
- See Flutter's hot reload in action
- Experience cross-platform development
- Build confidence in your setup

---

# Creating a New Project

> **Use the `flutter create` command** to generate a new Flutter project.

```bash
# Create a new Flutter project
flutter create my_first_app

# Output:
# Creating project my_first_app...
#   my_first_app/.gitignore (created)
#   my_first_app/README.md (created)
#   my_first_app/analysis_options.yaml (created)
#   my_first_app/pubspec.yaml (created)
#   ...
# Running "flutter pub get" in my_first_app... 2.3s
# Wrote 127 files.

# All done!
# You can find more help at: https://docs.flutter.dev/

# Navigate into the project
cd my_first_app

# List the files
ls
# android/  ios/  lib/  pubspec.yaml  README.md  test/  web/
```

What's happening here?
- `flutter create` generates a complete project
- Default app is a counter app
- Project includes all platform folders
- Dependencies are automatically fetched
- You can specify project name and location

---

# Project Name Options

```bash
# Basic project creation
flutter create my_first_app

# Create with a specific package name
flutter create --org com.example my_first_app

# Create with iOS language set to Swift
flutter create --ios-language swift my_first_app

# Create with Android language set to Kotlin
flutter create --android-language kotlin my_first_app

# Create in a specific directory
flutter create --project-name my_app ./my_app_folder

# Create with a template
flutter create -t skeleton my_app  # Advanced template
flutter create -t app my_app       # Full app (default)
flutter create -t module my_app    # Module for existing app
```

What's happening here?
- `--org` sets the reverse domain for package names
- Language options set native code languages
- Templates provide different starting points
- Project name must be lowercase with underscores

---

# The Default Counter App

> **The default app** is a simple counter application that demonstrates Flutter basics.

```dart
// lib/main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

What's happening here?
- `main()` runs the app with `runApp()`
- `MyApp` is the root widget
- `MaterialApp` provides Material Design
- `MyHomePage` is the main screen
- `setState()` triggers UI rebuilds
- `_counter` holds the state

---

# Running on Android

> **Run your Flutter app on Android** using an emulator or physical device.

```bash
# List available emulators
flutter emulators

# Output:
# 2 available emulators:
# Pixel_3a_API_34_extension_level_7_x86_64 • Pixel 3a • Google
# Pixel_6_API_34 • Pixel 6 • Google

# Start an emulator
flutter emulators --launch Pixel_3a_API_34_extension_level_7_x86_64

# Wait for emulator to boot
# Then run the app
flutter run

# Or run with a specific device
flutter run -d emulator-5554

# Or run using the device name
flutter run -d "Pixel 3a"
```

What's happening here?
- `flutter emulators` shows available emulators
- `--launch` starts the emulator
- `flutter run` builds and installs the app
- `-d` specifies a specific device
- App appears on the emulator screen

---

# Running on iOS

> **Run your Flutter app on iOS** using a simulator or physical device.

```bash
# List available simulators
flutter devices

# Output:
# 3 connected devices:
# iPhone 15 Pro (mobile) • ios • iOS 17.0
# macOS (desktop) • macos • macOS 14.0
# Chrome (web) • chrome • web

# Open iOS simulator
open -a Simulator

# Run the app on simulator
flutter run

# Run on a specific device
flutter run -d "iPhone 15 Pro"

# Build for release
flutter build ios --release
```

What's happening here?
- `flutter devices` lists all available devices
- `open -a Simulator` starts iOS simulator
- `flutter run` builds and installs
- iOS requires Xcode to be installed
- Physical devices need provisioning profiles

---

# Running on Web

> **Run your Flutter app in the browser.**

```bash
# Run on Chrome
flutter run -d chrome

# Run on Edge
flutter run -d edge

# Run with a specific browser
flutter run -d web-server --web-port 8080

# Build for web
flutter build web

# Serve the build output
cd build/web/
python -m http.server 8000
```

What's happening here?
- `-d chrome` runs in Chrome browser
- Web runs in development mode with debug tools
- Build for deployment with `flutter build web`
- Web apps can be hosted on any web server

---

# Running on Desktop

> **Run your Flutter app on desktop platforms.**

```bash
# Run on Windows
flutter run -d windows

# Run on macOS
flutter run -d macos

# Run on Linux
flutter run -d linux

# Build for desktop
flutter build windows
flutter build macos
flutter build linux
```

What's happening here?
- Desktop support is built-in
- Each platform requires specific setup
- Apps run natively on each platform
- Build creates executable files

---

# Hot Reload

> **Hot reload** lets you see changes instantly without restarting the app.

```dart
// While the app is running, make a change
// Change the text in the app

// Original:
Text('You have pushed the button this many times:'),

// Change to:
Text('You have clicked the button this many times:'),

// Save the file (Ctrl+S or Cmd+S)
// The app updates instantly!

// OR use keyboard shortcuts:
// r - Hot reload (preserves state)
// R - Hot restart (resets state)
// q - Quit the app
```

What's happening here?
- Hot reload preserves the app state
- Changes appear within seconds
- No need to recompile or restart
- Development is much faster
- State is preserved during reload

---

# Using Flutter Run Options

> **Various options** for running your Flutter app.

```bash
# Run in release mode (optimized)
flutter run --release

# Run in profile mode (performance profiling)
flutter run --profile

# Run with a specific flavor
flutter run --flavor production

# Run with verbose logging
flutter run --verbose

# Run with a specific target file
flutter run -t lib/main.dart

# Run with specific device
flutter run -d emulator-5554

# Run with custom port
flutter run --web-port 8080
```

What's happening here?
- Release mode is fully optimized
- Profile mode helps with performance
- Flavors support different environments
- Verbose helps with debugging

---

# Common Issues

## Device Not Found

Wrong:
```bash
# Error: No device connected
flutter run
```

Correct:
```bash
# Check connected devices
flutter devices

# Start an emulator
flutter emulators --launch Pixel_3a

# Or connect a physical device
# Enable USB debugging on Android
# Trust the computer on iOS
```

---

## Build Failed

Wrong:
```bash
# Error: Build failed with exception
```

Correct:
```bash
# Clean the project
flutter clean

# Get dependencies
flutter pub get

# Try running again
flutter run

# Check flutter doctor
flutter doctor
```

---

## Port Already in Use

Wrong:
```bash
# Error: Port 8080 already in use
```

Correct:
```bash
# Use a different port
flutter run --web-port 8081

# Or kill the existing process
kill -9 $(lsof -t -i:8080)
```

---

# Best Practices

- Use `flutter run` for development
- Use hot reload frequently
- Test on multiple devices
- Use `flutter clean` when builds fail
- Use `flutter doctor` regularly
- Test on real devices when possible
- Use `--release` for performance testing

---

# Summary

Running your first Flutter app involves creating a project with `flutter create`, running it with `flutter run`, and using hot reload to see changes instantly. The default counter app demonstrates key Flutter concepts.

---

# Next Steps

- [Hot Reload vs Hot Restart](hot-reload.md)
- [Flutter Architecture](../framework-architecture/flutter-architecture.md)
- [What is a Widget?](../widgets/what-is-a-widget.md)

---

# Did You Know?

- The default app has been updated with Material 3
- Hot reload works on all platforms
- You can run multiple instances of Flutter apps
- `flutter run` automatically handles code signing on iOS
- The debug banner can be removed with `debugShowCheckedModeBanner: false`
- Flutter apps run at 60fps by default (120fps on ProMotion displays)