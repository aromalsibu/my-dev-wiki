# Form

Understand how to create and manage forms in Flutter applications.

---

# What is it?

Form is a container widget that groups multiple form fields together and manages their state, validation, and submission. It provides a convenient way to handle form validation, save form data, and manage the overall form state. Form works with TextFormField and other form field widgets to create structured data collection interfaces.

---

# Why does it exist?

Form exists to:

- Group and manage multiple form fields
- Handle form validation collectively
- Save form data easily
- Manage form state and lifecycle
- Enable form submission
- Support field-level validation
- Provide a structured way to collect user input

---

# Basic Form

> **Creating a simple** form with validation.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic Form example
class BasicFormExample extends StatefulWidget {
  const BasicFormExample({super.key});

  @override
  State<BasicFormExample> createState() => _BasicFormExampleState();
}

class _BasicFormExampleState extends State<BasicFormExample> {
  // 1. Create a GlobalKey for the form
  // This key is used to uniquely identify the form
  // and access its state for validation and saving
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  // 2. Create controllers for form fields
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  void dispose() {
    // Dispose controllers to prevent memory leaks
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Basic Form'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          // 3. Assign the GlobalKey to the Form
          // This allows us to validate and save the form
          key: _formKey,
          
          // 4. Add form fields as children
          child: Column(
            children: [
              // Name field
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Name',
                  hintText: 'Enter your name',
                  border: OutlineInputBorder(),
                ),
                // 5. Add validation logic
                // Validator returns an error message if invalid, null if valid
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your name';
                  }
                  if (value.length < 2) {
                    return 'Name must be at least 2 characters';
                  }
                  return null; // Valid
                },
              ),
              const SizedBox(height: 16),
              
              // Email field
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
                  // Simple email validation using regex
                  if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
                    return 'Please enter a valid email';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // Password field
              TextFormField(
                controller: _passwordController,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter your password',
                  border: OutlineInputBorder(),
                ),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter a password';
                  }
                  if (value.length < 6) {
                    return 'Password must be at least 6 characters';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              
              // 6. Submit button
              // This will validate the form when pressed
              ElevatedButton(
                onPressed: () {
                  // Validate the form
                  // This calls the validator on each form field
                  // Returns true if all validators pass
                  if (_formKey.currentState!.validate()) {
                    // Form is valid
                    // Process the form data
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Form submitted successfully!'),
                        backgroundColor: Colors.green,
                      ),
                    );
                  } else {
                    // Form is invalid
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
- GlobalKey<FormState> identifies the form
- Form widget groups fields together
- TextFormField with validation
- validate() checks all fields
- SnackBar shows submission status

---

# Form with Save

> **Using save functionality** to retrieve form data.

```dart
/// Form with save functionality
class FormSaveExample extends StatefulWidget {
  const FormSaveExample({super.key});

  @override
  State<FormSaveExample> createState() => _FormSaveExampleState();
}

class _FormSaveExampleState extends State<FormSaveExample> {
  // Form key for validation
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Controllers for form fields
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _phoneController = TextEditingController();
  
  // Variables to store form data
  // These will be populated when we call save()
  String _savedName = '';
  String _savedEmail = '';
  String _savedPhone = '';
  bool _dataSaved = false;

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _phoneController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Form with Save'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // Name field with onSaved
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Name',
                  hintText: 'Enter your name',
                  border: OutlineInputBorder(),
                ),
                // Validator ensures data is valid
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your name';
                  }
                  return null;
                },
                // onSaved is called when the form is saved
                // It stores the field value for later use
                onSaved: (value) {
                  // Store the name when the form is saved
                  _savedName = value ?? '';
                },
              ),
              const SizedBox(height: 16),
              
              // Email field with onSaved
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
                  return null;
                },
                onSaved: (value) {
                  // Store the email when the form is saved
                  _savedEmail = value ?? '';
                },
              ),
              const SizedBox(height: 16),
              
              // Phone field with onSaved
              TextFormField(
                controller: _phoneController,
                decoration: const InputDecoration(
                  labelText: 'Phone',
                  hintText: 'Enter your phone number',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.phone,
                onSaved: (value) {
                  // Store the phone when the form is saved
                  _savedPhone = value ?? '';
                },
              ),
              const SizedBox(height: 24),
              
              // Buttons row
              Row(
                children: [
                  // Save button
                  Expanded(
                    child: ElevatedButton(
                      onPressed: () {
                        // Validate the form
                        if (_formKey.currentState!.validate()) {
                          // Save the form data
                          // This calls onSaved on each form field
                          _formKey.currentState!.save();
                          
                          // Update UI to show saved data
                          setState(() {
                            _dataSaved = true;
                          });
                          
                          // Show success message
                          ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(
                              content: Text('Data saved successfully!'),
                              backgroundColor: Colors.green,
                            ),
                          );
                        }
                      },
                      child: const Text('Save'),
                    ),
                  ),
                  const SizedBox(width: 8),
                  // Reset button
                  Expanded(
                    child: ElevatedButton(
                      onPressed: () {
                        // Reset the form
                        // This clears all fields and resets validation
                        _formKey.currentState!.reset();
                        
                        // Clear controllers
                        _nameController.clear();
                        _emailController.clear();
                        _phoneController.clear();
                        
                        // Reset saved data
                        setState(() {
                          _savedName = '';
                          _savedEmail = '';
                          _savedPhone = '';
                          _dataSaved = false;
                        });
                      },
                      style: ElevatedButton.styleFrom(
                        backgroundColor: Colors.grey,
                      ),
                      child: const Text('Reset'),
                    ),
                  ),
                ],
              ),
              
              const SizedBox(height: 24),
              
              // Display saved data
              Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.blue[50],
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.blue[200]!),
                ),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Saved Data:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 16,
                      ),
                    ),
                    const SizedBox(height: 8),
                    if (_dataSaved) ...[
                      Text('Name: $_savedName'),
                      Text('Email: $_savedEmail'),
                      Text('Phone: $_savedPhone'),
                    ] else
                      const Text(
                        'No data saved yet',
                        style: TextStyle(color: Colors.grey),
                      ),
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
- onSaved stores field values
- save() calls onSaved on all fields
- reset() clears the form
- Saved data displayed separately

---

# Form with Custom Validation

> **Advanced validation** in forms.

```dart
/// Form with custom validation rules
class CustomValidationForm extends StatefulWidget {
  const CustomValidationForm({super.key});

  @override
  State<CustomValidationForm> createState() => _CustomValidationFormState();
}

class _CustomValidationFormState extends State<CustomValidationForm> {
  // Form key
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Controllers
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _confirmPasswordController = TextEditingController();

  @override
  void dispose() {
    _usernameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }

  // Custom validation functions
  
  // Check if username is available (simulated)
  // In a real app, this would check with a backend
  Future<bool> _isUsernameAvailable(String username) async {
    // Simulate API call
    await Future.delayed(const Duration(milliseconds: 500));
    // For demo, only 'admin' is taken
    return username.toLowerCase() != 'admin';
  }

  // Check if email is valid with domain restriction
  bool _isValidEmail(String email) {
    // Allow only specific domains for demo
    final allowedDomains = ['gmail.com', 'yahoo.com', 'outlook.com'];
    final parts = email.split('@');
    if (parts.length != 2) return false;
    return allowedDomains.contains(parts[1].toLowerCase());
  }

  // Check password strength
  String? _validatePassword(String? value) {
    if (value == null || value.isEmpty) {
      return 'Please enter a password';
    }
    if (value.length < 8) {
      return 'Password must be at least 8 characters';
    }
    if (!value.contains(RegExp(r'[A-Z]'))) {
      return 'Password must contain at least one uppercase letter';
    }
    if (!value.contains(RegExp(r'[a-z]'))) {
      return 'Password must contain at least one lowercase letter';
    }
    if (!value.contains(RegExp(r'[0-9]'))) {
      return 'Password must contain at least one number';
    }
    if (!value.contains(RegExp(r'[!@#$%^&*(),.?":{}|<>]'))) {
      return 'Password must contain at least one special character';
    }
    return null;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Validation'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: SingleChildScrollView(
            child: Column(
              children: [
                // Username with async validation
                TextFormField(
                  controller: _usernameController,
                  decoration: const InputDecoration(
                    labelText: 'Username',
                    hintText: 'Choose a username',
                    border: OutlineInputBorder(),
                  ),
                  // Synchronous validation
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please enter a username';
                    }
                    if (value.length < 3) {
                      return 'Username must be at least 3 characters';
                    }
                    if (!RegExp(r'^[a-zA-Z0-9_]+$').hasMatch(value)) {
                      return 'Username can only contain letters, numbers, and underscore';
                    }
                    return null;
                  },
                  // Async validation example
                  // In real app, this would be done with a service
                  onChanged: (value) {
                    // Simulate async validation
                    _isUsernameAvailable(value).then((available) {
                      if (!available && value.isNotEmpty) {
                        setState(() {
                          // Show error in form
                          // For complete async validation, consider using a different approach
                        });
                      }
                    });
                  },
                ),
                const SizedBox(height: 16),
                
                // Email with domain validation
                TextFormField(
                  controller: _emailController,
                  decoration: const InputDecoration(
                    labelText: 'Email',
                    hintText: 'Enter your email',
                    border: OutlineInputBorder(),
                    helperText: 'Allowed domains: gmail.com, yahoo.com, outlook.com',
                  ),
                  keyboardType: TextInputType.emailAddress,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please enter your email';
                    }
                    if (!value.contains('@')) {
                      return 'Please enter a valid email';
                    }
                    if (!_isValidEmail(value)) {
                      return 'Please use a supported email domain';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 16),
                
                // Password with strength validation
                TextFormField(
                  controller: _passwordController,
                  decoration: const InputDecoration(
                    labelText: 'Password',
                    hintText: 'Enter a strong password',
                    border: OutlineInputBorder(),
                    helperText: '8+ chars, uppercase, lowercase, number, special',
                  ),
                  obscureText: true,
                  validator: _validatePassword,
                ),
                const SizedBox(height: 16),
                
                // Confirm password
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
                    if (value != _passwordController.text) {
                      return 'Passwords do not match';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 24),
                
                // Submit button
                ElevatedButton(
                  onPressed: () {
                    if (_formKey.currentState!.validate()) {
                      // All validations passed
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('Registration successful!'),
                          backgroundColor: Colors.green,
                        ),
                      );
                    }
                  },
                  style: ElevatedButton.styleFrom(
                    minimumSize: const Size(double.infinity, 50),
                  ),
                  child: const Text('Register'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Custom validation rules
- Password strength validation
- Email domain validation
- Confirm password match
- Async validation simulation

---

# Real-World Examples

> **Common patterns** with Form.

```dart
/// 1. Registration form with all features
class RegistrationForm extends StatefulWidget {
  const RegistrationForm({super.key});

  @override
  State<RegistrationForm> createState() => _RegistrationFormState();
}

class _RegistrationFormState extends State<RegistrationForm> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  
  // Controllers
  final TextEditingController _firstNameController = TextEditingController();
  final TextEditingController _lastNameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _phoneController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _confirmPasswordController = TextEditingController();
  
  // State for toggles
  bool _agreeToTerms = false;
  bool _showPassword = false;
  bool _isSubmitting = false;

  @override
  void dispose() {
    // Dispose all controllers
    _firstNameController.dispose();
    _lastNameController.dispose();
    _emailController.dispose();
    _phoneController.dispose();
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }

  // Submit form
  Future<void> _submitForm() async {
    // Validate form
    if (!_formKey.currentState!.validate()) {
      return;
    }
    
    // Check terms agreement
    if (!_agreeToTerms) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Please agree to terms and conditions'),
          backgroundColor: Colors.orange,
        ),
      );
      return;
    }
    
    // Show loading
    setState(() {
      _isSubmitting = true;
    });
    
    // Simulate API call
    await Future.delayed(const Duration(seconds: 2));
    
    // Hide loading
    if (mounted) {
      setState(() {
        _isSubmitting = false;
      });
      
      // Show success
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Registration successful!'),
          backgroundColor: Colors.green,
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Register'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: SingleChildScrollView(
            child: Column(
              children: [
                // First name and last name in a row
                Row(
                  children: [
                    // First name
                    Expanded(
                      child: TextFormField(
                        controller: _firstNameController,
                        decoration: const InputDecoration(
                          labelText: 'First Name',
                          hintText: 'John',
                          border: OutlineInputBorder(),
                        ),
                        validator: (value) {
                          if (value == null || value.isEmpty) {
                            return 'Required';
                          }
                          return null;
                        },
                      ),
                    ),
                    const SizedBox(width: 8),
                    // Last name
                    Expanded(
                      child: TextFormField(
                        controller: _lastNameController,
                        decoration: const InputDecoration(
                          labelText: 'Last Name',
                          hintText: 'Doe',
                          border: OutlineInputBorder(),
                        ),
                        validator: (value) {
                          if (value == null || value.isEmpty) {
                            return 'Required';
                          }
                          return null;
                        },
                      ),
                    ),
                  ],
                ),
                const SizedBox(height: 16),
                
                // Email
                TextFormField(
                  controller: _emailController,
                  decoration: const InputDecoration(
                    labelText: 'Email',
                    hintText: 'john@example.com',
                    border: OutlineInputBorder(),
                    prefixIcon: Icon(Icons.email),
                  ),
                  keyboardType: TextInputType.emailAddress,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please enter your email';
                    }
                    if (!value.contains('@')) {
                      return 'Invalid email';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 16),
                
                // Phone
                TextFormField(
                  controller: _phoneController,
                  decoration: const InputDecoration(
                    labelText: 'Phone',
                    hintText: '+1 234 567 8900',
                    border: OutlineInputBorder(),
                    prefixIcon: Icon(Icons.phone),
                  ),
                  keyboardType: TextInputType.phone,
                ),
                const SizedBox(height: 16),
                
                // Password
                TextFormField(
                  controller: _passwordController,
                  decoration: InputDecoration(
                    labelText: 'Password',
                    hintText: 'Enter a strong password',
                    border: const OutlineInputBorder(),
                    prefixIcon: const Icon(Icons.lock),
                    suffixIcon: IconButton(
                      icon: Icon(
                        _showPassword ? Icons.visibility : Icons.visibility_off,
                      ),
                      onPressed: () {
                        setState(() {
                          _showPassword = !_showPassword;
                        });
                      },
                    ),
                  ),
                  obscureText: !_showPassword,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please enter a password';
                    }
                    if (value.length < 6) {
                      return 'Password must be at least 6 characters';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 16),
                
                // Confirm password
                TextFormField(
                  controller: _confirmPasswordController,
                  decoration: const InputDecoration(
                    labelText: 'Confirm Password',
                    hintText: 'Re-enter your password',
                    border: OutlineInputBorder(),
                    prefixIcon: Icon(Icons.lock_outline),
                  ),
                  obscureText: !_showPassword,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please confirm your password';
                    }
                    if (value != _passwordController.text) {
                      return 'Passwords do not match';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 16),
                
                // Terms and conditions
                Row(
                  children: [
                    Checkbox(
                      value: _agreeToTerms,
                      onChanged: (value) {
                        setState(() {
                          _agreeToTerms = value ?? false;
                        });
                      },
                    ),
                    Expanded(
                      child: GestureDetector(
                        onTap: () {
                          setState(() {
                            _agreeToTerms = !_agreeToTerms;
                          });
                        },
                        child: const Text(
                          'I agree to the terms and conditions',
                          style: TextStyle(fontSize: 14),
                        ),
                      ),
                    ),
                  ],
                ),
                const SizedBox(height: 24),
                
                // Submit button with loading state
                SizedBox(
                  width: double.infinity,
                  height: 50,
                  child: ElevatedButton(
                    onPressed: _isSubmitting ? null : _submitForm,
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.blue,
                    ),
                    child: _isSubmitting
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
                            'Register',
                            style: TextStyle(fontSize: 16),
                          ),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

/// 2. Multi-step form
class MultiStepForm extends StatefulWidget {
  const MultiStepForm({super.key});

  @override
  State<MultiStepForm> createState() => _MultiStepFormState();
}

class _MultiStepFormState extends State<MultiStepForm> {
  // Form keys for each step
  final GlobalKey<FormState> _step1Key = GlobalKey<FormState>();
  final GlobalKey<FormState> _step2Key = GlobalKey<FormState>();
  final GlobalKey<FormState> _step3Key = GlobalKey<FormState>();
  
  // Controllers for step 1
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  
  // Controllers for step 2
  final TextEditingController _addressController = TextEditingController();
  final TextEditingController _cityController = TextEditingController();
  
  // Controllers for step 3
  final TextEditingController _cardNumberController = TextEditingController();
  final TextEditingController _expiryController = TextEditingController();
  
  // Current step index
  int _currentStep = 0;
  
  // Max steps
  final int _maxSteps = 3;

  @override
  void dispose() {
    // Dispose all controllers
    _nameController.dispose();
    _emailController.dispose();
    _addressController.dispose();
    _cityController.dispose();
    _cardNumberController.dispose();
    _expiryController.dispose();
    super.dispose();
  }

  // Navigate to next step
  void _nextStep() {
    // Validate current step
    bool isValid = false;
    switch (_currentStep) {
      case 0:
        isValid = _step1Key.currentState?.validate() ?? false;
        break;
      case 1:
        isValid = _step2Key.currentState?.validate() ?? false;
        break;
      case 2:
        isValid = _step3Key.currentState?.validate() ?? false;
        break;
    }
    
    if (isValid) {
      setState(() {
        if (_currentStep < _maxSteps - 1) {
          _currentStep++;
        }
      });
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Please fill all required fields'),
          backgroundColor: Colors.red,
        ),
      );
    }
  }

  // Navigate to previous step
  void _prevStep() {
    setState(() {
      if (_currentStep > 0) {
        _currentStep--;
      }
    });
  }

  // Submit the form
  void _submitForm() {
    // Final validation
    if (_step3Key.currentState!.validate()) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Form completed successfully!'),
          backgroundColor: Colors.green,
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Multi-Step Form'),
      ),
      body: Column(
        children: [
          // Step indicator
          Container(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                _buildStepIndicator(0, 'Personal'),
                _buildStepConnector(0),
                _buildStepIndicator(1, 'Address'),
                _buildStepConnector(1),
                _buildStepIndicator(2, 'Payment'),
              ],
            ),
          ),
          
          // Form content
          Expanded(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: IndexedStack(
                index: _currentStep,
                children: [
                  // Step 1: Personal Information
                  _buildStep1(),
                  // Step 2: Address
                  _buildStep2(),
                  // Step 3: Payment
                  _buildStep3(),
                ],
              ),
            ),
          ),
          
          // Navigation buttons
          Container(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                // Back button
                if (_currentStep > 0)
                  TextButton(
                    onPressed: _prevStep,
                    child: const Text('Back'),
                  )
                else
                  const SizedBox.shrink(),
                
                // Next/Submit button
                ElevatedButton(
                  onPressed: _currentStep == _maxSteps - 1
                      ? _submitForm
                      : _nextStep,
                  child: Text(
                    _currentStep == _maxSteps - 1 ? 'Submit' : 'Next',
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  // Build step indicator
  Widget _buildStepIndicator(int index, String label) {
    final isActive = index == _currentStep;
    final isCompleted = index < _currentStep;
    
    return Column(
      children: [
        CircleAvatar(
          radius: 20,
          backgroundColor: isActive 
              ? Colors.blue 
              : isCompleted 
                  ? Colors.green 
                  : Colors.grey,
          child: Text(
            isCompleted ? '✓' : '${index + 1}',
            style: const TextStyle(color: Colors.white),
          ),
        ),
        const SizedBox(height: 4),
        Text(
          label,
          style: TextStyle(
            fontSize: 12,
            fontWeight: isActive ? FontWeight.bold : FontWeight.normal,
            color: isActive ? Colors.blue : Colors.grey,
          ),
        ),
      ],
    );
  }

  // Build connector between steps
  Widget _buildStepConnector(int index) {
    return Expanded(
      child: Container(
        height: 2,
        color: index < _currentStep ? Colors.green : Colors.grey,
        margin: const EdgeInsets.symmetric(horizontal: 8),
      ),
    );
  }

  // Step 1: Personal Information
  Widget _buildStep1() {
    return Form(
      key: _step1Key,
      child: Column(
        children: [
          const Text(
            'Personal Information',
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _nameController,
            decoration: const InputDecoration(
              labelText: 'Full Name',
              border: OutlineInputBorder(),
            ),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Required';
              }
              return null;
            },
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _emailController,
            decoration: const InputDecoration(
              labelText: 'Email',
              border: OutlineInputBorder(),
            ),
            keyboardType: TextInputType.emailAddress,
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Required';
              }
              return null;
            },
          ),
        ],
      ),
    );
  }

  // Step 2: Address
  Widget _buildStep2() {
    return Form(
      key: _step2Key,
      child: Column(
        children: [
          const Text(
            'Address Information',
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _addressController,
            decoration: const InputDecoration(
              labelText: 'Street Address',
              border: OutlineInputBorder(),
            ),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Required';
              }
              return null;
            },
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _cityController,
            decoration: const InputDecoration(
              labelText: 'City',
              border: OutlineInputBorder(),
            ),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Required';
              }
              return null;
            },
          ),
        ],
      ),
    );
  }

  // Step 3: Payment
  Widget _buildStep3() {
    return Form(
      key: _step3Key,
      child: Column(
        children: [
          const Text(
            'Payment Information',
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _cardNumberController,
            decoration: const InputDecoration(
              labelText: 'Card Number',
              hintText: '1234 5678 9012 3456',
              border: OutlineInputBorder(),
            ),
            keyboardType: TextInputType.number,
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Required';
              }
              if (value.replaceAll(' ', '').length < 16) {
                return 'Invalid card number';
              }
              return null;
            },
          ),
          const SizedBox(height: 16),
          Row(
            children: [
              Expanded(
                child: TextFormField(
                  controller: _expiryController,
                  decoration: const InputDecoration(
                    labelText: 'Expiry Date',
                    hintText: 'MM/YY',
                    border: OutlineInputBorder(),
                  ),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Required';
                    }
                    return null;
                  },
                ),
              ),
              const SizedBox(width: 8),
              Expanded(
                child: TextFormField(
                  decoration: const InputDecoration(
                    labelText: 'CVV',
                    hintText: '123',
                    border: OutlineInputBorder(),
                  ),
                  obscureText: true,
                  keyboardType: TextInputType.number,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Required';
                    }
                    if (value.length < 3) {
                      return 'Invalid CVV';
                    }
                    return null;
                  },
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Registration form with all features
- Multi-step form with validation
- Step indicators and navigation
- Loading states and show/hide password

---

# Best Practices

## Use Form Validation

```dart
// Good - Always validate
@override
Widget build(BuildContext context) {
  return Form(
    key: _formKey,
    child: TextFormField(
      validator: (value) {
        if (value == null || value.isEmpty) {
          return 'Required';
        }
        return null;
      },
    ),
  );
}
```

## Use onSaved for Data Collection

```dart
// Good - Use onSaved
TextFormField(
  onSaved: (value) {
    _formData.name = value;
  },
)

// Bad - Manual data collection
ElevatedButton(
  onPressed: () {
    final name = _controller.text;
  },
)
```

## Reset Forms Properly

```dart
// Good - Full reset
void _resetForm() {
  _formKey.currentState?.reset();
  _controller.clear();
}
```

---

# Common Mistakes

## Not Using GlobalKey

Wrong:
```dart
// Can't validate form
Form(
  child: Column(
    children: [...],
  ),
)
```

Correct:
```dart
// With GlobalKey
final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

Form(
  key: _formKey,
  child: Column(
    children: [...],
  ),
)
```

## Forgetting to Validate

Wrong:
```dart
// No validation before submission
ElevatedButton(
  onPressed: () {
    // Direct submission without validation
  },
)
```

Correct:
```dart
// Validate before submission
ElevatedButton(
  onPressed: () {
    if (_formKey.currentState!.validate()) {
      // Submit
    }
  },
)
```

---

# Summary

Form is a powerful container for managing multiple form fields. Use GlobalKey<FormState> for form identification, TextFormField for input fields with validation, and validate() for checking all fields. Form provides a structured way to collect, validate, and process user input.

---

# Next Steps

- [Form Validation](form-validation.md)
- [Focus](focus.md)
- [Keyboard Handling](keyboard-handling.md)

---

# Did You Know?

- Form uses GlobalKey for identification
- validate() checks all fields
- save() stores field data
- reset() clears the form
- TextFormField supports validation
- onSaved captures field data
- Forms can be multi-step
- FormState provides form control