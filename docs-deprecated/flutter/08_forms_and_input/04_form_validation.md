# Form Validation

Understand how to validate user input in Flutter forms.

---

# What is it?

Form validation is the process of checking user input against defined rules and requirements before accepting or submitting it. It ensures data quality, prevents invalid data from being processed, and provides immediate feedback to users. Flutter provides built-in validation through TextFormField's validator property and Form's validate method.

---

# Why does it exist?

Form validation exists to:

- Ensure data quality and integrity
- Prevent invalid data submission
- Provide immediate user feedback
- Reduce server-side validation
- Guide users to enter correct data
- Improve user experience
- Enforce business rules

---

# Basic Validation

> **Simple validation** with TextFormField.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic validation example
class BasicValidationExample extends StatefulWidget {
  const BasicValidationExample({super.key});

  @override
  State<BasicValidationExample> createState() => _BasicValidationExampleState();
}

class _BasicValidationExampleState extends State<BasicValidationExample> {
  // Form key for validation
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Controllers for form fields
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _ageController = TextEditingController();

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _ageController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Basic Validation'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // 1. Name field with required validation
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Name',
                  hintText: 'Enter your name',
                  border: OutlineInputBorder(),
                ),
                // Validator function - returns error message or null
                validator: (value) {
                  // Check if the field is empty
                  if (value == null || value.isEmpty) {
                    // Return error message - this will be displayed below the field
                    return 'Please enter your name';
                  }
                  // Return null if valid
                  return null;
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
                ),
                keyboardType: TextInputType.emailAddress,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your email';
                  }
                  // Check if email contains @
                  if (!value.contains('@')) {
                    return 'Please enter a valid email address';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 3. Age field with custom validation
              TextFormField(
                controller: _ageController,
                decoration: const InputDecoration(
                  labelText: 'Age',
                  hintText: 'Enter your age',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.number,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your age';
                  }
                  // Parse the input to int
                  final age = int.tryParse(value);
                  if (age == null) {
                    return 'Please enter a valid number';
                  }
                  if (age < 0 || age > 120) {
                    return 'Please enter a valid age (0-120)';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              
              // Submit button
              ElevatedButton(
                onPressed: () {
                  // Validate all fields at once
                  if (_formKey.currentState!.validate()) {
                    // All fields are valid
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Form is valid!'),
                        backgroundColor: Colors.green,
                      ),
                    );
                  } else {
                    // Some fields are invalid
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
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- validator function returns error message or null
- validate() checks all fields
- Error messages appear below fields
- Form submission only when valid

---

# Common Validation Patterns

> **Common validation** rules for form fields.

```dart
/// Common validation patterns
class ValidationPatternsExample extends StatefulWidget {
  const ValidationPatternsExample({super.key});

  @override
  State<ValidationPatternsExample> createState() => _ValidationPatternsExampleState();
}

class _ValidationPatternsExampleState extends State<ValidationPatternsExample> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Controllers
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _confirmPasswordController = TextEditingController();
  final TextEditingController _phoneController = TextEditingController();
  final TextEditingController _urlController = TextEditingController();

  @override
  void dispose() {
    _usernameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    _phoneController.dispose();
    _urlController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Validation Patterns'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // 1. Username with alphanumeric validation
              TextFormField(
                controller: _usernameController,
                decoration: const InputDecoration(
                  labelText: 'Username',
                  hintText: 'Enter username (letters, numbers, underscore)',
                  border: OutlineInputBorder(),
                  helperText: '3-20 characters, letters, numbers, underscore only',
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Username is required';
                  }
                  // Check length
                  if (value.length < 3) {
                    return 'Username must be at least 3 characters';
                  }
                  if (value.length > 20) {
                    return 'Username must be less than 20 characters';
                  }
                  // Check for valid characters
                  if (!RegExp(r'^[a-zA-Z0-9_]+$').hasMatch(value)) {
                    return 'Only letters, numbers, and underscores allowed';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 2. Email with regex validation
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  hintText: 'Enter your email',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.emailAddress,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Email is required';
                  }
                  // Email regex pattern
                  final emailRegex = RegExp(
                    r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
                  );
                  if (!emailRegex.hasMatch(value)) {
                    return 'Enter a valid email address';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 3. Password with strength validation
              TextFormField(
                controller: _passwordController,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter strong password',
                  border: OutlineInputBorder(),
                  helperText: '8+ chars, uppercase, lowercase, number, special',
                ),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Password is required';
                  }
                  if (value.length < 8) {
                    return 'Password must be at least 8 characters';
                  }
                  // Check for uppercase
                  if (!RegExp(r'[A-Z]').hasMatch(value)) {
                    return 'Password must contain at least one uppercase letter';
                  }
                  // Check for lowercase
                  if (!RegExp(r'[a-z]').hasMatch(value)) {
                    return 'Password must contain at least one lowercase letter';
                  }
                  // Check for number
                  if (!RegExp(r'[0-9]').hasMatch(value)) {
                    return 'Password must contain at least one number';
                  }
                  // Check for special character
                  if (!RegExp(r'[!@#$%^&*(),.?":{}|<>]').hasMatch(value)) {
                    return 'Password must contain at least one special character';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 4. Confirm password (cross-field validation)
              TextFormField(
                controller: _confirmPasswordController,
                decoration: const InputDecoration(
                  labelText: 'Confirm Password',
                  hintText: 'Re-enter your password',
                  border: OutlineInputBorder(),
                ),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please confirm your password';
                  }
                  // Compare with password field
                  if (value != _passwordController.text) {
                    return 'Passwords do not match';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 5. Phone number validation
              TextFormField(
                controller: _phoneController,
                decoration: const InputDecoration(
                  labelText: 'Phone Number',
                  hintText: 'Enter phone number',
                  border: OutlineInputBorder(),
                  helperText: 'Format: +1 234 567 8900',
                ),
                keyboardType: TextInputType.phone,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Phone number is required';
                  }
                  // Remove spaces and special characters
                  final cleaned = value.replaceAll(RegExp(r'[\s\-()]'), '');
                  if (cleaned.length < 10) {
                    return 'Enter a valid phone number (min 10 digits)';
                  }
                  if (!RegExp(r'^\+?[0-9]+$').hasMatch(cleaned)) {
                    return 'Enter a valid phone number';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 6. URL validation
              TextFormField(
                controller: _urlController,
                decoration: const InputDecoration(
                  labelText: 'Website URL',
                  hintText: 'Enter your website URL',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.url,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return null; // Optional field
                  }
                  // URL validation
                  final urlRegex = RegExp(
                    r'^(https?://)?([\da-z.-]+)\.([a-z.]{2,6})([/\w .-]*)*/?$',
                  );
                  if (!urlRegex.hasMatch(value)) {
                    return 'Enter a valid URL';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              
              // Submit button
              ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('All validations passed!'),
                        backgroundColor: Colors.green,
                      ),
                    );
                  }
                },
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size(double.infinity, 50),
                ),
                child: const Text('Submit'),
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
- Alphanumeric validation for usernames
- Email validation with regex
- Password strength validation
- Cross-field password confirmation
- Phone number validation
- Optional URL validation

---

# Async Validation

> **Asynchronous validation** with Future.

```dart
/// Async validation example
class AsyncValidationExample extends StatefulWidget {
  const AsyncValidationExample({super.key});

  @override
  State<AsyncValidationExample> createState() => _AsyncValidationExampleState();
}

class _AsyncValidationExampleState extends State<AsyncValidationExample> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Controllers
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();

  @override
  void dispose() {
    _usernameController.dispose();
    _emailController.dispose();
    super.dispose();
  }

  // Simulate checking if username is available
  // In a real app, this would be an API call
  Future<bool> _isUsernameAvailable(String username) async {
    // Simulate network delay
    await Future.delayed(const Duration(seconds: 1));
    
    // Simulate checking against existing usernames
    // For demo, 'admin' and 'test' are taken
    final takenUsernames = ['admin', 'test', 'user'];
    return !takenUsernames.contains(username.toLowerCase());
  }

  // Simulate checking if email is already registered
  Future<bool> _isEmailAvailable(String email) async {
    await Future.delayed(const Duration(seconds: 1));
    
    // For demo, certain emails are registered
    final takenEmails = ['admin@example.com', 'test@example.com'];
    return !takenEmails.contains(email.toLowerCase());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Async Validation'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // Username with async validation
              TextFormField(
                controller: _usernameController,
                decoration: const InputDecoration(
                  labelText: 'Username',
                  hintText: 'Choose a username',
                  border: OutlineInputBorder(),
                  helperText: 'Checking availability...',
                ),
                // Sync validation for format
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter a username';
                  }
                  if (value.length < 3) {
                    return 'Username must be at least 3 characters';
                  }
                  return null;
                },
                // Async validation using Future
                // Called when the form is validated
                autovalidateMode: AutovalidateMode.onUserInteraction,
              ),
              const SizedBox(height: 16),
              
              // Email with async validation
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  hintText: 'Enter your email',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.emailAddress,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your email';
                  }
                  if (!value.contains('@')) {
                    return 'Please enter a valid email';
                  }
                  return null;
                },
                autovalidateMode: AutovalidateMode.onUserInteraction,
              ),
              const SizedBox(height: 24),
              
              // Submit button with async validation
              ElevatedButton(
                onPressed: () async {
                  // First, validate sync validation
                  if (!_formKey.currentState!.validate()) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Please fix validation errors'),
                        backgroundColor: Colors.red,
                      ),
                    );
                    return;
                  }
                  
                  // Then, perform async validation
                  final username = _usernameController.text;
                  final email = _emailController.text;
                  
                  // Check username availability
                  final usernameAvailable = await _isUsernameAvailable(username);
                  if (!usernameAvailable) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Username is already taken'),
                        backgroundColor: Colors.red,
                      ),
                    );
                    return;
                  }
                  
                  // Check email availability
                  final emailAvailable = await _isEmailAvailable(email);
                  if (!emailAvailable) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Email is already registered'),
                        backgroundColor: Colors.red,
                      ),
                    );
                    return;
                  }
                  
                  // All validations passed
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(
                      content: Text('All validations passed!'),
                      backgroundColor: Colors.green,
                    ),
                  );
                },
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size(double.infinity, 50),
                ),
                child: const Text('Submit'),
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
- Async validation with Future
- Simulated API calls
- Availability checking
- Combined sync and async validation

---

# Custom Validators

> **Creating reusable** custom validators.

```dart
/// Custom validators for reuse
class CustomValidators {
  // Required field validator
  static String? required(String? value, {String fieldName = 'Field'}) {
    if (value == null || value.isEmpty) {
      return '$fieldName is required';
    }
    return null;
  }

  // Email validator
  static String? email(String? value) {
    if (value == null || value.isEmpty) {
      return 'Email is required';
    }
    final emailRegex = RegExp(
      r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
    );
    if (!emailRegex.hasMatch(value)) {
      return 'Please enter a valid email address';
    }
    return null;
  }

  // Password strength validator
  static String? password(String? value) {
    if (value == null || value.isEmpty) {
      return 'Password is required';
    }
    if (value.length < 8) {
      return 'Password must be at least 8 characters';
    }
    if (!RegExp(r'[A-Z]').hasMatch(value)) {
      return 'Password must contain at least one uppercase letter';
    }
    if (!RegExp(r'[a-z]').hasMatch(value)) {
      return 'Password must contain at least one lowercase letter';
    }
    if (!RegExp(r'[0-9]').hasMatch(value)) {
      return 'Password must contain at least one number';
    }
    if (!RegExp(r'[!@#$%^&*(),.?":{}|<>]').hasMatch(value)) {
      return 'Password must contain at least one special character';
    }
    return null;
  }

  // Password confirmation validator
  static String? confirmPassword(String? value, String password) {
    if (value == null || value.isEmpty) {
      return 'Please confirm your password';
    }
    if (value != password) {
      return 'Passwords do not match';
    }
    return null;
  }

  // Phone number validator
  static String? phone(String? value) {
    if (value == null || value.isEmpty) {
      return 'Phone number is required';
    }
    final cleaned = value.replaceAll(RegExp(r'[\s\-()]'), '');
    if (cleaned.length < 10) {
      return 'Please enter a valid phone number';
    }
    if (!RegExp(r'^\+?[0-9]+$').hasMatch(cleaned)) {
      return 'Please enter a valid phone number';
    }
    return null;
  }

  // Username validator
  static String? username(String? value) {
    if (value == null || value.isEmpty) {
      return 'Username is required';
    }
    if (value.length < 3) {
      return 'Username must be at least 3 characters';
    }
    if (value.length > 20) {
      return 'Username must be less than 20 characters';
    }
    if (!RegExp(r'^[a-zA-Z0-9_]+$').hasMatch(value)) {
      return 'Username can only contain letters, numbers, and underscores';
    }
    return null;
  }

  // Age validator
  static String? age(String? value) {
    if (value == null || value.isEmpty) {
      return 'Age is required';
    }
    final age = int.tryParse(value);
    if (age == null) {
      return 'Please enter a valid number';
    }
    if (age < 0 || age > 120) {
      return 'Please enter a valid age (0-120)';
    }
    return null;
  }

  // URL validator (optional)
  static String? url(String? value) {
    if (value == null || value.isEmpty) {
      return null; // Optional field
    }
    final urlRegex = RegExp(
      r'^(https?://)?([\da-z.-]+)\.([a-z.]{2,6})([/\w .-]*)*/?$',
    );
    if (!urlRegex.hasMatch(value)) {
      return 'Please enter a valid URL';
    }
    return null;
  }
}

/// Using custom validators
class CustomValidatorExample extends StatefulWidget {
  const CustomValidatorExample({super.key});

  @override
  State<CustomValidatorExample> createState() => _CustomValidatorExampleState();
}

class _CustomValidatorExampleState extends State<CustomValidatorExample> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Controllers
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _confirmPasswordController = TextEditingController();
  final TextEditingController _phoneController = TextEditingController();
  final TextEditingController _ageController = TextEditingController();

  @override
  void dispose() {
    _usernameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    _phoneController.dispose();
    _ageController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Validators'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // Using custom validators
              TextFormField(
                controller: _usernameController,
                decoration: const InputDecoration(
                  labelText: 'Username',
                  hintText: 'Choose a username',
                  border: OutlineInputBorder(),
                ),
                validator: CustomValidators.username,
              ),
              const SizedBox(height: 16),
              
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  hintText: 'Enter your email',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.emailAddress,
                validator: CustomValidators.email,
              ),
              const SizedBox(height: 16),
              
              TextFormField(
                controller: _passwordController,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter a strong password',
                  border: OutlineInputBorder(),
                ),
                obscureText: true,
                validator: CustomValidators.password,
              ),
              const SizedBox(height: 16),
              
              TextFormField(
                controller: _confirmPasswordController,
                decoration: const InputDecoration(
                  labelText: 'Confirm Password',
                  hintText: 'Re-enter your password',
                  border: OutlineInputBorder(),
                ),
                obscureText: true,
                validator: (value) {
                  return CustomValidators.confirmPassword(
                    value,
                    _passwordController.text,
                  );
                },
              ),
              const SizedBox(height: 16),
              
              TextFormField(
                controller: _phoneController,
                decoration: const InputDecoration(
                  labelText: 'Phone Number',
                  hintText: 'Enter your phone number',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.phone,
                validator: CustomValidators.phone,
              ),
              const SizedBox(height: 16),
              
              TextFormField(
                controller: _ageController,
                decoration: const InputDecoration(
                  labelText: 'Age',
                  hintText: 'Enter your age',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.number,
                validator: CustomValidators.age,
              ),
              const SizedBox(height: 24),
              
              // Submit button
              ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('All validations passed!'),
                        backgroundColor: Colors.green,
                      ),
                    );
                  }
                },
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size(double.infinity, 50),
                ),
                child: const Text('Submit'),
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
- Reusable custom validators
- Organized validation logic
- Easy to maintain and test
- Consistent validation across app

---

# Best Practices

## Provide Clear Error Messages

```dart
// Good - Clear, specific error messages
validator: (value) {
  if (value == null || value.isEmpty) {
    return 'Email address is required';
  }
  if (!value.contains('@')) {
    return 'Email must contain @ symbol';
  }
  return null;
}

// Bad - Vague error messages
validator: (value) {
  if (value == null || value.isEmpty) {
    return 'Invalid';
  }
  return null;
}
```

## Validate on User Interaction

```dart
// Good - Validate as user types
TextFormField(
  autovalidateMode: AutovalidateMode.onUserInteraction,
  validator: (value) => ...,
)

// Bad - Only validate on submit
TextFormField(
  validator: (value) => ...,
)
```

## Use Consistent Validation

```dart
// Good - Consistent rules across app
class Validators {
  static String? email(String? value) {
    // Same validation used everywhere
  }
}

// Bad - Different validation in different places
// field1 uses regex, field2 uses contains
```

---

# Common Mistakes

## Not Handling Null Values

Wrong:
```dart
validator: (value) {
  // Throws error if value is null
  if (value.isEmpty) {
    return 'Required';
  }
  return null;
}
```

Correct:
```dart
validator: (value) {
  if (value == null || value.isEmpty) {
    return 'Required';
  }
  return null;
}
```

## Not Validating Before Submit

Wrong:
```dart
ElevatedButton(
  onPressed: () {
    // Direct submission without validation
    submitForm();
  },
)
```

Correct:
```dart
ElevatedButton(
  onPressed: () {
    if (_formKey.currentState!.validate()) {
      submitForm();
    }
  },
)
```

---

# Summary

Form validation ensures data quality through rules and constraints. Use TextFormField's validator property for field-level validation, Form's validate method for overall validation, and custom validators for reusable logic. Always validate before processing form data.

---

# Next Steps

- [Focus](focus.md)
- [Keyboard Handling](keyboard-handling.md)
- [Input Formatters](input-formatters.md)

---

# Did You Know?

- validator returns error message or null
- validate() checks all form fields
- autovalidateMode controls validation timing
- Custom validators can be reused
- Async validation can check server-side
- Error messages guide user input
- Validation can be both sync and async
- FormState manages validation state