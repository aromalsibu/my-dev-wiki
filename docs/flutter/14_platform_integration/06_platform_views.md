# Platform Views

Understand how to embed native Android and iOS views into Flutter applications.

---

# What is it?

Platform Views allow you to embed native Android (View) and iOS (UIView) components directly into your Flutter widget tree. This is useful when you need to use platform-specific UI components that aren't available in Flutter, such as Google Maps, WebViews, or native media players.

---

# Why does it exist?

Platform Views exist to:

- Embed native UI components in Flutter
- Use platform-specific widgets
- Display web content with WebView
- Show native maps
- Play video with native players
- Integrate with platform SDKs
- Handle complex native UI

---

# AndroidView

> **Embedding Android** views in Flutter.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

/// AndroidView example
class AndroidViewExample extends StatelessWidget {
  const AndroidViewExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('AndroidView'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. AndroidView widget
            // This embeds a native Android View
            const Text(
              'Native Android View',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            SizedBox(
              height: 200,
              width: 300,
              child: AndroidView(
                // 2. View type identifier
                viewType: 'native_view',
                // 3. Creation parameters
                creationParams: {
                  'color': 0xFF2196F3,
                  'text': 'Hello from Native!',
                },
                creationParamsCodec: const StandardMessageCodec(),
                // 4. Layout parameters
                layoutDirection: TextDirection.ltr,
                // 5. Gesture detection
                gestureRecognizers: const {},
              ),
            ),
            const SizedBox(height: 16),
            const Text(
              'This is a native Android View embedded in Flutter',
              style: TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- AndroidView embeds native Android views
- viewType identifies the view type
- creationParams pass initial data
- Native view appears in Flutter

---

# UiKitView

> **Embedding iOS** views in Flutter.

```dart
/// UiKitView example (iOS)
class UiKitViewExample extends StatelessWidget {
  const UiKitViewExample({super.key});

  @override
  Widget build(BuildContext context) {
    // This widget only works on iOS
    if (!Platform.isIOS) {
      return Scaffold(
        appBar: AppBar(
          title: const Text('UiKitView'),
        ),
        body: const Center(
          child: Text('UiKitView is only available on iOS'),
        ),
      );
    }

    return Scaffold(
      appBar: AppBar(
        title: const Text('UiKitView'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. UiKitView widget
            // This embeds a native iOS UIView
            const Text(
              'Native iOS View',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            SizedBox(
              height: 200,
              width: 300,
              child: UiKitView(
                // 2. View type identifier
                viewType: 'native_view',
                // 3. Creation parameters
                creationParams: {
                  'color': 0xFF2196F3,
                  'text': 'Hello from Native!',
                },
                creationParamsCodec: const StandardMessageCodec(),
                // 4. Layout parameters
                layoutDirection: TextDirection.ltr,
                // 5. Gesture detection
                gestureRecognizers: const {},
              ),
            ),
            const SizedBox(height: 16),
            const Text(
              'This is a native iOS View embedded in Flutter',
              style: TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- UiKitView embeds native iOS views
- viewType identifies the view type
- creationParams pass initial data
- Native view appears in Flutter

---

# Platform View Factory

> **Registering** platform view factories.

```kotlin
// Android implementation - View Factory

package com.example.app

import android.content.Context
import android.graphics.Color
import android.view.View
import android.widget.TextView
import io.flutter.plugin.common.StandardMessageCodec
import io.flutter.plugin.platform.PlatformView
import io.flutter.plugin.platform.PlatformViewFactory

// 1. PlatformView implementation
class NativeView(context: Context, creationParams: Map<String, Any>?) : PlatformView {
    private val view: TextView

    init {
        // Create the native view
        view = TextView(context)
        view.text = creationParams?.get("text") as? String ?: "Default Text"
        view.setTextColor(Color.WHITE)
        view.setBackgroundColor(creationParams?.get("color") as? Int ?: Color.BLUE)
        view.textSize = 20f
        view.gravity = android.view.Gravity.CENTER
    }

    // 2. Return the native view
    override fun getView(): View {
        return view
    }

    // 3. Dispose of resources
    override fun dispose() {
        // Clean up resources if needed
    }
}

// 4. PlatformViewFactory
class NativeViewFactory : PlatformViewFactory(StandardMessageCodec.INSTANCE) {
    override fun create(context: Context, viewId: Int, args: Any?): PlatformView {
        // Parse creation parameters
        val creationParams = args as? Map<String, Any>
        return NativeView(context, creationParams)
    }
}

// 5. Register the factory in MainActivity
class MainActivity : FlutterActivity() {
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        
        flutterEngine.platformViewsController.registry
            .registerViewFactory("native_view", NativeViewFactory())
    }
}
```

What's happening here?
- PlatformView creates native view
- PlatformViewFactory creates views
- registerViewFactory registers the factory
- Creation params pass data

---

# Platform View with Interaction

> **Interactive** platform views.

```dart
/// Interactive platform view example
class InteractivePlatformView extends StatefulWidget {
  const InteractivePlatformView({super.key});

  @override
  State<InteractivePlatformView> createState() => _InteractivePlatformViewState();
}

class _InteractivePlatformViewState extends State<InteractivePlatformView> {
  // 1. Controller for platform view
  final TextEditingController _messageController = TextEditingController();
  String _nativeMessage = 'No message';

  // 2. MethodChannel for communication
  static const MethodChannel _channel = MethodChannel('com.example.app/platform_view');

  // 3. Send message to native view
  Future<void> _sendMessageToNative() async {
    try {
      await _channel.invokeMethod('updateMessage', {
        'message': _messageController.text,
      });
      setState(() {
        _nativeMessage = 'Sent: ${_messageController.text}';
      });
    } catch (e) {
      print('Error sending message: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Interactive Platform View'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Platform view
            SizedBox(
              height: 200,
              width: double.infinity,
              child: AndroidView(
                viewType: 'interactive_view',
                creationParams: {
                  'initialText': 'Tap the button below',
                },
                creationParamsCodec: const StandardMessageCodec(),
                onPlatformViewCreated: (id) {
                  // 4. Platform view created
                  print('Platform view created: $id');
                },
              ),
            ),
            const SizedBox(height: 16),
            // Message input
            TextField(
              controller: _messageController,
              decoration: const InputDecoration(
                labelText: 'Message to Native',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 8),
            // Send button
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _sendMessageToNative,
                    child: const Text('Send to Native'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      setState(() {
                        _nativeMessage = 'No message';
                      });
                    },
                    child: const Text('Clear'),
                  ),
                ),
              ],
            ),
            // Message display
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.grey[100],
              child: Text(
                'Native Message: $_nativeMessage',
                style: const TextStyle(fontWeight: FontWeight.bold),
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
- MethodChannel for communication
- sendMessageToNative sends data
- Platform view can respond
- Interactive native views

---

# Real-World Examples

> **Common patterns** with platform views.

```dart
/// 1. WebView platform view
class WebViewPlatform extends StatelessWidget {
  const WebViewPlatform({
    super.key,
    required this.url,
    this.height = 300,
  });

  final String url;
  final double height;

  @override
  Widget build(BuildContext context) {
    if (Platform.isAndroid) {
      return SizedBox(
        height: height,
        child: AndroidView(
          viewType: 'webview',
          creationParams: {
            'url': url,
          },
          creationParamsCodec: const StandardMessageCodec(),
        ),
      );
    } else if (Platform.isIOS) {
      return SizedBox(
        height: height,
        child: UiKitView(
          viewType: 'webview',
          creationParams: {
            'url': url,
          },
          creationParamsCodec: const StandardMessageCodec(),
        ),
      );
    } else {
      return Container(
        height: height,
        color: Colors.grey[200],
        child: const Center(
          child: Text('WebView not supported on this platform'),
        ),
      );
    }
  }
}

/// 2. Map platform view
class MapPlatform extends StatelessWidget {
  const MapPlatform({
    super.key,
    this.latitude = 0.0,
    this.longitude = 0.0,
    this.zoom = 15,
    this.height = 300,
  });

  final double latitude;
  final double longitude;
  final double zoom;
  final double height;

  @override
  Widget build(BuildContext context) {
    if (Platform.isAndroid) {
      return SizedBox(
        height: height,
        child: AndroidView(
          viewType: 'map_view',
          creationParams: {
            'latitude': latitude,
            'longitude': longitude,
            'zoom': zoom,
          },
          creationParamsCodec: const StandardMessageCodec(),
        ),
      );
    } else if (Platform.isIOS) {
      return SizedBox(
        height: height,
        child: UiKitView(
          viewType: 'map_view',
          creationParams: {
            'latitude': latitude,
            'longitude': longitude,
            'zoom': zoom,
          },
          creationParamsCodec: const StandardMessageCodec(),
        ),
      );
    } else {
      return Container(
        height: height,
        color: Colors.grey[200],
        child: const Center(
          child: Text('Map not supported on this platform'),
        ),
      );
    }
  }
}

/// 3. Video player platform view
class VideoPlayerPlatform extends StatelessWidget {
  const VideoPlayerPlatform({
    super.key,
    required this.videoUrl,
    this.height = 300,
  });

  final String videoUrl;
  final double height;

  @override
  Widget build(BuildContext context) {
    if (Platform.isAndroid) {
      return SizedBox(
        height: height,
        child: AndroidView(
          viewType: 'video_player',
          creationParams: {
            'url': videoUrl,
            'autoPlay': true,
          },
          creationParamsCodec: const StandardMessageCodec(),
        ),
      );
    } else if (Platform.isIOS) {
      return SizedBox(
        height: height,
        child: UiKitView(
          viewType: 'video_player',
          creationParams: {
            'url': videoUrl,
            'autoPlay': true,
          },
          creationParamsCodec: const StandardMessageCodec(),
        ),
      );
    } else {
      return Container(
        height: height,
        color: Colors.grey[200],
        child: const Center(
          child: Text('Video player not supported on this platform'),
        ),
      );
    }
  }
}
```

What's happening here?
- WebView platform view
- Map platform view
- Video player platform view
- Platform-specific implementations

---

# Best Practices

## Handle Platform Differences

```dart
// Good - Platform-specific handling
if (Platform.isAndroid) {
  return AndroidView(viewType: 'view');
} else if (Platform.isIOS) {
  return UiKitView(viewType: 'view');
}
```

## Use Communication Channels

```dart
// Good - MethodChannel for communication
final channel = MethodChannel('com.example.app/platform_view');
channel.setMethodCallHandler((call) {
  // Handle calls from native
});
```

## Dispose Resources

```dart
// Good - Dispose native views
override fun dispose() {
  // Clean up resources
  // Remove listeners
  // Release memory
}
```

---

# Common Mistakes

## Not Registering View Factory

Wrong:
```dart
// Missing registration
// Platform view factory not registered
```

Correct:
```dart
// Register factory
flutterEngine.platformViewsController.registry
    .registerViewFactory("view_type", ViewFactory())
```

## Wrong Platform Checks

Wrong:
```dart
// Incorrect platform check
if (Platform.isAndroid) {
  // Android code
} else {
  // Assumes iOS
}
```

Correct:
```dart
// Explicit platform checks
if (Platform.isAndroid) {
  // Android code
} else if (Platform.isIOS) {
  // iOS code
} else {
  // Unsupported
}
```

---

# Summary

Platform Views embed native Android and iOS views in Flutter. Use AndroidView for Android, UiKitView for iOS, and register view factories for each platform. Handle platform differences and use communication channels for interaction.

---

# Next Steps

- [MethodChannel](methodchannel.md)
- [EventChannel](eventchannel.md)
- [FFI](ffi.md)

---

# Did You Know?

- AndroidView embeds Android views
- UiKitView embeds iOS views
- PlatformViewFactory creates views
- MethodChannel enables communication
- Views can be interactive
- WebView is a common use case
- Maps can be embedded
- Video players can be embedded