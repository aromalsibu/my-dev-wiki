# ScrollController

Understand how to programmatically control scrolling in Flutter.

---

# What is it?

ScrollController is a controller that manages and controls scrolling behavior in scrollable widgets. It allows you to read the current scroll position, animate to specific positions, listen to scroll events, and control multiple scrollable widgets. ScrollController is essential for implementing features like scroll-to-top, infinite scrolling, and synchronized scrolling.

---

# Why does it exist?

ScrollController exists to:

- Control scroll position programmatically
- Read current scroll offset
- Animate scrolling to positions
- Listen to scroll events
- Synchronize multiple scrollable widgets
- Implement infinite scrolling
- Create custom scroll behaviors

---

# Basic ScrollController

> **ScrollController** controls scrolling programmatically.

```dart
// Basic ScrollController usage
class BasicScrollControllerExample extends StatefulWidget {
  const BasicScrollControllerExample({super.key});

  @override
  State<BasicScrollControllerExample> createState() => _BasicScrollControllerExampleState();
}

class _BasicScrollControllerExampleState extends State<BasicScrollControllerExample> {
  final ScrollController _controller = ScrollController();

  @override
  void initState() {
    super.initState();
    // Listen to scroll events
    _controller.addListener(() {
      print('Scroll position: ${_controller.position.pixels}');
    });
  }

  @override
  void dispose() {
    // Clean up controller
    _controller.dispose();
    super.dispose();
  }

  void _scrollToTop() {
    // Animate to top
    _controller.animateTo(
      0,
      duration: const Duration(milliseconds: 500),
      curve: Curves.easeInOut,
    );
  }

  void _scrollToBottom() {
    // Animate to bottom
    _controller.animateTo(
      _controller.position.maxScrollExtent,
      duration: const Duration(milliseconds: 500),
      curve: Curves.easeInOut,
    );
  }

  void _jumpToMiddle() {
    // Jump to middle instantly
    _controller.jumpTo(_controller.position.maxScrollExtent / 2);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ScrollController'),
        actions: [
          TextButton(
            onPressed: () {
              setState(() {});
            },
            child: Text(
              'Position: ${_controller.position.pixels.toInt()}',
              style: const TextStyle(color: Colors.white),
            ),
          ),
        ],
      ),
      body: Column(
        children: [
          // Control buttons
          Container(
            padding: const EdgeInsets.all(8),
            color: Colors.grey[200],
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _scrollToTop,
                  child: const Text('Top'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _jumpToMiddle,
                  child: const Text('Middle'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _scrollToBottom,
                  child: const Text('Bottom'),
                ),
              ],
            ),
          ),
          
          // Scrollable content
          Expanded(
            child: ListView.builder(
              controller: _controller,
              itemCount: 50,
              itemBuilder: (context, index) {
                return Container(
                  height: 60,
                  color: Colors.blue[100 * (index % 9 + 1)],
                  margin: const EdgeInsets.symmetric(
                    horizontal: 8,
                    vertical: 4,
                  ),
                  child: Center(
                    child: Text(
                      'Item $index',
                      style: const TextStyle(
                        fontWeight: FontWeight.bold,
                      ),
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
- Controller tracks scroll position
- animateTo: animated scroll to position
- jumpTo: instant scroll to position
- position.pixels: current scroll offset
- position.maxScrollExtent: maximum scroll

---

# ScrollController Properties

> **Key properties** of ScrollController.

```dart
// ScrollController properties
class ScrollControllerProperties extends StatefulWidget {
  const ScrollControllerProperties({super.key});

  @override
  State<ScrollControllerProperties> createState() => _ScrollControllerPropertiesState();
}

class _ScrollControllerPropertiesState extends State<ScrollControllerProperties> {
  final ScrollController _controller = ScrollController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Controller Properties'),
      ),
      body: Column(
        children: [
          // Display controller info
          Container(
            padding: const EdgeInsets.all(16),
            color: Colors.grey[200],
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Has Clients: ${_controller.hasClients}'),
                Text('Has Listeners: ${_controller.hasListeners}'),
                if (_controller.hasClients) ...[
                  Text('Position: ${_controller.position.pixels}'),
                  Text('Min: ${_controller.position.minScrollExtent}'),
                  Text('Max: ${_controller.position.maxScrollExtent}'),
                  Text('Viewport: ${_controller.position.viewportDimension}'),
                ],
              ],
            ),
          ),
          
          Expanded(
            child: ListView.builder(
              controller: _controller,
              itemCount: 30,
              itemBuilder: (context, index) {
                return Container(
                  height: 50,
                  color: Colors.blue[100 * (index % 9 + 1)],
                  margin: const EdgeInsets.all(4),
                  child: Center(child: Text('Item $index')),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

// ScrollController properties:
// 1. hasClients - Whether attached to scrollable
// 2. hasListeners - Whether has listeners
// 3. position - Current scroll position
// 4. offset - Current scroll offset (deprecated)
```

What's happening here?
- hasClients: controller attached?
- position.pixels: current offset
- position.minScrollExtent: minimum scroll
- position.maxScrollExtent: maximum scroll
- position.viewportDimension: visible area

---

# Multiple ScrollControllers

> **Using multiple** controllers for synchronized scrolling.

```dart
// Multiple controllers example
class MultipleControllersExample extends StatefulWidget {
  const MultipleControllersExample({super.key});

  @override
  State<MultipleControllersExample> createState() => _MultipleControllersExampleState();
}

class _MultipleControllersExampleState extends State<MultipleControllersExample> {
  final ScrollController _controller1 = ScrollController();
  final ScrollController _controller2 = ScrollController();

  @override
  void initState() {
    super.initState();
    
    // Synchronize scrolling
    _controller1.addListener(() {
      if (_controller1.position.pixels != _controller2.position.pixels) {
        _controller2.jumpTo(_controller1.position.pixels);
      }
    });
  }

  @override
  void dispose() {
    _controller1.dispose();
    _controller2.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Synchronized Scrolling'),
      ),
      body: Row(
        children: [
          // First list
          Expanded(
            child: ListView.builder(
              controller: _controller1,
              itemCount: 30,
              itemBuilder: (context, index) {
                return Container(
                  height: 50,
                  color: Colors.blue[100 * (index % 9 + 1)],
                  margin: const EdgeInsets.all(4),
                  child: Center(child: Text('List 1 - $index')),
                );
              },
            ),
          ),
          
          const VerticalDivider(),
          
          // Second list (synchronized)
          Expanded(
            child: ListView.builder(
              controller: _controller2,
              itemCount: 30,
              itemBuilder: (context, index) {
                return Container(
                  height: 50,
                  color: Colors.green[100 * (index % 9 + 1)],
                  margin: const EdgeInsets.all(4),
                  child: Center(child: Text('List 2 - $index')),
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
- Two lists with separate controllers
- Synchronized scrolling between lists
- Controller listeners track positions
- jumpTo synchronizes positions

---

# Real-World Examples

> **Common patterns** using ScrollController.

```dart
// 1. Infinite scrolling with pagination
class InfiniteScrollExample extends StatefulWidget {
  const InfiniteScrollExample({super.key});

  @override
  State<InfiniteScrollExample> createState() => _InfiniteScrollExampleState();
}

class _InfiniteScrollExampleState extends State<InfiniteScrollExample> {
  final ScrollController _controller = ScrollController();
  List<int> _items = [];
  bool _isLoading = false;
  int _page = 0;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _controller.addListener(() {
      // Load more when near bottom
      if (_controller.position.pixels >= _controller.position.maxScrollExtent - 200) {
        _loadMore();
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Future<void> _loadMore() async {
    if (_isLoading) return;
    setState(() => _isLoading = true);
    
    // Simulate network request
    await Future.delayed(const Duration(seconds: 1));
    final start = _items.length;
    _items.addAll(List.generate(20, (i) => start + i));
    _page++;
    
    setState(() => _isLoading = false);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Infinite Scroll'),
      ),
      body: ListView.builder(
        controller: _controller,
        itemCount: _items.length + (_isLoading ? 1 : 0),
        itemBuilder: (context, index) {
          if (index == _items.length) {
            return const Padding(
              padding: EdgeInsets.all(16),
              child: Center(child: CircularProgressIndicator()),
            );
          }
          return Container(
            height: 60,
            color: Colors.blue[100 * (_items[index] % 9 + 1)],
            margin: const EdgeInsets.all(4),
            child: Center(
              child: Text(
                'Item ${_items[index]}',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
          );
        },
      ),
    );
  }
}

// 2. Scroll to top button
class ScrollToTopExample extends StatefulWidget {
  const ScrollToTopExample({super.key});

  @override
  State<ScrollToTopExample> createState() => _ScrollToTopExampleState();
}

class _ScrollToTopExampleState extends State<ScrollToTopExample> {
  final ScrollController _controller = ScrollController();
  bool _showButton = false;

  @override
  void initState() {
    super.initState();
    _controller.addListener(() {
      setState(() {
        _showButton = _controller.position.pixels > 200;
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _scrollToTop() {
    _controller.animateTo(
      0,
      duration: const Duration(milliseconds: 500),
      curve: Curves.easeInOut,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Scroll to Top'),
      ),
      body: Stack(
        children: [
          ListView.builder(
            controller: _controller,
            itemCount: 100,
            itemBuilder: (context, index) {
              return Container(
                height: 60,
                color: Colors.blue[100 * (index % 9 + 1)],
                margin: const EdgeInsets.all(4),
                child: Center(
                  child: Text(
                    'Item $index',
                    style: const TextStyle(fontWeight: FontWeight.bold),
                  ),
                ),
              );
            },
          ),
          if (_showButton)
            Positioned(
              bottom: 20,
              right: 20,
              child: FloatingActionButton(
                onPressed: _scrollToTop,
                child: const Icon(Icons.arrow_upward),
              ),
            ),
        ],
      ),
    );
  }
}

// 3. Scroll position indicator
class ScrollIndicatorExample extends StatefulWidget {
  const ScrollIndicatorExample({super.key});

  @override
  State<ScrollIndicatorExample> createState() => _ScrollIndicatorExampleState();
}

class _ScrollIndicatorExampleState extends State<ScrollIndicatorExample> {
  final ScrollController _controller = ScrollController();
  double _scrollProgress = 0;

  @override
  void initState() {
    super.initState();
    _controller.addListener(() {
      if (_controller.position.maxScrollExtent > 0) {
        setState(() {
          _scrollProgress = _controller.position.pixels / _controller.position.maxScrollExtent;
        });
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Scroll Progress'),
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(4),
          child: Container(
            height: 4,
            color: Colors.grey[300],
            child: Row(
              children: [
                Container(
                  width: _scrollProgress * MediaQuery.of(context).size.width,
                  height: 4,
                  color: Colors.blue,
                ),
              ],
            ),
          ),
        ),
      ),
      body: ListView.builder(
        controller: _controller,
        itemCount: 50,
        itemBuilder: (context, index) {
          return Container(
            height: 60,
            color: Colors.blue[100 * (index % 9 + 1)],
            margin: const EdgeInsets.all(4),
            child: Center(
              child: Text(
                'Item $index',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

What's happening here?
- Infinite scrolling with pagination
- Scroll to top button with animation
- Scroll progress indicator in app bar
- Controller listeners for updates

---

# Best Practices

## Dispose Controller

```dart
// Good - Always dispose
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}

// Bad - Memory leak
@override
void dispose() {
  super.dispose();
}
```

## Use animateTo for Smooth Scrolling

```dart
// Good - Smooth scrolling
_controller.animateTo(
  0,
  duration: const Duration(milliseconds: 500),
  curve: Curves.easeInOut,
);

// Bad - Jumping instantly
_controller.jumpTo(0);
```

## Check HasClients

```dart
// Good - Safe access
if (_controller.hasClients) {
  _controller.animateTo(...);
}

// Bad - Risk of error
_controller.animateTo(...);
```

---

# Common Mistakes

## Not Disposing Controller

Wrong:
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final ScrollController _controller = ScrollController();
  // No dispose - memory leak!
}
```

Correct:
```dart
class _MyWidgetState extends State<MyWidget> {
  final ScrollController _controller = ScrollController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

## Accessing Position Before Client Attached

Wrong:
```dart
// Error if no client attached
void _scrollToTop() {
  _controller.animateTo(0, duration: ...);
}
```

Correct:
```dart
// Check hasClients
void _scrollToTop() {
  if (_controller.hasClients) {
    _controller.animateTo(0, duration: ...);
  }
}
```

---

# Summary

ScrollController provides programmatic control over scrolling. Use it to read scroll position, animate to positions, listen to scroll events, and synchronize multiple scrollable widgets. Always dispose controllers to prevent memory leaks.

---

# Next Steps

- [ScrollPhysics](scrollphysics.md)
- [Nested Scrolling](nestedscrolling.md)
- [Infinite Scrolling](infinitescrolling.md)

---

# Did You Know?

- ScrollController must be disposed
- animateTo provides smooth scrolling
- jumpTo provides instant scrolling
- position.pixels gives current offset
- position.maxScrollExtent gives max scroll
- Controllers can be synchronized
- Controllers enable infinite scrolling
- Controllers can be shared between widgets