# Nested Scrolling

Understand how to handle nested scrollable widgets in Flutter.

---

# What is it?

Nested scrolling occurs when you have scrollable widgets inside other scrollable widgets (e.g., a ListView inside a CustomScrollView, or a TabBarView with lists). Flutter provides mechanisms to handle nested scrolling so that parent and child scroll views work together smoothly, without conflicts or unexpected behavior.

---

# Why does it exist?

Nested scrolling exists to:

- Enable complex scrollable layouts
- Combine multiple scrollable widgets
- Handle scroll conflicts automatically
- Create smooth scrolling experiences
- Support nested scroll views
- Implement complex UI patterns
- Provide flexible scrollable hierarchies

---

# Nested ScrollView

> **NestedScrollView** handles nested scrolling.

```dart
// Basic NestedScrollView
class BasicNestedScrollView extends StatelessWidget {
  const BasicNestedScrollView({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        body: NestedScrollView(
          headerSliverBuilder: (context, innerBoxIsScrolled) {
            return [
              SliverAppBar(
                title: const Text('Nested Scroll'),
                expandedHeight: 200,
                flexibleSpace: const FlexibleSpaceBar(
                  background: ColoredBox(
                    color: Colors.blue,
                    child: Center(
                      child: Text(
                        'Header',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                ),
                pinned: true,
                bottom: const TabBar(
                  tabs: [
                    Tab(text: 'Tab 1'),
                    Tab(text: 'Tab 2'),
                    Tab(text: 'Tab 3'),
                  ],
                ),
              ),
            ];
          },
          body: TabBarView(
            children: [
              // Each tab has its own scrollable content
              _buildTabContent('Tab 1', Colors.blue),
              _buildTabContent('Tab 2', Colors.green),
              _buildTabContent('Tab 3', Colors.orange),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildTabContent(String label, Color color) {
    return ListView.builder(
      itemCount: 30,
      itemBuilder: (context, index) {
        return Container(
          height: 60,
          color: color.withOpacity(0.3 + (index % 5) * 0.1),
          margin: const EdgeInsets.all(4),
          child: Center(
            child: Text(
              '$label - Item $index',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
        );
      },
    );
  }
}

// NestedScrollView properties:
// 1. headerSliverBuilder - Builds header slivers
// 2. body - Main scrollable content
// 3. physics - Scroll physics
// 4. controller - Scroll controller
// 5. floatHeaderSlivers - Whether headers float
```

What's happening here?
- NestedScrollView manages nested scrolling
- Header slivers scroll with content
- TabBarView contains scrollable lists
- Smooth scrolling between header and body

---

# NestedScrollView with Multiple Tabs

> **NestedScrollView** with tabbed content.

```dart
// NestedScrollView with tabs
class TabbedNestedScrollView extends StatefulWidget {
  const TabbedNestedScrollView({super.key});

  @override
  State<TabbedNestedScrollView> createState() => _TabbedNestedScrollViewState();
}

class _TabbedNestedScrollViewState extends State<TabbedNestedScrollView>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 3, vsync: this);
  }

  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: NestedScrollView(
        headerSliverBuilder: (context, innerBoxIsScrolled) {
          return [
            SliverAppBar(
              title: const Text('Tabbed Nested Scroll'),
              expandedHeight: 150,
              flexibleSpace: const FlexibleSpaceBar(
                background: ColoredBox(
                  color: Colors.blue,
                  child: Center(
                    child: Text(
                      'Profile Header',
                      style: TextStyle(color: Colors.white),
                    ),
                  ),
                ),
              ),
              pinned: true,
              bottom: TabBar(
                controller: _tabController,
                tabs: const [
                  Tab(icon: Icon(Icons.list), text: 'Posts'),
                  Tab(icon: Icon(Icons.photo), text: 'Photos'),
                  Tab(icon: Icon(Icons.person), text: 'About'),
                ],
              ),
            ),
          ];
        },
        body: TabBarView(
          controller: _tabController,
          children: [
            // Posts tab
            ListView.builder(
              itemCount: 30,
              itemBuilder: (context, index) {
                return ListTile(
                  leading: CircleAvatar(
                    child: Text('${index + 1}'),
                  ),
                  title: Text('Post ${index + 1}'),
                  subtitle: Text('This is post number ${index + 1}'),
                  trailing: const Icon(Icons.favorite_border),
                );
              },
            ),
            
            // Photos tab
            GridView.builder(
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 3,
                crossAxisSpacing: 2,
                mainAxisSpacing: 2,
              ),
              itemCount: 30,
              itemBuilder: (context, index) {
                return Container(
                  color: Colors.blue[100 * (index % 9 + 1)],
                  child: Center(
                    child: Text(
                      '${index + 1}',
                      style: const TextStyle(color: Colors.white),
                    ),
                  ),
                );
              },
            ),
            
            // About tab
            ListView(
              children: [
                ListTile(
                  leading: const Icon(Icons.person),
                  title: const Text('Name'),
                  subtitle: const Text('John Doe'),
                ),
                ListTile(
                  leading: const Icon(Icons.email),
                  title: const Text('Email'),
                  subtitle: const Text('john@example.com'),
                ),
                ListTile(
                  leading: const Icon(Icons.phone),
                  title: const Text('Phone'),
                  subtitle: const Text('+1 234 567 890'),
                ),
                ListTile(
                  leading: const Icon(Icons.location_on),
                  title: const Text('Location'),
                  subtitle: const Text('New York, USA'),
                ),
                const Padding(
                  padding: EdgeInsets.all(16),
                  child: Text(
                    'Bio: Flutter developer with 5 years of experience.',
                  ),
                ),
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
- TabController manages tabs
- Each tab has different scrollable content
- Header stays pinned while tabs change
- NestedScrollView coordinates scrolling

---

# Nested Scroll with CustomScrollView

> **CustomScrollView** with nested scrolling.

```dart
// Nested scrolling with CustomScrollView
class NestedCustomScrollView extends StatelessWidget {
  const NestedCustomScrollView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // App bar
          SliverAppBar(
            title: const Text('Nested Custom Scroll'),
            expandedHeight: 150,
            flexibleSpace: const FlexibleSpaceBar(
              background: ColoredBox(
                color: Colors.blue,
                child: Center(
                  child: Text(
                    'Header',
                    style: TextStyle(color: Colors.white),
                  ),
                ),
              ),
            ),
            pinned: true,
          ),
          
          // Section 1: Horizontal scroll
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'Horizontal Scroll',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          SliverToBoxAdapter(
            child: SizedBox(
              height: 120,
              child: ListView.builder(
                scrollDirection: Axis.horizontal,
                itemCount: 15,
                itemBuilder: (context, index) {
                  return Container(
                    width: 100,
                    margin: const EdgeInsets.symmetric(horizontal: 4),
                    color: Colors.blue[100 * (index % 9 + 1)],
                    child: Center(
                      child: Text(
                        'H $index',
                        style: const TextStyle(
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  );
                },
              ),
            ),
          ),
          
          // Section 2: Vertical list
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'Vertical List',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => Container(
                height: 60,
                color: Colors.green[100 * (index % 9 + 1)],
                margin: const EdgeInsets.all(4),
                child: Center(
                  child: Text(
                    'Item $index',
                    style: const TextStyle(fontWeight: FontWeight.bold),
                  ),
                ),
              ),
              childCount: 20,
            ),
          ),
          
          // Section 3: Grid
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'Grid',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          SliverGrid(
            delegate: SliverChildBuilderDelegate(
              (context, index) => Container(
                color: Colors.orange[100 * (index % 9 + 1)],
                child: Center(child: Text('$index')),
              ),
              childCount: 12,
            ),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 3,
              crossAxisSpacing: 4,
              mainAxisSpacing: 4,
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- CustomScrollView with mixed content
- Horizontal scroll nested inside vertical
- Different sliver types combined
- Smooth nested scrolling

---

# Scroll Conflicts

> **Handling scroll conflicts** in nested scroll.

```dart
// Handling scroll conflicts
class ScrollConflictExample extends StatelessWidget {
  const ScrollConflictExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Scroll Conflict'),
      ),
      body: Column(
        children: [
          // Fixed header
          Container(
            padding: const EdgeInsets.all(16),
            color: Colors.blue,
            child: const Text(
              'Fixed Header',
              style: TextStyle(
                color: Colors.white,
                fontSize: 18,
              ),
            ),
          ),
          
          // Nested scrollable content
          Expanded(
            child: ListView.builder(
              itemCount: 30,
              itemBuilder: (context, index) {
                return Card(
                  margin: const EdgeInsets.all(8),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      // Child scrollable widget
                      Container(
                        padding: const EdgeInsets.all(8),
                        child: const Text(
                          'Section Header',
                          style: TextStyle(
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ),
                      // Nested horizontal scroll
                      SizedBox(
                        height: 100,
                        child: ListView.builder(
                          scrollDirection: Axis.horizontal,
                          itemCount: 10,
                          itemBuilder: (context, childIndex) {
                            return Container(
                              width: 80,
                              margin: const EdgeInsets.all(4),
                              color: Colors.blue[100 * ((index + childIndex) % 9 + 1)],
                              child: Center(
                                child: Text(
                                  '${index + 1}-${childIndex + 1}',
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
- Vertical scroll with nested horizontal scroll
- No conflict because directions differ
- Horizontal scroll works independently
- Parent scrolls vertically, child scrolls horizontally

---

# Real-World Examples

> **Common patterns** with nested scrolling.

```dart
// 1. Profile with tabs (Instagram-like)
class InstagramProfile extends StatelessWidget {
  const InstagramProfile({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        body: NestedScrollView(
          headerSliverBuilder: (context, innerBoxIsScrolled) {
            return [
              SliverAppBar(
                title: const Text('@john_doe'),
                expandedHeight: 250,
                flexibleSpace: FlexibleSpaceBar(
                  background: Container(
                    padding: const EdgeInsets.all(16),
                    decoration: const BoxDecoration(
                      gradient: LinearGradient(
                        begin: Alignment.topCenter,
                        end: Alignment.bottomCenter,
                        colors: [Colors.blue, Colors.purple],
                      ),
                    ),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        const CircleAvatar(
                          radius: 50,
                          backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
                        ),
                        const SizedBox(height: 16),
                        const Text(
                          'John Doe',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 24,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        const Text(
                          'Flutter Developer | Blogger',
                          style: TextStyle(
                            color: Colors.white70,
                            fontSize: 16,
                          ),
                        ),
                        const SizedBox(height: 16),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                          children: [
                            _buildStat('Posts', '150'),
                            _buildStat('Followers', '2.5K'),
                            _buildStat('Following', '350'),
                          ],
                        ),
                      ],
                    ),
                  ),
                ),
                bottom: TabBar(
                  tabs: const [
                    Tab(icon: Icon(Icons.grid_on)),
                    Tab(icon: Icon(Icons.video_library)),
                    Tab(icon: Icon(Icons.person)),
                  ],
                  indicatorColor: Colors.white,
                ),
              ),
            ];
          },
          body: TabBarView(
            children: [
              // Grid posts
              GridView.builder(
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 3,
                  crossAxisSpacing: 2,
                  mainAxisSpacing: 2,
                ),
                itemCount: 30,
                itemBuilder: (context, index) {
                  return Container(
                    color: Colors.blue[100 * (index % 9 + 1)],
                    child: Center(
                      child: Text(
                        '${index + 1}',
                        style: const TextStyle(color: Colors.white),
                      ),
                    ),
                  );
                },
              ),
              
              // Videos
              ListView.builder(
                itemCount: 20,
                itemBuilder: (context, index) {
                  return ListTile(
                    leading: Container(
                      width: 80,
                      height: 60,
                      color: Colors.blue[100 * (index % 9 + 1)],
                      child: const Icon(Icons.play_arrow, color: Colors.white),
                    ),
                    title: Text('Video ${index + 1}'),
                    subtitle: Text('${index + 1} min ago'),
                  );
                },
              ),
              
              // About
              ListView(
                children: const [
                  ListTile(
                    leading: Icon(Icons.person),
                    title: Text('Name'),
                    subtitle: Text('John Doe'),
                  ),
                  ListTile(
                    leading: Icon(Icons.email),
                    title: Text('Email'),
                    subtitle: Text('john@example.com'),
                  ),
                  ListTile(
                    leading: Icon(Icons.link),
                    title: Text('Website'),
                    subtitle: Text('johndoe.dev'),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildStat(String label, String value) {
    return Column(
      children: [
        Text(
          value,
          style: const TextStyle(
            color: Colors.white,
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
        ),
        Text(
          label,
          style: const TextStyle(
            color: Colors.white70,
            fontSize: 12,
          ),
        ),
      ],
    );
  }
}

// 2. Product page with sections
class ProductPageNested extends StatelessWidget {
  const ProductPageNested({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            title: const Text('Product'),
            expandedHeight: 200,
            flexibleSpace: const FlexibleSpaceBar(
              background: ColoredBox(
                color: Colors.blue,
                child: Center(
                  child: Icon(
                    Icons.shopping_bag,
                    size: 80,
                    color: Colors.white,
                  ),
                ),
              ),
            ),
            pinned: true,
          ),
          
          // Product info
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Product Name',
                    style: TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  SizedBox(height: 8),
                  Text(
                    '\$99.99',
                    style: TextStyle(
                      fontSize: 20,
                      color: Colors.blue,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  SizedBox(height: 16),
                  Text(
                    'Product description goes here. '
                    'This is a detailed description of the product.',
                    style: TextStyle(fontSize: 16),
                  ),
                ],
              ),
            ),
          ),
          
          // Related products (horizontal scroll)
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'Related Products',
                style: TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          SliverToBoxAdapter(
            child: SizedBox(
              height: 200,
              child: ListView.builder(
                scrollDirection: Axis.horizontal,
                itemCount: 10,
                itemBuilder: (context, index) {
                  return Container(
                    width: 150,
                    margin: const EdgeInsets.symmetric(horizontal: 8),
                    decoration: BoxDecoration(
                      color: Colors.blue[100 * (index % 9 + 1)],
                      borderRadius: BorderRadius.circular(8),
                    ),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        const Icon(
                          Icons.shopping_bag,
                          color: Colors.white,
                          size: 40,
                        ),
                        const SizedBox(height: 8),
                        Text(
                          'Product ${index + 1}',
                          style: const TextStyle(
                            color: Colors.white,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ],
                    ),
                  );
                },
              ),
            ),
          ),
          
          // Reviews
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'Reviews',
                style: TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => ListTile(
                leading: CircleAvatar(
                  child: Text('${index + 1}'),
                ),
                title: Text('Review ${index + 1}'),
                subtitle: Text('Great product! ${index + 1} stars'),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Icon(
                      Icons.star,
                      color: Colors.yellow[700],
                      size: 16,
                    ),
                    Text(' ${5 - (index % 2)}'),
                  ],
                ),
              ),
              childCount: 10,
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Instagram-like profile with tabs
- Product page with horizontal scroll sections
- Nested scrolling in complex layouts
- Real-world scrolling patterns

---

# Best Practices

## Use NestedScrollView for Tabbed Content

```dart
// Good - NestedScrollView with tabs
NestedScrollView(
  headerSliverBuilder: (context, innerBoxIsScrolled) {
    return [
      SliverAppBar(
        title: const Text('Title'),
        bottom: const TabBar(tabs: [...])
      ),
    ];
  },
  body: TabBarView(children: [...]),
)
```

## Avoid Scroll Conflicts

```dart
// Good - Different scroll directions
ListView.builder(
  itemBuilder: (context, index) {
    return ListView.builder(
      scrollDirection: Axis.horizontal, // Different direction
      itemBuilder: (context, childIndex) => ...,
    );
  },
)
```

## Use Appropriate Physics

```dart
// Good - Physics for nested scroll
ListView.builder(
  physics: const ClampingScrollPhysics(),
  itemBuilder: (context, index) {
    return ListView.builder(
      physics: const NeverScrollableScrollPhysics(), // Disable nested scroll
      scrollDirection: Axis.horizontal,
      itemBuilder: (context, childIndex) => ...,
    );
  },
)
```

---

# Common Mistakes

## Conflicting Scroll Directions

Wrong:
```dart
// Both scrolling vertically - conflict
ListView.builder(
  itemBuilder: (context, index) {
    return ListView.builder( // Conflicts with parent
      itemBuilder: (context, childIndex) => ...,
    );
  },
)
```

Correct:
```dart
// Different directions or disable nested
ListView.builder(
  itemBuilder: (context, index) {
    return ListView.builder(
      scrollDirection: Axis.horizontal, // No conflict
      itemBuilder: (context, childIndex) => ...,
    );
  },
)
```

## Forgetting NestedScrollView

Wrong:
```dart
// Manual nesting without coordination
Scaffold(
  body: Column(
    children: [
      SliverAppBar(...), // Not a sliver
      Expanded(
        child: ListView.builder(...), // Won't coordinate
      ),
    ],
  ),
)
```

Correct:
```dart
// Use NestedScrollView
Scaffold(
  body: NestedScrollView(
    headerSliverBuilder: (context, innerBoxIsScrolled) {
      return [SliverAppBar(...)];
    },
    body: ListView.builder(...),
  ),
)
```

---

# Summary

Nested scrolling enables complex scrollable layouts where multiple scrollable widgets work together. Use NestedScrollView for tabbed content with headers, CustomScrollView for mixed slivers, and be mindful of scroll directions to avoid conflicts.

---

# Next Steps

- [Infinite Scrolling](infinitescrolling.md)
- [ScrollController](scrollcontroller.md)
- [ScrollPhysics](scrollphysics.md)

---

# Did You Know?

- NestedScrollView coordinates parent-child scrolling
- TabBarView works with NestedScrollView
- Scroll directions should differ to avoid conflicts
- CustomScrollView combines multiple slivers
- SliverAppBar can be pinned in nested scroll
- Nested scroll is used in most apps
- TabController manages tab navigation
- NestedScrollView is efficient and smooth