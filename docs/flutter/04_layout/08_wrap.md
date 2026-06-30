# Wrap

Understand how to create flexible, multi-line layouts with Wrap.

---

# What is it?

Wrap is a layout widget that arranges its children in a horizontal or vertical sequence and automatically wraps them to the next line or column when there isn't enough space. Unlike Row or Column, which try to fit all children in one direction, Wrap adapts to available space by creating multiple rows or columns as needed.

---

# Why does it exist?

Wrap exists to:

- Create flexible, responsive layouts
- Handle multiple children that need to wrap
- Automatically break to new lines when space runs out
- Support chips, tags, and flow layouts
- Create responsive UI without manual calculations
- Handle dynamic content that changes size

---

# Basic Wrap

> **Wrap** arranges children and wraps to new lines.

```dart
// Basic Wrap usage
class BasicWrap extends StatelessWidget {
  const BasicWrap({super.key});

  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 8, // Horizontal spacing between children
      runSpacing: 8, // Vertical spacing between rows
      children: [
        Chip(label: Text('Flutter')),
        Chip(label: Text('Dart')),
        Chip(label: Text('Mobile')),
        Chip(label: Text('Web')),
        Chip(label: Text('Desktop')),
        Chip(label: Text('iOS')),
        Chip(label: Text('Android')),
        Chip(label: Text('Development')),
      ],
    );
  }
}

// Wrap properties:
// 1. children - List of widgets
// 2. spacing - Space between children (horizontal)
// 3. runSpacing - Space between rows/columns (vertical)
// 4. direction - Axis.horizontal or Axis.vertical
// 5. alignment - How to align children
// 6. runAlignment - How to align rows/columns
// 7. crossAxisAlignment - How to align across axis
```

What's happening here?
- Children arranged horizontally
- Wrap to next line when space runs out
- spacing controls horizontal gap
- runSpacing controls vertical gap
- Adapts to available space

---

# Wrap Direction

> **Direction** controls whether Wrap arranges horizontally or vertically.

```dart
// Wrap direction examples
class WrapDirectionExample extends StatelessWidget {
  const WrapDirectionExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Horizontal (default)
        Container(
          width: 200,
          color: Colors.grey[200],
          child: Wrap(
            spacing: 8,
            runSpacing: 8,
            children: [
              _buildChip('Item 1'),
              _buildChip('Item 2'),
              _buildChip('Item 3'),
              _buildChip('Item 4'),
              _buildChip('Item 5'),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Vertical
        Container(
          height: 150,
          color: Colors.grey[200],
          child: Wrap(
            direction: Axis.vertical,
            spacing: 8,
            runSpacing: 8,
            children: [
              _buildChip('Item 1'),
              _buildChip('Item 2'),
              _buildChip('Item 3'),
              _buildChip('Item 4'),
              _buildChip('Item 5'),
              _buildChip('Item 6'),
              _buildChip('Item 7'),
              _buildChip('Item 8'),
            ],
          ),
        ),
      ],
    );
  }

  Widget _buildChip(String label) {
    return Chip(
      label: Text(label),
      backgroundColor: Colors.blue[100],
    );
  }
}
```

What's happening here?
- Horizontal wraps to next row
- Vertical wraps to next column
- Direction controls wrap behavior
- spacing controls along main axis
- runSpacing controls along cross axis

---

# Wrap Alignment

> **Alignment** controls how children are positioned.

```dart
// Wrap alignment examples
class WrapAlignmentExample extends StatelessWidget {
  const WrapAlignmentExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Start aligned
        _buildWrapWithTitle(
          'Start',
          alignment: WrapAlignment.start,
        ),
        
        // 2. Center aligned
        _buildWrapWithTitle(
          'Center',
          alignment: WrapAlignment.center,
        ),
        
        // 3. End aligned
        _buildWrapWithTitle(
          'End',
          alignment: WrapAlignment.end,
        ),
        
        // 4. SpaceBetween
        _buildWrapWithTitle(
          'SpaceBetween',
          alignment: WrapAlignment.spaceBetween,
        ),
        
        // 5. SpaceAround
        _buildWrapWithTitle(
          'SpaceAround',
          alignment: WrapAlignment.spaceAround,
        ),
        
        // 6. SpaceEvenly
        _buildWrapWithTitle(
          'SpaceEvenly',
          alignment: WrapAlignment.spaceEvenly,
        ),
      ],
    );
  }

  Widget _buildWrapWithTitle(String title, {required WrapAlignment alignment}) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(title),
        Container(
          width: 300,
          color: Colors.grey[200],
          child: Wrap(
            alignment: alignment,
            spacing: 8,
            runSpacing: 8,
            children: [
              _buildChip('Chip A'),
              _buildChip('Chip B'),
              _buildChip('Chip C'),
              _buildChip('Chip D'),
              _buildChip('Chip E'),
            ],
          ),
        ),
        const SizedBox(height: 10),
      ],
    );
  }

  Widget _buildChip(String label) {
    return Chip(label: Text(label));
  }
}

// Alignment options:
// 1. WrapAlignment.start - Align to start
// 2. WrapAlignment.center - Center align
// 3. WrapAlignment.end - Align to end
// 4. WrapAlignment.spaceBetween - Space between
// 5. WrapAlignment.spaceAround - Space around
// 6. WrapAlignment.spaceEvenly - Even space
```

What's happening here?
- start aligns children to beginning
- center centers children
- end aligns to end
- spaceBetween distributes space
- spaceAround spaces around each
- spaceEvenly distributes evenly

---

# Run Alignment

> **RunAlignment** controls how rows/columns are aligned.

```dart
// RunAlignment examples
class RunAlignmentExample extends StatelessWidget {
  const RunAlignmentExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Start
        _buildWrapWithRunTitle(
          'Start',
          runAlignment: WrapAlignment.start,
        ),
        
        // 2. Center
        _buildWrapWithRunTitle(
          'Center',
          runAlignment: WrapAlignment.center,
        ),
        
        // 3. End
        _buildWrapWithRunTitle(
          'End',
          runAlignment: WrapAlignment.end,
        ),
        
        // 4. SpaceEvenly
        _buildWrapWithRunTitle(
          'SpaceEvenly',
          runAlignment: WrapAlignment.spaceEvenly,
        ),
      ],
    );
  }

  Widget _buildWrapWithRunTitle(
    String title, {
    required WrapAlignment runAlignment,
  }) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(title),
        Container(
          width: 200,
          height: 150,
          color: Colors.grey[200],
          child: Wrap(
            runAlignment: runAlignment,
            spacing: 8,
            runSpacing: 8,
            children: [
              _buildChip('A'),
              _buildChip('B'),
              _buildChip('C'),
              _buildChip('D'),
              _buildChip('E'),
              _buildChip('F'),
              _buildChip('G'),
              _buildChip('H'),
              _buildChip('I'),
              _buildChip('J'),
            ],
          ),
        ),
        const SizedBox(height: 10),
      ],
    );
  }

  Widget _buildChip(String label) {
    return Chip(label: Text(label));
  }
}
```

What's happening here?
- runAlignment controls row alignment
- start aligns rows to top
- center centers rows
- end aligns rows to bottom
- spaceEvenly distributes rows

---

# Wrap with Different Sizes

> **Wrap handles** children of varying sizes.

```dart
// Wrap with varying children
class WrapWithVaryingSizes extends StatelessWidget {
  const WrapWithVaryingSizes({super.key});

  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 8,
      runSpacing: 8,
      children: [
        // Different size chips
        Chip(
          label: const Text('Small'),
          backgroundColor: Colors.blue[100],
        ),
        Chip(
          label: const Text('Medium Chip'),
          backgroundColor: Colors.green[100],
        ),
        Chip(
          label: const Text('Very Long Label'),
          backgroundColor: Colors.red[100],
        ),
        Chip(
          label: const Text('XL'),
          backgroundColor: Colors.yellow[100],
        ),
        Chip(
          label: const Text('Medium Plus'),
          backgroundColor: Colors.purple[100],
        ),
        Chip(
          label: const Text('Short'),
          backgroundColor: Colors.orange[100],
        ),
      ],
    );
  }
}
```

What's happening here?
- Children of different sizes
- Wrap adapts to each child's size
- Larger children take more space
- Wrap handles varied content
- Responsive to content changes

---

# Real-World Examples

> **Common patterns** using Wrap.

```dart
// 1. Tag cloud
class TagCloud extends StatelessWidget {
  const TagCloud({super.key});

  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 8,
      runSpacing: 8,
      children: [
        _buildTag('Flutter', 24),
        _buildTag('Dart', 18),
        _buildTag('Mobile', 20),
        _buildTag('Web', 16),
        _buildTag('Development', 22),
        _buildTag('iOS', 14),
        _buildTag('Android', 20),
        _buildTag('UI', 12),
        _buildTag('UX', 14),
        _buildTag('Design', 18),
        _buildTag('Programming', 22),
        _buildTag('Framework', 16),
      ],
    );
  }

  Widget _buildTag(String label, double size) {
    return Chip(
      label: Text(
        label,
        style: TextStyle(fontSize: size),
      ),
      backgroundColor: Colors.blue[100],
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
    );
  }
}

// 2. Filter chips
class FilterChips extends StatefulWidget {
  const FilterChips({super.key});

  @override
  State<FilterChips> createState() => _FilterChipsState();
}

class _FilterChipsState extends State<FilterChips> {
  List<String> selectedFilters = [];

  final List<String> filters = [
    'All',
    'Flutter',
    'Dart',
    'Mobile',
    'Web',
    'Desktop',
    'iOS',
    'Android',
    'Backend',
    'Frontend',
    'Full Stack',
    'DevOps',
  ];

  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 8,
      runSpacing: 8,
      children: filters.map((filter) {
        final isSelected = selectedFilters.contains(filter);
        return FilterChip(
          label: Text(filter),
          selected: isSelected,
          onSelected: (selected) {
            setState(() {
              if (selected) {
                selectedFilters.add(filter);
              } else {
                selectedFilters.remove(filter);
              }
            });
          },
          selectedColor: Colors.blue[200],
          backgroundColor: Colors.grey[200],
        );
      }).toList(),
    );
  }
}

// 3. Image gallery with labels
class ImageGallery extends StatelessWidget {
  const ImageGallery({super.key});

  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 16,
      runSpacing: 16,
      children: [
        _buildGalleryItem('Image 1', 'assets/image1.jpg'),
        _buildGalleryItem('Image 2', 'assets/image2.jpg'),
        _buildGalleryItem('Image 3', 'assets/image3.jpg'),
        _buildGalleryItem('Image 4', 'assets/image4.jpg'),
        _buildGalleryItem('Image 5', 'assets/image5.jpg'),
      ],
    );
  }

  Widget _buildGalleryItem(String title, String imagePath) {
    return Container(
      width: 150,
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        children: [
          Container(
            height: 100,
            decoration: BoxDecoration(
              color: Colors.blue[100],
              borderRadius: const BorderRadius.only(
                topLeft: Radius.circular(8),
                topRight: Radius.circular(8),
              ),
            ),
            child: const Center(child: Icon(Icons.image, size: 40)),
          ),
          Padding(
            padding: const EdgeInsets.all(8),
            child: Text(title),
          ),
        ],
      ),
    );
  }
}

// 4. Category selector
class CategorySelector extends StatelessWidget {
  const CategorySelector({super.key});

  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 8,
      runSpacing: 8,
      children: [
        _buildCategory('All', true),
        _buildCategory('Flutter', false),
        _buildCategory('Dart', false),
        _buildCategory('Mobile', false),
        _buildCategory('Web', false),
        _buildCategory('Desktop', false),
        _buildCategory('iOS', false),
        _buildCategory('Android', false),
      ],
    );
  }

  Widget _buildCategory(String label, bool isSelected) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      decoration: BoxDecoration(
        color: isSelected ? Colors.blue : Colors.grey[200],
        borderRadius: BorderRadius.circular(20),
        border: Border.all(
          color: isSelected ? Colors.blue : Colors.grey[300]!,
        ),
      ),
      child: Text(
        label,
        style: TextStyle(
          color: isSelected ? Colors.white : Colors.black87,
          fontWeight: isSelected ? FontWeight.bold : FontWeight.normal,
        ),
      ),
    );
  }
}
```

What's happening here?
- Tag cloud with varying sizes
- Interactive filter chips
- Image gallery with Wrap
- Category selector chips
- Common real-world patterns

---

# Best Practices

## Use Wrap for Dynamic Content

```dart
// Good - Wrap for dynamic chips
@override
Widget build(BuildContext context) {
  return Wrap(
    spacing: 8,
    runSpacing: 8,
    children: tags.map((tag) => Chip(label: Text(tag))).toList(),
  );
}

// Bad - Using Row for dynamic content
@override
Widget build(BuildContext context) {
  return Row(
    children: tags.map((tag) => Chip(label: Text(tag))).toList(),
    // Will overflow if too many tags
  );
}
```

## Set Appropriate Spacing

```dart
// Good - Proper spacing
@override
Widget build(BuildContext context) {
  return Wrap(
    spacing: 8, // Horizontal gap
    runSpacing: 8, // Vertical gap
    children: children,
  );
}
```

## Use CrossAxisAlignment for Alignment

```dart
// Good - CrossAxisAlignment control
@override
Widget build(BuildContext context) {
  return Wrap(
    crossAxisAlignment: WrapCrossAlignment.center,
    children: children,
  );
}
```

---

# Common Mistakes

## No Spacing

Wrong:
```dart
// No spacing between children
Wrap(
  children: [
    Chip(label: Text('A')),
    Chip(label: Text('B')),
    Chip(label: Text('C')),
  ],
)
```

Correct:
```dart
// Add spacing
Wrap(
  spacing: 8,
  runSpacing: 8,
  children: [
    Chip(label: Text('A')),
    Chip(label: Text('B')),
    Chip(label: Text('C')),
  ],
)
```

## Wrong Direction

Wrong:
```dart
// Using horizontal for vertical chips
Wrap(
  direction: Axis.horizontal,
  children: [
    Chip(label: Text('Long text that will wrap')),
    // Might not wrap as expected
  ],
)
```

Correct:
```dart
// Use appropriate direction for content
Wrap(
  direction: Axis.vertical,
  spacing: 8,
  children: [
    Chip(label: Text('Long text that will wrap')),
  ],
)
```

---

# Summary

Wrap arranges children in rows or columns and automatically wraps to new lines when space runs out. Use Wrap for chips, tags, categories, and any layout with dynamic content that needs to adapt to available space.

---

# Next Steps

- [Flow](flow.md)
- [Align](align.md)
- [Center](center.md)

---

# Did You Know?

- Wrap is like a flexible Row/Column
- Wrap automatically creates new rows/columns
- spacing controls horizontal gaps
- runSpacing controls vertical gaps
- Wrap handles children of different sizes
- Wrap supports both directions
- Wrap is perfect for responsive chips
- Wrap adapts to content changes