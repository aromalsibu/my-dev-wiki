# Platform Channels

Understand how to communicate between Flutter and native platform code using platform channels.

---

# What is it?

Platform Channels are a mechanism in Flutter that enables communication between Dart code and native platform code (Android/Kotlin/Java and iOS/Swift/Objective-C). They allow you to call platform-specific APIs, access device features, and integrate with native SDKs. Platform channels provide a flexible way to extend Flutter's capabilities when built-in plugins don't cover your needs.

---

# Why does it exist?

Platform Channels exist to:

- Access platform-specific APIs
- Call native code from Flutter
- Receive native events in Flutter
- Integrate with platform SDKs
- Use device-specific features
- Handle platform-specific logic
- Extend Flutter capabilities

---

# MethodChannel Basics

> **Creating and using** MethodChannel.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

/// 1. Dart side - MethodChannel usage
class MethodChannelExample extends StatefulWidget {
  const MethodChannelExample({super.key});

  @override
  State<MethodChannelExample> createState() => _MethodChannelExampleState();
}

class _MethodChannelExampleState extends State<MethodChannelExample> {
  // 1. Create a MethodChannel with a unique name
  // The name must match the native side channel name
  static const MethodChannel _channel = MethodChannel('com.example.app/platform');

  // State variables
  String _platformVersion = 'Unknown';
  String _batteryLevel = 'Unknown';
  bool _isLoading = false;

  @override
  void initState() {
    super.initState();
    _getPlatformVersion();
  }

  // 2. Call a native method
  Future<void> _getPlatformVersion() async {
    try {
      // Invoke a method on the native side
      final String result = await _channel.invokeMethod('getPlatformVersion');
      setState(() {
        _platformVersion = result;
      });
    } on PlatformException catch (e) {
      setState(() {
        _platformVersion = 'Failed to get platform version: ${e.message}';
      });
    }
  }

  // 3. Call a method with parameters
  Future<void> _getBatteryLevel() async {
    setState(() => _isLoading = true);

    try {
      // Pass parameters to the native method
      final int result = await _channel.invokeMethod('getBatteryLevel');
      setState(() {
        _batteryLevel = '$result%';
        _isLoading = false;
      });
    } on PlatformException catch (e) {
      setState(() {
        _batteryLevel = 'Failed to get battery level: ${e.message}';
        _isLoading = false;
      });
    }
  }

  // 4. Call a method with complex parameters
  Future<void> _showNativeToast(String message) async {
    try {
      // Pass a map of parameters
      await _channel.invokeMethod('showToast', {
        'message': message,
        'duration': 2,
      });
    } on PlatformException catch (e) {
      print('Failed to show toast: ${e.message}');
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
            // Platform version
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Platform Version',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    Text(_platformVersion),
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
                    if (_isLoading)
                      const CircularProgressIndicator()
                    else
                      Text(
                        _batteryLevel,
                        style: const TextStyle(fontSize: 20),
                      ),
                    const SizedBox(height: 8),
                    ElevatedButton(
                      onPressed: _getBatteryLevel,
                      child: const Text('Get Battery Level'),
                    ),
                  ],
                ),
              ),
            ),
            // Toast button
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
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                      children: [
                        ElevatedButton(
                          onPressed: () {
                            _showNativeToast('Hello from Flutter!');
                          },
                          child: const Text('Show Toast'),
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
- invokeMethod() calls native code
- Parameters can be passed as arguments
- PlatformException handles errors

---

# Native Implementation - Android

> **Implementing** MethodChannel on Android.

```kotlin
// MainActivity.kt (Android)

package com.example.app

import android.os.Build
import android.os.BatteryManager
import android.content.Context
import android.widget.Toast
import androidx.annotation.NonNull
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel
import io.flutter.plugin.common.MethodChannel.MethodCallHandler
import io.flutter.plugin.common.MethodChannel.Result
import io.flutter.plugin.common.MethodCall

class MainActivity : FlutterActivity() {
    // 1. Channel name must match Dart side
    private val CHANNEL = "com.example.app/platform"

    override fun configureFlutterEngine(@NonNull flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        
        // 2. Create MethodChannel
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
            .setMethodCallHandler { call, result ->
                // 3. Handle method calls
                when (call.method) {
                    "getPlatformVersion" -> {
                        // Return the platform version
                        result.success("Android ${Build.VERSION.RELEASE}")
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
                    "showToast" -> {
                        // Show toast with message
                        val message = call.argument<String>("message")
                        val duration = call.argument<Int>("duration") ?: 1
                        if (message != null) {
                            Toast.makeText(
                                this,
                                message,
                                if (duration > 1) Toast.LENGTH_LONG else Toast.LENGTH_SHORT
                            ).show()
                            result.success(null)
                        } else {
                            result.error("INVALID_ARGUMENT", "Message is required", null)
                        }
                    }
                    else -> {
                        // Method not implemented
                        result.notImplemented()
                    }
                }
            }
    }

    // Helper method to get battery level
    private fun getBatteryLevel(): Int {
        val batteryManager = getSystemService(Context.BATTERY_SERVICE) as BatteryManager
        return batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
    }
}
```

What's happening here?
- Channel name matches Dart side
- MethodCallHandler handles calls
- result.success() returns data
- result.error() handles errors

---

# Native Implementation - iOS

> **Implementing** MethodChannel on iOS.

```swift
// AppDelegate.swift (iOS)

import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        // 1. Get the root view controller
        let controller: FlutterViewController = window?.rootViewController as! FlutterViewController
        
        // 2. Create MethodChannel with the same name
        let channel = FlutterMethodChannel(
            name: "com.example.app/platform",
            binaryMessenger: controller.binaryMessenger
        )
        
        // 3. Set method call handler
        channel.setMethodCallHandler { (call: FlutterMethodCall, result: @escaping FlutterResult) in
            // 4. Handle method calls
            switch call.method {
            case "getPlatformVersion":
                // Return iOS version
                result("iOS \(UIDevice.current.systemVersion)")
                
            case "getBatteryLevel":
                // Get battery level
                let batteryLevel = self.getBatteryLevel()
                if batteryLevel != -1 {
                    result(batteryLevel)
                } else {
                    result(FlutterError(
                        code: "UNAVAILABLE",
                        message: "Battery level not available",
                        details: nil
                    ))
                }
                
            case "showToast":
                // Show toast (using UIAlertController as iOS doesn't have built-in toast)
                if let args = call.arguments as? [String: Any],
                   let message = args["message"] as? String {
                    self.showToast(message: message)
                    result(nil)
                } else {
                    result(FlutterError(
                        code: "INVALID_ARGUMENT",
                        message: "Message is required",
                        details: nil
                    ))
                }
                
            default:
                // Method not implemented
                result(FlutterMethodNotImplemented)
            }
        }
        
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
    
    // Helper method to get battery level
    private func getBatteryLevel() -> Int {
        let device = UIDevice.current
        device.isBatteryMonitoringEnabled = true
        let batteryLevel = Int(device.batteryLevel * 100)
        device.isBatteryMonitoringEnabled = false
        return batteryLevel
    }
    
    // Helper method to show a toast-like alert
    private func showToast(message: String) {
        // Simple alert for demonstration
        let alert = UIAlertController(
            title: nil,
            message: message,
            preferredStyle: .alert
        )
        
        // Show alert on the root view controller
        if let rootViewController = window?.rootViewController {
            rootViewController.present(alert, animated: true)
            DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
                alert.dismiss(animated: true)
            }
        }
    }
}
```

What's happening here?
- Channel name matches Dart side
- Method call handler handles calls
- FlutterResult returns data
- FlutterError handles errors

---

# EventChannel

> **Receiving native** events in Flutter.

```dart
/// EventChannel example (Dart side)
class EventChannelExample extends StatefulWidget {
  const EventChannelExample({super.key});

  @override
  State<EventChannelExample> createState() => _EventChannelExampleState();
}

class _EventChannelExampleState extends State<EventChannelExample> {
  // 1. Create EventChannel
  static const EventChannel _eventChannel = EventChannel('com.example.app/events');

  // 2. Stream of events
  Stream<dynamic>? _eventStream;
  String _lastEvent = 'No events received';
  int _eventCount = 0;

  @override
  void initState() {
    super.initState();
    // 3. Listen to events
    _eventStream = _eventChannel.receiveBroadcastStream();
    _eventStream?.listen(
      (event) {
        // 4. Handle received events
        setState(() {
          _lastEvent = event.toString();
          _eventCount++;
        });
      },
      onError: (error) {
        setState(() {
          _lastEvent = 'Error: $error';
        });
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('EventChannel'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
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
- EventChannel for receiving native events
- receiveBroadcastStream() listens to events
- Events are pushed from native side
- Stream handles incoming data

---

# Real-World Examples

> **Common patterns** with platform channels.

```dart
/// 1. Native location service
class LocationService {
  static const MethodChannel _channel = MethodChannel('com.example.app/location');

  static Future<Map<String, double>> getLocation() async {
    try {
      final result = await _channel.invokeMethod('getLocation');
      return Map<String, double>.from(result);
    } on PlatformException catch (e) {
      throw Exception('Failed to get location: ${e.message}');
    }
  }

  static Future<void> startLocationUpdates() async {
    await _channel.invokeMethod('startLocationUpdates');
  }

  static Future<void> stopLocationUpdates() async {
    await _channel.invokeMethod('stopLocationUpdates');
  }
}

/// 2. Native storage service
class StorageService {
  static const MethodChannel _channel = MethodChannel('com.example.app/storage');

  static Future<void> saveData(String key, String value) async {
    await _channel.invokeMethod('saveData', {
      'key': key,
      'value': value,
    });
  }

  static Future<String?> loadData(String key) async {
    try {
      final result = await _channel.invokeMethod('loadData', {'key': key});
      return result as String?;
    } catch (e) {
      return null;
    }
  }

  static Future<void> deleteData(String key) async {
    await _channel.invokeMethod('deleteData', {'key': key});
  }

  static Future<void> clearAll() async {
    await _channel.invokeMethod('clearAll');
  }
}
```

What's happening here?
- Location service using MethodChannel
- Storage service with CRUD operations
- Error handling and result parsing

---

# Best Practices

## Handle Errors Gracefully

```dart
// Good - Error handling
try {
  final result = await _channel.invokeMethod('methodName');
} on PlatformException catch (e) {
  print('Platform error: ${e.message}');
} catch (e) {
  print('Other error: $e');
}
```

## Use Strong Typing

```dart
// Good - Type-safe results
final String result = await _channel.invokeMethod('getString');
final int number = await _channel.invokeMethod('getNumber');
```

## Organize Channel Names

```dart
// Good - Organized channel names
class Channels {
  static const String location = 'com.example.app/location';
  static const String storage = 'com.example.app/storage';
  static const String camera = 'com.example.app/camera';
}
```

---

# Common Mistakes

## Channel Name Mismatch

Wrong:
```dart
// Dart side
static const MethodChannel _channel = MethodChannel('app/platform');

// Native side (Android)
val CHANNEL = "com.example.app/platform" // Mismatch!
```

Correct:
```dart
// Same name on both sides
static const MethodChannel _channel = MethodChannel('com.example.app/platform');

// Native side (Android)
val CHANNEL = "com.example.app/platform" // Match!
```

## Not Handling Errors

Wrong:
```dart
// No error handling
final result = await _channel.invokeMethod('methodName');
```

Correct:
```dart
// With error handling
try {
  final result = await _channel.invokeMethod('methodName');
} on PlatformException catch (e) {
  // Handle error
}
```

---

# Summary

Platform Channels enable communication between Flutter and native code. Use MethodChannel for calling native methods, EventChannel for receiving native events, and handle errors properly. Platform channels are essential for accessing platform-specific features.

---

# Next Steps

- [MethodChannel](methodchannel.md)
- [EventChannel](eventchannel.md)
- [FFI](ffi.md)

---

# Did You Know?

- MethodChannel calls native code
- EventChannel receives native events
- Channel names must match exactly
- PlatformException handles errors
- Parameters can be any JSON-serializable type
- Native code runs on platform thread
- Channels are asynchronous
- Multiple channels can exist