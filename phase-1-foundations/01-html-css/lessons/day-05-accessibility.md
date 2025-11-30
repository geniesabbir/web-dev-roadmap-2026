# Day 5: Web Accessibility (a11y)

## What is Web Accessibility?

Web accessibility means making websites usable by everyone, including people with disabilities. This includes:

- **Visual impairments** (blindness, low vision, color blindness)
- **Hearing impairments** (deafness, hard of hearing)
- **Motor impairments** (limited fine motor control, paralysis)
- **Cognitive impairments** (learning disabilities, memory issues)

The term "a11y" is a numeronym for accessibility (a + 11 letters + y).

---

## Why Accessibility Matters

### 1. It's the Right Thing to Do
About 15% of the world's population has some form of disability.

### 2. It's Often Required by Law
- ADA (Americans with Disabilities Act)
- Section 508 (US Government)
- WCAG (Web Content Accessibility Guidelines)
- EU Accessibility Act

### 3. It Improves SEO
Many accessibility practices also help search engines understand your content.

### 4. It Helps Everyone
- Captions help people in noisy environments
- Good contrast helps in bright sunlight
- Keyboard navigation helps power users

---

## WCAG Principles (POUR)

The Web Content Accessibility Guidelines are built on four principles:

### 1. Perceivable
Users must be able to perceive the content.
- Provide text alternatives for images
- Provide captions for videos
- Use sufficient color contrast

### 2. Operable
Users must be able to operate the interface.
- Make everything keyboard accessible
- Give users enough time
- Don't cause seizures

### 3. Understandable
Users must understand the content and interface.
- Use clear language
- Make navigation consistent
- Help users avoid and correct errors

### 4. Robust
Content must work with various technologies.
- Use valid HTML
- Follow web standards
- Test with assistive technologies

---

## Essential Accessibility Practices

### 1. Alternative Text for Images

Every meaningful image needs alt text:

```html
<!-- Informative image - describe the content -->
<img src="chart.png" alt="Sales chart showing 50% growth in Q4 2024">

<!-- Functional image - describe the function -->
<img src="search-icon.png" alt="Search">

<!-- Decorative image - empty alt -->
<img src="decorative-border.png" alt="">

<!-- Complex image - brief alt + longer description -->
<img src="complex-diagram.png" alt="Company organization chart"
     aria-describedby="org-desc">
<p id="org-desc">The CEO is at the top, with three departments below:
   Engineering, Marketing, and Sales. Each department has a director
   and multiple team leads.</p>
```

**Writing Good Alt Text:**
- Be concise but descriptive
- Don't start with "image of" or "picture of"
- Include important information
- For buttons/links, describe the action

```html
<!-- Bad -->
<img src="dog.jpg" alt="image">
<img src="dog.jpg" alt="dog.jpg">
<img src="dog.jpg" alt="image of a dog">

<!-- Good -->
<img src="dog.jpg" alt="Golden retriever playing fetch in a park">

<!-- For a linked logo -->
<a href="/">
    <img src="logo.png" alt="Acme Corp - Go to homepage">
</a>
```

---

### 2. Semantic HTML

Use the right element for the job:

```html
<!-- Bad - div soup -->
<div class="header">
    <div class="nav">
        <div class="nav-item">Home</div>
    </div>
</div>

<!-- Good - semantic elements -->
<header>
    <nav>
        <a href="/">Home</a>
    </nav>
</header>
```

**Semantic elements announce themselves to screen readers:**
- `<nav>` - "navigation"
- `<main>` - "main"
- `<article>` - "article"
- `<button>` - "button"
- `<a>` - "link"

---

### 3. Heading Hierarchy

Headings create an outline of the page for screen reader users:

```html
<!-- Bad - skipping levels, using headings for styling -->
<h1>Website Name</h1>
<h4>Section that should be h2</h4>
<h2>Another section</h2>
<h5>Subsection</h5>

<!-- Good - logical hierarchy -->
<h1>Website Name</h1>
    <h2>Products</h2>
        <h3>Electronics</h3>
        <h3>Clothing</h3>
    <h2>About Us</h2>
        <h3>Our Team</h3>
        <h3>Our Mission</h3>
```

**Screen reader users often navigate by headings**, so:
- Use headings to create structure
- Don't skip levels (h1 → h3)
- Use CSS for styling, not heading levels

---

### 4. Link Text

Links should make sense out of context:

```html
<!-- Bad -->
<p>To learn more about our products, <a href="/products">click here</a>.</p>

<!-- Good -->
<p>Learn more about our <a href="/products">available products</a>.</p>

<!-- Bad -->
<a href="/article">Read more</a>
<a href="/article2">Read more</a>
<a href="/article3">Read more</a>

<!-- Good -->
<a href="/article">Read more about JavaScript basics</a>
<a href="/article2">Read more about CSS layouts</a>
<a href="/article3">Read more about React hooks</a>
```

---

### 5. Form Labels

Every form input needs an associated label:

```html
<!-- Bad - no label -->
<input type="email" placeholder="Email">

<!-- Good - explicit association -->
<label for="email">Email Address</label>
<input type="email" id="email" name="email">

<!-- Good - wrapping -->
<label>
    Email Address
    <input type="email" name="email">
</label>
```

**Additional form accessibility:**

```html
<form>
    <!-- Required fields -->
    <label for="name">Name <span aria-hidden="true">*</span></label>
    <input type="text" id="name" required aria-required="true">

    <!-- Error messages -->
    <label for="email">Email</label>
    <input type="email" id="email" aria-describedby="email-error" aria-invalid="true">
    <span id="email-error" role="alert">Please enter a valid email address</span>

    <!-- Help text -->
    <label for="password">Password</label>
    <input type="password" id="password" aria-describedby="password-help">
    <span id="password-help">Must be at least 8 characters</span>
</form>
```

---

### 6. Color and Contrast

#### Don't Rely on Color Alone

```html
<!-- Bad - only color indicates error -->
<input type="text" style="border-color: red;">

<!-- Good - color + icon + text -->
<input type="text" aria-invalid="true">
<span class="error">
    <svg aria-hidden="true"><!-- error icon --></svg>
    Please enter your name
</span>
```

#### Ensure Sufficient Contrast

Minimum contrast ratios (WCAG AA):
- Normal text: 4.5:1
- Large text (18pt+ or 14pt+ bold): 3:1
- UI components and graphics: 3:1

```css
/* Bad - low contrast */
.text {
    color: #aaa;        /* light gray */
    background: #fff;   /* white */
    /* Contrast ratio: 2.32:1 - FAILS */
}

/* Good - sufficient contrast */
.text {
    color: #595959;     /* darker gray */
    background: #fff;   /* white */
    /* Contrast ratio: 7:1 - PASSES */
}
```

**Test your colors:** Use tools like WebAIM Contrast Checker.

---

### 7. Keyboard Navigation

All interactive elements must be keyboard accessible:

```html
<!-- These are naturally keyboard accessible -->
<a href="/page">Link</a>
<button>Button</button>
<input type="text">
<select>...</select>

<!-- This is NOT keyboard accessible -->
<div onclick="doSomething()">Click me</div>

<!-- Make it accessible -->
<div role="button" tabindex="0" onclick="doSomething()"
     onkeydown="if(event.key==='Enter') doSomething()">
    Click me
</div>

<!-- Or better, just use a button! -->
<button onclick="doSomething()">Click me</button>
```

**Keyboard Navigation Basics:**
- `Tab` - Move to next focusable element
- `Shift + Tab` - Move to previous element
- `Enter` - Activate buttons/links
- `Space` - Activate buttons, check checkboxes
- `Arrow keys` - Navigate within components

**Focus Indicators:**

```css
/* Don't remove focus outlines! */
/* Bad */
*:focus {
    outline: none;
}

/* Good - customize but don't remove */
*:focus {
    outline: 2px solid #007bff;
    outline-offset: 2px;
}

/* Or use focus-visible for modern browsers */
*:focus-visible {
    outline: 2px solid #007bff;
    outline-offset: 2px;
}
```

---

### 8. Skip Links

Allow keyboard users to skip repetitive content:

```html
<body>
    <a href="#main-content" class="skip-link">Skip to main content</a>

    <header>
        <nav><!-- lots of links --></nav>
    </header>

    <main id="main-content">
        <!-- Main content here -->
    </main>
</body>
```

```css
.skip-link {
    position: absolute;
    top: -40px;
    left: 0;
    padding: 8px;
    background: #000;
    color: #fff;
    z-index: 100;
}

.skip-link:focus {
    top: 0;
}
```

---

## ARIA (Accessible Rich Internet Applications)

ARIA provides additional accessibility information when HTML alone isn't enough.

### Rule #1: Don't Use ARIA If You Don't Need It

Native HTML is always preferred:

```html
<!-- Don't do this -->
<div role="button" tabindex="0">Submit</div>

<!-- Do this -->
<button>Submit</button>
```

### Common ARIA Attributes

#### `aria-label`
Provides an accessible name:

```html
<button aria-label="Close dialog">
    <svg><!-- X icon --></svg>
</button>

<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer navigation">...</nav>
```

#### `aria-labelledby`
References another element for the label:

```html
<div role="dialog" aria-labelledby="dialog-title">
    <h2 id="dialog-title">Confirm Delete</h2>
    <p>Are you sure you want to delete this item?</p>
</div>
```

#### `aria-describedby`
Provides additional description:

```html
<input type="password" aria-describedby="password-requirements">
<p id="password-requirements">Password must be at least 8 characters</p>
```

#### `aria-hidden`
Hides content from screen readers:

```html
<!-- Decorative icon -->
<button>
    <span aria-hidden="true">🔍</span>
    Search
</button>

<!-- Redundant text -->
<a href="/">
    <img src="logo.png" alt="Acme Corp">
    <span aria-hidden="true">Acme Corp</span>
</a>
```

#### `aria-live`
Announces dynamic content:

```html
<!-- Polite - waits for user to finish -->
<div aria-live="polite">
    3 items added to cart
</div>

<!-- Assertive - interrupts immediately -->
<div role="alert" aria-live="assertive">
    Error: Payment failed
</div>
```

#### `aria-expanded` and `aria-controls`
For expandable content:

```html
<button aria-expanded="false" aria-controls="menu">
    Menu
</button>
<ul id="menu" hidden>
    <li>Option 1</li>
    <li>Option 2</li>
</ul>
```

#### `role`
Defines the element's purpose:

```html
<!-- Navigation landmarks -->
<div role="navigation">...</div>
<div role="main">...</div>
<div role="search">...</div>

<!-- Widgets -->
<div role="tablist">
    <button role="tab" aria-selected="true">Tab 1</button>
    <button role="tab" aria-selected="false">Tab 2</button>
</div>
<div role="tabpanel">Tab content</div>

<!-- Alerts -->
<div role="alert">Form submitted successfully!</div>
```

---

## Testing Accessibility

### 1. Keyboard Testing
- Navigate your site using only the keyboard
- Can you reach all interactive elements?
- Is focus visible?
- Can you operate all components?

### 2. Screen Reader Testing
Free screen readers:
- **NVDA** (Windows) - Free download
- **VoiceOver** (Mac/iOS) - Built-in
- **TalkBack** (Android) - Built-in

### 3. Automated Tools
- **WAVE** - Browser extension
- **axe DevTools** - Browser extension
- **Lighthouse** - Built into Chrome DevTools
- **HTML Validator** - validator.w3.org

### 4. Manual Checklist
- [ ] All images have alt text
- [ ] All form inputs have labels
- [ ] Color contrast is sufficient
- [ ] Headings are in logical order
- [ ] Links are descriptive
- [ ] Focus is visible
- [ ] Site works with keyboard only

---

## Accessibility Checklist

### HTML Structure
- [ ] Use semantic HTML elements
- [ ] Maintain logical heading hierarchy
- [ ] Use landmarks (header, nav, main, footer)
- [ ] Validate your HTML

### Images and Media
- [ ] Meaningful images have descriptive alt text
- [ ] Decorative images have empty alt (`alt=""`)
- [ ] Videos have captions
- [ ] Audio has transcripts

### Forms
- [ ] All inputs have labels
- [ ] Required fields are indicated
- [ ] Error messages are clear and helpful
- [ ] Form can be completed with keyboard

### Navigation
- [ ] Skip link to main content
- [ ] Focus is visible
- [ ] Tab order is logical
- [ ] Links are descriptive

### Color and Visual
- [ ] Contrast ratios meet minimums
- [ ] Color isn't the only indicator
- [ ] Text can be resized to 200%

---

## Practice Exercise

Audit your "About Me" page for accessibility:

1. **Run an automated test** using WAVE or Lighthouse
2. **Navigate with keyboard only** - Can you access everything?
3. **Check all images** - Do they have appropriate alt text?
4. **Verify heading structure** - Is it logical?
5. **Test color contrast** - Do all text/backgrounds pass?
6. **Add a skip link** to the main content

---

## Key Takeaways

1. **Accessibility is for everyone** - It helps all users
2. **Use semantic HTML** - It's the foundation of accessibility
3. **Always label form inputs** - Required for screen readers
4. **Don't rely on color alone** - Use text, icons, and patterns too
5. **Test with keyboard** - If you can't tab to it, it's not accessible
6. **Use ARIA sparingly** - Only when HTML isn't enough

---

## Resources

- [WebAIM](https://webaim.org/) - Accessibility training and tools
- [A11Y Project](https://www.a11yproject.com/) - Accessibility guides
- [WAVE Tool](https://wave.webaim.org/) - Accessibility checker
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/) - Official standards

---

**Week 1 Complete!**

You've learned:
- HTML document structure
- Text, links, images, and lists
- Semantic HTML and forms
- Web accessibility fundamentals

**Next:** [Week 2 - CSS Fundamentals](./week-02-day-01-css-basics.md)
