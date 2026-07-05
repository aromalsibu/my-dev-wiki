# Infinite Scrolling

Understand how to implement infinite scrolling (endless scrolling) in Flutter.

---

# What is it?

Infinite scrolling is a technique where new content is loaded automatically as the user approaches the end of the current content. This creates a seamless, endless scrolling experience commonly seen in social media feeds, search results, and product lists. When the user scrolls to the bottom, more data is loaded, providing a continuous stream of content.

---

# Why does it exist?

Infinite scrolling exists to:

- Load content on demand as the user scrolls
- Improve performance by loading small batches
- Create seamless user experiences
- Handle large datasets efficiently
- Implement feeds and timelines
- Reduce initial load time
- Enable smooth browsing without pagination

---

# Basic Infinite Scrolling

> **Loading more content** as user scrolls.

```dart
// Basic infinite scrolling example
class BasicInfiniteScroll extends StatefulWidget {
  const BasicInfiniteScroll({super.key});

  @override
  State<BasicInfiniteScroll> createState() => _BasicInfiniteScrollState();
}

class _BasicInfiniteScrollState extends State<BasicInfiniteScroll> {
  final ScrollController _controller = ScrollController();
  List<int> _items = [];
  bool _isLoading = false;
  bool _hasMore = true;
  int _page = 0;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _controller.addListener(() {
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
    if (_isLoading || !_hasMore) return;
    
    setState(() {
      _isLoading = true;
    });

    try {
      // Simulate network request
      await Future.delayed(const Duration(seconds: 1));
      
      final newItems = List.generate(20, (index) => _page * 20 + index);
      _items.addAll(newItems);
      _page++;
      
      // Simulate end of data (after 5 pages)
      if (_page >= 5) {
        _hasMore = false;
      }
      
      if (mounted) {
        setState(() {
          _isLoading = false;
        });
      }
    } catch (e) {
      if (mounted) {
        setState(() {
          _isLoading = false;
          // Handle error
        });
      }
    }
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
              padding: EdgeInsets.symmetric(vertical: 16),
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
```

What's happening here?
- Load more when near bottom
- Show loading indicator
- Track loading state
- _hasMore controls whether to load more
- Uses ScrollController to detect scroll position

---

# Infinite Scrolling with Data Models

> **Loading structured data** with infinite scroll.

```dart
// Infinite scrolling with data models
class Post {
  final int id;
  final String title;
  final String body;
  final String author;

  Post({
    required this.id,
    required this.title,
    required this.body,
    required this.author,
  });
}

class InfiniteScrollWithData extends StatefulWidget {
  const InfiniteScrollWithData({super.key});

  @override
  State<InfiniteScrollWithData> createState() => _InfiniteScrollWithDataState();
}

class _InfiniteScrollWithDataState extends State<InfiniteScrollWithData> {
  final ScrollController _controller = ScrollController();
  List<Post> _posts = [];
  bool _isLoading = false;
  bool _hasMore = true;
  int _page = 1;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _controller.addListener(() {
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
    if (_isLoading || !_hasMore) return;
    
    setState(() => _isLoading = true);

    try {
      // Simulate API call
      await Future.delayed(const Duration(seconds: 1));
      
      final newPosts = List.generate(10, (index) {
        final id = (_page - 1) * 10 + index;
        return Post(
          id: id,
          title: 'Post ${id + 1}',
          body: 'This is the body of post ${id + 1}. It contains some content.',
          author: 'User ${id % 5 + 1}',
        );
      });
      
      _posts.addAll(newPosts);
      _page++;
      
      if (_page > 5) _hasMore = false;
      
      if (mounted) {
        setState(() => _isLoading = false);
      }
    } catch (e) {
      if (mounted) {
        setState(() => _isLoading = false);
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Feed'),
      ),
      body: ListView.builder(
        controller: _controller,
        itemCount: _posts.length + (_isLoading ? 1 : 0),
        itemBuilder: (context, index) {
          if (index == _posts.length) {
            return const Padding(
              padding: EdgeInsets.all(16),
              child: Center(child: CircularProgressIndicator()),
            );
          }
          
          final post = _posts[index];
          return Card(
            margin: const EdgeInsets.all(8),
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    children: [
                      CircleAvatar(
                        backgroundColor: Colors.blue[100],
                        child: Text(post.author[0]),
                      ),
                      const SizedBox(width: 12),
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              post.author,
                              style: const TextStyle(
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                            Text(
                              'Post ${post.id + 1}',
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
                  Text(
                    post.title,
                    style: const TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    post.body,
                    style: TextStyle(
                      color: Colors.grey[700],
                      fontSize: 14,
                      height: 1.5,
                    ),
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
                    ],
                  ),
                ],
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
- Structured data (Post model)
- Cards with rich content
- User avatar, title, body
- Interactive buttons
- Page-based loading

---

# Infinite Scrolling with Error Handling

> **Handling errors** in infinite scrolling.

```dart
// Infinite scrolling with error handling
class InfiniteScrollWithError extends StatefulWidget {
  const InfiniteScrollWithError({super.key});

  @override
  State<InfiniteScrollWithError> createState() => _InfiniteScrollWithErrorState();
}

class _InfiniteScrollWithErrorState extends State<InfiniteScrollWithError> {
  final ScrollController _controller = ScrollController();
  List<int> _items = [];
  bool _isLoading = false;
  bool _hasMore = true;
  int _page = 0;
  String? _error;
  bool _initialLoad = true;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _controller.addListener(() {
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
    if (_isLoading || !_hasMore) return;
    
    setState(() {
      _isLoading = true;
      _error = null;
    });

    try {
      // Simulate API with random errors
      await Future.delayed(const Duration(seconds: 1));
      
      // Randomly fail
      if (_page % 4 == 3) {
        throw Exception('Failed to load data');
      }
      
      final newItems = List.generate(15, (index) => _page * 15 + index);
      _items.addAll(newItems);
      _page++;
      
      if (_page >= 6) _hasMore = false;
      
      if (mounted) {
        setState(() {
          _isLoading = false;
          _initialLoad = false;
        });
      }
    } catch (e) {
      if (mounted) {
        setState(() {
          _isLoading = false;
          _error = e.toString();
          _initialLoad = false;
        });
      }
    }
  }

  Future<void> _retry() async {
    if (_error != null) {
      await _loadMore();
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_initialLoad && _isLoading) {
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }

    return Scaffold(
      appBar: AppBar(
        title: const Text('Infinite Scroll with Error'),
        actions: [
          if (_error != null)
            IconButton(
              icon: const Icon(Icons.refresh),
              onPressed: _retry,
            ),
        ],
      ),
      body: Column(
        children: [
          if (_error != null)
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.red[50],
              child: Row(
                children: [
                  const Icon(Icons.error, color: Colors.red),
                  const SizedBox(width: 8),
                  Expanded(
                    child: Text(
                      'Error: $_error',
                      style: const TextStyle(color: Colors.red),
                    ),
                  ),
                  TextButton(
                    onPressed: _retry,
                    child: const Text('Retry'),
                  ),
                ],
              ),
            ),
          Expanded(
            child: ListView.builder(
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
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Error handling with retry
- Error banner at top
- Retry button in app bar
- Graceful error recovery
- Initial loading state

---

# Real-World Examples

> **Common patterns** for infinite scrolling.

```dart
// 1. Search with infinite scrolling
class SearchWithInfiniteScroll extends StatefulWidget {
  const SearchWithInfiniteScroll({super.key});

  @override
  State<SearchWithInfiniteScroll> createState() => _SearchWithInfiniteScrollState();
}

class _SearchWithInfiniteScrollState extends State<SearchWithInfiniteScroll> {
  final ScrollController _controller = ScrollController();
  final TextEditingController _searchController = TextEditingController();
  List<String> _results = [];
  bool _isLoading = false;
  bool _hasMore = true;
  int _page = 0;
  String _query = '';

  @override
  void initState() {
    super.initState();
    _controller.addListener(() {
      if (_controller.position.pixels >= _controller.position.maxScrollExtent - 200) {
        _searchMore();
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    _searchController.dispose();
    super.dispose();
  }

  Future<void> _searchMore() async {
    if (_isLoading || !_hasMore || _query.isEmpty) return;
    
    setState(() => _isLoading = true);

    try {
      await Future.delayed(const Duration(seconds: 1));
      
      final newResults = List.generate(
        10,
        (index) => '${_query} result ${_page * 10 + index + 1}',
      );
      
      _results.addAll(newResults);
      _page++;
      
      if (_page >= 5) _hasMore = false;
      
      if (mounted) {
        setState(() => _isLoading = false);
      }
    } catch (e) {
      if (mounted) {
        setState(() => _isLoading = false);
      }
    }
  }

  void _performSearch(String query) {
    setState(() {
      _query = query;
      _results = [];
      _page = 0;
      _hasMore = true;
    });
    if (query.isNotEmpty) {
      _searchMore();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Search'),
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8),
            child: TextField(
              controller: _searchController,
              decoration: InputDecoration(
                hintText: 'Search...',
                prefixIcon: const Icon(Icons.search),
                suffixIcon: IconButton(
                  icon: const Icon(Icons.clear),
                  onPressed: () {
                    _searchController.clear();
                    _performSearch('');
                  },
                ),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(8),
                ),
              ),
              onChanged: _performSearch,
            ),
          ),
          Expanded(
            child: _results.isEmpty && !_isLoading
                ? Center(
                    child: Text(
                      _query.isEmpty ? 'Search for something' : 'No results found',
                      style: TextStyle(
                        color: Colors.grey[600],
                      ),
                    ),
                  )
                : ListView.builder(
                    controller: _controller,
                    itemCount: _results.length + (_isLoading ? 1 : 0),
                    itemBuilder: (context, index) {
                      if (index == _results.length) {
                        return const Padding(
                          padding: EdgeInsets.all(16),
                          child: Center(child: CircularProgressIndicator()),
                        );
                      }
                      return ListTile(
                        leading: CircleAvatar(
                          child: Text('${index + 1}'),
                        ),
                        title: Text(_results[index]),
                        onTap: () {},
                      );
                    },
                  ),
          ),
        ],
      ),
    );
  }
}

// 2. Grid with infinite scrolling (Instagram-like)
class InfiniteGridScroll extends StatefulWidget {
  const InfiniteGridScroll({super.key});

  @override
  State<InfiniteGridScroll> createState() => _InfiniteGridScrollState();
}

class _InfiniteGridScrollState extends State<InfiniteGridScroll> {
  final ScrollController _controller = ScrollController();
  List<int> _items = [];
  bool _isLoading = false;
  bool _hasMore = true;
  int _page = 0;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _controller.addListener(() {
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
    if (_isLoading || !_hasMore) return;
    
    setState(() => _isLoading = true);
    
    await Future.delayed(const Duration(seconds: 1));
    final newItems = List.generate(12, (index) => _page * 12 + index);
    _items.addAll(newItems);
    _page++;
    
    if (_page >= 5) _hasMore = false;
    
    if (mounted) {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Photo Grid'),
      ),
      body: GridView.builder(
        controller: _controller,
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 3,
          crossAxisSpacing: 2,
          mainAxisSpacing: 2,
        ),
        itemCount: _items.length + (_isLoading ? 1 : 0),
        itemBuilder: (context, index) {
          if (index == _items.length) {
            return const Center(child: CircularProgressIndicator());
          }
          return Container(
            color: Colors.blue[100 * (_items[index] % 9 + 1)],
            child: Center(
              child: Text(
                '${_items[index] + 1}',
                style: const TextStyle(
                  color: Colors.white,
                  fontWeight: FontWeight.bold,
                ),
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
- Search with pagination
- Grid view infinite scrolling
- Real-world social media patterns
- Search and filter functionality

---

# Best Practices

## Use Controller for Scroll Detection

```dart
// Good - Listen to scroll events
@override
void initState() {
  super.initState();
  _controller.addListener(() {
    if (_controller.position.pixels >= _controller.position.maxScrollExtent - 200) {
      _loadMore();
    }
  });
}
```

## Show Loading Indicator

```dart
// Good - Show loading state
if (index == _items.length) {
  return const Padding(
    padding: EdgeInsets.all(16),
    child: Center(child: CircularProgressIndicator()),
  );
}
```

## Handle Errors Gracefully

```dart
// Good - Error handling
try {
  await _loadMore();
} catch (e) {
  setState(() {
    _error = e.toString();
  });
}
```

---

# Common Mistakes

## Not Checking Loading State

Wrong:
```dart
// Multiple simultaneous requests
void _loadMore() {
  // No loading check
  loadData();
}
```

Correct:
```dart
// Check loading state
void _loadMore() {
  if (_isLoading) return;
  // Load data
}
```

## Not Handling End of Data

Wrong:
```dart
// Infinite loading even when no data
void _loadMore() {
  // No end check
  loadMoreData();
}
```

Correct:
```dart
// Check for end of data
void _loadMore() {
  if (!_hasMore) return;
  loadMoreData();
}
```

---

# Summary

Infinite scrolling loads content automatically as the user scrolls down. Use ScrollController to detect when the user approaches the end of the list, show loading indicators, handle errors gracefully, and stop loading when no more data exists.

---

# Next Steps

- [ScrollController](scrollcontroller.md)
- [ScrollPhysics](scrollphysics.md)
- [Nested Scrolling](nestedscrolling.md)

---

# Did You Know?

- Infinite scrolling improves UX by reducing pagination
- ScrollController detects scroll position
- Loading state prevents duplicate requests
- Error handling is essential for reliability
- Grid views also support infinite scrolling
- Search results use infinite scrolling
- Social media feeds use infinite scrolling
- Infinite scrolling is common in modern apps