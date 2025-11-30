# Day 2: Links, Images, and Lists

## Links (Anchor Elements)

Links are the backbone of the web - they connect pages together. The `<a>` (anchor) element creates hyperlinks.

### Basic Link Syntax

```html
<a href="https://www.example.com">Click here to visit Example</a>
```

- `href` - Hypertext Reference: The URL the link points to
- The text between the tags is what users see and click

---

### Types of Links

#### 1. External Links (Other Websites)

```html
<!-- Links to another website -->
<a href="https://www.google.com">Visit Google</a>

<!-- Always use https when possible -->
<a href="https://github.com">GitHub</a>
```

#### 2. Internal Links (Same Website)

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

**Path Types Explained:**

| Path | Meaning | Example |
|------|---------|---------|
| `page.html` | Same folder | Links to file in current folder |
| `folder/page.html` | Subfolder | Goes into folder first |
| `../page.html` | Parent folder | Goes up one level |
| `../../page.html` | Grandparent folder | Goes up two levels |
| `/page.html` | Root | Starts from website root |

#### 3. Anchor Links (Same Page)

Jump to a specific section on the same page:

```html
<!-- The link (put anywhere) -->
<a href="#section2">Jump to Section 2</a>

<!-- The target (where you want to jump to) -->
<h2 id="section2">Section 2</h2>
```

**Creating a Table of Contents:**
```html
<nav>
    <h2>Table of Contents</h2>
    <ul>
        <li><a href="#intro">Introduction</a></li>
        <li><a href="#chapter1">Chapter 1</a></li>
        <li><a href="#chapter2">Chapter 2</a></li>
        <li><a href="#conclusion">Conclusion</a></li>
    </ul>
</nav>

<section id="intro">
    <h2>Introduction</h2>
    <p>Content here...</p>
</section>

<section id="chapter1">
    <h2>Chapter 1</h2>
    <p>Content here...</p>
</section>

<!-- etc. -->
```

#### 4. Email and Phone Links

```html
<!-- Email link -->
<a href="mailto:contact@example.com">Send us an email</a>

<!-- Email with subject -->
<a href="mailto:contact@example.com?subject=Hello&body=I wanted to say hi">
    Contact Us
</a>

<!-- Phone link (great for mobile) -->
<a href="tel:+1234567890">Call us: (123) 456-7890</a>
```

---

### Link Attributes

#### Opening in New Tab

```html
<a href="https://external-site.com" target="_blank" rel="noopener noreferrer">
    External Site
</a>
```

- `target="_blank"` - Opens in new tab
- `rel="noopener noreferrer"` - Security measure (always include with `target="_blank"`)

**Target Values:**
| Value | Behavior |
|-------|----------|
| `_self` | Same tab (default) |
| `_blank` | New tab |
| `_parent` | Parent frame |
| `_top` | Full window |

#### Download Links

```html
<a href="files/document.pdf" download>Download PDF</a>

<!-- With custom filename -->
<a href="files/document.pdf" download="my-document.pdf">Download PDF</a>
```

#### Title Attribute (Tooltip)

```html
<a href="about.html" title="Learn more about our company">About Us</a>
```

---

### Best Practices for Links

```html
<!-- Bad: Non-descriptive -->
<p>To learn more, <a href="about.html">click here</a>.</p>

<!-- Good: Descriptive link text -->
<p>Learn more on our <a href="about.html">About page</a>.</p>

<!-- Bad: URL as link text -->
<a href="https://example.com">https://example.com</a>

<!-- Good: Descriptive text -->
<a href="https://example.com">Visit Example Website</a>
```

---

## Images

Images make your pages visual and engaging. The `<img>` element embeds images.

### Basic Image Syntax

```html
<img src="photo.jpg" alt="Description of the image">
```

- `src` - Source: Path to the image file
- `alt` - Alternative text: Description for screen readers and when image fails to load

**Alt text is REQUIRED** for accessibility!

---

### Image Sources

#### Local Images

```html
<!-- Same folder -->
<img src="photo.jpg" alt="My photo">

<!-- Images folder -->
<img src="images/hero.jpg" alt="Hero banner">

<!-- Parent folder -->
<img src="../assets/logo.png" alt="Company logo">
```

#### External Images

```html
<img src="https://example.com/image.jpg" alt="External image">
```

**Note:** External images can slow your page and may be removed by the host.

---

### Image Attributes

#### Width and Height

Always specify dimensions to prevent layout shift:

```html
<img src="photo.jpg" alt="Photo" width="800" height="600">
```

Or in CSS (preferred for responsive):
```html
<img src="photo.jpg" alt="Photo" class="responsive-img">
```
```css
.responsive-img {
    max-width: 100%;
    height: auto;
}
```

#### Loading Attribute

Lazy loading for better performance:

```html
<!-- Lazy load images below the fold -->
<img src="photo.jpg" alt="Photo" loading="lazy">

<!-- Eager load critical images -->
<img src="hero.jpg" alt="Hero" loading="eager">
```

---

### Image Formats

| Format | Best For | Notes |
|--------|----------|-------|
| **JPEG/JPG** | Photos | Good compression, no transparency |
| **PNG** | Graphics, logos | Supports transparency, larger files |
| **GIF** | Simple animations | Limited colors (256) |
| **SVG** | Icons, logos | Vector, scalable, small file size |
| **WebP** | Modern replacement | Better compression, wide support |
| **AVIF** | Newest format | Best compression, growing support |

```html
<!-- Use WebP with fallback -->
<picture>
    <source srcset="image.webp" type="image/webp">
    <source srcset="image.jpg" type="image/jpeg">
    <img src="image.jpg" alt="Description">
</picture>
```

---

### Figure and Figcaption

For images with captions:

```html
<figure>
    <img src="chart.png" alt="Sales chart showing 50% growth">
    <figcaption>Figure 1: Quarterly sales growth</figcaption>
</figure>
```

---

### Writing Good Alt Text

```html
<!-- Bad: No alt text -->
<img src="dog.jpg">

<!-- Bad: Useless alt text -->
<img src="dog.jpg" alt="image">
<img src="dog.jpg" alt="dog.jpg">

<!-- Bad: Too long -->
<img src="dog.jpg" alt="A photograph of a brown and white dog that is
a beagle breed, sitting in a grassy field on a sunny day looking at
the camera with its tongue out and appearing very happy">

<!-- Good: Descriptive but concise -->
<img src="dog.jpg" alt="Beagle sitting in a grassy field">

<!-- Good: Context-dependent -->
<img src="product.jpg" alt="Blue running shoes, size 10, $89.99">

<!-- Decorative images: Empty alt -->
<img src="decorative-border.png" alt="">
```

---

## Lists

Lists organize content into easy-to-read formats.

### Unordered Lists (Bullet Points)

For items where order doesn't matter:

```html
<ul>
    <li>Apples</li>
    <li>Oranges</li>
    <li>Bananas</li>
</ul>
```

**Output:**
- Apples
- Oranges
- Bananas

### Ordered Lists (Numbered)

For items where order matters:

```html
<ol>
    <li>Preheat oven to 350°F</li>
    <li>Mix dry ingredients</li>
    <li>Add wet ingredients</li>
    <li>Bake for 25 minutes</li>
</ol>
```

**Output:**
1. Preheat oven to 350°F
2. Mix dry ingredients
3. Add wet ingredients
4. Bake for 25 minutes

#### Ordered List Attributes

```html
<!-- Start from a different number -->
<ol start="5">
    <li>Step five</li>
    <li>Step six</li>
</ol>

<!-- Reverse order -->
<ol reversed>
    <li>Third place</li>
    <li>Second place</li>
    <li>First place</li>
</ol>

<!-- Different numbering types -->
<ol type="A">  <!-- A, B, C -->
<ol type="a">  <!-- a, b, c -->
<ol type="I">  <!-- I, II, III -->
<ol type="i">  <!-- i, ii, iii -->
<ol type="1">  <!-- 1, 2, 3 (default) -->
```

### Description Lists

For term-definition pairs:

```html
<dl>
    <dt>HTML</dt>
    <dd>HyperText Markup Language - the structure of web pages</dd>

    <dt>CSS</dt>
    <dd>Cascading Style Sheets - the styling of web pages</dd>

    <dt>JavaScript</dt>
    <dd>A programming language for web interactivity</dd>
</dl>
```

- `<dl>` - Description list container
- `<dt>` - Description term
- `<dd>` - Description details

### Nested Lists

Lists can contain other lists:

```html
<ul>
    <li>Frontend Development
        <ul>
            <li>HTML</li>
            <li>CSS</li>
            <li>JavaScript</li>
        </ul>
    </li>
    <li>Backend Development
        <ul>
            <li>Node.js</li>
            <li>Python</li>
            <li>Databases</li>
        </ul>
    </li>
</ul>
```

**Output:**
- Frontend Development
  - HTML
  - CSS
  - JavaScript
- Backend Development
  - Node.js
  - Python
  - Databases

---

## Combining Everything

Here's a practical example using links, images, and lists:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Portfolio</title>
</head>
<body>
    <header>
        <img src="logo.png" alt="My Logo" width="100" height="100">
        <nav>
            <ul>
                <li><a href="#about">About</a></li>
                <li><a href="#skills">Skills</a></li>
                <li><a href="#projects">Projects</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <section id="about">
            <h1>About Me</h1>
            <figure>
                <img src="profile.jpg" alt="My profile photo" width="300" height="300">
                <figcaption>That's me!</figcaption>
            </figure>
            <p>I am a full-stack developer in training.</p>
        </section>

        <section id="skills">
            <h2>My Skills</h2>
            <ul>
                <li>HTML & CSS</li>
                <li>JavaScript</li>
                <li>React</li>
                <li>Node.js</li>
            </ul>
        </section>

        <section id="projects">
            <h2>Projects</h2>
            <ol>
                <li>
                    <a href="https://project1.com" target="_blank" rel="noopener noreferrer">
                        Project 1 - Portfolio Website
                    </a>
                </li>
                <li>
                    <a href="https://project2.com" target="_blank" rel="noopener noreferrer">
                        Project 2 - Todo App
                    </a>
                </li>
            </ol>
        </section>

        <section id="contact">
            <h2>Contact Me</h2>
            <ul>
                <li><a href="mailto:myemail@example.com">Email Me</a></li>
                <li><a href="https://github.com/myusername" target="_blank" rel="noopener noreferrer">GitHub</a></li>
                <li><a href="https://linkedin.com/in/myusername" target="_blank" rel="noopener noreferrer">LinkedIn</a></li>
            </ul>
        </section>
    </main>

    <footer>
        <p><a href="#top">Back to top</a></p>
    </footer>
</body>
</html>
```

---

## Practice Exercise

Update your "About Me" page from Day 1 to include:

1. **Navigation links** to different sections
2. **A profile image** with proper alt text
3. **An unordered list** of your hobbies
4. **An ordered list** of your top 3 goals
5. **Links to your social profiles** (or placeholder links)
6. **A "Back to top" link** at the bottom

---

## Common Mistakes

### 1. Forgetting Alt Text
```html
<!-- Wrong -->
<img src="photo.jpg">

<!-- Correct -->
<img src="photo.jpg" alt="Description">
```

### 2. Using Images for Text
```html
<!-- Wrong - text as image -->
<img src="welcome-text.png" alt="Welcome">

<!-- Correct - real text -->
<h1>Welcome</h1>
```

### 3. Missing rel="noopener" on External Links
```html
<!-- Potentially unsafe -->
<a href="https://external.com" target="_blank">Link</a>

<!-- Safe -->
<a href="https://external.com" target="_blank" rel="noopener noreferrer">Link</a>
```

### 4. Putting Block Elements Inside Inline Elements
```html
<!-- Wrong -->
<a href="#">
    <h2>Title</h2>
    <p>Description</p>
</a>

<!-- Correct (HTML5 allows this for <a>) -->
<a href="#">
    <h2>Title</h2>
    <p>Description</p>
</a>
<!-- This is actually valid in HTML5! -->
```

---

## Key Takeaways

1. **Links connect the web** - Use descriptive link text
2. **Always include alt text** on images for accessibility
3. **Use lazy loading** for images below the fold
4. **Choose the right list type** - unordered for any order, ordered for sequences
5. **Nest lists properly** - li must be direct children of ul/ol

---

## Self-Check Questions

1. What's the difference between relative and absolute URLs?
2. Why should you include `rel="noopener noreferrer"` with `target="_blank"`?
3. What makes good alt text?
4. When should you use an ordered list vs unordered list?
5. How do you create a link that jumps to a section on the same page?

---

**Next Lesson:** [Day 3-4 - Semantic HTML & Forms](./day-03-04-semantic-html-forms.md)
