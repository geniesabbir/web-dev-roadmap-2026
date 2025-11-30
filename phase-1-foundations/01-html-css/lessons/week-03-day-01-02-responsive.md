# Week 3, Day 1-2: Responsive Web Design

## What is Responsive Design?

Responsive web design makes your website look good on all devices - from mobile phones to desktop monitors. Instead of creating separate websites for different devices, you create one website that adapts.

---

## The Viewport Meta Tag

This tag is **essential** for responsive design. Without it, mobile browsers will render your page at desktop width.

```html
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
```

**What it does:**
- `width=device-width` - Sets viewport width to device width
- `initial-scale=1.0` - Sets initial zoom level to 100%

**Never do this:**
```html
<!-- Bad - prevents user zoom (accessibility issue) -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```

---

## Mobile-First Approach

Write CSS for mobile first, then add styles for larger screens.

**Why mobile-first?**
1. Mobile is the most constrained - start simple
2. Progressive enhancement is easier than graceful degradation
3. Forces you to prioritize content
4. Better performance on mobile (less CSS to download)

```css
/* Mobile styles (default) */
.container {
    padding: 1rem;
}

.card {
    width: 100%;
}

/* Tablet and up */
@media (min-width: 768px) {
    .container {
        padding: 2rem;
    }

    .card {
        width: 50%;
    }
}

/* Desktop and up */
@media (min-width: 1024px) {
    .container {
        max-width: 1200px;
        margin: 0 auto;
    }

    .card {
        width: 33.333%;
    }
}
```

---

## Media Queries

Media queries apply CSS based on device characteristics.

### Basic Syntax

```css
@media media-type and (condition) {
    /* CSS rules */
}
```

### Media Types

```css
@media screen { }  /* Screens (default) */
@media print { }   /* Printers */
@media all { }     /* All devices */
```

### Width-Based Queries (Most Common)

```css
/* Minimum width - applies at this width AND above */
@media (min-width: 768px) {
    /* Tablet and larger */
}

/* Maximum width - applies at this width AND below */
@media (max-width: 767px) {
    /* Mobile only */
}

/* Range */
@media (min-width: 768px) and (max-width: 1023px) {
    /* Tablet only */
}
```

### Common Breakpoints

```css
/* Mobile first approach */

/* Small phones */
/* No media query - default styles */

/* Large phones */
@media (min-width: 480px) { }

/* Tablets */
@media (min-width: 768px) { }

/* Laptops / Small desktops */
@media (min-width: 1024px) { }

/* Large desktops */
@media (min-width: 1280px) { }

/* Extra large screens */
@media (min-width: 1536px) { }
```

**Modern approach - use content breakpoints:**

Instead of device-specific breakpoints, break when your content needs it:

```css
/* Break when cards get too wide */
@media (min-width: 600px) {
    .card-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (min-width: 900px) {
    .card-grid {
        grid-template-columns: repeat(3, 1fr);
    }
}
```

### Other Media Features

```css
/* Orientation */
@media (orientation: portrait) { }
@media (orientation: landscape) { }

/* High-resolution screens (retina) */
@media (min-resolution: 2dppx) { }
@media (-webkit-min-device-pixel-ratio: 2) { }

/* Hover capability */
@media (hover: hover) {
    /* Device has hover (mouse) */
    .button:hover {
        background: blue;
    }
}

@media (hover: none) {
    /* Touch device - no hover */
}

/* Pointer precision */
@media (pointer: coarse) {
    /* Touch - make targets bigger */
    .button {
        padding: 1rem 2rem;
    }
}

@media (pointer: fine) {
    /* Mouse - precise clicking */
}

/* Prefers reduced motion (accessibility) */
@media (prefers-reduced-motion: reduce) {
    * {
        animation: none !important;
        transition: none !important;
    }
}

/* Dark mode preference */
@media (prefers-color-scheme: dark) {
    body {
        background: #1a1a1a;
        color: #ffffff;
    }
}

@media (prefers-color-scheme: light) {
    body {
        background: #ffffff;
        color: #1a1a1a;
    }
}
```

---

## Responsive Units

### Relative Units for Typography

```css
/* Root font size */
html {
    font-size: 16px;  /* Base */
}

/* Use rem for consistent scaling */
h1 { font-size: 2.5rem; }    /* 40px */
h2 { font-size: 2rem; }      /* 32px */
h3 { font-size: 1.5rem; }    /* 24px */
p  { font-size: 1rem; }      /* 16px */
small { font-size: 0.875rem; } /* 14px */

/* Responsive font sizes */
html {
    font-size: 14px;
}

@media (min-width: 768px) {
    html {
        font-size: 16px;
    }
}

@media (min-width: 1024px) {
    html {
        font-size: 18px;
    }
}
```

### Fluid Typography with clamp()

```css
/* clamp(minimum, preferred, maximum) */
h1 {
    /* Min 24px, scales with viewport, max 48px */
    font-size: clamp(1.5rem, 5vw, 3rem);
}

p {
    font-size: clamp(1rem, 2vw, 1.25rem);
}
```

### Viewport Units

```css
.hero {
    /* Full viewport height */
    height: 100vh;

    /* Full viewport width */
    width: 100vw;
}

/* Responsive padding */
.section {
    padding: 5vh 5vw;
}

/* Note: 100vh can be problematic on mobile due to browser chrome */
/* Use svh (small viewport height) or dvh (dynamic viewport height) */
.hero {
    min-height: 100svh;
}
```

---

## Responsive Images

### Basic Responsive Image

```css
img {
    max-width: 100%;
    height: auto;
}
```

### Using srcset for Resolution

```html
<!-- Different resolutions for different screens -->
<img
    src="image-400.jpg"
    srcset="
        image-400.jpg 400w,
        image-800.jpg 800w,
        image-1200.jpg 1200w
    "
    sizes="(max-width: 600px) 100vw, 50vw"
    alt="Description"
>
```

**How it works:**
- `srcset` lists available images with their widths
- `sizes` tells browser how wide the image will display
- Browser picks the best image automatically

### The `<picture>` Element

For art direction (different images for different screens):

```html
<picture>
    <!-- Mobile: cropped/square image -->
    <source
        media="(max-width: 767px)"
        srcset="hero-mobile.jpg"
    >

    <!-- Tablet: medium crop -->
    <source
        media="(max-width: 1023px)"
        srcset="hero-tablet.jpg"
    >

    <!-- Desktop: full image -->
    <img src="hero-desktop.jpg" alt="Hero image">
</picture>
```

### Different Formats

```html
<picture>
    <!-- Modern format for supporting browsers -->
    <source type="image/avif" srcset="image.avif">
    <source type="image/webp" srcset="image.webp">

    <!-- Fallback -->
    <img src="image.jpg" alt="Description">
</picture>
```

### Background Images

```css
.hero {
    background-image: url('hero-mobile.jpg');
    background-size: cover;
    background-position: center;
}

@media (min-width: 768px) {
    .hero {
        background-image: url('hero-desktop.jpg');
    }
}

/* High-resolution screens */
@media (min-resolution: 2dppx) {
    .hero {
        background-image: url('hero-desktop@2x.jpg');
    }
}
```

---

## Responsive Layout Patterns

### 1. Fluid Grid

```css
.container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 1rem;
}

.row {
    display: flex;
    flex-wrap: wrap;
    margin: 0 -1rem;
}

.col {
    padding: 0 1rem;
    width: 100%;
}

@media (min-width: 768px) {
    .col-md-6 { width: 50%; }
    .col-md-4 { width: 33.333%; }
    .col-md-3 { width: 25%; }
}

@media (min-width: 1024px) {
    .col-lg-6 { width: 50%; }
    .col-lg-4 { width: 33.333%; }
    .col-lg-3 { width: 25%; }
}
```

### 2. CSS Grid Auto-Fit (No Media Queries!)

```css
.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 2rem;
}
```

### 3. Responsive Navigation

**Mobile: Hamburger Menu**
**Desktop: Horizontal Nav**

```html
<nav class="nav">
    <div class="nav__logo">Logo</div>
    <button class="nav__toggle" aria-label="Toggle menu">
        <span></span>
        <span></span>
        <span></span>
    </button>
    <ul class="nav__menu">
        <li><a href="#">Home</a></li>
        <li><a href="#">About</a></li>
        <li><a href="#">Services</a></li>
        <li><a href="#">Contact</a></li>
    </ul>
</nav>
```

```css
.nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
}

.nav__toggle {
    display: block;
    background: none;
    border: none;
    cursor: pointer;
}

.nav__toggle span {
    display: block;
    width: 25px;
    height: 3px;
    background: #333;
    margin: 5px 0;
}

.nav__menu {
    position: fixed;
    top: 60px;
    left: 0;
    right: 0;
    background: white;
    flex-direction: column;
    padding: 1rem;
    display: none;  /* Hidden by default */
}

.nav__menu.active {
    display: flex;
}

.nav__menu li {
    padding: 1rem;
    border-bottom: 1px solid #eee;
}

/* Desktop */
@media (min-width: 768px) {
    .nav__toggle {
        display: none;
    }

    .nav__menu {
        position: static;
        display: flex;
        flex-direction: row;
        padding: 0;
        background: transparent;
    }

    .nav__menu li {
        padding: 0 1rem;
        border-bottom: none;
    }
}
```

### 4. Stack to Side-by-Side

```css
.feature {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.feature__image,
.feature__content {
    width: 100%;
}

@media (min-width: 768px) {
    .feature {
        flex-direction: row;
        align-items: center;
    }

    .feature__image,
    .feature__content {
        width: 50%;
    }

    /* Alternate layout */
    .feature:nth-child(even) {
        flex-direction: row-reverse;
    }
}
```

### 5. Responsive Table

Tables are notoriously hard to make responsive:

**Option 1: Horizontal scroll**
```css
.table-container {
    overflow-x: auto;
}

table {
    min-width: 600px;
}
```

**Option 2: Stack on mobile**
```css
@media (max-width: 767px) {
    table, thead, tbody, th, td, tr {
        display: block;
    }

    thead {
        display: none;
    }

    tr {
        margin-bottom: 1rem;
        border: 1px solid #ddd;
    }

    td {
        display: flex;
        justify-content: space-between;
        padding: 0.5rem;
        border-bottom: 1px solid #eee;
    }

    td::before {
        content: attr(data-label);
        font-weight: bold;
    }
}
```

```html
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td data-label="Name">John Doe</td>
            <td data-label="Email">john@example.com</td>
            <td data-label="Role">Admin</td>
        </tr>
    </tbody>
</table>
```

---

## Container Queries (Modern CSS)

Instead of viewport-based queries, style based on container size:

```css
.card-container {
    container-type: inline-size;
    container-name: card;
}

/* When container is at least 400px wide */
@container card (min-width: 400px) {
    .card {
        display: flex;
    }

    .card__image {
        width: 40%;
    }
}

@container card (min-width: 600px) {
    .card__title {
        font-size: 1.5rem;
    }
}
```

---

## Complete Responsive Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Responsive Page</title>
    <style>
        /* Reset */
        *, *::before, *::after {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        /* Base styles (mobile) */
        html {
            font-size: 16px;
        }

        body {
            font-family: system-ui, -apple-system, sans-serif;
            line-height: 1.6;
            color: #333;
        }

        img {
            max-width: 100%;
            height: auto;
            display: block;
        }

        /* Container */
        .container {
            width: 100%;
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 1rem;
        }

        /* Header */
        .header {
            background: #2c3e50;
            color: white;
            padding: 1rem;
        }

        .header__inner {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .nav-toggle {
            display: flex;
            flex-direction: column;
            gap: 4px;
            background: none;
            border: none;
            cursor: pointer;
            padding: 0.5rem;
        }

        .nav-toggle span {
            display: block;
            width: 24px;
            height: 2px;
            background: white;
        }

        .nav {
            position: absolute;
            top: 100%;
            left: 0;
            right: 0;
            background: #34495e;
            display: none;
        }

        .nav.active {
            display: block;
        }

        .nav a {
            display: block;
            padding: 1rem;
            color: white;
            text-decoration: none;
            border-bottom: 1px solid #2c3e50;
        }

        /* Hero */
        .hero {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 3rem 1rem;
            text-align: center;
        }

        .hero h1 {
            font-size: clamp(1.75rem, 5vw, 3rem);
            margin-bottom: 1rem;
        }

        .hero p {
            font-size: clamp(1rem, 2vw, 1.25rem);
            opacity: 0.9;
        }

        /* Cards */
        .cards {
            padding: 2rem 0;
        }

        .cards__grid {
            display: grid;
            grid-template-columns: 1fr;
            gap: 1.5rem;
        }

        .card {
            background: white;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

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

        /* Footer */
        .footer {
            background: #2c3e50;
            color: white;
            padding: 2rem 1rem;
            text-align: center;
        }

        /* Tablet (768px+) */
        @media (min-width: 768px) {
            .container {
                padding: 0 2rem;
            }

            .nav-toggle {
                display: none;
            }

            .nav {
                position: static;
                display: flex;
                background: transparent;
            }

            .nav a {
                border-bottom: none;
                padding: 0.5rem 1rem;
            }

            .nav a:hover {
                text-decoration: underline;
            }

            .hero {
                padding: 5rem 2rem;
            }

            .cards__grid {
                grid-template-columns: repeat(2, 1fr);
            }
        }

        /* Desktop (1024px+) */
        @media (min-width: 1024px) {
            .hero {
                padding: 6rem 2rem;
            }

            .cards__grid {
                grid-template-columns: repeat(3, 1fr);
            }

            .cards {
                padding: 3rem 0;
            }
        }

        /* Dark mode */
        @media (prefers-color-scheme: dark) {
            body {
                background: #1a1a2e;
                color: #eee;
            }

            .card {
                background: #16213e;
            }
        }

        /* Reduced motion */
        @media (prefers-reduced-motion: reduce) {
            * {
                animation-duration: 0.01ms !important;
                transition-duration: 0.01ms !important;
            }
        }
    </style>
</head>
<body>
    <header class="header">
        <div class="header__inner container">
            <div class="logo">MyBrand</div>
            <button class="nav-toggle" aria-label="Toggle navigation">
                <span></span>
                <span></span>
                <span></span>
            </button>
            <nav class="nav">
                <a href="#">Home</a>
                <a href="#">About</a>
                <a href="#">Services</a>
                <a href="#">Contact</a>
            </nav>
        </div>
    </header>

    <section class="hero">
        <div class="container">
            <h1>Responsive Web Design</h1>
            <p>Building websites that work beautifully on every device</p>
        </div>
    </section>

    <section class="cards">
        <div class="container">
            <div class="cards__grid">
                <article class="card">
                    <img class="card__image" src="https://picsum.photos/400/200?1" alt="">
                    <div class="card__content">
                        <h2 class="card__title">Mobile First</h2>
                        <p>Start with mobile and progressively enhance.</p>
                    </div>
                </article>
                <article class="card">
                    <img class="card__image" src="https://picsum.photos/400/200?2" alt="">
                    <div class="card__content">
                        <h2 class="card__title">Flexible Grids</h2>
                        <p>Use percentages and fr units for flexible layouts.</p>
                    </div>
                </article>
                <article class="card">
                    <img class="card__image" src="https://picsum.photos/400/200?3" alt="">
                    <div class="card__content">
                        <h2 class="card__title">Media Queries</h2>
                        <p>Adapt styles based on device characteristics.</p>
                    </div>
                </article>
            </div>
        </div>
    </section>

    <footer class="footer">
        <div class="container">
            <p>&copy; 2024 MyBrand. All rights reserved.</p>
        </div>
    </footer>

    <script>
        // Simple toggle for mobile nav
        document.querySelector('.nav-toggle').addEventListener('click', () => {
            document.querySelector('.nav').classList.toggle('active');
        });
    </script>
</body>
</html>
```

---

## Testing Responsive Designs

### Browser DevTools

1. Open DevTools (F12 or Cmd+Opt+I)
2. Click the device toggle icon
3. Select device or enter custom dimensions
4. Test at various widths

### Real Device Testing

- Test on actual phones and tablets
- Use BrowserStack or similar for more devices
- Check both orientations (portrait/landscape)

### Checklist

- [ ] Test at common breakpoints (320, 375, 768, 1024, 1440)
- [ ] Test between breakpoints
- [ ] Test in both orientations
- [ ] Test with zoom (up to 200%)
- [ ] Test with long content
- [ ] Test with different font sizes
- [ ] Check touch targets (min 44x44px)

---

## Key Takeaways

1. **Always include viewport meta tag**
2. **Mobile-first** - base styles for mobile, enhance for larger
3. **Use relative units** - rem, em, %, vw/vh
4. **Use modern layout** - Grid and Flexbox respond naturally
5. **clamp()** - Great for fluid typography
6. **auto-fit + minmax()** - Responsive grids without media queries
7. **Test on real devices** - Emulators aren't perfect

---

**Next Lesson:** [Day 3 - CSS Architecture & BEM](./week-03-day-03-css-architecture.md)
