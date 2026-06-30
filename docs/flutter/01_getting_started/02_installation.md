# Installation

Set up Flutter development environment on your machine.

---

# What is it?

Flutter installation involves downloading the Flutter SDK, setting up environment variables, installing platform-specific dependencies (Android Studio, Xcode, etc.), and verifying everything works correctly.

---

# Why does it exist?

Installation exists to:

- Provide the Flutter framework and tools on your machine
- Set up platform-specific build tools for Android and iOS
- Configure your development environment for Flutter development
- Enable running Flutter commands from terminal
- Prepare your system for creating and building Flutter apps

---

# System Requirements

## Minimum Requirements

**For Windows:**

- Windows 10 or later (64-bit)
- 8 GB RAM (recommended)
- 4 GB available disk space
- PowerShell 5.0 or later

**For macOS:**

- macOS 10.14 (Mojave) or later
- 8 GB RAM (recommended)
- 4 GB available disk space
- Xcode 12 or later (for iOS)

**For Linux:**

- Ubuntu 18.04 LTS or later (64-bit)
- 8 GB RAM (recommended)
- 4 GB available disk space
- GCC, G++, and other build tools

---

# Installation Steps

## Windows

> **Download and install Flutter SDK on Windows**

```powershell
# 1. Download Flutter SDK
# Go to: https://docs.flutter.dev/get-started/install/windows
# Download the latest stable release zip file

# 2. Extract to desired location (e.g., C:\flutter)
# 3. Update PATH environment variable
# Add C:\flutter\bin to your PATH

# 4. Run Flutter doctor
flutter doctor

# 5. Install Android Studio for Android development
# Download from: https://developer.android.com/studio

# 6. Install Visual Studio for desktop development
# Download from: https://visualstudio.microsoft.com/
```

What's happening here?
- `flutter doctor` checks your setup and reports issues
- PATH allows running Flutter commands from anywhere
- Android Studio provides Android SDK and emulator
- Visual Studio provides C++ build tools for Windows apps

---

## macOS

> **Install Flutter SDK on macOS**

```bash
# 1. Install via Homebrew (Recommended)
brew install flutter

# OR manual installation
# 1. Download Flutter SDK
# Go to: https://docs.flutter.dev/get-started/install/macos
# Download the latest stable release zip file

# 2. Extract to desired location
cd ~/development
unzip ~/Downloads/flutter_macos_*.zip

# 3. Add to PATH (add to ~/.zshrc or ~/.bashrc)
export PATH="$PATH:$HOME/development/flutter/bin"

# 4. Install Xcode (for iOS development)
# Download from Mac App Store

# 5. Install CocoaPods
sudo gem install cocoapods

# 6. Run Flutter doctor
flutter doctor
```

What's happening here?
- Homebrew simplifies Flutter installation
- Xcode includes iOS SDK and simulator
- CocoaPods handles iOS dependencies
- PATH updated permanently via shell config

---

## Linux

> **Install Flutter SDK on Linux**

```bash
# 1. Install dependencies
sudo apt-get update
sudo apt-get install -y curl git unzip xz-utils zip libglu1-mesa

# 2. Download Flutter SDK
cd ~/development
wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_*.tar.xz

# 3. Extract the SDK
tar xf flutter_linux_*.tar.xz

# 4. Add to PATH (add to ~/.bashrc)
export PATH="$PATH:$HOME/development/flutter/bin"

# 5. Run Flutter doctor
flutter doctor

# 6. Install Android Studio (optional but recommended)
# Download from: https://developer.android.com/studio
```

What's happening here?
- System dependencies installed via apt
- Flutter SDK downloaded and extracted
- PATH updated for command access
- Android Studio recommended for UI development

---

# Flutter Doctor

> **The `flutter doctor` command** checks your environment and shows issues that need fixing.

```bash
flutter doctor

# Output example:
# Doctor summary (to see all details, run flutter doctor -v)
# [✓] Flutter (Channel stable, 3.16.0, on macOS 14.0)
# [✓] Android toolchain - develop for Android devices
# [✓] Xcode - develop for iOS and macOS
# [✓] Chrome - develop for the web
# [✓] Android Studio (version 2023.1)
# [✓] Connected device (2 available)
# 
# ! Doctor found issues in 1 category.
# Run 'flutter doctor -v' for more details.

# Common fixes from flutter doctor:
flutter doctor --android-licenses  # Accept Android licenses
flutter doctor --verbose           # Get detailed output
```

What's happening here?
- `flutter doctor` checks all components
- ✓ indicates installed properly
- ✗ or ! indicates missing or requires action
- Follow the suggestions to fix issues

---

# IDEs and Editors

## Android Studio

> **The most popular IDE for Flutter development**

```bash
# Install Flutter and Dart plugins
# 1. Open Android Studio
# 2. Go to Preferences > Plugins
# 3. Search for "Flutter" and click Install
# 4. The Dart plugin will be installed automatically
# 5. Restart Android Studio

# Create new Flutter project:
# File > New > New Flutter Project
# Choose Flutter Application
# Configure project settings
# Click Finish
```

Benefits:
- Flutter project templates
- Hot reload and hot restart buttons
- Advanced debugging and profiling
- Auto-completion and suggestions
- UI inspector for widgets
- Built-in emulator support

---

## VS Code

> **Lightweight editor with excellent Flutter support**

```bash
# 1. Install Flutter and Dart extensions
# Open VS Code
# Click Extensions icon (Ctrl+Shift+X)
# Search for "Flutter"
# Click Install on "Flutter" extension
# Click Install on "Dart" extension (if not auto-installed)

# Create new Flutter project:
# View > Command Palette (Ctrl+Shift+P)
# Type "Flutter: New Project"
# Enter project name
# Choose project location
# Wait for project creation
```

Benefits:
- Lightweight and fast
- Excellent code completion
- Hot reload with keyboard shortcuts
- DevTools integration
- Built-in terminal
- Git integration

---

# Updating Flutter

> **Keep Flutter up to date** for the latest features and fixes.

```bash
# Check for updates
flutter upgrade

# Switch to specific channel
flutter channel stable      # Recommended
flutter channel beta
flutter channel dev
flutter channel master

# Check current version
flutter --version

# Get specific version
flutter downgrade 3.16.0
```

What's happening here?
- `flutter upgrade` fetches latest stable version
- Channels offer different stability levels
- Stable is recommended for production
- `flutter --version` shows current version

---

# Common Installation Issues

## PATH Not Set Correctly

Wrong:
```bash
# Error: 'flutter' is not recognized
```

Correct:
```bash
# Windows
setx PATH "%PATH%;C:\flutter\bin"

# macOS/Linux
export PATH="$PATH:$HOME/development/flutter/bin"

# Verify
flutter --version
```

---

## Android Licenses Not Accepted

Wrong:
```bash
# Error: Android licenses not accepted
```

Correct:
```bash
flutter doctor --android-licenses
# Follow prompts and accept all licenses
```

---

## Xcode Command Line Tools Missing

Wrong:
```bash
# Error: Xcode not installed
```

Correct:
```bash
# Install Xcode from App Store
# Then install command line tools
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch
```

---

# Best Practices

- Use stable channel for production apps
- Update Flutter regularly for security patches
- Keep Android Studio and Xcode updated
- Use flutter doctor after installation
- Set up environment variables permanently
- Install both Android Studio and VS Code
- Use a package manager when possible

---

# Summary

Flutter installation requires downloading the SDK, setting up PATH, installing platform-specific tools (Android Studio, Xcode), and running flutter doctor to verify everything. The process varies by platform but is well-documented and straightforward.

---

# Next Steps

- [Project Structure](project-structure.md)
- [Running Your First App](first-app.md)
- [Hot Reload vs Hot Restart](hot-reload.md)

---

# Did You Know?

- Flutter SDK is about 1 GB in size
- Flutter can build iOS apps on Windows (via CI/CD)
- You don't need Android Studio - you can use any editor
- Flutter includes a built-in HTTP server for debugging
- Flutter doctor also checks emulator availability
- You can install multiple Flutter versions using FVM