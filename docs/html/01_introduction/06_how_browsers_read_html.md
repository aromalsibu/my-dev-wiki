
# Chapter: How Browsers Read HTML

---

## 1. Overview

### Definition

"How browsers read HTML" refers to the **parsing and rendering pipeline** that a browser executes to transform raw HTML bytes (received from a web server) into a visual, interactive web page. This pipeline is a complex, multi‑stage process that involves lexical analysis, tree construction, style computation, layout, painting, and compositing.

The browser doesn't simply "display" HTML; it **parses** it into a structured, in‑memory model called the **DOM** (Document Object Model), applies CSS to determine visual styles, computes the position of every element, draws pixels on the screen, and handles user interactions and dynamic updates.

### Purpose

Understanding how browsers read HTML is crucial for web developers because it directly impacts:

- **Performance** – knowing what blocks rendering helps you optimise load times.
- **Debugging** – understanding the pipeline helps diagnose layout issues and repaint problems.
- **Architecture** – decisions like script placement, CSS delivery, and image loading are informed by parsing behaviour.
- **Accessibility** – the DOM tree is the foundation for assistive technologies; understanding its construction helps you build accessible content.
- **Progressive Enhancement** – knowing that the browser renders incrementally allows you to deliver content faster.

### Where It Fits

The browser's reading of HTML is the **first and most critical step** in the rendering lifecycle. It sits between network reception (HTTP) and rendering (painting). Without parsing, there is no DOM, no CSSOM, no render tree, and no page.

```
Network (HTML bytes)
    │
    ▼
┌───────────────────────────────┐
│  How Browsers Read HTML      │  ← This chapter
│  (Parsing & Rendering Pipeline)│
└───────────────────────────────┘
    │
    ▼
Visual Page (pixels on screen)
```

---

## 2. Why It Exists

### The Problem

Raw HTML is just a text file. It has no inherent visual representation. To display it, a browser must:

1. **Understand the structure** – identify elements, attributes, text, and their relationships.
2. **Resolve dependencies** – load external resources (CSS, images, scripts, fonts) referenced in the HTML.
3. **Compute styles** – apply CSS rules to each element.
4. **Determine layout** – calculate where each element appears and how much space it occupies.
5. **Paint** – turn the layout into pixels on the screen.
6. **Handle interactivity** – respond to user input (clicks, scrolls, typing) and dynamic updates.

Without a well‑defined pipeline, each browser would interpret HTML differently, resulting in inconsistent rendering. The HTML specification defines a **standardised parsing algorithm** to ensure that all browsers produce the same DOM from the same HTML.

### Previous Limitations (Before Standardised Parsing)

- **Browser wars (1990s)** – Netscape and Internet Explorer implemented parsing differently, leading to "best viewed in" badges and conditional comments.
- **Quirks mode** – older browsers used different layout engines (quirks mode) for legacy pages; the `<!DOCTYPE>` was introduced to switch between quirks and standards modes.
- **Error recovery was inconsistent** – each browser handled malformed HTML differently, leading to cross‑browser bugs.
- **No defined script blocking behaviour** – scripts could block parsing unpredictably, hurting performance.
- **No incremental rendering** – pages had to be fully downloaded before rendering, which was slow on dial‑up.

### Why Standardised Parsing Was Introduced

The HTML5 specification (and the Living Standard) defines a **detailed parsing algorithm** that is mandatory for all browsers. This ensures:

- **Consistency** – the same HTML produces the same DOM in every modern browser.
- **Error recovery** – browsers must handle malformed markup in a defined way (e.g., implicitly closing tags, inserting missing elements).
- **Performance** – the algorithm is designed to parse incrementally and efficiently.
- **Interoperability** – web developers can rely on predictable behaviour without writing browser‑specific hacks.

The parsing algorithm is the reason why HTML is so forgiving – you can omit closing tags, mis‑nest elements, and the browser will still render something (though not always what you intended). This backward‑compatibility is a key reason for the web's success.

---

## 3. Syntax / Basic Usage (Illustrating the Pipeline)

While there isn't a "syntax" for the parsing process itself, we can illustrate it with a simple HTML document and trace its journey through the browser.

### Smallest Useful Example

```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1 id="title">Welcome</h1>
    <p>This is a <strong>simple</strong> page.</p>
    <script src="app.js"></script>
</body>
</html>
```

### Code Breakdown (From a Parsing Perspective)

| Part | What the Parser Does | Why It Matters |
|------|----------------------|----------------|
| `<!DOCTYPE html>` | Determines rendering mode (standards mode). | Prevents quirks mode. |
| `<html>` | Creates the root `html` element in the DOM. | The document's root. |
| `<head>` | Creates a `head` element; parser enters "in head" state. | Holds metadata; some elements (like `<title>`) have special handling. |
| `<title>Hello</title>` | Creates a `title` element; text node "Hello" added. | The title is used for the browser tab. |
| `<link rel="stylesheet" href="style.css">` | Creates a `link` element; triggers a network request for `style.css` (non‑blocking for parsing but render‑blocking). | CSS is loaded and will affect rendering. |
| `<body>` | Creates a `body` element; parser enters "in body" state. | Visible content begins. |
| `<h1 id="title">Welcome</h1>` | Creates an `h1` element with attribute `id="title"` and text node "Welcome". | Adds a heading to the DOM. |
| `<p>This is a <strong>simple</strong> page.</p>` | Creates a `p` element, then a text node "This is a ", then a `strong` element with text "simple", then text " page." – all nested correctly. | Paragraph with inline emphasis. |
| `<script src="app.js"></script>` | Creates a `script` element; triggers a network request for `app.js`; **blocks parsing** until the script is fetched and executed (unless `async`/`defer` are used). | JavaScript execution can modify the DOM. |

### The Parsing Timeline

```
Time ─────────────────────────────────────────────────────────────────►

0ms   Browser receives HTML bytes.
      Parser starts tokenizing.
      
5ms   <html> token → creates DOM node.
      <head> token → creates DOM node.
      
10ms  <title> → creates DOM node; text "Hello" added.
      
15ms  <link> → starts fetching style.css (non‑blocking).
      
20ms  <body> → creates DOM node.
      <h1> → creates DOM node; text "Welcome" added.
      
25ms  <p> → creates DOM node; text "This is a " added.
      <strong> → creates DOM node; text "simple" added.
      </strong> → closes strong.
      text " page." added.
      </p> → closes p.
      
30ms  <script> → starts fetching app.js.
      Parser **stops** and waits for script to download and execute.
      (If script is large, this delays parsing.)
      
100ms Script downloaded and executed.
      Parser resumes and finishes (closing tags).
      
120ms DOM is complete. CSS (if loaded) is applied.
      Render tree built, layout computed, pixels painted.
```

This timeline illustrates why script placement is critical for performance.

---

## 4. Mental Model

### The "Assembly Line" Analogy

Imagine a car assembly line:

1. **Raw materials** arrive (HTML bytes).
2. **Parts are stamped** (tokenization – characters become tokens).
3. **Chassis is built** (tree construction – tokens become a DOM tree).
4. **Engine and body are added** (CSSOM and render tree).
5. **Paint is applied** (painting).
6. **Quality checks** (layout, reflow).
7. **Car drives off** (interactive page).

The assembly line has **stages** that happen in order, but sometimes a stage can start before the previous one finishes (e.g., incremental rendering). Also, if a part is missing (a script), the line may pause.

### The "Recipe" Analogy

HTML is a recipe:
- **Ingredients** are the tags and text.
- **Instructions** are the parsing rules.
- **Chef** is the browser.
- The **kitchen** is the rendering engine.

The browser follows the recipe step‑by‑step:
1. Read the recipe (HTML bytes).
2. Chop ingredients (tokenization).
3. Mix ingredients (tree construction).
4. Add seasoning (CSS styles).
5. Arrange on a plate (layout).
6. Present (paint).

If the recipe says "add butter" but you're out, the browser might substitute (error recovery).

### The "Tree Growth" Analogy

Think of the DOM as a growing tree:
- The **root** is the `<html>` element.
- **Branches** are child elements (nested tags).
- **Leaves** are text nodes and attributes.
- The tree grows **incrementally** as the parser reads tokens.
- Sometimes a branch is delayed (scripts) or pruned (hidden elements).

Understanding this tree metaphor helps you visualise how the browser builds the document structure.

---

## 5. Core Concepts

### Concept 1: Tokenization

The first stage of parsing. The browser reads the HTML character stream and breaks it into a sequence of **tokens** (start tags, end tags, text, comments, doctype, etc.). Tokens are produced using a **state machine** defined in the HTML specification.

### Concept 2: Tree Construction

Tokens are consumed to build the **DOM tree**. The parser maintains a stack of open elements. When a start tag is encountered, a new element node is created and appended to the current parent (the top of the stack). When an end tag is encountered, the stack is popped, closing the element.

### Concept 3: The DOM (Document Object Model)

The DOM is an **in‑memory tree representation** of the HTML document. It is the interface through which JavaScript can read and modify the structure, style, and content of a page. The DOM is **not** the same as the HTML source; it is a live, mutable model that reflects the current state.

### Concept 4: Error Recovery (or "The Forgiving Parser")

The HTML parser is designed to recover from markup errors. For example, if a closing tag is missing, the parser will implicitly close the element when appropriate. This behaviour is defined in the specification and is consistent across browsers.

### Concept 5: Blocking vs Non‑Blocking Resources

- **Stylesheets** (CSS) block rendering but **do not** block parsing (the parser continues building the DOM).
- **Scripts** without `async`/`defer` **block parsing** – the parser stops until the script is fetched and executed.
- **Images and other media** do not block parsing or rendering (they load asynchronously).

### Concept 6: The Critical Rendering Path

The sequence of steps required to render the **first pixel** to the screen:
1. **DOM** (from HTML)
2. **CSSOM** (from CSS)
3. **Render Tree** (DOM + CSSOM)
4. **Layout** (box sizes and positions)
5. **Paint** (drawing pixels)

Optimising this path is key to performance.

---

## 6. How It Works (Step‑by‑Step)

### Detailed Parsing Pipeline

```
Step 1:  Network Reception
         └── Browser receives HTML bytes over HTTP (or from cache).

Step 2:  Encoding
         └── Bytes are decoded to characters based on the specified encoding (default UTF‑8).

Step 3:  Tokenization (Lexical Analysis)
         └── The character stream is scanned, and tokens (start tags, end tags, text, etc.) are produced.
         └── This is driven by a state machine that tracks the parser's current context (e.g., "in head", "in body").

Step 4:  Tree Construction (DOM Building)
         └── Tokens are consumed to create DOM nodes.
         └── A stack of open elements is maintained to handle nesting.
         └── Text nodes are created for text content.
         └── Attributes are added to element nodes.

Step 5:  CSSOM Construction (Concurrent)
         └── While the HTML is being parsed, the browser also downloads and parses CSS (from `<link>` and `<style>`).
         └── CSS rules are parsed into a CSSOM tree (mirroring the DOM but with style rules).

Step 6:  Render Tree Construction
         └── The DOM and CSSOM are combined to produce a **render tree**.
         └── Only visible elements are included (e.g., `display: none` elements are omitted).
         └── Each render tree node contains its computed styles and geometric information.

Step 7:  Layout (Reflow)
         └── The render tree is traversed to calculate the exact position and size of each element.
         └── This involves the CSS box model (margin, border, padding, content).

Step 8:  Paint
         └── The render tree is traversed again to draw pixels to the screen.
         └── Painting can be broken into layers (compositing) for performance.

Step 9:  Compositing (Optional)
         └── Different layers are combined (e.g., transformed elements, video, canvas) to produce the final image.
```

### Interactive Diagram (ASCII)

```
   HTML Bytes
        │
        ▼
   ┌─────────────┐
   │ Tokenization │  (characters → tokens)
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │ Tree Const. │  (tokens → DOM tree)
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐        ┌─────────────┐
   │    DOM      │        │    CSSOM    │ (from CSS)
   └──────┬──────┘        └──────┬──────┘
          │                      │
          └──────────┬───────────┘
                     ▼
           ┌─────────────────┐
           │  Render Tree    │ (DOM + CSSOM, only visible)
           └────────┬────────┘
                    ▼
           ┌─────────────────┐
           │  Layout (Reflow)│ (positions, sizes)
           └────────┬────────┘
                    ▼
           ┌─────────────────┐
           │     Paint       │ (pixels to screen)
           └─────────────────┘
```

### Script Execution Impact

If a `<script>` tag is encountered **without `async` or `defer`**:

- The parser **stops** building the DOM.
- It fetches the script (if external) and executes it immediately.
- Only after execution does the parser resume.

This is why you often see scripts placed at the bottom of the `<body>` – to avoid blocking rendering.

If a script uses `document.write()`, it can inject HTML into the parser stream, but this is discouraged in modern HTML5 (and `document.write` is often blocked in many contexts).

### The Role of the `defer` and `async` Attributes

| Attribute | Behaviour | When to Use |
|-----------|-----------|-------------|
| None (default) | Parser blocks; script fetched and executed immediately. | For essential scripts that must run before rendering (rare). |
| `async` | Script fetched asynchronously; execution occurs as soon as it's ready, potentially before DOM is complete. | For independent scripts (analytics, ads). |
| `defer` | Script fetched asynchronously; execution is deferred until after DOM parsing is complete. | For scripts that depend on the full DOM (most applications). |

---

## 7. Internal Architecture / Under the Hood

### The HTML Parser (The State Machine)

The HTML5 parsing algorithm is defined as a **state machine** with over 80 states. It is implemented in C++ in most browsers (e.g., Blink's `HTMLParser`, Gecko's `nsHTMLTokenizer`). The parser maintains:

- **Input stream** – the character stream being consumed.
- **Tokeniser state** – determines what token is being built.
- **Open element stack** – tracks currently open elements to handle nesting and error recovery.
- **Insertion mode** – determines where new nodes should be appended (e.g., "before head", "in body").

### The DOM Tree Representation

Each DOM node is an object with properties:
- `nodeType` (element, text, comment, etc.)
- `nodeName` (tag name, `#text`, etc.)
- `attributes` (for elements)
- `childNodes` (list of children)
- `parentNode`
- `textContent`, `innerHTML`, etc.

The DOM is **live** – changes made by JavaScript are immediately reflected in the tree.

### The CSSOM (CSS Object Model)

CSS rules are parsed into a tree structure that mirrors the DOM but contains style rules. The browser computes the **cascaded** style for each element by matching rules from the CSSOM based on specificity, cascade order, and inheritance.

### The Render Tree

The render tree is a separate tree that contains only **visible** elements. It is used for layout and painting. Elements with `display: none` are excluded. The render tree also includes pseudo‑elements (like `::before` and `::after`) as separate nodes.

### Compositor Threads

Modern browsers use **multi‑process architecture** (e.g., Chrome's Blink uses separate processes for each tab). The **main thread** handles DOM, CSSOM, layout, and painting. The **compositor thread** handles scrolling, animation, and compositing layers, improving smoothness.

---

## 8. Lifecycle / Workflow

### The Lifecycle of a Browser Parsing a Page

```
1.  Navigation Start
    │   └── User clicks a link or types a URL.
    │
    ▼
2.  Request
    │   └── Browser sends HTTP request for the HTML document.
    │
    ▼
3.  Response
    │   └── Server returns HTML bytes (maybe chunked).
    │
    ▼
4.  Bytes Received
    │   └── Browser starts receiving data.
    │
    ▼
5.  Parse HTML
    │   ├── Tokenization.
    │   ├── Tree Construction (DOM).
    │   ├── While parsing, external resources are discovered and fetched.
    │   │   ├── CSS → fetched, parsed (CSSOM).
    │   │   ├── Images → fetched (non‑blocking).
    │   │   └── Scripts → fetched; may block parsing.
    │   └── Error recovery.
    │
    ▼
6.  DOM Ready
    │   └── The DOM tree is complete (but may not be fully styled yet).
    │
    ▼
7.  Style Calculation
    │   └── Compute styles for each node using CSSOM.
    │
    ▼
8.  Layout (Reflow)
    │   └── Calculate positions and sizes.
    │
    ▼
9.  Paint
    │   └── Draw pixels.
    │
    ▼
10. Composite
    │   └── Layers combined.
    │
    ▼
11. Load Event Fires (window.load)
    │   └── All resources (images, scripts, styles) are loaded.
    │
    ▼
12. Page is Fully Interactive
```

### Events During Parsing

- **`DOMContentLoaded`** – fires when the DOM is fully parsed (but before images and other subresources have finished loading).
- **`window.load`** – fires when all resources (images, styles, scripts, etc.) have finished loading.

These events are used by developers to run code at appropriate times.

---

## 9. Practical Examples

### Example 1: Tracing Parsing with a Simple HTML Page

```html
<!DOCTYPE html>
<html>
<head>
    <title>Parsing Demo</title>
    <style>
        body { background: #f0f0f0; }
    </style>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Hello</h1>
    <p>This is a test.</p>
    <script>
        console.log('Script executed');
    </script>
    <img src="image.jpg" alt="Test">
</body>
</html>
```

**Step‑by‑Step Parsing Timeline (simplified):**

| Time | Event |
|------|-------|
| 0ms | Start parsing. `<!DOCTYPE html>` → standards mode. |
| 1ms | `<html>` → create root. |
| 2ms | `<head>` → push head context. |
| 3ms | `<title>` → create title, text "Parsing Demo". |
| 5ms | `<style>` → parse inline CSS; add to CSSOM. |
| 8ms | `<link>` → start fetching `styles.css`. |
| 10ms | `</head>` → close head; `body` starts. |
| 12ms | `<h1>` → create h1, text "Hello". |
| 14ms | `<p>` → create p, text "This is a test." |
| 16ms | `<script>` → parser **stops**; fetch and execute script. |
| 25ms | Script executes, prints to console. Parser resumes. |
| 27ms | `<img>` → create img, start fetching image (non‑blocking). |
| 30ms | `</body>`, `</html>` → parser finishes. |
| 35ms | DOM is complete; `DOMContentLoaded` fires. |
| 50ms | `styles.css` arrives; CSSOM updated; render tree built. Layout, paint. |
| 60ms | Image loads; page repaints. |
| 70ms | All resources loaded; `window.load` fires. |

### Example 2: Observing Parsing in DevTools

**DevTools Network Panel** shows the loading timeline, including when the HTML parser blocked on scripts. The **Performance** panel shows a flame chart of parsing, styling, layout, and painting.

**DevTools Elements Panel** shows the live DOM tree as the parser builds it (though it updates after the fact). You can see the DOM structure.

### Example 3: Handling Malformed HTML (Error Recovery)

```html
<p>This is a <strong>bold text.</p>
<p>Another paragraph.
```

**What the browser does:**
- The first `<p>` is opened.
- Inside, `<strong>` is opened.
- The parser encounters `</p>` but the current open element is `<strong>`. It **implicitly closes** the `<strong>` and then closes the `<p>`.
- The second `<p>` is opened, but missing a closing tag; the parser will implicitly close it at the end of the body.

**Resulting DOM (conceptually):**
```html
<p>This is a <strong>bold text.</strong></p>
<p>Another paragraph.</p>
```

This illustrates the forgiving nature of the HTML parser, which is a key reason why the web is so resilient.

### Example 4: Script Placement Impact

**Without `async`/`defer` (blocking):**
```html
<script src="heavy.js"></script>
```
The parser pauses until `heavy.js` is downloaded and executed. This can delay rendering.

**With `defer` (non‑blocking, executed after parsing):**
```html
<script src="app.js" defer></script>
```
The script is fetched in parallel; parsing continues; the script runs after the DOM is complete, but before `DOMContentLoaded`.

**With `async` (non‑blocking, executed when ready):**
```html
<script src="analytics.js" async></script>
```
The script is fetched in parallel; it executes as soon as it's ready, potentially before the DOM is fully built. Useful for independent scripts.

---

## 10. Common Use Cases

- **Optimising First Paint** – understanding parsing blockers helps you load critical CSS and scripts efficiently.
- **Implementing Single Page Apps (SPAs)** – you need to understand how the history API and DOM updates affect the parsing/render cycle.
- **Server‑Side Rendering (SSR)** – pre‑rendered HTML is parsed faster because the DOM is already complete, reducing client‑side work.
- **Lazy Loading** – deferring non‑critical resources (images, scripts) to improve initial parse time.
- **Error Handling** – knowing that the parser recovers from errors helps you write robust HTML.
- **Accessibility** – screen readers use the DOM; understanding parsing ensures your content is correctly exposed.

---

## 11. Best Practices

1. **Place `<script>` tags at the bottom** of the `<body>` or use `defer`/`async` to avoid blocking parsing.

2. **Use `defer` for most scripts** that depend on the DOM (e.g., your application code). Use `async` for independent scripts (analytics, ads).

3. **Preload critical CSS** (`<link rel="preload" as="style" href="critical.css">`) to ensure it's loaded early.

4. **Avoid `document.write()`** – it can block parsing and cause performance issues, especially on slow connections.

5. **Minify HTML** – reduces download size and speeds up tokenization.

6. **Use valid HTML** – while the parser can recover, invalid markup can lead to unexpected DOM structures, hurting layout and accessibility.

7. **Set the character encoding** (`<meta charset="UTF-8">`) early in the `<head>` to prevent the parser from having to reload.

8. **Avoid large inline scripts** – they block parsing; extract them to external files with `defer`.

9. **Use `fetchpriority`** to hint the browser about critical resources (e.g., `fetchpriority="high"` on hero images).

10. **Test with the Network panel** – simulate slow connections to see how parsing behaves under latency.

---

## 12. Common Mistakes

### ❌ Mistake: Placing Scripts in the `<head>` Without `defer` or `async`
```html
<head>
    <script src="large.js"></script>
</head>
```
**Why it's wrong**: The parser blocks, delaying rendering. Users see a blank page until the script loads.

**✅ Correct**:
```html
<head>
    <script src="large.js" defer></script>
</head>
```

### ❌ Mistake: Not Specifying `charset`
```html
<head>
    <title>Page</title>
</head>
```
**Why it's wrong**: If the encoding isn't specified, the browser may guess incorrectly, leading to garbled text.

**✅ Correct**:
```html
<meta charset="UTF-8">
```

### ❌ Mistake: Over‑Nesting Elements
```html
<div><div><div><div><p>Deep nesting</p></div></div></div></div>
```
**Why it's wrong**: Deeper nesting slows parsing and increases DOM size, hurting performance.

**✅ Correct**: Keep nesting shallow.

### ❌ Mistake: Using `async` for Scripts That Depend on the DOM
```html
<script src="app.js" async></script>
```
**Why it's wrong**: The script may execute before the DOM is ready, causing errors.

**✅ Correct**: Use `defer` instead.

### ❌ Mistake: Inlining Large CSS in the `<head>`
```html
<style>
    /* 100KB of CSS */
</style>
```
**Why it's wrong**: Blocks rendering until the CSS is parsed; increases HTML size.

**✅ Correct**: Use external `defer` or preload critical CSS and load the rest asynchronously.

---

## 13. Performance Considerations

- **Parse time** – smaller DOM trees parse faster. Avoid unnecessary elements.
- **Script blocking** – each blocking script delays parsing. Use `async`/`defer`.
- **CSS blocking** – CSS blocks rendering, not parsing. However, large CSS delays first paint.
- **Re‑flows and re‑paints** – after the initial parse, JavaScript can trigger re‑flows (layout recalculations) and re‑paints, which are expensive. Batch DOM updates.
- **Chunked encoding** – the browser can start parsing before the entire HTML is downloaded (streaming). Use this to your advantage by sending critical HTML first.
- **Preload scanner** – browsers have a "preload scanner" that runs in parallel with the parser to discover and fetch resources early, improving performance.

---

## 14. Security Considerations

- **XSS (Cross‑Site Scripting)** – if an attacker injects `<script>` tags into your HTML, the parser will execute them. Always sanitize user input.
- **`document.write()`** – can be used to inject scripts; browsers block it for slow network conditions to prevent attacks.
- **CSP (Content Security Policy)** – restricts which scripts can be executed, mitigating XSS.
- **Subresource Integrity (SRI)** – ensures that fetched scripts haven't been tampered with (`integrity` attribute).

---

## 15. Debugging Tips

- **DevTools Elements panel** – inspect the DOM tree; see how the parser structured your HTML.
- **Network panel** – view the order and timing of resource fetches; identify blockers.
- **Performance panel** – record a page load; see the parser's timeline (yellow bars) and identify long tasks.
- **Coverage tool** – shows unused CSS/JS; helps reduce parse time.
- **Console** – check for parser errors (e.g., "unclosed tags" warnings).
- **`DOMContentLoaded` and `load` events** – measure when the DOM is ready and when all resources are loaded.

---

## 16. When to Use

- **Always** – the browser's parsing is automatic; you don't "use" it, but you must be aware of it.
- **Use knowledge of parsing** when optimising page load performance, especially for mobile users.
- **Use `defer`/`async`** when you have scripts that are not critical for initial rendering.

---

## 17. When Not to Use

- You can't "turn off" parsing – it's an intrinsic part of the browser.
- However, you can **avoid** certain practices (like blocking scripts) that harm parsing.

---

## 18. Related Concepts

- DOM (Document Object Model)
- CSSOM (CSS Object Model)
- Render Tree
- Layout / Reflow / Repaint
- Critical Rendering Path
- Preload Scanner
- Resource Hints (preconnect, preload, prefetch)
- Async / Defer Scripts
- `DOMContentLoaded` and `load` events
- Content Security Policy (CSP)
- Subresource Integrity (SRI)
- Server‑Side Rendering (SSR)
- Progressive Web Apps (Service Workers)
- Web Performance APIs (Navigation Timing, Resource Timing)

---

## 19. Did You Know?

- **The HTML parser is specified in over 1,200 pages** of the HTML Living Standard. It defines every possible token and error recovery case.

- **The parser can run in "incremental" mode** – it can start building the DOM before the entire HTML document is downloaded. This is why you sometimes see a page render progressively.

- **The browser's "preload scanner"** was introduced in Chrome 2013 and is now in all modern browsers. It scans the HTML ahead of the parser to fetch resources (images, scripts, styles) early, often reducing load times by hundreds of milliseconds.

- **Error recovery is so robust** that even if you write `<p>Hello <p>World`, the parser will produce two paragraphs (it implicitly closes the first `<p>`). This is why HTML is sometimes called "tolerant".

- **`<script>` tags with `async`** can execute in any order. If you have multiple async scripts, the order is not guaranteed. Use `defer` if order matters.

- **The `document.write()` method** is considered harmful in modern HTML5 because it can break streaming parsing. Many browsers throttle it on slow connections.

- **The DOM tree and the HTML source are not always 1:1** – the parser may add missing elements (like `<tbody>` inside a `<table>`), omit elements (like `<html>` if missing), or change nesting to correct errors.

---

## 20. Summary

- **Browsers parse HTML** through a defined pipeline: tokenization → tree construction → DOM.
- **CSS is parsed** separately into the CSSOM; both are combined into a render tree.
- **Layout** calculates positions; **paint** draws pixels; **compositing** layers them.
- **Scripts block parsing** unless marked `async` or `defer`.
- **CSS blocks rendering** (not parsing) – the browser waits for styles before painting.
- **Error recovery** ensures that even malformed HTML produces a consistent DOM.
- **The parser is incremental** – it can start rendering before the entire HTML is downloaded.
- **Understanding parsing** is essential for performance optimisation (critical rendering path).
- **Best practices** include using `defer` for scripts, placing styles in the `<head>`, and avoiding `document.write()`.
- **Debugging** tools (DevTools) allow you to inspect DOM, network timing, and performance.

---

