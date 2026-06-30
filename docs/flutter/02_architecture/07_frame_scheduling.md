# Frame Scheduling

Understand how Flutter schedules frames for rendering.

---

# What is it?

Frame scheduling is the mechanism by which Flutter determines when to render frames. It coordinates the rendering pipeline with the screen's refresh rate, manages frame requests, and ensures smooth animations and UI updates. The scheduler decides which frames to render and when, optimizing performance and battery life.

---

# Why does it exist?

Frame scheduling exists to:

- Synchronize rendering with screen refresh rate
- Optimize performance and battery usage
- Coordinate multiple animation sources
- Prevent unnecessary frame rendering
- Manage frame budget and jank
- Support smooth animations at 60/120fps

---

# The Frame Scheduler

> **The Frame Scheduler** manages when frames are rendered.

```dart
// Frame scheduling in Flutter
class FrameSchedulerExample extends StatefulWidget {
  const FrameSchedulerExample({super.key});

  @override
  State<FrameSchedulerExample> createState() => _FrameSchedulerExampleState();
}

class _FrameSchedulerExampleState extends State<FrameSchedulerExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    // 1. AnimationController schedules frames
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    )..repeat();
    
    // 2. Listen to frame callbacks
    WidgetsBinding.instance.addPersistentFrameCallback((timeStamp) {
      // Called every frame
      print('Frame scheduled at: $timeStamp');
    });
  }

  @override
  Widget build(BuildContext context) {
    // 3. Widget rebuilds trigger frame requests
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Transform.rotate(
          angle: _controller.value * 2 * 3.14,
          child: const Icon(Icons.refresh, size: 100),
        );
      },
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

What's happening here?
- AnimationController requests frames
- Frame callbacks run every frame
- Widget rebuilds schedule frames
- Vsync synchronizes with display
- Scheduler manages frame timing

---

# Frame Request Sources

> **Various sources** can trigger frame requests.

```dart
// 1. setState() triggers frames
class SetStateExample extends StatefulWidget {
  const SetStateExample({super.key});

  @override
  State<SetStateExample> createState() => _SetStateExampleState();
}

class _SetStateExampleState extends State<SetStateExample> {
  int _counter = 0;

  void _increment() {
    setState(() {
      _counter++;
      // setState schedules a frame
      // WidgetsBinding.instance.scheduleFrame() is called
    });
  }

  @override
  Widget build(BuildContext context) {
    return Text('Counter: $_counter');
  }
}

// 2. Animation controllers
class AnimationFrameExample extends StatefulWidget {
  const AnimationFrameExample({super.key});

  @override
  State<AnimationFrameExample> createState() => _AnimationFrameExampleState();
}

class _AnimationFrameExampleState extends State<AnimationFrameExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..forward();
    // AnimationController schedules frames automatically
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        // Called every frame
        return Container(
          width: _controller.value * 100,
          height: 50,
          color: Colors.blue,
        );
      },
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}

// 3. Manual frame scheduling
class ManualFrameExample extends StatelessWidget {
  const ManualFrameExample({super.key});

  void _scheduleFrame() {
    // Manually schedule a frame
    WidgetsBinding.instance.scheduleFrame();
  }

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: _scheduleFrame,
      child: const Text('Schedule Frame'),
    );
  }
}

// 4. Ticker-based scheduling
class TickerExample extends StatefulWidget {
  const TickerExample({super.key});

  @override
  State<TickerExample> createState() => _TickerExampleState();
}

class _TickerExampleState extends State<TickerExample>
    with SingleTickerProviderStateMixin {
  late Ticker _ticker;

  @override
  void initState() {
    super.initState();
    _ticker = createTicker((elapsed) {
      // Called every frame
      print('Ticker tick: $elapsed');
    });
    _ticker.start();
  }

  @override
  void dispose() {
    _ticker.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return const Text('Ticker Example');
  }
}
```

What's happening here?
- setState schedules frames for UI updates
- Animation controllers drive frame requests
- Manual scheduling for custom needs
- Tickers provide frame callbacks
- Multiple sources coordinated together

---

# Vsync and Frame Timing

> **Vsync synchronizes** rendering with the display.

```dart
// Vsync integration
class VsyncExample extends StatefulWidget {
  const VsyncExample({super.key});

  @override
  State<VsyncExample> createState() => _VsyncExampleState();
}

class _VsyncExampleState extends State<VsyncExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    // Vsync is provided by TickerProvider
    _controller = AnimationController(
      vsync: this, // Provides vsync signals
      duration: const Duration(seconds: 1),
    )..repeat();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        // Synced with vsync
        // Called at screen refresh rate
        return Container(
          width: _controller.value * 200,
          height: 50,
          color: Colors.blue,
        );
      },
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}

// Frame timing:
// 1. Vsync signal arrives
// 2. Frame begins
// 3. Build phase runs
// 4. Layout phase runs
// 5. Paint phase runs
// 6. Frame ends
// 7. Wait for next vsync

// Frame budget:
// 60fps = 16.67ms per frame
// 120fps = 8.33ms per frame
// All work must fit in budget
// Or frame drops occur
```

What's happening here?
- Vsync synchronizes with display
- Ticker provides vsync signals
- Frame starts at vsync
- Work must finish before next vsync
- Frame drops if budget exceeded

---

# Frame Callbacks

> **Frame callbacks** run at different times.

```dart
// Types of frame callbacks
class FrameCallbacksExample extends StatefulWidget {
  const FrameCallbacksExample({super.key});

  @override
  State<FrameCallbacksExample> createState() => _FrameCallbacksExampleState();
}

class _FrameCallbacksExampleState extends State<FrameCallbacksExample> {
  @override
  void initState() {
    super.initState();

    // 1. Persistent frame callbacks
    // Run every frame until removed
    WidgetsBinding.instance.addPersistentFrameCallback((timeStamp) {
      print('Persistent frame: $timeStamp');
    });

    // 2. Post-frame callbacks
    // Run once after the frame
    WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
      print('Post frame: $timeStamp');
    });

    // 3. Frame timing callbacks
    // Run when frame timing is available
    WidgetsBinding.instance.addTimingsCallback((timings) {
      for (final timing in timings) {
        print('Frame timing: ${timing.totalSpan}');
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return const Text('Frame Callbacks');
  }
}

// Timing of callbacks:
// 1. Persistent callbacks (before build)
// 2. Build phase
// 3. Layout phase
// 4. Paint phase
// 5. Post-frame callbacks
// 6. Frame complete

// Use cases:
// 1. Persistent: Animations, game loops
// 2. Post-frame: Size/position queries
// 3. Timing: Performance monitoring
```

What's happening here?
- Persistent callbacks run every frame
- Post-frame callbacks run once
- Timing callbacks monitor performance
- Callbacks have specific timing
- Choose appropriate callback type

---

# Frame Budget Management

> **Managing frame budget** prevents jank.

```dart
// Monitoring frame budget
class FrameBudgetMonitor extends StatelessWidget {
  const FrameBudgetMonitor({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Check frame timing
    WidgetsBinding.instance.addTimingsCallback((timings) {
      for (final timing in timings) {
        final buildTime = timing.buildDuration;
        final layoutTime = timing.layoutDuration;
        final paintTime = timing.paintDuration;
        final totalTime = timing.totalSpan;
        
        // 2. Check if frame took too long
        if (totalTime.inMilliseconds > 16) {
          print('Frame took ${totalTime.inMilliseconds}ms');
          print('Build: ${buildTime.inMilliseconds}ms');
          print('Layout: ${layoutTime.inMilliseconds}ms');
          print('Paint: ${paintTime.inMilliseconds}ms');
          
          // 3. Log for analysis
          // Report to performance monitoring
        }
      }
    });

    return const Text('Performance Monitor');
  }
}

// Optimizing frame budget:
// 1. Reduce widget rebuilds
// 2. Use const widgets
// 3. Split large widgets
// 4. Use RepaintBoundary
// 5. Optimize animations
// 6. Use efficient layouts

// Frame budget breakdown:
// Build: 2-5ms
// Layout: 1-3ms
// Paint: 2-4ms
// Composite: 1-2ms
// Total: <16.6ms for 60fps
```

What's happening here?
- Monitor frame timing
- Identify slow frames
- Optimize heavy operations
- Keep within budget
- Prevent jank

---

# Animation and Frames

> **Animations** drive continuous frame requests.

```dart
// Animation-driven frames
class AnimationFramesExample extends StatefulWidget {
  const AnimationFramesExample({super.key});

  @override
  State<AnimationFramesExample> createState() => _AnimationFramesExampleState();
}

class _AnimationFramesExampleState extends State<AnimationFramesExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    );
    
    _animation = Tween<double>(begin: 0, end: 1).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ),
    );
    
    // Animation starts frame requests
    _controller.repeat();
  }

  @override
  Widget build(BuildContext context) {
    // Called every frame during animation
    return AnimatedBuilder(
      animation: _animation,
      builder: (context, child) {
        return Transform.scale(
          scale: _animation.value,
          child: Container(
            width: 100,
            height: 100,
            color: Colors.blue,
          ),
        );
      },
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}

// Frame scheduling in animations:
// 1. Animation starts
// 2. Frame requests begin
// 3. Each frame updates animation
// 4. Widget rebuilds with new value
// 5. Frame is rendered
// 6. Repeat until animation ends

// Stopping frame requests:
// 1. Animation completes
// 2. Controller stops
// 3. No more frame requests
// 4. Battery is saved
```

What's happening here?
- Animations drive frames continuously
- Each frame updates animation value
- Widget rebuilds on each frame
- Frames stop when animation ends
- Efficient frame usage

---

# Frame Scheduling in Different Modes

> **Frame scheduling** behaves differently in various modes.

```dart
// Debug mode scheduling
class DebugScheduling extends StatelessWidget {
  const DebugScheduling({super.key});

  @override
  Widget build(BuildContext context) {
    // In debug mode:
    // 1. More frames scheduled
    // 2. Hot reload triggers frames
    // 3. Extra debug checks
    // 4. Slightly slower
    
    return const Text('Debug mode scheduling');
  }
}

// Profile mode scheduling
class ProfileScheduling extends StatelessWidget {
  const ProfileScheduling({super.key});

  @override
  Widget build(BuildContext context) {
    // In profile mode:
    // 1. Normal frame scheduling
    // 2. Performance tracking
    // 3. Timing callbacks enabled
    // 4. Close to release mode
    
    return const Text('Profile mode scheduling');
  }
}

// Release mode scheduling
class ReleaseScheduling extends StatelessWidget {
  const ReleaseScheduling({super.key});

  @override
  Widget build(BuildContext context) {
    // In release mode:
    // 1. Optimized scheduling
    // 2. Fewer frame requests
    // 3. No debug checks
    // 4. Fastest performance
    
    return const Text('Release mode scheduling');
  }
}
```

What's happening here?
- Debug has extra frame requests
- Profile tracks frame timing
- Release has optimal scheduling
- Mode affects performance
- Choose based on needs

---

# Best Practices

## Minimize Frame Requests

```dart
// Good - Batch updates
@override
Widget build(BuildContext context) {
  // Multiple setState calls batched
  return Column(
    children: [
      ElevatedButton(
        onPressed: () {
          setState(() {
            _counter++;
            _total++;
            _changed = true;
          });
          // Single frame scheduled
        },
        child: const Text('Update'),
      ),
    ],
  );
}

// Bad - Multiple frame requests
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      ElevatedButton(
        onPressed: () {
          setState(() { _counter++; }); // Frame 1
          setState(() { _total++; });   // Frame 2
          setState(() { _changed = true; }); // Frame 3
        },
        child: const Text('Update'),
      ),
    ],
  );
}
```

## Use Frame Callbacks Wisely

```dart
// Good - One-time measurement
@override
void initState() {
  super.initState();
  WidgetsBinding.instance.addPostFrameCallback((_) {
    // Measure sizes after layout
    final size = context.size;
    print('Widget size: $size');
  });
}

// Bad - Heavy work in every frame
@override
void initState() {
  super.initState();
  WidgetsBinding.instance.addPersistentFrameCallback((_) {
    // Heavy work every frame
    // Avoid doing this
    expensiveOperation();
  });
}
```

## Dispose Controllers

```dart
// Good - Dispose animation controllers
@override
void dispose() {
  _controller.dispose(); // Stops frame requests
  super.dispose();
}

// Bad - Not disposing
@override
void dispose() {
  // Missing controller dispose
  super.dispose();
}
```

---

# Common Mistakes

## Scheduling Too Many Frames

Wrong:
```dart
// Requesting frames too often
Future.delayed(Duration(milliseconds: 100), () {
  setState(() {});
}); // Too many frame requests
```

Correct:
```dart
// Batch updates or use animation
setState(() {
  // Multiple changes in one setState
});
```

## Not Disposing Resources

Wrong:
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  late Ticker _ticker;
  
  @override
  void initState() {
    super.initState();
    _ticker = createTicker((_) {
      // Missing dispose
    });
    _ticker.start();
  }
  // No dispose method
}
```

Correct:
```dart
class _MyWidgetState extends State<MyWidget> {
  late Ticker _ticker;
  
  @override
  void initState() {
    super.initState();
    _ticker = createTicker((_) {
      setState(() {});
    });
    _ticker.start();
  }
  
  @override
  void dispose() {
    _ticker.dispose(); // Stops frames
    super.dispose();
  }
}
```

---

# Summary

Frame scheduling coordinates rendering with the display refresh rate. Animations, setState, and manual requests drive frames. Understanding frame scheduling helps optimize performance and prevent jank. Use frame callbacks wisely and always dispose resources.

---

# Next Steps

- [Constraints](constraints.md)
- [Layout Process](layout-process.md)
- [Widget Lifecycle](../widgets/widget-lifecycle.md)

---

# Did You Know?

- Flutter schedules frames on vsync
- 60fps = 16.6ms frame budget
- 120fps = 8.3ms frame budget
- Frame drops cause jank
- Animation controllers drive frames
- Tickers provide frame callbacks
- Frames stop when not needed
- Scheduling optimizes battery life