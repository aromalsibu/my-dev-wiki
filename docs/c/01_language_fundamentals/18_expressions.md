# Expressions

## 1. Overview

### Definition
An expression in C is a combination of operators, operands (variables, constants, function calls), and parentheses that evaluates to a single value. Expressions are the fundamental building blocks of C programs – they perform computations, make decisions, and manipulate data. Every expression has a type and a value, and some expressions also have side effects.

### Purpose
Expressions are how you perform all computation in C. They allow you to:
- Calculate values (e.g., `x + y * 2`)
- Assign values (e.g., `x = 5`)
- Compare values (e.g., `x > y`)
- Call functions (e.g., `sqrt(25.0)`)
- Access memory (e.g., `*ptr` or `arr[i]`)
- Control flow (e.g., `x ? y : z`)

### Where It Fits
Expressions appear almost everywhere in C source code:
- **Statements** – `x = a + b;` (expression statement)
- **Conditionals** – `if (x > 0) { ... }`
- **Loops** – `for (int i = 0; i < n; i++)`
- **Function arguments** – `printf("%d", x + y);`
- **Initialisers** – `int x = 5 * 3;`
- **Return statements** – `return a + b;`

---

## 2. Why It Exists

### The Problem Without Expressions
Without expressions, you would need separate statements for every operation. Instead of writing:
```c
x = (a + b) * (c - d);
```
You might need something like:
```c
temp1 = a + b;
temp2 = c - d;
x = temp1 * temp2;
```
This would be verbose, inefficient, and unreadable.

### The Solution: Composable Expressions
Expressions allow you to combine operations in a natural, mathematical syntax. They are:
- **Concise** – write complex calculations in a single line.
- **Composable** – expressions can be nested inside other expressions.
- **Efficient** – compilers can optimise complex expressions into efficient machine code.
- **Flexible** – expressions can be of many types (integer, float, pointer, etc.).

### Why They Were Introduced
Expressions are a fundamental concept in all imperative and functional languages. C's expression syntax is influenced by ALGOL and BCPL, providing a powerful way to compose operations. C also includes many operators that are themselves expressions (assignment, increment, etc.), making the language both concise and expressive.

---

## 3. Syntax / Basic Usage

### Simple Expressions
```c
#include <stdio.h>

int main(void) {
    // Literal expressions
    42;           // integer literal expression
    3.14;         // float literal expression
    'A';          // character literal expression
    "Hello";      // string literal expression

    // Variable expressions
    int x = 10;
    double y = 3.14;
    x;            // expression using variable x
    y;            // expression using variable y

    // Arithmetic expressions
    int sum = x + 5;          // addition
    int diff = x - 3;          // subtraction
    int product = x * 2;       // multiplication
    int quotient = x / 4;      // division
    int remainder = x % 3;     // modulus

    // Relational expressions (return int: 1 true, 0 false)
    int is_equal = (x == 10);   // 1
    int is_greater = (x > 5);   // 1

    // Logical expressions
    int logical_and = (x > 5) && (x < 15);  // 1
    int logical_or = (x < 5) || (x > 8);    // 1
    int logical_not = !(x == 10);           // 0

    // Assignment expressions (value is assigned value)
    int a = 5;
    int b = (a = 10);   // b = 10, a = 10

    // Conditional (ternary) expression
    int max = (x > 15) ? x : 15;   // max = 15

    // Comma expression (evaluates left to right, value is rightmost)
    int result = (a = 5, b = 10, a + b);   // result = 15

    printf("sum: %d, diff: %d, product: %d, quotient: %d, remainder: %d\n",
           sum, diff, product, quotient, remainder);
    printf("is_equal: %d, is_greater: %d\n", is_equal, is_greater);
    printf("logical_and: %d, logical_or: %d, logical_not: %d\n",
           logical_and, logical_or, logical_not);
    printf("max: %d, result: %d\n", max, result);

    return 0;
}
```

### Function Call Expressions
```c
#include <stdio.h>
#include <math.h>

int add(int a, int b) {
    return a + b;
}

int main(void) {
    // Function call expression
    int sum = add(5, 3);           // 8

    // Library function call
    double root = sqrt(25.0);      // 5.0

    // Nested function calls
    double result = sqrt(add(9, 16)); // sqrt(25) = 5.0

    printf("sum: %d, root: %f, result: %f\n", sum, root, result);
    return 0;
}
```

### Pointer Expressions
```c
#include <stdio.h>

int main(void) {
    int value = 42;
    int *ptr = &value;          // address-of expression

    // Dereference expression
    int deref = *ptr;           // 42

    // Pointer arithmetic expression
    int arr[5] = {10, 20, 30, 40, 50};
    int *p = arr;               // points to arr[0]
    int val1 = *p;              // 10
    int val2 = *(p + 1);        // 20 (pointer arithmetic)

    // Array indexing expression (equivalent to *(arr + i))
    int val3 = arr[2];          // 30

    printf("value: %d, deref: %d, val1: %d, val2: %d, val3: %d\n",
           value, deref, val1, val2, val3);

    return 0;
}
```

### Complex Expressions with Parentheses
```c
#include <stdio.h>

int main(void) {
    int a = 10, b = 5, c = 2, d = 3;

    // Without parentheses – precedence matters.
    int result1 = a + b * c - d;          // 10 + 10 - 3 = 17

    // With parentheses – explicit grouping.
    int result2 = (a + b) * (c - d);      // 15 * -1 = -15

    // Nested parentheses.
    int result3 = ((a + b) * (c - d)) / 2; // -15 / 2 = -7 (integer division)

    printf("result1: %d, result2: %d, result3: %d\n",
           result1, result2, result3);

    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>

int main(void) {
    // An expression is any combination of operators and operands.

    // Simple expressions:
    42;           // Literal expression – evaluates to int 42.
    3.14;         // Literal expression – evaluates to double 3.14.
    'A';          // Character literal – evaluates to int 65 (ASCII).

    // Variable expressions:
    int x = 10;
    x;            // Evaluates to the current value of x (10).

    // Arithmetic expressions:
    int sum = x + 5;          // '+' is the addition operator.
    int diff = x - 3;         // '-' is the subtraction operator.
    int product = x * 2;      // '*' is the multiplication operator.
    int quotient = x / 4;     // '/' is the division operator (integer).
    int remainder = x % 3;    // '%' is the modulus operator.

    // Relational expressions – return int (1 for true, 0 for false).
    int is_equal = (x == 10);   // 1 (true)
    int is_greater = (x > 5);   // 1 (true)

    // Logical expressions – short‑circuit evaluation.
    int logical_and = (x > 5) && (x < 15);  // 1 (both true)
    int logical_or = (x < 5) || (x > 8);    // 1 (second true)
    int logical_not = !(x == 10);           // 0 (false)

    // Assignment expressions – the value is the assigned value.
    int a = 5;
    int b = (a = 10);   // a = 10, then b = 10 (value of assignment).

    // Conditional (ternary) expression – if condition true, eval expr1 else expr2.
    int max = (x > 15) ? x : 15;   // max = 15

    // Comma expression – evaluates left to right, value is rightmost.
    int result = (a = 5, b = 10, a + b);   // result = 15

    // Function call expressions – call a function, value is the return value.
    int sum_func = add(5, 3);   // add returns 8.

    // Pointer expressions:
    int value = 42;
    int *ptr = &value;          // & is the address‑of operator.
    int deref = *ptr;           // * is the dereference operator (42).

    // Array indexing expressions:
    int arr[5] = {10, 20, 30, 40, 50};
    int val = arr[2];           // 30 (equivalent to *(arr + 2)).

    return 0;
}
```

---

## 4. Mental Model – Expressions as Sentences

Think of an expression as a sentence in a mathematical language:

- **Operands** are the **nouns** – the values you're working with (variables, constants, function calls).
- **Operators** are the **verbs** – the actions you perform (+ for addition, * for multiplication, etc.).
- **Parentheses** are like **punctuation** – they group parts of the sentence to clarify meaning.

Just as you can build complex sentences from simple words, you can build complex expressions from simple operands and operators.

### Example
```
Expression:  (a + b) * (c - d)
             │       │   │       │
             └──┬───┘   └──┬───┘
             ┌─┴──┐      ┌─┴──┐
             a   + b    c   - d
             └──┬─────────┘
                │
                └───────────────┘
                           │
                       (result) * (result)
                           │
                         (final value)
```

The evaluation of an expression is like solving a mathematical equation – you follow the rules of precedence and associativity, just as you follow the order of operations in mathematics.

---

## 5. Core Concepts

### Expression Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **Constant expressions** | Value known at compile time | `42`, `3.14`, `'A'`, `sizeof(int)` |
| **Arithmetic expressions** | Mathematical operations | `a + b`, `x * y`, `10 / 3` |
| **Relational expressions** | Comparisons (true/false) | `x > y`, `a == b`, `c != 0` |
| **Logical expressions** | Boolean logic | `x && y`, `a || b`, `!flag` |
| **Assignment expressions** | Store a value | `x = 10`, `y += 5` |
| **Conditional expressions** | If‑then‑else | `x > 0 ? x : -x` |
| **Comma expressions** | Sequential evaluation | `(a = 5, b = 10, a + b)` |
| **Function call expressions** | Call a function | `sqrt(25.0)`, `printf("Hello")` |
| **Pointer expressions** | Address operations | `&x`, `*ptr`, `ptr + 1` |
| **Subscript expressions** | Array indexing | `arr[3]`, `matrix[i][j]` |
| **Member access expressions** | Struct/union members | `point.x`, `ptr->y` |

### Lvalues and Rvalues

- **Lvalue** (locator value) – an expression that represents a memory location that can be assigned to. Examples: variables, array elements, structure members, dereferenced pointers.
- **Rvalue** – an expression that represents a value, not a memory location. Examples: constants, arithmetic results, function return values.

```c
int x = 10;          // x is an lvalue, 10 is an rvalue.
int y = x;           // x is an rvalue (read its value), y is an lvalue.
x = x + 1;           // x on left is lvalue (assigned to), x on right is rvalue (read).
// 10 = x;           // ❌ Error – 10 is an rvalue, cannot be on left of assignment.
```

### Value Categories in C
- **Modifiable lvalue** – can be assigned to (not `const`).
- **Non‑modifiable lvalue** – `const` variable; cannot be assigned to.
- **Rvalue** – any expression that is not an lvalue.

### Expression Evaluation

#### Precedence and Associativity
- **Precedence** – determines which operator is applied first (e.g., `*` before `+`).
- **Associativity** – determines the order when operators have the same precedence (e.g., `a + b + c` is `(a + b) + c`).

#### Sequence Points
A **sequence point** is a point in the program where all side effects of previous expressions are guaranteed to be complete. Sequence points occur at:
- The end of a full expression (e.g., `x = 5;`).
- The `&&`, `||`, and `?:` operators (short‑circuit).
- Function call boundaries (arguments evaluated before call).
- The comma operator (between operands).
- The end of a `return` statement.

#### Order of Evaluation
- **Unspecified** – the order in which operands are evaluated is often unspecified (e.g., `f() + g()` – `f()` and `g()` may be evaluated in any order).
- **Undefined behaviour** – modifying the same variable multiple times without an intervening sequence point is undefined (e.g., `i++ + i++`).

---

## 6. How It Works – Expression Evaluation

### Step‑by‑Step
1. **Lexical analysis** – the expression is tokenised into operators and operands.
2. **Parsing** – the expression is parsed according to precedence and associativity, forming an **abstract syntax tree (AST)**.
3. **Type checking** – the types of operands are checked for compatibility.
4. **Integer promotions** – `char` and `short` are promoted to `int`.
5. **Usual arithmetic conversions** – operands are converted to a common type (e.g., `int` + `float` → `float`).
6. **Evaluation** – the AST is evaluated (either at compile time for constants, or at runtime).
7. **Side effects** – assignments, increments, etc., modify memory.
8. **Result** – the final value and type are produced.

### ASCII Diagram – Expression Parsing
```
Expression: a * b + c / d

Abstract Syntax Tree:
        ┌───────────────┐
        │       +       │
        │    /     \    │
        │   *       /   │
        │  / \     / \  │
        │ a   b   c   d │
        └───────────────┘

Evaluation (postorder traversal):
1. Evaluate a * b   → result1
2. Evaluate c / d   → result2
3. Add result1 + result2 → final
```

### Example with Side Effects
```c
int x = 5;
int y = 10;
int z = (x++, y++, x + y);   // Comma operator, sequence points.
// x = 6, y = 11, z = 17
```
1. `x++` – evaluated, side effect: x becomes 6.
2. `y++` – evaluated, side effect: y becomes 11.
3. `x + y` – evaluated (6 + 11 = 17).
4. The comma operator returns the value of the rightmost expression (17).
5. `z` is assigned 17.

---

## 7. Internal Architecture – Expression Compilation

### Abstract Syntax Tree (AST)
The compiler builds an AST representing the expression. Each node is an operator, and its children are operands.

### Code Generation
- **Constant expressions** – folded at compile time (constant propagation).
- **Arithmetic expressions** – generate machine instructions (`add`, `sub`, `mul`, `div`).
- **Logical expressions** – generate conditional branches (short‑circuit).
- **Assignment** – generates `mov` or `st` instructions.

### Optimisation
Compilers perform many optimisations on expressions:
- **Constant folding** – `3 + 5` → `8` at compile time.
- **Constant propagation** – `x = 5; y = x + 3;` → `y = 8;`.
- **Strength reduction** – `x * 2` → `x << 1`.
- **Common subexpression elimination** – avoid recomputing the same expression.
- **Dead code elimination** – remove code that doesn't affect the result.

---

## 8. Lifecycle / Workflow of an Expression

1. **Written** – programmer writes an expression in source code.
2. **Parsed** – compiler builds an AST based on precedence and associativity.
3. **Typed** – types of operands are determined and converted if necessary.
4. **Optimised** – compiler applies optimisations.
5. **Evaluated** – at runtime, the expression is evaluated by the CPU.
6. **Side effects** – if any (assignments, increments), they are applied.
7. **Result** – the final value is used (assigned, passed, returned, etc.).

---

## 9. Practical Examples

### Example 1: Arithmetic Expressions and Type Conversions
```c
#include <stdio.h>

int main(void) {
    // Integer arithmetic
    int a = 17, b = 5;
    int integer_div = a / b;       // 3 (truncates toward zero)
    int integer_mod = a % b;       // 2

    // Floating‑point arithmetic
    double c = 17.0, d = 5.0;
    double float_div = c / d;      // 3.4

    // Mixed type arithmetic (int + double → double)
    double mixed = a + c;          // 17 + 17.0 = 34.0

    // Type casting
    double cast_div = (double)a / b;  // 3.4
    int cast_int = (int)c / b;        // 3 (cast c to int: 17 / 5 = 3)

    printf("integer_div: %d, integer_mod: %d\n", integer_div, integer_mod);
    printf("float_div: %f\n", float_div);
    printf("mixed: %f\n", mixed);
    printf("cast_div: %f, cast_int: %d\n", cast_div, cast_int);

    return 0;
}
```
**Code Breakdown:**
- Integer division truncates toward zero (C99+).
- Mixed types are promoted: `int` + `double` → `double`.
- Use `(double)` cast to force floating‑point division.

### Example 2: Relational and Logical Expressions
```c
#include <stdio.h>

int main(void) {
    int x = 10, y = 20, z = 0;

    // Relational expressions return int (1 or 0).
    int cmp1 = x < y;          // 1
    int cmp2 = x > y;          // 0
    int cmp3 = x == 10;        // 1
    int cmp4 = y != 20;        // 0

    // Logical expressions with short‑circuit.
    // AND: if left is false, right is NOT evaluated.
    int logical_and = (x > y) && (z++ > 0);   // x > y is false, so z++ is skipped.
    printf("logical_and: %d, z: %d\n", logical_and, z); // 0, 0

    // OR: if left is true, right is NOT evaluated.
    int logical_or = (x < y) || (z++ > 0);    // x < y is true, so z++ is skipped.
    printf("logical_or: %d, z: %d\n", logical_or, z); // 1, 0

    // Using relational expressions in conditionals.
    if (x < y && y > 0) {
        printf("Both conditions true\n");
    }

    // Assigning the result of a comparison.
    int is_positive = (x > 0);   // 1
    printf("is_positive: %d\n", is_positive);

    return 0;
}
```
**Code Breakdown:**
- Relational operators return `int` (1 for true, 0 for false).
- `&&` and `||` short‑circuit – they don't evaluate the right operand if the result is determined by the left.
- This is useful for safety checks (e.g., `ptr != NULL && *ptr == value`).

### Example 3: Assignment Expressions
```c
#include <stdio.h>

int main(void) {
    int a, b, c;

    // Simple assignment – value is the assigned value.
    a = 10;          // a = 10

    // Chained assignment – right‑associative.
    b = c = a + 5;   // a + 5 = 15, c = 15, b = 15

    // Compound assignment (op=).
    int x = 10;
    x += 5;          // x = 15
    x -= 3;          // x = 12
    x *= 2;          // x = 24
    x /= 4;          // x = 6
    x %= 4;          // x = 2

    // Assignment in expressions.
    int y;
    if ((y = x * 2) > 10) {   // y = 4, then check if 4 > 10 (false)
        printf("y > 10\n");
    } else {
        printf("y <= 10\n");
    }

    printf("a: %d, b: %d, c: %d\n", a, b, c);
    printf("x: %d, y: %d\n", x, y);

    return 0;
}
```
**Code Breakdown:**
- Assignment is an expression – it has a value (the assigned value).
- Chained assignment is possible because `a = b = 5` is `a = (b = 5)`.
- Compound assignments combine operation and assignment in one step.
- Assignments can be used in conditionals (though this is often considered bad style).

### Example 4: Conditional (Ternary) Expressions
```c
#include <stdio.h>

int main(void) {
    int a = 10, b = 20;

    // Basic conditional expression.
    int max = (a > b) ? a : b;   // max = 20
    printf("max: %d\n", max);

    // Nested conditional (use sparingly).
    int x = 30, y = 20, z = 10;
    int max3 = (x > y) ? ((x > z) ? x : z) : ((y > z) ? y : z);
    printf("max of three: %d\n", max3);

    // Conditional expression in printf.
    printf("a is %s\n", (a > 5) ? "greater than 5" : "less than or equal to 5");

    // Assigning different types (need compatible types).
    int value = 5;
    const char *status = (value > 0) ? "Positive" : "Non‑positive";
    printf("status: %s\n", status);

    // Conditional expression returning a value.
    int abs_value = (value < 0) ? -value : value;
    printf("abs: %d\n", abs_value);

    return 0;
}
```
**Code Breakdown:**
- `condition ? expr1 : expr2` – if condition true, evaluate expr1; otherwise expr2.
- The operands must have compatible types.
- Nested ternary is hard to read; use if‑else for complex logic.
- Often used for simple conditional assignments.

### Example 5: Pointer Expressions and Array Indexing
```c
#include <stdio.h>

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};
    int *ptr = arr;          // ptr points to arr[0]

    // Array indexing (a[i] is equivalent to *(a + i)).
    int val1 = arr[2];       // 30
    int val2 = *(arr + 2);   // 30 (same as arr[2])

    // Pointer arithmetic.
    int val3 = *ptr;         // 10
    int val4 = *(ptr + 1);   // 20
    int val5 = ptr[2];       // 30 (pointer can be indexed like an array)

    // Pointer difference.
    int *ptr2 = &arr[4];     // points to arr[4]
    int diff = ptr2 - ptr;   // 4 (number of elements between them)

    // Address‑of and dereference.
    int value = 42;
    int *p = &value;         // p holds address of value
    int deref = *p;          // dereference p: 42
    *p = 100;                // modifies value through pointer

    printf("val1: %d, val2: %d, val3: %d, val4: %d, val5: %d\n",
           val1, val2, val3, val4, val5);
    printf("diff: %d, value: %d\n", diff, value);

    return 0;
}
```
**Code Breakdown:**
- Array indexing `a[i]` is equivalent to `*(a + i)`.
- Pointer arithmetic moves by `sizeof(*ptr)` bytes.
- Pointer difference returns the number of elements.
- `&` (address‑of) and `*` (dereference) are inverses.

### Example 6: Comma Expressions
```c
#include <stdio.h>

int main(void) {
    int a, b, c;

    // Comma operator: evaluate left, discard result, evaluate right.
    // The value of the expression is the rightmost operand.
    int result = (a = 5, b = 10, c = 15, a + b + c);
    printf("result: %d\n", result); // 30

    // Using comma in for loops.
    for (int i = 0, j = 10; i < 5; i++, j--) {
        printf("i: %d, j: %d\n", i, j);
    }

    // Multiple statements in one expression (not recommended for readability).
    int x = (printf("Hello, "), printf("World!\n"), 42);
    printf("x: %d\n", x); // prints "Hello, World!\n42"

    // Comma operator with assignment.
    int y = (a = 5), (b = 10);   // ❌ Error: comma operator in declarations is invalid.
    // Correct:
    int y = (a = 5, b = 10);     // y = 10

    return 0;
}
```
**Code Breakdown:**
- The comma operator evaluates left to right and discards left values.
- The result is the value of the rightmost expression.
- Often used in `for` loops for multiple initialisations and increments.
- Using the comma operator in declarations (e.g., `int a = 5, b = 10;`) is not the comma operator – it's a declaration of two variables.

### Example 7: sizeof Expressions
```c
#include <stdio.h>

int main(void) {
    int x = 10;
    double arr[20];

    // sizeof(type) – compile‑time constant.
    size_t size_int = sizeof(int);
    size_t size_double = sizeof(double);
    size_t size_arr = sizeof(arr);       // 20 * sizeof(double)

    // sizeof(expression) – size of the expression's type.
    size_t size_x = sizeof(x);
    size_t size_arr_elem = sizeof(arr[0]);
    size_t size_ptr = sizeof(int *);

    // sizeof on a variable‑length array (C99) – evaluated at runtime.
    int n = 10;
    int vla[n];
    size_t size_vla = sizeof(vla);       // n * sizeof(int) = 40 (runtime)

    // Common use: array length.
    size_t array_len = sizeof(arr) / sizeof(arr[0]); // 20

    printf("sizeof(int): %zu\n", size_int);
    printf("sizeof(arr): %zu\n", size_arr);
    printf("sizeof(arr) / sizeof(arr[0]): %zu\n", array_len);
    printf("sizeof(vla): %zu\n", size_vla);

    return 0;
}
```
**Code Breakdown:**
- `sizeof` is a compile‑time operator for fixed‑size types.
- For variable‑length arrays (C99), `sizeof` is evaluated at runtime.
- `sizeof` returns `size_t` – use `%zu` to print.
- Common pattern to get array length: `sizeof(arr) / sizeof(arr[0])`.

### Example 8: Function Call Expressions
```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int multiply(int a, int b) {
    return a * b;
}

int main(void) {
    // Simple function call.
    int sum = add(5, 3);        // 8

    // Nested function calls.
    int result = multiply(add(2, 3), add(4, 5));   // multiply(5, 9) = 45

    // Function call as part of an expression.
    int x = 10;
    int y = add(x, 5) * 2;     // (10 + 5) * 2 = 30

    // Functions can be called without using the return value.
    printf("Hello, World!\n"); // expression with side effect (printing)

    // Function pointer expression.
    int (*func_ptr)(int, int) = add;
    int z = func_ptr(10, 20);  // 30

    printf("sum: %d, result: %d, y: %d, z: %d\n", sum, result, y, z);

    return 0;
}
```
**Code Breakdown:**
- Function calls are expressions – they have a value (the return value).
- They can be nested and used as operands in other expressions.
- Functions with `void` return type can be used as expressions (but they have no value, only side effects).
- Function pointers can be used to call functions dynamically.

---

## 10. Common Use Cases

| Expression Type | Use Case | Example |
|-----------------|----------|---------|
| **Arithmetic** | Calculations | `area = PI * radius * radius;` |
| **Assignment** | Storing values | `x = 5;` |
| **Relational** | Comparisons | `if (x > max) max = x;` |
| **Logical** | Combining conditions | `if (status == OK && ready)` |
| **Conditional** | Simple if‑else | `max = (a > b) ? a : b;` |
| **Comma** | Multiple operations | `for (i=0, j=0; i<n; i++, j++)` |
| **Function call** | Reusable logic | `result = sqrt(25.0);` |
| **Pointer** | Memory access | `*ptr = value;` |
| **Subscript** | Array indexing | `arr[i] = 5;` |
| **Member access** | Struct/union | `p.x = 10;` |
| **`sizeof`** | Memory allocation | `malloc(sizeof(int) * n);` |
| **Cast** | Type conversion | `(double)a / b;` |

---

## 11. Best Practices

### General
- **Use parentheses** – to make precedence explicit and avoid bugs.
- **Avoid side effects in complex expressions** – multiple modifications without sequence points cause undefined behaviour.
- **Use meaningful variable names** – makes expressions self‑documenting.
- **Break complex expressions into multiple statements** – improves readability and debugging.
- **Use `const`** – for read‑only values to prevent accidental modification.

### Assignment
- **Use assignment as a statement** – not inside conditionals (unless necessary).
- **Initialise variables when declared** – avoids using uninitialised values.
- **Use compound assignment** (`+=`, `-=`, etc.) – more concise and less error‑prone.

### Comparisons
- **Use `==` for equality, not `=`** – a common bug.
- **Prefer `if (ptr != NULL)`** over `if (ptr)` – more explicit.
- **Be careful with floating‑point comparisons** – due to precision issues.

### Logical Operators
- **Use short‑circuit for safety** – check before dereferencing.
- **Avoid deep nesting of logical expressions** – use helper variables.

### Conditional Operator
- **Use for simple conditions only** – if‑else is clearer for complex logic.
- **Ensure operands have compatible types** – avoid unexpected conversions.

### sizeof
- **Use `sizeof` for array length** – `sizeof(arr) / sizeof(arr[0])`.
- **Use `sizeof` with `malloc`** – `malloc(sizeof(int) * n)`.
- **Always use `%zu`** to print `size_t`.

### Casts
- **Use casts sparingly** – they can hide bugs.
- **Use explicit casts when converting** – makes intent clear.

---

## 12. Common Mistakes

### Mistake 1: Using `=` Instead of `==`
```c
// ❌ Wrong – assignment, not comparison.
if (x = 10) {   // assigns 10 to x, condition is true (10 != 0).
    // Always executes.
}
// ✅ Correct – use ==.
if (x == 10) { ... }
```

### Mistake 2: Integer Division Instead of Float Division
```c
// ❌ Wrong – integer division truncates.
int a = 5, b = 2;
float result = a / b;   // result = 2.0 (not 2.5)
// ✅ Correct – cast to float.
float result = (float)a / b; // 2.5
```

### Mistake 3: Multiple Side Effects (Undefined Behaviour)
```c
// ❌ Wrong – undefined behaviour (modifying i twice without sequence point).
int i = 0;
int x = i++ + i++;   // Undefined.
// ✅ Correct – avoid.
int i = 0;
int x = i + 1;
i += 2;
```

### Mistake 4: Precedence Errors
```c
// ❌ Wrong – precedence of << is lower than +.
int x = 3;
int value = x + 5 << 2;   // Actually (x + 5) << 2, not x + (5 << 2).
// ✅ Correct – use parentheses.
int value = x + (5 << 2);
```

### Mistake 5: Forgetting Short‑Circuit
```c
// ❌ Wrong – may cause division by zero if a is 0.
int a = 0, b = 10;
if (a != 0 && b / a > 0) { ... }   // This is safe (short‑circuit).
// But this is NOT:
int result = (a != 0) & (b / a > 0); // ❌ Bitwise &, no short‑circuit – division by zero!
// ✅ Correct – use &&.
int result = (a != 0) && (b / a > 0);
```

### Mistake 6: Misunderstanding Lvalue/Rvalue
```c
// ❌ Wrong – can't assign to an rvalue.
10 = x;   // Error: 10 is an rvalue.
// ✅ Correct – lvalue on left.
x = 10;
```

### Mistake 7: Ignoring `sizeof` Return Type
```c
// ❌ Wrong – sizeof returns size_t, not int.
int size = sizeof(int);   // May warn about conversion.
// ✅ Correct – use size_t.
size_t size = sizeof(int);
```

### Mistake 8: Using `&` Instead of `&&`
```c
// ❌ Wrong – bitwise AND, not logical AND.
if (x & y) { ... }   // Bitwise AND of x and y.
// ✅ Correct – use && for logical AND.
if (x && y) { ... }
```

### Mistake 9: Confusing Precedence with `++` and `--`
```c
// ❌ Wrong – post‑increment returns old value.
int x = 5;
int y = x++;   // y = 5, x = 6
// If you expected y = 6, use pre‑increment.
// ✅ Correct – use pre‑increment.
int y = ++x;   // x = 6, y = 6
```

### Mistake 10: Using the Comma Operator Accidentally
```c
// ❌ Wrong – not the comma operator (it's a declaration).
int a = 5, b = 10;   // Declares two variables.

// ❌ Wrong – comma operator in a statement.
int x = (a = 5, b = 10);   // x = 10 (comma operator).
// If you intended to declare two variables, use:
int a = 5, b = 10;
```

---

## 13. Performance Considerations

- **Arithmetic expressions** – compile to efficient machine code (single instructions).
- **Division and modulus** – slower than addition/multiplication (use shifts when possible).
- **Short‑circuit** – can avoid expensive operations.
- **Constant folding** – compile‑time evaluation reduces runtime work.
- **`sizeof`** – compile‑time (except VLA), no runtime overhead.
- **Casts** – no runtime overhead in most cases (they just reinterpret bits).
- **Function calls** – have overhead; consider `inline` for small functions.
- **Pre‑increment vs post‑increment** – in C, both compile to similar code; use whichever is clearer.

---

## 14. Security Considerations

- **Integer overflow** – signed overflow is undefined, leading to vulnerabilities.
- **Division by zero** – check before division.
- **Null pointer dereference** – check before dereferencing.
- **Type casts** – can hide mismatches; use with care.
- **Side effects** – in complex expressions can lead to undefined behaviour, which is exploitable.
- **Short‑circuit** – use to prevent unsafe operations (e.g., checking `ptr != NULL` before dereferencing).

---

## 15. Debugging Tips

- **Use parentheses** – to make intent clear and avoid precedence bugs.
- **Compile with warnings** – `-Wall -Wextra` catches many expression‑related issues.
- **Use `-Wparentheses`** – warns about ambiguous precedence (GCC/Clang).
- **Use `-Wsign-conversion`** – warns about sign changes in expressions.
- **Use `-Wconversion`** – warns about implicit conversions that may lose information.
- **Use static analysis** – tools like Clang Static Analyzer catch many expression bugs.
- **Debugger** – step through expressions to see evaluation order and values.
- **Use `printf` debugging** – print intermediate values to understand complex expressions.

---

## 16. When to Use

- **All the time** – expressions are fundamental to C programming.
- **Use the right expression** for the job – arithmetic for math, relational for comparisons, logical for conditions.

---

## 17. When Not to Use

- **Avoid overly complex expressions** – break them down for clarity.
- **Avoid expressions with side effects** in conditionals (e.g., `if (x = y)`) – it's confusing.
- **Avoid expressions that modify the same variable multiple times** – undefined behaviour.
- **Avoid unnecessary casts** – they can hide bugs.

---

## 18. Related Concepts

- **Operators** – the building blocks of expressions.
- **Precedence and associativity** – rules for expression evaluation.
- **Type conversions** – implicit and explicit (casting).
- **Lvalues and rvalues** – value categories.
- **Sequence points** – points where side effects are guaranteed.
- **Undefined behaviour** – common with expression misuse.
- **Statements** – expressions are often part of statements.
- **Function calls** – a type of expression.

---

## 19. Did You Know?

- In C, an assignment expression's value is the assigned value, allowing `a = b = 5;`.
- The comma operator has the lowest precedence of all operators.
- `sizeof` is a compile‑time operator except for variable‑length arrays (C99).
- Function calls are expressions and can be used anywhere an expression is allowed.
- The `?:` operator is right‑associative: `a ? b : c ? d : e` → `a ? b : (c ? d : e)`.
- In C, the `&&` and `||` operators evaluate left to right and short‑circuit.
- C23 introduces `typeof` and `constexpr`, but they don't change expression semantics.
- The order of evaluation of operands is **unspecified** (e.g., `f() + g()` – `f()` and `g()` may be evaluated in any order).

---

## 20. Summary

- **An expression is a combination of operators and operands** that evaluates to a value.
- **Every expression has a type and a value**; some also have side effects.
- **Lvalues represent memory locations** (can be assigned to); **rvalues are pure values**.
- **Precedence and associativity** determine evaluation order – use parentheses for clarity.
- **Sequence points** ensure side effects are complete at certain points.
- **Common types of expressions**: arithmetic, relational, logical, assignment, conditional, comma, function call, pointer, subscript, member access, sizeof, cast.
- **Common mistakes**: using `=` instead of `==`, integer division, multiple side effects, precedence errors, forgetting short‑circuit, misunderstanding lvalues/rvalues.
- **Best practices**: use parentheses, avoid side effects in complex expressions, use `sizeof` for array length, break complex expressions into multiple statements.
- **Understanding expressions** is fundamental to writing correct, efficient, and readable C code – they are the verbs of the language.