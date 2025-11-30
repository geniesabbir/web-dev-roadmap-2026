# Week 3, Day 4-5: Modern CSS Features

## CSS Transitions

Transitions animate changes between CSS property values.

### Basic Syntax

```css
.element {
    transition: property duration timing-function delay;
}
```

### Simple Transition

```css
.button {
    background: #3498db;
    transition: background 0.3s ease;
}

.button:hover {
    background: #2980b9;
}
```

### Multiple Properties

```css
.card {
    transform: translateY(0);
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(0,0,0,0.15);
}

/* Or transition all properties */
.card {
    transition: all 0.3s ease;
}
```

### Transition Properties

```css
.element {
    /* Individual properties */
    transition-property: transform, opacity;
    transition-duration: 0.3s, 0.5s;
    transition-timing-function: ease, linear;
    transition-delay: 0s, 0.1s;

    /* Shorthand */
    transition: transform 0.3s ease 0s, opacity 0.5s linear 0.1s;
}
```

### Timing Functions

```css
.element {
    /* Built-in */
    transition-timing-function: linear;      /* Constant speed */
    transition-timing-function: ease;        /* Default - slow start/end */
    transition-timing-function: ease-in;     /* Slow start */
    transition-timing-function: ease-out;    /* Slow end */
    transition-timing-function: ease-in-out; /* Slow start and end */

    /* Custom cubic-bezier */
    transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);

    /* Steps */
    transition-timing-function: steps(4, end);
}
```

### Practical Examples

**Button Hover:**
```css
.button {
    padding: 0.75rem 1.5rem;
    background: #3498db;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.2s ease, transform 0.2s ease;
}

.button:hover {
    background: #2980b9;
    transform: translateY(-2px);
}

.button:active {
    transform: translateY(0);
}
```

**Link Underline:**
```css
.link {
    position: relative;
    text-decoration: none;
    color: #3498db;
}

.link::after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 0;
    width: 0;
    height: 2px;
    background: currentColor;
    transition: width 0.3s ease;
}

.link:hover::after {
    width: 100%;
}
```

**Fade In/Out:**
```css
.modal {
    opacity: 0;
    visibility: hidden;
    transition: opacity 0.3s ease, visibility 0.3s ease;
}

.modal.active {
    opacity: 1;
    visibility: visible;
}
```

---

## CSS Transforms

Transforms modify the visual appearance of elements without affecting layout.

### 2D Transforms

```css
.element {
    /* Move */
    transform: translateX(50px);
    transform: translateY(50px);
    transform: translate(50px, 50px);

    /* Scale */
    transform: scaleX(1.5);      /* Width */
    transform: scaleY(1.5);      /* Height */
    transform: scale(1.5);       /* Both */
    transform: scale(1.5, 2);    /* X, Y */

    /* Rotate */
    transform: rotate(45deg);
    transform: rotate(-45deg);
    transform: rotate(0.5turn);  /* 180deg */

    /* Skew */
    transform: skewX(10deg);
    transform: skewY(10deg);
    transform: skew(10deg, 5deg);

    /* Multiple transforms */
    transform: translateX(50px) rotate(45deg) scale(1.2);
}
```

### Transform Origin

```css
.element {
    transform-origin: center;       /* Default */
    transform-origin: top left;
    transform-origin: 50% 50%;
    transform-origin: 0 0;
}

/* Rotate from corner */
.element {
    transform-origin: top left;
    transform: rotate(45deg);
}
```

### 3D Transforms

```css
.container {
    perspective: 1000px;  /* Depth of 3D space */
}

.element {
    transform: rotateX(45deg);
    transform: rotateY(45deg);
    transform: rotateZ(45deg);      /* Same as rotate() */
    transform: rotate3d(1, 1, 0, 45deg);

    transform: translateZ(50px);
    transform: translate3d(50px, 50px, 50px);

    transform: scaleZ(1.5);
    transform: scale3d(1, 1, 1.5);

    /* Preserve 3D for nested elements */
    transform-style: preserve-3d;
}
```

### Practical Transform Examples

**Card Flip:**
```css
.flip-card {
    perspective: 1000px;
    width: 300px;
    height: 200px;
}

.flip-card__inner {
    position: relative;
    width: 100%;
    height: 100%;
    transition: transform 0.6s;
    transform-style: preserve-3d;
}

.flip-card:hover .flip-card__inner {
    transform: rotateY(180deg);
}

.flip-card__front,
.flip-card__back {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    border-radius: 8px;
}

.flip-card__front {
    background: #3498db;
    color: white;
}

.flip-card__back {
    background: #2ecc71;
    color: white;
    transform: rotateY(180deg);
}
```

**Zoom on Hover:**
```css
.image-container {
    overflow: hidden;
}

.image-container img {
    transition: transform 0.5s ease;
}

.image-container:hover img {
    transform: scale(1.1);
}
```

---

## CSS Animations

Animations allow complex multi-step transitions using keyframes.

### Basic Animation

```css
/* Define keyframes */
@keyframes fadeIn {
    from {
        opacity: 0;
    }
    to {
        opacity: 1;
    }
}

/* Apply animation */
.element {
    animation: fadeIn 1s ease;
}
```

### Animation Properties

```css
.element {
    animation-name: fadeIn;
    animation-duration: 1s;
    animation-timing-function: ease;
    animation-delay: 0s;
    animation-iteration-count: 1;        /* or infinite */
    animation-direction: normal;         /* or reverse, alternate */
    animation-fill-mode: forwards;       /* keeps end state */
    animation-play-state: running;       /* or paused */

    /* Shorthand */
    animation: fadeIn 1s ease 0s 1 normal forwards running;

    /* Multiple animations */
    animation: fadeIn 1s ease, slideUp 1s ease;
}
```

### Keyframe Percentages

```css
@keyframes bounce {
    0% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(-20px);
    }
    100% {
        transform: translateY(0);
    }
}

@keyframes colorChange {
    0% { background: red; }
    25% { background: yellow; }
    50% { background: green; }
    75% { background: blue; }
    100% { background: red; }
}
```

### Practical Animation Examples

**Loading Spinner:**
```css
@keyframes spin {
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
}

.spinner {
    width: 40px;
    height: 40px;
    border: 3px solid #f3f3f3;
    border-top: 3px solid #3498db;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}
```

**Pulse Effect:**
```css
@keyframes pulse {
    0% {
        transform: scale(1);
        box-shadow: 0 0 0 0 rgba(52, 152, 219, 0.7);
    }
    70% {
        transform: scale(1.05);
        box-shadow: 0 0 0 10px rgba(52, 152, 219, 0);
    }
    100% {
        transform: scale(1);
        box-shadow: 0 0 0 0 rgba(52, 152, 219, 0);
    }
}

.pulse-button {
    animation: pulse 2s infinite;
}
```

**Slide In:**
```css
@keyframes slideInLeft {
    from {
        transform: translateX(-100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

@keyframes slideInUp {
    from {
        transform: translateY(50px);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}

.animate-slide-in {
    animation: slideInUp 0.5s ease forwards;
}
```

**Typing Effect:**
```css
@keyframes typing {
    from { width: 0; }
    to { width: 100%; }
}

@keyframes blink {
    50% { border-color: transparent; }
}

.typing-text {
    width: 0;
    overflow: hidden;
    white-space: nowrap;
    border-right: 3px solid #333;
    animation:
        typing 3.5s steps(40) forwards,
        blink 0.75s step-end infinite;
}
```

**Stagger Animation:**
```css
.card {
    opacity: 0;
    animation: fadeInUp 0.5s ease forwards;
}

.card:nth-child(1) { animation-delay: 0.1s; }
.card:nth-child(2) { animation-delay: 0.2s; }
.card:nth-child(3) { animation-delay: 0.3s; }
.card:nth-child(4) { animation-delay: 0.4s; }

@keyframes fadeInUp {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}
```

---

## CSS Filters

Apply visual effects to elements.

```css
.element {
    filter: blur(5px);
    filter: brightness(1.2);
    filter: contrast(1.5);
    filter: grayscale(100%);
    filter: hue-rotate(90deg);
    filter: invert(100%);
    filter: opacity(50%);
    filter: saturate(2);
    filter: sepia(100%);
    filter: drop-shadow(2px 2px 4px rgba(0,0,0,0.5));

    /* Multiple filters */
    filter: grayscale(50%) blur(2px);
}
```

**Image Hover Effect:**
```css
.image {
    filter: grayscale(100%);
    transition: filter 0.3s ease;
}

.image:hover {
    filter: grayscale(0%);
}
```

**Frosted Glass Effect:**
```css
.glass {
    background: rgba(255, 255, 255, 0.2);
    backdrop-filter: blur(10px);
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-radius: 10px;
}
```

---

## CSS Functions

### calc()

Perform calculations:

```css
.element {
    /* Mixed units */
    width: calc(100% - 40px);
    height: calc(100vh - 60px);
    padding: calc(1rem + 5px);

    /* Nested calculations */
    width: calc((100% - 40px) / 3);

    /* With variables */
    --sidebar: 250px;
    width: calc(100% - var(--sidebar));
}
```

### min(), max(), clamp()

Responsive values without media queries:

```css
.element {
    /* min() - picks smaller value */
    width: min(100%, 600px);

    /* max() - picks larger value */
    width: max(50%, 300px);

    /* clamp(min, preferred, max) */
    font-size: clamp(1rem, 2.5vw, 2rem);
    width: clamp(200px, 50%, 800px);
    padding: clamp(1rem, 5vw, 3rem);
}
```

### CSS Functions for Colors

```css
.element {
    /* Modern color functions */
    background: rgb(52 152 219);           /* Space-separated */
    background: rgb(52 152 219 / 50%);     /* With alpha */

    background: hsl(204 70% 53%);
    background: hsl(204 70% 53% / 50%);

    /* Color manipulation */
    background: color-mix(in srgb, blue 50%, red 50%);
}
```

---

## Aspect Ratio

Maintain proportions:

```css
.video-container {
    aspect-ratio: 16 / 9;
    width: 100%;
}

.square {
    aspect-ratio: 1;  /* 1:1 */
}

.portrait {
    aspect-ratio: 3 / 4;
}
```

---

## Scroll Behavior

```css
html {
    scroll-behavior: smooth;
}

/* Or target specific element */
.container {
    scroll-behavior: smooth;
    overflow-y: auto;
}
```

---

## Scroll Snap

Create carousel-like scrolling:

```css
.scroll-container {
    display: flex;
    overflow-x: auto;
    scroll-snap-type: x mandatory;
    gap: 1rem;
}

.scroll-item {
    flex: 0 0 100%;
    scroll-snap-align: start;
}

/* Vertical scroll snap */
.vertical-container {
    height: 100vh;
    overflow-y: auto;
    scroll-snap-type: y mandatory;
}

.section {
    height: 100vh;
    scroll-snap-align: start;
}
```

---

## Logical Properties

Write CSS that works for any text direction (LTR/RTL):

```css
/* Instead of margin-left, margin-right */
.element {
    margin-inline-start: 1rem;  /* Left in LTR, Right in RTL */
    margin-inline-end: 1rem;
    margin-inline: 1rem;        /* Both sides */

    margin-block-start: 1rem;   /* Top */
    margin-block-end: 1rem;     /* Bottom */
    margin-block: 1rem;         /* Top and bottom */

    padding-inline: 1rem;
    padding-block: 1rem;

    border-inline-start: 1px solid;
    border-block-end: 1px solid;

    /* Size */
    inline-size: 200px;         /* Width */
    block-size: 100px;          /* Height */
}
```

---

## :has() Selector (Parent Selector)

Select parents based on children:

```css
/* Card with image */
.card:has(img) {
    padding-top: 0;
}

/* Form with errors */
.form:has(.input--error) {
    border-color: red;
}

/* If any checkbox is checked */
.list:has(input:checked) {
    background: #f0f0f0;
}

/* Style preceding sibling */
h2:has(+ p) {
    margin-bottom: 0.5rem;
}
```

---

## Complete Modern CSS Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Modern CSS Demo</title>
    <style>
        :root {
            --primary: #3498db;
            --primary-dark: #2980b9;
            --text: #333;
            --bg: #f5f5f5;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        html {
            scroll-behavior: smooth;
        }

        body {
            font-family: system-ui, sans-serif;
            background: var(--bg);
            color: var(--text);
        }

        /* Hero with gradient animation */
        .hero {
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            background: linear-gradient(135deg, #667eea, #764ba2);
            animation: gradientShift 10s ease infinite;
            background-size: 200% 200%;
        }

        @keyframes gradientShift {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        .hero__title {
            font-size: clamp(2rem, 8vw, 5rem);
            color: white;
            text-align: center;
            animation: fadeInUp 1s ease;
        }

        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        /* Cards section */
        .cards {
            padding: clamp(2rem, 5vw, 4rem);
            max-width: 1200px;
            margin: 0 auto;
        }

        .cards__grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 2rem;
        }

        .card {
            background: white;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        .card:hover {
            transform: translateY(-10px);
            box-shadow: 0 20px 40px rgba(0,0,0,0.15);
        }

        .card__image {
            width: 100%;
            aspect-ratio: 16 / 9;
            object-fit: cover;
            filter: grayscale(20%);
            transition: filter 0.3s ease, transform 0.3s ease;
        }

        .card:hover .card__image {
            filter: grayscale(0%);
            transform: scale(1.05);
        }

        .card__content {
            padding: 1.5rem;
        }

        .card__title {
            font-size: 1.25rem;
            margin-bottom: 0.5rem;
        }

        /* Button with animation */
        .button {
            display: inline-block;
            padding: 0.75rem 1.5rem;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 1rem;
            cursor: pointer;
            position: relative;
            overflow: hidden;
            transition: background 0.3s ease;
        }

        .button::after {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            width: 0;
            height: 0;
            background: rgba(255,255,255,0.2);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            transition: width 0.6s ease, height 0.6s ease;
        }

        .button:hover::after {
            width: 300px;
            height: 300px;
        }

        .button:hover {
            background: var(--primary-dark);
        }

        /* Loading spinner */
        .spinner {
            width: 50px;
            height: 50px;
            border: 4px solid #f3f3f3;
            border-top: 4px solid var(--primary);
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin: 2rem auto;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        /* Glass card */
        .glass-card {
            background: rgba(255, 255, 255, 0.2);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.3);
            border-radius: 16px;
            padding: 2rem;
            color: white;
        }

        /* Dark mode */
        @media (prefers-color-scheme: dark) {
            :root {
                --text: #eee;
                --bg: #1a1a2e;
            }

            .card {
                background: #16213e;
            }
        }

        /* Reduce motion for accessibility */
        @media (prefers-reduced-motion: reduce) {
            *, *::before, *::after {
                animation-duration: 0.01ms !important;
                transition-duration: 0.01ms !important;
            }
        }
    </style>
</head>
<body>
    <section class="hero">
        <h1 class="hero__title">Modern CSS</h1>
    </section>

    <section class="cards">
        <div class="cards__grid">
            <article class="card">
                <img class="card__image" src="https://picsum.photos/400/225?1" alt="">
                <div class="card__content">
                    <h2 class="card__title">Transitions</h2>
                    <p>Smooth state changes</p>
                </div>
            </article>
            <article class="card">
                <img class="card__image" src="https://picsum.photos/400/225?2" alt="">
                <div class="card__content">
                    <h2 class="card__title">Transforms</h2>
                    <p>2D and 3D effects</p>
                </div>
            </article>
            <article class="card">
                <img class="card__image" src="https://picsum.photos/400/225?3" alt="">
                <div class="card__content">
                    <h2 class="card__title">Animations</h2>
                    <p>Complex keyframe animations</p>
                </div>
            </article>
        </div>

        <div style="text-align: center; margin-top: 2rem;">
            <button class="button">Hover Me</button>
        </div>

        <div class="spinner"></div>
    </section>
</body>
</html>
```

---

## Key Takeaways

1. **Transitions** - For simple A→B state changes
2. **Animations** - For complex multi-step effects
3. **Transforms** - Move, rotate, scale without affecting layout
4. **Filters** - Visual effects like blur, grayscale
5. **clamp()** - Responsive values without media queries
6. **aspect-ratio** - Maintain proportions easily
7. **:has()** - Finally, a parent selector!
8. **Always consider reduced motion** for accessibility

---

## HTML/CSS Complete!

You've learned:
- HTML structure and semantics
- CSS fundamentals and selectors
- Flexbox and Grid layouts
- Responsive design
- CSS architecture with BEM
- Modern CSS features

**Next:** [Phase 1: JavaScript](../../02-javascript/README.md)
