# Widget Splitting

Understand how to split large widgets into smaller, focused widgets to improve performance and maintainability.

---

# What is it?

Widget Splitting is the practice of breaking down large, monolithic widgets into smaller, focused widgets that each handle a specific part of the UI. This improves performance by isolating rebuilds to only the parts that need to change, and improves maintainability by making code more organized and reusable.

---

# Why does it exist?

Widget Splitting exists to:

- Reduce unnecessary rebuilds
- Improve performance by isolating changes
- Make code more maintainable
- Enable reusability of components
- Simplify debugging and testing
- Improve code organization
- Reduce complexity

---

# Why Split Widgets?

> **Understanding** the need for widget splitting.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Widget splitting demonstration
class WidgetSplittingDemo extends StatefulWidget {
  const WidgetSplittingDemo({super.key});

  @override
  State<WidgetSplittingDemo> createState() => _WidgetSplittingDemoState();
}

class _WidgetSplittingDemoState extends State<WidgetSplittingDemo> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print('🟢 Parent widget rebuilt');
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Widget Splitting'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Bad: Everything in one widget
            // Everything rebuilds when counter changes
            const Text(
              '❌ Bad: Everything Rebuilds',
              style: TextStyle(
                fontWeight: FontWeight.bold,
                color: Colors.red,
              ),
            ),
            _buildBadWidget(),
            
            const SizedBox(height: 24),
            
            // 2. Good: Split into smaller widgets
            // Only the counter part rebuilds
            const Text(
              '✅ Good: Only Counter Rebuilds',
              style: TextStyle(
                fontWeight: FontWeight.bold,
                color: Colors.green,
              ),
            ),
            _buildGoodWidget(),
          ],
        ),
      ),
    );
  }

  // Bad: Everything in one widget
  Widget _buildBadWidget() {
    print('🔴 Building bad widget');
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.red[50],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        children: [
          // Static header
          const Text(
            'Header',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          // Dynamic counter
          Text(
            'Counter: $_counter',
            style: const TextStyle(fontSize: 24),
          ),
          // Static footer
          const Text(
            'Footer',
            style: TextStyle(color: Colors.grey),
          ),
          // Button that triggers rebuild
          ElevatedButton(
            onPressed: () {
              setState(() {
                _counter++;
              });
            },
            child: const Text('Increment'),
          ),
        ],
      ),
    );
  }

  // Good: Split into smaller widgets
  Widget _buildGoodWidget() {
    print('🟡 Building good widget');
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.green[50],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        children: [
          // Static header (doesn't rebuild)
          const _StaticHeader(),
          // Dynamic counter (rebuilds)
          _DynamicCounter(counter: _counter),
          // Static footer (doesn't rebuild)
          const _StaticFooter(),
          // Button that triggers rebuild
          _IncrementButton(onPressed: () {
            setState(() {
              _counter++;
            });
          }),
        ],
      ),
    );
  }
}

/// Static header - doesn't rebuild
class _StaticHeader extends StatelessWidget {
  const _StaticHeader();

  @override
  Widget build(BuildContext context) {
    print('🟢 _StaticHeader built (cached)');
    return const Text(
      'Header',
      style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
    );
  }
}

/// Dynamic counter - rebuilds
class _DynamicCounter extends StatelessWidget {
  const _DynamicCounter({required this.counter});

  final int counter;

  @override
  Widget build(BuildContext context) {
    print('🔴 _DynamicCounter rebuilt');
    return Text(
      'Counter: $counter',
      style: const TextStyle(fontSize: 24),
    );
  }
}

/// Static footer - doesn't rebuild
class _StaticFooter extends StatelessWidget {
  const _StaticFooter();

  @override
  Widget build(BuildContext context) {
    print('🟢 _StaticFooter built (cached)');
    return const Text(
      'Footer',
      style: TextStyle(color: Colors.grey),
    );
  }
}

/// Increment button
class _IncrementButton extends StatelessWidget {
  const _IncrementButton({required this.onPressed});

  final VoidCallback onPressed;

  @override
  Widget build(BuildContext context) {
    print('🟡 _IncrementButton built');
    return ElevatedButton(
      onPressed: onPressed,
      child: const Text('Increment'),
    );
  }
}
```

What's happening here?
- Bad: Everything rebuilds
- Good: Only counter part rebuilds
- Static parts are cached
- Better performance

---

# Splitting Patterns

> **Common patterns** for splitting widgets.

```dart
/// Splitting patterns example
class SplittingPatternsExample extends StatefulWidget {
  const SplittingPatternsExample({super.key});

  @override
  State<SplittingPatternsExample> createState() => _SplittingPatternsExampleState();
}

class _SplittingPatternsExampleState extends State<SplittingPatternsExample> {
  String _name = 'John';
  int _age = 25;
  bool _isActive = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Splitting Patterns'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Pattern: Extract static widgets
            const _SectionTitle('Static Widgets'),
            
            // 2. Pattern: Extract dynamic widgets
            _DynamicNameWidget(name: _name),
            _DynamicAgeWidget(age: _age),
            _DynamicStatusWidget(isActive: _isActive),
            
            const SizedBox(height: 16),
            
            // 3. Pattern: Extract widget with callbacks
            _ActionButtons(
              onNameChange: () {
                setState(() {
                  _name = _name == 'John' ? 'Jane' : 'John';
                });
              },
              onAgeChange: () {
                setState(() {
                  _age++;
                });
              },
              onStatusChange: () {
                setState(() {
                  _isActive = !_isActive;
                });
              },
            ),
            
            const SizedBox(height: 16),
            
            // 4. Pattern: Extract complex widgets
            const _ComplexCard(),
          ],
        ),
      ),
    );
  }
}

/// Section title (static)
class _SectionTitle extends StatelessWidget {
  const _SectionTitle(this.title);

  final String title;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(8),
      child: Text(
        title,
        style: const TextStyle(
          fontSize: 18,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}

/// Dynamic name widget
class _DynamicNameWidget extends StatelessWidget {
  const _DynamicNameWidget({required this.name});

  final String name;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.blue[100],
      child: Text('Name: $name'),
    );
  }
}

/// Dynamic age widget
class _DynamicAgeWidget extends StatelessWidget {
  const _DynamicAgeWidget({required this.age});

  final int age;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(8),
      color: Colors.green[100],
      child: Text('Age: $age'),
    );
  }
}

/// Dynamic status widget
class _DynamicStatusWidget extends StatelessWidget {
  const _DynamicStatusWidget({required this.isActive});

  final bool isActive;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(8),
      color: isActive ? Colors.green[100] : Colors.red[100],
      child: Text(
        'Status: ${isActive ? "Active" : "Inactive"}',
        style: TextStyle(
          fontWeight: FontWeight.bold,
          color: isActive ? Colors.green : Colors.red,
        ),
      ),
    );
  }
}

/// Widget with callbacks
class _ActionButtons extends StatelessWidget {
  const _ActionButtons({
    required this.onNameChange,
    required this.onAgeChange,
    required this.onStatusChange,
  });

  final VoidCallback onNameChange;
  final VoidCallback onAgeChange;
  final VoidCallback onStatusChange;

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Expanded(
          child: ElevatedButton(
            onPressed: onNameChange,
            child: const Text('Change Name'),
          ),
        ),
        const SizedBox(width: 8),
        Expanded(
          child: ElevatedButton(
            onPressed: onAgeChange,
            child: const Text('Increase Age'),
          ),
        ),
        const SizedBox(width: 8),
        Expanded(
          child: ElevatedButton(
            onPressed: onStatusChange,
            child: const Text('Toggle Status'),
          ),
        ),
      ],
    );
  }
}

/// Complex widget (extracted for clarity)
class _ComplexCard extends StatelessWidget {
  const _ComplexCard();

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            const Text(
              'Complex Widget',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            const Text(
              'This widget contains multiple sub-widgets',
              style: TextStyle(color: Colors.grey),
            ),
            const SizedBox(height: 8),
            const Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Icon(Icons.star, color: Colors.yellow),
                Icon(Icons.favorite, color: Colors.red),
                Icon(Icons.thumb_up, color: Colors.blue),
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
- Static widgets extracted
- Dynamic widgets isolated
- Widgets with callbacks
- Complex widgets extracted

---

# Real-World Examples

> **Common patterns** with widget splitting.

```dart
/// 1. Form with splitting
class SplitFormExample extends StatefulWidget {
  const SplitFormExample({super.key});

  @override
  State<SplitFormExample> createState() => _SplitFormExampleState();
}

class _SplitFormExampleState extends State<SplitFormExample> {
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  String _submittedData = '';

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Split Form'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Name field (isolated)
            _FormField(
              controller: _nameController,
              label: 'Name',
              hint: 'Enter your name',
            ),
            const SizedBox(height: 16),
            
            // Email field (isolated)
            _FormField(
              controller: _emailController,
              label: 'Email',
              hint: 'Enter your email',
              keyboardType: TextInputType.emailAddress,
            ),
            const SizedBox(height: 16),
            
            // Submit button
            _SubmitButton(
              onPressed: () {
                setState(() {
                  _submittedData =
                      'Name: ${_nameController.text}\nEmail: ${_emailController.text}';
                });
              },
            ),
            
            const SizedBox(height: 16),
            
            // Result display (isolated)
            _ResultDisplay(data: _submittedData),
          ],
        ),
      ),
    );
  }
}

/// Form field widget
class _FormField extends StatelessWidget {
  const _FormField({
    required this.controller,
    required this.label,
    required this.hint,
    this.keyboardType = TextInputType.text,
  });

  final TextEditingController controller;
  final String label;
  final String hint;
  final TextInputType keyboardType;

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: controller,
      decoration: InputDecoration(
        labelText: label,
        hintText: hint,
        border: const OutlineInputBorder(),
      ),
      keyboardType: keyboardType,
    );
  }
}

/// Submit button widget
class _SubmitButton extends StatelessWidget {
  const _SubmitButton({required this.onPressed});

  final VoidCallback onPressed;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: double.infinity,
      child: ElevatedButton(
        onPressed: onPressed,
        child: const Text('Submit'),
      ),
    );
  }
}

/// Result display widget
class _ResultDisplay extends StatelessWidget {
  const _ResultDisplay({required this.data});

  final String data;

  @override
  Widget build(BuildContext context) {
    if (data.isEmpty) {
      return Container(
        padding: const EdgeInsets.all(16),
        color: Colors.grey[100],
        child: const Text(
          'No data submitted yet',
          style: TextStyle(color: Colors.grey),
        ),
      );
    }

    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.green[50],
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text(
            'Submitted Data:',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          Text(data),
        ],
      ),
    );
  }
}

/// 2. Reusable card pattern
class ReusableCardPattern extends StatelessWidget {
  const ReusableCardPattern({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Reusable Cards'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Card with different configurations
            _InfoCard(
              title: 'User Info',
              subtitle: 'John Doe',
              icon: Icons.person,
              color: Colors.blue,
            ),
            const SizedBox(height: 8),
            _InfoCard(
              title: 'Email',
              subtitle: 'john@example.com',
              icon: Icons.email,
              color: Colors.green,
            ),
            const SizedBox(height: 8),
            _InfoCard(
              title: 'Phone',
              subtitle: '+1 234 567 890',
              icon: Icons.phone,
              color: Colors.orange,
            ),
          ],
        ),
      ),
    );
  }
}

/// Reusable info card
class _InfoCard extends StatelessWidget {
  const _InfoCard({
    required this.title,
    required this.subtitle,
    required this.icon,
    required this.color,
  });

  final String title;
  final String subtitle;
  final IconData icon;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: Container(
          padding: const EdgeInsets.all(8),
          decoration: BoxDecoration(
            color: color.withOpacity(0.2),
            shape: BoxShape.circle,
          ),
          child: Icon(icon, color: color),
        ),
        title: Text(title, style: const TextStyle(fontWeight: FontWeight.bold)),
        subtitle: Text(subtitle),
        trailing: const Icon(Icons.arrow_forward_ios, size: 16),
      ),
    );
  }
}
```

What's happening here?
- Form with isolated fields
- Reusable card pattern
- Separate UI from logic
- Clean and maintainable code

---

# Best Practices

## Split by Responsibility

```dart
// Good - Each widget has one responsibility
class UserAvatar extends StatelessWidget { ... }
class UserName extends StatelessWidget { ... }
class UserEmail extends StatelessWidget { ... }

// Bad - One widget does everything
class UserCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Avatar, name, email, actions all in one widget
  }
}
```

## Use const for Static Parts

```dart
// Good - const for static widgets
const _StaticHeader()
const _StaticFooter()

// Bad - No const for static parts
_StaticHeader()
_StaticFooter()
```

## Extract Reusable Components

```dart
// Good - Reusable card
class _InfoCard extends StatelessWidget { ... }

// Bad - Duplicated code
Card(child: ...) // Repeated multiple times
```

---

# Common Mistakes

## Over-Splitting

Wrong:
```dart
// Too many widgets for simple UI
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        _Text1(),
        _Text2(),
        _Text3(),
      ],
    );
  }
}
```

Correct:
```dart
// Simple UI doesn't need splitting
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: const [
        Text('Text 1'),
        Text('Text 2'),
        Text('Text 3'),
      ],
    );
  }
}
```

## Not Splitting When Needed

Wrong:
```dart
// Everything rebuilds
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        StaticHeader(),
        DynamicContent(), // Rebuilds
        StaticFooter(), // Also rebuilds unnecessarily
      ],
    );
  }
}
```

Correct:
```dart
// Only dynamic part rebuilds
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const StaticHeader(),
        DynamicContent(), // Only this rebuilds
        const StaticFooter(),
      ],
    );
  }
}
```

---

# Summary

Widget Splitting improves performance and maintainability by breaking large widgets into smaller, focused pieces. Split widgets by responsibility, use const for static parts, and extract reusable components. Widget splitting is essential for building scalable, performant Flutter applications.

---

# Next Steps

- [Keys](keys.md)
- [Const Widgets](const-widgets.md)
- [RepaintBoundary](repaintboundary.md)

---

# Did You Know?

- Widget splitting reduces rebuilds
- Static widgets can be cached
- Dynamic widgets isolate changes
- Splitting improves maintainability
- Reusable components save time
- Const widgets improve performance
- Widget splitting is a best practice
- Split by responsibility, not by size