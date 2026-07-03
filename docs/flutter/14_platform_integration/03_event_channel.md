# EventChannel

Understand how to receive continuous streams of events from native platform code in Flutter.

---

# What is it?

EventChannel is a type of platform channel that enables receiving streams of data from native platform code (Android/iOS) to Flutter. Unlike MethodChannel which handles discrete method calls, EventChannel is designed for continuous streams of events like sensor data, location updates, battery level changes, or other real-time data.

---

# Why does it exist?

EventChannel exists to:

- Receive continuous data streams from native
- Handle real-time sensor data
- Process location updates
- Monitor battery and system events
- Listen to hardware events
- Receive broadcast notifications
- Support streaming data

---

# Basic EventChannel

> **Creating and using** EventChannel.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

/// Basic EventChannel example
class BasicEventChannelExample extends StatefulWidget {
  const BasicEventChannelExample({super.key});

  @override
  State<BasicEventChannelExample> createState() => _BasicEventChannelExampleState();
}

class _BasicEventChannelExampleState extends State<BasicEventChannelExample> {
  // 1. Create EventChannel with unique name
  static const EventChannel _eventChannel = EventChannel('com.example.app/events');

  // 2. Create a stream subscription
  StreamSubscription? _subscription;
  List<String> _events = [];
  String _lastEvent = 'No events';
  int _eventCount = 0;

  @override
  void initState() {
    super.initState();
    // 3. Start listening to events
    _startListening();
  }

  @override
  void dispose() {
    // 4. Cancel subscription to prevent memory leaks
    _subscription?.cancel();
    super.dispose();
  }

  void _startListening() {
    // 5. Receive broadcast stream of events
    _subscription = _eventChannel.receiveBroadcastStream().listen(
      (event) {
        // 6. Handle received event
        setState(() {
          final eventString = event.toString();
          _events.add(eventString);
          if (_events.length > 20) {
            _events.removeAt(0);
          }
          _lastEvent = eventString;
          _eventCount++;
        });
      },
      onError: (error) {
        // 7. Handle errors
        setState(() {
          _lastEvent = 'Error: $error';
        });
      },
      onDone: () {
        // 8. Handle stream completion
        setState(() {
          _lastEvent = 'Stream ended';
        });
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('EventChannel'),
        actions: [
          // 9. Clear events
          IconButton(
            icon: const Icon(Icons.clear),
            onPressed: () {
              setState(() {
                _events.clear();
                _eventCount = 0;
              });
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Event statistics
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: [
                    Column(
                      children: [
                        const Text(
                          'Total Events',
                          style: TextStyle(fontWeight: FontWeight.bold),
                        ),
                        Text(
                          '$_eventCount',
                          style: const TextStyle(
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ],
                    ),
                    Column(
                      children: [
                        const Text(
                          'Last Event',
                          style: TextStyle(fontWeight: FontWeight.bold),
                        ),
                        Text(
                          _lastEvent,
                          style: const TextStyle(fontSize: 14),
                          softWrap: true,
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            // Event list
            Expanded(
              child: Card(
                child: Padding(
                  padding: const EdgeInsets.all(8),
                  child: _events.isEmpty
                      ? const Center(
                          child: Text('No events received yet'),
                        )
                      : ListView.builder(
                          itemCount: _events.length,
                          itemBuilder: (context, index) {
                            return ListTile(
                              leading: CircleAvatar(
                                child: Text('${_events.length - index}'),
                              ),
                              title: Text(
                                _events[_events.length - 1 - index],
                                style: const TextStyle(fontSize: 12),
                              ),
                              dense: true,
                            );
                          },
                        ),
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
- EventChannel for streaming events
- receiveBroadcastStream() listens to events
- StreamSubscription manages the stream
- Events are processed in real-time

---

# Native Implementation - Android

> **Implementing** EventChannel on Android.

```kotlin
// MainActivity.kt (Android)

package com.example.app

import android.os.Build
import android.os.Handler
import android.os.Looper
import androidx.annotation.NonNull
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.EventChannel
import io.flutter.plugin.common.EventChannel.EventSink
import io.flutter.plugin.common.EventChannel.StreamHandler

class MainActivity : FlutterActivity() {
    // 1. Channel name must match Dart side
    private val CHANNEL = "com.example.app/events"

    // 2. Handler for sending events
    private var eventSink: EventSink? = null
    private val handler = Handler(Looper.getMainLooper())
    private var counter = 0

    override fun configureFlutterEngine(@NonNull flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        // 3. Create EventChannel
        EventChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
            .setStreamHandler(
                object : StreamHandler {
                    // 4. Called when listener is attached
                    override fun onListen(arguments: Any?, events: EventSink) {
                        eventSink = events
                        // Start sending events
                        startSendingEvents()
                    }

                    // 5. Called when listener is detached
                    override fun onCancel(arguments: Any?) {
                        eventSink = null
                        stopSendingEvents()
                    }
                }
            )
    }

    // 6. Send events to Flutter
    private fun startSendingEvents() {
        // Send a stream of events
        handler.post(object : Runnable {
            override fun run() {
                if (eventSink != null) {
                    counter++
                    val event = mapOf(
                        "id" to counter,
                        "timestamp" to System.currentTimeMillis(),
                        "message" to "Event #$counter",
                        "data" to "Sample data $counter"
                    )
                    // Send event to Flutter
                    eventSink?.success(event)
                    // Schedule next event
                    handler.postDelayed(this, 2000)
                }
            }
        })
    }

    private fun stopSendingEvents() {
        handler.removeCallbacksAndMessages(null)
    }
}
```

What's happening here?
- EventChannel streams events
- StreamHandler manages listeners
- onListen starts streaming
- onCancel stops streaming
- EventSink sends events to Flutter

---

# Native Implementation - iOS

> **Implementing** EventChannel on iOS.

```swift
// AppDelegate.swift (iOS)

import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    // 1. Channel name must match Dart side
    private let CHANNEL = "com.example.app/events"
    
    // 2. Handler for sending events
    private var eventSink: FlutterEventSink?
    private var timer: Timer?
    private var counter = 0

    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        // 3. Get the root view controller
        let controller: FlutterViewController = window?.rootViewController as! FlutterViewController
        
        // 4. Create EventChannel
        let eventChannel = FlutterEventChannel(
            name: CHANNEL,
            binaryMessenger: controller.binaryMessenger
        )
        
        // 5. Set stream handler
        eventChannel.setStreamHandler(self)
        
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
}

// 6. Implement StreamHandler
extension AppDelegate: FlutterStreamHandler {
    // 7. Called when listener is attached
    func onListen(
        withArguments arguments: Any?,
        eventSink events: @escaping FlutterEventSink
    ) -> FlutterError? {
        self.eventSink = events
        startSendingEvents()
        return nil
    }
    
    // 8. Called when listener is detached
    func onCancel(withArguments arguments: Any?) -> FlutterError? {
        eventSink = nil
        stopSendingEvents()
        return nil
    }
    
    // 9. Send events to Flutter
    private func startSendingEvents() {
        timer = Timer.scheduledTimer(withTimeInterval: 2.0, repeats: true) { _ in
            if let sink = self.eventSink {
                self.counter += 1
                let event: [String: Any] = [
                    "id": self.counter,
                    "timestamp": Int(Date().timeIntervalSince1970 * 1000),
                    "message": "Event #\(self.counter)",
                    "data": "Sample data \(self.counter)"
                ]
                sink(event)
            }
        }
    }
    
    private func stopSendingEvents() {
        timer?.invalidate()
        timer = nil
    }
}
```

What's happening here?
- EventChannel streams events
- FlutterStreamHandler manages listeners
- onListen starts streaming
- onCancel stops streaming
- FlutterEventSink sends events

---

# Real-World Examples

> **Common patterns** with EventChannel.

```dart
/// 1. Battery level listener
class BatteryListener {
  static const EventChannel _channel = EventChannel('com.example.app/battery');

  static Stream<int> get batteryLevelStream {
    return _channel.receiveBroadcastStream().map((event) {
      return event as int;
    });
  }

  static Stream<bool> get chargingStream {
    return _channel.receiveBroadcastStream('charging').map((event) {
      return event as bool;
    });
  }
}

/// 2. Location listener
class LocationListener {
  static const EventChannel _channel = EventChannel('com.example.app/location');

  static Stream<Map<String, double>> get locationStream {
    return _channel.receiveBroadcastStream().map((event) {
      final data = event as Map<dynamic, dynamic>;
      return {
        'latitude': data['latitude'] as double,
        'longitude': data['longitude'] as double,
        'accuracy': data['accuracy'] as double,
      };
    });
  }

  static Stream<bool> get locationEnabledStream {
    return _channel.receiveBroadcastStream('enabled').map((event) {
      return event as bool;
    });
  }
}

/// 3. Sensor data listener
class SensorListener {
  static const EventChannel _channel = EventChannel('com.example.app/sensors');

  static Stream<Map<String, double>> get accelerometerStream {
    return _channel.receiveBroadcastStream('accelerometer').map((event) {
      final data = event as Map<dynamic, dynamic>;
      return {
        'x': data['x'] as double,
        'y': data['y'] as double,
        'z': data['z'] as double,
      };
    });
  }

  static Stream<Map<String, double>> get gyroscopeStream {
    return _channel.receiveBroadcastStream('gyroscope').map((event) {
      final data = event as Map<dynamic, dynamic>;
      return {
        'x': data['x'] as double,
        'y': data['y'] as double,
        'z': data['z'] as double,
      };
    });
  }

  static Stream<double> get lightSensorStream {
    return _channel.receiveBroadcastStream('light').map((event) {
      return event as double;
    });
  }
}
```

What's happening here?
- Battery level monitoring
- Location updates streaming
- Sensor data processing
- Type-safe stream transformations

---

# Best Practices

## Cancel Subscriptions

```dart
// Good - Cancel subscription
@override
void dispose() {
  _subscription?.cancel();
  super.dispose();
}

// Bad - Memory leak
@override
void dispose() {
  super.dispose();
}
```

## Handle Errors

```dart
// Good - Error handling
_subscription = _channel.receiveBroadcastStream().listen(
  (event) => handleEvent(event),
  onError: (error) => handleError(error),
);
```

## Use Transformations

```dart
// Good - Type-safe streams
Stream<int> get intStream {
  return _channel.receiveBroadcastStream().map((event) => event as int);
}
```

---

# Common Mistakes

## Not Cancelling Subscription

Wrong:
```dart
// Memory leak
@override
void initState() {
  super.initState();
  _channel.receiveBroadcastStream().listen((event) {});
}
```

Correct:
```dart
// Proper disposal
StreamSubscription? _subscription;

@override
void initState() {
  super.initState();
  _subscription = _channel.receiveBroadcastStream().listen((event) {});
}

@override
void dispose() {
  _subscription?.cancel();
  super.dispose();
}
```

## Not Handling onCancel

Wrong:
```dart
// Native side
override fun onCancel(arguments: Any?) {
  // Not stopping event source
}
```

Correct:
```dart
// Proper cleanup
override fun onCancel(arguments: Any?) {
  eventSink = null
  stopSendingEvents()
}
```

---

# Summary

EventChannel enables streaming events from native code to Flutter. Use receiveBroadcastStream() to listen to events, handle errors and completion, and cancel subscriptions to prevent memory leaks. EventChannel is essential for real-time data streams.

---

# Next Steps

- [FFI](ffi.md)
- [Plugins](plugins.md)
- [Platform Views](platform-views.md)

---

# Did You Know?

- EventChannel streams continuous events
- receiveBroadcastStream() listens to events
- StreamSubscription manages the stream
- onListen starts streaming on native side
- onCancel stops streaming
- Events can be transformed in Dart
- Multiple streams can exist
- Memory leaks if not disposed