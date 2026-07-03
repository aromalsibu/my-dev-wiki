# CustomPainter

Understand how to create custom drawings and graphics using CustomPainter in Flutter.

---

# What is it?

CustomPainter is a class that allows you to create custom drawings and graphics on a Canvas. It provides a paint method where you can use the Canvas API to draw shapes, text, images, and complex graphics. CustomPainter is used with the CustomPaint widget to render custom visual elements.

---

# Why does it exist?

CustomPainter exists to:

- Create custom graphics and visualizations
- Draw custom shapes and paths
- Implement charts and graphs
- Create custom UI elements
- Build games and animations
- Design unique visual components
- Achieve pixel-perfect custom designs

---

# Basic CustomPainter

> **Creating a simple** CustomPainter.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic CustomPainter example
class BasicCustomPainterExample extends StatelessWidget {
  const BasicCustomPainterExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('CustomPainter'),
      ),
      body: Center(
        child: CustomPaint(
          // 1. Size of the painting area
          size: const Size(300, 300),
          // 2. The custom painter that does the drawing
          painter: BasicPainter(),
        ),
      ),
    );
  }
}

/// 1. CustomPainter class
/// This is where all the drawing logic goes
class BasicPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 2. Paint objects for styling
    // Paint controls color, style, stroke width, etc.
    final fillPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    final strokePaint = Paint()
      ..color = Colors.black
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;

    // 3. Draw a rectangle
    // Rect.fromLTWH: left, top, width, height
    canvas.drawRect(
      const Rect.fromLTWH(20, 20, 80, 60),
      fillPaint,
    );
    canvas.drawRect(
      const Rect.fromLTWH(20, 20, 80, 60),
      strokePaint,
    );

    // 4. Draw a circle
    // Offset: x, y position
    canvas.drawCircle(
      const Offset(160, 50),
      30,
      fillPaint..color = Colors.green,
    );
    canvas.drawCircle(
      const Offset(160, 50),
      30,
      strokePaint,
    );

    // 5. Draw a line
    canvas.drawLine(
      const Offset(20, 120),
      const Offset(120, 120),
      Paint()
        ..color = Colors.red
        ..strokeWidth = 3,
    );

    // 6. Draw an arc
    final arcPaint = Paint()
      ..color = Colors.purple
      ..style = PaintingStyle.stroke
      ..strokeWidth = 4;

    canvas.drawArc(
      const Rect.fromLTWH(140, 100, 60, 60),
      0,
      1.5 * 3.14159,
      false,
      arcPaint,
    );

    // 7. Draw text
    final textPainter = TextPainter(
      text: const TextSpan(
        text: 'Custom Painter',
        style: TextStyle(
          color: Colors.black,
          fontSize: 20,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );

    textPainter.layout();
    textPainter.paint(
      canvas,
      const Offset(80, 200),
    );
  }

  // 8. shouldRepaint controls when the painter redraws
  // Return true to redraw, false to skip
  @override
  bool shouldRepaint(BasicPainter oldDelegate) {
    // For static painting, return false
    return false;
  }
}
```

What's happening here?
- CustomPaint provides the canvas
- CustomPainter defines what to draw
- Paint objects control styling
- Canvas methods draw shapes
- shouldRepaint controls redraws

---

# Painting with Paint Properties

> **Using different** Paint properties.

```dart
/// Paint properties example
class PaintPropertiesExample extends StatelessWidget {
  const PaintPropertiesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Paint Properties'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(400, 400),
          painter: PaintPropertiesPainter(),
        ),
      ),
    );
  }
}

class PaintPropertiesPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1. Fill style
    final fillPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    canvas.drawCircle(
      const Offset(60, 50),
      40,
      fillPaint,
    );

    // 2. Stroke style with width
    final strokePaint = Paint()
      ..color = Colors.red
      ..style = PaintingStyle.stroke
      ..strokeWidth = 4;

    canvas.drawCircle(
      const Offset(160, 50),
      40,
      strokePaint,
    );

    // 3. Stroke cap: round
    final roundCapPaint = Paint()
      ..color = Colors.green
      ..style = PaintingStyle.stroke
      ..strokeWidth = 10
      ..strokeCap = StrokeCap.round;

    canvas.drawLine(
      const Offset(220, 20),
      const Offset(220, 80),
      roundCapPaint,
    );

    // 4. Stroke join: round
    final roundJoinPaint = Paint()
      ..color = Colors.orange
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8
      ..strokeJoin = StrokeJoin.round;

    final path = Path()
      ..moveTo(260, 20)
      ..lineTo(300, 50)
      ..lineTo(260, 80);

    canvas.drawPath(path, roundJoinPaint);

    // 5. Gradient shader
    final gradientPaint = Paint()
      ..shader = const LinearGradient(
        colors: [Colors.blue, Colors.red],
        begin: Alignment.topLeft,
        end: Alignment.bottomRight,
      ).createShader(const Rect.fromLTWH(20, 110, 120, 60))
      ..style = PaintingStyle.fill;

    canvas.drawRect(
      const Rect.fromLTWH(20, 110, 120, 60),
      gradientPaint,
    );

    // 6. Radial gradient
    final radialPaint = Paint()
      ..shader = const RadialGradient(
        colors: [Colors.yellow, Colors.orange],
        center: Alignment.center,
        radius: 0.5,
      ).createShader(const Rect.fromLTWH(160, 110, 80, 80))
      ..style = PaintingStyle.fill;

    canvas.drawCircle(
      const Offset(200, 150),
      40,
      radialPaint,
    );

    // 7. Sweep gradient
    final sweepPaint = Paint()
      ..shader = const SweepGradient(
        colors: [Colors.red, Colors.blue, Colors.green, Colors.red],
        startAngle: 0,
        endAngle: 2 * 3.14159,
      ).createShader(const Rect.fromLTWH(280, 110, 80, 80))
      ..style = PaintingStyle.fill;

    canvas.drawCircle(
      const Offset(320, 150),
      40,
      sweepPaint,
    );

    // 8. Shadow
    final shadowPaint = Paint()
      ..color = Colors.purple
      ..style = PaintingStyle.fill
      ..maskFilter = const MaskFilter.blur(BlurStyle.normal, 10);

    canvas.drawCircle(
      const Offset(80, 260),
      40,
      shadowPaint,
    );

    // 9. Blend mode
    final blendPaint = Paint()
      ..color = Colors.blue.withOpacity(0.5)
      ..style = PaintingStyle.fill
      ..blendMode = BlendMode.multiply;

    canvas.drawCircle(
      const Offset(180, 260),
      40,
      blendPaint,
    );
    canvas.drawCircle(
      const Offset(220, 260),
      40,
      blendPaint,
    );
  }

  @override
  bool shouldRepaint(PaintPropertiesPainter oldDelegate) => false;
}
```

What's happening here?
- Fill and stroke styles
- StrokeCap: round, square, butt
- StrokeJoin: round, bevel, miter
- Linear, radial, and sweep gradients
- Shadows with MaskFilter
- Blend modes for compositing

---

# Path Drawing

> **Creating complex** paths with CustomPainter.

```dart
/// Path drawing example
class PathDrawingExample extends StatelessWidget {
  const PathDrawingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Path Drawing'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(400, 400),
          painter: PathPainter(),
        ),
      ),
    );
  }
}

class PathPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..style = PaintingStyle.fill
      ..color = Colors.blue;

    final strokePaint = Paint()
      ..style = PaintingStyle.stroke
      ..color = Colors.black
      ..strokeWidth = 2;

    // 1. Basic path (triangle)
    final path1 = Path()
      ..moveTo(50, 30)
      ..lineTo(100, 80)
      ..lineTo(20, 80)
      ..close();

    canvas.drawPath(path1, paint);
    canvas.drawPath(path1, strokePaint);

    // 2. Quad bezier curve
    final path2 = Path()
      ..moveTo(120, 80)
      ..quadraticBezierTo(150, 20, 180, 80);

    canvas.drawPath(
      path2,
      Paint()
        ..color = Colors.red
        ..style = PaintingStyle.stroke
        ..strokeWidth = 3,
    );

    // 3. Cubic bezier curve
    final path3 = Path()
      ..moveTo(200, 80)
      ..cubicBezierTo(220, 20, 270, 20, 290, 80);

    canvas.drawPath(
      path3,
      Paint()
        ..color = Colors.green
        ..style = PaintingStyle.stroke
        ..strokeWidth = 3,
    );

    // 4. Star shape
    final starPath = Path();
    const int points = 5;
    const double outerRadius = 40;
    const double innerRadius = 16;
    const double centerX = 70;
    const double centerY = 180;

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

    canvas.drawPath(
      starPath,
      Paint()
        ..color = Colors.orange
        ..style = PaintingStyle.fill,
    );
    canvas.drawPath(
      starPath,
      Paint()
        ..color = Colors.black
        ..style = PaintingStyle.stroke
        ..strokeWidth = 1,
    );

    // 5. Heart shape
    final heartPath = Path()
      ..moveTo(160, 220)
      ..cubicTo(130, 180, 80, 200, 160, 250)
      ..cubicTo(240, 200, 190, 180, 160, 220);

    canvas.drawPath(
      heartPath,
      Paint()
        ..color = Colors.red
        ..style = PaintingStyle.fill,
    );
    canvas.drawPath(
      heartPath,
      Paint()
        ..color = Colors.black
        ..style = PaintingStyle.stroke
        ..strokeWidth = 1,
    );

    // 6. Combined path (pacman)
    final pacmanPath = Path()
      ..moveTo(260, 200)
      ..arcTo(
        const Rect.fromLTWH(230, 170, 60, 60),
        0.2,
        1.2 * 3.14159,
        false,
      )
      ..lineTo(260, 200);

    canvas.drawPath(
      pacmanPath,
      Paint()
        ..color = Colors.yellow
        ..style = PaintingStyle.fill,
    );
  }

  @override
  bool shouldRepaint(PathPainter oldDelegate) => false;
}
```

What's happening here?
- Basic path with moveTo, lineTo, close
- Quadratic bezier curve
- Cubic bezier curve
- Star with trigonometric calculations
- Heart with cubic curves
- Arc with arcTo

---

# Animated CustomPainter

> **Animating** custom paintings.

```dart
/// Animated CustomPainter example
class AnimatedCustomPainterExample extends StatefulWidget {
  const AnimatedCustomPainterExample({super.key});

  @override
  State<AnimatedCustomPainterExample> createState() => _AnimatedCustomPainterExampleState();
}

class _AnimatedCustomPainterExampleState extends State<AnimatedCustomPainterExample>
    with SingleTickerProviderStateMixin {
  // 1. Animation controller
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
        title: const Text('Animated CustomPainter'),
      ),
      body: Center(
        child: AnimatedBuilder(
          // 2. Rebuild when animation changes
          animation: _animation,
          builder: (context, child) {
            return CustomPaint(
              size: const Size(400, 400),
              // 3. Pass animation value to painter
              painter: AnimatedPainter(
                progress: _animation.value,
              ),
            );
          },
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

    // 1. Background
    final bgPaint = Paint()
      ..color = Colors.grey[200]!
      ..style = PaintingStyle.fill;

    canvas.drawCircle(center, radius, bgPaint);

    // 2. Animated arc
    final arcPaint = Paint()
      ..shader = SweepGradient(
        colors: [Colors.blue, Colors.purple],
        startAngle: 0,
        endAngle: 2 * 3.14159,
      ).createShader(
        Rect.fromCircle(center: center, radius: radius),
      )
      ..style = PaintingStyle.stroke
      ..strokeWidth = 20
      ..strokeCap = StrokeCap.round;

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -3.14159 / 2,
      2 * 3.14159 * progress,
      false,
      arcPaint,
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
      15 + progress * 10,
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

    // 5. Animated dots around the circle
    final dotPaint = Paint()
      ..color = Colors.orange
      ..style = PaintingStyle.fill;

    for (int i = 0; i < 12; i++) {
      final dotAngle = i * 3.14159 / 6 + progress * 2 * 3.14159;
      final dotRadius = radius + 20 + 10 * sin(progress * 2 * 3.14159 + i);
      final dotX = center.dx + dotRadius * cos(dotAngle);
      final dotY = center.dy + dotRadius * sin(dotAngle);

      canvas.drawCircle(
        Offset(dotX, dotY),
        4 + 2 * sin(progress * 2 * 3.14159 + i),
        dotPaint,
      );
    }
  }

  @override
  bool shouldRepaint(AnimatedPainter oldDelegate) {
    // Redraw when progress changes
    return oldDelegate.progress != progress;
  }
}
```

What's happening here?
- Animation drives progress
- Animated arc with gradient
- Moving circle on the arc
- Dynamic text size
- Rotating dots with varying sizes

---

# Real-World Examples

> **Common patterns** with CustomPainter.

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

    // Progress arc
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
      final fillAmount = (rating - i).clamp(0.0, 1.0);

      // Background
      final bgPaint = Paint()
        ..color = backgroundColor
        ..style = PaintingStyle.fill;

      canvas.save();
      canvas.translate(x, size / 2);
      canvas.drawPath(starPath, bgPaint);

      // Filled portion
      if (fillAmount > 0) {
        final fillPaint = Paint()
          ..color = color
          ..style = PaintingStyle.fill;

        canvas.save();
        canvas.clipRect(Rect.fromLTWH(-size / 2, -size / 2, size * fillAmount, size));
        canvas.drawPath(starPath, fillPaint);
        canvas.restore();
      }

      canvas.restore();
    }
  }

  Path _createStarPath(double radius) {
    const int points = 5;
    final path = Path();

    for (int i = 0; i < points * 2; i++) {
      final double r = i.isEven ? radius : radius * 0.4;
      final double angle = i * 3.14159 / points - 3.14159 / 2;
      final double x = r * cos(angle);
      final double y = r * sin(angle);

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
- Professional custom designs

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
// Good - Pre-calculate values
@override
void paint(Canvas canvas, Size size) {
  final center = Offset(size.width / 2, size.height / 2);
  // Use center multiple times
}

// Bad - Recalculate values
@override
void paint(Canvas canvas, Size size) {
  canvas.drawCircle(Offset(size.width / 2, size.height / 2), ...);
  canvas.drawArc(Rect.fromCircle(center: Offset(size.width / 2, size.height / 2), ...));
}
```

## Use const Where Possible

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
  // Missing shouldRepaint
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

CustomPainter enables custom graphics and drawings with complete control. Use Paint for styling, Canvas for drawing, and Path for complex shapes. Animation with CustomPainter creates dynamic visual effects. CustomPainter is essential for unique custom designs.

---

# Next Steps

- [Canvas](canvas.md)
- [RenderObject](renderobject.md)
- [RenderBox](renderbox.md)

---

# Did You Know?

- CustomPainter provides full drawing control
- shouldRepaint controls redraw frequency
- Path creates complex shapes
- Paint controls style and color
- Shader creates gradients
- Canvas provides drawing methods
- CustomPainter is highly performant
- CustomPainter enables unique designs