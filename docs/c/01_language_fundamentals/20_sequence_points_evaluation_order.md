# Sequence Points & Evaluation Order

## 1. Overview

### Definition
Sequence points are specific points in a C program's execution where all side effects of previous evaluations are guaranteed to be complete, and no side effects of subsequent evaluations have yet occurred. They define boundaries in the execution order where the compiler must commit all pending operations before moving on. The evaluation order of subexpressions in C is largely unspecified, but sequence points establish clear boundaries where the state of the program is well-defined.

### Purpose
Understanding sequence points and evaluation order is essential for:
- **Writing correct code** – avoiding undefined behaviour caused by modifying the same variable multiple times without a sequence point.
- **Understanding expression evaluation** – knowing when side effects (assignments, increments) are guaranteed to be applied.
- **Reading complex expressions** – understanding the order in which subexpressions are evaluated.
- **Writing portable code** – avoiding assumptions about evaluation order that vary across compilers.

### Where It Fits
Sequence points and evaluation order sit at the intersection of:
- **Expressions** – define how operators combine operands.
- **Operators** – some operators introduce sequence points (e.g., `&&`, `||`, `? :`, `,`).
- **Statements** – the end of a full expression is a sequence point.
- **Function calls** – function call boundaries are sequence points.
- **Undefined behaviour** – many common bugs involve violating sequence point rules.

---

## 2. Why It Exists

### The Problem Without Sequence Points
Without sequence points, the compiler would have complete freedom to reorder operations, leading to ambiguity. Consider:
```c
int i = 0;
int x = i++ + i++;
```
- Should `i` be incremented once or twice before the addition?
- Should the addition use the original value or the incremented value?
- Different compilers could produce different results, and the behaviour might even change with optimisation levels.

### The Solution: Sequence Points
Sequence points provide a contract between the programmer and the compiler:
- Within a sequence point, the compiler can reorder evaluations for optimisation.
- At a sequence point, all side effects must be complete.
- Between sequence points, the evaluation order is unspecified.

This gives the compiler freedom to optimise while ensuring that the programmer can reason about certain points in the execution.

### Why They Were Introduced
Sequence points were formalised in the C standard to define the boundaries of undefined behaviour. They allow optimising compilers to reorder operations while still providing a well-defined execution model. The rules are designed to balance performance (allowing reordering) with safety (defining clear boundaries where the state is consistent).

---

## 3. Syntax / Basic Usage

### Sequence Points in Common Constructs

```c
#include <stdio.h>

int main(void) {
    // Full expression statements: the semicolon is a sequence point.
    int x = 5;              // Sequence point at the end of the statement.
    x = x + 1;              // Sequence point at the end.

    // Function calls: sequence point before and after the call.
    printf("Hello\n");      // Sequence point at function call boundary.

    // Logical AND: sequence point between left and right operands.
    int a = 0, b = 10;
    if (a != 0 && b / a > 0) {   // Sequence point after evaluating (a != 0)
        // b / a is NOT evaluated because of short-circuit.
        printf("Division safe\n");
    }

    // Logical OR: sequence point between left and right operands.
    if (a == 0 || b / a > 0) {   // Sequence point after (a == 0)
        // b / a is NOT evaluated because a == 0 is true.
    }

    // Conditional operator: sequence point after condition.
    int result = (a > 0) ? (b = 5, a + b) : (b = 10, a + b);
    // Sequence point after evaluating (a > 0).

    // Comma operator: sequence point between each operand.
    int c = (a = 5, b = 10, a + b);  // Sequence points after a=5, after b=10.
    // c = 15

    // The &&, ||, ? :, and comma operators introduce sequence points.

    return 0;               // Sequence point at return.
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>

int main(void) {
    // A sequence point is a point where all side effects are complete.

    // 1. End of a full expression statement (semicolon).
    int x = 5;   // Sequence point here – x is guaranteed to be 5.
    x++;         // Sequence point here – x is guaranteed to be incremented.

    // 2. Function call (before and after the call).
    printf("%d", x);   // Sequence point before and after the call.
    // The arguments are evaluated before the call, and the call itself
    // is a sequence point.

    // 3. Logical AND (&&) – sequence point between left and right.
    int a = 0, b = 10;
    if (a != 0 && b / a > 0) {
        // Sequence point after evaluating (a != 0).
        // If (a != 0) is false, the right operand (b / a) is NOT evaluated.
        // This prevents division by zero.
    }

    // 4. Logical OR (||) – sequence point between left and right.
    if (a == 0 || b / a > 0) {
        // Sequence point after evaluating (a == 0).
        // If (a == 0) is true, the right operand is NOT evaluated.
    }

    // 5. Conditional operator (? :) – sequence point after condition.
    int result = (a > 0) ? (b = 5, a + b) : (b = 10, a + b);
    // Sequence point after evaluating (a > 0).
    // Only the selected expression is evaluated.

    // 6. Comma operator (,) – sequence point between operands.
    int c = (a = 5, b = 10, a + b);
    // Sequence point after a = 5, after b = 10.
    // The comma operator evaluates left to right, with sequence points.

    // 7. End of a function (return).
    return 0;   // Sequence point at return – side effects are complete.
}
```

---

## 4. Mental Model – Sequence Points as Checkpoints

Imagine a factory assembly line:

- **Operations (subexpressions)** are like workers performing tasks.
- **Sequence points** are like quality checkpoints along the line.
- Before a checkpoint, workers can perform tasks in any order (the compiler is free to reorder for efficiency).
- At the checkpoint, all tasks must be complete. The state of the product is fully committed.
- After the checkpoint, the next set of tasks can begin.

If you try to modify the same part (variable) multiple times between two checkpoints, the order of modifications is ambiguous, and the final state might be unpredictable (undefined behaviour).

### Example
```
Expression: x = i++ + i++;

Checkpoints:
   │
   ▼
┌─────────────────────────────────────────────┐
│    i++    │   +    │    i++    │   =    │   ;  │
│  (worker) │        │  (worker) │        │ (seq)│
└─────────────────────────────────────────────┘
   │                      │
   └─────No sequence──────┘
         point between
         them.

Behaviour is UNDEFINED – the compiler can evaluate the two i++ operations
in any order, and i is modified twice without a sequence point.
```

---

## 5. Core Concepts

### Definition of a Sequence Point
A sequence point is a point in the execution sequence where:
- All side effects of previous evaluations are complete.
- No side effects of subsequent evaluations have occurred.

### Where Sequence Points Occur

| Context | Example | Sequence Point Location |
|---------|---------|-------------------------|
| **End of a full expression** | `x = 5;` | After the semicolon |
| **`&&` operator** | `a && b` | Between `a` and `b` |
| **`||` operator** | `a || b` | Between `a` and `b` |
| **`? :` operator** | `c ? a : b` | After `c` (only one branch evaluated) |
| **Comma operator** | `a, b` | Between `a` and `b` |
| **Function call** | `f(a, b)` | Before and after the call (arguments evaluated before) |
| **Return statement** | `return x;` | At the return |
| **Initialisation** | `int x = 5;` | After the initialiser |
| **End of a statement** | `do { ... } while (c);` | At the `;` |
| **End of a loop body** | `for (;;) { ... }` | At the end of the body |
| **`switch` expression** | `switch (x) { ... }` | After evaluating `x` |

### Evaluation Order (Unspecified or Defined)

| Construct | Evaluation Order |
|-----------|------------------|
| **Function arguments** | Unspecified (e.g., `f(a, b)` may evaluate `a` then `b`, or `b` then `a`). |
| **Operands of most operators** | Unspecified (e.g., `a + b` may evaluate `a` then `b`, or `b` then `a`). |
| **`&&`, `||`, `,`, `?:`** | Left‑to‑right with sequence points. |
| **Function calls** | Arguments evaluated before the call (unspecified order). |

### Undefined Behaviour
Behaviour is undefined if:
- A variable is modified more than once between two sequence points.
- A variable is modified and also accessed (for its value) between two sequence points, *unless* the access is used to determine the new value (e.g., `x = x + 1` is allowed because `x` is read and written, but the read is used to compute the new value).

### Examples of Undefined Behaviour
```c
// ❌ Undefined – i modified twice.
i = i++ + i++;

// ❌ Undefined – i modified and accessed (multiple reads without sequence point).
int x = i++ + i;

// ❌ Undefined – i modified in function argument and in statement.
printf("%d %d", i++, i++);

// ✅ Defined – i modified once between sequence points.
i = i + 1;

// ✅ Defined – comma operator introduces sequence points.
i = (i = 5, i + 3);   // Sequence point between (i=5) and (i+3)
```

---

## 6. How It Works – Compiler Handling

### Compiler Optimisation and Reordering
1. **Parsing** – the compiler builds an AST of the expression.
2. **Sequence point identification** – the compiler identifies where sequence points occur.
3. **Reordering** – between sequence points, the compiler is free to reorder subexpression evaluations for optimisation.
4. **Side effect commitment** – at a sequence point, all pending side effects (assignments, increments) must be committed.
5. **Undefined behaviour detection** – if a variable is modified multiple times between sequence points, the compiler does not need to diagnose it (but it's undefined).

### Example of Compiler Reordering
```c
int a = 5, b = 10;
int c = a + b;
```
- The compiler may evaluate `a` then `b`, or `b` then `a`. Both are valid.
- The result is the same, so reordering doesn't matter.

```c
int i = 0;
int x = i++ + ++i;   // Undefined – compiler may produce different results.
```
- The compiler may evaluate the left `i++` first, then `++i`, or vice versa.
- The result depends on the evaluation order, so behaviour is undefined.

### Sequence Points in `&&` and `||`
```c
// && (AND) – left-to-right with short-circuit.
if (a != 0 && b / a > 0) {
    // Sequence point after (a != 0).
    // If a != 0 is false, b / a is not evaluated.
    // This is safe.
}
```
- The compiler must evaluate `a != 0` first.
- There is a sequence point after that evaluation.
- If `a != 0` is false, the right operand is not evaluated.
- This is essential for safety checks.

---

## 7. Internal Architecture – Sequence Points in the AST

### AST Representation
Each operator node in the AST has information about whether it introduces a sequence point.

```
Expression: (a > 0) ? (b = 5, a + b) : (b = 10, a + b)

AST:
        ┌───────────────────────────────┐
        │        Conditional (?:)        │
        │          (seq point)           │
        │          /      |      \       │
        │       cond    expr1    expr2   │
        │     (a > 0)  (comma)  (comma)  │
        │                │         │     │
        │              ...       ...     │
        └───────────────────────────────┘
```

- The `?:` operator has a sequence point after evaluating the condition.
- The comma operator has sequence points between each operand.

### Side Effect Tracking
The compiler tracks side effects (assignments, increments) and ensures they are completed at sequence points. This is part of the compiler's **data flow analysis**.

---

## 8. Lifecycle / Workflow of Sequence Point Evaluation

1. **Expression parsing** – the compiler identifies all operators and operands.
2. **Sequence point identification** – the compiler notes where sequence points occur.
3. **Subexpression evaluation** – the compiler evaluates subexpressions (order is unspecified unless sequence points dictate otherwise).
4. **Side effect commitment** – at each sequence point, all side effects are applied.
5. **Result computation** – the final value is produced.
6. **Program continues** – execution proceeds to the next statement.

---

## 9. Practical Examples

### Example 1: Sequence Points in Logical Operators
```c
#include <stdio.h>

int main(void) {
    int a = 0, b = 10;

    // Logical AND – sequence point after left operand.
    if (a != 0 && b / a > 0) {
        // The right operand (b / a) is NOT evaluated because a != 0 is false.
        // This prevents division by zero.
        printf("Both true\n");
    }

    // Logical OR – sequence point after left operand.
    int c = 5;
    if (c == 0 || (c = 10) > 5) {
        // The right operand (c = 10) is NOT evaluated because c == 0 is false.
        // So c remains 5.
        printf("c = %d\n", c);  // prints 5
    }

    // Another example: checking a pointer before dereferencing.
    int *ptr = NULL;
    if (ptr != NULL && *ptr == 10) {
        // ptr != NULL is false, so *ptr is NOT evaluated.
        // This prevents a null pointer dereference.
        printf("Value is 10\n");
    }

    return 0;
}
```
**Code Breakdown:**
- `&&` and `||` introduce sequence points and short‑circuit evaluation.
- This is crucial for safety – checking for null pointers before dereferencing, or checking for division by zero.

### Example 2: Sequence Points in the Comma Operator
```c
#include <stdio.h>

int main(void) {
    int a, b, c;

    // Comma operator – sequence point between each operand.
    // Evaluates left to right, discarding left values.
    int result = (a = 5, b = 10, c = 15, a + b + c);

    // Sequence points after a=5, after b=10, after c=15.
    // a, b, c are guaranteed to be set before the addition.
    printf("result: %d\n", result); // 30

    // Using comma in a for loop.
    for (int i = 0, j = 10; i < 5; i++, j--) {
        // Sequence point after i++, after j-- (due to comma).
        printf("i: %d, j: %d\n", i, j);
    }

    // Comma operator with side effects.
    int x = 0;
    int y = (x = 5, x + 3);   // x = 5, then x + 3 = 8
    // Sequence point after x = 5, so x is guaranteed to be 5.
    printf("x: %d, y: %d\n", x, y); // 5, 8

    return 0;
}
```
**Code Breakdown:**
- The comma operator evaluates left to right, with sequence points between operands.
- The value of the expression is the rightmost operand.
- It's often used in `for` loops and macros.

### Example 3: Undefined Behaviour – Multiple Modifications
```c
#include <stdio.h>

int main(void) {
    int i = 0;

    // ❌ Undefined behaviour – i modified twice without sequence point.
    i = i++ + i++;   // Which i++ happens first? The result is undefined.

    // ❌ Undefined behaviour – i modified and read multiple times.
    int x = i++ + i;

    // ❌ Undefined behaviour – i modified in function arguments.
    printf("%d %d %d\n", i++, i++, i++);

    // ❌ Undefined behaviour – i modified in assignment and increment.
    i = (i++);

    // ❌ Undefined behaviour – i modified in both operands of assignment.
    (i = 5) = i++;

    // ❌ Undefined behaviour – i modified multiple times in a statement.
    i = ++i + i++;

    // ✅ Defined – i modified once between sequence points.
    i = i + 1;    // Sequence point at end of statement.

    // ✅ Defined – i modified in assignment, but only once.
    i = 5;

    // ✅ Defined – comma operator introduces sequence points.
    i = (i = 5, i + 3);   // Sequence point after i = 5.

    return 0;
}
```
**Code Breakdown:**
- The problematic examples have undefined behaviour because `i` is modified more than once between sequence points.
- The compiler is not required to diagnose this; it's the programmer's responsibility.
- The results can vary between compilers, optimisation levels, and even different runs.

### Example 4: Sequence Points and Function Calls
```c
#include <stdio.h>

int global = 0;

int increment(int val) {
    global++;
    return val + 1;
}

int main(void) {
    // Function call – sequence point before and after the call.
    int x = increment(5);   // global becomes 1, x = 6

    // Arguments evaluation order is UNSPECIFIED.
    // This is dangerous if the arguments have side effects.
    int a = 0, b = 0;
    // The order of evaluation of function arguments is unspecified.
    // So this may increment a first, or b first.
    int result = increment(a++) + increment(b++);
    // The result depends on the evaluation order.

    printf("a: %d, b: %d, result: %d\n", a, b, result);

    // To avoid ambiguity, evaluate arguments separately.
    a = 0; b = 0;
    int val1 = increment(a++);
    int val2 = increment(b++);
    int safe_result = val1 + val2;   // Defined behaviour.
    printf("safe_result: %d\n", safe_result);

    return 0;
}
```
**Code Breakdown:**
- Function call boundaries are sequence points.
- However, the order of evaluation of function arguments is **unspecified**.
- If arguments have side effects (e.g., `a++`), the result is undefined.
- Always evaluate arguments separately if they have side effects.

### Example 5: Sequence Points in the Conditional Operator
```c
#include <stdio.h>

int main(void) {
    int a = 5, b = 10;

    // Conditional operator – sequence point after the condition.
    // Only the selected branch is evaluated.
    int result = (a > b) ? (a = 100, a) : (b = 200, b);

    // Sequence point after (a > b).
    // If a > b is true, (a = 100, a) is evaluated.
    // If false, (b = 200, b) is evaluated.
    // The other branch is NOT evaluated.

    printf("a: %d, b: %d, result: %d\n", a, b, result);
    // a: 5, b: 200, result: 200

    // Using conditional operator with side effects.
    int x = 0, y = 0;
    int max = (x > y) ? (x++) : (y++);
    // Sequence point after (x > y).
    // Only the selected branch modifies x or y.
    printf("x: %d, y: %d, max: %d\n", x, y, max); // 0, 1, 0

    return 0;
}
```
**Code Breakdown:**
- The `? :` operator has a sequence point after the condition.
- Only the selected expression is evaluated; the other is not.
- This can be used to conditionally execute side effects.

### Example 6: Sequence Points in `for` Loops
```c
#include <stdio.h>

int main(void) {
    // For loop structure:
    // for (initialisation; condition; increment) { body }

    // Sequence points:
    // 1. After initialisation
    // 2. After condition (before body)
    // 3. After body (before increment)
    // 4. After increment (before next condition check)

    for (int i = 0; i < 3; i++) {
        // Sequence point after the condition (i < 3).
        // Sequence point after the body.
        // Sequence point after the increment (i++).
        printf("i: %d\n", i);
    }

    // Using comma operator in for loop.
    int x, y;
    for (x = 0, y = 10; x < 3; x++, y--) {
        // Sequence points between x=0 and y=10 (comma).
        // Sequence points between x++ and y-- (comma).
        printf("x: %d, y: %d\n", x, y);
    }

    return 0;
}
```
**Code Breakdown:**
- `for` loops have multiple sequence points: after initialisation, after condition, after body, after increment.
- The comma operator within the `for` loop introduces additional sequence points.

---

## 10. Common Use Cases

| Scenario | Use of Sequence Points |
|----------|------------------------|
| **Safety checks** | `if (ptr != NULL && *ptr == 10)` |
| **Division by zero** | `if (denominator != 0 && num / denom > 0)` |
| **Conditional assignments** | `max = (a > b) ? a : b;` |
| **Multiple operations in loops** | `for (i=0, j=10; i<n; i++, j--)` |
| **Function argument evaluation** | Evaluate arguments separately to avoid side‑effect issues. |
| **Macro safety** | Use comma operator or `do { ... } while (0)` for multi‑statement macros. |

---

## 11. Best Practices

### General
- **Never modify a variable more than once between sequence points** – it leads to undefined behaviour.
- **Never modify a variable and also read it (for a different purpose) between sequence points** – unless the read is used to determine the new value (e.g., `x = x + 1` is safe).
- **Use parentheses** – to make complex expressions clear, though parentheses do not introduce sequence points.
- **Prefer simple expressions** – break complex expressions into multiple statements to avoid sequence point issues.
- **Use commas carefully** – the comma operator introduces sequence points, but it can reduce readability.

### Function Calls
- **Evaluate arguments separately** if they have side effects – avoid `f(a++, b++)`.
- **Assume unspecified evaluation order** for function arguments – do not rely on a particular order.

### Logical Operators
- **Use `&&` and `||` for short‑circuit safety** – check conditions before performing potentially unsafe operations.
- **Be aware of short‑circuit** – the right operand is not evaluated if the left determines the result.

### Conditional Operator
- **Use `?:` for simple conditional expressions** – it has a sequence point after the condition.
- **Avoid side effects in the branches** – it can be confusing.

### For Loops
- **Use the comma operator carefully** in `for` loop initialisation and increment sections – it introduces sequence points.

---

## 12. Common Mistakes

### Mistake 1: Multiple Modifications Without Sequence Point
```c
// ❌ Wrong – undefined behaviour.
int i = 0;
int x = i++ + i++;   // i modified twice.

// ✅ Correct – use separate statements.
int i = 0;
int x = i + 1;
i += 2;
```

### Mistake 2: Using `++` in Function Arguments
```c
// ❌ Wrong – undefined behaviour (argument evaluation order unspecified).
printf("%d %d", i++, i++);

// ✅ Correct – evaluate arguments separately.
int a = i++;
int b = i++;
printf("%d %d", a, b);
```

### Mistake 3: Relying on Evaluation Order
```c
// ❌ Wrong – assumes left-to-right evaluation (not guaranteed).
int x = 5;
int y = (x = 10) + x;   // Result may be 20 or 15, depending on evaluation order.

// ✅ Correct – use separate statements.
int x = 5;
x = 10;
int y = x + x;   // y = 20 (defined).
```

### Mistake 4: Misunderstanding Short‑Circuit
```c
// ❌ Wrong – may cause division by zero if a is 0.
int a = 0, b = 10;
int result = (a != 0) & (b / a > 0);   // & is bitwise AND, no short‑circuit.

// ✅ Correct – use logical AND.
int result = (a != 0) && (b / a > 0);   // Short‑circuit prevents division.
```

### Mistake 5: Forgetting Sequence Points in Macros
```c
// ❌ Wrong – macro with side effects can cause undefined behaviour.
#define SQUARE(x) ((x) * (x))
int i = 5;
int result = SQUARE(i++);   // Expands to (i++) * (i++) – undefined!

// ✅ Correct – evaluate argument once.
#define SQUARE(x) ({ int _x = (x); _x * _x; })   // GCC extension.
// Or:
int temp = i++;
int result = temp * temp;
```

### Mistake 6: Assuming Parentheses Introduce Sequence Points
```c
// ❌ Wrong – parentheses don't introduce sequence points.
int i = 0;
int x = (i++ + i++);   // Still undefined (parentheses don't add sequence points).

// ✅ Correct – only certain constructs introduce sequence points.
```

---

## 13. Performance Considerations

- **Sequence points** – do not directly affect performance; they are a semantic constraint.
- **Reordering** – compilers can reorder operations between sequence points for optimisation.
- **Short‑circuit** – can improve performance by avoiding unnecessary evaluations.
- **Comma operator** – evaluating left‑to‑right may prevent certain optimisations.
- **Function calls** – sequence points at call boundaries can limit some optimisations.

---

## 14. Security Considerations

- **Undefined behaviour** – can lead to security vulnerabilities (e.g., buffer overflows, arbitrary code execution).
- **Short‑circuit** – essential for safe code (e.g., null pointer checks).
- **Function argument evaluation** – unspecified order can lead to unexpected behaviour if arguments have side effects.
- **Macro safety** – macros with side effects can cause undefined behaviour; wrap arguments in parentheses or use `do { ... } while (0)`.

---

## 15. Debugging Tips

- **Use `-Wall -Wextra`** – catches some sequence point‑related issues.
- **Use `-Wsequence-point`** (GCC) – warns about violations of sequence point rules.
- **Use static analysers** – tools like Clang Static Analyzer detect undefined behaviour.
- **Use `-O0`** – disable optimisation to preserve evaluation order during debugging.
- **Simplify expressions** – break complex expressions into multiple statements for clarity.
- **Use a debugger** – step through the code to see the actual evaluation order.

---

## 16. When to Use

- **Always be aware** – sequence points affect every expression you write.
- **Use short‑circuit** for safety checks.
- **Use separate statements** when you need a defined order of evaluation.

---

## 17. When Not to Use

- **Avoid relying on unspecified evaluation order** – it's not portable.
- **Avoid writing expressions that modify the same variable multiple times** – they are undefined.
- **Avoid using side effects in complex expressions** – it's error‑prone.

---

## 18. Related Concepts

- **Undefined behaviour** – sequence point violations are a common cause.
- **Expressions** – sequence points define evaluation boundaries.
- **Operators** – some operators introduce sequence points.
- **Statements** – the end of a full expression statement is a sequence point.
- **Side effects** – assignments, increments, function calls.
- **Short‑circuit evaluation** – `&&` and `||` introduce sequence points.
- **Order of evaluation** – unspecified vs. defined.

---

## 19. Did You Know?

- In C, the `&&` and `||` operators are the only operators that guarantee left‑to‑right evaluation with sequence points.
- The comma operator also guarantees left‑to‑right evaluation, but it's often overlooked.
- The conditional operator (`?:`) has a sequence point after the condition, but the order of evaluation of the two branches is unspecified.
- Function arguments are evaluated before the call, but the order of evaluation is unspecified.
- C++ has similar rules, but C++ also has additional sequence points (e.g., after evaluating the operands of `,` and `?:`).
- Sequence points are sometimes called "full expression points" or "order points."
- The C standard does not require the compiler to warn about sequence point violations; it's the programmer's responsibility.

---

## 20. Summary

- **Sequence points** are points in program execution where all side effects are complete.
- **They occur** at the end of full expressions, function calls, and certain operators (`&&`, `||`, `?:`, `,`).
- **Evaluation order** is unspecified for most operators – only `&&`, `||`, `?:`, `,`, and function calls have defined order.
- **Undefined behaviour** occurs when a variable is modified more than once between sequence points, or modified and accessed in an ambiguous way.
- **Common mistakes** include `i++ + i++`, modifying variables in function arguments, and relying on unspecified evaluation order.
- **Best practices**: avoid complex expressions with side effects, use short‑circuit for safety, and separate operations into multiple statements when needed.
- **Understanding sequence points** is essential for writing correct, portable, and safe C code.