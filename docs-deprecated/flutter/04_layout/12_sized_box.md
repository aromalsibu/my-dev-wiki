# SizedBox

Understand how to create boxes with fixed or constrained dimensions.

---

# What is it?

SizedBox is a widget that creates a box with a specific width, height, or both. It can be used to create fixed-size spaces, constrain a child to specific dimensions, or create empty space in layouts. SizedBox is one of the most versatile and commonly used layout widgets in Flutter.

---

# Why does it exist?

SizedBox exists to:

- Create fixed-size boxes
- Add spacing between widgets
- Constrain child dimensions
- Create empty space in layouts
- Force specific sizes on children
- Create responsive gaps
- Simplify layout spacing

---

# Basic SizedBox

> **SizedBox** creates a box with fixed dimensions.

```dart
// Basic SizedBox usage
class BasicSizedBox extends StatelessWidget {
  const BasicSizedBox({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Fixed size box
        const SizedBox(
          width: 100,
          height: 100,
          child: ColoredBox(color: Colors.blue),
        ),
        
        // 2. Width only
        const SizedBox(
          width: 200,
          child: ColoredBox(color: Colors.green),
        ),
        
        // 3. Height only
        const SizedBox(
          height: 50,
          child: ColoredBox(color: Colors.red),
        ),
        
        // 4. Empty space
        const SizedBox(height: 20),
        
        // 5. No constraints (empty box)
        const SizedBox(),
      ],
    );
  }
}

// SizedBox properties:
// 1. width - Fixed width (optional)
// 2. height - Fixed height (optional)
// 3. child - Child widget (optional)
```

What's happening here?
- Fixed width and height
- Can specify only width or height
- Child can be sized by SizedBox
- Empty space for spacing
- No child = empty box

---

# SizedBox for Spacing

> **SizedBox** is ideal for creating spacing.

```dart
// Spacing with SizedBox
class SpacingWithSizedBox extends StatelessWidget {
  const SpacingWithSizedBox({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Vertical spacing
        const Text('Top Text'),
        const SizedBox(height: 20), // Vertical gap
        const Text('Middle Text'),
        const SizedBox(height: 30), // Larger gap
        const Text('Bottom Text'),
        
        const SizedBox(height: 20),
        
        // 2. Horizontal spacing
        Row(
          children: [
            const Icon(Icons.star),
            const SizedBox(width: 10), // Horizontal gap
            const Icon(Icons.star),
            const SizedBox(width: 20), // Larger gap
            const Icon(Icons.star),
          ],
        ),
        
        const SizedBox(height: 20),
        
        // 3. Combined spacing
        Row(
          children: [
            Container(
              width: 50,
              height: 50,
              color: Colors.red,
            ),
            const SizedBox(width: 16),
            Container(
              width: 50,
              height: 50,
              color: Colors.green,
            ),
            const SizedBox(width: 16),
            Container(
              width: 50,
              height: 50,
              color: Colors.blue,
            ),
          ],
        ),
      ],
    );
  }
}

// Common spacing values:
// SizedBox(height: 4) - Tiny gap
// SizedBox(height: 8) - Small gap
// SizedBox(height: 16) - Medium gap
// SizedBox(height: 24) - Large gap
// SizedBox(height: 32) - Extra large gap
```

What's happening here?
- SizedBox for vertical gaps
- SizedBox for horizontal gaps
- Consistent spacing values
- Clean and readable code
- Reusable spacing patterns

---

# SizedBox with Child

> **SizedBox constrains** its child to specific dimensions.

```dart
// SizedBox with child
class SizedBoxWithChild extends StatelessWidget {
  const SizedBoxWithChild({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Child constrained to size
        const SizedBox(
          width: 150,
          height: 100,
          child: ColoredBox(
            color: Colors.blue,
            child: Center(
              child: Text('Constrained'),
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Width constraint only
        Row(
          children: [
            const SizedBox(
              width: 100,
              child: ColoredBox(
                color: Colors.green,
                child: Text('Width 100'),
              ),
            ),
            const SizedBox(width: 10),
            const SizedBox(
              width: 150,
              child: ColoredBox(
                color: Colors.red,
                child: Text('Width 150'),
              ),
            ),
          ],
        ),
        
        const SizedBox(height: 20),
        
        // 3. Height constraint only
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            const SizedBox(
              height: 50,
              child: ColoredBox(
                color: Colors.purple,
                child: Text('Height 50'),
              ),
            ),
            const SizedBox(
              height: 80,
              child: ColoredBox(
                color: Colors.orange,
                child: Text('Height 80'),
              ),
            ),
          ],
        ),
      ],
    );
  }
}
```

What's happening here?
- Child forced to specific size
- Width only or height only
- Child expands to fill SizedBox
- Useful for uniform sizing

---

# SizedBox vs Container

> **Comparing SizedBox** with Container.

```dart
// SizedBox vs Container
class SizedBoxVsContainer extends StatelessWidget {
  const SizedBoxVsContainer({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. SizedBox (simpler for sizing)
        const SizedBox(
          width: 100,
          height: 100,
          child: ColoredBox(color: Colors.blue),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Container (more features)
        Container(
          width: 100,
          height: 100,
          color: Colors.green,
          child: const Text('Container'),
        ),
        
        const SizedBox(height: 10),
        
        // 3. When to use each
        Row(
          children: [
            // SizedBox: Just sizing
            const SizedBox(
              width: 80,
              height: 80,
              child: ColoredBox(color: Colors.red),
            ),
            
            const SizedBox(width: 10),
            
            // Container: Sizing + styling
            Container(
              width: 80,
              height: 80,
              decoration: BoxDecoration(
                color: Colors.blue,
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Center(child: Text('Styled')),
            ),
          ],
        ),
      ],
    );
  }
}

// When to use:
// SizedBox: Simple sizing, spacing
// Container: Sizing + decoration, padding, margin
// Use SizedBox for spacing and simple sizing
// Use Container for styling + sizing
```

What's happening here?
- SizedBox: simple sizing only
- Container: sizing + styling
- Choose based on needs
- SizedBox is more lightweight

---

# SizedBox.expand

> **SizedBox.expand** fills available space.

```dart
// SizedBox.expand example
class SizedBoxExpandExample extends StatelessWidget {
  const SizedBoxExpandExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Expand to fill parent
        Container(
          height: 100,
          color: Colors.grey[200],
          child: const SizedBox.expand(
            child: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Text('Expanded'),
              ),
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Expand in Row
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            children: [
              const SizedBox(
                width: 50,
                child: ColoredBox(color: Colors.red),
              ),
              const SizedBox.expand(
                child: ColoredBox(
                  color: Colors.green,
                  child: Center(child: Text('Expand')),
                ),
              ),
              const SizedBox(
                width: 50,
                child: ColoredBox(color: Colors.blue),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 3. Expand with constraints
        Container(
          height: 100,
          color: Colors.grey[200],
          child: const SizedBox.expand(
            child: ColoredBox(
              color: Colors.orange,
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Text('Expanded with padding'),
              ),
            ),
          ),
        ),
      ],
    );
  }
}

// SizedBox.expand = SizedBox(
//   width: double.infinity,
//   height: double.infinity,
// )
// Fills all available space
```

What's happening here?
- Expands to fill parent
- Equivalent to infinite size
- Common for background widgets
- Useful for filling space

---

# SizedBox.shrink

> **SizedBox.shrink** creates an empty box with zero size.

```dart
// SizedBox.shrink example
class SizedBoxShrinkExample extends StatelessWidget {
  const SizedBoxShrinkExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Shrink vs empty space
        Row(
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            const SizedBox.shrink(), // Takes no space
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
        
        const SizedBox(height: 20),
        
        // 2. Conditional rendering with shrink
        Row(
          children: [
            Container(width: 50, height: 50, color: Colors.green),
            if (false) 
              const SizedBox(width: 20) // Not shown
            else
              const SizedBox.shrink(), // Shown but takes no space
            Container(width: 50, height: 50, color: Colors.orange),
          ],
        ),
        
        const SizedBox(height: 20),
        
        // 3. Shrink in Column
        Column(
          children: [
            Container(height: 50, color: Colors.purple),
            const SizedBox.shrink(), // Takes no vertical space
            Container(height: 50, color: Colors.pink),
          ],
        ),
      ],
    );
  }
}

// SizedBox.shrink = SizedBox(width: 0, height: 0)
// Takes no space
// Useful for conditional rendering
// Similar to SizedBox()
```

What's happening here?
- Takes no space (zero size)
- Useful for conditional rendering
- Similar to SizedBox()
- Lightweight empty widget

---

# Real-World Examples

> **Common patterns** using SizedBox.

```dart
// 1. Form spacing
class FormWithSpacing extends StatelessWidget {
  const FormWithSpacing({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        children: [
          const TextField(
            decoration: InputDecoration(labelText: 'Email'),
          ),
          const SizedBox(height: 16),
          const TextField(
            decoration: InputDecoration(labelText: 'Password'),
          ),
          const SizedBox(height: 24),
          SizedBox(
            width: double.infinity,
            height: 50,
            child: ElevatedButton(
              onPressed: () {},
              child: const Text('Login'),
            ),
          ),
        ],
      ),
    );
  }
}

// 2. Card with fixed image
class FixedImageCard extends StatelessWidget {
  const FixedImageCard({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Fixed height image
          const SizedBox(
            height: 200,
            width: double.infinity,
            child: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Icon(
                  Icons.image,
                  size: 64,
                  color: Colors.white,
                ),
              ),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'Card Title',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                Text(
                  'Card description goes here. This is the content of the card.',
                  style: TextStyle(
                    color: Colors.grey[600],
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

// 3. Uniform grid items
class UniformGrid extends StatelessWidget {
  const UniformGrid({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.count(
      crossAxisCount: 3,
      padding: const EdgeInsets.all(8),
      crossAxisSpacing: 8,
      mainAxisSpacing: 8,
      children: [
        for (int i = 0; i < 9; i++)
          const SizedBox(
            height: 100,
            child: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Text(
                  'Item',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
      ],
    );
  }
}

// 4. Responsive spacing
class ResponsiveSpacing extends StatelessWidget {
  const ResponsiveSpacing({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Responsive spacing based on screen width
        final spacing = constraints.maxWidth > 600 ? 32.0 : 16.0;
        
        return Column(
          children: [
            const Text('Large Screen: More spacing'),
            SizedBox(height: spacing),
            const Text('Small Screen: Less spacing'),
          ],
        );
      },
    );
  }
}
```

What's happening here?
- Form with consistent spacing
- Card with fixed image height
- Uniform grid items
- Responsive spacing

---

# Best Practices

## Use SizedBox for Spacing

```dart
// Good - Using SizedBox
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      Text('First'),
      const SizedBox(height: 16),
      Text('Second'),
    ],
  );
}
```

## Use SizedBox.shrink for Conditional Rendering

```dart
// Good - Conditional widget
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      Text('Always shown'),
      if (showWidget) 
        const SizedBox.shrink() // Placeholder
      else
        Container(),
    ],
  );
}
```

## Combine with Expanded

```dart
// Good - With Expanded
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Expanded(
        child: SizedBox(
          height: 50,
          child: Container(color: Colors.blue),
        ),
      ),
    ],
  );
}
```

---

# Common Mistakes

## Using SizedBox for Everything

Wrong:
```dart
// Overusing SizedBox
SizedBox(
  width: 100,
  height: 100,
  child: Container(
    color: Colors.blue,
    child: Text('Hello'),
  ),
)
```

Correct:
```dart
// Use Container directly
Container(
  width: 100,
  height: 100,
  color: Colors.blue,
  child: const Text('Hello'),
)
```

## Forgetting Child

Wrong:
```dart
// No effect without child
const SizedBox(width: 100, height: 100);
```

Correct:
```dart
// With child
const SizedBox(
  width: 100,
  height: 100,
  child: ColoredBox(color: Colors.blue),
)
```

---

# Summary

SizedBox creates boxes with fixed dimensions, ideal for spacing and constraining children. Use SizedBox for simple sizing, spacing between widgets, and creating uniform layouts. SizedBox.expand fills available space, while SizedBox.shrink creates an empty box.

---

# Next Steps

- [Expanded](expanded.md)
- [Flexible](flexible.md)
- [Spacer](spacer.md)

---

# Did You Know?

- SizedBox is one of the most used widgets
- SizedBox.expand fills parent
- SizedBox.shrink takes zero space
- SizedBox is lighter than Container
- SizedBox can be used for spacing
- SizedBox constrains child size
- SizedBox works in any layout
- SizedBox creates responsive gaps