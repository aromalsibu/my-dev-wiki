# Explicit Animations

Understand how to create complex, controlled animations using explicit animation widgets in Flutter.

---

# What is it?

Explicit Animations are a type of animation in Flutter where you have full control over the animation lifecycle. Unlike implicit animations that automatically animate property changes, explicit animations require you to manually control the animation using an AnimationController, Animation objects, and widgets like AnimatedBuilder. This gives you complete control over timing, direction, and behavior.

---

# Why does it exist?

Explicit Animations exist to:

- Provide full control over animation lifecycle
- Enable complex animation sequences
- Support advanced animation patterns
- Allow custom animations and effects
- Control animation playback (play, pause, stop)
- Create long-running or looped animations
- Implement custom animation widgets

---

# AnimationController Basics

> **Creating and using** AnimationController.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic AnimationController example
class BasicAnimationControllerExample extends StatefulWidget {
  const BasicAnimationControllerExample({super.key});

  @override
  State<BasicAnimationControllerExample> createState() => _BasicAnimationControllerExampleState();
}

class _BasicAnimationControllerExampleState extends State<BasicAnimationControllerExample>
    with SingleTickerProviderStateMixin {
  // 1. Create AnimationController
  // AnimationController manages the animation lifecycle
  // vsync: this ensures smooth performance
  late AnimationController _controller;
  
  // 2. Create Animation objects
  // Animation can be used to transform values
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    
    // Initialize the controller with duration and vsync
    _controller = AnimationController(
      // Duration of the animation
      duration: const Duration(seconds: 2),
      // vsync prevents off-screen animations from running
      vsync: this,
    );
    
    // Create an animation that goes from 0.0 to 1.0
    // This is called a "Tween" animation
    _animation = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(_controller)
      ..addListener(() {
        // Called when the animation value changes
        // We use setState to rebuild the widget
        setState(() {});
      })
      ..addStatusListener((status) {
        // Called when animation status changes
        print('Animation status: $status');
      });
    
    // Start the animation
    _controller.forward();
  }

  @override
  void dispose() {
    // Always dispose the controller to prevent memory leaks
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('AnimationController'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 3. Use the animation value
            // The animation value changes from 0.0 to 1.0 over time
            Container(
              width: _animation.value * 200,
              height: _animation.value * 200,
              color: Colors.blue,
              child: Center(
                child: Text(
                  '${(_animation.value * 100).round()}%',
                  style: const TextStyle(color: Colors.white),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 4. Control buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Play animation
                ElevatedButton(
                  onPressed: () {
                    // Start animation from beginning
                    _controller.forward(from: 0.0);
                  },
                  child: const Text('Play'),
                ),
                const SizedBox(width: 8),
                
                // Pause animation
                ElevatedButton(
                  onPressed: () {
                    // Pause the animation
                    _controller.stop();
                  },
                  child: const Text('Pause'),
                ),
                const SizedBox(width: 8),
                
                // Reverse animation
                ElevatedButton(
                  onPressed: () {
                    // Play animation in reverse
                    _controller.reverse();
                  },
                  child: const Text('Reverse'),
                ),
                const SizedBox(width: 8),
                
                // Reset animation
                ElevatedButton(
                  onPressed: () {
                    // Reset to beginning
                    _controller.value = 0.0;
                  },
                  child: const Text('Reset'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- AnimationController manages animation timing
- Tween defines value range (0.0 to 1.0)
- addListener triggers rebuilds on value change
- forward() starts animation
- reverse() plays backwards
- stop() pauses animation

---

# AnimatedBuilder

> **Using AnimatedBuilder** for efficient animations.

```dart
/// AnimatedBuilder example
class AnimatedBuilderExample extends StatefulWidget {
  const AnimatedBuilderExample({super.key});

  @override
  State<AnimatedBuilderExample> createState() => _AnimatedBuilderExampleState();
}

class _AnimatedBuilderExampleState extends State<AnimatedBuilderExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _sizeAnimation;
  late Animation<Color?> _colorAnimation;
  late Animation<double> _rotationAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    )..repeat(reverse: true); // Loop animation back and forth
    
    // 1. Size animation (0.5 to 1.5)
    _sizeAnimation = Tween<double>(
      begin: 0.5,
      end: 1.5,
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ),
    );
    
    // 2. Color animation (blue to red)
    _colorAnimation = ColorTween(
      begin: Colors.blue,
      end: Colors.red,
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ),
    );
    
    // 3. Rotation animation (0 to 2π)
    _rotationAnimation = Tween<double>(
      begin: 0.0,
      end: 2 * 3.14159, // Full rotation
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ),
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
        title: const Text('AnimatedBuilder'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. AnimatedBuilder rebuilds when animation changes
            // It provides the current animation value
            AnimatedBuilder(
              animation: _controller,
              builder: (context, child) {
                return Transform.scale(
                  // Use size animation value
                  scale: _sizeAnimation.value,
                  child: Transform.rotate(
                    // Use rotation animation value
                    angle: _rotationAnimation.value,
                    child: Container(
                      width: 100,
                      height: 100,
                      // Use color animation value
                      color: _colorAnimation.value,
                      child: const Center(
                        child: Text(
                          'Build',
                          style: TextStyle(color: Colors.white),
                        ),
                      ),
                    ),
                  ),
                );
              },
            ),
            
            const SizedBox(height: 24),
            
            // 2. AnimatedBuilder with child optimization
            // The child is built once and reused
            AnimatedBuilder(
              animation: _controller,
              child: Container(
                width: 80,
                height: 80,
                color: Colors.green,
                child: const Center(
                  child: Text(
                    'Child',
                    style: TextStyle(color: Colors.white),
                  ),
                ),
              ),
              builder: (context, child) {
                return Transform.scale(
                  scale: _sizeAnimation.value,
                  child: Transform.rotate(
                    angle: _rotationAnimation.value,
                    child: child,
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
- AnimatedBuilder rebuilds on animation change
- Multiple animations can be combined
- Child parameter optimizes performance
- CurvedAnimation adds easing

---

# Custom Tween Animations

> **Creating custom** tween animations.

```dart
/// Custom Tween animations
class CustomTweenExample extends StatefulWidget {
  const CustomTweenExample({super.key});

  @override
  State<CustomTweenExample> createState() => _CustomTweenExampleState();
}

class _CustomTweenExampleState extends State<CustomTweenExample>
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
    
    // 1. Custom Tween with curve
    _animation = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).chain(
      CurveTween(curve: Curves.easeInOut),
    ).animate(_controller);
    
    // 2. Multiple curves in sequence
    final sequentialAnimation = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).chain(
      CurveTween(curve: const Interval(0.0, 0.5, curve: Curves.easeIn)),
    ).chain(
      CurveTween(curve: const Interval(0.5, 1.0, curve: Curves.easeOut)),
    ).animate(_controller);
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
        title: const Text('Custom Tween'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Basic tween animation
            AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                return Container(
                  width: 100 + _animation.value * 100,
                  height: 100 + _animation.value * 100,
                  color: Colors.blue.withOpacity(_animation.value),
                  child: Center(
                    child: Text(
                      '${(_animation.value * 100).round()}%',
                      style: const TextStyle(color: Colors.white),
                    ),
                  ),
                );
              },
            ),
            
            const SizedBox(height: 24),
            
            // 3. Custom interpolation
            AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                // Custom interpolation: bounce effect
                final value = _animation.value;
                final bouncedValue = 1 - (value * (1 - value) * 4);
                
                return Transform.scale(
                  scale: 0.5 + bouncedValue * 0.5,
                  child: Container(
                    width: 100,
                    height: 100,
                    color: Colors.green,
                    child: const Center(
                      child: Text(
                        'Bounce',
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
- Tween defines value range
- Chain adds curves
- Interval controls timing
- Custom interpolation for effects

---

# AnimatedWidget

> **Using AnimatedWidget** for reusable animations.

```dart
/// AnimatedWidget example
class AnimatedWidgetExample extends StatefulWidget {
  const AnimatedWidgetExample({super.key});

  @override
  State<AnimatedWidgetExample> createState() => _AnimatedWidgetExampleState();
}

class _AnimatedWidgetExampleState extends State<AnimatedWidgetExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..repeat(reverse: true);
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
        title: const Text('AnimatedWidget'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Use custom AnimatedWidget
            const AnimatedLogo(
              duration: Duration(seconds: 2),
            ),
            
            const SizedBox(height: 24),
            
            // 2. Use another animated widget
            const PulsingContainer(
              duration: Duration(seconds: 2),
            ),
          ],
        ),
      ),
    );
  }
}

/// Custom AnimatedWidget - Logo animation
class AnimatedLogo extends AnimatedWidget {
  const AnimatedLogo({
    super.key,
    required Duration duration,
  }) : super(
          listenable: AnimationController(
            duration: duration,
            vsync: const _VsyncPlaceholder(),
          )..repeat(reverse: true),
        );

  Animation<double> get _animation => listenable as Animation<double>;

  @override
  Widget build(BuildContext context) {
    return Transform.scale(
      scale: 0.5 + _animation.value * 0.5,
      child: Container(
        width: 100,
        height: 100,
        color: Colors.blue,
        child: const Icon(
          Icons.flutter_dash,
          color: Colors.white,
          size: 50,
        ),
      ),
    );
  }
}

/// Custom AnimatedWidget - Pulsing container
class PulsingContainer extends AnimatedWidget {
  const PulsingContainer({
    super.key,
    required Duration duration,
  }) : super(
          listenable: AnimationController(
            duration: duration,
            vsync: const _VsyncPlaceholder(),
          )..repeat(reverse: true),
        );

  Animation<double> get _animation => listenable as Animation<double>;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 100 + _animation.value * 50,
      height: 100 + _animation.value * 50,
      decoration: BoxDecoration(
        color: Colors.red,
        borderRadius: BorderRadius.circular(8 + _animation.value * 20),
      ),
      child: const Center(
        child: Text(
          'Pulse',
          style: TextStyle(color: Colors.white),
        ),
      ),
    );
  }
}

/// Simple vsync placeholder (for demonstration only)
class _VsyncPlaceholder extends TickerProvider {
  @override
  Ticker createTicker(TickerCallback onTick) {
    return Ticker(onTick);
  }
}
```

What's happening here?
- AnimatedWidget creates reusable animations
- Animation is provided through listenable
- Build method uses animation value
- Easy to reuse across the app

---

# Real-World Examples

> **Common patterns** with explicit animations.

```dart
/// 1. Animated splash screen
class AnimatedSplashScreen extends StatefulWidget {
  const AnimatedSplashScreen({super.key});

  @override
  State<AnimatedSplashScreen> createState() => _AnimatedSplashScreenState();
}

class _AnimatedSplashScreenState extends State<AnimatedSplashScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  late Animation<double> _fadeAnimation;
  late Animation<double> _rotationAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    );
    
    // Scale animation: 0.5 to 1.5
    _scaleAnimation = Tween<double>(
      begin: 0.5,
      end: 1.5,
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ),
    );
    
    // Fade animation: 0.0 to 1.0
    _fadeAnimation = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ),
    );
    
    // Rotation animation: 0 to 2π
    _rotationAnimation = Tween<double>(
      begin: 0.0,
      end: 2 * 3.14159,
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ),
    );
    
    // Start the animation
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        color: Colors.blue,
        child: Center(
          child: AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return Transform.scale(
                scale: _scaleAnimation.value,
                child: Transform.rotate(
                  angle: _rotationAnimation.value,
                  child: Opacity(
                    opacity: _fadeAnimation.value,
                    child: Container(
                      width: 100,
                      height: 100,
                      decoration: const BoxDecoration(
                        color: Colors.white,
                        shape: BoxShape.circle,
                      ),
                      child: const Icon(
                        Icons.flutter_dash,
                        color: Colors.blue,
                        size: 50,
                      ),
                    ),
                  ),
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}

/// 2. Animated page transition
class PageTransitionWidget extends StatelessWidget {
  const PageTransitionWidget({
    super.key,
    required this.child,
    required this.animation,
  });

  final Widget child;
  final Animation<double> animation;

  @override
  Widget build(BuildContext context) {
    // Slide and fade transition
    return FadeTransition(
      opacity: animation,
      child: SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(1.0, 0.0),
          end: Offset.zero,
        ).animate(animation),
        child: child,
      ),
    );
  }
}

/// 3. Animated loading spinner
class AnimatedSpinner extends StatefulWidget {
  const AnimatedSpinner({super.key});

  @override
  State<AnimatedSpinner> createState() => _AnimatedSpinnerState();
}

class _AnimatedSpinnerState extends State<AnimatedSpinner>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 1),
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
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Transform.rotate(
          angle: _controller.value * 2 * 3.14159,
          child: Container(
            width: 50,
            height: 50,
            decoration: const BoxDecoration(
              color: Colors.blue,
              shape: BoxShape.circle,
            ),
            child: const Center(
              child: Icon(
                Icons.refresh,
                color: Colors.white,
              ),
            ),
          ),
        );
      },
    );
  }
}
```

What's happening here?
- Splash screen with scale, fade, rotation
- Page transition with slide and fade
- Loading spinner with rotation

---

# Best Practices

## Dispose Controllers

```dart
// Good - Dispose controller
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}

// Bad - Memory leak
@override
void dispose() {
  super.dispose();
}
```

## Use Curves for Easing

```dart
// Good - Use curves for natural motion
CurvedAnimation(
  parent: _controller,
  curve: Curves.easeInOut,
)

// Bad - Linear motion
// No curve = linear animation
```

## Use AnimatedBuilder Efficiently

```dart
// Good - Use child parameter
AnimatedBuilder(
  animation: _controller,
  child: ExpensiveWidget(),
  builder: (context, child) {
    return Transform.scale(
      scale: _animation.value,
      child: child,
    );
  },
)

// Bad - Rebuilding child each frame
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) {
    return Transform.scale(
      scale: _animation.value,
      child: ExpensiveWidget(), // Rebuilt every frame
    );
  },
)
```

---

# Common Mistakes

## Not Disposing Controller

Wrong:
```dart
// Missing dispose - memory leak
late AnimationController _controller;
```

Correct:
```dart
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}
```

## Not Using vsync

Wrong:
```dart
// No vsync - animation may run off-screen
AnimationController(
  duration: const Duration(seconds: 1),
)
```

Correct:
```dart
// With vsync - optimized performance
AnimationController(
  duration: const Duration(seconds: 1),
  vsync: this,
)
```

---

# Summary

Explicit animations provide full control over animation lifecycle. Use AnimationController for timing, Tween for value ranges, AnimatedBuilder for performance, and AnimatedWidget for reusable animations. Always dispose controllers and use curves for natural motion.

---

# Next Steps

- [AnimationController](animationcontroller.md)
- [Tween](tween.md)
- [Hero](hero.md)

---

# Did You Know?

- AnimationController manages animation timing
- Tween defines value range
- AnimatedBuilder rebuilds on animation change
- AnimatedWidget creates reusable animations
- Curves provide easing functions
- vsync optimizes performance
- Controllers must be disposed
- Animations can be reversed and looped