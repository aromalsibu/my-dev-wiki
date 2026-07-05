# CustomPainter Animation

Understand how to create custom animated drawings and graphics using CustomPainter in Flutter.

---

# What is it?

CustomPainter Animation combines Flutter's CustomPainter with animation to create dynamic, custom-drawn graphics. By using CustomPainter with AnimationController, you can create custom animations like loading spinners, charts, particle effects, and complex graphics that are fully customizable and highly performant.

---

# Why does it exist?

CustomPainter Animation exists to:

- Create custom graphics and visualizations
- Animate drawings and shapes
- Implement custom loading indicators
- Build animated charts and graphs
- Create particle systems and effects
- Draw custom shapes and paths
- Achieve high-performance animations

---

# Basic CustomPainter Animation

> **Creating an animated** custom painter.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic CustomPainter Animation example
class BasicCustomPainterAnimation extends StatefulWidget {
  const BasicCustomPainterAnimation({super.key});

  @override
  State<BasicCustomPainterAnimation> createState() => _BasicCustomPainterAnimationState();
}

class _BasicCustomPainterAnimationState extends State<BasicCustomPainterAnimation>
    with SingleTickerProviderStateMixin {
  // 1. AnimationController for the painter
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
        title: const Text('CustomPainter Animation'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 2. AnimatedBuilder to trigger repaint
            AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                return Container(
                  width: 200,
                  height: 200,
                  // 3. CustomPaint with our custom painter
                  child: CustomPaint(
                    painter: BasicCirclePainter(
                      progress: _animation.value,
                    ),
                  ),
                );
              },
            ),
            
            const SizedBox(height: 16),
            
            // 4. Progress indicator
            Text(
              'Progress: ${(_animation.value * 100).round()}%',
              style: const TextStyle(fontSize: 16),
            ),
          ],
        ),
      ),
    );
  }
}

/// Custom painter for animated circle
class BasicCirclePainter extends CustomPainter {
  // 1. Constructor with progress parameter
  BasicCirclePainter({
    required this.progress,
  });

  final double progress;

  @override
  void paint(Canvas canvas, Size size) {
    // 2. Calculate center and radius
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - 10;

    // 3. Create paint for the circle
    final paint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 4;

    // 4. Draw the progress arc
    canvas.drawArc(
      // Rectangle bounding the circle
      Rect.fromCircle(center: center, radius: radius),
      // Start angle (-90 degrees = top)
      -3.14159 / 2,
      // Sweep angle (progress * full circle)
      2 * 3.14159 * progress,
      false,
      paint,
    );

    // 5. Draw text in the center
    final textPainter = TextPainter(
      text: TextSpan(
        text: '${(progress * 100).round()}%',
        style: const TextStyle(
          color: Colors.blue,
          fontSize: 24,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );

    // Layout and paint text
    textPainter.layout();
    textPainter.paint(
      canvas,
      Offset(
        center.dx - textPainter.width / 2,
        center.dy - textPainter.height / 2,
      ),
    );
  }

  @override
  bool shouldRepaint(BasicCirclePainter oldDelegate) {
    // 6. Repaint when progress changes
    return oldDelegate.progress != progress;
  }
}
```

What's happening here?
- CustomPainter draws custom graphics
- progress parameter controls animation
- AnimatedBuilder triggers repaints
- shouldRepaint controls redraws

---

# Animated Loading Spinner

> **Creating a custom** loading spinner.

```dart
/// Animated loading spinner
class AnimatedSpinner extends StatefulWidget {
  const AnimatedSpinner({
    super.key,
    this.size = 100,
    this.strokeWidth = 6,
    this.color = Colors.blue,
  });

  final double size;
  final double strokeWidth;
  final Color color;

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
    return Center(
      child: AnimatedBuilder(
        animation: _controller,
        builder: (context, child) {
          return CustomPaint(
            size: Size(widget.size, widget.size),
            painter: SpinnerPainter(
              progress: _controller.value,
              strokeWidth: widget.strokeWidth,
              color: widget.color,
            ),
          );
        },
      ),
    );
  }
}

/// Spinner painter
class SpinnerPainter extends CustomPainter {
  SpinnerPainter({
    required this.progress,
    required this.strokeWidth,
    required this.color,
  });

  final double progress;
  final double strokeWidth;
  final Color color;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - strokeWidth / 2;

    // 1. Background circle
    final backgroundPaint = Paint()
      ..color = Colors.grey[300]!
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth;

    canvas.drawCircle(center, radius, backgroundPaint);

    // 2. Progress arc with gradient
    final progressPaint = Paint()
      ..shader = SweepGradient(
        colors: [
          color.withOpacity(0.3),
          color,
        ],
        startAngle: 0,
        endAngle: 2 * 3.14159,
      ).createShader(
        Rect.fromCircle(center: center, radius: radius),
      )
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth
      ..strokeCap = StrokeCap.round;

    // 3. Draw the progress arc
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -3.14159 / 2 + progress * 2 * 3.14159,
      1.5 * 3.14159,
      false,
      progressPaint,
    );
  }

  @override
  bool shouldRepaint(SpinnerPainter oldDelegate) {
    return oldDelegate.progress != progress ||
           oldDelegate.strokeWidth != strokeWidth ||
           oldDelegate.color != color;
  }
}
```

What's happening here?
- Custom spinner with gradient
- Progress arc with rounded caps
- Smooth rotation animation
- Configurable size and color

---

# Animated Wave

> **Creating an animated** wave effect.

```dart
/// Animated wave painter
class AnimatedWavePainter extends CustomPainter {
  AnimatedWavePainter({
    required this.progress,
    required this.color,
    required this.waveCount,
  });

  final double progress;
  final Color color;
  final int waveCount;

  @override
  void paint(Canvas canvas, Size size) {
    // 1. Create path for the wave
    final path = Path();
    final width = size.width;
    final height = size.height;

    // 2. Start from bottom-left
    path.moveTo(0, height);

    // 3. Generate wave points
    for (double x = 0; x <= width; x++) {
      // Calculate y position using sine wave
      final y = height / 2 +
          (height / 4) * 
          (1 - progress) *
          sin(
            x / (width / waveCount) * 2 * 3.14159 +
            progress * 2 * 3.14159,
          );

      path.lineTo(x, y);
    }

    // 4. Close the path
    path.lineTo(width, height);
    path.close();

    // 5. Fill the wave with color
    final paint = Paint()
      ..color = color.withOpacity(0.5 + 0.5 * progress)
      ..style = PaintingStyle.fill;

    canvas.drawPath(path, paint);

    // 6. Draw wave outline
    final outlinePaint = Paint()
      ..color = color
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;

    canvas.drawPath(path, outlinePaint);
  }

  @override
  bool shouldRepaint(AnimatedWavePainter oldDelegate) {
    return oldDelegate.progress != progress ||
           oldDelegate.color != color ||
           oldDelegate.waveCount != waveCount;
  }
}

/// Animated wave widget
class AnimatedWave extends StatefulWidget {
  const AnimatedWave({
    super.key,
    this.height = 100,
    this.color = Colors.blue,
    this.waveCount = 3,
    this.duration = const Duration(seconds: 2),
  });

  final double height;
  final Color color;
  final int waveCount;
  final Duration duration;

  @override
  State<AnimatedWave> createState() => _AnimatedWaveState();
}

class _AnimatedWaveState extends State<AnimatedWave>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: widget.duration,
      vsync: this,
    )..repeat();
    
    _animation = Tween<double>(begin: 0.0, end: 1.0).animate(_controller);
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
          height: widget.height,
          child: CustomPaint(
            painter: AnimatedWavePainter(
              progress: _animation.value,
              color: widget.color,
              waveCount: widget.waveCount,
            ),
          ),
        );
      },
    );
  }
}
```

What's happening here?
- Animated wave effect
- Sine wave calculation
- Progress-based animation
- Configurable wave count

---

# Particle System

> **Creating an animated** particle system.

```dart
/// Particle class
class Particle {
  double x;
  double y;
  double speedX;
  double speedY;
  double size;
  double opacity;
  Color color;

  Particle({
    required this.x,
    required this.y,
    required this.speedX,
    required this.speedY,
    required this.size,
    required this.opacity,
    required this.color,
  });

  void update(double progress) {
    x += speedX;
    y += speedY;
    opacity = (1 - progress) * 0.8 + 0.2;
  }
}

/// Particle system painter
class ParticlePainter extends CustomPainter {
  ParticlePainter({
    required this.progress,
    required this.particles,
  });

  final double progress;
  final List<Particle> particles;

  @override
  void paint(Canvas canvas, Size size) {
    // Update and draw each particle
    for (final particle in particles) {
      particle.update(progress);

      // Paint particle
      final paint = Paint()
        ..color = particle.color.withOpacity(particle.opacity)
        ..style = PaintingStyle.fill;

      // Draw particle as circle
      canvas.drawCircle(
        Offset(particle.x, particle.y),
        particle.size * (1 + progress * 0.5),
        paint,
      );
    }
  }

  @override
  bool shouldRepaint(ParticlePainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}

/// Animated particle system widget
class AnimatedParticles extends StatefulWidget {
  const AnimatedParticles({
    super.key,
    this.particleCount = 50,
    this.color = Colors.blue,
    this.duration = const Duration(seconds: 3),
  });

  final int particleCount;
  final Color color;
  final Duration duration;

  @override
  State<AnimatedParticles> createState() => _AnimatedParticlesState();
}

class _AnimatedParticlesState extends State<AnimatedParticles>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;
  late List<Particle> _particles;

  @override
  void initState() {
    super.initState();
    
    // Initialize particles
    _particles = List.generate(widget.particleCount, (index) {
      return Particle(
        x: 50 + 300 * (index % 10) / 10,
        y: 50 + 300 * (index ~/ 10) / 10,
        speedX: (index % 5 - 2) * 0.5,
        speedY: (index % 7 - 3) * 0.5,
        size: 2 + index % 8,
        opacity: 0.5 + (index % 5) / 10,
        color: widget.color.withOpacity(0.5 + (index % 5) / 10),
      );
    });

    _controller = AnimationController(
      duration: widget.duration,
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
    return Container(
      width: 300,
      height: 300,
      decoration: BoxDecoration(
        color: Colors.grey[900],
        borderRadius: BorderRadius.circular(16),
      ),
      child: AnimatedBuilder(
        animation: _animation,
        builder: (context, child) {
          return CustomPaint(
            painter: ParticlePainter(
              progress: _animation.value,
              particles: _particles,
            ),
          );
        },
      ),
    );
  }
}
```

What's happening here?
- Particle system with multiple particles
- Each particle has unique properties
- Particles move and change over time
- Visual particle effects

---

# Real-World Examples

> **Common patterns** with CustomPainter Animation.

```dart
/// 1. Animated chart
class AnimatedChart extends StatefulWidget {
  const AnimatedChart({
    super.key,
    required this.data,
    this.height = 200,
    this.color = Colors.blue,
  });

  final List<double> data;
  final double height;
  final Color color;

  @override
  State<AnimatedChart> createState() => _AnimatedChartState();
}

class _AnimatedChartState extends State<AnimatedChart>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..forward();
    
    _animation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeOut),
    );
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
          height: widget.height,
          child: CustomPaint(
            painter: ChartPainter(
              data: widget.data,
              progress: _animation.value,
              color: widget.color,
            ),
          ),
        );
      },
    );
  }
}

/// Chart painter
class ChartPainter extends CustomPainter {
  ChartPainter({
    required this.data,
    required this.progress,
    required this.color,
  });

  final List<double> data;
  final double progress;
  final Color color;

  @override
  void paint(Canvas canvas, Size size) {
    if (data.isEmpty) return;

    // 1. Find max value for scaling
    final maxValue = data.reduce((a, b) => a > b ? a : b);
    if (maxValue == 0) return;

    // 2. Calculate width per bar
    final barWidth = size.width / data.length * 0.7;
    final spacing = size.width / data.length * 0.3;

    // 3. Draw bars
    for (int i = 0; i < data.length; i++) {
      final height = (data[i] / maxValue) * size.height * progress;
      final x = i * (barWidth + spacing) + spacing / 2;
      final y = size.height - height;

      // Bar paint
      final paint = Paint()
        ..color = color.withOpacity(0.3 + 0.7 * (i / data.length))
        ..style = PaintingStyle.fill;

      // Draw bar with rounded corners
      final rect = Rect.fromLTWH(
        x,
        y,
        barWidth,
        height,
      );
      
      final rrect = RRect.fromRectAndRadius(
        rect,
        const Radius.circular(4),
      );
      
      canvas.drawRRect(rrect, paint);

      // Value text
      final textPainter = TextPainter(
        text: TextSpan(
          text: '${(data[i]).toStringAsFixed(1)}',
          style: const TextStyle(
            color: Colors.black,
            fontSize: 10,
          ),
        ),
        textDirection: TextDirection.ltr,
      );
      
      textPainter.layout();
      textPainter.paint(
        canvas,
        Offset(
          x + barWidth / 2 - textPainter.width / 2,
          y - textPainter.height - 4,
        ),
      );
    }
  }

  @override
  bool shouldRepaint(ChartPainter oldDelegate) {
    return oldDelegate.progress != progress ||
           oldDelegate.data != data ||
           oldDelegate.color != color;
  }
}
```

What's happening here?
- Animated bar chart
- Progress-based animation
- Custom drawing and text
- Data visualization

---

# Best Practices

## Use shouldRepaint Properly

```dart
// Good - Only repaint when needed
@override
bool shouldRepaint(MyPainter oldDelegate) {
  return oldDelegate.progress != progress ||
         oldDelegate.color != color;
}

// Bad - Always repaint
@override
bool shouldRepaint(MyPainter oldDelegate) {
  return true;
}
```

## Optimize Paint Methods

```dart
// Good - Efficient painting
@override
void paint(Canvas canvas, Size size) {
  // Pre-calculate values
  final center = Offset(size.width / 2, size.height / 2);
  // Use for loops efficiently
}

// Bad - Inefficient painting
@override
void paint(Canvas canvas, Size size) {
  // Complex calculations in loops
}
```

## Use AnimatedBuilder for Updates

```dart
// Good - AnimatedBuilder triggers repaint
AnimatedBuilder(
  animation: _animation,
  builder: (context, child) {
    return CustomPaint(
      painter: MyPainter(progress: _animation.value),
    );
  },
)
```

---

# Common Mistakes

## Not Implementing shouldRepaint

Wrong:
```dart
// Missing shouldRepaint - causes issues
class MyPainter extends CustomPainter {
  @override
  void paint(...) { ... }
  // Missing shouldRepaint
}
```

Correct:
```dart
// Proper shouldRepaint
class MyPainter extends CustomPainter {
  @override
  void paint(...) { ... }
  
  @override
  bool shouldRepaint(MyPainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}
```

## Heavy Computations in paint

Wrong:
```dart
@override
void paint(Canvas canvas, Size size) {
  // Heavy computation every frame
  final complexData = calculateComplexData();
}
```

Correct:
```dart
// Pre-calculate data
final complexData = calculateComplexData();

@override
void paint(Canvas canvas, Size size) {
  // Use pre-calculated data
}
```

---

# Summary

CustomPainter Animation enables custom animated graphics and visualizations. Use CustomPainter with AnimationController for custom drawings like loading spinners, waves, particle systems, and charts. Optimize with proper shouldRepaint and efficient painting logic.

---

# Next Steps

- [Physics Animations](physics-animations.md)
- [Hero](hero.md)
- [AnimatedBuilder](animatedbuilder.md)

---

# Did You Know?

- CustomPainter provides full drawing control
- shouldRepaint controls redraw frequency
- Canvas provides drawing methods
- Paint controls style and color
- Path defines complex shapes
- AnimationController drives animation
- AnimatedBuilder triggers repaints
- CustomPainter is highly performant