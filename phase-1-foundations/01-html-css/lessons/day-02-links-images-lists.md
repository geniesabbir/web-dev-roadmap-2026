# Day 2: Links, Images, and Lists

> **Estimated Time:** 4-5 hours | **Difficulty:** Beginner | **Prerequisites:** Day 1 - HTML Basics

---

## Learning Objectives

By the end of this lesson, you will be able to:

- [ ] Create and use all types of hyperlinks (external, internal, anchor, email, tel)
- [ ] Implement proper link accessibility with ARIA attributes
- [ ] Add images with proper alt text following accessibility best practices
- [ ] Use modern image formats (WebP, AVIF) with fallbacks
- [ ] Implement responsive images using `srcset` and `sizes`
- [ ] Optimize images for Core Web Vitals and LCP performance
- [ ] Create and structure all list types (ordered, unordered, description)
- [ ] Build navigation menus using semantic HTML
- [ ] Understand and apply WCAG 2.2 accessibility standards

---

## Table of Contents

1. [Links (Anchor Elements)](#links-anchor-elements)
2. [Link Types and Use Cases](#link-types-and-use-cases)
3. [Link Accessibility Best Practices](#link-accessibility-best-practices)
4. [Images](#images)
5. [Modern Image Formats (2025)](#modern-image-formats-2025)
6. [Responsive Images](#responsive-images)
7. [Image Performance Optimization](#image-performance-optimization)
8. [Writing Accessible Alt Text](#writing-accessible-alt-text)
9. [Lists](#lists)
10. [Building Navigation with Lists](#building-navigation-with-lists)
11. [Combining Everything](#combining-everything)
12. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
13. [Live Examples](#live-examples)
14. [Assignments](#assignments)
15. [Mini Projects](#mini-projects)
16. [Quick Reference](#quick-reference)
17. [Further Learning](#further-learning)

---

## Links (Anchor Elements)

Links are the backbone of the web—they connect pages together. The `<a>` (anchor) element creates hyperlinks that users can click to navigate.

### Basic Link Anatomy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  <a href="https://example.com" target="_blank">Visit Example</a>        │
│   ↑    ↑                        ↑                  ↑           ↑        │
│   │    │                        │                  │           │        │
│   │    │                        │                  │     Closing Tag    │
│   │    │                        │                  │                    │
│   │    │                        │            Link Text (Visible)        │
│   │    │                        │                                       │
│   │    │                   Target Attribute                             │
│   │    │                                                                │
│   │    └─ Hypertext Reference (URL)                                     │
│   │                                                                     │
│   └─ Opening Tag (Anchor Element)                                       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Basic Link Syntax

```html
<a href="https://www.example.com">Click here to visit Example</a>
```

| Part | Purpose |
|------|---------|
| `<a>` | The anchor tag - creates a clickable link |
| `href` | Hypertext Reference - the destination URL |
| Link text | What users see and click on |

**Preview (Browser Output):**

> [Click here to visit Example](#) ← This would be a clickable blue underlined link

---

## Link Types and Use Cases

### 1. External Links (Other Websites)

Links to pages outside your website:

```html
<!-- Basic external link -->
<a href="https://www.google.com">Visit Google</a>

<!-- Always use HTTPS when possible for security -->
<a href="https://github.com">GitHub</a>

<!-- External link opening in new tab (with security) -->
<a href="https://developer.mozilla.org" target="_blank" rel="noopener noreferrer">
    MDN Web Docs
</a>
```

**Preview:**
> [Visit Google](#) | [GitHub](#) | [MDN Web Docs](#) ↗

---

### 2. Internal Links (Same Website)

Links to other pages within your website:

```html
<!-- Same folder -->
<a href="about.html">About Us</a>

<!-- Subfolder -->
<a href="pages/contact.html">Contact</a>

<!-- Parent folder -->
<a href="../index.html">Back to Home</a>

<!-- Root-relative (from website root) -->
<a href="/products/item.html">View Product</a>
```

### Path Types Explained

```
Your Website Structure:
─────────────────────────────────────────────
my-website/
├── index.html          ← You are here
├── about.html
├── pages/
│   ├── contact.html
│   └── services.html
├── products/
│   └── item.html
└── assets/
    └── images/
─────────────────────────────────────────────
```

| Path Type | Syntax | From `index.html` | Goes To |
|-----------|--------|-------------------|---------|
| Same folder | `about.html` | `href="about.html"` | `/my-website/about.html` |
| Subfolder | `folder/file.html` | `href="pages/contact.html"` | `/my-website/pages/contact.html` |
| Parent folder | `../file.html` | N/A from root | Goes up one level |
| Root-relative | `/file.html` | `href="/products/item.html"` | Always from domain root |

---

### 3. Anchor Links (Same Page Navigation)

Jump to specific sections on the current page:

```html
<!-- Navigation links at top -->
<nav>
    <a href="#introduction">Introduction</a>
    <a href="#features">Features</a>
    <a href="#pricing">Pricing</a>
    <a href="#contact">Contact Us</a>
</nav>

<!-- Target sections (where links jump to) -->
<section id="introduction">
    <h2>Introduction</h2>
    <p>Welcome to our product...</p>
</section>

<section id="features">
    <h2>Features</h2>
    <p>Our amazing features include...</p>
</section>

<section id="pricing">
    <h2>Pricing</h2>
    <p>Affordable plans for everyone...</p>
</section>

<section id="contact">
    <h2>Contact Us</h2>
    <p>Get in touch...</p>
</section>

<!-- Back to top link -->
<a href="#top">↑ Back to Top</a>
```

**How it works:**
- `href="#section-id"` - The `#` targets an element's `id` attribute
- The browser scrolls to that element
- Modern browsers support smooth scrolling via CSS

---

### 4. Email and Telephone Links

```html
<!-- Email link - opens default email client -->
<a href="mailto:contact@example.com">Email Us</a>

<!-- Email with pre-filled subject and body -->
<a href="mailto:contact@example.com?subject=Hello&body=I would like to inquire about...">
    Send Inquiry
</a>

<!-- Multiple recipients -->
<a href="mailto:sales@example.com,support@example.com?subject=General Question">
    Contact Team
</a>

<!-- Phone link - essential for mobile users! -->
<a href="tel:+1-555-123-4567">Call: (555) 123-4567</a>

<!-- SMS link -->
<a href="sms:+1-555-123-4567">Send Text Message</a>

<!-- WhatsApp link -->
<a href="https://wa.me/15551234567">Chat on WhatsApp</a>
```

**Preview:**
> 📧 [Email Us](#) | 📞 [Call: (555) 123-4567](#) | 💬 [Send Text Message](#)

---

### 5. Download Links

Force the browser to download a file instead of opening it:

```html
<!-- Basic download -->
<a href="files/document.pdf" download>Download PDF</a>

<!-- Download with custom filename -->
<a href="files/report-2024-q4.pdf" download="quarterly-report.pdf">
    Download Q4 Report
</a>

<!-- Download various file types -->
<a href="files/data.csv" download>Download Spreadsheet (CSV)</a>
<a href="files/presentation.pptx" download>Download Presentation</a>
<a href="files/archive.zip" download>Download Archive (ZIP)</a>
```

---

## Link Attributes Reference

### Essential Attributes

| Attribute | Values | Purpose |
|-----------|--------|---------|
| `href` | URL, path, `#id`, `mailto:`, `tel:` | Link destination |
| `target` | `_self`, `_blank`, `_parent`, `_top` | Where to open |
| `rel` | `noopener`, `noreferrer`, `nofollow`, etc. | Relationship to target |
| `download` | filename (optional) | Force download |
| `title` | text | Tooltip on hover |

### Target Values Explained

```html
<!-- Default: Same tab/window -->
<a href="page.html" target="_self">Same Tab (Default)</a>

<!-- New tab/window -->
<a href="https://external.com" target="_blank" rel="noopener noreferrer">
    New Tab
</a>

<!-- Parent frame (for iframes) -->
<a href="page.html" target="_parent">Parent Frame</a>

<!-- Full window (breaks out of all frames) -->
<a href="page.html" target="_top">Full Window</a>
```

### Security: The `rel` Attribute

**CRITICAL:** Always add `rel="noopener noreferrer"` when using `target="_blank"`:

```html
<!-- ❌ UNSAFE - Potential security vulnerability -->
<a href="https://malicious-site.com" target="_blank">External Link</a>

<!-- ✅ SAFE - Prevents tabnapping attack -->
<a href="https://external-site.com" target="_blank" rel="noopener noreferrer">
    External Link
</a>
```

**Why?**
- `noopener` - Prevents the new page from accessing `window.opener`
- `noreferrer` - Prevents sending the referrer header (privacy)
- Without these, malicious sites can redirect your original page!

### Additional `rel` Values

| Value | Purpose |
|-------|---------|
| `nofollow` | Tell search engines not to follow this link (ads, user-generated content) |
| `sponsored` | Mark paid/sponsored links |
| `ugc` | User-generated content links |
| `external` | Indicates link goes to external site |

```html
<!-- Sponsored content -->
<a href="https://sponsor.com" rel="noopener noreferrer sponsored">
    Our Sponsor
</a>

<!-- User-generated link (e.g., blog comments) -->
<a href="https://user-link.com" rel="noopener noreferrer ugc nofollow">
    User's Website
</a>
```

---

## Link Accessibility Best Practices

Following **WCAG 2.2** guidelines for accessible links:

### 1. Descriptive Link Text

The most important rule: **Link text should describe the destination, not the action.**

```html
<!-- ❌ BAD: Non-descriptive (never use these!) -->
<p>To learn more about our services, <a href="services.html">click here</a>.</p>
<p><a href="report.pdf">Read more</a></p>
<p><a href="https://example.com">https://example.com</a></p>

<!-- ✅ GOOD: Descriptive link text -->
<p>Learn more on our <a href="services.html">services page</a>.</p>
<p>Read the <a href="report.pdf">2024 Annual Report (PDF, 2.5MB)</a></p>
<p>Visit <a href="https://example.com">Example Company's website</a>.</p>
```

**Why it matters:**
- Screen readers often list all links on a page
- "Click here" repeated 10 times tells the user nothing
- Descriptive text helps everyone understand where links lead

---

### 2. ARIA Attributes for Enhanced Accessibility

Use ARIA when native HTML isn't sufficient:

```html
<!-- aria-label: Override visible text for screen readers -->
<a href="cart.html" aria-label="View shopping cart with 3 items">
    🛒 (3)
</a>

<!-- aria-describedby: Additional context -->
<p id="pdf-note">Opens in a new window. File size: 2.5MB</p>
<a href="report.pdf" target="_blank" rel="noopener" aria-describedby="pdf-note">
    Download Annual Report
</a>

<!-- aria-current: Indicate current page in navigation -->
<nav>
    <a href="/" aria-current="page">Home</a>
    <a href="/about">About</a>
    <a href="/contact">Contact</a>
</nav>
```

---

### 3. Skip Links for Keyboard Navigation

Essential for users who navigate with keyboards or assistive technology:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        .skip-link {
            position: absolute;
            top: -40px;
            left: 0;
            background: #000;
            color: #fff;
            padding: 8px 16px;
            z-index: 100;
            text-decoration: none;
        }
        .skip-link:focus {
            top: 0;
        }
    </style>
</head>
<body>
    <!-- Skip link - first element in body -->
    <a href="#main-content" class="skip-link">Skip to main content</a>

    <header>
        <nav><!-- Long navigation menu --></nav>
    </header>

    <main id="main-content">
        <!-- Main page content -->
    </main>
</body>
</html>
```

---

### 4. Touch Target Size (WCAG 2.2)

Ensure links are large enough to tap on mobile:

```html
<!-- Minimum 24x24 CSS pixels (WCAG 2.2 Level AA) -->
<!-- Recommended: 44x44 CSS pixels for comfortable tapping -->

<style>
    .touch-friendly-link {
        display: inline-block;
        padding: 12px 16px;
        min-height: 44px;
        min-width: 44px;
    }

    /* Add spacing between adjacent links */
    nav a {
        margin: 4px;
    }
</style>

<nav>
    <a href="/home" class="touch-friendly-link">Home</a>
    <a href="/about" class="touch-friendly-link">About</a>
    <a href="/contact" class="touch-friendly-link">Contact</a>
</nav>
```

---

### 5. Focus Indicators

Never remove focus outlines—enhance them instead:

```html
<style>
    /* ❌ NEVER DO THIS - Removes accessibility */
    a:focus {
        outline: none; /* BAD! */
    }

    /* ✅ GOOD - Enhanced focus indicator */
    a:focus {
        outline: 3px solid #005fcc;
        outline-offset: 2px;
    }

    /* ✅ BETTER - Visible focus with smooth transition */
    a:focus-visible {
        outline: 3px solid #005fcc;
        outline-offset: 2px;
        border-radius: 2px;
    }
</style>
```

---

## Images

Images make your pages visual and engaging. The `<img>` element embeds images into your HTML.

### Basic Image Syntax

```html
<img src="photo.jpg" alt="A golden retriever playing in a park">
```

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  <img src="photo.jpg" alt="Description" width="800" height="600" />     │
│    ↑       ↑              ↑               ↑            ↑                │
│    │       │              │               │            │                │
│    │       │              │               │       Height (pixels)       │
│    │       │              │               │                             │
│    │       │              │          Width (pixels)                     │
│    │       │              │                                             │
│    │       │        Alt text (REQUIRED!)                                │
│    │       │                                                            │
│    │    Image source (path or URL)                                      │
│    │                                                                    │
│    Self-closing image element                                           │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Essential Image Attributes

| Attribute | Required? | Purpose |
|-----------|-----------|---------|
| `src` | ✅ Yes | Path to the image file |
| `alt` | ✅ Yes | Alternative text for accessibility |
| `width` | Recommended | Intrinsic width (prevents layout shift) |
| `height` | Recommended | Intrinsic height (prevents layout shift) |
| `loading` | Optional | `lazy` or `eager` loading behavior |
| `decoding` | Optional | `async`, `sync`, or `auto` |
| `fetchpriority` | Optional | `high`, `low`, or `auto` (for LCP images) |

---

### Image Sources

```html
<!-- Local image: Same folder -->
<img src="photo.jpg" alt="My photo">

<!-- Local image: Images subfolder -->
<img src="images/hero.jpg" alt="Hero banner showing mountains at sunset">

<!-- Local image: Parent folder -->
<img src="../assets/logo.png" alt="Company logo">

<!-- External image (use sparingly) -->
<img src="https://example.com/image.jpg" alt="External image">
```

**⚠️ Warning about external images:**
- They can slow your page (no control over optimization)
- They may be removed or changed by the host
- They create a dependency on third-party servers
- Consider downloading and hosting them yourself

---

## Modern Image Formats (2025)

### Format Comparison

| Format | Best For | Compression | Transparency | Animation | Browser Support |
|--------|----------|-------------|--------------|-----------|-----------------|
| **JPEG** | Photos | Good | ❌ No | ❌ No | 100% |
| **PNG** | Graphics, logos | Lossless | ✅ Yes | ❌ No | 100% |
| **GIF** | Simple animations | Poor | ✅ Yes (1-bit) | ✅ Yes | 100% |
| **SVG** | Icons, logos | Vector (tiny) | ✅ Yes | ✅ Yes | 100% |
| **WebP** | Photos & graphics | Excellent (25-30% smaller than JPEG) | ✅ Yes | ✅ Yes | ~95% |
| **AVIF** | Photos & graphics | Superior (50% smaller than JPEG) | ✅ Yes | ✅ Yes | ~80% |

### The Modern Image Stack (2025)

**Strategy:** Lead with AVIF, fall back to WebP, keep JPEG as safety net.

```html
<!-- Modern image with format fallbacks -->
<picture>
    <!-- Best: AVIF (50% smaller than JPEG) -->
    <source srcset="image.avif" type="image/avif">

    <!-- Good: WebP (25-30% smaller than JPEG) -->
    <source srcset="image.webp" type="image/webp">

    <!-- Fallback: JPEG (universal support) -->
    <img src="image.jpg" alt="Description of the image" width="800" height="600">
</picture>
```

### When to Use Each Format

```
Decision Tree for Image Formats:
───────────────────────────────────────────────
Is it a logo, icon, or illustration with simple shapes?
├── YES → Use SVG (vector, infinitely scalable)
└── NO → Is it a photograph or complex image?
    ├── YES → Use AVIF → WebP → JPEG fallback chain
    └── NO → Is transparency needed?
        ├── YES → Use WebP or PNG
        └── NO → Use WebP or JPEG
───────────────────────────────────────────────
```

---

## Responsive Images

Responsive images serve different image sizes based on the user's device, screen size, and resolution.

### Why Responsive Images Matter

```
Desktop (1920px)          Tablet (768px)           Mobile (375px)
┌─────────────────┐      ┌───────────────┐        ┌─────────┐
│                 │      │               │        │         │
│   1920px wide   │      │  768px wide   │        │ 375px   │
│   ~400KB        │      │  ~150KB       │        │ ~50KB   │
│                 │      │               │        │         │
└─────────────────┘      └───────────────┘        └─────────┘

Without responsive images: Mobile downloads 400KB
With responsive images: Mobile downloads 50KB (8x smaller!)
```

### Using `srcset` and `sizes`

```html
<!-- Responsive image with multiple resolutions -->
<img
    src="image-800.jpg"
    srcset="
        image-400.jpg 400w,
        image-800.jpg 800w,
        image-1200.jpg 1200w,
        image-1600.jpg 1600w
    "
    sizes="
        (max-width: 600px) 100vw,
        (max-width: 1200px) 50vw,
        33vw
    "
    alt="A responsive image example"
    width="800"
    height="600"
    loading="lazy"
>
```

**Breaking it down:**

| Attribute | What it does |
|-----------|--------------|
| `src` | Fallback image for browsers that don't support srcset |
| `srcset` | List of available images with their widths (in `w` units) |
| `sizes` | Tells browser how wide the image will display at different viewport sizes |

**How `sizes` works:**
- `(max-width: 600px) 100vw` → On screens ≤600px, image is 100% viewport width
- `(max-width: 1200px) 50vw` → On screens ≤1200px, image is 50% viewport width
- `33vw` → Default: image is 33% viewport width

### Complete Responsive Image with Modern Formats

```html
<picture>
    <!-- AVIF format with responsive sizes -->
    <source
        type="image/avif"
        srcset="
            hero-400.avif 400w,
            hero-800.avif 800w,
            hero-1200.avif 1200w,
            hero-1600.avif 1600w
        "
        sizes="(max-width: 768px) 100vw, 50vw"
    >

    <!-- WebP format with responsive sizes -->
    <source
        type="image/webp"
        srcset="
            hero-400.webp 400w,
            hero-800.webp 800w,
            hero-1200.webp 1200w,
            hero-1600.webp 1600w
        "
        sizes="(max-width: 768px) 100vw, 50vw"
    >

    <!-- JPEG fallback with responsive sizes -->
    <img
        src="hero-800.jpg"
        srcset="
            hero-400.jpg 400w,
            hero-800.jpg 800w,
            hero-1200.jpg 1200w,
            hero-1600.jpg 1600w
        "
        sizes="(max-width: 768px) 100vw, 50vw"
        alt="Hero image showing our product"
        width="1600"
        height="900"
        loading="eager"
        fetchpriority="high"
        decoding="async"
    >
</picture>
```

### Art Direction with `<picture>`

Use different images (not just sizes) for different viewports:

```html
<picture>
    <!-- Mobile: Cropped vertical image -->
    <source
        media="(max-width: 600px)"
        srcset="hero-mobile.jpg"
    >

    <!-- Tablet: Square crop -->
    <source
        media="(max-width: 1024px)"
        srcset="hero-tablet.jpg"
    >

    <!-- Desktop: Wide panoramic -->
    <img src="hero-desktop.jpg" alt="Hero image" width="1920" height="600">
</picture>
```

---

## Image Performance Optimization

### Core Web Vitals: LCP (Largest Contentful Paint)

The LCP metric measures when the largest content element becomes visible. Often, this is your hero image.

**Target:** LCP should be ≤ 2.5 seconds

### The `fetchpriority` Attribute (2025)

Control how the browser prioritizes loading images:

```html
<!-- ✅ HIGH PRIORITY: Your LCP/hero image (above the fold) -->
<img
    src="hero.jpg"
    alt="Hero banner"
    fetchpriority="high"
    loading="eager"
    decoding="async"
>

<!-- 📦 AUTO (Default): Regular content images -->
<img src="product.jpg" alt="Product" fetchpriority="auto">

<!-- ⬇️ LOW PRIORITY: Below-the-fold, decorative, or non-critical images -->
<img
    src="decoration.jpg"
    alt=""
    fetchpriority="low"
    loading="lazy"
>
```

**Rules for `fetchpriority`:**
- Use `high` for **only ONE image** (your LCP element)
- Multiple `high` priority images compete and hurt performance
- Use `low` for images that can wait
- Browser support: Chrome 102+, Safari 17.2+, Firefox 132+

### Lazy Loading

Defer loading images until they're about to enter the viewport:

```html
<!-- Lazy load images below the fold -->
<img
    src="below-fold.jpg"
    alt="Content image"
    loading="lazy"
    decoding="async"
    width="800"
    height="600"
>

<!-- NEVER lazy load your LCP/hero image! -->
<img
    src="hero.jpg"
    alt="Hero banner"
    loading="eager"
    fetchpriority="high"
>
```

### Preventing Layout Shift (CLS)

Always specify width and height to reserve space:

```html
<!-- ❌ BAD: Causes layout shift -->
<img src="photo.jpg" alt="Photo">

<!-- ✅ GOOD: Dimensions prevent layout shift -->
<img src="photo.jpg" alt="Photo" width="800" height="600">

<!-- ✅ ALSO GOOD: Using aspect-ratio in CSS -->
<style>
    .responsive-image {
        width: 100%;
        height: auto;
        aspect-ratio: 16 / 9;
        object-fit: cover;
    }
</style>
<img src="photo.jpg" alt="Photo" class="responsive-image">
```

### Preloading Critical Images

For images discovered late (CSS backgrounds, JS-loaded):

```html
<head>
    <!-- Preload LCP image for faster loading -->
    <link
        rel="preload"
        as="image"
        href="hero.webp"
        type="image/webp"
        fetchpriority="high"
    >

    <!-- Preload with responsive images -->
    <link
        rel="preload"
        as="image"
        href="hero.jpg"
        imagesrcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1200.jpg 1200w"
        imagesizes="100vw"
    >
</head>
```

---

## Writing Accessible Alt Text

Alt text is **REQUIRED** for accessibility. It's announced by screen readers and displayed when images fail to load.

### Alt Text Decision Tree

```
Is the image purely decorative (adds no information)?
├── YES → Use empty alt: alt=""
└── NO → Does the image contain text?
    ├── YES → Alt text should include the text
    └── NO → Does it convey information?
        ├── YES → Describe the information it conveys
        └── NO → Is it a link/button?
            ├── YES → Describe the action/destination
            └── NO → Describe the image content concisely
```

### Alt Text Examples

```html
<!-- ❌ BAD: Missing alt -->
<img src="dog.jpg">

<!-- ❌ BAD: Useless alt text -->
<img src="dog.jpg" alt="image">
<img src="dog.jpg" alt="dog.jpg">
<img src="dog.jpg" alt="photo">

<!-- ❌ BAD: Redundant "image of" -->
<img src="dog.jpg" alt="Image of a dog">
<img src="dog.jpg" alt="Photo of a dog in a park">

<!-- ❌ BAD: Too long and detailed -->
<img src="dog.jpg" alt="A photograph of a golden retriever dog with
light fur that is sitting in a green grassy field on a sunny summer
day with blue skies and white clouds in the background looking
directly at the camera with its tongue out appearing very happy">

<!-- ✅ GOOD: Concise and descriptive -->
<img src="dog.jpg" alt="Golden retriever sitting in a sunny park">

<!-- ✅ GOOD: Context-appropriate -->
<img src="product.jpg" alt="Blue Nike Air Max shoes, size 10, $129.99">

<!-- ✅ GOOD: Decorative image (empty alt) -->
<img src="decorative-border.png" alt="">

<!-- ✅ GOOD: Logo with company name -->
<img src="logo.png" alt="Acme Corporation">

<!-- ✅ GOOD: Functional image (describes action) -->
<a href="https://twitter.com/company">
    <img src="twitter-icon.png" alt="Follow us on Twitter">
</a>

<!-- ✅ GOOD: Chart with data -->
<img src="sales-chart.png" alt="Bar chart showing Q1 sales at $50K,
Q2 at $75K, Q3 at $60K, and Q4 at $90K">
```

### Complex Images

For complex images like infographics, provide extended descriptions:

```html
<!-- Method 1: Using aria-describedby -->
<figure>
    <img
        src="infographic.png"
        alt="2024 Web Development Survey Results"
        aria-describedby="infographic-desc"
    >
    <figcaption id="infographic-desc">
        Survey of 10,000 developers showing:
        React leads at 40%, Vue at 25%, Angular at 20%,
        and other frameworks at 15%.
    </figcaption>
</figure>

<!-- Method 2: Link to full description -->
<img src="complex-diagram.png" alt="System architecture diagram">
<p><a href="architecture-description.html">View full description of system architecture</a></p>
```

---

## Figure and Figcaption

Use `<figure>` to associate images with captions:

```html
<!-- Image with caption -->
<figure>
    <img
        src="chart.png"
        alt="Line chart showing 50% revenue growth from 2023 to 2024"
        width="800"
        height="400"
    >
    <figcaption>Figure 1: Quarterly revenue growth 2023-2024</figcaption>
</figure>

<!-- Multiple images in one figure -->
<figure>
    <img src="before.jpg" alt="Website design before redesign">
    <img src="after.jpg" alt="Website design after redesign">
    <figcaption>Before and after: Our website redesign comparison</figcaption>
</figure>

<!-- Code listing with figure -->
<figure>
    <pre><code>function greet(name) {
    return `Hello, ${name}!`;
}</code></pre>
    <figcaption>Listing 1: A simple greeting function in JavaScript</figcaption>
</figure>
```

---

## Lists

Lists organize content into easy-to-read formats. HTML provides three types of lists.

### Unordered Lists (Bullet Points)

For items where order **doesn't matter**:

```html
<h3>Shopping List</h3>
<ul>
    <li>Milk</li>
    <li>Bread</li>
    <li>Eggs</li>
    <li>Cheese</li>
</ul>
```

**Preview (Browser Output):**
> • Milk
> • Bread
> • Eggs
> • Cheese

**Use for:** Navigation menus, feature lists, ingredient lists, any collection without sequence

---

### Ordered Lists (Numbered)

For items where order **matters**:

```html
<h3>Recipe Instructions</h3>
<ol>
    <li>Preheat oven to 350°F (175°C)</li>
    <li>Mix flour, sugar, and salt in a bowl</li>
    <li>Add eggs and butter, mix until smooth</li>
    <li>Pour into greased baking pan</li>
    <li>Bake for 25-30 minutes until golden</li>
</ol>
```

**Preview (Browser Output):**
> 1. Preheat oven to 350°F (175°C)
> 2. Mix flour, sugar, and salt in a bowl
> 3. Add eggs and butter, mix until smooth
> 4. Pour into greased baking pan
> 5. Bake for 25-30 minutes until golden

**Use for:** Step-by-step instructions, rankings, procedures, sequences

---

### Ordered List Attributes

```html
<!-- Start from a different number -->
<ol start="5">
    <li>Fifth item</li>
    <li>Sixth item</li>
</ol>

<!-- Reverse order (countdown) -->
<ol reversed>
    <li>Bronze medal</li>
    <li>Silver medal</li>
    <li>Gold medal</li>
</ol>

<!-- Different numbering types -->
<ol type="A">  <!-- A, B, C, D... -->
    <li>First item</li>
    <li>Second item</li>
</ol>

<ol type="a">  <!-- a, b, c, d... -->
    <li>First item</li>
    <li>Second item</li>
</ol>

<ol type="I">  <!-- I, II, III, IV... (Roman numerals) -->
    <li>First item</li>
    <li>Second item</li>
</ol>

<ol type="i">  <!-- i, ii, iii, iv... (lowercase Roman) -->
    <li>First item</li>
    <li>Second item</li>
</ol>

<!-- Individual item value -->
<ol>
    <li>First</li>
    <li value="10">This is number 10</li>
    <li>This continues as 11</li>
</ol>
```

---

### Description Lists

For term-definition pairs (glossaries, metadata, key-value pairs):

```html
<h3>Web Development Glossary</h3>
<dl>
    <dt>HTML</dt>
    <dd>HyperText Markup Language - defines the structure of web content</dd>

    <dt>CSS</dt>
    <dd>Cascading Style Sheets - controls the visual presentation</dd>

    <dt>JavaScript</dt>
    <dd>Programming language that adds interactivity to websites</dd>
    <dd>Can run in browsers and on servers (Node.js)</dd>
</dl>
```

| Element | Purpose |
|---------|---------|
| `<dl>` | Description list container |
| `<dt>` | Description term (the word/phrase being defined) |
| `<dd>` | Description details (the definition/explanation) |

**Use for:** Glossaries, FAQs, metadata displays, product specifications

---

### Nested Lists

Lists can contain other lists for hierarchical content:

```html
<h3>Full-Stack Developer Roadmap</h3>
<ul>
    <li>Frontend Development
        <ul>
            <li>HTML & CSS
                <ul>
                    <li>Semantic HTML</li>
                    <li>Responsive Design</li>
                    <li>CSS Grid & Flexbox</li>
                </ul>
            </li>
            <li>JavaScript
                <ul>
                    <li>ES6+ Features</li>
                    <li>DOM Manipulation</li>
                    <li>Async Programming</li>
                </ul>
            </li>
            <li>React.js</li>
        </ul>
    </li>
    <li>Backend Development
        <ul>
            <li>Node.js & Express</li>
            <li>Databases (PostgreSQL, MongoDB)</li>
            <li>REST APIs</li>
            <li>Authentication</li>
        </ul>
    </li>
    <li>DevOps
        <ul>
            <li>Git & GitHub</li>
            <li>Docker</li>
            <li>CI/CD</li>
        </ul>
    </li>
</ul>
```

**Preview (Browser Output):**
> • Frontend Development
>   • HTML & CSS
>     • Semantic HTML
>     • Responsive Design
>     • CSS Grid & Flexbox
>   • JavaScript
>     • ES6+ Features
>     • DOM Manipulation
>     • Async Programming
>   • React.js
> • Backend Development
>   • Node.js & Express
>   • Databases (PostgreSQL, MongoDB)
>   • REST APIs
>   • Authentication
> • DevOps
>   • Git & GitHub
>   • Docker
>   • CI/CD

---

## Building Navigation with Lists

Navigation menus should be built using semantic HTML lists:

### Basic Navigation

```html
<nav aria-label="Main navigation">
    <ul>
        <li><a href="/" aria-current="page">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/services">Services</a></li>
        <li><a href="/portfolio">Portfolio</a></li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>
```

### Navigation with Dropdowns

```html
<nav aria-label="Main navigation">
    <ul>
        <li><a href="/">Home</a></li>
        <li>
            <a href="/products">Products</a>
            <ul>
                <li><a href="/products/software">Software</a></li>
                <li><a href="/products/hardware">Hardware</a></li>
                <li><a href="/products/services">Services</a></li>
            </ul>
        </li>
        <li>
            <a href="/resources">Resources</a>
            <ul>
                <li><a href="/blog">Blog</a></li>
                <li><a href="/docs">Documentation</a></li>
                <li><a href="/tutorials">Tutorials</a></li>
            </ul>
        </li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>
```

### Breadcrumb Navigation

```html
<nav aria-label="Breadcrumb">
    <ol>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/products/software">Software</a></li>
        <li aria-current="page">Visual Studio Code</li>
    </ol>
</nav>

<style>
    nav[aria-label="Breadcrumb"] ol {
        display: flex;
        list-style: none;
        padding: 0;
    }
    nav[aria-label="Breadcrumb"] li:not(:last-child)::after {
        content: "›";
        margin: 0 8px;
    }
</style>
```

**Preview:**
> Home › Products › Software › Visual Studio Code

---

## Combining Everything

### Complete Portfolio Page Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Portfolio of Jane Doe, Full-Stack Developer specializing in React and Node.js">
    <title>Jane Doe | Full-Stack Developer Portfolio</title>

    <!-- Preload critical LCP image -->
    <link rel="preload" as="image" href="images/hero.webp" type="image/webp">

    <style>
        /* Skip link styles */
        .skip-link {
            position: absolute;
            top: -40px;
            left: 0;
            background: #1a1a2e;
            color: #fff;
            padding: 8px 16px;
            text-decoration: none;
            z-index: 100;
        }
        .skip-link:focus { top: 0; }

        /* Basic responsive styles */
        img { max-width: 100%; height: auto; }
        nav ul { list-style: none; padding: 0; display: flex; gap: 20px; }
        nav a { text-decoration: none; padding: 8px 16px; }
        nav a:focus { outline: 3px solid #005fcc; outline-offset: 2px; }
    </style>
</head>
<body>
    <!-- Skip link for keyboard users -->
    <a href="#main-content" class="skip-link">Skip to main content</a>

    <!-- ============================================
         HEADER SECTION
    ============================================= -->
    <header>
        <img src="images/logo.svg" alt="Jane Doe" width="120" height="40">

        <nav aria-label="Main navigation">
            <ul>
                <li><a href="#about">About</a></li>
                <li><a href="#skills">Skills</a></li>
                <li><a href="#projects">Projects</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <!-- ============================================
         MAIN CONTENT
    ============================================= -->
    <main id="main-content">
        <!-- Hero Section with LCP Image -->
        <section id="hero">
            <picture>
                <source srcset="images/hero.avif" type="image/avif">
                <source srcset="images/hero.webp" type="image/webp">
                <img
                    src="images/hero.jpg"
                    alt="Jane Doe working at her desk with multiple monitors"
                    width="1200"
                    height="600"
                    fetchpriority="high"
                    loading="eager"
                    decoding="async"
                >
            </picture>
            <h1>Jane Doe</h1>
            <p><em>Full-Stack Developer | React & Node.js Specialist</em></p>
        </section>

        <!-- About Section -->
        <section id="about">
            <h2>About Me</h2>
            <figure>
                <img
                    src="images/profile.jpg"
                    alt="Professional headshot of Jane Doe"
                    width="300"
                    height="300"
                    loading="lazy"
                >
                <figcaption>Jane Doe, Full-Stack Developer</figcaption>
            </figure>
            <p>I'm a passionate full-stack developer with <strong>5+ years of experience</strong>
            building scalable web applications. I specialize in <mark>React, Node.js, and TypeScript</mark>.</p>
            <p>When I'm not coding, you can find me contributing to open-source projects or
            writing technical articles on <a href="https://dev.to/janedoe" target="_blank"
            rel="noopener noreferrer">DEV Community</a>.</p>
        </section>

        <!-- Skills Section -->
        <section id="skills">
            <h2>Technical Skills</h2>

            <h3>Frontend</h3>
            <ul>
                <li>React.js / Next.js</li>
                <li>TypeScript</li>
                <li>Tailwind CSS</li>
                <li>HTML5 & CSS3</li>
            </ul>

            <h3>Backend</h3>
            <ul>
                <li>Node.js / Express</li>
                <li>PostgreSQL / MongoDB</li>
                <li>GraphQL / REST APIs</li>
                <li>Docker / Kubernetes</li>
            </ul>

            <h3>Tools & Practices</h3>
            <ul>
                <li>Git & GitHub</li>
                <li>CI/CD (GitHub Actions)</li>
                <li>Test-Driven Development</li>
                <li>Agile / Scrum</li>
            </ul>
        </section>

        <!-- Projects Section -->
        <section id="projects">
            <h2>Featured Projects</h2>

            <ol>
                <li>
                    <h3>E-Commerce Platform</h3>
                    <figure>
                        <picture>
                            <source srcset="images/project1.webp" type="image/webp">
                            <img
                                src="images/project1.jpg"
                                alt="Screenshot of e-commerce dashboard showing product management interface"
                                width="600"
                                height="400"
                                loading="lazy"
                            >
                        </picture>
                        <figcaption>Full-stack e-commerce solution with React and Node.js</figcaption>
                    </figure>
                    <p>Built a complete e-commerce solution with <strong>React, Node.js, and Stripe integration</strong>.</p>
                    <p>
                        <a href="https://github.com/janedoe/ecommerce" target="_blank" rel="noopener noreferrer">
                            View on GitHub
                        </a> |
                        <a href="https://ecommerce-demo.janedoe.dev" target="_blank" rel="noopener noreferrer">
                            Live Demo
                        </a>
                    </p>
                </li>

                <li>
                    <h3>Task Management App</h3>
                    <figure>
                        <picture>
                            <source srcset="images/project2.webp" type="image/webp">
                            <img
                                src="images/project2.jpg"
                                alt="Task management app showing kanban board with drag and drop cards"
                                width="600"
                                height="400"
                                loading="lazy"
                            >
                        </picture>
                        <figcaption>Real-time collaborative task management</figcaption>
                    </figure>
                    <p>Real-time collaborative app with <strong>drag-and-drop, WebSockets, and team features</strong>.</p>
                    <p>
                        <a href="https://github.com/janedoe/taskapp" target="_blank" rel="noopener noreferrer">
                            View on GitHub
                        </a>
                    </p>
                </li>
            </ol>
        </section>

        <!-- Contact Section -->
        <section id="contact">
            <h2>Get In Touch</h2>

            <dl>
                <dt>Email</dt>
                <dd><a href="mailto:jane@janedoe.dev">jane@janedoe.dev</a></dd>

                <dt>Phone</dt>
                <dd><a href="tel:+1-555-123-4567">(555) 123-4567</a></dd>

                <dt>Location</dt>
                <dd>San Francisco, CA (Remote Available)</dd>
            </dl>

            <h3>Social Links</h3>
            <ul>
                <li>
                    <a href="https://github.com/janedoe" target="_blank" rel="noopener noreferrer">
                        <img src="images/github-icon.svg" alt="" width="24" height="24" aria-hidden="true">
                        GitHub
                    </a>
                </li>
                <li>
                    <a href="https://linkedin.com/in/janedoe" target="_blank" rel="noopener noreferrer">
                        <img src="images/linkedin-icon.svg" alt="" width="24" height="24" aria-hidden="true">
                        LinkedIn
                    </a>
                </li>
                <li>
                    <a href="https://twitter.com/janedoedev" target="_blank" rel="noopener noreferrer">
                        <img src="images/twitter-icon.svg" alt="" width="24" height="24" aria-hidden="true">
                        Twitter
                    </a>
                </li>
            </ul>

            <p>
                <a href="files/jane-doe-resume.pdf" download="Jane-Doe-Resume.pdf">
                    📄 Download My Resume (PDF)
                </a>
            </p>
        </section>
    </main>

    <!-- ============================================
         FOOTER
    ============================================= -->
    <footer>
        <nav aria-label="Footer navigation">
            <ul>
                <li><a href="#about">About</a></li>
                <li><a href="#skills">Skills</a></li>
                <li><a href="#projects">Projects</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>

        <p><a href="#hero">↑ Back to top</a></p>
        <p><small>&copy; 2024 Jane Doe. All rights reserved.</small></p>
    </footer>
</body>
</html>
```

---

## Common Mistakes to Avoid

### Links

```html
<!-- ❌ WRONG: Non-descriptive link text -->
<a href="about.html">Click here</a>
<a href="report.pdf">Read more</a>

<!-- ✅ CORRECT: Descriptive link text -->
<a href="about.html">Learn about our company</a>
<a href="report.pdf">Download the 2024 Annual Report (PDF, 2.5MB)</a>
```

```html
<!-- ❌ WRONG: Missing security attributes on external links -->
<a href="https://external.com" target="_blank">External Site</a>

<!-- ✅ CORRECT: Secure external links -->
<a href="https://external.com" target="_blank" rel="noopener noreferrer">External Site</a>
```

### Images

```html
<!-- ❌ WRONG: Missing alt text -->
<img src="product.jpg">

<!-- ❌ WRONG: Missing dimensions (causes layout shift) -->
<img src="product.jpg" alt="Product photo">

<!-- ❌ WRONG: Lazy loading LCP image -->
<img src="hero.jpg" alt="Hero" loading="lazy">

<!-- ✅ CORRECT: Complete image element -->
<img
    src="product.jpg"
    alt="Blue wireless headphones with noise cancellation"
    width="400"
    height="400"
    loading="lazy"
>

<!-- ✅ CORRECT: LCP image with priority -->
<img
    src="hero.jpg"
    alt="Hero banner"
    width="1200"
    height="600"
    loading="eager"
    fetchpriority="high"
>
```

### Lists

```html
<!-- ❌ WRONG: Using divs instead of list -->
<div class="nav-item">Home</div>
<div class="nav-item">About</div>
<div class="nav-item">Contact</div>

<!-- ✅ CORRECT: Semantic list for navigation -->
<nav>
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>
```

```html
<!-- ❌ WRONG: Incorrect list nesting -->
<ul>
    <li>Item 1</li>
    <ul>  <!-- ul should be inside li -->
        <li>Subitem</li>
    </ul>
</ul>

<!-- ✅ CORRECT: Proper nesting -->
<ul>
    <li>Item 1
        <ul>
            <li>Subitem</li>
        </ul>
    </li>
</ul>
```

---

## Live Examples

### Example 1: Product Card

```html
<article class="product-card">
    <figure>
        <picture>
            <source srcset="headphones.avif" type="image/avif">
            <source srcset="headphones.webp" type="image/webp">
            <img
                src="headphones.jpg"
                alt="Sony WH-1000XM5 wireless headphones in black"
                width="300"
                height="300"
                loading="lazy"
            >
        </picture>
    </figure>

    <h3>Sony WH-1000XM5</h3>
    <p>Industry-leading noise cancellation wireless headphones</p>

    <dl>
        <dt>Price</dt>
        <dd><del>$399.99</del> <strong>$349.99</strong></dd>

        <dt>Rating</dt>
        <dd>★★★★★ (4.8/5 from 2,500 reviews)</dd>

        <dt>Availability</dt>
        <dd>In Stock - Ships in 1-2 days</dd>
    </dl>

    <ul>
        <li>30-hour battery life</li>
        <li>Multipoint connection</li>
        <li>Touch controls</li>
    </ul>

    <p>
        <a href="/products/sony-wh1000xm5">View Details</a> |
        <a href="/cart/add/sony-wh1000xm5">Add to Cart</a>
    </p>
</article>
```

---

### Example 2: Documentation Page

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>API Reference | Authentication</title>
</head>
<body>
    <a href="#main-content" class="skip-link">Skip to main content</a>

    <nav aria-label="Breadcrumb">
        <ol>
            <li><a href="/docs">Docs</a></li>
            <li><a href="/docs/api">API Reference</a></li>
            <li aria-current="page">Authentication</li>
        </ol>
    </nav>

    <main id="main-content">
        <h1>Authentication API</h1>
        <p>Learn how to authenticate requests to our API.</p>

        <nav aria-label="Table of contents">
            <h2>On this page</h2>
            <ul>
                <li><a href="#overview">Overview</a></li>
                <li><a href="#api-keys">API Keys</a></li>
                <li><a href="#oauth">OAuth 2.0</a></li>
                <li><a href="#examples">Examples</a></li>
            </ul>
        </nav>

        <section id="overview">
            <h2>Overview</h2>
            <p>Our API supports two authentication methods:</p>
            <ol>
                <li><strong>API Keys</strong> — Best for server-to-server communication</li>
                <li><strong>OAuth 2.0</strong> — Best for user-facing applications</li>
            </ol>
        </section>

        <section id="api-keys">
            <h2>API Keys</h2>
            <p>Generate an API key in your
            <a href="https://dashboard.example.com" target="_blank" rel="noopener noreferrer">
                dashboard
            </a>.</p>

            <figure>
                <img
                    src="images/api-key-screenshot.png"
                    alt="Dashboard showing API key section with 'Generate New Key' button"
                    width="600"
                    height="300"
                    loading="lazy"
                >
                <figcaption>Figure 1: API Keys section in the dashboard</figcaption>
            </figure>
        </section>

        <section id="oauth">
            <h2>OAuth 2.0</h2>
            <p>For applications that act on behalf of users.</p>

            <h3>Supported Grant Types</h3>
            <dl>
                <dt>Authorization Code</dt>
                <dd>For web applications with a backend server</dd>

                <dt>PKCE</dt>
                <dd>For single-page apps and mobile applications</dd>

                <dt>Client Credentials</dt>
                <dd>For machine-to-machine authentication</dd>
            </dl>
        </section>

        <section id="examples">
            <h2>Code Examples</h2>
            <p>Download our SDK or view examples:</p>
            <ul>
                <li>
                    <a href="https://github.com/example/sdk-python" target="_blank" rel="noopener noreferrer">
                        Python SDK
                    </a>
                </li>
                <li>
                    <a href="https://github.com/example/sdk-node" target="_blank" rel="noopener noreferrer">
                        Node.js SDK
                    </a>
                </li>
                <li>
                    <a href="examples/auth-examples.zip" download>
                        Download all examples (ZIP, 2.1MB)
                    </a>
                </li>
            </ul>
        </section>
    </main>

    <footer>
        <p>
            <a href="/docs">← Back to Documentation</a> |
            <a href="#main-content">↑ Back to top</a>
        </p>
    </footer>
</body>
</html>
```

---

## Assignments

### 🟢 Beginner Level

#### Assignment 1: Personal Profile Page

Update your Day 1 profile page to include:

**Requirements:**
- [ ] Navigation links to different sections (`#about`, `#hobbies`, `#contact`)
- [ ] A profile image with proper `alt` text, `width`, and `height`
- [ ] An unordered list of your hobbies/interests
- [ ] An ordered list of your top 3 goals
- [ ] Links to your social profiles (or placeholder links)
- [ ] Email and phone links using `mailto:` and `tel:`
- [ ] A "Back to top" anchor link at the bottom
- [ ] Skip link for accessibility

---

#### Assignment 2: Fix the Broken Links & Images

Debug and fix all issues in this HTML:

```html
<nav>
    <a href="home">Home</a>
    <a href="https://external.com" target="_blank">External</a>
    <a>About</a>
</nav>

<img src="hero.jpg">
<img src="product.jpg" alt="image">

<ul>
    <li>Item 1
    <li>Item 2</li>
    <ul>
        <li>Subitem</li>
    </ul>
</ul>
```

**Requirements:**
- [ ] Fix the relative path for internal links
- [ ] Add security attributes to external link
- [ ] Add `href` to the missing link
- [ ] Add proper `alt` text, `width`, `height` to all images
- [ ] Add `loading` attributes appropriately
- [ ] Fix list nesting issues
- [ ] Add all closing tags

---

### 🟡 Intermediate Level

#### Assignment 3: Image Gallery Page

Create `gallery.html` with an accessible image gallery.

**Requirements:**
- [ ] 6+ images using `<picture>` with WebP/AVIF fallbacks
- [ ] Responsive images using `srcset` and `sizes`
- [ ] Proper `alt` text for all images
- [ ] `loading="lazy"` for below-fold images
- [ ] `loading="eager"` and `fetchpriority="high"` for first/hero image
- [ ] `<figure>` and `<figcaption>` for image captions
- [ ] Navigation to filter/category links
- [ ] Width and height on all images to prevent CLS
- [ ] Light/dark image versions using `<picture>` and `prefers-color-scheme`

---

#### Assignment 4: Restaurant Menu Page

Create `menu.html` for a restaurant website.

**Requirements:**
- [ ] Skip link for accessibility
- [ ] Navigation with anchor links to menu sections
- [ ] Sections: Appetizers, Main Courses, Desserts, Drinks
- [ ] Use `<dl>` for menu items (name as `<dt>`, description + price as `<dd>`)
- [ ] Images for featured dishes with proper optimization
- [ ] Dietary indicators using an unordered list (V = Vegetarian, GF = Gluten-free)
- [ ] Link to download menu as PDF
- [ ] Phone number link for reservations
- [ ] "Order Online" external link (with proper security)

---

### 🔴 Advanced Level

#### Assignment 5: Complete Documentation Site

Create a multi-page documentation site structure.

**Required Files:**
- `index.html` (Home/Getting Started)
- `installation.html`
- `api-reference.html`

**Requirements:**
- [ ] Consistent navigation across all pages
- [ ] Breadcrumb navigation on subpages
- [ ] Table of contents with anchor links on each page
- [ ] Code examples with copy-to-clipboard links (use `#` for now)
- [ ] Images/diagrams with full responsive treatment
- [ ] External links to GitHub, npm, etc. (with security)
- [ ] Download links for SDKs/examples
- [ ] Previous/Next navigation between pages
- [ ] Preload critical images in `<head>`
- [ ] Skip links on all pages
- [ ] ARIA labels for all navigation elements

---

## Mini Projects

### 🟢 Beginner: Navigation Component

**Goal:** Build a reusable navigation component with HTML only.

**Time:** 30-45 minutes

**File:** `navigation.html`

**Requirements:**
1. Logo image (SVG preferred)
2. Main navigation with 5 links
3. One link marked as current with `aria-current="page"`
4. Mobile menu trigger (link with `#menu`)
5. Skip link at the very beginning
6. Footer navigation with different links

---

### 🟡 Intermediate: Image Showcase

**Goal:** Create a responsive image showcase demonstrating all modern techniques.

**Time:** 60-90 minutes

**File:** `showcase.html`

**Requirements:**
1. Hero image with:
   - AVIF → WebP → JPEG fallback chain
   - `srcset` with 4 different sizes
   - `fetchpriority="high"` and `loading="eager"`
   - Preload link in `<head>`
2. Gallery of 6 images with:
   - `loading="lazy"`
   - `<figure>` and `<figcaption>`
   - Proper alt text
3. Art direction example:
   - Different image for mobile vs desktop using `<picture>` media queries

---

### 🔴 Advanced: Link Directory

**Goal:** Create an accessible link directory (like a wiki's portal page).

**Time:** 2-3 hours

**File:** `directory.html`

**Requirements:**
1. Skip link to main content
2. Category navigation (anchor links)
3. 4+ categories, each with:
   - Section heading
   - Description list of 5+ links with descriptions
   - Icon images for each category
4. Mix of internal and external links
5. Featured section with large image cards
6. Sidebar with "Quick Links" unordered list
7. Footer with:
   - Sitemap navigation
   - Social media links
   - Contact links (email, phone)
   - Download link for "offline version"
8. Pass WAVE accessibility tool with zero errors

---

## Quick Reference

### Link Syntax

```html
<!-- Internal link -->
<a href="page.html">Link Text</a>

<!-- External link (secure) -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer">External</a>

<!-- Anchor link -->
<a href="#section-id">Jump to Section</a>

<!-- Email link -->
<a href="mailto:email@example.com">Email Us</a>

<!-- Phone link -->
<a href="tel:+1234567890">Call Us</a>

<!-- Download link -->
<a href="file.pdf" download="custom-name.pdf">Download</a>
```

### Image Syntax

```html
<!-- Basic image -->
<img src="image.jpg" alt="Description" width="800" height="600">

<!-- Lazy loaded image -->
<img src="image.jpg" alt="Description" width="800" height="600" loading="lazy">

<!-- LCP/Hero image -->
<img src="hero.jpg" alt="Description" loading="eager" fetchpriority="high">

<!-- Responsive image with modern formats -->
<picture>
    <source srcset="image.avif" type="image/avif">
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="Description" width="800" height="600">
</picture>

<!-- Fully responsive image -->
<img
    src="image.jpg"
    srcset="image-400.jpg 400w, image-800.jpg 800w, image-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, 50vw"
    alt="Description"
    width="1200"
    height="800"
>
```

### List Syntax

```html
<!-- Unordered list -->
<ul>
    <li>Item</li>
</ul>

<!-- Ordered list -->
<ol>
    <li>First</li>
    <li>Second</li>
</ol>

<!-- Ordered list with attributes -->
<ol start="5" type="A" reversed>
    <li>Item</li>
</ol>

<!-- Description list -->
<dl>
    <dt>Term</dt>
    <dd>Definition</dd>
</dl>

<!-- Navigation list -->
<nav aria-label="Main">
    <ul>
        <li><a href="/" aria-current="page">Home</a></li>
        <li><a href="/about">About</a></li>
    </ul>
</nav>
```

### Accessibility Checklist

| Element | Requirement |
|---------|-------------|
| Links | Descriptive text, not "click here" |
| External links | `rel="noopener noreferrer"` with `target="_blank"` |
| Images | `alt` text (empty for decorative) |
| Images | `width` and `height` attributes |
| LCP Images | `loading="eager"` and `fetchpriority="high"` |
| Other Images | `loading="lazy"` |
| Navigation | `<nav>` with `aria-label` |
| Current page | `aria-current="page"` |
| Skip link | First element, links to `#main-content` |
| Focus | Visible focus indicators (never `outline: none`) |

---

## Self-Check Questions

1. What's the difference between `target="_blank"` and `target="_self"`?
2. Why must you include `rel="noopener noreferrer"` on external links with `target="_blank"`?
3. What makes good alt text? What should decorative images have for alt?
4. When should you use `loading="lazy"` vs `loading="eager"`?
5. What is `fetchpriority="high"` for and when should you use it?
6. What's the difference between `srcset` and the `<picture>` element?
7. When should you use an ordered list vs unordered list vs description list?
8. What is a skip link and why is it important?
9. How do you indicate the current page in navigation for screen readers?
10. What are the three modern image formats and their browser support levels?

<details>
<summary><strong>Click to see answers</strong></summary>

1. `_blank` opens in new tab, `_self` (default) opens in same tab
2. Security: prevents tabnapping attack where new page can access `window.opener`
3. Concise description of image content/purpose; decorative images use empty alt (`alt=""`)
4. `lazy` for below-fold images, `eager` for LCP/above-fold critical images
5. Prioritizes the LCP image download; use only for your single most important hero image
6. `srcset` provides size variations of same image; `<picture>` allows different images (art direction) or formats
7. Ordered for sequences/rankings, unordered for non-sequential items, description for term-definition pairs
8. First focusable element that lets keyboard users jump past navigation to main content
9. `aria-current="page"` on the current page's navigation link
10. AVIF (~80%), WebP (~95%), JPEG/PNG (100%)

</details>

---

## Further Learning

### Browser DevTools

Learn to inspect and debug links and images:

1. **Elements Panel:** Inspect link attributes and image sources
2. **Network Panel:** See image file sizes and loading times
3. **Lighthouse:** Audit for accessibility and performance
4. **Coverage:** Find unused CSS/JS affecting images

**Keyboard Shortcut:** Press `F12` or `Ctrl+Shift+I` (Windows/Linux) or `Cmd+Option+I` (Mac)

### Testing Your Work

**Accessibility Testing:**
- [WAVE Web Accessibility Evaluator](https://wave.webaim.org/)
- [axe DevTools Browser Extension](https://www.deque.com/axe/)
- Screen reader testing (NVDA, VoiceOver, JAWS)

**Performance Testing:**
- [PageSpeed Insights](https://pagespeed.web.dev/)
- [WebPageTest](https://www.webpagetest.org/)
- Lighthouse in Chrome DevTools

**Validation:**
- [W3C Markup Validator](https://validator.w3.org/)
- [Nu HTML Checker](https://validator.w3.org/nu/)

### Recommended Resources

**Official Documentation:**
- [MDN: HTML Links](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Creating_hyperlinks)
- [MDN: Images in HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Images_in_HTML)
- [MDN: Responsive Images](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)
- [web.dev: Optimize LCP](https://web.dev/articles/optimize-lcp)

**Accessibility:**
- [W3C WAI Images Tutorial](https://www.w3.org/WAI/tutorials/images/)
- [W3C WAI Links Tutorial](https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-in-context.html)
- [WebAIM: Links and Hypertext](https://webaim.org/techniques/hypertext/)
- [WCAG 2.2 Guidelines](https://www.w3.org/TR/WCAG22/)

**Performance:**
- [Core Web Vitals](https://web.dev/vitals/)
- [Image Optimization Guide](https://web.dev/fast/#optimize-your-images)

**Video Resources:**
- [Kevin Powell - Responsive Images](https://www.youtube.com/watch?v=2QYpkrX2N48)
- [Fireship - Image Optimization](https://www.youtube.com/watch?v=E2v9hfDVnSE)
- [Web Dev Simplified - HTML Links](https://www.youtube.com/watch?v=tsEQgGjSmkM)

---

## Key Takeaways

1. **Links connect the web** — Use descriptive text, never "click here"
2. **Security matters** — Always add `rel="noopener noreferrer"` to external links with `target="_blank"`
3. **Alt text is mandatory** — Describe images concisely; use empty alt for decorative images
4. **Modern formats save bandwidth** — Use AVIF → WebP → JPEG fallback chain
5. **Responsive images are non-negotiable** — Use `srcset` and `sizes` for different screen sizes
6. **Optimize for Core Web Vitals** — `fetchpriority="high"` on LCP, `loading="lazy"` on others
7. **Lists provide structure** — Navigation should use `<nav>` with `<ul>` or `<ol>`
8. **Accessibility benefits everyone** — Skip links, focus states, proper ARIA attributes
9. **Prevent layout shift** — Always include `width` and `height` on images
10. **Test with real tools** — Use Lighthouse, WAVE, and screen readers

---

**Next Lesson:** [Day 3-4 - Semantic HTML & Forms](./day-03-04-semantic-html-forms.md)
