# Box Model

Understand the box model and how Flutter handles spacing and sizing.

---

# What is it?

The box model in Flutter is the system for managing the space around and within widgets. It includes padding (space inside the box), margin (space outside the box), and the content itself. Understanding the box model helps you control spacing, alignment, and sizing in your layouts.

---

# Why does it exist?

The box model exists to:

- Control spacing between widgets
- Manage internal and external spacing
- Create consistent layouts
- Handle sizing and constraints
- Support responsive design
- Enable visual hierarchy
- Provide flexible layout options

---

# The Box Model Components

> **The box model** consists of four main components.

```
┌─────────────────────────────────────┐
│             Margin                   │
│  ┌─────────────────────────────────┐ │
│  │          Border                 │ │
│  │  ┌───────────────────────────┐  │ │
│  │  │        Padding            │  │ │
│  │  │  ┌─────────────────────┐  │  │ │
│  │  │  │      Content        │  │  │ │
│  │  │  │                     │  │  │ │
│  │  │  └─────────────────────┘  │  │ │
│  │  └───────────────────────────┘  │ │
│  └─────────────────────────────────┘ │
└─────────────────────────────────────┘

// Components:
// 1. Content - The actual widget content
// 2. Padding - Space inside the box
// 3. Border - Border around the box
// 4. Margin - Space outside the box
```

What's happening here?
- Content is the innermost element
- Padding adds space inside the border
- Border wraps around padding and content
- Margin adds space outside the border
- All components affect layout

---

# Container and Box Model

> **Container** implements the full box model.

```dart
// Container with full box model
class ContainerBoxModel extends StatelessWidget {
  const ContainerBoxModel({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      // 1. Margin - Space outside
      margin: const EdgeInsets.all(20),
      
      // 2. Padding - Space inside
      padding: const EdgeInsets.all(16),
      
      // 3. Decoration - Border and background
      decoration: BoxDecoration(
        color: Colors.blue,
        border: Border.all(
          color: Colors.black,
          width: 3,
        ),
        borderRadius: BorderRadius.circular(10),
      ),
      
      // 4. Content
      child: const Text(
        'Hello World',
        style: TextStyle(color: Colors.white),
      ),
    );
  }
}

// Visual breakdown:
// margin: 20px outside
// border: 3px border
// padding: 16px inside
// content: text
// Total space: margin + border + padding + content
```

What's happening here?
- Container provides full box model
- Margin creates external spacing
- Padding creates internal spacing
- Decoration adds border and background
- Child is the content

---

# EdgeInsets

> **EdgeInsets** controls padding and margin.

```dart
// EdgeInsets variations
class EdgeInsetsExample extends StatelessWidget {
  const EdgeInsetsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. All sides same
        Container(
          margin: const EdgeInsets.all(16),
          padding: const EdgeInsets.all(8),
          color: Colors.blue,
          child: const Text('All sides'),
        ),
        
        // 2. Individual sides
        Container(
          margin: const EdgeInsets.only(
            left: 10,
            right: 20,
            top: 5,
            bottom: 15,
          ),
          padding: const EdgeInsets.fromLTRB(10, 5, 20, 15),
          color: Colors.red,
          child: const Text('Individual sides'),
        ),
        
        // 3. Symmetric (horizontal/vertical)
        Container(
          margin: const EdgeInsets.symmetric(
            horizontal: 20,
            vertical: 10,
          ),
          padding: const EdgeInsets.symmetric(
            horizontal: 15,
            vertical: 5,
          ),
          color: Colors.green,
          child: const Text('Symmetric'),
        ),
        
        // 4. Only specific sides
        Container(
          margin: const EdgeInsets.only(
            left: 20,
            right: 20,
          ),
          padding: const EdgeInsets.only(
            top: 10,
            bottom: 10,
          ),
          color: Colors.orange,
          child: const Text('Only specific sides'),
        ),
        
        // 5. Zero (no spacing)
        Container(
          margin: EdgeInsets.zero,
          padding: EdgeInsets.zero,
          color: Colors.purple,
          child: const Text('No spacing'),
        ),
      ],
    );
  }
}

// EdgeInsets methods:
// 1. EdgeInsets.all(double value)
// 2. EdgeInsets.only({left, right, top, bottom})
// 3. EdgeInsets.fromLTRB(left, top, right, bottom)
// 4. EdgeInsets.symmetric({horizontal, vertical})
// 5. EdgeInsets.zero
```

What's happening here?
- all() applies same value to all sides
- only() sets specific sides
- fromLTRB() sets all sides individually
- symmetric() sets horizontal/vertical
- zero creates no spacing

---

# BoxDecoration

> **BoxDecoration** adds visual styling.

```dart
// BoxDecoration properties
class BoxDecorationExample extends StatelessWidget {
  const BoxDecorationExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        // 1. Color
        color: Colors.blue,
        
        // 2. Border
        border: Border.all(
          color: Colors.black,
          width: 2,
          style: BorderStyle.solid,
        ),
        
        // 3. BorderRadius
        borderRadius: BorderRadius.circular(10),
        // or individual corners:
        // borderRadius: BorderRadius.only(
        //   topLeft: Radius.circular(10),
        //   topRight: Radius.circular(10),
        // ),
        
        // 4. BoxShadow
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.3),
            offset: const Offset(5, 5),
            blurRadius: 10,
            spreadRadius: 2,
          ),
        ],
        
        // 5. Gradient
        gradient: const LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: [Colors.blue, Colors.purple],
        ),
        
        // 6. Shape
        shape: BoxShape.rectangle,
        // or BoxShape.circle
        
        // 7. Image
        // image: DecorationImage(
        //   image: AssetImage('assets/image.png'),
        //   fit: BoxFit.cover,
        // ),
      ),
      child: const Padding(
        padding: EdgeInsets.all(20),
        child: Text(
          'Styled Container',
          style: TextStyle(color: Colors.white),
        ),
      ),
    );
  }
}

// Decoration types:
// 1. BoxDecoration - Most common
// 2. ShapeDecoration - For shapes
// 3. FlutterLogoDecoration - For logo
// 4. CustomPainter - Custom decoration
```

What's happening here?
- Color fills the background
- Border surrounds the container
- BorderRadius rounds corners
- BoxShadow adds shadow effects
- Gradient creates color transitions
- Shape changes box shape

---

# Spacing Widgets

> **Widgets for spacing** in Flutter layouts.

```dart
// 1. Padding - Adds internal space
class PaddingExample extends StatelessWidget {
  const PaddingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(20),
      child: Container(
        color: Colors.blue,
        child: const Text('Padded content'),
      ),
    );
  }
}

// 2. Margin - Using Container
class MarginExample extends StatelessWidget {
  const MarginExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.all(20),
      color: Colors.blue,
      child: const Text('Content with margin'),
    );
  }
}

// 3. SizedBox - Fixed spacing
class SizedBoxExample extends StatelessWidget {
  const SizedBoxExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text('Top text'),
        // Fixed space
        const SizedBox(height: 20),
        const Text('Middle text'),
        // Fixed space
        const SizedBox(width: 50, height: 50),
        const Text('Bottom text'),
      ],
    );
  }
}

// 4. Spacer - Flexible spacing
class SpacerExample extends StatelessWidget {
  const SpacerExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        const Text('Left'),
        // Takes available space
        const Spacer(),
        const Text('Center'),
        const Spacer(),
        const Text('Right'),
      ],
    );
  }
}

// 5. Expanded - Flexible with constraints
class ExpandedExample extends StatelessWidget {
  const ExpandedExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Container(
          width: 50,
          color: Colors.red,
          child: const Text('Fixed'),
        ),
        Expanded(
          child: Container(
            color: Colors.blue,
            child: const Text('Flexible'),
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- Padding adds internal spacing
- Container margin adds external spacing
- SizedBox adds fixed spacing
- Spacer adds flexible spacing
- Expanded takes available space

---

# Customizing Box Model

> **Custom box model** for specific needs.

```dart
// Custom box model example
class CustomBoxModel extends StatelessWidget {
  const CustomBoxModel({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      // Custom margin
      margin: const EdgeInsets.only(
        left: 20,
        right: 20,
        top: 10,
        bottom: 10,
      ),
      
      // Custom padding
      padding: const EdgeInsets.only(
        left: 15,
        right: 15,
        top: 8,
        bottom: 8,
      ),
      
      // Custom decoration
      decoration: BoxDecoration(
        color: Colors.blue,
        border: Border(
          left: BorderSide(
            color: Colors.red,
            width: 5,
          ),
          right: BorderSide(
            color: Colors.green,
            width: 5,
          ),
          top: BorderSide(
            color: Colors.yellow,
            width: 5,
          ),
          bottom: BorderSide(
            color: Colors.purple,
            width: 5,
          ),
        ),
        borderRadius: const BorderRadius.only(
          topLeft: Radius.circular(10),
          bottomRight: Radius.circular(10),
        ),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.2),
            blurRadius: 10,
            spreadRadius: 2,
          ),
        ],
      ),
      
      child: const Text(
        'Custom Box',
        style: TextStyle(
          color: Colors.white,
          fontSize: 18,
        ),
      ),
    );
  }
}

// Box model calculations:
// Total width = margin.left + border.left + padding.left + content.width + padding.right + border.right + margin.right
// Total height = margin.top + border.top + padding.top + content.height + padding.bottom + border.bottom + margin.bottom
```

What's happening here?
- Custom margins and padding
- Individual border sides
- Specific border radius
- Shadow effects
- Full box model control

---

# Common Layout Patterns

> **Common patterns** using the box model.

```dart
// 1. Card with elevation
class CardPattern extends StatelessWidget {
  const CardPattern({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.all(10),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(10),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 10,
            offset: const Offset(0, 5),
          ),
        ],
      ),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Card Title',
              style: TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            const Text(
              'Card content goes here. This is a typical card pattern.',
            ),
          ],
        ),
      ),
    );
  }
}

// 2. Button with padding
class ButtonPattern extends StatelessWidget {
  const ButtonPattern({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.all(10),
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(8),
      ),
      child: const Padding(
        padding: EdgeInsets.symmetric(
          horizontal: 20,
          vertical: 12,
        ),
        child: Text(
          'Click Me',
          style: TextStyle(
            color: Colors.white,
            fontSize: 16,
          ),
        ),
      ),
    );
  }
}

// 3. Section with spacing
class SectionPattern extends StatelessWidget {
  const SectionPattern({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Padding(
          padding: EdgeInsets.all(16),
          child: Text(
            'Section Title',
            style: TextStyle(
              fontSize: 24,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
        const Divider(height: 1),
        Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              Container(
                margin: const EdgeInsets.only(bottom: 10),
                padding: const EdgeInsets.all(12),
                color: Colors.grey[200],
                child: const Text('Item 1'),
              ),
              Container(
                margin: const EdgeInsets.only(bottom: 10),
                padding: const EdgeInsets.all(12),
                color: Colors.grey[200],
                child: const Text('Item 2'),
              ),
              Container(
                padding: const EdgeInsets.all(12),
                color: Colors.grey[200],
                child: const Text('Item 3'),
              ),
            ],
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- Card with shadow and padding
- Button with internal spacing
- Section with consistent spacing
- Common UI patterns
- Reusable designs

---

# Best Practices

## Use Consistent Spacing

```dart
// Good - Consistent spacing
class ConsistentSpacing extends StatelessWidget {
  const ConsistentSpacing({super.key});

  static const double spacing = 8;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const SizedBox(height: spacing),
        Container(
          padding: const EdgeInsets.all(spacing),
          child: const Text('Item 1'),
        ),
        const SizedBox(height: spacing),
        Container(
          padding: const EdgeInsets.all(spacing),
          child: const Text('Item 2'),
        ),
      ],
    );
  }
}

// Bad - Inconsistent spacing
class InconsistentSpacing extends StatelessWidget {
  const InconsistentSpacing({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const SizedBox(height: 5),
        Container(
          padding: const EdgeInsets.all(10),
          child: const Text('Item 1'),
        ),
        const SizedBox(height: 15),
        Container(
          padding: const EdgeInsets.all(8),
          child: const Text('Item 2'),
        ),
      ],
    );
  }
}
```

## Use EdgeInsets Constants

```dart
// Good - Using constants
class SpacingConstants {
  static const EdgeInsets cardPadding = EdgeInsets.all(16);
  static const EdgeInsets buttonPadding = EdgeInsets.symmetric(
    horizontal: 20,
    vertical: 12,
  );
}

// Bad - Hardcoding values everywhere
@override
Widget build(BuildContext context) {
  return Padding(
    padding: const EdgeInsets.all(16), // Repeated
    child: Container(
      padding: const EdgeInsets.all(12), // Different
      child: const Text('Content'),
    ),
  );
}
```

## Avoid Overuse of Container

```dart
// Good - Using specific widgets
@override
Widget build(BuildContext context) {
  return Padding(
    padding: const EdgeInsets.all(16),
    child: const Text('Content'),
  );
}

// Bad - Using Container for everything
@override
Widget build(BuildContext context) {
  return Container(
    padding: const EdgeInsets.all(16),
    child: const Text('Content'),
  );
}
```

---

# Common Mistakes

## Incorrect Margin Usage

Wrong:
```dart
// Margin on child to separate from parent
Container(
  child: Container(
    margin: const EdgeInsets.all(20), // Better on parent
    color: Colors.blue,
    child: const Text('Content'),
  ),
)
```

Correct:
```dart
// Margin on parent or using Padding
Padding(
  padding: const EdgeInsets.all(20),
  child: Container(
    color: Colors.blue,
    child: const Text('Content'),
  ),
)
```

## Not Considering Box Model

Wrong:
```dart
// Text might overflow due to padding
Container(
  padding: const EdgeInsets.all(20),
  width: 100,
  child: const Text(
    'Very long text that might overflow',
  ),
)
```

Correct:
```dart
// Account for padding in constraints
Container(
  padding: const EdgeInsets.all(20),
  width: 100, // Width includes padding
  child: const Text(
    'Long text',
    overflow: TextOverflow.ellipsis,
  ),
)
```

---

# Summary

The box model controls spacing around and within widgets through margin, border, padding, and content. Understanding these components helps create consistent, well-spaced layouts. Use EdgeInsets for spacing, BoxDecoration for styling, and follow best practices for maintainable code.

---

# Next Steps

- [Flex Layout](flex-layout.md)
- [Row](row.md)
- [Column](column.md)

---

# Did You Know?

- Container implements the full box model
- EdgeInsets provides 5 different constructors
- Margin and padding are both EdgeInsets
- BoxDecoration supports gradient and shadows
- The box model affects layout calculations
- CSS and Flutter box models are similar
- Padding adds space inside, margin adds space outside