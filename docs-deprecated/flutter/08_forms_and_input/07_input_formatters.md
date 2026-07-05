# Input Formatters

Understand how to format and restrict user input in Flutter.

---

# What is it?

Input Formatters are a powerful feature in Flutter that allow you to control, restrict, and format user input as they type. They act as filters that process each keystroke, allowing you to enforce specific formats, limit character length, allow only certain characters, and automatically format the input. This is essential for creating user-friendly input fields.

---

# Why does it exist?

Input Formatters exist to:

- Restrict input to specific characters (e.g., only digits)
- Enforce maximum character limits
- Auto-format input (e.g., phone numbers, dates)
- Prevent invalid input before submission
- Improve user experience with real-time formatting
- Reduce validation errors
- Handle specialized input formats

---

# Basic Input Formatters

> **Using input formatters** to restrict input.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

/// Basic input formatter example
class BasicInputFormatterExample extends StatefulWidget {
  const BasicInputFormatterExample({super.key});

  @override
  State<BasicInputFormatterExample> createState() => _BasicInputFormatterExampleState();
}

class _BasicInputFormatterExampleState extends State<BasicInputFormatterExample> {
  // Controllers for each field
  final TextEditingController _digitsController = TextEditingController();
  final TextEditingController _lettersController = TextEditingController();
  final TextEditingController _alphanumericController = TextEditingController();
  final TextEditingController _lengthController = TextEditingController();

  @override
  void dispose() {
    _digitsController.dispose();
    _lettersController.dispose();
    _alphanumericController.dispose();
    _lengthController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Input Formatters'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Only digits (0-9)
            TextField(
              controller: _digitsController,
              decoration: const InputDecoration(
                labelText: 'Only Digits',
                hintText: 'Enter numbers only',
                border: OutlineInputBorder(),
                helperText: 'Only 0-9 allowed',
              ),
              keyboardType: TextInputType.number,
              // 1. FilteringTextInputFormatter restricts characters
              // This allows only digits
              inputFormatters: [
                FilteringTextInputFormatter.digitsOnly,
              ],
            ),
            const SizedBox(height: 16),
            
            // 2. Only letters (A-Z, a-z)
            TextField(
              controller: _lettersController,
              decoration: const InputDecoration(
                labelText: 'Only Letters',
                hintText: 'Enter letters only',
                border: OutlineInputBorder(),
                helperText: 'Only A-Z and a-z allowed',
              ),
              // 2. Custom pattern for letters only
              inputFormatters: [
                FilteringTextInputFormatter.allow(
                  RegExp(r'[a-zA-Z]'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            
            // 3. Alphanumeric (letters and numbers)
            TextField(
              controller: _alphanumericController,
              decoration: const InputDecoration(
                labelText: 'Alphanumeric',
                hintText: 'Enter letters and numbers',
                border: OutlineInputBorder(),
                helperText: 'A-Z, a-z, 0-9 allowed',
              ),
              // 3. Custom pattern for alphanumeric
              inputFormatters: [
                FilteringTextInputFormatter.allow(
                  RegExp(r'[a-zA-Z0-9]'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            
            // 4. Length limiting
            TextField(
              controller: _lengthController,
              decoration: const InputDecoration(
                labelText: 'Max 10 Characters',
                hintText: 'Type something...',
                border: OutlineInputBorder(),
                helperText: 'Maximum 10 characters',
              ),
              // 4. LengthLimitingTextInputFormatter limits characters
              inputFormatters: [
                LengthLimitingTextInputFormatter(10),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- FilteringTextInputFormatter.digitsOnly: Only digits
- FilteringTextInputFormatter.allow: Custom allowed pattern
- FilteringTextInputFormatter.deny: Block specific pattern
- LengthLimitingTextInputFormatter: Max length

---

# Formatters for Specific Formats

> **Creating formatted** input fields.

```dart
/// 1. Phone number formatter
class PhoneInputFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    // Get the new text without formatting
    final text = newValue.text.replaceAll(RegExp(r'[\s\-()]'), '');
    
    // Only allow digits
    if (text.isEmpty) {
      return newValue.copyWith(text: '');
    }
    
    // Format as (XXX) XXX-XXXX
    final buffer = StringBuffer();
    
    // Add opening parenthesis
    buffer.write('(');
    
    // Add first 3 digits
    final firstThree = text.substring(0, text.length > 3 ? 3 : text.length);
    buffer.write(firstThree);
    
    // Close parenthesis and add space after 3 digits
    if (text.length > 3) {
      buffer.write(') ');
      final remaining = text.substring(3);
      final nextThree = remaining.length > 3 ? 3 : remaining.length;
      buffer.write(remaining.substring(0, nextThree));
    }
    
    // Add dash and last 4 digits
    if (text.length > 6) {
      buffer.write('-');
      final remaining = text.substring(6);
      buffer.write(remaining);
    }
    
    return newValue.copyWith(
      text: buffer.toString(),
      selection: TextSelection.collapsed(offset: buffer.length),
    );
  }
}

/// 2. Credit card number formatter
class CreditCardInputFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    // Remove all non-digits
    final text = newValue.text.replaceAll(RegExp(r'[^0-9]'), '');
    
    if (text.isEmpty) {
      return newValue.copyWith(text: '');
    }
    
    // Format as XXXX XXXX XXXX XXXX
    final buffer = StringBuffer();
    for (int i = 0; i < text.length; i++) {
      if (i > 0 && i % 4 == 0) {
        buffer.write(' ');
      }
      buffer.write(text[i]);
    }
    
    return newValue.copyWith(
      text: buffer.toString(),
      selection: TextSelection.collapsed(offset: buffer.length),
    );
  }
}

/// 3. Date formatter (MM/DD/YYYY)
class DateInputFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    // Remove all non-digits
    final text = newValue.text.replaceAll(RegExp(r'[^0-9]'), '');
    
    if (text.isEmpty) {
      return newValue.copyWith(text: '');
    }
    
    // Format as MM/DD/YYYY
    final buffer = StringBuffer();
    for (int i = 0; i < text.length && i < 8; i++) {
      if (i == 2 || i == 4) {
        buffer.write('/');
      }
      buffer.write(text[i]);
    }
    
    return newValue.copyWith(
      text: buffer.toString(),
      selection: TextSelection.collapsed(offset: buffer.length),
    );
  }
}

/// 4. Currency formatter
class CurrencyInputFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    // Remove all non-digits
    final text = newValue.text.replaceAll(RegExp(r'[^0-9]'), '');
    
    if (text.isEmpty) {
      return newValue.copyWith(text: '');
    }
    
    // Format as $X,XXX.XX
    final number = int.parse(text);
    final formatted = '${(number / 100).toStringAsFixed(2)}';
    
    // Add commas and dollar sign
    final parts = formatted.split('.');
    final dollars = parts[0];
    final cents = parts[1];
    
    // Add commas to dollars
    final buffer = StringBuffer('\$');
    for (int i = 0; i < dollars.length; i++) {
      if (i > 0 && (dollars.length - i) % 3 == 0) {
        buffer.write(',');
      }
      buffer.write(dollars[i]);
    }
    buffer.write('.$cents');
    
    return newValue.copyWith(
      text: buffer.toString(),
      selection: TextSelection.collapsed(offset: buffer.length),
    );
  }
}

/// Using custom formatters
class CustomFormatterExample extends StatefulWidget {
  const CustomFormatterExample({super.key});

  @override
  State<CustomFormatterExample> createState() => _CustomFormatterExampleState();
}

class _CustomFormatterExampleState extends State<CustomFormatterExample> {
  // Controllers for formatted fields
  final TextEditingController _phoneController = TextEditingController();
  final TextEditingController _cardController = TextEditingController();
  final TextEditingController _dateController = TextEditingController();
  final TextEditingController _currencyController = TextEditingController();

  @override
  void dispose() {
    _phoneController.dispose();
    _cardController.dispose();
    _dateController.dispose();
    _currencyController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Formatters'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Phone number with custom formatter
            TextField(
              controller: _phoneController,
              decoration: const InputDecoration(
                labelText: 'Phone Number',
                hintText: '(XXX) XXX-XXXX',
                border: OutlineInputBorder(),
                helperText: 'Format: (XXX) XXX-XXXX',
              ),
              keyboardType: TextInputType.phone,
              inputFormatters: [
                PhoneInputFormatter(),
                // Also limit to 14 characters (with formatting)
                LengthLimitingTextInputFormatter(14),
              ],
            ),
            const SizedBox(height: 16),
            
            // 2. Credit card with custom formatter
            TextField(
              controller: _cardController,
              decoration: const InputDecoration(
                labelText: 'Credit Card',
                hintText: 'XXXX XXXX XXXX XXXX',
                border: OutlineInputBorder(),
                helperText: 'Format: XXXX XXXX XXXX XXXX',
              ),
              keyboardType: TextInputType.number,
              inputFormatters: [
                CreditCardInputFormatter(),
                LengthLimitingTextInputFormatter(19),
              ],
            ),
            const SizedBox(height: 16),
            
            // 3. Date with custom formatter
            TextField(
              controller: _dateController,
              decoration: const InputDecoration(
                labelText: 'Date',
                hintText: 'MM/DD/YYYY',
                border: OutlineInputBorder(),
                helperText: 'Format: MM/DD/YYYY',
              ),
              keyboardType: TextInputType.number,
              inputFormatters: [
                DateInputFormatter(),
                LengthLimitingTextInputFormatter(10),
              ],
            ),
            const SizedBox(height: 16),
            
            // 4. Currency with custom formatter
            TextField(
              controller: _currencyController,
              decoration: const InputDecoration(
                labelText: 'Amount',
                hintText: '$0.00',
                border: OutlineInputBorder(),
                helperText: 'Format: $X,XXX.XX',
              ),
              keyboardType: TextInputType.number,
              inputFormatters: [
                CurrencyInputFormatter(),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Phone number formatting (XXX) XXX-XXXX
- Credit card formatting XXXX XXXX XXXX XXXX
- Date formatting MM/DD/YYYY
- Currency formatting $X,XXX.XX

---

# Combining Formatters

> **Using multiple** formatters together.

```dart
/// Combining multiple formatters
class CombinedFormatterExample extends StatefulWidget {
  const CombinedFormatterExample({super.key});

  @override
  State<CombinedFormatterExample> createState() => _CombinedFormatterExampleState();
}

class _CombinedFormatterExampleState extends State<CombinedFormatterExample> {
  // Controllers for combined formatters
  final TextEditingController _controller1 = TextEditingController();
  final TextEditingController _controller2 = TextEditingController();
  final TextEditingController _controller3 = TextEditingController();

  @override
  void dispose() {
    _controller1.dispose();
    _controller2.dispose();
    _controller3.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Combined Formatters'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Uppercase only with length limit
            TextField(
              controller: _controller1,
              decoration: const InputDecoration(
                labelText: 'Uppercase + Limit',
                hintText: 'Uppercase only, max 10 chars',
                border: OutlineInputBorder(),
                helperText: 'A-Z only, uppercase, max 10',
              ),
              inputFormatters: [
                // 1. Allow only uppercase letters
                FilteringTextInputFormatter.allow(
                  RegExp(r'[A-Z]'),
                ),
                // 2. Limit length
                LengthLimitingTextInputFormatter(10),
              ],
            ),
            const SizedBox(height: 16),
            
            // 2. Alphanumeric with formatting
            TextField(
              controller: _controller2,
              decoration: const InputDecoration(
                labelText: 'Alphanumeric + Format',
                hintText: 'ABC-123-XYZ',
                border: OutlineInputBorder(),
                helperText: 'Format: ABC-123-XYZ',
              ),
              inputFormatters: [
                // 1. Allow alphanumeric
                FilteringTextInputFormatter.allow(
                  RegExp(r'[a-zA-Z0-9]'),
                ),
                // 2. Auto-capitalize
                TextInputFormatter.withFunction(
                  (oldValue, newValue) {
                    return newValue.copyWith(
                      text: newValue.text.toUpperCase(),
                    );
                  },
                ),
                // 3. Length limit
                LengthLimitingTextInputFormatter(11),
              ],
            ),
            const SizedBox(height: 16),
            
            // 3. Price with formatters
            TextField(
              controller: _controller3,
              decoration: const InputDecoration(
                labelText: 'Price',
                hintText: '$0.00',
                border: OutlineInputBorder(),
                helperText: 'Format: $X,XXX.XX',
              ),
              keyboardType: TextInputType.number,
              inputFormatters: [
                // 1. Allow digits and decimal
                FilteringTextInputFormatter.allow(
                  RegExp(r'[0-9.]'),
                ),
                // 2. Currency formatting
                CurrencyInputFormatter(),
                // 3. Limit total length
                LengthLimitingTextInputFormatter(12),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Combining allow and length limit
- Auto-capitalization with formatters
- Multiple filters for price input
- Formatters execute in order

---

# Custom TextInputFormatter

> **Creating custom** formatters.

```dart
/// Custom formatter for specific patterns
class SSNInputFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    // Remove all non-digits
    final text = newValue.text.replaceAll(RegExp(r'[^0-9]'), '');
    
    if (text.isEmpty) {
      return newValue.copyWith(text: '');
    }
    
    // Format as XXX-XX-XXXX
    final buffer = StringBuffer();
    for (int i = 0; i < text.length && i < 9; i++) {
      if (i == 3 || i == 5) {
        buffer.write('-');
      }
      buffer.write(text[i]);
    }
    
    return newValue.copyWith(
      text: buffer.toString(),
      selection: TextSelection.collapsed(offset: buffer.length),
    );
  }
}

/// Custom formatter with validation
class ValidatingFormatter extends TextInputFormatter {
  final String pattern;
  final String errorMessage;

  ValidatingFormatter(this.pattern, this.errorMessage);

  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    // Check if the new value matches the pattern
    if (newValue.text.isNotEmpty && 
        !RegExp(pattern).hasMatch(newValue.text)) {
      // Revert to old value if invalid
      return oldValue;
    }
    return newValue;
  }
}

/// Custom formatter for hexadecimal input
class HexInputFormatter extends TextInputFormatter {
  @override
  TextEditingValue formatEditUpdate(
    TextEditingValue oldValue,
    TextEditingValue newValue,
  ) {
    // Allow only hex characters
    final filtered = newValue.text.replaceAll(
      RegExp(r'[^0-9a-fA-F]'),
      '',
    );
    
    // Auto-uppercase
    final formatted = filtered.toUpperCase();
    
    return newValue.copyWith(
      text: formatted,
      selection: TextSelection.collapsed(offset: formatted.length),
    );
  }
}

/// Using custom formatters
class CustomTextInputFormatterExample extends StatefulWidget {
  const CustomTextInputFormatterExample({super.key});

  @override
  State<CustomTextInputFormatterExample> createState() => _CustomTextInputFormatterExampleState();
}

class _CustomTextInputFormatterExampleState extends State<CustomTextInputFormatterExample> {
  final TextEditingController _ssnController = TextEditingController();
  final TextEditingController _hexController = TextEditingController();
  final TextEditingController _validatedController = TextEditingController();

  @override
  void dispose() {
    _ssnController.dispose();
    _hexController.dispose();
    _validatedController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Formatters'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. SSN formatter
            TextField(
              controller: _ssnController,
              decoration: const InputDecoration(
                labelText: 'SSN',
                hintText: 'XXX-XX-XXXX',
                border: OutlineInputBorder(),
                helperText: 'Format: XXX-XX-XXXX',
              ),
              keyboardType: TextInputType.number,
              inputFormatters: [
                SSNInputFormatter(),
                LengthLimitingTextInputFormatter(11),
              ],
            ),
            const SizedBox(height: 16),
            
            // 2. Hex formatter
            TextField(
              controller: _hexController,
              decoration: const InputDecoration(
                labelText: 'Hex Color',
                hintText: 'A1B2C3',
                border: OutlineInputBorder(),
                helperText: 'Hex characters only (0-9, A-F)',
              ),
              inputFormatters: [
                HexInputFormatter(),
                LengthLimitingTextInputFormatter(6),
              ],
            ),
            const SizedBox(height: 16),
            
            // 3. Validating formatter
            TextField(
              controller: _validatedController,
              decoration: const InputDecoration(
                labelText: 'Postal Code',
                hintText: '12345 or 12345-6789',
                border: OutlineInputBorder(),
                helperText: 'US Postal Code format',
              ),
              inputFormatters: [
                // Only allow digits and dash
                FilteringTextInputFormatter.allow(
                  RegExp(r'[0-9\-]'),
                ),
                // Format: 12345 or 12345-6789
                ValidatingFormatter(
                  r'^\d{5}(-\d{4})?$',
                  'Invalid postal code format',
                ),
                LengthLimitingTextInputFormatter(10),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- SSN format XXX-XX-XXXX
- Hexadecimal auto-uppercase
- Validating formatter for postal codes
- Custom formatting logic

---

# Best Practices

## Use Appropriate Formatters

```dart
// Good - Correct formatter for the input
TextField(
  inputFormatters: [
    FilteringTextInputFormatter.digitsOnly,
  ],
  decoration: InputDecoration(labelText: 'Age'),
)

// Bad - Wrong formatter
TextField(
  inputFormatters: [
    FilteringTextInputFormatter.allow(RegExp(r'[a-zA-Z]')),
  ],
  decoration: InputDecoration(labelText: 'Age'),
)
```

## Combine Formatters

```dart
// Good - Multiple formatters
TextField(
  inputFormatters: [
    FilteringTextInputFormatter.allow(RegExp(r'[0-9]')),
    LengthLimitingTextInputFormatter(10),
  ],
)

// Bad - One formatter for everything
TextField(
  inputFormatters: [
    // Trying to do everything in one formatter
  ],
)
```

## Test Formatters

```dart
// Good - Test custom formatters
test('Phone formatter formats correctly', () {
  final formatter = PhoneInputFormatter();
  // Test various inputs
  expect(formatter.formatEditUpdate(...), ...);
});
```

---

# Common Mistakes

## Not Considering Keyboard Type

Wrong:
```dart
// Text keyboard with number formatter
TextField(
  keyboardType: TextInputType.text,
  inputFormatters: [FilteringTextInputFormatter.digitsOnly],
)
```

Correct:
```dart
// Number keyboard with number formatter
TextField(
  keyboardType: TextInputType.number,
  inputFormatters: [FilteringTextInputFormatter.digitsOnly],
)
```

## Formatters in Wrong Order

Wrong:
```dart
// Length limit before formatting
inputFormatters: [
  LengthLimitingTextInputFormatter(14),
  PhoneInputFormatter(),
]
```

Correct:
```dart
// Formatting before length limit
inputFormatters: [
  PhoneInputFormatter(),
  LengthLimitingTextInputFormatter(14),
]
```

---

# Summary

Input Formatters control and format user input as they type. Use filtering formatters for character restrictions, length limiters for max characters, and custom formatters for specific formats like phone numbers, credit cards, and dates. Combine formatters for complex formatting requirements.

---

# Next Steps

- [Forms & Input Overview](forms-input-overview.md)
- [TextEditingController](texteditingcontroller.md)
- [Form Validation](form-validation.md)

---

# Did You Know?

- Formatters process each keystroke
- Formatters execute in order
- Custom formatters can implement any logic
- Formatters can auto-format input
- Formatters can validate input
- Formatters work with any TextField
- Formatters can be combined
- Formatters improve user experience