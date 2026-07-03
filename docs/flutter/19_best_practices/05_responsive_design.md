# Responsive Design

Understand how to create layouts that adapt to different screen sizes and orientations in Flutter.

---

# What is it?

Responsive Design is the practice of building user interfaces that adapt to different screen sizes, orientations, and devices. In Flutter, this means creating layouts that work well on phones, tablets, desktops, and everything in between. Responsive design ensures your app looks great and functions properly on any device.

---

# Why does it exist?

Responsive Design exists to:

- Support multiple screen sizes
- Handle different orientations
- Provide consistent experience
- Adapt to device capabilities
- Improve usability
- Reach wider audience
- Follow platform conventions

---

# Basic Responsive Design

> **Creating** responsive layouts.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// 1. Basic responsive layout
class BasicResponsiveExample extends StatelessWidget {
  const BasicResponsiveExample({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Get screen size
    final size = MediaQuery.of(context).size;
    final width = size.width;
    final height = size.height;

    // 2. Determine device type
    final isMobile = width < 600;
    final isTablet = width >= 600 && width < 1200;
    final isDesktop = width >= 1200;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Responsive Design'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 3. Responsive text
            Text(
              'Screen Size: ${width.toStringAsFixed(0)} x ${height.toStringAsFixed(0)}',
              style: const TextStyle(fontSize: 16),
            ),
            const SizedBox(height: 8),
            Text(
              'Device Type: ${_getDeviceType(isMobile, isTablet, isDesktop)}',
              style: const TextStyle(fontSize: 16),
            ),
            const SizedBox(height: 16),
            
            // 4. Responsive layout
            isDesktop
                ? _buildDesktopLayout()
                : isTablet
                    ? _buildTabletLayout()
                    : _buildMobileLayout(),
          ],
        ),
      ),
    );
  }

  String _getDeviceType(bool isMobile, bool isTablet, bool isDesktop) {
    if (isMobile) return 'Mobile';
    if (isTablet) return 'Tablet';
    return 'Desktop';
  }

  Widget _buildMobileLayout() {
    return Column(
      children: [
        _buildCard('Mobile Card 1', Colors.blue),
        const SizedBox(height: 8),
        _buildCard('Mobile Card 2', Colors.green),
        const SizedBox(height: 8),
        _buildCard('Mobile Card 3', Colors.orange),
      ],
    );
  }

  Widget _buildTabletLayout() {
    return GridView.count(
      crossAxisCount: 2,
      crossAxisSpacing: 8,
      mainAxisSpacing: 8,
      shrinkWrap: true,
      children: [
        _buildCard('Tablet Card 1', Colors.blue),
        _buildCard('Tablet Card 2', Colors.green),
        _buildCard('Tablet Card 3', Colors.orange),
        _buildCard('Tablet Card 4', Colors.purple),
      ],
    );
  }

  Widget _buildDesktopLayout() {
    return GridView.count(
      crossAxisCount: 3,
      crossAxisSpacing: 8,
      mainAxisSpacing: 8,
      shrinkWrap: true,
      children: [
        _buildCard('Desktop Card 1', Colors.blue),
        _buildCard('Desktop Card 2', Colors.green),
        _buildCard('Desktop Card 3', Colors.orange),
        _buildCard('Desktop Card 4', Colors.purple),
        _buildCard('Desktop Card 5', Colors.red),
        _buildCard('Desktop Card 6', Colors.teal),
      ],
    );
  }

  Widget _buildCard(String title, Color color) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: color,
        borderRadius: BorderRadius.circular(8),
      ),
      child: Center(
        child: Text(
          title,
          style: const TextStyle(
            color: Colors.white,
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- MediaQuery gets screen size
- Device type detection
- Different layouts for different sizes
- Adaptive grid counts

---

# LayoutBuilder

> **Using LayoutBuilder** for responsive layouts.

```dart
/// 2. LayoutBuilder for responsive design
class LayoutBuilderExample extends StatelessWidget {
  const LayoutBuilderExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('LayoutBuilder'),
      ),
      body: LayoutBuilder(
        builder: (context, constraints) {
          // 1. Get available space
          final maxWidth = constraints.maxWidth;
          final maxHeight = constraints.maxHeight;

          // 2. Determine layout based on width
          if (maxWidth > 800) {
            return _buildWideLayout(maxWidth, maxHeight);
          } else if (maxWidth > 400) {
            return _buildMediumLayout(maxWidth, maxHeight);
          } else {
            return _buildNarrowLayout(maxWidth, maxHeight);
          }
        },
      ),
    );
  }

  Widget _buildWideLayout(double width, double height) {
    return Row(
      children: [
        // Sidebar
        Container(
          width: width * 0.3,
          color: Colors.blue,
          child: const Center(
            child: Text(
              'Sidebar',
              style: TextStyle(color: Colors.white, fontSize: 24),
            ),
          ),
        ),
        // Content
        Expanded(
          child: Container(
            color: Colors.grey[200],
            child: const Center(
              child: Text(
                'Content',
                style: TextStyle(fontSize: 24),
              ),
            ),
          ),
        ),
      ],
    );
  }

  Widget _buildMediumLayout(double width, double height) {
    return Column(
      children: [
        // Header
        Container(
          height: height * 0.2,
          color: Colors.blue,
          child: const Center(
            child: Text(
              'Header',
              style: TextStyle(color: Colors.white, fontSize: 24),
            ),
          ),
        ),
        // Content
        Expanded(
          child: Container(
            color: Colors.grey[200],
            child: const Center(
              child: Text(
                'Content',
                style: TextStyle(fontSize: 24),
              ),
            ),
          ),
        ),
      ],
    );
  }

  Widget _buildNarrowLayout(double width, double height) {
    return Column(
      children: [
        // Top bar
        Container(
          height: height * 0.15,
          color: Colors.blue,
          child: const Center(
            child: Text(
              'Top Bar',
              style: TextStyle(color: Colors.white, fontSize: 24),
            ),
          ),
        ),
        // Main content
        Expanded(
          child: Container(
            color: Colors.grey[200],
            child: const Center(
              child: Text(
                'Content',
                style: TextStyle(fontSize: 24),
              ),
            ),
          ),
        ),
        // Bottom bar
        Container(
          height: height * 0.15,
          color: Colors.blue,
          child: const Center(
            child: Text(
              'Bottom Bar',
              style: TextStyle(color: Colors.white, fontSize: 24),
            ),
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- LayoutBuilder provides constraints
- Different layouts based on available width
- Sidebar for wide screens
- Stacked layout for narrow screens

---

# Responsive Widgets

> **Using** responsive widgets.

```dart
/// 3. Responsive widgets
class ResponsiveWidgetsExample extends StatelessWidget {
  const ResponsiveWidgetsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Responsive Widgets'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Expanded - Fills available space
            const Text(
              'Expanded Widget',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Row(
              children: [
                Container(
                  width: 50,
                  height: 50,
                  color: Colors.red,
                ),
                Expanded(
                  child: Container(
                    height: 50,
                    color: Colors.blue,
                    child: const Center(child: Text('Expanded')),
                  ),
                ),
                Container(
                  width: 50,
                  height: 50,
                  color: Colors.green,
                ),
              ],
            ),
            const SizedBox(height: 16),

            // 2. Flexible - Flexible sizing
            const Text(
              'Flexible Widget',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Row(
              children: [
                Flexible(
                  flex: 2,
                  child: Container(
                    height: 50,
                    color: Colors.red,
                    child: const Center(child: Text('Flex 2')),
                  ),
                ),
                const SizedBox(width: 8),
                Flexible(
                  flex: 1,
                  child: Container(
                    height: 50,
                    color: Colors.blue,
                    child: const Center(child: Text('Flex 1')),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 16),

            // 3. FractionallySizedBox - Percentage sizing
            const Text(
              'FractionallySizedBox',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            FractionallySizedBox(
              widthFactor: 0.8,
              heightFactor: 0.5,
              child: Container(
                color: Colors.green,
                child: const Center(
                  child: Text(
                    '80% width, 50% height',
                    style: TextStyle(color: Colors.white),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 16),

            // 4. AspectRatio - Maintain aspect ratio
            const Text(
              'AspectRatio',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            AspectRatio(
              aspectRatio: 16 / 9,
              child: Container(
                color: Colors.orange,
                child: const Center(
                  child: Text(
                    '16:9 Aspect Ratio',
                    style: TextStyle(color: Colors.white),
                  ),
                ),
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
- Expanded fills available space
- Flexible sizes proportionally
- FractionallySizedBox uses percentage
- AspectRatio maintains ratio

---

# Real-World Examples

> **Common patterns** for responsive design.

```dart
/// 4. Responsive dashboard
class ResponsiveDashboard extends StatelessWidget {
  const ResponsiveDashboard({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Responsive Dashboard'),
      ),
      body: LayoutBuilder(
        builder: (context, constraints) {
          final isMobile = constraints.maxWidth < 600;
          final crossAxisCount = isMobile ? 1 : 2;

          return Padding(
            padding: const EdgeInsets.all(16),
            child: GridView.builder(
              gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: crossAxisCount,
                crossAxisSpacing: 8,
                mainAxisSpacing: 8,
                childAspectRatio: isMobile ? 0.8 : 1.0,
              ),
              itemCount: 6,
              itemBuilder: (context, index) {
                return _buildDashboardCard(index, isMobile);
              },
            ),
          );
        },
      ),
    );
  }

  Widget _buildDashboardCard(int index, bool isMobile) {
    final colors = [Colors.blue, Colors.green, Colors.orange, Colors.red, Colors.purple, Colors.teal];
    final icons = [
      Icons.people,
      Icons.shopping_cart,
      Icons.attach_money,
      Icons.star,
      Icons.settings,
      Icons.info,
    ];
    final labels = [
      'Users',
      'Orders',
      'Revenue',
      'Ratings',
      'Settings',
      'About',
    ];
    final values = [
      '1,234',
      '567',
      '$12,345',
      '4.8',
      'Configure',
      'Version 1.0',
    ];

    return Card(
      elevation: 4,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              icons[index],
              size: isMobile ? 32 : 48,
              color: colors[index],
            ),
            const SizedBox(height: 8),
            Text(
              labels[index],
              style: const TextStyle(
                fontWeight: FontWeight.bold,
                fontSize: 16,
              ),
            ),
            const SizedBox(height: 4),
            Text(
              values[index],
              style: const TextStyle(
                fontSize: isMobile ? 20 : 24,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// 5. Responsive navigation
class ResponsiveNavigation extends StatelessWidget {
  const ResponsiveNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final isMobile = constraints.maxWidth < 600;

        if (isMobile) {
          // Mobile: Bottom navigation
          return Scaffold(
            appBar: AppBar(
              title: const Text('Responsive Nav'),
            ),
            body: const Center(
              child: Text('Mobile Content'),
            ),
            bottomNavigationBar: BottomNavigationBar(
              items: const [
                BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
                BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
                BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
              ],
            ),
          );
        } else {
          // Desktop: Side navigation
          return Scaffold(
            body: Row(
              children: [
                // Sidebar
                Container(
                  width: 250,
                  color: Colors.blue,
                  child: Column(
                    children: [
                      const SizedBox(height: 32),
                      const Text(
                        'App Logo',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 32),
                      _buildNavItem(Icons.home, 'Home'),
                      _buildNavItem(Icons.search, 'Search'),
                      _buildNavItem(Icons.person, 'Profile'),
                      _buildNavItem(Icons.settings, 'Settings'),
                    ],
                  ),
                ),
                // Content
                Expanded(
                  child: Container(
                    color: Colors.grey[200],
                    child: const Center(
                      child: Text(
                        'Desktop Content',
                        style: TextStyle(fontSize: 24),
                      ),
                    ),
                  ),
                ),
              ],
            ),
          );
        }
      },
    );
  }

  Widget _buildNavItem(IconData icon, String label) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Row(
        children: [
          Icon(icon, color: Colors.white),
          const SizedBox(width: 16),
          Text(
            label,
            style: const TextStyle(
              color: Colors.white,
              fontSize: 16,
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Responsive dashboard grid
- Different card sizes
- Mobile bottom navigation
- Desktop sidebar navigation

---

# Best Practices

## Use LayoutBuilder for Responsive Layouts

```dart
// Good - LayoutBuilder for responsive design
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth > 600) {
      return WideLayout();
    } else {
      return NarrowLayout();
    }
  },
)

// Bad - Using MediaQuery for everything
// MediaQuery is fine but LayoutBuilder is more precise
```

## Use Responsive Widgets

```dart
// Good - Responsive widgets
Expanded() // Fills space
Flexible() // Flexible sizing
FractionallySizedBox() // Percentage sizing
AspectRatio() // Maintains ratio

// Bad - Fixed sizes
Container(width: 200, height: 200)
```

## Test on Multiple Screen Sizes

```dart
// Good - Test different sizes
// Use device preview in DevTools
// Test on emulators of different sizes
// Use responsive testing tools

// Bad - Only testing on one device
// Only testing on development device
```

---

# Common Mistakes

## Using Fixed Sizes

Wrong:
```dart
// Fixed size doesn't adapt
Container(
  width: 300,
  height: 200,
  child: ...,
)
```

Correct:
```dart
// Responsive size
LayoutBuilder(
  builder: (context, constraints) {
    return Container(
      width: constraints.maxWidth * 0.8,
      height: constraints.maxHeight * 0.5,
      child: ...,
    );
  },
)
```

## Not Testing on Different Devices

Wrong:
```dart
// Only testing on one device
// Not testing on tablets
// Not testing on different orientations
```

Correct:
```dart
// Testing on multiple devices
// Testing different orientations
// Using device preview
```

---

# Summary

Responsive Design ensures your app works well on all screen sizes. Use MediaQuery for screen size, LayoutBuilder for precise constraints, and responsive widgets like Expanded, Flexible, and FractionallySizedBox. Test on multiple devices and screen sizes for best results.

---

# Next Steps

- [Adaptive Design](adaptive-design.md)
- [Architecture Patterns](architecture-patterns.md)
- [Project Structure](project-structure.md)

---

# Did You Know?

- MediaQuery provides screen size
- LayoutBuilder gives precise constraints
- Responsive design improves UX
- Different layouts for different sizes
- Grid count can adapt to width
- Navigation can change layout
- Responsive design is essential for mobile apps
- Flutter provides many responsive tools