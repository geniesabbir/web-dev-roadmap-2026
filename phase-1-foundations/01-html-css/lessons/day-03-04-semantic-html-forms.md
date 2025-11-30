# Day 3-4: Semantic HTML & Forms

## What is Semantic HTML?

Semantic HTML uses elements that describe their meaning to both the browser and the developer. Instead of using generic `<div>` elements everywhere, we use elements that indicate what type of content they contain.

### Why Semantic HTML Matters

1. **Accessibility**: Screen readers understand the page structure
2. **SEO**: Search engines better understand your content
3. **Maintainability**: Code is easier to read and maintain
4. **Consistency**: Standard elements work predictably across browsers

---

## Semantic Structure Elements

### Page Layout Elements

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Semantic HTML Example</title>
</head>
<body>
    <header>
        <!-- Site header, logo, navigation -->
    </header>

    <nav>
        <!-- Main navigation links -->
    </nav>

    <main>
        <!-- Primary content of the page -->

        <article>
            <!-- Self-contained content -->
        </article>

        <section>
            <!-- Thematic grouping of content -->
        </section>

        <aside>
            <!-- Sidebar, related content -->
        </aside>
    </main>

    <footer>
        <!-- Site footer, copyright, links -->
    </footer>
</body>
</html>
```

---

### `<header>` Element

The header typically contains introductory content or navigation aids.

```html
<!-- Site header -->
<header>
    <img src="logo.png" alt="Company Logo">
    <h1>My Website</h1>
    <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
        <a href="/contact">Contact</a>
    </nav>
</header>

<!-- Article header -->
<article>
    <header>
        <h2>Article Title</h2>
        <p>Published on <time datetime="2024-01-15">January 15, 2024</time></p>
        <p>By <a href="/author/john">John Doe</a></p>
    </header>
    <p>Article content...</p>
</article>
```

**Note:** You can have multiple `<header>` elements - one for the page and one for each `<article>` or `<section>`.

---

### `<nav>` Element

Contains navigation links. Use for major navigation blocks.

```html
<!-- Main navigation -->
<nav aria-label="Main navigation">
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>

<!-- Breadcrumb navigation -->
<nav aria-label="Breadcrumb">
    <ol>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li aria-current="page">Widget Pro</li>
    </ol>
</nav>

<!-- Footer navigation (don't need <nav> for every link group) -->
<footer>
    <a href="/privacy">Privacy Policy</a>
    <a href="/terms">Terms of Service</a>
</footer>
```

---

### `<main>` Element

Contains the main content of the page. There should be **only one** per page.

```html
<body>
    <header>...</header>
    <nav>...</nav>

    <main>
        <!-- The primary, unique content of this page -->
        <h1>Welcome to Our Store</h1>
        <p>Browse our amazing products...</p>
    </main>

    <footer>...</footer>
</body>
```

**Rules for `<main>`:**
- Only ONE per page
- Must not be a descendant of `<article>`, `<aside>`, `<footer>`, `<header>`, or `<nav>`
- Should contain content unique to that page (not repeated across pages)

---

### `<article>` Element

Self-contained content that could be distributed independently.

```html
<!-- Blog post -->
<article>
    <header>
        <h2>How to Learn JavaScript</h2>
        <time datetime="2024-01-15">January 15, 2024</time>
    </header>
    <p>JavaScript is a powerful programming language...</p>
    <footer>
        <p>Tags: <a href="/tag/javascript">JavaScript</a>, <a href="/tag/coding">Coding</a></p>
    </footer>
</article>

<!-- Comment -->
<article>
    <header>
        <strong>Jane Smith</strong>
        <time datetime="2024-01-16T10:30">Jan 16, 10:30 AM</time>
    </header>
    <p>Great article! Very helpful.</p>
</article>

<!-- Product card -->
<article>
    <img src="product.jpg" alt="Blue T-Shirt">
    <h3>Blue T-Shirt</h3>
    <p>$29.99</p>
    <button>Add to Cart</button>
</article>
```

**Ask yourself:** Could this content make sense on its own, like in an RSS feed? If yes, use `<article>`.

---

### `<section>` Element

Groups related content together with a heading.

```html
<main>
    <section>
        <h2>About Us</h2>
        <p>We are a company that...</p>
    </section>

    <section>
        <h2>Our Services</h2>
        <ul>
            <li>Web Development</li>
            <li>Mobile Apps</li>
            <li>Consulting</li>
        </ul>
    </section>

    <section>
        <h2>Testimonials</h2>
        <article>...</article>
        <article>...</article>
    </section>
</main>
```

**Rule:** Every `<section>` should have a heading (`<h1>`-`<h6>`).

---

### `<aside>` Element

Content tangentially related to the main content - like sidebars.

```html
<main>
    <article>
        <h2>Main Article</h2>
        <p>The main content of this page...</p>

        <!-- Aside within an article -->
        <aside>
            <h3>Did You Know?</h3>
            <p>An interesting fact related to this article...</p>
        </aside>
    </article>
</main>

<!-- Sidebar -->
<aside>
    <h2>Related Articles</h2>
    <ul>
        <li><a href="#">Article 1</a></li>
        <li><a href="#">Article 2</a></li>
    </ul>

    <h2>Follow Us</h2>
    <ul>
        <li><a href="#">Twitter</a></li>
        <li><a href="#">Facebook</a></li>
    </ul>
</aside>
```

---

### `<footer>` Element

Contains footer information - author info, copyright, links, etc.

```html
<!-- Page footer -->
<footer>
    <nav>
        <a href="/about">About</a>
        <a href="/privacy">Privacy</a>
        <a href="/terms">Terms</a>
    </nav>
    <p>&copy; 2024 My Company. All rights reserved.</p>
</footer>

<!-- Article footer -->
<article>
    <h2>Article Title</h2>
    <p>Content...</p>
    <footer>
        <p>Written by John Doe</p>
        <p>Tags: HTML, CSS, Web Development</p>
    </footer>
</article>
```

---

## Other Semantic Elements

### `<time>` Element

Represents dates and times in a machine-readable format:

```html
<p>Published on <time datetime="2024-01-15">January 15, 2024</time></p>

<p>The event starts at <time datetime="14:30">2:30 PM</time></p>

<p>The meeting is on <time datetime="2024-03-15T09:00">March 15 at 9 AM</time></p>

<!-- Duration -->
<p>The movie is <time datetime="PT2H30M">2 hours and 30 minutes</time> long.</p>
```

### `<address>` Element

Contact information for the author/owner:

```html
<address>
    <p>Written by <a href="mailto:john@example.com">John Doe</a></p>
    <p>123 Main Street<br>New York, NY 10001</p>
</address>
```

### `<blockquote>` and `<cite>`

For quotations:

```html
<blockquote cite="https://example.com/source">
    <p>The only way to do great work is to love what you do.</p>
    <footer>— <cite>Steve Jobs</cite></footer>
</blockquote>
```

### `<figure>` and `<figcaption>`

For images, diagrams, code with captions:

```html
<figure>
    <img src="chart.png" alt="Sales chart showing growth">
    <figcaption>Figure 1: Quarterly sales growth 2024</figcaption>
</figure>

<figure>
    <pre><code>
function greet(name) {
    console.log(`Hello, ${name}!`);
}
    </code></pre>
    <figcaption>Example: A simple greeting function</figcaption>
</figure>
```

### `<details>` and `<summary>`

Expandable/collapsible content (no JavaScript needed!):

```html
<details>
    <summary>Click to show more</summary>
    <p>This content is hidden by default.</p>
    <p>Click the summary to reveal it.</p>
</details>

<!-- Open by default -->
<details open>
    <summary>FAQ: What is HTML?</summary>
    <p>HTML stands for HyperText Markup Language...</p>
</details>
```

### `<mark>` Element

Highlighted/marked text:

```html
<p>Search results for "JavaScript":</p>
<p>Learn <mark>JavaScript</mark> fundamentals in this tutorial.</p>
```

---

## Complete Semantic Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tech Blog - Semantic HTML</title>
</head>
<body>
    <header>
        <h1>Tech Blog</h1>
        <nav aria-label="Main navigation">
            <ul>
                <li><a href="/">Home</a></li>
                <li><a href="/articles">Articles</a></li>
                <li><a href="/about">About</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <article>
            <header>
                <h2>Understanding Semantic HTML</h2>
                <p>
                    By <address style="display: inline;"><a href="/author/jane">Jane Doe</a></address>
                    on <time datetime="2024-01-15">January 15, 2024</time>
                </p>
            </header>

            <section>
                <h3>What is Semantic HTML?</h3>
                <p>Semantic HTML is the use of HTML markup to reinforce the meaning of content...</p>

                <figure>
                    <img src="semantic-example.png" alt="Diagram showing semantic HTML structure">
                    <figcaption>Figure 1: Semantic HTML document structure</figcaption>
                </figure>
            </section>

            <section>
                <h3>Why It Matters</h3>
                <p>Using semantic elements improves:</p>
                <ul>
                    <li>Accessibility</li>
                    <li>SEO</li>
                    <li>Maintainability</li>
                </ul>
            </section>

            <aside>
                <h4>Pro Tip</h4>
                <p>Always ask yourself: "Does this element describe its content?"</p>
            </aside>

            <footer>
                <p>Tags: <a href="/tag/html">HTML</a>, <a href="/tag/accessibility">Accessibility</a></p>
            </footer>
        </article>

        <section>
            <h2>Comments</h2>

            <article>
                <header>
                    <strong>John Smith</strong>
                    <time datetime="2024-01-16T10:30">Jan 16, 2024</time>
                </header>
                <p>Great article! Very informative.</p>
            </article>
        </section>
    </main>

    <aside>
        <section>
            <h2>Related Articles</h2>
            <ul>
                <li><a href="/article/1">HTML Basics</a></li>
                <li><a href="/article/2">CSS Fundamentals</a></li>
            </ul>
        </section>

        <section>
            <h2>Newsletter</h2>
            <form action="/subscribe" method="post">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
                <button type="submit">Subscribe</button>
            </form>
        </section>
    </aside>

    <footer>
        <nav aria-label="Footer navigation">
            <a href="/privacy">Privacy Policy</a>
            <a href="/terms">Terms of Service</a>
        </nav>
        <p>&copy; 2024 Tech Blog. All rights reserved.</p>
    </footer>
</body>
</html>
```

---

# HTML Forms

Forms collect user input and are essential for interactive websites.

## Basic Form Structure

```html
<form action="/submit" method="post">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name">

    <button type="submit">Submit</button>
</form>
```

### Form Attributes

| Attribute | Description |
|-----------|-------------|
| `action` | URL where form data is sent |
| `method` | HTTP method (get or post) |
| `enctype` | Encoding type (needed for file uploads) |
| `autocomplete` | Enable/disable browser autocomplete |

```html
<!-- GET - data in URL (for searches, filters) -->
<form action="/search" method="get">

<!-- POST - data in request body (for sensitive data) -->
<form action="/login" method="post">

<!-- For file uploads -->
<form action="/upload" method="post" enctype="multipart/form-data">
```

---

## Input Types

HTML5 provides many specialized input types:

### Text Inputs

```html
<!-- Basic text -->
<input type="text" name="username" placeholder="Enter username">

<!-- Password (hidden characters) -->
<input type="password" name="password" placeholder="Enter password">

<!-- Email (validates format) -->
<input type="email" name="email" placeholder="you@example.com">

<!-- URL -->
<input type="url" name="website" placeholder="https://example.com">

<!-- Phone -->
<input type="tel" name="phone" placeholder="(123) 456-7890">

<!-- Search (with clear button in some browsers) -->
<input type="search" name="query" placeholder="Search...">
```

### Number Inputs

```html
<!-- Number with arrows -->
<input type="number" name="quantity" min="1" max="10" step="1" value="1">

<!-- Range slider -->
<input type="range" name="volume" min="0" max="100" value="50">
```

### Date and Time Inputs

```html
<!-- Date picker -->
<input type="date" name="birthday" min="1900-01-01" max="2024-12-31">

<!-- Time picker -->
<input type="time" name="appointment">

<!-- Date and time -->
<input type="datetime-local" name="meeting">

<!-- Month -->
<input type="month" name="expiry">

<!-- Week -->
<input type="week" name="week">
```

### Other Input Types

```html
<!-- Color picker -->
<input type="color" name="favorite-color" value="#ff0000">

<!-- File upload -->
<input type="file" name="document" accept=".pdf,.doc,.docx">

<!-- Multiple files -->
<input type="file" name="photos" multiple accept="image/*">

<!-- Hidden field -->
<input type="hidden" name="user-id" value="12345">
```

### Checkbox and Radio

```html
<!-- Checkboxes (multiple selections) -->
<fieldset>
    <legend>Select your interests:</legend>

    <label>
        <input type="checkbox" name="interests" value="coding"> Coding
    </label>
    <label>
        <input type="checkbox" name="interests" value="design"> Design
    </label>
    <label>
        <input type="checkbox" name="interests" value="music"> Music
    </label>
</fieldset>

<!-- Radio buttons (single selection) -->
<fieldset>
    <legend>Select your gender:</legend>

    <label>
        <input type="radio" name="gender" value="male"> Male
    </label>
    <label>
        <input type="radio" name="gender" value="female"> Female
    </label>
    <label>
        <input type="radio" name="gender" value="other"> Other
    </label>
</fieldset>
```

---

## Labels - Always Use Them!

Labels are crucial for accessibility:

```html
<!-- Method 1: Explicit association (recommended) -->
<label for="email">Email Address:</label>
<input type="email" id="email" name="email">

<!-- Method 2: Wrapping -->
<label>
    Email Address:
    <input type="email" name="email">
</label>
```

**Never do this:**
```html
<!-- Bad - no label -->
<input type="text" name="username" placeholder="Username">

<!-- Placeholder is NOT a substitute for label! -->
```

---

## Textarea (Multi-line Text)

```html
<label for="message">Your Message:</label>
<textarea id="message" name="message" rows="5" cols="40"
          placeholder="Enter your message here..."></textarea>
```

---

## Select Dropdown

```html
<label for="country">Country:</label>
<select id="country" name="country">
    <option value="">Select a country</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="ca">Canada</option>
    <option value="au">Australia</option>
</select>

<!-- With option groups -->
<select name="car">
    <optgroup label="Swedish Cars">
        <option value="volvo">Volvo</option>
        <option value="saab">Saab</option>
    </optgroup>
    <optgroup label="German Cars">
        <option value="mercedes">Mercedes</option>
        <option value="audi">Audi</option>
    </optgroup>
</select>

<!-- Multiple selection -->
<select name="languages" multiple>
    <option value="html">HTML</option>
    <option value="css">CSS</option>
    <option value="js">JavaScript</option>
</select>
```

---

## Datalist (Autocomplete Suggestions)

```html
<label for="browser">Choose your browser:</label>
<input list="browsers" id="browser" name="browser">

<datalist id="browsers">
    <option value="Chrome">
    <option value="Firefox">
    <option value="Safari">
    <option value="Edge">
</datalist>
```

---

## Form Validation

HTML5 provides built-in validation:

```html
<form action="/submit" method="post">
    <!-- Required field -->
    <input type="text" name="username" required>

    <!-- Minimum/maximum length -->
    <input type="text" name="username" minlength="3" maxlength="20">

    <!-- Pattern (regex) -->
    <input type="text" name="zip" pattern="[0-9]{5}" title="5 digit zip code">

    <!-- Number range -->
    <input type="number" name="age" min="18" max="120">

    <!-- Email validation (automatic) -->
    <input type="email" name="email" required>

    <button type="submit">Submit</button>
</form>
```

---

## Buttons

```html
<!-- Submit button -->
<button type="submit">Submit Form</button>

<!-- Reset button (clears form) -->
<button type="reset">Clear Form</button>

<!-- Regular button (for JavaScript) -->
<button type="button">Click Me</button>

<!-- Alternative submit -->
<input type="submit" value="Submit">
```

---

## Complete Form Example

```html
<form action="/register" method="post">
    <h2>Create Account</h2>

    <fieldset>
        <legend>Personal Information</legend>

        <div>
            <label for="fullname">Full Name: *</label>
            <input type="text" id="fullname" name="fullname" required
                   minlength="2" maxlength="100">
        </div>

        <div>
            <label for="email">Email: *</label>
            <input type="email" id="email" name="email" required>
        </div>

        <div>
            <label for="password">Password: *</label>
            <input type="password" id="password" name="password" required
                   minlength="8" pattern="(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}"
                   title="Must contain at least one number, one uppercase and lowercase letter, and at least 8 characters">
        </div>

        <div>
            <label for="birthday">Birthday:</label>
            <input type="date" id="birthday" name="birthday">
        </div>
    </fieldset>

    <fieldset>
        <legend>Preferences</legend>

        <div>
            <label for="country">Country:</label>
            <select id="country" name="country">
                <option value="">Select...</option>
                <option value="us">United States</option>
                <option value="uk">United Kingdom</option>
                <option value="other">Other</option>
            </select>
        </div>

        <div>
            <p>Interests:</p>
            <label>
                <input type="checkbox" name="interests" value="tech"> Technology
            </label>
            <label>
                <input type="checkbox" name="interests" value="sports"> Sports
            </label>
            <label>
                <input type="checkbox" name="interests" value="music"> Music
            </label>
        </div>
    </fieldset>

    <fieldset>
        <legend>Additional</legend>

        <div>
            <label for="bio">About You:</label>
            <textarea id="bio" name="bio" rows="4"
                      placeholder="Tell us about yourself..."></textarea>
        </div>

        <div>
            <label>
                <input type="checkbox" name="newsletter" value="yes"> Subscribe to newsletter
            </label>
        </div>

        <div>
            <label>
                <input type="checkbox" name="terms" required> I agree to the Terms of Service *
            </label>
        </div>
    </fieldset>

    <div>
        <button type="submit">Create Account</button>
        <button type="reset">Clear Form</button>
    </div>
</form>
```

---

## Key Takeaways

1. **Use semantic elements** - They convey meaning and improve accessibility
2. **Always label your inputs** - Essential for accessibility
3. **Use appropriate input types** - They improve mobile experience
4. **Validate on the client AND server** - Never trust client-side validation alone
5. **Group related fields** - Use `<fieldset>` and `<legend>`

---

**Next Lesson:** [Day 5 - Accessibility (a11y)](./day-05-accessibility.md)
