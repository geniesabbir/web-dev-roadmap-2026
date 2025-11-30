# Week 2, Day 3-4: CSS Flexbox

## What is Flexbox?

Flexbox (Flexible Box Layout) is a one-dimensional layout system for arranging items in rows OR columns. It makes it easy to:

- Align items horizontally and vertically
- Distribute space between items
- Reorder items without changing HTML
- Create responsive layouts

---

## Flex Container and Flex Items

```html
<div class="container">     <!-- Flex Container -->
    <div class="item">1</div>  <!-- Flex Item -->
    <div class="item">2</div>  <!-- Flex Item -->
    <div class="item">3</div>  <!-- Flex Item -->
</div>
```

```css
.container {
    display: flex;  /* This makes it a flex container */
}
```

When you set `display: flex`:
- The container becomes a **flex container**
- Direct children become **flex items**
- Items flow in a row by default
- Items stretch to fill the container height

---

## Flex Container Properties

### display: flex vs inline-flex

```css
/* Block-level flex container (takes full width) */
.container {
    display: flex;
}

/* Inline-level flex container (width of content) */
.container {
    display: inline-flex;
}
```

---

### flex-direction

Controls the main axis (direction items flow):

```css
.container {
    flex-direction: row;            /* Default: left to right */
    flex-direction: row-reverse;    /* Right to left */
    flex-direction: column;         /* Top to bottom */
    flex-direction: column-reverse; /* Bottom to top */
}
```

**Visual representation:**

```
row (default):          row-reverse:
┌─────────────────┐     ┌─────────────────┐
│ [1] [2] [3]     │     │     [3] [2] [1] │
└─────────────────┘     └─────────────────┘

column:                 column-reverse:
┌─────┐                 ┌─────┐
│ [1] │                 │ [3] │
│ [2] │                 │ [2] │
│ [3] │                 │ [1] │
└─────┘                 └─────┘
```

---

### flex-wrap

Controls whether items wrap to new lines:

```css
.container {
    flex-wrap: nowrap;       /* Default: all items on one line */
    flex-wrap: wrap;         /* Wrap to next line if needed */
    flex-wrap: wrap-reverse; /* Wrap upwards */
}
```

```
nowrap:                 wrap:
┌─────────────────┐     ┌─────────────────┐
│ [1][2][3][4][5] │     │ [1] [2] [3]     │
│ (items shrink)  │     │ [4] [5]         │
└─────────────────┘     └─────────────────┘
```

### flex-flow (Shorthand)

```css
/* flex-flow: <direction> <wrap> */
.container {
    flex-flow: row wrap;
    flex-flow: column nowrap;
}
```

---

### justify-content

Aligns items along the **main axis**:

```css
.container {
    justify-content: flex-start;    /* Default: items at start */
    justify-content: flex-end;      /* Items at end */
    justify-content: center;        /* Items centered */
    justify-content: space-between; /* First/last at edges, space between */
    justify-content: space-around;  /* Equal space around each item */
    justify-content: space-evenly;  /* Equal space between all */
}
```

**Visual representation (row direction):**

```
flex-start:             flex-end:
┌─────────────────┐     ┌─────────────────┐
│ [1][2][3]       │     │       [1][2][3] │
└─────────────────┘     └─────────────────┘

center:                 space-between:
┌─────────────────┐     ┌─────────────────┐
│   [1][2][3]     │     │ [1]  [2]  [3]   │
└─────────────────┘     └─────────────────┘

space-around:           space-evenly:
┌─────────────────┐     ┌─────────────────┐
│  [1]  [2]  [3]  │     │  [1]  [2]  [3]  │
│ (half space on  │     │ (equal space    │
│  outer edges)   │     │  everywhere)    │
└─────────────────┘     └─────────────────┘
```

---

### align-items

Aligns items along the **cross axis** (perpendicular to main):

```css
.container {
    align-items: stretch;     /* Default: stretch to fill container */
    align-items: flex-start;  /* Align to start of cross axis */
    align-items: flex-end;    /* Align to end of cross axis */
    align-items: center;      /* Center on cross axis */
    align-items: baseline;    /* Align by text baseline */
}
```

**Visual representation (row direction, container has height):**

```
stretch (default):      flex-start:
┌─────────────────┐     ┌─────────────────┐
│ ┌──┐┌──┐┌──┐    │     │ [1][2][3]       │
│ │1 ││2 ││3 │    │     │                 │
│ │  ││  ││  │    │     │                 │
│ └──┘└──┘└──┘    │     │                 │
└─────────────────┘     └─────────────────┘

flex-end:               center:
┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │
│                 │     │ [1][2][3]       │
│                 │     │                 │
│ [1][2][3]       │     │                 │
└─────────────────┘     └─────────────────┘

baseline:
┌─────────────────┐
│ [Big1]          │
│   [2] [tiny3]   │  ← Text baselines align
│                 │
└─────────────────┘
```

---

### align-content

Aligns **lines** when there are multiple rows (only works with `flex-wrap: wrap`):

```css
.container {
    flex-wrap: wrap;
    align-content: flex-start;    /* Lines packed to start */
    align-content: flex-end;      /* Lines packed to end */
    align-content: center;        /* Lines centered */
    align-content: space-between; /* Lines spread out */
    align-content: space-around;  /* Space around each line */
    align-content: stretch;       /* Lines stretch to fill */
}
```

---

### gap

Adds space between items:

```css
.container {
    gap: 20px;                /* Same gap for rows and columns */
    gap: 20px 10px;           /* row-gap column-gap */
    row-gap: 20px;            /* Gap between rows */
    column-gap: 10px;         /* Gap between columns */
}
```

**Advantage over margin:** Gap only applies between items, not on outer edges.

---

## Flex Item Properties

### flex-grow

How much an item should grow relative to others:

```css
.item {
    flex-grow: 0;  /* Default: don't grow */
    flex-grow: 1;  /* Grow to fill available space */
    flex-grow: 2;  /* Grow twice as much as flex-grow: 1 */
}
```

```
All items flex-grow: 0 (default):
┌─────────────────────────────┐
│ [1] [2] [3]                 │  ← Extra space unused
└─────────────────────────────┘

All items flex-grow: 1:
┌─────────────────────────────┐
│ [   1   ] [   2   ] [   3   ]│  ← Space distributed equally
└─────────────────────────────┘

Item 2 has flex-grow: 2, others 1:
┌─────────────────────────────┐
│ [ 1 ] [     2     ] [ 3 ]   │  ← Item 2 grows twice as much
└─────────────────────────────┘
```

---

### flex-shrink

How much an item should shrink when space is limited:

```css
.item {
    flex-shrink: 1;  /* Default: can shrink */
    flex-shrink: 0;  /* Don't shrink */
    flex-shrink: 2;  /* Shrink twice as much as others */
}
```

---

### flex-basis

The initial size of an item before growing/shrinking:

```css
.item {
    flex-basis: auto;   /* Default: use width/height */
    flex-basis: 0;      /* Start from zero, then grow */
    flex-basis: 200px;  /* Start at 200px */
    flex-basis: 25%;    /* Start at 25% of container */
}
```

---

### flex (Shorthand)

Combines grow, shrink, and basis:

```css
.item {
    /* flex: grow shrink basis */
    flex: 0 1 auto;     /* Default */
    flex: 1;            /* flex: 1 1 0 - grow equally */
    flex: auto;         /* flex: 1 1 auto */
    flex: none;         /* flex: 0 0 auto - fixed size */
    flex: 1 0 200px;    /* Grow, don't shrink, start at 200px */
}
```

**Common patterns:**

```css
/* Equal width items */
.item {
    flex: 1;
}

/* Fixed width item */
.sidebar {
    flex: 0 0 250px;  /* 250px, no grow, no shrink */
}

/* Flexible content area */
.main {
    flex: 1;  /* Takes remaining space */
}
```

---

### align-self

Override `align-items` for a single item:

```css
.container {
    align-items: center;
}

.special-item {
    align-self: flex-start;  /* This item aligns differently */
}
```

```
┌─────────────────────┐
│ [1]                 │  ← align-self: flex-start
│     [2] [3] [4]     │  ← align-items: center
│                     │
└─────────────────────┘
```

---

### order

Change the visual order of items:

```css
.item {
    order: 0;  /* Default: source order */
}

.item:first-child {
    order: 3;  /* Move to end */
}

.item:last-child {
    order: -1;  /* Move to beginning */
}
```

```
HTML order: [1] [2] [3]

.item-1 { order: 3; }
.item-3 { order: -1; }

Visual order: [3] [2] [1]
```

**Note:** This only changes visual order, not tab order for accessibility.

---

## Common Flexbox Patterns

### 1. Centering Everything

```css
.container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}
```

### 2. Navigation Bar

```css
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
}

.nav-links {
    display: flex;
    gap: 1rem;
}
```

```html
<nav class="navbar">
    <div class="logo">Logo</div>
    <div class="nav-links">
        <a href="#">Home</a>
        <a href="#">About</a>
        <a href="#">Contact</a>
    </div>
</nav>
```

### 3. Card Layout

```css
.card-container {
    display: flex;
    flex-wrap: wrap;
    gap: 1.5rem;
}

.card {
    flex: 1 1 300px;  /* Grow, shrink, min 300px */
    max-width: 400px;
}
```

### 4. Holy Grail Layout

```css
.page {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.header, .footer {
    flex: 0 0 auto;  /* Don't grow or shrink */
}

.main {
    display: flex;
    flex: 1;  /* Take remaining space */
}

.sidebar {
    flex: 0 0 250px;  /* Fixed width sidebar */
}

.content {
    flex: 1;  /* Take remaining space */
}
```

```html
<div class="page">
    <header class="header">Header</header>
    <div class="main">
        <nav class="sidebar">Sidebar</nav>
        <main class="content">Content</main>
    </div>
    <footer class="footer">Footer</footer>
</div>
```

### 5. Sticky Footer

```css
body {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    margin: 0;
}

main {
    flex: 1;  /* Pushes footer to bottom */
}
```

### 6. Equal Height Cards

```css
.card-row {
    display: flex;
    gap: 1rem;
}

.card {
    flex: 1;
    display: flex;
    flex-direction: column;
}

.card-body {
    flex: 1;  /* Makes all card bodies equal height */
}

.card-footer {
    margin-top: auto;  /* Push to bottom */
}
```

### 7. Media Object

```css
.media {
    display: flex;
    gap: 1rem;
}

.media-image {
    flex: 0 0 auto;  /* Fixed size */
}

.media-content {
    flex: 1;  /* Take remaining space */
}
```

```html
<div class="media">
    <img class="media-image" src="avatar.jpg" alt="User">
    <div class="media-content">
        <h3>Username</h3>
        <p>Comment text goes here...</p>
    </div>
</div>
```

---

## Complete Example: Card Grid

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox Card Grid</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            padding: 2rem;
            background: #f5f5f5;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
        }

        .card-grid {
            display: flex;
            flex-wrap: wrap;
            gap: 1.5rem;
        }

        .card {
            flex: 1 1 300px;
            max-width: 400px;
            background: white;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);

            /* Make card a flex container for equal heights */
            display: flex;
            flex-direction: column;
        }

        .card-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
        }

        .card-content {
            padding: 1.5rem;
            flex: 1;  /* Takes available space */
            display: flex;
            flex-direction: column;
        }

        .card-title {
            font-size: 1.25rem;
            margin-bottom: 0.5rem;
        }

        .card-text {
            color: #666;
            line-height: 1.6;
            flex: 1;  /* Pushes button down */
        }

        .card-button {
            margin-top: 1rem;
            padding: 0.75rem 1.5rem;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            align-self: flex-start;
        }

        .card-button:hover {
            background: #2980b9;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="card-grid">
            <article class="card">
                <img class="card-image" src="https://picsum.photos/400/200?1" alt="Card image">
                <div class="card-content">
                    <h2 class="card-title">Card Title One</h2>
                    <p class="card-text">This is a short description.</p>
                    <button class="card-button">Learn More</button>
                </div>
            </article>

            <article class="card">
                <img class="card-image" src="https://picsum.photos/400/200?2" alt="Card image">
                <div class="card-content">
                    <h2 class="card-title">Card Title Two</h2>
                    <p class="card-text">This card has more text content to demonstrate how flexbox handles different content heights. The button will stay at the bottom regardless of content length.</p>
                    <button class="card-button">Learn More</button>
                </div>
            </article>

            <article class="card">
                <img class="card-image" src="https://picsum.photos/400/200?3" alt="Card image">
                <div class="card-content">
                    <h2 class="card-title">Card Title Three</h2>
                    <p class="card-text">Medium length description here.</p>
                    <button class="card-button">Learn More</button>
                </div>
            </article>
        </div>
    </div>
</body>
</html>
```

---

## Flexbox Cheat Sheet

### Container Properties

| Property | Values |
|----------|--------|
| `display` | `flex`, `inline-flex` |
| `flex-direction` | `row`, `row-reverse`, `column`, `column-reverse` |
| `flex-wrap` | `nowrap`, `wrap`, `wrap-reverse` |
| `justify-content` | `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-items` | `stretch`, `flex-start`, `flex-end`, `center`, `baseline` |
| `align-content` | `stretch`, `flex-start`, `flex-end`, `center`, `space-between`, `space-around` |
| `gap` | `<length>` |

### Item Properties

| Property | Values |
|----------|--------|
| `flex-grow` | `<number>` (default: 0) |
| `flex-shrink` | `<number>` (default: 1) |
| `flex-basis` | `auto`, `<length>` |
| `flex` | `<grow> <shrink> <basis>` |
| `align-self` | `auto`, `stretch`, `flex-start`, `flex-end`, `center`, `baseline` |
| `order` | `<integer>` (default: 0) |

---

## Key Takeaways

1. **`display: flex`** makes an element a flex container
2. **Main axis** = direction items flow (controlled by `flex-direction`)
3. **Cross axis** = perpendicular to main axis
4. **`justify-content`** = alignment on main axis
5. **`align-items`** = alignment on cross axis
6. **`flex: 1`** = grow to fill space equally
7. **`gap`** = space between items (better than margins)

---

## Practice Exercises

1. Create a navigation bar with logo on left, links on right
2. Build a card grid that wraps responsively
3. Create a sticky footer layout
4. Build a horizontal form with labels and inputs aligned

---

**Next Lesson:** [Day 5 - CSS Grid](./week-02-day-05-grid.md)
