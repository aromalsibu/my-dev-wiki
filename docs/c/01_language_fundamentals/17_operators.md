# Operators

## 1. Overview

### Definition
Operators in C are symbols that perform operations on operands (values or variables). They form the foundation of expressions, allowing you to perform arithmetic, comparisons, logical decisions, bit manipulation, assignment, and more. C provides a rich set of operators, many of which map directly to hardware instructions.

### Purpose
Operators are the verbs of the C language. They enable you to:
- Perform mathematical calculations (`+`, `-`, `*`, `/`, `%`).
- Compare values (`==`, `!=`, `<`, `>`, `<=`, `>=`).
- Make logical decisions (`&&`, `||`, `!`).
- Manipulate bits at the hardware level (`&`, `|`, `^`, `~`, `<<`, `>>`).
- Assign values (`=`, `+=`, `-=`, etc.).
- Access memory (`*` dereference, `&` address-of).
- Determine sizes (`sizeof`).
- Control flow with conditional expressions (`? :`).

### Where It Fits
Operators are ubiquitous in C code. They appear in:
- **Expressions** – `x = a + b * c;`
- **Conditionals** – `if (x > 0 && y < 10)`
- **Loops** – `for (int i = 0; i < n; i++)`
- **Function calls** – passing arguments (`func(a + b, c - d)`).
- **Initialisers** – `int x = 5 + 3;`

---

## 2. Why It Exists

### The Problem Without Operators
Without operators, every operation would require a function call:
```c
int result = add(a, multiply(b, c));  // instead of a + b * c
```
This would be cumbersome, slow, and unreadable.

### The Solution: Symbolic Operators
Operators provide a concise, expressive syntax that closely mirrors mathematical notation. They are:
- **Intuitive** – `a + b` is universally understood.
- **Efficient** – many operators compile to a single machine instruction.
- **Flexible** – combined with data types, operators adapt to different contexts (e.g., pointer arithmetic).

### Why They Were Introduced
C inherited its operators from B and BCPL, which in turn were influenced by ALGOL. The design philosophy was to provide a rich set of operators that map efficiently to hardware, giving programmers low‑level control while maintaining high‑level readability. C's bitwise operators, in particular, were designed for systems programming (manipulating hardware registers, flags, etc.).

---

## 3. Syntax / Basic Usage

### Arithmetic Operators
```c
#include <stdio.h>

int main(void) {
    int a = 10, b = 3;

    // Addition
    int sum = a + b;           // 13

    // Subtraction
    int diff = a - b;          // 7

    // Multiplication
    int product = a * b;       // 30

    // Division (integer)
    int quotient = a / b;      // 3 (truncates toward zero)

    // Modulus (remainder)
    int remainder = a % b;     // 1

    printf("Sum: %d, Diff: %d, Product: %d, Quotient: %d, Remainder: %d\n",
           sum, diff, product, quotient, remainder);

    // Float division
    float result = 10.0f / 3.0f;  // 3.333333
    printf("Float division: %f\n", result);

    return 0;
}
```

### Relational (Comparison) Operators
```c
#include <stdio.h>

int main(void) {
    int x = 10, y = 20;

    printf("x == y: %d\n", x == y);   // 0 (false)
    printf("x != y: %d\n", x != y);   // 1 (true)
    printf("x < y: %d\n", x < y);     // 1
    printf("x > y: %d\n", x > y);     // 0
    printf("x <= y: %d\n", x <= y);   // 1
    printf("x >= y: %d\n", x >= y);   // 0

    return 0;
}
```

### Logical Operators
```c
#include <stdio.h>
#include <stdbool.h>

int main(void) {
    bool a = true, b = false;

    // Logical AND – true only if both are true
    printf("a && b: %d\n", a && b);   // 0

    // Logical OR – true if at least one is true
    printf("a || b: %d\n", a || b);   // 1

    // Logical NOT – flips truth value
    printf("!a: %d\n", !a);           // 0
    printf("!b: %d\n", !b);           // 1

    // Short-circuit evaluation
    int x = 0;
    if (x != 0 && 10 / x > 0) {       // x != 0 is false, so 10 / x is NOT evaluated
        printf("Division safe\n");
    }
    printf("x: %d\n", x);

    return 0;
}
```

### Bitwise Operators
```c
#include <stdio.h>

int main(void) {
    unsigned int a = 0b1010;   // 10 in decimal
    unsigned int b = 0b1100;   // 12 in decimal

    // Bitwise AND
    unsigned int and_result = a & b;   // 0b1000 (8)

    // Bitwise OR
    unsigned int or_result = a | b;    // 0b1110 (14)

    // Bitwise XOR (exclusive OR)
    unsigned int xor_result = a ^ b;   // 0b0110 (6)

    // Bitwise NOT (complement)
    unsigned int not_result = ~a;      // ...11110101 (implementation-dependent)

    // Left shift
    unsigned int left_shift = a << 2;  // 0b101000 (40)

    // Right shift
    unsigned int right_shift = a >> 1; // 0b0101 (5)

    printf("a & b: %u\n", and_result);
    printf("a | b: %u\n", or_result);
    printf("a ^ b: %u\n", xor_result);
    printf("~a: %u\n", not_result);
    printf("a << 2: %u\n", left_shift);
    printf("a >> 1: %u\n", right_shift);

    return 0;
}
```

### Assignment Operators
```c
#include <stdio.h>

int main(void) {
    int x = 10;

    // Simple assignment
    x = 5;
    printf("x = 5: %d\n", x);

    // Add and assign
    x += 3;        // x = x + 3;  -> 8
    printf("x += 3: %d\n", x);

    // Subtract and assign
    x -= 2;        // x = x - 2;  -> 6
    printf("x -= 2: %d\n", x);

    // Multiply and assign
    x *= 2;        // x = x * 2;  -> 12
    printf("x *= 2: %d\n", x);

    // Divide and assign
    x /= 3;        // x = x / 3;  -> 4
    printf("x /= 3: %d\n", x);

    // Modulus and assign
    x %= 3;        // x = x % 3;  -> 1
    printf("x %%= 3: %d\n", x);

    // Bitwise AND and assign
    int y = 0b1100;
    y &= 0b1010;   // y = y & 0b1010; -> 0b1000 (8)
    printf("y &= 0b1010: %d\n", y);

    // Bitwise OR and assign
    y |= 0b0101;   // y = y | 0b0101; -> 0b1101 (13)
    printf("y |= 0b0101: %d\n", y);

    return 0;
}
```

### Increment and Decrement Operators
```c
#include <stdio.h>

int main(void) {
    int x = 5;

    // Post-increment: returns current value, then increments
    int y = x++;   // y = 5, x = 6
    printf("x++: x = %d, y = %d\n", x, y);

    // Pre-increment: increments, then returns new value
    int z = ++x;   // x = 7, z = 7
    printf("++x: x = %d, z = %d\n", x, z);

    // Post-decrement
    int a = x--;   // a = 7, x = 6
    printf("x--: x = %d, a = %d\n", x, a);

    // Pre-decrement
    int b = --x;   // x = 5, b = 5
    printf("--x: x = %d, b = %d\n", x, b);

    return 0;
}
```

### Conditional (Ternary) Operator
```c
#include <stdio.h>

int main(void) {
    int a = 10, b = 20;

    // condition ? expression_if_true : expression_if_false
    int max = (a > b) ? a : b;   // max = 20
    printf("Max: %d\n", max);

    // Can be nested (but readability suffers)
    int min = (a < b) ? a : (b < 10 ? b : 0);
    printf("Min: %d\n", min);    // 10

    return 0;
}
```

### Sizeof Operator
```c
#include <stdio.h>

int main(void) {
    int x = 10;
    double arr[10];

    // sizeof(type) – returns size in bytes
    printf("sizeof(int): %zu\n", sizeof(int));
    printf("sizeof(double): %zu\n", sizeof(double));
    printf("sizeof(arr): %zu\n", sizeof(arr));      // 80 bytes (10 * 8)
    printf("sizeof(x): %zu\n", sizeof(x));          // 4 bytes (typically)
    printf("sizeof(arr) / sizeof(arr[0]): %zu\n", 
           sizeof(arr) / sizeof(arr[0]));           // 10 (array length)

    // sizeof on expressions
    printf("sizeof(x + 5): %zu\n", sizeof(x + 5));  // sizeof(int)

    return 0;
}
```

### Pointer Operators
```c
#include <stdio.h>

int main(void) {
    int x = 42;
    int *ptr = &x;          // & – address-of operator

    // * – dereference operator (access value at address)
    printf("x: %d\n", x);
    printf("*ptr: %d\n", *ptr);

    *ptr = 100;             // Modifies x through pointer
    printf("x after *ptr = 100: %d\n", x);

    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>

int main(void) {
    // Arithmetic operators
    int a = 10, b = 3;
    int sum = a + b;        // '+' adds two operands.
    int diff = a - b;       // '-' subtracts.
    int product = a * b;    // '*' multiplies.
    int quotient = a / b;   // '/' divides integers (truncates toward zero).
    int remainder = a % b;  // '%' gives remainder of division.

    // Note: integer division truncates, not rounds.
    // For float division, at least one operand must be float/double.
    float f_result = 10.0f / 3.0f; // 3.333333

    // Relational operators – return int (1 for true, 0 for false).
    int is_equal = (a == b);       // 0 (false)
    int is_not_equal = (a != b);   // 1 (true)
    int is_greater = (a > b);      // 1
    int is_less = (a < b);         // 0
    int is_ge = (a >= b);          // 1 (10 >= 3)
    int is_le = (a <= b);          // 0

    // Logical operators – operate on boolean values (0/false, nonzero/true).
    int x = 10, y = 0;
    int and_result = (x > 5 && y == 0); // 1 (both true)
    int or_result = (x < 5 || y == 0);   // 1 (second true)
    int not_result = !(x > 5);           // 0 (x > 5 is true, so !true = false)

    // Short-circuit evaluation: second operand not evaluated if first determines result.
    if (y != 0 && x / y > 0) {   // y != 0 is false, so division is skipped.
        // Safe from division by zero.
    }

    // Bitwise operators – work on integer types bit-by-bit.
    unsigned int u = 0b1010;       // 10 in binary (C23 binary literal)
    unsigned int v = 0b1100;       // 12 in binary
    unsigned int bit_and = u & v;  // 0b1000 (8)   – AND
    unsigned int bit_or = u | v;   // 0b1110 (14)  – OR
    unsigned int bit_xor = u ^ v;  // 0b0110 (6)   – XOR
    unsigned int bit_not = ~u;     // Complements all bits (implementation-defined).
    unsigned int shift_left = u << 2; // 0b101000 (40) – multiply by 2^2
    unsigned int shift_right = u >> 1; // 0b0101 (5)  – divide by 2

    // Assignment operators – perform operation and assign.
    int val = 10;
    val += 5;   // val = val + 5 = 15
    val -= 3;   // val = val - 3 = 12
    val *= 2;   // val = val * 2 = 24
    val /= 4;   // val = val / 4 = 6
    val %= 4;   // val = val % 4 = 2

    // Increment/decrement – add/subtract 1.
    int count = 5;
    int post_increment = count++; // post_increment = 5, count = 6
    int pre_increment = ++count;  // count = 7, pre_increment = 7

    // Conditional (ternary) – shorthand for if-else.
    int max = (a > b) ? a : b;    // if a>b, max=a; else max=b

    // sizeof – compile-time operator returns size in bytes.
    size_t size_int = sizeof(int);      // usually 4
    size_t size_array = sizeof(arr);    // total bytes of array

    // Pointer operators: & (address-of), * (dereference).
    int value = 42;
    int *ptr = &value;    // ptr holds address of value
    *ptr = 100;           // changes value through pointer

    // Comma operator – evaluates left, discards result, evaluates right.
    int result = (a = 5, b = 10, a + b); // result = 15

    return 0;
}
```

---

## 4. Mental Model – Operators as Tools in a Workshop

Imagine you have a workshop with various tools:

- **Arithmetic operators** (`+`, `-`, `*`, `/`, `%`) – like a calculator. You put numbers in, and you get a result.

- **Relational operators** (`==`, `!=`, `<`, `>`) – like a scale. You compare two items and get a yes/no (true/false) answer.

- **Logical operators** (`&&`, `||`, `!`) – like logic gates in electronics. AND requires both inputs; OR requires at least one; NOT flips the signal.

- **Bitwise operators** (`&`, `|`, `^`, `~`, `<<`, `>>`) – like a microscope that looks at individual bits. You can see each tiny 1 or 0 and manipulate them.

- **Assignment operators** (`=`, `+=`, `-=`) – like a label maker. You take a value and stick it onto a variable (assignment). The compound ones (`+=`) are like "take the current label, add 5, and re-stick it."

- **Increment/decrement** (`++`, `--`) – like a step counter. Add or subtract 1 in one quick motion.

- **Conditional operator** (`? :`) – like a fork in the road. Depending on a condition, you go one way or the other.

- **Pointer operators** (`&`, `*`) – like a treasure map. `&` gives you the coordinates; `*` lets you dig up the treasure at those coordinates.

- **`sizeof`** – like a measuring tape. You measure how much space something takes.

Each tool has its own rules for how it works (precedence, associativity), and you can combine them to build complex expressions.

---

## 5. Core Concepts

### Operator Categories

| Category | Operators | Description |
|----------|-----------|-------------|
| **Arithmetic** | `+`, `-`, `*`, `/`, `%` | Basic math operations |
| **Relational** | `==`, `!=`, `<`, `>`, `<=`, `>=` | Comparisons (return int: 1 true, 0 false) |
| **Logical** | `&&`, `||`, `!` | Boolean logic (short‑circuit) |
| **Bitwise** | `&`, `|`, `^`, `~`, `<<`, `>>` | Bit‑level operations |
| **Assignment** | `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=` | Assign values with optional operation |
| **Increment/Decrement** | `++`, `--` | Add/subtract 1 (pre/post) |
| **Conditional** | `? :` | Ternary operator (if‑else shorthand) |
| **Pointer** | `&` (address-of), `*` (dereference) | Work with memory addresses |
| **Sizeof** | `sizeof` | Get size in bytes (compile‑time) |
| **Comma** | `,` | Evaluate multiple expressions |
| **Cast** | `(type)` | Explicit type conversion |
| **Member Access** | `.`, `->` | Access structure/union members |
| **Subscript** | `[]` | Array indexing |
| **Function Call** | `()` | Call a function |

### Operator Precedence and Associativity

Operators have **precedence** (which operator is evaluated first) and **associativity** (left‑to‑right or right‑to‑left when precedence is equal).

#### Highest to Lowest Precedence

| Precedence | Operator | Description | Associativity |
|------------|----------|-------------|---------------|
| 1 (highest) | `()` `[]` `.` `->` | Function call, subscript, member access | Left‑to‑right |
| 2 | `++` `--` `+` `-` `!` `~` `*` `&` `(type)` `sizeof` | Unary operators | Right‑to‑left |
| 3 | `*` `/` `%` | Multiplication, division, modulus | Left‑to‑right |
| 4 | `+` `-` | Addition, subtraction | Left‑to‑right |
| 5 | `<<` `>>` | Shift | Left‑to‑right |
| 6 | `<` `<=` `>` `>=` | Relational | Left‑to‑right |
| 7 | `==` `!=` | Equality | Left‑to‑right |
| 8 | `&` | Bitwise AND | Left‑to‑right |
| 9 | `^` | Bitwise XOR | Left‑to‑right |
| 10 | `|` | Bitwise OR | Left‑to‑right |
| 11 | `&&` | Logical AND | Left‑to‑right |
| 12 | `||` | Logical OR | Left‑to‑right |
| 13 | `? :` | Conditional | Right‑to‑left |
| 14 | `=` `+=` `-=` `*=` `/=` `%=` `&=` `|=` `^=` `<<=` `>>=` | Assignment | Right‑to‑left |
| 15 (lowest) | `,` | Comma | Left‑to‑right |

### Important Rules
- **Parentheses** `()` override precedence – use them to make expressions clear.
- **Short‑circuit evaluation** – `&&` and `||` do not evaluate the right operand if the left operand determines the result.
- **Integer promotion** – `char` and `short` are promoted to `int` in expressions.
- **Usual arithmetic conversions** – for binary operators, operands are converted to a common type (e.g., `int` + `float` → `float`).

---

## 6. How It Works – Expression Evaluation

### Step‑by‑Step
1. **Lexical analysis** – operators are recognised as tokens.
2. **Parsing** – expression is parsed according to precedence and associativity.
3. **Type checking** – operands' types are checked for compatibility.
4. **Integer promotions** – `char`, `short` are promoted to `int`.
5. **Usual arithmetic conversions** – operands are converted to a common type.
6. **Evaluation** – the expression is evaluated:
   - For many operators, this happens at compile time if operands are constants.
   - For runtime, the machine code is generated.
7. **Result** – the final value and type are produced.

### ASCII Diagram – Expression Evaluation
```
Expression: a * b + c / d

Step 1: Parse according to precedence (* and / before +).
        ┌───────────────────────────────────────┐
        │              +                        │
        │         /     \                      │
        │      *          /                    │
        │     / \        / \                   │
        │    a   b      c   d                  │
        └───────────────────────────────────────┘

Step 2: Evaluate * (a * b) and / (c / d).
Step 3: Evaluate + (result1 + result2).
```

### Short‑Circuit Evaluation
```c
int a = 0, b = 10;
if (a != 0 && b / a > 0) {
    // b / a is NOT evaluated because a != 0 is false.
}
// This prevents division by zero.
```

### Side Effects
Some operators have **side effects** – they modify operands (e.g., `++`, `--`, assignment). The order of evaluation of side effects is not always guaranteed (especially in C89/90), leading to undefined behaviour.
```c
int i = 0;
int x = i++ + i++;   // ❌ Undefined behaviour – multiple side effects on i.
```

---

## 7. Internal Architecture – Operator Implementation

### Compiler Code Generation
- **Arithmetic operators** – compile to hardware instructions (e.g., `add`, `sub`, `mul`, `div`).
- **Bitwise operators** – compile to bitwise instructions (e.g., `and`, `or`, `xor`, `shl`, `shr`).
- **Relational operators** – compile to compare instructions (e.g., `cmp`, `test`), often followed by a conditional jump.
- **Logical operators** – compiled to conditional branches (short‑circuit).
- **Assignment** – compiled to store instructions (`mov`, `st`).
- **Increment/Decrement** – compiled to `inc`/`dec` instructions or `add`/`sub` with immediate 1.
- **`sizeof`** – evaluated at compile time (not present in machine code).

### Operator Overloading
- C does **not** support operator overloading (unlike C++). Each operator has a fixed meaning for built‑in types.

---

## 8. Lifecycle / Workflow of an Operator

1. **Written** – programmer uses an operator in source code.
2. **Parsed** – compiler determines precedence and associativity.
3. **Type checked** – operands are validated.
4. **Code generated** – appropriate machine instruction is emitted.
5. **Executed** – CPU performs the operation at runtime.

---

## 9. Practical Examples

### Example 1: Arithmetic and Integer Division
```c
#include <stdio.h>

int main(void) {
    int a = 17, b = 5;

    // Integer division truncates toward zero.
    int quotient = a / b;      // 3 (not 3.4)
    int remainder = a % b;     // 2

    // Negative division truncates toward zero (C99+).
    int neg_quot = -17 / 5;    // -3 (not -4)
    int neg_rem = -17 % 5;     // -2

    printf("%d / %d = %d, remainder %d\n", a, b, quotient, remainder);
    printf("-17 / 5 = %d, remainder %d\n", neg_quot, neg_rem);

    // Float division with casting.
    float f_result = (float)a / b;  // 3.4
    printf("(float)a / b = %f\n", f_result);

    return 0;
}
```
**Code Breakdown:**
- Integer division truncates toward zero (C99+). Before C99, it was implementation‑defined.
- Negative division truncates toward zero.
- Cast one operand to `float` to get floating‑point division.

### Example 2: Logical Operators and Short‑Circuit
```c
#include <stdio.h>

int main(void) {
    int x = 10, y = 0;

    // Short‑circuit AND: if left is false, right is NOT evaluated.
    int result1 = (x > 5) && (++y > 0);
    // x > 5 is true, so ++y is evaluated. y becomes 1.
    printf("Result1: %d, y: %d\n", result1, y); // 1, 1

    // Short‑circuit OR: if left is true, right is NOT evaluated.
    int result2 = (x < 5) || (++y > 0);
    // x < 5 is false, so ++y is evaluated. y becomes 2.
    printf("Result2: %d, y: %d\n", result2, y); // 1, 2

    // Short‑circuit prevents division by zero.
    int a = 0, b = 10;
    if (a != 0 && b / a > 0) {
        // This block is not entered, and division is skipped.
        printf("Division safe\n");
    }
    printf("No division by zero.\n");

    return 0;
}
```
**Code Breakdown:**
- `&&` and `||` evaluate left operand first.
- If the result is determined, the right operand is not evaluated.
- This is crucial for safety (e.g., checking `ptr != NULL` before dereferencing).

### Example 3: Bitwise Operators for Flags
```c
#include <stdio.h>

// Define bit flags
#define FLAG_READ  (1 << 0)  // 0b0001
#define FLAG_WRITE (1 << 1)  // 0b0010
#define FLAG_EXEC  (1 << 2)  // 0b0100

int main(void) {
    unsigned int permissions = 0;

    // Set flags: enable READ and WRITE
    permissions |= FLAG_READ;   // 0b0001
    permissions |= FLAG_WRITE;  // 0b0011

    printf("Permissions: 0x%X\n", permissions); // 0x3

    // Check if READ is set
    if (permissions & FLAG_READ) {
        printf("READ is enabled\n");
    }

    // Check if EXEC is set
    if (permissions & FLAG_EXEC) {
        printf("EXEC is enabled\n");
    } else {
        printf("EXEC is not enabled\n");
    }

    // Toggle WRITE flag
    permissions ^= FLAG_WRITE;  // 0b0001 (WRITE disabled)
    printf("After toggle WRITE: 0x%X\n", permissions);

    // Clear WRITE flag (disable)
    permissions &= ~FLAG_WRITE; // 0b0001
    printf("After clear WRITE: 0x%X\n", permissions);

    return 0;
}
```
**Code Breakdown:**
- Bitwise operators are used to manipulate individual bits (flags).
- `|=` sets a bit; `&=` with `~` clears a bit; `^=` toggles a bit.
- This is common in systems programming (file permissions, hardware registers).

### Example 4: Increment/Decrement in Loops
```c
#include <stdio.h>

int main(void) {
    // Pre‑increment in for loop – common pattern.
    for (int i = 0; i < 5; ++i) {
        printf("%d ", i);
    }
    printf("\n");

    // Post‑increment also works.
    for (int i = 0; i < 5; i++) {
        printf("%d ", i);
    }
    printf("\n");

    // Difference between pre and post in expressions.
    int x = 5;
    int y = x++;   // y = 5, x = 6
    int z = ++x;   // x = 7, z = 7
    printf("x: %d, y: %d, z: %d\n", x, y, z);

    return 0;
}
```
**Code Breakdown:**
- `++i` and `i++` have the same effect when used alone.
- In expressions, `i++` returns the old value; `++i` returns the new value.
- Pre‑increment can be slightly faster in C++ (but not in C); use whichever is clearer.

### Example 5: Conditional (Ternary) Operator
```c
#include <stdio.h>

int main(void) {
    int a = 10, b = 20;

    // Simple if‑else replacement.
    int max = (a > b) ? a : b;
    printf("Max: %d\n", max);

    // Using for min.
    int min = (a < b) ? a : b;
    printf("Min: %d\n", min);

    // Nested ternary (use sparingly).
    int x = 30, y = 20, z = 10;
    int max3 = (x > y) ? ((x > z) ? x : z) : ((y > z) ? y : z);
    printf("Max of three: %d\n", max3);

    // Assign based on condition.
    int value = 5;
    const char *status = (value > 0) ? "Positive" : "Non‑positive";
    printf("Value is %s\n", status);

    return 0;
}
```
**Code Breakdown:**
- `condition ? expr1 : expr2` – if condition is true, evaluates expr1; otherwise expr2.
- The operands must have compatible types.
- Nested ternary can be hard to read; use if‑else for complex logic.

### Example 6: Pointer Operators
```c
#include <stdio.h>

int main(void) {
    int value = 42;
    int *ptr = &value;      // & – address‑of operator
    int **ptr2 = &ptr;      // pointer to pointer

    printf("value: %d\n", value);
    printf("&value: %p\n", (void *)&value);
    printf("ptr: %p\n", (void *)ptr);
    printf("*ptr: %d\n", *ptr);     // * – dereference

    // Modify through pointer.
    *ptr = 100;
    printf("value after *ptr = 100: %d\n", value);

    // Pointer arithmetic.
    int arr[5] = {10, 20, 30, 40, 50};
    int *p = arr;   // points to arr[0]
    printf("*p: %d\n", *p);         // 10
    printf("*(p + 1): %d\n", *(p + 1)); // 20 (pointer arithmetic)

    // Null pointer check.
    int *null_ptr = NULL;
    if (null_ptr == NULL) {
        printf("Pointer is NULL\n");
    }

    return 0;
}
```
**Code Breakdown:**
- `&` – address‑of operator, returns the memory address.
- `*` – dereference operator, accesses the value at the address.
- Pointer arithmetic (`p + 1`) moves by `sizeof(*p)` bytes.
- Always check for `NULL` before dereferencing.

### Example 7: Comma Operator
```c
#include <stdio.h>

int main(void) {
    int a, b, c;

    // Comma operator: evaluate left, discard, evaluate right.
    // The value of the expression is the rightmost operand.
    int result = (a = 5, b = 10, c = 15, a + b + c);
    printf("result: %d\n", result); // 30

    // In for loop initialisation.
    for (int i = 0, j = 10; i < 5; i++, j--) {
        printf("i: %d, j: %d\n", i, j);
    }

    return 0;
}
```
**Code Breakdown:**
- The comma operator evaluates each expression from left to right.
- The result is the value of the last expression.
- Often used in `for` loops and for grouping multiple statements.

### Example 8: `sizeof` with Types and Expressions
```c
#include <stdio.h>

int main(void) {
    // sizeof(type) – returns size in bytes.
    printf("char: %zu\n", sizeof(char));
    printf("int: %zu\n", sizeof(int));
    printf("float: %zu\n", sizeof(float));
    printf("double: %zu\n", sizeof(double));
    printf("long long: %zu\n", sizeof(long long));
    printf("pointer: %zu\n", sizeof(void *));

    // sizeof(expression) – returns size of the expression's type.
    int x = 10;
    double d = 3.14;
    printf("sizeof(x): %zu\n", sizeof(x));        // sizeof(int)
    printf("sizeof(d): %zu\n", sizeof(d));        // sizeof(double)
    printf("sizeof(x + d): %zu\n", sizeof(x + d)); // sizeof(double) (promoted)

    // Array size.
    int arr[20];
    printf("sizeof(arr): %zu\n", sizeof(arr));        // 20 * sizeof(int)
    printf("sizeof(arr) / sizeof(arr[0]): %zu\n", 
           sizeof(arr) / sizeof(arr[0]));              // 20 (array length)

    return 0;
}
```
**Code Breakdown:**
- `sizeof` is a compile‑time operator (except for variable‑length arrays in C99).
- `sizeof` returns `size_t` (unsigned integer type).
- Use `%zu` to print `size_t`.
- `sizeof` on an array returns total bytes; divide by element size to get length.

---

## 10. Common Use Cases

| Operator Category | Common Use Cases | Example |
|-------------------|------------------|---------|
| **Arithmetic** | Calculations, counters | `sum += value;` |
| **Relational** | Comparisons in conditionals | `if (x > max) max = x;` |
| **Logical** | Combining conditions | `if (status == OK && ready)` |
| **Bitwise** | Flags, hardware registers | `flags |= ENABLE_BIT;` |
| **Assignment** | Storing values | `x = 5;` |
| **Increment/Decrement** | Loop counters | `for (int i = 0; i < n; i++)` |
| **Conditional** | Simple if‑else | `max = (a > b) ? a : b;` |
| **Pointer** | Dynamic memory, indirection | `*ptr = value;` |
| **Sizeof** | Memory allocation, array length | `malloc(sizeof(int) * n);` |
| **Comma** | Multiple expressions in one | `for (i=0, j=0; ...)` |

---

## 11. Best Practices

### General
- **Use parentheses** to make precedence explicit, even when not required.
- **Avoid undefined behaviour** – do not modify a variable multiple times in one expression.
- **Use `const`** for read‑only values to prevent accidental assignment.
- **Prefer `==` for equality** – a common typo is `=` (assignment) instead of `==`.
- **Use `size_t`** for `sizeof` results and array indices.

### Logical Operators
- **Use short‑circuit for safety** – check pointers before dereferencing:
  ```c
  if (ptr != NULL && *ptr == value) { ... }
  ```
- **Avoid unnecessary negation** – `if (!flag)` is clearer than `if (flag == 0)`.

### Bitwise Operators
- **Use `unsigned` types** for bitwise operations (signed shift is implementation‑defined).
- **Use named constants** (macros or enums) for bit flags.
- **Use parentheses** with shifts to avoid precedence issues:
  ```c
  if (flags & (1 << BIT_POS)) { ... }
  ```

### Increment/Decrement
- **Use pre‑increment or post‑increment consistently** – but be aware of the difference in expressions.
- **Do not use increment/decrement on the same variable multiple times in one expression** – undefined behaviour.

### Conditional Operator
- **Use for simple conditions only** – if‑else is clearer for complex logic.
- **Ensure operands have compatible types** to avoid unexpected conversions.

### Pointer Operators
- **Always initialise pointers** – to `NULL` if they don't point to valid memory.
- **Check for `NULL` before dereferencing**.
- **Use `const` for read‑only pointers** (`const int *ptr`).

---

## 12. Common Mistakes

### Mistake 1: Using `=` Instead of `==` in Conditionals
```c
// ❌ Wrong – assignment, not comparison.
if (x = 10) {   // assigns 10 to x, then checks if x is non‑zero (always true).
    // This will always execute.
}
// ✅ Correct – use ==.
if (x == 10) { ... }
```
**Why it's wrong:** `=` is assignment; it will modify `x` and the condition will be true if the assigned value is non‑zero.

### Mistake 2: Bitwise vs Logical Operators
```c
// ❌ Wrong – using & instead of &&.
if (x & 0 && y) { ... }  // Bitwise AND, not logical.
// ✅ Correct – use && for logical AND.
if (x && y) { ... }
```
**Why it's wrong:** `&` is bitwise, `&&` is logical; they have different semantics and precedence.

### Mistake 3: Integer Division Truncation
```c
// ❌ Wrong – expecting floating‑point division.
int a = 5, b = 2;
float result = a / b;   // result = 2.0 (integer division, then converted to float)
// ✅ Correct – cast to float.
float result = (float)a / b;  // 2.5
```

### Mistake 4: Operator Precedence Errors
```c
// ❌ Wrong – precedence of << is lower than +, so this is (x + 3) << 2?
int value = x + 3 << 2;  // Actually (x + 3) << 2, not x + (3 << 2).
// ✅ Correct – use parentheses.
int value = x + (3 << 2);
```

### Mistake 5: Multiple Side Effects (Undefined Behaviour)
```c
// ❌ Wrong – undefined behaviour.
int i = 0;
int x = i++ + i++;   // i is modified twice without a sequence point.
// ✅ Correct – avoid.
int i = 0;
int x = i + 1;
i += 2;
```

### Mistake 6: Forgetting Short‑Circuit
```c
// ❌ Wrong – may cause division by zero.
int a = 0, b = 10;
if (a != 0 && b / a > 0) { ... }  // This is safe (short‑circuit).
// But this is not:
int result = (a != 0) && (b / a > 0); // Safe because of short‑circuit.
// However:
int result = (a != 0) & (b / a > 0); // ❌ Uses bitwise &, no short‑circuit – division by zero!
```

### Mistake 7: Assuming `sizeof` Returns Int
```c
// ❌ Wrong – sizeof returns size_t, not int.
int size = sizeof(int);   // May warn about conversion.
// ✅ Correct – use size_t.
size_t size = sizeof(int);
```

---

## 13. Performance Considerations

- **Arithmetic operators** – compile to a single machine instruction in most cases (fast).
- **Bitwise operators** – also compile to single instructions (fast).
- **Division and modulus** – are slower than addition/multiplication (use shifts when possible).
- **Increment/decrement** – often compile to `inc`/`dec` (fast).
- **Short‑circuit** – can avoid expensive operations (beneficial).
- **`sizeof`** – compile‑time, no runtime overhead.
- **Conditional operator** – compiles to conditional move or branch; similar to if‑else.

---

## 14. Security Considerations

- **Integer overflow** – signed overflow is undefined; can be exploited.
- **Bitwise operations** – with signed types can have implementation‑defined behaviour; use unsigned.
- **Null pointer dereference** – check before dereferencing.
- **Type casts** – can hide mismatches; use with care.
- **Side effects** – in complex expressions can lead to undefined behaviour, which is exploitable.

---

## 15. Debugging Tips

- **Use parentheses** – to make intent clear and avoid precedence bugs.
- **Compile with warnings** – `-Wall -Wextra` catches many operator‑related issues.
- **Use `-Wparentheses`** – warns about ambiguous precedence (GCC/Clang).
- **Use `-Wsign-conversion`** – warns about sign changes in expressions.
- **Use static analysis** – tools like Clang Static Analyzer catch many operator bugs.
- **Debugger** – step through expressions to see evaluation order.

---

## 16. When to Use

- **All the time** – operators are fundamental to C programming.
- **Use the right operator** for the job – arithmetic for math, bitwise for flags, logical for conditions.

---

## 17. When Not to Use

- **Avoid obscure expressions** – don't try to be clever; clarity is more important than brevity.
- **Avoid using bitwise operators on signed integers** – use unsigned.
- **Avoid multiple side effects** in one expression – it's undefined behaviour.

---

## 18. Related Concepts

- **Expressions** – operators form the core of expressions.
- **Precedence and associativity** – rules governing evaluation order.
- **Type conversions** – implicit and explicit (casting).
- **Undefined behaviour** – common with operator misuse.
- **Sequence points** – points where side effects are guaranteed to be complete.
- **Short‑circuit evaluation** – logical operators.

---

## 19. Did You Know?

- The `sizeof` operator is a compile‑time operator (except for variable‑length arrays in C99).
- The comma operator has the lowest precedence of all operators.
- In C, the `+` operator can also be used as a unary operator (positive sign) – though rarely needed.
- The `?:` operator is right‑associative, so `a ? b : c ? d : e` is parsed as `a ? b : (c ? d : e)`.
- C has no exponentiation operator – use `pow()` from `<math.h>`.
- The `++` and `--` operators can be applied to pointers, not just integers.
- C23 introduces `typeof` and `constexpr` but doesn't add new operators.

---

## 20. Summary

- **Operators are symbols** that perform operations on operands (values/variables).
- **C has a rich set of operators**: arithmetic, relational, logical, bitwise, assignment, increment/decrement, conditional, pointer, sizeof, and more.
- **Operator precedence and associativity** determine evaluation order – use parentheses to make it explicit.
- **Short‑circuit evaluation** – `&&` and `||` stop evaluating once the result is determined.
- **Integer promotion** and **usual arithmetic conversions** occur in expressions.
- **Common mistakes**: using `=` instead of `==`, integer division, precedence errors, undefined behaviour with side effects.
- **Best practices**: use parentheses, avoid multiple side effects, use unsigned for bitwise ops, check for `NULL` before dereferencing.
- **Understanding operators** is fundamental to writing correct, efficient, and readable C code.