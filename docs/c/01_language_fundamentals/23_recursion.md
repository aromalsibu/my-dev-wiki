# Recursion

## 1. Overview

### Definition
Recursion is a programming technique where a function calls itself to solve a problem by breaking it down into smaller, similar subproblems. A recursive function consists of two essential parts: a **base case** that terminates the recursion, and a **recursive case** that reduces the problem and calls itself. This approach is particularly powerful for problems with a naturally recursive structure, such as tree traversal, divide‑and‑conquer algorithms, and mathematical sequences.

### Purpose
Recursion provides an elegant and expressive way to solve complex problems by:
- **Simplifying code** – many algorithms are more naturally expressed recursively than iteratively.
- **Handling nested structures** – trees, graphs, and hierarchical data are inherently recursive.
- **Enabling divide‑and‑conquer** – break problems into smaller subproblems.
- **Providing a different perspective** – recursion can lead to clearer, more maintainable solutions for certain classes of problems.

### Where It Fits
Recursion is a fundamental technique in C programming, building directly on functions. It appears in:
- **Mathematical computations** – factorial, Fibonacci, exponentiation.
- **Data structure operations** – traversing trees and linked lists.
- **Sorting algorithms** – quicksort, mergesort.
- **Search algorithms** – binary search, depth‑first search.
- **String processing** – backtracking algorithms, parsing.
- **Divide‑and‑conquer algorithms** – many efficient algorithms use recursion.

---

## 2. Why It Exists

### The Problem Without Recursion
Without recursion, many natural problems would require complex iterative solutions with explicit stack management. For example, traversing a tree structure without recursion requires manually maintaining a stack of nodes, which is error‑prone and less readable. Problems like computing factorials, generating permutations, or parsing nested expressions become significantly more difficult without recursion.

### The Solution: Self‑Referential Functions
Recursion allows a function to leverage its own definition to solve smaller instances of the same problem. This mirrors the mathematical concept of **induction** – if you can solve the base case, and you can reduce every other case to a smaller instance of the same problem, you can solve the entire problem.

### Why It Was Introduced
Recursion has been a feature of programming languages since the early days of LISP and ALGOL. C supports recursion natively through its function call mechanism – each call creates a new stack frame, allowing functions to call themselves safely. This makes C suitable for implementing a wide range of algorithms that are naturally recursive.

---

## 3. Syntax / Basic Usage

### Simple Recursive Function – Factorial
```c
#include <stdio.h>

// Recursive factorial: n! = n * (n-1)!
long long factorial(int n) {
    // Base case: factorial of 0 or 1 is 1.
    if (n <= 1) {
        return 1;
    }
    // Recursive case: n * factorial(n-1).
    return n * factorial(n - 1);
}

int main(void) {
    printf("5! = %lld\n", factorial(5));   // 120
    printf("10! = %lld\n", factorial(10)); // 3628800
    return 0;
}
```

### Recursive Fibonacci Sequence
```c
#include <stdio.h>

// Recursive Fibonacci: fib(n) = fib(n-1) + fib(n-2)
long long fibonacci(int n) {
    // Base cases: fib(0) = 0, fib(1) = 1
    if (n <= 1) {
        return n;
    }
    // Recursive case.
    return fibonacci(n - 1) + fibonacci(n - 2);
}

int main(void) {
    printf("fib(10) = %lld\n", fibonacci(10)); // 55
    printf("fib(20) = %lld\n", fibonacci(20)); // 6765
    return 0;
}
```

### Recursive Binary Search
```c
#include <stdio.h>

// Recursive binary search in a sorted array.
int binary_search(int arr[], int low, int high, int target) {
    // Base case: target not found.
    if (low > high) {
        return -1;
    }

    int mid = low + (high - low) / 2;   // Avoid overflow.

    if (arr[mid] == target) {
        return mid;                     // Found!
    } else if (arr[mid] > target) {
        return binary_search(arr, low, mid - 1, target);   // Search left half.
    } else {
        return binary_search(arr, mid + 1, high, target);  // Search right half.
    }
}

int main(void) {
    int arr[8] = {1, 3, 5, 7, 9, 11, 13, 15};
    int index = binary_search(arr, 0, 7, 7);
    printf("7 found at index: %d\n", index); // 3

    index = binary_search(arr, 0, 7, 10);
    printf("10 found at index: %d\n", index); // -1 (not found)
    return 0;
}
```

### Recursive Tree Traversal (Binary Tree)
```c
#include <stdio.h>
#include <stdlib.h>

// Node structure for a binary tree.
struct Node {
    int data;
    struct Node *left;
    struct Node *right;
};

// Recursive function to print inorder traversal: left, root, right.
void inorder_traversal(struct Node *root) {
    if (root == NULL) {
        return;   // Base case: empty tree.
    }
    inorder_traversal(root->left);   // Recursive call on left subtree.
    printf("%d ", root->data);       // Visit the node.
    inorder_traversal(root->right);  // Recursive call on right subtree.
}

// Recursive function to print preorder traversal: root, left, right.
void preorder_traversal(struct Node *root) {
    if (root == NULL) {
        return;
    }
    printf("%d ", root->data);
    preorder_traversal(root->left);
    preorder_traversal(root->right);
}

// Recursive function to print postorder traversal: left, right, root.
void postorder_traversal(struct Node *root) {
    if (root == NULL) {
        return;
    }
    postorder_traversal(root->left);
    postorder_traversal(root->right);
    printf("%d ", root->data);
}

// Helper to create a new node.
struct Node *create_node(int data) {
    struct Node *node = (struct Node *)malloc(sizeof(struct Node));
    node->data = data;
    node->left = NULL;
    node->right = NULL;
    return node;
}

int main(void) {
    // Build a simple tree:
    //       1
    //      / \
    //     2   3
    //    / \
    //   4   5
    struct Node *root = create_node(1);
    root->left = create_node(2);
    root->right = create_node(3);
    root->left->left = create_node(4);
    root->left->right = create_node(5);

    printf("Inorder: ");
    inorder_traversal(root);   // 4 2 5 1 3
    printf("\n");

    printf("Preorder: ");
    preorder_traversal(root);  // 1 2 4 5 3
    printf("\n");

    printf("Postorder: ");
    postorder_traversal(root); // 4 5 2 3 1
    printf("\n");

    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>

// Recursive function to compute factorial.
// It calls itself with a smaller argument until it reaches the base case.
long long factorial(int n) {
    // Base case: the simplest instance of the problem.
    // For factorial, 0! = 1 and 1! = 1.
    if (n <= 1) {
        return 1;
    }
    // Recursive case: reduce the problem to a smaller instance.
    // n! = n * (n-1)! – the function calls itself.
    return n * factorial(n - 1);
}

// Recursive binary search: works on a sorted array.
int binary_search(int arr[], int low, int high, int target) {
    // Base case: the search range is empty – target not found.
    if (low > high) {
        return -1;
    }

    // Calculate the middle index.
    int mid = low + (high - low) / 2;

    // If the target is at the middle, we're done.
    if (arr[mid] == target) {
        return mid;
    }
    // If the target is smaller, search the left half (recursive case).
    else if (arr[mid] > target) {
        return binary_search(arr, low, mid - 1, target);
    }
    // If the target is larger, search the right half (recursive case).
    else {
        return binary_search(arr, mid + 1, high, target);
    }
}

// Recursive function to compute the nth Fibonacci number.
// This is an example of multiple recursive calls.
long long fibonacci(int n) {
    // Base cases: fib(0) = 0, fib(1) = 1.
    if (n <= 1) {
        return n;
    }
    // Recursive case: fib(n) = fib(n-1) + fib(n-2).
    return fibonacci(n - 1) + fibonacci(n - 2);
}

int main(void) {
    // Calling the recursive function from main.
    long long fact_5 = factorial(5);       // 120
    int search_result = binary_search(arr, 0, 7, 7); // 3
    long long fib_10 = fibonacci(10);      // 55

    printf("Factorial: %lld\n", fact_5);
    printf("Binary search: %d\n", search_result);
    printf("Fibonacci: %lld\n", fib_10);
    return 0;
}
```

---

## 4. Mental Model – Recursion as Russian Dolls

Imagine a set of Russian nesting dolls (matryoshka dolls):

- Each doll contains a slightly smaller doll inside it.
- To get to the smallest doll, you open each doll one by one.

In recursion:
- The **recursive function** is like a doll.
- The **recursive call** opens the next doll (calls a smaller instance).
- The **base case** is the smallest doll – it stops the process.
- The **unwinding** is like closing the dolls back up, returning the results.

Each doll (function call) waits for the inner doll to finish before it can complete itself.

### Visual Representation
```
factorial(5)
   │
   ├── 5 * factorial(4)
   │       │
   │       ├── 4 * factorial(3)
   │       │       │
   │       │       ├── 3 * factorial(2)
   │       │       │       │
   │       │       │       ├── 2 * factorial(1)
   │       │       │       │       │
   │       │       │       │       └── 1   (base case)
   │       │       │       │
   │       │       │       └── 2 * 1 = 2
   │       │       │
   │       │       └── 3 * 2 = 6
   │       │
   │       └── 4 * 6 = 24
   │
   └── 5 * 24 = 120   (final result)
```

---

## 5. Core Concepts

### Essential Components of Recursion

| Component | Description | Example |
|-----------|-------------|---------|
| **Base case** | The simplest instance that can be solved directly without recursion. | `if (n <= 1) return 1;` |
| **Recursive case** | The reduction step that calls the function with a smaller/simpler input. | `return n * factorial(n-1);` |
| **Recursive call** | The function calling itself. | `factorial(n - 1)` |
| **Unwinding** | The process of returning values back up the call stack. | Returning `2 * 1` to `3 * 2`, etc. |
| **Call stack** | The stack that holds activation records (frames) for each call. | Each recursive call adds a frame. |

### Types of Recursion

| Type | Description | Example |
|------|-------------|---------|
| **Direct recursion** | A function calls itself directly. | `factorial()` calls `factorial()`. |
| **Indirect recursion** | Function A calls Function B, which calls Function A. | `even()` → `odd()` → `even()` |
| **Tail recursion** | The recursive call is the last operation in the function. | `factorial_tail(n, acc)` |
| **Multiple recursion** | A function makes multiple recursive calls. | `fibonacci(n-1) + fibonacci(n-2)` |
| **Mutual recursion** | Two or more functions call each other. | `is_even()` and `is_odd()` calling each other. |

### Depth of Recursion
- The number of recursive calls that must be active before reaching the base case.
- Limited by the **call stack size** – recursion that is too deep can cause a **stack overflow**.

### Mathematical Induction
Recursion is the programming equivalent of mathematical induction:
- **Base case** – prove the statement for the smallest value.
- **Inductive step** – prove that if the statement holds for n-1, it holds for n.

---

## 6. How It Works – The Call Stack

### Step‑by‑Step Execution (Factorial)
1. `factorial(5)` is called from `main`.
2. A stack frame for `factorial(5)` is created on the call stack.
3. `factorial(5)` calls `factorial(4)`.
4. A new frame for `factorial(4)` is pushed on top.
5. `factorial(4)` calls `factorial(3)`, and so on, until `factorial(1)`.
6. `factorial(1)` reaches the base case (`n <= 1`) and returns `1`.
7. The frame for `factorial(1)` is popped.
8. `factorial(2)` receives `1` from `factorial(1)`, computes `2 * 1 = 2`, and returns.
9. `factorial(3)` receives `2`, computes `3 * 2 = 6`, and returns.
10. `factorial(4)` receives `6`, computes `4 * 6 = 24`, and returns.
11. `factorial(5)` receives `24`, computes `5 * 24 = 120`, and returns.
12. Control returns to `main` with the result.

### Call Stack Diagram
```
factorial(5) call:
    ┌─────────────────────┐
    │ main()              │
    ├─────────────────────┤
    │ factorial(5) frame  │ ← waiting for factorial(4) result
    ├─────────────────────┤
    │ factorial(4) frame  │ ← waiting for factorial(3) result
    ├─────────────────────┤
    │ factorial(3) frame  │ ← waiting for factorial(2) result
    ├─────────────────────┤
    │ factorial(2) frame  │ ← waiting for factorial(1) result
    ├─────────────────────┤
    │ factorial(1) frame  │ ← base case, returns 1
    └─────────────────────┘
```

### Stack Overflow
If the recursion is too deep (e.g., `factorial(1000000)`), the call stack will grow beyond its limit, causing a stack overflow and the program to crash.

---

## 7. Internal Architecture – Activation Records

### What's in a Stack Frame?
For each function call (including recursive calls), the compiler creates an **activation record** (stack frame) containing:
- **Return address** – where to go back to after the function returns.
- **Saved base pointer** – the previous frame's base address.
- **Parameters** – the function's arguments.
- **Local variables** – variables declared inside the function.
- **Temporary storage** – for intermediate results.

### Performance Impact
- Each recursive call adds a new frame – **overhead** in both time and memory.
- Deep recursion can be slower and more memory‑intensive than iteration.
- **Tail recursion** can be optimised to avoid growing the stack (if the compiler supports it).

---

## 8. Lifecycle / Workflow of a Recursive Call

1. **Call** – the function is invoked.
2. **Frame creation** – a new stack frame is pushed.
3. **Parameter assignment** – arguments are copied to parameters.
4. **Check base case** – if the base case is true, return the result.
5. **Recursive case** – the function calls itself with a smaller input.
6. **Wait** – the calling function pauses, waiting for the recursive call to return.
7. **Return** – when the recursive call returns, the function continues.
8. **Compute result** – combine the returned value with other calculations.
9. **Return** – the function returns its result to its caller.
10. **Frame pop** – the stack frame is popped.

---

## 9. Practical Examples

### Example 1: Direct Recursion – Power Function
```c
#include <stdio.h>

// Recursive power function: x^y.
double power(double base, int exponent) {
    // Base case: any number to the power of 0 is 1.
    if (exponent == 0) {
        return 1.0;
    }
    // Negative exponent: x^(-n) = 1 / x^n
    if (exponent < 0) {
        return 1.0 / power(base, -exponent);
    }
    // Recursive case: x^n = x * x^(n-1)
    return base * power(base, exponent - 1);
}

int main(void) {
    printf("2^3 = %.2f\n", power(2.0, 3));     // 8.00
    printf("2^-3 = %.2f\n", power(2.0, -3));   // 0.125
    printf("5^0 = %.2f\n", power(5.0, 0));     // 1.00
    return 0;
}
```
**Code Breakdown:**
- `power(2, 3)` expands: `2 * power(2, 2)` → `2 * (2 * power(2, 1))` → `2 * (2 * (2 * power(2, 0)))` → `2 * 2 * 2 * 1 = 8`.
- Handles negative exponents by taking the reciprocal.

### Example 2: Indirect Recursion (Mutual Recursion)
```c
#include <stdio.h>
#include <stdbool.h>

// Forward declarations needed for mutual recursion.
bool is_even(int n);
bool is_odd(int n);

// is_even calls is_odd for n-1.
bool is_even(int n) {
    if (n == 0) {
        return true;
    }
    return is_odd(n - 1);
}

// is_odd calls is_even for n-1.
bool is_odd(int n) {
    if (n == 0) {
        return false;
    }
    return is_even(n - 1);
}

int main(void) {
    printf("4 is even: %s\n", is_even(4) ? "true" : "false");   // true
    printf("5 is odd: %s\n", is_odd(5) ? "true" : "false");     // true
    printf("7 is even: %s\n", is_even(7) ? "true" : "false");   // false
    return 0;
}
```
**Code Breakdown:**
- `is_even` and `is_odd` call each other repeatedly until they reach the base case.
- For `is_even(4)`: `is_even(4)` → `is_odd(3)` → `is_even(2)` → `is_odd(1)` → `is_even(0)` → `true`.
- This is an example of **mutual recursion**.

### Example 3: Tail Recursion – Factorial (Optimised)
```c
#include <stdio.h>

// Tail-recursive factorial: the recursive call is the last operation.
long long factorial_tail(int n, long long accumulator) {
    // Base case: return the accumulator.
    if (n <= 1) {
        return accumulator;
    }
    // Recursive call with updated accumulator – this is the last operation.
    return factorial_tail(n - 1, n * accumulator);
}

// Wrapper function for convenient calling.
long long factorial(int n) {
    return factorial_tail(n, 1);
}

int main(void) {
    printf("5! = %lld\n", factorial(5));   // 120
    printf("10! = %lld\n", factorial(10)); // 3628800
    return 0;
}
```
**Code Breakdown:**
- The recursive call is the **last operation** – no additional computation after it returns.
- The compiler can optimise tail recursion (if supported) by reusing the current stack frame, avoiding stack growth.
- `factorial_tail(5, 1)` → `factorial_tail(4, 5)` → `factorial_tail(3, 20)` → `factorial_tail(2, 60)` → `factorial_tail(1, 120)` → returns `120`.

### Example 4: Recursive Reverse of a String
```c
#include <stdio.h>
#include <string.h>

// Recursive function to reverse a string in place.
void reverse_string(char *str, int left, int right) {
    // Base case: left >= right – we're done.
    if (left >= right) {
        return;
    }
    // Swap characters at left and right.
    char temp = str[left];
    str[left] = str[right];
    str[right] = temp;

    // Recursive call with smaller range.
    reverse_string(str, left + 1, right - 1);
}

int main(void) {
    char str[] = "Hello, World!";
    printf("Original: %s\n", str);
    reverse_string(str, 0, strlen(str) - 1);
    printf("Reversed: %s\n", str);   // !dlroW ,olleH
    return 0;
}
```
**Code Breakdown:**
- Swaps the first and last characters, then recurses on the inner substring.
- Base case: when the pointers cross.

### Example 5: Recursive Permutation Generation
```c
#include <stdio.h>
#include <string.h>

// Swap two characters.
void swap(char *a, char *b) {
    char temp = *a;
    *a = *b;
    *b = temp;
}

// Recursive function to generate all permutations of a string.
void permute(char *str, int start, int end) {
    // Base case: we've reached the end – print the permutation.
    if (start == end) {
        printf("%s\n", str);
        return;
    }
    // Recursive case: try each character at the current position.
    for (int i = start; i <= end; i++) {
        swap(&str[start], &str[i]);          // swap
        permute(str, start + 1, end);        // recurse
        swap(&str[start], &str[i]);          // backtrack
    }
}

int main(void) {
    char str[] = "ABC";
    printf("Permutations of ABC:\n");
    permute(str, 0, 2);
    // Output: ABC, ACB, BAC, BCA, CAB, CBA
    return 0;
}
```
**Code Breakdown:**
- For each position, we try every remaining character.
- `swap` is used to generate each permutation.
- **Backtracking** – after recursing, we swap back to restore the original order.
- This is a classic example of recursion with backtracking.

### Example 6: Recursive Solution for the Tower of Hanoi
```c
#include <stdio.h>

// Recursive function to solve the Tower of Hanoi.
void tower_of_hanoi(int n, char from, char to, char aux) {
    // Base case: only one disk to move.
    if (n == 1) {
        printf("Move disk 1 from %c to %c\n", from, to);
        return;
    }
    // Move n-1 disks from 'from' to 'aux' using 'to' as auxiliary.
    tower_of_hanoi(n - 1, from, aux, to);
    // Move the largest disk from 'from' to 'to'.
    printf("Move disk %d from %c to %c\n", n, from, to);
    // Move n-1 disks from 'aux' to 'to' using 'from' as auxiliary.
    tower_of_hanoi(n - 1, aux, to, from);
}

int main(void) {
    int n = 3;   // Number of disks.
    printf("Tower of Hanoi with %d disks:\n", n);
    tower_of_hanoi(n, 'A', 'C', 'B');
    return 0;
}
```
**Code Breakdown:**
- A classic recursive problem – move disks from peg A to peg C, using B as auxiliary.
- The recursive strategy: move the top `n-1` disks to the auxiliary peg, move the largest disk, then move the `n-1` disks to the destination.
- The number of moves is `2^n - 1`.

### Example 7: Recursive Depth‑First Search in a Binary Tree
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

struct Node {
    int data;
    struct Node *left;
    struct Node *right;
};

// Recursive DFS to search for a value in a binary tree.
bool search_tree(struct Node *root, int target) {
    // Base case: empty tree – target not found.
    if (root == NULL) {
        return false;
    }
    // If found at the current node.
    if (root->data == target) {
        return true;
    }
    // Recursive search in left and right subtrees.
    return search_tree(root->left, target) || search_tree(root->right, target);
}

// Create a new node.
struct Node *create_node(int data) {
    struct Node *node = (struct Node *)malloc(sizeof(struct Node));
    node->data = data;
    node->left = NULL;
    node->right = NULL;
    return node;
}

int main(void) {
    // Build a simple tree.
    struct Node *root = create_node(10);
    root->left = create_node(5);
    root->right = create_node(15);
    root->left->left = create_node(3);
    root->left->right = create_node(7);

    printf("Search for 7: %s\n", search_tree(root, 7) ? "Found" : "Not found");
    printf("Search for 20: %s\n", search_tree(root, 20) ? "Found" : "Not found");
    return 0;
}
```
**Code Breakdown:**
- DFS checks the current node, then recursively checks the left and right subtrees.
- `||` short‑circuits: if the left subtree returns `true`, the right subtree isn't searched.

### Example 8: Recursive GCD (Euclidean Algorithm)
```c
#include <stdio.h>

// Recursive function to compute GCD using Euclid's algorithm.
int gcd(int a, int b) {
    // Base case: if b == 0, gcd is a.
    if (b == 0) {
        return a;
    }
    // Recursive case: gcd(b, a % b).
    return gcd(b, a % b);
}

int main(void) {
    printf("GCD of 48 and 18: %d\n", gcd(48, 18)); // 6
    printf("GCD of 100 and 25: %d\n", gcd(100, 25)); // 25
    printf("GCD of 17 and 13: %d\n", gcd(17, 13)); // 1
    return 0;
}
```
**Code Breakdown:**
- Euclid's algorithm: `gcd(a, b) = gcd(b, a % b)`.
- The recursion reduces the problem size quickly; the base case is when `b == 0`.

---

## 10. Common Use Cases

| Use Case | Example |
|----------|---------|
| **Mathematical sequences** | Factorial, Fibonacci, GCD |
| **Tree traversals** | Inorder, preorder, postorder |
| **Graph algorithms** | DFS, BFS (DFS is naturally recursive) |
| **Divide‑and‑conquer** | Mergesort, Quicksort, Binary search |
| **Backtracking** | N‑Queens, Permutations, Sudoku solver |
| **String processing** | Reverse string, Palindrome check |
| **Dynamic programming** | Memoized recursion (top‑down DP) |
| **Parsing** | Recursive descent parsers |
| **Tower of Hanoi** | Classic recursive problem |

---

## 11. Best Practices

### General
- **Always define a base case** – ensure the recursion terminates.
- **Ensure progress toward the base case** – each recursive call must reduce the problem.
- **Prefer iteration for simple loops** – recursion adds overhead.
- **Use recursion for naturally recursive structures** – trees, graphs, nested lists.
- **Consider tail recursion** – if the compiler supports optimisation, it can prevent stack overflow.

### Base Case Design
- **Make base cases simple** – solve the smallest instance directly.
- **Handle edge cases** – e.g., `NULL` pointers, empty arrays.
- **Ensure the base case is reachable** – otherwise infinite recursion.

### Performance
- **Avoid exponential recursion** – like naive Fibonacci, which has O(2^n) complexity.
- **Use memoisation** – cache results of subproblems (top‑down DP).
- **Consider iterative alternatives** – for performance‑critical code.

### Safety
- **Be aware of stack depth** – deep recursion can cause stack overflow.
- **Use `static` or `extern`** for helper functions.
- **Avoid recursion in real‑time systems** – stack size is limited.

---

## 12. Common Mistakes

### Mistake 1: Missing Base Case
```c
// ❌ Wrong – no base case, infinite recursion.
int bad_factorial(int n) {
    return n * bad_factorial(n - 1);   // No base case!
}
// ✅ Correct – add a base case.
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

### Mistake 2: Base Case Not Reached
```c
// ❌ Wrong – base case never reached (n decrements by 2, so n may skip 1).
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 2);   // If n is odd, it never reaches 0.
}
// ✅ Correct – handle both odd and even.
int sum(int n) {
    if (n <= 0) return 0;
    return n + sum(n - 2);
}
```

### Mistake 3: Excessive Recursion (Stack Overflow)
```c
// ❌ Wrong – may cause stack overflow for large n.
long long fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);   // Exponential calls.
}
// ✅ Better – use iteration or memoisation.
long long fib_iter(int n) {
    if (n <= 1) return n;
    long long a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        long long c = a + b;
        a = b;
        b = c;
    }
    return b;
}
```

### Mistake 4: Forgetting to Restore State (Backtracking)
```c
// ❌ Wrong – doesn't restore the original state after recursion.
void permute(char *str, int start, int end) {
    if (start == end) { printf("%s\n", str); return; }
    for (int i = start; i <= end; i++) {
        swap(&str[start], &str[i]);
        permute(str, start + 1, end);
        // Missing swap back – state is corrupted!
    }
}
// ✅ Correct – swap back to restore state.
void permute(char *str, int start, int end) {
    if (start == end) { printf("%s\n", str); return; }
    for (int i = start; i <= end; i++) {
        swap(&str[start], &str[i]);
        permute(str, start + 1, end);
        swap(&str[start], &str[i]);   // Backtrack.
    }
}
```

### Mistake 5: Modifying Global State Without Care
```c
int global_counter = 0;   // ❌ Danger – shared state.

void recursive_func(int n) {
    global_counter++;   // Modifies global state.
    if (n <= 1) return;
    recursive_func(n - 1);
}
// If called from multiple places, the counter may be incorrect.
// ✅ Better – pass the counter as a parameter.
void recursive_func(int n, int *counter) {
    (*counter)++;
    if (n <= 1) return;
    recursive_func(n - 1, counter);
}
```

---

## 13. Performance Considerations

- **Call overhead** – each recursive call has overhead (stack frame allocation).
- **Memory usage** – recursion uses O(depth) stack space; iteration uses O(1).
- **Exponential time** – naive Fibonacci has O(2^n) time; use memoisation or iteration.
- **Tail recursion** – can be optimised to O(1) stack space (if the compiler supports it).
- **Cache locality** – recursive functions may have poor cache performance (deep call stacks).

### Optimisation Techniques
- **Memoisation** – cache results of subproblems (top‑down DP).
- **Tail recursion** – write functions so the recursive call is the last operation.
- **Iterative conversion** – many recursive functions can be converted to loops.

---

## 14. Security Considerations

- **Stack overflow** – deep recursion can crash the program; ensure recursion depth is bounded.
- **Denial of service** – malicious input could cause deep recursion; validate input size.
- **Global state** – recursion with shared mutable state can lead to race conditions in multi‑threaded code.

---

## 15. Debugging Tips

- **Use a debugger** – step through recursive calls to see the unwinding.
- **Add print statements** – trace calls and returns:
  ```c
  int factorial(int n) {
      printf("Entering factorial(%d)\n", n);
      if (n <= 1) { printf("Base case: %d\n", n); return 1; }
      int result = n * factorial(n - 1);
      printf("Returning %d from factorial(%d)\n", result, n);
      return result;
  }
  ```
- **Use `-Wall -Wextra`** – catches some recursion‑related warnings.
- **Compile with `-O0`** – to preserve the call stack for debugging.
- **Check stack size** – use `ulimit -s` on Unix to view the stack limit.

---

## 16. When to Use

- **Naturally recursive problems** – trees, graphs, divide‑and‑conquer.
- **Code clarity** – when recursion makes the code significantly clearer.
- **Mathematical functions** – factorial, GCD, Fibonacci (with memoisation).
- **Backtracking** – puzzles, permutations, combinations.
- **Exploring all possibilities** – where recursion is the most natural approach.

---

## 17. When Not to Use

- **Performance‑critical code** – iteration is often faster.
- **Large input sizes** – recursion depth may exceed the stack limit.
- **Simple loops** – don't use recursion for trivial iterative tasks.
- **Real‑time systems** – unpredictable stack usage can be problematic.

---

## 18. Related Concepts

- **Functions** – recursion is a special case of function calls.
- **Call stack** – the mechanism that supports recursion.
- **Stack overflow** – a risk with deep recursion.
- **Tail recursion** – an optimisable form of recursion.
- **Memoisation** – caching recursive results (DP).
- **Backtracking** – recursion with state restoration.
- **Divide‑and‑conquer** – a common algorithmic pattern using recursion.
- **Tree traversal** – naturally recursive operations.

---

## 19. Did You Know?

- C compilers generally do not guarantee tail‑call optimisation; it's implementation‑specific.
- The maximum recursion depth is limited by the stack size (typically 1‑8 MB on modern systems).
- Some functional languages (e.g., Scheme) require tail‑call optimisation as part of the language standard.
- Recursive functions can be defined to be `static` to limit visibility to the current translation unit.
- The Tower of Hanoi problem requires `2^n - 1` moves – a classic exercise in recursion.
- Recursion can be used to implement parsers for context‑free grammars.

---

## 20. Summary

- **Recursion is a technique where a function calls itself** to solve smaller instances of the same problem.
- **Every recursive function must have** a **base case** (termination condition) and a **recursive case** (reduction step).
- **Types of recursion**: direct, indirect, tail, multiple, and mutual recursion.
- **Recursion uses the call stack** – each call creates a stack frame; deep recursion can cause stack overflow.
- **Common uses**: tree/graph traversal, divide‑and‑conquer algorithms, backtracking, mathematical sequences.
- **Best practices**: always define a base case, ensure progress toward the base case, prefer iteration for simple tasks, and consider tail recursion.
- **Common mistakes**: missing base case, base case not reachable, excessive recursion depth, forgetting to restore state in backtracking.
- **Understanding recursion** is essential for solving problems with hierarchical or nested structures and is a fundamental tool in the programmer's toolkit.