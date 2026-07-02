# Curves

Understand how to use curves to create natural and engaging animations in Flutter.

---

# What is it?

Curves define the rate of change of an animation over time. They control how an animation progresses from start to finish, allowing you to create natural, smooth, and engaging motion. Instead of moving at a constant speed (linear), curves can accelerate, decelerate, bounce, or follow various mathematical functions.

---

# Why does it exist?

Curves exist to:

- Create natural and realistic motion
- Add personality and character to animations
- Control animation acceleration and deceleration
- Implement complex easing functions
- Make UI feel responsive and polished
- Support various animation styles

---

# Basic Curve Usage

> **Using curves** with animations.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic curve usage example
class BasicCurveExample extends StatefulWidget {
  const BasicCurveExample({super.key});

  @override
  State<BasicCurveExample> createState() => _BasicCurveExampleState();
}

class _BasicCurveExampleState extends State<BasicCurveExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  
  // 1. Different curves for different animations
  late Animation<double> _linearAnimation;
  late Animation<double> _easeInAnimation;
  late Animation<double> _easeOutAnimation;
  late Animation<double> _easeInOutAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..repeat(reverse: true);
    
    // 2. Linear - Constant speed (no easing)
    _linearAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.linear,
      ),
    )..addListener(() { setState(() {}); });
    
    // 3. EaseIn - Starts slow, ends fast
    _easeInAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeIn,
      ),
    )..addListener(() { setState(() {}); });
    
    // 4. EaseOut - Starts fast, ends slow
    _easeOutAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeOut,
      ),
    )..addListener(() { setState(() {}); });
    
    // 5. EaseInOut - Slow, fast, slow
    _easeInOutAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOut,
      ),
    )..addListener(() { setState(() {}); });
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
        title: const Text('Animation Curves'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Linear
            _buildCurveDemo('Linear', _linearAnimation, Colors.grey),
            
            // 2. EaseIn
            _buildCurveDemo('EaseIn', _easeInAnimation, Colors.blue),
            
            // 3. EaseOut
            _buildCurveDemo('EaseOut', _easeOutAnimation, Colors.green),
            
            // 4. EaseInOut
            _buildCurveDemo('EaseInOut', _easeInOutAnimation, Colors.purple),
          ],
        ),
      ),
    );
  }

  Widget _buildCurveDemo(String label, Animation<double> animation, Color color) {
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              label,
              style: const TextStyle(
                fontWeight: FontWeight.bold,
                fontSize: 16,
              ),
            ),
            const SizedBox(height: 8),
            Container(
              width: double.infinity,
              height: 40,
              color: Colors.grey[200],
              child: AnimatedBuilder(
                animation: animation,
                builder: (context, child) {
                  return Container(
                    width: animation.value * 300,
                    height: 40,
                    color: color,
                    child: Center(
                      child: Text(
                        '${(animation.value * 100).round()}%',
                        style: const TextStyle(color: Colors.white),
                      ),
                    ),
                  );
                },
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
- CurvedAnimation applies curve to animation
- linear: Constant speed
- easeIn: Starts slow, ends fast
- easeOut: Starts fast, ends slow
- easeInOut: Slow, fast, slow

---

# Common Curves

> **Different curve** types and their effects.

```dart
/// Common curves comparison
class CommonCurvesExample extends StatefulWidget {
  const CommonCurvesExample({super.key});

  @override
  State<CommonCurvesExample> createState() => _CommonCurvesExampleState();
}

class _CommonCurvesExampleState extends State<CommonCurvesExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
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
        title: const Text('Common Curves'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Linear - No easing
            _buildCurveDemo(
              'Linear',
              Curves.linear,
              Colors.grey,
              'Constant speed',
            ),
            
            // 2. Ease - Smooth easing
            _buildCurveDemo(
              'Ease',
              Curves.ease,
              Colors.blue,
              'Smooth acceleration',
            ),
            
            // 3. BounceOut - Bounces at the end
            _buildCurveDemo(
              'BounceOut',
              Curves.bounceOut,
              Colors.orange,
              'Bounces at the end',
            ),
            
            // 4. ElasticOut - Elastic effect
            _buildCurveDemo(
              'ElasticOut',
              Curves.elasticOut,
              Colors.purple,
              'Elastic effect at end',
            ),
            
            // 5. Decelerate - Starts fast, ends slow
            _buildCurveDemo(
              'Decelerate',
              Curves.decelerate,
              Colors.green,
              'Starts fast, ends slow',
            ),
            
            // 6. Accelerate - Starts slow, ends fast
            _buildCurveDemo(
              'Accelerate',
              Curves.accelerate,
              Colors.red,
              'Starts slow, ends fast',
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildCurveDemo(
    String label,
    Curve curve,
    Color color,
    String description,
  ) {
    // Create animation with the specified curve
    final animation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: curve),
    )..addListener(() { setState(() {}); });
    
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text(
                  label,
                  style: const TextStyle(
                    fontWeight: FontWeight.bold,
                    fontSize: 16,
                  ),
                ),
                Text(
                  description,
                  style: TextStyle(
                    fontSize: 12,
                    color: Colors.grey[600],
                  ),
                ),
              ],
            ),
            const SizedBox(height: 8),
            Container(
              width: double.infinity,
              height: 50,
              color: Colors.grey[200],
              child: AnimatedBuilder(
                animation: animation,
                builder: (context, child) {
                  return Container(
                    width: animation.value * 300,
                    height: 50,
                    color: color,
                    child: Center(
                      child: Text(
                        '${(animation.value * 100).round()}%',
                        style: const TextStyle(color: Colors.white),
                      ),
                    ),
                  );
                },
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
- bounceOut: Bounces at the end
- elasticOut: Elastic effect
- decelerate: Fast to slow
- accelerate: Slow to fast
- ease: Smooth acceleration

---

# Custom Curves

> **Creating custom** curve functions.

```dart
/// Custom curve example
class CustomCurveExample extends StatefulWidget {
  const CustomCurveExample({super.key});

  @override
  State<CustomCurveExample> createState() => _CustomCurveExampleState();
}

class _CustomCurveExampleState extends State<CustomCurveExample>
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
        title: const Text('Custom Curves'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Custom bounce curve
            _buildCustomCurveDemo(
              'Custom Bounce',
              CustomBounceCurve(),
              Colors.blue,
            ),
            
            // 2. Custom elastic curve
            _buildCustomCurveDemo(
              'Custom Elastic',
              CustomElasticCurve(),
              Colors.green,
            ),
            
            // 3. Custom spring curve
            _buildCustomCurveDemo(
              'Custom Spring',
              CustomSpringCurve(),
              Colors.orange,
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildCustomCurveDemo(
    String label,
    Curve curve,
    Color color,
  ) {
    final animation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: curve),
    )..addListener(() { setState(() {}); });
    
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text(
              label,
              style: const TextStyle(
                fontWeight: FontWeight.bold,
                fontSize: 16,
              ),
            ),
            const SizedBox(height: 8),
            Container(
              width: double.infinity,
              height: 50,
              color: Colors.grey[200],
              child: AnimatedBuilder(
                animation: animation,
                builder: (context, child) {
                  return Container(
                    width: animation.value * 300,
                    height: 50,
                    color: color,
                    child: Center(
                      child: Text(
                        '${(animation.value * 100).round()}%',
                        style: const TextStyle(color: Colors.white),
                      ),
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// 1. Custom bounce curve
class CustomBounceCurve extends Curve {
  const CustomBounceCurve();

  @override
  double transform(double t) {
    // Bounce effect: overshoot and bounce back
    if (t < 0.5) {
      return 2 * t * t;
    } else {
      return 1 - 2 * (1 - t) * (1 - t);
    }
  }
}

/// 2. Custom elastic curve
class CustomElasticCurve extends Curve {
  const CustomElasticCurve();

  @override
  double transform(double t) {
    // Elastic effect: overshoot and oscillate
    const double c = 2 * 3.14159 / 3;
    return t == 0 || t == 1 
        ? t 
        : 1 - pow(2, -10 * t) * cos((t * 10 - 0.75) * c);
  }
}

/// 3. Custom spring curve
class CustomSpringCurve extends Curve {
  const CustomSpringCurve();

  @override
  double transform(double t) {
    // Spring effect: overshoot and settle
    const double damping = 0.8;
    const double stiffness = 10.0;
    
    // Simple spring physics simulation
    double x = t;
    double v = 0;
    final double dt = 0.01;
    
    for (double time = 0; time < 1; time += dt) {
      final double force = -stiffness * (x - t) - damping * v;
      v += force * dt;
      x += v * dt;
    }
    
    return x.clamp(0.0, 1.0);
  }
}
```

What's happening here?
- CustomBounceCurve: Overshoot and bounce
- CustomElasticCurve: Oscillating overshoot
- CustomSpringCurve: Physics-based spring
- Override transform() to define behavior

---

# Curve Intervals

> **Using intervals** to control timing.

```dart
/// Curve interval example
class CurveIntervalExample extends StatefulWidget {
  const CurveIntervalExample({super.key});

  @override
  State<CurveIntervalExample> createState() => _CurveIntervalExampleState();
}

class _CurveIntervalExampleState extends State<CurveIntervalExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _delayedAnimation;
  late Animation<double> _staggeredAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    )..repeat();
    
    // 1. Delayed animation (starts after 0.5 seconds)
    _delayedAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: const Interval(
          0.5, // Start at 50% of animation time
          1.0, // End at 100% of animation time
          curve: Curves.easeInOut,
        ),
      ),
    )..addListener(() { setState(() {}); });
    
    // 2. Staggered animation (multiple phases)
    _staggeredAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: const Interval(
          0.2,
          0.8,
          curve: Curves.elasticOut,
        ),
      ),
    )..addListener(() { setState(() {}); });
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
        title: const Text('Curve Intervals'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Delayed animation
            _buildIntervalDemo(
              'Delayed (0.5s delay)',
              _delayedAnimation,
              Colors.blue,
            ),
            
            const SizedBox(height: 16),
            
            // 2. Staggered animation
            _buildIntervalDemo(
              'Staggered (20%-80%)',
              _staggeredAnimation,
              Colors.green,
            ),
            
            const SizedBox(height: 24),
            
            // 3. Interval visualization
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Animation Progress:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  LinearProgressIndicator(
                    value: _controller.value,
                    backgroundColor: Colors.grey[300],
                    valueColor: const AlwaysStoppedAnimation<Color>(Colors.blue),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    '${(_controller.value * 100).round()}%',
                    style: const TextStyle(fontSize: 12),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildIntervalDemo(
    String label,
    Animation<double> animation,
    Color color,
  ) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(label, style: const TextStyle(fontWeight: FontWeight.bold)),
        const SizedBox(height: 4),
        Container(
          width: 200,
          height: 30,
          color: Colors.grey[200],
          child: AnimatedBuilder(
            animation: animation,
            builder: (context, child) {
              return Container(
                width: animation.value * 200,
                height: 30,
                color: color,
                child: Center(
                  child: Text(
                    '${(animation.value * 100).round()}%',
                    style: const TextStyle(color: Colors.white, fontSize: 12),
                  ),
                ),
              );
            },
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- Interval defines when animation runs (0.0 to 1.0)
- Delayed: Starts at 50% of total time
- Staggered: Runs between 20% and 80%
- Combined with curves for complex effects

---

# Real-World Examples

> **Common patterns** with curves.

```dart
/// 1. Animated button with bounce
class BounceButton extends StatefulWidget {
  const BounceButton({
    super.key,
    required this.onPressed,
    required this.child,
  });

  final VoidCallback onPressed;
  final Widget child;

  @override
  State<BounceButton> createState() => _BounceButtonState();
}

class _BounceButtonState extends State<BounceButton>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _bounceAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 800),
      vsync: this,
    );
    
    // Bounce effect when pressed
    _bounceAnimation = Tween<double>(begin: 1.0, end: 1.2).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.bounceOut,
      ),
    )..addListener(() { setState(() {}); });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) {
        // Start bounce animation
        _controller.forward(from: 0.0);
      },
      onTap: widget.onPressed,
      child: AnimatedBuilder(
        animation: _bounceAnimation,
        builder: (context, child) {
          return Transform.scale(
            scale: _bounceAnimation.value,
            child: Container(
              padding: const EdgeInsets.symmetric(
                horizontal: 24,
                vertical: 12,
              ),
              decoration: BoxDecoration(
                color: Colors.blue,
                borderRadius: BorderRadius.circular(8),
              ),
              child: widget.child,
            ),
          );
        },
      ),
    );
  }
}

/// 2. Elastic page transition
class ElasticPageTransition extends StatelessWidget {
  const ElasticPageTransition({
    super.key,
    required this.child,
    required this.animation,
  });

  final Widget child;
  final Animation<double> animation;

  @override
  Widget build(BuildContext context) {
    // Elastic curve for smooth page transition
    final elasticAnimation = Tween<Offset>(
      begin: const Offset(1.0, 0.0),
      end: Offset.zero,
    ).animate(
      CurvedAnimation(
        parent: animation,
        curve: Curves.elasticOut,
      ),
    );
    
    return SlideTransition(
      position: elasticAnimation,
      child: child,
    );
  }
}

/// 3. Staggered animation with intervals
class StaggeredAnimation extends StatefulWidget {
  const StaggeredAnimation({super.key});

  @override
  State<StaggeredAnimation> createState() => _StaggeredAnimationState();
}

class _StaggeredAnimationState extends State<StaggeredAnimation>
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
        title: const Text('Staggered Animation'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. First widget (0-33% of time)
            _buildStaggeredWidget(
              'Widget 1',
              const Interval(0.0, 0.33, curve: Curves.easeIn),
              Colors.blue,
            ),
            
            // 2. Second widget (33-66% of time)
            _buildStaggeredWidget(
              'Widget 2',
              const Interval(0.33, 0.66, curve: Curves.easeInOut),
              Colors.green,
            ),
            
            // 3. Third widget (66-100% of time)
            _buildStaggeredWidget(
              'Widget 3',
              const Interval(0.66, 1.0, curve: Curves.easeOut),
              Colors.red,
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildStaggeredWidget(String label, Interval interval, Color color) {
    final animation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: interval),
    )..addListener(() { setState(() {}); });
    
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Container(
        width: 200,
        height: 50,
        color: color,
        child: AnimatedBuilder(
          animation: animation,
          builder: (context, child) {
            return Transform.scale(
              scale: 0.3 + animation.value * 0.7,
              child: Opacity(
                opacity: animation.value,
                child: Center(
                  child: Text(
                    label,
                    style: const TextStyle(color: Colors.white),
                  ),
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

What's happening here?
- BounceButton: Bounce effect on press
- ElasticPageTransition: Elastic page transition
- StaggeredAnimation: Different timing per widget

---

# Best Practices

## Use Appropriate Curve

```dart
// Good - EaseInOut for natural UI animations
Curves.easeInOut

// Bad - Bounce for UI buttons
Curves.bounceOut // Use for fun, playful effects only
```

## Use Intervals for Control

```dart
// Good - Precise timing control
const Interval(0.2, 0.8, curve: Curves.easeInOut)

// Bad - Only using full animation
// Everything animates at once
```

## Create Custom Curves for Unique Effects

```dart
// Good - Custom curve for unique effect
class CustomCurve extends Curve {
  @override
  double transform(double t) {
    // Custom logic
  }
}
```

---

# Common Mistakes

## Using Wrong Curve

Wrong:
```dart
// Linear for all animations - feels robotic
Curves.linear
```

Correct:
```dart
// EaseInOut for most animations - feels natural
Curves.easeInOut
```

## Not Using Intervals

Wrong:
```dart
// All animations start at the same time
```

Correct:
```dart
// Staggered timing with intervals
const Interval(0.0, 0.5)
const Interval(0.5, 1.0)
```

---

# Summary

Curves control the rate of animation over time. Use built-in curves like easeInOut for natural motion, bounceOut for playful effects, and intervals for timing control. Create custom curves for unique effects. Curves are essential for creating polished, engaging animations.

---

# Next Steps

- [Hero](hero.md)
- [AnimatedBuilder](animatedbuilder.md)
- [CustomPainter Animation](custompainter-animation.md)

---

# Did You Know?

- Curves define animation rate
- easeInOut is most common for UI
- bounceOut creates playful effects
- elasticOut creates elastic effects
- Intervals control animation timing
- Custom curves can be created
- Curves make animations feel natural
- Curves add personality to UI