# Widget Catalog

Understand the extensive catalog of built-in widgets available in Flutter.

---

# What is it?

The Widget Catalog is the complete collection of built-in widgets that Flutter provides out of the box. It includes everything from basic building blocks like Text and Container to complex widgets like ListView and CustomPaint. Understanding what widgets are available and when to use them is essential for efficient Flutter development.

---

# Why does it exist?

The Widget Catalog exists to:

- Provide ready-to-use UI components
- Speed up development with pre-built widgets
- Ensure consistent and optimized implementations
- Cover common UI patterns and use cases
- Support Material Design and Cupertino styles
- Enable rapid prototyping and development
- Reduce the need for custom implementations

---

# Widget Categories

> **Widgets are organized** into functional categories.

```dart
// Widget catalog overview
class WidgetCatalogOverview extends StatelessWidget {
  const WidgetCatalogOverview({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Basic Widgets - Fundamental building blocks
        const Text('Basic Widgets'),
        const Container(),
        const SizedBox(),
        const Icon(Icons.star),
        const Image(image: AssetImage('')),
        
        // 2. Layout Widgets - Arranging other widgets
        const Row(children: []),
        const Column(children: []),
        const Stack(children: []),
        const Expanded(child: SizedBox()),
        const Padding(padding: EdgeInsets.zero, child: SizedBox()),
        
        // 3. Input Widgets - User interaction
        const TextField(),
        const Checkbox(value: false, onChanged: null),
        const Radio(value: null, groupValue: null, onChanged: null),
        const Slider(value: 0, onChanged: null),
        const Switch(value: false, onChanged: null),
        
        // 4. Scrolling Widgets - Scrollable content
        const ListView(children: []),
        const GridView(children: []),
        const SingleChildScrollView(child: SizedBox()),
        const CustomScrollView(slivers: []),
        
        // 5. Navigation Widgets - Screen navigation
        // Navigator, Router, Route
        
        // 6. Material Widgets - Material Design components
        const Scaffold(),
        const AppBar(),
        const Card(),
        const Drawer(),
        const FloatingActionButton(),
        
        // 7. Cupertino Widgets - iOS-style components
        // CupertinoApp, CupertinoNavigationBar, CupertinoButton
        
        // 8. Animation Widgets - Animated UI elements
        const AnimatedContainer(duration: Duration.zero),
        const AnimatedOpacity(duration: Duration.zero),
        const Hero(tag: '', child: SizedBox()),
        
        // 9. Async Widgets - Handling async data
        const FutureBuilder(future: null, builder: null),
        const StreamBuilder(stream: null, builder: null),
        const ValueListenableBuilder(valueListenable: null, builder: null),
        
        // 10. Accessibility Widgets - Making apps accessible
        const Semantics(),
      ],
    );
  }
}
```

What's happening here?
- Widgets are categorized by purpose
- Each category serves different needs
- Many widgets are platform-specific
- Categories cover all UI needs
- Comprehensive library of components

---

# Basic Widgets

> **Fundamental widgets** that form the building blocks of UI.

```dart
// 1. Text - Display text
class TextExample extends StatelessWidget {
  const TextExample({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text(
      'Hello World',
      style: TextStyle(
        fontSize: 24,
        fontWeight: FontWeight.bold,
        color: Colors.blue,
      ),
    );
  }
}

// 2. Container - Box with styling
class ContainerExample extends StatelessWidget {
  const ContainerExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 200,
      height: 100,
      padding: const EdgeInsets.all(16),
      margin: const EdgeInsets.all(8),
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(8),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.2),
            blurRadius: 10,
          ),
        ],
      ),
      child: const Center(
        child: Text(
          'Container',
          style: TextStyle(color: Colors.white),
        ),
      ),
    );
  }
}

// 3. SizedBox - Fixed size box
class SizedBoxExample extends StatelessWidget {
  const SizedBoxExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Fixed size
        const SizedBox(
          width: 100,
          height: 50,
          child: ColoredBox(color: Colors.red),
        ),
        // Spacing
        const SizedBox(height: 20),
        // Width only
        const SizedBox(
          width: 200,
          child: Text('Text with fixed width'),
        ),
      ],
    );
  }
}

// 4. Icon - Display icons
class IconExample extends StatelessWidget {
  const IconExample({super.key});

  @override
  Widget build(BuildContext context) {
    return const Row(
      children: [
        Icon(Icons.star, color: Colors.yellow, size: 30),
        Icon(Icons.favorite, color: Colors.red, size: 30),
        Icon(Icons.home, color: Colors.blue, size: 30),
      ],
    );
  }
}

// 5. Image - Display images
class ImageExample extends StatelessWidget {
  const ImageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Asset image
        Image.asset(
          'assets/logo.png',
          width: 100,
          height: 100,
          fit: BoxFit.cover,
        ),
        // Network image
        Image.network(
          'https://example.com/image.jpg',
          width: 100,
          height: 100,
          fit: BoxFit.cover,
        ),
        // Memory image
        // Image.memory(bytes),
      ],
    );
  }
}

// 6. ElevatedButton - Material button
class ButtonExample extends StatelessWidget {
  const ButtonExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ElevatedButton(
          onPressed: () {},
          child: const Text('Elevated Button'),
        ),
        TextButton(
          onPressed: () {},
          child: const Text('Text Button'),
        ),
        OutlinedButton(
          onPressed: () {},
          child: const Text('Outlined Button'),
        ),
        IconButton(
          icon: const Icon(Icons.star),
          onPressed: () {},
        ),
      ],
    );
  }
}
```

What's happening here?
- Text displays strings
- Container provides styling
- SizedBox controls size
- Icon displays icons
- Image loads images
- Buttons handle actions

---

# Layout Widgets

> **Widgets for arranging** other widgets on screen.

```dart
// 1. Row - Horizontal arrangement
class RowExample extends StatelessWidget {
  const RowExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      crossAxisAlignment: CrossAxisAlignment.center,
      children: [
        Container(width: 50, height: 50, color: Colors.red),
        Container(width: 50, height: 50, color: Colors.green),
        Container(width: 50, height: 50, color: Colors.blue),
      ],
    );
  }
}

// 2. Column - Vertical arrangement
class ColumnExample extends StatelessWidget {
  const ColumnExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      crossAxisAlignment: CrossAxisAlignment.center,
      children: [
        Container(width: 50, height: 50, color: Colors.red),
        Container(width: 50, height: 50, color: Colors.green),
        Container(width: 50, height: 50, color: Colors.blue),
      ],
    );
  }
}

// 3. Stack - Overlay widgets
class StackExample extends StatelessWidget {
  const StackExample({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: 200,
      height: 200,
      child: Stack(
        children: [
          // Background
          Container(color: Colors.blue),
          // Positioned text
          Positioned(
            top: 20,
            left: 20,
            child: const Text(
              'Top Left',
              style: TextStyle(color: Colors.white),
            ),
          ),
          // Positioned icon
          Positioned(
            bottom: 20,
            right: 20,
            child: const Icon(
              Icons.star,
              color: Colors.yellow,
              size: 40,
            ),
          ),
        ],
      ),
    );
  }
}

// 4. Expanded - Fill available space
class ExpandedExample extends StatelessWidget {
  const ExpandedExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Container(width: 50, color: Colors.red),
        Expanded(
          flex: 2,
          child: Container(color: Colors.green),
        ),
        Expanded(
          flex: 1,
          child: Container(color: Colors.blue),
        ),
      ],
    );
  }
}

// 5. Padding - Add internal spacing
class PaddingExample extends StatelessWidget {
  const PaddingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Container(
        color: Colors.blue,
        child: const Text('Padded content'),
      ),
    );
  }
}

// 6. Center - Center child
class CenterExample extends StatelessWidget {
  const CenterExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 200,
      height: 200,
      color: Colors.grey[200],
      child: const Center(
        child: Text('Centered'),
      ),
    );
  }
}
```

What's happening here?
- Row arranges horizontally
- Column arranges vertically
- Stack overlays widgets
- Expanded fills space
- Padding adds internal space
- Center positions centrally

---

# Scrolling Widgets

> **Widgets for scrollable** content.

```dart
// 1. ListView - Vertical scrollable list
class ListViewExample extends StatelessWidget {
  const ListViewExample({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView(
      padding: const EdgeInsets.all(8),
      children: [
        for (int i = 0; i < 20; i++)
          Container(
            height: 50,
            color: Colors.blue[100 * (i % 9 + 1)],
            margin: const EdgeInsets.only(bottom: 8),
            child: Center(child: Text('Item $i')),
          ),
      ],
    );
  }
}

// 2. ListView.builder - Efficient scrolling
class ListBuilderExample extends StatelessWidget {
  const ListBuilderExample({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 1000,
      itemBuilder: (context, index) {
        return ListTile(
          leading: CircleAvatar(
            child: Text('${index + 1}'),
          ),
          title: Text('Item $index'),
          subtitle: Text('Subtitle for item $index'),
          trailing: const Icon(Icons.arrow_forward),
          onTap: () {},
        );
      },
    );
  }
}

// 3. ListView.separated - With separators
class ListSeparatedExample extends StatelessWidget {
  const ListSeparatedExample({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.separated(
      itemCount: 20,
      separatorBuilder: (context, index) => const Divider(),
      itemBuilder: (context, index) {
        return ListTile(
          title: Text('Item $index'),
          onTap: () {},
        );
      },
    );
  }
}

// 4. GridView - Grid layout
class GridViewExample extends StatelessWidget {
  const GridViewExample({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.count(
      crossAxisCount: 3,
      padding: const EdgeInsets.all(8),
      crossAxisSpacing: 8,
      mainAxisSpacing: 8,
      children: [
        for (int i = 0; i < 20; i++)
          Container(
            color: Colors.blue[100 * (i % 9 + 1)],
            child: Center(child: Text('$i')),
          ),
      ],
    );
  }
}

// 5. SingleChildScrollView - Single scrollable child
class SingleChildScrollViewExample extends StatelessWidget {
  const SingleChildScrollViewExample({super.key});

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        children: [
          for (int i = 0; i < 50; i++)
            Container(
              height: 60,
              color: Colors.blue[100 * (i % 9 + 1)],
              margin: const EdgeInsets.only(bottom: 8),
              child: Center(child: Text('Widget $i')),
            ),
        ],
      ),
    );
  }
}
```

What's happening here?
- ListView scrolls lists
- ListView.builder is efficient
- GridView arranges in grid
- SingleChildScrollView scrolls one child
- Various scroll configurations available

---

# Input Widgets

> **Widgets for user input** and interaction.

```dart
// 1. TextField - Text input
class TextFieldExample extends StatefulWidget {
  const TextFieldExample({super.key});

  @override
  State<TextFieldExample> createState() => _TextFieldExampleState();
}

class _TextFieldExampleState extends State<TextFieldExample> {
  final TextEditingController _controller = TextEditingController();
  String _text = '';

  @override
  void initState() {
    super.initState();
    _controller.addListener(() {
      setState(() {
        _text = _controller.text;
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          controller: _controller,
          decoration: InputDecoration(
            labelText: 'Enter text',
            border: OutlineInputBorder(
              borderRadius: BorderRadius.circular(8),
            ),
            prefixIcon: const Icon(Icons.person),
            suffixIcon: const Icon(Icons.clear),
          ),
          keyboardType: TextInputType.text,
          textInputAction: TextInputAction.done,
          onChanged: (value) {
            setState(() {
              _text = value;
            });
          },
        ),
        const SizedBox(height: 16),
        Text('You typed: $_text'),
      ],
    );
  }
}

// 2. Checkbox - Boolean input
class CheckboxExample extends StatefulWidget {
  const CheckboxExample({super.key});

  @override
  State<CheckboxExample> createState() => _CheckboxExampleState();
}

class _CheckboxExampleState extends State<CheckboxExample> {
  bool _isChecked = false;

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Checkbox(
          value: _isChecked,
          onChanged: (value) {
            setState(() {
              _isChecked = value ?? false;
            });
          },
        ),
        Text(
          'Accept terms',
          style: TextStyle(
            decoration: _isChecked 
                ? TextDecoration.lineThrough 
                : null,
          ),
        ),
      ],
    );
  }
}

// 3. Radio - Single choice from group
class RadioExample extends StatefulWidget {
  const RadioExample({super.key});

  @override
  State<RadioExample> createState() => _RadioExampleState();
}

class _RadioExampleState extends State<RadioExample> {
  String _selectedOption = 'Option 1';

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        RadioListTile(
          title: const Text('Option 1'),
          value: 'Option 1',
          groupValue: _selectedOption,
          onChanged: (value) {
            setState(() {
              _selectedOption = value as String;
            });
          },
        ),
        RadioListTile(
          title: const Text('Option 2'),
          value: 'Option 2',
          groupValue: _selectedOption,
          onChanged: (value) {
            setState(() {
              _selectedOption = value as String;
            });
          },
        ),
        RadioListTile(
          title: const Text('Option 3'),
          value: 'Option 3',
          groupValue: _selectedOption,
          onChanged: (value) {
            setState(() {
              _selectedOption = value as String;
            });
          },
        ),
        Text('Selected: $_selectedOption'),
      ],
    );
  }
}

// 4. Slider - Range selection
class SliderExample extends StatefulWidget {
  const SliderExample({super.key});

  @override
  State<SliderExample> createState() => _SliderExampleState();
}

class _SliderExampleState extends State<SliderExample> {
  double _value = 0.5;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Slider(
          value: _value,
          min: 0,
          max: 1,
          divisions: 10,
          label: '${(_value * 100).round()}%',
          onChanged: (value) {
            setState(() {
              _value = value;
            });
          },
        ),
        Text('Value: ${(_value * 100).round()}%'),
      ],
    );
  }
}

// 5. Switch - Toggle input
class SwitchExample extends StatefulWidget {
  const SwitchExample({super.key});

  @override
  State<SwitchExample> createState() => _SwitchExampleState();
}

class _SwitchExampleState extends State<SwitchExample> {
  bool _isOn = false;

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Switch(
          value: _isOn,
          onChanged: (value) {
            setState(() {
              _isOn = value;
            });
          },
        ),
        Text(_isOn ? 'ON' : 'OFF'),
      ],
    );
  }
}
```

What's happening here?
- TextField: text input
- Checkbox: boolean selection
- Radio: single choice selection
- Slider: range selection
- Switch: toggle input

---

# Material Widgets

> **Material Design** widgets for Android-style UIs.

```dart
// 1. Scaffold - Basic app structure
class ScaffoldExample extends StatelessWidget {
  const ScaffoldExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('App Title'),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: () {},
          ),
          PopupMenuButton(
            itemBuilder: (context) => [
              const PopupMenuItem(value: 'settings', child: Text('Settings')),
              const PopupMenuItem(value: 'logout', child: Text('Logout')),
            ],
          ),
        ],
      ),
      drawer: Drawer(
        child: ListView(
          children: [
            const DrawerHeader(
              decoration: BoxDecoration(color: Colors.blue),
              child: Text('Menu'),
            ),
            ListTile(
              leading: const Icon(Icons.home),
              title: const Text('Home'),
              onTap: () {},
            ),
            ListTile(
              leading: const Icon(Icons.settings),
              title: const Text('Settings'),
              onTap: () {},
            ),
          ],
        ),
      ),
      body: const Center(
        child: Text('Body content'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        child: const Icon(Icons.add),
      ),
      bottomNavigationBar: BottomNavigationBar(
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}

// 2. Card - Material card
class CardExample extends StatelessWidget {
  const CardExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 4,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                const CircleAvatar(
                  radius: 20,
                  backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        'John Doe',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 16,
                        ),
                      ),
                      Text(
                        '2 hours ago',
                        style: TextStyle(
                          color: Colors.grey[600],
                          fontSize: 12,
                        ),
                      ),
                    ],
                  ),
                ),
              ],
            ),
            const SizedBox(height: 12),
            const Text(
              'Lorem ipsum dolor sit amet, consectetur adipiscing elit. '
              'Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.',
            ),
          ],
        ),
      ),
    );
  }
}

// 3. Dialog - Popup dialog
class DialogExample extends StatelessWidget {
  const DialogExample({super.key});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        showDialog(
          context: context,
          builder: (context) => AlertDialog(
            title: const Text('Dialog Title'),
            content: const Text('Dialog content goes here'),
            actions: [
              TextButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('Cancel'),
              ),
              ElevatedButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('Confirm'),
              ),
            ],
          ),
        );
      },
      child: const Text('Show Dialog'),
    );
  }
}
```

What's happening here?
- Scaffold: app structure
- AppBar: top navigation
- Drawer: side menu
- Card: material cards
- Dialog: popup windows

---

# Best Practices

## Use Appropriate Widgets

```dart
// Good - Using appropriate widgets
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: 100,
    itemBuilder: (context, index) {
      return ListTile(
        title: Text('Item $index'),
      );
    },
  );
}

// Bad - Inefficient for large lists
@override
Widget build(BuildContext context) {
  return SingleChildScrollView(
    child: Column(
      children: List.generate(100, (index) {
        return Text('Item $index');
      }),
    ),
  );
}
```

## Combine Widgets Effectively

```dart
// Good - Combining widgets
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(title: const Text('Home')),
    body: Padding(
      padding: const EdgeInsets.all(16),
      child: Center(
        child: Column(
          children: [
            Text('Welcome'),
            ElevatedButton(
              onPressed: () {},
              child: const Text('Click'),
            ),
          ],
        ),
      ),
    ),
  );
}
```

## Leverage Built-in Widgets

```dart
// Good - Use built-in widgets
@override
Widget build(BuildContext context) {
  return Card(
    child: Padding(
      padding: const EdgeInsets.all(16),
      child: Text('Content'),
    ),
  );
}

// Bad - Reinventing wheels
@override
Widget build(BuildContext context) {
  return Container(
    decoration: BoxDecoration(
      color: Colors.white,
      border: Border.all(),
      borderRadius: BorderRadius.circular(4),
    ),
    child: Padding(
      padding: const EdgeInsets.all(16),
      child: Text('Content'),
    ),
  );
}
```

---

# Summary

The Widget Catalog provides a comprehensive set of built-in widgets for building Flutter apps. Widgets are organized into categories like basic, layout, scrolling, input, and Material widgets. Understanding the catalog helps you build UIs faster and more efficiently.

---

# Next Steps

- [Constraints](../layout/constraints.md)
- [Layout Process](../framework-architecture/layout-process.md)
- [State Management](../state-management/local-state.md)

---

# Did You Know?

- Flutter has over 100 built-in widgets
- Widgets are organized by functionality
- Material and Cupertino widgets are platform-specific
- Custom widgets can be composed from built-in ones
- The catalog covers most UI needs
- New widgets are added in each release
- Widgets are optimized for performance
- The catalog is fully documented