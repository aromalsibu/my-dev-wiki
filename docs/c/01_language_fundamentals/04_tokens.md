# Tokens

## 1. Overview

### Definition
A token is the smallest unit of a C program that has meaning to the compiler. When the compiler reads your source code, it scans the characters from left to right and groups them into tokens based on defined lexical rules. These tokens are the building blocks from which the compiler constructs syntax trees and ultimately generates machine code.

### Purpose
Understanding tokens is fundamental to understanding how the C language is parsed. By grouping characters into tokens (keywords, identifiers, operators, etc.), the compiler can apply grammar rules to validate program structure. For you as a programmer, knowing the token categories helps you write correct code and avoid errors that arise from ambiguous character sequences (e.g., `++` vs `+ +`).

### Where It Fits
Tokens sit at the very first stage of the compilation pipeline, immediately after the preprocessor finishes its textual substitutions. The lexer (or scanner) reads the preprocessed source and produces a stream of tokens that the parser then consumes to build an abstract syntax tree. This is the foundation of all syntactic analysis.

---

## 2. Why It Exists

### The Problem Without Tokenisation
If the compiler did not tokenise source code, it would have to interpret character‑by‑character, struggling to differentiate between, say, `int` as a keyword and `int` as part of a variable name. Every character would need context, leading to massive complexity and ambiguity.

### The Solution: Token Categories
By defining a finite set of token types, the language becomes unambiguous. The lexer applies the **maximal munch** rule – it always takes the longest possible sequence of characters that forms a valid token. This eliminates confusion and provides a clean interface to the parser.

### Why It Was Introduced
Tokenisation is a classic principle of compiler construction, present in almost all high‑level languages. For C, it was designed to be simple, efficient, and unambiguous, enabling a fast lexical analysis that aligns with C's systems‑programming philosophy.

---

## 3. Syntax / Basic Usage

C defines six main token categories. Here’s an example of a full program and the tokens it generates.

```c
#include <stdio.h>   // Preprocessor directive (handled separately)
int main(void) {     // int: keyword, main: identifier, ( : punctuation, void: keyword, ) : punctuation, { : punctuation
    int count = 42;  // int: keyword, count: identifier, =: operator, 42: constant, ;: punctuation
    count++;         // count: identifier, ++: operator, ;: punctuation
    return 0;        // return: keyword, 0: constant, ;: punctuation
}                    // }: punctuation
```

### Tokenisation of the Program
The lexer produces the following token stream (each token includes its type and sometimes a value):

```
<keyword> int
<identifier> main
<punctuator> (
<keyword> void
<punctuator> )
<punctuator> {
<keyword> int
<identifier> count
<operator> =
<constant> 42
<punctuator> ;
<identifier> count
<operator> ++
<punctuator> ;
<keyword> return
<constant> 0
<punctuator> ;
<punctuator> }
```

### Code Breakdown (with comments)
```c
#include <stdio.h>   // This is a preprocessor directive, not a token.
                     // The preprocessor handles it before tokenisation.
int main(void) {     // int -> keyword (type specifier)
                     // main -> identifier (function name)
                     // ( -> left parenthesis (punctuator)
                     // void -> keyword (parameter type)
                     // ) -> right parenthesis (punctuator)
                     // { -> left brace (punctuator)
    int count = 42;  // int -> keyword
                     // count -> identifier (variable name)
                     // = -> assignment operator
                     // 42 -> integer constant (literal)
                     // ; -> statement terminator (punctuator)
    count++;         // count -> identifier
                     // ++ -> increment operator (two plus signs form one token)
                     // ; -> punctuation
    return 0;        // return -> keyword
                     // 0 -> integer constant
                     // ; -> punctuation
}                    // } -> right brace
```

---

## 4. Mental Model – Tokens as LEGO Bricks

Imagine your source code as a pile of individual LEGO bricks. Each token is one brick. The bricks come in different shapes and colours:

- **Keywords** – special bricks that have a fixed meaning (e.g., `int`, `if`, `while`).
- **Identifiers** – custom‑shaped bricks that you name (e.g., `count`, `calculate_sum`).
- **Constants** – bricks that represent numeric values or characters (e.g., `42`, `'a'`).
- **Strings** – bricks that represent text sequences (e.g., `"hello"`).
- **Operators** – bricks that connect other bricks to perform operations (e.g., `+`, `*`, `=`).
- **Punctuators** – bricks that separate and structure the assembly (e.g., `;`, `(`, `)`).

The compiler’s lexer sorts through your character stream and snaps these bricks apart, grouping them into meaningful tokens. The parser then takes these tokens and checks that they are assembled according to the “instruction manual” – the grammar of C – to build a valid program.

---

## 5. Core Concepts

| Token Type | Description | Examples |
|------------|-------------|----------|
| **Keywords** | Reserved words with predefined meaning. They cannot be used as identifiers. | `auto`, `break`, `case`, `char`, `const`, `continue`, `default`, `do`, `double`, `else`, `enum`, `extern`, `float`, `for`, `goto`, `if`, `inline`, `int`, `long`, `register`, `restrict`, `return`, `short`, `signed`, `sizeof`, `static`, `struct`, `switch`, `typedef`, `union`, `unsigned`, `void`, `volatile`, `while`, `_Bool`, `_Complex`, `_Imaginary` (plus some C11/C23 additions like `_Alignas`, `_Alignof`, `_Atomic`, `_Generic`, `_Noreturn`, `_Static_assert`, `_Thread_local`). |
| **Identifiers** | Names given to variables, functions, structs, etc. Must start with a letter or underscore, followed by letters, digits, or underscores. Case‑sensitive. | `count`, `sum`, `_temp`, `MAX_VALUE` |
| **Constants (Literals)** | Fixed values that do not change. Includes integer, floating‑point, character, and enumeration constants. | `123`, `3.14`, `'A'`, `'\n'` |
| **String Literals** | Sequences of characters enclosed in double quotes. Actually a char array with a null terminator. | `"hello"`, `"Line 1\n"` |
| **Operators** | Symbols that perform computations or actions. They can be unary, binary, or ternary. | `+`, `-`, `*`, `/`, `%`, `=`, `==`, `!=`, `&&`, `||`, `++`, `--`, `&`, `*` (dereference), `? :` |
| **Punctuators** | Symbols that have syntactic meaning but are not operators. They separate tokens and define structure. | `;`, `{`, `}`, `(`, `)`, `[`, `]`, `.`, `->`, `,`, `:` |
| **Preprocessing Tokens** | These exist only during preprocessing and are not part of the C token stream seen by the compiler. They include `#`, `##`, and macro names. | `#define`, `#include`, `#ifdef` |

---

## 6. How It Works – Step‑by‑Step Tokenisation

1. **The lexer reads the preprocessed source code character by character.**
2. **It skips whitespace characters** (space, newline, tab) – they are not tokens.
3. **It applies the “maximal munch” rule** – for a given starting point, it consumes the longest sequence that forms a valid token.
   - For example, `++` is a single token (increment operator), not two `+` tokens.
4. **It classifies the token**:
   - If the sequence matches a keyword, it is a keyword token.
   - If it matches a number or character literal, it’s a constant token.
   - If it matches an identifier pattern, it’s an identifier token.
   - If it matches an operator/punctuator, it is classified accordingly.
5. **The token is passed to the parser**, along with its type and possibly its value (e.g., the integer value for a numeric constant).
6. **The process repeats** until the entire source file is consumed.

### ASCII Diagram – The Lexer Pipeline
```
Source characters: "int x = 5;"
        │
        ▼
   ┌────────────┐
   │   Lexer    │  (scans, skips whitespace, applies maximal munch)
   └────────────┘
        │
        ▼
   Token stream: [KEYWORD(int)] [IDENTIFIER(x)] [OPERATOR(=)] [CONSTANT(5)] [PUNCTUATOR(;)]
        │
        ▼
   ┌────────────┐
   │   Parser   │
   └────────────┘
```

---

## 7. Internal Architecture – Lexer Implementation Concepts

In a typical C compiler, the lexer is implemented as a finite state machine (FSM) or uses regular expressions. The C standard defines the syntax of tokens using a context‑free grammar; the lexer implements that grammar at the character level.

- **Keywords** are often stored in a hash table for fast lookup – if an identifier matches one of them, it is reclassified as a keyword.
- **Identifiers** are checked for conformance to the naming rules and stored in a symbol table (for later use by the compiler).
- **Constants** are parsed and converted to internal representations (integer, floating‑point, etc.) for use in expressions.
- **String literals** are stored in the literal pool (often in the `.rodata` section) and referenced by a pointer.
- **Comment and whitespace removal** happens before tokenisation (comments are replaced by whitespace during preprocessing, though the lexer often handles it).

---

## 8. Lifecycle / Workflow of a Token

1. **Creation** – the lexer extracts a token from the source text.
2. **Classification** – its type is determined.
3. **Consumption** – the parser takes the token and processes it according to the grammar.
4. **Discard** – after use, the token can be discarded; only the AST and symbol tables persist.
5. **Error handling** – if the lexer cannot recognise a character sequence as a token, it produces a lexical error (e.g., an illegal character like `@` in C).

---

## 9. Practical Examples

### Example 1: Ambiguity Resolved by Maximal Munch
```c
int x = 1+2;    // Tokens: int, x, =, 1, +, 2, ;
int y = 1+++2;  // Lexer sees: 1, ++, +, 2 → (1++)+2, not 1+(++2) or 1++(+2)
```

**Code Breakdown:**
- The lexer, when reading `1+++2`, consumes characters:
  - `1` – constant.
  - Next char `+`, next char `+` → forms `++` (maximal munch) – increment operator.
  - Next char `+` – it cannot form `+++` (invalid token), so it takes just `+`.
  - `2` – constant.
- This results in a syntax error if used incorrectly (e.g., applying `++` to a constant), but illustrates the tokenisation rule.

### Example 2: String and Character Tokens
```c
char c = 'A';       // Tokens: char, c, =, 'A' (character constant), ;
char *s = "Hello";  // Tokens: char, *, s, =, "Hello" (string literal), ;
```

**Code Breakdown:**
- `'A'` is a character constant token – the lexer parses the single quote and the character inside.
- `"Hello"` is a string literal token – the lexer scans until the closing quote, respecting escape sequences like `\n`.

### Example 3: Operators with Multiple Characters
```c
int a = 5, b = 10;
if (a == b) { ... }   // == is one token (equality operator), not two = tokens.
a <<= 3;               // <<= is one token (shift‑and‑assign), not << and =.
```

**Code Breakdown:**
- `==` – maximal munch consumes `=` then sees another `=`, forming a single token.
- `<<=` – similarly, `<<` is taken, then `=` forms a combined token.

### Example 4: Comments Are Not Tokens
```c
/* This is a comment – it is removed by the preprocessor/lexer */
int main(void) { // This is another comment
    return 0;    /* Inline comment */
}
```
- Comments are stripped before tokenisation; they do not appear in the token stream.

---

## 10. Common Use Cases

- **Understanding compiler errors** – many errors come from mis‑tokenisation (e.g., using a keyword as an identifier, or missing a semicolon which is a punctuator token).
- **Code formatting** – understanding token boundaries helps in writing linters and code formatters.
- **Macro expansion** – macros are expanded before tokenisation; sometimes the resulting tokens can be surprising.
- **Writing parsers** – if you ever need to write a C parser or transpiler, tokenisation is the first step.

---

## 11. Best Practices

- **Avoid ambiguous sequences** – for readability, add spaces around operators; it reduces confusion for human readers even if the lexer handles it.
- **Do not use keywords as identifiers** – the compiler will reject it.
- **Use meaningful identifiers** – they are tokens; clear names improve readability.
- **Use constant names (via `#define` or `const`)** – they help avoid “magic numbers” in code.
- **String literals** – use const char* for read‑only strings to avoid undefined behaviour.

---

## 12. Common Mistakes

### Mistake 1: Using a Keyword as an Identifier
```c
// ❌ Wrong – int is a keyword, cannot be used as a variable name
int int = 5;
// ✅ Correct – choose a different name
int value = 5;
```

### Mistake 2: Forgetting the Semicolon (Punctuator)
```c
// ❌ Wrong – missing semicolon token
int x = 5
int y = 10;   // Syntax error
// ✅ Correct – add semicolon
int x = 5;
```

### Mistake 3: Misplacing Spaces That Change Tokenisation
```c
// ❌ Wrong – spaces affect tokenisation in some cases
int a = 10;
++ a;   // This is parsed as ++ followed by a? Actually the space doesn't affect tokenisation
        // but it's still invalid syntax because ++ applied to a variable requires a lvalue.
// ✅ Correct – use proper spacing for readability
++a;
```

### Mistake 4: Using a Character Constant with Multiple Characters
```c
// ❌ Wrong – multi‑character constant is implementation‑defined, usually not what you want
char c = 'AB';   // Not a valid single character
// ✅ Correct – use a string for multiple characters
const char *s = "AB";
```

---

## 13. Performance Considerations

- **Tokenisation is typically fast** – it’s a simple linear scan with minimal overhead.
- **Whitespace and comments** – they are skipped, so the number of spaces does not affect performance.
- **Macros** – excessive macro usage can create many tokens after expansion, slowing down compilation.
- **String concatenation** – adjacent string literals are concatenated during preprocessing, which can produce a single token after expansion.

---

## 14. Security Considerations

- **String literal boundaries** – be careful with input that may contain null bytes or quote characters; in C, a string literal cannot contain a null byte (it ends the string). This is more about runtime security.
- **Preprocessor injection** – tokens generated from macro expansion can lead to unexpected code if macros are misused; this is a build‑time concern.

---

## 15. Debugging Tips

- **Use `-E` to see preprocessed output** – this shows the code after preprocessing, before tokenisation, which can help you see how macros expand.
- **Compile with `-save-temps`** to keep the preprocessed file (`.i`) – you can inspect it to see exactly what the lexer receives.
- **Use a tokeniser tool** – some compilers can dump token streams (e.g., `clang -Xclang -dump-tokens`).
- **Watch for cryptic error messages** – many parser errors stem from a missing semicolon (a punctuator token); the compiler often points to the next line.

---

## 16. When to Use

- **All the time** – tokens are an integral part of writing C code. Understanding them helps you write syntactically correct code.
- **When reading compiler error messages** – they often refer to unexpected tokens.

---

## 17. When Not to Use

- **You cannot avoid tokens** – they are a fundamental part of the language.

---

## 18. Related Concepts

- **Lexical analysis** – the process of tokenisation.
- **Parsing** – the next step after tokenisation.
- **Preprocessor** – runs before tokenisation and alters the source text.
- **Regular expressions** – often used to define token patterns.
- **Finite automata** – theoretical basis for lexers.

---

## 19. Did You Know?

- The C standard uses the term “token” in a very precise way; for example, it defines a **preprocessing token** separately, which includes things like `#` and `##` that are not part of the C token stream after preprocessing.
- C compilers are allowed to use the **“maximal munch”** rule, but they must also consider that the token sequence must form a valid program.
- The longest token in C is the `long long` keyword? Or perhaps `_Complex`? In terms of characters, `_Alignof` is relatively long, but all are fixed.

---

## 20. Summary

- **Tokens are the smallest syntactic units** in a C program; they are produced by the lexer.
- **There are six categories**: keywords, identifiers, constants, string literals, operators, and punctuators.
- **The lexer applies the maximal munch rule** – it always takes the longest possible token.
- **Whitespace and comments are not tokens** – they are skipped.
- **Every C program is a sequence of tokens**; the parser consumes them to build the program structure.
- **Understanding tokens** helps in writing correct code, reading error messages, and appreciating the compiler's inner workings.