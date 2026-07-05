# Intrinsic Widgets

Understand how to size widgets based on their intrinsic dimensions.

---

# What is it?

Intrinsic widgets (IntrinsicWidth and IntrinsicHeight) are widgets that size themselves based on their children's intrinsic dimensions. They allow you to create layouts where the size of a parent is determined by the natural size of its children, even when the parent normally wouldn't have that information. This is useful for creating flexible, content-aware layouts.

---

# Why does it exist?

Intrinsic widgets exist to:

- Size parents based on children's natural dimensions
- Create content-aware layouts
- Handle complex layout scenarios
- Enable flexible sizing
- Solve specific layout problems
- Make widgets adapt to content
- Provide more layout control

---

# Basic IntrinsicWidth

> **IntrinsicWidth** sizes itself to fit its child's intrinsic width.

```dart
// Basic IntrinsicWidth usage
class BasicIntrinsicWidth extends StatelessWidget {
  const BasicIntrinsicWidth({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Without IntrinsicWidth
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(color: Colors.red, height: 50),
              Container(color: Colors.blue, height: 50),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. With IntrinsicWidth
        Container(
          color: Colors.grey[200],
          child: IntrinsicWidth(
            child: Row(
              children: [
                Container(color: Colors.red, height: 50),
                Container(color: Colors.blue, height: 50),
              ],
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 3. IntrinsicWidth with different children
        Container(
          color: Colors.grey[200],
          child: IntrinsicWidth(
            child: Row(
              children: [
                const Text('Short'),
                const Text('Very long text that determines width'),
              ],
            ),
          ),
        ),
      ],
    );
  }
}

// IntrinsicWidth properties:
// 1. child - The widget to size
// 2. stepWidth - Optional step value (default: null)
// 3. stepHeight - Optional step value (default: null)
```

What's happening here?
- IntrinsicWidth sizes to child's width
- Useful for content-aware layouts
- Children can be different sizes
- Parent adapts to content

---

# Basic IntrinsicHeight

> **IntrinsicHeight** sizes itself to fit its child's intrinsic height.

```dart
// Basic IntrinsicHeight usage
class BasicIntrinsicHeight extends StatelessWidget {
  const BasicIntrinsicHeight({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Without IntrinsicHeight
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(width: 50, height: 30, color: Colors.red),
              Container(width: 50, height: 60, color: Colors.green),
              Container(width: 50, height: 45, color: Colors.blue),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. With IntrinsicHeight
        Container(
          color: Colors.grey[200],
          child: IntrinsicHeight(
            child: Row(
              children: [
                Container(width: 50, height: 30, color: Colors.red),
                Container(width: 50, height: 60, color: Colors.green),
                Container(width: 50, height: 45, color: Colors.blue),
              ],
            ),
          ),
        ),
      ],
    );
  }
}

// IntrinsicHeight properties:
// 1. child - The widget to size
// 2. stepHeight - Optional step value (default: null)
```

What's happening here?
- IntrinsicHeight sizes to child's height
- Makes children equal height
- Parent adapts to tallest child
- Useful for consistent heights

---

# IntrinsicWidth vs SizedBox

> **Comparing** IntrinsicWidth with fixed sizing.

```dart
// IntrinsicWidth vs SizedBox
class IntrinsicWidthComparison extends StatelessWidget {
  const IntrinsicWidthComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Fixed width with SizedBox
        Container(
          color: Colors.grey[200],
          child: SizedBox(
            width: 200,
            child: Row(
              children: [
                Container(color: Colors.red, height: 50),
                Container(color: Colors.blue, height: 50),
              ],
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Content-based width with IntrinsicWidth
        Container(
          color: Colors.grey[200],
          child: IntrinsicWidth(
            child: Row(
              children: [
                Container(color: Colors.green, height: 50),
                Container(color: Colors.orange, height: 50),
              ],
            ),
          ),
        ),
      ],
    );
  }
}

// When to use:
// SizedBox: Fixed width regardless of content
// IntrinsicWidth: Adapts to content width
// IntrinsicWidth is responsive to children
// SizedBox is fixed and predictable
```

What's happening here?
- SizedBox: fixed size
- IntrinsicWidth: adapts to content
- Choose based on need

---

# IntrinsicHeight with Different Content

> **IntrinsicHeight** handles varying child heights.

```dart
// IntrinsicHeight with different content
class IntrinsicHeightWithContent extends StatelessWidget {
  const IntrinsicHeightWithContent({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.grey[200],
      child: IntrinsicHeight(
        child: Row(
          children: [
            // Short widget
            Container(
              width: 80,
              color: Colors.red,
              padding: const EdgeInsets.all(8),
              child: const Text('Short'),
            ),
            
            // Medium widget
            Container(
              width: 80,
              color: Colors.green,
              padding: const EdgeInsets.all(8),
              child: const Column(
                children: [
                  Text('Line 1'),
                  Text('Line 2'),
                ],
              ),
            ),
            
            // Tall widget
            Container(
              width: 80,
              color: Colors.blue,
              padding: const EdgeInsets.all(8),
              child: const Column(
                children: [
                  Text('Line 1'),
                  Text('Line 2'),
                  Text('Line 3'),
                  Text('Line 4'),
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
- All children equal height
- Height determined by tallest child
- Content adapts to fit
- Consistent alignment

---

# IntrinsicWidth with Different Content

> **IntrinsicWidth** handles varying child widths.

```dart
// IntrinsicWidth with different content
class IntrinsicWidthWithContent extends StatelessWidget {
  const IntrinsicWidthWithContent({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.grey[200],
      child: IntrinsicWidth(
        child: Column(
          children: [
            // Short text
            Container(
              padding: const EdgeInsets.all(8),
              color: Colors.red,
              child: const Text('Short'),
            ),
            
            // Long text
            Container(
              padding: const EdgeInsets.all(8),
              color: Colors.green,
              child: const Text('Very long text that determines width'),
            ),
            
            // Medium text
            Container(
              padding: const EdgeInsets.all(8),
              color: Colors.blue,
              child: const Text('Medium text'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- All children equal width
- Width determined by widest child
- Content adapts to fit
- Consistent alignment

---

# Real-World Examples

> **Common patterns** using intrinsic widgets.

```dart
// 1. Equal-height cards
class EqualHeightCards extends StatelessWidget {
  const EqualHeightCards({super.key});

  @override
  Widget build(BuildContext context) {
    return IntrinsicHeight(
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          // Card 1 (short content)
          Expanded(
            child: Card(
              margin: const EdgeInsets.all(8),
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Card 1',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 18,
                      ),
                    ),
                    const SizedBox(height: 8),
                    const Text('Short content'),
                  ],
                ),
              ),
            ),
          ),
          
          // Card 2 (long content)
          Expanded(
            child: Card(
              margin: const EdgeInsets.all(8),
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Card 2',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 18,
                      ),
                    ),
                    const SizedBox(height: 8),
                    const Text(
                      'This card has much longer content '
                      'that spans multiple lines and will '
                      'determine the height of all cards.',
                    ),
                  ],
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// 2. Equal-width buttons
class EqualWidthButtons extends StatelessWidget {
  const EqualWidthButtons({super.key});

  @override
  Widget build(BuildContext context) {
    return IntrinsicWidth(
      child: Column(
        children: [
          ElevatedButton(
            onPressed: () {},
            child: const Text('Short'),
          ),
          const SizedBox(height: 8),
          ElevatedButton(
            onPressed: () {},
            child: const Text('Medium Button'),
          ),
          const SizedBox(height: 8),
          ElevatedButton(
            onPressed: () {},
            child: const Text('Very Long Button Text'),
          ),
        ],
      ),
    );
  }
}

// 3. Auto-sizing table
class AutoSizingTable extends StatelessWidget {
  const AutoSizingTable({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.grey[200],
      child: IntrinsicWidth(
        child: Column(
          children: [
            _buildRow('Name', 'John Doe'),
            _buildRow('Email', 'john.doe@example.com'),
            _buildRow('Phone', '+1 234 567 890'),
            _buildRow('Address', '123 Main Street, City'),
          ],
        ),
      ),
    );
  }

  Widget _buildRow(String label, String value) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey[300]!),
      ),
      child: Row(
        children: [
          Container(
            width: 100,
            child: Text(
              label,
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          const SizedBox(width: 16),
          Text(value),
        ],
      ),
    );
  }
}

// 4. Auto-sizing dialog
class AutoSizingDialog extends StatelessWidget {
  const AutoSizingDialog({super.key});

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text('Dialog Title'),
      content: IntrinsicHeight(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'This is a dialog that sizes itself '
              'based on its content height.',
            ),
            const SizedBox(height: 16),
            const Text('Some more content here.'),
            const SizedBox(height: 16),
            Row(
              children: [
                const Text('Name: '),
                Expanded(
                  child: Container(
                    color: Colors.grey[200],
                    child: const TextField(
                      decoration: InputDecoration(
                        border: InputBorder.none,
                        contentPadding: EdgeInsets.all(8),
                      ),
                    ),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
      actions: [
        TextButton(
          onPressed: () {},
          child: const Text('Cancel'),
        ),
        ElevatedButton(
          onPressed: () {},
          child: const Text('OK'),
        ),
      ],
    );
  }
}
```

What's happening here?
- Equal-height cards with varying content
- Equal-width buttons
- Auto-sizing table with labels
- Auto-sizing dialog with content

---

# Best Practices

## Use IntrinsicHeight for Equal Heights

```dart
// Good - Equal height columns
@override
Widget build(BuildContext context) {
  return IntrinsicHeight(
    child: Row(
      children: [
        Container(child: Text('Short')),
        Container(child: Text('Long content\nmultiple lines')),
      ],
    ),
  );
}
```

## Use IntrinsicWidth for Equal Widths

```dart
// Good - Equal width buttons
@override
Widget build(BuildContext context) {
  return IntrinsicWidth(
    child: Column(
      children: [
        Button('Short'),
        Button('Long Button Text'),
      ],
    ),
  );
}
```

## Avoid Overusing

```dart
// Good - Use when needed
@override
Widget build(BuildContext context) {
  return IntrinsicHeight(
    child: Row(children: varyingHeightChildren),
  );
}

// Bad - Unnecessary use
@override
Widget build(BuildContext context) {
  return IntrinsicHeight(
    child: Row(children: equalHeightChildren),
  );
}
```

---

# Common Mistakes

## Overusing Intrinsic Widgets

Wrong:
```dart
// Performance impact with many children
IntrinsicHeight(
  child: ListView.builder(
    itemCount: 1000,
    itemBuilder: (context, index) {
      return Container(
        height: 50 + index % 50, // Varying heights
      );
    },
  ),
)
```

Correct:
```dart
// Use for small number of children
IntrinsicHeight(
  child: Row(
    children: [
      // 3-4 children only
    ],
  ),
)
```

## Not Understanding Intrinsic Size

Wrong:
```dart
// IntrinsicHeight doesn't work with unbounded constraints
IntrinsicHeight(
  child: Column(
    children: [
      Container(height: 30),
      Container(height: 50),
      // Column with unbounded height
    ],
  ),
)
```

Correct:
```dart
// Use with bounded constraints
Container(
  height: 100, // Bounded height
  child: IntrinsicHeight(
    child: Row(
      children: [
        // Children with bounded parent
      ],
    ),
  ),
)
```

---

# Summary

Intrinsic widgets (IntrinsicWidth and IntrinsicHeight) size themselves based on their children's natural dimensions. Use them for equal-height/width layouts, auto-sizing containers, and content-aware designs. Be mindful of performance and use them when necessary.

---

# Next Steps

- [Custom Layouts](custom-layouts.md)
- [Layout Process](../framework-architecture/layout-process.md)
- [Constraints](../framework-architecture/constraints.md)

---

# Did You Know?

- IntrinsicWidth sizes to child's width
- IntrinsicHeight sizes to child's height
- Intrinsic widgets can impact performance
- Use for small numbers of children
- IntrinsicWidth can make children equal width
- IntrinsicHeight can make children equal height
- Step values control size increments
- Intrinsic widgets help with complex layouts