# InkWell

Understand how to create interactive material design touch effects with InkWell.

---

# What is it?

InkWell is a Material Design widget that provides a splash effect when tapped or interacted with. It's a specialized version of GestureDetector that adds visual feedback through ink splashes, ripples, and highlights. InkWell is the standard way to make widgets interactive in Material Design applications.

---

# Why does it exist?

InkWell exists to:

- Provide Material Design touch feedback
- Create splash and ripple effects
- Make widgets interactive
- Support Material Design guidelines
- Enhance user experience with visual feedback
- Integrate with Material themes
- Provide accessibility features

---

# Basic InkWell

> **Creating interactive** widgets with InkWell.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic InkWell example
class BasicInkWellExample extends StatefulWidget {
  const BasicInkWellExample({super.key});

  @override
  State<BasicInkWellExample> createState() => _BasicInkWellExampleState();
}

class _BasicInkWellExampleState extends State<BasicInkWellExample> {
  // 1. State for tracking interactions
  String _lastAction = 'None';
  int _tapCount = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('InkWell'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. InkWell wrapping a Container
            // InkWell provides material splash effects
            InkWell(
              // 3. Tap gesture
              onTap: () {
                setState(() {
                  _lastAction = 'Tap';
                  _tapCount++;
                });
              },
              
              // 4. Double tap gesture
              onDoubleTap: () {
                setState(() {
                  _lastAction = 'Double Tap';
                  _tapCount++;
                });
              },
              
              // 5. Long press gesture
              onLongPress: () {
                setState(() {
                  _lastAction = 'Long Press';
                  _tapCount++;
                });
              },
              
              // 6. Custom splash color
              splashColor: Colors.blue[100],
              
              // 7. Custom highlight color
              highlightColor: Colors.blue[50],
              
              // 8. Radius of the splash effect
              radius: 30,
              
              // 9. Child widget
              child: Container(
                width: double.infinity,
                height: 150,
                color: Colors.blue,
                child: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Text(
                        'Tap Me!',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 8),
                      Text(
                        'Last action: $_lastAction',
                        style: const TextStyle(
                          color: Colors.white70,
                          fontSize: 16,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 10. InkWell with custom shape
            InkWell(
              onTap: () {
                setState(() {
                  _lastAction = 'Rounded Tap';
                  _tapCount++;
                });
              },
              // Custom shape with borderRadius
              borderRadius: BorderRadius.circular(16),
              child: Container(
                width: double.infinity,
                height: 100,
                decoration: BoxDecoration(
                  color: Colors.green,
                  borderRadius: BorderRadius.circular(16),
                ),
                child: const Center(
                  child: Text(
                    'Rounded InkWell',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 11. Interaction statistics
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                children: [
                  Column(
                    children: [
                      const Text('Taps', style: TextStyle(fontWeight: FontWeight.bold)),
                      Text('$_tapCount'),
                    ],
                  ),
                  Column(
                    children: [
                      const Text('Last Action', style: TextStyle(fontWeight: FontWeight.bold)),
                      Text(_lastAction),
                    ],
                  ),
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
- InkWell provides splash effects
- onTap, onDoubleTap, onLongPress
- splashColor and highlightColor customize effects
- borderRadius creates rounded splashes
- Material Design touch feedback

---

# InkWell vs GestureDetector

> **Comparing** InkWell with GestureDetector.

```dart
/// InkWell vs GestureDetector comparison
class InkWellVsGestureDetector extends StatelessWidget {
  const InkWellVsGestureDetector({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('InkWell vs GestureDetector'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. InkWell with Material Design effects
            // Provides splash and ripple effects
            const Text(
              'InkWell (Material Design)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            InkWell(
              onTap: () {
                print('InkWell tapped');
              },
              // Custom splash color
              splashColor: Colors.blue[100],
              // Custom highlight color
              highlightColor: Colors.blue[50],
              child: Container(
                width: double.infinity,
                height: 80,
                color: Colors.blue,
                child: const Center(
                  child: Text(
                    'InkWell',
                    style: TextStyle(color: Colors.white, fontSize: 18),
                  ),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 2. GestureDetector without Material effects
            // No splash or ripple effects
            const Text(
              'GestureDetector (No Effects)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            GestureDetector(
              onTap: () {
                print('GestureDetector tapped');
              },
              child: Container(
                width: double.infinity,
                height: 80,
                color: Colors.green,
                child: const Center(
                  child: Text(
                    'GestureDetector',
                    style: TextStyle(color: Colors.white, fontSize: 18),
                  ),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 3. InkWell with custom widget
            // InkWell only works with Material widgets
            const Text(
              'InkWell with Container (Works)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            InkWell(
              onTap: () {
                print('Container tapped');
              },
              child: Container(
                width: double.infinity,
                height: 80,
                color: Colors.purple,
                child: const Center(
                  child: Text(
                    'Container',
                    style: TextStyle(color: Colors.white, fontSize: 18),
                  ),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 4. InkWell with Ink
            // Ink is the preferred way with Material widgets
            const Text(
              'InkWell with Ink (Preferred)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Ink(
              color: Colors.orange,
              child: InkWell(
                onTap: () {
                  print('Ink tapped');
                },
                child: Container(
                  width: double.infinity,
                  height: 80,
                  child: const Center(
                    child: Text(
                      'Ink + InkWell',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 18,
                      ),
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// When to use:
// InkWell: Material Design apps, need splash effects
// GestureDetector: Custom effects, non-Material apps
// Ink + InkWell: Best practice for Material widgets
```

What's happening here?
- InkWell: Material splash effects
- GestureDetector: No visual effects
- Ink + InkWell: Best practice
- Choice depends on design needs

---

# InkWell with Different Shapes

> **Custom shapes** with InkWell.

```dart
/// InkWell with custom shapes
class InkWellShapesExample extends StatelessWidget {
  const InkWellShapesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('InkWell Shapes'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Wrap(
          spacing: 16,
          runSpacing: 16,
          children: [
            // 1. Circular InkWell
            _buildCircularInkWell(),
            
            // 2. Rounded InkWell
            _buildRoundedInkWell(),
            
            // 3. Oval InkWell
            _buildOvalInkWell(),
            
            // 4. Custom shape InkWell
            _buildCustomShapeInkWell(),
          ],
        ),
      ),
    );
  }

  Widget _buildCircularInkWell() {
    return InkWell(
      onTap: () {
        print('Circular tapped');
      },
      // 1. Circular shape
      customBorder: const CircleBorder(),
      child: Container(
        width: 100,
        height: 100,
        decoration: const BoxDecoration(
          color: Colors.blue,
          shape: BoxShape.circle,
        ),
        child: const Center(
          child: Text(
            'Circle',
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }

  Widget _buildRoundedInkWell() {
    return InkWell(
      onTap: () {
        print('Rounded tapped');
      },
      // 2. Rounded rectangle
      borderRadius: BorderRadius.circular(16),
      child: Container(
        width: 100,
        height: 100,
        decoration: BoxDecoration(
          color: Colors.green,
          borderRadius: BorderRadius.circular(16),
        ),
        child: const Center(
          child: Text(
            'Rounded',
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }

  Widget _buildOvalInkWell() {
    return InkWell(
      onTap: () {
        print('Oval tapped');
      },
      // 3. Oval shape
      customBorder: const StadiumBorder(),
      child: Container(
        width: 140,
        height: 70,
        decoration: BoxDecoration(
          color: Colors.purple,
          borderRadius: BorderRadius.circular(35),
        ),
        child: const Center(
          child: Text(
            'Oval',
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }

  Widget _buildCustomShapeInkWell() {
    // 4. Custom shape path
    final path = Path()
      ..moveTo(0, 50)
      ..lineTo(50, 0)
      ..lineTo(100, 50)
      ..lineTo(50, 100)
      ..close();

    return InkWell(
      onTap: () {
        print('Custom tapped');
      },
      customBorder: ShapeBorder.fromPath(path),
      child: Container(
        width: 100,
        height: 100,
        color: Colors.orange,
        child: const Center(
          child: Text(
            'Custom',
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- CircleBorder: Circular shape
- StadiumBorder: Oval shape
- borderRadius: Rounded corners
- customBorder: Custom shapes

---

# Real-World Examples

> **Common patterns** with InkWell.

```dart
/// 1. Interactive card
class InteractiveCard extends StatelessWidget {
  const InteractiveCard({
    super.key,
    required this.title,
    required this.subtitle,
    required this.icon,
    required this.onTap,
  });

  final String title;
  final String subtitle;
  final IconData icon;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      // 1. Splash effect on card
      splashColor: Colors.blue[100],
      borderRadius: BorderRadius.circular(12),
      child: Container(
        margin: const EdgeInsets.all(8),
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(12),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.1),
              blurRadius: 8,
              offset: const Offset(0, 2),
            ),
          ],
        ),
        child: Row(
          children: [
            // Icon
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Icon(icon, color: Colors.blue),
            ),
            const SizedBox(width: 16),
            
            // Text
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    title,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    subtitle,
                    style: TextStyle(
                      color: Colors.grey[600],
                      fontSize: 14,
                    ),
                  ),
                ],
              ),
            ),
            
            // Arrow
            const Icon(Icons.arrow_forward_ios, size: 16),
          ],
        ),
      ),
    );
  }
}

/// 2. Navigation button
class NavigationButton extends StatelessWidget {
  const NavigationButton({
    super.key,
    required this.label,
    required this.icon,
    required this.onTap,
    this.color = Colors.blue,
  });

  final String label;
  final IconData icon;
  final VoidCallback onTap;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      // 2. Custom splash colors
      splashColor: color.withOpacity(0.2),
      highlightColor: color.withOpacity(0.1),
      borderRadius: BorderRadius.circular(8),
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
        decoration: BoxDecoration(
          color: color,
          borderRadius: BorderRadius.circular(8),
        ),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(icon, color: Colors.white),
            const SizedBox(width: 8),
            Text(
              label,
              style: const TextStyle(
                color: Colors.white,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// 3. Animated InkWell
class AnimatedInkWell extends StatefulWidget {
  const AnimatedInkWell({
    super.key,
    required this.child,
    required this.onTap,
  });

  final Widget child;
  final VoidCallback onTap;

  @override
  State<AnimatedInkWell> createState() => _AnimatedInkWellState();
}

class _AnimatedInkWellState extends State<AnimatedInkWell> {
  bool _isPressed = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) {
        setState(() => _isPressed = true);
      },
      onTapUp: (_) {
        setState(() => _isPressed = false);
        widget.onTap();
      },
      onTapCancel: () {
        setState(() => _isPressed = false);
      },
      child: AnimatedContainer(
        duration: const Duration(milliseconds: 200),
        transform: Matrix4.identity()
          ..scale(_isPressed ? 0.95 : 1.0),
        child: widget.child,
      ),
    );
  }
}
```

What's happening here?
- Interactive card with InkWell
- Navigation button with custom colors
- Animated InkWell with scale effect
- Real-world interactive components

---

# Best Practices

## Use InkWell with Material Widgets

```dart
// Good - InkWell with Container
InkWell(
  onTap: () {},
  child: Container(...),
)

// Good - Ink + InkWell (preferred)
Ink(
  color: Colors.blue,
  child: InkWell(
    onTap: () {},
    child: Container(...),
  ),
)
```

## Customize Splash Effects

```dart
// Good - Custom splash
InkWell(
  splashColor: Colors.blue[100],
  highlightColor: Colors.blue[50],
  radius: 30,
  onTap: () {},
  child: ...,
)
```

## Handle Different Shapes

```dart
// Good - Shape-specific InkWell
InkWell(
  customBorder: const CircleBorder(),
  onTap: () {},
  child: Container(
    decoration: BoxDecoration(
      shape: BoxShape.circle,
      color: Colors.blue,
    ),
  ),
)
```

---

# Common Mistakes

## Using InkWell Without Material

Wrong:
```dart
// No Material ancestor
Scaffold(
  body: InkWell(
    onTap: () {},
    child: Container(...),
  ),
)
```

Correct:
```dart
// InkWell requires Material ancestor
Scaffold(
  body: Material(
    child: InkWell(
      onTap: () {},
      child: Container(...),
    ),
  ),
)
```

## Not Using Ink with Color

Wrong:
```dart
// Color set on Container prevents splash
InkWell(
  onTap: () {},
  child: Container(
    color: Colors.blue, // Splash not visible
  ),
)
```

Correct:
```dart
// Use Ink for color
Ink(
  color: Colors.blue,
  child: InkWell(
    onTap: () {},
    child: Container(),
  ),
)
```

---

# Summary

InkWell provides Material Design touch feedback with splash and ripple effects. Use InkWell for interactive widgets in Material apps, customize splash colors and shapes, and combine with Ink for best results. InkWell is essential for creating touch-friendly Material Design interfaces.

---

# Next Steps

- [Pointer Events](pointer-events.md)
- [Drag & Drop](drag-drop.md)
- [Hit Testing](hit-testing.md)

---

# Did You Know?

- InkWell provides Material splash effects
- splashColor customizes the splash
- customBorder creates custom shapes
- Ink + InkWell is best practice
- InkWell requires Material ancestor
- InkWell supports tap, double-tap, long-press
- borderRadius creates rounded splashes
- InkWell is preferred over GestureDetector in Material apps