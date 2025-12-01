# Day 1: HTML Basics - Document Structure & Text Elements

> **Estimated Time:** 3-4 hours | **Difficulty:** Beginner | **Prerequisites:** None

---

## Learning Objectives

By the end of this lesson, you will be able to:

- [ ] Understand what HTML is and its role in web development
- [ ] Create a properly structured HTML5 document from scratch
- [ ] Use all heading levels (h1-h6) with correct hierarchy
- [ ] Format text using semantic inline elements
- [ ] Distinguish between block and inline elements
- [ ] Write clean, well-commented HTML code
- [ ] Validate your HTML using W3C Validator

---

## Table of Contents

1. [What is HTML?](#what-is-html)
2. [Setting Up Your Environment](#setting-up-your-environment)
3. [HTML Element Anatomy](#html-element-anatomy)
4. [HTML Document Structure](#html-document-structure)
5. [Text Elements](#text-elements)
6. [Inline Text Formatting](#inline-text-formatting)
7. [Block vs Inline Elements](#block-vs-inline-elements)
8. [HTML Comments](#html-comments)
9. [HTML Entities](#html-entities)
10. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
11. [Live Examples](#live-examples)
12. [Assignments](#assignments)
13. [Mini Projects](#mini-projects)
14. [Quick Reference](#quick-reference)
15. [Further Learning](#further-learning)

---

## What is HTML?

**HTML** (HyperText Markup Language) is the standard language for creating web pages. It describes the **structure** of a web page using elements that tell the browser how to display content.

### The Web Development Trinity

```
┌─────────────────────────────────────────────────────────────┐
│                      WEB PAGE                               │
├─────────────────┬─────────────────┬─────────────────────────┤
│      HTML       │       CSS       │       JavaScript        │
│   (Structure)   │    (Style)      │     (Behavior)          │
│                 │                 │                         │
│   "What it is"  │ "How it looks"  │   "What it does"        │
│                 │                 │                         │
│   • Content     │ • Colors        │   • Interactions        │
│   • Headings    │ • Fonts         │   • Animations          │
│   • Paragraphs  │ • Layout        │   • Data handling       │
│   • Images      │ • Spacing       │   • Form validation     │
└─────────────────┴─────────────────┴─────────────────────────┘
```

**Think of it like building a house:**
- **HTML** = The skeleton/frame (walls, doors, rooms)
- **CSS** = The decoration (paint, furniture, curtains)
- **JavaScript** = The functionality (electricity, plumbing, smart home)

---

## Setting Up Your Environment

### What You Need

1. **A Text Editor** (choose one):
   - [VS Code](https://code.visualstudio.com/) (Recommended)
   - [Sublime Text](https://www.sublimetext.com/)
   - [Atom](https://atom.io/)

2. **A Web Browser** (for testing):
   - Chrome (Recommended - best DevTools)
   - Firefox
   - Edge

### VS Code Extensions for HTML

Install these extensions for a better experience:

| Extension | Purpose |
|-----------|---------|
| **Live Server** | Auto-refresh browser on save |
| **Auto Rename Tag** | Rename paired tags together |
| **HTML CSS Support** | CSS class autocomplete |
| **Prettier** | Code formatting |

### Your First HTML File

1. Create a new folder called `html-practice`
2. Open it in VS Code
3. Create a new file called `index.html`
4. Type `!` and press `Tab` to generate HTML boilerplate

---

## HTML Element Anatomy

Every HTML element follows this structure:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  <tagname attribute="value">Content goes here</tagname>         │
│   ↑          ↑         ↑           ↑              ↑             │
│   │          │         │           │              │             │
│   │          │         │           │              └─ Closing Tag│
│   │          │         │           └─ Content                   │
│   │          │         └─ Attribute Value                       │
│   │          └─ Attribute Name                                  │
│   └─ Opening Tag                                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Example Breakdown

```html
<a href="https://google.com" target="_blank">Visit Google</a>
```

| Part | Value | Description |
|------|-------|-------------|
| Opening Tag | `<a>` | Anchor element starts |
| Attribute 1 | `href="https://google.com"` | Link destination |
| Attribute 2 | `target="_blank"` | Open in new tab |
| Content | `Visit Google` | Clickable text |
| Closing Tag | `</a>` | Anchor element ends |

### Self-Closing (Void) Elements

Some elements don't have content and don't need a closing tag:

```html
<img src="photo.jpg" alt="A beautiful sunset" />
<br />
<hr />
<input type="text" />
<meta charset="UTF-8" />
<link rel="stylesheet" href="styles.css" />
```

> **Note:** The trailing `/` is optional in HTML5 but recommended for clarity.

---

## HTML Document Structure

Every HTML document must follow this structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="A brief description of your page">
    <title>Page Title - Shown in Browser Tab</title>
</head>
<body>
    <!-- Your visible content goes here -->
</body>
</html>
```

### Visual Structure

```
┌──────────────────────────────────────────────────────────────┐
│ <!DOCTYPE html>  ← Tells browser this is HTML5               │
├──────────────────────────────────────────────────────────────┤
│ <html lang="en">  ← Root element, specifies language         │
│ ┌────────────────────────────────────────────────────────┐   │
│ │ <head>  ← Metadata (not visible on page)               │   │
│ │   • charset (character encoding)                       │   │
│ │   • viewport (responsive settings)                     │   │
│ │   • title (browser tab)                                │   │
│ │   • description (SEO)                                  │   │
│ │   • CSS links                                          │   │
│ │   • Favicon                                            │   │
│ │ </head>                                                │   │
│ └────────────────────────────────────────────────────────┘   │
│ ┌────────────────────────────────────────────────────────┐   │
│ │ <body>  ← Visible content                              │   │
│ │   • Headings                                           │   │
│ │   • Paragraphs                                         │   │
│ │   • Images                                             │   │
│ │   • Links                                              │   │
│ │   • Everything users see                               │   │
│ │ </body>                                                │   │
│ └────────────────────────────────────────────────────────┘   │
│ </html>                                                      │
└──────────────────────────────────────────────────────────────┘
```

### Breaking It Down

#### 1. DOCTYPE Declaration

```html
<!DOCTYPE html>
```

- **Must be the first line** of every HTML document
- Tells the browser to use HTML5 standards mode
- Not a tag - it's a declaration

#### 2. HTML Element

```html
<html lang="en">
    <!-- Everything goes here -->
</html>
```

The root element that wraps everything. Common language codes:

| Code | Language |
|------|----------|
| `en` | English |
| `es` | Spanish |
| `fr` | French |
| `de` | German |
| `zh` | Chinese |
| `bn` | Bengali |
| `ar` | Arabic |
| `hi` | Hindi |

#### 3. Head Element

```html
<head>
    <!-- Character encoding - ALWAYS use UTF-8 -->
    <meta charset="UTF-8">

    <!-- Responsive viewport - REQUIRED for mobile -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Page title - Shown in browser tab and search results -->
    <title>My Awesome Website</title>

    <!-- SEO description - Shown in search results -->
    <meta name="description" content="Learn HTML basics with practical examples">

    <!-- Author information -->
    <meta name="author" content="Your Name">

    <!-- Favicon - Browser tab icon -->
    <link rel="icon" href="favicon.ico" type="image/x-icon">

    <!-- CSS Stylesheet -->
    <link rel="stylesheet" href="styles.css">
</head>
```

#### 4. Body Element

```html
<body>
    <h1>Welcome to My Website</h1>
    <p>This content is visible to users.</p>
</body>
```

---

## Text Elements

### Headings (h1-h6)

Headings create a hierarchical structure for your content. Think of them like a book outline.

**Code:**
```html
<h1>Main Title (Level 1)</h1>
<h2>Section Heading (Level 2)</h2>
<h3>Subsection Heading (Level 3)</h3>
<h4>Sub-subsection (Level 4)</h4>
<h5>Minor Heading (Level 5)</h5>
<h6>Smallest Heading (Level 6)</h6>
```

**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Main Title (Level 1)                    ← Largest, bold    │
│  ═══════════════════════════════════════                    │
│                                                             │
│  Section Heading (Level 2)               ← Large, bold      │
│  ───────────────────────────────────────                    │
│                                                             │
│  Subsection Heading (Level 3)            ← Medium-large     │
│                                                             │
│  Sub-subsection (Level 4)                ← Medium           │
│                                                             │
│  Minor Heading (Level 5)                 ← Small            │
│                                                             │
│  Smallest Heading (Level 6)              ← Smallest         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Heading Hierarchy Rules

```
✅ CORRECT                          ❌ WRONG
─────────────────────────────────────────────────────────────
<h1>Website Title</h1>              <h1>Title</h1>
  <h2>Section 1</h2>                  <h3>Section</h3>  ← Skipped h2!
    <h3>Subsection 1.1</h3>           <h1>Another</h1>  ← Multiple h1!
    <h3>Subsection 1.2</h3>
  <h2>Section 2</h2>
    <h3>Subsection 2.1</h3>
```

**Best Practices:**
- ✅ Use only ONE `<h1>` per page (main title)
- ✅ Don't skip levels (h1 → h2 → h3, not h1 → h3)
- ✅ Use headings for structure, NOT for styling
- ❌ Don't use `<h3>` just because you want smaller text

### Real-World Example: Blog Post Structure

```html
<h1>Complete Guide to Learning HTML</h1>

<h2>1. Introduction</h2>
<p>HTML is the foundation of web development...</p>

<h2>2. Getting Started</h2>

<h3>2.1 Installing a Text Editor</h3>
<p>First, download VS Code...</p>

<h3>2.2 Creating Your First File</h3>
<p>Create a file called index.html...</p>

<h2>3. Basic Elements</h2>

<h3>3.1 Headings</h3>
<p>Headings range from h1 to h6...</p>

<h3>3.2 Paragraphs</h3>
<p>The p element defines paragraphs...</p>

<h2>4. Conclusion</h2>
<p>Now you know the basics...</p>
```

### Paragraphs

The `<p>` element defines a paragraph of text.

**Code:**
```html
<p>This is a paragraph. It contains a block of text that forms a
distinct section of content. Browsers automatically add space
before and after paragraphs.</p>

<p>This is another paragraph. Each paragraph starts on a new line
with vertical spacing between them.</p>
```

**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  This is a paragraph. It contains a block of text that      │
│  forms a distinct section of content. Browsers              │
│  automatically add space before and after paragraphs.       │
│                                                             │
│  This is another paragraph. Each paragraph starts on a      │
│  new line with vertical spacing between them.               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Line Breaks and Horizontal Rules

#### Line Break `<br>`

Use for line breaks **within** content (addresses, poems):

**Code:**
```html
<p>
    John Doe<br>
    123 Main Street<br>
    New York, NY 10001<br>
    United States
</p>
```

**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│  John Doe                                                   │
│  123 Main Street                                            │
│  New York, NY 10001                                         │
│  United States                                              │
└─────────────────────────────────────────────────────────────┘
```

#### Horizontal Rule `<hr>`

Creates a thematic break (horizontal line):

**Code:**
```html
<h2>Chapter 1: The Beginning</h2>
<p>It was a dark and stormy night...</p>

<hr>

<h2>Chapter 2: The Journey</h2>
<p>The next morning brought sunshine...</p>
```

**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│  Chapter 1: The Beginning                                   │
│                                                             │
│  It was a dark and stormy night...                          │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  Chapter 2: The Journey                                     │
│                                                             │
│  The next morning brought sunshine...                       │
└─────────────────────────────────────────────────────────────┘
```

---

## Inline Text Formatting

Inline elements style text **within** block elements like `<p>`.

### Strong and Emphasis

**Code:**
```html
<p>This is <strong>very important</strong> information.</p>
<p>You <em>must</em> read this carefully.</p>
<p>This is <strong><em>extremely critical</em></strong>!</p>
```

**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│  This is **very important** information.                    │
│  You _must_ read this carefully.                            │
│  This is **_extremely critical_**!                          │
└─────────────────────────────────────────────────────────────┘
```

### Semantic vs Presentational Elements

| Semantic (Use These) | Visual | Presentational (Avoid) | Visual |
|---------------------|--------|----------------------|--------|
| `<strong>` | **Bold** | `<b>` | **Bold** |
| `<em>` | *Italic* | `<i>` | *Italic* |

**Why semantic matters:**
- Screen readers announce `<strong>` with emphasis
- Search engines understand importance
- Better accessibility

### All Inline Text Elements

**Code with Preview:**

```html
<!-- Strong - Important text -->
<p>Warning: <strong>Do not delete this file!</strong></p>
```
**Preview:** Warning: **Do not delete this file!**

---

```html
<!-- Emphasis - Stressed text -->
<p>I <em>really</em> want to learn HTML.</p>
```
**Preview:** I *really* want to learn HTML.

---

```html
<!-- Mark - Highlighted/relevant text -->
<p>Search results for "HTML": <mark>HTML</mark> is a markup language.</p>
```
**Preview:** Search results for "HTML": ==HTML== is a markup language.

---

```html
<!-- Delete - Removed text -->
<p>Price: <del>$100</del> $75</p>
```
**Preview:** Price: ~~$100~~ $75

---

```html
<!-- Insert - Added text -->
<p>Status: <del>Pending</del> <ins>Approved</ins></p>
```
**Preview:** Status: ~~Pending~~ <u>Approved</u>

---

```html
<!-- Subscript - Below baseline -->
<p>Water formula: H<sub>2</sub>O</p>
<p>Carbon dioxide: CO<sub>2</sub></p>
```
**Preview:**
```
Water formula: H₂O
Carbon dioxide: CO₂
```

---

```html
<!-- Superscript - Above baseline -->
<p>Einstein's equation: E = mc<sup>2</sup></p>
<p>Area: 100 m<sup>2</sup></p>
<p>Footnote reference<sup>[1]</sup></p>
```
**Preview:**
```
Einstein's equation: E = mc²
Area: 100 m²
Footnote reference¹
```

---

```html
<!-- Small - Fine print -->
<p><small>Copyright © 2024. All rights reserved.</small></p>
<p><small>Terms and conditions apply.</small></p>
```
**Preview:** <sub>Copyright © 2024. All rights reserved.</sub>

---

```html
<!-- Code - Inline code -->
<p>Use the <code>console.log()</code> function to debug.</p>
<p>The <code>&lt;div&gt;</code> element is a container.</p>
```
**Preview:** Use the `console.log()` function to debug.

---

```html
<!-- Keyboard - Keyboard input -->
<p>Press <kbd>Ctrl</kbd> + <kbd>C</kbd> to copy.</p>
<p>Press <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>I</kbd> to open DevTools.</p>
```
**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│  Press [Ctrl] + [C] to copy.                                │
│  Press [Ctrl] + [Shift] + [I] to open DevTools.             │
└─────────────────────────────────────────────────────────────┘
```

---

```html
<!-- Abbreviation - With tooltip -->
<p>The <abbr title="World Wide Web">WWW</abbr> was invented in 1989.</p>
<p>Use <abbr title="HyperText Markup Language">HTML</abbr> for structure.</p>
```
**Preview:** The WWW (hover shows: "World Wide Web") was invented in 1989.

---

```html
<!-- Quote - Inline quotation -->
<p>She said, <q>The future belongs to those who learn.</q></p>
```
**Preview:** She said, "The future belongs to those who learn."

---

```html
<!-- Span - Generic inline container (for styling) -->
<p>My favorite color is <span style="color: blue;">blue</span>.</p>
```
**Preview:** My favorite color is <span style="color: blue;">blue</span>.

### Complete Inline Elements Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Inline Elements Demo</title>
</head>
<body>
    <h1>HTML Inline Elements Showcase</h1>

    <h2>Text Importance</h2>
    <p>This text contains <strong>strongly important</strong> and
    <em>emphasized</em> words.</p>

    <h2>Scientific Notation</h2>
    <p>Water (H<sub>2</sub>O) boils at 100°C or 212°F.</p>
    <p>The speed of light is approximately 3 × 10<sup>8</sup> m/s.</p>

    <h2>Pricing Update</h2>
    <p>Original price: <del>$199.99</del></p>
    <p>Sale price: <ins>$149.99</ins></p>
    <p>You save: <mark>$50.00</mark></p>

    <h2>Code References</h2>
    <p>In JavaScript, use <code>document.getElementById()</code> to
    select elements.</p>
    <p>To save your file, press <kbd>Ctrl</kbd> + <kbd>S</kbd>.</p>

    <h2>Abbreviations</h2>
    <p>The <abbr title="Application Programming Interface">API</abbr>
    documentation is available online.</p>

    <h2>Quotation</h2>
    <p>As Tim Berners-Lee said, <q>The Web does not just connect
    machines, it connects people.</q></p>

    <hr>

    <p><small>© 2024 HTML Tutorial. All rights reserved.</small></p>
</body>
</html>
```

**Browser Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│  HTML Inline Elements Showcase                              │
│  ═══════════════════════════════════════                    │
│                                                             │
│  Text Importance                                            │
│  ───────────────────────────────────────                    │
│  This text contains **strongly important** and              │
│  _emphasized_ words.                                        │
│                                                             │
│  Scientific Notation                                        │
│  ───────────────────────────────────────                    │
│  Water (H₂O) boils at 100°C or 212°F.                       │
│  The speed of light is approximately 3 × 10⁸ m/s.           │
│                                                             │
│  Pricing Update                                             │
│  ───────────────────────────────────────                    │
│  Original price: ~~$199.99~~                                │
│  Sale price: _$149.99_                                      │
│  You save: [HIGHLIGHTED] $50.00                             │
│                                                             │
│  Code References                                            │
│  ───────────────────────────────────────                    │
│  In JavaScript, use `document.getElementById()` to          │
│  select elements.                                           │
│  To save your file, press [Ctrl] + [S].                     │
│                                                             │
│  Abbreviations                                              │
│  ───────────────────────────────────────                    │
│  The API (hover: Application Programming Interface)         │
│  documentation is available online.                         │
│                                                             │
│  Quotation                                                  │
│  ───────────────────────────────────────                    │
│  As Tim Berners-Lee said, "The Web does not just connect    │
│  machines, it connects people."                             │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  © 2024 HTML Tutorial. All rights reserved.                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Block vs Inline Elements

This is one of the most important concepts in HTML!

### Visual Comparison

```
BLOCK ELEMENTS                      INLINE ELEMENTS
─────────────────────────────────────────────────────────────────
┌───────────────────────────────┐   ┌─────┐┌─────┐┌─────┐
│ <div> Takes full width        │   │span ││span ││span │
└───────────────────────────────┘   └─────┘└─────┘└─────┘
┌───────────────────────────────┐
│ <p> Starts on new line        │   They stay on the same line
└───────────────────────────────┘   and only take needed space
┌───────────────────────────────┐
│ <h1> Also full width          │
└───────────────────────────────┘
```

### Block Elements

**Characteristics:**
- Start on a new line
- Take up the full width available
- Can contain other block and inline elements
- Have margins above and below

**Common Block Elements:**
```html
<div>       <!-- Generic container -->
<p>         <!-- Paragraph -->
<h1>-<h6>   <!-- Headings -->
<hr>        <!-- Horizontal rule -->
<blockquote> <!-- Block quotation -->
<pre>       <!-- Preformatted text -->
```

**Example:**
```html
<div>First div - takes full width</div>
<div>Second div - starts on new line</div>
<p>A paragraph is also a block element</p>
```

**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│ First div - takes full width                                │
├─────────────────────────────────────────────────────────────┤
│ Second div - starts on new line                             │
├─────────────────────────────────────────────────────────────┤
│ A paragraph is also a block element                         │
└─────────────────────────────────────────────────────────────┘
```

### Inline Elements

**Characteristics:**
- Don't start on a new line
- Only take up as much width as necessary
- Cannot contain block elements
- Flow with surrounding text

**Common Inline Elements:**
```html
<span>      <!-- Generic inline container -->
<strong>    <!-- Strong importance -->
<em>        <!-- Emphasis -->
<a>         <!-- Links -->
<code>      <!-- Code -->
<mark>      <!-- Highlighted text -->
```

**Example:**
```html
<span>First span</span>
<span>Second span</span>
<span>Third span</span>
<strong>Strong text</strong>
<em>Emphasized text</em>
```

**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│ First spanSecond spanThird span**Strong text**_Emphasized_  │
└─────────────────────────────────────────────────────────────┘
```

### The `<div>` and `<span>` Elements

These are **generic containers** with no semantic meaning:

| Element | Type | Use Case |
|---------|------|----------|
| `<div>` | Block | Group block-level content for styling/layout |
| `<span>` | Inline | Style specific text within a paragraph |

**Example:**
```html
<div class="card">
    <h2>Card Title</h2>
    <p>This is a <span class="highlight">highlighted word</span> in the card.</p>
</div>
```

---

## HTML Comments

Comments are notes in your code that browsers ignore.

```html
<!-- This is a single-line comment -->

<!--
    This is a
    multi-line comment
    for longer explanations
-->

<!-- TODO: Add navigation menu here -->

<!-- FIXME: This section needs refactoring -->
```

**Use Comments For:**
- Explaining complex code
- Marking sections of your page
- Temporarily disabling code
- Leaving notes for yourself or team members
- TODO items

**Example: Well-Commented Code**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My Website</title>
</head>
<body>
    <!-- ============================================
         HEADER SECTION
         Contains logo and main navigation
    ============================================= -->
    <div class="header">
        <h1>My Website</h1>
        <!-- TODO: Add navigation menu -->
    </div>

    <!-- ============================================
         MAIN CONTENT SECTION
    ============================================= -->
    <div class="main">
        <h2>Welcome</h2>
        <p>This is the main content area.</p>
    </div>

    <!-- ============================================
         FOOTER SECTION
    ============================================= -->
    <div class="footer">
        <p>&copy; 2024 My Website</p>
    </div>
</body>
</html>
```

---

## HTML Entities

Special characters that can't be typed directly or have special meaning in HTML:

### Common HTML Entities

| Entity | Symbol | Description |
|--------|--------|-------------|
| `&lt;` | < | Less than |
| `&gt;` | > | Greater than |
| `&amp;` | & | Ampersand |
| `&quot;` | " | Quotation mark |
| `&apos;` | ' | Apostrophe |
| `&nbsp;` | (space) | Non-breaking space |
| `&copy;` | © | Copyright |
| `&reg;` | ® | Registered trademark |
| `&trade;` | ™ | Trademark |
| `&mdash;` | — | Em dash |
| `&ndash;` | – | En dash |
| `&hellip;` | … | Ellipsis |
| `&euro;` | € | Euro sign |
| `&pound;` | £ | Pound sign |
| `&yen;` | ¥ | Yen sign |

### When to Use Entities

**Code:**
```html
<!-- Displaying HTML code as text -->
<p>The <code>&lt;div&gt;</code> element is a container.</p>

<!-- Copyright symbol -->
<p>&copy; 2024 My Company&trade;</p>

<!-- Preventing line breaks -->
<p>Price:&nbsp;$100</p>

<!-- Mathematical expressions -->
<p>5 &lt; 10 &amp;&amp; 10 &gt; 5</p>
```

**Preview:**
```
The <div> element is a container.
© 2024 My Company™
Price: $100 (won't break between Price: and $100)
5 < 10 && 10 > 5
```

---

## Common Mistakes to Avoid

### 1. Missing DOCTYPE

```html
<!-- ❌ WRONG -->
<html>
<head>...</head>
</html>

<!-- ✅ CORRECT -->
<!DOCTYPE html>
<html>
<head>...</head>
</html>
```

### 2. Missing Closing Tags

```html
<!-- ❌ WRONG -->
<p>First paragraph
<p>Second paragraph

<!-- ✅ CORRECT -->
<p>First paragraph</p>
<p>Second paragraph</p>
```

### 3. Improper Nesting

```html
<!-- ❌ WRONG - Tags overlap -->
<p><strong>Bold and <em>italic</strong> text</em></p>

<!-- ✅ CORRECT - Tags nest properly -->
<p><strong>Bold and <em>italic</em></strong> text</p>
```

**Visual:**
```
WRONG:   <strong> ─────────┐
              <em> ────│────┐
         </strong> ────┘    │   ← Tags cross!
              </em> ────────┘

CORRECT: <strong> ─────────┐
              <em> ────┐   │
              </em> ────┘   │   ← Tags nest properly
         </strong> ────────┘
```

### 4. Skipping Heading Levels

```html
<!-- ❌ WRONG -->
<h1>Main Title</h1>
<h4>Section</h4>  <!-- Skipped h2, h3! -->

<!-- ✅ CORRECT -->
<h1>Main Title</h1>
<h2>Section</h2>
```

### 5. Using Headings for Styling

```html
<!-- ❌ WRONG - Using h3 for smaller text -->
<h1>Title</h1>
<h3>I want this smaller but it's not a real heading</h3>

<!-- ✅ CORRECT - Use CSS for styling -->
<h1>Title</h1>
<p class="subtitle">This is a subtitle styled with CSS</p>
```

### 6. Multiple h1 Elements

```html
<!-- ❌ WRONG -->
<h1>Website Name</h1>
<h1>Page Title</h1>
<h1>Another Section</h1>

<!-- ✅ CORRECT -->
<h1>Website Name - Page Title</h1>
<h2>Section 1</h2>
<h2>Section 2</h2>
```

### 7. Using `<br>` for Spacing

```html
<!-- ❌ WRONG -->
<p>First paragraph</p>
<br>
<br>
<br>
<p>Second paragraph</p>

<!-- ✅ CORRECT - Use CSS for spacing -->
<p>First paragraph</p>
<p>Second paragraph</p>
```

---

## Live Examples

### Example 1: Personal Profile Page

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Personal profile page of John Doe, Web Developer">
    <title>John Doe - Web Developer</title>
</head>
<body>
    <h1>John Doe</h1>
    <p><em>Full-Stack Web Developer</em></p>

    <hr>

    <h2>About Me</h2>
    <p>Hello! I'm <strong>John Doe</strong>, a passionate web developer
    with <mark>5 years of experience</mark> building modern web applications.</p>

    <p>I specialize in <abbr title="HyperText Markup Language">HTML</abbr>,
    <abbr title="Cascading Style Sheets">CSS</abbr>, and
    <abbr title="JavaScript">JS</abbr>.</p>

    <h2>Contact</h2>
    <p>
        123 Developer Lane<br>
        San Francisco, CA 94102<br>
        United States
    </p>

    <h2>Favorite Quote</h2>
    <p><q>The best way to predict the future is to create it.</q>
    — <em>Abraham Lincoln</em></p>

    <hr>

    <p><small>&copy; 2024 John Doe. All rights reserved.</small></p>
</body>
</html>
```

**Preview:**
```
┌─────────────────────────────────────────────────────────────┐
│  John Doe                                                   │
│  _Full-Stack Web Developer_                                 │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  About Me                                                   │
│                                                             │
│  Hello! I'm **John Doe**, a passionate web developer with   │
│  [HIGHLIGHTED: 5 years of experience] building modern       │
│  web applications.                                          │
│                                                             │
│  I specialize in HTML, CSS, and JS.                         │
│                                                             │
│  Contact                                                    │
│                                                             │
│  123 Developer Lane                                         │
│  San Francisco, CA 94102                                    │
│  United States                                              │
│                                                             │
│  Favorite Quote                                             │
│                                                             │
│  "The best way to predict the future is to create it."      │
│  — _Abraham Lincoln_                                        │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│  © 2024 John Doe. All rights reserved.                      │
└─────────────────────────────────────────────────────────────┘
```

### Example 2: Technical Documentation Page

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="JavaScript Console Methods Reference">
    <title>JavaScript Console Methods - Documentation</title>
</head>
<body>
    <h1>JavaScript Console Methods</h1>
    <p><small>Last updated: December 2024</small></p>

    <hr>

    <h2>Overview</h2>
    <p>The <code>console</code> object provides access to the browser's
    debugging console. Open it with <kbd>F12</kbd> or
    <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>I</kbd>.</p>

    <h2>Methods</h2>

    <h3><code>console.log()</code></h3>
    <p>Outputs a message to the console. This is the most commonly
    used method for debugging.</p>
    <p><strong>Usage:</strong> General debugging and variable inspection.</p>

    <h3><code>console.error()</code></h3>
    <p>Outputs an <strong>error message</strong> to the console,
    typically displayed in <span style="color: red;">red</span>.</p>
    <p><strong>Usage:</strong> Logging errors and exceptions.</p>

    <h3><code>console.warn()</code></h3>
    <p>Outputs a <strong>warning message</strong> to the console,
    typically displayed in <span style="color: orange;">yellow/orange</span>.</p>
    <p><strong>Usage:</strong> Non-critical issues and deprecation notices.</p>

    <h3><del><code>console.debug()</code></del> <ins>(Use <code>console.log()</code>)</ins></h3>
    <p><small>Deprecated in some browsers. Use <code>console.log()</code> instead.</small></p>

    <hr>

    <h2>Quick Reference</h2>
    <div>
        <p><code>console.log("Hello")</code> — General output</p>
        <p><code>console.error("Error!")</code> — Error output</p>
        <p><code>console.warn("Warning")</code> — Warning output</p>
        <p><code>console.table(data)</code> — Tabular data</p>
        <p><code>console.clear()</code> — Clear console</p>
    </div>

    <hr>

    <p><small>&copy; 2024 JavaScript Documentation.
    <abbr title="Application Programming Interface">API</abbr> reference guide.</small></p>
</body>
</html>
```

### Example 3: Recipe Page

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Easy homemade chocolate chip cookies recipe">
    <title>Classic Chocolate Chip Cookies Recipe</title>
</head>
<body>
    <!-- ========== HEADER ========== -->
    <h1>Classic Chocolate Chip Cookies</h1>
    <p><em>Prep: 15 min | Cook: 12 min | Yield: 24 cookies</em></p>

    <hr>

    <!-- ========== INTRODUCTION ========== -->
    <h2>Introduction</h2>
    <p>These <strong>homemade chocolate chip cookies</strong> are
    <mark>crispy on the outside and chewy on the inside</mark>.
    Perfect for any occasion!</p>

    <!-- ========== INGREDIENTS ========== -->
    <h2>Ingredients</h2>

    <h3>Dry Ingredients</h3>
    <p><em>2 ¼ cups</em> all-purpose flour</p>
    <p><em>1 tsp</em> baking soda</p>
    <p><em>1 tsp</em> salt</p>

    <h3>Wet Ingredients</h3>
    <p><em>1 cup</em> butter, softened</p>
    <p><em>¾ cup</em> granulated sugar</p>
    <p><em>¾ cup</em> packed brown sugar</p>
    <p><em>2</em> large eggs</p>
    <p><em>2 tsp</em> vanilla extract</p>

    <h3>Add-ins</h3>
    <p><em>2 cups</em> chocolate chips</p>

    <!-- ========== INSTRUCTIONS ========== -->
    <h2>Instructions</h2>

    <h3>Step 1: Preheat</h3>
    <p>Preheat your oven to <strong>375°F</strong> (190°C).</p>

    <h3>Step 2: Mix Dry Ingredients</h3>
    <p>In a bowl, combine flour, baking soda, and salt.</p>

    <h3>Step 3: Cream Butter and Sugars</h3>
    <p><mark>This is the most important step!</mark> Beat butter and
    both sugars until light and fluffy, about 3-4 minutes.</p>

    <h3>Step 4: Add Eggs and Vanilla</h3>
    <p>Beat in eggs one at a time, then add vanilla.</p>

    <h3>Step 5: Combine</h3>
    <p>Gradually add dry ingredients to wet ingredients.
    <strong>Do not overmix!</strong></p>

    <h3>Step 6: Add Chocolate Chips</h3>
    <p>Fold in the chocolate chips.</p>

    <h3>Step 7: Bake</h3>
    <p>Drop rounded tablespoons onto baking sheets.
    Bake for <strong>9-12 minutes</strong> until golden brown.</p>

    <!-- ========== CHEF'S NOTES ========== -->
    <hr>

    <h2>Chef's Notes</h2>
    <p><strong>Pro tip:</strong> Chill the dough for 30 minutes for
    thicker cookies.</p>
    <p><strong>Storage:</strong> Keep in an airtight container for
    up to <del>5 days</del> <ins>1 week</ins>.</p>

    <!-- ========== NUTRITION ========== -->
    <h2>Nutrition Info</h2>
    <p><small>Per cookie (approximate):</small></p>
    <p>Calories: 150 | Fat: 7g | Carbs: 21g | Protein: 2g</p>

    <hr>

    <p><small>&copy; 2024 Delicious Recipes</small></p>
</body>
</html>
```

---

## Assignments

### 🟢 Beginner Level

#### Assignment 1: Fix the Broken HTML

Debug and fix all errors in this HTML document:

```html
<html>
<head>
<title>My Page
</head>
<body>
<h1>Welcome</h1>
<p>This is my first paragraph
<p>This is <strong>bold and <em>italic</strong> text</em></p>
<h3>Section Title</h3>
<p>Some content here.
</body>
```

**Requirements:**
- [ ] Add missing DOCTYPE declaration
- [ ] Add `lang` attribute to `<html>`
- [ ] Add required meta tags in `<head>`
- [ ] Fix all missing closing tags
- [ ] Fix improper tag nesting
- [ ] Fix heading hierarchy (h3 should be h2)

**Expected Output:** A valid HTML5 document that passes W3C validation.

---

#### Assignment 2: Personal Profile Page

Create `profile.html` with your personal information.

**Requirements:**
- [ ] Proper HTML5 document structure
- [ ] Meaningful `<title>` and `<meta description>`
- [ ] Your name as `<h1>`
- [ ] "About Me" section with `<h2>` and 2-3 paragraphs
- [ ] Use `<strong>` for important information
- [ ] Use `<em>` for emphasis
- [ ] Use `<mark>` to highlight a key achievement
- [ ] Include `<hr>` divider
- [ ] Footer with `<small>` copyright text

---

### 🟡 Intermediate Level

#### Assignment 3: Recipe Page

Create `recipe.html` for your favorite recipe.

**Requirements:**
- [ ] Complete HTML5 structure with SEO meta tags
- [ ] Recipe name as `<h1>`
- [ ] Author and date using `<small>` and `<em>`
- [ ] Sections: Introduction, Ingredients, Instructions, Chef's Notes
- [ ] Proper heading hierarchy (h1 → h2 → h3)
- [ ] Use `<strong>` for important tips
- [ ] Use `<em>` for measurements
- [ ] Use `<mark>` to highlight key steps
- [ ] Use `<del>` and `<ins>` for recipe modifications
- [ ] Include nutritional info with `<sub>`/`<sup>` if needed
- [ ] Use `<hr>` between major sections
- [ ] Well-organized HTML comments

---

#### Assignment 4: Technical Documentation

Create `documentation.html` - a JavaScript console methods reference.

**Requirements:**
- [ ] Title: "JavaScript Console Methods Reference"
- [ ] Overview section with keyboard shortcuts using `<kbd>`
- [ ] At least 4 method sections (log, error, warn, table)
- [ ] Each method has:
  - Name in `<code>` tags
  - Description paragraph
  - Usage example in `<code>`
- [ ] Use `<abbr>` for abbreviations (DOM, API, JS)
- [ ] Show deprecated method with `<del>` and replacement with `<ins>`
- [ ] Use `<div>` to group related content
- [ ] Include a quick reference section

---

### 🔴 Advanced Level

#### Assignment 5: Multi-Section Article

Create `article.html` - "The History of the World Wide Web"

**Requirements:**
- [ ] Professional HTML5 structure with all meta tags
- [ ] Article structured with:
  - Main title (h1)
  - Author byline with `<small>` and `<em>`
  - Table of Contents (text list of sections)
  - At least 5 major sections (h2) with subsections (h3)
  - Proper paragraph structure
- [ ] Demonstrate ALL inline text elements:
  - `<strong>`, `<em>`, `<mark>`, `<del>`, `<ins>`
  - `<sub>`, `<sup>`, `<small>`, `<code>`, `<kbd>`
  - `<abbr>` with title (WWW, HTML, HTTP, URL, DNS)
  - `<q>` for Tim Berners-Lee quotes
- [ ] Use `<div>` and `<span>` appropriately
- [ ] Comprehensive HTML comments
- [ ] Footer with copyright and last updated date
- [ ] Must pass W3C Validator

---

## Mini Projects

### 🟢 Beginner Project: Introduction Card

**Goal:** Create a simple "Hello, I'm [Name]" introduction card.

**File:** `intro-card.html`

**Time:** 20-30 minutes

**Requirements:**
1. Proper HTML5 boilerplate
2. Content inside a `<div>` container:
   - Your name (h1)
   - Your role/title (h2)
   - Short bio (2-3 sentences)
   - Location with `<br>` for formatting
   - A motivational quote using `<q>`
3. Use `<strong>` for name in bio
4. Use `<em>` for skills
5. `<hr>` before footer with `<small>` text

---

### 🟡 Intermediate Project: HTML Resume

**Goal:** Create a single-page HTML resume/CV.

**File:** `resume.html`

**Time:** 45-60 minutes

**Requirements:**
1. Complete HTML5 structure with meta description
2. Sections with proper heading hierarchy:
   - **Header**: Name (h1), Contact, Title
   - **Summary**: 2-3 sentence professional summary
   - **Experience**: 2+ job entries with company, position, dates, description
   - **Education**: School, degree, dates
   - **Skills**: Your technical skills
   - **Certifications** (optional)
3. Semantic elements:
   - `<strong>` for company/school names
   - `<em>` for job titles
   - `<mark>` for key achievements
   - `<small>` for dates
   - `<abbr>` for abbreviations
4. `<div>` to group sections
5. `<hr>` between sections
6. Professional HTML comments

---

### 🔴 Advanced Project: Technical Blog Post

**Goal:** Create a complete blog post about "Getting Started with HTML5"

**File:** `blog-post.html`

**Time:** 90-120 minutes

**Requirements:**
1. Professional document structure:
   - Descriptive title
   - Meta description for SEO
   - Meta author tag
2. Article structure:
   - Header: Blog title, author, publish date
   - Featured excerpt (highlighted with `<mark>`)
   - Table of contents
   - Introduction
   - 5+ sections covering HTML topics
   - Code examples using `<code>`
   - Tips/Notes using `<div>` with classes
   - Conclusion
   - Author bio
   - Footer with copyright
3. Use ALL Day 1 elements appropriately
4. Clean, properly indented code
5. Comprehensive comments

**Bonus Challenges:**
- "Last updated" vs "Originally published" using `<del>`/`<ins>`
- Keyboard shortcuts section with multiple `<kbd>` tags
- Mathematical formulas using `<sub>`/`<sup>`

---

## Submission Checklist

Before submitting, verify:

- [ ] Document passes [W3C Validator](https://validator.w3.org/)
- [ ] `<!DOCTYPE html>` is present
- [ ] `<html lang="en">` has language attribute
- [ ] `<head>` contains charset, viewport, title, description
- [ ] All tags are properly closed
- [ ] All tags are properly nested
- [ ] Heading hierarchy is correct (no skipped levels)
- [ ] Only ONE `<h1>` per page
- [ ] Semantic elements used appropriately
- [ ] Code is properly indented (2 or 4 spaces)
- [ ] Meaningful comments are included

---

## Quick Reference

### Document Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Page description">
    <title>Page Title</title>
</head>
<body>
    <!-- Content here -->
</body>
</html>
```

### Element Reference Table

| Element | Type | Purpose | Example |
|---------|------|---------|---------|
| `<h1>`-`<h6>` | Block | Headings | `<h1>Title</h1>` |
| `<p>` | Block | Paragraph | `<p>Text here</p>` |
| `<div>` | Block | Container | `<div class="box">...</div>` |
| `<span>` | Inline | Inline container | `<span class="red">text</span>` |
| `<strong>` | Inline | Important | `<strong>important</strong>` |
| `<em>` | Inline | Emphasis | `<em>emphasized</em>` |
| `<mark>` | Inline | Highlight | `<mark>highlighted</mark>` |
| `<code>` | Inline | Code | `<code>const x = 1</code>` |
| `<kbd>` | Inline | Keyboard | `<kbd>Ctrl</kbd>` |
| `<abbr>` | Inline | Abbreviation | `<abbr title="...">HTML</abbr>` |
| `<q>` | Inline | Quote | `<q>quoted text</q>` |
| `<del>` | Inline | Deleted | `<del>removed</del>` |
| `<ins>` | Inline | Inserted | `<ins>added</ins>` |
| `<sub>` | Inline | Subscript | `H<sub>2</sub>O` |
| `<sup>` | Inline | Superscript | `x<sup>2</sup>` |
| `<small>` | Inline | Small print | `<small>fine print</small>` |
| `<br>` | Inline | Line break | `Line 1<br>Line 2` |
| `<hr>` | Block | Thematic break | `<hr>` |

### Common HTML Entities

| Entity | Symbol | Entity | Symbol |
|--------|--------|--------|--------|
| `&lt;` | < | `&copy;` | © |
| `&gt;` | > | `&reg;` | ® |
| `&amp;` | & | `&trade;` | ™ |
| `&nbsp;` | (space) | `&mdash;` | — |
| `&quot;` | " | `&hellip;` | … |

---

## Self-Check Questions

Test your understanding:

1. What is the purpose of `<!DOCTYPE html>`?
2. What's the difference between `<strong>` and `<b>`?
3. Should you have multiple `<h1>` elements on a page? Why?
4. What's the difference between block and inline elements?
5. Where do you put the page title - `<head>` or `<body>`?
6. How do you display `<div>` as text on a webpage?
7. What element would you use for a keyboard shortcut?
8. When should you use `<br>` vs separate `<p>` elements?

<details>
<summary><strong>Click to see answers</strong></summary>

1. Tells the browser to use HTML5 standards mode
2. `<strong>` has semantic meaning (important), `<b>` is just visual
3. No - only one h1 per page for SEO and accessibility
4. Block: starts new line, full width. Inline: flows with text
5. `<head>` - inside `<title>` element
6. Use HTML entities: `&lt;div&gt;`
7. `<kbd>` element
8. `<br>` for line breaks within content (addresses), `<p>` for separate paragraphs

</details>

---

## Further Learning

### Additional Topics to Explore

#### Preformatted Text

```html
<pre>
    This text preserves
        all whitespace
    and line breaks exactly
        as written.
</pre>

<pre><code>
function greet(name) {
    console.log("Hello, " + name);
}
</code></pre>
```

#### Block Quotations

```html
<blockquote cite="https://example.com">
    <p>The Web does not just connect machines, it connects people.</p>
    <footer>— <cite>Tim Berners-Lee</cite></footer>
</blockquote>
```

#### Address Element

```html
<address>
    Contact: <strong>John Doe</strong><br>
    123 Main Street<br>
    New York, NY 10001
</address>
```

#### Time Element

```html
<p>Published on <time datetime="2024-12-01">December 1, 2024</time></p>
<p>Meeting at <time datetime="14:30">2:30 PM</time></p>
```

### Recommended Resources

**Official Documentation:**
- [MDN Web Docs - HTML](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [W3C HTML Specification](https://html.spec.whatwg.org/)
- [W3C Validator](https://validator.w3.org/)

**Interactive Learning:**
- [freeCodeCamp HTML Course](https://www.freecodecamp.org/learn)
- [Codecademy HTML](https://www.codecademy.com/learn/learn-html)
- [W3Schools HTML Tutorial](https://www.w3schools.com/html/)

**Practice Platforms:**
- [Frontend Mentor](https://www.frontendmentor.io/)
- [CodePen](https://codepen.io/)

**Video Resources:**
- [Traversy Media - HTML Crash Course](https://www.youtube.com/watch?v=UB1O30fR-EE)
- [Kevin Powell](https://www.youtube.com/kevinpowell)
- [Web Dev Simplified](https://www.youtube.com/webdevsimplified)

---

## Key Takeaways

1. **Every HTML document needs:** DOCTYPE, html, head, and body
2. **Use semantic elements:** Choose based on meaning, not appearance
3. **Follow heading hierarchy:** h1 → h2 → h3, never skip levels
4. **One h1 per page:** For SEO and accessibility
5. **Block vs inline:** Understand how elements display
6. **Always close tags:** Except self-closing elements
7. **Validate your code:** Use W3C Validator
8. **Comment your code:** Help yourself and others

---

**Next Lesson:** [Day 2 - Links, Images, and Lists](./day-02-links-images-lists.md)
