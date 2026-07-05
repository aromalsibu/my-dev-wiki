# Functions

## 1. Overview

### Definition
A function in C is a self‑contained block of code that performs a specific task. It encapsulates logic, accepts parameters (input), and optionally returns a value (output). Functions are the fundamental building blocks of modular programming in C, allowing you to break complex problems into smaller, manageable pieces.

### Purpose
Functions serve several essential purposes:
- **Modularity** – break large programs into smaller, logical units.
- **Reusability** – write code once and use it multiple times.
- **Abstraction** – hide implementation details behind a clean interface.
- **Maintainability** – isolate changes to a single function.
- **Testability** – test each function independently.
- **Code organisation** – group related functionality together.

### Where It Fits
Functions are the primary organisational unit in C. A C program consists of one or more functions, with `main()` being the mandatory entry point. Functions are declared in headers (`.h`) and defined in source files (`.c`), enabling separate compilation and modular development.

---

## 2. Why It Exists

### The Problem Without Functions
Without functions, every program would be a single, monolithic block of code. This leads to:
- **Code duplication** – the same logic written multiple times.
- **Poor readability** – thousands of lines of linear code.
- **Hard maintenance** – a change ripples through the entire program.
- **Difficult debugging** – no clear separation of concerns.
- **No reuse** – you can't use logic from one program in another.

### The Solution: Functions
Functions provide a way to:
- **Encapsulate logic** – group related statements into a named unit.
- **Create interfaces** – define what a function does without exposing how.
- **Support recursion** – functions can call themselves.
- **Enable libraries** – share reusable code across projects.
- **Facilitate testing** – test individual functions in isolation.

### Why They Were Introduced
Functions were a key innovation in high‑level languages, enabling structured programming. C's function model was influenced by ALGOL and BCPL, providing a simple but powerful mechanism for procedural abstraction. The ability to compile functions separately (separate compilation) was essential for building large systems like Unix.

---

## 3. Syntax / Basic Usage

### Function Declaration (Prototype) and Definition
```c
#include <stdio.h>

// Function prototype (declaration) – tells the compiler about the function.
int add(int a, int b);

// Function definition – the actual implementation.
int add(int a, int b) {
    return a + b;
}

int main(void) {
    // Function call
    int result = add(5, 3);
    printf("Result: %d\n", result);  // 8
    return 0;
}
```

### Functions with Different Signatures
```c
#include <stdio.h>
#include <stdbool.h>

// Function with no parameters and no return value.
void print_hello(void) {
    printf("Hello, World!\n");
}

// Function with parameters but no return value.
void print_sum(int a, int b) {
    printf("Sum: %d\n", a + b);
}

// Function with parameters and a return value.
int multiply(int a, int b) {
    return a * b;
}

// Function returning void, but with early return.
void check_positive(int x) {
    if (x <= 0) {
        return;   // Early exit – no value.
    }
    printf("Positive: %d\n", x);
}

// Function returning a boolean.
bool is_even(int x) {
    return x % 2 == 0;
}

int main(void) {
    print_hello();                    // void, no args
    print_sum(10, 20);                // void, with args
    int product = multiply(5, 3);     // returns int
    check_positive(-5);               // early return
    if (is_even(4)) {
        printf("4 is even\n");
    }
    return 0;
}
```

### Function with Void Parameter
```c
// In C, an empty parameter list means any number of arguments.
void func();         // ❌ Not recommended – no parameter information.

// Use void to explicitly say "no parameters".
void func(void);     // ✅ Correct – no parameters.
```

### Function with Array Parameters
```c
#include <stdio.h>

// Arrays decay to pointers when passed to functions.
void print_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

// Equivalent to the above – the compiler sees `int *arr`.
void print_array_ptr(int *arr, int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

// Modifying array elements.
void double_elements(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        arr[i] *= 2;   // modifies the original array.
    }
}

int main(void) {
    int arr[5] = {1, 2, 3, 4, 5};
    print_array(arr, 5);
    double_elements(arr, 5);
    print_array(arr, 5);   // Output: 2 4 6 8 10
    return 0;
}
```

### Function with Pointer Parameters
```c
#include <stdio.h>

// Swap two integers using pointers.
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// Pass by value – does NOT modify original.
void bad_swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
}

int main(void) {
    int x = 10, y = 20;
    printf("Before: x=%d, y=%d\n", x, y);
    bad_swap(x, y);
    printf("After bad_swap: x=%d, y=%d\n", x, y);   // Unchanged
    swap(&x, &y);
    printf("After swap: x=%d, y=%d\n", x, y);       // Swapped!
    return 0;
}
```

### Function with Const Parameters
```c
#include <stdio.h>

// Const prevents modification of the data pointed to.
void print_readonly(const int *arr, int size) {
    // arr[0] = 10;   // ❌ Error – cannot modify const data.
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main(void) {
    int arr[3] = {1, 2, 3};
    print_readonly(arr, 3);
    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>

// Function declaration (prototype):
// Tells the compiler: "There is a function called add,
// which takes two ints and returns an int."
int add(int a, int b);

// Function definition:
// The actual implementation of the function.
// The function body is enclosed in curly braces.
int add(int a, int b) {
    // a and b are parameters – local variables.
    // The return statement returns the sum and exits the function.
    return a + b;   // The value of a + b is returned.
}

// Void function – no return value.
// void parameters means no arguments.
void print_message(void) {
    // No return statement needed, but can be used for early exit.
    printf("Message\n");
}

// Function with a pointer parameter – can modify caller's variable.
void increment(int *x) {
    // Dereference the pointer to access the caller's variable.
    (*x)++;   // Equivalent to: *x = *x + 1;
}

// Function returning a pointer – must be careful with lifetime.
int *create_array(int size) {
    // ❌ Warning: returning a pointer to a local variable is wrong!
    // int local_arr[10];   // local_arr destroyed when function returns.
    // return local_arr;    // Dangling pointer – undefined behaviour.

    // ✅ Correct: use malloc for memory that persists.
    int *arr = (int *)malloc(size * sizeof(int));
    return arr;   // Caller must free this memory.
}

int main(void) {
    // Function call – passes arguments to the function.
    int result = add(5, 3);   // result = 8

    // Calling a void function – no return value to store.
    print_message();

    // Passing a pointer – allows the function to modify x.
    int x = 10;
    increment(&x);   // x becomes 11
    printf("x: %d\n", x);

    // Returning a pointer from malloc – caller must free.
    int *arr = create_array(10);
    // use arr...
    free(arr);   // Free the allocated memory.

    return 0;
}
```

---

## 4. Mental Model – Functions as Machines in a Factory

Imagine a factory with many machines:

- **Function declaration (prototype)** – like a sign on the factory wall: "Machine X: takes two pieces of steel (ints) and produces one piece of iron (int)." It tells everyone what the machine does, but not how it works.

- **Function definition** – the actual machine. Inside, it takes raw materials (parameters), processes them, and outputs a product (return value). The details of how it works are hidden inside the machine.

- **Function call** – like sending materials to the machine. You put the raw materials in, the machine works, and you collect the product.

- **Parameters** – the raw materials you feed into the machine. They are copies of your data (pass by value), so the machine can't change your original materials unless you send a "pointer" that tells it where they are.

- **Return value** – the finished product the machine produces.

- **`void` functions** – machines that don't produce a product; they do work (side effects) like painting or welding.

- **`void` parameter list** – a machine that accepts no raw materials.

---

## 5. Core Concepts

### Function Components
| Component | Description | Example |
|-----------|-------------|---------|
| **Declaration (prototype)** | Function signature without body | `int add(int a, int b);` |
| **Definition** | Function signature with body | `int add(int a, int b) { return a+b; }` |
| **Call** | Invoking the function | `add(5, 3);` |
| **Parameters** | Input values (local variables) | `int a, int b` |
| **Arguments** | Values passed when calling | `5, 3` |
| **Return type** | Type of the returned value | `int`, `void`, etc. |
| **Return statement** | Exits function, returns value | `return sum;` |

### Parameter Passing
- **Pass by value** – a copy of the argument is passed. Modifying the parameter does not affect the original.
- **Pass by reference (via pointers)** – a pointer to the original is passed. The function can modify the original.

### Function Prototypes
- Declared before use so the compiler knows the function's signature.
- Located in headers (`.h`) for sharing across files.
- If a function is used before its declaration, the compiler assumes `int` return (in C89), which is dangerous.

### Return Types
- **`void`** – no return value.
- **`int`** – returns an integer.
- **`float`**, **`double`**, etc.
- **Pointers** – returns a memory address.
- **Structures/Unions** – returns a struct by value (copies the struct).

### Function Scope
- **File scope** – if declared at the top level (outside any function), it's visible in the file (and with `extern`, across files).
- **Block scope** – functions cannot be nested in C (but you can declare a function inside a block, which is unusual).

### Linkage
- **External** – default for functions; visible across translation units.
- **Internal** – `static` functions; visible only within the translation unit.

---

## 6. How It Works – Function Call Mechanics

### Step‑by‑Step
1. **Caller prepares arguments** – evaluates the arguments.
2. **Function call** – control is transferred to the function.
3. **Stack frame creation** – a new frame is pushed onto the stack:
   - Return address (where to go back).
   - Saved base pointer (to restore the caller's frame).
   - Space for local variables.
   - Space for parameters (or they are passed via registers).
4. **Parameter initialisation** – arguments are copied into parameters.
5. **Function body executes** – statements run.
6. **Return** – when `return` is reached, the return value is stored.
7. **Stack frame pop** – the frame is removed (local variables destroyed).
8. **Control returns** – to the caller, with the return value.

### Call Stack Diagram
```
Before call:                     During call:
┌──────────────────┐            ┌──────────────────┐
│ Caller's frame   │            │ Caller's frame   │
│  local vars      │            │  local vars      │
│  ...             │            │  ...             │
├──────────────────┤            ├──────────────────┤
│                  │            │ Called function  │
│                  │            │  parameters      │
│                  │            │  return addr     │
│                  │            │  saved base ptr  │
│                  │            │  local vars      │
├──────────────────┤            ├──────────────────┤
│ Stack pointer    │            │ Stack pointer    │
└──────────────────┘            └──────────────────┘
```

---

## 7. Internal Architecture – Function Representation

### Machine Code
- Each function is compiled to a sequence of machine instructions.
- The function's entry point is a label (symbol) in the object file.
- The compiler generates prologue/epilogue code:
  - **Prologue** – sets up the stack frame (saves `ebp`, allocates local variables).
  - **Epilogue** – tears down the stack frame (restores `ebp`, returns).

### Symbol Table
- The function's name is a symbol in the object file.
- For external linkage, the symbol is exported; for `static`, it's local.

### Calling Conventions
- Determines how arguments are passed (registers vs. stack), who cleans up the stack, etc.
- Common conventions: `cdecl` (C default), `stdcall`, `fastcall`.

### Function Prologue/Epilogue Example (x86)
```
Prologue (push ebp; mov ebp, esp; sub esp, N):
    push ebp           ; Save caller's base pointer.
    mov ebp, esp       ; Set base pointer to current stack.
    sub esp, 16        ; Allocate space for locals.

Epilogue (mov esp, ebp; pop ebp; ret):
    mov esp, ebp       ; Deallocate locals.
    pop ebp            ; Restore caller's base pointer.
    ret                ; Return to caller.
```

---

## 8. Lifecycle / Workflow of a Function

1. **Declaration** – introduced in code (header or before use).
2. **Definition** – body is implemented.
3. **Compilation** – compiled to machine code.
4. **Linking** – if external, linked with other object files.
5. **Call** – control transferred to the function at runtime.
6. **Execution** – instructions run.
7. **Return** – control returns to caller.

---

## 9. Practical Examples

### Example 1: Basic Arithmetic Functions
```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}

int multiply(int a, int b) {
    return a * b;
}

float divide(int a, int b) {
    if (b == 0) {
        printf("Error: Division by zero\n");
        return 0.0f;
    }
    return (float)a / b;
}

int main(void) {
    int x = 10, y = 3;
    printf("Add: %d\n", add(x, y));
    printf("Sub: %d\n", subtract(x, y));
    printf("Mul: %d\n", multiply(x, y));
    printf("Div: %.2f\n", divide(x, y));
    return 0;
}
```
**Code Breakdown:**
- Each function performs a simple arithmetic operation.
- `divide` checks for division by zero and returns 0 on error.
- `divide` casts `a` to `float` to perform floating‑point division.

### Example 2: String Manipulation Functions
```c
#include <stdio.h>
#include <string.h>   // for strlen

// Custom function to get string length.
size_t string_length(const char *str) {
    size_t len = 0;
    while (str[len] != '\0') {
        len++;
    }
    return len;
}

// Custom function to copy a string.
void string_copy(char *dest, const char *src) {
    int i = 0;
    while (src[i] != '\0') {
        dest[i] = src[i];
        i++;
    }
    dest[i] = '\0';   // null‑terminate
}

// Custom function to concatenate strings.
void string_concat(char *dest, const char *src) {
    // Find the end of dest.
    int i = 0;
    while (dest[i] != '\0') {
        i++;
    }
    // Copy src to the end of dest.
    int j = 0;
    while (src[j] != '\0') {
        dest[i + j] = src[j];
        j++;
    }
    dest[i + j] = '\0';
}

int main(void) {
    char buffer[100] = "Hello";
    char src[] = " World!";

    printf("Length of '%s': %zu\n", buffer, string_length(buffer));
    printf("Length of '%s': %zu\n", src, string_length(src));

    string_concat(buffer, src);
    printf("Concatenated: '%s'\n", buffer);

    char copy[100];
    string_copy(copy, buffer);
    printf("Copied: '%s'\n", copy);

    return 0;
}
```
**Code Breakdown:**
- `string_length` iterates through the string until it finds the null terminator.
- `string_copy` copies characters from source to destination and adds a null terminator.
- `string_concat` finds the end of the destination and copies the source there.
- All functions operate on `char*` arrays.

### Example 3: Functions Returning Pointers
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Allocate memory for a string copy.
char *duplicate_string(const char *src) {
    size_t len = strlen(src);
    char *dest = (char *)malloc((len + 1) * sizeof(char));
    if (dest == NULL) {
        return NULL;   // allocation failed
    }
    strcpy(dest, src);
    return dest;
}

// Function returning a pointer to a static buffer.
char *get_default_message(void) {
    static char msg[] = "Default message";
    return msg;   // Safe: static lifetime.
}

// Function returning a pointer to a local variable – ❌ UNSAFE.
char *bad_function(void) {
    char local[] = "Local string";   // automatic storage
    return local;   // ❌ Dangling pointer – undefined behaviour!
}

int main(void) {
    // Using duplicate_string.
    char *dup = duplicate_string("Hello, World!");
    if (dup != NULL) {
        printf("Duplicated: %s\n", dup);
        free(dup);   // Must free.
    }

    // Using a static buffer.
    char *msg = get_default_message();
    printf("Message: %s\n", msg);   // Safe, no free needed.

    // ❌ Danger – do NOT do this.
    // char *bad = bad_function();
    // printf("%s\n", bad);   // Undefined behaviour.

    return 0;
}
```
**Code Breakdown:**
- `duplicate_string` allocates memory on the heap, returning a pointer that the caller must `free`.
- `get_default_message` returns a pointer to a static local; safe because static variables persist.
- `bad_function` returns a pointer to an automatic variable – this is a dangling pointer and should never be done.

### Example 4: Functions with Function Pointers (Callbacks)
```c
#include <stdio.h>

// A simple function that adds 5 to a number.
int add_five(int x) {
    return x + 5;
}

// A function that doubles a number.
int double_number(int x) {
    return x * 2;
}

// A function that applies a function to an array.
void apply_to_array(int arr[], int size, int (*func)(int)) {
    for (int i = 0; i < size; i++) {
        arr[i] = func(arr[i]);
    }
}

// A function that takes a function pointer as a parameter.
int process(int x, int (*operation)(int)) {
    return operation(x);
}

int main(void) {
    int arr[5] = {1, 2, 3, 4, 5};

    // Apply add_five to the array.
    apply_to_array(arr, 5, add_five);
    printf("After add_five: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);   // 6 7 8 9 10
    }
    printf("\n");

    // Apply double_number to the array.
    apply_to_array(arr, 5, double_number);
    printf("After double: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);   // 12 14 16 18 20
    }
    printf("\n");

    // Using a function pointer directly.
    int result = process(10, double_number);
    printf("process(10, double_number): %d\n", result);   // 20

    return 0;
}
```
**Code Breakdown:**
- A function pointer `int (*func)(int)` is a variable that holds the address of a function taking an `int` and returning an `int`.
- `apply_to_array` takes a function pointer and applies it to each element.
- This pattern is called a **callback** and is used in many contexts (sorting, event handling).

### Example 5: Functions with Variable Arguments (Variadic)
```c
#include <stdio.h>
#include <stdarg.h>   // for va_list, va_start, va_arg, va_end

// A function that sums any number of integers.
// The first argument is the number of integers to sum.
int sum(int count, ...) {
    va_list args;          // Declare a variable to hold the arguments.
    va_start(args, count); // Initialise args with the number of arguments.

    int total = 0;
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);   // Get the next argument as an int.
    }

    va_end(args);          // Clean up.
    return total;
}

// Function to print a variable number of strings.
void print_messages(int count, ...) {
    va_list args;
    va_start(args, count);

    for (int i = 0; i < count; i++) {
        char *msg = va_arg(args, char *);
        printf("%s ", msg);
    }
    printf("\n");
    va_end(args);
}

int main(void) {
    int s1 = sum(3, 10, 20, 30);      // 60
    int s2 = sum(5, 1, 2, 3, 4, 5);   // 15
    printf("Sum 1: %d, Sum 2: %d\n", s1, s2);

    print_messages(3, "Hello", "World", "!");   // Hello World !
    return 0;
}
```
**Code Breakdown:**
- `stdarg.h` provides macros for handling variable arguments.
- `va_list` – declares a variable to hold the argument list.
- `va_start(args, last)` – initialises `args`; `last` is the last named parameter.
- `va_arg(args, type)` – retrieves the next argument, cast to the given type.
- `va_end(args)` – cleans up.
- Important: There is **no type safety** – the programmer must ensure the correct types are passed.

### Example 6: Functions with Static Local Variables
```c
#include <stdio.h>

// Function with a static counter – persists across calls.
int next_id(void) {
    static int id = 0;   // initialised once.
    return id++;
}

// Function with a static flag for one‑time initialisation.
int get_configuration(void) {
    static int initialised = 0;
    static int config = 0;

    if (!initialised) {
        // Simulate loading configuration once.
        config = 42;   // Load from file, etc.
        initialised = 1;
        printf("Configuration loaded\n");
    }
    return config;
}

int main(void) {
    printf("ID: %d\n", next_id());   // 0
    printf("ID: %d\n", next_id());   // 1
    printf("ID: %d\n", next_id());   // 2

    printf("Config: %d\n", get_configuration());   // loads once
    printf("Config: %d\n", get_configuration());   // returns cached value
    return 0;
}
```
**Code Breakdown:**
- `static` locals are initialised once, at program startup.
- They retain their value between function calls.
- This is useful for counters, one‑time initialisation, and caching.

### Example 7: Recursion (Factorial)
```c
#include <stdio.h>

// Recursive function to compute factorial.
long long factorial(int n) {
    if (n <= 1) {
        return 1;   // base case
    }
    return n * factorial(n - 1);   // recursive case
}

// Recursive function to compute Fibonacci.
long long fibonacci(int n) {
    if (n <= 1) {
        return n;
    }
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// Tail recursion – can be optimised by the compiler.
long long factorial_tail(int n, long long acc) {
    if (n <= 1) {
        return acc;
    }
    return factorial_tail(n - 1, n * acc);
}

int main(void) {
    printf("Factorial 5: %lld\n", factorial(5));         // 120
    printf("Factorial 10: %lld\n", factorial(10));       // 3628800
    printf("Fibonacci 10: %lld\n", fibonacci(10));       // 55
    printf("Factorial tail 5: %lld\n", factorial_tail(5, 1)); // 120
    return 0;
}
```
**Code Breakdown:**
- Recursion is a function calling itself.
- Must have a **base case** to stop recursion (otherwise infinite recursion → stack overflow).
- Tail recursion can be optimised by the compiler to avoid growing the stack (if supported).

---

## 10. Common Use Cases

| Scenario | Example |
|----------|---------|
| **Encapsulate logic** | `int max(int a, int b)` |
| **Perform a task** | `void print_error(const char *msg)` |
| **Calculate a value** | `double sqrt(double x)` |
| **Allocate resources** | `FILE *open_file(const char *name)` |
| **Release resources** | `void close_file(FILE *f)` |
| **Process data** | `void sort(int arr[], int n)` |
| **Callback / Event handling** | `void register_callback(void (*cb)())` |
| **Factory pattern** | `struct *create_object()` |

---

## 11. Best Practices

### General
- **Keep functions small** – a function should do one thing and do it well.
- **Use descriptive names** – `calculate_average` is better than `calc`.
- **Use `const` for read‑only parameters** – documents intent and enables optimisations.
- **Check for errors** – always validate inputs (e.g., `NULL` pointers, boundary conditions).
- **Minimise side effects** – a function should ideally only affect its parameters and return a value.
- **Use `static` for file‑private functions** – encapsulation.

### Parameter Design
- **Pass by value for small types** – `int`, `char`, etc.
- **Pass by pointer for large structs** – avoids copying.
- **Pass by pointer when you need to modify** – allows the function to change the original.
- **Use `const` pointers for read‑only access** – `const int *arr`.
- **Avoid too many parameters** – if more than 4-5, consider grouping them in a struct.

### Return Values
- **Return `int` status for error handling** – e.g., `0` for success, non‑zero for error.
- **Use `void` if no return value** – but remember you can still return early.
- **Be careful returning pointers** – ensure the pointed‑to memory persists.
- **Don't return a pointer to a local variable** – undefined behaviour.

### Function Prototypes
- **Always prototype functions** before use.
- **Put prototypes in headers** – for sharing across files.
- **Use `void` for empty parameter lists** – `int func(void)`.
- **Use `extern` for function declarations in headers** – though functions are `extern` by default.

### Recursion
- **Ensure a base case** – prevents infinite recursion.
- **Prefer iteration over recursion** for performance (unless recursion is clearer).
- **Be aware of stack depth** – recursion can cause stack overflow.

---

## 12. Common Mistakes

### Mistake 1: Missing Prototype
```c
// ❌ Wrong – no prototype before use.
int main(void) {
    int x = add(5, 3);   // Compiler assumes int add(int, int).
    return 0;
}
int add(int a, int b) {
    return a + b;
}
// ✅ Correct – add prototype before main.
int add(int a, int b);
int main(void) { ... }
```

### Mistake 2: Returning a Pointer to a Local Variable
```c
// ❌ Wrong – returns pointer to local (dangling).
int *bad_function(void) {
    int x = 10;
    return &x;   // x is destroyed when function returns.
}
// ✅ Correct – use static or malloc.
int *good_function(void) {
    static int x = 10;
    return &x;
}
```

### Mistake 3: Mismatched Function Signature
```c
// ❌ Wrong – prototype says int, but definition returns float.
int add(int a, int b);
float add(int a, int b) { return (float)a + b; }   // Mismatch.
// ✅ Correct – match the prototype.
int add(int a, int b) { return a + b; }
```

### Mistake 4: Using an Uninitialised Variable Return
```c
// ❌ Wrong – path where return is missing.
int max(int a, int b) {
    if (a > b) {
        return a;
    }
    // No return if a <= b – undefined behaviour.
}
// ✅ Correct – handle all paths.
int max(int a, int b) {
    if (a > b) {
        return a;
    } else {
        return b;
    }
}
```

### Mistake 5: Forgetting to Check `malloc` Return
```c
// ❌ Wrong – no NULL check.
int *arr = (int *)malloc(100 * sizeof(int));
arr[0] = 10;   // May crash if malloc failed.
// ✅ Correct – check for NULL.
int *arr = (int *)malloc(100 * sizeof(int));
if (arr == NULL) {
    // handle error.
}
```

### Mistake 6: Modifying a `const` Parameter
```c
// ❌ Wrong – trying to modify const data.
void print(const int *arr, int size) {
    arr[0] = 10;   // Error: arr[0] is read‑only.
}
// ✅ Correct – use const only for read‑only data.
```

### Mistake 7: Forgetting to Include `stdarg.h` for Variadic Functions
```c
// ❌ Wrong – missing header.
int sum(int count, ...) {
    va_list args;   // Error: va_list not defined.
    // ...
}
// ✅ Correct – include <stdarg.h>.
#include <stdarg.h>
```

### Mistake 8: Incorrect Use of `va_arg` with Wrong Type
```c
// ❌ Wrong – mismatched type.
int sum(int count, ...) {
    va_list args;
    va_start(args, count);
    for (int i = 0; i < count; i++) {
        // If arguments are doubles, this will read garbage.
        int val = va_arg(args, int);
    }
    va_end(args);
}
// ✅ Correct – ensure type matches.
```

### Mistake 9: Function with Too Many Parameters
```c
// ❌ Wrong – hard to read and use.
void process_order(int id, int qty, float price, char *name, int stock, int status) { ... }
// ✅ Better – group in a struct.
struct Order { int id; int qty; float price; char *name; int stock; int status; };
void process_order(struct Order *order) { ... }
```

---

## 13. Performance Considerations

- **Function call overhead** – small, but can add up in tight loops. Use `inline` for small functions.
- **Pass by value vs pointer** – for large structs, pass by pointer to avoid copying.
- **Recursion** – can be slower and use more stack; consider iteration for performance‑critical code.
- **`static` functions** – can be inlined more aggressively by the compiler (no external linkage).
- **`inline` keyword** – suggests inlining to the compiler (C99) – but the compiler may ignore it.

---

## 14. Security Considerations

- **Avoid returning pointers to stack** – dangling pointers can lead to use‑after‑free vulnerabilities.
- **Check array bounds** – when passing arrays to functions, always pass the size.
- **Use `const`** – prevents accidental modification of read‑only data.
- **Validate inputs** – check for `NULL`, negative values, overflow, etc.
- **Be careful with variadic functions** – type safety is lost; ensure correct argument types.

---

## 15. Debugging Tips

- **Use `-Wall -Wextra`** – catches many function‑related warnings.
- **Use `-Wmissing-prototypes`** – warns about functions without prototypes.
- **Use `-Wunused-parameter`** – warns about unused parameters.
- **Use a debugger** – set breakpoints inside functions, step through calls.
- **Add logging** – trace function calls and parameter values.
- **Use `assert`** – to verify invariants inside functions.

---

## 16. When to Use

- **Always** – functions are the primary organisational unit in C.
- **Use for any reusable logic** – avoid code duplication.
- **Use for modular design** – separate concerns into different functions.
- **Use for libraries** – define public functions in headers.

---

## 17. When Not to Use

- **Avoid functions with too many responsibilities** – they become hard to maintain.
- **Avoid functions that modify global state excessively** – makes them hard to test.
- **Avoid over‑using recursion** – for simple loops, iteration is often better.

---

## 18. Related Concepts

- **Parameters and arguments** – input to functions.
- **Return values** – output from functions.
- **Scope and lifetime** – affect local variables inside functions.
- **Linkage** – external/internal (`static`) functions.
- **Function pointers** – pointers to functions (callbacks).
- **Variadic functions** – functions with variable arguments (`stdarg.h`).
- **`inline` functions** – hint to the compiler for inlining.
- **Recursion** – functions calling themselves.

---

## 19. Did You Know?

- In C, all functions are global (external linkage) by default; use `static` to restrict visibility.
- Functions cannot be nested in C, but you can declare a function inside another function (which is unusual and doesn't create a nested function).
- C has no function overloading – two functions cannot have the same name (unlike C++).
- The `main` function is not special to the compiler – it's just a convention that the linker looks for.
- `void` functions can still use `return;` to exit early (with no value).
- `inline` is a suggestion, not a command – the compiler may ignore it.
- Variadic functions (`printf`, `scanf`) lose type safety – a common source of bugs.

---

## 20. Summary

- **Functions are reusable blocks of code** that perform specific tasks.
- **They have a return type, name, parameters, and a body**.
- **Prototypes** declare functions before use; **definitions** provide the implementation.
- **Passing by value** copies arguments; **passing by pointer** allows modification.
- **Functions can be `static`** (internal linkage) or **external** (visible across files).
- **Returning pointers** requires careful attention to lifetime – never return a pointer to a local variable.
- **Variadic functions** (`...`) handle variable numbers of arguments (with `stdarg.h`).
- **Function pointers** enable callbacks and dynamic behaviour.
- **Best practices**: keep functions small, use descriptive names, validate inputs, use `const` for read‑only parameters.
- **Common mistakes**: missing prototypes, returning pointers to locals, mismatched signatures, forgetting to check `malloc`.
- **Understanding functions** is essential for writing modular, reusable, and maintainable C code.