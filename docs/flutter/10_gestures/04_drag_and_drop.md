# Drag & Drop

Understand how to implement drag and drop functionality in Flutter.

---

# What is it?

Drag & Drop is an interaction pattern where users can drag a widget from one location and drop it onto another. Flutter provides robust support for drag and drop through dedicated widgets like Draggable, DragTarget, and LongPressDraggable. These widgets handle the complex interactions of dragging, providing visual feedback during the drag operation.

---

# Why does it exist?

Drag & Drop exists to:

- Enable intuitive user interactions
- Support reordering of items
- Allow dragging content between areas
- Create interactive UIs
- Support file and data transfer
- Enhance user experience
- Enable gesture-based interactions

---

# Basic Draggable

> **Creating draggable** widgets.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic Draggable example
class BasicDraggableExample extends StatefulWidget {
  const BasicDraggableExample({super.key});

  @override
  State<BasicDraggableExample> createState() => _BasicDraggableExampleState();
}

class _BasicDraggableExampleState extends State<BasicDraggableExample> {
  // 1. Track drop target color
  Color _targetColor = Colors.grey[300]!;
  String _lastDroppedData = 'Nothing dropped yet';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Draggable Example'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. Row of draggable widgets
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                // Red draggable
                _buildDraggable(
                  color: Colors.red,
                  data: 'Red',
                  label: 'Red',
                ),
                
                // Green draggable
                _buildDraggable(
                  color: Colors.green,
                  data: 'Green',
                  label: 'Green',
                ),
                
                // Blue draggable
                _buildDraggable(
                  color: Colors.blue,
                  data: 'Blue',
                  label: 'Blue',
                ),
              ],
            ),
            
            const SizedBox(height: 24),
            
            // 3. Drop target
            _buildDropTarget(),
            
            const SizedBox(height: 24),
            
            // 4. Status display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Text(
                'Last dropped: $_lastDroppedData',
                style: const TextStyle(fontSize: 16),
              ),
            ),
          ],
        ),
      ),
    );
  }

  // Helper to build draggable widgets
  Widget _buildDraggable({
    required Color color,
    required String data,
    required String label,
  }) {
    return Draggable<String>(
      // 5. Data to be transferred
      data: data,
      
      // 6. Feedback widget shown during drag
      feedback: Container(
        width: 80,
        height: 80,
        decoration: BoxDecoration(
          color: color,
          borderRadius: BorderRadius.circular(8),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.3),
              blurRadius: 10,
              spreadRadius: 2,
            ),
          ],
        ),
        child: Center(
          child: Text(
            label,
            style: const TextStyle(color: Colors.white),
          ),
        ),
      ),
      
      // 7. Child when not dragging
      child: Container(
        width: 60,
        height: 60,
        decoration: BoxDecoration(
          color: color,
          borderRadius: BorderRadius.circular(8),
        ),
        child: Center(
          child: Text(
            label,
            style: const TextStyle(color: Colors.white),
          ),
        ),
      ),
      
      // 8. Child shown when dragging
      childWhenDragging: Container(
        width: 60,
        height: 60,
        decoration: BoxDecoration(
          color: color.withOpacity(0.3),
          borderRadius: BorderRadius.circular(8),
        ),
        child: const Center(
          child: Text(
            'Dragging',
            style: TextStyle(
              color: Colors.white,
              fontSize: 10,
            ),
          ),
        ),
      ),
    );
  }

  // Helper to build drop target
  Widget _buildDropTarget() {
    return DragTarget<String>(
      // 9. Accept condition
      onWillAccept: (data) {
        // Only accept if data is valid
        return data != null;
      },
      
      // 10. When data is dropped
      onAccept: (data) {
        setState(() {
          // Update target color based on dropped data
          switch (data) {
            case 'Red':
              _targetColor = Colors.red;
              break;
            case 'Green':
              _targetColor = Colors.green;
              break;
            case 'Blue':
              _targetColor = Colors.blue;
              break;
          }
          _lastDroppedData = data;
        });
      },
      
      // 11. Builder for target appearance
      builder: (context, candidateData, rejectedData) {
        return Container(
          width: double.infinity,
          height: 150,
          decoration: BoxDecoration(
            color: _targetColor,
            borderRadius: BorderRadius.circular(16),
            border: Border.all(
              color: candidateData.isNotEmpty ? Colors.blue : Colors.grey,
              width: candidateData.isNotEmpty ? 3 : 1,
            ),
          ),
          child: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Show what's being dragged
                if (candidateData.isNotEmpty)
                  const Text(
                    'Drop here!',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  )
                else
                  const Text(
                    'Drop your color here',
                    style: TextStyle(
                      color: Colors.white70,
                      fontSize: 18,
                    ),
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
- Draggable: Makes widget draggable
- DragTarget: Accepts dropped items
- data: Transfers information
- feedback: Widget shown during drag
- onAccept: Handles successful drop

---

# LongPressDraggable

> **Drag on long press** for better UX.

```dart
/// LongPressDraggable example
class LongPressDraggableExample extends StatefulWidget {
  const LongPressDraggableExample({super.key});

  @override
  State<LongPressDraggableExample> createState() => _LongPressDraggableExampleState();
}

class _LongPressDraggableExampleState extends State<LongPressDraggableExample> {
  // 1. Track items
  List<String> _items = ['Item 1', 'Item 2', 'Item 3', 'Item 4', 'Item 5'];
  List<String> _droppedItems = [];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('LongPressDraggable'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            // 2. Source list (items to drag)
            Expanded(
              child: Column(
                children: [
                  const Text(
                    'Drag Items',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 18,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Expanded(
                    child: Container(
                      decoration: BoxDecoration(
                        color: Colors.grey[100],
                        borderRadius: BorderRadius.circular(8),
                      ),
                      child: ListView.builder(
                        itemCount: _items.length,
                        itemBuilder: (context, index) {
                          return _buildDraggableItem(_items[index]);
                        },
                      ),
                    ),
                  ),
                ],
              ),
            ),
            
            const SizedBox(width: 16),
            
            // 3. Target list (drop destination)
            Expanded(
              child: Column(
                children: [
                  const Text(
                    'Drop Here',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 18,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Expanded(
                    child: _buildDropArea(),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  // Build draggable item
  Widget _buildDraggableItem(String item) {
    return LongPressDraggable<String>(
      // 4. Data to transfer
      data: item,
      
      // 5. Delay before drag starts
      delay: const Duration(milliseconds: 200),
      
      // 6. Feedback widget
      feedback: Container(
        padding: const EdgeInsets.all(16),
        color: Colors.blue[100],
        child: Text(
          item,
          style: const TextStyle(fontWeight: FontWeight.bold),
        ),
      ),
      
      // 7. Child widget
      child: Container(
        margin: const EdgeInsets.all(4),
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: Colors.blue[50],
          borderRadius: BorderRadius.circular(4),
          border: Border.all(color: Colors.blue[200]!),
        ),
        child: Row(
          children: [
            const Icon(Icons.drag_handle, color: Colors.grey),
            const SizedBox(width: 8),
            Text(item),
          ],
        ),
      ),
      
      // 8. Handle drag start/end
      onDragStarted: () {
        print('Started dragging: $item');
      },
      onDragEnd: (details) {
        print('Drag ended: $item');
      },
    );
  }

  // Build drop area
  Widget _buildDropArea() {
    return DragTarget<String>(
      // 9. Accept condition
      onWillAccept: (data) {
        return data != null && _items.contains(data);
      },
      
      // 10. Accept drop
      onAccept: (data) {
        setState(() {
          // Remove from source
          _items.remove(data);
          // Add to dropped
          _droppedItems.add(data);
        });
      },
      
      // 11. Builder
      builder: (context, candidateData, rejectedData) {
        return Container(
          width: double.infinity,
          decoration: BoxDecoration(
            color: candidateData.isNotEmpty ? Colors.blue[100] : Colors.grey[100],
            borderRadius: BorderRadius.circular(8),
            border: Border.all(
              color: candidateData.isNotEmpty ? Colors.blue : Colors.grey,
              width: candidateData.isNotEmpty ? 3 : 1,
            ),
          ),
          child: _droppedItems.isEmpty
              ? const Center(
                  child: Text(
                    'Drop items here',
                    style: TextStyle(color: Colors.grey),
                  ),
                )
              : ListView.builder(
                  itemCount: _droppedItems.length,
                  itemBuilder: (context, index) {
                    return Container(
                      margin: const EdgeInsets.all(4),
                      padding: const EdgeInsets.all(12),
                      decoration: BoxDecoration(
                        color: Colors.green[100],
                        borderRadius: BorderRadius.circular(4),
                        border: Border.all(color: Colors.green[200]!),
                      ),
                      child: Text(_droppedItems[index]),
                    );
                  },
                ),
        );
      },
    );
  }
}
```

What's happening here?
- LongPressDraggable: Drag on long press
- delay: Time before drag starts
- Lists management
- Drag from one list to another

---

# Reorderable List

> **Reordering list** items with drag.

```dart
/// Reorderable list example
class ReorderableListExample extends StatefulWidget {
  const ReorderableListExample({super.key});

  @override
  State<ReorderableListExample> createState() => _ReorderableListExampleState();
}

class _ReorderableListExampleState extends State<ReorderableListExample> {
  // 1. List of items
  List<String> _items = [
    'Item 1',
    'Item 2',
    'Item 3',
    'Item 4',
    'Item 5',
    'Item 6',
    'Item 7',
    'Item 8',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Reorderable List'),
        actions: [
          // 2. Reset button
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              setState(() {
                _items = [
                  'Item 1',
                  'Item 2',
                  'Item 3',
                  'Item 4',
                  'Item 5',
                  'Item 6',
                  'Item 7',
                  'Item 8',
                ];
              });
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 3. Instruction text
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Row(
                children: [
                  Icon(Icons.drag_handle, color: Colors.blue),
                  SizedBox(width: 8),
                  Expanded(
                    child: Text(
                      'Drag the handle (☰) to reorder items',
                      style: TextStyle(fontSize: 14),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 16),
            
            // 4. ReorderableListView
            Expanded(
              child: ReorderableListView(
                // 5. Items list
                children: _items.map((item) {
                  return _buildListItem(item);
                }).toList(),
                
                // 6. Handle reorder
                onReorder: (oldIndex, newIndex) {
                  setState(() {
                    // Adjust index for removal
                    if (newIndex > oldIndex) {
                      newIndex -= 1;
                    }
                    
                    // Reorder the list
                    final item = _items.removeAt(oldIndex);
                    _items.insert(newIndex, item);
                  });
                },
              ),
            ),
          ],
        ),
      ),
    );
  }

  // Build list item
  Widget _buildListItem(String item) {
    return Container(
      // 7. Key must be unique
      key: Key(item),
      margin: const EdgeInsets.symmetric(vertical: 4),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(8),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.05),
            blurRadius: 4,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: ListTile(
        // 8. Drag handle
        leading: const Icon(
          Icons.drag_handle,
          color: Colors.grey,
        ),
        title: Text(item),
        trailing: IconButton(
          icon: const Icon(Icons.delete_outline, color: Colors.grey),
          onPressed: () {
            setState(() {
              _items.remove(item);
            });
          },
        ),
      ),
    );
  }
}
```

What's happening here?
- ReorderableListView handles reordering
- Unique keys identify items
- Drag handle for better UX
- Items can be removed

---

# Real-World Examples

> **Common drag & drop** patterns.

```dart
/// 1. Color palette with drag and drop
class ColorPaletteDragDrop extends StatefulWidget {
  const ColorPaletteDragDrop({super.key});

  @override
  State<ColorPaletteDragDrop> createState() => _ColorPaletteDragDropState();
}

class _ColorPaletteDragDropState extends State<ColorPaletteDragDrop> {
  // 1. Color palette
  List<Color> _colors = [
    Colors.red,
    Colors.green,
    Colors.blue,
    Colors.yellow,
    Colors.purple,
    Colors.orange,
  ];
  
  // 2. Selected color
  Color _selectedColor = Colors.blue;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Color Palette'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 3. Drag targets for color selection
            Expanded(
              child: GridView.builder(
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 3,
                  crossAxisSpacing: 8,
                  mainAxisSpacing: 8,
                ),
                itemCount: _colors.length,
                itemBuilder: (context, index) {
                  final color = _colors[index];
                  final isSelected = color == _selectedColor;
                  
                  return DragTarget<Color>(
                    // 4. Accept drop
                    onAccept: (data) {
                      setState(() {
                        // Swap colors
                        _colors[index] = data;
                        // Find where the color came from
                        final sourceIndex = _colors.indexOf(data);
                        _colors[sourceIndex] = color;
                      });
                    },
                    
                    builder: (context, candidateData, rejectedData) {
                      return Draggable<Color>(
                        data: color,
                        feedback: Container(
                          width: 60,
                          height: 60,
                          color: color,
                          child: const Icon(
                            Icons.palette,
                            color: Colors.white,
                          ),
                        ),
                        child: Container(
                          decoration: BoxDecoration(
                            color: color,
                            borderRadius: BorderRadius.circular(8),
                            border: Border.all(
                              color: isSelected 
                                  ? Colors.black 
                                  : Colors.transparent,
                              width: 3,
                            ),
                            boxShadow: [
                              if (isSelected)
                                BoxShadow(
                                  color: color.withOpacity(0.3),
                                  blurRadius: 10,
                                ),
                            ],
                          ),
                          child: const Center(
                            child: Icon(
                              Icons.palette,
                              color: Colors.white,
                            ),
                          ),
                        ),
                      );
                    },
                  );
                },
              ),
            ),
            
            // 5. Status display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: _selectedColor,
                borderRadius: BorderRadius.circular(8),
              ),
              child: Text(
                'Selected Color: ${_selectedColor.toString()}',
                style: const TextStyle(
                  color: Colors.white,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// 2. Shopping cart drag & drop
class ShoppingCartDragDrop extends StatefulWidget {
  const ShoppingCartDragDrop({super.key});

  @override
  State<ShoppingCartDragDrop> createState() => _ShoppingCartDragDropState();
}

class _ShoppingCartDragDropState extends State<ShoppingCartDragDrop> {
  // 6. Product list
  List<String> _products = ['Item 1', 'Item 2', 'Item 3', 'Item 4'];
  
  // 7. Cart
  List<String> _cart = [];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Shopping Cart'),
        actions: [
          // 8. Cart count
          Stack(
            alignment: Alignment.center,
            children: [
              const Icon(Icons.shopping_cart, color: Colors.white),
              if (_cart.isNotEmpty)
                Positioned(
                  top: 0,
                  right: 0,
                  child: Container(
                    padding: const EdgeInsets.all(4),
                    decoration: const BoxDecoration(
                      color: Colors.red,
                      shape: BoxShape.circle,
                    ),
                    child: Text(
                      '${_cart.length}',
                      style: const TextStyle(
                        color: Colors.white,
                        fontSize: 12,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                ),
            ],
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            // 9. Product list
            Expanded(
              child: Column(
                children: [
                  const Text(
                    'Products',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 18,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Expanded(
                    child: Container(
                      decoration: BoxDecoration(
                        color: Colors.grey[100],
                        borderRadius: BorderRadius.circular(8),
                      ),
                      child: ReorderableListView(
                        onReorder: (oldIndex, newIndex) {
                          setState(() {
                            if (newIndex > oldIndex) {
                              newIndex -= 1;
                            }
                            final item = _products.removeAt(oldIndex);
                            _products.insert(newIndex, item);
                          });
                        },
                        children: _products.map((item) {
                          return _buildProductItem(item);
                        }).toList(),
                      ),
                    ),
                  ),
                ],
              ),
            ),
            
            const SizedBox(width: 16),
            
            // 10. Cart
            Expanded(
              child: Column(
                children: [
                  const Text(
                    'Cart',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 18,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Expanded(
                    child: _buildCartDropArea(),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildProductItem(String item) {
    return LongPressDraggable<String>(
      data: item,
      delay: const Duration(milliseconds: 200),
      feedback: Container(
        padding: const EdgeInsets.all(16),
        color: Colors.blue[100],
        child: Text(item),
      ),
      child: Container(
        key: Key(item),
        margin: const EdgeInsets.all(4),
        padding: const EdgeInsets.all(12),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(4),
          border: Border.all(color: Colors.grey[300]!),
        ),
        child: Row(
          children: [
            const Icon(Icons.drag_handle, color: Colors.grey),
            const SizedBox(width: 8),
            Text(item),
          ],
        ),
      ),
    );
  }

  Widget _buildCartDropArea() {
    return DragTarget<String>(
      onWillAccept: (data) => data != null && _products.contains(data),
      onAccept: (data) {
        setState(() {
          _products.remove(data);
          _cart.add(data);
        });
      },
      builder: (context, candidateData, rejectedData) {
        return Container(
          width: double.infinity,
          decoration: BoxDecoration(
            color: candidateData.isNotEmpty ? Colors.blue[100] : Colors.grey[100],
            borderRadius: BorderRadius.circular(8),
            border: Border.all(
              color: candidateData.isNotEmpty ? Colors.blue : Colors.grey,
              width: candidateData.isNotEmpty ? 3 : 1,
            ),
          ),
          child: _cart.isEmpty
              ? const Center(
                  child: Text(
                    'Drag products here',
                    style: TextStyle(color: Colors.grey),
                  ),
                )
              : ListView.builder(
                  itemCount: _cart.length,
                  itemBuilder: (context, index) {
                    final item = _cart[index];
                    return Container(
                      margin: const EdgeInsets.all(4),
                      padding: const EdgeInsets.all(12),
                      decoration: BoxDecoration(
                        color: Colors.green[100],
                        borderRadius: BorderRadius.circular(4),
                        border: Border.all(color: Colors.green[200]!),
                      ),
                      child: Row(
                        children: [
                          const Icon(Icons.shopping_cart, color: Colors.green),
                          const SizedBox(width: 8),
                          Expanded(child: Text(item)),
                          IconButton(
                            icon: const Icon(
                              Icons.close,
                              size: 16,
                            ),
                            onPressed: () {
                              setState(() {
                                _cart.remove(item);
                                _products.add(item);
                              });
                            },
                          ),
                        ],
                      ),
                    );
                  },
                ),
        );
      },
    );
  }
}
```

What's happening here?
- Color palette with drag and drop
- Shopping cart with drag to cart
- Reorderable products
- Visual feedback for drag/drop

---

# Best Practices

## Provide Visual Feedback

```dart
// Good - Clear feedback
Draggable(
  feedback: Container(
    // Custom dragged widget
  ),
  childWhenDragging: Container(
    // Transparency when being dragged
  ),
)
```

## Handle Drag Properly

```dart
// Good - Handle all states
Draggable(
  onDragStarted: () => handleStart(),
  onDragEnd: (details) => handleEnd(),
  data: item,
)
```

## Use Appropriate Gestures

```dart
// Good - Long press for mobile
LongPressDraggable(
  delay: const Duration(milliseconds: 200),
  data: item,
)

// Good - Touch for desktop/tablet
Draggable(
  data: item,
)
```

---

# Common Mistakes

## Not Using Keys

Wrong:
```dart
// No key - performance issues
ReorderableListView(
  children: items.map((item) => Text(item)).toList(),
)
```

Correct:
```dart
// With keys
ReorderableListView(
  children: items.map((item) {
    return Container(
      key: Key(item),
      child: Text(item),
    );
  }).toList(),
)
```

## Not Handling Drop State

Wrong:
```dart
// No visual state change
DragTarget(
  onAccept: (data) => handleData(data),
  builder: (context, candidate, rejected) => Container(),
)
```

Correct:
```dart
// Visual feedback on hover
DragTarget(
  onAccept: (data) => handleData(data),
  builder: (context, candidate, rejected) {
    return Container(
      color: candidate.isNotEmpty ? Colors.blue : Colors.grey,
    );
  },
)
```

---

# Summary

Drag & Drop enables intuitive interactions where users can drag and drop widgets. Use Draggable for simple drag operations, LongPressDraggable for mobile-friendly drag, and DragTarget for drop destinations. ReorderableListView provides built-in reordering functionality.

---

# Next Steps

- [Hit Testing](hit-testing.md)
- [InkWell](inkwell.md)
- [Pointer Events](pointer-events.md)

---

# Did You Know?

- Draggable transfers data
- DragTarget accepts drops
- LongPressDraggable works on mobile
- ReorderableListView reorders items
- feedback shows during drag
- childWhenDragging shows original widget
- onWillAccept validates drops
- Drag & Drop supports gestures