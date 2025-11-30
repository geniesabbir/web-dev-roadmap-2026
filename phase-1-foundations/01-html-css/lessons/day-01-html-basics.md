# Day 1: HTML Basics - Document Structure & Text Elements

## What is HTML?

HTML (HyperText Markup Language) is the standard language for creating web pages. It describes the structure of a web page using a series of elements that tell the browser how to display content.

Think of HTML as the skeleton of a website - it provides the basic structure that CSS will style and JavaScript will make interactive.

---

## The Anatomy of an HTML Element

```html
<tagname attribute="value">Content goes here</tagname>
```

- **Opening tag**: `<tagname>` - starts the element
- **Attribute**: `attribute="value"` - provides additional information
- **Content**: The text or nested elements inside
- **Closing tag**: `</tagname>` - ends the element

### Self-Closing Elements

Some elements don't have content and don't need a closing tag:

```html
<img src="photo.jpg" alt="Description" />
<br />
<hr />
<input type="text" />
```

---

## HTML Document Structure

Every HTML document follows this basic structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Page</title>
</head>
<body>
    <!-- Your visible content goes here -->
</body>
</html>
```

### Breaking It Down:

#### `<!DOCTYPE html>`
This declaration tells the browser this is an HTML5 document. Always include it as the first line.

```html
<!DOCTYPE html>
```

#### `<html>` Element
The root element that contains all other elements. The `lang` attribute specifies the language.

```html
<html lang="en">
    <!-- Everything goes inside here -->
</html>
```

Common language codes:
- `en` - English
- `es` - Spanish
- `fr` - French
- `de` - German
- `zh` - Chinese
- `bn` - Bengali

#### `<head>` Element
Contains metadata (information about the page) that isn't displayed on the page itself.

```html
<head>
    <!-- Character encoding - always use UTF-8 -->
    <meta charset="UTF-8">

    <!-- Responsive viewport settings -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Page title shown in browser tab -->
    <title>Page Title</title>

    <!-- Description for search engines -->
    <meta name="description" content="A brief description of your page">

    <!-- Link to CSS stylesheet -->
    <link rel="stylesheet" href="styles.css">

    <!-- Favicon (browser tab icon) -->
    <link rel="icon" href="favicon.ico">
</head>
```

#### `<body>` Element
Contains all the visible content of your web page.

```html
<body>
    <h1>Welcome to My Website</h1>
    <p>This is visible content.</p>
</body>
```

---

## Text Elements

### Headings (h1-h6)

Headings create a hierarchy in your content. Use them in order - don't skip levels!

```html
<h1>Main Title (only one per page)</h1>
<h2>Section Heading</h2>
<h3>Subsection Heading</h3>
<h4>Sub-subsection Heading</h4>
<h5>Minor Heading</h5>
<h6>Smallest Heading</h6>
```

**Best Practices:**
- Use only ONE `<h1>` per page (the main title)
- Don't skip heading levels (don't go from h2 to h4)
- Use headings for structure, not for styling (use CSS for that)

**Example of proper heading hierarchy:**
```html
<h1>Cooking Recipes</h1>

<h2>Breakfast Recipes</h2>
<h3>Pancakes</h3>
<h3>Omelettes</h3>

<h2>Lunch Recipes</h2>
<h3>Sandwiches</h3>
<h3>Salads</h3>
```

### Paragraphs

The `<p>` element defines a paragraph of text.

```html
<p>This is a paragraph. It contains a block of text that forms a
distinct section of content. Browsers automatically add space
before and after paragraphs.</p>

<p>This is another paragraph. Each paragraph starts on a new line.</p>
```

### Line Breaks vs Paragraphs

Use `<br>` for line breaks within content (like addresses or poems):

```html
<!-- Good use of <br> -->
<p>
    123 Main Street<br>
    New York, NY 10001<br>
    United States
</p>

<!-- Bad - don't use <br> for spacing between paragraphs -->
<p>First paragraph.</p>
<br>
<br>
<p>Second paragraph.</p>

<!-- Good - just use paragraphs -->
<p>First paragraph.</p>
<p>Second paragraph.</p>
```

### Horizontal Rule

The `<hr>` element creates a thematic break (a horizontal line):

```html
<h2>Chapter 1</h2>
<p>Content of chapter 1...</p>

<hr>

<h2>Chapter 2</h2>
<p>Content of chapter 2...</p>
```

---

## Inline Text Elements

These elements style text within a block element like `<p>`:

### Strong (Bold) and Emphasis (Italic)

```html
<p>This is <strong>very important</strong> text.</p>
<p>This is <em>emphasized</em> text.</p>
```

**Note:** Use `<strong>` and `<em>` instead of `<b>` and `<i>`. The semantic elements convey meaning, not just visual styling.

| Element | Meaning | Visual Default |
|---------|---------|----------------|
| `<strong>` | Strong importance | Bold |
| `<em>` | Stress emphasis | Italic |
| `<b>` | Stylistically different (no importance) | Bold |
| `<i>` | Alternate voice/mood | Italic |

### Other Inline Elements

```html
<!-- Marked/highlighted text -->
<p>Search results: <mark>JavaScript</mark> is a programming language.</p>

<!-- Deleted and inserted text -->
<p>Price: <del>$100</del> <ins>$75</ins></p>

<!-- Subscript and superscript -->
<p>H<sub>2</sub>O is water.</p>
<p>E = mc<sup>2</sup></p>

<!-- Small print -->
<p><small>Copyright 2024. All rights reserved.</small></p>

<!-- Code -->
<p>Use the <code>console.log()</code> function to debug.</p>

<!-- Keyboard input -->
<p>Press <kbd>Ctrl</kbd> + <kbd>C</kbd> to copy.</p>

<!-- Abbreviation -->
<p>The <abbr title="World Wide Web">WWW</abbr> was invented in 1989.</p>

<!-- Quotations -->
<p>She said, <q>Hello, world!</q></p>
```

### Span Element

`<span>` is a generic inline container with no semantic meaning. Use it for styling:

```html
<p>My favorite color is <span class="highlight">blue</span>.</p>
```

---

## Block vs Inline Elements

Understanding this is crucial:

### Block Elements
- Start on a new line
- Take up the full width available
- Examples: `<div>`, `<p>`, `<h1>-<h6>`, `<ul>`, `<li>`

```html
<div>I take up the whole line.</div>
<div>I start on a new line.</div>
```

### Inline Elements
- Don't start on a new line
- Only take up as much width as necessary
- Examples: `<span>`, `<a>`, `<strong>`, `<em>`, `<img>`

```html
<span>I stay</span><span>on the same line.</span>
```

### The Div Element

`<div>` is a generic block container with no semantic meaning:

```html
<div class="card">
    <h2>Card Title</h2>
    <p>Card content goes here.</p>
</div>
```

---

## HTML Comments

Comments are not displayed in the browser:

```html
<!-- This is a comment -->

<!--
    This is a
    multi-line comment
-->

<!-- TODO: Add navigation here -->
```

Use comments to:
- Explain complex code
- Mark sections
- Temporarily hide code
- Leave notes for yourself or others

---

## Practice Exercise: About Me Page

Create a file called `about-me.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>About Me</title>
</head>
<body>
    <h1>About Me</h1>

    <h2>Introduction</h2>
    <p>Hello! My name is <strong>[Your Name]</strong>. I am learning
    to become a <em>full-stack developer</em>.</p>

    <h2>My Background</h2>
    <p>I am from <mark>[Your City]</mark> and I have been interested
    in technology since [year].</p>

    <h2>What I'm Learning</h2>
    <p>Currently, I am studying:</p>
    <!-- We'll learn lists tomorrow! For now, use paragraphs -->
    <p>- HTML</p>
    <p>- CSS</p>
    <p>- JavaScript</p>

    <h2>Fun Facts</h2>
    <p>Here are some things about me:</p>
    <p>1. I love <strong>coding</strong></p>
    <p>2. My favorite language is <code>JavaScript</code></p>
    <p>3. I want to build <em>amazing web applications</em></p>

    <hr>

    <p><small>Page created in 2024</small></p>
</body>
</html>
```

---

## Common Mistakes to Avoid

### 1. Missing DOCTYPE
```html
<!-- Wrong -->
<html>
<head>...</head>
</html>

<!-- Correct -->
<!DOCTYPE html>
<html>
<head>...</head>
</html>
```

### 2. Missing Closing Tags
```html
<!-- Wrong -->
<p>First paragraph
<p>Second paragraph

<!-- Correct -->
<p>First paragraph</p>
<p>Second paragraph</p>
```

### 3. Improper Nesting
```html
<!-- Wrong -->
<p><strong>Bold and <em>italic</strong> text</em></p>

<!-- Correct -->
<p><strong>Bold and <em>italic</em></strong> text</p>
```

### 4. Using Headings for Styling
```html
<!-- Wrong - using h3 just to make text smaller -->
<h1>Title</h1>
<h3>This should be a paragraph but I want smaller text</h3>

<!-- Correct - use CSS for styling -->
<h1>Title</h1>
<p class="subtitle">This is a subtitle styled with CSS</p>
```

---

## Key Takeaways

1. **Every HTML document needs**: DOCTYPE, html, head, and body
2. **Use semantic elements**: Choose elements based on meaning, not appearance
3. **Follow heading hierarchy**: h1 → h2 → h3, never skip levels
4. **Understand block vs inline**: This affects how elements display
5. **Always close your tags**: Except self-closing elements like `<img>` and `<br>`

---

## Self-Check Questions

1. What is the purpose of the `<!DOCTYPE html>` declaration?
2. What's the difference between `<strong>` and `<b>`?
3. Should you have multiple `<h1>` elements on a page?
4. What's the difference between block and inline elements?
5. Where do you put the page title?

---

**Next Lesson:** [Day 2 - Links, Images, and Lists](./day-02-links-images-lists.md)
