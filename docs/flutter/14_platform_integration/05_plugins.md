# Plugins

Understand how to create and use Flutter plugins to access native platform features.

---

# What is it?

Flutter plugins are packages that contain platform-specific code (Android/Kotlin/Java, iOS/Swift/Objective-C, Web, Windows, macOS, Linux) that can be called from Flutter. Plugins encapsulate native functionality and provide a unified Dart API, making it easy to use platform-specific features in a cross-platform way.

---

# Why does it exist?

Plugins exist to:

- Access native platform features
- Encapsulate platform-specific code
- Provide unified Dart API
- Share native functionality across projects
- Simplify platform integration
- Enable community contributions
- Support multiple platforms

---

# Creating a Plugin

> **Setting up** a Flutter plugin.

```dart
// 1. Create plugin using CLI
// flutter create --template=plugin --org com.example native_plugin
// cd native_plugin

// 2. Plugin structure
// native_plugin/
// ├── lib/
// │   ├── native_plugin.dart
// │   └── native_plugin_platform_interface.dart
// ├── android/
// │   └── src/
// │       └── main/
// │           └── kotlin/
// │               └── com/example/native_plugin/
// │                   └── NativePlugin.kt
// ├── ios/
// │   ├── Classes/
// │   │   └── NativePlugin.m
// │   └── NativePlugin.podspec
// ├── example/
// │   └── lib/
// │       └── main.dart
// └── pubspec.yaml

/// 3. Plugin API (lib/native_plugin.dart)
/// This is the main Dart API that users will interact with
import 'dart:async';

import 'package:flutter/services.dart';
import 'package:native_plugin/native_plugin_platform_interface.dart';

/// A plugin that provides native functionality
class NativePlugin {
  // MethodChannel for communication with native
  static const MethodChannel _channel = MethodChannel('com.example.native_plugin/methods');

  /// Get the device information
  static Future<String> getDeviceInfo() async {
    return await NativePluginPlatform.instance.getDeviceInfo();
  }

  /// Calculate using native code
  static Future<int> calculate(String operation, int a, int b) async {
    return await NativePluginPlatform.instance.calculate(operation, a, b);
  }

  /// Show a native toast
  static Future<void> showToast(String message) async {
    return await NativePluginPlatform.instance.showToast(message);
  }

  /// Get the battery level
  static Future<int> getBatteryLevel() async {
    return await NativePluginPlatform.instance.getBatteryLevel();
  }
}

/// 4. Platform Interface (lib/native_plugin_platform_interface.dart)
/// Defines the interface that platform-specific implementations must follow
abstract class NativePluginPlatform {
  /// The platform interface instance
  static NativePluginPlatform _instance = MethodChannelNativePlugin();

  /// Get the current instance
  static NativePluginPlatform get instance => _instance;

  /// Set the platform interface instance (for testing)
  static set instance(NativePluginPlatform instance) {
    _instance = instance;
  }

  /// Get device information
  Future<String> getDeviceInfo();

  /// Calculate using native code
  Future<int> calculate(String operation, int a, int b);

  /// Show a native toast
  Future<void> showToast(String message);

  /// Get battery level
  Future<int> getBatteryLevel();
}

/// 5. MethodChannel Implementation (lib/native_plugin_platform_interface.dart)
/// Implements the platform interface using MethodChannel
class MethodChannelNativePlugin implements NativePluginPlatform {
  // MethodChannel for communication
  static const MethodChannel _channel = MethodChannel('com.example.native_plugin/methods');

  @override
  Future<String> getDeviceInfo() async {
    try {
      return await _channel.invokeMethod('getDeviceInfo');
    } on PlatformException catch (e) {
      throw Exception('Failed to get device info: ${e.message}');
    }
  }

  @override
  Future<int> calculate(String operation, int a, int b) async {
    try {
      return await _channel.invokeMethod('calculate', {
        'operation': operation,
        'a': a,
        'b': b,
      });
    } on PlatformException catch (e) {
      throw Exception('Failed to calculate: ${e.message}');
    }
  }

  @override
  Future<void> showToast(String message) async {
    try {
      await _channel.invokeMethod('showToast', {'message': message});
    } on PlatformException catch (e) {
      throw Exception('Failed to show toast: ${e.message}');
    }
  }

  @override
  Future<int> getBatteryLevel() async {
    try {
      return await _channel.invokeMethod('getBatteryLevel');
    } on PlatformException catch (e) {
      throw Exception('Failed to get battery level: ${e.message}');
    }
  }
}
```

What's happening here?
- Plugin structure with platform interface
- MethodChannel for native communication
- Platform-specific implementations
- Unified Dart API for users

---

# Android Implementation

> **Implementing** the plugin on Android.

```kotlin
// android/src/main/kotlin/com/example/native_plugin/NativePlugin.kt

package com.example.native_plugin

import android.content.Context
import android.os.BatteryManager
import android.os.Build
import android.widget.Toast
import androidx.annotation.NonNull
import io.flutter.embedding.engine.plugins.FlutterPlugin
import io.flutter.plugin.common.MethodCall
import io.flutter.plugin.common.MethodChannel
import io.flutter.plugin.common.MethodChannel.MethodCallHandler
import io.flutter.plugin.common.MethodChannel.Result
import io.flutter.plugin.common.PluginRegistry.Registrar

/// NativePlugin - Android implementation
class NativePlugin : FlutterPlugin, MethodCallHandler {
    private lateinit var channel: MethodChannel
    private lateinit var context: Context

    // 1. Called when plugin is attached to Flutter
    override fun onAttachedToEngine(@NonNull flutterPluginBinding: FlutterPlugin.FlutterPluginBinding) {
        channel = MethodChannel(flutterPluginBinding.binaryMessenger, "com.example.native_plugin/methods")
        channel.setMethodCallHandler(this)
        context = flutterPluginBinding.applicationContext
    }

    // 2. Handle method calls from Dart
    override fun onMethodCall(@NonNull call: MethodCall, @NonNull result: Result) {
        when (call.method) {
            "getDeviceInfo" -> {
                // Return device information
                val info = "Android ${Build.VERSION.RELEASE} (SDK ${Build.VERSION.SDK_INT})"
                result.success(info)
            }
            "calculate" -> {
                // Perform calculation with parameters
                val operation = call.argument<String>("operation")
                val a = call.argument<Int>("a") ?: 0
                val b = call.argument<Int>("b") ?: 0
                
                val resultValue = when (operation) {
                    "add" -> a + b
                    "subtract" -> a - b
                    "multiply" -> a * b
                    "divide" -> if (b != 0) a / b else 0
                    else -> null
                }
                
                if (resultValue != null) {
                    result.success(resultValue)
                } else {
                    result.error("INVALID_OPERATION", "Unknown operation: $operation", null)
                }
            }
            "showToast" -> {
                // Show toast message
                val message = call.argument<String>("message")
                if (message != null) {
                    Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
                    result.success(null)
                } else {
                    result.error("INVALID_ARGUMENT", "Message is required", null)
                }
            }
            "getBatteryLevel" -> {
                // Get battery level
                val batteryLevel = getBatteryLevel()
                if (batteryLevel != -1) {
                    result.success(batteryLevel)
                } else {
                    result.error("UNAVAILABLE", "Battery level not available", null)
                }
            }
            else -> {
                // Method not implemented
                result.notImplemented()
            }
        }
    }

    // 3. Called when plugin is detached
    override fun onDetachedFromEngine(@NonNull binding: FlutterPlugin.FlutterPluginBinding) {
        channel.setMethodCallHandler(null)
    }

    // Helper method to get battery level
    private fun getBatteryLevel(): Int {
        val batteryManager = context.getSystemService(Context.BATTERY_SERVICE) as BatteryManager
        return batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
    }
}
```

What's happening here?
- FlutterPlugin interface
- MethodCallHandler handles calls
- onAttachedToEngine initializes
- Platform-specific implementations

---

# iOS Implementation

> **Implementing** the plugin on iOS.

```swift
// ios/Classes/NativePlugin.swift

import Flutter
import UIKit

/// NativePlugin - iOS implementation
public class NativePlugin: NSObject, FlutterPlugin {
    // 1. Register the plugin
    public static func register(with registrar: FlutterPluginRegistrar) {
        let channel = FlutterMethodChannel(
            name: "com.example.native_plugin/methods",
            binaryMessenger: registrar.messenger()
        )
        let instance = NativePlugin()
        registrar.addMethodCallDelegate(instance, channel: channel)
    }

    // 2. Handle method calls from Dart
    public func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
        switch call.method {
        case "getDeviceInfo":
            // Return device information
            let info = "iOS \(UIDevice.current.systemVersion) (\(UIDevice.current.model))"
            result(info)
            
        case "calculate":
            // Perform calculation with parameters
            guard let args = call.arguments as? [String: Any],
                  let operation = args["operation"] as? String,
                  let a = args["a"] as? Int,
                  let b = args["b"] as? Int else {
                result(FlutterError(
                    code: "INVALID_ARGUMENTS",
                    message: "Invalid arguments",
                    details: nil
                ))
                return
            }
            
            let resultValue: Int
            switch operation {
            case "add":
                resultValue = a + b
            case "subtract":
                resultValue = a - b
            case "multiply":
                resultValue = a * b
            case "divide":
                resultValue = b != 0 ? a / b : 0
            default:
                result(FlutterError(
                    code: "INVALID_OPERATION",
                    message: "Unknown operation: \(operation)",
                    details: nil
                ))
                return
            }
            result(resultValue)
            
        case "showToast":
            // Show toast (using alert)
            guard let args = call.arguments as? [String: Any],
                  let message = args["message"] as? String else {
                result(FlutterError(
                    code: "INVALID_ARGUMENT",
                    message: "Message is required",
                    details: nil
                ))
                return
            }
            
            showToast(message: message)
            result(nil)
            
        case "getBatteryLevel":
            // Get battery level
            let batteryLevel = getBatteryLevel()
            if batteryLevel != -1 {
                result(batteryLevel)
            } else {
                result(FlutterError(
                    code: "UNAVAILABLE",
                    message: "Battery level not available",
                    details: nil
                ))
            }
            
        default:
            result(FlutterMethodNotImplemented)
        }
    }

    // Helper method to show toast-like alert
    private func showToast(message: String) {
        let alert = UIAlertController(
            title: nil,
            message: message,
            preferredStyle: .alert
        )
        
        if let rootViewController = UIApplication.shared.windows.first?.rootViewController {
            rootViewController.present(alert, animated: true)
            DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
                alert.dismiss(animated: true)
            }
        }
    }

    // Helper method to get battery level
    private func getBatteryLevel() -> Int {
        let device = UIDevice.current
        device.isBatteryMonitoringEnabled = true
        let batteryLevel = Int(device.batteryLevel * 100)
        device.isBatteryMonitoringEnabled = false
        return batteryLevel
    }
}
```

What's happening here?
- FlutterPlugin protocol
- register method
- handle method for calls
- Platform-specific implementations

---

# Using the Plugin

> **Using** the plugin in a Flutter app.

```dart
// example/lib/main.dart

import 'package:flutter/material.dart';
import 'package:native_plugin/native_plugin.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Native Plugin Example',
      home: const PluginExampleScreen(),
    );
  }
}

class PluginExampleScreen extends StatefulWidget {
  const PluginExampleScreen({super.key});

  @override
  State<PluginExampleScreen> createState() => _PluginExampleScreenState();
}

class _PluginExampleScreenState extends State<PluginExampleScreen> {
  String _deviceInfo = 'Unknown';
  String _calculationResult = 'N/A';
  int _batteryLevel = 0;
  bool _isLoading = false;

  @override
  void initState() {
    super.initState();
    _getDeviceInfo();
    _getBatteryLevel();
  }

  Future<void> _getDeviceInfo() async {
    try {
      final info = await NativePlugin.getDeviceInfo();
      setState(() {
        _deviceInfo = info;
      });
    } catch (e) {
      setState(() {
        _deviceInfo = 'Error: $e';
      });
    }
  }

  Future<void> _calculate(String operation, int a, int b) async {
    setState(() => _isLoading = true);

    try {
      final result = await NativePlugin.calculate(operation, a, b);
      setState(() {
        _calculationResult = '$a $operation $b = $result';
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _calculationResult = 'Error: $e';
        _isLoading = false;
      });
    }
  }

  Future<void> _getBatteryLevel() async {
    try {
      final level = await NativePlugin.getBatteryLevel();
      setState(() {
        _batteryLevel = level;
      });
    } catch (e) {
      setState(() {
        _batteryLevel = -1;
      });
    }
  }

  Future<void> _showToast(String message) async {
    try {
      await NativePlugin.showToast(message);
    } catch (e) {
      print('Toast error: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Native Plugin'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Device info
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Device Info',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    Text(_deviceInfo),
                  ],
                ),
              ),
            ),
            // Battery level
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Battery Level',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      _batteryLevel >= 0 ? '$_batteryLevel%' : 'Unknown',
                      style: const TextStyle(fontSize: 20),
                    ),
                    const SizedBox(height: 8),
                    ElevatedButton(
                      onPressed: _getBatteryLevel,
                      child: const Text('Refresh Battery'),
                    ),
                  ],
                ),
              ),
            ),
            // Calculator
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Native Calculator',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    if (_isLoading)
                      const CircularProgressIndicator()
                    else
                      Text(
                        _calculationResult,
                        style: const TextStyle(fontSize: 18),
                      ),
                    const SizedBox(height: 8),
                    Wrap(
                      spacing: 8,
                      runSpacing: 8,
                      children: [
                        ElevatedButton(
                          onPressed: () => _calculate('add', 10, 5),
                          child: const Text('10 + 5'),
                        ),
                        ElevatedButton(
                          onPressed: () => _calculate('subtract', 10, 5),
                          child: const Text('10 - 5'),
                        ),
                        ElevatedButton(
                          onPressed: () => _calculate('multiply', 10, 5),
                          child: const Text('10 × 5'),
                        ),
                        ElevatedButton(
                          onPressed: () => _calculate('divide', 10, 5),
                          child: const Text('10 ÷ 5'),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            // Toast
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Native Toast',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    ElevatedButton(
                      onPressed: () {
                        _showToast('Hello from Flutter Plugin!');
                      },
                      child: const Text('Show Toast'),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Using plugin API in Flutter app
- Platform-specific features
- Error handling
- Loading states

---

# Best Practices

## Use Platform Interface

```dart
// Good - Platform interface pattern
abstract class PluginPlatform {
  Future<String> getInfo();
}

class Plugin extends PluginPlatform {
  // Implementation
}

// Bad - Direct platform code
class Plugin {
  static const MethodChannel _channel = MethodChannel('...');
}
```

## Handle Errors

```dart
// Good - Error handling
try {
  final result = await plugin.method();
} on PlatformException catch (e) {
  // Handle platform error
} catch (e) {
  // Handle other errors
}
```

## Test Platform Code

```dart
// Good - Testable plugin
class MockPlugin extends PluginPlatform {
  @override
  Future<String> getInfo() async => 'Mock Info';
}
```

---

# Common Mistakes

## Not Handling Platform Errors

Wrong:
```dart
// No error handling
final result = await plugin.method();
```

Correct:
```dart
// With error handling
try {
  final result = await plugin.method();
} on PlatformException catch (e) {
  // Handle error
}
```

## Not Using Platform Interface

Wrong:
```dart
// Direct platform code
class Plugin {
  static const MethodChannel _channel = MethodChannel('...');
}
```

Correct:
```dart
// With platform interface
abstract class PluginPlatform { ... }
class Plugin extends PluginPlatform { ... }
```

---

# Summary

Plugins encapsulate platform-specific code and provide unified Dart APIs. Use the platform interface pattern for clean architecture, handle errors properly, and test platform code with mocks. Plugins are essential for accessing native platform features.

---

# Next Steps

- [Platform Views](platform-views.md)
- [MethodChannel](methodchannel.md)
- [EventChannel](eventchannel.md)

---

# Did You Know?

- Plugins encapsulate native code
- Platform interface pattern improves testing
- MethodChannel handles communication
- Android uses Kotlin/Java
- iOS uses Swift/Objective-C
- Plugins can support multiple platforms
- Pub.dev hosts many plugins
- Plugin development requires native knowledge