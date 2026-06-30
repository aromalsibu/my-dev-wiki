# Flutter Architecture

Understand the layered architecture of Flutter framework.

---

# What is it?

Flutter Architecture is the layered design that powers the Flutter framework. It consists of three main layers: the Embedder, the Engine, and the Framework. Each layer builds upon the previous one, providing a complete system for building cross-platform applications.

---

# Why does it exist?

Flutter Architecture exists to:

- Provide a clean separation of concerns
- Enable cross-platform compatibility
- Support high-performance rendering
- Allow platform-specific customization
- Make the framework extensible
- Simplify development and debugging
- Enable hot reload functionality

---

# The Three Layers

> **Flutter is built on three layers:** Embedder, Engine, and Framework.

```
┌─────────────────────────────────────────────┐
│           Framework (Dart)                   │
│  ┌─────────────────────────────────────────┐ │
│  │  Material   │  Cupertino   │  Widgets   │ │
│  ├─────────────┼─────────────┼────────────┤ │
│  │     Rendering Layer                     │ │
│  ├─────────────────────────────────────────┤ │
│  │     Widgets Layer                       │ │
│  ├─────────────────────────────────────────┤ │
│  │     Foundation Layer                    │ │
│  └─────────────────────────────────────────┘ │
├─────────────────────────────────────────────┤
│            Engine (C/C++)                    │
│  ┌─────────────────────────────────────────┐ │
│  │  Skia   │  Dart VM   │  Text Layout    │ │
│  ├─────────┼────────────┼─────────────────┤ │
│  │  GPU    │  IO        │  Platform       │ │
│  └─────────────────────────────────────────┘ │
├─────────────────────────────────────────────┤
│            Embedder (Platform Specific)      │
│  ┌─────────────────────────────────────────┐ │
│  │  Android │  iOS  │  Web │  Desktop     │ │
│  └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

What's happening here?
- Embedder is platform-specific code
- Engine handles rendering and Dart execution
- Framework is what developers interact with
- Each layer has specific responsibilities
- Layers communicate through defined interfaces

---

# Embedder Layer

> **The Embedder** is the platform-specific layer that integrates Flutter with the host operating system.

```dart
// Not directly used in Dart code
// But here's what it handles:

// Android Embedder (Java/Kotlin)
// - Activity lifecycle
// - SurfaceView for rendering
// - Platform channels
// - Input handling

// iOS Embedder (Objective-C/Swift)
// - UIViewController lifecycle
// - Metal/OpenGL rendering
// - Platform channels
// - Touch events

// Web Embedder (JavaScript)
// - DOM manipulation
// - Canvas rendering
// - WebGL support
// - Browser events

// Desktop Embedder (C++)
// - Window management
// - OpenGL/DirectX
// - Input events
// - File system access
```

What's happening here?
- Embedder connects Flutter to OS
- Handles window creation and events
- Manages thread lifecycle
- Provides platform-specific APIs
- Different for each platform

---

# Engine Layer

> **The Engine** is written in C/C++ and provides the core functionality of Flutter.

```c++
// Engine components (conceptual)

// Skia Graphics Engine
SkCanvas canvas;
canvas.drawRect(rect, paint);
canvas.drawText("Hello", position);

// Dart VM
Dart_Isolate isolate;
Dart_EnterScope();
Dart_Invoke(object, method, args);

// Text Layout
Paragraph paragraph = ParagraphBuilder(style)
    .addText("Hello World")
    .build()
    .layout(containerWidth);

// Platform Channels
PlatformChannel channel = PlatformChannel(name);
channel.setMethodCallHandler((call) async {
  return platformMethod(call);
});

// GPU Thread
void renderFrame() {
  frame_scheduler.scheduleFrame();
  rasterizer.draw(frame);
}

// IO Thread
void loadImage(String url) {
  image_cache.load(url);
}
```

What's happening here?
- Skia handles all graphics rendering
- Dart VM executes Dart code
- Text layout is handled by Skia
- Platform channels enable native communication
- GPU thread handles rendering
- IO thread manages assets and networking

---

# Framework Layer

> **The Framework** is written in Dart and is what developers use to build Flutter apps.

```dart
// Framework components

// 1. Foundation Layer
// Basic utilities and core classes
import 'dart:ui'; // Canvas, Paint, etc.
import 'package:flutter/foundation.dart'; // ChangeNotifier, etc.

// 2. Widgets Layer
// Core widget system
import 'package:flutter/widgets.dart';
// - Widget
// - StatelessWidget
// - StatefulWidget
// - InheritedWidget
// - RenderObjectWidget

// 3. Rendering Layer
// Layout and painting
import 'package:flutter/rendering.dart';
// - RenderBox
// - RenderObject
// - PaintingContext
// - Layout constraints

// 4. Material Library
// Material Design widgets
import 'package:flutter/material.dart';
// - MaterialApp
// - Scaffold
// - AppBar
// - ElevatedButton

// 5. Cupertino Library
// iOS-style widgets
import 'package:flutter/cupertino.dart';
// - CupertinoApp
// - CupertinoNavigationBar
// - CupertinoButton
```

What's happening here?
- Foundation provides utilities
- Widgets layer is the UI building system
- Rendering handles layout and painting
- Material provides Android-style widgets
- Cupertino provides iOS-style widgets

---

# Framework Components in Detail

## Foundation Layer

```dart
// Foundation provides core utilities
import 'package:flutter/foundation.dart';

// ChangeNotifier for state management
class CounterModel extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners(); // Notify listeners of changes
  }
}

// ValueNotifier for simple values
final counter = ValueNotifier<int>(0);
counter.value++; // Triggers rebuild

// Key for widget identification
Key key = UniqueKey();
Key valueKey = ValueKey<int>(42);

// Debug tools
debugPrint('Debug message');
assert(condition, 'Error message');
```

What's happening here?
- ChangeNotifier enables reactive UI
- ValueNotifier for simple reactive values
- Keys for widget identity
- Debug tools for development

---

## Widgets Layer

```dart
// Widgets layer provides the widget system
import 'package:flutter/widgets.dart';

// Basic widget hierarchy
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Row(
        children: [
          Text('Hello'),
          Icon(Icons.star),
        ],
      ),
    );
  }
}

// BuildContext provides widget tree information
@override
Widget build(BuildContext context) {
  // Access theme
  ThemeData theme = Theme.of(context);
  
  // Access inherited widgets
  var data = InheritedData.of(context);
  
  // Media query for screen size
  var size = MediaQuery.of(context).size;
  
  return Container();
}
```

What's happening here?
- Widgets are immutable configurations
- BuildContext locates widgets in tree
- Inherited widgets share data
- MediaQuery provides screen information

---

## Rendering Layer

```dart
// Rendering layer handles layout and painting
import 'package:flutter/rendering.dart';

// RenderObject for custom rendering
class MyRenderObject extends RenderBox {
  @override
  void paint(PaintingContext context, Offset offset) {
    final canvas = context.canvas;
    canvas.drawRect(
      offset & size,
      Paint()..color = Colors.blue,
    );
  }
  
  @override
  void performLayout() {
    size = constraints.constrain(Size(100, 100));
  }
}

// RenderObjectWidget for custom widgets
class MyRenderWidget extends LeafRenderObjectWidget {
  @override
  RenderObject createRenderObject(BuildContext context) {
    return MyRenderObject();
  }
}
```

What's happening here?
- RenderObject handles painting
- PerformLayout determines size
- Paint draws to canvas
- Layout constraints flow downward

---

# Data Flow in Flutter

> **Data flows in one direction:** from the top of the widget tree down.

```dart
// Top-down data flow
class ParentWidget extends StatelessWidget {
  final String data = 'Hello';
  
  @override
  Widget build(BuildContext context) {
    return ChildWidget(data: data);
  }
}

class ChildWidget extends StatelessWidget {
  final String data;
  
  const ChildWidget({required this.data});
  
  @override
  Widget build(BuildContext context) {
    return Text(data); // Receives data from parent
  }
}

// Events flow bottom-up
class ButtonWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        // Event flows up through callbacks
        Navigator.push(context, route);
      },
      child: Text('Tap me'),
    );
  }
}
```

What's happening here?
- Data flows down from parent to child
- Events flow up through callbacks
- State management sits in between
- Widgets receive data through parameters

---

# Rendering Pipeline

> **The rendering pipeline** consists of three trees: Widget, Element, and RenderObject.

```dart
// 1. Widget Tree (Configuration)
// Immutable widgets
Widget widget = Container(
  color: Colors.blue,
  child: Text('Hello'),
);

// 2. Element Tree (State)
// Mutable elements that hold widget state
Element element = widget.createElement();

// 3. RenderObject Tree (Layout/Paint)
// Actual rendering objects
RenderObject renderObject = element.renderObject;

// The pipeline process:
// 1. Build widgets
// 2. Update elements
// 3. Layout render objects
// 4. Paint to canvas
// 5. Composite layers
// 6. Display on screen
```

What's happening here?
- Widgets are the blueprint
- Elements hold state
- RenderObjects handle painting
- Pipeline runs every frame

---

# Platform Channels

> **Platform channels** enable communication between Dart and native code.

```dart
// Dart side
import 'package:flutter/services.dart';

class PlatformService {
  static const MethodChannel _channel = 
      MethodChannel('com.example.app/platform');
  
  static Future<String> getPlatformVersion() async {
    try {
      final String result = await _channel.invokeMethod('getPlatformVersion');
      return result;
    } on PlatformException catch (e) {
      return 'Failed: ${e.message}';
    }
  }
  
  static Future<void> showNativeToast(String message) async {
    await _channel.invokeMethod('showToast', {'message': message});
  }
}

// Usage
void main() async {
  String version = await PlatformService.getPlatformVersion();
  print('Platform: $version');
  
  await PlatformService.showNativeToast('Hello from Flutter!');
}

// Android side (Kotlin)
class MainActivity: FlutterActivity() {
  override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    MethodChannel(flutterEngine.dartExecutor, "com.example.app/platform")
      .setMethodCallHandler { call, result ->
        if (call.method == "getPlatformVersion") {
          result.success("Android ${android.os.Build.VERSION.RELEASE}")
        } else if (call.method == "showToast") {
          Toast.makeText(this, call.argument("message"), Toast.LENGTH_SHORT).show()
          result.success(null)
        } else {
          result.notImplemented()
        }
      }
  }
}

// iOS side (Swift)
class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller: FlutterViewController = window?.rootViewController as! FlutterViewController
    let channel = FlutterMethodChannel(
      name: "com.example.app/platform",
      binaryMessenger: controller.binaryMessenger
    )
    
    channel.setMethodCallHandler { (call, result) in
      if call.method == "getPlatformVersion" {
        result("iOS \(UIDevice.current.systemVersion)")
      } else if call.method == "showToast" {
        // Show native toast
        result(nil)
      } else {
        result(FlutterMethodNotImplemented)
      }
    }
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

What's happening here?
- MethodChannel enables two-way communication
- Platform channels are asynchronous
- Data must be serializable (JSON-like)
- Errors are propagated via PlatformException
- Works across all platforms

---

# Best Practices

## Understand the Layers

```dart
// Use the right layer for the right task
// Framework: Build UI
// Engine: Rendering, platform integration
// Embedder: Platform-specific integration

// Most development happens in Framework
import 'package:flutter/material.dart';

// For custom rendering, use RenderObject
import 'package:flutter/rendering.dart';

// For platform integration, use Platform Channels
import 'package:flutter/services.dart';
```

## Follow Data Flow

```dart
// Data flows down
// Events flow up
// Keep state management at appropriate levels

// Good
class Parent extends StatelessWidget {
  final String data;
  
  @override
  Widget build(BuildContext context) {
    return Child(data: data);
  }
}

// Bad
class Child extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Accessing data from too high
    return Text(globalData);
  }
}
```

---

# Common Mistakes

## Overusing Platform Channels

Wrong:
```dart
// Too many platform calls
for (var item in items) {
  await platformChannel.invokeMethod('process', item);
}
```

Correct:
```dart
// Batch operations
await platformChannel.invokeMethod('processBatch', items);
```

---

## Not Handling Platform Exceptions

Wrong:
```dart
Future<void> callPlatform() async {
  await channel.invokeMethod('doSomething');
}
```

Correct:
```dart
Future<void> callPlatform() async {
  try {
    await channel.invokeMethod('doSomething');
  } on PlatformException catch (e) {
    print('Platform error: ${e.message}');
    // Handle error gracefully
  }
}
```

---

# Summary

Flutter Architecture consists of three layers: Embedder (platform-specific), Engine (C/C++ core), and Framework (Dart). Understanding these layers helps you build better apps and debug issues effectively.

---

# Next Steps

- [Widget Tree](widget-tree.md)
- [Element Tree](element-tree.md)
- [Render Tree](render-tree.md)

---

# Did You Know?

- The Flutter Engine is about 3MB in size
- Flutter can run without the Framework
- The Dart VM runs in AOT or JIT mode
- Flutter supports custom platform channels
