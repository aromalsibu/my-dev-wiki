# ListView

Understand how to create scrollable lists in Flutter.

---

# What is it?

ListView is the most commonly used scrolling widget in Flutter. It displays a scrollable list of widgets arranged linearly, either vertically or horizontally. ListView provides multiple constructors for different use cases, from simple fixed lists to large, efficient lists with thousands of items.

---

# Why does it exist?

ListView exists to:

- Display scrollable lists of widgets
- Handle large datasets efficiently
- Support both vertical and horizontal scrolling
- Provide multiple constructors for different needs
- Enable lazy loading for performance
- Support interactive list items
- Create common UI patterns like menus, feeds, etc.

---

# ListView Constructors

> **Multiple constructors** for different use cases.

```dart
// 1. ListView() - Fixed list
class FixedListView extends StatelessWidget {
  const FixedListView({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView(
      padding: const EdgeInsets.all(8),
      children: [
        for (int i = 0; i < 20; i++)
          Container(
            height: 50,
            color: Colors.blue[100 * (i % 9 + 1)],
            margin: const EdgeInsets.only(bottom: 4),
            child: Center(child: Text('Item $i')),
          ),
      ],
    );
  }
}

// 2. ListView.builder() - Efficient for large lists
class BuilderListView extends StatelessWidget {
  const BuilderListView({super.key});

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
          onTap: () {},
        );
      },
    );
  }
}

// 3. ListView.separated() - With separators
class SeparatedListView extends StatelessWidget {
  const SeparatedListView({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.separated(
      itemCount: 20,
      separatorBuilder: (context, index) => const Divider(
        color: Colors.grey,
        thickness: 1,
        indent: 16,
        endIndent: 16,
      ),
      itemBuilder: (context, index) {
        return ListTile(
          leading: Icon(Icons.star, color: Colors.blue[100 * (index % 9 + 1)]),
          title: Text('Item $index'),
          trailing: const Icon(Icons.arrow_forward),
          onTap: () {},
        );
      },
    );
  }
}

// 4. ListView.custom() - Custom children
class CustomListView extends StatelessWidget {
  const CustomListView({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.custom(
      childrenDelegate: SliverChildListDelegate(
        [
          for (int i = 0; i < 20; i++)
            Container(
              height: 50,
              color: Colors.green[100 * (i % 9 + 1)],
              margin: const EdgeInsets.all(4),
              child: Center(child: Text('Custom $i')),
            ),
        ],
      ),
    );
  }
}
```

What's happening here?
- ListView(): fixed list of children
- ListView.builder(): efficient for large lists
- ListView.separated(): with separators between items
- ListView.custom(): custom child delegate

---

# ListView Properties

> **Key properties** for customizing ListView.

```dart
// ListView with various properties
class ListViewPropertiesExample extends StatelessWidget {
  const ListViewPropertiesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      // 1. scrollDirection - Horizontal or vertical
      scrollDirection: Axis.vertical,
      
      // 2. reverse - Reverse order
      reverse: false,
      
      // 3. controller - Scroll controller
      // controller: _controller,
      
      // 4. physics - Scroll physics
      physics: const BouncingScrollPhysics(),
      
      // 5. padding - Padding around list
      padding: const EdgeInsets.all(16),
      
      // 6. itemCount - Number of items
      itemCount: 50,
      
      // 7. itemBuilder - Build each item
      itemBuilder: (context, index) {
        return Container(
          height: 50,
          color: Colors.blue[100 * (index % 9 + 1)],
          margin: const EdgeInsets.only(bottom: 8),
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
      
      // 8. shrinkWrap - Shrink to fit content
      // shrinkWrap: true,
      
      // 9. cacheExtent - Caching range
      // cacheExtent: 1000,
      
      // 10. semanticChildCount - Semantic child count
      // semanticChildCount: 50,
    );
  }
}
```

What's happening here?
- scrollDirection: vertical or horizontal
- reverse: reverse order
- physics: scroll behavior
- shrinkWrap: fit to content
- cacheExtent: performance tuning

---

# Horizontal ListView

> **Horizontal scrolling** with ListView.

```dart
// Horizontal ListView
class HorizontalListView extends StatelessWidget {
  const HorizontalListView({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 120,
      child: ListView.builder(
        scrollDirection: Axis.horizontal,
        itemCount: 20,
        padding: const EdgeInsets.symmetric(horizontal: 8),
        itemBuilder: (context, index) {
          return Container(
            width: 100,
            margin: const EdgeInsets.symmetric(horizontal: 4),
            decoration: BoxDecoration(
              color: Colors.blue[100 * (index % 9 + 1)],
              borderRadius: BorderRadius.circular(8),
            ),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(
                  Icons.star,
                  color: Colors.yellow[700],
                  size: 30,
                ),
                const SizedBox(height: 4),
                Text(
                  'Item $index',
                  style: const TextStyle(
                    fontWeight: FontWeight.bold,
                    color: Colors.white,
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

What's happening here?
- Horizontal scrolling with scrollDirection
- Items arranged left to right
- Useful for carousels, galleries
- Can be customized like vertical

---

# ListView with Complex Items

> **Building complex** list items.

```dart
// Complex list items
class ComplexListItems extends StatelessWidget {
  const ComplexListItems({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
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
                // Header with avatar and name
                Row(
                  children: [
                    CircleAvatar(
                      backgroundColor: Colors.blue[100 * (index % 9 + 1)],
                      child: Text('${index + 1}'),
                    ),
                    const SizedBox(width: 12),
                    Expanded(
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(
                            'User ${index + 1}',
                            style: const TextStyle(
                              fontWeight: FontWeight.bold,
                              fontSize: 16,
                            ),
                          ),
                          Text(
                            'Posted 2 hours ago',
                            style: TextStyle(
                              color: Colors.grey[600],
                              fontSize: 12,
                            ),
                          ),
                        ],
                      ),
                    ),
                    IconButton(
                      icon: const Icon(Icons.more_vert),
                      onPressed: () {},
                    ),
                  ],
                ),
                
                const SizedBox(height: 12),
                
                // Content
                Text(
                  'This is the content for item $index. '
                  'It contains some text that explains what this post is about.',
                  style: TextStyle(
                    fontSize: 14,
                    height: 1.5,
                    color: Colors.grey[800],
                  ),
                ),
                
                const SizedBox(height: 12),
                
                // Image placeholder
                Container(
                  height: 150,
                  decoration: BoxDecoration(
                    color: Colors.blue[100 * (index % 9 + 1)],
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: const Center(
                    child: Icon(
                      Icons.image,
                      size: 40,
                      color: Colors.white,
                    ),
                  ),
                ),
                
                const SizedBox(height: 12),
                
                // Actions
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
    );
  }
}
```

What's happening here?
- Card-based list items
- Complex layouts in each item
- Images, text, and actions
- Interactive elements

---

# ListView with Different Item Types

> **Mixed item types** in a list.

```dart
// Mixed item types
class MixedItemListView extends StatelessWidget {
  const MixedItemListView({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 15,
      itemBuilder: (context, index) {
        // Different item types based on index
        if (index == 0) {
          return _buildHeaderItem();
        } else if (index % 3 == 0) {
          return _buildAdvertisementItem(index);
        } else if (index % 2 == 0) {
          return _buildImageItem(index);
        } else {
          return _buildTextItem(index);
        }
      },
    );
  }

  Widget _buildHeaderItem() {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.blue,
      width: double.infinity,
      child: const Column(
        children: [
          Text(
            'Welcome to the List',
            style: TextStyle(
              color: Colors.white,
              fontSize: 24,
              fontWeight: FontWeight.bold,
            ),
          ),
          SizedBox(height: 8),
          Text(
            'Scroll down to see different item types',
            style: TextStyle(
              color: Colors.white70,
              fontSize: 16,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildAdvertisementItem(int index) {
    return Container(
      margin: const EdgeInsets.all(8),
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.orange[100],
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: Colors.orange),
      ),
      child: const Row(
        children: [
          Icon(Icons.ads_click, color: Colors.orange),
          SizedBox(width: 12),
          Text(
            'Sponsored Content',
            style: TextStyle(
              fontWeight: FontWeight.bold,
              color: Colors.orange,
            ),
          ),
          Spacer(),
          Text('Learn More →'),
        ],
      ),
    );
  }

  Widget _buildImageItem(int index) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Container(
            height: 150,
            decoration: BoxDecoration(
              color: Colors.green[100 * (index % 9 + 1)],
              borderRadius: const BorderRadius.only(
                topLeft: Radius.circular(4),
                topRight: Radius.circular(4),
              ),
            ),
            child: const Center(
              child: Icon(Icons.image, size: 40, color: Colors.white),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(12),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'Image Post #$index',
                  style: const TextStyle(
                    fontWeight: FontWeight.bold,
                    fontSize: 16,
                  ),
                ),
                const SizedBox(height: 4),
                const Text('This is a post with an image'),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildTextItem(int index) {
    return ListTile(
      leading: CircleAvatar(
        backgroundColor: Colors.purple[100 * (index % 9 + 1)],
        child: Text('${index + 1}'),
      ),
      title: Text('Text Post #$index'),
      subtitle: Text('This is a regular text-based post item'),
      trailing: const Icon(Icons.arrow_forward),
      onTap: () {},
    );
  }
}
```

What's happening here?
- Different item types in one list
- Header, advertisements, images, text
- Each type has different layout
- Flexible and dynamic

---

# Real-World Examples

> **Common patterns** using ListView.

```dart
// 1. Settings screen
class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Settings')),
      body: ListView(
        children: [
          // Profile section
          Container(
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
                    Text('john.doe@example.com'),
                  ],
                ),
                const Spacer(),
                IconButton(
                  icon: const Icon(Icons.edit),
                  onPressed: () {},
                ),
              ],
            ),
          ),
          
          const Divider(height: 1),
          
          // Settings groups
          _buildSettingsGroup(
            'General',
            [
              _buildSettingsTile(Icons.person, 'Profile', () {}),
              _buildSettingsTile(Icons.palette, 'Theme', () {}),
              _buildSettingsTile(Icons.language, 'Language', () {}),
            ],
          ),
          
          _buildSettingsGroup(
            'Privacy',
            [
              _buildSettingsTile(Icons.lock, 'Privacy Settings', () {}),
              _buildSettingsTile(Icons.notifications, 'Notifications', () {}),
              _buildSettingsTile(Icons.security, 'Security', () {}),
            ],
          ),
          
          _buildSettingsGroup(
            'Support',
            [
              _buildSettingsTile(Icons.help, 'Help & Feedback', () {}),
              _buildSettingsTile(Icons.info, 'About', () {}),
            ],
          ),
        ],
      ),
    );
  }

  Widget _buildSettingsGroup(String title, List<Widget> tiles) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Padding(
          padding: const EdgeInsets.fromLTRB(16, 16, 16, 8),
          child: Text(
            title,
            style: TextStyle(
              color: Colors.grey[600],
              fontSize: 14,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
        Column(children: tiles),
      ],
    );
  }

  Widget _buildSettingsTile(IconData icon, String title, VoidCallback onTap) {
    return ListTile(
      leading: Icon(icon),
      title: Text(title),
      trailing: const Icon(Icons.arrow_forward_ios, size: 16),
      onTap: onTap,
    );
  }
}

// 2. Contact list with sections
class ContactList extends StatelessWidget {
  const ContactList({super.key});

  @override
  Widget build(BuildContext context) {
    final contacts = {
      'A': ['Alice', 'Alex', 'Amanda'],
      'B': ['Bob', 'Brian', 'Betty'],
      'C': ['Charlie', 'Carol', 'Chris'],
      'D': ['David', 'Diana', 'Dan'],
    };

    return ListView.builder(
      itemCount: contacts.length,
      itemBuilder: (context, index) {
        final letter = contacts.keys.elementAt(index);
        final names = contacts[letter]!;
        
        return Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Section header
            Container(
              padding: const EdgeInsets.all(8),
              color: Colors.grey[200],
              child: Text(
                letter,
                style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  fontSize: 16,
                ),
              ),
            ),
            // Contact tiles
            ...names.map((name) {
              return ListTile(
                leading: CircleAvatar(
                  backgroundColor: Colors.blue[100],
                  child: Text(name[0]),
                ),
                title: Text(name),
                onTap: () {},
              );
            }),
          ],
        );
      },
    );
  }
}
```

What's happening here?
- Settings screen with groups
- Contact list with sections
- Common real-world patterns
- Interactive list items

---

# Best Practices

## Use Builder for Large Lists

```dart
// Good - Efficient for large lists
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: 10000,
    itemBuilder: (context, index) {
      return ListTile(title: Text('Item $index'));
    },
  );
}

// Bad - Inefficient for large lists
@override
Widget build(BuildContext context) {
  return ListView(
    children: List.generate(10000, (index) {
      return ListTile(title: Text('Item $index'));
    }),
  );
}
```

## Use Keys for Dynamic Lists

```dart
// Good - With keys
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) {
      return ListTile(
        key: ValueKey(items[index].id),
        title: Text(items[index].name),
      );
    },
  );
}
```

## Optimize Item Builds

```dart
// Good - Optimized item building
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) {
      return const MyListItem(); // Const where possible
    },
  );
}
```

---

# Common Mistakes

## Not Using Builder

Wrong:
```dart
// All items built at once
ListView(
  children: List.generate(1000, (index) {
    return Text('Item $index');
  }),
)
```

Correct:
```dart
// Lazy loading
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return Text('Item $index');
  },
)
```

## Forgetting Keys

Wrong:
```dart
// No keys - performance issues
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
)
```

Correct:
```dart
// With keys
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(
      key: ValueKey(index),
      title: Text(items[index]),
    );
  },
)
```

---

# Summary

ListView is the most common scrolling widget for displaying lists. Use different constructors based on your needs: ListView for fixed lists, ListView.builder for large lists, ListView.separated for lists with separators. Always use builder for large datasets and keys for dynamic lists.

---

# Next Steps

- [GridView](gridview.md)
- [CustomScrollView](customscrollview.md)
- [Slivers](slivers.md)

---

# Did You Know?

- ListView.builder builds items lazily
- ListView.separated adds separators automatically
- ListView can scroll horizontally
- Keys improve list performance
- ListView supports caching via cacheExtent
- ListView can shrink to fit content
- ListView supports custom physics
- ListView is used in most Flutter apps