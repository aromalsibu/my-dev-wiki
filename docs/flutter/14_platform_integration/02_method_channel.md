# MethodChannel

Understand how to use MethodChannel for two-way communication between Flutter and native platform code.

---

# What is it?

MethodChannel is a specific type of platform channel that enables calling methods on the native platform (Android/iOS) from Flutter and vice versa. It provides a bidirectional communication path where Flutter can invoke native methods and receive results, and native code can also invoke methods on Flutter.

---

# Why does it exist?

MethodChannel exists to:

- Call native platform APIs from Flutter
- Receive results from native code
- Pass complex data between Flutter and native
- Implement platform-specific functionality
- Handle asynchronous operations
- Provide bidirectional communication
- Enable native SDK integration

---

# Basic MethodChannel

> **Creating and using** MethodChannel.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

/// Basic MethodChannel example
class BasicMethodChannelExample extends StatefulWidget {
  const BasicMethodChannelExample({super.key});

  @override
  State<BasicMethodChannelExample> createState() => _BasicMethodChannelExampleState();
}

class _BasicMethodChannelExampleState extends State<BasicMethodChannelExample> {
  // 1. Create MethodChannel with unique name
  // This name must match exactly on the native side
  static const MethodChannel _channel = MethodChannel('com.example.app/methods');

  // State variables
  String _deviceInfo = 'Unknown';
  String _calculationResult = 'N/A';
  bool _isLoading = false;

  @override
  void initState() {
    super.initState();
    _getDeviceInfo();
  }

  // 2. Call a method without parameters
  Future<void> _getDeviceInfo() async {
    try {
      // Invoke the native method
      final String result = await _channel.invokeMethod('getDeviceInfo');
      setState(() {
        _deviceInfo = result;
      });
    } on PlatformException catch (e) {
      setState(() {
        _deviceInfo = 'Error: ${e.message}';
      });
    }
  }

  // 3. Call a method with parameters
  Future<void> _calculate(String operation, int a, int b) async {
    setState(() => _isLoading = true);

    try {
      // Pass parameters as a map
      final int result = await _channel.invokeMethod(
        'calculate',
        {
          'operation': operation,
          'a': a,
          'b': b,
        },
      );
      setState(() {
        _calculationResult = '$a $operation $b = $result';
        _isLoading = false;
      });
    } on PlatformException catch (e) {
      setState(() {
        _calculationResult = 'Error: ${e.message}';
        _isLoading = false;
      });
    }
  }

  // 4. Call a method with complex parameters
  Future<void> _processData(Map<String, dynamic> data) async {
    try {
      final Map<dynamic, dynamic> result = await _channel.invokeMethod(
        'processData',
        data,
      );
      print('Processed data: $result');
    } on PlatformException catch (e) {
      print('Error processing data: ${e.message}');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('MethodChannel'),
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
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
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
                      ],
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
- MethodChannel for calling native methods
- invokeMethod() calls native methods
- Parameters can be passed as maps
- Results can be any JSON-serializable type

---

# Native Implementation - Android

> **Implementing** MethodChannel on Android.

```kotlin
// MainActivity.kt (Android)

package com.example.app

import android.os.Build
import android.os.Bundle
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel
import io.flutter.plugin.common.MethodChannel.Result
import io.flutter.plugin.common.MethodCall

class MainActivity : FlutterActivity() {
    // 1. Channel name must match Dart side
    private val CHANNEL = "com.example.app/methods"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        // 2. Create MethodChannel
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
            .setMethodCallHandler { call, result ->
                // 3. Handle different methods
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
                    "processData" -> {
                        // Process complex data
                        val data = call.arguments as? Map<*, *>
                        if (data != null) {
                            // Process the data
                            val processedData = mutableMapOf<String, Any>()
                            processedData["status"] = "success"
                            processedData["processed"] = data.size
                            processedData["timestamp"] = System.currentTimeMillis()
                            result.success(processedData)
                        } else {
                            result.error("INVALID_DATA", "Invalid data received", null)
                        }
                    }
                    else -> {
                        // Method not implemented
                        result.notImplemented()
                    }
                }
            }
    }
}
```

What's happening here?
- Channel name must match Dart side
- MethodCallHandler handles all calls
- call.arguments extracts parameters
- result.success returns data
- result.error handles errors

---

# Native Implementation - iOS

> **Implementing** MethodChannel on iOS.

```swift
// AppDelegate.swift (iOS)

import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    // 1. Channel name must match Dart side
    private let CHANNEL = "com.example.app/methods"

    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        // 2. Get the root view controller
        let controller: FlutterViewController = window?.rootViewController as! FlutterViewController
        
        // 3. Create MethodChannel
        let channel = FlutterMethodChannel(
            name: CHANNEL,
            binaryMessenger: controller.binaryMessenger
        )
        
        // 4. Set method call handler
        channel.setMethodCallHandler { (call: FlutterMethodCall, result: @escaping FlutterResult) in
            // 5. Handle different methods
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
                
            case "processData":
                // Process complex data
                guard let data = call.arguments as? [String: Any] else {
                    result(FlutterError(
                        code: "INVALID_DATA",
                        message: "Invalid data received",
                        details: nil
                    ))
                    return
                }
                
                let processedData: [String: Any] = [
                    "status": "success",
                    "processed": data.count,
                    "timestamp": Int(Date().timeIntervalSince1970 * 1000)
                ]
                result(processedData)
                
            default:
                // Method not implemented
                result(FlutterMethodNotImplemented)
            }
        }
        
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
}
```

What's happening here?
- Channel name matches Dart side
- Method call handler handles all calls
- call.arguments extracts parameters
- FlutterResult returns data
- FlutterError handles errors

---

# Bidirectional Communication

> **Native to Flutter** method calls.

```dart
/// 2. Receiving calls from native
class NativeToFlutterExample extends StatefulWidget {
  const NativeToFlutterExample({super.key});

  @override
  State<NativeToFlutterExample> createState() => _NativeToFlutterExampleState();
}

class _NativeToFlutterExampleState extends State<NativeToFlutterExample> {
  // 1. Create MethodChannel for receiving calls
  static const MethodChannel _channel = MethodChannel('com.example.app/callback');

  String _lastEvent = 'No events';
  int _eventCount = 0;

  @override
  void initState() {
    super.initState();
    // 2. Set up method call handler
    _channel.setMethodCallHandler(_handleMethodCall);
  }

  // 3. Handle incoming method calls from native
  Future<dynamic> _handleMethodCall(MethodCall call) async {
    switch (call.method) {
      case 'onEvent':
        // Handle event from native
        final data = call.arguments as Map<dynamic, dynamic>;
        setState(() {
          _lastEvent = 'Event: ${data['message']}';
          _eventCount++;
        });
        return {'status': 'received'};
        
      case 'onDataUpdate':
        // Handle data update
        final data = call.arguments as Map<dynamic, dynamic>;
        print('Data updated: $data');
        return {'status': 'processed'};
        
      default:
        throw PlatformException(
          code: 'NOT_IMPLEMENTED',
          message: 'Method not implemented',
        );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Native to Flutter'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Events Received',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'Total: $_eventCount',
                      style: const TextStyle(fontSize: 20),
                    ),
                  ],
                ),
              ),
            ),
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Last Event',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    Text(_lastEvent),
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
- setMethodCallHandler receives native calls
- MethodCall contains method name and arguments
- Return values are sent back to native
- Multiple methods can be handled

---

# Real-World Examples

> **Common patterns** with MethodChannel.

```dart
/// 1. Native permission service
class PermissionService {
  static const MethodChannel _channel = MethodChannel('com.example.app/permissions');

  static Future<bool> requestPermission(String permission) async {
    try {
      final bool result = await _channel.invokeMethod(
        'requestPermission',
        {'permission': permission},
      );
      return result;
    } on PlatformException catch (e) {
      print('Permission error: ${e.message}');
      return false;
    }
  }

  static Future<bool> checkPermission(String permission) async {
    try {
      final bool result = await _channel.invokeMethod(
        'checkPermission',
        {'permission': permission},
      );
      return result;
    } on PlatformException catch (e) {
      return false;
    }
  }

  static Future<Map<String, bool>> getGrantedPermissions() async {
    try {
      final result = await _channel.invokeMethod('getGrantedPermissions');
      return Map<String, bool>.from(result);
    } on PlatformException catch (e) {
      return {};
    }
  }
}

/// 2. Native file service
class FileService {
  static const MethodChannel _channel = MethodChannel('com.example.app/files');

  static Future<String?> readFile(String path) async {
    try {
      final String content = await _channel.invokeMethod(
        'readFile',
        {'path': path},
      );
      return content;
    } on PlatformException catch (e) {
      print('Read error: ${e.message}');
      return null;
    }
  }

  static Future<bool> writeFile(String path, String content) async {
    try {
      final bool result = await _channel.invokeMethod(
        'writeFile',
        {'path': path, 'content': content},
      );
      return result;
    } on PlatformException catch (e) {
      print('Write error: ${e.message}');
      return false;
    }
  }

  static Future<bool> deleteFile(String path) async {
    try {
      final bool result = await _channel.invokeMethod(
        'deleteFile',
        {'path': path},
      );
      return result;
    } on PlatformException catch (e) {
      return false;
    }
  }

  static Future<List<String>> listFiles(String directory) async {
    try {
      final List<dynamic> result = await _channel.invokeMethod(
        'listFiles',
        {'directory': directory},
      );
      return result.cast<String>().toList();
    } on PlatformException catch (e) {
      return [];
    }
  }
}

/// 3. Native notification service
class NotificationService {
  static const MethodChannel _channel = MethodChannel('com.example.app/notifications');

  static Future<void> showNotification({
    required String title,
    required String body,
    String? channelId,
  }) async {
    try {
      await _channel.invokeMethod('showNotification', {
        'title': title,
        'body': body,
        'channelId': channelId ?? 'default',
      });
    } on PlatformException catch (e) {
      print('Notification error: ${e.message}');
    }
  }

  static Future<void> cancelNotification(int id) async {
    try {
      await _channel.invokeMethod('cancelNotification', {'id': id});
    } on PlatformException catch (e) {
      print('Cancel error: ${e.message}');
    }
  }

  static Future<void> cancelAllNotifications() async {
    try {
      await _channel.invokeMethod('cancelAllNotifications');
    } on PlatformException catch (e) {
      print('Cancel all error: ${e.message}');
    }
  }
}
```

What's happening here?
- Permission service with MethodChannel
- File service with CRUD operations
- Notification service with native alerts
- Error handling and type conversion

---

# Best Practices

## Use Strong Typing

```dart
// Good - Type-safe methods
class NativeService {
  static Future<String> getString() async {
    final String result = await _channel.invokeMethod('getString');
    return result;
  }
}

// Bad - Dynamic typing
class NativeService {
  static Future getString() async {
    final result = await _channel.invokeMethod('getString');
    return result; // No type safety
  }
}
```

## Handle Errors Gracefully

```dart
// Good - Error handling
try {
  final result = await _channel.invokeMethod('methodName');
} on PlatformException catch (e) {
  // Handle platform error
} catch (e) {
  // Handle other errors
}
```

## Document Method Signatures

```dart
// Good - Documented methods
class NativeMethods {
  /// Gets the device information
  /// Returns: String with device info
  /// Throws: PlatformException if error occurs
  static Future<String> getDeviceInfo() async {
    // Implementation
  }
}
```

---

# Common Mistakes

## Channel Name Mismatch

Wrong:
```dart
// Dart side
static const MethodChannel _channel = MethodChannel('app/methods');

// Native side (Android)
val CHANNEL = "com.example.app/methods" // Mismatch!
```

Correct:
```dart
// Same name on both sides
static const MethodChannel _channel = MethodChannel('com.example.app/methods');

// Native side (Android)
val CHANNEL = "com.example.app/methods" // Match!
```

## Not Checking Arguments

Wrong:
```dart
// Native side
val operation = call.argument<String>("operation") // Might be null
```

Correct:
```dart
// Check arguments
val operation = call.argument<String>("operation")
if (operation == null) {
  result.error("INVALID_ARGUMENT", "Missing operation", null)
  return
}
```

---

# Summary

MethodChannel enables calling native methods from Flutter and receiving results. Use invokeMethod() for calling, setMethodCallHandler for receiving, and handle errors properly. MethodChannel is essential for accessing platform-specific features.

---

# Next Steps

- [EventChannel](eventchannel.md)
- [FFI](ffi.md)
- [Plugins](plugins.md)

---

# Did You Know?

- MethodChannel calls native methods
- invokeMethod() passes parameters
- setMethodCallHandler receives calls
- PlatformException handles errors
- Type safety improves reliability
- Channel names must match exactly
- Parameters must be JSON-serializable
- Methods run on platform thread