# Physics Animations

Understand how to create realistic, physics-based animations in Flutter.

---

# What is it?

Physics Animations are animations that simulate real-world physics like spring motion, friction, gravity, and momentum. They create natural, responsive interactions that feel realistic and intuitive. Flutter provides physics simulations through widgets like AnimatedPhysics, SpringSimulation, and physics-based animation controllers.

---

# Why does it exist?

Physics Animations exist to:

- Create realistic and natural motion
- Simulate real-world physics
- Provide responsive interactions
- Enhance user experience with fluid animations
- Implement spring effects and bouncing
- Create physics-based gestures
- Make UI feel more tactile and engaging

---

# Spring Simulation

> **Creating spring-based** animations.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/physics.dart';

/// Spring simulation example
class SpringSimulationExample extends StatefulWidget {
  const SpringSimulationExample({super.key});

  @override
  State<SpringSimulationExample> createState() => _SpringSimulationExampleState();
}

class _SpringSimulationExampleState extends State<SpringSimulationExample>
    with SingleTickerProviderStateMixin {
  // 1. Animation controller with spring simulation
  late AnimationController _controller;
  late Animation<double> _springAnimation;
  
  // 2. Track the current state
  double _position = 0.0;
  bool _isExpanded = false;

  @override
  void initState() {
    super.initState();
    
    // 3. Create animation controller with spring simulation
    _controller = AnimationController(
      // Spring simulation parameters
      // Physics: SpringSimulation creates realistic spring motion
      vsync: this,
      duration: const Duration(seconds: 1),
    );
    
    // 4. Create spring animation with custom physics
    _springAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.elasticOut, // Built-in spring-like curve
      ),
    )..addListener(() {
        setState(() {
          _position = _springAnimation.value;
        });
      });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _toggleAnimation() {
    setState(() {
      _isExpanded = !_isExpanded;
    });
    
    // 5. Start animation with spring effect
    if (_isExpanded) {
      _controller.forward(from: 0.0);
    } else {
      _controller.reverse();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Spring Simulation'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 6. Animated container with spring effect
            AnimatedContainer(
              duration: const Duration(milliseconds: 500),
              width: 100 + _position * 100,
              height: 100 + _position * 100,
              decoration: BoxDecoration(
                color: Colors.blue,
                borderRadius: BorderRadius.circular(_position * 20),
                boxShadow: [
                  BoxShadow(
                    color: Colors.blue.withOpacity(0.3),
                    blurRadius: _position * 20,
                    spreadRadius: _position * 10,
                  ),
                ],
              ),
              child: Center(
                child: Text(
                  '${(_position * 100).round()}%',
                  style: const TextStyle(color: Colors.white),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 7. Control buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _toggleAnimation,
                  child: Text(_isExpanded ? 'Collapse' : 'Expand'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    _controller.stop();
                    setState(() {
                      _position = 0.0;
                      _isExpanded = false;
                    });
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
- Spring simulation creates natural motion
- elasticOut curve simulates spring
- Position animates with spring effect
- Realistic bounce and overshoot

---

# Spring Simulation with Custom Physics

> **Using custom** spring physics.

```dart
/// Custom spring simulation with physics parameters
class CustomSpringExample extends StatefulWidget {
  const CustomSpringExample({super.key});

  @override
  State<CustomSpringExample> createState() => _CustomSpringExampleState();
}

class _CustomSpringExampleState extends State<CustomSpringExample>
    with SingleTickerProviderStateMixin {
  // 1. Animation controller with custom spring
  late AnimationController _controller;
  late Animation<double> _customSpring;
  
  // 2. Spring parameters
  double _stiffness = 150.0; // Spring stiffness (higher = more rigid)
  double _damping = 15.0;    // Damping (higher = less bounce)
  
  // 3. Track current value
  double _value = 0.0;
  bool _isAnimating = false;

  @override
  void initState() {
    super.initState();
    
    // 4. Create physics-based animation
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    );
    
    // 5. Custom spring simulation
    _customSpring = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.bounceOut, // Built-in bounce
      ),
    )..addListener(() {
        setState(() {
          _value = _customSpring.value;
        });
      })
      ..addStatusListener((status) {
        if (status == AnimationStatus.completed) {
          setState(() {
            _isAnimating = false;
          });
        }
      });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  // 6. Trigger spring animation
  void _triggerSpring() {
    setState(() {
      _isAnimating = true;
    });
    _controller.forward(from: 0.0);
  }

  // 7. Update spring parameters
  void _updateSpring({double? stiffness, double? damping}) {
    setState(() {
      if (stiffness != null) _stiffness = stiffness;
      if (damping != null) _damping = damping;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Spring Physics'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 8. Animated widget with custom spring
            Container(
              width: 200,
              height: 200,
              child: Center(
                child: AnimatedContainer(
                  duration: const Duration(milliseconds: 500),
                  width: 50 + _value * 100,
                  height: 50 + _value * 100,
                  decoration: BoxDecoration(
                    color: Colors.blue,
                    borderRadius: BorderRadius.circular(8),
                    boxShadow: [
                      BoxShadow(
                        color: Colors.blue.withOpacity(0.3),
                        blurRadius: 20,
                        spreadRadius: 5,
                      ),
                    ],
                  ),
                  child: Center(
                    child: Text(
                      '${(_value * 100).round()}%',
                      style: const TextStyle(color: Colors.white),
                    ),
                  ),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 9. Control buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _isAnimating ? null : _triggerSpring,
                  child: const Text('Spring'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    _controller.stop();
                    setState(() {
                      _value = 0.0;
                      _isAnimating = false;
                    });
                  },
                  child: const Text('Reset'),
                ),
              ],
            ),
            
            const SizedBox(height: 24),
            
            // 10. Spring parameter controls
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Spring Parameters:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  Row(
                    children: [
                      const Text('Stiffness:'),
                      Expanded(
                        child: Slider(
                          value: _stiffness,
                          min: 50,
                          max: 300,
                          onChanged: (value) {
                            _updateSpring(stiffness: value);
                          },
                        ),
                      ),
                      Text('${_stiffness.round()}'),
                    ],
                  ),
                  Row(
                    children: [
                      const Text('Damping:'),
                      Expanded(
                        child: Slider(
                          value: _damping,
                          min: 5,
                          max: 30,
                          onChanged: (value) {
                            _updateSpring(damping: value);
                          },
                        ),
                      ),
                      Text('${_damping.round()}'),
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
- Custom spring physics parameters
- Stiffness controls rigidity
- Damping controls bounce
- Interactive parameter adjustment

---

# Physics Simulation with Draggable

> **Creating draggable** physics interactions.

```dart
/// Physics simulation with draggable widget
class DraggablePhysicsExample extends StatefulWidget {
  const DraggablePhysicsExample({super.key});

  @override
  State<DraggablePhysicsExample> createState() => _DraggablePhysicsExampleState();
}

class _DraggablePhysicsExampleState extends State<DraggablePhysicsExample>
    with SingleTickerProviderStateMixin {
  // 1. Animation controller for physics
  late AnimationController _controller;
  
  // 2. Physics simulation
  SpringSimulation? _simulation;
  
  // 3. Track drag position
  double _dragPosition = 0.0;
  double _currentPosition = 0.0;
  bool _isDragging = false;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );
    
    // 4. Add listener for animation updates
    _controller.addListener(() {
      setState(() {
        _currentPosition = _controller.value * 300;
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  // 5. Handle drag start
  void _onDragStart(DragStartDetails details) {
    setState(() {
      _isDragging = true;
    });
    _controller.stop();
  }

  // 6. Handle drag update
  void _onDragUpdate(DragUpdateDetails details) {
    setState(() {
      _dragPosition += details.delta.dx;
      // Clamp position
      _dragPosition = _dragPosition.clamp(0.0, 300.0);
      _currentPosition = _dragPosition;
    });
  }

  // 7. Handle drag end with physics
  void _onDragEnd(DragEndDetails details) {
    setState(() {
      _isDragging = false;
    });
    
    // 8. Create spring simulation
    // Physics simulation will animate to target
    final target = _dragPosition > 150 ? 1.0 : 0.0;
    
    // Convert position to animation value
    final currentValue = _dragPosition / 300;
    
    // Create spring simulation with physics
    _simulation = SpringSimulation(
      // Spring parameters
      SpringDescription(
        mass: 1.0,
        stiffness: 100.0,
        damping: 15.0,
      ),
      // Start position
      currentValue,
      // Target position
      target,
      // Velocity (from drag)
      details.velocity.pixelsPerSecond.dx / 1000,
    );
    
    // 9. Animate with physics
    _controller.animateWith(_simulation!);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Physics Draggable'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Container(
              width: 300,
              height: 200,
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(16),
              ),
              child: GestureDetector(
                onHorizontalDragStart: _onDragStart,
                onHorizontalDragUpdate: _onDragUpdate,
                onHorizontalDragEnd: _onDragEnd,
                child: Stack(
                  children: [
                    // Track
                    Container(
                      width: 300,
                      height: 4,
                      margin: const EdgeInsets.symmetric(vertical: 98),
                      color: Colors.grey[300],
                    ),
                    
                    // 10. Draggable ball with physics
                    AnimatedBuilder(
                      animation: _controller,
                      builder: (context, child) {
                        return Positioned(
                          left: _isDragging ? _dragPosition : _currentPosition,
                          top: 50,
                          child: Container(
                            width: 100,
                            height: 100,
                            decoration: BoxDecoration(
                              shape: BoxShape.circle,
                              color: Colors.blue,
                              boxShadow: [
                                BoxShadow(
                                  color: Colors.blue.withOpacity(0.3),
                                  blurRadius: 20,
                                  spreadRadius: 5,
                                ),
                              ],
                            ),
                            child: const Center(
                              child: Text(
                                'Drag',
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
            ),
            
            const SizedBox(height: 24),
            
            // 11. Position display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  Text('Position: ${_currentPosition.round()}px'),
                  Text('Status: ${_isDragging ? "Dragging" : "Physics"}'),
                  Text('Progress: ${(_currentPosition / 300 * 100).round()}%'),
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
- Draggable widget with physics
- Spring simulation on release
- Realistic spring motion
- Interactive drag and release

---

# Real-World Examples

> **Common patterns** with physics animations.

```dart
/// 1. Animated button with spring
class SpringButton extends StatefulWidget {
  const SpringButton({
    super.key,
    required this.onPressed,
    required this.child,
  });

  final VoidCallback onPressed;
  final Widget child;

  @override
  State<SpringButton> createState() => _SpringButtonState();
}

class _SpringButtonState extends State<SpringButton>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _springAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(milliseconds: 500),
      vsync: this,
    );
    
    _springAnimation = Tween<double>(begin: 1.0, end: 0.9).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.elasticOut,
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
        _controller.forward(from: 0.0);
      },
      onTapUp: (_) {
        _controller.reverse();
      },
      onTapCancel: () {
        _controller.reverse();
      },
      onTap: widget.onPressed,
      child: AnimatedBuilder(
        animation: _springAnimation,
        builder: (context, child) {
          return Transform.scale(
            scale: _springAnimation.value,
            child: Container(
              padding: const EdgeInsets.symmetric(
                horizontal: 24,
                vertical: 12,
              ),
              decoration: BoxDecoration(
                color: Colors.blue,
                borderRadius: BorderRadius.circular(8),
                boxShadow: [
                  BoxShadow(
                    color: Colors.blue.withOpacity(0.3),
                    blurRadius: 10 * (1 - _springAnimation.value),
                    spreadRadius: 5 * (1 - _springAnimation.value),
                  ),
                ],
              ),
              child: widget.child,
            ),
          );
        },
      ),
    );
  }
}

/// 2. Physics-based list item
class PhysicsListItem extends StatefulWidget {
  const PhysicsListItem({
    super.key,
    required this.title,
    this.onDismissed,
  });

  final String title;
  final VoidCallback? onDismissed;

  @override
  State<PhysicsListItem> createState() => _PhysicsListItemState();
}

class _PhysicsListItemState extends State<PhysicsListItem>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _dismissAnimation;

  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: const Duration(milliseconds: 300),
      vsync: this,
    );
    
    _dismissAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.bounceOut,
      ),
    )..addListener(() { setState(() {}); })
      ..addStatusListener((status) {
        if (status == AnimationStatus.completed) {
          widget.onDismissed?.call();
        }
      });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _dismiss() {
    _controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return Dismissible(
      key: ValueKey(widget.title),
      onDismissed: (_) {
        widget.onDismissed?.call();
      },
      background: Container(
        color: Colors.red,
        alignment: Alignment.centerRight,
        padding: const EdgeInsets.only(right: 20),
        child: const Icon(
          Icons.delete,
          color: Colors.white,
        ),
      ),
      child: Card(
        margin: const EdgeInsets.all(8),
        child: ListTile(
          title: Text(widget.title),
          trailing: IconButton(
            icon: const Icon(Icons.delete_outline),
            onPressed: _dismiss,
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- SpringButton with elastic feedback
- Dismissible with physics-based animation
- Realistic interaction feedback
- Physics-based list items

---

# Best Practices

## Use Appropriate Physics

```dart
// Good - Realistic spring
SpringSimulation(
  SpringDescription(
    mass: 1.0,
    stiffness: 100.0,
    damping: 15.0,
  ),
  start, target, velocity,
)

// Bad - Unrealistic physics
SpringSimulation(
  SpringDescription(
    mass: 100.0, // Too heavy
    stiffness: 1.0, // Too soft
    damping: 1.0, // Too bouncy
  ),
  start, target, velocity,
)
```

## Combine with Gestures

```dart
// Good - Physics with gestures
GestureDetector(
  onHorizontalDragUpdate: (details) {
    // Update position
  },
  onHorizontalDragEnd: (details) {
    // Apply physics simulation
  },
)
```

## Use CurvedAnimation

```dart
// Good - Physics with curves
CurvedAnimation(
  parent: _controller,
  curve: Curves.elasticOut,
)
```

---

# Common Mistakes

## Incorrect Physics Parameters

Wrong:
```dart
// Unrealistic spring
SpringDescription(
  mass: 0.1,
  stiffness: 1000,
  damping: 1,
)
```

Correct:
```dart
// Realistic spring
SpringDescription(
  mass: 1.0,
  stiffness: 100,
  damping: 15,
)
```

## Not Handling Drag End

Wrong:
```dart
// No physics on drag end
onHorizontalDragEnd: (details) {
  // Missing physics simulation
}
```

Correct:
```dart
// Apply physics on drag end
onHorizontalDragEnd: (details) {
  _controller.animateWith(_simulation!);
}
```

---

# Summary

Physics Animations create realistic, natural motion using physics simulations. Use SpringSimulation for spring effects, combine with gestures for interactive experiences, and adjust parameters for different behaviors. Physics animations make UI feel responsive and engaging.

---

# Next Steps

- [Hero](hero.md)
- [AnimatedBuilder](animatedbuilder.md)
- [CustomPainter Animation](custompainter-animation.md)

---

# Did You Know?

- SpringSimulation creates realistic spring motion
- Stiffness controls rigidity
- Damping controls bounce
- Physics with gestures feels natural
- CurvedAnimation can simulate physics
- Physics animations are responsive
- Physics can be customized
- Physics animations enhance UX