# Week 3, Day 3: CSS Architecture & BEM

## Why CSS Architecture Matters

As projects grow, CSS can become:
- **Unmaintainable** - Changes break unexpected things
- **Bloated** - Dead code accumulates
- **Inconsistent** - Multiple ways to do the same thing
- **Hard to understand** - No clear organization

Good CSS architecture solves these problems.

---

## BEM Naming Convention

BEM (Block Element Modifier) is the most popular CSS naming convention. It creates meaningful, reusable class names.

### The Three Parts

```
.block__element--modifier
```

- **Block**: Standalone component (`.card`, `.nav`, `.form`)
- **Element**: Part of a block (`.card__title`, `.nav__link`)
- **Modifier**: Variation or state (`.card--featured`, `.nav__link--active`)

### Block

A standalone, reusable component:

```css
.card { }
.nav { }
.button { }
.form { }
.hero { }
.sidebar { }
```

### Element

A part of a block that has no meaning on its own:

```css
.card__image { }
.card__title { }
.card__content { }
.card__footer { }

.nav__list { }
.nav__item { }
.nav__link { }

.form__group { }
.form__label { }
.form__input { }
```

### Modifier

A variation or state of a block or element:

```css
/* Block modifiers */
.card--featured { }
.card--horizontal { }
.button--primary { }
.button--large { }

/* Element modifiers */
.nav__link--active { }
.form__input--error { }
.card__title--large { }
```

---

## BEM in Practice

### Example 1: Card Component

```html
<article class="card">
    <img class="card__image" src="photo.jpg" alt="">
    <div class="card__content">
        <h2 class="card__title">Card Title</h2>
        <p class="card__text">Card description text goes here.</p>
        <a class="card__link" href="#">Read more</a>
    </div>
</article>

<!-- Featured card variation -->
<article class="card card--featured">
    <img class="card__image" src="photo.jpg" alt="">
    <div class="card__content">
        <h2 class="card__title card__title--large">Featured Card</h2>
        <p class="card__text">This card is featured.</p>
        <a class="card__link" href="#">Read more</a>
    </div>
</article>
```

```css
/* Block */
.card {
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Elements */
.card__image {
    width: 100%;
    height: 200px;
    object-fit: cover;
}

.card__content {
    padding: 1.5rem;
}

.card__title {
    font-size: 1.25rem;
    margin-bottom: 0.5rem;
}

.card__text {
    color: #666;
    margin-bottom: 1rem;
}

.card__link {
    color: #3498db;
    text-decoration: none;
}

/* Block Modifier */
.card--featured {
    border: 2px solid #f39c12;
}

/* Element Modifier */
.card__title--large {
    font-size: 1.5rem;
}
```

### Example 2: Navigation

```html
<nav class="nav">
    <a class="nav__logo" href="/">Logo</a>
    <ul class="nav__list">
        <li class="nav__item">
            <a class="nav__link nav__link--active" href="/">Home</a>
        </li>
        <li class="nav__item">
            <a class="nav__link" href="/about">About</a>
        </li>
        <li class="nav__item">
            <a class="nav__link" href="/contact">Contact</a>
        </li>
    </ul>
    <button class="nav__toggle">Menu</button>
</nav>
```

```css
.nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 2rem;
    background: #2c3e50;
}

.nav__logo {
    font-size: 1.5rem;
    font-weight: bold;
    color: white;
    text-decoration: none;
}

.nav__list {
    display: flex;
    list-style: none;
    gap: 2rem;
}

.nav__item {
    /* Styling if needed */
}

.nav__link {
    color: rgba(255,255,255,0.8);
    text-decoration: none;
    transition: color 0.2s;
}

.nav__link:hover {
    color: white;
}

.nav__link--active {
    color: white;
    font-weight: bold;
}

.nav__toggle {
    display: none;
}

@media (max-width: 768px) {
    .nav__list {
        display: none;
    }

    .nav__toggle {
        display: block;
    }
}
```

### Example 3: Form

```html
<form class="form">
    <div class="form__group">
        <label class="form__label" for="email">Email</label>
        <input class="form__input" type="email" id="email" name="email">
    </div>

    <div class="form__group">
        <label class="form__label" for="password">Password</label>
        <input class="form__input form__input--error" type="password" id="password">
        <span class="form__error">Password is required</span>
    </div>

    <div class="form__group">
        <label class="form__checkbox">
            <input type="checkbox" name="remember">
            <span class="form__checkbox-text">Remember me</span>
        </label>
    </div>

    <button class="form__button form__button--primary" type="submit">
        Sign In
    </button>
</form>
```

```css
.form {
    max-width: 400px;
    padding: 2rem;
}

.form__group {
    margin-bottom: 1.5rem;
}

.form__label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 500;
}

.form__input {
    width: 100%;
    padding: 0.75rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
}

.form__input:focus {
    outline: none;
    border-color: #3498db;
}

.form__input--error {
    border-color: #e74c3c;
}

.form__error {
    display: block;
    color: #e74c3c;
    font-size: 0.875rem;
    margin-top: 0.25rem;
}

.form__checkbox {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    cursor: pointer;
}

.form__button {
    width: 100%;
    padding: 0.75rem;
    border: none;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
}

.form__button--primary {
    background: #3498db;
    color: white;
}

.form__button--primary:hover {
    background: #2980b9;
}
```

---

## BEM Rules and Guidelines

### 1. Elements Don't Nest in Class Names

```css
/* Wrong - element of element */
.card__content__title { }

/* Correct - elements are flat */
.card__title { }
.card__content { }
```

The HTML can nest, but class names stay flat:

```html
<div class="card">
    <div class="card__content">
        <h2 class="card__title">Title</h2>  <!-- Not card__content__title -->
    </div>
</div>
```

### 2. Modifiers Need the Base Class

```html
<!-- Wrong -->
<button class="button--primary">Click</button>

<!-- Correct -->
<button class="button button--primary">Click</button>
```

```css
.button {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
}

.button--primary {
    background: blue;
    color: white;
}

.button--secondary {
    background: gray;
    color: white;
}
```

### 3. Use Meaningful Names

```css
/* Bad */
.card__div { }
.card__span { }
.card__a { }

/* Good */
.card__header { }
.card__title { }
.card__link { }
```

### 4. When to Create a New Block

If a component:
- Can be used independently
- Appears in multiple contexts
- Has its own distinct purpose

Then make it a separate block:

```html
<!-- Button is its own block, can be used anywhere -->
<div class="card">
    <h2 class="card__title">Title</h2>
    <button class="button button--primary">Action</button>
</div>

<div class="form">
    <button class="button button--primary">Submit</button>
</div>
```

---

## CSS Custom Properties (Variables)

Use CSS variables for consistent, maintainable styles:

```css
:root {
    /* Colors */
    --color-primary: #3498db;
    --color-primary-dark: #2980b9;
    --color-secondary: #2ecc71;
    --color-danger: #e74c3c;
    --color-warning: #f39c12;

    --color-text: #333;
    --color-text-light: #666;
    --color-text-muted: #999;

    --color-background: #fff;
    --color-background-alt: #f5f5f5;

    --color-border: #ddd;

    /* Typography */
    --font-family: system-ui, -apple-system, sans-serif;
    --font-family-mono: 'Fira Code', monospace;

    --font-size-xs: 0.75rem;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.125rem;
    --font-size-xl: 1.25rem;
    --font-size-2xl: 1.5rem;
    --font-size-3xl: 2rem;

    --line-height: 1.6;

    /* Spacing */
    --spacing-xs: 0.25rem;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 1.5rem;
    --spacing-xl: 2rem;
    --spacing-2xl: 3rem;

    /* Border radius */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;
    --radius-full: 9999px;

    /* Shadows */
    --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
    --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
    --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);

    /* Transitions */
    --transition-fast: 150ms ease;
    --transition-base: 200ms ease;
    --transition-slow: 300ms ease;

    /* Breakpoints (for documentation - can't use in media queries) */
    --breakpoint-sm: 640px;
    --breakpoint-md: 768px;
    --breakpoint-lg: 1024px;
    --breakpoint-xl: 1280px;
}
```

### Using Variables

```css
.button {
    padding: var(--spacing-sm) var(--spacing-md);
    font-size: var(--font-size-base);
    border-radius: var(--radius-md);
    transition: all var(--transition-base);
}

.button--primary {
    background: var(--color-primary);
    color: white;
}

.button--primary:hover {
    background: var(--color-primary-dark);
}

.card {
    background: var(--color-background);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-md);
    box-shadow: var(--shadow-md);
    padding: var(--spacing-lg);
}

.card__title {
    font-size: var(--font-size-xl);
    color: var(--color-text);
    margin-bottom: var(--spacing-sm);
}

.card__text {
    color: var(--color-text-light);
    line-height: var(--line-height);
}
```

### Dark Mode with Variables

```css
:root {
    --color-text: #333;
    --color-background: #fff;
}

@media (prefers-color-scheme: dark) {
    :root {
        --color-text: #eee;
        --color-background: #1a1a1a;
    }
}

/* Or with a class toggle */
.dark-mode {
    --color-text: #eee;
    --color-background: #1a1a1a;
}
```

---

## Organizing CSS Files

### Simple Project Structure

```
styles/
├── main.css          # Imports all other files
├── base/
│   ├── reset.css     # CSS reset/normalize
│   ├── typography.css
│   └── variables.css
├── components/
│   ├── button.css
│   ├── card.css
│   ├── form.css
│   └── nav.css
├── layout/
│   ├── header.css
│   ├── footer.css
│   ├── sidebar.css
│   └── grid.css
└── pages/
    ├── home.css
    └── about.css
```

### main.css

```css
/* Base */
@import 'base/reset.css';
@import 'base/variables.css';
@import 'base/typography.css';

/* Layout */
@import 'layout/grid.css';
@import 'layout/header.css';
@import 'layout/footer.css';
@import 'layout/sidebar.css';

/* Components */
@import 'components/button.css';
@import 'components/card.css';
@import 'components/form.css';
@import 'components/nav.css';

/* Pages */
@import 'pages/home.css';
@import 'pages/about.css';
```

---

## CSS Reset vs Normalize

### Simple Reset

```css
/* Reset box-sizing */
*, *::before, *::after {
    box-sizing: border-box;
}

/* Reset margins */
* {
    margin: 0;
    padding: 0;
}

/* Better line height */
html {
    line-height: 1.5;
    -webkit-text-size-adjust: 100%;
}

/* Remove list styles */
ul, ol {
    list-style: none;
}

/* Make images responsive */
img, picture, video, canvas, svg {
    display: block;
    max-width: 100%;
}

/* Inherit fonts for form elements */
input, button, textarea, select {
    font: inherit;
}

/* Remove default button styles */
button {
    cursor: pointer;
    background: none;
    border: none;
}

/* Avoid text overflows */
p, h1, h2, h3, h4, h5, h6 {
    overflow-wrap: break-word;
}
```

---

## Utility Classes (Optional Approach)

Sometimes you need quick, reusable utilities:

```css
/* Display */
.hidden { display: none; }
.block { display: block; }
.flex { display: flex; }
.grid { display: grid; }

/* Flexbox */
.flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
}

.flex-between {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

/* Text alignment */
.text-left { text-align: left; }
.text-center { text-align: center; }
.text-right { text-align: right; }

/* Font weight */
.font-normal { font-weight: 400; }
.font-medium { font-weight: 500; }
.font-bold { font-weight: 700; }

/* Margin utilities */
.mt-1 { margin-top: var(--spacing-sm); }
.mt-2 { margin-top: var(--spacing-md); }
.mt-3 { margin-top: var(--spacing-lg); }
.mb-1 { margin-bottom: var(--spacing-sm); }
.mb-2 { margin-bottom: var(--spacing-md); }
.mb-3 { margin-bottom: var(--spacing-lg); }

/* Visibility */
.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    border: 0;
}
```

**Note:** For extensive utility classes, consider using Tailwind CSS.

---

## Best Practices Summary

1. **Use BEM naming** - Consistent, meaningful class names
2. **Use CSS variables** - For colors, spacing, typography
3. **Keep specificity low** - Avoid IDs and deep nesting
4. **One component per file** - Easier to maintain
5. **Mobile-first** - Base styles for mobile
6. **Don't repeat yourself** - Use variables and modifiers
7. **Comment your code** - Explain complex sections

---

## Key Takeaways

1. **BEM = Block__Element--Modifier**
2. **Keep elements flat** - No .block__element__subelement
3. **Modifiers need base class** - `.button .button--primary`
4. **Use CSS variables** for consistency
5. **Organize by component** not by property type
6. **Start with a reset** for consistent cross-browser styles

---

**Next Lesson:** [Day 4-5 - Modern CSS Features](./week-03-day-04-05-modern-css.md)
