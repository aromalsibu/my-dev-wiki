# Widget Composition

Understand how to build complex UIs by composing widgets together.

---

# What is it?

Widget composition is the process of building complex user interfaces by combining smaller, reusable widgets together. Instead of creating one large widget, Flutter encourages breaking down the UI into smaller, focused widgets that can be composed together to create rich and interactive interfaces. This is a fundamental principle of Flutter's design philosophy.

---

# Why does it exist?

Widget composition exists to:

- Create reusable UI components
- Simplify complex UIs by breaking them down
- Improve code organization and maintainability
- Enable separation of concerns
- Reduce code duplication
- Make testing easier
- Allow for flexible and customizable UIs

---

# Composition Principles

> **Composition over inheritance** is a key principle in Flutter.

```dart
// 1. Composition example - Building complex UI from simple widgets
class UserCard extends StatelessWidget {
  const UserCard({
    super.key,
    required this.name,
    required this.email,
    required this.avatarUrl,
  });

  final String name;
  final String email;
  final String avatarUrl;

  @override
  Widget build(BuildContext context) {
    // Compose simple widgets to create a complex card
    return Card(
      margin: const EdgeInsets.all(8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            // Compose avatar
            CircleAvatar(
              radius: 30,
              backgroundImage: NetworkImage(avatarUrl),
            ),
            const SizedBox(width: 16),
            
            // Compose user info
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    name,
                    style: const TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    email,
                    style: TextStyle(
                      color: Colors.grey[600],
                      fontSize: 14,
                    ),
                  ),
                ],
              ),
            ),
            
            // Compose actions
            IconButton(
              icon: const Icon(Icons.more_vert),
              onPressed: () {},
            ),
          ],
        ),
      ),
    );
  }
}

// 2. Reusable sub-widgets
class UserAvatar extends StatelessWidget {
  const UserAvatar({
    super.key,
    required this.imageUrl,
    this.radius = 30,
  });

  final String imageUrl;
  final double radius;

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      radius: radius,
      backgroundImage: NetworkImage(imageUrl),
    );
  }
}

class UserInfo extends StatelessWidget {
  const UserInfo({
    super.key,
    required this.name,
    required this.email,
  });

  final String name;
  final String email;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          name,
          style: const TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
        ),
        Text(
          email,
          style: TextStyle(
            color: Colors.grey[600],
            fontSize: 14,
          ),
        ),
      ],
    );
  }
}

// Composed UserCard using reusable widgets
class ComposedUserCard extends StatelessWidget {
  const ComposedUserCard({
    super.key,
    required this.name,
    required this.email,
    required this.avatarUrl,
  });

  final String name;
  final String email;
  final String avatarUrl;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            UserAvatar(imageUrl: avatarUrl),
            const SizedBox(width: 16),
            Expanded(
              child: UserInfo(name: name, email: email),
            ),
            IconButton(
              icon: const Icon(Icons.more_vert),
              onPressed: () {},
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Complex UI broken into smaller widgets
- Widgets are composed together
- Each widget has a specific responsibility
- Reusable widgets can be used in multiple places
- Composition makes code more maintainable

---

# Composition Patterns

> **Common patterns** for composing widgets.

```dart
// 1. Container pattern - Wrapping widgets with styling
class ContainerPattern extends StatelessWidget {
  const ContainerPattern({
    super.key,
    required this.child,
    this.padding,
    this.margin,
    this.color,
    this.borderRadius,
  });

  final Widget child;
  final EdgeInsets? padding;
  final EdgeInsets? margin;
  final Color? color;
  final BorderRadius? borderRadius;

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: margin ?? const EdgeInsets.all(8),
      padding: padding ?? const EdgeInsets.all(16),
      color: color,
      decoration: BoxDecoration(
        borderRadius: borderRadius ?? BorderRadius.circular(8),
      ),
      child: child,
    );
  }
}

// 2. Layout pattern - Arranging widgets
class LayoutPattern extends StatelessWidget {
  const LayoutPattern({
    super.key,
    required this.header,
    required this.body,
    this.footer,
  });

  final Widget header;
  final Widget body;
  final Widget? footer;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        // Header section
        Container(
          padding: const EdgeInsets.all(16),
          color: Colors.blue[100],
          child: header,
        ),
        
        // Body section
        Expanded(
          child: Container(
            padding: const EdgeInsets.all(16),
            child: body,
          ),
        ),
        
        // Footer section (optional)
        if (footer != null)
          Container(
            padding: const EdgeInsets.all(16),
            color: Colors.grey[200],
            child: footer,
          ),
      ],
    );
  }
}

// 3. Builder pattern - Widget factory
class WidgetBuilderPattern extends StatelessWidget {
  const WidgetBuilderPattern({
    super.key,
    required this.builder,
    this.condition = true,
  });

  final Widget Function(BuildContext) builder;
  final bool condition;

  @override
  Widget build(BuildContext context) {
    return Builder(
      builder: (context) {
        if (condition) {
          return builder(context);
        }
        return const SizedBox.shrink();
      },
    );
  }
}

// 4. Configuration pattern - Configurable widgets
class ConfigurableWidget extends StatelessWidget {
  const ConfigurableWidget({
    super.key,
    this.size = Size(100, 100),
    this.color = Colors.blue,
    this.borderRadius = 8.0,
    this.child,
  });

  final Size size;
  final Color color;
  final double borderRadius;
  final Widget? child;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: size.width,
      height: size.height,
      decoration: BoxDecoration(
        color: color,
        borderRadius: BorderRadius.circular(borderRadius),
      ),
      child: child,
    );
  }
}

// Usage example
class CompositionPatternsExample extends StatelessWidget {
  const CompositionPatternsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Container pattern
        ContainerPattern(
          color: Colors.blue,
          borderRadius: BorderRadius.circular(16),
          child: const Text('Container Pattern'),
        ),
        
        // Layout pattern
        LayoutPattern(
          header: const Text('Header', style: TextStyle(fontSize: 20)),
          body: const Text('Body content here'),
          footer: const Text('Footer'),
        ),
        
        // Configurable widget
        ConfigurableWidget(
          size: const Size(150, 100),
          color: Colors.green,
          borderRadius: 16,
          child: const Center(child: Text('Configured')),
        ),
      ],
    );
  }
}
```

What's happening here?
- Container pattern: wraps with styling
- Layout pattern: arranges sections
- Builder pattern: creates widgets dynamically
- Configuration pattern: reusable configurable widgets
- Each pattern serves a specific purpose

---

# Nested Composition

> **Nesting widgets** creates complex UI structures.

```dart
// Nested composition example
class NestedComposition extends StatelessWidget {
  const NestedComposition({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Nested Composition'),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: () {},
          ),
          IconButton(
            icon: const Icon(Icons.more_vert),
            onPressed: () {},
          ),
        ],
      ),
      body: Column(
        children: [
          // Header section
          Container(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                const Expanded(
                  child: Text(
                    'Today\'s Feed',
                    style: TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.refresh),
                  onPressed: () {},
                ),
              ],
            ),
          ),
          
          // Feed items
          Expanded(
            child: ListView.builder(
              itemCount: 10,
              itemBuilder: (context, index) {
                return Card(
                  margin: const EdgeInsets.symmetric(
                    horizontal: 16,
                    vertical: 8,
                  ),
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Row(
                          children: [
                            CircleAvatar(
                              backgroundColor: Colors.blue[100],
                              child: Text('${index + 1}'),
                            ),
                            const SizedBox(width: 12),
                            Expanded(
                              child: Text(
                                'User ${index + 1}',
                                style: const TextStyle(
                                  fontWeight: FontWeight.bold,
                                ),
                              ),
                            ),
                            Text(
                              '2h ago',
                              style: TextStyle(
                                color: Colors.grey[600],
                                fontSize: 12,
                              ),
                            ),
                          ],
                        ),
                        const SizedBox(height: 12),
                        const Text(
                          'Lorem ipsum dolor sit amet, consectetur adipiscing elit. '
                          'Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.',
                        ),
                        const SizedBox(height: 12),
                        Row(
                          children: [
                            IconButton(
                              icon: const Icon(Icons.favorite_border),
                              onPressed: () {},
                            ),
                            const Text('25'),
                            const SizedBox(width: 16),
                            IconButton(
                              icon: const Icon(Icons.comment_outlined),
                              onPressed: () {},
                            ),
                            const Text('8'),
                            const Spacer(),
                            IconButton(
                              icon: const Icon(Icons.share_outlined),
                              onPressed: () {},
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Multiple levels of nesting
- Each widget has specific responsibility
- Complex UI from simple widgets
- Readable and maintainable structure

---

# Composition vs Inheritance

> **Prefer composition** over inheritance in Flutter.

```dart
// Inheritance approach (not recommended)
// ❌ Don't extend widgets
class BaseWidget extends StatelessWidget {
  const BaseWidget({super.key, required this.child});

  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      child: child,
    );
  }
}

class InheritedWidget extends BaseWidget {
  const InheritedWidget({super.key, super.child, required this.color})
      : super();

  final Color color;

  @override
  Widget build(BuildContext context) {
    return Container(
      color: color,
      padding: const EdgeInsets.all(16),
      child: child,
    );
  }
}

// Composition approach (recommended)
// ✅ Compose widgets instead
class ComposedWidget extends StatelessWidget {
  const ComposedWidget({
    super.key,
    required this.child,
    this.color,
  });

  final Widget child;
  final Color? color;

  @override
  Widget build(BuildContext context) {
    Widget result = child;
    
    // Add padding
    result = Padding(
      padding: const EdgeInsets.all(16),
      child: result,
    );
    
    // Add color if provided
    if (color != null) {
      result = Container(
        color: color,
        child: result,
      );
    }
    
    return result;
  }
}

// Usage comparison
class UsageExample extends StatelessWidget {
  const UsageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // ❌ Inheritance
        InheritedWidget(
          color: Colors.blue,
          child: const Text('Inherited'),
        ),
        
        // ✅ Composition
        ComposedWidget(
          color: Colors.blue,
          child: const Text('Composed'),
        ),
        
        // ✅ Even simpler composition
        Container(
          color: Colors.blue,
          padding: const EdgeInsets.all(16),
          child: const Text('Simple'),
        ),
      ],
    );
  }
}

// Why composition is preferred:
// 1. More flexible
// 2. Easier to maintain
// 3. Better reusability
// 4. Clearer purpose
// 5. No deep hierarchies
```

What's happening here?
- Avoid deep inheritance
- Compose widgets together
- Each widget is focused
- Easier to understand
- More maintainable

---

# Reusable Components

> **Building reusable components** through composition.

```dart
// Reusable component library
class AppComponents {
  // 1. App Button
  static Widget appButton({
    required String label,
    required VoidCallback onPressed,
    bool isPrimary = true,
    double width = double.infinity,
  }) {
    return SizedBox(
      width: width,
      child: ElevatedButton(
        onPressed: onPressed,
        style: ElevatedButton.styleFrom(
          backgroundColor: isPrimary ? Colors.blue : Colors.grey,
          padding: const EdgeInsets.symmetric(vertical: 16),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(8),
          ),
        ),
        child: Text(
          label,
          style: TextStyle(
            color: isPrimary ? Colors.white : Colors.black,
            fontSize: 16,
          ),
        ),
      ),
    );
  }

  // 2. App Card
  static Widget appCard({
    required Widget child,
    double? elevation,
    EdgeInsets? padding,
    VoidCallback? onTap,
  }) {
    Widget card = Card(
      elevation: elevation ?? 4,
      margin: const EdgeInsets.all(8),
      child: Padding(
        padding: padding ?? const EdgeInsets.all(16),
        child: child,
      ),
    );

    if (onTap != null) {
      card = InkWell(
        onTap: onTap,
        child: card,
      );
    }

    return card;
  }

  // 3. App List Tile
  static Widget appListTile({
    required String title,
    String? subtitle,
    Widget? leading,
    Widget? trailing,
    VoidCallback? onTap,
    Color? backgroundColor,
  }) {
    return Container(
      color: backgroundColor,
      child: ListTile(
        leading: leading,
        title: Text(title),
        subtitle: subtitle != null ? Text(subtitle) : null,
        trailing: trailing,
        onTap: onTap,
        contentPadding: const EdgeInsets.symmetric(
          horizontal: 16,
          vertical: 8,
        ),
      ),
    );
  }

  // 4. App Loading
  static Widget appLoading({
    String? message,
    double size = 40,
  }) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          SizedBox(
            width: size,
            height: size,
            child: const CircularProgressIndicator(),
          ),
          if (message != null) ...[
            const SizedBox(height: 16),
            Text(
              message,
              style: const TextStyle(color: Colors.grey),
            ),
          ],
        ],
      ),
    );
  }

  // 5. App Error
  static Widget appError({
    required String error,
    VoidCallback? onRetry,
  }) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(
              Icons.error_outline,
              size: 64,
              color: Colors.red,
            ),
            const SizedBox(height: 16),
            Text(
              'Something went wrong',
              style: const TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              error,
              textAlign: TextAlign.center,
              style: const TextStyle(color: Colors.grey),
            ),
            if (onRetry != null) ...[
              const SizedBox(height: 16),
              appButton(
                label: 'Retry',
                onPressed: onRetry,
              ),
            ],
          ],
        ),
      ),
    );
  }
}

// Using the component library
class ComponentLibraryUsage extends StatelessWidget {
  const ComponentLibraryUsage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Components')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            AppComponents.appButton(
              label: 'Primary Button',
              onPressed: () {},
            ),
            const SizedBox(height: 8),
            AppComponents.appButton(
              label: 'Secondary Button',
              onPressed: () {},
              isPrimary: false,
            ),
            AppComponents.appCard(
              child: const Text('Card content here'),
              onTap: () {},
            ),
            AppComponents.appListTile(
              title: 'List Item',
              subtitle: 'Subtitle here',
              leading: const Icon(Icons.star),
              trailing: const Icon(Icons.arrow_forward),
              onTap: () {},
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Reusable components library
- Consistent styling
- Configurable options
- Easy to use anywhere
- Maintainable design system

---

# Best Practices

## Keep Widgets Focused

```dart
// Good - Focused widget
class UserAvatar extends StatelessWidget {
  const UserAvatar({super.key, required this.imageUrl});

  final String imageUrl;

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      backgroundImage: NetworkImage(imageUrl),
      radius: 30,
    );
  }
}

// Bad - Doing too much
class UserWidget extends StatelessWidget {
  const UserWidget({super.key, required this.user});

  final User user;

  @override
  Widget build(BuildContext context) {
    // Everything in one widget
    return Row(
      children: [
        CircleAvatar(backgroundImage: NetworkImage(user.avatarUrl)),
        Text(user.name),
        Text(user.email),
        IconButton(
          icon: const Icon(Icons.more_vert),
          onPressed: () {},
        ),
      ],
    );
  }
}
```

## Extract Repeating Patterns

```dart
// Good - Extract repeated pattern
class SectionHeader extends StatelessWidget {
  const SectionHeader({
    super.key,
    required this.title,
    this.action,
  });

  final String title;
  final Widget? action;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Row(
        children: [
          Text(
            title,
            style: const TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          const Spacer(),
          if (action != null) action!,
        ],
      ),
    );
  }
}

// Bad - Repeating code
class BadExample extends StatelessWidget {
  const BadExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Padding(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
          child: Row(
            children: [
              const Text(
                'Section 1',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              const Spacer(),
              ElevatedButton(onPressed: () {}, child: const Text('Action')),
            ],
          ),
        ),
        Padding(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
          child: Row(
            children: [
              const Text(
                'Section 2',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              const Spacer(),
              ElevatedButton(onPressed: () {}, child: const Text('Action')),
            ],
          ),
        ),
      ],
    );
  }
}
```

## Compose, Don't Duplicate

```dart
// Good - Compose widgets
class AppScreen extends StatelessWidget {
  const AppScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Home'),
        leading: const BackButton(),
        actions: [
          IconButton(icon: const Icon(Icons.search), onPressed: () {}),
          IconButton(icon: const Icon(Icons.more_vert), onPressed: () {}),
        ],
      ),
      body: const BodyWidget(),
    );
  }
}

// Bad - Duplicate code
class AnotherScreen extends StatelessWidget {
  const AnotherScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Another'),
        leading: const BackButton(),
        actions: [
          IconButton(icon: const Icon(Icons.search), onPressed: () {}),
          IconButton(icon: const Icon(Icons.more_vert), onPressed: () {}),
        ],
      ),
      body: const AnotherBody(),
    );
  }
}
```

---

# Summary

Widget composition is the foundation of building UIs in Flutter. By composing smaller, focused widgets together, you create complex, maintainable, and reusable UIs. Follow composition principles, use common patterns, and extract reusable components for consistent and efficient development.

---

# Next Steps

- [Widget Catalog](widget-catalog.md)
- [Constraints](../layout/constraints.md)
- [Box Model](../layout/box-model.md)

---

# Did You Know?

- Composition over inheritance is a core Flutter principle
- Widgets can be composed infinitely
- Small widgets are easier to test
- Reusable components save development time
- Composition enables design systems
- Flutter encourages shallow widget trees
- Custom widgets are just composed widgets
- Composition enables hot reload benefits