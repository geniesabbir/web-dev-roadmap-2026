# JavaScript

**Duration:** 3-4 weeks

## Learning Objectives

By the end of this section, you will:
- Understand JavaScript fundamentals deeply
- Manipulate the DOM confidently
- Handle events and user interactions
- Work with asynchronous code
- Use modern ES6+ features
- Debug JavaScript effectively

---

## Week 1: Core Fundamentals

### Day 1: Variables & Data Types
**Topics:**
- `var`, `let`, `const` (and why to avoid `var`)
- Primitive types (string, number, boolean, null, undefined, symbol, bigint)
- Reference types (objects, arrays)
- Type coercion and `typeof`
- Template literals

**Exercises:**
```javascript
// exercises/01-variables.js

// 1. Declare variables for a user profile
// 2. Practice string interpolation
// 3. Explore type coercion quirks
// 4. Understand null vs undefined
```

### Day 2: Operators & Control Flow
**Topics:**
- Arithmetic operators
- Comparison operators (`==` vs `===`)
- Logical operators (`&&`, `||`, `!`, `??`)
- Conditionals (`if`, `else if`, `else`, ternary)
- Switch statements
- Truthy and falsy values

**Exercises:**
```javascript
// exercises/02-control-flow.js

// 1. Build a grade calculator
// 2. Create a simple tax calculator
// 3. Write a day-of-week switch statement
```

### Day 3: Functions
**Topics:**
- Function declarations vs expressions
- Arrow functions
- Parameters and arguments
- Default parameters
- Rest parameters
- Return values
- Pure functions

**Exercises:**
```javascript
// exercises/03-functions.js

// 1. Convert between function syntaxes
// 2. Build a tip calculator function
// 3. Create a greeting generator with defaults
// 4. Write a function that accepts unlimited arguments
```

### Day 4: Scope & Closures
**Topics:**
- Global scope
- Function scope
- Block scope
- Lexical scope
- Closures and their uses
- The module pattern

**Exercises:**
```javascript
// exercises/04-scope-closures.js

// 1. Identify scope in code snippets
// 2. Create a counter using closures
// 3. Build a private variable pattern
```

### Day 5: Arrays
**Topics:**
- Creating and accessing arrays
- Array methods: `push`, `pop`, `shift`, `unshift`
- Iteration: `forEach`, `map`, `filter`, `reduce`
- Searching: `find`, `findIndex`, `includes`
- Sorting and reversing
- Spread operator with arrays

**Exercises:**
```javascript
// exercises/05-arrays.js

// 1. Filter an array of products by price
// 2. Map user objects to usernames
// 3. Reduce an array to calculate totals
// 4. Chain multiple array methods
```

---

## Week 2: Objects & The DOM

### Day 1: Objects
**Topics:**
- Object literals
- Accessing properties (dot vs bracket notation)
- Adding/removing properties
- Object methods
- `this` keyword basics
- Object destructuring
- Spread operator with objects

**Exercises:**
```javascript
// exercises/06-objects.js

// 1. Create a book object with methods
// 2. Practice destructuring
// 3. Merge objects with spread
// 4. Dynamic property access
```

### Day 2: Advanced Objects
**Topics:**
- Constructor functions
- Prototypes and prototype chain
- ES6 Classes
- Inheritance with `extends`
- Static methods
- Getters and setters

**Exercises:**
```javascript
// exercises/07-classes.js

// 1. Create a BankAccount class
// 2. Extend it to SavingsAccount
// 3. Use getters for computed properties
```

### Day 3: DOM Fundamentals
**Topics:**
- What is the DOM?
- Selecting elements (`getElementById`, `querySelector`, `querySelectorAll`)
- Traversing the DOM (parent, children, siblings)
- Creating elements
- Modifying content (`textContent`, `innerHTML`)
- Modifying attributes

**Exercises:**
```html
<!-- exercises/08-dom-basics/index.html -->
<!--
1. Select and log various elements
2. Change text content dynamically
3. Build a simple element tree
-->
```

### Day 4: DOM Manipulation
**Topics:**
- Adding/removing classes (`classList`)
- Modifying styles
- Creating and appending elements
- Removing elements
- Cloning elements
- Document fragments

**Exercises:**
```html
<!-- exercises/09-dom-manipulation/index.html -->
<!--
1. Build a todo list (add/remove items)
2. Toggle dark mode with class manipulation
3. Create a dynamic card generator
-->
```

### Day 5: Events
**Topics:**
- Event listeners (`addEventListener`)
- Event object
- Event bubbling and capturing
- Event delegation
- Preventing default behavior
- Common events (click, submit, input, keydown, load)

**Exercises:**
```html
<!-- exercises/10-events/index.html -->
<!--
1. Form validation on submit
2. Real-time input filtering
3. Keyboard shortcuts
4. Click outside to close modal
-->
```

---

## Week 3: Async JavaScript & Modern Features

### Day 1: Asynchronous Basics
**Topics:**
- Synchronous vs asynchronous
- Callbacks
- Callback hell
- `setTimeout` and `setInterval`

**Exercises:**
```javascript
// exercises/11-async-basics.js

// 1. Simulate async operations with setTimeout
// 2. Experience callback hell
// 3. Build a countdown timer
```

### Day 2: Promises
**Topics:**
- Creating promises
- `.then()`, `.catch()`, `.finally()`
- Promise chaining
- `Promise.all()`, `Promise.race()`
- Error handling in promises

**Exercises:**
```javascript
// exercises/12-promises.js

// 1. Convert callback functions to promises
// 2. Chain multiple API-like calls
// 3. Handle errors gracefully
// 4. Use Promise.all for parallel operations
```

### Day 3: Async/Await
**Topics:**
- `async` functions
- `await` keyword
- Error handling with try/catch
- Parallel execution with async/await
- Top-level await

**Exercises:**
```javascript
// exercises/13-async-await.js

// 1. Refactor promise chains to async/await
// 2. Fetch data from a public API
// 3. Handle multiple async operations
```

### Day 4: Fetch API & HTTP
**Topics:**
- Fetch basics
- Request methods (GET, POST, PUT, DELETE)
- Headers and options
- Handling responses
- JSON parsing
- Error handling with fetch

**Exercises:**
```javascript
// exercises/14-fetch.js

// 1. Fetch users from JSONPlaceholder API
// 2. Post new data to an API
// 3. Build a simple API wrapper function
```

### Day 5: Error Handling
**Topics:**
- `try`, `catch`, `finally`
- `throw` statement
- Error types
- Custom errors
- Debugging with console methods
- Using breakpoints

**Exercises:**
```javascript
// exercises/15-error-handling.js

// 1. Create custom error classes
// 2. Build a robust API fetcher with error handling
// 3. Practice debugging techniques
```

---

## Week 4: ES6+ & Practical Skills

### Day 1: ES6+ Features
**Topics:**
- Destructuring (deep dive)
- Spread and rest operators
- Optional chaining (`?.`)
- Nullish coalescing (`??`)
- Short-circuit evaluation
- Object shorthand

**Exercises:**
```javascript
// exercises/16-es6-features.js

// 1. Deep destructuring practice
// 2. Safely access nested properties
// 3. Default values with nullish coalescing
```

### Day 2: Modules
**Topics:**
- Why modules?
- `export` and `import`
- Named vs default exports
- Re-exporting
- Dynamic imports
- Module patterns

**Exercises:**
```javascript
// exercises/17-modules/
// Create a modular calculator application

// math.js - export math functions
// utils.js - export utility functions
// main.js - import and use
```

### Day 3: Local Storage & JSON
**Topics:**
- `localStorage` API
- `sessionStorage`
- `JSON.stringify()` and `JSON.parse()`
- Storing complex data
- Storage events

**Exercises:**
```javascript
// exercises/18-storage.js

// 1. Persist todo list to localStorage
// 2. Build a theme preference saver
// 3. Create a simple caching mechanism
```

### Day 4-5: Mini Projects
Build these projects using everything learned:

**Project 1: Quiz App**
- Multiple choice questions
- Score tracking
- Timer
- Results screen

**Project 2: Weather Dashboard**
- Fetch from weather API
- Display current weather
- Search by city
- Save favorite cities

---

## Coding Challenges

Practice these patterns regularly:

```javascript
// challenges/

// 1. Reverse a string
// 2. Check if palindrome
// 3. FizzBuzz
// 4. Find duplicates in array
// 5. Flatten nested array
// 6. Debounce function
// 7. Deep clone an object
// 8. Group array by property
```

---

## Resources

### Documentation
- [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
- [JavaScript.info](https://javascript.info/)

### Practice
- [Exercism JavaScript Track](https://exercism.org/tracks/javascript)
- [JavaScript30](https://javascript30.com/) (Free)
- [LeetCode](https://leetcode.com/) (For algorithms)

### Videos
- [Traversy Media](https://www.youtube.com/c/TraversyMedia)
- [Fireship](https://www.youtube.com/c/Fireship)

---

## Checklist Before Moving On

- [ ] Understand `let`, `const`, and why not `var`
- [ ] Can write and use functions (including arrow functions)
- [ ] Understand scope and closures
- [ ] Can manipulate arrays with `map`, `filter`, `reduce`
- [ ] Can work with objects and classes
- [ ] Can select and manipulate DOM elements
- [ ] Can handle events effectively
- [ ] Understand Promises and async/await
- [ ] Can fetch data from APIs
- [ ] Can handle errors properly
- [ ] Completed both mini projects

---

**Next:** [TypeScript](../03-typescript/README.md)
