# Spacer

Understand how to create flexible spacing between widgets.

---

# What is it?

Spacer is a widget that creates flexible space between children of a Row, Column, or Flex. It takes up all available space in its direction, pushing other widgets apart. Spacer is the simplest way to add flexible spacing without using Expanded with empty containers.

---

# Why does it exist?

Spacer exists to:

- Create flexible spacing between widgets
- Push widgets apart in flex layouts
- Distribute available space
- Simplify layout spacing
- Replace Expanded with empty Container
- Create responsive gaps
- Align widgets to ends

---

# Basic Spacer

> **Spacer** takes up available space.

```dart
// Basic Spacer usage
class BasicSpacer extends StatelessWidget {
  const BasicSpacer({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Spacer in Row
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(width: 50, height: 50, color: Colors.red),
              const Spacer(),
              Container(width: 50, height: 50, color: Colors.blue),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Spacer in Column
        Container(
          height: 150,
          color: Colors.grey[200],
          child: Column(
            children: [
              Container(height: 30, color: Colors.red),
              const Spacer(),
              Container(height: 30, color: Colors.green),
              const Spacer(),
              Container(height: 30, color: Colors.blue),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 3. Multiple Spacers
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(width: 50, height: 50, color: Colors.red),
              const Spacer(),
              Container(width: 50, height: 50, color: Colors.green),
              const Spacer(),
              Container(width: 50, height: 50, color: Colors.blue),
            ],
          ),
        ),
      ],
    );
  }
}

// Spacer properties:
// 1. flex - How much space to take (default: 1)
// 2. child - No child (Spacer is empty)
```

What's happening here?
- Spacer takes available space
- Pushes widgets to ends
- Multiple Spacers share space
- Works in Row and Column
- Simple and clean

---

# Spacer vs SizedBox

> **Comparing Spacer** with SizedBox for spacing.

```dart
// Spacer vs SizedBox
class SpacerVsSizedBox extends StatelessWidget {
  const SpacerVsSizedBox({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Spacer - Flexible space
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Text('Left'),
              const Spacer(),
              const Text('Right'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. SizedBox - Fixed space
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Text('Left'),
              const SizedBox(width: 100), // Fixed space
              const Text('Right'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Multiple Spacers with flex
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Text('Left'),
              const Spacer(flex: 2),
              const Text('Center'),
              const Spacer(flex: 1),
              const Text('Right'),
            ],
          ),
        ),
      ],
    );
  }
}

// When to use:
// Spacer: Flexible, responsive spacing
// SizedBox: Fixed, predictable spacing
// Spacer adapts to available space
// SizedBox has fixed size
```

What's happening here?
- Spacer: flexible, responsive
- SizedBox: fixed, predictable
- Spacer adapts to parent
- SizedBox has fixed dimension
- Choose based on need

---

# Spacer with Flex Factors

> **Flex factors** control space distribution.

```dart
// Spacer with flex factors
class SpacerFlexFactor extends StatelessWidget {
  const SpacerFlexFactor({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Equal flex (1:1)
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Text('Left'),
              const Spacer(),
              const Text('Center'),
              const Spacer(),
              const Text('Right'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Different flex (2:1)
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Text('Left'),
              const Spacer(flex: 2),
              const Text('Center'),
              const Spacer(flex: 1),
              const Text('Right'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Single Spacer
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Text('Left'),
              const Spacer(),
              const Text('Right'),
            ],
          ),
        ),
      ],
    );
  }
}

// Flex factor effects:
// Higher flex = more space
// Spacer(flex: 2) takes twice the space
// Useful for proportional spacing
```

What's happening here?
- flex controls space amount
- Higher flex = more space
- Proportional distribution
- Flexible spacing control

---

# Spacer in Different Layouts

> **Spacer works** in various flex layouts.

```dart
// Spacer in different contexts
class SpacerInLayouts extends StatelessWidget {
  const SpacerInLayouts({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. App bar with Spacer
        Container(
          height: 56,
          color: Colors.blue,
          padding: const EdgeInsets.symmetric(horizontal: 16),
          child: Row(
            children: [
              const Icon(Icons.menu, color: Colors.white),
              const SizedBox(width: 16),
              const Text(
                'Title',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 20,
                ),
              ),
              const Spacer(),
              const Icon(Icons.search, color: Colors.white),
              const SizedBox(width: 16),
              const Icon(Icons.more_vert, color: Colors.white),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Centered with Spacer
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Spacer(),
              const Text('Center'),
              const Spacer(),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 3. Three sections with Spacer
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(
                width: 80,
                color: Colors.red,
                child: const Center(child: Text('Left')),
              ),
              const Spacer(),
              Container(
                width: 80,
                color: Colors.green,
                child: const Center(child: Text('Center')),
              ),
              const Spacer(),
              Container(
                width: 80,
                color: Colors.blue,
                child: const Center(child: Text('Right')),
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
- App bar with Spacer
- Centering with Spacer
- Three-section layout
- Flexible and clean

---

# Real-World Examples

> **Common patterns** using Spacer.

```dart
// 1. App bar with actions
class AppBarWithActions extends StatelessWidget {
  const AppBarWithActions({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 56,
      color: Colors.blue,
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Row(
        children: [
          const Icon(Icons.arrow_back, color: Colors.white),
          const SizedBox(width: 16),
          const Text(
            'Screen Title',
            style: TextStyle(
              color: Colors.white,
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          const Spacer(),
          IconButton(
            icon: const Icon(Icons.search, color: Colors.white),
            onPressed: () {},
          ),
          IconButton(
            icon: const Icon(Icons.more_vert, color: Colors.white),
            onPressed: () {},
          ),
        ],
      ),
    );
  }
}

// 2. Bottom action bar
class BottomActionBar extends StatelessWidget {
  const BottomActionBar({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 80,
      padding: const EdgeInsets.symmetric(horizontal: 16),
      decoration: BoxDecoration(
        color: Colors.white,
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 8,
          ),
        ],
      ),
      child: Row(
        children: [
          const Icon(Icons.favorite_border, size: 30),
          const SizedBox(width: 16),
          const Icon(Icons.comment_outlined, size: 30),
          const Spacer(),
          ElevatedButton(
            onPressed: () {},
            child: const Text('Buy Now'),
          ),
        ],
      ),
    );
  }
}

// 3. Profile header
class ProfileHeader extends StatelessWidget {
  const ProfileHeader({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: [
          const CircleAvatar(
            radius: 30,
            backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
          ),
          const SizedBox(width: 16),
          const Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                'John Doe',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
              Text(
                'Flutter Developer',
                style: TextStyle(
                  color: Colors.grey,
                  fontSize: 14,
                ),
              ),
            ],
          ),
          const Spacer(),
          IconButton(
            icon: const Icon(Icons.edit),
            onPressed: () {},
          ),
        ],
      ),
    );
  }
}

// 4. Quiz progress bar
class QuizProgress extends StatelessWidget {
  const QuizProgress({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.grey[100],
      child: Row(
        children: [
          const Text(
            'Question 3 of 10',
            style: TextStyle(
              fontWeight: FontWeight.bold,
            ),
          ),
          const Spacer(),
          // Progress dots
          ...List.generate(5, (index) {
            return Container(
              width: 12,
              height: 12,
              margin: const EdgeInsets.symmetric(horizontal: 4),
              decoration: BoxDecoration(
                color: index < 3 ? Colors.blue : Colors.grey[300],
                shape: BoxShape.circle,
              ),
            );
          }),
          const Spacer(),
          const Text(
            '75%',
            style: TextStyle(
              fontWeight: FontWeight.bold,
              color: Colors.blue,
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- App bar with actions pushed right
- Bottom bar with centered actions
- Profile header with edit button
- Quiz progress with spacing

---

# Best Practices

## Use Spacer for Flexible Spacing

```dart
// Good - Flexible spacing
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      const Text('Left'),
      const Spacer(),
      const Text('Right'),
    ],
  );
}
```

## Use Multiple Spacers for Distribution

```dart
// Good - Multiple spacers
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      const Text('Left'),
      const Spacer(),
      const Text('Center'),
      const Spacer(),
      const Text('Right'),
    ],
  );
}
```

## Use Flex Factors for Proportions

```dart
// Good - Proportional spacing
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      const Text('Left'),
      const Spacer(flex: 2),
      const Text('Center'),
      const Spacer(flex: 1),
      const Text('Right'),
    ],
  );
}
```

---

# Common Mistakes

## Using Spacer Outside Flex

Wrong:
```dart
// Error: Spacer must be inside Flex
Column(
  children: [
    Text('Top'),
    const Spacer(), // Error
    Text('Bottom'),
  ],
)
```

Correct:
```dart
// Spacer inside Flex
Column(
  children: [
    const Text('Top'),
    const Expanded(
      child: Center(
        child: Text('Middle'),
      ),
    ),
    const Text('Bottom'),
  ],
)
```

## Overusing Spacer

Wrong:
```dart
// Too many spacers
Row(
  children: [
    const Spacer(),
    const Spacer(), // Redundant
    const Text('Center'),
    const Spacer(),
    const Spacer(), // Redundant
  ],
)
```

Correct:
```dart
// Clean spacing
Row(
  children: [
    const Spacer(),
    const Text('Center'),
    const Spacer(),
  ],
)
```

---

# Summary

Spacer creates flexible spacing in flex layouts. Use Spacer for responsive gaps, pushing widgets to ends, and distributing space. Spacer is cleaner than Expanded with empty containers and adapts to available space.

---

# Next Steps

- [AspectRatio](aspectratio.md)
- [FractionallySizedBox](fractionallysizedbox.md)
- [Intrinsic Widgets](intrinsic-widgets.md)

---

# Did You Know?

- Spacer = Expanded with empty child
- Spacer only works in Flex (Row/Column)
- flex controls space distribution
- Spacer adapts to available space
- Multiple Spacers share space
- Spacer is more readable than Expanded
- Spacer has no child
- Spacer is responsive by default