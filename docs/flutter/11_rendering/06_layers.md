# Layers

Understand how Flutter handles compositing and layers in the rendering pipeline.

---

# What is it?

Layers are the building blocks of Flutter's compositing system. They represent visual elements that can be combined, transformed, and rendered efficiently. Each layer contains a set of drawing commands or child layers that are composited together to create the final image displayed on screen.

---

# Why does it exist?

Layers exist to:

- Enable efficient compositing
- Support visual effects and transformations
- Handle rendering optimization
- Manage z-ordering and layering
- Support opacity and clipping
- Enable complex UI structures
- Provide smooth animations

---

# Understanding Layers

> **The layer system** in Flutter.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

/// Layer system example
class LayerExample extends StatelessWidget {
  const LayerExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Layers'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(300, 300),
          painter: LayerPainter(),
        ),
      ),
    );
  }
}

/// Custom painter demonstrating layer concepts
class LayerPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1. Create layers by saving and restoring canvas state
    // Each save/restore creates a layer

    // Layer 1: Background
    canvas.save();
    final bgPaint = Paint()
      ..color = Colors.grey[200]!
      ..style = PaintingStyle.fill;
    canvas.drawRect(Rect.fromLTWH(0, 0, size.width, size.height), bgPaint);
    canvas.restore();

    // Layer 2: Circle with shadow
    canvas.save();
    final shadowPaint = Paint()
      ..color = Colors.black.withOpacity(0.2)
      ..maskFilter = const MaskFilter.blur(BlurStyle.normal, 10);
    canvas.drawCircle(const Offset(100, 100), 60, shadowPaint);

    final circlePaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;
    canvas.drawCircle(const Offset(100, 100), 60, circlePaint);
    canvas.restore();

    // Layer 3: Rectangle with opacity
    canvas.save();
    final rectPaint = Paint()
      ..color = Colors.red.withOpacity(0.5)
      ..style = PaintingStyle.fill;
    canvas.drawRect(
      const Rect.fromLTWH(150, 50, 80, 80),
      rectPaint,
    );
    canvas.restore();

    // Layer 4: Text
    canvas.save();
    final textPainter = TextPainter(
      text: const TextSpan(
        text: 'Layers',
        style: TextStyle(
          color: Colors.white,
          fontSize: 24,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    textPainter.layout();
    textPainter.paint(
      canvas,
      const Offset(100, 200),
    );
    canvas.restore();
  }

  @override
  bool shouldRepaint(LayerPainter oldDelegate) => false;
}
```

What's happening here?
- Layers are created with canvas.save/restore
- Each layer can have independent transformations
- Layers are composited in order
- Layers support opacity, shadows, and effects

---

# Layer Types

> **Different types** of layers in Flutter.

```dart
/// Layer types example
class LayerTypesExample extends StatelessWidget {
  const LayerTypesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Layer Types'),
      ),
      body: Center(
        child: Container(
          width: 300,
          height: 300,
          color: Colors.grey[200],
          child: Stack(
            children: [
              // 1. PictureLayer - Contains drawing commands
              // This is the most common layer type
              CustomPaint(
                painter: PictureLayerPainter(),
              ),

              // 2. OpacityLayer - Applies opacity to child
              Opacity(
                opacity: 0.7,
                child: Container(
                  color: Colors.red,
                  width: 100,
                  height: 100,
                ),
              ),

              // 3. TransformLayer - Applies transformation
              Transform.rotate(
                angle: 0.5,
                child: Container(
                  width: 80,
                  height: 80,
                  color: Colors.green,
                ),
              ),

              // 4. ClipLayer - Clips child content
              ClipRRect(
                borderRadius: BorderRadius.circular(20),
                child: Container(
                  width: 100,
                  height: 100,
                  color: Colors.blue,
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

/// PictureLayer painter
class PictureLayerPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.purple
      ..style = PaintingStyle.fill;

    canvas.drawRect(
      Rect.fromLTWH(20, 20, 60, 60),
      paint,
    );
  }

  @override
  bool shouldRepaint(PictureLayerPainter oldDelegate) => false;
}

/// Custom layer implementation
class CustomLayerWidget extends SingleChildRenderObjectWidget {
  const CustomLayerWidget({
    super.key,
    required super.child,
    this.opacity = 1.0,
  });

  final double opacity;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomLayerRenderObject(opacity: opacity);
  }

  @override
  void updateRenderObject(
    BuildContext context,
    CustomLayerRenderObject renderObject,
  ) {
    renderObject.opacity = opacity;
  }
}

/// RenderObject with custom layer
class CustomLayerRenderObject extends RenderProxyBox {
  CustomLayerRenderObject({required double opacity})
      : _opacity = opacity;

  double _opacity;
  set opacity(double value) {
    if (_opacity == value) return;
    _opacity = value;
    markNeedsPaint();
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    if (_opacity < 1.0) {
      // Create an opacity layer
      context.pushOpacity(
        offset,
        (_opacity * 255).round(),
        super.paint,
        oldLayer: null,
      );
    } else {
      super.paint(context, offset);
    }
  }
}
```

What's happening here?
- PictureLayer: Contains drawing commands
- OpacityLayer: Applies transparency
- TransformLayer: Applies transformations
- ClipLayer: Clips content
- Custom layers with RenderObject

---

# Layer Composition

> **How layers** are composed together.

```dart
/// Layer composition example
class LayerCompositionExample extends StatelessWidget {
  const LayerCompositionExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Layer Composition'),
      ),
      body: Center(
        child: Container(
          width: 300,
          height: 300,
          color: Colors.grey[200],
          child: Stack(
            children: [
              // 1. Background layer
              Container(
                color: Colors.blue,
                width: double.infinity,
                height: double.infinity,
              ),

              // 2. Middle layer with opacity
              Positioned(
                left: 50,
                top: 50,
                child: Opacity(
                  opacity: 0.7,
                  child: Container(
                    width: 100,
                    height: 100,
                    color: Colors.red,
                  ),
                ),
              ),

              // 3. Top layer with transform
              Positioned(
                left: 120,
                top: 120,
                child: Transform.rotate(
                  angle: 0.3,
                  child: Container(
                    width: 80,
                    height: 80,
                    color: Colors.green,
                  ),
                ),
              ),

              // 4. Layer with shadow
              Positioned(
                left: 180,
                top: 50,
                child: Container(
                  width: 80,
                  height: 80,
                  decoration: BoxDecoration(
                    color: Colors.orange,
                    boxShadow: [
                      BoxShadow(
                        color: Colors.black.withOpacity(0.3),
                        blurRadius: 10,
                        spreadRadius: 2,
                      ),
                    ],
                  ),
                ),
              ),

              // 5. Text layer on top
              const Positioned(
                left: 50,
                top: 200,
                child: Text(
                  'Layer Composition',
                  style: TextStyle(
                    fontSize: 20,
                    fontWeight: FontWeight.bold,
                    color: Colors.white,
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Layers are composed in order
- Later layers appear on top
- Each layer can have effects
- Layers combine to form the final image

---

# Real-World Examples

> **Common patterns** with layers.

```dart
/// 1. Gradient overlay layer
class GradientOverlayExample extends StatelessWidget {
  const GradientOverlayExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Gradient Overlay'),
      ),
      body: Container(
        width: double.infinity,
        height: 300,
        child: Stack(
          children: [
            // Background image
            Container(
              decoration: const BoxDecoration(
                image: DecorationImage(
                  image: NetworkImage('https://example.com/image.jpg'),
                  fit: BoxFit.cover,
                ),
              ),
            ),
            // Gradient overlay layer
            Container(
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  begin: Alignment.topCenter,
                  end: Alignment.bottomCenter,
                  colors: [
                    Colors.transparent,
                    Colors.black.withOpacity(0.7),
                  ],
                ),
              ),
            ),
            // Text layer on top
            const Positioned(
              bottom: 20,
              left: 20,
              right: 20,
              child: Text(
                'Overlay Text',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// 2. Custom animated layer
class AnimatedLayerExample extends StatefulWidget {
  const AnimatedLayerExample({super.key});

  @override
  State<AnimatedLayerExample> createState() => _AnimatedLayerExampleState();
}

class _AnimatedLayerExampleState extends State<AnimatedLayerExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
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
    return Scaffold(
      appBar: AppBar(
        title: const Text('Animated Layer'),
      ),
      body: Center(
        child: AnimatedBuilder(
          animation: _animation,
          builder: (context, child) {
            return Container(
              width: 200,
              height: 200,
              color: Colors.grey[200],
              child: Stack(
                children: [
                  // Animated position layer
                  Positioned(
                    left: _animation.value * 100,
                    top: _animation.value * 100,
                    child: Container(
                      width: 60,
                      height: 60,
                      color: Colors.blue,
                    ),
                  ),
                  // Animated opacity layer
                  Positioned(
                    left: 100,
                    top: 0,
                    child: Opacity(
                      opacity: 0.5 + 0.5 * _animation.value,
                      child: Container(
                        width: 60,
                        height: 60,
                        color: Colors.red,
                      ),
                    ),
                  ),
                  // Animated rotation layer
                  Positioned(
                    left: 0,
                    top: 100,
                    child: Transform.rotate(
                      angle: _animation.value * 2 * 3.14159,
                      child: Container(
                        width: 60,
                        height: 60,
                        color: Colors.green,
                      ),
                    ),
                  ),
                ],
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
- Gradient overlay for images
- Multiple layers in Stack
- Animated layers with transformations
- Opacity and position animations

---

# Best Practices

## Use Layers for Compositing

```dart
// Good - Use layers for effects
Stack(
  children: [
    BackgroundWidget(),
    OpacityLayer(),
    TransformLayer(),
  ],
)
```

## Use save/restore for Canvas Layers

```dart
// Good - Save/restore canvas state
canvas.save();
// Draw layer content
canvas.restore();
```

## Use Appropriate Layer Types

```dart
// Good - Right layer for the job
OpacityLayer -> For transparency
TransformLayer -> For transformations
ClipLayer -> For clipping
```

---

# Common Mistakes

## Too Many Layers

Wrong:
```dart
// Excessive layering
Stack(
  children: [
    Container(),
    Container(),
    Container(),
    // Too many layers
  ],
)
```

Correct:
```dart
// Efficient layering
Stack(
  children: [
    Container(),
    // Only necessary layers
  ],
)
```

## Not Using save/restore

Wrong:
```dart
// Canvas state not restored
canvas.translate(10, 10);
// Drawing...
// Transformations accumulate
```

Correct:
```dart
// With save/restore
canvas.save();
canvas.translate(10, 10);
// Drawing...
canvas.restore();
```

---

# Summary

Layers are the building blocks of compositing in Flutter. They enable visual effects, transformations, and efficient rendering. Use different layer types for opacity, transformations, clipping, and custom effects. Layers combine to create the final visual output.

---

# Next Steps

- [Compositing](compositing.md)
- [RenderObject](renderobject.md)
- [RenderBox](renderbox.md)

---

# Did You Know?

- Layers are composited in order
- PictureLayer contains drawing commands
- OpacityLayer applies transparency
- TransformLayer applies transformations
- ClipLayer clips content
- save/restore creates layers
- Layers enable efficient rendering
- Layers support animations