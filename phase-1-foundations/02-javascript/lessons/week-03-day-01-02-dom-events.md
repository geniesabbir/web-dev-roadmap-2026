# Week 3, Day 1-2: DOM Events

## What are Events?

Events are things that happen in the browser: user clicks, keyboard presses, page loads, form submissions, etc. JavaScript can "listen" for these events and respond to them.

```javascript
// When button is clicked, run this function
button.addEventListener("click", function() {
    console.log("Button was clicked!");
});
```

---

## Event Types

### Mouse Events

```javascript
const element = document.getElementById("box");

// Click events
element.addEventListener("click", () => console.log("clicked"));
element.addEventListener("dblclick", () => console.log("double clicked"));

// Mouse button events
element.addEventListener("mousedown", () => console.log("mouse button pressed"));
element.addEventListener("mouseup", () => console.log("mouse button released"));

// Mouse movement
element.addEventListener("mousemove", () => console.log("mouse moving"));
element.addEventListener("mouseenter", () => console.log("mouse entered"));
element.addEventListener("mouseleave", () => console.log("mouse left"));
element.addEventListener("mouseover", () => console.log("mouse over (bubbles)"));
element.addEventListener("mouseout", () => console.log("mouse out (bubbles)"));

// Context menu (right-click)
element.addEventListener("contextmenu", (e) => {
    e.preventDefault();  // Prevent default menu
    console.log("right clicked");
});
```

### Keyboard Events

```javascript
const input = document.getElementById("input");

// Key events
input.addEventListener("keydown", (e) => {
    console.log(`Key pressed: ${e.key}`);
    console.log(`Key code: ${e.code}`);
});

input.addEventListener("keyup", (e) => {
    console.log(`Key released: ${e.key}`);
});

input.addEventListener("keypress", (e) => {
    // Deprecated - use keydown instead
    console.log(`Key pressed (old): ${e.key}`);
});

// Detect specific keys
document.addEventListener("keydown", (e) => {
    if (e.key === "Escape") {
        console.log("Escape pressed");
    }
    if (e.key === "Enter") {
        console.log("Enter pressed");
    }
    if (e.ctrlKey && e.key === "s") {
        e.preventDefault();  // Prevent browser save
        console.log("Ctrl+S pressed");
    }
    if (e.key === "ArrowUp") {
        console.log("Up arrow pressed");
    }
});

// Key properties
document.addEventListener("keydown", (e) => {
    console.log({
        key: e.key,           // "a", "Enter", "ArrowUp"
        code: e.code,         // "KeyA", "Enter", "ArrowUp"
        ctrlKey: e.ctrlKey,   // true if Ctrl pressed
        shiftKey: e.shiftKey, // true if Shift pressed
        altKey: e.altKey,     // true if Alt pressed
        metaKey: e.metaKey,   // true if Cmd/Win pressed
        repeat: e.repeat      // true if key held down
    });
});
```

### Form Events

```html
<form id="myForm">
    <input type="text" id="name" name="name">
    <input type="email" id="email" name="email">
    <select id="country" name="country">
        <option value="us">USA</option>
        <option value="uk">UK</option>
    </select>
    <button type="submit">Submit</button>
</form>
```

```javascript
const form = document.getElementById("myForm");
const nameInput = document.getElementById("name");
const emailInput = document.getElementById("email");
const countrySelect = document.getElementById("country");

// Form submission
form.addEventListener("submit", (e) => {
    e.preventDefault();  // Prevent page reload

    const formData = new FormData(form);
    console.log(Object.fromEntries(formData));
});

// Input events
nameInput.addEventListener("input", (e) => {
    console.log(`Current value: ${e.target.value}`);
});

nameInput.addEventListener("change", (e) => {
    console.log(`Changed to: ${e.target.value}`);
    // Fires when input loses focus AND value changed
});

// Focus events
nameInput.addEventListener("focus", () => {
    console.log("Input focused");
});

nameInput.addEventListener("blur", () => {
    console.log("Input lost focus");
});

// Select change
countrySelect.addEventListener("change", (e) => {
    console.log(`Selected: ${e.target.value}`);
});

// Form reset
form.addEventListener("reset", () => {
    console.log("Form was reset");
});
```

### Focus Events

```javascript
const input = document.getElementById("input");

// Focus and blur
input.addEventListener("focus", () => {
    console.log("Focused");
    input.style.borderColor = "blue";
});

input.addEventListener("blur", () => {
    console.log("Blurred");
    input.style.borderColor = "";
});

// focusin and focusout (bubble)
const container = document.getElementById("container");
container.addEventListener("focusin", () => {
    console.log("Something inside gained focus");
});

container.addEventListener("focusout", () => {
    console.log("Something inside lost focus");
});
```

### Window/Document Events

```javascript
// Page load events
window.addEventListener("DOMContentLoaded", () => {
    console.log("DOM ready (before images/stylesheets)");
});

window.addEventListener("load", () => {
    console.log("Page fully loaded (including images)");
});

window.addEventListener("beforeunload", (e) => {
    e.preventDefault();
    e.returnValue = "";  // Shows "Leave page?" dialog
});

window.addEventListener("unload", () => {
    console.log("Page is unloading");
});

// Scroll events
window.addEventListener("scroll", () => {
    console.log(`Scrolled to: ${window.scrollY}`);
});

// Resize events
window.addEventListener("resize", () => {
    console.log(`Window size: ${window.innerWidth}x${window.innerHeight}`);
});

// Visibility change (tab switch)
document.addEventListener("visibilitychange", () => {
    if (document.hidden) {
        console.log("Tab is hidden");
    } else {
        console.log("Tab is visible");
    }
});

// Online/Offline
window.addEventListener("online", () => console.log("Back online"));
window.addEventListener("offline", () => console.log("Went offline"));
```

### Drag and Drop Events

```html
<div id="draggable" draggable="true">Drag me</div>
<div id="dropzone">Drop here</div>
```

```javascript
const draggable = document.getElementById("draggable");
const dropzone = document.getElementById("dropzone");

// Drag events (on draggable element)
draggable.addEventListener("dragstart", (e) => {
    e.dataTransfer.setData("text/plain", draggable.id);
    draggable.classList.add("dragging");
});

draggable.addEventListener("dragend", () => {
    draggable.classList.remove("dragging");
});

// Drop events (on drop target)
dropzone.addEventListener("dragover", (e) => {
    e.preventDefault();  // Required to allow drop
    dropzone.classList.add("drag-over");
});

dropzone.addEventListener("dragleave", () => {
    dropzone.classList.remove("drag-over");
});

dropzone.addEventListener("drop", (e) => {
    e.preventDefault();
    const id = e.dataTransfer.getData("text/plain");
    const element = document.getElementById(id);
    dropzone.appendChild(element);
    dropzone.classList.remove("drag-over");
});
```

### Touch Events (Mobile)

```javascript
const element = document.getElementById("touch-area");

element.addEventListener("touchstart", (e) => {
    const touch = e.touches[0];
    console.log(`Touch start at: ${touch.clientX}, ${touch.clientY}`);
});

element.addEventListener("touchmove", (e) => {
    const touch = e.touches[0];
    console.log(`Touch move at: ${touch.clientX}, ${touch.clientY}`);
});

element.addEventListener("touchend", (e) => {
    console.log("Touch ended");
});

element.addEventListener("touchcancel", (e) => {
    console.log("Touch cancelled");
});

// Detect swipe
let startX, startY;

element.addEventListener("touchstart", (e) => {
    startX = e.touches[0].clientX;
    startY = e.touches[0].clientY;
});

element.addEventListener("touchend", (e) => {
    const endX = e.changedTouches[0].clientX;
    const endY = e.changedTouches[0].clientY;

    const diffX = endX - startX;
    const diffY = endY - startY;

    if (Math.abs(diffX) > Math.abs(diffY)) {
        if (diffX > 50) console.log("Swipe right");
        else if (diffX < -50) console.log("Swipe left");
    } else {
        if (diffY > 50) console.log("Swipe down");
        else if (diffY < -50) console.log("Swipe up");
    }
});
```

---

## Adding Event Listeners

### addEventListener (Recommended)

```javascript
const button = document.getElementById("btn");

// Basic usage
button.addEventListener("click", function() {
    console.log("Clicked!");
});

// Arrow function
button.addEventListener("click", () => {
    console.log("Clicked with arrow!");
});

// Named function (for removal)
function handleClick() {
    console.log("Clicked!");
}
button.addEventListener("click", handleClick);

// Multiple listeners for same event
button.addEventListener("click", () => console.log("First"));
button.addEventListener("click", () => console.log("Second"));
// Both run!

// Options object
button.addEventListener("click", handleClick, {
    once: true,      // Remove after first trigger
    passive: true,   // Won't call preventDefault (scroll performance)
    capture: true    // Use capture phase
});
```

### Removing Event Listeners

```javascript
const button = document.getElementById("btn");

function handleClick(e) {
    console.log("Clicked!");
}

// Add listener
button.addEventListener("click", handleClick);

// Remove listener (must use same function reference!)
button.removeEventListener("click", handleClick);

// THIS WON'T WORK (different function references)
button.addEventListener("click", () => console.log("Click"));
button.removeEventListener("click", () => console.log("Click"));  // Different function!

// Using AbortController (modern)
const controller = new AbortController();

button.addEventListener("click", handleClick, {
    signal: controller.signal
});

// Later, remove the listener
controller.abort();

// Remove all listeners (replace element)
const newButton = button.cloneNode(true);
button.parentNode.replaceChild(newButton, button);
```

### HTML Attribute Method (Avoid)

```html
<!-- Inline handlers (avoid!) -->
<button onclick="handleClick()">Click</button>
<button onclick="console.log('clicked')">Click</button>
```

```javascript
// Or via property (avoid)
button.onclick = function() {
    console.log("Clicked");
};

// Problems:
// 1. Can only have one handler
// 2. Mixes HTML and JavaScript
// 3. Less flexible
```

---

## The Event Object

Every event handler receives an event object with information about the event:

```javascript
button.addEventListener("click", function(event) {
    // Or use 'e' as shorthand
    console.log(event);
});
```

### Common Event Properties

```javascript
element.addEventListener("click", (e) => {
    // Event type
    console.log(e.type);  // "click"

    // Target element (where event originated)
    console.log(e.target);  // Element that was clicked

    // Current target (element with listener)
    console.log(e.currentTarget);  // Element with event listener

    // Timestamp
    console.log(e.timeStamp);  // Milliseconds since page load

    // Event phase (1=capture, 2=target, 3=bubble)
    console.log(e.eventPhase);

    // Is event trusted (user action vs script)
    console.log(e.isTrusted);
});
```

### Mouse Event Properties

```javascript
element.addEventListener("click", (e) => {
    // Position relative to viewport
    console.log(e.clientX, e.clientY);

    // Position relative to document
    console.log(e.pageX, e.pageY);

    // Position relative to screen
    console.log(e.screenX, e.screenY);

    // Position relative to element
    console.log(e.offsetX, e.offsetY);

    // Mouse button (0=left, 1=middle, 2=right)
    console.log(e.button);

    // Buttons currently pressed (bitmask)
    console.log(e.buttons);

    // Modifier keys
    console.log(e.ctrlKey, e.shiftKey, e.altKey, e.metaKey);
});
```

### Keyboard Event Properties

```javascript
document.addEventListener("keydown", (e) => {
    // Key pressed
    console.log(e.key);   // "a", "Enter", "ArrowUp", " "
    console.log(e.code);  // "KeyA", "Enter", "ArrowUp", "Space"

    // Deprecated (avoid)
    console.log(e.keyCode);  // 65, 13, 38, 32
    console.log(e.which);    // Same as keyCode

    // Modifier keys
    console.log(e.ctrlKey);   // Ctrl
    console.log(e.shiftKey);  // Shift
    console.log(e.altKey);    // Alt/Option
    console.log(e.metaKey);   // Cmd/Windows

    // Is key held down (repeat)
    console.log(e.repeat);
});
```

---

## Event Propagation

Events travel through the DOM in three phases:

1. **Capture Phase** - Event travels from document down to target
2. **Target Phase** - Event reaches the target element
3. **Bubble Phase** - Event travels back up to document

```html
<div id="outer">
    <div id="inner">
        <button id="btn">Click</button>
    </div>
</div>
```

```javascript
const outer = document.getElementById("outer");
const inner = document.getElementById("inner");
const btn = document.getElementById("btn");

// Bubbling (default) - fires from target upward
outer.addEventListener("click", () => console.log("Outer - bubble"));
inner.addEventListener("click", () => console.log("Inner - bubble"));
btn.addEventListener("click", () => console.log("Button - bubble"));

// Click button: "Button - bubble", "Inner - bubble", "Outer - bubble"

// Capturing - fires from document downward
outer.addEventListener("click", () => console.log("Outer - capture"), true);
inner.addEventListener("click", () => console.log("Inner - capture"), true);
btn.addEventListener("click", () => console.log("Button - capture"), true);

// Click button (all phases):
// "Outer - capture", "Inner - capture", "Button - capture"
// "Button - bubble", "Inner - bubble", "Outer - bubble"
```

### stopPropagation()

Stop event from propagating further:

```javascript
inner.addEventListener("click", (e) => {
    e.stopPropagation();
    console.log("Inner clicked - stops here");
});

// Click inner: Only "Inner clicked" fires
// Outer handler doesn't run
```

### stopImmediatePropagation()

Stop all handlers on current element too:

```javascript
btn.addEventListener("click", (e) => {
    e.stopImmediatePropagation();
    console.log("First handler");
});

btn.addEventListener("click", () => {
    console.log("Second handler");  // Won't run!
});

// Click button: Only "First handler" fires
```

### preventDefault()

Prevent the default browser behavior:

```javascript
// Prevent form submission
form.addEventListener("submit", (e) => {
    e.preventDefault();
    // Handle form data with JavaScript
});

// Prevent link navigation
link.addEventListener("click", (e) => {
    e.preventDefault();
    console.log("Link clicked but not followed");
});

// Prevent context menu
element.addEventListener("contextmenu", (e) => {
    e.preventDefault();
    // Show custom menu
});

// Prevent default keyboard shortcuts
document.addEventListener("keydown", (e) => {
    if (e.ctrlKey && e.key === "s") {
        e.preventDefault();
        // Custom save function
    }
});

// Check if default was prevented
element.addEventListener("click", (e) => {
    console.log(e.defaultPrevented);  // true if prevented
});
```

---

## Event Delegation

Instead of adding listeners to many elements, add one to their parent:

```html
<ul id="list">
    <li data-id="1">Item 1</li>
    <li data-id="2">Item 2</li>
    <li data-id="3">Item 3</li>
</ul>
```

### Without Delegation (Bad)

```javascript
// Adding listener to each item
const items = document.querySelectorAll("li");
items.forEach(item => {
    item.addEventListener("click", (e) => {
        console.log("Clicked:", e.target.dataset.id);
    });
});

// Problems:
// 1. Many listeners (memory)
// 2. New items don't have listeners
// 3. More code to manage
```

### With Delegation (Good)

```javascript
// Single listener on parent
const list = document.getElementById("list");

list.addEventListener("click", (e) => {
    // Check if clicked element is a list item
    if (e.target.tagName === "LI") {
        console.log("Clicked:", e.target.dataset.id);
    }
});

// Benefits:
// 1. Single listener
// 2. Works for dynamically added items
// 3. Less memory usage
```

### Advanced Delegation Patterns

```javascript
// Using closest() for nested elements
document.getElementById("cards").addEventListener("click", (e) => {
    const card = e.target.closest(".card");
    if (!card) return;  // Clicked outside any card

    // Now work with the card
    console.log("Card clicked:", card.dataset.id);

    // Handle specific buttons within card
    if (e.target.matches(".delete-btn")) {
        card.remove();
    } else if (e.target.matches(".edit-btn")) {
        editCard(card);
    }
});

// Multiple element types
document.addEventListener("click", (e) => {
    // Handle different elements
    if (e.target.matches("button.save")) {
        handleSave(e);
    } else if (e.target.matches("button.cancel")) {
        handleCancel(e);
    } else if (e.target.matches("a.external")) {
        e.preventDefault();
        openExternalLink(e.target.href);
    }
});

// Data attribute routing
document.addEventListener("click", (e) => {
    const action = e.target.dataset.action;
    if (action) {
        switch (action) {
            case "save":
                saveData();
                break;
            case "delete":
                deleteItem(e.target.dataset.id);
                break;
            case "toggle":
                toggleState();
                break;
        }
    }
});
```

---

## Custom Events

Create and dispatch your own events:

```javascript
// Create custom event
const myEvent = new CustomEvent("userLogin", {
    detail: {
        username: "john",
        timestamp: Date.now()
    },
    bubbles: true,
    cancelable: true
});

// Listen for custom event
document.addEventListener("userLogin", (e) => {
    console.log("User logged in:", e.detail.username);
    console.log("At:", new Date(e.detail.timestamp));
});

// Dispatch event
document.dispatchEvent(myEvent);

// On specific element
const userElement = document.getElementById("user");
userElement.dispatchEvent(myEvent);

// Check if event was cancelled
const userElement2 = document.getElementById("user2");
userElement2.addEventListener("userLogin", (e) => {
    if (someCondition) {
        e.preventDefault();  // Cancel the event
    }
});

const notCancelled = userElement2.dispatchEvent(myEvent);
if (notCancelled) {
    console.log("Event was not cancelled");
}
```

### Event Communication Pattern

```javascript
// Pub/Sub with custom events
class EventBus {
    constructor() {
        this.element = document.createElement("div");
    }

    on(event, callback) {
        this.element.addEventListener(event, (e) => callback(e.detail));
    }

    off(event, callback) {
        this.element.removeEventListener(event, callback);
    }

    emit(event, data) {
        this.element.dispatchEvent(new CustomEvent(event, { detail: data }));
    }
}

// Usage
const bus = new EventBus();

bus.on("itemAdded", (data) => {
    console.log("Item added:", data);
});

bus.emit("itemAdded", { id: 1, name: "New Item" });
```

---

## Debouncing and Throttling

Control how often event handlers run:

### Debounce

Wait until action stops, then run once:

```javascript
function debounce(fn, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
            fn.apply(this, args);
        }, delay);
    };
}

// Usage: Search input
const searchInput = document.getElementById("search");

const handleSearch = debounce((e) => {
    console.log("Searching for:", e.target.value);
    // Make API call
}, 300);

searchInput.addEventListener("input", handleSearch);
// Only fires 300ms after user stops typing
```

### Throttle

Run at most once per time period:

```javascript
function throttle(fn, limit) {
    let lastRun = 0;
    return function(...args) {
        const now = Date.now();
        if (now - lastRun >= limit) {
            lastRun = now;
            fn.apply(this, args);
        }
    };
}

// Usage: Scroll handler
const handleScroll = throttle(() => {
    console.log("Scroll position:", window.scrollY);
}, 100);

window.addEventListener("scroll", handleScroll);
// Fires at most every 100ms while scrolling
```

### requestAnimationFrame Throttle

For visual updates:

```javascript
function rafThrottle(fn) {
    let rafId = null;
    return function(...args) {
        if (rafId) return;
        rafId = requestAnimationFrame(() => {
            fn.apply(this, args);
            rafId = null;
        });
    };
}

// Usage: Animation during scroll
const handleScrollAnimation = rafThrottle(() => {
    // Update element positions
    updateParallax();
});

window.addEventListener("scroll", handleScrollAnimation);
```

---

## Practical Examples

### Example 1: Modal Dialog

```html
<button id="openModal">Open Modal</button>
<div id="modal" class="modal hidden">
    <div class="modal-content">
        <span class="close">&times;</span>
        <h2>Modal Title</h2>
        <p>Modal content here...</p>
    </div>
</div>
```

```javascript
const modal = document.getElementById("modal");
const openBtn = document.getElementById("openModal");
const closeBtn = modal.querySelector(".close");

function openModal() {
    modal.classList.remove("hidden");
    document.body.style.overflow = "hidden";
}

function closeModal() {
    modal.classList.add("hidden");
    document.body.style.overflow = "";
}

// Open modal
openBtn.addEventListener("click", openModal);

// Close modal - X button
closeBtn.addEventListener("click", closeModal);

// Close modal - click outside
modal.addEventListener("click", (e) => {
    if (e.target === modal) {
        closeModal();
    }
});

// Close modal - Escape key
document.addEventListener("keydown", (e) => {
    if (e.key === "Escape" && !modal.classList.contains("hidden")) {
        closeModal();
    }
});
```

### Example 2: Form Validation

```html
<form id="registerForm">
    <div class="form-group">
        <input type="text" id="username" placeholder="Username">
        <span class="error" id="usernameError"></span>
    </div>
    <div class="form-group">
        <input type="email" id="email" placeholder="Email">
        <span class="error" id="emailError"></span>
    </div>
    <div class="form-group">
        <input type="password" id="password" placeholder="Password">
        <span class="error" id="passwordError"></span>
    </div>
    <button type="submit">Register</button>
</form>
```

```javascript
const form = document.getElementById("registerForm");

const validators = {
    username: {
        validate: (value) => value.length >= 3,
        message: "Username must be at least 3 characters"
    },
    email: {
        validate: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
        message: "Please enter a valid email"
    },
    password: {
        validate: (value) => value.length >= 8,
        message: "Password must be at least 8 characters"
    }
};

function validateField(field) {
    const value = field.value.trim();
    const validator = validators[field.id];
    const errorElement = document.getElementById(`${field.id}Error`);

    if (!validator.validate(value)) {
        field.classList.add("invalid");
        field.classList.remove("valid");
        errorElement.textContent = validator.message;
        return false;
    } else {
        field.classList.add("valid");
        field.classList.remove("invalid");
        errorElement.textContent = "";
        return true;
    }
}

// Validate on input (debounced)
const debouncedValidate = debounce((e) => {
    validateField(e.target);
}, 300);

Object.keys(validators).forEach(id => {
    const field = document.getElementById(id);
    field.addEventListener("input", debouncedValidate);
    field.addEventListener("blur", (e) => validateField(e.target));
});

// Validate on submit
form.addEventListener("submit", (e) => {
    e.preventDefault();

    let isValid = true;
    Object.keys(validators).forEach(id => {
        const field = document.getElementById(id);
        if (!validateField(field)) {
            isValid = false;
        }
    });

    if (isValid) {
        console.log("Form is valid! Submitting...");
        // Submit form data
    }
});

function debounce(fn, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn.apply(this, args), delay);
    };
}
```

### Example 3: Infinite Scroll

```javascript
const container = document.getElementById("content");
let page = 1;
let loading = false;

async function loadMore() {
    if (loading) return;
    loading = true;

    showLoadingSpinner();

    try {
        const response = await fetch(`/api/items?page=${page}`);
        const items = await response.json();

        items.forEach(item => {
            const element = createItemElement(item);
            container.appendChild(element);
        });

        page++;
    } catch (error) {
        console.error("Failed to load items:", error);
    } finally {
        loading = false;
        hideLoadingSpinner();
    }
}

// Throttled scroll handler
const handleScroll = throttle(() => {
    const scrollTop = window.scrollY;
    const windowHeight = window.innerHeight;
    const documentHeight = document.documentElement.scrollHeight;

    // Load more when near bottom (100px threshold)
    if (scrollTop + windowHeight >= documentHeight - 100) {
        loadMore();
    }
}, 200);

window.addEventListener("scroll", handleScroll);

// Alternative: Intersection Observer (modern)
const observer = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting) {
        loadMore();
    }
}, { rootMargin: "100px" });

observer.observe(document.getElementById("loadTrigger"));
```

### Example 4: Drag and Drop List

```html
<ul id="sortable">
    <li draggable="true" data-id="1">Item 1</li>
    <li draggable="true" data-id="2">Item 2</li>
    <li draggable="true" data-id="3">Item 3</li>
</ul>
```

```javascript
const list = document.getElementById("sortable");
let draggedItem = null;

list.addEventListener("dragstart", (e) => {
    if (e.target.tagName !== "LI") return;

    draggedItem = e.target;
    e.target.classList.add("dragging");

    // Required for Firefox
    e.dataTransfer.effectAllowed = "move";
    e.dataTransfer.setData("text/html", e.target.innerHTML);
});

list.addEventListener("dragover", (e) => {
    e.preventDefault();
    const afterElement = getDragAfterElement(list, e.clientY);
    const dragging = document.querySelector(".dragging");

    if (afterElement) {
        list.insertBefore(dragging, afterElement);
    } else {
        list.appendChild(dragging);
    }
});

list.addEventListener("dragend", (e) => {
    e.target.classList.remove("dragging");
    draggedItem = null;

    // Save new order
    const items = [...list.querySelectorAll("li")];
    const order = items.map(item => item.dataset.id);
    console.log("New order:", order);
});

function getDragAfterElement(container, y) {
    const draggableElements = [...container.querySelectorAll("li:not(.dragging)")];

    return draggableElements.reduce((closest, child) => {
        const box = child.getBoundingClientRect();
        const offset = y - box.top - box.height / 2;

        if (offset < 0 && offset > closest.offset) {
            return { offset, element: child };
        } else {
            return closest;
        }
    }, { offset: Number.NEGATIVE_INFINITY }).element;
}
```

---

## Practice Exercises

### Exercise 1: Keyboard Shortcuts

```javascript
// Create a keyboard shortcut system:
// Ctrl+S: Save
// Ctrl+Z: Undo
// Ctrl+Shift+Z: Redo
// Escape: Close modal
```

**Solution:**
```javascript
const shortcuts = {
    "ctrl+s": () => { console.log("Save"); },
    "ctrl+z": () => { console.log("Undo"); },
    "ctrl+shift+z": () => { console.log("Redo"); },
    "escape": () => { console.log("Close modal"); }
};

document.addEventListener("keydown", (e) => {
    const parts = [];

    if (e.ctrlKey) parts.push("ctrl");
    if (e.shiftKey) parts.push("shift");
    if (e.altKey) parts.push("alt");
    parts.push(e.key.toLowerCase());

    const combo = parts.join("+");

    if (shortcuts[combo]) {
        e.preventDefault();
        shortcuts[combo]();
    }
});
```

### Exercise 2: Double Click vs Single Click

```javascript
// Differentiate between single and double click
// Single click: Select item
// Double click: Edit item
```

**Solution:**
```javascript
let clickTimer = null;
const DOUBLE_CLICK_DELAY = 300;

element.addEventListener("click", (e) => {
    if (clickTimer) {
        // Double click
        clearTimeout(clickTimer);
        clickTimer = null;
        handleDoubleClick(e);
    } else {
        // Possible single click - wait to see if double
        clickTimer = setTimeout(() => {
            handleSingleClick(e);
            clickTimer = null;
        }, DOUBLE_CLICK_DELAY);
    }
});

function handleSingleClick(e) {
    console.log("Single click - select item");
}

function handleDoubleClick(e) {
    console.log("Double click - edit item");
}
```

---

## Key Takeaways

1. **Use addEventListener** - More flexible than onclick
2. **Event delegation** - Single listener on parent for multiple children
3. **Event object** - Contains all information about the event
4. **preventDefault()** - Stops default browser behavior
5. **stopPropagation()** - Stops event from bubbling/capturing
6. **Debounce** for input/resize - Wait for user to stop
7. **Throttle** for scroll/mousemove - Limit frequency
8. **Custom events** - Create your own event-based communication

---

## Self-Check Questions

1. What's the difference between `target` and `currentTarget`?
2. What are the three phases of event propagation?
3. When would you use event delegation?
4. What's the difference between debounce and throttle?
5. How do you prevent a form from submitting?
6. How do you detect keyboard shortcuts?
7. What's the difference between `mouseenter` and `mouseover`?

---

**Next Lesson:** [Day 3-4 - Async JavaScript](./week-03-day-03-04-async-js.md)
