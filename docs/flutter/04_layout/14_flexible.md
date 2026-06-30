# Flexible

Understand how to create flexible widgets that can expand or shrink.

---

# What is it?

Flexible is a widget that controls how a child of a Row, Column, or Flex flexes to fill available space. Unlike Expanded which always fills space, Flexible can be configured to either fill space (tight fit) or only take as much space as needed (loose fit). This gives you more control over flex behavior.

---

# Why does it exist?

Flexible exists to:

- Control flex behavior with more precision
- Allow children to size naturally
- Provide both tight and loose fitting
- Create responsive layouts
- Handle content-based sizing
- Prevent overflow with natural sizing
- Give more control than Expanded

---

# Flexible vs Expanded

> **Flexible offers more control** than Expanded.

```dart
// Flexible vs Expanded comparison
class FlexibleVsExpanded extends StatelessWidget {
  const FlexibleVsExpanded({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Expanded - always fills space
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Expanded(
                child: Container(
                  color: Colors.red,
                  child: const Center(
                    child: Text('Expanded'),
                  ),
                ),
              ),
              Expanded(
                child: Container(
                  color: Colors.blue,
                  child: const Center(
                    child: Text('Expanded'),
                  ),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Flexible with tight fit (same as Expanded)
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.green,
                  child: const Center(
                    child: Text('Tight'),
                  ),
                ),
              ),
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.orange,
                  child: const Center(
                    child: Text('Tight'),
                  ),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Flexible with loose fit (natural size)
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.purple,
                  child: const Text('Loose'),
                ),
              ),
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.pink,
                  child: const Text('Loose'),
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// Expanded = Flexible(fit: FlexFit.tight)
// Flexible gives more options
// Tight fills space, Loose sizes to content
```

What's happening here?
- Expanded: always fills space
- Flexible tight: same as Expanded
- Flexible loose: sizes to content
- More control with Flexible

---

# FlexFit Types

> **Two types of fit** control flexible behavior.

```dart
// FlexFit types in detail
class FlexFitTypes extends StatelessWidget {
  const FlexFitTypes({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Tight fit - Fills available space
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.red,
                  height: 50,
                  child: const Center(
                    child: Text('Tight - Fills'),
                  ),
                ),
              ),
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.blue,
                  height: 30,
                  child: const Center(
                    child: Text('Tight - Fills'),
                  ),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Loose fit - Sizes to content
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.green,
                  height: 50,
                  child: const Center(
                    child: Text('Loose - Sizes'),
                  ),
                ),
              ),
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.orange,
                  height: 30,
                  child: const Center(
                    child: Text('Loose - Sizes'),
                  ),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Mixed tight and loose
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.purple,
                  child: const Center(
                    child: Text('Tight'),
                  ),
                ),
              ),
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.pink,
                  child: const Text('Loose'),
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// FlexFit comparison:
// Tight: Forces child to fill space
// Loose: Allows child to size naturally
// Tight = Expanded behavior
// Loose = More flexible behavior
```

What's happening here?
- Tight: child forced to fill
- Loose: child sizes naturally
- Can mix tight and loose
- Choose based on need

---

# Flexible with Content

> **Flexible adapts** to content size.

```dart
// Flexible with varying content
class FlexibleWithContent extends StatelessWidget {
  const FlexibleWithContent({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Different text lengths with loose fit
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.red[100],
                  padding: const EdgeInsets.all(8),
                  child: const Text('Short'),
                ),
              ),
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.green[100],
                  padding: const EdgeInsets.all(8),
                  child: const Text('Medium length text'),
                ),
              ),
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.blue[100],
                  padding: const EdgeInsets.all(8),
                  child: const Text('Very long text that takes up more space'),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Tight fit ignores content size
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.red[100],
                  padding: const EdgeInsets.all(8),
                  child: const Text('Short'),
                ),
              ),
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.green[100],
                  padding: const EdgeInsets.all(8),
                  child: const Text('Medium length text'),
                ),
              ),
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.blue[100],
                  padding: const EdgeInsets.all(8),
                  child: const Text('Very long text that takes up more space'),
                ),
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
- Loose: content determines size
- Tight: space determines size
- Loose prevents overflow
- Tight ensures equal space

---

# Flexible with Flex Factor

> **Flex factors** work with Flexible too.

```dart
// Flexible with flex factors
class FlexibleWithFlex extends StatelessWidget {
  const FlexibleWithFlex({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Tight with different flex factors
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                flex: 2,
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.red,
                  child: const Center(child: Text('2')),
                ),
              ),
              Flexible(
                flex: 1,
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.blue,
                  child: const Center(child: Text('1')),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Loose with different flex factors
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                flex: 2,
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.green,
                  child: const Text('Flex 2 Loose'),
                ),
              ),
              Flexible(
                flex: 1,
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.orange,
                  child: const Text('Flex 1 Loose'),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Mixed flex and fit
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                flex: 2,
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.purple,
                  child: const Center(child: Text('Tight 2')),
                ),
              ),
              Flexible(
                flex: 1,
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.pink,
                  child: const Text('Loose 1'),
                ),
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
- Flex factors work with Flexible
- Can combine with fit types
- More control over space
- Flexible and proportional

---

# Real-World Examples

> **Common patterns** using Flexible.

```dart
// 1. Responsive text layout
class ResponsiveTextLayout extends StatelessWidget {
  const ResponsiveTextLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.grey[200],
      child: Row(
        children: [
          // Label takes as much as needed
          Flexible(
            fit: FlexFit.loose,
            child: Text(
              'Label:',
              style: const TextStyle(
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          const SizedBox(width: 8),
          // Value takes remaining space if needed
          Flexible(
            fit: FlexFit.tight,
            child: Text(
              'This is a very long value that might need more space',
              overflow: TextOverflow.ellipsis,
            ),
          ),
        ],
      ),
    );
  }
}

// 2. Icon with text
class IconWithText extends StatelessWidget {
  const IconWithText({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.grey[200],
      child: Row(
        children: [
          const Icon(Icons.star, size: 24),
          const SizedBox(width: 8),
          // Text takes remaining space but wraps if needed
          Flexible(
            fit: FlexFit.loose,
            child: Text(
              'This is a very long text that might need to wrap',
              softWrap: true,
            ),
          ),
        ],
      ),
    );
  }
}

// 3. Dynamic chip layout
class DynamicChipLayout extends StatelessWidget {
  const DynamicChipLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.grey[200],
      child: Wrap(
        spacing: 8,
        runSpacing: 8,
        children: [
          'Flutter', 'Dart', 'Mobile', 'Web', 'Desktop',
          'iOS', 'Android', 'Development', 'UI', 'UX'
        ].map((label) {
          return Flexible(
            fit: FlexFit.loose,
            child: Chip(
              label: Text(label),
              backgroundColor: Colors.blue[100],
            ),
          );
        }).toList(),
      ),
    );
  }
}

// 4. Flexible form layout
class FlexibleFormLayout extends StatelessWidget {
  const FlexibleFormLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      child: Column(
        children: [
          // Label and input on same line
          Row(
            children: [
              Flexible(
                fit: FlexFit.loose,
                child: const Text(
                  'Name:',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
              ),
              const SizedBox(width: 8),
              Flexible(
                fit: FlexFit.tight,
                child: const TextField(
                  decoration: InputDecoration(
                    border: OutlineInputBorder(),
                    contentPadding: EdgeInsets.symmetric(horizontal: 8),
                  ),
                ),
              ),
            ],
          ),
          const SizedBox(height: 16),
          // Another row with flexible input
          Row(
            children: [
              Flexible(
                fit: FlexFit.loose,
                child: const Text(
                  'Email:',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
              ),
              const SizedBox(width: 8),
              Flexible(
                fit: FlexFit.tight,
                child: const TextField(
                  decoration: InputDecoration(
                    border: OutlineInputBorder(),
                    contentPadding: EdgeInsets.symmetric(horizontal: 8),
                  ),
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Label and value with flexible spacing
- Icon with wrapping text
- Dynamic chip layout
- Flexible form rows

---

# Best Practices

## Use Flexible for Natural Sizing

```dart
// Good - Loose fit for natural sizing
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Flexible(
        fit: FlexFit.loose,
        child: Text('Natural size'),
      ),
    ],
  );
}
```

## Use Flexible to Prevent Overflow

```dart
// Good - Prevents overflow
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      const Icon(Icons.star),
      const SizedBox(width: 8),
      Flexible(
        child: Text(
          'Long text that might overflow',
          overflow: TextOverflow.ellipsis,
        ),
      ),
    ],
  );
}
```

## Choose Appropriate Fit

```dart
// Good - Choose based on need
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      // Tight: fill space
      Flexible(
        fit: FlexFit.tight,
        child: Container(color: Colors.red),
      ),
      // Loose: size naturally
      Flexible(
        fit: FlexFit.loose,
        child: Text('Content'),
      ),
    ],
  );
}
```

---

# Common Mistakes

## Using Flexible When Expanded Works

Wrong:
```dart
// Overcomplicating
Flexible(
  fit: FlexFit.tight,
  child: Container(color: Colors.blue),
)
```

Correct:
```dart
// Simpler
Expanded(
  child: Container(color: Colors.blue),
)
```

## Not Choosing Fit Type

Wrong:
```dart
// Default is tight (same as Expanded)
Flexible(
  child: Text('Default tight'),
)
```

Correct:
```dart
// Explicit choice
Flexible(
  fit: FlexFit.loose,
  child: Text('Loose fit'),
)
```

---

# Summary

Flexible provides more control than Expanded with tight and loose fit options. Tight fit forces filling space (like Expanded), while loose fit allows natural sizing. Use Flexible when you need content-based sizing or prevent overflow.

---

# Next Steps

- [Spacer](spacer.md)
- [AspectRatio](aspectratio.md)
- [FractionallySizedBox](fractionallysizedbox.md)

---

# Did You Know?

- Flexible tight = Expanded
- Flexible loose allows natural sizing
- Flex factors work with Flexible
- Flexible helps prevent overflow
- Flexible is more flexible than Expanded
- Choose fit based on needs
- Flexible can be used with any child
- Flexible is often better for text