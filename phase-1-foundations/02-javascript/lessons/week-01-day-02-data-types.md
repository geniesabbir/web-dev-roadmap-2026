# Week 1, Day 2: JavaScript Data Types

## Overview of Data Types

JavaScript has two categories of data types:

**Primitive Types (immutable):**
1. String
2. Number
3. Boolean
4. undefined
5. null
6. Symbol
7. BigInt

**Reference Types (mutable):**
1. Object
2. Array
3. Function

---

## Strings

Strings represent text. They're enclosed in quotes.

### Creating Strings

```javascript
// Single quotes
let single = 'Hello';

// Double quotes
let double = "World";

// Backticks (template literals) - PREFERRED
let template = `Hello World`;

// All are valid, but be consistent
```

### String Properties and Methods

```javascript
let text = "Hello, World!";

// Length
console.log(text.length);  // 13

// Access characters
console.log(text[0]);       // "H"
console.log(text.charAt(0)); // "H"
console.log(text.at(-1));   // "!" (last character)

// Case conversion
console.log(text.toUpperCase());  // "HELLO, WORLD!"
console.log(text.toLowerCase());  // "hello, world!"

// Finding substrings
console.log(text.indexOf("World"));     // 7
console.log(text.indexOf("world"));     // -1 (not found)
console.log(text.includes("World"));    // true
console.log(text.startsWith("Hello"));  // true
console.log(text.endsWith("!"));        // true

// Extracting substrings
console.log(text.slice(0, 5));      // "Hello"
console.log(text.slice(7));         // "World!"
console.log(text.slice(-6));        // "World!"
console.log(text.substring(0, 5));  // "Hello"

// Replacing
console.log(text.replace("World", "JavaScript"));  // "Hello, JavaScript!"
console.log(text.replaceAll("l", "L"));            // "HeLLo, WorLd!"

// Splitting
console.log(text.split(", "));  // ["Hello", "World!"]
console.log(text.split(""));    // ["H","e","l","l","o",",",...]

// Trimming whitespace
let messy = "   Hello   ";
console.log(messy.trim());       // "Hello"
console.log(messy.trimStart());  // "Hello   "
console.log(messy.trimEnd());    // "   Hello"

// Padding
let num = "5";
console.log(num.padStart(3, "0"));  // "005"
console.log(num.padEnd(3, "0"));    // "500"

// Repeating
console.log("ha".repeat(3));  // "hahaha"
```

### Template Literals (Backticks)

The modern, preferred way to work with strings:

```javascript
// String interpolation
let name = "John";
let age = 25;
let message = `My name is ${name} and I'm ${age} years old.`;
console.log(message);  // "My name is John and I'm 25 years old."

// Expressions in templates
let a = 5;
let b = 10;
console.log(`Sum: ${a + b}`);  // "Sum: 15"
console.log(`Is adult: ${age >= 18}`);  // "Is adult: true"

// Multi-line strings
let multiLine = `
    This is
    a multi-line
    string.
`;

// Function calls
function greet(name) {
    return `Hello, ${name}!`;
}
console.log(`Greeting: ${greet("World")}`);
```

### String Immutability

Strings cannot be changed in place:

```javascript
let text = "Hello";
text[0] = "J";      // Doesn't work!
console.log(text);  // "Hello" (unchanged)

// Must create new string
text = "J" + text.slice(1);
console.log(text);  // "Jello"
```

### Escape Characters

```javascript
let text = "He said \"Hello\"";  // He said "Hello"
let path = "C:\\Users\\John";    // C:\Users\John
let newLine = "Line 1\nLine 2";  // Two lines
let tab = "Col1\tCol2";          // Tab between
```

---

## Numbers

JavaScript has one number type for both integers and decimals.

### Creating Numbers

```javascript
let integer = 42;
let decimal = 3.14;
let negative = -10;
let scientific = 2.5e6;  // 2,500,000
let hex = 0xff;          // 255
let binary = 0b1010;     // 10
let octal = 0o17;        // 15
```

### Number Operations

```javascript
// Basic math
console.log(5 + 3);   // 8
console.log(10 - 4);  // 6
console.log(4 * 3);   // 12
console.log(15 / 3);  // 5
console.log(17 % 5);  // 2 (remainder/modulo)
console.log(2 ** 3);  // 8 (exponentiation)

// Increment/Decrement
let x = 5;
x++;        // x is now 6
x--;        // x is now 5
++x;        // x is now 6 (prefix)
--x;        // x is now 5 (prefix)

// Assignment operators
let y = 10;
y += 5;     // y = y + 5 = 15
y -= 3;     // y = y - 3 = 12
y *= 2;     // y = y * 2 = 24
y /= 4;     // y = y / 4 = 6
y %= 4;     // y = y % 4 = 2
```

### Number Methods

```javascript
let num = 123.456;

// Rounding
console.log(Math.round(num));   // 123
console.log(Math.floor(num));   // 123 (round down)
console.log(Math.ceil(num));    // 124 (round up)
console.log(Math.trunc(num));   // 123 (remove decimal)

// Fixed decimal places (returns string!)
console.log(num.toFixed(2));    // "123.46"
console.log(num.toFixed(0));    // "123"

// Convert to integer
console.log(parseInt("42"));      // 42
console.log(parseInt("42.9"));    // 42
console.log(parseInt("42px"));    // 42
console.log(parseInt("hello"));   // NaN

// Convert to float
console.log(parseFloat("3.14"));  // 3.14
console.log(parseFloat("3.14.15")); // 3.14

// Number() conversion
console.log(Number("42"));     // 42
console.log(Number("3.14"));   // 3.14
console.log(Number("42px"));   // NaN (stricter)
console.log(Number(""));       // 0
console.log(Number(true));     // 1
console.log(Number(false));    // 0
```

### Math Object

```javascript
// Constants
console.log(Math.PI);       // 3.141592653589793
console.log(Math.E);        // 2.718281828459045

// Absolute value
console.log(Math.abs(-5));  // 5

// Power and roots
console.log(Math.pow(2, 3));   // 8
console.log(Math.sqrt(16));    // 4
console.log(Math.cbrt(27));    // 3

// Min/Max
console.log(Math.min(1, 5, 3));  // 1
console.log(Math.max(1, 5, 3));  // 5

// Random (0 to 1, exclusive of 1)
console.log(Math.random());  // 0.123456...

// Random integer between min and max (inclusive)
function randomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
console.log(randomInt(1, 10));  // 1-10

// Sign
console.log(Math.sign(-5));  // -1
console.log(Math.sign(0));   // 0
console.log(Math.sign(5));   // 1
```

### Special Number Values

```javascript
// Infinity
console.log(1 / 0);         // Infinity
console.log(-1 / 0);        // -Infinity
console.log(Infinity + 1);  // Infinity

// NaN (Not a Number)
console.log("hello" / 2);   // NaN
console.log(NaN + 5);       // NaN
console.log(NaN === NaN);   // false (!)
console.log(Number.isNaN(NaN));  // true

// Checking for finite numbers
console.log(Number.isFinite(123));      // true
console.log(Number.isFinite(Infinity)); // false
console.log(Number.isFinite(NaN));      // false

// Safe integers
console.log(Number.MAX_SAFE_INTEGER);  // 9007199254740991
console.log(Number.MIN_SAFE_INTEGER);  // -9007199254740991
console.log(Number.isSafeInteger(9007199254740991));  // true
console.log(Number.isSafeInteger(9007199254740992));  // false
```

### Floating Point Precision

```javascript
// Famous gotcha
console.log(0.1 + 0.2);  // 0.30000000000000004 (!)
console.log(0.1 + 0.2 === 0.3);  // false (!)

// Solutions
console.log((0.1 + 0.2).toFixed(1));  // "0.3"
console.log(Math.abs(0.1 + 0.2 - 0.3) < 0.0001);  // true

// For money, use integers (cents)
let priceInCents = 299;  // $2.99
```

---

## BigInt

For integers larger than Number can safely represent:

```javascript
// Create BigInt
let big = 9007199254740991n;
let big2 = BigInt("9007199254740991");

// Operations (must use BigInt with BigInt)
console.log(big + 1n);   // 9007199254740992n
console.log(big * 2n);   // 18014398509481982n

// Can't mix with regular numbers
// console.log(big + 1);  // ERROR!
console.log(big + BigInt(1));  // OK
```

---

## Booleans

Represent true or false values.

```javascript
let isActive = true;
let isLoggedIn = false;

// Comparison results are booleans
console.log(5 > 3);   // true
console.log(5 < 3);   // false
console.log(5 === 5); // true

// Logical operators
console.log(true && true);   // true (AND)
console.log(true && false);  // false
console.log(true || false);  // true (OR)
console.log(!true);          // false (NOT)
```

### Truthy and Falsy Values

JavaScript converts values to booleans in conditions:

```javascript
// Falsy values (evaluate to false)
Boolean(false);      // false
Boolean(0);          // false
Boolean(-0);         // false
Boolean(0n);         // false
Boolean("");         // false
Boolean(null);       // false
Boolean(undefined);  // false
Boolean(NaN);        // false

// Everything else is truthy
Boolean(true);       // true
Boolean(1);          // true
Boolean(-1);         // true
Boolean("hello");    // true
Boolean(" ");        // true (space is truthy!)
Boolean([]);         // true (empty array is truthy!)
Boolean({});         // true (empty object is truthy!)
Boolean(function(){}); // true
```

### Using in Conditions

```javascript
let name = "";

if (name) {
    console.log("Name is set");
} else {
    console.log("Name is empty");  // This runs
}

// Common pattern: default value
let username = inputName || "Guest";

// Nullish coalescing (only null/undefined)
let value = null;
let result = value ?? "default";  // "default"

let value2 = "";
let result2 = value2 ?? "default";  // "" (empty string is not null)
let result3 = value2 || "default";  // "default" (empty string is falsy)
```

---

## undefined vs null

Both represent "no value" but have different meanings:

```javascript
// undefined: variable declared but not assigned
let x;
console.log(x);  // undefined

// null: intentionally empty/no value
let y = null;
console.log(y);  // null

// Checking
console.log(typeof undefined);  // "undefined"
console.log(typeof null);       // "object" (historical bug!)

console.log(undefined == null);   // true (loose equality)
console.log(undefined === null);  // false (strict equality)

// Use cases
let user = null;        // No user yet (intentional)
let data;              // Not initialized (undefined)

function greet(name) {
    if (name === undefined) {
        console.log("No name provided");
    }
}
```

---

## typeof Operator

Check the type of a value:

```javascript
console.log(typeof "hello");     // "string"
console.log(typeof 42);          // "number"
console.log(typeof 42n);         // "bigint"
console.log(typeof true);        // "boolean"
console.log(typeof undefined);   // "undefined"
console.log(typeof null);        // "object" (bug!)
console.log(typeof Symbol());    // "symbol"
console.log(typeof {});          // "object"
console.log(typeof []);          // "object" (arrays are objects)
console.log(typeof function(){}); // "function"

// Check for array
console.log(Array.isArray([]));  // true
console.log(Array.isArray({}));  // false
```

---

## Type Conversion

### Implicit (Automatic) Conversion

```javascript
// String concatenation
console.log("5" + 3);       // "53" (number to string)
console.log("5" + true);    // "5true"

// Numeric operations (except +)
console.log("5" - 3);       // 2
console.log("5" * 3);       // 15
console.log("5" / 2);       // 2.5
console.log("5" - "2");     // 3

// Boolean context
if ("hello") {              // String to boolean
    console.log("truthy");
}

// Comparison
console.log("5" == 5);      // true (type coercion)
console.log("5" === 5);     // false (strict, no coercion)
```

### Explicit Conversion

```javascript
// To String
String(123);        // "123"
String(true);       // "true"
String(null);       // "null"
(123).toString();   // "123"

// To Number
Number("123");      // 123
Number("12.34");    // 12.34
Number("");         // 0
Number("hello");    // NaN
Number(true);       // 1
Number(false);      // 0
Number(null);       // 0
Number(undefined);  // NaN
parseInt("123");    // 123
parseFloat("12.34"); // 12.34
+"123";             // 123 (unary plus)

// To Boolean
Boolean(1);         // true
Boolean(0);         // false
Boolean("hello");   // true
Boolean("");        // false
!!value;            // Double NOT (converts to boolean)
```

---

## Symbol (Brief Introduction)

Symbols are unique identifiers, mainly used for object properties:

```javascript
// Creating symbols
let sym1 = Symbol();
let sym2 = Symbol();
console.log(sym1 === sym2);  // false (always unique)

// Symbols with description
let id = Symbol("id");
console.log(id.description);  // "id"

// Use case: unique object keys
const ID = Symbol("id");
const user = {
    name: "John",
    [ID]: 12345
};
console.log(user[ID]);  // 12345

// Symbols are not enumerable
console.log(Object.keys(user));  // ["name"]
```

---

## Practice Exercises

### Exercise 1: String Manipulation

```javascript
// Given this string:
let text = "  JavaScript is AWESOME!  ";

// 1. Trim whitespace
// 2. Convert to lowercase
// 3. Replace "awesome" with "amazing"
// 4. Get the length

// Your code here:
```

**Solution:**
```javascript
let text = "  JavaScript is AWESOME!  ";
let result = text.trim().toLowerCase().replace("awesome", "amazing");
console.log(result);  // "javascript is amazing!"
console.log(result.length);  // 23
```

### Exercise 2: Type Checking

```javascript
// Determine the type and converted values:
let a = "123";
let b = null;
let c = undefined;
let d = "hello";

// What are their types?
// What happens when you convert to number?
```

**Solution:**
```javascript
console.log(typeof "123", Number("123"));      // string, 123
console.log(typeof null, Number(null));        // object, 0
console.log(typeof undefined, Number(undefined)); // undefined, NaN
console.log(typeof "hello", Number("hello"));  // string, NaN
```

### Exercise 3: Random Number Generator

```javascript
// Create a function that generates a random integer
// between two numbers (inclusive)

function randomBetween(min, max) {
    // Your code here
}

console.log(randomBetween(1, 6));  // Dice roll
```

**Solution:**
```javascript
function randomBetween(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
```

---

## Key Takeaways

1. **Strings** are immutable - methods return new strings
2. **Template literals** (backticks) are the modern way for strings
3. **Numbers** include both integers and decimals
4. **0.1 + 0.2 !== 0.3** due to floating-point precision
5. **Falsy values**: false, 0, "", null, undefined, NaN
6. **typeof null** returns "object" (historical bug)
7. **Use === instead of ==** for type-safe comparisons
8. **Symbol** creates unique identifiers

---

**Next Lesson:** [Day 3 - Operators & Conditionals](./week-01-day-03-operators-conditionals.md)
