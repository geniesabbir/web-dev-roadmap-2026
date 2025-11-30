# Week 2, Day 1: CSS Basics

## What is CSS?

CSS (Cascading Style Sheets) controls how HTML elements look on screen. While HTML provides structure, CSS provides style - colors, fonts, spacing, layout, and more.

---

## How to Add CSS

There are three ways to add CSS to your HTML:

### 1. Inline CSS (Avoid)

Added directly to elements using the `style` attribute:

```html
<p style="color: red; font-size: 18px;">This is red text.</p>
```

**Problems:**
- Can't reuse styles
- Mixes content with presentation
- Hard to maintain

### 2. Internal CSS (Okay for small projects)

Added in the `<head>` using a `<style>` tag:

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        p {
            color: red;
            font-size: 18px;
        }
    </style>
</head>
<body>
    <p>This is red text.</p>
</body>
</html>
```

### 3. External CSS (Recommended)

Styles in a separate `.css` file linked to HTML:

**styles.css:**
```css
p {
    color: red;
    font-size: 18px;
}
```

**index.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <p>This is red text.</p>
</body>
</html>
```

**Benefits:**
- Reusable across multiple pages
- Separation of concerns
- Browser caching improves performance
- Easier to maintain

---

## CSS Syntax

```css
selector {
    property: value;
    property: value;
}
```

**Example:**
```css
h1 {
    color: blue;
    font-size: 24px;
    text-align: center;
}
```

- **Selector**: What elements to style (`h1`)
- **Property**: What aspect to change (`color`)
- **Value**: The new value (`blue`)
- **Declaration**: Property + value pair
- **Declaration Block**: Everything inside `{ }`

---

## CSS Selectors

### Element Selector

Selects all elements of that type:

```css
p {
    color: gray;
}

h1 {
    font-size: 32px;
}

/* Multiple elements */
h1, h2, h3 {
    font-family: Arial, sans-serif;
}
```

### Class Selector

Selects elements with a specific class (prefix with `.`):

```html
<p class="highlight">Highlighted text</p>
<p>Normal text</p>
<p class="highlight">Also highlighted</p>
```

```css
.highlight {
    background-color: yellow;
}
```

**Classes can be reused** on multiple elements.

### ID Selector

Selects ONE element with a specific ID (prefix with `#`):

```html
<div id="header">This is the header</div>
```

```css
#header {
    background-color: navy;
    color: white;
}
```

**IDs must be unique** - only one element per page.

### Universal Selector

Selects ALL elements:

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}
```

### Attribute Selector

Selects elements based on attributes:

```css
/* Elements with a title attribute */
[title] {
    cursor: help;
}

/* Elements with specific attribute value */
[type="text"] {
    border: 1px solid gray;
}

/* Attribute contains value */
[class*="btn"] {
    cursor: pointer;
}

/* Attribute starts with value */
[href^="https"] {
    color: green;
}

/* Attribute ends with value */
[href$=".pdf"] {
    color: red;
}
```

---

## The Box Model

Every element is a rectangular box with four parts:

```
┌──────────────────────────────────────┐
│              MARGIN                  │
│  ┌────────────────────────────────┐  │
│  │           BORDER               │  │
│  │  ┌──────────────────────────┐  │  │
│  │  │        PADDING           │  │  │
│  │  │  ┌────────────────────┐  │  │  │
│  │  │  │     CONTENT        │  │  │  │
│  │  │  │                    │  │  │  │
│  │  │  └────────────────────┘  │  │  │
│  │  │                          │  │  │
│  │  └──────────────────────────┘  │  │
│  │                                │  │
│  └────────────────────────────────┘  │
│                                      │
└──────────────────────────────────────┘
```

### Content

The actual content (text, images, etc.):

```css
.box {
    width: 300px;
    height: 200px;
}
```

### Padding

Space between content and border:

```css
.box {
    /* All sides */
    padding: 20px;

    /* Top/Bottom | Left/Right */
    padding: 10px 20px;

    /* Top | Right | Bottom | Left (clockwise) */
    padding: 10px 20px 30px 40px;

    /* Individual sides */
    padding-top: 10px;
    padding-right: 20px;
    padding-bottom: 30px;
    padding-left: 40px;
}
```

### Border

The edge around padding:

```css
.box {
    /* Shorthand: width style color */
    border: 2px solid black;

    /* Individual properties */
    border-width: 2px;
    border-style: solid; /* solid, dashed, dotted, double, none */
    border-color: black;

    /* Individual sides */
    border-top: 2px solid red;
    border-right: 3px dashed blue;

    /* Rounded corners */
    border-radius: 10px;
    border-radius: 50%; /* Circle */
}
```

### Margin

Space outside the border:

```css
.box {
    /* Same syntax as padding */
    margin: 20px;
    margin: 10px 20px;
    margin: 10px 20px 30px 40px;

    /* Individual sides */
    margin-top: 10px;

    /* Auto for centering */
    margin: 0 auto; /* Center horizontally */
}
```

### Box Sizing

By default, `width` and `height` only apply to content:

```css
/* Default behavior */
.box {
    width: 300px;
    padding: 20px;
    border: 5px solid black;
    /* Actual width: 300 + 20*2 + 5*2 = 350px */
}

/* Better behavior - include padding and border */
.box {
    box-sizing: border-box;
    width: 300px;
    padding: 20px;
    border: 5px solid black;
    /* Actual width: 300px */
}

/* Apply to all elements (recommended) */
*, *::before, *::after {
    box-sizing: border-box;
}
```

---

## Display Property

Controls how elements are displayed:

### Block

Takes full width, starts on new line:

```css
div, p, h1, ul, li, section, article {
    display: block;
}

.full-width {
    display: block;
    width: 100%;
}
```

### Inline

Takes only needed width, doesn't break line:

```css
span, a, strong, em {
    display: inline;
}

/* Note: width and height have no effect on inline elements */
```

### Inline-Block

Inline but respects width/height:

```css
.button {
    display: inline-block;
    width: 150px;
    padding: 10px 20px;
}
```

### None

Hides element completely (removes from document flow):

```css
.hidden {
    display: none;
}
```

### Comparison

| Property | Width | Height | Margin Top/Bottom | Starts New Line |
|----------|-------|--------|-------------------|-----------------|
| block | Yes | Yes | Yes | Yes |
| inline | No | No | No | No |
| inline-block | Yes | Yes | Yes | No |

---

## Colors

### Named Colors

```css
.text {
    color: red;
    color: blue;
    color: coral;
    color: rebeccapurple;
}
```

There are 140+ named colors. Common ones:
- `black`, `white`, `gray`
- `red`, `green`, `blue`
- `orange`, `yellow`, `purple`
- `pink`, `cyan`, `magenta`

### Hexadecimal

```css
.text {
    /* #RRGGBB - Red, Green, Blue (00-FF each) */
    color: #ff0000;    /* Red */
    color: #00ff00;    /* Green */
    color: #0000ff;    /* Blue */
    color: #000000;    /* Black */
    color: #ffffff;    /* White */
    color: #808080;    /* Gray */

    /* Shorthand #RGB */
    color: #f00;       /* Same as #ff0000 */
    color: #fff;       /* Same as #ffffff */

    /* With alpha (transparency) #RRGGBBAA */
    color: #ff000080;  /* 50% transparent red */
}
```

### RGB and RGBA

```css
.text {
    /* rgb(red, green, blue) - 0-255 each */
    color: rgb(255, 0, 0);      /* Red */
    color: rgb(0, 128, 0);      /* Green */
    color: rgb(0, 0, 255);      /* Blue */

    /* rgba(red, green, blue, alpha) - alpha 0-1 */
    color: rgba(255, 0, 0, 0.5);    /* 50% transparent red */
    color: rgba(0, 0, 0, 0.8);      /* 80% opaque black */
}
```

### HSL and HSLA

```css
.text {
    /* hsl(hue, saturation, lightness) */
    /* hue: 0-360 (color wheel degrees) */
    /* saturation: 0%-100% (gray to full color) */
    /* lightness: 0%-100% (black to white) */

    color: hsl(0, 100%, 50%);      /* Pure red */
    color: hsl(120, 100%, 50%);    /* Pure green */
    color: hsl(240, 100%, 50%);    /* Pure blue */

    /* hsla with alpha */
    color: hsla(0, 100%, 50%, 0.5);  /* 50% transparent red */
}
```

**Color Wheel (Hue values):**
- 0° / 360° = Red
- 60° = Yellow
- 120° = Green
- 180° = Cyan
- 240° = Blue
- 300° = Magenta

---

## Units

### Absolute Units

```css
.box {
    /* Pixels - most common */
    width: 300px;
    font-size: 16px;

    /* Other absolute units (rarely used) */
    width: 2in;   /* inches */
    width: 5cm;   /* centimeters */
    width: 12pt;  /* points */
}
```

### Relative Units

```css
.box {
    /* em - relative to parent font-size */
    font-size: 1.5em;   /* 1.5 times parent font-size */
    padding: 1em;       /* equals current font-size */

    /* rem - relative to root (html) font-size */
    font-size: 1.5rem;  /* 1.5 times root font-size */
    margin: 2rem;

    /* Percentage - relative to parent */
    width: 50%;         /* 50% of parent width */
    font-size: 120%;    /* 120% of parent font-size */

    /* Viewport units */
    width: 100vw;       /* 100% of viewport width */
    height: 100vh;      /* 100% of viewport height */
    width: 50vmin;      /* 50% of smaller viewport dimension */
    width: 50vmax;      /* 50% of larger viewport dimension */
}
```

### When to Use Each

| Unit | Best For |
|------|----------|
| `px` | Borders, shadows, small fixed sizes |
| `rem` | Font sizes, spacing (consistent scaling) |
| `em` | Padding/margin relative to text size |
| `%` | Widths, responsive layouts |
| `vw/vh` | Full-screen sections, responsive typography |

**Recommended approach:**

```css
/* Root font size */
html {
    font-size: 16px;  /* Base size */
}

/* Typography with rem */
h1 {
    font-size: 2rem;    /* 32px */
}

p {
    font-size: 1rem;    /* 16px */
    line-height: 1.5;   /* Unitless multiplier */
}

/* Spacing with rem */
.section {
    padding: 2rem;
    margin-bottom: 1.5rem;
}

/* Layouts with percentage */
.container {
    width: 100%;
    max-width: 1200px;
}

/* Fixed sizes with px */
.card {
    border: 1px solid #ccc;
    border-radius: 8px;
}
```

---

## CSS Comments

```css
/* This is a single-line comment */

/*
   This is a
   multi-line comment
*/

/* ===================================
   HEADER STYLES
   =================================== */

.header {
    /* Background color for the header */
    background-color: #333;
}
```

---

## Practice Exercise

Create a simple styled page:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Basics Practice</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1 class="title">My Styled Page</h1>

        <div class="card">
            <h2>Card Title</h2>
            <p>This is a card with some content. Practice using padding, margin, and borders.</p>
            <a href="#" class="button">Learn More</a>
        </div>

        <div class="card highlight">
            <h2>Highlighted Card</h2>
            <p>This card has a different style using a modifier class.</p>
            <a href="#" class="button">Learn More</a>
        </div>
    </div>
</body>
</html>
```

**style.css:**
```css
/* Reset and base styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
    color: #333;
    background-color: #f4f4f4;
}

/* Container */
.container {
    width: 90%;
    max-width: 800px;
    margin: 0 auto;
    padding: 2rem;
}

/* Title */
.title {
    font-size: 2.5rem;
    color: #2c3e50;
    text-align: center;
    margin-bottom: 2rem;
}

/* Card */
.card {
    background-color: white;
    padding: 1.5rem;
    margin-bottom: 1.5rem;
    border-radius: 8px;
    border: 1px solid #ddd;
}

.card h2 {
    color: #2c3e50;
    margin-bottom: 1rem;
}

.card p {
    color: #666;
    margin-bottom: 1rem;
}

/* Highlight modifier */
.card.highlight {
    border-left: 4px solid #3498db;
    background-color: #f8f9fa;
}

/* Button */
.button {
    display: inline-block;
    padding: 0.5rem 1rem;
    background-color: #3498db;
    color: white;
    text-decoration: none;
    border-radius: 4px;
}
```

---

## Key Takeaways

1. **Use external CSS** - Keep styles in separate files
2. **Understand the box model** - Content, padding, border, margin
3. **Use `box-sizing: border-box`** - Makes sizing predictable
4. **Know your units** - `rem` for fonts, `%` for widths, `px` for borders
5. **Class selectors are your friend** - Reusable and specific

---

## Self-Check Questions

1. What are the three ways to add CSS to HTML?
2. What's the difference between class and ID selectors?
3. What are the four parts of the box model?
4. What does `box-sizing: border-box` do?
5. When would you use `rem` vs `px`?

---

**Next Lesson:** [Day 2 - Selectors & Specificity](./week-02-day-02-selectors-specificity.md)
