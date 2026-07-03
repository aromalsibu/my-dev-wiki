# Canvas

Understand how to draw directly on a canvas using the Canvas API in Flutter.

---

# What is it?

Canvas is the drawing surface provided by Flutter's rendering engine. It provides a comprehensive API for drawing shapes, text, images, and complex graphics. Canvas is accessed through CustomPainter and provides low-level control over rendering. It is the foundation for all custom drawing in Flutter.

---

# Why does it exist?

Canvas exists to:

- Provide low-level drawing capabilities
- Enable custom graphics rendering
- Support complex visual effects
- Draw shapes, text, and images
- Create custom UI elements
- Implement games and visualizations
- Achieve pixel-perfect designs

---

# Basic Canvas Drawing

> **Drawing shapes** on Canvas.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic Canvas drawing example
class BasicCanvasExample extends StatelessWidget {
  const BasicCanvasExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Canvas Drawing'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(300, 300),
          painter: BasicCanvasPainter(),
        ),
      ),
    );
  }
}

/// Basic Canvas painter
class BasicCanvasPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1. Paint objects for styling
    // Paint controls color, style, stroke width, etc.
    final fillPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    final strokePaint = Paint()
      ..color = Colors.black
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;

    // 2. Draw rectangle
    // Rect defines position and size
    canvas.drawRect(
      const Rect.fromLTWH(20, 20, 80, 60),
      fillPaint,
    );
    canvas.drawRect(
      const Rect.fromLTWH(20, 20, 80, 60),
      strokePaint,
    );

    // 3. Draw circle
    // Offset is a point in 2D space
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

    // 4. Draw oval
    canvas.drawOval(
      const Rect.fromLTWH(200, 20, 60, 40),
      fillPaint..color = Colors.orange,
    );

    // 5. Draw arc
    final arcPaint = Paint()
      ..color = Colors.purple
      ..style = PaintingStyle.stroke
      ..strokeWidth = 3;

    canvas.drawArc(
      const Rect.fromLTWH(20, 100, 80, 80),
      0,
      1.5 * 3.14159,
      false,
      arcPaint,
    );

    // 6. Draw line
    canvas.drawLine(
      const Offset(120, 100),
      const Offset(180, 160),
      Paint()
        ..color = Colors.red
        ..strokeWidth = 3,
    );

    // 7. Draw path (triangle)
    final path = Path()
      ..moveTo(200, 100)
      ..lineTo(250, 160)
      ..lineTo(150, 160)
      ..close();

    canvas.drawPath(
      path,
      fillPaint..color = Colors.teal,
    );
  }

  @override
  bool shouldRepaint(BasicCanvasPainter oldDelegate) => false;
}
```

What's happening here?
- Paint objects control drawing style
- Rect defines rectangular areas
- Offset defines points in 2D space
- Canvas methods draw shapes
- Path creates complex shapes

---

# Canvas Transformations

> **Transforming** the canvas.

```dart
/// Canvas transformations example
class CanvasTransformationsExample extends StatelessWidget {
  const CanvasTransformationsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Canvas Transformations'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(400, 400),
          painter: TransformationsPainter(),
        ),
      ),
    );
  }
}

/// Transformations painter
class TransformationsPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    // 1. Translation - Moving the canvas
    // translate moves the origin point
    canvas.save();
    canvas.translate(50, 50);
    canvas.drawRect(
      const Rect.fromLTWH(0, 0, 60, 40),
      paint..color = Colors.red,
    );
    canvas.restore();

    // 2. Rotation - Rotating the canvas
    // rotate takes radians (2π = 360°)
    canvas.save();
    canvas.translate(150, 50);
    canvas.rotate(0.5); // ~30 degrees
    canvas.drawRect(
      const Rect.fromLTWH(-30, -20, 60, 40),
      paint..color = Colors.green,
    );
    canvas.restore();

    // 3. Scale - Scaling the canvas
    // scale multiplies all coordinates
    canvas.save();
    canvas.translate(250, 50);
    canvas.scale(1.5, 1.0);
    canvas.drawRect(
      const Rect.fromLTWH(0, -20, 60, 40),
      paint..color = Colors.blue,
    );
    canvas.restore();

    // 4. Skew - Shearing the canvas
    canvas.save();
    canvas.translate(50, 150);
    canvas.skew(0.5, 0);
    canvas.drawRect(
      const Rect.fromLTWH(0, 0, 60, 40),
      paint..color = Colors.purple,
    );
    canvas.restore();

    // 5. Combined transformations
    canvas.save();
    canvas.translate(200, 150);
    canvas.rotate(0.3);
    canvas.scale(1.2, 0.8);
    canvas.drawRect(
      const Rect.fromLTWH(-30, -20, 60, 40),
      paint..color = Colors.orange,
    );
    canvas.restore();

    // 6. Matrix transformation
    // Matrix4 allows full control over transformations
    final matrix = Matrix4.identity()
      ..translate(350.0, 50.0)
      ..rotateZ(0.5)
      ..scale(1.2);

    canvas.save();
    canvas.transform(matrix.storage);
    canvas.drawRect(
      const Rect.fromLTWH(-30, -20, 60, 40),
      paint..color = Colors.teal,
    );
    canvas.restore();
  }

  @override
  bool shouldRepaint(TransformationsPainter oldDelegate) => false;
}
```

What's happening here?
- translate: Moves the origin
- rotate: Rotates the canvas
- scale: Scales all coordinates
- skew: Shears the canvas
- Matrix: Full control over transformations

---

# Canvas Text Rendering

> **Drawing text** on Canvas.

```dart
/// Canvas text rendering example
class CanvasTextExample extends StatelessWidget {
  const CanvasTextExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Canvas Text'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(400, 300),
          painter: TextPainterCanvas(),
        ),
      ),
    );
  }
}

/// Text painter on canvas
class TextPainterCanvas extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1. Basic text
    final textPainter1 = TextPainter(
      text: const TextSpan(
        text: 'Hello Canvas!',
        style: TextStyle(
          color: Colors.black,
          fontSize: 24,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    textPainter1.layout();
    textPainter1.paint(
      canvas,
      const Offset(20, 20),
    );

    // 2. Text with background
    final textPainter2 = TextPainter(
      text: const TextSpan(
        text: 'Styled Text',
        style: TextStyle(
          color: Colors.white,
          fontSize: 20,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    textPainter2.layout();

    // Draw background rectangle
    final bgPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    canvas.drawRect(
      Rect.fromLTWH(
        20,
        70,
        textPainter2.width + 20,
        textPainter2.height + 20,
      ),
      bgPaint,
    );

    textPainter2.paint(
      canvas,
      const Offset(30, 80),
    );

    // 3. Text with shadow
    final shadowPainter = TextPainter(
      text: const TextSpan(
        text: 'Shadow Text',
        style: TextStyle(
          color: Colors.red,
          fontSize: 28,
          fontWeight: FontWeight.bold,
          shadows: [
            Shadow(
              color: Colors.black,
              blurRadius: 10,
              offset: Offset(2, 2),
            ),
          ],
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    shadowPainter.layout();
    shadowPainter.paint(
      canvas,
      const Offset(20, 140),
    );

    // 4. Multi-line text
    final multiLinePainter = TextPainter(
      text: const TextSpan(
        text: 'Multi-line text\nrendered on\ncanvas',
        style: TextStyle(
          color: Colors.black,
          fontSize: 18,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    multiLinePainter.layout(
      maxWidth: 150,
    );
    multiLinePainter.paint(
      canvas,
      const Offset(200, 20),
    );

    // 5. Text with gradient
    final gradient = LinearGradient(
      colors: [Colors.blue, Colors.red],
    ).createShader(
      const Rect.fromLTWH(200, 140, 200, 40),
    );

    final gradientPainter = TextPainter(
      text: TextSpan(
        text: 'Gradient Text',
        style: TextStyle(
          fontSize: 24,
          fontWeight: FontWeight.bold,
          foreground: Paint()..shader = gradient,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    gradientPainter.layout();
    gradientPainter.paint(
      canvas,
      const Offset(200, 140),
    );
  }

  @override
  bool shouldRepaint(TextPainterCanvas oldDelegate) => false;
}
```

What's happening here?
- TextPainter renders text on canvas
- Text with background rectangle
- Text with shadows and effects
- Multi-line text with maxWidth
- Gradient text using shaders

---

# Canvas Images

> **Drawing images** on Canvas.

```dart
/// Canvas images example
class CanvasImagesExample extends StatelessWidget {
  const CanvasImagesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Canvas Images'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(400, 400),
          painter: ImagesPainter(),
        ),
      ),
    );
  }
}

/// Images painter
class ImagesPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1. Draw a circle as an image placeholder
    final imagePaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    canvas.drawCircle(
      const Offset(80, 80),
      60,
      imagePaint,
    );

    // 2. Draw an icon as an image
    const iconData = Icons.image;
    final textPainter = TextPainter(
      text: TextSpan(
        text: String.fromCharCode(iconData.codePoint),
        style: TextStyle(
          fontFamily: iconData.fontFamily,
          package: iconData.fontPackage,
          fontSize: 60,
          color: Colors.white,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    textPainter.layout();
    textPainter.paint(
      canvas,
      const Offset(50, 50),
    );

    // 3. Draw a painted rectangle with gradient
    final gradientPaint = Paint()
      ..shader = const LinearGradient(
        colors: [Colors.blue, Colors.green],
      ).createShader(const Rect.fromLTWH(160, 20, 200, 120))
      ..style = PaintingStyle.fill;

    canvas.drawRect(
      const Rect.fromLTWH(160, 20, 200, 120),
      gradientPaint,
    );

    // 4. Draw an oval with border
    final ovalPaint = Paint()
      ..color = Colors.orange
      ..style = PaintingStyle.stroke
      ..strokeWidth = 4;

    canvas.drawOval(
      const Rect.fromLTWH(20, 160, 120, 80),
      ovalPaint,
    );

    // 5. Draw a rounded rectangle with shadow
    final shadowPaint = Paint()
      ..color = Colors.black.withOpacity(0.2)
      ..style = PaintingStyle.fill;

    // Shadow
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        const Rect.fromLTWH(165, 160, 180, 80),
        const Radius.circular(10),
      ),
      shadowPaint..color = Colors.black.withOpacity(0.2),
    );

    // Main shape
    final shapePaint = Paint()
      ..color = Colors.purple
      ..style = PaintingStyle.fill;

    canvas.drawRRect(
      RRect.fromRectAndRadius(
        const Rect.fromLTWH(160, 155, 180, 80),
        const Radius.circular(10),
      ),
      shapePaint,
    );

    // 6. Draw text inside the shape
    const text = 'Canvas Images';
    final text2Painter = TextPainter(
      text: const TextSpan(
        text: text,
        style: TextStyle(
          color: Colors.white,
          fontSize: 18,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    text2Painter.layout();
    text2Painter.paint(
      canvas,
      Offset(
        160 + (180 - text2Painter.width) / 2,
        155 + (80 - text2Painter.height) / 2,
      ),
    );
  }

  @override
  bool shouldRepaint(ImagesPainter oldDelegate) => false;
}
```

What's happening here?
- Icons rendered as text
- Gradients on shapes
- Shadows on shapes
- Text centered in shapes

---

# Real-World Examples

> **Common patterns** with Canvas.

```dart
/// 1. Custom chart with Canvas
class CustomChart extends StatelessWidget {
  const CustomChart({
    super.key,
    required this.data,
    this.height = 200,
    this.color = Colors.blue,
  });

  final List<double> data;
  final double height;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(300, height),
      painter: ChartPainter(
        data: data,
        color: color,
      ),
    );
  }
}

class ChartPainter extends CustomPainter {
  ChartPainter({
    required this.data,
    required this.color,
  });

  final List<double> data;
  final Color color;

  @override
  void paint(Canvas canvas, Size size) {
    if (data.isEmpty) return;

    // 1. Find max value
    final maxValue = data.reduce((a, b) => a > b ? a : b);
    if (maxValue == 0) return;

    // 2. Calculate bar dimensions
    final barWidth = size.width / data.length * 0.7;
    final spacing = size.width / data.length * 0.3;

    // 3. Draw bars
    for (int i = 0; i < data.length; i++) {
      final height = (data[i] / maxValue) * size.height * 0.9;
      final x = i * (barWidth + spacing) + spacing / 2;
      final y = size.height - height;

      // Bar with gradient
      final paint = Paint()
        ..shader = LinearGradient(
          begin: Alignment.topCenter,
          end: Alignment.bottomCenter,
          colors: [
            color.withOpacity(0.3 + 0.7 * (i / data.length)),
            color.withOpacity(0.1 + 0.7 * (i / data.length)),
          ],
        ).createShader(
          Rect.fromLTWH(x, y, barWidth, height),
        )
        ..style = PaintingStyle.fill;

      // Rounded bar
      final rrect = RRect.fromRectAndRadius(
        Rect.fromLTWH(x, y, barWidth, height),
        const Radius.circular(4),
      );
      canvas.drawRRect(rrect, paint);

      // Value on top
      final textPainter = TextPainter(
        text: TextSpan(
          text: data[i].toStringAsFixed(1),
          style: const TextStyle(
            fontSize: 10,
            color: Colors.black,
          ),
        ),
        textDirection: TextDirection.ltr,
      );
      textPainter.layout();
      textPainter.paint(
        canvas,
        Offset(
          x + barWidth / 2 - textPainter.width / 2,
          y - textPainter.height - 2,
        ),
      );
    }
  }

  @override
  bool shouldRepaint(ChartPainter oldDelegate) {
    return oldDelegate.data != data || oldDelegate.color != color;
  }
}

/// 2. Radial progress with Canvas
class RadialProgress extends StatelessWidget {
  const RadialProgress({
    super.key,
    required this.progress,
    this.size = 100,
    this.color = Colors.blue,
    this.backgroundColor = Colors.grey,
  });

  final double progress;
  final double size;
  final Color color;
  final Color backgroundColor;

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(size, size),
      painter: RadialProgressPainter(
        progress: progress.clamp(0.0, 1.0),
        color: color,
        backgroundColor: backgroundColor,
      ),
    );
  }
}

class RadialProgressPainter extends CustomPainter {
  RadialProgressPainter({
    required this.progress,
    required this.color,
    required this.backgroundColor,
  });

  final double progress;
  final Color color;
  final Color backgroundColor;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - 8;
    final strokeWidth = 8.0;

    // 1. Background circle
    final bgPaint = Paint()
      ..color = backgroundColor.withOpacity(0.3)
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth;

    canvas.drawCircle(center, radius, bgPaint);

    // 2. Progress arc with gradient
    final progressPaint = Paint()
      ..shader = SweepGradient(
        center: center,
        colors: [color.withOpacity(0.3), color],
        startAngle: 0,
        endAngle: 2 * 3.14159,
      ).createShader(
        Rect.fromCircle(center: center, radius: radius),
      )
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth
      ..strokeCap = StrokeCap.round;

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -3.14159 / 2,
      2 * 3.14159 * progress,
      false,
      progressPaint,
    );

    // 3. Percentage text
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
  bool shouldRepaint(RadialProgressPainter oldDelegate) {
    return oldDelegate.progress != progress ||
           oldDelegate.color != color ||
           oldDelegate.backgroundColor != backgroundColor;
  }
}
```

What's happening here?
- Custom bar chart with Canvas
- Radial progress indicator with gradient
- Professional-looking custom components
- Data visualization with Canvas

---

# Best Practices

## Save and Restore Canvas State

```dart
// Good - Save/restore transformations
canvas.save();
canvas.translate(100, 100);
// Draw something
canvas.restore();

// Bad - Not restoring state
canvas.translate(100, 100);
// Draw something
// Transformations accumulate
```

## Use Efficient Painting

```dart
// Good - Pre-calculate values
final center = Offset(size.width / 2, size.height / 2);
// Use center multiple times

// Bad - Recalculate values
canvas.drawCircle(Offset(size.width / 2, size.height / 2), ...);
canvas.drawArc(Rect.fromCircle(center: Offset(size.width / 2, size.height / 2), ...));
```

## Use Path for Complex Shapes

```dart
// Good - Path for complex shapes
final path = Path()
  ..moveTo(0, 0)
  ..lineTo(100, 0)
  ..quadraticBezierTo(100, 100, 0, 100)
  ..close();

// Bad - Multiple primitive calls
```

---

# Common Mistakes

## Not Using save/restore

Wrong:
```dart
// Transformations accumulate
canvas.translate(10, 10);
canvas.translate(20, 20);
// Total translation is (30, 30)
```

Correct:
```dart
// Separate transformations
canvas.save();
canvas.translate(10, 10);
// Draw
canvas.restore();

canvas.save();
canvas.translate(20, 20);
// Draw
canvas.restore();
```

## Forgetting Layout for Text

Wrong:
```dart
// Text not laid out
final textPainter = TextPainter(text: ...);
textPainter.paint(canvas, offset); // Error
```

Correct:
```dart
// Layout before painting
final textPainter = TextPainter(text: ...);
textPainter.layout();
textPainter.paint(canvas, offset);
```

---

# Summary

Canvas provides low-level drawing capabilities for custom graphics. Use Canvas methods for shapes, transformations, text, and images. Combine with Paint for styling, Path for complex shapes, and TextPainter for text rendering. Canvas is essential for custom graphics and visualizations.

---

# Next Steps

- [RenderObject](renderobject.md)
- [RenderBox](renderbox.md)
- [Layers](layers.md)

---

# Did You Know?

- Canvas provides 2D drawing API
- save/restore manage transformations
- Path creates complex shapes
- TextPainter renders text
- Paint controls drawing style
- Shader creates gradients
- Canvas supports images and icons
- Canvas is the foundation of Flutter rendering