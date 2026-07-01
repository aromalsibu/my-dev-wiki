# Implicit Animations

Understand how to create smooth animations with minimal code using Implicit Animations in Flutter.

---

# What is it?

Implicit Animations are a type of animation in Flutter that automatically animates changes to a widget's properties over time. When you change a property like width, height, color, or opacity, the widget smoothly transitions from the old value to the new value. Implicit animations are the simplest way to add animations to your Flutter app, requiring only a single widget and minimal code.

---

# Why does it exist?

Implicit Animations exist to:

- Simplify animation implementation
- Reduce boilerplate code
- Provide smooth transitions automatically
- Handle common animation scenarios
- Improve user experience with minimal effort
- Support hot reload and development
- Enable rapid prototyping

---

# Basic Implicit Animations

> **Simple animations** with AnimatedContainer.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic implicit animation example
class BasicImplicitAnimationExample extends StatefulWidget {
  const BasicImplicitAnimationExample({super.key});

  @override
  State<BasicImplicitAnimationExample> createState() => _BasicImplicitAnimationExampleState();
}

class _BasicImplicitAnimationExampleState extends State<BasicImplicitAnimationExample> {
  // State variables for animation properties
  double _width = 100;
  double _height = 100;
  Color _color = Colors.blue;
  double _borderRadius = 8;

  // Method to toggle animation properties
  void _toggleProperties() {
    setState(() {
      // When these values change, AnimatedContainer will animate between old and new values
      _width = _width == 100 ? 200 : 100;
      _height = _height == 100 ? 200 : 100;
      _color = _color == Colors.blue ? Colors.red : Colors.blue;
      _borderRadius = _borderRadius == 8 ? 50 : 8;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('AnimatedContainer'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. AnimatedContainer - Automatically animates property changes
            // When any property changes, it smoothly transitions to the new value
            AnimatedContainer(
              // Duration of the animation
              duration: const Duration(milliseconds: 500),
              
              // Curve for the animation (easing function)
              curve: Curves.easeInOut,
              
              // Animated properties
              width: _width,
              height: _height,
              decoration: BoxDecoration(
                color: _color,
                borderRadius: BorderRadius.circular(_borderRadius),
              ),
              child: const Center(
                child: Text(
                  'Tap Button',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 2. Button to trigger animation
            ElevatedButton(
              onPressed: _toggleProperties,
              child: const Text('Animate'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- AnimatedContainer animates property changes
- duration controls animation speed
- curve controls easing (easeInOut, linear, bounceOut, etc.)
- State changes trigger animations automatically

---

# Common Implicit Animation Widgets

> **Different implicit** animation widgets.

```dart
/// Common implicit animation widgets
class CommonImplicitAnimationsExample extends StatefulWidget {
  const CommonImplicitAnimationsExample({super.key});

  @override
  State<CommonImplicitAnimationsExample> createState() => _CommonImplicitAnimationsExampleState();
}

class _CommonImplicitAnimationsExampleState extends State<CommonImplicitAnimationsExample> {
  // State variables for different animations
  bool _isExpanded = false;
  double _opacity = 0.5;
  double _fontSize = 16;
  Alignment _alignment = Alignment.center;
  double _rotationAngle = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Implicit Animations'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. AnimatedContainer - Animates multiple properties
            // This can animate width, height, color, padding, margin, decoration, etc.
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'AnimatedContainer',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    AnimatedContainer(
                      width: _isExpanded ? 200 : 100,
                      height: _isExpanded ? 200 : 100,
                      duration: const Duration(milliseconds: 500),
                      curve: Curves.easeInOut,
                      decoration: BoxDecoration(
                        color: _isExpanded ? Colors.blue : Colors.green,
                        borderRadius: BorderRadius.circular(_isExpanded ? 20 : 8),
                      ),
                      child: const Center(
                        child: Text(
                          'Expand',
                          style: TextStyle(color: Colors.white),
                        ),
                      ),
                    ),
                    const SizedBox(height: 8),
                    ElevatedButton(
                      onPressed: () {
                        setState(() {
                          _isExpanded = !_isExpanded;
                        });
                      },
                      child: const Text('Toggle'),
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 2. AnimatedOpacity - Animates opacity changes
            // Perfect for fade in/out effects
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'AnimatedOpacity',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    AnimatedOpacity(
                      // Opacity value: 0.0 = invisible, 1.0 = fully visible
                      opacity: _opacity,
                      duration: const Duration(milliseconds: 500),
                      curve: Curves.easeInOut,
                      child: Container(
                        width: 100,
                        height: 100,
                        color: Colors.red,
                        child: const Center(
                          child: Text(
                            'Fade',
                            style: TextStyle(color: Colors.white),
                          ),
                        ),
                      ),
                    ),
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _opacity = 0.0; // Fade out
                            });
                          },
                          child: const Text('Fade Out'),
                        ),
                        const SizedBox(width: 8),
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _opacity = 1.0; // Fade in
                            });
                          },
                          child: const Text('Fade In'),
                        ),
                        const SizedBox(width: 8),
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _opacity = _opacity == 1.0 ? 0.5 : 1.0;
                            });
                          },
                          child: const Text('Toggle'),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 3. AnimatedAlign - Animates alignment changes
            // Perfect for moving widgets around
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'AnimatedAlign',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    Container(
                      width: 200,
                      height: 100,
                      color: Colors.grey[200],
                      child: AnimatedAlign(
                        // Alignment values: topLeft, center, bottomRight, etc.
                        alignment: _alignment,
                        duration: const Duration(milliseconds: 500),
                        curve: Curves.easeInOut,
                        child: Container(
                          width: 40,
                          height: 40,
                          color: Colors.orange,
                          child: const Center(
                            child: Text(
                              'Move',
                              style: TextStyle(
                                color: Colors.white,
                                fontSize: 10,
                              ),
                            ),
                          ),
                        ),
                      ),
                    ),
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _alignment = Alignment.topLeft;
                            });
                          },
                          child: const Text('Top Left'),
                        ),
                        const SizedBox(width: 8),
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _alignment = Alignment.center;
                            });
                          },
                          child: const Text('Center'),
                        ),
                        const SizedBox(width: 8),
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _alignment = Alignment.bottomRight;
                            });
                          },
                          child: const Text('Bottom Right'),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 4. AnimatedDefaultTextStyle - Animates text style changes
            // Perfect for animating font size, color, weight, etc.
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'AnimatedDefaultTextStyle',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    AnimatedDefaultTextStyle(
                      style: TextStyle(
                        fontSize: _fontSize,
                        color: _fontSize > 20 ? Colors.red : Colors.black,
                        fontWeight: _fontSize > 20 ? FontWeight.bold : FontWeight.normal,
                      ),
                      duration: const Duration(milliseconds: 500),
                      curve: Curves.easeInOut,
                      child: const Text('Text Style Animation'),
                    ),
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _fontSize = 16;
                            });
                          },
                          child: const Text('Small'),
                        ),
                        const SizedBox(width: 8),
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _fontSize = 32;
                            });
                          },
                          child: const Text('Large'),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 5. AnimatedRotation - Animates rotation
            // Perfect for spin animations
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'AnimatedRotation',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    AnimatedRotation(
                      // Rotation angle in radians (2π = full rotation)
                      turns: _rotationAngle,
                      duration: const Duration(milliseconds: 500),
                      curve: Curves.easeInOut,
                      child: Container(
                        width: 60,
                        height: 60,
                        color: Colors.purple,
                        child: const Center(
                          child: Text(
                            'Spin',
                            style: TextStyle(color: Colors.white),
                          ),
                        ),
                      ),
                    ),
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _rotationAngle += 0.25; // 90 degrees
                            });
                          },
                          child: const Text('Rotate 90°'),
                        ),
                        const SizedBox(width: 8),
                        ElevatedButton(
                          onPressed: () {
                            setState(() {
                              _rotationAngle = 0; // Reset
                            });
                          },
                          child: const Text('Reset'),
                        ),
                      ],
                    ),
                  ],
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
- AnimatedOpacity: Fade in/out
- AnimatedAlign: Move widgets
- AnimatedDefaultTextStyle: Text style changes
- AnimatedRotation: Rotate widgets
- Each animates specific properties

---

# AnimatedPositioned

> **Animating positioning** in Stack.

```dart
/// AnimatedPositioned example
class AnimatedPositionedExample extends StatefulWidget {
  const AnimatedPositionedExample({super.key});

  @override
  State<AnimatedPositionedExample> createState() => _AnimatedPositionedExampleState();
}

class _AnimatedPositionedExampleState extends State<AnimatedPositionedExample> {
  // Position variables
  double _left = 20;
  double _top = 20;
  double _right = 20;
  double _bottom = 20;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('AnimatedPositioned'),
      ),
      body: Stack(
        children: [
          // AnimatedPositioned - Animates position changes in Stack
          // Perfect for moving widgets around the screen
          AnimatedPositioned(
            left: _left,
            top: _top,
            right: _right,
            bottom: _bottom,
            duration: const Duration(milliseconds: 500),
            curve: Curves.easeInOut,
            child: Container(
              width: 80,
              height: 80,
              color: Colors.blue,
              child: const Center(
                child: Text(
                  'Move',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
          
          // Control buttons at the bottom
          Positioned(
            bottom: 20,
            left: 0,
            right: 0,
            child: Column(
              children: [
                Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    // Top left position
                    ElevatedButton(
                      onPressed: () {
                        setState(() {
                          _left = 20;
                          _top = 20;
                          _right = 0;
                          _bottom = 0;
                        });
                      },
                      child: const Text('Top Left'),
                    ),
                    const SizedBox(width: 8),
                    
                    // Center position
                    ElevatedButton(
                      onPressed: () {
                        setState(() {
                          final screenWidth = MediaQuery.of(context).size.width;
                          final screenHeight = MediaQuery.of(context).size.height;
                          
                          // Center the widget
                          _left = screenWidth / 2 - 40;
                          _top = screenHeight / 2 - 40 - 100; // Offset for buttons
                          _right = 0;
                          _bottom = 0;
                        });
                      },
                      child: const Text('Center'),
                    ),
                    const SizedBox(width: 8),
                    
                    // Bottom right position
                    ElevatedButton(
                      onPressed: () {
                        setState(() {
                          _left = 0;
                          _top = 0;
                          _right = 20;
                          _bottom = 80;
                        });
                      },
                      child: const Text('Bottom Right'),
                    ),
                  ],
                ),
                const SizedBox(height: 8),
                
                // Reset button
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _left = 20;
                      _top = 20;
                      _right = 20;
                      _bottom = 20;
                    });
                  },
                  child: const Text('Reset'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- AnimatedPositioned animates position changes
- Moves widget smoothly across screen
- Works within Stack widget
- Left, top, right, bottom control position

---

# Real-World Examples

> **Common patterns** with implicit animations.

```dart
/// 1. Animated card with expandable content
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

class _ExpandableCardState extends State<ExpandableCard> {
  bool _isExpanded = false;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: Column(
        children: [
          // Header section (always visible)
          ListTile(
            title: Text(
              widget.title,
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
            trailing: IconButton(
              icon: Icon(
                // Animate icon rotation
                _isExpanded ? Icons.expand_less : Icons.expand_more,
              ),
              onPressed: () {
                setState(() {
                  _isExpanded = !_isExpanded;
                });
              },
            ),
          ),
          
          // Content section (expandable)
          AnimatedContainer(
            duration: const Duration(milliseconds: 300),
            curve: Curves.easeInOut,
            height: _isExpanded ? 200 : 0,
            child: _isExpanded
                ? Padding(
                    padding: const EdgeInsets.all(16),
                    child: widget.content,
                  )
                : null,
          ),
        ],
      ),
    );
  }
}

/// 2. Animated like button
class AnimatedLikeButton extends StatefulWidget {
  const AnimatedLikeButton({super.key});

  @override
  State<AnimatedLikeButton> createState() => _AnimatedLikeButtonState();
}

class _AnimatedLikeButtonState extends State<AnimatedLikeButton>
    with SingleTickerProviderStateMixin {
  bool _isLiked = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        setState(() {
          _isLiked = !_isLiked;
        });
      },
      child: AnimatedScale(
        scale: _isLiked ? 1.2 : 1.0,
        duration: const Duration(milliseconds: 200),
        curve: Curves.elasticOut,
        child: AnimatedContainer(
          duration: const Duration(milliseconds: 300),
          padding: const EdgeInsets.all(12),
          decoration: BoxDecoration(
            color: _isLiked ? Colors.red : Colors.grey[200],
            shape: BoxShape.circle,
          ),
          child: Icon(
            _isLiked ? Icons.favorite : Icons.favorite_border,
            color: _isLiked ? Colors.white : Colors.grey[600],
            size: 30,
          ),
        ),
      ),
    );
  }
}

/// 3. Animated progress bar
class AnimatedProgressBar extends StatefulWidget {
  const AnimatedProgressBar({super.key});

  @override
  State<AnimatedProgressBar> createState() => _AnimatedProgressBarState();
}

class _AnimatedProgressBarState extends State<AnimatedProgressBar> {
  double _progress = 0.0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Animated Progress'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // AnimatedContainer for progress bar
            Container(
              height: 20,
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(10),
              ),
              child: AnimatedContainer(
                duration: const Duration(milliseconds: 500),
                curve: Curves.easeInOut,
                width: _progress * 300,
                decoration: BoxDecoration(
                  color: Colors.blue,
                  borderRadius: BorderRadius.circular(10),
                ),
                child: Center(
                  child: Text(
                    '${(_progress * 100).round()}%',
                    style: const TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                      fontSize: 12,
                    ),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 24),
            
            // Controls
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Increment by 10%
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _progress = (_progress + 0.1).clamp(0.0, 1.0);
                    });
                  },
                  child: const Text('+10%'),
                ),
                const SizedBox(width: 8),
                
                // Increment by 25%
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _progress = (_progress + 0.25).clamp(0.0, 1.0);
                    });
                  },
                  child: const Text('+25%'),
                ),
                const SizedBox(width: 8),
                
                // Reset to 0
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _progress = 0.0;
                    });
                  },
                  child: const Text('Reset'),
                ),
                const SizedBox(width: 8),
                
                // Set to 100%
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _progress = 1.0;
                    });
                  },
                  child: const Text('Complete'),
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
- Expandable card with animation
- Like button with scale and color
- Progress bar with smooth transitions

---

# Best Practices

## Use Appropriate Duration

```dart
// Good - Appropriate duration for the animation
AnimatedContainer(
  duration: const Duration(milliseconds: 300), // 0.3 seconds
  // ...
)

// Bad - Too fast or too slow
AnimatedContainer(
  duration: const Duration(milliseconds: 50), // Too fast - jarring
  // or
  duration: const Duration(seconds: 5), // Too slow - annoying
)
```

## Use Curves for Natural Motion

```dart
// Good - Easing curves
AnimatedContainer(
  curve: Curves.easeInOut, // Smooth acceleration and deceleration
  // ...
)

// Bad - Linear motion
AnimatedContainer(
  curve: Curves.linear, // No easing - unnatural
  // ...
)
```

## Keep Animations Simple

```dart
// Good - Simple, focused animation
AnimatedOpacity(
  opacity: _isVisible ? 1.0 : 0.0,
  duration: const Duration(milliseconds: 300),
  child: ...
)

// Bad - Too many animations at once
// Multiple animated widgets changing simultaneously
```

---

# Common Mistakes

## Forgetting setState

Wrong:
```dart
void _toggle() {
  _isExpanded = !_isExpanded;
  // Missing setState - animation won't trigger
}
```

Correct:
```dart
void _toggle() {
  setState(() {
    _isExpanded = !_isExpanded;
  });
}
```

## Not Using Curves

Wrong:
```dart
// No curve - default is linear
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  // No curve specified
)
```

Correct:
```dart
// With curve for better user experience
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
)
```

---

# Summary

Implicit Animations provide automatic animations when widget properties change. Use AnimatedContainer for multiple properties, AnimatedOpacity for fading, AnimatedAlign for movement, and AnimatedRotation for rotation. Implicit animations are simple to implement and perfect for basic animation needs.

---

# Next Steps

- [Explicit Animations](explicit-animations.md)
- [AnimationController](animationcontroller.md)
- [Tween](tween.md)

---

# Did You Know?

- Implicit animations require setState
- duration controls animation speed
- curves control easing
- AnimatedContainer animates multiple properties
- AnimatedOpacity is perfect for fade effects
- AnimatedAlign moves widgets smoothly
- Implicit animations are great for prototyping
- Implicit animations are more performant than explicit