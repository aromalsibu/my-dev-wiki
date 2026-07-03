# RepaintBoundary

Understand how to optimize painting performance using RepaintBoundary in Flutter.

---

# What is it?

RepaintBoundary is a widget that creates a new layer for its child, isolating it from the rest of the widget tree during painting. This prevents the child from being repainted when other parts of the UI change. RepaintBoundary is essential for optimizing performance in complex UIs with animations, videos, and other frequently updating content.

---

# Why does it exist?

RepaintBoundary exists to:

- Optimize painting performance
- Isolate frequently updating widgets
- Prevent unnecessary repaints
- Reduce GPU and CPU usage
- Improve frame rate
- Support complex animations
- Handle video and heavy graphics

---

# Understanding Repaint

> **How painting** works in Flutter.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Understanding repaint example
class RepaintUnderstanding extends StatefulWidget {
  const RepaintUnderstanding({super.key});

  @override
  State<RepaintUnderstanding> createState() => _RepaintUnderstandingState();
}

class _RepaintUnderstandingState extends State<RepaintUnderstanding> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('🔵 Parent rebuilt');
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Understanding Repaint'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. This widget repaints every time (bad)
            const Text(
              'Without RepaintBoundary - Repaints every time',
              style: TextStyle(color: Colors.red, fontSize: 12),
            ),
            _buildHeavyWidget(),
            
            const SizedBox(height: 24),
            
            // 2. This widget only repaints when needed (good)
            const Text(
              'With RepaintBoundary - Only repaints when needed',
              style: TextStyle(color: Colors.green, fontSize: 12),
            ),
            _buildRepaintBoundaryWidget(),
            
            const SizedBox(height: 16),
            
            // 3. Counter
            Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 24),
            ),
            
            const SizedBox(height: 16),
            
            // 4. Button triggers rebuild
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _counter++;
                });
              },
              child: const Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildHeavyWidget() {
    // This widget repaints every time
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.blue[100],
      child: const Text(
        'This widget repaints every frame',
        style: TextStyle(fontSize: 16),
      ),
    );
  }

  Widget _buildRepaintBoundaryWidget() {
    // This widget is isolated by RepaintBoundary
    return RepaintBoundary(
      child: Container(
        padding: const EdgeInsets.all(16),
        color: Colors.green[100],
        child: const Text(
          'This widget only repaints when changed',
          style: TextStyle(fontSize: 16),
        ),
      ),
    );
  }
}
```

What's happening here?
- Without RepaintBoundary: Everything repaints
- With RepaintBoundary: Only changed parts repaint
- RepaintBoundary improves performance
- Use for frequently updating widgets

---

# RepaintBoundary with Animations

> **Using RepaintBoundary** with animated widgets.

```dart
/// RepaintBoundary with animations
class RepaintBoundaryAnimation extends StatefulWidget {
  const RepaintBoundaryAnimation({super.key});

  @override
  State<RepaintBoundaryAnimation> createState() => _RepaintBoundaryAnimationState();
}

class _RepaintBoundaryAnimationState extends State<RepaintBoundaryAnimation>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..repeat(reverse: true);
    
    _animation = Tween<double>(begin: 0.5, end: 1.5).animate(_controller);
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
        title: const Text('RepaintBoundary Animation'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Without RepaintBoundary
            // Parent rebuilds on every frame
            const Text(
              'Without RepaintBoundary - Parent rebuilds every frame',
              style: TextStyle(color: Colors.red, fontSize: 12),
            ),
            _buildWithoutBoundary(),
            
            const SizedBox(height: 24),
            
            // 2. With RepaintBoundary
            // Only the animated part repaints
            const Text(
              'With RepaintBoundary - Only animated part repaints',
              style: TextStyle(color: Colors.green, fontSize: 12),
            ),
            _buildWithBoundary(),
          ],
        ),
      ),
    );
  }

  Widget _buildWithoutBoundary() {
    return AnimatedBuilder(
      animation: _animation,
      builder: (context, child) {
        return Container(
          width: 100 * _animation.value,
          height: 100 * _animation.value,
          color: Colors.blue,
          child: const Center(
            child: Text(
              'Animated',
              style: TextStyle(color: Colors.white),
            ),
          ),
        );
      },
    );
  }

  Widget _buildWithBoundary() {
    return RepaintBoundary(
      child: AnimatedBuilder(
        animation: _animation,
        builder: (context, child) {
          return Container(
            width: 100 * _animation.value,
            height: 100 * _animation.value,
            color: Colors.green,
            child: const Center(
              child: Text(
                'Isolated',
                style: TextStyle(color: Colors.white),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

What's happening here?
- Without boundary: Parent and child repaint
- With boundary: Only child repaints
- Better performance for animations
- Isolate animated parts

---

# RepaintBoundary with Lists

> **Using RepaintBoundary** in lists.

```dart
/// RepaintBoundary with lists
class RepaintBoundaryList extends StatefulWidget {
  const RepaintBoundaryList({super.key});

  @override
  State<RepaintBoundaryList> createState() => _RepaintBoundaryListState();
}

class _RepaintBoundaryListState extends State<RepaintBoundaryList> {
  final List<int> _items = List.generate(100, (index) => index);
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('RepaintBoundary Lists'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              setState(() {
                _counter++;
              });
            },
          ),
        ],
      ),
      body: Column(
        children: [
          // Counter display
          Container(
            padding: const EdgeInsets.all(8),
            color: Colors.blue[50],
            child: Text(
              'Rebuild count: $_counter',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          // List with RepaintBoundary items
          Expanded(
            child: ListView.builder(
              itemCount: _items.length,
              itemBuilder: (context, index) {
                // Use RepaintBoundary for each item
                return RepaintBoundary(
                  child: _buildListItem(index),
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildListItem(int index) {
    return Container(
      padding: const EdgeInsets.all(8),
      margin: const EdgeInsets.all(2),
      color: Colors.blue[100 * (index % 9 + 1)],
      child: Text('Item $index'),
    );
  }
}
```

What's happening here?
- Each list item is isolated
- Changes only affect changed items
- Better scrolling performance
- Efficient list updates

---

# RepaintBoundary with Video

> **Isolating video** widgets.

```dart
/// RepaintBoundary with video
class VideoWithRepaintBoundary extends StatefulWidget {
  const VideoWithRepaintBoundary({super.key});

  @override
  State<VideoWithRepaintBoundary> createState() => _VideoWithRepaintBoundaryState();
}

class _VideoWithRepaintBoundaryState extends State<VideoWithRepaintBoundary> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Video with RepaintBoundary'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              setState(() {
                _counter++;
              });
            },
          ),
        ],
      ),
      body: Column(
        children: [
          // Video player isolated
          RepaintBoundary(
            child: Container(
              height: 200,
              color: Colors.black,
              child: const Center(
                child: Icon(
                  Icons.play_circle_filled,
                  color: Colors.white,
                  size: 64,
                ),
              ),
            ),
          ),
          
          // Controls (not affected by video repaint)
          Container(
            padding: const EdgeInsets.all(8),
            color: Colors.grey[200],
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                IconButton(
                  icon: const Icon(Icons.play_arrow),
                  onPressed: () {},
                ),
                IconButton(
                  icon: const Icon(Icons.pause),
                  onPressed: () {},
                ),
                IconButton(
                  icon: const Icon(Icons.stop),
                  onPressed: () {},
                ),
              ],
            ),
          ),
          
          // Counter (to test repaint)
          Container(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 20),
            ),
          ),
          
          ElevatedButton(
            onPressed: () {
              setState(() {
                _counter++;
              });
            },
            child: const Text('Increment'),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Video player is isolated
- Controls don't trigger video repaint
- Counter changes don't affect video
- Better video performance

---

# Real-World Examples

> **Common patterns** with RepaintBoundary.

```dart
/// 1. Animated icon with RepaintBoundary
class AnimatedIconWithBoundary extends StatefulWidget {
  const AnimatedIconWithBoundary({super.key});

  @override
  State<AnimatedIconWithBoundary> createState() => _AnimatedIconWithBoundaryState();
}

class _AnimatedIconWithBoundaryState extends State<AnimatedIconWithBoundary>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

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
        title: const Text('Animated Icon'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Isolated animated icon
            RepaintBoundary(
              child: AnimatedBuilder(
                animation: _controller,
                builder: (context, child) {
                  return Transform.rotate(
                    angle: _controller.value * 2 * 3.14159,
                    child: const Icon(
                      Icons.refresh,
                      size: 80,
                      color: Colors.blue,
                    ),
                  );
                },
              ),
            ),
            const SizedBox(height: 16),
            const Text(
              'The icon is isolated with RepaintBoundary',
              style: TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}

/// 2. Complex dashboard with RepaintBoundary
class DashboardWithBoundary extends StatefulWidget {
  const DashboardWithBoundary({super.key});

  @override
  State<DashboardWithBoundary> createState() => _DashboardWithBoundaryState();
}

class _DashboardWithBoundaryState extends State<DashboardWithBoundary> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Dashboard'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Chart (isolated)
            RepaintBoundary(
              child: Container(
                height: 150,
                color: Colors.grey[200],
                child: const Center(
                  child: Text('Chart Widget (Isolated)'),
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 2. Metrics (isolated)
            RepaintBoundary(
              child: Container(
                padding: const EdgeInsets.all(16),
                color: Colors.blue[50],
                child: const Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: [
                    _MetricWidget(label: 'Users', value: '1,234'),
                    _MetricWidget(label: 'Revenue', value: '$12,345'),
                    _MetricWidget(label: 'Orders', value: '567'),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 3. Activity feed (isolated)
            RepaintBoundary(
              child: Container(
                height: 150,
                color: Colors.green[50],
                child: const Center(
                  child: Text('Activity Feed (Isolated)'),
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 4. Counter (not isolated)
            Text(
              'Counter: $_counter',
              style: const TextStyle(fontSize: 20),
            ),
            
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _counter++;
                });
              },
              child: const Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }
}

/// Metric widget
class _MetricWidget extends StatelessWidget {
  const _MetricWidget({
    required this.label,
    required this.value,
  });

  final String label;
  final String value;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          value,
          style: const TextStyle(
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
        Text(
          label,
          style: const TextStyle(color: Colors.grey),
        ),
      ],
    );
  }
}
```

What's happening here?
- Animated icon with RepaintBoundary
- Dashboard with isolated components
- Complex widgets isolated
- Performance optimization

---

# Best Practices

## Use for Frequent Updates

```dart
// Good - Isolate frequently updating widgets
RepaintBoundary(
  child: AnimatedWidget(...),
)

// Bad - Isolate rarely updating widgets
RepaintBoundary(
  child: const Text('Static text'),
)
```

## Use for Heavy Widgets

```dart
// Good - Isolate heavy widgets
RepaintBoundary(
  child: HeavyWidget(),
)

// Bad - No isolation for heavy widgets
HeavyWidget()
```

## Use for Lists

```dart
// Good - Isolate list items
ListView.builder(
  itemBuilder: (context, index) {
    return RepaintBoundary(
      child: ListItem(index: index),
    );
  },
)
```

---

# Common Mistakes

## Overusing RepaintBoundary

Wrong:
```dart
// Too many boundaries
RepaintBoundary(
  child: RepaintBoundary(
    child: RepaintBoundary(
      child: SimpleWidget(),
    ),
  ),
)
```

Correct:
```dart
// Only when needed
RepaintBoundary(
  child: HeavyWidget(),
)
```

## Not Using for Animations

Wrong:
```dart
// No boundary - repaints everything
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) {
    return ComplexWidget();
  },
)
```

Correct:
```dart
// With boundary - only animated part repaints
RepaintBoundary(
  child: AnimatedBuilder(
    animation: _controller,
    builder: (context, child) {
      return ComplexWidget();
    },
  ),
)
```

---

# Summary

RepaintBoundary isolates widgets to prevent unnecessary repaints. Use it for frequently updating widgets, animations, videos, and heavy widgets. RepaintBoundary improves performance by reducing painting overhead and optimizing frame rates.

---

# Next Steps

- [Image Optimization](image-optimization.md)
- [DevTools](devtools.md)
- [Profiling](profiling.md)

---

# Did You Know?

- RepaintBoundary creates a new layer
- It isolates painting from parent
- Improves animation performance
- Reduces GPU usage
- Useful for video and animations
- Prevents unnecessary repaints
- Use for frequently updating content
- Overuse can hurt performance