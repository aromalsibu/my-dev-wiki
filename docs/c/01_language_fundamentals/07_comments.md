# Comments

## 1. Overview

### Definition
Comments are non‑executable text embedded in C source code that are ignored by the compiler. They serve as human‑readable explanations, annotations, and documentation within the code. C supports two styles: block comments (`/* ... */`) inherited from its predecessors, and line comments (`// ...`) introduced in C99.

### Purpose
Comments bridge the gap between the code (what the machine executes) and the programmer’s intent (why the code exists and how it works). They make source code more readable, aid in maintenance, document assumptions, and can even generate external documentation when used consistently.

### Where It Fits
Comments are removed during the preprocessing phase (or by the lexer) and do not appear in the token stream. They exist purely at the source level. They are vital in every C project, from a one‑off script to large‑scale systems.

---

## 2. Why It Exists

### The Problem Without Comments
Code is written once but read many times—by your future self, by colleagues, and by maintainers. Without comments, even well‑written code can be obscure. The logic might be clear today, but months later, the reasons behind a design choice or a complex algorithm may be forgotten.

### The Solution: Non‑Executable Annotations
Comments allow you to:
- Explain **what** the code does (high‑level).
- Explain **why** a particular approach was taken (rationale).
- Provide **usage examples** for functions and types.
- Temporarily **disable** code during debugging (commenting out).
- Add **metadata** (author, date, version, license).
- Generate **documentation** (e.g., Doxygen) from structured comments.

### Why It Was Introduced
Comments have been a staple of programming languages since the earliest assemblers. C’s `/* ... */` style comes from B and BCPL, allowing multi‑line comments. The `//` style, borrowed from C++ and widely used in other languages, was added in C99 for convenience and to avoid the pitfalls of nested block comments.

---

## 3. Syntax / Basic Usage

### Block Comments (`/* ... */`)
```c
/* This is a block comment.
   It can span multiple lines.
   Everything between the delimiters is ignored. */

int main(void) {
    /* This comment can appear anywhere whitespace is allowed. */
    int a = 5; /* It can even be inline. */
    return 0;
}
```

### Line Comments (`// ...`)
```c
// This is a line comment. It starts with // and goes to the end of the line.
int main(void) {
    int a = 5;  // This is an inline line comment.
    return 0;
}
```

### Complete Example with Both Styles
```c
/*
 * File:   example.c
 * Author: Jane Doe
 * Date:   2026-07-05
 * Purpose: Demonstrates comment usage.
 */

#include <stdio.h>   // Include standard I/O

int main(void) {
    // Define an array of test scores
    int scores[5] = {85, 92, 78, 90, 88};

    // Compute the average
    int sum = 0;
    for (int i = 0; i < 5; i++) {
        sum += scores[i];   // accumulate
    }

    // Cast to float to avoid integer division
    float average = sum / 5.0f;

    /* Display the result using printf.
       The format specifier %.2f prints with two decimals. */
    printf("Average score: %.2f\n", average);

    return 0;   // Success
}
```

### Code Breakdown (with comments)
```c
/*
 * File:   example.c
 * Author: Jane Doe
 * Date:   2026-07-05
 * Purpose: Demonstrates comment usage.
 */
// This is a block comment often used at the top of files for metadata.

#include <stdio.h>   // Include standard I/O
// A line comment that explains the #include directive.

int main(void) {
    // Define an array of test scores
    // Line comment describing the variable.

    int scores[5] = {85, 92, 78, 90, 88};
    // Array initialisation.

    // Compute the average
    int sum = 0;
    for (int i = 0; i < 5; i++) {
        sum += scores[i];   // accumulate
        // Inline line comment explaining the operation.
    }

    // Cast to float to avoid integer division
    float average = sum / 5.0f;
    // A comment explaining a potential pitfall (integer division).

    /* Display the result using printf.
       The format specifier %.2f prints with two decimals. */
    printf("Average score: %.2f\n", average);
    // The block comment here explains the function and its format string.

    return 0;   // Success
    // Indicating the return value meaning.
}
```

---

## 4. Mental Model – Comments as Post‑It Notes

Imagine your code is a complex machine with many levers, gears, and pipes. Comments are like sticky notes you attach to various parts:

- They explain **what** each lever does.
- They warn **why** you must not pull a certain lever too hard (edge cases).
- They give **instructions** on how to service the machine.
- You can also cover a part with a sticky note to “disable” it temporarily (commenting out).

These notes are not part of the machine; they are purely for the people who operate or maintain it. The machine (compiler) ignores them completely.

---

## 5. Core Concepts

| Concept | Explanation |
|---------|-------------|
| **Block comments** | Start with `/*` and end with `*/`. Cannot be nested (e.g., `/* /* */ */` is invalid). |
| **Line comments** | Start with `//` and continue to the end of the line. Introduced in C99; can be nested within block comments? (they are ignored inside a block comment). |
| **Whitespace** | Comments are treated as whitespace; they are token separators. |
| **Preprocessing** | Comments are removed early; they are not seen by the compiler proper. |
| **Documentation comments** | Special comments (e.g., `/** ... */` or `///`) used by tools like Doxygen to generate API documentation. |
| **Commented‑out code** | A common practice to disable code during testing, but should be removed or kept minimal to avoid clutter. |

---

## 6. How It Works – Lexical Analysis

1. **The preprocessor or lexer** reads the source character by character.
2. When it encounters `/*`:
   - It ignores everything until it finds the next `*/`.
   - It does not process any tokens inside.
   - It cannot be nested; a second `/*` inside is treated as plain text.
3. When it encounters `//`:
   - It ignores everything until the end of the line (newline character).
4. In both cases, the comment is effectively replaced by a single space (or removed), so it separates tokens.
5. The resulting token stream contains no comment tokens.

### ASCII Diagram – Comment Removal
```
Source: int x = 5; /* comment */ int y = 10;
    │
    ▼
Lexer: sees "int", "x", "=", "5", ";", then sees "/*", skips to "*/", then sees "int", "y", "=", "10", ";"
    │
    ▼
Token stream: int x = 5 ; int y = 10 ;
(no trace of the comment remains)
```

---

## 7. Internal Architecture – Preprocessor vs Compiler

- In traditional C implementations, comments are stripped by the **preprocessor** before macro expansion.
- Some compilers integrate the lexer and preprocessor; the effect is the same: comments never reach the parser.
- Because of this, you cannot use comments to influence macro expansion or token concatenation; they are simply removed.
- The preprocessor also removes `//` comments; however, in older (C89) compilers that didn't support `//`, they might be treated as an error or as a division operator.

---

## 8. Lifecycle / Workflow of a Comment

1. **Written** by the programmer in the source file.
2. **Preprocessed** – removed entirely.
3. **Compiled** – never seen by the compiler.
4. **Stored** – in source control, forever part of the source history.
5. **Read** – by developers, reviewers, and documentation tools.

---

## 9. Practical Examples

### Example 1: Block Comment with Metadata
```c
/*
 * =====================================================================
 *  Project   : MyFirmware
 *  Module    : sensor.c
 *  Author    : A. Engineer
 *  Version   : 2.1
 *  Created   : 2026-01-15
 *  Purpose   : Read data from temperature sensor via I²C.
 * =====================================================================
 */
#include <stdio.h>
#include "sensor.h"

/*  I²C address of the sensor (from datasheet) */
#define SENSOR_ADDR 0x4A

int read_temperature(float *temp_out) {
    // Implementation...
    return 0;
}
```
**Code Breakdown:**
- The large block comment at the top is a standard file header.
- It provides metadata that is invaluable for maintenance.
- The inline `#define` comment explains the magic number.

### Example 2: Commenting Out Code
```c
int main(void) {
    int a = 5, b = 10;
    // int result = a + b;   // Temporarily disabled for debugging
    int result = a * b;       // New version
    printf("Result: %d\n", result);
    return 0;
}
```
**Code Breakdown:**
- Commenting out code is useful for testing alternatives.
- Be cautious: it can make code messy if left in long‑term; prefer version control for removal.

### Example 3: Doxygen‑Style Comments
```c
/**
 * @brief  Calculate the average of an integer array.
 * @param  arr   Pointer to the array.
 * @param  n     Number of elements.
 * @return       The average as a float, or -1.0 if n == 0.
 */
float array_avg(const int *arr, size_t n) {
    if (n == 0) return -1.0f;
    long sum = 0;
    for (size_t i = 0; i < n; i++) {
        sum += arr[i];
    }
    return (float)sum / (float)n;
}
```
**Code Breakdown:**
- Doxygen uses `/**` to start a block comment that will be parsed.
- Special tags (`@brief`, `@param`, `@return`) generate structured documentation.
- This is a standard practice in many C projects.

### Example 4: Nested Block Comments – Not Allowed
```c
/* This is a comment
   /* This inner comment will cause an error */
   because the first */ closes the outer comment prematurely.
*/
```
**Code Breakdown:**
- The inner `/*` is not special; the lexer just reads it as characters.
- The first `*/` encountered ends the entire block; the outer closing `*/` becomes a stray token, causing a syntax error.
- To nest, you can use `#if 0 ... #endif` in the preprocessor, or switch to line comments.

---

## 10. Common Use Cases

- **File and function headers** – describe purpose, author, date, parameters, returns.
- **Complex algorithm explanation** – outline the steps.
- **Implementation notes** – explain non‑obvious decisions, trade‑offs, and edge cases.
- **To‑do items** – `// TODO: implement error handling`.
- **Bug references** – `// FIXME: this may overflow if input > 2^31`.
- **License and copyright** – required for open‑source code.
- **Debugging** – comment out sections to isolate issues.

---

## 11. Best Practices

- **Write comments that explain “why”, not “what”** – the code itself shows what it does; comments should clarify the reasoning.
- **Keep comments up‑to‑date** – outdated comments are worse than no comments.
- **Use consistent formatting** – e.g., use `/* ... */` for file headers, `//` for inline explanations.
- **Avoid obvious comments** – `i++; // increment i` adds no value.
- **Prefer clear code over excessive comments** – well‑named variables and functions reduce the need for comments.
- **Use documentation comments** (`/** ... */`) for public APIs to enable auto‑generation of documentation.
- **Do not comment out large blocks of code permanently** – use version control; commented‑out code clutters the file.
- **Be concise** – but not so brief that it's cryptic.
- **Use `FIXME`, `TODO`, `NOTE`** – helps with task tracking if the IDE supports it.

---

## 12. Common Mistakes

### Mistake 1: Nested Block Comments
```c
/* Outer comment
   /* Inner comment */  // This prematurely ends the outer comment
   This is now outside the comment
*/
```
**Why it’s wrong:** The first `*/` ends the comment; the second `*/` is unmatched, causing an error.
**Correct:** Use `//` for the inner explanation or avoid nesting.
```c
/* Outer comment
   // Inner comment (now using line comment, which is fine)
   This is still inside the outer comment
*/
```

### Mistake 2: Using `//` in C89 Mode
```c
// This is a line comment
int main() { return 0; }
```
**Why it’s wrong:** C89 does not recognise `//`; it may cause errors.
**Correct:** Compile with `-std=c99` or later, or use `/* ... */`.

### Mistake 3: Commenting Out Code That Contains `*/`
```c
/*
int x = 5;  // This line is fine
// But if you have a literal like "end of comment: */" inside, it will break
*/
```
**Why it’s wrong:** The `*/` inside the string literal is still recognised as the end of the block comment (the preprocessor does not parse strings). This is a classic pitfall.
**Correct:** Use `#if 0 ... #endif` for large blocks, or use line comments for each line, or escape the `*/`.

### Mistake 4: Outdated Comments
```c
/* Function adds two numbers */
int multiply(int a, int b) {
    return a * b;   // Actually multiplies, not adds
}
```
**Why it’s wrong:** The comment is misleading and will confuse maintainers.
**Correct:** Update the comment or remove it.

### Mistake 5: Over‑commenting Obvious Code
```c
int main() {
    int i = 0;  // Set i to zero
    // Loop until i is 10
    while (i < 10) {
        printf("%d", i); // print i
        i++;             // increment i
    }
}
```
**Why it’s wrong:** These comments add clutter; the code is self‑explanatory.
**Correct:** Focus comments on non‑obvious logic or rationale.

---

## 13. Performance Considerations

- Comments have **zero runtime impact** – they are removed.
- They can slightly increase compilation time (more characters to scan), but the effect is negligible.
- Excessive comments can bloat source file size, but that only affects disk storage and maybe network transfers in version control.

---

## 14. Security Considerations

- Comments can inadvertently reveal sensitive information (e.g., `// password = "admin"`). Never put secrets in comments.
- Comments may contain internal IP (intellectual property) or security‑relevant details that should not be exposed in open‑source code.
- Use comments to **warn** about security pitfalls (e.g., `// This function does not validate input length; caller must ensure`).

---

## 15. Debugging Tips

- **Temporarily comment out sections** to isolate bugs.
- **Add debugging prints** but wrap them in `#ifdef DEBUG` so they can be enabled/disabled; use comments to document.
- **Use `//` comments to quickly disable a line** without disrupting the surrounding block.
- **Be careful when commenting out code with `#if 0 ... #endif`** – that’s a preprocessor directive and works even with nested block comments.

---

## 16. When to Use

- **Always in public APIs** – document every function, parameter, and return value.
- **For complex logic** – when the code is not immediately obvious.
- **For workarounds or hacks** – explain why you had to take a non‑standard approach.
- **For legal notices** – copyright and license information.

---

## 17. When Not to Use

- **Avoid comments that duplicate the code** – they add noise.
- **Avoid comments that are not maintained** – they become harmful.
- **In code that is self‑documenting** – if variable/function names are clear, comments may be unnecessary.

---

## 18. Related Concepts

- **Preprocessor** – `#if 0` can be used for block‑commenting code.
- **Documentation generators** – Doxygen, Sphinx for C.
- **Code review** – comments help reviewers understand rationale.
- **Style guides** – e.g., Linux kernel coding style, MISRA C.

---

## 19. Did You Know?

- The `/* */` comment style originated in BCPL (1967) and was carried into B and C.
- In C99, `//` was added to align with C++ and many other languages, making C more convenient for single‑line explanations.
- Some compilers support **nested comments** as an extension (e.g., `-Wcomment` in GCC warns about nested comments).
- The preprocessor can be used to “comment out” code more safely than `/* */` because `#if 0` can contain any code, including block comments, without breaking.
- There is no standard way to write **doc comments** in C; tools like Doxygen added their own conventions (`/**`, `/*!`, etc.).

---

## 20. Summary

- **Comments are non‑executable annotations** that help humans understand the code.
- **Two styles**: block comments (`/* ... */`) and line comments (`// ...`).
- **Block comments cannot be nested**; line comments go to the end of the line.
- **Comments are removed during preprocessing** – no runtime overhead.
- **Best practices**: explain “why”, not “what”; keep them up‑to‑date; avoid over‑commenting; use documentation comments for public APIs.
- **Common mistakes**: nesting block comments, using `//` in C89, outdated comments, over‑commenting.
- **Comments are an essential tool** for communication, maintenance, and documentation in every C project.