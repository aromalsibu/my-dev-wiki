# StreamBuilder

Understand how to handle asynchronous data streams in Flutter using StreamBuilder.

---

# What is it?

StreamBuilder is a widget that builds itself based on the latest snapshot of interaction with a Stream. It's similar to FutureBuilder but designed for continuous data streams, making it perfect for real-time data like location updates, chat messages, sensor data, or any other streaming data source.

---

# Why does it exist?

StreamBuilder exists to:

- Handle continuous data streams
- Build UI based on stream data
- Support real-time updates
- Manage stream subscriptions
- Handle loading and error states
- Provide reactive UI updates

---

# Basic StreamBuilder

> **Creating** a basic StreamBuilder.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic StreamBuilder example
class BasicStreamBuilderExample extends StatefulWidget {
  const BasicStreamBuilderExample({super.key});

  @override
  State<BasicStreamBuilderExample> createState() => _BasicStreamBuilderExampleState();
}

class _BasicStreamBuilderExampleState extends State<BasicStreamBuilderExample> {
  // 1. Create a StreamController
  // This will be used to add data to the stream
  final StreamController<int> _controller = StreamController<int>();

  @override
  void initState() {
    super.initState();
    // 2. Start a timer to add data to the stream
    _startTimer();
  }

  @override
  void dispose() {
    // 3. Close the stream controller to prevent memory leaks
    _controller.close();
    super.dispose();
  }

  void _startTimer() {
    int counter = 0;
    // Simulate data streaming every second
    Timer.periodic(const Duration(seconds: 1), (timer) {
      counter++;
      // 4. Add data to the stream
      _controller.sink.add(counter);
      
      // Stop after 10 items
      if (counter >= 10) {
        timer.cancel();
        _controller.close();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('StreamBuilder'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 5. StreamBuilder listens to the stream
            StreamBuilder<int>(
              stream: _controller.stream,
              initialData: 0,
              builder: (BuildContext context, AsyncSnapshot<int> snapshot) {
                // 6. Check connection state
                if (snapshot.connectionState == ConnectionState.waiting) {
                  return const CircularProgressIndicator();
                }
                
                // 7. Check for errors
                if (snapshot.hasError) {
                  return Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Icon(Icons.error, color: Colors.red, size: 48),
                      const SizedBox(height: 8),
                      Text('Error: ${snapshot.error}'),
                    ],
                  );
                }
                
                // 8. Display data
                return Column(
                  children: [
                    const Text(
                      'Current Count:',
                      style: TextStyle(fontSize: 16),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      '${snapshot.data ?? 0}',
                      style: const TextStyle(
                        fontSize: 48,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    if (snapshot.connectionState == ConnectionState.done)
                      const Text(
                        'Stream completed!',
                        style: TextStyle(
                          color: Colors.green,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                  ],
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- StreamController creates a stream
- sink.add() sends data to stream
- StreamBuilder listens to stream
- AsyncSnapshot provides current state

---

# StreamBuilder with Real-Time Data

> **Handling** real-time data streams.

```dart
/// StreamBuilder with real-time data
class RealTimeStreamExample extends StatefulWidget {
  const RealTimeStreamExample({super.key});

  @override
  State<RealTimeStreamExample> createState() => _RealTimeStreamExampleState();
}

class _RealTimeStreamExampleState extends State<RealTimeStreamExample> {
  // 1. Create a behavior subject for real-time data
  final StreamController<Map<String, dynamic>> _controller =
      StreamController<Map<String, dynamic>>.broadcast();
  
  // 2. List to store messages
  List<String> _messages = [];

  @override
  void initState() {
    super.initState();
    _startSimulatedUpdates();
  }

  @override
  void dispose() {
    _controller.close();
    super.dispose();
  }

  // 3. Simulate real-time updates
  void _startSimulatedUpdates() {
    final List<String> names = ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'];
    final List<String> actions = ['joined', 'left', 'sent a message', 'reacted', 'typed'];
    
    Timer.periodic(const Duration(seconds: 2), (timer) {
      final randomName = names[DateTime.now().millisecondsSinceEpoch % names.length];
      final randomAction = actions[DateTime.now().millisecondsSinceEpoch % actions.length];
      
      // 4. Add data to stream
      _controller.sink.add({
        'name': randomName,
        'action': randomAction,
        'timestamp': DateTime.now().toLocal(),
      });
      
      // 5. Update local state
      setState(() {
        _messages.insert(0, '$randomName $randomAction');
        if (_messages.length > 20) {
          _messages.removeLast();
        }
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Real-Time Stream'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 6. StreamBuilder for live activity
            StreamBuilder<Map<String, dynamic>>(
              stream: _controller.stream,
              builder: (context, snapshot) {
                if (!snapshot.hasData) {
                  return Card(
                    child: Padding(
                      padding: const EdgeInsets.all(16),
                      child: Row(
                        children: [
                          const Icon(Icons.info_outline, color: Colors.blue),
                          const SizedBox(width: 8),
                          const Text('Waiting for data...'),
                        ],
                      ),
                    ),
                  );
                }

                if (snapshot.hasError) {
                  return Card(
                    child: Padding(
                      padding: const EdgeInsets.all(16),
                      child: Row(
                        children: [
                          const Icon(Icons.error, color: Colors.red),
                          const SizedBox(width: 8),
                          Text('Error: ${snapshot.error}'),
                        ],
                      ),
                    ),
                  );
                }

                final data = snapshot.data!;
                return Card(
                  color: Colors.blue[50],
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Row(
                      children: [
                        const Icon(Icons.person, color: Colors.blue),
                        const SizedBox(width: 8),
                        Expanded(
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Text(
                                '${data['name']} ${data['action']}',
                                style: const TextStyle(
                                  fontWeight: FontWeight.bold,
                                ),
                              ),
                              Text(
                                '${data['timestamp']}',
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
                  ),
                );
              },
            ),
            const SizedBox(height: 8),
            // 7. Message list
            Expanded(
              child: ListView.builder(
                reverse: true,
                itemCount: _messages.length,
                itemBuilder: (context, index) {
                  return ListTile(
                    leading: const Icon(Icons.message),
                    title: Text(
                      _messages[index],
                      style: const TextStyle(fontSize: 14),
                    ),
                    dense: true,
                  );
                },
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
- Broadcast stream for multiple subscribers
- Real-time data simulation
- Activity feed updates
- Message history tracking

---

# StreamBuilder with Data Models

> **Using StreamBuilder** with complex data.

```dart
/// Data model
class ChatMessage {
  final String id;
  final String sender;
  final String text;
  final DateTime timestamp;

  ChatMessage({
    required this.id,
    required this.sender,
    required this.text,
    required this.timestamp,
  });

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'sender': sender,
      'text': text,
      'timestamp': timestamp.millisecondsSinceEpoch,
    };
  }

  factory ChatMessage.fromJson(Map<String, dynamic> json) {
    return ChatMessage(
      id: json['id'],
      sender: json['sender'],
      text: json['text'],
      timestamp: DateTime.fromMillisecondsSinceEpoch(json['timestamp']),
    );
  }
}

/// StreamBuilder with data models
class ChatStreamExample extends StatefulWidget {
  const ChatStreamExample({super.key});

  @override
  State<ChatStreamExample> createState() => _ChatStreamExampleState();
}

class _ChatStreamExampleState extends State<ChatStreamExample> {
  // 1. Stream controller for chat messages
  final StreamController<ChatMessage> _chatController =
      StreamController<ChatMessage>.broadcast();
  
  // 2. Text controller for input
  final TextEditingController _textController = TextEditingController();
  int _messageCount = 0;

  @override
  void dispose() {
    _chatController.close();
    _textController.dispose();
    super.dispose();
  }

  // 3. Send a chat message
  void _sendMessage() {
    final text = _textController.text.trim();
    if (text.isEmpty) return;

    final message = ChatMessage(
      id: '${++_messageCount}',
      sender: 'You',
      text: text,
      timestamp: DateTime.now(),
    );

    _chatController.sink.add(message);
    _textController.clear();

    // Simulate reply
    _simulateReply();
  }

  // 4. Simulate a reply
  void _simulateReply() {
    Future.delayed(const Duration(seconds: 1), () {
      final reply = ChatMessage(
        id: '${++_messageCount}',
        sender: 'Bot',
        text: 'Reply to: ${_textController.text}',
        timestamp: DateTime.now(),
      );
      _chatController.sink.add(reply);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Chat Stream'),
      ),
      body: Column(
        children: [
          // 5. Chat message stream
          Expanded(
            child: StreamBuilder<ChatMessage>(
              stream: _chatController.stream,
              builder: (context, snapshot) {
                // 6. Check if there are messages
                if (snapshot.connectionState == ConnectionState.waiting) {
                  return const Center(child: CircularProgressIndicator());
                }

                if (snapshot.hasError) {
                  return Center(
                    child: Text('Error: ${snapshot.error}'),
                  );
                }

                // 7. Display messages in ListView
                return ListView.builder(
                  reverse: true,
                  itemCount: _messageCount,
                  itemBuilder: (context, index) {
                    // We'll use a separate list to store messages
                    // This is simplified for demonstration
                    return const SizedBox.shrink();
                  },
                );
              },
            ),
          ),
          // 8. Input field
          Container(
            padding: const EdgeInsets.all(8),
            decoration: BoxDecoration(
              color: Colors.white,
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(0.1),
                  blurRadius: 4,
                ),
              ],
            ),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _textController,
                    decoration: const InputDecoration(
                      hintText: 'Type a message...',
                      border: InputBorder.none,
                    ),
                    onSubmitted: (_) => _sendMessage(),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.send, color: Colors.blue),
                  onPressed: _sendMessage,
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Chat message model
- Stream for real-time messages
- Send and receive functionality
- UI updates on each message

---

# Real-World Examples

> **Common patterns** with StreamBuilder.

```dart
/// 1. Countdown timer with StreamBuilder
class CountdownTimer extends StatefulWidget {
  const CountdownTimer({super.key, required this.duration});

  final Duration duration;

  @override
  State<CountdownTimer> createState() => _CountdownTimerState();
}

class _CountdownTimerState extends State<CountdownTimer> {
  late StreamController<int> _controller;
  int _remaining = 0;

  @override
  void initState() {
    super.initState();
    _controller = StreamController<int>.broadcast();
    _remaining = widget.duration.inSeconds;
    _startCountdown();
  }

  @override
  void dispose() {
    _controller.close();
    super.dispose();
  }

  void _startCountdown() {
    Timer.periodic(const Duration(seconds: 1), (timer) {
      if (_remaining <= 0) {
        timer.cancel();
        _controller.close();
        return;
      }
      _remaining--;
      _controller.sink.add(_remaining);
    });
  }

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<int>(
      stream: _controller.stream,
      initialData: widget.duration.inSeconds,
      builder: (context, snapshot) {
        final seconds = snapshot.data ?? 0;
        final minutes = seconds ~/ 60;
        final remainingSeconds = seconds % 60;

        return Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              '${minutes.toString().padLeft(2, '0')}:'
              '${remainingSeconds.toString().padLeft(2, '0')}',
              style: const TextStyle(
                fontSize: 48,
                fontWeight: FontWeight.bold,
                color: Colors.blue,
              ),
            ),
            if (seconds == 0)
              const Text(
                'Time\'s up!',
                style: TextStyle(
                  fontSize: 20,
                  color: Colors.red,
                  fontWeight: FontWeight.bold,
                ),
              ),
          ],
        );
      },
    );
  }
}

/// 2. Data synchronization with StreamBuilder
class DataSyncExample extends StatefulWidget {
  const DataSyncExample({super.key});

  @override
  State<DataSyncExample> createState() => _DataSyncExampleState();
}

class _DataSyncExampleState extends State<DataSyncExample> {
  final StreamController<List<String>> _dataController =
      StreamController<List<String>>.broadcast();

  List<String> _localData = [];

  @override
  void initState() {
    super.initState();
    _simulateDataSync();
  }

  @override
  void dispose() {
    _dataController.close();
    super.dispose();
  }

  void _simulateDataSync() {
    Timer.periodic(const Duration(seconds: 3), (timer) {
      // Simulate data sync
      final newData = List.generate(
        3,
        (i) => 'Item ${DateTime.now().millisecondsSinceEpoch + i}',
      );
      _localData.addAll(newData);
      _dataController.sink.add(_localData);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Data Sync'),
      ),
      body: StreamBuilder<List<String>>(
        stream: _dataController.stream,
        initialData: const [],
        builder: (context, snapshot) {
          final data = snapshot.data ?? [];

          return Column(
            children: [
              // Sync status
              Container(
                padding: const EdgeInsets.all(8),
                color: Colors.blue[50],
                child: Row(
                  children: [
                    const Icon(Icons.sync, color: Colors.blue),
                    const SizedBox(width: 8),
                    Text(
                      snapshot.connectionState == ConnectionState.waiting
                          ? 'Syncing...'
                          : 'Last synced: ${DateTime.now().toLocal()}',
                      style: const TextStyle(fontSize: 14),
                    ),
                  ],
                ),
              ),
              // Data list
              Expanded(
                child: ListView.builder(
                  itemCount: data.length,
                  itemBuilder: (context, index) {
                    return ListTile(
                      leading: const Icon(Icons.folder),
                      title: Text(data[index]),
                    );
                  },
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

What's happening here?
- Countdown timer with StreamBuilder
- Data synchronization stream
- Real-time updates and status
- Stream-based data management

---

# Best Practices

## Use Broadcast Streams for Multiple Listeners

```dart
// Good - Broadcast stream for multiple listeners
final StreamController<int> _controller = StreamController<int>.broadcast();

// Bad - Single subscription stream
final StreamController<int> _controller = StreamController<int>();
```

## Cancel Subscriptions

```dart
// Good - Cancel subscriptions
@override
void dispose() {
  _controller.close();
  super.dispose();
}
```

## Use Initial Data

```dart
// Good - Provide initial data
StreamBuilder<int>(
  stream: _stream,
  initialData: 0,
  builder: (context, snapshot) => ...
)
```

---

# Common Mistakes

## Not Closing StreamController

Wrong:
```dart
// Memory leak
@override
void dispose() {
  super.dispose();
}
```

Correct:
```dart
// Close controller
@override
void dispose() {
  _controller.close();
  super.dispose();
}
```

## Using Single Subscription Stream for Multiple Listeners

Wrong:
```dart
// Error - can't listen twice
final controller = StreamController<int>();
// Multiple listeners will cause error
```

Correct:
```dart
// Use broadcast for multiple listeners
final controller = StreamController<int>.broadcast();
```

---

# Summary

StreamBuilder handles continuous data streams in Flutter. Use StreamController to create streams, handle connection states, and manage errors. StreamBuilder is essential for real-time data, chat applications, and any streaming data scenarios.

---

# Next Steps

- [ValueListenableBuilder](valuelistenablebuilder.md)
- [AnimatedBuilder](animatedbuilder.md)
- [Async Patterns](async-patterns.md)

---

# Did You Know?

- StreamBuilder handles continuous data
- StreamController creates streams
- broadcast() allows multiple listeners
- sink.add() sends data to stream
- ConnectionState tracks stream status
- close() prevents memory leaks
- initialData provides default value
- StreamBuilder is reactive by nature