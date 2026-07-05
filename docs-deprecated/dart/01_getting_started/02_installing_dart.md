
---

# Installing Dart

Set up the Dart SDK on your development machine.

---

# What is it?

Dart installation involves downloading and configuring the Dart Software Development Kit (SDK) on your operating system. The SDK includes:

- The `dart` command-line tool
- The `pub` package manager
- The Dart runtime
- Core libraries and tools

---

# Why does it exist?

You need to install Dart to:

- Compile and run Dart programs
- Manage dependencies with pub
- Use the Dart development tools
- Build Flutter applications
- Access the standard library

---

# Installation Methods

## Windows

### Using Chocolatey (Recommended)

```powershell
# Install Chocolatey first if needed
# Then install Dart
choco install dart-sdk
```

---

### Using the installer

1. Download the Dart SDK from [dart.dev/get-dart](https://dart.dev/get-dart)
2. Run the installer
3. Follow the installation wizard
4. Add the Dart SDK bin directory to your PATH

---

### Using winget

```powershell
winget install Dart.Dart
```

---

## macOS

### Using Homebrew (Recommended)

```bash
# Install Homebrew first if needed
brew tap dart-lang/dart
brew install dart
```

---

### Using the installer

1. Download the Dart SDK from [dart.dev/get-dart](https://dart.dev/get-dart)
2. Open the downloaded `.dmg` file
3. Drag the Dart SDK to your Applications folder
4. Add the SDK bin directory to your PATH

---

### Manual installation

```bash
# Download and extract
curl -O https://storage.googleapis.com/dart-archive/channels/stable/release/latest/sdk/dartsdk-macos-x64-release.zip
unzip dartsdk-macos-x64-release.zip
mv dart-sdk /usr/local/dart-sdk

# Add to PATH
echo 'export PATH="$PATH:/usr/local/dart-sdk/bin"' >> ~/.zshrc
source ~/.zshrc
```

---

## Linux

### Using apt (Ubuntu/Debian)

```bash
# Add the official repository
sudo apt-get update
sudo apt-get install apt-transport-https

# Setup repository
sudo sh -c 'wget -qO- https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -'
sudo sh -c 'wget -qO- https://storage.googleapis.com/download.dartlang.org/linux/debian/dart_stable.list > /etc/apt/sources.list.d/dart_stable.list'

# Install Dart
sudo apt-get update
sudo apt-get install dart
```

---

### Using snap

```bash
# Install via snap
sudo snap install dart --classic
```

---

### Using brew (Linux)

```bash
# Install Homebrew first
brew tap dart-lang/dart
brew install dart
```

---

### Manual installation

```bash
# Download and extract
wget https://storage.googleapis.com/dart-archive/channels/stable/release/latest/sdk/dartsdk-linux-x64-release.zip
unzip dartsdk-linux-x64-release.zip
sudo mv dart-sdk /usr/local/dart-sdk

# Add to PATH
echo 'export PATH="$PATH:/usr/local/dart-sdk/bin"' >> ~/.bashrc
source ~/.bashrc
```

---

## VS Code Extension (Optional)

1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X)
3. Search for "Dart"
4. Install the "Dart" extension by Dart Code
5. Install "Flutter" extension if building Flutter apps

---

## IntelliJ IDEA (Optional)

1. Open IntelliJ IDEA
2. Go to Settings > Plugins
3. Search for "Dart"
4. Install the Dart plugin
5. Restart the IDE
6. Configure the SDK path

---

# Verify Installation

## Check the version

```bash
dart --version
```

Expected output:

```text
Dart SDK version: 3.x.x (stable)
```

---

## Check the path

```bash
which dart
```

Expected output:

```text
/usr/local/dart-sdk/bin/dart
# or similar path
```

---

## Run a test program

Create a file called `hello.dart`:

```dart
void main() {
  print('Hello, World!');
}
```

Run it:

```bash
dart hello.dart
```

Expected output:

```text
Hello, World!
```

---

# Common Issues

## 'dart' command not found

**Solution:**

- Ensure the Dart SDK bin directory is in your PATH
- Restart your terminal after installation
- On Windows, check that the installation path is correct

---

## Permission denied on Linux/macOS

**Solution:**

```bash
chmod +x /usr/local/dart-sdk/bin/dart
```

---

## Version mismatch

**Solution:**

- Update to the latest stable version
- Check if multiple versions are installed
- Use `dart --version` to confirm the active version

---

## Installing Flutter (Optional)

If you plan to use Flutter:

```bash
# Follow Flutter installation guide
flutter doctor
```

Dart is included with Flutter, so you don't need to install it separately.

---

# Best Practices

- Use the stable channel for production work
- Consider using version managers like `fvm` for Flutter projects
- Keep Dart updated to the latest stable version
- Use `dart upgrade` to update to the latest version
- Add `dart/bin` to your system PATH

---

# Summary

Installing Dart is straightforward on all major platforms. Choose the method that works best for your development environment. After installation, verify it works by running a simple program. The SDK includes everything you need to start developing Dart applications.

---

# Next Steps

Now that Dart is installed, continue to:

- [Running Dart Programs](running-programs.md)
- [Project Structure](project-structure.md)
- [Language Tour](language-tour.md)

---

# Did You Know?

- The Dart SDK is about 200-400 MB depending on the platform
- Dart can be installed without admin privileges using the zip/tar archives
- The SDK includes a built-in package manager called `pub`
- Dart updates are released approximately every 6 weeks
- You can run Dart code directly from the web using DartPad