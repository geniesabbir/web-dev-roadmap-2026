# Week 2, Day 5: DOM Basics

## What is the DOM?

The **Document Object Model (DOM)** is a programming interface for web documents. It represents the HTML as a tree structure where each element is a node.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
  </head>
  <body>
    <h1>Hello</h1>
    <p>World</p>
  </body>
</html>
```

**DOM Tree:**
```
document
└── html
    ├── head
    │   └── title
    │       └── "My Page"
    └── body
        ├── h1
        │   └── "Hello"
        └── p
            └── "World"
```

JavaScript can:
- **Read** the DOM (get elements, attributes, content)
- **Modify** the DOM (change content, styles, attributes)
- **Create** new elements
- **Remove** elements
- **Respond** to user interactions (events)

---

## Selecting Elements

### getElementById

Select a single element by its ID:

```html
<div id="header">Header Content</div>
<div id="main">Main Content</div>
```

```javascript
// Returns the element or null if not found
const header = document.getElementById("header");
console.log(header);  // <div id="header">...</div>

const notFound = document.getElementById("footer");
console.log(notFound);  // null
```

### getElementsByClassName

Select multiple elements by class name:

```html
<p class="text">First</p>
<p class="text">Second</p>
<p class="text highlight">Third</p>
```

```javascript
// Returns HTMLCollection (live collection)
const texts = document.getElementsByClassName("text");
console.log(texts.length);  // 3
console.log(texts[0]);      // <p class="text">First</p>

// Multiple classes
const highlighted = document.getElementsByClassName("text highlight");
console.log(highlighted.length);  // 1

// HTMLCollection is array-like but not an array
// Convert to array if needed
const textsArray = Array.from(texts);
// or
const textsArray2 = [...texts];

// HTMLCollection is LIVE (updates automatically)
document.body.innerHTML += '<p class="text">New</p>';
console.log(texts.length);  // 4 (automatically updated!)
```

### getElementsByTagName

Select elements by tag name:

```html
<p>Paragraph 1</p>
<p>Paragraph 2</p>
<div>A div</div>
```

```javascript
// Returns HTMLCollection
const paragraphs = document.getElementsByTagName("p");
console.log(paragraphs.length);  // 2

// Get all elements
const all = document.getElementsByTagName("*");
console.log(all.length);  // All elements in document
```

### querySelector (Modern - Preferred)

Select first matching element using CSS selector:

```html
<div class="container">
  <p id="intro">Intro</p>
  <ul class="list">
    <li>Item 1</li>
    <li class="active">Item 2</li>
    <li>Item 3</li>
  </ul>
</div>
```

```javascript
// By ID
const intro = document.querySelector("#intro");

// By class
const container = document.querySelector(".container");

// By tag
const firstParagraph = document.querySelector("p");

// By attribute
const activeItem = document.querySelector("[class='active']");
const activeItem2 = document.querySelector(".active");

// Complex selectors
const listItem = document.querySelector(".container .list li");
const activeInList = document.querySelector("ul.list li.active");

// Pseudo-selectors
const firstLi = document.querySelector("li:first-child");
const lastLi = document.querySelector("li:last-child");

// Returns null if not found
const notFound = document.querySelector(".nonexistent");
console.log(notFound);  // null
```

### querySelectorAll (Modern - Preferred)

Select all matching elements:

```javascript
// Returns NodeList (static, not live)
const allLis = document.querySelectorAll("li");
console.log(allLis.length);  // 3

// NodeList supports forEach (unlike HTMLCollection)
allLis.forEach((li, index) => {
  console.log(`${index}: ${li.textContent}`);
});

// Multiple selectors
const headings = document.querySelectorAll("h1, h2, h3");

// Complex selectors
const items = document.querySelectorAll(".list > li");

// NodeList is STATIC (doesn't update)
document.querySelector(".list").innerHTML += "<li>Item 4</li>";
console.log(allLis.length);  // Still 3!

// Get fresh NodeList
const freshList = document.querySelectorAll("li");
console.log(freshList.length);  // 4
```

### Selection Summary

```javascript
// ID - single element
document.getElementById("id");

// Class - multiple elements (live HTMLCollection)
document.getElementsByClassName("class");

// Tag - multiple elements (live HTMLCollection)
document.getElementsByTagName("tag");

// CSS Selector - single element
document.querySelector("selector");

// CSS Selector - multiple elements (static NodeList)
document.querySelectorAll("selector");

// RECOMMENDATION: Use querySelector and querySelectorAll
// More flexible, consistent return types, CSS selector power
```

---

## Traversing the DOM

Navigate between related elements:

```html
<div id="parent">
  <p>First child</p>
  <p id="middle">Middle child</p>
  <p>Last child</p>
</div>
```

### Parent, Children, Siblings

```javascript
const middle = document.getElementById("middle");

// Parent
console.log(middle.parentElement);  // <div id="parent">
console.log(middle.parentNode);     // Same, but can be non-element

// Children (from parent)
const parent = document.getElementById("parent");
console.log(parent.children);        // HTMLCollection of child elements
console.log(parent.childNodes);      // NodeList including text nodes!
console.log(parent.firstElementChild);  // First <p>
console.log(parent.lastElementChild);   // Last <p>
console.log(parent.childElementCount);  // 3

// Siblings
console.log(middle.previousElementSibling);  // First <p>
console.log(middle.nextElementSibling);      // Last <p>

// Element vs Node versions
// Element: Only elements
// Node: Includes text nodes, comments, etc.
console.log(middle.previousSibling);     // Might be text node (whitespace)
console.log(middle.previousElementSibling);  // Previous element
```

### Closest Ancestor

```html
<div class="card">
  <div class="card-body">
    <button class="btn">Click me</button>
  </div>
</div>
```

```javascript
const btn = document.querySelector(".btn");

// Find closest ancestor matching selector
const cardBody = btn.closest(".card-body");
const card = btn.closest(".card");

console.log(cardBody);  // <div class="card-body">
console.log(card);      // <div class="card">

// Returns null if not found
const form = btn.closest("form");
console.log(form);  // null

// Useful for event delegation
document.addEventListener("click", (e) => {
  const card = e.target.closest(".card");
  if (card) {
    console.log("Clicked inside a card!");
  }
});
```

### Contains and Matches

```javascript
const parent = document.getElementById("parent");
const middle = document.getElementById("middle");

// Check if element contains another
console.log(parent.contains(middle));  // true
console.log(middle.contains(parent));  // false

// Check if element matches selector
console.log(middle.matches("p"));           // true
console.log(middle.matches("#middle"));     // true
console.log(middle.matches(".highlight"));  // false
```

---

## Reading Element Content

### textContent vs innerHTML vs innerText

```html
<div id="content">
  <p>Hello <strong>World</strong></p>
  <p style="display: none;">Hidden text</p>
</div>
```

```javascript
const content = document.getElementById("content");

// textContent - all text, including hidden
console.log(content.textContent);
// "Hello World" + "Hidden text" (with whitespace)

// innerHTML - HTML as string
console.log(content.innerHTML);
// "<p>Hello <strong>World</strong></p><p style='display: none;'>Hidden text</p>"

// innerText - visible text only (slower, causes reflow)
console.log(content.innerText);
// "Hello World" (hidden text excluded)

// Recommendation:
// - Use textContent for text operations (fastest)
// - Use innerHTML when you need HTML structure
// - Avoid innerText (performance issues)
```

### Getting Attributes

```html
<a id="link" href="https://example.com" target="_blank" data-id="123">Link</a>
<input type="text" value="Hello" disabled>
```

```javascript
const link = document.getElementById("link");

// getAttribute - any attribute
console.log(link.getAttribute("href"));    // "https://example.com"
console.log(link.getAttribute("target"));  // "_blank"
console.log(link.getAttribute("data-id")); // "123"

// Direct property access
console.log(link.href);    // Full URL: "https://example.com"
console.log(link.target);  // "_blank"
console.log(link.id);      // "link"

// Dataset for data-* attributes
console.log(link.dataset.id);  // "123"

// Boolean attributes
const input = document.querySelector("input");
console.log(input.disabled);  // true (property)
console.log(input.getAttribute("disabled"));  // "" (attribute)

// hasAttribute
console.log(link.hasAttribute("target"));  // true
console.log(link.hasAttribute("rel"));     // false

// Get all attributes
console.log(link.attributes);  // NamedNodeMap
for (const attr of link.attributes) {
  console.log(`${attr.name}: ${attr.value}`);
}
```

### Getting Styles

```html
<div id="box" style="width: 100px; background-color: red;"></div>
```

```css
#box {
  height: 50px;
  border: 1px solid black;
}
```

```javascript
const box = document.getElementById("box");

// Inline styles only
console.log(box.style.width);           // "100px"
console.log(box.style.backgroundColor); // "red"
console.log(box.style.height);          // "" (not inline!)

// Get computed (actual) styles
const computed = getComputedStyle(box);
console.log(computed.width);   // "100px"
console.log(computed.height);  // "50px"
console.log(computed.border);  // "1px solid rgb(0, 0, 0)"

// Get specific computed value
console.log(computed.getPropertyValue("background-color"));
// "rgb(255, 0, 0)"
```

### Getting Dimensions

```javascript
const box = document.getElementById("box");

// Content dimensions (excluding padding, border, scrollbar)
console.log(box.clientWidth);
console.log(box.clientHeight);

// Including padding
console.log(box.scrollWidth);
console.log(box.scrollHeight);

// Including padding and border
console.log(box.offsetWidth);
console.log(box.offsetHeight);

// Position relative to viewport
const rect = box.getBoundingClientRect();
console.log(rect.top);     // Distance from top of viewport
console.log(rect.left);    // Distance from left of viewport
console.log(rect.width);   // Element width
console.log(rect.height);  // Element height
console.log(rect.bottom);  // top + height
console.log(rect.right);   // left + width

// Position relative to parent
console.log(box.offsetTop);
console.log(box.offsetLeft);
console.log(box.offsetParent);  // Positioned parent element
```

---

## Modifying Elements

### Changing Content

```html
<div id="content">Original content</div>
<ul id="list">
  <li>Item 1</li>
</ul>
```

```javascript
const content = document.getElementById("content");

// Set text content (safe - escapes HTML)
content.textContent = "New text content";
content.textContent = "<p>This shows as text, not HTML</p>";

// Set HTML content (be careful - XSS risk!)
content.innerHTML = "<p>This is <strong>HTML</strong></p>";

// Danger: Never use innerHTML with user input!
// const userInput = "<script>alert('hacked!')</script>";
// content.innerHTML = userInput;  // Security risk!

// Safe: Use textContent for user input
const userInput = "<script>alert('safe')</script>";
content.textContent = userInput;  // Displays as text

// Add to existing content
const list = document.getElementById("list");
list.innerHTML += "<li>Item 2</li>";  // Adds new item
```

### Changing Attributes

```html
<img id="image" src="old.jpg" alt="Old image">
<input id="input" type="text" value="Hello">
```

```javascript
const image = document.getElementById("image");

// setAttribute
image.setAttribute("src", "new.jpg");
image.setAttribute("alt", "New image");
image.setAttribute("data-id", "123");

// Direct property
image.src = "another.jpg";
image.alt = "Another image";

// Remove attribute
image.removeAttribute("data-id");

// Toggle attribute
const input = document.getElementById("input");
input.disabled = true;   // Disable
input.disabled = false;  // Enable

// Special case: class
image.className = "featured";  // Replaces all classes
// Better: use classList (see below)

// Special case: style
image.style.cssText = "width: 100px; height: 100px;";
```

### Working with Classes

```html
<div id="box" class="card">Content</div>
```

```javascript
const box = document.getElementById("box");

// classList is the modern way
console.log(box.classList);  // DOMTokenList ["card"]

// Add class
box.classList.add("active");
box.classList.add("large", "featured");  // Multiple

// Remove class
box.classList.remove("card");
box.classList.remove("large", "featured");  // Multiple

// Toggle class (add if missing, remove if present)
box.classList.toggle("active");  // Returns true if added, false if removed

// Conditional toggle
box.classList.toggle("active", true);   // Always adds
box.classList.toggle("active", false);  // Always removes

// Check for class
console.log(box.classList.contains("active"));  // true or false

// Replace class
box.classList.replace("card", "box");

// Get all classes
box.classList.forEach(cls => console.log(cls));

// Old way (avoid)
box.className = "card active";  // Replaces all classes
box.className += " new-class";  // Adds (with space!)
```

### Changing Styles

```html
<div id="box">Content</div>
```

```javascript
const box = document.getElementById("box");

// Individual styles
box.style.width = "200px";
box.style.height = "100px";
box.style.backgroundColor = "blue";  // camelCase!
box.style.border = "2px solid black";
box.style.marginTop = "20px";

// Remove inline style
box.style.backgroundColor = "";  // Removes the property

// Multiple styles at once
box.style.cssText = `
  width: 300px;
  height: 150px;
  background-color: green;
`;

// Or use Object.assign
Object.assign(box.style, {
  width: "250px",
  height: "125px",
  backgroundColor: "yellow"
});

// Better: Add/remove classes for styling
box.classList.add("highlighted");
box.classList.remove("highlighted");

// CSS Variables
box.style.setProperty("--main-color", "purple");
const color = box.style.getPropertyValue("--main-color");
```

---

## Creating and Removing Elements

### Creating Elements

```javascript
// Create element
const newDiv = document.createElement("div");
newDiv.id = "new-div";
newDiv.className = "box";
newDiv.textContent = "I am new!";

// Create with HTML
const template = document.createElement("template");
template.innerHTML = `
  <div class="card">
    <h2>Title</h2>
    <p>Content</p>
  </div>
`;
const card = template.content.firstElementChild;

// Create text node
const textNode = document.createTextNode("Plain text");

// Create document fragment (for multiple elements)
const fragment = document.createDocumentFragment();
for (let i = 0; i < 5; i++) {
  const li = document.createElement("li");
  li.textContent = `Item ${i + 1}`;
  fragment.appendChild(li);
}
// Append all at once (one reflow)
document.querySelector("ul").appendChild(fragment);
```

### Adding Elements to DOM

```html
<div id="container">
  <p id="first">First</p>
  <p id="last">Last</p>
</div>
```

```javascript
const container = document.getElementById("container");
const first = document.getElementById("first");

// Create new element
const newP = document.createElement("p");
newP.textContent = "New paragraph";

// appendChild - add to end
container.appendChild(newP);

// insertBefore - add before specific element
const anotherP = document.createElement("p");
anotherP.textContent = "Before first";
container.insertBefore(anotherP, first);

// Modern: append, prepend, before, after
const p1 = document.createElement("p");
const p2 = document.createElement("p");
p1.textContent = "Prepended";
p2.textContent = "Appended";

container.prepend(p1);    // Add to beginning
container.append(p2);     // Add to end
first.before(newP);       // Add before element
first.after(anotherP);    // Add after element

// Insert multiple at once
container.append(
  document.createElement("p"),
  document.createElement("p"),
  "Plain text"  // Can append text too
);

// insertAdjacentHTML - add HTML string at position
container.insertAdjacentHTML("beforebegin", "<p>Before container</p>");
container.insertAdjacentHTML("afterbegin", "<p>First inside container</p>");
container.insertAdjacentHTML("beforeend", "<p>Last inside container</p>");
container.insertAdjacentHTML("afterend", "<p>After container</p>");

// insertAdjacentElement, insertAdjacentText also available
```

### Removing Elements

```html
<ul id="list">
  <li id="item1">Item 1</li>
  <li id="item2">Item 2</li>
  <li id="item3">Item 3</li>
</ul>
```

```javascript
const list = document.getElementById("list");
const item2 = document.getElementById("item2");

// Modern way: remove()
item2.remove();

// Old way: removeChild() (needs parent reference)
const item3 = document.getElementById("item3");
list.removeChild(item3);

// Remove all children
list.innerHTML = "";  // Simple but destroys event listeners

// Better: remove children one by one
while (list.firstChild) {
  list.removeChild(list.firstChild);
}

// Or use replaceChildren (modern)
list.replaceChildren();  // Remove all
list.replaceChildren(newChild1, newChild2);  // Replace all
```

### Replacing Elements

```javascript
const oldElement = document.getElementById("old");
const newElement = document.createElement("div");
newElement.textContent = "I'm new!";

// Modern: replaceWith()
oldElement.replaceWith(newElement);

// Old way: replaceChild()
const parent = oldElement.parentElement;
parent.replaceChild(newElement, oldElement);
```

### Cloning Elements

```javascript
const original = document.getElementById("original");

// Shallow clone (element only, no children)
const shallowClone = original.cloneNode(false);

// Deep clone (element and all descendants)
const deepClone = original.cloneNode(true);

// Clones include attributes but NOT event listeners
// (unless added with setAttribute for onclick, etc.)
```

---

## Document and Window Objects

### Document Object

```javascript
// Document info
console.log(document.title);           // Page title
console.log(document.URL);             // Current URL
console.log(document.domain);          // Domain name
console.log(document.referrer);        // Previous page URL
console.log(document.characterSet);    // Character encoding
console.log(document.contentType);     // MIME type

// Document structure
console.log(document.documentElement); // <html>
console.log(document.head);            // <head>
console.log(document.body);            // <body>

// All elements of type
console.log(document.forms);           // All forms
console.log(document.images);          // All images
console.log(document.links);           // All links
console.log(document.scripts);         // All scripts

// Document state
console.log(document.readyState);      // "loading" | "interactive" | "complete"

// Modify document
document.title = "New Title";
```

### Window Object

```javascript
// Window dimensions
console.log(window.innerWidth);   // Viewport width
console.log(window.innerHeight);  // Viewport height
console.log(window.outerWidth);   // Browser width
console.log(window.outerHeight);  // Browser height

// Scroll position
console.log(window.scrollX);  // Horizontal scroll
console.log(window.scrollY);  // Vertical scroll

// Screen info
console.log(window.screen.width);
console.log(window.screen.height);
console.log(window.screen.availWidth);
console.log(window.screen.availHeight);

// Location
console.log(window.location.href);      // Full URL
console.log(window.location.hostname);  // Domain
console.log(window.location.pathname);  // Path
console.log(window.location.search);    // Query string
console.log(window.location.hash);      // Fragment

// Navigation
// window.location.href = "https://example.com";  // Navigate
// window.location.reload();  // Reload page

// History
// window.history.back();     // Go back
// window.history.forward();  // Go forward
// window.history.go(-2);     // Go back 2 pages

// Dialogs (avoid in production)
// window.alert("Message");
// const confirmed = window.confirm("Are you sure?");
// const input = window.prompt("Enter name:", "default");

// Scroll
window.scrollTo(0, 100);  // Scroll to position
window.scrollBy(0, 50);   // Scroll by amount
document.querySelector("#section").scrollIntoView();  // Scroll element into view
```

---

## Practical Examples

### Example 1: Todo List

```html
<div id="todo-app">
  <input type="text" id="todo-input" placeholder="Add todo...">
  <button id="add-btn">Add</button>
  <ul id="todo-list"></ul>
</div>
```

```javascript
const input = document.getElementById("todo-input");
const addBtn = document.getElementById("add-btn");
const todoList = document.getElementById("todo-list");

function addTodo() {
  const text = input.value.trim();
  if (!text) return;

  const li = document.createElement("li");
  li.innerHTML = `
    <span class="todo-text">${text}</span>
    <button class="delete-btn">Delete</button>
  `;

  // Add delete functionality
  const deleteBtn = li.querySelector(".delete-btn");
  deleteBtn.addEventListener("click", () => {
    li.remove();
  });

  // Add toggle complete
  const todoText = li.querySelector(".todo-text");
  todoText.addEventListener("click", () => {
    todoText.classList.toggle("completed");
  });

  todoList.appendChild(li);
  input.value = "";
  input.focus();
}

addBtn.addEventListener("click", addTodo);
input.addEventListener("keypress", (e) => {
  if (e.key === "Enter") addTodo();
});
```

### Example 2: Dynamic Content Loader

```javascript
function createCard(data) {
  const card = document.createElement("div");
  card.className = "card";

  card.innerHTML = `
    <img src="${data.image}" alt="${data.title}" class="card-image">
    <div class="card-body">
      <h3 class="card-title">${data.title}</h3>
      <p class="card-text">${data.description}</p>
      <span class="card-price">$${data.price.toFixed(2)}</span>
      <button class="add-to-cart" data-id="${data.id}">Add to Cart</button>
    </div>
  `;

  return card;
}

function renderProducts(products) {
  const container = document.getElementById("products");
  const fragment = document.createDocumentFragment();

  products.forEach(product => {
    const card = createCard(product);
    fragment.appendChild(card);
  });

  container.innerHTML = "";  // Clear existing
  container.appendChild(fragment);
}

// Usage
const products = [
  { id: 1, title: "Product 1", description: "Description 1", price: 29.99, image: "img1.jpg" },
  { id: 2, title: "Product 2", description: "Description 2", price: 49.99, image: "img2.jpg" },
];

renderProducts(products);
```

### Example 3: Tab Component

```html
<div class="tabs">
  <div class="tab-buttons">
    <button class="tab-btn active" data-tab="tab1">Tab 1</button>
    <button class="tab-btn" data-tab="tab2">Tab 2</button>
    <button class="tab-btn" data-tab="tab3">Tab 3</button>
  </div>
  <div class="tab-content">
    <div id="tab1" class="tab-pane active">Content 1</div>
    <div id="tab2" class="tab-pane">Content 2</div>
    <div id="tab3" class="tab-pane">Content 3</div>
  </div>
</div>
```

```javascript
class TabComponent {
  constructor(container) {
    this.container = container;
    this.buttons = container.querySelectorAll(".tab-btn");
    this.panes = container.querySelectorAll(".tab-pane");

    this.init();
  }

  init() {
    this.buttons.forEach(btn => {
      btn.addEventListener("click", () => {
        const tabId = btn.dataset.tab;
        this.switchTab(tabId);
      });
    });
  }

  switchTab(tabId) {
    // Deactivate all
    this.buttons.forEach(btn => btn.classList.remove("active"));
    this.panes.forEach(pane => pane.classList.remove("active"));

    // Activate selected
    const activeBtn = this.container.querySelector(`[data-tab="${tabId}"]`);
    const activePane = this.container.querySelector(`#${tabId}`);

    activeBtn.classList.add("active");
    activePane.classList.add("active");
  }
}

// Initialize
const tabs = new TabComponent(document.querySelector(".tabs"));
```

---

## Performance Tips

### Minimize DOM Access

```javascript
// BAD: Multiple DOM accesses
for (let i = 0; i < 1000; i++) {
  document.getElementById("container").innerHTML += `<p>Item ${i}</p>`;
}

// GOOD: Cache reference, build string, single DOM update
const container = document.getElementById("container");
let html = "";
for (let i = 0; i < 1000; i++) {
  html += `<p>Item ${i}</p>`;
}
container.innerHTML = html;

// BETTER: Use document fragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const p = document.createElement("p");
  p.textContent = `Item ${i}`;
  fragment.appendChild(p);
}
container.appendChild(fragment);
```

### Batch DOM Changes

```javascript
// BAD: Multiple reflows
const box = document.getElementById("box");
box.style.width = "100px";
box.style.height = "100px";
box.style.backgroundColor = "red";
box.style.margin = "10px";

// GOOD: Single reflow using class
box.classList.add("styled-box");

// GOOD: Batch with cssText
box.style.cssText = "width: 100px; height: 100px; background: red; margin: 10px;";

// GOOD: Use requestAnimationFrame for animations
function animate() {
  box.style.left = (parseInt(box.style.left) + 1) + "px";
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

### Use Event Delegation

```javascript
// BAD: Attach listener to each item
document.querySelectorAll(".item").forEach(item => {
  item.addEventListener("click", handleClick);
});

// GOOD: Single listener on parent
document.getElementById("list").addEventListener("click", (e) => {
  if (e.target.classList.contains("item")) {
    handleClick(e);
  }
});
```

---

## Practice Exercises

### Exercise 1: Build a Counter

```html
<div id="counter">
  <button id="decrease">-</button>
  <span id="count">0</span>
  <button id="increase">+</button>
</div>
```

```javascript
// Create a counter that:
// 1. Increases on + click
// 2. Decreases on - click
// 3. Changes color based on value (negative=red, positive=green, zero=black)
```

**Solution:**
```javascript
const decreaseBtn = document.getElementById("decrease");
const increaseBtn = document.getElementById("increase");
const countDisplay = document.getElementById("count");

let count = 0;

function updateDisplay() {
  countDisplay.textContent = count;

  if (count > 0) {
    countDisplay.style.color = "green";
  } else if (count < 0) {
    countDisplay.style.color = "red";
  } else {
    countDisplay.style.color = "black";
  }
}

decreaseBtn.addEventListener("click", () => {
  count--;
  updateDisplay();
});

increaseBtn.addEventListener("click", () => {
  count++;
  updateDisplay();
});
```

### Exercise 2: Dynamic List

```javascript
// Create functions to:
// 1. Add item to list
// 2. Remove item from list
// 3. Move item up/down in list
```

**Solution:**
```javascript
const list = document.getElementById("list");

function addItem(text) {
  const li = document.createElement("li");
  li.innerHTML = `
    <span>${text}</span>
    <button class="up">↑</button>
    <button class="down">↓</button>
    <button class="remove">×</button>
  `;
  list.appendChild(li);
}

list.addEventListener("click", (e) => {
  const li = e.target.closest("li");
  if (!li) return;

  if (e.target.classList.contains("remove")) {
    li.remove();
  } else if (e.target.classList.contains("up")) {
    const prev = li.previousElementSibling;
    if (prev) list.insertBefore(li, prev);
  } else if (e.target.classList.contains("down")) {
    const next = li.nextElementSibling;
    if (next) list.insertBefore(next, li);
  }
});
```

---

## Key Takeaways

1. **Use querySelector/querySelectorAll** - Modern, flexible CSS selector support
2. **Cache DOM references** - Store elements in variables to avoid repeated lookups
3. **Use classList** for class manipulation - More methods than className
4. **Use textContent for text** - innerHTML for HTML (watch XSS risks)
5. **Document fragments** for multiple elements - Single DOM update
6. **Event delegation** - Single listener on parent vs. many on children
7. **requestAnimationFrame** for animations - Smooth, optimized rendering
8. **Batch DOM changes** - Minimize reflows/repaints

---

## Self-Check Questions

1. What's the difference between getElementById and querySelector?
2. What's the difference between HTMLCollection and NodeList?
3. How do you safely insert user content into the DOM?
4. What's the difference between textContent and innerHTML?
5. How do you efficiently add many elements to the DOM?
6. What is event delegation and why use it?
7. How do you traverse from child to parent element?

---

**Next Lesson:** [Week 3, Day 1-2 - DOM Events](./week-03-day-01-02-dom-events.md)
