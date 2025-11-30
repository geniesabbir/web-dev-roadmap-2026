# Week 2, Day 2: CSS Selectors & Specificity

## Combinator Selectors

Combinators define the relationship between selectors.

### Descendant Combinator (Space)

Selects elements **nested inside** another element (at any depth):

```css
/* All <p> inside <article> */
article p {
    color: gray;
}

/* All <a> inside <nav> inside <header> */
header nav a {
    color: white;
}
```

```html
<article>
    <p>This is styled</p>
    <div>
        <p>This is also styled (nested deeper)</p>
    </div>
</article>
<p>This is NOT styled (outside article)</p>
```

### Child Combinator (>)

Selects elements that are **direct children** only:

```css
/* Only <li> that are direct children of <ul> */
ul > li {
    list-style: square;
}

/* Only <p> directly inside <article> */
article > p {
    font-size: 1.1rem;
}
```

```html
<ul>
    <li>Styled (direct child)</li>
    <li>Styled (direct child)
        <ul>
            <li>NOT styled (grandchild)</li>
        </ul>
    </li>
</ul>
```

### Adjacent Sibling Combinator (+)

Selects element **immediately after** another:

```css
/* <p> immediately following <h2> */
h2 + p {
    font-size: 1.2rem;
    color: gray;
}
```

```html
<h2>Title</h2>
<p>This paragraph is styled (immediately after h2)</p>
<p>This is NOT styled</p>
```

**Common use case - spacing between elements:**
```css
/* Add margin to paragraphs that follow other paragraphs */
p + p {
    margin-top: 1rem;
}
```

### General Sibling Combinator (~)

Selects **all siblings** that come after:

```css
/* All <p> that come after <h2> (siblings only) */
h2 ~ p {
    color: gray;
}
```

```html
<h2>Title</h2>
<p>Styled</p>
<div>Not a paragraph</div>
<p>Also styled</p>
<p>Also styled</p>
```

### Combinator Summary

| Combinator | Example | Meaning |
|------------|---------|---------|
| (space) | `div p` | All `<p>` inside `<div>` |
| `>` | `div > p` | `<p>` directly inside `<div>` |
| `+` | `h2 + p` | `<p>` immediately after `<h2>` |
| `~` | `h2 ~ p` | All `<p>` after `<h2>` (siblings) |

---

## Pseudo-Classes

Pseudo-classes select elements based on **state** or **position**.

### Interactive States

```css
/* Unvisited link */
a:link {
    color: blue;
}

/* Visited link */
a:visited {
    color: purple;
}

/* Mouse over element */
a:hover {
    color: red;
    text-decoration: underline;
}

/* Currently focused (keyboard navigation) */
a:focus {
    outline: 2px solid orange;
}

/* Being clicked */
a:active {
    color: darkred;
}
```

**LVHA Order:** Link, Visited, Hover, Active (remember: "LoVe HAte")

### Form States

```css
/* Focused input */
input:focus {
    border-color: blue;
    outline: none;
}

/* Disabled elements */
input:disabled {
    background-color: #eee;
    cursor: not-allowed;
}

/* Enabled elements */
input:enabled {
    background-color: white;
}

/* Checked checkbox/radio */
input:checked {
    accent-color: green;
}

/* Required fields */
input:required {
    border-left: 3px solid red;
}

/* Optional fields */
input:optional {
    border-left: 3px solid gray;
}

/* Valid input */
input:valid {
    border-color: green;
}

/* Invalid input */
input:invalid {
    border-color: red;
}

/* Placeholder shown (empty) */
input:placeholder-shown {
    font-style: italic;
}
```

### Structural Pseudo-Classes

```css
/* First child of its parent */
li:first-child {
    font-weight: bold;
}

/* Last child of its parent */
li:last-child {
    border-bottom: none;
}

/* Elements with no children */
div:empty {
    display: none;
}

/* Only child of parent */
p:only-child {
    text-align: center;
}

/* First of its type in parent */
p:first-of-type {
    font-size: 1.2rem;
}

/* Last of its type in parent */
p:last-of-type {
    margin-bottom: 0;
}

/* Only of its type */
h1:only-of-type {
    text-align: center;
}
```

### nth-child Selectors

```css
/* Second child */
li:nth-child(2) {
    color: red;
}

/* Every odd child (1, 3, 5...) */
tr:nth-child(odd) {
    background-color: #f5f5f5;
}

/* Every even child (2, 4, 6...) */
tr:nth-child(even) {
    background-color: white;
}

/* Every 3rd child (3, 6, 9...) */
li:nth-child(3n) {
    color: blue;
}

/* Every 3rd child, starting at 1 (1, 4, 7...) */
li:nth-child(3n+1) {
    color: green;
}

/* First 3 children */
li:nth-child(-n+3) {
    font-weight: bold;
}

/* From 4th child onwards */
li:nth-child(n+4) {
    opacity: 0.8;
}
```

**Formula: `an+b`**
- `n` starts at 0 and increments
- `a` is the cycle size
- `b` is the offset

| Selector | Selects |
|----------|---------|
| `:nth-child(3)` | 3rd child only |
| `:nth-child(odd)` | 1st, 3rd, 5th... |
| `:nth-child(even)` | 2nd, 4th, 6th... |
| `:nth-child(3n)` | 3rd, 6th, 9th... |
| `:nth-child(3n+1)` | 1st, 4th, 7th... |
| `:nth-child(-n+3)` | 1st, 2nd, 3rd |
| `:nth-child(n+4)` | 4th and beyond |

### nth-last-child (Count from end)

```css
/* Second to last */
li:nth-last-child(2) {
    color: orange;
}

/* Last 3 items */
li:nth-last-child(-n+3) {
    font-style: italic;
}
```

### Negation Pseudo-Class

```css
/* All paragraphs except those with class "intro" */
p:not(.intro) {
    color: gray;
}

/* All inputs except submit buttons */
input:not([type="submit"]) {
    border: 1px solid #ccc;
}

/* All links except those in nav */
a:not(nav a) {
    color: blue;
}
```

### The :is() and :where() Pseudo-Classes

```css
/* Without :is() - repetitive */
article h1,
article h2,
article h3 {
    color: navy;
}

/* With :is() - cleaner */
article :is(h1, h2, h3) {
    color: navy;
}

/* :where() - same but zero specificity */
article :where(h1, h2, h3) {
    color: navy;
}
```

### The :has() Pseudo-Class (Parent Selector!)

```css
/* Card that contains an image */
.card:has(img) {
    padding-top: 0;
}

/* Form with invalid inputs */
form:has(input:invalid) {
    border: 2px solid red;
}

/* Figure with a figcaption */
figure:has(figcaption) {
    border: 1px solid #ccc;
    padding: 1rem;
}
```

---

## Pseudo-Elements

Pseudo-elements create **virtual elements** for styling.

### ::before and ::after

Insert content before or after an element:

```css
/* Add arrow before links */
a::before {
    content: "→ ";
}

/* Add "(external)" after external links */
a[href^="https://"]::after {
    content: " (external)";
    font-size: 0.8em;
    color: gray;
}

/* Required field asterisk */
label.required::after {
    content: " *";
    color: red;
}

/* Decorative elements */
.quote::before {
    content: """;
    font-size: 2rem;
}

.quote::after {
    content: """;
    font-size: 2rem;
}
```

**`content` property is required!** Even if empty:

```css
.clearfix::after {
    content: "";
    display: block;
    clear: both;
}
```

### ::first-letter and ::first-line

```css
/* Drop cap effect */
p::first-letter {
    font-size: 3rem;
    float: left;
    line-height: 1;
    margin-right: 0.1em;
}

/* Style first line */
p::first-line {
    font-weight: bold;
    color: navy;
}
```

### ::selection

Style selected (highlighted) text:

```css
::selection {
    background-color: yellow;
    color: black;
}

/* Specific to paragraphs */
p::selection {
    background-color: lightblue;
}
```

### ::placeholder

Style placeholder text:

```css
input::placeholder {
    color: #999;
    font-style: italic;
}
```

### ::marker

Style list markers:

```css
li::marker {
    color: red;
    font-weight: bold;
}

ul li::marker {
    content: "✓ ";
    color: green;
}
```

---

## CSS Specificity

When multiple rules target the same element, specificity determines which applies.

### Specificity Hierarchy

From lowest to highest:

1. **Element selectors** (0,0,0,1)
2. **Class selectors, pseudo-classes** (0,0,1,0)
3. **ID selectors** (0,1,0,0)
4. **Inline styles** (1,0,0,0)
5. **!important** (overrides everything)

### Calculating Specificity

Count the number of each type:

```
Format: (inline, IDs, classes, elements)
```

| Selector | Specificity | Calculation |
|----------|-------------|-------------|
| `p` | 0,0,0,1 | 1 element |
| `.intro` | 0,0,1,0 | 1 class |
| `#header` | 0,1,0,0 | 1 ID |
| `p.intro` | 0,0,1,1 | 1 class + 1 element |
| `#header .nav a` | 0,1,1,1 | 1 ID + 1 class + 1 element |
| `#header .nav a:hover` | 0,1,2,1 | 1 ID + 2 classes + 1 element |

### Specificity Examples

```css
/* Specificity: 0,0,0,1 */
p {
    color: black;
}

/* Specificity: 0,0,1,0 - WINS over element */
.text {
    color: blue;
}

/* Specificity: 0,0,1,1 - WINS over class alone */
p.text {
    color: green;
}

/* Specificity: 0,1,0,0 - WINS over everything except inline/!important */
#main {
    color: red;
}
```

### Equal Specificity

When specificity is equal, the **last rule wins**:

```css
.text {
    color: blue;
}

.text {
    color: red;  /* This wins - comes last */
}
```

### The !important Rule

```css
p {
    color: red !important;  /* Overrides everything except inline !important */
}
```

**Avoid !important!** It makes debugging difficult. If you need it, your specificity is probably wrong.

### Dealing with Specificity Issues

**Problem:**
```css
#sidebar .widget p {
    color: gray;
}

/* This won't work - lower specificity */
.special-text {
    color: blue;
}
```

**Solutions:**

1. **Match the specificity:**
```css
#sidebar .widget p.special-text {
    color: blue;
}
```

2. **Use a more specific selector:**
```css
.widget p.special-text {
    color: blue;
}
```

3. **Restructure your CSS** (best solution):
```css
/* Avoid high-specificity selectors in the first place */
.widget-text {
    color: gray;
}

.special-text {
    color: blue;
}
```

---

## The Cascade

CSS applies rules in this order:

1. **Origin and Importance**
   - User agent (browser) styles
   - User styles
   - Author (your) styles
   - Author !important
   - User !important

2. **Specificity** (as described above)

3. **Source Order** (last rule wins if equal specificity)

---

## Inheritance

Some properties are inherited from parent to child:

### Inherited Properties

```css
/* These are inherited */
body {
    font-family: Arial, sans-serif;  /* Inherited */
    font-size: 16px;                 /* Inherited */
    color: #333;                     /* Inherited */
    line-height: 1.5;                /* Inherited */
}

/* Children automatically get these styles */
```

### Non-Inherited Properties

```css
/* These are NOT inherited */
div {
    border: 1px solid black;  /* NOT inherited */
    background: white;         /* NOT inherited */
    padding: 1rem;            /* NOT inherited */
    margin: 1rem;             /* NOT inherited */
}
```

### Force Inheritance

```css
.child {
    /* Inherit from parent */
    border: inherit;

    /* Reset to browser default */
    color: initial;

    /* Remove any styling */
    all: unset;

    /* Reset to inherited value if inheritable, else initial */
    color: revert;
}
```

---

## Best Practices

### 1. Keep Specificity Low

```css
/* Bad - high specificity */
#header #nav ul li a.active {
    color: red;
}

/* Good - low specificity */
.nav-link--active {
    color: red;
}
```

### 2. Use Classes Over IDs for Styling

```css
/* Avoid */
#header { }

/* Prefer */
.header { }
.site-header { }
```

### 3. Avoid !important

If you need it, refactor your CSS instead.

### 4. Use BEM Naming (Preview)

```css
/* Block */
.card { }

/* Element */
.card__title { }
.card__image { }

/* Modifier */
.card--featured { }
.card__title--large { }
```

---

## Practice Exercise

Style this navigation:

```html
<nav class="main-nav">
    <ul class="nav-list">
        <li class="nav-item">
            <a href="/" class="nav-link nav-link--active">Home</a>
        </li>
        <li class="nav-item">
            <a href="/about" class="nav-link">About</a>
        </li>
        <li class="nav-item">
            <a href="/services" class="nav-link">Services</a>
            <ul class="dropdown">
                <li><a href="/services/web">Web Design</a></li>
                <li><a href="/services/mobile">Mobile Apps</a></li>
            </ul>
        </li>
        <li class="nav-item">
            <a href="/contact" class="nav-link">Contact</a>
        </li>
    </ul>
</nav>
```

```css
/* Your challenge: Style this navigation with:
   - Horizontal layout
   - Hover effects
   - Active state styling
   - Dropdown that appears on hover
   Use appropriate selectors and keep specificity low */
```

---

## Key Takeaways

1. **Combinators** define relationships between elements
2. **Pseudo-classes** select based on state or position
3. **Pseudo-elements** create virtual elements for styling
4. **Specificity** determines which rules apply
5. **Keep specificity low** for maintainable CSS
6. **Avoid !important** - fix specificity instead

---

**Next Lesson:** [Day 3-4 - Flexbox](./week-02-day-03-04-flexbox.md)
