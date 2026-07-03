# Profiling

Understand how to profile Flutter applications to identify and fix performance issues.

---

# What is it?

Profiling is the process of measuring and analyzing your application's performance to identify bottlenecks, memory leaks, and optimization opportunities. It involves collecting data about frame timings, CPU usage, memory allocation, and other performance metrics to understand how your app behaves under real-world conditions.

---

# Why does it exist?

Profiling exists to:

- Identify performance bottlenecks
- Measure frame rendering times
- Detect memory leaks
- Analyze CPU usage
- Optimize resource usage
- Improve user experience
- Prevent jank and lag

---

# Profile Mode

> **Running** your app in profile mode.

```bash
# 1. Run in profile mode (Android/iOS)
flutter run --profile

# 2. Build for profile mode
flutter build apk --profile
flutter build ios --profile

# 3. Run with specific device
flutter run --profile -d emulator-5554

# 4. Run with performance overlay
flutter run --profile --show-performance-overlay
```

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

/// Profile mode example
class ProfilingExample extends StatefulWidget {
  const ProfilingExample({super.key});

  @override
  State<ProfilingExample> createState() => _ProfilingExampleState();
}

class _ProfilingExampleState extends State<ProfilingExample>
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
        title: const Text('Profiling Example'),
        // 1. Performance overlay toggle
        actions: [
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
            // 2. Animated widget for profiling
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
                  onPressed: _performHeavyOperation,
                  child: const Text('Heavy Operation'),
                ),
              ],
            ),
            
            const SizedBox(height: 16),
            
            // 5. Performance tips
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Column(
                children: [
                  Text(
                    'Profiling Tips:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('• Run in profile mode for accurate data'),
                  Text('• Monitor frame timings in DevTools'),
                  Text('• Check for jank and frame drops'),
                  Text('• Profile on real devices when possible'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  void _performHeavyOperation() {
    // Simulate heavy operation (causes frame drops)
    int result = 0;
    for (int i = 0; i < 1000000; i++) {
      result += i;
    }
    print('Heavy operation result: $result');
  }
}
```

What's happening here?
- Profile mode gives accurate performance data
- Performance overlay shows frame timings
- Animation performance visible
- Heavy operations cause frame drops

---

# Timeline Profiling

> **Analyzing** frame timings.

```dart
/// Timeline profiling example
class TimelineProfilingExample extends StatefulWidget {
  const TimelineProfilingExample({super.key});

  @override
  State<TimelineProfilingExample> createState() => _TimelineProfilingExampleState();
}

class _TimelineProfilingExampleState extends State<TimelineProfilingExample> {
  // 1. Track frame timings
  List<double> _frameTimings = [];
  double _averageFrameTime = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Timeline Profiling'),
        actions: [
          // 2. Clear timings
          IconButton(
            icon: const Icon(Icons.clear),
            onPressed: () {
              setState(() {
                _frameTimings.clear();
                _averageFrameTime = 0;
              });
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 3. Frame timing display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Frame Performance',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceAround,
                    children: [
                      Column(
                        children: [
                          const Text('Average Frame Time:'),
                          Text(
                            '${_averageFrameTime.toStringAsFixed(2)}ms',
                            style: const TextStyle(
                              fontSize: 20,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                      Column(
                        children: [
                          const Text('FPS:'),
                          Text(
                            _averageFrameTime > 0
                                ? '${(1000 / _averageFrameTime).toStringAsFixed(1)}'
                                : '0',
                            style: const TextStyle(
                              fontSize: 20,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 4. Timeline visualization (simulated)
            _buildTimelineVisualization(),
            
            const SizedBox(height: 16),
            
            // 5. Action buttons
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _simulateGoodFrame,
                    child: const Text('Good Frame'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _simulateBadFrame,
                    child: const Text('Bad Frame'),
                  ),
                ),
              ],
            ),
            
            const SizedBox(height: 16),
            
            // 6. Tips
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
                    'Timeline Tips:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('• Aim for 60fps (16.67ms per frame)'),
                  Text('• Frame times > 16.67ms cause jank'),
                  Text('• Check build, layout, and paint times'),
                  Text('• Identify slow operations in timeline'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildTimelineVisualization() {
    return Container(
      height: 100,
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(8),
      ),
      child: CustomPaint(
        painter: TimelinePainter(
          frameTimings: _frameTimings,
        ),
      ),
    );
  }

  void _simulateGoodFrame() {
    setState(() {
      // Good frame: ~8ms
      final frameTime = 8.0 + (DateTime.now().millisecondsSinceEpoch % 3).toDouble();
      _frameTimings.add(frameTime);
      if (_frameTimings.length > 20) {
        _frameTimings.removeAt(0);
      }
      _calculateAverage();
    });
  }

  void _simulateBadFrame() {
    setState(() {
      // Bad frame: ~25ms (exceeds 16.67ms target)
      final frameTime = 25.0 + (DateTime.now().millisecondsSinceEpoch % 5).toDouble();
      _frameTimings.add(frameTime);
      if (_frameTimings.length > 20) {
        _frameTimings.removeAt(0);
      }
      _calculateAverage();
    });
  }

  void _calculateAverage() {
    if (_frameTimings.isEmpty) {
      _averageFrameTime = 0;
      return;
    }
    final sum = _frameTimings.reduce((a, b) => a + b);
    _averageFrameTime = sum / _frameTimings.length;
  }
}

/// Timeline painter
class TimelinePainter extends CustomPainter {
  TimelinePainter({required this.frameTimings});

  final List<double> frameTimings;

  @override
  void paint(Canvas canvas, Size size) {
    if (frameTimings.isEmpty) return;

    final paint = Paint()
      ..style = PaintingStyle.fill;

    final padding = 20.0;
    final chartWidth = size.width - padding * 2;
    final chartHeight = size.height - padding * 2;
    final maxTime = 30.0; // Max frame time in ms

    // Draw background grid
    final gridPaint = Paint()
      ..color = Colors.grey[300]!
      ..style = PaintingStyle.stroke
      ..strokeWidth = 1;

    for (int i = 0; i < 5; i++) {
      final y = padding + (i / 4) * chartHeight;
      canvas.drawLine(
        Offset(padding, y),
        Offset(size.width - padding, y),
        gridPaint,
      );
    }

    // Draw frame bars
    for (int i = 0; i < frameTimings.length; i++) {
      final x = padding + (i / frameTimings.length) * chartWidth;
      final barHeight = (frameTimings[i] / maxTime) * chartHeight;
      
      // Color based on frame time
      if (frameTimings[i] > 16.67) {
        paint.color = Colors.red; // Bad frame
      } else if (frameTimings[i] > 12) {
        paint.color = Colors.orange; // Warning
      } else {
        paint.color = Colors.green; // Good frame
      }

      canvas.drawRect(
        Rect.fromLTWH(
          x - 2,
          size.height - padding - barHeight,
          4,
          barHeight,
        ),
        paint,
      );
    }

    // Draw target line (16.67ms for 60fps)
    final targetPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 1
      ..strokeCap = StrokeCap.round;

    final targetY = size.height - padding - (16.67 / maxTime) * chartHeight;
    canvas.drawLine(
      Offset(padding, targetY),
      Offset(size.width - padding, targetY),
      targetPaint,
    );
  }

  @override
  bool shouldRepaint(TimelinePainter oldDelegate) {
    return oldDelegate.frameTimings != frameTimings;
  }
}
```

What's happening here?
- Frame timing visualization
- Good vs bad frames
- Target of 16.67ms (60fps)
- Color-coded performance

---

# CPU Profiling

> **Analyzing** CPU usage.

```dart
/// CPU profiling example
class CpuProfilingExample extends StatefulWidget {
  const CpuProfilingExample({super.key});

  @override
  State<CpuProfilingExample> createState() => _CpuProfilingExampleState();
}

class _CpuProfilingExampleState extends State<CpuProfilingExample> {
  // 1. Track CPU usage
  bool _isRunning = false;
  double _cpuUsage = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('CPU Profiling'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. CPU usage display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'CPU Usage',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    '${_cpuUsage.toStringAsFixed(1)}%',
                    style: const TextStyle(
                      fontSize: 40,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  // CPU usage bar
                  Container(
                    width: double.infinity,
                    height: 20,
                    decoration: BoxDecoration(
                      color: Colors.grey[200],
                      borderRadius: BorderRadius.circular(10),
                    ),
                    child: Container(
                      width: (_cpuUsage / 100) * 300,
                      decoration: BoxDecoration(
                        color: _cpuUsage > 80 
                            ? Colors.red 
                            : _cpuUsage > 50 
                                ? Colors.orange 
                                : Colors.green,
                        borderRadius: BorderRadius.circular(10),
                      ),
                    ),
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 3. CPU operation buttons
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _runCpuIntensiveTask,
                    child: const Text('CPU Intensive'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _runMemoryIntensiveTask,
                    child: const Text('Memory Intensive'),
                  ),
                ),
              ],
            ),
            
            const SizedBox(height: 8),
            
            Expanded(
              child: _cpuUsage > 0
                  ? _buildCpuActivity()
                  : const Center(
                      child: Text(
                        'Run a task to see CPU activity',
                        style: TextStyle(color: Colors.grey),
                      ),
                    ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildCpuActivity() {
    return ListView.builder(
      itemCount: 10,
      itemBuilder: (context, index) {
        return ListTile(
          leading: CircleAvatar(
            child: Text('${index + 1}'),
          ),
          title: Text('CPU Task ${index + 1}'),
          subtitle: Text('Priority: ${index % 3 == 0 ? "High" : "Normal"}'),
          trailing: Container(
            width: 60,
            height: 4,
            decoration: BoxDecoration(
              color: Colors.grey[200],
              borderRadius: BorderRadius.circular(2),
            ),
            child: Container(
              width: (index + 1) * 6,
              decoration: BoxDecoration(
                color: Colors.blue,
                borderRadius: BorderRadius.circular(2),
              ),
            ),
          ),
        );
      },
    );
  }

  void _runCpuIntensiveTask() async {
    setState(() {
      _isRunning = true;
    });

    // Simulate CPU-intensive work
    await Future.delayed(const Duration(milliseconds: 100));
    int result = 0;
    for (int i = 0; i < 10000000; i++) {
      result += i;
      // Update CPU usage periodically
      if (i % 100000 == 0) {
        final usage = (i / 10000000) * 80 + 20;
        setState(() {
          _cpuUsage = usage;
        });
        await Future.delayed(Duration.zero);
      }
    }

    setState(() {
      _isRunning = false;
      _cpuUsage = 0;
    });
  }

  void _runMemoryIntensiveTask() {
    setState(() {
      _cpuUsage = 60;
    });

    // Simulate memory-intensive work
    List<int> data = [];
    for (int i = 0; i < 100000; i++) {
      data.add(i);
      if (i % 1000 == 0) {
        setState(() {
          _cpuUsage = 40 + (i / 100000) * 40;
        });
      }
    }
    data.clear();

    setState(() {
      _cpuUsage = 0;
    });
  }
}
```

What's happening here?
- CPU usage monitoring
- CPU-intensive tasks
- Memory-intensive tasks
- Activity visualization

---

# Real-World Examples

> **Common profiling** patterns.

```dart
/// 1. Performance monitor
class PerformanceMonitor extends StatefulWidget {
  const PerformanceMonitor({super.key, required this.child});

  final Widget child;

  @override
  State<PerformanceMonitor> createState() => _PerformanceMonitorState();
}

class _PerformanceMonitorState extends State<PerformanceMonitor> {
  // 1. Track performance metrics
  double _fps = 0;
  double _frameTime = 0;
  int _frameCount = 0;
  int _jankCount = 0;

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        widget.child,
        // 2. Performance overlay
        Positioned(
          top: 40,
          right: 8,
          child: Container(
            padding: const EdgeInsets.all(8),
            decoration: BoxDecoration(
              color: Colors.black.withOpacity(0.7),
              borderRadius: BorderRadius.circular(8),
            ),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.end,
              children: [
                Text(
                  'FPS: ${_fps.toStringAsFixed(1)}',
                  style: TextStyle(
                    color: _fps > 55 ? Colors.green : Colors.red,
                    fontSize: 12,
                  ),
                ),
                Text(
                  'Frame: ${_frameTime.toStringAsFixed(2)}ms',
                  style: TextStyle(
                    color: _frameTime < 16.67 ? Colors.green : Colors.red,
                    fontSize: 12,
                  ),
                ),
                Text(
                  'Jank: $_jankCount',
                  style: const TextStyle(
                    color: Colors.orange,
                    fontSize: 12,
                  ),
                ),
              ],
            ),
          ),
        ),
      ],
    );
  }
}

/// 2. Profiling helper
class ProfilingHelper {
  // 3. Measure execution time
  static Future<T> measureTime<T>(
    Future<T> Function() operation,
    String name,
  ) async {
    final stopwatch = Stopwatch()..start();
    final result = await operation();
    stopwatch.stop();
    print('⏱️ $name took ${stopwatch.elapsedMilliseconds}ms');
    return result;
  }

  // 4. Measure frame time
  static void measureFrame() {
    // Use SchedulerBinding to measure frame time
    // This would be implemented in a real app
  }

  // 5. Log performance data
  static void logPerformance() {
    final imageCache = PaintingBinding.instance.imageCache;
    print('📊 Performance Log:');
    print('  Image Cache: ${imageCache.currentSize} images');
    print('  Cache Size: ${imageCache.currentSizeBytes ~/ 1024} KB');
  }
}
```

What's happening here?
- Performance overlay
- FPS and frame time monitoring
- Jank detection
- Execution time measurement

---

# Best Practices

## Profile on Real Devices

```dart
// Good - Profile on real devices
flutter run --profile -d "device-name"

// Bad - Profile on emulator (inaccurate)
flutter run --profile -d emulator
```

## Use Profile Mode

```dart
// Good - Profile mode for accurate data
flutter run --profile

// Bad - Debug mode for performance testing
flutter run --debug
```

## Monitor Frame Timings

```dart
// Good - Check frame timings
// Use DevTools timeline
// Monitor for frames > 16.67ms
```

---

# Common Mistakes

## Profiling in Debug Mode

Wrong:
```dart
// Debug mode has performance overhead
flutter run --debug
// Inaccurate performance data
```

Correct:
```dart
// Profile mode for accurate data
flutter run --profile
```

## Ignoring Jank

Wrong:
```dart
// Not monitoring frame drops
// Not fixing performance issues
```

Correct:
```dart
// Monitor frame timings
// Fix jank issues
// Optimize slow operations
```

---

# Summary

Profiling helps identify and fix performance issues in Flutter applications. Use profile mode for accurate data, monitor frame timings in DevTools, analyze CPU and memory usage, and optimize slow operations. Regular profiling leads to smooth, high-performance apps.

---

# Next Steps

- [Performance Best Practices](performance-best-practices.md)
- [Testing](testing.md)
- [DevTools](devtools.md)

---

# Did You Know?

- Profile mode gives accurate performance data
- 60fps target is 16.67ms per frame
- Jank occurs when frames exceed budget
- DevTools timeline shows frame breakdown
- CPU profiling identifies bottlenecks
- Memory profiling detects leaks
- Profile on real devices for accuracy
- Regular profiling prevents performance issues