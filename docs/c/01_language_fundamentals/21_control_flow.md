# Control Flow

## 1. Overview

### Definition
Control flow in C refers to the order in which individual statements are executed. By default, statements execute sequentially, but C provides constructs that alter this flow: **selection** (conditional branching), **iteration** (loops), and **jump** statements. Together, they allow you to direct the program's execution path based on conditions, repeat blocks of code, and transfer control to different parts of the program.

### Purpose
Control flow is what makes programs dynamic. Without it, every program would be a straight‑line sequence of fixed operations, incapable of reacting to input, making decisions, or repeating tasks. Control flow enables:
- **Decision making** – performing different actions based on conditions (`if`, `switch`).
- **Repetition** – executing code multiple times (`for`, `while`, `do-while`).
- **Early exits** – breaking out of loops or functions (`break`, `return`, `goto`).

### Where It Fits
Control flow is the backbone of procedural programming. It sits at the heart of every function, determining which statements run and in what order. All other concepts (variables, expressions, functions) serve to support the control flow of the program.

---

## 2. Why It Exists

### The Problem Without Control Flow
Without control flow, programs would be purely linear scripts. They could not:
- Choose between different paths based on user input.
- Repeat operations (e.g., processing lists).
- Handle errors gracefully.
- Implement complex algorithms.

### The Solution: Structured Control Flow
C provides structured control flow constructs that are:
- **Composable** – can be nested inside each other.
- **Predictable** – behaviour is well‑defined.
- **Efficient** – compile to conditional jumps and loops in machine code.
- **Readable** – clear syntax that matches common algorithmic patterns.

### Why It Was Introduced
C's control flow constructs are inherited from ALGOL and BCPL, representing a significant improvement over the unstructured `goto`‑heavy code of early languages. They encourage structured programming, making code easier to understand, maintain, and prove correct.

---

## 3. Syntax / Basic Usage

### Selection Statements

#### `if` and `if-else`
```c
#include <stdio.h>

int main(void) {
    int score = 85;

    // Simple if
    if (score >= 90) {
        printf("Grade: A\n");
    }

    // if-else
    if (score >= 50) {
        printf("Pass\n");
    } else {
        printf("Fail\n");
    }

    // if-else if ladder
    if (score >= 90) {
        printf("A\n");
    } else if (score >= 80) {
        printf("B\n");
    } else if (score >= 70) {
        printf("C\n");
    } else if (score >= 60) {
        printf("D\n");
    } else {
        printf("F\n");
    }

    // Nested if
    int x = 5, y = 10;
    if (x > 0) {
        if (y > 0) {
            printf("Both positive\n");
        } else {
            printf("x positive, y non-positive\n");
        }
    } else {
        printf("x non-positive\n");
    }

    return 0;
}
```

#### `switch`
```c
#include <stdio.h>

int main(void) {
    int day = 3;

    switch (day) {
        case 1:
            printf("Monday\n");
            break;
        case 2:
            printf("Tuesday\n");
            break;
        case 3:
            printf("Wednesday\n");
            break;
        case 4:
            printf("Thursday\n");
            break;
        case 5:
            printf("Friday\n");
            break;
        case 6:
            printf("Saturday\n");
            break;
        case 7:
            printf("Sunday\n");
            break;
        default:
            printf("Invalid day\n");
            break;
    }

    // Fallthrough example (multiple cases without break)
    int month = 2;
    switch (month) {
        case 1: case 3: case 5: case 7: case 8: case 10: case 12:
            printf("31 days\n");
            break;
        case 4: case 6: case 9: case 11:
            printf("30 days\n");
            break;
        case 2:
            printf("28 or 29 days\n");
            break;
        default:
            printf("Invalid month\n");
            break;
    }

    return 0;
}
```

### Iteration Statements

#### `while` Loop
```c
#include <stdio.h>

int main(void) {
    int i = 0;

    // while loop – checks condition before each iteration
    while (i < 5) {
        printf("while: %d\n", i);
        i++;
    }

    // Infinite loop with break
    int count = 0;
    while (1) {
        if (count >= 5) {
            break;
        }
        printf("infinite while: %d\n", count);
        count++;
    }

    return 0;
}
```

#### `do-while` Loop
```c
#include <stdio.h>

int main(void) {
    int i = 0;

    // do-while – executes body at least once
    do {
        printf("do-while: %d\n", i);
        i++;
    } while (i < 5);

    // Example: read input until valid
    int value;
    do {
        printf("Enter a positive number: ");
        scanf("%d", &value);
    } while (value <= 0);

    printf("You entered: %d\n", value);

    return 0;
}
```

#### `for` Loop
```c
#include <stdio.h>

int main(void) {
    // Basic for loop
    for (int i = 0; i < 5; i++) {
        printf("for: %d\n", i);
    }

    // Multiple initialisations and increments
    for (int i = 0, j = 10; i < 5; i++, j--) {
        printf("i: %d, j: %d\n", i, j);
    }

    // Empty parts (all optional)
    int k = 0;
    for (; k < 5; ) {
        printf("k: %d\n", k);
        k++;
    }

    // Infinite loop
    for (;;) {
        static int loop_count = 0;
        if (loop_count >= 3) {
            break;
        }
        printf("infinite for\n");
        loop_count++;
    }

    // Nested for loops (e.g., 2D array)
    for (int row = 0; row < 3; row++) {
        for (int col = 0; col < 4; col++) {
            printf("[%d][%d] ", row, col);
        }
        printf("\n");
    }

    return 0;
}
```

### Jump Statements

#### `break` and `continue`
```c
#include <stdio.h>

int main(void) {
    // break – exits the nearest loop or switch
    printf("break: ");
    for (int i = 0; i < 10; i++) {
        if (i == 5) {
            break;           // exits loop when i == 5
        }
        printf("%d ", i);
    }
    printf("\n");

    // continue – skips rest of iteration
    printf("continue: ");
    for (int i = 0; i < 5; i++) {
        if (i == 2) {
            continue;        // skips printing 2
        }
        printf("%d ", i);
    }
    printf("\n");

    // break in nested loops – only breaks inner loop
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (i == 1 && j == 1) {
                break;       // only breaks inner loop
            }
            printf("[%d][%d] ", i, j);
        }
        printf("\n");
    }

    return 0;
}
```

#### `goto`
```c
#include <stdio.h>
#include <stdlib.h>   // for malloc, free

int main(void) {
    // Simple forward jump
    int x = -5;
    if (x < 0) {
        goto error;          // jump to error label
    }
    printf("No error\n");
    return 0;

    error:                   // label
    printf("Error occurred\n");
    return 1;

    // Example: error cleanup with goto
    FILE *file = NULL;
    int *data = NULL;

    file = fopen("data.txt", "r");
    if (file == NULL) {
        goto cleanup;
    }

    data = (int *)malloc(100 * sizeof(int));
    if (data == NULL) {
        goto cleanup;
    }

    // ... use file and data

    cleanup:
    if (data != NULL) {
        free(data);
    }
    if (file != NULL) {
        fclose(file);
    }

    return 0;
}
```

#### `return`
```c
#include <stdio.h>

// Function with early return
int max(int a, int b) {
    if (a > b) {
        return a;    // early exit
    }
    return b;        // else return b
}

// void function with early return
void check_positive(int x) {
    if (x <= 0) {
        return;      // exit early, no value
    }
    printf("Positive: %d\n", x);
}

int main(void) {
    int m = max(10, 20);
    check_positive(m);
    return 0;        // exit main
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>
#include <stdlib.h>   // for malloc, free

int main(void) {
    // 1. SELECTION: if-else
    int score = 85;
    if (score >= 90) {          // condition: if score >= 90
        printf("A\n");          // executed if true
    } else if (score >= 80) {   // else-if chain
        printf("B\n");
    } else {                    // default case
        printf("C or below\n");
    }

    // 2. SELECTION: switch
    int day = 3;
    switch (day) {               // expression evaluated (int)
        case 1:                  // if day == 1
            printf("Mon\n");
            break;               // exit switch; without break, fallthrough
        case 2:
            printf("Tue\n");
            break;
        default:                 // optional; if no case matches
            printf("Other\n");
            break;
    }

    // 3. ITERATION: while
    int i = 0;
    while (i < 5) {              // condition checked before each iteration
        printf("%d ", i);
        i++;                     // increment to avoid infinite loop
    }

    // 4. ITERATION: do-while
    int j = 0;
    do {                         // executes body at least once
        printf("%d ", j);
        j++;
    } while (j < 5);             // condition checked after body

    // 5. ITERATION: for
    for (int k = 0; k < 5; k++) { // init; condition; increment
        printf("%d ", k);
    }

    // 6. JUMP: break – exits loop
    for (int k = 0; k < 10; k++) {
        if (k == 5) {
            break;               // exits loop when k == 5
        }
    }

    // 7. JUMP: continue – skips to next iteration
    for (int k = 0; k < 5; k++) {
        if (k == 2) {
            continue;            // skips rest of body for k == 2
        }
        printf("%d ", k);
    }

    // 8. JUMP: goto – unconditional jump
    int x = -1;
    if (x < 0) {
        goto error;              // jumps to the label 'error'
    }
    printf("No error\n");
    error:                       // label
    printf("Error\n");

    // 9. JUMP: return – exits function
    return 0;                    // returns 0 to caller
}
```

---

## 4. Mental Model – Control Flow as a Road Network

Think of your program as a map of roads:

- **Sequential execution** – driving straight down a road.
- **`if` / `else`** – a fork in the road; you choose one path based on a sign (condition).
- **`switch`** – a multi‑way junction; you take one of several exits based on the exit number.
- **`while`** – a roundabout; you keep going around as long as the traffic light is green.
- **`do-while`** – a roundabout you enter at least once, even if the light is red.
- **`for`** – a roundabout with a counter; you go around a fixed number of times.
- **`break`** – an emergency exit from the roundabout.
- **`continue`** – skipping the rest of the current lap and going to the next.
- **`goto`** – a helicopter that can land anywhere (use sparingly!).
- **`return`** – exiting the road network entirely (leaving the function).

---

## 5. Core Concepts

### Selection

| Construct | Purpose | Syntax |
|-----------|---------|--------|
| **`if`** | Execute block if condition is true | `if (cond) { ... }` |
| **`if-else`** | Execute one block if true, another if false | `if (cond) { ... } else { ... }` |
| **`if-else if`** | Chain multiple conditions | `if (c1) { ... } else if (c2) { ... } else { ... }` |
| **`switch`** | Branch based on integral value | `switch (expr) { case c1: ... break; default: ... }` |

### Iteration

| Construct | Purpose | Syntax |
|-----------|---------|--------|
| **`while`** | Pre‑test loop; may execute zero times | `while (cond) { ... }` |
| **`do-while`** | Post‑test loop; always executes at least once | `do { ... } while (cond);` |
| **`for`** | Initialisation, condition, and increment in one line | `for (init; cond; inc) { ... }` |

### Jump

| Construct | Purpose | Syntax |
|-----------|---------|--------|
| **`break`** | Exit nearest loop or switch | `break;` |
| **`continue`** | Skip to next iteration of loop | `continue;` |
| **`goto`** | Jump to a label (unstructured) | `goto label;` |
| **`return`** | Exit the current function | `return expr;` (or `return;` in `void`) |

### Dangling `else`
- The `else` binds to the nearest unmatched `if` (without braces).
- Use braces to clarify.

### Loop Scope (C99+)
- Variables declared in the `for` initialiser have scope limited to the loop.

### Infinite Loops
- `while (1)` or `for (;;)` – use with `break` to exit.

---

## 6. How It Works – Under the Hood

### Conditional Branching
- `if (cond)` – the condition is evaluated; if true, the first block is executed; otherwise, control jumps to the `else` block (or the next statement).
- Compiled to `cmp` and conditional jumps (e.g., `je`, `jne`).

### Switch
- The expression is evaluated (integral type).
- The value is compared to each `case` constant; if a match is found, execution jumps to that case.
- Without `break`, execution falls through to the next case.
- Often compiled to a **jump table** for efficiency (for dense ranges).

### Loops
- **`while`** – condition checked at the top; if false, skip the body.
- **`do-while`** – body executed first, then condition checked.
- **`for`** – initialisation executed once; then condition checked; if true, body executed; then increment; repeat.
- All loops compile to conditional jumps.

### Jump Statements
- **`break`** – compiled to a jump to the end of the enclosing loop/switch.
- **`continue`** – compiled to a jump to the loop's increment/condition.
- **`goto`** – compiled to an unconditional jump.
- **`return`** – compiled to a jump back to the caller (with cleanup).

### ASCII Diagram – while Loop Flow
```
   ┌─────────────────┐
   │  Initialise     │
   └────────┬────────┘
            ▼
   ┌─────────────────┐
   │  Condition      │── false ──→ exit loop
   │  (expr)         │
   └────────┬────────┘
            │ true
            ▼
   ┌─────────────────┐
   │  Loop Body      │
   └────────┬────────┘
            │
            ▼
   ┌─────────────────┐
   │  Increment      │
   └────────┬────────┘
            │
            └───────→ back to condition
```

---

## 7. Internal Architecture – Compiler Representation

### Abstract Syntax Tree (AST)
- Each control flow construct is a node: `IfStmt`, `SwitchStmt`, `WhileStmt`, `ForStmt`, `DoWhileStmt`, `BreakStmt`, `ContinueStmt`, `GotoStmt`, `ReturnStmt`.
- Subnodes include the condition, body, and optional branches.

### Control Flow Graph (CFG)
- Each block of code is a basic block.
- Branches create edges to different blocks.
- Loops create back‑edges.
- The compiler uses the CFG for optimisation and code generation.

### Jump Table for `switch`
For a `switch` with contiguous integer cases, the compiler may generate a jump table:
```
jump_table:
    case 0: jump to code0
    case 1: jump to code1
    ...
```
This gives O(1) branching instead of O(n) comparisons.

---

## 8. Lifecycle / Workflow of a Control Flow Construct

1. **Written** – programmer writes the construct.
2. **Parsed** – compiler identifies the construct and builds an AST node.
3. **Semantic analysis** – checks types, scopes, and validity (e.g., `break` only in loops/switches).
4. **Optimisation** – common patterns may be simplified (e.g., `if (1) { ... }`).
5. **Code generation** – emits machine code (conditional jumps, branches).
6. **Execution** – CPU follows the jumps and branches at runtime.

---

## 9. Practical Examples

### Example 1: `if-else` for Value Classification
```c
#include <stdio.h>

int main(void) {
    int x = 15;

    // Classify a number.
    if (x > 0) {
        printf("Positive\n");
    } else if (x < 0) {
        printf("Negative\n");
    } else {
        printf("Zero\n");
    }

    // Check multiple conditions.
    if (x >= 0 && x <= 10) {
        printf("Between 0 and 10\n");
    } else if (x > 10 && x <= 20) {
        printf("Between 11 and 20\n");
    } else {
        printf("Outside range\n");
    }

    return 0;
}
```
**Code Breakdown:**
- `if-else if` chain handles multiple mutually exclusive conditions.
- Only the first true condition's block executes.
- Logical operators combine conditions.

### Example 2: `switch` for Menu Processing
```c
#include <stdio.h>

int main(void) {
    int choice = 2;

    switch (choice) {
        case 1:
            printf("Option 1: Add\n");
            break;
        case 2:
            printf("Option 2: Delete\n");
            // intentional fallthrough to case 3? Not here, break.
            break;
        case 3:
            printf("Option 3: List\n");
            break;
        default:
            printf("Invalid choice\n");
            break;
    }

    // Use switch with characters.
    char grade = 'B';
    switch (grade) {
        case 'A':
            printf("Excellent\n");
            break;
        case 'B':
            printf("Good\n");
            break;
        case 'C':
            printf("Average\n");
            break;
        default:
            printf("Needs improvement\n");
            break;
    }

    return 0;
}
```
**Code Breakdown:**
- `switch` evaluates an integral expression (`int`, `char`, `enum`).
- `case` labels must be constant expressions.
- `break` prevents fallthrough; without it, execution continues to the next case.
- Use `default` for handling unexpected values.

### Example 3: `while` for File Reading
```c
#include <stdio.h>

int main(void) {
    FILE *file = fopen("data.txt", "r");
    if (file == NULL) {
        printf("Could not open file\n");
        return 1;
    }

    int ch;
    // Read until end of file.
    while ((ch = fgetc(file)) != EOF) {
        putchar(ch);   // print each character
    }

    fclose(file);
    return 0;
}
```
**Code Breakdown:**
- `while` loop reads characters until `EOF` (end‑of‑file).
- The condition checks the return value of `fgetc` and assigns it to `ch` in one expression.
- The loop body prints each character.

### Example 4: `do-while` for Input Validation
```c
#include <stdio.h>

int main(void) {
    int num;

    // Keep asking until user enters a number between 1 and 10.
    do {
        printf("Enter a number between 1 and 10: ");
        scanf("%d", &num);
    } while (num < 1 || num > 10);

    printf("You entered: %d\n", num);
    return 0;
}
```
**Code Breakdown:**
- `do-while` guarantees the input prompt appears at least once.
- The condition checks if the input is invalid; if so, it loops again.

### Example 5: `for` Loop with Arrays
```c
#include <stdio.h>

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};
    int sum = 0;

    // Sum all elements.
    for (int i = 0; i < 5; i++) {
        sum += arr[i];
    }
    printf("Sum: %d\n", sum);

    // Find the maximum.
    int max = arr[0];
    for (int i = 1; i < 5; i++) {
        if (arr[i] > max) {
            max = arr[i];
        }
    }
    printf("Max: %d\n", max);

    // Reverse the array (in-place).
    for (int i = 0, j = 4; i < j; i++, j--) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    printf("Reversed: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    return 0;
}
```
**Code Breakdown:**
- `for` loops are ideal for array traversal where the number of iterations is known.
- Multiple variables can be declared/updated using the comma operator.
- The loop variable is scoped to the loop in C99+.

### Example 6: Nested Loops (Multiplication Table)
```c
#include <stdio.h>

int main(void) {
    // Print a multiplication table up to 5x5.
    for (int i = 1; i <= 5; i++) {
        for (int j = 1; j <= 5; j++) {
            printf("%3d ", i * j);
        }
        printf("\n");
    }
    return 0;
}
```
**Code Breakdown:**
- Nested `for` loops: outer loop controls rows, inner loop controls columns.
- `%3d` formats each number to 3 characters wide.

### Example 7: `break` and `continue` in a Loop
```c
#include <stdio.h>

int main(void) {
    // Print numbers 0-9, but skip 3 and stop at 7.
    for (int i = 0; i < 10; i++) {
        if (i == 3) {
            continue;      // skip printing 3
        }
        if (i == 7) {
            break;         // exit loop at 7
        }
        printf("%d ", i);
    }
    printf("\n");   // Output: 0 1 2 4 5 6
    return 0;
}
```
**Code Breakdown:**
- `continue` jumps to the next iteration (i++) when i == 3.
- `break` exits the loop when i == 7.

### Example 8: Error Handling with `goto`
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    FILE *file = NULL;
    char *buffer = NULL;

    file = fopen("input.txt", "r");
    if (file == NULL) {
        printf("Failed to open file\n");
        goto cleanup;
    }

    buffer = (char *)malloc(1024 * sizeof(char));
    if (buffer == NULL) {
        printf("Memory allocation failed\n");
        goto cleanup;
    }

    // ... use file and buffer (e.g., read data)
    printf("Processing data...\n");

cleanup:
    if (buffer != NULL) {
        free(buffer);
    }
    if (file != NULL) {
        fclose(file);
    }
    return 0;
}
```
**Code Breakdown:**
- `goto` jumps to the `cleanup` label when an error occurs.
- This pattern centralises resource release, avoiding duplication.
- It's a common idiom in C, especially in systems code.

---

## 10. Common Use Cases

| Construct | Use Case |
|-----------|----------|
| **`if`** | Single‑condition checks, error handling. |
| **`if-else`** | Binary choices (e.g., pass/fail). |
| **`if-else if`** | Multiple exclusive conditions (e.g., grade assignment). |
| **`switch`** | Multi‑way branching on an integral value (e.g., menu options, state machines). |
| **`while`** | Indefinite loops (e.g., reading input until EOF). |
| **`do-while`** | Loops that must run at least once (e.g., input validation). |
| **`for`** | Counted loops (e.g., array traversal, fixed‑iteration tasks). |
| **`break`** | Early exit from a loop/switch (e.g., when a condition is met). |
| **`continue`** | Skip current iteration (e.g., when processing invalid items). |
| **`goto`** | Error cleanup, breaking out of deeply nested loops. |
| **`return`** | Exiting a function early (e.g., on error). |

---

## 11. Best Practices

### General
- **Use braces `{}`** even for single‑statement bodies – improves readability and reduces bugs.
- **Indent consistently** – make the structure visually clear.
- **Avoid deep nesting** – refactor into functions or use early returns.
- **Prefer structured control flow** over `goto` (except for cleanup).

### `if` / `else`
- **Check error conditions first** – handle errors early and return.
- **Use `else if` for mutually exclusive conditions**.
- **Avoid `if (x = 5)`** – use `==` for comparison.

### `switch`
- **Always include a `default` case** – even if it does nothing.
- **Use `break`** to prevent accidental fallthrough.
- **Consider `switch` for many integer comparisons** – it can be more efficient and clearer than long `if-else if` chains.

### Loops
- **Prefer `for`** when the number of iterations is known.
- **Prefer `while`** when the loop depends on a condition that may change.
- **Ensure loops terminate** – update the condition variable.
- **Use `for (;;)`** for infinite loops – it's idiomatic.

### `break` / `continue`
- **Use `break` to exit loops early** when a condition is met.
- **Use `continue` to skip iterations** for specific cases.
- **Avoid `break` in deeply nested loops** – consider refactoring.

### `goto`
- **Use `goto` only for error cleanup** in a single function.
- **Jump forward only** – to a label at the end of the function.
- **Avoid `goto` for general control flow** – it makes code hard to follow.

### `return`
- **Use early returns** for error handling.
- **Keep return statements clear** – compute the return value before returning.

---

## 12. Common Mistakes

### Mistake 1: Missing Braces Causing Dangling `else`
```c
// ❌ Wrong – else binds to the inner if, not the outer.
if (x > 0)
    if (y > 0)
        printf("Both positive\n");
else
    printf("x <= 0\n");   // Actually executes when x > 0 && y <= 0.
// ✅ Correct – use braces.
if (x > 0) {
    if (y > 0) {
        printf("Both positive\n");
    }
} else {
    printf("x <= 0\n");
}
```

### Mistake 2: Using `=` Instead of `==` in Condition
```c
// ❌ Wrong – assignment, not comparison.
if (x = 5) {   // x becomes 5, condition is true (non-zero).
    // Always executes.
}
// ✅ Correct – use ==.
if (x == 5) { ... }
```

### Mistake 3: Missing `break` in `switch`
```c
// ❌ Wrong – accidental fallthrough.
switch (x) {
    case 1: printf("One\n");
    case 2: printf("Two\n");   // Falls through from case 1.
}
// ✅ Correct – add break.
switch (x) {
    case 1: printf("One\n"); break;
    case 2: printf("Two\n"); break;
}
```

### Mistake 4: Infinite Loop (Condition Never Changes)
```c
// ❌ Wrong – i never increments.
int i = 0;
while (i < 5) {
    printf("%d\n", i);
    // forgot i++;
}
// ✅ Correct – update i.
while (i < 5) {
    printf("%d\n", i);
    i++;
}
```

### Mistake 5: Off‑by‑One in `for` Loops
```c
// ❌ Wrong – loops 5 times (i = 0..4) but you might want 0..4 or 1..5.
for (int i = 0; i <= 5; i++) {   // runs 6 times (0..5)
    // ...
}
// ✅ Correct – choose appropriate condition.
for (int i = 0; i < 5; i++) {   // runs 5 times (0..4)
    // ...
}
```

### Mistake 6: `break` Outside a Loop or Switch
```c
// ❌ Wrong – break can only be used inside loops or switch.
if (x > 0) {
    break;   // Error.
}
// ✅ Correct – use return or restructure.
if (x > 0) {
    return;   // if in a function.
}
```

### Mistake 7: `continue` Outside a Loop
```c
// ❌ Wrong – continue can only be used inside loops.
if (x > 0) {
    continue;   // Error.
}
```

### Mistake 8: Using `goto` to Jump into a Block
```c
// ❌ Wrong – jumping into a block can bypass initialisation.
goto label;
{
    int x = 10;
label:
    printf("%d\n", x);   // x may be uninitialised.
}
// ✅ Correct – avoid jumping into blocks.
```

### Mistake 9: `return` with No Value in Non‑`void` Function
```c
// ❌ Wrong – missing return value (undefined behaviour).
int add(int a, int b) {
    if (a > b) {
        return a + b;
    }
    // No return for the else case.
}
// ✅ Correct – ensure all paths return a value.
int add(int a, int b) {
    if (a > b) {
        return a + b;
    } else {
        return a - b;
    }
}
```

---

## 13. Performance Considerations

- **`if` vs `switch`** – `switch` may be faster for many cases due to jump tables.
- **Loop overhead** – `for` and `while` compile to similar code; choose based on clarity.
- **`do-while`** – can be slightly faster if the loop always executes at least once (avoids a branch).
- **`break` / `continue`** – add minimal overhead (conditional jumps).
- **`goto`** – compiles to an unconditional jump, very fast.
- **Loop unrolling** – compilers may unroll small loops for performance.
- **Short‑circuit** – `&&` and `||` can improve performance by avoiding expensive operations.

---

## 14. Security Considerations

- **Infinite loops** – can cause denial of service; always ensure termination.
- **`goto`** – can bypass variable initialisation, leading to undefined behaviour.
- **`switch` fallthrough** – unintended fallthrough can cause logic errors.
- **`return`** – ensure all paths return a value to avoid undefined behaviour.
- **`break` / `continue`** – use carefully in nested loops to avoid skipping cleanup.

---

## 15. Debugging Tips

- **Use `-Wall -Wextra`** – catches many control‑flow issues (e.g., missing returns, unused variables).
- **Use `-Wswitch`** – warns about missing `case` or `default`.
- **Use a debugger** – step through to see which branches are taken.
- **Add print statements** – trace the flow of execution.
- **Use `assert`** – to verify invariants.

---

## 16. When to Use

- **All the time** – control flow is fundamental to every program.
- **Use the right construct** for the task: `if` for binary choices, `switch` for multi‑way, `for` for counted loops, `while` for indefinite loops.

---

## 17. When Not to Use

- **Avoid `goto`** for general control flow.
- **Avoid `switch`** for non‑integer or floating‑point values.
- **Avoid `do-while`** if the loop might need to execute zero times – use `while`.
- **Avoid over‑nesting** – extract nested logic into functions.

---

## 18. Related Concepts

- **Expressions** – conditions in control flow are expressions.
- **Sequence points** – loops and `switch` have sequence points.
- **Functions** – contain control flow within their bodies.
- **Scope** – affects variable visibility in blocks.
- **Labels** – used with `goto`.

---

## 19. Did You Know?

- In C, `switch` can only evaluate integral expressions; for floating‑point or strings, use `if-else`.
- The `for` loop's initialisation, condition, and increment are all optional; `for (;;)` is an infinite loop.
- `break` and `continue` are allowed only inside loops or `switch`; using them elsewhere is a syntax error.
- `goto` can only jump within the same function; you cannot `goto` another function.
- The `do-while` loop is the only loop that always executes at least once.
- C compilers often optimise `switch` with dense cases into a jump table for O(1) performance.

---

## 20. Summary

- **Control flow determines the order of execution** in a C program.
- **Selection** – `if`, `if-else`, `else if`, `switch` – makes decisions.
- **Iteration** – `while`, `do-while`, `for` – repeats code.
- **Jump** – `break`, `continue`, `goto`, `return` – transfers control.
- **Best practices**: use braces, avoid dangling `else`, prefer `for` for counted loops, `switch` for multi‑way branching, `goto` only for cleanup.
- **Common mistakes**: using `=` instead of `==`, missing `break` in `switch`, infinite loops, off‑by‑one errors.
- **Understanding control flow** is essential for writing programs that can react, repeat, and make decisions – it's the heart of imperative programming.