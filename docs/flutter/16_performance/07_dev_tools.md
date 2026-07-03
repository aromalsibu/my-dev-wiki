# DevTools

Understand how to use Flutter DevTools for debugging, profiling, and optimizing your applications.

---

# What is it?

Flutter DevTools is a suite of performance and debugging tools that help you inspect, profile, and debug your Flutter applications. It provides a visual interface for analyzing widget trees, performance timelines, memory usage, network requests, and more. DevTools is essential for developing high-quality Flutter applications.

---

# Why does it exist?

DevTools exists to:

- Debug widget trees and layouts
- Profile application performance
- Analyze memory usage
- Inspect network requests
- Track CPU usage and frame timings
- Debug animations and rendering
- Optimize application performance

---

# Starting DevTools

> **Launching** and connecting DevTools.

```bash
# 1. Start DevTools from command line
flutter pub global run devtools

# 2. Start DevTools with your app
flutter run --devtools-server-address=http://127.0.0.1:9100

# 3. Open DevTools from IDE
# Android Studio: View > Tool Windows > Flutter Inspector
# VS Code: Command Palette > Flutter: Open DevTools

# 4. DevTools URL
# http://127.0.0.1:9100
```

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

/// DevTools setup example
class DevToolsSetupExample extends StatelessWidget {
  const DevToolsSetupExample({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'DevTools Example',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const DevToolsHome(),
    );
  }
}

class DevToolsHome extends StatefulWidget {
  const DevToolsHome({super.key});

  @override
  State<DevToolsHome> createState() => _DevToolsHomeState();
}

class _DevToolsHomeState extends State<DevToolsHome> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('DevTools Example'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Enable debug painting
            // This shows layout boundaries and helps debug layout issues
            const Text(
              'Enable Debug Painting:',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            ElevatedButton(
              onPressed: () {
                // Toggle debug painting
                debugPaintSizeEnabled = !debugPaintSizeEnabled;
                // Force rebuild to show changes
                setState(() {});
              },
              child: Text(
                debugPaintSizeEnabled ? 'Disable' : 'Enable',
              ),
            ),
            const SizedBox(height: 16),
            
            // 2. Widget with layout for inspection
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[100],
                border: Border.all(color: Colors.blue),
              ),
              child: Row(
                children: [
                  const Icon(Icons.star),
                  const SizedBox(width: 8),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        const Text(
                          'Widget Title',
                          style: TextStyle(fontWeight: FontWeight.bold),
                        ),
                        Text(
                          'Subtitle text goes here',
                          style: TextStyle(color: Colors.grey[600]),
                        ),
                      ],
                    ),
                  ),
                  const Icon(Icons.arrow_forward),
                ],
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
- DevTools runs in browser
- debugPaintSizeEnabled shows layout
- Visual debugging tools

---

# Widget Inspector

> **Inspecting** the widget tree.

```dart
/// Widget Inspector example
class WidgetInspectorExample extends StatelessWidget {
  const WidgetInspectorExample({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Widget Inspector',
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Widget Inspector'),
        ),
        body: const Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // 1. Simple widget
              _InspectedWidget(
                title: 'Widget 1',
                color: Colors.blue,
              ),
              SizedBox(height: 8),
              // 2. Another widget
              _InspectedWidget(
                title: 'Widget 2',
                color: Colors.green,
              ),
              SizedBox(height: 8),
              // 3. Complex widget
              _ComplexWidget(),
            ],
          ),
        ),
      ),
    );
  }
}

/// Widget that can be inspected
class _InspectedWidget extends StatelessWidget {
  const _InspectedWidget({
    required this.title,
    required this.color,
  });

  final String title;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: color[100],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Text(
        title,
        style: TextStyle(
          color: color[800],
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}

/// Complex widget with nested structure
class _ComplexWidget extends StatelessWidget {
  const _ComplexWidget();

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.grey[100],
      child: Row(
        children: [
          // Left column
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'Complex',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                const Text(
                  'Widget',
                  style: TextStyle(color: Colors.grey),
                ),
              ],
            ),
          ),
          // Right column
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
            decoration: BoxDecoration(
              color: Colors.blue[100],
              borderRadius: BorderRadius.circular(4),
            ),
            child: const Text('Tag'),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Inspector shows widget tree
- Widget properties visible
- Layout boundaries visible
- Debug widgets easily

---

# Performance Profiling

> **Measuring** application performance.

```dart
/// Performance profiling example
class PerformanceProfilingExample extends StatefulWidget {
  const PerformanceProfilingExample({super.key});

  @override
  State<PerformanceProfilingExample> createState() => _PerformanceProfilingExampleState();
}

class _PerformanceProfilingExampleState extends State<PerformanceProfilingExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  int _counter = 0;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Performance Profiling'),
        actions: [
          // 1. Performance overlay toggle
          IconButton(
            icon: const Icon(Icons.speed),
            onPressed: () {
              // Toggle performance overlay
              // This shows frame timings
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. Animated widget (good for profiling)
            RepaintBoundary(
              child: AnimatedBuilder(
                animation: _controller,
                builder: (context, child) {
                  return Container(
                    width: 100 + _controller.value * 100,
                    height: 100 + _controller.value * 100,
                    color: Colors.blue,
                    child: const Center(
                      child: Text(
                        'Animated',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  );
                },
              ),
            ),
            const SizedBox(height: 16),
            
            // 3. Counter (triggers rebuilds)
            Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 20),
            ),
            const SizedBox(height: 8),
            
            // 4. Buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Increment counter
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _counter++;
                    });
                  },
                  child: const Text('Increment'),
                ),
                const SizedBox(width: 8),
                // Heavy operation
                ElevatedButton(
                  onPressed: () {
                    // Simulate heavy operation
                    // This will show in performance profile
                    _performHeavyOperation();
                  },
                  child: const Text('Heavy Operation'),
                ),
              ],
            ),
            
            const SizedBox(height: 16),
            
            // 5. Performance info
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Column(
                children: [
                  Text(
                    'Performance Tips:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('• Use RepaintBoundary for animations'),
                  Text('• Avoid heavy operations on main thread'),
                  Text('• Use const widgets when possible'),
                  Text('• Monitor frame timings in DevTools'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  // Simulate heavy operation
  void _performHeavyOperation() {
    // This will cause a performance spike
    int result = 0;
    for (int i = 0; i < 1000000; i++) {
      result += i;
    }
    print('Heavy operation result: $result');
  }
}
```

What's happening here?
- Performance overlay shows frame timings
- Animated widgets for profiling
- Heavy operations visible in timeline
- Frame drops and jank detectable

---

# Memory Profiling

> **Analyzing** memory usage.

```dart
/// Memory profiling example
class MemoryProfilingExample extends StatefulWidget {
  const MemoryProfilingExample({super.key});

  @override
  State<MemoryProfilingExample> createState() => _MemoryProfilingExampleState();
}

class _MemoryProfilingExampleState extends State<MemoryProfilingExample> {
  // 1. Track memory usage
  List<Widget> _widgets = [];
  int _widgetCount = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Memory Profiling'),
        actions: [
          // 2. Memory info
          IconButton(
            icon: const Icon(Icons.memory),
            onPressed: () {
              // Show memory info
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text('Widgets: ${_widgets.length}'),
                ),
              );
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 3. Add/Remove widgets
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _addWidgets,
                    child: const Text('Add 100 Widgets'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _removeWidgets,
                    child: const Text('Remove All'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 8),
            
            // 4. Widget count
            Container(
              padding: const EdgeInsets.all(8),
              color: Colors.blue[50],
              child: Text(
                'Active Widgets: ${_widgets.length}',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 5. Widget list
            Expanded(
              child: ListView.builder(
                itemCount: _widgets.length,
                itemBuilder: (context, index) {
                  return _widgets[index];
                },
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 6. Memory tips
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.orange[50],
                borderRadius: BorderRadius.circular(8),
                border: Border.all(color: Colors.orange[200]!),
              ),
              child: const Column(
                children: [
                  Text(
                    'Memory Tips:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('• Avoid holding onto widget references'),
                  Text('• Dispose controllers and listeners'),
                  Text('• Use const widgets when possible'),
                  Text('• Monitor memory in DevTools'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  void _addWidgets() {
    setState(() {
      // Add 100 widgets (this will increase memory usage)
      for (int i = 0; i < 100; i++) {
        _widgets.add(
          Container(
            padding: const EdgeInsets.all(8),
            margin: const EdgeInsets.all(2),
            color: Colors.blue[100 * ((_widgetCount + i) % 9 + 1)],
            child: Text('Widget ${_widgetCount + i + 1}'),
          ),
        );
      }
      _widgetCount += 100;
    });
  }

  void _removeWidgets() {
    setState(() {
      _widgets.clear();
      _widgetCount = 0;
    });
  }
}
```

What's happening here?
- Memory usage visible in DevTools
- Widget creation and disposal
- Memory leak detection
- Heap snapshots

---

# Network Profiling

> **Inspecting** network requests.

```dart
/// Network profiling example
class NetworkProfilingExample extends StatefulWidget {
  const NetworkProfilingExample({super.key});

  @override
  State<NetworkProfilingExample> createState() => _NetworkProfilingExampleState();
}

class _NetworkProfilingExampleState extends State<NetworkProfilingExample> {
  // 1. Track network requests
  List<String> _requests = [];
  bool _isLoading = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Network Profiling'),
        actions: [
          IconButton(
            icon: const Icon(Icons.clear),
            onPressed: () {
              setState(() {
                _requests.clear();
              });
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. Network action buttons
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _makeNetworkRequest,
                    child: const Text('Make Request'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 8),
            
            // 3. Request count
            Container(
              padding: const EdgeInsets.all(8),
              color: Colors.green[50],
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  const Text(
                    'Requests:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  Text('${_requests.length}'),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 4. Request list
            Expanded(
              child: _requests.isEmpty
                  ? const Center(
                      child: Text('No network requests made'),
                    )
                  : ListView.builder(
                      itemCount: _requests.length,
                      itemBuilder: (context, index) {
                        return Card(
                          child: Padding(
                            padding: const EdgeInsets.all(8),
                            child: Text(
                              _requests[index],
                              style: const TextStyle(fontSize: 12),
                            ),
                          ),
                        );
                      },
                    ),
            ),
            
            const SizedBox(height: 16),
            
            // 5. Network tips
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Column(
                children: [
                  Text(
                    'Network Tips:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('• Check request/response times'),
                  Text('• Monitor error rates'),
                  Text('• Optimize payload sizes'),
                  Text('• Implement caching strategies'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  // Simulate network request
  void _makeNetworkRequest() async {
    setState(() {
      _isLoading = true;
    });

    try {
      // Simulate network delay
      await Future.delayed(const Duration(seconds: 1));
      
      // Simulate different response types
      final random = DateTime.now().millisecondsSinceEpoch % 3;
      String response;
      if (random == 0) {
        response = '✅ Success: Data loaded (200)';
      } else if (random == 1) {
        response = '⚠️ Not Found (404)';
      } else {
        response = '❌ Server Error (500)';
      }
      
      setState(() {
        _requests.insert(0, '${DateTime.now().toLocal()}: $response');
      });
    } catch (e) {
      setState(() {
        _requests.insert(0, '${DateTime.now().toLocal()}: Error: $e');
      });
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }
}
```

What's happening here?
- Network requests visible
- Response times and statuses
- Error tracking
- Request/response inspection

---

# Best Practices

## Use DevTools Regularly

```dart
// Good - Regular profiling
// 1. Check widget tree
// 2. Monitor performance
// 3. Inspect memory
// 4. Debug layout issues
```

## Profile in Profile Mode

```dart
// Good - Profile mode for accurate performance data
flutter run --profile

// Bad - Debug mode gives inaccurate performance data
flutter run --debug
```

## Monitor Memory

```dart
// Good - Check memory regularly
// 1. Monitor heap size
// 2. Check for memory leaks
// 3. Optimize image usage
```

---

# Common Mistakes

## Not Using DevTools

Wrong:
```dart
// Never using DevTools
// No performance monitoring
// No debugging of layout issues
```

Correct:
```dart
// Regular DevTools usage
// Performance monitoring
// Layout debugging
// Memory profiling
```

## Using Debug Mode for Performance

Wrong:
```dart
// Profiling in debug mode
flutter run --debug

// Inaccurate performance data
```

Correct:
```dart
// Profiling in profile mode
flutter run --profile

// Accurate performance data
```

---

# Summary

DevTools is essential for debugging, profiling, and optimizing Flutter applications. Use Widget Inspector for layout debugging, Performance Profiler for frame timings, Memory Profiler for memory usage, and Network Profiler for network requests. Regular use of DevTools leads to better performing applications.

---

# Next Steps

- [Profiling](profiling.md)
- [Performance Best Practices](performance-best-practices.md)
- [Testing](testing.md)

---

# Did You Know?

- DevTools runs in a browser
- Performance overlay shows frame timings
- Widget Inspector shows widget tree
- Memory Profiler tracks heap usage
- Network Profiler inspects requests
- Timeline shows frame breakdown
- DevTools is essential for debugging
- Use profile mode for accurate results