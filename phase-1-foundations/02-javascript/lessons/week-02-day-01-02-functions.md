# Week 2, Day 1-2: Functions

## What are Functions?

Functions are reusable blocks of code that perform specific tasks. They help you:

- **Organize code** into logical units
- **Reuse code** without repetition (DRY - Don't Repeat Yourself)
- **Abstract complexity** behind simple names
- **Test code** more easily

---

## Function Declaration

The traditional way to define functions:

```javascript
// Syntax
function functionName(parameters) {
    // code to execute
    return value;  // optional
}

// Basic function
function greet() {
    console.log("Hello!");
}

// Call the function
greet();  // "Hello!"
greet();  // "Hello!" (can call multiple times)

// Function with parameter
function greetUser(name) {
    console.log(`Hello, ${name}!`);
}

greetUser("John");  // "Hello, John!"
greetUser("Jane");  // "Hello, Jane!"

// Function with return value
function add(a, b) {
    return a + b;
}

let sum = add(5, 3);
console.log(sum);  // 8

// Using return value directly
console.log(add(10, 20));  // 30
console.log(add(add(1, 2), 3));  // 6
```

### Function Hoisting

Function declarations are hoisted (moved to top):

```javascript
// Can call before declaration
sayHello();  // "Hello!" (works!)

function sayHello() {
    console.log("Hello!");
}

// But variables assigned to functions are NOT hoisted
sayGoodbye();  // ERROR: Cannot access before initialization

const sayGoodbye = function() {
    console.log("Goodbye!");
};
```

---

## Function Expression

Functions assigned to variables:

```javascript
// Anonymous function expression
const greet = function() {
    console.log("Hello!");
};

greet();  // "Hello!"

// Named function expression (useful for recursion and debugging)
const factorial = function fact(n) {
    if (n <= 1) return 1;
    return n * fact(n - 1);  // Can reference itself
};

console.log(factorial(5));  // 120

// The name is only available inside the function
// console.log(fact(5));  // ERROR: fact is not defined
```

### Declaration vs Expression

```javascript
// Declaration - hoisted, can use before definition
function declaredFunc() {
    return "I'm declared";
}

// Expression - NOT hoisted, must define before use
const expressedFunc = function() {
    return "I'm expressed";
};

// Both work the same once defined
console.log(declaredFunc());   // "I'm declared"
console.log(expressedFunc());  // "I'm expressed"
```

---

## Arrow Functions (ES6)

Modern, concise syntax for functions:

```javascript
// Basic arrow function
const greet = () => {
    console.log("Hello!");
};

// With one parameter (parentheses optional)
const double = n => {
    return n * 2;
};

// Or simply:
const double = n => n * 2;

// With multiple parameters (parentheses required)
const add = (a, b) => a + b;

// Returning object (wrap in parentheses)
const createUser = (name, age) => ({ name, age });
console.log(createUser("John", 25));  // { name: "John", age: 25 }

// Multi-line with explicit return
const calculate = (a, b) => {
    const sum = a + b;
    const product = a * b;
    return { sum, product };
};
```

### Arrow Function vs Regular Function

```javascript
// 1. Syntax is shorter
const regular = function(x) { return x * 2; };
const arrow = x => x * 2;

// 2. No own 'this' binding (important!)
const obj = {
    name: "John",
    regularGreet: function() {
        console.log(`Hello, ${this.name}`);
    },
    arrowGreet: () => {
        console.log(`Hello, ${this.name}`);  // 'this' is NOT the object!
    }
};

obj.regularGreet();  // "Hello, John"
obj.arrowGreet();    // "Hello, undefined" (arrow doesn't bind 'this')

// 3. Arrow functions are great for callbacks
const numbers = [1, 2, 3, 4, 5];

// Regular function
numbers.map(function(n) {
    return n * 2;
});

// Arrow function (cleaner)
numbers.map(n => n * 2);

// 4. Arrow functions can't be used as constructors
const Person = (name) => {
    this.name = name;
};
// new Person("John");  // ERROR: Person is not a constructor
```

### When to Use Arrow Functions

```javascript
// USE arrow functions for:
// - Callbacks
array.map(item => item.value);
array.filter(item => item.active);
array.reduce((acc, item) => acc + item, 0);

// - Short, simple functions
const double = n => n * 2;
const isEven = n => n % 2 === 0;

// - When you need to preserve outer 'this'
class Timer {
    constructor() {
        this.seconds = 0;
    }
    start() {
        setInterval(() => {
            this.seconds++;  // 'this' refers to Timer instance
            console.log(this.seconds);
        }, 1000);
    }
}

// DON'T use arrow functions for:
// - Object methods (when you need 'this')
const obj = {
    name: "John",
    greet: () => console.log(this.name)  // Wrong!
};

// - Constructors
const Person = (name) => { this.name = name; };  // Can't use 'new'

// - When you need 'arguments' object
const func = () => console.log(arguments);  // Error!
```

---

## Parameters and Arguments

### Basic Parameters

```javascript
function greet(name, greeting) {
    console.log(`${greeting}, ${name}!`);
}

greet("John", "Hello");  // "Hello, John!"
greet("Jane", "Hi");     // "Hi, Jane!"

// Missing arguments become undefined
greet("John");  // "undefined, John!"
```

### Default Parameters

```javascript
// Old way (ES5)
function greetOld(name, greeting) {
    name = name || "Guest";
    greeting = greeting || "Hello";
    console.log(`${greeting}, ${name}!`);
}

// Modern way (ES6+)
function greet(name = "Guest", greeting = "Hello") {
    console.log(`${greeting}, ${name}!`);
}

greet();                  // "Hello, Guest!"
greet("John");            // "Hello, John!"
greet("Jane", "Hi");      // "Hi, Jane!"
greet(undefined, "Hey");  // "Hey, Guest!" (undefined triggers default)

// Default can use previous parameters
function createUser(name, email = `${name.toLowerCase()}@example.com`) {
    return { name, email };
}

console.log(createUser("John"));
// { name: "John", email: "john@example.com" }

// Default can be function call
function getTimestamp() {
    return Date.now();
}

function logMessage(message, timestamp = getTimestamp()) {
    console.log(`[${timestamp}] ${message}`);
}
```

### Rest Parameters

Collect remaining arguments into an array:

```javascript
function sum(...numbers) {
    return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2));           // 3
console.log(sum(1, 2, 3, 4, 5));  // 15

// Combine with regular parameters
function greetAll(greeting, ...names) {
    names.forEach(name => {
        console.log(`${greeting}, ${name}!`);
    });
}

greetAll("Hello", "John", "Jane", "Bob");
// Hello, John!
// Hello, Jane!
// Hello, Bob!

// Rest must be last parameter
// function bad(...rest, last) {}  // ERROR!
```

### Spread Operator in Function Calls

```javascript
const numbers = [1, 2, 3, 4, 5];

// Without spread
console.log(Math.max(1, 2, 3, 4, 5));  // 5

// With spread
console.log(Math.max(...numbers));  // 5

// Combining arrays
function sum(a, b, c) {
    return a + b + c;
}

const arr1 = [1, 2];
const arr2 = [3];
console.log(sum(...arr1, ...arr2));  // 6
```

### Destructuring Parameters

```javascript
// Object destructuring in parameters
function createUser({ name, age, email = "none" }) {
    console.log(`Name: ${name}, Age: ${age}, Email: ${email}`);
}

createUser({ name: "John", age: 25 });
// Name: John, Age: 25, Email: none

// Array destructuring
function getFirst([first, second]) {
    console.log(`First: ${first}, Second: ${second}`);
}

getFirst([1, 2, 3]);  // First: 1, Second: 2

// With defaults for the whole parameter
function greet({ name = "Guest", greeting = "Hello" } = {}) {
    console.log(`${greeting}, ${name}!`);
}

greet();  // "Hello, Guest!" (empty object default)
greet({ name: "John" });  // "Hello, John!"
```

---

## Return Values

### Basic Return

```javascript
function add(a, b) {
    return a + b;
    // Code after return doesn't execute
    console.log("This never runs");
}

console.log(add(2, 3));  // 5

// Functions without return
function noReturn() {
    console.log("No return");
}

let result = noReturn();
console.log(result);  // undefined
```

### Multiple Returns

```javascript
function getGrade(score) {
    if (score >= 90) return "A";
    if (score >= 80) return "B";
    if (score >= 70) return "C";
    if (score >= 60) return "D";
    return "F";
}

console.log(getGrade(85));  // "B"
console.log(getGrade(55));  // "F"

// Early return pattern (guard clauses)
function divide(a, b) {
    if (b === 0) {
        return "Cannot divide by zero";
    }
    return a / b;
}
```

### Returning Objects

```javascript
function createUser(name, age) {
    return {
        name: name,
        age: age,
        createdAt: new Date()
    };
}

// Shorthand property names (ES6)
function createUserShort(name, age) {
    return { name, age, createdAt: new Date() };
}

// Returning multiple values (using object)
function getStats(numbers) {
    const sum = numbers.reduce((a, b) => a + b, 0);
    const avg = sum / numbers.length;
    const min = Math.min(...numbers);
    const max = Math.max(...numbers);

    return { sum, avg, min, max };
}

const stats = getStats([1, 2, 3, 4, 5]);
console.log(stats);  // { sum: 15, avg: 3, min: 1, max: 5 }

// Destructure the return
const { sum, avg } = getStats([1, 2, 3, 4, 5]);
console.log(sum, avg);  // 15, 3
```

### Returning Arrays

```javascript
function getMinMax(numbers) {
    return [Math.min(...numbers), Math.max(...numbers)];
}

const [min, max] = getMinMax([5, 2, 8, 1, 9]);
console.log(min, max);  // 1, 9
```

### Returning Functions

```javascript
function createMultiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// Arrow function version
const createAdder = (a) => (b) => a + b;

const add5 = createAdder(5);
console.log(add5(3));  // 8
```

---

## Scope and Closures

### Scope

```javascript
// Global scope
let globalVar = "I'm global";

function outer() {
    // Function scope
    let outerVar = "I'm in outer";

    function inner() {
        // Nested scope
        let innerVar = "I'm in inner";

        console.log(globalVar);  // Accessible
        console.log(outerVar);   // Accessible
        console.log(innerVar);   // Accessible
    }

    inner();
    // console.log(innerVar);  // ERROR: not accessible
}

outer();
// console.log(outerVar);  // ERROR: not accessible
```

### Block Scope

```javascript
function example() {
    if (true) {
        var varScoped = "var";     // Function scoped
        let letScoped = "let";     // Block scoped
        const constScoped = "const"; // Block scoped
    }

    console.log(varScoped);    // "var" (accessible)
    // console.log(letScoped);    // ERROR
    // console.log(constScoped);  // ERROR
}
```

### Closures

A closure is a function that remembers its outer variables:

```javascript
function createCounter() {
    let count = 0;  // Private variable

    return function() {
        count++;
        return count;
    };
}

const counter = createCounter();
console.log(counter());  // 1
console.log(counter());  // 2
console.log(counter());  // 3

// Each call to createCounter creates a new closure
const counter2 = createCounter();
console.log(counter2());  // 1 (independent count)

// Practical closure example: function factory
function createGreeter(greeting) {
    return function(name) {
        return `${greeting}, ${name}!`;
    };
}

const sayHello = createGreeter("Hello");
const sayHi = createGreeter("Hi");

console.log(sayHello("John"));  // "Hello, John!"
console.log(sayHi("Jane"));     // "Hi, Jane!"
```

### Closure Use Cases

```javascript
// 1. Data privacy
function createBankAccount(initialBalance) {
    let balance = initialBalance;  // Private

    return {
        deposit(amount) {
            balance += amount;
            return balance;
        },
        withdraw(amount) {
            if (amount > balance) {
                return "Insufficient funds";
            }
            balance -= amount;
            return balance;
        },
        getBalance() {
            return balance;
        }
    };
}

const account = createBankAccount(100);
console.log(account.getBalance());  // 100
console.log(account.deposit(50));   // 150
console.log(account.withdraw(30));  // 120
// account.balance = 1000000;  // Doesn't affect the real balance
console.log(account.getBalance());  // 120

// 2. Partial application
function multiply(a) {
    return function(b) {
        return a * b;
    };
}

const double = multiply(2);
const triple = multiply(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// 3. Memoization (caching)
function memoize(fn) {
    const cache = {};

    return function(...args) {
        const key = JSON.stringify(args);
        if (cache[key]) {
            console.log("From cache");
            return cache[key];
        }
        const result = fn(...args);
        cache[key] = result;
        return result;
    };
}

const expensiveCalculation = (n) => {
    console.log("Calculating...");
    return n * n;
};

const memoized = memoize(expensiveCalculation);
console.log(memoized(5));  // "Calculating...", 25
console.log(memoized(5));  // "From cache", 25
console.log(memoized(6));  // "Calculating...", 36
```

---

## Higher-Order Functions

Functions that take or return other functions:

### Functions as Arguments

```javascript
// forEach, map, filter, reduce are higher-order functions
const numbers = [1, 2, 3, 4, 5];

numbers.forEach(function(n) {
    console.log(n);
});

const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);

// Custom higher-order function
function repeat(n, action) {
    for (let i = 0; i < n; i++) {
        action(i);
    }
}

repeat(3, console.log);  // 0, 1, 2

repeat(3, i => console.log("Hello " + i));
// Hello 0
// Hello 1
// Hello 2
```

### Functions Returning Functions

```javascript
function createFormatter(prefix, suffix) {
    return function(text) {
        return `${prefix}${text}${suffix}`;
    };
}

const wrapInBrackets = createFormatter("[", "]");
const wrapInParens = createFormatter("(", ")");

console.log(wrapInBrackets("Hello"));  // "[Hello]"
console.log(wrapInParens("World"));    // "(World)"

// Practical example: event handler factory
function createClickHandler(message) {
    return function(event) {
        console.log(message);
        console.log("Clicked:", event.target);
    };
}

// document.querySelector(".btn").addEventListener("click", createClickHandler("Button clicked!"));
```

### Common Higher-Order Patterns

```javascript
// 1. Composition
const compose = (f, g) => x => f(g(x));

const addOne = x => x + 1;
const double = x => x * 2;

const addOneThenDouble = compose(double, addOne);
console.log(addOneThenDouble(5));  // 12 ((5 + 1) * 2)

// 2. Pipe (left to right)
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

const process = pipe(
    x => x + 1,
    x => x * 2,
    x => x - 3
);
console.log(process(5));  // 9 (((5 + 1) * 2) - 3)

// 3. Currying
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        }
        return function(...moreArgs) {
            return curried.apply(this, args.concat(moreArgs));
        };
    };
}

function add(a, b, c) {
    return a + b + c;
}

const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3));     // 6
console.log(curriedAdd(1, 2)(3));     // 6
console.log(curriedAdd(1)(2, 3));     // 6
console.log(curriedAdd(1, 2, 3));     // 6
```

---

## Callback Functions

Functions passed to other functions, called later:

```javascript
// Synchronous callback
function processData(data, callback) {
    // Process data
    const result = data.map(x => x * 2);
    // Call callback with result
    callback(result);
}

processData([1, 2, 3], function(result) {
    console.log("Processed:", result);  // [2, 4, 6]
});

// Array methods use callbacks
const numbers = [1, 2, 3, 4, 5];

numbers.forEach(n => console.log(n));
numbers.map(n => n * 2);
numbers.filter(n => n > 2);
numbers.find(n => n === 3);
numbers.every(n => n > 0);
numbers.some(n => n > 4);

// Sort with callback
const people = [
    { name: "John", age: 30 },
    { name: "Jane", age: 25 },
    { name: "Bob", age: 35 }
];

people.sort((a, b) => a.age - b.age);
console.log(people);  // Sorted by age ascending
```

### Asynchronous Callbacks

```javascript
// setTimeout
console.log("Start");

setTimeout(function() {
    console.log("After 1 second");
}, 1000);

console.log("End");
// Output: Start, End, After 1 second

// setInterval
let count = 0;
const intervalId = setInterval(function() {
    count++;
    console.log(`Count: ${count}`);
    if (count >= 5) {
        clearInterval(intervalId);
    }
}, 1000);

// Event listeners (browser)
// document.querySelector("button").addEventListener("click", function(event) {
//     console.log("Button clicked!");
// });

// Simulated async operation
function fetchData(callback) {
    setTimeout(function() {
        const data = { id: 1, name: "John" };
        callback(data);
    }, 1000);
}

fetchData(function(data) {
    console.log("Data received:", data);
});
console.log("Waiting for data...");
// Output: "Waiting for data...", then after 1s: "Data received: ..."
```

### Callback Hell (Anti-Pattern)

```javascript
// Nested callbacks become hard to read
fetchUser(userId, function(user) {
    fetchPosts(user.id, function(posts) {
        fetchComments(posts[0].id, function(comments) {
            fetchReplies(comments[0].id, function(replies) {
                console.log(replies);
                // More nesting...
            });
        });
    });
});

// Solution: Use Promises or async/await (covered in later lessons)
```

---

## Immediately Invoked Function Expressions (IIFE)

Functions that run immediately after being defined:

```javascript
// Basic IIFE
(function() {
    console.log("I run immediately!");
})();

// IIFE with arrow function
(() => {
    console.log("Arrow IIFE");
})();

// IIFE with parameters
(function(name) {
    console.log(`Hello, ${name}!`);
})("John");

// Return value from IIFE
const result = (function() {
    return 42;
})();
console.log(result);  // 42

// Use case: Create private scope
const counter = (function() {
    let count = 0;  // Private

    return {
        increment() { return ++count; },
        decrement() { return --count; },
        getCount() { return count; }
    };
})();

console.log(counter.increment());  // 1
console.log(counter.increment());  // 2
console.log(counter.getCount());   // 2
// counter.count is undefined (private)

// Use case: Avoid polluting global scope
(function() {
    const privateVar = "I'm private";

    // Do stuff with privateVar
    console.log(privateVar);

    // privateVar doesn't leak to global scope
})();

// console.log(privateVar);  // ERROR: not defined
```

---

## Recursion

Functions that call themselves:

```javascript
// Factorial: n! = n * (n-1) * (n-2) * ... * 1
function factorial(n) {
    // Base case
    if (n <= 1) return 1;
    // Recursive case
    return n * factorial(n - 1);
}

console.log(factorial(5));  // 120 (5 * 4 * 3 * 2 * 1)

// How it works:
// factorial(5) = 5 * factorial(4)
//              = 5 * 4 * factorial(3)
//              = 5 * 4 * 3 * factorial(2)
//              = 5 * 4 * 3 * 2 * factorial(1)
//              = 5 * 4 * 3 * 2 * 1
//              = 120

// Fibonacci: 0, 1, 1, 2, 3, 5, 8, 13, ...
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(7));  // 13

// Count down
function countdown(n) {
    if (n <= 0) {
        console.log("Done!");
        return;
    }
    console.log(n);
    countdown(n - 1);
}

countdown(5);  // 5, 4, 3, 2, 1, Done!

// Sum of array
function sumArray(arr) {
    if (arr.length === 0) return 0;
    return arr[0] + sumArray(arr.slice(1));
}

console.log(sumArray([1, 2, 3, 4, 5]));  // 15

// Deep object traversal
function flattenObject(obj, prefix = "") {
    let result = {};

    for (let key in obj) {
        const newKey = prefix ? `${prefix}.${key}` : key;

        if (typeof obj[key] === "object" && obj[key] !== null) {
            Object.assign(result, flattenObject(obj[key], newKey));
        } else {
            result[newKey] = obj[key];
        }
    }

    return result;
}

const nested = {
    a: 1,
    b: {
        c: 2,
        d: {
            e: 3
        }
    }
};

console.log(flattenObject(nested));
// { "a": 1, "b.c": 2, "b.d.e": 3 }
```

### Recursion Best Practices

```javascript
// Always have a base case to prevent infinite recursion
function badRecursion() {
    return badRecursion();  // Stack overflow!
}

// Consider using tail recursion for optimization
function factorialTail(n, acc = 1) {
    if (n <= 1) return acc;
    return factorialTail(n - 1, n * acc);
}

// Consider iteration for performance (recursion has overhead)
function factorialIterative(n) {
    let result = 1;
    for (let i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}
```

---

## Practical Examples

### Example 1: Form Validation

```javascript
function validateForm(formData) {
    const errors = [];

    // Validate name
    if (!formData.name || formData.name.trim() === "") {
        errors.push("Name is required");
    } else if (formData.name.length < 2) {
        errors.push("Name must be at least 2 characters");
    }

    // Validate email
    if (!formData.email || formData.email.trim() === "") {
        errors.push("Email is required");
    } else if (!isValidEmail(formData.email)) {
        errors.push("Email is invalid");
    }

    // Validate password
    if (!formData.password) {
        errors.push("Password is required");
    } else if (formData.password.length < 8) {
        errors.push("Password must be at least 8 characters");
    }

    return {
        isValid: errors.length === 0,
        errors
    };
}

function isValidEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
}

// Usage
const result = validateForm({
    name: "John",
    email: "john@example.com",
    password: "12345678"
});

if (result.isValid) {
    console.log("Form is valid!");
} else {
    console.log("Errors:", result.errors);
}
```

### Example 2: Data Transformer

```javascript
function createDataTransformer(transformations) {
    return function(data) {
        return transformations.reduce((result, transform) => {
            return transform(result);
        }, data);
    };
}

// Define transformations
const addTimestamp = data => ({ ...data, timestamp: Date.now() });
const uppercase = data => ({ ...data, name: data.name.toUpperCase() });
const validate = data => ({ ...data, valid: Boolean(data.name && data.email) });

// Create transformer pipeline
const transformer = createDataTransformer([
    addTimestamp,
    uppercase,
    validate
]);

// Use it
const input = { name: "john", email: "john@example.com" };
const output = transformer(input);
console.log(output);
// { name: "JOHN", email: "john@example.com", timestamp: 1234567890, valid: true }
```

### Example 3: Event Emitter

```javascript
function createEventEmitter() {
    const events = {};

    return {
        on(event, callback) {
            if (!events[event]) {
                events[event] = [];
            }
            events[event].push(callback);
        },

        off(event, callback) {
            if (!events[event]) return;
            events[event] = events[event].filter(cb => cb !== callback);
        },

        emit(event, data) {
            if (!events[event]) return;
            events[event].forEach(callback => callback(data));
        },

        once(event, callback) {
            const wrapper = (data) => {
                callback(data);
                this.off(event, wrapper);
            };
            this.on(event, wrapper);
        }
    };
}

// Usage
const emitter = createEventEmitter();

function handleMessage(data) {
    console.log("Received:", data);
}

emitter.on("message", handleMessage);
emitter.emit("message", "Hello!");  // "Received: Hello!"
emitter.emit("message", "World!");  // "Received: World!"

emitter.off("message", handleMessage);
emitter.emit("message", "Goodbye");  // Nothing happens

// Once
emitter.once("connect", () => console.log("Connected!"));
emitter.emit("connect");  // "Connected!"
emitter.emit("connect");  // Nothing (handler removed)
```

---

## Practice Exercises

### Exercise 1: Create a Calculator

```javascript
// Create a calculator with add, subtract, multiply, divide
// Use closures to maintain running total

function createCalculator(initialValue = 0) {
    // Your code here
}

const calc = createCalculator(10);
console.log(calc.add(5));       // 15
console.log(calc.subtract(3));  // 12
console.log(calc.multiply(2));  // 24
console.log(calc.divide(4));    // 6
console.log(calc.getTotal());   // 6
console.log(calc.reset());      // 0
```

**Solution:**
```javascript
function createCalculator(initialValue = 0) {
    let total = initialValue;

    return {
        add(n) {
            total += n;
            return total;
        },
        subtract(n) {
            total -= n;
            return total;
        },
        multiply(n) {
            total *= n;
            return total;
        },
        divide(n) {
            if (n === 0) {
                console.log("Cannot divide by zero");
                return total;
            }
            total /= n;
            return total;
        },
        getTotal() {
            return total;
        },
        reset() {
            total = 0;
            return total;
        }
    };
}
```

### Exercise 2: Debounce Function

```javascript
// Create a debounce function that delays execution
// until N milliseconds after the last call

function debounce(fn, delay) {
    // Your code here
}

// Usage
const expensiveOperation = () => console.log("Executed!");
const debouncedOp = debounce(expensiveOperation, 500);

debouncedOp();  // Wait
debouncedOp();  // Wait
debouncedOp();  // Wait
// After 500ms of no calls: "Executed!" (only once)
```

**Solution:**
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
```

### Exercise 3: Array Methods Recreation

```javascript
// Recreate map, filter, and reduce using only for loops

function myMap(arr, callback) {
    // Your code here
}

function myFilter(arr, callback) {
    // Your code here
}

function myReduce(arr, callback, initialValue) {
    // Your code here
}
```

**Solution:**
```javascript
function myMap(arr, callback) {
    const result = [];
    for (let i = 0; i < arr.length; i++) {
        result.push(callback(arr[i], i, arr));
    }
    return result;
}

function myFilter(arr, callback) {
    const result = [];
    for (let i = 0; i < arr.length; i++) {
        if (callback(arr[i], i, arr)) {
            result.push(arr[i]);
        }
    }
    return result;
}

function myReduce(arr, callback, initialValue) {
    let accumulator = initialValue;
    let startIndex = 0;

    if (accumulator === undefined) {
        accumulator = arr[0];
        startIndex = 1;
    }

    for (let i = startIndex; i < arr.length; i++) {
        accumulator = callback(accumulator, arr[i], i, arr);
    }

    return accumulator;
}

// Test
console.log(myMap([1, 2, 3], x => x * 2));  // [2, 4, 6]
console.log(myFilter([1, 2, 3, 4], x => x % 2 === 0));  // [2, 4]
console.log(myReduce([1, 2, 3, 4], (a, b) => a + b, 0));  // 10
```

---

## Key Takeaways

1. **Function declarations are hoisted**, expressions are not
2. **Arrow functions** are concise but don't have their own `this`
3. **Default parameters** provide fallback values
4. **Rest parameters** collect arguments into an array
5. **Closures** remember their outer scope
6. **Higher-order functions** take or return functions
7. **Callbacks** are functions passed as arguments
8. **IIFE** creates private scope
9. **Recursion** needs a base case to avoid infinite loops
10. **Pure functions** don't modify external state

---

## Self-Check Questions

1. What's the difference between function declaration and expression?
2. When should you NOT use arrow functions?
3. What is a closure?
4. What is a higher-order function?
5. What's the difference between rest parameters and spread operator?
6. Why might you use an IIFE?
7. What are the requirements for a recursive function?

---

**Next Lesson:** [Day 3-4 - Objects](./week-02-day-03-04-objects.md)
