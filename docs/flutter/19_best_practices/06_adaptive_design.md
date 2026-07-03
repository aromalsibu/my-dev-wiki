# Adaptive Design

Understand how to create layouts that adapt to different platforms and input methods in Flutter.

---

# What is it?

Adaptive Design is the practice of building user interfaces that adapt to different platforms (Android, iOS, Web, Desktop) and input methods (touch, mouse, keyboard). Unlike responsive design which focuses on screen sizes, adaptive design focuses on platform conventions, interaction patterns, and user expectations.

---

# Why does it exist?

Adaptive Design exists to:

- Support multiple platforms
- Follow platform conventions
- Adapt to input methods
- Provide native experience
- Support different interactions
- Reach wider audience
- Follow platform guidelines

---

# Platform Detection

> **Detecting** platforms and adapting accordingly.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'dart:io' show Platform;

/// 1. Platform detection
class PlatformDetectionExample extends StatelessWidget {
  const PlatformDetectionExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Platform Detection'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Platform-specific widgets
            _buildPlatformSpecificWidget(),
            
            const SizedBox(height: 16),
            
            // 2. Platform-specific layout
            _buildPlatformSpecificLayout(),
            
            const SizedBox(height: 16),
            
            // 3. Platform info
            _buildPlatformInfo(),
          ],
        ),
      ),
    );
  }

  Widget _buildPlatformSpecificWidget() {
    // Use Platform enum for different platforms
    if (Theme.of(context).platform == TargetPlatform.iOS) {
      // iOS-specific widget
      return CupertinoButton(
        onPressed: () {},
        color: Colors.blue,
        child: const Text('iOS Button'),
      );
    } else {
      // Android/Default widget
      return ElevatedButton(
        onPressed: () {},
        child: const Text('Android Button'),
      );
    }
  }

  Widget _buildPlatformSpecificLayout() {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;
    
    if (isIOS) {
      return _buildIOSLayout();
    } else {
      return _buildAndroidLayout();
    }
  }

  Widget _buildIOSLayout() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        children: [
          const Text(
            'iOS Style Layout',
            style: TextStyle(
              fontWeight: FontWeight.bold,
              fontSize: 18,
            ),
          ),
          const SizedBox(height: 8),
          CupertinoTextField(
            placeholder: 'iOS Text Field',
            padding: const EdgeInsets.all(12),
          ),
          const SizedBox(height: 8),
          CupertinoButton(
            onPressed: () {},
            color: Colors.blue,
            child: const Text('iOS Button'),
          ),
        ],
      ),
    );
  }

  Widget _buildAndroidLayout() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(4),
      ),
      child: Column(
        children: [
          const Text(
            'Android Style Layout',
            style: TextStyle(
              fontWeight: FontWeight.bold,
              fontSize: 18,
            ),
          ),
          const SizedBox(height: 8),
          const TextField(
            decoration: InputDecoration(
              labelText: 'Android Text Field',
              border: OutlineInputBorder(),
            ),
          ),
          const SizedBox(height: 8),
          ElevatedButton(
            onPressed: () {},
            child: const Text('Android Button'),
          ),
        ],
      ),
    );
  }

  Widget _buildPlatformInfo() {
    final platform = Theme.of(context).platform;
    final isIOS = platform == TargetPlatform.iOS;
    final isAndroid = platform == TargetPlatform.android;
    final isMacOS = platform == TargetPlatform.macOS;
    final isWindows = platform == TargetPlatform.windows;

    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.blue[50],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        children: [
          const Text(
            'Platform Information',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          Text('Platform: $platform'),
          Text('Is iOS: $isIOS'),
          Text('Is Android: $isAndroid'),
          Text('Is macOS: $isMacOS'),
          Text('Is Windows: $isWindows'),
        ],
      ),
    );
  }
}
```

What's happening here?
- Theme.of(context).platform detects platform
- Different widgets for iOS and Android
- Platform-specific layouts
- Platform info display

---

# Platform-Specific Widgets

> **Using** platform-specific widgets.

```dart
/// 2. Platform-specific widgets
class PlatformSpecificWidgetsExample extends StatelessWidget {
  const PlatformSpecificWidgetsExample({super.key});

  @override
  Widget build(BuildContext context) {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;

    return Scaffold(
      appBar: isIOS
          ? CupertinoNavigationBar(
              middle: const Text('Adaptive Widgets'),
              trailing: CupertinoButton(
                onPressed: () {},
                child: const Icon(CupertinoIcons.settings),
              ),
            )
          : AppBar(
              title: const Text('Adaptive Widgets'),
              actions: [
                IconButton(
                  icon: const Icon(Icons.settings),
                  onPressed: () {},
                ),
              ],
            ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Adaptive button
            _buildAdaptiveButton(),
            const SizedBox(height: 16),
            
            // 2. Adaptive text field
            _buildAdaptiveTextField(),
            const SizedBox(height: 16),
            
            // 3. Adaptive switch
            _buildAdaptiveSwitch(),
            const SizedBox(height: 16),
            
            // 4. Adaptive progress indicator
            _buildAdaptiveProgress(),
          ],
        ),
      ),
    );
  }

  Widget _buildAdaptiveButton() {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;
    
    return isIOS
        ? CupertinoButton(
            onPressed: () {},
            color: Colors.blue,
            child: const Text('iOS Button'),
          )
        : ElevatedButton(
            onPressed: () {},
            child: const Text('Android Button'),
          );
  }

  Widget _buildAdaptiveTextField() {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;
    
    return isIOS
        ? CupertinoTextField(
            placeholder: 'iOS Text Field',
            padding: const EdgeInsets.all(12),
            decoration: BoxDecoration(
              color: Colors.grey[100],
              borderRadius: BorderRadius.circular(8),
            ),
          )
        : const TextField(
            decoration: InputDecoration(
              labelText: 'Android Text Field',
              border: OutlineInputBorder(),
            ),
          );
  }

  Widget _buildAdaptiveSwitch() {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;
    
    return Row(
      children: [
        const Text('Adaptive Switch: '),
        isIOS
            ? CupertinoSwitch(
                value: true,
                onChanged: (value) {},
              )
            : Switch(
                value: true,
                onChanged: (value) {},
              ),
      ],
    );
  }

  Widget _buildAdaptiveProgress() {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;
    
    return Row(
      children: [
        const Text('Adaptive Progress: '),
        isIOS
            ? const CupertinoActivityIndicator()
            : const SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(),
              ),
      ],
    );
  }
}
```

What's happening here?
- Adaptive button for iOS/Android
- Adaptive text field
- Adaptive switch
- Adaptive progress indicator

---

# Adaptive Navigation

> **Adapting** navigation for different platforms.

```dart
/// 3. Adaptive navigation
class AdaptiveNavigationExample extends StatelessWidget {
  const AdaptiveNavigationExample({super.key});

  @override
  Widget build(BuildContext context) {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;

    return Scaffold(
      appBar: isIOS
          ? CupertinoNavigationBar(
              middle: const Text('Adaptive Navigation'),
            )
          : AppBar(
              title: const Text('Adaptive Navigation'),
            ),
      body: isIOS
          ? _buildIOSNavigation()
          : _buildAndroidNavigation(),
    );
  }

  Widget _buildIOSNavigation() {
    return ListView(
      children: [
        CupertinoListTile(
          title: const Text('Home'),
          trailing: const Icon(CupertinoIcons.chevron_right),
          onTap: () {},
        ),
        CupertinoListTile(
          title: const Text('Profile'),
          trailing: const Icon(CupertinoIcons.chevron_right),
          onTap: () {},
        ),
        CupertinoListTile(
          title: const Text('Settings'),
          trailing: const Icon(CupertinoIcons.chevron_right),
          onTap: () {},
        ),
      ],
    );
  }

  Widget _buildAndroidNavigation() {
    return ListView(
      children: [
        ListTile(
          title: const Text('Home'),
          trailing: const Icon(Icons.arrow_forward_ios),
          onTap: () {},
        ),
        ListTile(
          title: const Text('Profile'),
          trailing: const Icon(Icons.arrow_forward_ios),
          onTap: () {},
        ),
        ListTile(
          title: const Text('Settings'),
          trailing: const Icon(Icons.arrow_forward_ios),
          onTap: () {},
        ),
      ],
    );
  }
}
```

What's happening here?
- iOS-style Cupertino navigation
- Android-style Material navigation
- Platform-appropriate list tiles
- Platform-appropriate icons

---

# Real-World Examples

> **Common patterns** for adaptive design.

```dart
/// 4. Adaptive input detection
class AdaptiveInputDetection extends StatelessWidget {
  const AdaptiveInputDetection({super.key});

  @override
  Widget build(BuildContext context) {
    // Detect input method
    final isTouchDevice = MediaQuery.of(context).size.width < 800;
    final isDesktop = !isTouchDevice;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Adaptive Input'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Show appropriate input method
            Text(
              'Input Method: ${isTouchDevice ? 'Touch' : 'Mouse/Keyboard'}',
              style: const TextStyle(fontSize: 16),
            ),
            const SizedBox(height: 16),

            // 2. Adaptive controls
            if (isTouchDevice)
              _buildTouchControls()
            else
              _buildDesktopControls(),
          ],
        ),
      ),
    );
  }

  Widget _buildTouchControls() {
    return Column(
      children: [
        const Text(
          'Touch Controls',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 8),
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            _buildTouchButton('⬅️', 'Swipe'),
            const SizedBox(width: 16),
            _buildTouchButton('➡️', 'Swipe'),
          ],
        ),
      ],
    );
  }

  Widget _buildTouchButton(String icon, String label) {
    return Container(
      width: 80,
      height: 80,
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text(
            icon,
            style: const TextStyle(fontSize: 32),
          ),
          Text(
            label,
            style: const TextStyle(
              color: Colors.white,
              fontSize: 12,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildDesktopControls() {
    return Column(
      children: [
        const Text(
          'Desktop Controls',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 8),
        const Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Use keyboard shortcuts:'),
            SizedBox(width: 16),
            Text('⌘S', style: TextStyle(fontWeight: FontWeight.bold)),
            Text(' Save'),
            SizedBox(width: 16),
            Text('⌘Z', style: TextStyle(fontWeight: FontWeight.bold)),
            Text(' Undo'),
          ],
        ),
      ],
    );
  }
}

/// 5. Adaptive theme
class AdaptiveThemeExample extends StatelessWidget {
  const AdaptiveThemeExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Adaptive Theme'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            _buildPlatformThemeInfo(),
            const SizedBox(height: 16),
            _buildAdaptiveCard(),
          ],
        ),
      ),
    );
  }

  Widget _buildPlatformThemeInfo() {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;
    
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: isIOS ? Colors.blue[50] : Colors.green[50],
        borderRadius: BorderRadius.circular(8),
        border: Border.all(
          color: isIOS ? Colors.blue : Colors.green,
        ),
      ),
      child: Column(
        children: [
          Text(
            isIOS ? 'iOS Theme' : 'Material Theme',
            style: TextStyle(
              fontWeight: FontWeight.bold,
              color: isIOS ? Colors.blue : Colors.green,
            ),
          ),
          const SizedBox(height: 8),
          Text(
            isIOS
                ? 'Cupertino design with iOS conventions'
                : 'Material design with Android conventions',
            textAlign: TextAlign.center,
          ),
        ],
      ),
    );
  }

  Widget _buildAdaptiveCard() {
    final isIOS = Theme.of(context).platform == TargetPlatform.iOS;
    
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: isIOS ? Colors.grey[100] : Colors.white,
        borderRadius: BorderRadius.circular(isIOS ? 16 : 8),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(isIOS ? 0.1 : 0.2),
            blurRadius: isIOS ? 20 : 8,
            offset: Offset(0, isIOS ? 0 : 2),
          ),
        ],
      ),
      child: isIOS
          ? Column(
              children: [
                const Text(
                  'iOS Card',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 8),
                const Text(
                  'This card follows iOS design conventions.',
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 8),
                CupertinoButton(
                  onPressed: () {},
                  color: Colors.blue,
                  child: const Text('iOS Action'),
                ),
              ],
            )
          : Column(
              children: [
                const Text(
                  'Android Card',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 8),
                const Text(
                  'This card follows Android design conventions.',
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 8),
                ElevatedButton(
                  onPressed: () {},
                  child: const Text('Android Action'),
                ),
              ],
            ),
    );
  }
}
```

What's happening here?
- Input method detection
- Touch vs desktop controls
- Platform-appropriate theming
- Adaptive card styles

---

# Best Practices

## Use Platform Detection

```dart
// Good - Platform detection
final isIOS = Theme.of(context).platform == TargetPlatform.iOS;

// Bad - Hardcoding platform
// Assuming Android only
```

## Use Platform-Specific Widgets

```dart
// Good - Platform-specific widgets
if (isIOS) {
  return CupertinoButton(...);
} else {
  return ElevatedButton(...);
}

// Bad - Using Material on iOS
// Using Material widgets everywhere
```

## Follow Platform Conventions

```dart
// Good - Platform conventions
// iOS: Cupertino widgets
// Android: Material widgets
// Desktop: Material with keyboard support

// Bad - Ignoring conventions
// Using iOS widgets on Android
// Using Android widgets on iOS
```

---

# Common Mistakes

## Not Adapting to Platform

Wrong:
```dart
// Only Material design
return ElevatedButton(
  onPressed: () {},
  child: const Text('Button'),
);
```

Correct:
```dart
// Platform-adaptive
final isIOS = Theme.of(context).platform == TargetPlatform.iOS;
return isIOS
    ? CupertinoButton(
        onPressed: () {},
        child: const Text('Button'),
      )
    : ElevatedButton(
        onPressed: () {},
        child: const Text('Button'),
      );
```

## Ignoring Input Methods

Wrong:
```dart
// Only touch input
// No keyboard support
// No hover states
```

Correct:
```dart
// Multiple input methods
// Keyboard navigation
// Hover states
```

---

# Summary

Adaptive Design ensures your app works well on different platforms and input methods. Use platform detection, platform-specific widgets, and follow platform conventions. Adapt to both touch and desktop input for the best user experience.

---

# Next Steps

- [Architecture Patterns](architecture-patterns.md)
- [Responsive Design](responsive-design.md)
- [Project Structure](project-structure.md)

---

# Did You Know?

- Theme.of(context).platform detects platform
- Cupertino widgets for iOS
- Material widgets for Android
- Adaptive design follows platform conventions
- Input methods vary by platform
- Desktop apps need keyboard support
- Platform detection enables adaptation
- Adaptive design improves user experience