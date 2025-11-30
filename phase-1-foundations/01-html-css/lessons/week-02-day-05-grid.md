# Week 2, Day 5: CSS Grid

## What is CSS Grid?

CSS Grid is a **two-dimensional** layout system. While Flexbox works in one dimension (row OR column), Grid works in both dimensions simultaneously (rows AND columns).

**Use Grid for:**
- Overall page layouts
- Complex two-dimensional designs
- Layouts where you need precise control over both axes

**Use Flexbox for:**
- One-dimensional layouts
- Navigation bars
- Aligning items within a component

---

## Grid Container and Grid Items

```html
<div class="grid-container">
    <div class="grid-item">1</div>
    <div class="grid-item">2</div>
    <div class="grid-item">3</div>
    <div class="grid-item">4</div>
    <div class="grid-item">5</div>
    <div class="grid-item">6</div>
</div>
```

```css
.grid-container {
    display: grid;
}
```

---

## Defining Columns and Rows

### grid-template-columns

```css
.container {
    display: grid;

    /* Fixed widths */
    grid-template-columns: 200px 200px 200px;

    /* Percentage widths */
    grid-template-columns: 33.33% 33.33% 33.33%;

    /* fr units (fractions of available space) */
    grid-template-columns: 1fr 1fr 1fr;

    /* Mixed units */
    grid-template-columns: 200px 1fr 2fr;

    /* repeat() function */
    grid-template-columns: repeat(3, 1fr);
    grid-template-columns: repeat(4, 200px);

    /* auto - sizes to content */
    grid-template-columns: auto 1fr auto;
}
```

### grid-template-rows

```css
.container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);

    /* Define row heights */
    grid-template-rows: 100px 200px 100px;
    grid-template-rows: 1fr 2fr 1fr;
    grid-template-rows: auto 1fr auto;
}
```

### The fr Unit

The `fr` (fraction) unit divides available space proportionally:

```css
.container {
    grid-template-columns: 1fr 2fr 1fr;
    /* First column: 1/4 of space
       Second column: 2/4 (half) of space
       Third column: 1/4 of space */
}
```

```
┌────────┬────────────────┬────────┐
│  1fr   │      2fr       │  1fr   │
│  25%   │      50%       │  25%   │
└────────┴────────────────┴────────┘
```

---

## The repeat() Function

```css
/* Without repeat */
grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr;

/* With repeat */
grid-template-columns: repeat(6, 1fr);

/* Mixed with repeat */
grid-template-columns: 200px repeat(4, 1fr) 200px;

/* Pattern repeat */
grid-template-columns: repeat(3, 100px 200px);
/* Creates: 100px 200px 100px 200px 100px 200px */
```

---

## Gap (Gutters)

```css
.container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);

    /* Same gap for rows and columns */
    gap: 20px;

    /* Different gaps */
    gap: 20px 10px;  /* row-gap column-gap */

    /* Individual properties */
    row-gap: 20px;
    column-gap: 10px;
}
```

---

## Placing Items on the Grid

By default, items fill the grid in order. You can explicitly place items using:

### Grid Lines

Grid lines are numbered starting at 1:

```
Column lines:    1    2    3    4
                 ↓    ↓    ↓    ↓
              ┌────┬────┬────┐
Row line 1 →  │    │    │    │
              ├────┼────┼────┤
Row line 2 →  │    │    │    │
              ├────┼────┼────┤
Row line 3 →  │    │    │    │
              └────┴────┴────┘
                             ↑
                         Row line 4
```

### grid-column and grid-row

```css
.item {
    /* Start at line 1, end at line 3 */
    grid-column: 1 / 3;
    grid-row: 1 / 2;

    /* Shorthand for above */
    grid-column-start: 1;
    grid-column-end: 3;
    grid-row-start: 1;
    grid-row-end: 2;

    /* Span X cells */
    grid-column: 1 / span 2;  /* Start at 1, span 2 columns */
    grid-column: span 2;      /* Span 2 from wherever placed */

    /* Span to end */
    grid-column: 1 / -1;      /* -1 = last line */
}
```

### grid-area

Shorthand for row-start, column-start, row-end, column-end:

```css
.item {
    /* grid-area: row-start / col-start / row-end / col-end */
    grid-area: 1 / 1 / 3 / 3;

    /* Equivalent to: */
    grid-row: 1 / 3;
    grid-column: 1 / 3;
}
```

---

## Named Grid Lines

```css
.container {
    display: grid;
    grid-template-columns:
        [sidebar-start] 250px
        [sidebar-end content-start] 1fr
        [content-end];
    grid-template-rows:
        [header-start] 80px
        [header-end main-start] 1fr
        [main-end footer-start] 60px
        [footer-end];
}

.header {
    grid-column: sidebar-start / content-end;
    grid-row: header-start / header-end;
}

.sidebar {
    grid-column: sidebar-start / sidebar-end;
    grid-row: main-start / main-end;
}
```

---

## Named Grid Areas

The most intuitive way to define layouts:

```css
.container {
    display: grid;
    grid-template-columns: 250px 1fr;
    grid-template-rows: 80px 1fr 60px;
    grid-template-areas:
        "header header"
        "sidebar main"
        "footer footer";
    gap: 10px;
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

```
┌──────────────────────────┐
│         header           │
├───────┬──────────────────┤
│       │                  │
│sidebar│      main        │
│       │                  │
├───────┴──────────────────┤
│         footer           │
└──────────────────────────┘
```

**Empty cells:**
```css
grid-template-areas:
    "header header header"
    "sidebar main ."        /* . = empty cell */
    "footer footer footer";
```

---

## Implicit vs Explicit Grid

**Explicit grid:** What you define with `grid-template-*`
**Implicit grid:** Extra rows/columns created automatically for overflow items

```css
.container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);  /* Explicit: 3 columns */
    grid-template-rows: 100px 100px;        /* Explicit: 2 rows */

    /* Style implicit rows */
    grid-auto-rows: 150px;

    /* Style implicit columns */
    grid-auto-columns: 200px;

    /* Direction for auto-placed items */
    grid-auto-flow: row;     /* Default: fill rows first */
    grid-auto-flow: column;  /* Fill columns first */
    grid-auto-flow: dense;   /* Fill gaps if possible */
}
```

---

## Responsive Grids with auto-fill and auto-fit

### auto-fill

Creates as many tracks as will fit:

```css
.container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 20px;
}
```

### auto-fit

Same as auto-fill, but collapses empty tracks:

```css
.container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
}
```

**The magic formula for responsive grids:**

```css
.container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1.5rem;
}
```

This creates a responsive grid where:
- Each item is minimum 250px
- Items grow to fill available space
- Columns automatically wrap based on container width
- No media queries needed!

---

## The minmax() Function

Sets minimum and maximum sizes:

```css
.container {
    grid-template-columns:
        minmax(200px, 1fr)   /* Min 200px, max 1fr */
        minmax(100px, 300px) /* Min 100px, max 300px */
        1fr;

    grid-template-rows:
        minmax(100px, auto)  /* Min 100px, grows with content */
        1fr;
}
```

---

## Alignment in Grid

### Aligning the Entire Grid

```css
.container {
    display: grid;
    height: 500px;  /* Container needs size */

    /* Align grid horizontally */
    justify-content: start;    /* Default */
    justify-content: end;
    justify-content: center;
    justify-content: space-between;
    justify-content: space-around;
    justify-content: space-evenly;

    /* Align grid vertically */
    align-content: start;
    align-content: end;
    align-content: center;
    /* etc. */

    /* Shorthand */
    place-content: center;  /* align-content justify-content */
}
```

### Aligning Items Within Cells

```css
.container {
    /* Align all items horizontally within cells */
    justify-items: start;
    justify-items: end;
    justify-items: center;
    justify-items: stretch;  /* Default */

    /* Align all items vertically within cells */
    align-items: start;
    align-items: end;
    align-items: center;
    align-items: stretch;  /* Default */

    /* Shorthand */
    place-items: center;  /* align-items justify-items */
}
```

### Aligning Individual Items

```css
.item {
    /* Horizontal alignment */
    justify-self: start;
    justify-self: end;
    justify-self: center;
    justify-self: stretch;

    /* Vertical alignment */
    align-self: start;
    align-self: end;
    align-self: center;
    align-self: stretch;

    /* Shorthand */
    place-self: center;  /* align-self justify-self */
}
```

---

## Common Grid Patterns

### 1. Basic Page Layout

```css
body {
    display: grid;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
    margin: 0;
}

header { /* auto height */ }
main   { /* fills remaining space */ }
footer { /* auto height */ }
```

### 2. Sidebar Layout

```css
.layout {
    display: grid;
    grid-template-columns: 250px 1fr;
    min-height: 100vh;
}

@media (max-width: 768px) {
    .layout {
        grid-template-columns: 1fr;
    }
}
```

### 3. Card Grid

```css
.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 2rem;
}
```

### 4. Featured + Grid

```css
.featured-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 1rem;
}

.featured {
    grid-column: span 2;
    grid-row: span 2;
}
```

### 5. Magazine Layout

```css
.magazine {
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    gap: 1rem;
}

.main-story {
    grid-column: 1 / 8;
}

.sidebar-story {
    grid-column: 8 / 13;
}

.small-story {
    grid-column: span 4;
}
```

---

## Complete Example: Dashboard Layout

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grid Dashboard</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            background: #f0f0f0;
        }

        .dashboard {
            display: grid;
            grid-template-columns: 250px 1fr;
            grid-template-rows: 60px 1fr;
            grid-template-areas:
                "sidebar header"
                "sidebar main";
            min-height: 100vh;
        }

        .header {
            grid-area: header;
            background: white;
            padding: 0 2rem;
            display: flex;
            align-items: center;
            justify-content: space-between;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .sidebar {
            grid-area: sidebar;
            background: #2c3e50;
            color: white;
            padding: 1rem;
        }

        .sidebar-nav {
            list-style: none;
            margin-top: 2rem;
        }

        .sidebar-nav li {
            padding: 0.75rem 1rem;
            border-radius: 4px;
            cursor: pointer;
        }

        .sidebar-nav li:hover {
            background: rgba(255,255,255,0.1);
        }

        .main {
            grid-area: main;
            padding: 2rem;
        }

        /* Card grid inside main */
        .cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 1.5rem;
            margin-bottom: 2rem;
        }

        .card {
            background: white;
            padding: 1.5rem;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .card-header {
            color: #666;
            font-size: 0.875rem;
            text-transform: uppercase;
        }

        .card-value {
            font-size: 2rem;
            font-weight: bold;
            margin: 0.5rem 0;
        }

        /* Chart section - spans multiple columns */
        .chart-section {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 1.5rem;
        }

        .chart {
            background: white;
            padding: 1.5rem;
            border-radius: 8px;
            min-height: 300px;
        }

        /* Responsive */
        @media (max-width: 768px) {
            .dashboard {
                grid-template-columns: 1fr;
                grid-template-rows: 60px auto 1fr;
                grid-template-areas:
                    "header"
                    "sidebar"
                    "main";
            }

            .sidebar {
                padding: 0.5rem 1rem;
            }

            .sidebar-nav {
                display: flex;
                gap: 1rem;
                margin-top: 0;
                overflow-x: auto;
            }

            .sidebar-nav li {
                white-space: nowrap;
            }

            .chart-section {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="dashboard">
        <header class="header">
            <h1>Dashboard</h1>
            <span>Welcome, User</span>
        </header>

        <nav class="sidebar">
            <h2>Menu</h2>
            <ul class="sidebar-nav">
                <li>Dashboard</li>
                <li>Analytics</li>
                <li>Reports</li>
                <li>Settings</li>
            </ul>
        </nav>

        <main class="main">
            <div class="cards">
                <div class="card">
                    <div class="card-header">Total Users</div>
                    <div class="card-value">12,345</div>
                    <div>+12% from last month</div>
                </div>
                <div class="card">
                    <div class="card-header">Revenue</div>
                    <div class="card-value">$45,678</div>
                    <div>+8% from last month</div>
                </div>
                <div class="card">
                    <div class="card-header">Orders</div>
                    <div class="card-value">1,234</div>
                    <div>+23% from last month</div>
                </div>
                <div class="card">
                    <div class="card-header">Conversion</div>
                    <div class="card-value">3.2%</div>
                    <div>+0.5% from last month</div>
                </div>
            </div>

            <div class="chart-section">
                <div class="chart">
                    <h3>Sales Over Time</h3>
                    <p>Chart would go here</p>
                </div>
                <div class="chart">
                    <h3>Top Products</h3>
                    <p>Chart would go here</p>
                </div>
            </div>
        </main>
    </div>
</body>
</html>
```

---

## Grid Cheat Sheet

### Container Properties

| Property | Values |
|----------|--------|
| `display` | `grid`, `inline-grid` |
| `grid-template-columns` | track sizes |
| `grid-template-rows` | track sizes |
| `grid-template-areas` | named areas |
| `gap` | `<row-gap> <col-gap>` |
| `justify-items` | `start`, `end`, `center`, `stretch` |
| `align-items` | `start`, `end`, `center`, `stretch` |
| `justify-content` | `start`, `end`, `center`, `space-between`, etc. |
| `align-content` | `start`, `end`, `center`, `space-between`, etc. |
| `grid-auto-rows` | track size for implicit rows |
| `grid-auto-columns` | track size for implicit columns |
| `grid-auto-flow` | `row`, `column`, `dense` |

### Item Properties

| Property | Values |
|----------|--------|
| `grid-column` | `start / end` or `span X` |
| `grid-row` | `start / end` or `span X` |
| `grid-area` | `name` or `row-start / col-start / row-end / col-end` |
| `justify-self` | `start`, `end`, `center`, `stretch` |
| `align-self` | `start`, `end`, `center`, `stretch` |

---

## Key Takeaways

1. **Grid is 2D**, Flexbox is 1D
2. **`fr` unit** divides available space
3. **`repeat()`** reduces repetition
4. **`minmax()`** sets size constraints
5. **`auto-fit` + `minmax()`** = responsive grid without media queries
6. **Named grid areas** are the most intuitive for layouts
7. **Use Grid for layout**, Flexbox for components

---

**Week 2 Complete!**

**Next:** [Week 3 - Responsive Design & Modern CSS](./week-03-day-01-02-responsive.md)
