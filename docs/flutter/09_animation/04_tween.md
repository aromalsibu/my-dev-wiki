# Tween

Understand how to animate values between a beginning and ending state in Flutter.

---

# What is it?

Tween is a class that defines the range of values for an animation. It interpolates between a beginning value (begin) and an ending value (end) based on an animation's progress (0.0 to 1.0). Tweens are used with AnimationController to create smooth transitions for any type of value, including numbers, colors, sizes, and custom objects.

---

# Why does it exist?

Tween exists to:

- Define animation value ranges
- Interpolate between two values
- Support various data types
- Enable custom interpolation
- Create smooth transitions
- Chain and combine animations
- Provide type-safe animations

---

# Basic Tween

> **Creating and using** basic Tweens.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic Tween example
class BasicTweenExample extends StatefulWidget {
  const BasicTweenExample({super.key});

  @override
  State<BasicTweenExample> createState() => _BasicTweenExampleState();
}

class _BasicTweenExampleState extends State<BasicTweenExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  
  // 1. Tween for double values
  // Interpolates between 0.0 and 1.0
  late Animation<double> _doubleTween;
  
  // 2. Tween for colors
  // Interpolates between two colors
  late Animation<Color?> _colorTween;
  
  // 3. Tween for sizes
  // Interpolates between two sizes
  late Animation<double> _sizeTween;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..repeat(reverse: true);
    
    // 4. Double Tween - between 0.0 and 1.0
    // The most common type of Tween
    _doubleTween = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(_controller)
      ..addListener(() { setState(() {}); });
    
    // 5. Color Tween - between blue and red
    // ColorTween handles color interpolation
    _colorTween = ColorTween(
      begin: Colors.blue,
      end: Colors.red,
    ).animate(_controller)
      ..addListener(() { setState(() {}); });
    
    // 6. Size Tween - between 50 and 200
    _sizeTween = Tween<double>(
      begin: 50.0,
      end: 200.0,
    ).animate(
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
        title: const Text('Basic Tween'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 7. Use Tween values
            Container(
              width: _sizeTween.value,
              height: _sizeTween.value,
              color: _colorTween.value,
              child: Center(
                child: Text(
                  '${(_doubleTween.value * 100).round()}%',
                  style: const TextStyle(color: Colors.white),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 8. Tween values display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  Text('Double: ${_doubleTween.value.toStringAsFixed(2)}'),
                  Text('Color: ${_colorTween.value}'),
                  Text('Size: ${_sizeTween.value.toStringAsFixed(0)}'),
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
- Tween<double>: Interpolates between double values
- ColorTween: Interpolates between colors
- begin: Starting value
- end: Ending value
- animate(): Connects to AnimationController

---

# Tween Types

> **Different Tween** types for different data.

```dart
/// Various Tween types
class TweenTypesExample extends StatefulWidget {
  const TweenTypesExample({super.key});

  @override
  State<TweenTypesExample> createState() => _TweenTypesExampleState();
}

class _TweenTypesExampleState extends State<TweenTypesExample>
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
        title: const Text('Tween Types'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. IntTween - Between integer values
            _buildTweenDemo(
              'IntTween',
              IntTween(begin: 0, end: 100).animate(_controller),
              (value) => 'Count: $value',
              Colors.blue,
            ),
            
            // 2. ColorTween - Between colors
            _buildTweenDemo(
              'ColorTween',
              ColorTween(begin: Colors.blue, end: Colors.red).animate(_controller),
              (value) => 'Color: $value',
              Colors.blue,
              isColor: true,
            ),
            
            // 3. AlignmentTween - Between alignments
            _buildTweenDemo(
              'AlignmentTween',
              AlignmentTween(
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
              ).animate(_controller),
              (value) => 'Alignment: ${value.x.toStringAsFixed(2)}, ${value.y.toStringAsFixed(2)}',
              Colors.green,
            ),
            
            // 4. BorderRadiusTween - Between border radii
            _buildTweenDemo(
              'BorderRadiusTween',
              BorderRadiusTween(
                begin: BorderRadius.circular(0),
                end: BorderRadius.circular(50),
              ).animate(_controller),
              (value) => 'BorderRadius: ${value.topLeft.x.toStringAsFixed(0)}',
              Colors.orange,
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildTweenDemo<T>(
    String title,
    Animation<T> animation,
    String Function(T) display,
    Color color, {
    bool isColor = false,
  }) {
    animation.addListener(() { setState(() {}); });
    
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              title,
              style: const TextStyle(
                fontWeight: FontWeight.bold,
                fontSize: 16,
              ),
            ),
            const SizedBox(height: 8),
            
            // Visual representation
            if (isColor)
              Container(
                width: double.infinity,
                height: 50,
                color: animation.value as Color,
              )
            else
              Container(
                width: double.infinity,
                height: 50,
                color: Colors.grey[200],
                child: Center(
                  child: Text(
                    display(animation.value),
                    style: const TextStyle(fontWeight: FontWeight.bold),
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
- IntTween: Integer values
- ColorTween: Color transitions
- AlignmentTween: Alignment changes
- BorderRadiusTween: Border radius changes
- Each Tween type handles specific data

---

# Custom Tween

> **Creating custom** Tween classes.

```dart
/// Custom Tween example
class CustomTweenExample extends StatefulWidget {
  const CustomTweenExample({super.key});

  @override
  State<CustomTweenExample> createState() => _CustomTweenExampleState();
}

class _CustomTweenExampleState extends State<CustomTweenExample>
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
        title: const Text('Custom Tween'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Custom SizeTween
            _buildCustomTweenDemo(),
            
            const SizedBox(height: 24),
            
            // 2. Custom TransformTween
            _buildTransformTweenDemo(),
          ],
        ),
      ),
    );
  }

  Widget _buildCustomTweenDemo() {
    // Custom Tween for Size
    final sizeTween = SizeTween(
      begin: const Size(50, 50),
      end: const Size(200, 100),
    ).animate(_controller)
      ..addListener(() { setState(() {}); });
    
    return Container(
      width: sizeTween.value.width,
      height: sizeTween.value.height,
      color: Colors.blue,
      child: const Center(
        child: Text(
          'Size',
          style: TextStyle(color: Colors.white),
        ),
      ),
    );
  }

  Widget _buildTransformTweenDemo() {
    // Custom Tween for Transform
    final transformTween = TransformTween(
      begin: Transform.rotate(angle: 0),
      end: Transform.rotate(angle: 3.14159),
    ).animate(_controller)
      ..addListener(() { setState(() {}); });
    
    return Container(
      width: 100,
      height: 100,
      color: Colors.green,
      child: Transform(
        transform: transformTween.value.transform,
        child: const Center(
          child: Text(
            'Transform',
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }
}

/// Custom SizeTween class
/// Interpolates between two Size values
class SizeTween extends Tween<Size> {
  SizeTween({super.begin, super.end});

  @override
  Size lerp(double t) {
    return Size(
      begin!.width + (end!.width - begin!.width) * t,
      begin!.height + (end!.height - begin!.height) * t,
    );
  }
}

/// Custom TransformTween class
/// Interpolates between two Transform values
class TransformTween extends Tween<Transform> {
  TransformTween({super.begin, super.end});

  @override
  Transform lerp(double t) {
    // Interpolate the transform matrix
    final beginMatrix = begin!.transform;
    final endMatrix = end!.transform;
    
    // Simple interpolation of matrix values
    final matrix = Matrix4.zero();
    for (int i = 0; i < 16; i++) {
      matrix[i] = beginMatrix[i] + (endMatrix[i] - beginMatrix[i]) * t;
    }
    
    return Transform(matrix: matrix);
  }
}
```

What's happening here?
- SizeTween: Custom size interpolation
- TransformTween: Custom transform interpolation
- Override lerp() for custom logic
- Type-safe custom Tweens

---

# Tween Sequences

> **Chaining Tweens** for sequences.

```dart
/// Tween sequence example
class TweenSequenceExample extends StatefulWidget {
  const TweenSequenceExample({super.key});

  @override
  State<TweenSequenceExample> createState() => _TweenSequenceExampleState();
}

class _TweenSequenceExampleState extends State<TweenSequenceExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _sequenceAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(seconds: 4),
      vsync: this,
    )..repeat();
    
    // 1. Sequence of tweens
    // Each tween runs in sequence
    _sequenceAnimation = TweenSequence<double>([
      // First: Grow from 0 to 1 (0-1 seconds)
      TweenSequenceItem(
        tween: Tween<double>(begin: 0.0, end: 1.0),
        weight: 1, // Equal weight
      ),
      // Second: Stay at 1 (1-2 seconds)
      TweenSequenceItem(
        tween: ConstantTween<double>(1.0),
        weight: 1,
      ),
      // Third: Shrink from 1 to 0 (2-3 seconds)
      TweenSequenceItem(
        tween: Tween<double>(begin: 1.0, end: 0.0),
        weight: 1,
      ),
      // Fourth: Stay at 0 (3-4 seconds)
      TweenSequenceItem(
        tween: ConstantTween<double>(0.0),
        weight: 1,
      ),
    ]).animate(_controller)
      ..addListener(() { setState(() {}); });
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
        title: const Text('Tween Sequence'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 2. Visual representation of sequence
            Container(
              width: 100 + _sequenceAnimation.value * 100,
              height: 100 + _sequenceAnimation.value * 100,
              color: Colors.blue,
              child: Center(
                child: Text(
                  '${(_sequenceAnimation.value * 100).round()}%',
                  style: const TextStyle(color: Colors.white),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 3. Sequence status
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Sequence:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 4),
                  Text('Value: ${_sequenceAnimation.value.toStringAsFixed(2)}'),
                  Text('Grow → Stay → Shrink → Stay'),
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
- TweenSequence combines multiple tweens
- Each TweenSequenceItem has a tween and weight
- weight determines duration proportion
- ConstantTween holds a constant value

---

# Real-World Examples

> **Common patterns** with Tween.

```dart
/// 1. Animated counter with IntTween
class AnimatedCounter extends StatefulWidget {
  const AnimatedCounter({
    super.key,
    required this.targetValue,
    this.duration = const Duration(seconds: 2),
  });

  final int targetValue;
  final Duration duration;

  @override
  State<AnimatedCounter> createState() => _AnimatedCounterState();
}

class _AnimatedCounterState extends State<AnimatedCounter>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<int> _animation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: widget.duration,
      vsync: this,
    );
    
    // IntTween for counting
    _animation = IntTween(
      begin: 0,
      end: widget.targetValue,
    ).animate(_controller)
      ..addListener(() { setState(() {}); });
    
    _controller.forward();
  }

  @override
  void didUpdateWidget(covariant AnimatedCounter oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.targetValue != oldWidget.targetValue) {
      _animation = IntTween(
        begin: oldWidget.targetValue,
        end: widget.targetValue,
      ).animate(_controller)
        ..addListener(() { setState(() {}); });
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
    return Text(
      _animation.value.toString(),
      style: const TextStyle(
        fontSize: 48,
        fontWeight: FontWeight.bold,
      ),
    );
  }
}

/// 2. Animated progress with ColorTween
class AnimatedProgress extends StatefulWidget {
  const AnimatedProgress({
    super.key,
    required this.progress,
    this.duration = const Duration(milliseconds: 500),
  });

  final double progress;
  final Duration duration;

  @override
  State<AnimatedProgress> createState() => _AnimatedProgressState();
}

class _AnimatedProgressState extends State<AnimatedProgress>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _progressAnimation;
  late Animation<Color?> _colorAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: widget.duration,
      vsync: this,
    );
    
    // Progress animation
    _progressAnimation = Tween<double>(
      begin: 0.0,
      end: widget.progress,
    ).animate(_controller)
      ..addListener(() { setState(() {}); });
    
    // Color animation based on progress
    _colorAnimation = ColorTween(
      begin: Colors.red,
      end: Colors.green,
    ).animate(_controller)
      ..addListener(() { setState(() {}); });
    
    _controller.forward();
  }

  @override
  void didUpdateWidget(covariant AnimatedProgress oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.progress != oldWidget.progress) {
      _progressAnimation = Tween<double>(
        begin: oldWidget.progress,
        end: widget.progress,
      ).animate(_controller)
        ..addListener(() { setState(() {}); });
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
    return Column(
      children: [
        Container(
          height: 20,
          decoration: BoxDecoration(
            color: Colors.grey[200],
            borderRadius: BorderRadius.circular(10),
          ),
          child: AnimatedBuilder(
            animation: _progressAnimation,
            builder: (context, child) {
              return Container(
                width: _progressAnimation.value * 300,
                decoration: BoxDecoration(
                  color: _colorAnimation.value,
                  borderRadius: BorderRadius.circular(10),
                ),
                child: Center(
                  child: Text(
                    '${(_progressAnimation.value * 100).round()}%',
                    style: const TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                      fontSize: 12,
                    ),
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
- AnimatedCounter with IntTween
- AnimatedProgress with Tween and ColorTween
- Updated on value changes
- Smooth transitions between values

---

# Best Practices

## Use Appropriate Tween Type

```dart
// Good - Correct tween type
IntTween(begin: 0, end: 100) // For integers
ColorTween(begin: Colors.blue, end: Colors.red) // For colors

// Bad - Wrong tween type
Tween<double>(begin: 0.0, end: 100.0) // Should be IntTween if used for integers
```

## Use Curves with Tweens

```dart
// Good - With curve
Tween<double>(begin: 0.0, end: 1.0).animate(
  CurvedAnimation(parent: controller, curve: Curves.easeInOut),
)

// Bad - Linear
Tween<double>(begin: 0.0, end: 1.0).animate(controller)
```

## Create Custom Tweens for Complex Types

```dart
// Good - Custom tween for complex type
class CustomTween extends Tween<CustomType> {
  CustomTween({super.begin, super.end});
  
  @override
  CustomType lerp(double t) {
    // Custom interpolation logic
  }
}
```

---

# Common Mistakes

## Wrong Tween Type

Wrong:
```dart
// Using double Tween for integers
Tween<double>(begin: 0.0, end: 10.0) // Returns double, not int
```

Correct:
```dart
// Using IntTween for integers
IntTween(begin: 0, end: 10) // Returns integer
```

## Not Handling Null Values

Wrong:
```dart
// begin or end might be null
ColorTween(begin: null, end: Colors.red) // Causes issues
```

Correct:
```dart
// Always provide values
ColorTween(begin: Colors.blue, end: Colors.red)
```

---

# Summary

Tween defines the range of values for animations. Use Tween<double> for numbers, ColorTween for colors, IntTween for integers, and custom Tweens for custom types. TweenSequence combines multiple tweens in sequence. Tweens are essential for creating smooth animations.

---

# Next Steps

- [Hero](hero.md)
- [AnimatedBuilder](animatedbuilder.md)
- [CustomPainter Animation](custompainter-animation.md)

---

# Did You Know?

- Tween interpolates between values
- ColorTween handles color interpolation
- IntTween handles integer values
- AlignmentTween handles alignments
- BorderRadiusTween handles border radii
- TweenSequence chains animations
- ConstantTween holds constant values
- Custom Tweens can be created