# AnimatedBuilder

Understand how to create custom animations using AnimatedBuilder in Flutter.

---

# What is it?

AnimatedBuilder is a widget that rebuilds itself when an animation changes. It's the most efficient way to create custom animations because it only rebuilds the parts of the widget tree that need to change. AnimatedBuilder is perfect for complex custom animations where you need full control over the animation rendering process.

---

# Why does it exist?

AnimatedBuilder exists to:

- Build custom animations efficiently
- Separate animation logic from UI
- Optimize performance by rebuilding only what's needed
- Support complex animation scenarios
- Enable custom animation widgets
- Work with any Listenable

---

# Basic AnimatedBuilder

> **Creating animations** with AnimatedBuilder.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic AnimatedBuilder example
class BasicAnimatedBuilderExample extends StatefulWidget {
  const BasicAnimatedBuilderExample({super.key});

  @override
  State<BasicAnimatedBuilderExample> createState() => _BasicAnimatedBuilderExampleState();
}

class _BasicAnimatedBuilderExampleState extends State<BasicAnimatedBuilderExample>
    with SingleTickerProviderStateMixin {
  // 1. AnimationController to drive the animation
  late AnimationController _controller;
  
  // 2. Animation to provide values
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..repeat(reverse: true);
    
    _animation = Tween<double>(begin: 0.0, end: 1.0).animate(_controller);
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
        title: const Text('AnimatedBuilder'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 3. AnimatedBuilder listens to animation changes
            // It rebuilds only when the animation value changes
            AnimatedBuilder(
              // The listenable that triggers rebuilds
              animation: _animation,
              
              // 4. The builder is called whenever the animation changes
              builder: (context, child) {
                // Use the animation value to transform widgets
                return Transform.scale(
                  // Scale from 0.5 to 1.5 based on animation
                  scale: 0.5 + _animation.value,
                  child: Container(
                    width: 100,
                    height: 100,
                    color: Colors.blue,
                    child: const Center(
                      child: Text(
                        'Scale',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                );
              },
            ),
            
            const SizedBox(height: 24),
            
            // 5. Another AnimatedBuilder for a different effect
            AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                // Rotate based on animation value
                return Transform.rotate(
                  angle: _animation.value * 2 * 3.14159,
                  child: Container(
                    width: 80,
                    height: 80,
                    color: Colors.green,
                    child: const Center(
                      child: Text(
                        'Rotate',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- AnimatedBuilder listens to animation changes
- builder is called on every animation frame
- Only the AnimatedBuilder rebuilds, not the whole widget

---

# AnimatedBuilder with Child Optimization

> **Using child parameter** for performance optimization.

```dart
/// AnimatedBuilder with child optimization
class AnimatedBuilderChildExample extends StatefulWidget {
  const AnimatedBuilderChildExample({super.key});

  @override
  State<AnimatedBuilderChildExample> createState() => _AnimatedBuilderChildExampleState();
}

class _AnimatedBuilderChildExampleState extends State<AnimatedBuilderChildExample>
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
    
    _animation = Tween<double>(begin: 0.0, end: 1.0).animate(_controller);
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
        title: const Text('AnimatedBuilder Child'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Without child optimization - entire widget rebuilds
            // The Text widget is rebuilt every frame
            const Text(
              'Without Child Optimization (Rebuilds every frame)',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
            const SizedBox(height: 8),
            AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                return Transform.scale(
                  scale: 0.5 + _animation.value,
                  child: Container(
                    width: 100,
                    height: 100,
                    color: Colors.blue,
                    child: const Center(
                      child: Text(
                        'Rebuilt',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                );
              },
            ),
            
            const SizedBox(height: 24),
            
            // 2. With child optimization - child is built once and reused
            // The child widget is only built once and cached
            const Text(
              'With Child Optimization (Child is cached)',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
            const SizedBox(height: 8),
            AnimatedBuilder(
              animation: _animation,
              // 3. The child parameter is built once
              // It is then reused in the builder
              child: Container(
                width: 100,
                height: 100,
                color: Colors.green,
                child: const Center(
                  child: Text(
                    'Cached',
                    style: TextStyle(color: Colors.white),
                  ),
                ),
              ),
              // 4. The builder receives the cached child
              builder: (context, child) {
                // Only the transform is applied each frame
                // The child widget is reused
                return Transform.scale(
                  scale: 0.5 + _animation.value,
                  child: child,
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- child parameter caches the static widget
- Only the transform is rebuilt each frame
- Significantly better performance
- Best practice for complex animations

---

# AnimatedBuilder with Multiple Animations

> **Combining multiple** animations in one builder.

```dart
/// Multiple animations with AnimatedBuilder
class MultipleAnimationsExample extends StatefulWidget {
  const MultipleAnimationsExample({super.key});

  @override
  State<MultipleAnimationsExample> createState() => _MultipleAnimationsExampleState();
}

class _MultipleAnimationsExampleState extends State<MultipleAnimationsExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  
  // 1. Multiple animations with different tweens
  late Animation<double> _scaleAnimation;
  late Animation<double> _rotationAnimation;
  late Animation<double> _opacityAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    )..repeat();
    
    // 2. Scale animation (0.5 to 1.5)
    _scaleAnimation = Tween<double>(begin: 0.5, end: 1.5).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
    
    // 3. Rotation animation (0 to 2π)
    _rotationAnimation = Tween<double>(begin: 0.0, end: 2 * 3.14159).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
    
    // 4. Opacity animation (0.0 to 1.0)
    _opacityAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
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
        title: const Text('Multiple Animations'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 5. AnimatedBuilder with multiple animations
            // It listens to the controller which drives all animations
            AnimatedBuilder(
              animation: _controller,
              builder: (context, child) {
                return Opacity(
                  // Opacity animation
                  opacity: _opacityAnimation.value,
                  child: Transform.scale(
                    // Scale animation
                    scale: _scaleAnimation.value,
                    child: Transform.rotate(
                      // Rotation animation
                      angle: _rotationAnimation.value,
                      child: Container(
                        width: 100,
                        height: 100,
                        color: Colors.blue,
                        child: const Center(
                          child: Text(
                            'Multi',
                            style: TextStyle(color: Colors.white),
                          ),
                        ),
                      ),
                    ),
                  ),
                );
              },
            ),
            
            const SizedBox(height: 24),
            
            // 6. Animation values display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  Text('Scale: ${_scaleAnimation.value.toStringAsFixed(2)}'),
                  Text('Rotation: ${(_rotationAnimation.value * 180 / 3.14159).toStringAsFixed(0)}°'),
                  Text('Opacity: ${_opacityAnimation.value.toStringAsFixed(2)}'),
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
- One AnimatedBuilder listens to controller
- Multiple animations use the same controller
- Combined effects in one builder
- Efficient and organized

---

# Real-World Examples

> **Common patterns** with AnimatedBuilder.

```dart
/// 1. Animated progress indicator with AnimatedBuilder
class AnimatedProgressIndicator extends StatefulWidget {
  const AnimatedProgressIndicator({
    super.key,
    required this.progress,
    this.duration = const Duration(milliseconds: 500),
  });

  final double progress;
  final Duration duration;

  @override
  State<AnimatedProgressIndicator> createState() => _AnimatedProgressIndicatorState();
}

class _AnimatedProgressIndicatorState extends State<AnimatedProgressIndicator>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: widget.duration,
      vsync: this,
    );
    
    _animation = Tween<double>(begin: 0.0, end: widget.progress).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
    
    _controller.forward();
  }

  @override
  void didUpdateWidget(covariant AnimatedProgressIndicator oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.progress != oldWidget.progress) {
      _animation = Tween<double>(begin: oldWidget.progress, end: widget.progress).animate(
        CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
      );
      _controller.forward(from: 0.0);
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _animation,
      builder: (context, child) {
        return Container(
          height: 20,
          decoration: BoxDecoration(
            color: Colors.grey[200],
            borderRadius: BorderRadius.circular(10),
          ),
          child: Stack(
            children: [
              // Progress bar
              Container(
                width: _animation.value * 300,
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    colors: [Colors.blue, Colors.purple],
                  ),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: Center(
                  child: Text(
                    '${(_animation.value * 100).round()}%',
                    style: const TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                      fontSize: 12,
                    ),
                  ),
                ),
              ),
            ],
          ),
        );
      },
    );
  }
}

/// 2. Animated card with expand/collapse
class ExpandableCard extends StatefulWidget {
  const ExpandableCard({
    super.key,
    required this.title,
    required this.content,
  });

  final String title;
  final Widget content;

  @override
  State<ExpandableCard> createState() => _ExpandableCardState();
}

class _ExpandableCardState extends State<ExpandableCard>
    with SingleTickerProviderStateMixin {
  bool _isExpanded = false;
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(milliseconds: 300),
      vsync: this,
    );
    
    _animation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _toggleExpand() {
    setState(() {
      _isExpanded = !_isExpanded;
    });
    
    if (_isExpanded) {
      _controller.forward();
    } else {
      _controller.reverse();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: Column(
        children: [
          // Header
          ListTile(
            title: Text(
              widget.title,
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
            trailing: AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                return Transform.rotate(
                  angle: _animation.value * 3.14159,
                  child: const Icon(Icons.expand_more),
                );
              },
            ),
            onTap: _toggleExpand,
          ),
          
          // Content
          AnimatedBuilder(
            animation: _animation,
            builder: (context, child) {
              return Container(
                height: _animation.value * 200,
                child: _animation.value > 0
                    ? Padding(
                        padding: const EdgeInsets.all(16),
                        child: widget.content,
                      )
                    : null,
              );
            },
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- AnimatedProgressIndicator with smooth progress
- ExpandableCard with AnimatedBuilder
- Reusable animated components
- Real-world animation patterns

---

# Best Practices

## Use Child Parameter

```dart
// Good - Optimized with child
AnimatedBuilder(
  animation: _animation,
  child: ExpensiveWidget(),
  builder: (context, child) {
    return Transform.scale(
      scale: _animation.value,
      child: child,
    );
  },
)
```

## Separate Logic from UI

```dart
// Good - Clean separation
class MyAnimatedWidget extends StatelessWidget {
  final Animation<double> animation;
  final Widget child;
  
  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: animation,
      child: child,
      builder: (context, child) {
        return Transform.scale(
          scale: animation.value,
          child: child,
        );
      },
    );
  }
}
```

## Use const Where Possible

```dart
// Good - Const widgets
const Text('Static Text')
```

---

# Common Mistakes

## Not Using Child Parameter

Wrong:
```dart
// Rebuilds child every frame
AnimatedBuilder(
  animation: _animation,
  builder: (context, child) {
    return Transform.scale(
      scale: _animation.value,
      child: Container(color: Colors.blue),
    );
  },
)
```

Correct:
```dart
// Caches child
AnimatedBuilder(
  animation: _animation,
  child: Container(color: Colors.blue),
  builder: (context, child) {
    return Transform.scale(
      scale: _animation.value,
      child: child,
    );
  },
)
```

---

# Summary

AnimatedBuilder provides efficient custom animations by rebuilding only when animation changes. Use the child parameter for performance optimization, combine multiple animations in one builder, and create reusable animated widgets. AnimatedBuilder is perfect for complex custom animations.

---

# Next Steps

- [Async Patterns](async-patterns.md)
- [Provider](../state-management/provider.md)
- [Riverpod](../state-management/riverpod.md)

---

# Did You Know?

- AnimatedBuilder only rebuilds when animation changes
- child parameter caches widgets for performance
- Multiple animations can be combined
- AnimatedBuilder works with any Listenable
- AnimatedBuilder is more efficient than setState
- Custom animated widgets can be created
- AnimatedBuilder separates logic from UI
- AnimatedBuilder is preferred for complex animations