# Stack

Understand how to overlay and position widgets using Stack.

---

# What is it?

Stack is a layout widget that allows you to overlay multiple children on top of each other. Children in a Stack are positioned relative to the edges of the Stack container, and can be placed at specific coordinates using Positioned widgets. This is useful for creating complex UIs where elements need to overlap, like badges, tooltips, floating buttons, or custom designs.

---

# Why does it exist?

Stack exists to:

- Overlap widgets on top of each other
- Create complex layered UIs
- Position widgets relative to parent edges
- Build custom UI components like badges
- Implement floating elements
- Create overlay effects and animations
- Support complex design patterns

---

# Basic Stack

> **Stack** overlays children on top of each other.

```dart
// Basic Stack usage
class BasicStack extends StatelessWidget {
  const BasicStack({super.key});

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        // 1. Background layer
        Container(
          width: 200,
          height: 200,
          color: Colors.blue,
        ),
        
        // 2. Middle layer
        Container(
          width: 150,
          height: 150,
          color: Colors.green.withOpacity(0.5),
        ),
        
        // 3. Foreground layer
        Container(
          width: 100,
          height: 100,
          color: Colors.red.withOpacity(0.5),
        ),
        
        // 4. Text on top
        const Center(
          child: Text(
            'Stack',
            style: TextStyle(
              color: Colors.white,
              fontSize: 24,
            ),
          ),
        ),
      ],
    );
  }
}

// Stack properties:
// 1. children - List of widgets (later children appear on top)
// 2. alignment - Default alignment for children
// 3. textDirection - LTR or RTL
// 4. fit - How to size the stack
// 5. clipBehavior - Clipping behavior
```

What's happening here?
- Children are stacked in order
- Later children appear on top
- Widgets can overlap
- Positioned widgets for specific placement
- Center for automatic centering

---

# Positioned Widget

> **Positioned** places children at specific locations.

```dart
// Positioned widget examples
class PositionedExample extends StatelessWidget {
  const PositionedExample({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: 300,
      height: 300,
      child: Stack(
        children: [
          // Background
          Container(color: Colors.grey[200]),
          
          // 1. Positioned with top-left
          Positioned(
            top: 20,
            left: 20,
            child: Container(
              padding: const EdgeInsets.all(8),
              color: Colors.red,
              child: const Text('Top Left'),
            ),
          ),
          
          // 2. Positioned with top-right
          Positioned(
            top: 20,
            right: 20,
            child: Container(
              padding: const EdgeInsets.all(8),
              color: Colors.green,
              child: const Text('Top Right'),
            ),
          ),
          
          // 3. Positioned with bottom-left
          Positioned(
            bottom: 20,
            left: 20,
            child: Container(
              padding: const EdgeInsets.all(8),
              color: Colors.blue,
              child: const Text('Bottom Left'),
            ),
          ),
          
          // 4. Positioned with bottom-right
          Positioned(
            bottom: 20,
            right: 20,
            child: Container(
              padding: const EdgeInsets.all(8),
              color: Colors.orange,
              child: const Text('Bottom Right'),
            ),
          ),
          
          // 5. Positioned with center
          Positioned(
            top: 0,
            bottom: 0,
            left: 0,
            right: 0,
            child: const Center(
              child: Text(
                'Center',
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// Positioned properties:
// 1. top - Distance from top edge
// 2. bottom - Distance from bottom edge
// 3. left - Distance from left edge
// 4. right - Distance from right edge
// 5. width - Fixed width
// 6. height - Fixed height
// 7. child - The widget to position
```

What's happening here?
- Positioned places child at specific location
- Can use top, bottom, left, right
- Can use width and height constraints
- Positions relative to Stack edges
- Can combine edges for stretching

---

# Stack Alignment

> **Alignment** controls default child positioning.

```dart
// Stack alignment examples
class StackAlignmentExample extends StatelessWidget {
  const StackAlignmentExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Default alignment (top-left)
        Container(
          width: 200,
          height: 200,
          color: Colors.grey[200],
          child: Stack(
            children: [
              Container(
                width: 50,
                height: 50,
                color: Colors.red,
              ),
              const Text('Default'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Center alignment
        Container(
          width: 200,
          height: 200,
          color: Colors.grey[200],
          child: Stack(
            alignment: Alignment.center,
            children: [
              Container(
                width: 50,
                height: 50,
                color: Colors.red,
              ),
              const Text('Center'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Bottom-right alignment
        Container(
          width: 200,
          height: 200,
          color: Colors.grey[200],
          child: Stack(
            alignment: Alignment.bottomRight,
            children: [
              Container(
                width: 50,
                height: 50,
                color: Colors.red,
              ),
              const Text('Bottom Right'),
            ],
          ),
        ),
      ],
    );
  }
}

// Alignment values:
// 1. Alignment.topLeft
// 2. Alignment.topCenter
// 3. Alignment.topRight
// 4. Alignment.centerLeft
// 5. Alignment.center
// 6. Alignment.centerRight
// 7. Alignment.bottomLeft
// 8. Alignment.bottomCenter
// 9. Alignment.bottomRight
```

What's happening here?
- Alignment centers children by default
- Applies to non-positioned children
- Different alignment options available
- Can be combined with Positioned
- Default is top-left

---

# Stack with Sized Box

> **Controlling Stack** size and fit.

```dart
// Stack sizing examples
class StackSizingExample extends StatelessWidget {
  const StackSizingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Stack with fixed size
        SizedBox(
          width: 200,
          height: 200,
          child: Stack(
            children: [
              Container(color: Colors.blue),
              const Positioned(
                top: 20,
                left: 20,
                child: Text('Fixed Size'),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Stack with fit - expand
        Container(
          color: Colors.grey[200],
          child: Stack(
            fit: StackFit.expand,
            children: [
              Container(color: Colors.blue),
              const Positioned(
                top: 20,
                left: 20,
                child: Text('Expand Fit'),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Stack with fit - loose
        Container(
          color: Colors.grey[200],
          child: Stack(
            fit: StackFit.loose,
            children: [
              Container(
                width: 100,
                height: 100,
                color: Colors.blue,
              ),
              const Positioned(
                top: 20,
                left: 20,
                child: Text('Loose Fit'),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 4. Stack with fit - passthrough
        Container(
          color: Colors.grey[200],
          child: Stack(
            fit: StackFit.passthrough,
            children: [
              Container(
                width: 100,
                height: 100,
                color: Colors.blue,
              ),
              const Positioned(
                top: 20,
                left: 20,
                child: Text('Passthrough'),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// StackFit options:
// 1. StackFit.loose - Children determine size
// 2. StackFit.expand - Stack expands to fill parent
// 3. StackFit.passthrough - Constraints passed through
```

What's happening here?
- fit controls how stack sizes itself
- expand fills parent
- loose allows children to determine size
- passthrough passes constraints
- Choose based on layout needs

---

# Real-World Examples

> **Common patterns** using Stack.

```dart
// 1. Profile picture with badge
class ProfileWithBadge extends StatelessWidget {
  const ProfileWithBadge({super.key});

  @override
  Widget build(BuildContext context) {
    return Stack(
      alignment: Alignment.bottomRight,
      children: [
        // Profile picture
        const CircleAvatar(
          radius: 40,
          backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
        ),
        
        // Online badge
        Positioned(
          bottom: 0,
          right: 0,
          child: Container(
            width: 16,
            height: 16,
            decoration: const BoxDecoration(
              color: Colors.green,
              shape: BoxShape.circle,
              border: Border.fromBorderSide(
                BorderSide(color: Colors.white, width: 2),
              ),
            ),
          ),
        ),
      ],
    );
  }
}

// 2. Image with overlay
class ImageWithOverlay extends StatelessWidget {
  const ImageWithOverlay({super.key});

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        // Image
        Image.network(
          'https://example.com/image.jpg',
          width: double.infinity,
          height: 200,
          fit: BoxFit.cover,
        ),
        
        // Gradient overlay
        Container(
          height: 200,
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
        
        // Text overlay
        Positioned(
          bottom: 20,
          left: 20,
          right: 20,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text(
                'Title',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 4),
              Text(
                'Subtitle goes here',
                style: TextStyle(
                  color: Colors.white.withOpacity(0.8),
                  fontSize: 16,
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// 3. Floating action button
class FloatingActionExample extends StatelessWidget {
  const FloatingActionExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          // Content
          Container(color: Colors.grey[200]),
          
          // FAB with overlay
          Positioned(
            bottom: 30,
            right: 30,
            child: Stack(
              alignment: Alignment.topRight,
              children: [
                // FAB
                Container(
                  width: 56,
                  height: 56,
                  decoration: const BoxDecoration(
                    color: Colors.blue,
                    shape: BoxShape.circle,
                  ),
                  child: const Icon(
                    Icons.add,
                    color: Colors.white,
                    size: 30,
                  ),
                ),
                
                // Badge
                Positioned(
                  top: 0,
                  right: 0,
                  child: Container(
                    padding: const EdgeInsets.all(4),
                    decoration: const BoxDecoration(
                      color: Colors.red,
                      shape: BoxShape.circle,
                    ),
                    child: const Text(
                      '3',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 10,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// 4. Badge on widget
class BadgeWidget extends StatelessWidget {
  const BadgeWidget({
    super.key,
    required this.child,
    required this.count,
  });

  final Widget child;
  final int count;

  @override
  Widget build(BuildContext context) {
    return Stack(
      alignment: Alignment.topRight,
      children: [
        child,
        if (count > 0)
          Positioned(
            top: 4,
            right: 4,
            child: Container(
              padding: const EdgeInsets.symmetric(
                horizontal: 6,
                vertical: 2,
              ),
              decoration: const BoxDecoration(
                color: Colors.red,
                shape: BoxShape.circle,
              ),
              child: Text(
                count > 99 ? '99+' : count.toString(),
                style: const TextStyle(
                  color: Colors.white,
                  fontSize: 10,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
      ],
    );
  }
}
```

What's happening here?
- Profile with online badge
- Image with text overlay
- Floating action button with badge
- Reusable badge widget

---

# Best Practices

## Use Positioned for Exact Placement

```dart
// Good - Using Positioned for exact placement
@override
Widget build(BuildContext context) {
  return Stack(
    children: [
      Container(color: Colors.blue),
      Positioned(
        top: 20,
        left: 20,
        child: const Text('Positioned'),
      ),
    ],
  );
}
```

## Use Alignment for Automatic Positioning

```dart
// Good - Using Alignment for centering
@override
Widget build(BuildContext context) {
  return Stack(
    alignment: Alignment.center,
    children: [
      Container(color: Colors.blue),
      const Text('Centered'),
    ],
  );
}
```

## Keep Stack Efficient

```dart
// Good - Stack with RepaintBoundary
@override
Widget build(BuildContext context) {
  return RepaintBoundary(
    child: Stack(
      children: [
        // Complex widgets
        Container(color: Colors.blue),
        Positioned(
          top: 20,
          left: 20,
          child: const Text('Text'),
        ),
      ],
    ),
  );
}
```

---

# Common Mistakes

## Positioned Without Stack

Wrong:
```dart
// Positioned must be inside Stack
Positioned(
  top: 20,
  left: 20,
  child: const Text('Error'),
)
```

Correct:
```dart
// Positioned inside Stack
Stack(
  children: [
    Positioned(
      top: 20,
      left: 20,
      child: const Text('Correct'),
    ),
  ],
)
```

## Overlapping Without Stack

Wrong:
```dart
// Can't overlap without Stack
Container(color: Colors.blue),
Container(color: Colors.red), // Overlapping
```

Correct:
```dart
// Use Stack for overlapping
Stack(
  children: [
    Container(width: 100, height: 100, color: Colors.blue),
    Container(width: 100, height: 100, color: Colors.red),
  ],
)
```

---

# Summary

Stack overlays widgets on top of each other, enabling complex UI designs. Positioned places children at specific locations, while Alignment provides default positioning. Stack is essential for badges, overlays, floating elements, and custom UI designs.

---

# Next Steps

- [Wrap](wrap.md)
- [Flow](flow.md)
- [Align](align.md)

---

# Did You Know?

- Later children appear on top
- Positioned places widgets relative to Stack edges
- Alignment centers non-positioned children
- Stack can be nested inside Stack
- RepaintBoundary optimizes Stack performance
- Stack is used in many Flutter widgets
- Positioned can stretch with top/bottom/left/right
- Stack supports RTL text direction