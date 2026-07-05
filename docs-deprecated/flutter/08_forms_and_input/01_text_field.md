# TextField

Understand how to create and manage text input fields in Flutter.

---

# What is it?

TextField is a material design widget that allows users to enter text into your Flutter application. It provides a wide range of customization options including styling, validation, keyboard types, input formatting, and focus management. TextField is one of the most commonly used input widgets in Flutter applications.

---

# Why does it exist?

TextField exists to:

- Collect user text input
- Support various input types (text, numbers, emails, etc.)
- Provide visual feedback for user interaction
- Enable form validation and error handling
- Support input formatting and masking
- Manage focus and keyboard behavior
- Create accessible and usable input fields

---

# Basic TextField

> **Creating a simple** text input field.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic TextField example
class BasicTextFieldExample extends StatefulWidget {
  const BasicTextFieldExample({super.key});

  @override
  State<BasicTextFieldExample> createState() => _BasicTextFieldExampleState();
}

class _BasicTextFieldExampleState extends State<BasicTextFieldExample> {
  // Controller to manage the text field's state
  // This gives us access to the text content and allows us to control the field programmatically
  final TextEditingController _controller = TextEditingController();
  
  // Variable to store the current text value
  String _currentText = '';

  @override
  void initState() {
    super.initState();
    
    // Add a listener to the controller to be notified when the text changes
    // This is called whenever the user types or the text is programmatically changed
    _controller.addListener(() {
      // Update the current text when the controller's text changes
      setState(() {
        _currentText = _controller.text;
      });
    });
  }

  @override
  void dispose() {
    // Always dispose controllers when the widget is removed to prevent memory leaks
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Basic TextField'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            // Basic TextField with default styling
            // This is the simplest form of TextField
            TextField(
              // The controller manages the text state
              controller: _controller,
              
              // Called when the user submits the text (presses done/enter)
              // This is useful for form submission or search actions
              onSubmitted: (value) {
                print('Submitted: $value');
              },
              
              // Called when the text changes (every keystroke)
              // This is useful for real-time validation or updates
              onChanged: (value) {
                // This is also called when the text changes
                // The controller listener above handles this as well
              },
            ),
            
            const SizedBox(height: 16),
            
            // Display the current text
            // This shows how to access the text from the controller
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'Current Text:',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 14,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    _currentText.isEmpty ? '(empty)' : _currentText,
                    style: const TextStyle(fontSize: 16),
                  ),
                ],
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
- TextEditingController manages text state
- controller.text accesses current text
- addListener listens for text changes
- dispose prevents memory leaks

---

# TextField with Decoration

> **Styling TextField** with InputDecoration.

```dart
/// TextField with various decorations
class DecoratedTextFieldExample extends StatefulWidget {
  const DecoratedTextFieldExample({super.key});

  @override
  State<DecoratedTextFieldExample> createState() => _DecoratedTextFieldExampleState();
}

class _DecoratedTextFieldExampleState extends State<DecoratedTextFieldExample> {
  // Separate controllers for each text field
  final TextEditingController _basicController = TextEditingController();
  final TextEditingController _outlinedController = TextEditingController();
  final TextEditingController _filledController = TextEditingController();
  final TextEditingController _prefixController = TextEditingController();
  final TextEditingController _suffixController = TextEditingController();

  @override
  void dispose() {
    // Dispose all controllers
    _basicController.dispose();
    _outlinedController.dispose();
    _filledController.dispose();
    _prefixController.dispose();
    _suffixController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Decorated TextField'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Basic decoration with label and hint text
            // The label tells the user what to enter, hint gives an example
            TextField(
              controller: _basicController,
              decoration: const InputDecoration(
                // Label text appears above the field or inside when empty
                labelText: 'Username',
                // Hint text appears when the field is empty
                hintText: 'Enter your username',
                // Helper text provides additional guidance
                helperText: 'Username must be unique',
                // Border style when not focused
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            
            // 2. Outlined border with custom colors
            // This shows how to customize colors for different states
            TextField(
              controller: _outlinedController,
              decoration: InputDecoration(
                labelText: 'Email',
                hintText: 'Enter your email',
                helperText: 'We\'ll never share your email',
                
                // Customize the border colors for different states
                // focusedBorder - when the field is selected
                focusedBorder: const OutlineInputBorder(
                  borderSide: BorderSide(color: Colors.blue, width: 2),
                ),
                // enabledBorder - when the field is active but not focused
                enabledBorder: OutlineInputBorder(
                  borderSide: BorderSide(color: Colors.grey[400]!),
                ),
                // errorBorder - when there is a validation error
                errorBorder: const OutlineInputBorder(
                  borderSide: BorderSide(color: Colors.red),
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            // 3. Filled style with background color
            // Filled fields are common in modern UIs
            TextField(
              controller: _filledController,
              decoration: InputDecoration(
                labelText: 'Search',
                hintText: 'Search for something...',
                // Fill the background with a color
                filled: true,
                fillColor: Colors.grey[100],
                // Prefix icon - shows an icon before the text
                prefixIcon: const Icon(Icons.search),
                // Border radius can be customized
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                  // No border when filled is true
                  borderSide: BorderSide.none,
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            // 4. With prefix and suffix widgets
            // Prefix shows before the text, suffix shows after
            TextField(
              controller: _prefixController,
              decoration: const InputDecoration(
                labelText: 'Price',
                hintText: '0.00',
                prefixIcon: Icon(Icons.attach_money),
                prefixText: '\$',
                suffixText: 'USD',
                border: OutlineInputBorder(),
              ),
              keyboardType: TextInputType.number,
            ),
            const SizedBox(height: 16),
            
            // 5. With suffix and clear button
            // This demonstrates a common pattern for search fields
            TextField(
              controller: _suffixController,
              decoration: InputDecoration(
                labelText: 'Search',
                hintText: 'Search...',
                prefixIcon: const Icon(Icons.search),
                // Suffix icon - shows a clear button when text is not empty
                suffixIcon: IconButton(
                  icon: const Icon(Icons.clear),
                  onPressed: () {
                    // Clear the text when the button is pressed
                    _suffixController.clear();
                  },
                ),
                border: const OutlineInputBorder(),
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
- InputDecoration customizes TextField appearance
- labelText provides context
- hintText shows example input
- prefixIcon and suffixIcon add icons
- filled adds background color

---

# TextField Types

> **Different keyboard** and input types.

```dart
/// TextField with different input types
class TextFieldTypesExample extends StatelessWidget {
  TextFieldTypesExample({super.key});

  // Controllers for each input type
  final TextEditingController _textController = TextEditingController();
  final TextEditingController _numberController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _phoneController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _multilineController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('TextField Types'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Text input (default)
            // Standard text input for general use
            TextField(
              controller: _textController,
              decoration: const InputDecoration(
                labelText: 'Text Input',
                hintText: 'Enter text',
                border: OutlineInputBorder(),
              ),
              // Default keyboard type
              keyboardType: TextInputType.text,
            ),
            const SizedBox(height: 16),
            
            // 2. Number input
            // Shows numeric keyboard, useful for ages, quantities, etc.
            TextField(
              controller: _numberController,
              decoration: const InputDecoration(
                labelText: 'Number Input',
                hintText: 'Enter a number',
                border: OutlineInputBorder(),
              ),
              // Shows number keyboard
              keyboardType: TextInputType.number,
            ),
            const SizedBox(height: 16),
            
            // 3. Email input
            // Shows email keyboard with @ and .com shortcuts
            TextField(
              controller: _emailController,
              decoration: const InputDecoration(
                labelText: 'Email Input',
                hintText: 'Enter your email',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.email),
              ),
              // Shows email keyboard
              keyboardType: TextInputType.emailAddress,
              // Autocorrect is usually disabled for emails
              autocorrect: false,
            ),
            const SizedBox(height: 16),
            
            // 4. Phone input
            // Shows phone number keyboard with number pad
            TextField(
              controller: _phoneController,
              decoration: const InputDecoration(
                labelText: 'Phone Input',
                hintText: 'Enter phone number',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.phone),
              ),
              // Shows phone keyboard
              keyboardType: TextInputType.phone,
            ),
            const SizedBox(height: 16),
            
            // 5. Password input
            // Hides the text with dots for security
            TextField(
              controller: _passwordController,
              decoration: const InputDecoration(
                labelText: 'Password',
                hintText: 'Enter password',
                border: OutlineInputBorder(),
                // Common pattern: show/hide password toggle
                suffixIcon: Icon(Icons.visibility),
              ),
              // Hides the text with dots (secure input)
              obscureText: true,
              // Disable autocorrect for passwords
              autocorrect: false,
              // Don't enable auto-complete for passwords
              enableSuggestions: false,
            ),
            const SizedBox(height: 16),
            
            // 6. Multiline input
            // Expands vertically for longer text (notes, descriptions, etc.)
            TextField(
              controller: _multilineController,
              decoration: const InputDecoration(
                labelText: 'Multiline Input',
                hintText: 'Enter multiple lines of text',
                border: OutlineInputBorder(),
              ),
              // Allows multiple lines
              maxLines: 4,
              // Minimum number of lines
              minLines: 2,
              // Show a counter for maximum characters
              maxLength: 200,
              // This adds a character counter below the field
              // Note: maxLength must be set for the counter to appear
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- keyboardType controls keyboard appearance
- obscureText for password fields
- maxLines and minLines for multiline
- maxLength for character limits

---

# TextField Validation

> **Validating user** input in TextField.

```dart
/// TextField with validation
class ValidationTextFieldExample extends StatefulWidget {
  const ValidationTextFieldExample({super.key});

  @override
  State<ValidationTextFieldExample> createState() => _ValidationTextFieldExampleState();
}

class _ValidationTextFieldExampleState extends State<ValidationTextFieldExample> {
  // Controllers for form fields
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  
  // Form key for validation
  // GlobalKey is used to identify and validate the form
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  // Track if the form has been submitted
  bool _isSubmitted = false;

  @override
  void dispose() {
    // Clean up controllers
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Validation Example'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          // Form widget with key for validation
          // The GlobalKey allows us to validate the form from outside
          key: _formKey,
          child: Column(
            children: [
              // 1. Name field with validation
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Name',
                  hintText: 'Enter your name',
                  border: OutlineInputBorder(),
                  // Prefix icon for visual enhancement
                  prefixIcon: Icon(Icons.person),
                ),
                // Validator function called when form is validated
                // Returns an error message if validation fails, null if successful
                validator: (value) {
                  // Check if the field is empty
                  if (value == null || value.isEmpty) {
                    return 'Please enter your name';
                  }
                  // Check minimum length
                  if (value.length < 2) {
                    return 'Name must be at least 2 characters';
                  }
                  // Check maximum length
                  if (value.length > 50) {
                    return 'Name must be less than 50 characters';
                  }
                  return null; // Valid
                },
                // Called when the user submits the form (presses done/enter)
                onFieldSubmitted: (value) {
                  // This is called when the user presses the done/enter key
                  // Useful for moving to the next field
                },
                // Called when the text changes (every keystroke)
                // Useful for real-time validation
                onChanged: (value) {
                  // You could perform real-time validation here
                  // Example: update UI based on validity
                },
              ),
              const SizedBox(height: 16),
              
              // 2. Email field with validation
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  hintText: 'Enter your email',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.email),
                ),
                // Keyboard type for email
                keyboardType: TextInputType.emailAddress,
                // Auto-correction should be off for emails
                autocorrect: false,
                // Validator for email
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your email';
                  }
                  
                  // Simple email validation using regex
                  // Check if the email contains @ and a domain
                  final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
                  if (!emailRegex.hasMatch(value)) {
                    return 'Please enter a valid email address';
                  }
                  
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 3. Password field with validation
              TextFormField(
                controller: _passwordController,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter your password',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.lock),
                ),
                // Hide password text
                obscureText: true,
                // Disable autocorrect for security
                autocorrect: false,
                // Validator for password
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter a password';
                  }
                  
                  // Check minimum length for security
                  if (value.length < 6) {
                    return 'Password must be at least 6 characters';
                  }
                  
                  // Check for at least one number
                  if (!value.contains(RegExp(r'[0-9]'))) {
                    return 'Password must contain at least one number';
                  }
                  
                  // Check for at least one uppercase letter
                  if (!value.contains(RegExp(r'[A-Z]'))) {
                    return 'Password must contain at least one uppercase letter';
                  }
                  
                  return null; // Valid
                },
              ),
              const SizedBox(height: 24),
              
              // 4. Submit button
              // This button validates the entire form
              ElevatedButton(
                onPressed: () {
                  // Validate the form
                  // This calls the validator on each form field
                  // Returns true if all validators return null
                  if (_formKey.currentState!.validate()) {
                    // Form is valid
                    setState(() {
                      _isSubmitted = true;
                    });
                    
                    // Show success message
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Form submitted successfully!'),
                        backgroundColor: Colors.green,
                      ),
                    );
                    
                    // You can now access the form data
                    // Example: _nameController.text, _emailController.text, etc.
                  } else {
                    // Form is invalid
                    // Show error message
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Please fix the errors above'),
                        backgroundColor: Colors.red,
                      ),
                    );
                  }
                },
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size(double.infinity, 50),
                ),
                child: const Text('Submit'),
              ),
              
              // Show submission status
              if (_isSubmitted) ...[
                const SizedBox(height: 16),
                Container(
                  padding: const EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: Colors.green[50],
                    borderRadius: BorderRadius.circular(8),
                    border: Border.all(color: Colors.green[200]!),
                  ),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        'Submitted Data:',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 16,
                        ),
                      ),
                      const SizedBox(height: 8),
                      Text('Name: ${_nameController.text}'),
                      Text('Email: ${_emailController.text}'),
                      Text('Password: ${'*' * _passwordController.text.length}'),
                    ],
                  ),
                ),
              ],
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Form widget groups fields for validation
- validator function checks input and returns errors
- GlobalKey<FormState> identifies the form
- validate() checks all fields
- SnackBar shows validation results

---

# Real-World Examples

> **Common patterns** with TextField.

```dart
/// 1. Search bar with debouncing
class SearchBarExample extends StatefulWidget {
  const SearchBarExample({super.key});

  @override
  State<SearchBarExample> createState() => _SearchBarExampleState();
}

class _SearchBarExampleState extends State<SearchBarExample> {
  // Controller for the search field
  final TextEditingController _searchController = TextEditingController();
  
  // Timer for debouncing
  // Debouncing prevents too many searches while typing
  Timer? _debounceTimer;
  
  // Current search query
  String _searchQuery = '';
  List<String> _results = [];

  // Sample data to search through
  final List<String> _items = [
    'Apple', 'Banana', 'Orange', 'Grape', 'Watermelon',
    'Strawberry', 'Pineapple', 'Mango', 'Peach', 'Pear',
  ];

  @override
  void initState() {
    super.initState();
    // Add listener for real-time search
    _searchController.addListener(_onSearchChanged);
  }

  @override
  void dispose() {
    // Cancel any pending timer
    _debounceTimer?.cancel();
    _searchController.dispose();
    super.dispose();
  }

  // Called when the search text changes
  void _onSearchChanged() {
    // Cancel any pending debounce timer
    // This ensures we only search after the user stops typing
    _debounceTimer?.cancel();
    
    // Set a new timer for 500ms (debounce delay)
    // The search will only run after 500ms of no typing
    _debounceTimer = Timer(const Duration(milliseconds: 500), () {
      setState(() {
        _searchQuery = _searchController.text;
        _performSearch(_searchQuery);
      });
    });
  }

  // Perform the actual search
  void _performSearch(String query) {
    if (query.isEmpty) {
      _results = [];
      return;
    }

    // Filter items that contain the query (case-insensitive)
    _results = _items
        .where((item) => item.toLowerCase().contains(query.toLowerCase()))
        .toList();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Search Bar'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Search field with clear button
            TextField(
              controller: _searchController,
              decoration: InputDecoration(
                hintText: 'Search...',
                prefixIcon: const Icon(Icons.search),
                // Show clear button when text is not empty
                suffixIcon: _searchController.text.isNotEmpty
                    ? IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          // Clear the search field
                          _searchController.clear();
                        },
                      )
                    : null,
                border: const OutlineInputBorder(
                  borderRadius: BorderRadius.all(Radius.circular(12)),
                ),
                filled: true,
                fillColor: Colors.grey[50],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // Show search results
            if (_searchQuery.isNotEmpty) ...[
              Text(
                'Results for "${_searchQuery}"',
                style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  fontSize: 16,
                ),
              ),
              const SizedBox(height: 8),
            ],
            
            // Display results
            Expanded(
              child: _results.isEmpty && _searchQuery.isNotEmpty
                  ? const Center(
                      child: Text(
                        'No results found',
                        style: TextStyle(color: Colors.grey),
                      ),
                    )
                  : ListView.builder(
                      itemCount: _results.isEmpty ? _items.length : _results.length,
                      itemBuilder: (context, index) {
                        final item = _results.isEmpty 
                            ? _items[index] 
                            : _results[index];
                        return ListTile(
                          leading: const Icon(Icons.search),
                          title: Text(item),
                          // Highlight matching text
                          subtitle: _searchQuery.isNotEmpty
                              ? Text(
                                  'Matches: $_searchQuery',
                                  style: const TextStyle(
                                    fontSize: 12,
                                    color: Colors.grey,
                                  ),
                                )
                              : null,
                          // Show search indicator
                          trailing: _searchQuery.isNotEmpty && 
                                    _results.contains(item)
                              ? const Icon(Icons.check, color: Colors.green)
                              : null,
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

/// 2. Login form with error handling
class LoginFormExample extends StatefulWidget {
  const LoginFormExample({super.key});

  @override
  State<LoginFormExample> createState() => _LoginFormExampleState();
}

class _LoginFormExampleState extends State<LoginFormExample> {
  // Controllers for login form
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  
  // Form key for validation
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Loading state for submit button
  bool _isLoading = false;
  
  // Track if password is visible
  bool _obscurePassword = true;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  // Simulate login API call
  Future<bool> _login(String email, String password) async {
    // Simulate network delay
    await Future.delayed(const Duration(seconds: 2));
    
    // Simple validation for demo
    // In a real app, this would make an API call
    return email == 'test@example.com' && password == 'Password123!';
  }

  // Handle login submission
  Future<void> _submitLogin() async {
    // Validate the form
    if (!_formKey.currentState!.validate()) {
      return; // Form is invalid
    }

    // Show loading state
    setState(() {
      _isLoading = true;
    });

    try {
      // Attempt login
      final success = await _login(
        _emailController.text,
        _passwordController.text,
      );

      if (success) {
        // Login successful
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('Login successful!'),
            backgroundColor: Colors.green,
          ),
        );
        // Navigate to home screen
      } else {
        // Login failed
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('Invalid email or password'),
            backgroundColor: Colors.red,
          ),
        );
      }
    } catch (e) {
      // Handle error
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Error: ${e.toString()}'),
          backgroundColor: Colors.red,
        ),
      );
    } finally {
      // Hide loading state
      if (mounted) {
        setState(() {
          _isLoading = false;
        });
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Login'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Form(
          key: _formKey,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // App logo/icon
              const Icon(
                Icons.lock_outline,
                size: 80,
                color: Colors.blue,
              ),
              const SizedBox(height: 32),
              
              // Email field
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  hintText: 'Enter your email',
                  prefixIcon: Icon(Icons.email),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.all(Radius.circular(12)),
                  ),
                ),
                keyboardType: TextInputType.emailAddress,
                autocorrect: false,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your email';
                  }
                  if (!value.contains('@')) {
                    return 'Please enter a valid email';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // Password field
              TextFormField(
                controller: _passwordController,
                decoration: InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter your password',
                  prefixIcon: const Icon(Icons.lock),
                  // Show/hide password toggle
                  suffixIcon: IconButton(
                    icon: Icon(
                      _obscurePassword 
                          ? Icons.visibility 
                          : Icons.visibility_off,
                    ),
                    onPressed: () {
                      // Toggle password visibility
                      setState(() {
                        _obscurePassword = !_obscurePassword;
                      });
                    },
                  ),
                  border: const OutlineInputBorder(
                    borderRadius: BorderRadius.all(Radius.circular(12)),
                  ),
                ),
                obscureText: _obscurePassword,
                autocorrect: false,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your password';
                  }
                  if (value.length < 6) {
                    return 'Password must be at least 6 characters';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              
              // Login button with loading state
              SizedBox(
                width: double.infinity,
                height: 50,
                child: ElevatedButton(
                  onPressed: _isLoading ? null : _submitLogin,
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.blue,
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(12),
                    ),
                  ),
                  child: _isLoading
                      ? const SizedBox(
                          height: 20,
                          width: 20,
                          child: CircularProgressIndicator(
                            strokeWidth: 2,
                            valueColor: AlwaysStoppedAnimation<Color>(
                              Colors.white,
                            ),
                          ),
                        )
                      : const Text(
                          'Login',
                          style: TextStyle(fontSize: 16),
                        ),
                ),
              ),
              
              const SizedBox(height: 16),
              
              // Forgot password link
              TextButton(
                onPressed: () {
                  // Navigate to forgot password
                },
                child: const Text('Forgot Password?'),
              ),
              
              // Demo credentials hint
              Container(
                margin: const EdgeInsets.only(top: 32),
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.blue[50],
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.blue[100]!),
                ),
                child: Column(
                  children: [
                    const Text(
                      'Demo Credentials:',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 4),
                    Text('Email: test@example.com'),
                    Text('Password: Password123!'),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Search with debouncing to reduce calls
- Real-time filtering results
- Login form with validation
- Loading state on submit
- Show/hide password toggle

---

# Best Practices

## Use Controllers Properly

```dart
// Good - Controllers in State
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final TextEditingController _controller = TextEditingController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}

// Bad - Creating controllers in build
@override
Widget build(BuildContext context) {
  final controller = TextEditingController(); // Created every rebuild
  return TextField(controller: controller);
}
```

## Use Form for Validation

```dart
// Good - Form with validation
Form(
  key: _formKey,
  child: TextFormField(
    validator: (value) {
      if (value == null || value.isEmpty) {
        return 'Required';
      }
      return null;
    },
  ),
)

// Bad - Manual validation
TextField(
  onChanged: (value) {
    if (value.isEmpty) {
      // Manual error handling
    }
  },
)
```

## Use InputDecoration for Styling

```dart
// Good - Consistent styling
InputDecoration(
  labelText: 'Email',
  hintText: 'Enter your email',
  border: OutlineInputBorder(),
  prefixIcon: Icon(Icons.email),
)

// Bad - Inconsistent styling
TextField(
  decoration: InputDecoration(labelText: 'Email'),
)
```

---

# Common Mistakes

## Not Disposing Controllers

Wrong:
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final TextEditingController _controller = TextEditingController();
  // No dispose - memory leak
}
```

Correct:
```dart
class _MyWidgetState extends State<MyWidget> {
  final TextEditingController _controller = TextEditingController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

## Not Using keyboardType

Wrong:
```dart
// Shows text keyboard for numbers
TextField(
  // Missing keyboardType
  decoration: InputDecoration(labelText: 'Age'),
)
```

Correct:
```dart
// Shows number keyboard
TextField(
  keyboardType: TextInputType.number,
  decoration: InputDecoration(labelText: 'Age'),
)
```

---

# Summary

TextField is a versatile widget for collecting user input. Use TextEditingController to manage text state, InputDecoration for styling, and Form for validation. Support different input types with keyboardType and handle user interactions with onChanged and onSubmitted.

---

# Next Steps

- [TextEditingController](texteditingcontroller.md)
- [Form](form.md)
- [Form Validation](form-validation.md)

---

# Did You Know?

- TextEditingController must be disposed
- InputDecoration provides extensive styling
- keyboardType controls keyboard appearance
- Form validates multiple fields
- TextField supports multiline input
- TextField can be password-protected
- InputFormatter can restrict input
- TextField supports focus management