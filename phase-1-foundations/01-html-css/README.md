# HTML & CSS

**Duration:** 2-3 weeks

## Learning Objectives

By the end of this section, you will:
- Write semantic, accessible HTML
- Style layouts with Flexbox and Grid
- Build responsive, mobile-first designs
- Understand CSS specificity and the cascade
- Follow CSS best practices (BEM naming)

---

## Day-by-Day Lessons

Complete internal learning documentation - learn everything without leaving this repo!

### Week 1: HTML Fundamentals
- [Day 1: HTML Basics - Document Structure & Text](./lessons/day-01-html-basics.md)
- [Day 2: Links, Images, and Lists](./lessons/day-02-links-images-lists.md)
- [Day 3-4: Semantic HTML & Forms](./lessons/day-03-04-semantic-html-forms.md)
- [Day 5: Web Accessibility (a11y)](./lessons/day-05-accessibility.md)

### Week 2: CSS Fundamentals
- [Day 1: CSS Basics - Selectors, Box Model, Colors](./lessons/week-02-day-01-css-basics.md)
- [Day 2: Selectors & Specificity](./lessons/week-02-day-02-selectors-specificity.md)
- [Day 3-4: Flexbox](./lessons/week-02-day-03-04-flexbox.md)
- [Day 5: CSS Grid](./lessons/week-02-day-05-grid.md)

### Week 3: Responsive Design & Best Practices
- Coming soon: Responsive Design & Media Queries
- Coming soon: CSS Architecture & BEM
- Coming soon: Modern CSS Features

---

## Week 1: HTML Fundamentals

### Day 1-2: HTML Basics
**Topics:**
- Document structure (`<!DOCTYPE>`, `<html>`, `<head>`, `<body>`)
- Text elements (`<h1>`-`<h6>`, `<p>`, `<span>`, `<strong>`, `<em>`)
- Lists (`<ul>`, `<ol>`, `<li>`)
- Links (`<a>`) and images (`<img>`)
- Attributes (id, class, src, href, alt)

**Exercise:** Create a simple "About Me" page with:
- [ ] Proper document structure
- [ ] At least 3 headings
- [ ] Paragraphs of text
- [ ] An unordered list of hobbies
- [ ] Links to social profiles
- [ ] A profile image

### Day 3-4: Semantic HTML
**Topics:**
- Semantic elements (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`)
- Forms (`<form>`, `<input>`, `<label>`, `<button>`, `<select>`, `<textarea>`)
- Tables (`<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>`)
- Media (`<video>`, `<audio>`, `<picture>`)

**Exercise:** Build a blog post page with:
- [ ] Header with navigation
- [ ] Article with proper semantic structure
- [ ] Sidebar with related links
- [ ] Comment form
- [ ] Footer

### Day 5: Accessibility (a11y)
**Topics:**
- Why accessibility matters
- ARIA attributes
- Alt text best practices
- Keyboard navigation
- Screen reader basics

**Exercise:** Audit your previous exercises for accessibility:
- [ ] Add proper alt text to all images
- [ ] Ensure form labels are connected to inputs
- [ ] Test keyboard navigation
- [ ] Add ARIA labels where needed

---

## Week 2: CSS Fundamentals

### Day 1: CSS Basics
**Topics:**
- Selectors (element, class, id, attribute)
- Properties and values
- The box model (margin, border, padding, content)
- Display property (block, inline, inline-block, none)
- Colors (hex, rgb, hsl)
- Units (px, em, rem, %, vh, vw)

**Exercise:** Style your "About Me" page:
- [ ] Add a color scheme
- [ ] Style typography
- [ ] Add spacing with margin/padding
- [ ] Create a centered container

### Day 2: Selectors & Specificity
**Topics:**
- Combinators (descendant, child, sibling)
- Pseudo-classes (`:hover`, `:focus`, `:first-child`, `:nth-child`)
- Pseudo-elements (`::before`, `::after`)
- Specificity calculation
- The cascade and inheritance
- `!important` (and why to avoid it)

**Exercise:** Create a styled navigation menu:
- [ ] Horizontal layout
- [ ] Hover effects
- [ ] Active state styling
- [ ] Dropdown menu (CSS only)

### Day 3-4: Flexbox
**Topics:**
- Flex container properties (`display: flex`, `flex-direction`, `justify-content`, `align-items`, `flex-wrap`, `gap`)
- Flex item properties (`flex-grow`, `flex-shrink`, `flex-basis`, `order`, `align-self`)
- Common patterns (centering, navigation, cards)

**Exercise:** Build a card layout:
- [ ] Row of 3 cards
- [ ] Cards have image, title, text, button
- [ ] Equal height cards
- [ ] Proper spacing with gap

### Day 5: CSS Grid
**Topics:**
- Grid container properties (`display: grid`, `grid-template-columns`, `grid-template-rows`, `gap`)
- Grid item properties (`grid-column`, `grid-row`, `grid-area`)
- `fr` units and `repeat()`
- `minmax()` and `auto-fit`/`auto-fill`
- Named grid areas

**Exercise:** Create a page layout with Grid:
- [ ] Header spanning full width
- [ ] Main content area
- [ ] Sidebar
- [ ] Footer spanning full width

---

## Week 3: Responsive Design & Best Practices

### Day 1-2: Responsive Design
**Topics:**
- Mobile-first approach
- Media queries (`@media`)
- Breakpoints strategy
- Responsive images (`srcset`, `<picture>`)
- Viewport meta tag
- Responsive typography

**Exercise:** Make all previous exercises responsive:
- [ ] Mobile layout (< 768px)
- [ ] Tablet layout (768px - 1024px)
- [ ] Desktop layout (> 1024px)
- [ ] Test on real devices

### Day 3: CSS Architecture
**Topics:**
- BEM naming convention (Block__Element--Modifier)
- Organizing CSS files
- CSS custom properties (variables)
- DRY principles

**Exercise:** Refactor your CSS using BEM:
- [ ] Rename all classes to BEM format
- [ ] Create CSS variables for colors and spacing
- [ ] Organize into logical sections

### Day 4-5: Modern CSS Features
**Topics:**
- CSS transitions
- CSS animations (`@keyframes`)
- Transforms (translate, rotate, scale)
- Filters and blend modes
- CSS functions (`calc()`, `clamp()`, `min()`, `max()`)

**Exercise:** Add polish to your portfolio:
- [ ] Smooth hover transitions
- [ ] Loading animation
- [ ] Scroll-triggered effects (CSS only)

---

## Projects

### Mini Project 1: Responsive Landing Page
Build a product landing page with:
- Hero section with CTA
- Features grid
- Testimonials
- Pricing cards
- Contact form
- Mobile responsive

### Mini Project 2: CSS Art Challenge
Create a simple illustration using only CSS:
- No images
- Pure CSS shapes
- Bonus: Add animation

---

## Resources

### Documentation
- [MDN HTML Reference](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [MDN CSS Reference](https://developer.mozilla.org/en-US/docs/Web/CSS)

### Interactive Learning
- [Flexbox Froggy](https://flexboxfroggy.com/)
- [Grid Garden](https://cssgridgarden.com/)
- [CSS Diner](https://flukeout.github.io/)

### Design Inspiration
- [Dribbble](https://dribbble.com/)
- [Awwwards](https://www.awwwards.com/)

---

## Checklist Before Moving On

- [ ] Can build semantic HTML documents from scratch
- [ ] Understand the box model completely
- [ ] Can create layouts with Flexbox
- [ ] Can create layouts with Grid
- [ ] Can build mobile-first responsive designs
- [ ] Understand CSS specificity
- [ ] Using BEM naming convention
- [ ] Completed both mini projects

---

**Next:** [JavaScript](../02-javascript/README.md)
