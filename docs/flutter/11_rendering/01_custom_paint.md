# CustomPaint

Understand how to create custom graphics and drawings with CustomPaint in Flutter.

---

# What is it?

CustomPaint is a widget that provides a canvas for custom drawing and painting. It allows you to create any visual element by drawing directly on a canvas using the Canvas API. CustomPaint is used for custom shapes, charts, animations, and any graphics that cannot be created with standard Flutter widgets.

---

# Why does it exist?

CustomPaint exists to:

- Create custom graphics and shapes
- Draw custom charts and graphs
- Implement custom UI elements
- Create animations and effects
- Design unique visual components
- Draw complex paths and shapes
- Achieve pixel-perfect designs

---

# Basic CustomPaint

> **Creating custom** drawings with CustomPaint.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic CustomPaint example
class BasicCustomPaintExample extends StatelessWidget {
  const BasicCustomPaintExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('CustomPaint'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. CustomPaint widget with custom painter
            CustomPaint(
              // Size of the painting area
              size: const Size(200, 200),
              // Custom painter class that does the drawing
              painter: BasicShapePainter(),
            ),
            const SizedBox(height: 16),
            const Text(
              'Custom drawings with CustomPaint',
              style: TextStyle(fontSize: 16),
            ),
          ],
        ),
      ),
    );
  }
}

/// Custom painter for basic shapes
class BasicShapePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1. Create paint objects for styling
    // Paint defines how shapes are drawn (color, style, stroke width, etc.)
    
    // Paint for filled shapes
    final fillPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;
    
    // Paint for outlined shapes
    final strokePaint = Paint()
      ..color = Colors.black
      ..style = PaintingStyle.stroke
      ..strokeWidth = 3;

    // 2. Draw a rectangle
    // Rect.fromLTWH creates a rectangle from left, top, width, height
    canvas.drawRect(
      const Rect.fromLTWH(20, 20, 80, 60),
      fillPaint,
    );
    canvas.drawRect(
      const Rect.fromLTWH(20, 20, 80, 60),
      strokePaint,
    );

    // 3. Draw a circle
    // Offset represents a point in 2D space
    // Radius controls the size of the circle
    canvas.drawCircle(
      const Offset(140, 50),
      30,
      fillPaint..color = Colors.green,
    );
    canvas.drawCircle(
      const Offset(140, 50),
      30,
      strokePaint,
    );

    // 4. Draw a line
    // A line is drawn from one point to another
    canvas.drawLine(
      const Offset(20, 120),
      const Offset(180, 120),
      strokePaint..color = Colors.red,
    );

    // 5. Draw a path
    // Path allows complex shapes with curves and lines
    final path = Path()
      ..moveTo(20, 140)
      ..lineTo(60, 180)
      ..lineTo(100, 140)
      ..lineTo(140, 180)
      ..lineTo(180, 140);
    
    canvas.drawPath(
      path,
      strokePaint..color = Colors.purple,
    );

    // 6. Draw text
    // TextPainter handles text rendering on canvas
    final textPainter = TextPainter(
      text: const TextSpan(
        text: 'Custom Paint',
        style: TextStyle(
          color: Colors.black,
          fontSize: 16,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    
    // Layout the text to calculate its size
    textPainter.layout();
    
    // Paint the text at a specific position
    textPainter.paint(
      canvas,
      const Offset(50, 20),
    );
  }

  @override
  bool shouldRepaint(BasicShapePainter oldDelegate) {
    // Return true if the painter needs to redraw
    // For static drawings, return false
    return false;
  }
}
```

What's happening here?
- CustomPaint provides a canvas for drawing
- CustomPainter defines what to draw
- Paint objects control style (color, stroke, etc.)
- Canvas methods draw shapes (rect, circle, line, path)
- TextPainter adds text to the canvas

---

# Custom Shapes

> **Drawing complex** shapes with CustomPaint.

```dart
/// Custom shapes example
class CustomShapesExample extends StatelessWidget {
  const CustomShapesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Shapes'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(300, 300),
          painter: CustomShapesPainter(),
        ),
      ),
    );
  }
}

/// Custom painter for various shapes
class CustomShapesPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1. Star shape using Path
    final starPaint = Paint()
      ..color = Colors.orange
      ..style = PaintingStyle.fill;
    
    // Create a star path
    final starPath = Path();
    const int points = 5;
    const double outerRadius = 40;
    const double innerRadius = 16;
    const double centerX = 60;
    const double centerY = 50;
    
    // Calculate star points using trigonometry
    for (int i = 0; i < points * 2; i++) {
      final double radius = i.isEven ? outerRadius : innerRadius;
      final double angle = i * 3.14159 / points - 3.14159 / 2;
      final double x = centerX + radius * cos(angle);
      final double y = centerY + radius * sin(angle);
      
      if (i == 0) {
        starPath.moveTo(x, y);
      } else {
        starPath.lineTo(x, y);
      }
    }
    starPath.close();
    
    canvas.drawPath(starPath, starPaint);

    // 2. Heart shape
    final heartPaint = Paint()
      ..color = Colors.red
      ..style = PaintingStyle.fill;
    
    final heartPath = Path()
      ..moveTo(150, 80)
      ..cubicTo(120, 30, 70, 50, 150, 100)
      ..cubicTo(230, 50, 180, 30, 150, 80);
    
    canvas.drawPath(heartPath, heartPaint);

    // 3. Arc (partial circle)
    final arcPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 4;
    
    canvas.drawArc(
      const Rect.fromLTWH(200, 20, 80, 80),
      0,
      1.5 * 3.14159,
      false,
      arcPaint,
    );

    // 4. Rounded rectangle
    final roundedRectPaint = Paint()
      ..color = Colors.green
      ..style = PaintingStyle.fill;
    
    final rrect = RRect.fromRectAndRadius(
      const Rect.fromLTWH(30, 120, 100, 60),
      const Radius.circular(10),
    );
    canvas.drawRRect(rrect, roundedRectPaint);

    // 5. Triangle
    final trianglePaint = Paint()
      ..color = Colors.purple
      ..style = PaintingStyle.fill;
    
    final trianglePath = Path()
      ..moveTo(180, 180)
      ..lineTo(230, 120)
      ..lineTo(280, 180)
      ..close();
    
    canvas.drawPath(trianglePath, trianglePaint);

    // 6. Gradient circle
    final gradientPaint = Paint()
      ..shader = const RadialGradient(
        colors: [Colors.yellow, Colors.orange],
        center: Alignment.center,
        radius: 0.5,
      ).createShader(const Rect.fromLTWH(40, 200, 60, 60))
      ..style = PaintingStyle.fill;
    
    canvas.drawCircle(
      const Offset(70, 230),
      30,
      gradientPaint,
    );

    // 7. Dashed line
    final dashPaint = Paint()
      ..color = Colors.blueGrey
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;
    
    // Create a dashed path
    final dashPath = Path();
    double x = 130;
    double y = 200;
    const double dashLength = 10;
    const double gapLength = 5;
    
    while (x < 250) {
      dashPath.moveTo(x, y);
      x += dashLength;
      dashPath.lineTo(x, y);
      x += gapLength;
    }
    
    canvas.drawPath(dashPath, dashPaint);
  }

  @override
  bool shouldRepaint(CustomShapesPainter oldDelegate) => false;
}
```

What's happening here?
- Star: Complex path with trigonometric calculations
- Heart: Cubic Bézier curves
- Arc: Partial circle drawing
- Rounded rectangle: RRect for rounded corners
- Triangle: Simple path with close()
- Gradient: Shader for gradient effects
- Dashed line: Manual dashed path creation

---

# CustomPaint with Animation

> **Animating** custom drawings.

```dart
/// Animated CustomPaint example
class AnimatedCustomPaintExample extends StatefulWidget {
  const AnimatedCustomPaintExample({super.key});

  @override
  State<AnimatedCustomPaintExample> createState() => _AnimatedCustomPaintExampleState();
}

class _AnimatedCustomPaintExampleState extends State<AnimatedCustomPaintExample>
    with SingleTickerProviderStateMixin {
  // 1. Animation controller for the painting
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
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
        title: const Text('Animated CustomPaint'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 2. Animated CustomPaint
            AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                return CustomPaint(
                  size: const Size(300, 300),
                  painter: AnimatedPainter(
                    progress: _animation.value,
                  ),
                );
              },
            ),
            
            const SizedBox(height: 16),
            
            // 3. Progress indicator
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

/// Animated painter
class AnimatedPainter extends CustomPainter {
  AnimatedPainter({required this.progress});

  final double progress;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - 20;

    // 1. Background circle
    final bgPaint = Paint()
      ..color = Colors.grey[300]!
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8;

    canvas.drawCircle(center, radius, bgPaint);

    // 2. Progress arc
    final progressPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8
      ..strokeCap = StrokeCap.round;

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -3.14159 / 2,
      2 * 3.14159 * progress,
      false,
      progressPaint,
    );

    // 3. Animated circle
    final circlePaint = Paint()
      ..color = Colors.red
      ..style = PaintingStyle.fill;

    final angle = progress * 2 * 3.14159 - 3.14159 / 2;
    final circleX = center.dx + radius * cos(angle);
    final circleY = center.dy + radius * sin(angle);

    canvas.drawCircle(
      Offset(circleX, circleY),
      10 + progress * 10,
      circlePaint,
    );

    // 4. Animated text
    final textPainter = TextPainter(
      text: TextSpan(
        text: '${(progress * 100).round()}%',
        style: TextStyle(
          color: Colors.blue,
          fontSize: 24 + progress * 16,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );

    textPainter.layout();
    textPainter.paint(
      canvas,
      Offset(
        center.dx - textPainter.width / 2,
        center.dy - textPainter.height / 2,
      ),
    );

    // 5. Animated dots
    final dotPaint = Paint()
      ..color = Colors.orange
      ..style = PaintingStyle.fill;

    for (int i = 0; i < 8; i++) {
      final dotAngle = i * 3.14159 / 4 + progress * 2 * 3.14159 / 8;
      final dotRadius = radius + 20 + 10 * sin(progress * 2 * 3.14159 + i);
      final dotX = center.dx + dotRadius * cos(dotAngle);
      final dotY = center.dy + dotRadius * sin(dotAngle);
      
      canvas.drawCircle(
        Offset(dotX, dotY),
        4 + 2 * sin(progress * 2 * 3.14159 + i * 2),
        dotPaint,
      );
    }
  }

  @override
  bool shouldRepaint(AnimatedPainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}
```

What's happening here?
- Progress arc with animated circle
- Animating text size and color
- Rotating dots with varying sizes
- progress drives all animations

---

# Real-World Examples

> **Common patterns** with CustomPaint.

```dart
/// 1. Custom progress indicator
class CustomProgressIndicator extends StatelessWidget {
  const CustomProgressIndicator({
    super.key,
    required this.progress,
    this.size = 100,
    this.color = Colors.blue,
  });

  final double progress;
  final double size;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(size, size),
      painter: ProgressIndicatorPainter(
        progress: progress.clamp(0.0, 1.0),
        color: color,
      ),
    );
  }
}

class ProgressIndicatorPainter extends CustomPainter {
  ProgressIndicatorPainter({
    required this.progress,
    required this.color,
  });

  final double progress;
  final Color color;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - 8;

    // Background
    final bgPaint = Paint()
      ..color = Colors.grey[200]!
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8;

    canvas.drawCircle(center, radius, bgPaint);

    // Progress
    final progressPaint = Paint()
      ..color = color
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8
      ..strokeCap = StrokeCap.round;

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -3.14159 / 2,
      2 * 3.14159 * progress,
      false,
      progressPaint,
    );

    // Percentage text
    final textPainter = TextPainter(
      text: TextSpan(
        text: '${(progress * 100).round()}%',
        style: TextStyle(
          color: color,
          fontSize: 20,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );

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
  bool shouldRepaint(ProgressIndicatorPainter oldDelegate) {
    return oldDelegate.progress != progress ||
           oldDelegate.color != color;
  }
}

/// 2. Custom rating stars
class CustomRatingStars extends StatelessWidget {
  const CustomRatingStars({
    super.key,
    required this.rating,
    this.size = 30,
    this.maxStars = 5,
    this.color = Colors.yellow,
    this.backgroundColor = Colors.grey,
  });

  final double rating;
  final double size;
  final int maxStars;
  final Color color;
  final Color backgroundColor;

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(size * maxStars, size),
      painter: StarsPainter(
        rating: rating.clamp(0.0, maxStars.toDouble()),
        size: size,
        maxStars: maxStars,
        color: color,
        backgroundColor: backgroundColor,
      ),
    );
  }
}

class StarsPainter extends CustomPainter {
  StarsPainter({
    required this.rating,
    required this.size,
    required this.maxStars,
    required this.color,
    required this.backgroundColor,
  });

  final double rating;
  final double size;
  final int maxStars;
  final Color color;
  final Color backgroundColor;

  @override
  void paint(Canvas canvas, Size size) {
    final starPath = _createStarPath(size / 2);

    for (int i = 0; i < maxStars; i++) {
      final x = i * size + size / 2;
      
      // Determine how much of this star is filled
      double fillAmount = (rating - i).clamp(0.0, 1.0);

      // Draw background star
      final bgPaint = Paint()
        ..color = backgroundColor
        ..style = PaintingStyle.fill;

      canvas.save();
      canvas.translate(x, size / 2);
      canvas.drawPath(starPath, bgPaint);

      // Draw filled portion
      if (fillAmount > 0) {
        final fillPaint = Paint()
          ..color = color
          ..style = PaintingStyle.fill;

        // Clip to show partial star
        canvas.save();
        canvas.clipRect(Rect.fromLTWH(-size / 2, -size / 2, size * fillAmount, size));
        canvas.drawPath(starPath, fillPaint);
        canvas.restore();
      }

      canvas.restore();
    }
  }

  Path _createStarPath(double size) {
    const int points = 5;
    final path = Path();
    
    for (int i = 0; i < points * 2; i++) {
      final double radius = i.isEven ? size : size * 0.4;
      final double angle = i * 3.14159 / points - 3.14159 / 2;
      final double x = radius * cos(angle);
      final double y = radius * sin(angle);
      
      if (i == 0) {
        path.moveTo(x, y);
      } else {
        path.lineTo(x, y);
      }
    }
    path.close();
    return path;
  }

  @override
  bool shouldRepaint(StarsPainter oldDelegate) {
    return oldDelegate.rating != rating ||
           oldDelegate.size != size ||
           oldDelegate.maxStars != maxStars ||
           oldDelegate.color != color ||
           oldDelegate.backgroundColor != backgroundColor;
  }
}
```

What's happening here?
- Custom progress indicator with percentage
- Custom rating stars with partial fills
- Reusable custom components
- Professional-looking custom widgets

---

# Best Practices

## Use shouldRepaint Correctly

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
  // Use efficient loops
}

// Bad - Inefficient painting
@override
void paint(Canvas canvas, Size size) {
  // Complex calculations in loops
}
```

## Use const When Possible

```dart
// Good - Const values
const Rect rect = Rect.fromLTWH(0, 0, 100, 100);

// Bad - Non-const
final Rect rect = Rect.fromLTWH(0, 0, 100, 100);
```

---

# Common Mistakes

## Forgetting shouldRepaint

Wrong:
```dart
// Missing shouldRepaint
class MyPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) { ... }
  // Missing shouldRepaint - causes issues
}
```

Correct:
```dart
// Proper shouldRepaint
class MyPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) { ... }
  
  @override
  bool shouldRepaint(MyPainter oldDelegate) {
    return oldDelegate != this;
  }
}
```

## Heavy Computations in paint

Wrong:
```dart
@override
void paint(Canvas canvas, Size size) {
  // Heavy computation every frame
  final data = calculateComplexData();
}
```

Correct:
```dart
// Pre-calculate data
final data = calculateComplexData();

@override
void paint(Canvas canvas, Size size) {
  // Use pre-calculated data
}
```

---

# Summary

CustomPaint enables custom graphics and animations through canvas drawing. Use CustomPainter for custom shapes, animations, and visualizations. Optimize with proper shouldRepaint and efficient painting logic. CustomPaint is essential for unique, custom-designed visual elements.

---

# Next Steps

- [CustomPainter](custompainter.md)
- [Canvas](canvas.md)
- [RenderObject](renderobject.md)

---

# Did You Know?

- CustomPaint provides a canvas for drawing
- Canvas supports shapes, text, paths, and gradients
- shouldRepaint controls redraw frequency
- Path creates complex shapes
- Paint controls style and color
- Shader creates gradients
- TextPainter adds text to canvas
- CustomPaint is highly performant