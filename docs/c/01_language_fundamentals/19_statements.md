# Statements

## 1. Overview

### Definition
A statement in C is a syntactic unit that expresses an action to be performed. It is the fundamental building block of imperative programming in C. Statements are terminated with a semicolon (`;`) and are executed sequentially in the order they appear, unless control‑flow statements alter that flow.

### Purpose
Statements are how you tell the computer what to do. They allow you to:
- Declare variables (`int x;`)
- Perform computations and assignments (`x = y + 5;`)
- Make decisions (`if (x > 0) { ... }`)
- Repeat blocks of code (`for (i = 0; i < n; i++)`)
- Transfer control (`return 0;`, `goto error;`)
- Do nothing (`;` – the null statement)

### Where It Fits
Statements are the executable units within functions. A C program consists of:
- **Declarations** – introduce types, variables, and functions.
- **Statements** – perform actions (they appear inside function bodies).
- **Expressions** – evaluate to values and are part of statements.

Every function body is a block (compound statement) containing zero or more statements.

---

## 2. Why It Exists

### The Problem Without Statements
Without statements, you could declare variables and define functions, but you couldn't actually *do* anything with them. Programs would be static data, not executable logic.

### The Solution: Executable Actions
Statements provide the imperative "do this" actions that make programs dynamic. They are the verbs of the language, driving computation, decision‑making, iteration, and control transfer.

### Why They Were Introduced
Statements are a fundamental concept in all imperative programming languages, derived from the earliest von Neumann architectures where programs were sequences of instructions. C's statement set is deliberately minimal and close to the hardware, with control‑flow constructs that translate efficiently to conditional jumps and branches.

---

## 3. Syntax / Basic Usage

### Categories of Statements

#### 1. Declaration Statements
```c
#include <stdio.h>

int main(void) {
    // Declaration statement (C99+ allows anywhere in a block)
    int x;                 // declares x
    int y = 10;            // declares and initialises y
    int a = 5, b = 6;      // multiple declarations

    // C99 – declarations can appear after statements.
    x = 20;
    int z = x + y;         // valid in C99+ (not in C89)

    return 0;
}
```

#### 2. Expression Statements
```c
#include <stdio.h>

int main(void) {
    int x = 10;
    int y = 20;

    // Expression statement – expression followed by ;
    x = y + 5;            // assignment statement
    y++;                  // increment statement
    x += y;               // compound assignment
    printf("x: %d\n", x); // function call expression

    // An expression with no value can be a statement (e.g., function returning void)
    // printf already returns int, but we discard it.

    // The null statement – does nothing.
    ;                     // a lone semicolon

    return 0;
}
```

#### 3. Compound Statements (Blocks)
```c
#include <stdio.h>

int main(void) {
    int x = 10;           // outer block

    {                     // compound statement (block) begins
        int y = 20;       // inner block
        printf("x: %d, y: %d\n", x, y); // x is accessible here
    }                     // y is destroyed here

    // y is NOT accessible here (out of scope)

    return 0;
}
```

#### 4. Selection Statements
```c
#include <stdio.h>

int main(void) {
    int score = 85;

    // if statement
    if (score >= 90) {
        printf("Grade: A\n");
    } else if (score >= 80) {
        printf("Grade: B\n");
    } else if (score >= 70) {
        printf("Grade: C\n");
    } else {
        printf("Grade: F\n");
    }

    // switch statement
    int option = 2;
    switch (option) {
        case 1:
            printf("Option 1\n");
            break;
        case 2:
            printf("Option 2\n");
            break;
        case 3:
            printf("Option 3\n");
            break;
        default:
            printf("Invalid option\n");
            break;
    }

    return 0;
}
```

#### 5. Iteration Statements (Loops)
```c
#include <stdio.h>

int main(void) {
    // while loop
    int i = 0;
    while (i < 5) {
        printf("while: %d\n", i);
        i++;
    }

    // do-while loop – executes at least once
    int j = 0;
    do {
        printf("do-while: %d\n", j);
        j++;
    } while (j < 5);

    // for loop
    for (int k = 0; k < 5; k++) {
        printf("for: %d\n", k);
    }

    // Infinite loop
    // while (1) { ... } or for (;;) { ... }

    return 0;
}
```

#### 6. Jump Statements
```c
#include <stdio.h>

int main(void) {
    // break – exits the nearest enclosing loop or switch
    for (int i = 0; i < 10; i++) {
        if (i == 5) {
            break;         // exits the loop when i == 5
        }
        printf("break: %d\n", i);
    }

    // continue – skips to the next iteration
    for (int i = 0; i < 5; i++) {
        if (i == 2) {
            continue;      // skips printing 2
        }
        printf("continue: %d\n", i);
    }

    // goto – unconditional jump (use sparingly)
    int x = 0;
    if (x < 0) {
        goto error;
    }
    printf("No error\n");
    error:
    printf("Error handler\n");

    // return – exits the function (optionally with a value)
    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>

// Declaration statements – appear in the function body.
// In C99+, declarations can be mixed with statements.
int main(void) {
    // 1. Declaration statements:
    int x;                 // declares x (uninitialised)
    int y = 10;            // declares and initialises y
    int a = 5, b = 6;      // declares a and b in one statement

    // 2. Expression statements:
    x = 20;                // assignment expression followed by ;
    y++;                   // increment expression
    a += b;                // compound assignment
    printf("x: %d\n", x);  // function call expression (returns int, discarded)

    // 3. Null statement:
    ;                      // a semicolon with no expression – does nothing

    // 4. Compound statement (block):
    {                      // start of block
        int z = x + y;     // local to this block
        printf("z: %d\n", z);
    }                      // end of block; z is destroyed

    // 5. Selection statements:
    if (x > 10) {          // if statement
        printf("x > 10\n");
    } else {
        printf("x <= 10\n");
    }

    switch (y) {           // switch statement
        case 10:
            printf("y is 10\n");
            break;         // break prevents fallthrough
        case 20:
            printf("y is 20\n");
            break;
        default:
            printf("y is something else\n");
            break;
    }

    // 6. Iteration statements:
    for (int i = 0; i < 3; i++) {   // for loop
        printf("for: %d\n", i);
    }

    int j = 0;
    while (j < 3) {                 // while loop
        printf("while: %d\n", j);
        j++;
    }

    int k = 0;
    do {                            // do-while loop (executes at least once)
        printf("do-while: %d\n", k);
        k++;
    } while (k < 3);

    // 7. Jump statements:
    for (int i = 0; i < 5; i++) {
        if (i == 2) {
            continue;        // skips rest of loop body when i == 2
        }
        if (i == 4) {
            break;           // exits the loop when i == 4
        }
        printf("loop: %d\n", i);
    }

    if (x < 0) {
        goto error;          // jumps to the label 'error'
    }

    // Label statement:
    error:                   // label (used with goto)
    printf("Error occurred\n");

    return 0;                // return statement
}
```

---

## 4. Mental Model – Statements as Actions in a Play

Think of a C program as a play script:

- **Declaration statements** – like stage directions: "Enter the character named `x` (with type `int`)."
- **Expression statements** – like actions: "Character `x` picks up the value 5."
- **Compound statements** – like scenes: a block of actions grouped together.
- **Selection statements** – like a branching plot: "If the king is alive, go to Act 1; otherwise, go to Act 2."
- **Iteration statements** – like a repeated action: "The army marches while the general waves."
- **Jump statements** – like an unexpected plot twist: "Jump to the final scene."
- **Null statement** – a pause: do nothing for one beat.

The flow of statements determines the story of the program.

---

## 5. Core Concepts

### Summary of Statement Types

| Category | Keywords/Syntax | Purpose |
|----------|-----------------|---------|
| **Declaration** | `type name;` | Introduce variables |
| **Expression** | `expr;` | Evaluate an expression (including assignments, function calls) |
| **Compound** | `{ ... }` | Group statements into a block |
| **Null** | `;` | No operation |
| **Selection** | `if`, `if-else`, `switch` | Make decisions |
| **Iteration** | `while`, `do-while`, `for` | Repeat blocks of code |
| **Jump** | `break`, `continue`, `goto`, `return` | Transfer control |
| **Label** | `label:` | Target for `goto` |

### Control Flow Overview

```
┌─────────────────────────────────────────────────────────────┐
│                       Program Flow                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌───┐    ┌───┐    ┌───┐    ┌───┐    ┌───┐              │
│   │ S │ → │ S │ → │ S │ → │ S │ → │ S │  Sequential       │
│   └───┘    └───┘    └───┘    └───┘    └───┘              │
│                                                             │
│   ┌─────────┐                                              │
│   │ if (c)  │── true  → ┌───┐                              │
│   │ { ... } │          │ S │                              │
│   └─────────┘          └───┘                              │
│       │ false            │                                  │
│       ▼                  ▼                                  │
│   ┌─────────┐                                              │
│   │ else    │──→  ┌───┐                                    │
│   │ { ... } │      │ S │                                  │
│   └─────────┘      └───┘                                  │
│                                                             │
│   ┌─────────────┐                                          │
│   │ while(c)    │── true → ┌───┐                          │
│   │ { ... }     │         │ S │                          │
│   └─────────────┘         └───┘                          │
│       │ false               │                              │
│       ▼                     ▼                              │
│   (exit loop)        (go back to condition)                │
│                                                             │
│   ┌─────────┐                                              │
│   │ switch  │──→ case 1 → ┌───┐                           │
│   │ (expr)  │   case 2 → │ S │                           │
│   └─────────┘   case 3 → └───┘                           │
│       │ default                                           │
│       ▼                                                    │
│   ┌─────────┐                                              │
│   │ break;  │──→ exit switch                               │
│   └─────────┘                                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. How It Works – Statement Execution Flow

### Sequential Execution
By default, statements are executed in the order they appear, from top to bottom. The CPU fetches each instruction in sequence.

### Control Flow
Selection and iteration statements alter this flow:
- **`if`** – conditionally executes a block based on a test.
- **`switch`** – branches to a case based on an integer value.
- **`for`/`while`/`do-while`** – repeatedly execute a block based on a condition.
- **`goto`** – unconditionally jumps to a label.
- **`break`** – exits the nearest loop or switch.
- **`continue`** – skips to the next iteration of the loop.
- **`return`** – exits the function.

### Compiler Implementation
- **`if`/`else`** – compiled to conditional jump instructions (e.g., `cmp`, `je`).
- **`switch`** – often compiled to a jump table (efficient for many cases).
- **`for`/`while`** – compiled to conditional jumps and loops.
- **`goto`** – compiled to an unconditional jump instruction.
- **`break`/`continue`** – compiled to jumps to the loop's exit or increment.

### ASCII Diagram – while Loop Assembly
```
C Code:
    int i = 0;
    while (i < 5) {
        // do something
        i++;
    }

x86‑64 Assembly (conceptual):
    mov eax, 0          ; i = 0
    jmp check
loop:
    ; do something
    inc eax              ; i++
check:
    cmp eax, 5
    jl loop              ; if i < 5, jump to loop
    ; exit
```

---

## 7. Internal Architecture – AST and Control Flow Graphs

### Abstract Syntax Tree (AST)
The compiler builds an AST where:
- Each statement is a node (e.g., `IfStmt`, `WhileStmt`, `ForStmt`).
- Expressions are subnodes of statements.

### Control Flow Graph (CFG)
- Each statement becomes a basic block (a sequence of instructions with one entry and one exit).
- Branches (`if`, `switch`) create edges to different blocks.
- Loops (`for`, `while`, `do-while`) create back‑edges.

---

## 8. Lifecycle / Workflow of a Statement

1. **Written** – programmer writes a statement in the source code.
2. **Parsed** – compiler identifies the statement type and builds an AST node.
3. **Semantic analysis** – checks types, declarations, and scopes.
4. **Code generation** – emits machine code for the statement.
5. **Execution** – CPU executes the statement at runtime.

---

## 9. Practical Examples

### Example 1: `if-else` Statements
```c
#include <stdio.h>

int main(void) {
    int x = 10;

    // Simple if statement
    if (x > 0) {
        printf("x is positive\n");
    }

    // if-else statement
    if (x % 2 == 0) {
        printf("x is even\n");
    } else {
        printf("x is odd\n");
    }

    // if-else if ladder
    if (x > 0) {
        printf("x > 0\n");
    } else if (x < 0) {
        printf("x < 0\n");
    } else {
        printf("x == 0\n");
    }

    // Nested if statements
    if (x > 0) {
        if (x > 5) {
            printf("x > 5\n");
        } else {
            printf("0 < x <= 5\n");
        }
    }

    // Dangling else – the else binds to the nearest if.
    int y = 5;
    if (x > 0)
        if (y > 0)
            printf("Both positive\n");
        else
            printf("y <= 0\n");   // This else binds to the inner if.

    // Use braces to avoid ambiguity.
    if (x > 0) {
        if (y > 0) {
            printf("Both positive\n");
        }
    } else {
        printf("x <= 0\n");
    }

    return 0;
}
```
**Code Breakdown:**
- `if` statements conditionally execute a statement (or block).
- `else` is optional and binds to the nearest unmatched `if`.
- Use braces `{}` for clarity, especially with nested `if` statements.

### Example 2: `switch` Statements
```c
#include <stdio.h>

int main(void) {
    int day = 3;

    // Basic switch
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

    // Fallthrough example (no break)
    int month = 2;
    switch (month) {
        case 1:
        case 3:
        case 5:
        case 7:
        case 8:
        case 10:
        case 12:
            printf("31 days\n");
            break;
        case 4:
        case 6:
        case 9:
        case 11:
            printf("30 days\n");
            break;
        case 2:
            printf("28 or 29 days\n");
            break;
        default:
            printf("Invalid month\n");
            break;
    }

    // Using switch with characters
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
            printf("Below average\n");
            break;
    }

    return 0;
}
```
**Code Breakdown:**
- `switch` evaluates an integer expression and jumps to a matching `case`.
- `break` exits the switch; without it, execution "falls through" to the next case.
- `default` is optional, executed if no case matches.
- `case` labels must be integer constant expressions (compile‑time constants).

### Example 3: `while` and `do-while` Loops
```c
#include <stdio.h>

int main(void) {
    // while loop – checks condition before execution
    int i = 0;
    while (i < 5) {
        printf("while: %d\n", i);
        i++;
    }

    // do-while loop – executes at least once
    int j = 0;
    do {
        printf("do-while: %d\n", j);
        j++;
    } while (j < 5);

    // while loop with break
    int k = 0;
    while (1) {              // infinite loop
        if (k >= 5) {
            break;           // exit the loop
        }
        printf("break: %d\n", k);
        k++;
    }

    // while loop with continue
    int m = 0;
    while (m < 5) {
        m++;
        if (m == 2) {
            continue;        // skip printing 2
        }
        printf("continue: %d\n", m);
    }

    // Sentinel-controlled loop (read input until sentinel)
    int value;
    printf("Enter numbers (0 to stop): ");
    while (1) {
        scanf("%d", &value);
        if (value == 0) {
            break;
        }
        printf("You entered: %d\n", value);
    }

    return 0;
}
```
**Code Breakdown:**
- `while` checks the condition before executing the loop body.
- `do-while` executes the body first, then checks the condition.
- `break` exits the nearest enclosing loop.
- `continue` skips the rest of the loop body and goes to the next iteration.
- `while (1)` creates an infinite loop; use `break` to exit.

### Example 4: `for` Loops
```c
#include <stdio.h>

int main(void) {
    // Basic for loop: initialise; condition; increment
    for (int i = 0; i < 5; i++) {
        printf("for: %d\n", i);
    }

    // Multiple initialisations and increments (comma operator)
    for (int i = 0, j = 10; i < 5; i++, j--) {
        printf("i: %d, j: %d\n", i, j);
    }

    // Using a variable declared outside the loop
    int k;
    for (k = 0; k < 5; k++) {
        printf("k: %d\n", k);
    }
    // k is still accessible here (C89 style)

    // Infinite for loop
    for (;;) {
        // break to exit
        static int count = 0;
        if (count >= 3) {
            break;
        }
        printf("infinite loop iteration: %d\n", count);
        count++;
    }

    // For loop with continue
    for (int i = 0; i < 5; i++) {
        if (i == 2) {
            continue;
        }
        printf("for continue: %d\n", i);
    }

    // Nested for loops (e.g., 2D array)
    for (int row = 0; row < 3; row++) {
        for (int col = 0; col < 4; col++) {
            printf("[%d][%d] ", row, col);
        }
        printf("\n");
    }

    // For loop with empty body (e.g., busy wait)
    // for (volatile int i = 0; i < 1000000; i++) { } // delay

    return 0;
}
```
**Code Breakdown:**
- `for (init; condition; increment)` – initialises, checks condition, executes body, then increments.
- All three parts are optional: `for (;;)` is an infinite loop.
- Multiple expressions can be used with the comma operator.
- In C99, `for` loop variables (`int i`) are scoped to the loop.

### Example 5: `break` and `continue`
```c
#include <stdio.h>

int main(void) {
    // break in a for loop
    printf("break: ");
    for (int i = 0; i < 10; i++) {
        if (i == 5) {
            break;           // exits the loop when i == 5
        }
        printf("%d ", i);
    }
    printf("\n");

    // continue in a for loop
    printf("continue: ");
    for (int i = 0; i < 5; i++) {
        if (i == 2) {
            continue;        // skips printing 2
        }
        printf("%d ", i);
    }
    printf("\n");

    // break in a while loop
    int j = 0;
    while (j < 10) {
        if (j == 5) {
            break;
        }
        printf("%d ", j);
        j++;
    }
    printf("\n");

    // continue in a while loop
    int k = 0;
    while (k < 5) {
        k++;
        if (k == 2) {
            continue;
        }
        printf("%d ", k);
    }
    printf("\n");

    // break in a switch (not a loop)
    int option = 2;
    switch (option) {
        case 1:
            printf("Option 1\n");
            break;     // exits the switch
        case 2:
            printf("Option 2\n");
            break;     // exits the switch
        default:
            printf("Invalid\n");
            break;
    }

    // break nested loops (only breaks the inner loop)
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (i == 1 && j == 1) {
                break;     // only breaks the inner loop
            }
            printf("[%d][%d] ", i, j);
        }
        printf("\n");
    }

    // To break out of nested loops, use a flag or goto.
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (i == 1 && j == 1) {
                goto out;   // breaks out of both loops
            }
            printf("[%d][%d] ", i, j);
        }
    }
    out:
    printf("\nExited nested loops\n");

    return 0;
}
```
**Code Breakdown:**
- `break` exits the nearest enclosing `for`, `while`, `do-while`, or `switch`.
- `continue` skips the rest of the body and moves to the next iteration.
- `break` in nested loops only breaks the inner loop; use `goto` to break multiple levels.

### Example 6: `goto` and Labels
```c
#include <stdio.h>

int main(void) {
    int x = 10;

    // Forward jump
    if (x < 0) {
        goto error;          // jump to error label
    }

    printf("No error\n");
    return 0;                // skip error handler

    error:                   // label
    printf("Error occurred\n");
    return 1;

    // goto can also jump backward
    int i = 0;
    start:                   // label
    if (i < 3) {
        printf("goto loop: %d\n", i);
        i++;
        goto start;          // jump back to start
    }

    // goto can be used for error cleanup (common pattern)
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

    // ... use data and file

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
**Code Breakdown:**
- `goto` jumps to a label within the same function.
- Labels have function scope (visible anywhere in the function).
- `goto` is often used for error‑handling cleanup in the Linux kernel and other C codebases.
- Excessive use of `goto` can make code hard to follow; use sparingly and only when it improves readability (e.g., cleanup).

### Example 7: `return` Statements
```c
#include <stdio.h>
#include <stdlib.h>

// Function returning int
int add(int a, int b) {
    return a + b;   // returns the sum
}

// Function with early return
int max(int a, int b) {
    if (a > b) {
        return a;
    }
    return b;      // else returns b
}

// Function returning void (no value)
void print_message(const char *msg) {
    if (msg == NULL) {
        return;    // early exit, no value
    }
    printf("%s\n", msg);
}

// Function with multiple return points
int divide(int a, int b, int *result) {
    if (b == 0) {
        return -1;   // error: division by zero
    }
    *result = a / b;
    return 0;        // success
}

int main(void) {
    int sum = add(5, 3);
    printf("sum: %d\n", sum);

    int m = max(10, 20);
    printf("max: %d\n", m);

    print_message("Hello");

    int result;
    int status = divide(10, 2, &result);
    if (status == 0) {
        printf("10 / 2 = %d\n", result);
    } else {
        printf("Error: division by zero\n");
    }

    return EXIT_SUCCESS;   // using EXIT_SUCCESS from stdlib.h
}
```
**Code Breakdown:**
- `return` exits the current function.
- In a non‑`void` function, `return` must be followed by an expression of the correct type.
- In a `void` function, `return;` (no value) exits early.
- Early returns can simplify logic by handling error cases upfront.

### Example 8: Null and Empty Statements
```c
#include <stdio.h>

int main(void) {
    // Null statement – does nothing.
    ;                     // a lone semicolon

    // Sometimes used in loops where the body is empty.
    // Wait for a condition (busy wait – not recommended in production).
    volatile int flag = 0;
    // while (flag == 0);   // null statement as loop body (infinite loop)

    // Using a null statement in a for loop.
    int i = 0;
    for (; i < 5; ) {
        printf("%d ", i);
        i++;
    }
    printf("\n");

    // Empty block (compound statement with no statements).
    {
        // does nothing
    }

    return 0;
}
```
**Code Breakdown:**
- A null statement is a single semicolon `;`.
- It is sometimes used as an empty loop body.
- An empty block `{}` is also a valid compound statement that does nothing.

---

## 10. Common Use Cases

| Statement Type | Use Case | Example |
|----------------|----------|---------|
| **Declaration** | Define variables | `int count = 0;` |
| **Expression** | Perform operations, assignments, function calls | `x = y + 5;` |
| **Compound** | Group multiple statements | `{ int x = 5; printf("%d", x); }` |
| **Null** | Empty loop body | `while (*src++) { }` |
| **`if`** | Conditional execution | `if (x > 0) handle_positive();` |
| **`switch`** | Multi‑way branching | `switch (command) { ... }` |
| **`while`** | Pre‑test loop | `while (i < n) { ... }` |
| **`do-while`** | Post‑test loop (executes at least once) | `do { ... } while (condition);` |
| **`for`** | Counted loops | `for (int i=0; i<n; i++)` |
| **`break`** | Exit loop or switch | `if (error) break;` |
| **`continue`** | Skip iteration | `if (skip) continue;` |
| **`goto`** | Unconditional jump (error cleanup) | `goto cleanup;` |
| **`return`** | Exit function | `return result;` |
| **Label** | Target for `goto` | `cleanup:` |

---

## 11. Best Practices

### General
- **Use braces** `{}` for all `if`, `else`, `for`, `while`, and `do-while` bodies, even if they contain a single statement – improves readability and prevents bugs.
- **Keep functions small** – a function should be readable on one screen; helps manage statement complexity.
- **Indent properly** – consistent indentation makes control flow visible.

### `if` / `else`
- **Avoid dangling `else`** – use braces to clarify which `if` the `else` belongs to.
- **Check error cases first** – use early returns or `if (error) { ... }` at the top.
- **Use `else if`** for mutually exclusive conditions (instead of nested `if` statements).

### `switch`
- **Always include a `default` case** – even if it does nothing; it documents intent.
- **Break each `case`** unless you intentionally want fallthrough (and document it).
- **Use `switch` for comparing a single variable against many constants** – it's clearer and often more efficient than a long `if-else` chain.

### Loops
- **Prefer `for` when you know the number of iterations** – it keeps initialisation, condition, and increment together.
- **Prefer `while` when the number of iterations is unknown** (e.g., reading input).
- **Avoid infinite loops** – unless intentional, and always have a `break` condition.
- **Use `for (;;)` for infinite loops** – it's idiomatic and clear.

### `break` / `continue`
- **Use `break` to exit loops early** when a condition is met.
- **Use `continue` to skip remaining loop body** for specific cases.
- **Avoid `break` in deeply nested loops** – consider extracting the loop into a function or using a flag.

### `goto`
- **Use `goto` only for error cleanup** in a single function – it can simplify resource release.
- **Avoid `goto` for general control flow** – it can make code spaghetti.
- **Jump forward only** when using `goto` for cleanup (to a label at the end of the function).

### `return`
- **Use early returns** to handle error conditions upfront.
- **Keep return statements simple** – prefer computing a result and returning it at the end.
- **Use `return` consistently** – either all early returns or a single return at the end.

---

## 12. Common Mistakes

### Mistake 1: Forgetting Braces in `if` / `else`
```c
// ❌ Wrong – dangling else: else binds to the inner if.
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

### Mistake 2: Missing `break` in `switch`
```c
// ❌ Wrong – fallthrough (may be intentional, but often a bug).
switch (x) {
    case 1:
        printf("One\n");   // No break – falls through to case 2.
    case 2:
        printf("Two\n");
        break;
}
// ✅ Correct – add break.
switch (x) {
    case 1:
        printf("One\n");
        break;
    case 2:
        printf("Two\n");
        break;
}
```

### Mistake 3: Infinite Loop
```c
// ❌ Wrong – infinite loop (condition never changes).
int i = 0;
while (i < 5) {
    printf("%d\n", i);
    // forgot i++;
}
// ✅ Correct – increment i.
int i = 0;
while (i < 5) {
    printf("%d\n", i);
    i++;
}
```

### Mistake 4: Using `=` Instead of `==` in Conditionals
```c
// ❌ Wrong – assignment, not comparison.
if (x = 5) {   // x becomes 5, condition is true.
    // Always executes.
}
// ✅ Correct – use ==.
if (x == 5) { ... }
```

### Mistake 5: `break` Outside a Loop or Switch
```c
// ❌ Wrong – break can only be used in loops and switch.
if (x > 0) {
    break;   // Error: break statement not within loop or switch.
}
// ✅ Correct – only use break in loops or switch.
```

### Mistake 6: `continue` Outside a Loop
```c
// ❌ Wrong – continue can only be used in loops.
if (x > 0) {
    continue;   // Error: continue statement not within a loop.
}
// ✅ Correct – only use continue in loops.
```

### Mistake 7: Using `goto` to Jump Into a Block
```c
// ❌ Wrong – jumping into a block can bypass variable initialisation.
goto label;
{
    int x = 10;   // May not be initialised if jumped over.
label:
    printf("%d\n", x);   // Undefined behaviour.
}
// ✅ Correct – avoid jumping into blocks.
```

### Mistake 8: Empty Loop Body Without Proper Intent
```c
// ❌ Wrong – busy wait with no indication of intent.
while (flag == 0);   // This is a null statement; may spin forever.
// ✅ Correct – if intentional, add a comment.
while (flag == 0) { }   // Wait for flag to be set (busy wait).
```

### Mistake 9: `return` Without a Value in a Non‑void Function
```c
// ❌ Wrong – missing return value (undefined behaviour if used).
int add(int a, int b) {
    if (a > b) {
        return a + b;
    }
    // No return for the else case – undefined behaviour.
}
// ✅ Correct – always return a value.
int add(int a, int b) {
    if (a > b) {
        return a + b;
    } else {
        return a - b;
    }
}
```

### Mistake 10: Omitting the `default` Case in `switch`
```c
// ❌ Wrong – default case missing; unexpected values are ignored.
switch (option) {
    case 1: handle_one(); break;
    case 2: handle_two(); break;
    // No default – option 3 does nothing silently.
}
// ✅ Correct – add a default case.
switch (option) {
    case 1: handle_one(); break;
    case 2: handle_two(); break;
    default: handle_invalid(); break;
}
```

---

## 13. Performance Considerations

- **`if` vs `switch`** – `switch` is often more efficient for multiple integer comparisons (jump table).
- **Loops** – `for` and `while` compile to similar code; choose based on clarity.
- **`do-while`** – executes the body at least once, which can avoid an extra condition check in some cases.
- **`break` / `continue`** – compile to jumps, minimal overhead.
- **`goto`** – compiles to an unconditional jump, very fast.
- **Recursion** – can be inefficient due to function call overhead; prefer loops for iterative tasks.
- **Loop unrolling** – compilers may unroll small loops to reduce overhead.

---

## 14. Security Considerations

- **Infinite loops** – can cause denial of service if not properly controlled.
- **`goto`** – can bypass variable initialisation, leading to undefined behaviour.
- **`switch` fallthrough** – accidental fallthrough can cause unexpected behaviour; always use `break` unless intentional.
- **`return`** – ensure all code paths return a value in non‑`void` functions to avoid undefined behaviour.
- **`break` / `continue`** – use carefully in nested loops; ensure they don't skip critical cleanup.

---

## 15. Debugging Tips

- **Use a debugger** – step through statements to see control flow.
- **Add print statements** – trace the execution path.
- **Use `-Wall -Wextra`** – catches many statement‑related issues (e.g., missing returns, unused variables).
- **Use `-Wswitch`** – warns about missing `case` or `default` in `switch`.
- **Use `-Wunreachable-code`** – warns about code that can't be executed.
- **Watch for dangling `else`** – use braces to make the control flow clear.

---

## 16. When to Use

- **All the time** – statements are the executable units of a C program.
- **Use the most appropriate construct** – `if` for decisions, `switch` for multi‑way branching, `for` for counted loops, `while` for indefinite loops.

---

## 17. When Not to Use

- **Avoid `goto`** for general control flow – use structured statements.
- **Avoid `switch`** for non‑integer comparisons or floating‑point values.
- **Avoid `do-while`** if the loop might need to execute zero times – use `while` instead.
- **Avoid overly nested statements** – extract logic into functions.

---

## 18. Related Concepts

- **Expressions** – statements are built from expressions.
- **Control flow** – how statements direct execution.
- **Scope** – affects visibility of declarations inside compound statements.
- **Functions** – contain statement blocks.
- **Labels** – used with `goto`.
- **Sequence points** – affect statement evaluation order.

---

## 19. Did You Know?

- In C89, declarations had to be at the beginning of a block; C99 relaxed this.
- The `for` loop's three parts (init, condition, increment) are all optional; `for (;;)` is an infinite loop.
- The `switch` statement in C can have `case` labels that are not unique.
- `goto` cannot jump into a block that is entered by a different means (e.g., you can't jump into a `for` loop).
- `break` and `continue` are only allowed in loops and `switch`; they cannot be used inside `if` statements alone.
- The `do-while` loop is the only loop that guarantees at least one execution.
- A label can appear before a declaration without a statement (though it's unusual).

---

## 20. Summary

- **Statements are the executable units** of a C program – they perform actions.
- **Types**: declaration, expression, compound (block), null, selection (`if`, `switch`), iteration (`while`, `do-while`, `for`), jump (`break`, `continue`, `goto`, `return`), and label.
- **Control flow** – selection and iteration statements direct execution flow; jump statements transfer control.
- **Best practices**: use braces for all control structures, prefer `for` for counted loops, `while` for indefinite loops, `switch` for multi‑way branching, and `goto` only for cleanup.
- **Common mistakes**: missing `break` in `switch`, dangling `else`, infinite loops, using `=` instead of `==`, and forgetting `return` in non‑void functions.
- **Understanding statements** is fundamental to writing any C program – they are the verbs that make your code do work.