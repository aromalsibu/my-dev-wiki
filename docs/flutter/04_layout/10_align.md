# Align

Understand how to position a child within a parent using Align.

---

# What is it?

Align is a widget that positions its child within itself based on alignment values. It allows you to place a child at a specific position (like top-left, center, bottom-right) or at custom fractional positions. Align is simpler than Positioned and is used when you want to position a single child relative to its parent's bounds.

---

# Why does it exist?

Align exists to:

- Position a child within a parent container
- Provide simple alignment options
- Enable fractional positioning
- Support relative positioning
- Create responsive layouts
- Center or offset widgets easily
- Control child positioning without complex calculations

---

# Basic Align

> **Align** positions a single child within itself.

```dart
// Basic Align usage
class BasicAlign extends StatelessWidget {
  const BasicAlign({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 200,
      height: 200,
      color: Colors.grey[200],
      child: Align(
        alignment: Alignment.center,
        child: Container(
          width: 50,
          height: 50,
          color: Colors.blue,
        ),
      ),
    );
  }
}

// Align properties:
// 1. alignment - Where to position the child
// 2. child - The widget to position
// 3. widthFactor - Scale child width (optional)
// 4. heightFactor - Scale child height (optional)
```

What's happening here?
- Align positions child within parent
- alignment controls position
- Child keeps its own size
- Parent determines bounds
- Simple and predictable

---

# Alignment Values

> **Different alignment** values position the child.

```dart
// Alignment values examples
class AlignmentValuesExample extends StatelessWidget {
  const AlignmentValuesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Top-left
        _buildAlignWithLabel(
          'Top Left',
          alignment: Alignment.topLeft,
        ),
        
        // 2. Top-center
        _buildAlignWithLabel(
          'Top Center',
          alignment: Alignment.topCenter,
        ),
        
        // 3. Top-right
        _buildAlignWithLabel(
          'Top Right',
          alignment: Alignment.topRight,
        ),
        
        // 4. Center-left
        _buildAlignWithLabel(
          'Center Left',
          alignment: Alignment.centerLeft,
        ),
        
        // 5. Center
        _buildAlignWithLabel(
          'Center',
          alignment: Alignment.center,
        ),
        
        // 6. Center-right
        _buildAlignWithLabel(
          'Center Right',
          alignment: Alignment.centerRight,
        ),
        
        // 7. Bottom-left
        _buildAlignWithLabel(
          'Bottom Left',
          alignment: Alignment.bottomLeft,
        ),
        
        // 8. Bottom-center
        _buildAlignWithLabel(
          'Bottom Center',
          alignment: Alignment.bottomCenter,
        ),
        
        // 9. Bottom-right
        _buildAlignWithLabel(
          'Bottom Right',
          alignment: Alignment.bottomRight,
        ),
      ],
    );
  }

  Widget _buildAlignWithLabel(
    String label, {
    required Alignment alignment,
  }) {
    return Container(
      width: 150,
      height: 100,
      color: Colors.grey[200],
      margin: const EdgeInsets.only(bottom: 8),
      child: Stack(
        children: [
          Align(
            alignment: alignment,
            child: Container(
              width: 40,
              height: 30,
              color: Colors.blue,
              child: Center(
                child: Text(
                  label.split(' ').last,
                  style: const TextStyle(
                    color: Colors.white,
                    fontSize: 10,
                  ),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// Standard alignments:
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
- 9 standard alignment positions
- Each positions child differently
- Child is placed relative to parent
- Simple and intuitive

---

# Custom Alignment

> **Custom alignment** values provide precise control.

```dart
// Custom alignment examples
class CustomAlignmentExample extends StatelessWidget {
  const CustomAlignmentExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 300,
      color: Colors.grey[200],
      child: Column(
        children: [
          // 1. Fractional alignment (0-1 range)
          Expanded(
            child: Align(
              alignment: const Alignment(0.2, 0.3),
              child: Container(
                width: 50,
                height: 30,
                color: Colors.red,
                child: const Center(
                  child: Text(
                    '0.2,0.3',
                    style: TextStyle(color: Colors.white, fontSize: 10),
                  ),
                ),
              ),
            ),
          ),
          
          // 2. Negative values
          Expanded(
            child: Align(
              alignment: const Alignment(-0.5, -0.5),
              child: Container(
                width: 50,
                height: 30,
                color: Colors.green,
                child: const Center(
                  child: Text(
                    '-0.5,-0.5',
                    style: TextStyle(color: Colors.white, fontSize: 10),
                  ),
                ),
              ),
            ),
          ),
          
          // 3. Using AlignmentDirectional
          Expanded(
            child: Align(
              alignment: const AlignmentDirectional(0.8, 0.2),
              child: Container(
                width: 50,
                height: 30,
                color: Colors.blue,
                child: const Center(
                  child: Text(
                    '0.8,0.2',
                    style: TextStyle(color: Colors.white, fontSize: 10),
                  ),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// Alignment values explained:
// Alignment(0, 0) = center
// Alignment(-1, -1) = top-left
// Alignment(1, 1) = bottom-right
// Alignment(0.5, 0) = center-right
// Values range from -1 to 1
// 0 = center, -1 = start, 1 = end
```

What's happening here?
- Custom values from -1 to 1
- Fine-grained control
- Negative values for opposite side
- Fractional positioning

---

# Align with Size Factors

> **WidthFactor and heightFactor** scale the child.

```dart
// Size factor examples
class SizeFactorExample extends StatelessWidget {
  const SizeFactorExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: Column(
        children: [
          // 1. No factor (child keeps size)
          Expanded(
            child: Align(
              alignment: Alignment.center,
              child: Container(
                color: Colors.red,
                child: const Text('No Factor'),
              ),
            ),
          ),
          
          // 2. Width factor 0.5 (child takes half width)
          Expanded(
            child: Align(
              alignment: Alignment.center,
              widthFactor: 0.5,
              child: Container(
                color: Colors.green,
                child: const Text('Width 0.5'),
              ),
            ),
          ),
          
          // 3. Both factors (child scales)
          Expanded(
            child: Align(
              alignment: Alignment.center,
              widthFactor: 0.5,
              heightFactor: 0.5,
              child: Container(
                color: Colors.blue,
                child: const Text('Both 0.5'),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// Size factor effects:
// widthFactor: null (use child width)
// widthFactor: 0.5 (child width = 50% of parent)
// widthFactor: 1.0 (child fills parent width)
// heightFactor: null (use child height)
// heightFactor: 0.5 (child height = 50% of parent)
// heightFactor: 1.0 (child fills parent height)
```

What's happening here?
- widthFactor scales child width
- heightFactor scales child height
- Null uses child's own size
- 1.0 fills parent dimension
- Useful for responsive sizing

---

# Align vs Other Positioning

> **Comparing Align** with similar widgets.

```dart
// Align vs Container alignment
class AlignmentComparison extends StatelessWidget {
  const AlignmentComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Align widget
        Container(
          width: 200,
          height: 100,
          color: Colors.grey[200],
          child: Align(
            alignment: Alignment.center,
            child: Container(
              width: 50,
              height: 30,
              color: Colors.blue,
            ),
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Container alignment
        Container(
          width: 200,
          height: 100,
          color: Colors.grey[200],
          alignment: Alignment.center,
          child: Container(
            width: 50,
            height: 30,
            color: Colors.green,
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Center widget
        Container(
          width: 200,
          height: 100,
          color: Colors.grey[200],
          child: Center(
            child: Container(
              width: 50,
              height: 30,
              color: Colors.red,
            ),
          ),
        ),
      ],
    );
  }
}

// When to use each:
// Align: Any alignment, fractional positioning
// Center: Only centering (simpler)
// Container.alignment: Built-in alignment
// Positioned: Only in Stack
```

What's happening here?
- Align: flexible alignment
- Container.alignment: built-in
- Center: simple centering
- Each has different use cases

---

# Real-World Examples

> **Common patterns** using Align.

```dart
// 1. Badge on widget
class BadgeWithAlign extends StatelessWidget {
  const BadgeWithAlign({
    super.key,
    required this.child,
    required this.count,
  });

  final Widget child;
  final int count;

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        child,
        Positioned(
          top: 0,
          right: 0,
          child: Align(
            alignment: Alignment.topRight,
            child: Container(
              padding: const EdgeInsets.all(4),
              decoration: const BoxDecoration(
                color: Colors.red,
                shape: BoxShape.circle,
              ),
              child: Text(
                count > 99 ? '99+' : count.toString(),
                style: const TextStyle(
                  color: Colors.white,
                  fontSize: 10,
                ),
              ),
            ),
          ),
        ),
      ],
    );
  }
}

// 2. Floating action button with alignment
class FloatingActionWithAlign extends StatelessWidget {
  const FloatingActionWithAlign({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          // Content
          Container(color: Colors.grey[200]),
          
          // FAB positioned with Align
          Align(
            alignment: Alignment.bottomRight,
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Container(
                width: 56,
                height: 56,
                decoration: const BoxDecoration(
                  color: Colors.blue,
                  shape: BoxShape.circle,
                ),
                child: const Icon(
                  Icons.add,
                  color: Colors.white,
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// 3. Logo with text overlay
class LogoWithOverlay extends StatelessWidget {
  const LogoWithOverlay({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 200,
      height: 200,
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(16),
        image: const DecorationImage(
          image: NetworkImage('https://example.com/logo.jpg'),
          fit: BoxFit.cover,
        ),
      ),
      child: Align(
        alignment: Alignment.bottomCenter,
        child: Container(
          width: double.infinity,
          padding: const EdgeInsets.all(8),
          color: Colors.black.withOpacity(0.7),
          child: const Text(
            'Logo Text',
            style: TextStyle(
              color: Colors.white,
              fontSize: 16,
              fontWeight: FontWeight.bold,
            ),
            textAlign: TextAlign.center,
          ),
        ),
      ),
    );
  }
}

// 4. Responsive positioning
class ResponsivePosition extends StatelessWidget {
  const ResponsivePosition({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Adjust alignment based on screen size
        final isLarge = constraints.maxWidth > 600;
        
        return Container(
          width: double.infinity,
          height: 200,
          color: Colors.grey[200],
          child: Align(
            alignment: isLarge 
                ? Alignment.centerRight 
                : Alignment.center,
            child: Container(
              padding: const EdgeInsets.all(16),
              color: Colors.blue,
              child: Text(
                isLarge ? 'Large Screen' : 'Small Screen',
                style: const TextStyle(color: Colors.white),
              ),
            ),
          ),
        );
      },
    );
  }
}
```

What's happening here?
- Badge positioned with Align
- Floating action button
- Image overlay with text
- Responsive positioning

---

# Best Practices

## Use Align for Single Child Positioning

```dart
// Good - Using Align
@override
Widget build(BuildContext context) {
  return Container(
    child: Align(
      alignment: Alignment.center,
      child: const Text('Positioned'),
    ),
  );
}
```

## Use Alignment Values

```dart
// Good - Using standard alignments
@override
Widget build(BuildContext context) {
  return Align(
    alignment: Alignment.topRight,
    child: child,
  );
}
```

## Use Size Factors for Responsive Sizing

```dart
// Good - Responsive sizing
@override
Widget build(BuildContext context) {
  return Align(
    alignment: Alignment.center,
    widthFactor: 0.8, // 80% of parent width
    child: Container(
      color: Colors.blue,
      child: const Text('Responsive'),
    ),
  );
}
```

---

# Common Mistakes

## Overusing Positioned

Wrong:
```dart
// Using Positioned outside Stack
Positioned(
  top: 20,
  left: 20,
  child: const Text('Error'),
)
```

Correct:
```dart
// Use Align for simple positioning
Align(
  alignment: Alignment.topLeft,
  child: const Text('Correct'),
)
```

## Wrong Alignment Values

Wrong:
```dart
// Values outside -1 to 1 range
Align(
  alignment: Alignment(2, 2), // Invalid
  child: child,
)
```

Correct:
```dart
// Values within -1 to 1 range
Align(
  alignment: Alignment(0.5, 0.5),
  child: child,
)
```

---

# Summary

Align positions a child within a parent using alignment values. It provides 9 standard positions and custom fractional positioning. Use Align for simple positioning, responsive layouts, and when you need to place a child relative to its parent's bounds.

---

# Next Steps

- [Center](center.md)
- [SizedBox](sizedbox.md)
- [Expanded](expanded.md)

---

# Did You Know?

- Align positions relative to parent bounds
- Alignment values range from -1 to 1
- widthFactor and heightFactor scale child size
- Align is simpler than Positioned
- Container.alignment uses the same system
- Align works with any child widget
- Align supports RTL layouts
- Align is used in many Flutter widgets