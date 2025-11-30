# Week 1, Day 3: Operators & Conditionals

## Operators Overview

Operators perform operations on values. JavaScript has several types:

1. **Arithmetic** - Math operations
2. **Assignment** - Assign values
3. **Comparison** - Compare values
4. **Logical** - Combine conditions
5. **String** - Work with text
6. **Ternary** - Conditional shorthand

---

## Arithmetic Operators

Perform mathematical calculations:

```javascript
let a = 10;
let b = 3;

// Addition
console.log(a + b);   // 13

// Subtraction
console.log(a - b);   // 7

// Multiplication
console.log(a * b);   // 30

// Division
console.log(a / b);   // 3.3333...

// Modulo (Remainder)
console.log(a % b);   // 1 (10 divided by 3 = 3 remainder 1)

// Exponentiation (Power)
console.log(a ** b);  // 1000 (10^3)

// Increment
let x = 5;
x++;                  // x is now 6
++x;                  // x is now 7

// Decrement
let y = 5;
y--;                  // y is now 4
--y;                  // y is now 3
```

### Prefix vs Postfix Increment

```javascript
let a = 5;
let b = 5;

// Postfix: returns value THEN increments
console.log(a++);  // 5 (returns 5, then a becomes 6)
console.log(a);    // 6

// Prefix: increments THEN returns value
console.log(++b);  // 6 (b becomes 6, then returns 6)
console.log(b);    // 6

// Practical example
let count = 0;
let values = [];

// Postfix - uses current value, then increments
values.push(count++);  // pushes 0, count becomes 1
values.push(count++);  // pushes 1, count becomes 2
console.log(values);   // [0, 1]

// Prefix - increments first, then uses value
let count2 = 0;
let values2 = [];
values2.push(++count2);  // count2 becomes 1, pushes 1
values2.push(++count2);  // count2 becomes 2, pushes 2
console.log(values2);    // [1, 2]
```

### Modulo Use Cases

```javascript
// Check if number is even or odd
function isEven(num) {
    return num % 2 === 0;
}
console.log(isEven(4));  // true
console.log(isEven(7));  // false

// Cycle through values (0, 1, 2, 0, 1, 2, ...)
for (let i = 0; i < 10; i++) {
    console.log(i % 3);  // 0, 1, 2, 0, 1, 2, 0, 1, 2, 0
}

// Every nth item
let items = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i'];
for (let i = 0; i < items.length; i++) {
    if (i % 3 === 0) {
        console.log(items[i]);  // a, d, g (every 3rd starting from first)
    }
}

// Wrap around (circular array)
let colors = ['red', 'green', 'blue'];
for (let i = 0; i < 10; i++) {
    console.log(colors[i % colors.length]);
    // red, green, blue, red, green, blue, red, green, blue, red
}
```

---

## Assignment Operators

Assign and modify values:

```javascript
// Basic assignment
let x = 10;

// Addition assignment
x += 5;   // x = x + 5 = 15

// Subtraction assignment
x -= 3;   // x = x - 3 = 12

// Multiplication assignment
x *= 2;   // x = x * 2 = 24

// Division assignment
x /= 4;   // x = x / 4 = 6

// Modulo assignment
x %= 4;   // x = x % 4 = 2

// Exponentiation assignment
x **= 3;  // x = x ** 3 = 8

// String concatenation assignment
let greeting = "Hello";
greeting += " World";  // "Hello World"
```

---

## Comparison Operators

Compare values and return boolean:

### Equality Operators

```javascript
// Loose equality (==) - compares with type coercion
console.log(5 == "5");     // true (string converted to number)
console.log(0 == false);   // true (false converted to 0)
console.log(null == undefined);  // true
console.log("" == 0);      // true

// Strict equality (===) - compares without type coercion
console.log(5 === "5");    // false (different types)
console.log(0 === false);  // false
console.log(null === undefined);  // false
console.log("" === 0);     // false

// Loose inequality (!=)
console.log(5 != "5");     // false

// Strict inequality (!==)
console.log(5 !== "5");    // true

// ALWAYS USE === and !== (strict comparison)
```

### Why Strict Equality Matters

```javascript
// Loose equality can be confusing
console.log(0 == "");       // true
console.log(0 == "0");      // true
console.log("" == "0");     // false (!)

console.log(false == "false");  // false
console.log(false == "0");      // true

console.log(null == undefined);  // true
console.log(null == 0);          // false

// Strict equality is predictable
console.log(0 === "");      // false
console.log(0 === "0");     // false
console.log(null === undefined);  // false
```

### Relational Operators

```javascript
let a = 10;
let b = 5;

// Greater than
console.log(a > b);   // true
console.log(5 > 5);   // false

// Less than
console.log(a < b);   // false
console.log(3 < 5);   // true

// Greater than or equal
console.log(a >= b);  // true
console.log(5 >= 5);  // true

// Less than or equal
console.log(a <= b);  // false
console.log(5 <= 5);  // true
```

### Comparing Strings

```javascript
// Strings compared alphabetically (lexicographically)
console.log("apple" < "banana");  // true (a comes before b)
console.log("apple" < "Apple");   // false (lowercase > uppercase in ASCII)
console.log("a" < "A");           // false (a=97, A=65 in ASCII)

// Compare by character codes
console.log("abc".charCodeAt(0));  // 97
console.log("ABC".charCodeAt(0));  // 65

// Case-insensitive comparison
let str1 = "Apple";
let str2 = "apple";
console.log(str1.toLowerCase() === str2.toLowerCase());  // true

// Comparing numbers as strings (careful!)
console.log("10" < "9");   // true (!) - compares "1" < "9"
console.log(10 < 9);       // false - proper number comparison
```

---

## Logical Operators

Combine or modify boolean values:

### AND (&&)

Returns true only if BOTH operands are true:

```javascript
console.log(true && true);    // true
console.log(true && false);   // false
console.log(false && true);   // false
console.log(false && false);  // false

// Practical examples
let age = 25;
let hasLicense = true;

if (age >= 18 && hasLicense) {
    console.log("Can drive");
}

// Multiple conditions
let isLoggedIn = true;
let isAdmin = true;
let hasPermission = true;

if (isLoggedIn && isAdmin && hasPermission) {
    console.log("Access granted");
}
```

### OR (||)

Returns true if AT LEAST ONE operand is true:

```javascript
console.log(true || true);    // true
console.log(true || false);   // true
console.log(false || true);   // true
console.log(false || false);  // false

// Practical examples
let isWeekend = false;
let isHoliday = true;

if (isWeekend || isHoliday) {
    console.log("Day off!");
}

// Default values (old way)
let username = "";
let displayName = username || "Guest";
console.log(displayName);  // "Guest"

let count = 0;
let display = count || "No items";
console.log(display);  // "No items" (!) - 0 is falsy
```

### NOT (!)

Inverts the boolean value:

```javascript
console.log(!true);   // false
console.log(!false);  // true

// Double NOT (!!) converts to boolean
console.log(!!"hello");  // true
console.log(!!"");       // false
console.log(!!0);        // false
console.log(!!1);        // true
console.log(!!null);     // false
console.log(!!undefined);  // false

// Practical use
function isLoggedIn(user) {
    return !!user;  // Returns true if user exists, false otherwise
}
```

### Nullish Coalescing (??)

Returns right operand ONLY if left is null or undefined:

```javascript
// Different from || which checks for any falsy value
let value1 = null;
let value2 = undefined;
let value3 = 0;
let value4 = "";
let value5 = false;

// Using ||
console.log(value1 || "default");  // "default"
console.log(value3 || "default");  // "default" (0 is falsy)
console.log(value4 || "default");  // "default" ("" is falsy)
console.log(value5 || "default");  // "default" (false is falsy)

// Using ?? (only null/undefined)
console.log(value1 ?? "default");  // "default"
console.log(value3 ?? "default");  // 0 (0 is not null/undefined)
console.log(value4 ?? "default");  // "" ("" is not null/undefined)
console.log(value5 ?? "default");  // false

// Practical example
function getConfig(options) {
    // 0 and false should be valid values
    let timeout = options.timeout ?? 5000;
    let retries = options.retries ?? 3;
    let enabled = options.enabled ?? true;

    return { timeout, retries, enabled };
}

console.log(getConfig({ timeout: 0, enabled: false }));
// { timeout: 0, retries: 3, enabled: false }

// With || this would be wrong:
// { timeout: 5000, retries: 3, enabled: true }
```

### Short-Circuit Evaluation

Logical operators don't always evaluate both operands:

```javascript
// AND (&&) - stops at first falsy value
console.log(false && anything);  // false (doesn't evaluate 'anything')
console.log(true && "hello");    // "hello"
console.log("hello" && "world"); // "world"
console.log(0 && "hello");       // 0

// OR (||) - stops at first truthy value
console.log(true || anything);   // true (doesn't evaluate 'anything')
console.log(false || "hello");   // "hello"
console.log("" || "default");    // "default"
console.log("hello" || "world"); // "hello"

// Practical uses
// Conditional execution
let user = { name: "John" };
user && console.log(user.name);  // "John" (only runs if user exists)

// Default value
let name = inputName || "Anonymous";

// Guard clause
function greet(person) {
    person && person.name && console.log(`Hello, ${person.name}`);
}
```

---

## Optional Chaining (?.)

Safely access nested properties:

```javascript
let user = {
    name: "John",
    address: {
        city: "New York"
    }
};

// Without optional chaining (can throw error)
// console.log(user.profile.avatar);  // ERROR: Cannot read property 'avatar' of undefined

// With optional chaining (returns undefined)
console.log(user.profile?.avatar);  // undefined (no error)
console.log(user.address?.city);    // "New York"

// Chaining multiple levels
let company = {
    name: "Acme",
    ceo: {
        name: "Jane",
        contact: {
            email: "jane@acme.com"
        }
    }
};

console.log(company.ceo?.contact?.email);  // "jane@acme.com"
console.log(company.cfo?.contact?.email);  // undefined

// With arrays
let users = [{ name: "John" }, { name: "Jane" }];
console.log(users[0]?.name);   // "John"
console.log(users[5]?.name);   // undefined

// With methods
let obj = {
    greet() {
        return "Hello";
    }
};

console.log(obj.greet?.());    // "Hello"
console.log(obj.goodbye?.());  // undefined (method doesn't exist)

// Combining with nullish coalescing
let userCity = user.address?.city ?? "Unknown";
console.log(userCity);  // "New York"

let unknownCity = user.location?.city ?? "Unknown";
console.log(unknownCity);  // "Unknown"
```

---

## Conditionals

Make decisions in your code based on conditions.

### if Statement

```javascript
let age = 18;

if (age >= 18) {
    console.log("You are an adult");
}

// Single statement (no braces needed, but recommended)
if (age >= 18) console.log("Adult");

// With variable
let canVote = false;
if (age >= 18) {
    canVote = true;
}
```

### if...else

```javascript
let hour = 14;

if (hour < 12) {
    console.log("Good morning");
} else {
    console.log("Good afternoon/evening");
}

// Assigning based on condition
let greeting;
if (hour < 12) {
    greeting = "Good morning";
} else {
    greeting = "Good afternoon";
}
```

### if...else if...else

```javascript
let score = 85;
let grade;

if (score >= 90) {
    grade = "A";
} else if (score >= 80) {
    grade = "B";
} else if (score >= 70) {
    grade = "C";
} else if (score >= 60) {
    grade = "D";
} else {
    grade = "F";
}

console.log(`Score: ${score}, Grade: ${grade}`);  // Score: 85, Grade: B

// Time-based greeting
let hour = new Date().getHours();
let message;

if (hour < 6) {
    message = "It's late, go to sleep!";
} else if (hour < 12) {
    message = "Good morning!";
} else if (hour < 18) {
    message = "Good afternoon!";
} else if (hour < 22) {
    message = "Good evening!";
} else {
    message = "Good night!";
}
```

### Nested if Statements

```javascript
let isLoggedIn = true;
let isAdmin = true;
let hasPermission = false;

if (isLoggedIn) {
    if (isAdmin) {
        if (hasPermission) {
            console.log("Full access granted");
        } else {
            console.log("Admin without permission");
        }
    } else {
        console.log("Regular user access");
    }
} else {
    console.log("Please log in");
}

// Better: flatten with logical operators
if (!isLoggedIn) {
    console.log("Please log in");
} else if (!isAdmin) {
    console.log("Regular user access");
} else if (!hasPermission) {
    console.log("Admin without permission");
} else {
    console.log("Full access granted");
}
```

---

## Ternary Operator

Shorthand for simple if...else:

```javascript
// Syntax: condition ? valueIfTrue : valueIfFalse

let age = 20;
let status = age >= 18 ? "adult" : "minor";
console.log(status);  // "adult"

// Equivalent to:
let status2;
if (age >= 18) {
    status2 = "adult";
} else {
    status2 = "minor";
}

// In template literals
console.log(`You are ${age >= 18 ? "an adult" : "a minor"}`);

// Multiple conditions (nested ternary - use sparingly!)
let score = 85;
let grade = score >= 90 ? "A"
          : score >= 80 ? "B"
          : score >= 70 ? "C"
          : score >= 60 ? "D"
          : "F";

// For complex conditions, use if...else instead
// Ternary is best for simple, single conditions

// Ternary for function calls
function greet(isMorning) {
    return isMorning ? "Good morning!" : "Hello!";
}

// Ternary in JSX/React (preview)
// <div>{isLoggedIn ? <Dashboard /> : <Login />}</div>
```

---

## Switch Statement

Handle multiple specific values:

```javascript
let day = 3;
let dayName;

switch (day) {
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    case 3:
        dayName = "Wednesday";
        break;
    case 4:
        dayName = "Thursday";
        break;
    case 5:
        dayName = "Friday";
        break;
    case 6:
        dayName = "Saturday";
        break;
    case 7:
        dayName = "Sunday";
        break;
    default:
        dayName = "Invalid day";
}

console.log(dayName);  // "Wednesday"
```

### Break Statement

Without break, execution "falls through" to next case:

```javascript
let fruit = "apple";

// Without break (wrong - falls through)
switch (fruit) {
    case "apple":
        console.log("It's an apple");
    case "banana":
        console.log("It's a banana");
    case "orange":
        console.log("It's an orange");
}
// Outputs: "It's an apple", "It's a banana", "It's an orange"

// With break (correct)
switch (fruit) {
    case "apple":
        console.log("It's an apple");
        break;
    case "banana":
        console.log("It's a banana");
        break;
    case "orange":
        console.log("It's an orange");
        break;
}
// Outputs: "It's an apple"
```

### Fall-Through (Intentional)

Sometimes fall-through is useful:

```javascript
let month = 2;
let days;

switch (month) {
    case 1: case 3: case 5: case 7: case 8: case 10: case 12:
        days = 31;
        break;
    case 4: case 6: case 9: case 11:
        days = 30;
        break;
    case 2:
        days = 28;  // Ignoring leap years
        break;
    default:
        days = 0;
}

console.log(`Month ${month} has ${days} days`);

// Grouping weekend days
let dayNum = 6;
let dayType;

switch (dayNum) {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
        dayType = "Weekday";
        break;
    case 6:
    case 7:
        dayType = "Weekend";
        break;
    default:
        dayType = "Invalid";
}
```

### Default Case

Handles unmatched values:

```javascript
let color = "purple";

switch (color) {
    case "red":
        console.log("Stop");
        break;
    case "yellow":
        console.log("Caution");
        break;
    case "green":
        console.log("Go");
        break;
    default:
        console.log("Invalid color");
}
// Outputs: "Invalid color"
```

### Switch with Strings

```javascript
let command = "start";

switch (command.toLowerCase()) {
    case "start":
        console.log("Starting...");
        break;
    case "stop":
        console.log("Stopping...");
        break;
    case "restart":
        console.log("Restarting...");
        break;
    default:
        console.log("Unknown command");
}
```

### Switch vs if...else

```javascript
// Use switch when:
// - Comparing ONE variable against multiple EXACT values
// - You have many specific cases

// Use if...else when:
// - Comparing ranges (>, <, >=, <=)
// - Complex conditions with multiple variables
// - Conditions with logical operators

// Switch example (good use case)
let status = "pending";
switch (status) {
    case "pending": /* ... */ break;
    case "approved": /* ... */ break;
    case "rejected": /* ... */ break;
}

// if...else example (good use case)
let score = 85;
if (score >= 90) { /* A */ }
else if (score >= 80) { /* B */ }
else if (score >= 70) { /* C */ }
```

---

## Truthy and Falsy in Conditions

Understanding what evaluates to true/false:

```javascript
// Falsy values
if (false) { }      // Won't run
if (0) { }          // Won't run
if (-0) { }         // Won't run
if (0n) { }         // Won't run (BigInt zero)
if ("") { }         // Won't run
if (null) { }       // Won't run
if (undefined) { }  // Won't run
if (NaN) { }        // Won't run

// Everything else is truthy
if (true) { }       // Runs
if (1) { }          // Runs
if (-1) { }         // Runs
if ("hello") { }    // Runs
if (" ") { }        // Runs (space is truthy!)
if ("0") { }        // Runs (string "0" is truthy!)
if ("false") { }    // Runs (string "false" is truthy!)
if ([]) { }         // Runs (empty array is truthy!)
if ({}) { }         // Runs (empty object is truthy!)
if (function(){}) { }  // Runs

// Common patterns
let name = "";
if (name) {
    console.log(`Hello, ${name}`);
} else {
    console.log("Name is empty");
}

// Check if array has items
let items = [];
if (items.length) {
    console.log("Has items");
} else {
    console.log("Empty array");
}

// Check if object has property
let user = { name: "John" };
if (user.email) {
    console.log(`Email: ${user.email}`);
} else {
    console.log("No email");
}
```

---

## Practical Examples

### Example 1: Login Validation

```javascript
function validateLogin(username, password) {
    // Check for empty fields
    if (!username || !password) {
        return { success: false, message: "All fields required" };
    }

    // Check username length
    if (username.length < 3) {
        return { success: false, message: "Username too short" };
    }

    // Check password length
    if (password.length < 8) {
        return { success: false, message: "Password must be at least 8 characters" };
    }

    // Simulate checking credentials
    if (username === "admin" && password === "password123") {
        return { success: true, message: "Login successful" };
    }

    return { success: false, message: "Invalid credentials" };
}

console.log(validateLogin("", "pass"));
// { success: false, message: "All fields required" }

console.log(validateLogin("admin", "password123"));
// { success: true, message: "Login successful" }
```

### Example 2: Pricing Calculator

```javascript
function calculatePrice(basePrice, memberType, quantity) {
    let discount = 0;

    // Member discount
    switch (memberType) {
        case "gold":
            discount = 0.20;  // 20% off
            break;
        case "silver":
            discount = 0.10;  // 10% off
            break;
        case "bronze":
            discount = 0.05;  // 5% off
            break;
        default:
            discount = 0;
    }

    // Quantity discount (additional)
    if (quantity >= 100) {
        discount += 0.15;
    } else if (quantity >= 50) {
        discount += 0.10;
    } else if (quantity >= 10) {
        discount += 0.05;
    }

    // Cap discount at 30%
    discount = Math.min(discount, 0.30);

    let subtotal = basePrice * quantity;
    let discountAmount = subtotal * discount;
    let total = subtotal - discountAmount;

    return {
        subtotal: subtotal.toFixed(2),
        discount: (discount * 100).toFixed(0) + "%",
        discountAmount: discountAmount.toFixed(2),
        total: total.toFixed(2)
    };
}

console.log(calculatePrice(10, "gold", 50));
// { subtotal: "500.00", discount: "30%", discountAmount: "150.00", total: "350.00" }
```

### Example 3: Age Group Classifier

```javascript
function classifyAge(age) {
    if (typeof age !== "number" || age < 0 || !Number.isInteger(age)) {
        return "Invalid age";
    }

    if (age < 1) {
        return "Infant";
    } else if (age < 3) {
        return "Toddler";
    } else if (age < 13) {
        return "Child";
    } else if (age < 20) {
        return "Teenager";
    } else if (age < 30) {
        return "Young Adult";
    } else if (age < 60) {
        return "Adult";
    } else {
        return "Senior";
    }
}

console.log(classifyAge(5));   // "Child"
console.log(classifyAge(25));  // "Young Adult"
console.log(classifyAge(65));  // "Senior"
console.log(classifyAge(-5));  // "Invalid age"
```

### Example 4: Traffic Light

```javascript
function getAction(light) {
    switch (light.toLowerCase()) {
        case "red":
            return "Stop";
        case "yellow":
        case "amber":
            return "Prepare to stop";
        case "green":
            return "Go";
        case "flashing red":
            return "Stop, then proceed when safe";
        case "flashing yellow":
            return "Proceed with caution";
        default:
            return "Unknown signal - proceed with caution";
    }
}

console.log(getAction("GREEN"));       // "Go"
console.log(getAction("Flashing Red")); // "Stop, then proceed when safe"
```

---

## Practice Exercises

### Exercise 1: Number Classifier

```javascript
// Write a function that takes a number and returns:
// - "positive" if > 0
// - "negative" if < 0
// - "zero" if === 0

function classifyNumber(num) {
    // Your code here
}

// Test
console.log(classifyNumber(5));   // "positive"
console.log(classifyNumber(-3));  // "negative"
console.log(classifyNumber(0));   // "zero"
```

**Solution:**
```javascript
function classifyNumber(num) {
    if (num > 0) {
        return "positive";
    } else if (num < 0) {
        return "negative";
    } else {
        return "zero";
    }
}

// Or with ternary
function classifyNumber(num) {
    return num > 0 ? "positive" : num < 0 ? "negative" : "zero";
}
```

### Exercise 2: Leap Year Checker

```javascript
// A year is a leap year if:
// - Divisible by 4 AND not divisible by 100
// - OR divisible by 400

function isLeapYear(year) {
    // Your code here
}

// Test
console.log(isLeapYear(2024));  // true
console.log(isLeapYear(2023));  // false
console.log(isLeapYear(2000));  // true
console.log(isLeapYear(1900));  // false
```

**Solution:**
```javascript
function isLeapYear(year) {
    return (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0);
}
```

### Exercise 3: Grade Calculator

```javascript
// Convert a percentage to a letter grade:
// 90-100: A, 80-89: B, 70-79: C, 60-69: D, below 60: F

function getGrade(percentage) {
    // Your code here
}

// Test
console.log(getGrade(95));  // "A"
console.log(getGrade(82));  // "B"
console.log(getGrade(55));  // "F"
```

**Solution:**
```javascript
function getGrade(percentage) {
    if (percentage >= 90) return "A";
    if (percentage >= 80) return "B";
    if (percentage >= 70) return "C";
    if (percentage >= 60) return "D";
    return "F";
}
```

---

## Key Takeaways

1. **Always use === and !==** for comparison (strict equality)
2. **&& returns first falsy** or last value; **|| returns first truthy** or last value
3. **?? only checks for null/undefined**, not other falsy values
4. **?.** safely accesses nested properties without errors
5. **Ternary operator** is for simple conditions only
6. **Switch** is best for comparing one value against many exact matches
7. **Don't forget break** in switch statements
8. **Know your falsy values**: false, 0, "", null, undefined, NaN

---

## Self-Check Questions

1. What's the difference between == and ===?
2. What does && return when all values are truthy?
3. When would you use ?? instead of ||?
4. What happens if you forget break in a switch?
5. What are all the falsy values in JavaScript?
6. When should you use switch vs if...else?

---

**Next Lesson:** [Day 4-5 - Loops & Arrays](./week-01-day-04-05-loops-arrays.md)
