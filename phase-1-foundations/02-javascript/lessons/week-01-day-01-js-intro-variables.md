# Week 1, Day 1: JavaScript Introduction & Variables

## What is JavaScript?

JavaScript is a programming language that makes websites interactive. While HTML provides structure and CSS provides styling, JavaScript provides **behavior**.

**JavaScript can:**
- Respond to user actions (clicks, typing, scrolling)
- Update content dynamically without page reload
- Validate forms before submission
- Create animations and effects
- Communicate with servers (fetch data)
- Build full applications (frontend and backend)

---

## Running JavaScript

### In the Browser Console

1. Open any webpage
2. Right-click → Inspect (or press F12)
3. Go to the Console tab
4. Type JavaScript and press Enter

```javascript
console.log("Hello, World!");
```

### In an HTML File

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Page</title>
</head>
<body>
    <h1>Hello</h1>

    <!-- Inline JavaScript (avoid for large code) -->
    <script>
        console.log("Hello from inline script!");
    </script>

    <!-- External JavaScript (recommended) -->
    <script src="script.js"></script>
</body>
</html>
```

**script.js:**
```javascript
console.log("Hello from external file!");
```

**Best practice:** Place `<script>` tags at the end of `<body>` or use `defer`:

```html
<head>
    <script src="script.js" defer></script>
</head>
```

### In Node.js

```bash
# Create a file
echo 'console.log("Hello from Node!");' > app.js

# Run it
node app.js
```

---

## Console Methods

The console is your debugging best friend:

```javascript
// Basic logging
console.log("Regular message");

// Warnings (yellow)
console.warn("This is a warning");

// Errors (red)
console.error("This is an error");

// Informational (blue icon)
console.info("FYI message");

// Tabular data
console.table([
    { name: "John", age: 25 },
    { name: "Jane", age: 30 }
]);

// Grouping
console.group("User Details");
console.log("Name: John");
console.log("Age: 25");
console.groupEnd();

// Timing
console.time("Loop");
for (let i = 0; i < 1000000; i++) {}
console.timeEnd("Loop");  // Loop: 5.123ms

// Count calls
function myFunction() {
    console.count("myFunction called");
}
myFunction();  // myFunction called: 1
myFunction();  // myFunction called: 2

// Clear console
console.clear();
```

---

## Comments

Comments explain your code and are ignored by JavaScript:

```javascript
// This is a single-line comment

/*
   This is a
   multi-line comment
*/

/**
 * This is a documentation comment (JSDoc)
 * Used to document functions
 * @param {string} name - The user's name
 * @returns {string} A greeting message
 */
function greet(name) {
    return "Hello, " + name;
}
```

---

## Variables

Variables store data values. Think of them as labeled boxes that hold information.

### Declaring Variables

JavaScript has three ways to declare variables:

```javascript
var oldWay = "avoid this";     // Old way (function-scoped)
let changeable = "use this";   // Modern way (block-scoped)
const constant = "can't change"; // For values that won't change
```

### let - For Variables That Change

```javascript
let age = 25;
console.log(age);  // 25

age = 26;          // Can reassign
console.log(age);  // 26

let name;          // Can declare without value
name = "John";     // Assign later
```

### const - For Variables That Don't Change

```javascript
const PI = 3.14159;
console.log(PI);  // 3.14159

PI = 3.14;        // ERROR! Cannot reassign

const name;       // ERROR! Must initialize
```

**When to use const:**
- Default to `const` for everything
- Only use `let` when you need to reassign
- Never use `var` (legacy code only)

### var - The Old Way (Avoid)

```javascript
var x = 10;

// Problems with var:
// 1. Function-scoped, not block-scoped
if (true) {
    var y = 20;
}
console.log(y);  // 20 - Still accessible! (bad)

// 2. Can be redeclared
var x = 10;
var x = 20;  // No error (confusing)

// 3. Hoisting issues (more on this later)
```

---

## Variable Naming Rules

### Valid Names

```javascript
let name = "John";
let firstName = "John";
let first_name = "John";
let _private = "internal";
let $element = document.body;
let camelCase = "preferred";
let PascalCase = "for classes";
let SCREAMING_CASE = "for constants";
let name2 = "can include numbers";
```

### Invalid Names

```javascript
let 2name = "bad";       // Can't start with number
let my-name = "bad";     // No hyphens
let my name = "bad";     // No spaces
let class = "bad";       // Reserved keyword
let function = "bad";    // Reserved keyword
```

### Naming Conventions

```javascript
// camelCase for variables and functions
let firstName = "John";
let calculateTotal = function() {};

// PascalCase for classes and constructors
class UserAccount {}
function Person() {}

// SCREAMING_SNAKE_CASE for constants
const MAX_SIZE = 100;
const API_KEY = "abc123";

// Descriptive names
let x = 5;           // Bad - what is x?
let userAge = 5;     // Good - clear meaning

let temp = getData(); // Bad - what temp?
let userData = getData(); // Good - clear
```

---

## Variable Scope

Scope determines where variables are accessible.

### Block Scope (let and const)

```javascript
{
    let blockScoped = "only here";
    const alsoBlockScoped = "only here too";
}
console.log(blockScoped);  // ERROR! Not defined

if (true) {
    let x = 10;
}
console.log(x);  // ERROR! x is not defined

for (let i = 0; i < 3; i++) {
    console.log(i);  // 0, 1, 2
}
console.log(i);  // ERROR! i is not defined
```

### Function Scope

```javascript
function myFunction() {
    let functionScoped = "only in this function";
    var alsoFunctionScoped = "same here";
}

console.log(functionScoped);  // ERROR!
console.log(alsoFunctionScoped);  // ERROR!
```

### Global Scope

```javascript
let globalVariable = "accessible everywhere";

function myFunction() {
    console.log(globalVariable);  // Works!
}

if (true) {
    console.log(globalVariable);  // Works!
}
```

**Avoid global variables when possible** - they can cause naming conflicts and bugs.

---

## Hoisting

JavaScript moves declarations to the top of their scope before execution.

### var Hoisting

```javascript
console.log(x);  // undefined (not error!)
var x = 5;
console.log(x);  // 5

// JavaScript sees it as:
var x;           // Declaration hoisted
console.log(x);  // undefined
x = 5;           // Assignment stays
console.log(x);  // 5
```

### let and const Hoisting (Temporal Dead Zone)

```javascript
console.log(x);  // ERROR! Cannot access before initialization
let x = 5;

// The variable exists but is in the "Temporal Dead Zone"
// until the declaration is reached
```

### Function Hoisting

```javascript
// Function declarations are fully hoisted
sayHello();  // Works! "Hello"

function sayHello() {
    console.log("Hello");
}

// Function expressions are NOT hoisted
sayGoodbye();  // ERROR! Cannot access before initialization

const sayGoodbye = function() {
    console.log("Goodbye");
};
```

---

## Practical Examples

### Swapping Variables

```javascript
let a = 1;
let b = 2;

// Using a temporary variable
let temp = a;
a = b;
b = temp;
console.log(a, b);  // 2, 1

// Modern way with destructuring
let x = 1;
let y = 2;
[x, y] = [y, x];
console.log(x, y);  // 2, 1
```

### Counters

```javascript
let count = 0;

function increment() {
    count = count + 1;  // or count++
    console.log(count);
}

increment();  // 1
increment();  // 2
increment();  // 3
```

### Configuration

```javascript
const CONFIG = {
    apiUrl: "https://api.example.com",
    maxRetries: 3,
    timeout: 5000
};

// CONFIG = {}; // ERROR! Can't reassign
CONFIG.timeout = 10000;  // But can modify properties!
```

### User Information

```javascript
let isLoggedIn = false;
let userName = "";
const MAX_LOGIN_ATTEMPTS = 3;
let loginAttempts = 0;

function login(name) {
    if (loginAttempts >= MAX_LOGIN_ATTEMPTS) {
        console.log("Too many attempts");
        return;
    }

    isLoggedIn = true;
    userName = name;
    console.log(`Welcome, ${userName}!`);
}

function logout() {
    isLoggedIn = false;
    userName = "";
    console.log("Goodbye!");
}
```

---

## Common Mistakes

### 1. Forgetting to Declare

```javascript
// Bad - creates global variable accidentally
function addNumbers() {
    result = 5 + 3;  // Missing let/const!
}
addNumbers();
console.log(result);  // 8 - Oops, it's global!

// Good
function addNumbers() {
    const result = 5 + 3;
    return result;
}
```

### 2. Using const for Arrays/Objects You Modify

```javascript
// This is actually FINE
const numbers = [1, 2, 3];
numbers.push(4);  // OK - modifying, not reassigning
console.log(numbers);  // [1, 2, 3, 4]

numbers = [5, 6, 7];  // ERROR! Can't reassign

// Same with objects
const user = { name: "John" };
user.name = "Jane";  // OK
user = {};  // ERROR!
```

### 3. Redeclaring with let

```javascript
let x = 10;
let x = 20;  // ERROR! Already declared

// Use reassignment instead
let y = 10;
y = 20;  // OK
```

---

## Exercises

### Exercise 1: Variable Practice

```javascript
// 1. Declare a variable for your name
// 2. Declare a constant for your birth year
// 3. Calculate and store your age
// 4. Log a message with all three

// Your code here:
```

**Solution:**
```javascript
const myName = "John";
const birthYear = 1995;
let age = 2024 - birthYear;
console.log(`My name is ${myName}, I was born in ${birthYear}, and I'm ${age} years old.`);
```

### Exercise 2: Scope Understanding

```javascript
// What will this print?
let a = 1;

if (true) {
    let a = 2;
    console.log(a);  // ?
}

console.log(a);  // ?

// Answer: 2, then 1
```

### Exercise 3: Fix the Bug

```javascript
// This code has bugs. Fix them!
const counter = 0;

function incrementCounter() {
    counter++;
    console.log(counter);
}

incrementCounter();
```

**Solution:**
```javascript
let counter = 0;  // Changed const to let

function incrementCounter() {
    counter++;
    console.log(counter);
}

incrementCounter();  // 1
```

---

## Key Takeaways

1. **Use `const` by default**, `let` when reassignment is needed
2. **Never use `var`** - it has scoping issues
3. **Use meaningful names** - `userAge` not `x`
4. **camelCase** for variables and functions
5. **Variables are block-scoped** with let/const
6. **Be aware of hoisting** - declare before use

---

## Self-Check Questions

1. What's the difference between `let` and `const`?
2. Why should you avoid `var`?
3. What is block scope?
4. What is hoisting?
5. Can you modify an array declared with `const`?

---

**Next Lesson:** [Day 2 - Data Types](./week-01-day-02-data-types.md)
