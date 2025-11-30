# Week 4, Day 1-2: ES6+ Features

## Introduction

ES6 (ECMAScript 2015) and later versions introduced major improvements to JavaScript. These features make code more readable, maintainable, and powerful.

---

## let and const (ES6)

Already covered in Week 1, but as a reminder:

```javascript
// let - for variables that can be reassigned
let count = 0;
count = 1;

// const - for variables that won't be reassigned
const PI = 3.14159;
// PI = 3; // Error!

// const for objects/arrays - content can change, reference can't
const user = { name: "John" };
user.name = "Jane";  // OK
// user = {};  // Error!

// Block scoping
if (true) {
    let x = 10;
    const y = 20;
}
// console.log(x, y);  // Error - not accessible
```

---

## Arrow Functions (ES6)

Concise function syntax:

```javascript
// Regular function
function add(a, b) {
    return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// With single parameter (no parentheses needed)
const double = n => n * 2;

// With no parameters
const greet = () => "Hello!";

// With multiple statements
const process = (x) => {
    const result = x * 2;
    return result + 1;
};

// Returning object (wrap in parentheses)
const createUser = (name, age) => ({ name, age });

// Arrow functions and 'this'
const obj = {
    name: "John",
    greet: function() {
        // Regular function: 'this' is the object
        setTimeout(() => {
            // Arrow function: inherits 'this' from parent
            console.log(this.name);  // "John"
        }, 100);
    }
};
```

---

## Template Literals (ES6)

String interpolation and multi-line strings:

```javascript
const name = "John";
const age = 30;

// String interpolation
const message = `Hello, ${name}! You are ${age} years old.`;

// Expressions in templates
const price = 19.99;
const quantity = 3;
const total = `Total: $${(price * quantity).toFixed(2)}`;

// Multi-line strings
const html = `
    <div class="card">
        <h2>${name}</h2>
        <p>Age: ${age}</p>
    </div>
`;

// Tagged templates
function highlight(strings, ...values) {
    return strings.reduce((result, str, i) => {
        return result + str + (values[i] ? `<mark>${values[i]}</mark>` : "");
    }, "");
}

const highlighted = highlight`Hello ${name}, you are ${age}!`;
// "Hello <mark>John</mark>, you are <mark>30</mark>!"

// Raw strings
const path = String.raw`C:\Users\John\Documents`;
// "C:\Users\John\Documents" (backslashes not escaped)
```

---

## Destructuring (ES6)

Extract values from arrays and objects:

### Array Destructuring

```javascript
const numbers = [1, 2, 3, 4, 5];

// Basic destructuring
const [first, second] = numbers;
console.log(first, second);  // 1, 2

// Skip elements
const [, , third] = numbers;
console.log(third);  // 3

// Rest pattern
const [head, ...rest] = numbers;
console.log(head);  // 1
console.log(rest);  // [2, 3, 4, 5]

// Default values
const [a, b, c, d, e, f = 6] = numbers;
console.log(f);  // 6

// Swap variables
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y);  // 2, 1

// Nested destructuring
const nested = [1, [2, 3], 4];
const [one, [two, three]] = nested;
```

### Object Destructuring

```javascript
const user = {
    name: "John",
    age: 30,
    email: "john@example.com",
    address: {
        city: "New York",
        country: "USA"
    }
};

// Basic destructuring
const { name, age } = user;
console.log(name, age);  // "John", 30

// Rename variables
const { name: userName, age: userAge } = user;
console.log(userName, userAge);

// Default values
const { phone = "N/A" } = user;
console.log(phone);  // "N/A"

// Nested destructuring
const { address: { city, country } } = user;
console.log(city, country);  // "New York", "USA"

// Rest pattern
const { name: n, ...rest } = user;
console.log(rest);  // { age: 30, email: "...", address: {...} }

// Function parameters
function greet({ name, age = 0 }) {
    console.log(`Hello ${name}, you are ${age}`);
}
greet(user);
```

---

## Spread Operator (ES6)

Expand iterables and objects:

```javascript
// Array spreading
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

const combined = [...arr1, ...arr2];  // [1, 2, 3, 4, 5, 6]
const copy = [...arr1];  // Shallow copy

// Insert in middle
const middle = [1, ...arr2, 10];  // [1, 4, 5, 6, 10]

// Function arguments
console.log(Math.max(...arr1));  // 3

// Object spreading
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

const merged = { ...obj1, ...obj2 };  // { a: 1, b: 2, c: 3, d: 4 }
const updated = { ...obj1, b: 10 };   // { a: 1, b: 10 }

// Clone object
const clone = { ...obj1 };

// Shallow copy warning
const withNested = { data: { value: 1 } };
const shallowClone = { ...withNested };
shallowClone.data.value = 2;
console.log(withNested.data.value);  // 2 (affected!)
```

---

## Rest Parameters (ES6)

Collect remaining arguments:

```javascript
// Rest in functions
function sum(...numbers) {
    return numbers.reduce((total, n) => total + n, 0);
}
console.log(sum(1, 2, 3, 4));  // 10

// With regular parameters
function greet(greeting, ...names) {
    return names.map(name => `${greeting}, ${name}!`);
}
console.log(greet("Hello", "John", "Jane"));
// ["Hello, John!", "Hello, Jane!"]

// Rest must be last
// function bad(...rest, last) {}  // Error!
```

---

## Default Parameters (ES6)

```javascript
function greet(name = "Guest", greeting = "Hello") {
    return `${greeting}, ${name}!`;
}

console.log(greet());                    // "Hello, Guest!"
console.log(greet("John"));              // "Hello, John!"
console.log(greet("John", "Hi"));        // "Hi, John!"
console.log(greet(undefined, "Hey"));    // "Hey, Guest!"

// Using previous parameters
function createUser(name, email = `${name.toLowerCase()}@example.com`) {
    return { name, email };
}

// Using functions
function getDefaultId() {
    return Date.now();
}

function createItem(name, id = getDefaultId()) {
    return { name, id };
}
```

---

## Enhanced Object Literals (ES6)

```javascript
const name = "John";
const age = 30;

// Shorthand property names
const user = { name, age };
// Same as: { name: name, age: age }

// Shorthand methods
const obj = {
    greet() {
        return "Hello!";
    },
    // Same as: greet: function() { return "Hello!"; }
};

// Computed property names
const key = "dynamic";
const obj2 = {
    [key]: "value",
    [`${key}Key`]: "value2"
};
console.log(obj2);  // { dynamic: "value", dynamicKey: "value2" }

// Getters and setters
const person = {
    firstName: "John",
    lastName: "Doe",

    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    },

    set fullName(value) {
        [this.firstName, this.lastName] = value.split(" ");
    }
};

console.log(person.fullName);  // "John Doe"
person.fullName = "Jane Smith";
console.log(person.firstName);  // "Jane"
```

---

## Classes (ES6)

Object-oriented programming with classes:

```javascript
class Person {
    // Constructor
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    // Instance method
    greet() {
        return `Hello, I'm ${this.name}`;
    }

    // Getter
    get info() {
        return `${this.name}, ${this.age} years old`;
    }

    // Setter
    set nickname(value) {
        this._nickname = value;
    }

    get nickname() {
        return this._nickname || this.name;
    }

    // Static method
    static create(name, age) {
        return new Person(name, age);
    }
}

// Creating instances
const john = new Person("John", 30);
console.log(john.greet());  // "Hello, I'm John"
console.log(john.info);     // "John, 30 years old"

// Static method
const jane = Person.create("Jane", 25);
```

### Inheritance

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }

    speak() {
        return `${this.name} makes a sound`;
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);  // Call parent constructor
        this.breed = breed;
    }

    speak() {
        return `${this.name} barks`;  // Override method
    }

    fetch() {
        return `${this.name} fetches the ball`;
    }
}

const dog = new Dog("Rex", "German Shepherd");
console.log(dog.speak());  // "Rex barks"
console.log(dog.fetch());  // "Rex fetches the ball"
console.log(dog instanceof Dog);     // true
console.log(dog instanceof Animal);  // true
```

### Private Fields (ES2022)

```javascript
class BankAccount {
    #balance = 0;  // Private field

    constructor(initialBalance) {
        this.#balance = initialBalance;
    }

    deposit(amount) {
        if (amount > 0) {
            this.#balance += amount;
        }
    }

    withdraw(amount) {
        if (amount <= this.#balance) {
            this.#balance -= amount;
            return amount;
        }
        return 0;
    }

    get balance() {
        return this.#balance;
    }

    // Private method
    #validateAmount(amount) {
        return amount > 0 && amount <= this.#balance;
    }
}

const account = new BankAccount(100);
account.deposit(50);
console.log(account.balance);  // 150
// console.log(account.#balance);  // Error!
```

---

## Symbols (ES6)

Unique identifiers:

```javascript
// Creating symbols
const sym1 = Symbol();
const sym2 = Symbol();
console.log(sym1 === sym2);  // false (always unique)

// With description
const id = Symbol("id");
console.log(id.description);  // "id"

// Use as object keys
const ID = Symbol("id");
const user = {
    name: "John",
    [ID]: 12345
};
console.log(user[ID]);  // 12345

// Symbols are not enumerable
console.log(Object.keys(user));  // ["name"]
console.log(Object.getOwnPropertySymbols(user));  // [Symbol(id)]

// Well-known symbols
const customIterable = {
    data: [1, 2, 3],
    [Symbol.iterator]() {
        let index = 0;
        const data = this.data;
        return {
            next() {
                if (index < data.length) {
                    return { value: data[index++], done: false };
                }
                return { done: true };
            }
        };
    }
};

for (const item of customIterable) {
    console.log(item);  // 1, 2, 3
}

// Global symbol registry
const globalSym = Symbol.for("app.id");
const sameSym = Symbol.for("app.id");
console.log(globalSym === sameSym);  // true
console.log(Symbol.keyFor(globalSym));  // "app.id"
```

---

## Iterators and Generators (ES6)

### Iterators

```javascript
// Custom iterator
const range = {
    start: 1,
    end: 5,

    [Symbol.iterator]() {
        let current = this.start;
        const end = this.end;

        return {
            next() {
                if (current <= end) {
                    return { value: current++, done: false };
                }
                return { done: true };
            }
        };
    }
};

for (const num of range) {
    console.log(num);  // 1, 2, 3, 4, 5
}

console.log([...range]);  // [1, 2, 3, 4, 5]
```

### Generators

```javascript
// Generator function
function* numberGenerator() {
    yield 1;
    yield 2;
    yield 3;
}

const gen = numberGenerator();
console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }
console.log(gen.next());  // { value: 3, done: false }
console.log(gen.next());  // { value: undefined, done: true }

// Infinite generator
function* infiniteCounter() {
    let count = 0;
    while (true) {
        yield count++;
    }
}

const counter = infiniteCounter();
console.log(counter.next().value);  // 0
console.log(counter.next().value);  // 1
console.log(counter.next().value);  // 2

// Generator with parameters
function* range(start, end) {
    for (let i = start; i <= end; i++) {
        yield i;
    }
}

console.log([...range(1, 5)]);  // [1, 2, 3, 4, 5]

// Passing values into generator
function* conversation() {
    const name = yield "What's your name?";
    const age = yield `Hello ${name}! How old are you?`;
    return `${name} is ${age} years old`;
}

const chat = conversation();
console.log(chat.next().value);           // "What's your name?"
console.log(chat.next("John").value);     // "Hello John! How old are you?"
console.log(chat.next(30).value);         // "John is 30 years old"

// Delegating to another generator
function* combined() {
    yield* range(1, 3);
    yield* range(10, 12);
}

console.log([...combined()]);  // [1, 2, 3, 10, 11, 12]
```

---

## Map and Set (ES6)

### Map

```javascript
// Creating Map
const map = new Map();

// Set values
map.set("name", "John");
map.set("age", 30);
map.set({ id: 1 }, "object key");

// Get values
console.log(map.get("name"));  // "John"
console.log(map.get("unknown"));  // undefined

// Check existence
console.log(map.has("name"));  // true

// Size
console.log(map.size);  // 3

// Delete
map.delete("age");

// Clear all
map.clear();

// Initialize with entries
const map2 = new Map([
    ["key1", "value1"],
    ["key2", "value2"]
]);

// Iteration
for (const [key, value] of map2) {
    console.log(`${key}: ${value}`);
}

map2.forEach((value, key) => {
    console.log(`${key}: ${value}`);
});

// Keys, values, entries
console.log([...map2.keys()]);
console.log([...map2.values()]);
console.log([...map2.entries()]);

// Objects as keys
const objKey = { id: 1 };
const userMap = new Map();
userMap.set(objKey, { name: "John" });
console.log(userMap.get(objKey));  // { name: "John" }
```

### Set

```javascript
// Creating Set
const set = new Set();

// Add values
set.add(1);
set.add(2);
set.add(2);  // Duplicate ignored
set.add("hello");

console.log(set.size);  // 3

// Check existence
console.log(set.has(1));  // true
console.log(set.has(3));  // false

// Delete
set.delete(2);

// Clear all
set.clear();

// Initialize with values
const set2 = new Set([1, 2, 3, 3, 4]);
console.log([...set2]);  // [1, 2, 3, 4]

// Remove duplicates from array
const arr = [1, 2, 2, 3, 3, 3];
const unique = [...new Set(arr)];
console.log(unique);  // [1, 2, 3]

// Iteration
for (const value of set2) {
    console.log(value);
}

set2.forEach(value => console.log(value));

// Set operations
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

// Union
const union = new Set([...setA, ...setB]);  // {1, 2, 3, 4}

// Intersection
const intersection = new Set([...setA].filter(x => setB.has(x)));  // {2, 3}

// Difference
const difference = new Set([...setA].filter(x => !setB.has(x)));  // {1}
```

### WeakMap and WeakSet

```javascript
// WeakMap - keys must be objects, allows garbage collection
const weakMap = new WeakMap();
let obj = { id: 1 };
weakMap.set(obj, "data");
console.log(weakMap.get(obj));  // "data"

obj = null;  // Object can be garbage collected

// Use case: storing private data
const privateData = new WeakMap();

class User {
    constructor(name) {
        privateData.set(this, { secret: "hidden" });
        this.name = name;
    }

    getSecret() {
        return privateData.get(this).secret;
    }
}

// WeakSet - values must be objects
const weakSet = new WeakSet();
const obj2 = { id: 1 };
weakSet.add(obj2);
console.log(weakSet.has(obj2));  // true
```

---

## Promise Methods (ES2020-2021)

```javascript
// Promise.allSettled (ES2020)
const promises = [
    Promise.resolve(1),
    Promise.reject("error"),
    Promise.resolve(3)
];

const results = await Promise.allSettled(promises);
// [
//   { status: "fulfilled", value: 1 },
//   { status: "rejected", reason: "error" },
//   { status: "fulfilled", value: 3 }
// ]

// Promise.any (ES2021)
const fastest = await Promise.any([
    fetch("/api/server1"),
    fetch("/api/server2")
]);  // First successful result
```

---

## Optional Chaining (ES2020)

```javascript
const user = {
    name: "John",
    address: {
        city: "NYC"
    }
};

// Without optional chaining
const city = user && user.address && user.address.city;

// With optional chaining
const city2 = user?.address?.city;  // "NYC"
const zip = user?.address?.zip;      // undefined (no error)

// With arrays
const users = [{ name: "John" }];
console.log(users?.[0]?.name);  // "John"
console.log(users?.[5]?.name);  // undefined

// With methods
const obj = {
    greet() { return "Hello"; }
};
console.log(obj.greet?.());    // "Hello"
console.log(obj.goodbye?.());  // undefined
```

---

## Nullish Coalescing (ES2020)

```javascript
// || returns first truthy value
console.log(0 || "default");       // "default" (0 is falsy)
console.log("" || "default");      // "default" ("" is falsy)
console.log(false || "default");   // "default"

// ?? returns first defined (non-null/undefined) value
console.log(0 ?? "default");       // 0
console.log("" ?? "default");      // ""
console.log(false ?? "default");   // false
console.log(null ?? "default");    // "default"
console.log(undefined ?? "default"); // "default"

// Useful for default values
function getConfig(options = {}) {
    return {
        timeout: options.timeout ?? 5000,
        retries: options.retries ?? 3,
        enabled: options.enabled ?? true
    };
}

getConfig({ timeout: 0 });  // { timeout: 0, retries: 3, enabled: true }
```

---

## Logical Assignment Operators (ES2021)

```javascript
// ||= (or-assign)
let x = null;
x ||= "default";  // x = x || "default"
console.log(x);   // "default"

// &&= (and-assign)
let y = { name: "John" };
y &&= { ...y, updated: true };  // Only assign if y is truthy
console.log(y);  // { name: "John", updated: true }

// ??= (nullish-assign)
let z = null;
z ??= "default";  // Only assign if z is null/undefined
console.log(z);   // "default"

let w = 0;
w ??= 10;
console.log(w);   // 0 (0 is not null/undefined)
```

---

## Array Methods (ES2022-2023)

```javascript
// Array.at() - negative indexing (ES2022)
const arr = [1, 2, 3, 4, 5];
console.log(arr.at(-1));   // 5 (last element)
console.log(arr.at(-2));   // 4 (second to last)

// toSorted, toReversed, toSpliced (ES2023) - non-mutating
const nums = [3, 1, 4, 1, 5];
const sorted = nums.toSorted((a, b) => a - b);
console.log(sorted);  // [1, 1, 3, 4, 5]
console.log(nums);    // [3, 1, 4, 1, 5] (unchanged)

const reversed = nums.toReversed();
console.log(reversed);  // [5, 1, 4, 1, 3]
console.log(nums);      // [3, 1, 4, 1, 5] (unchanged)

const spliced = nums.toSpliced(1, 2, 10, 20);
console.log(spliced);  // [3, 10, 20, 1, 5]
console.log(nums);     // [3, 1, 4, 1, 5] (unchanged)

// with() - replace at index without mutating (ES2023)
const updated = nums.with(0, 100);
console.log(updated);  // [100, 1, 4, 1, 5]
console.log(nums);     // [3, 1, 4, 1, 5] (unchanged)

// findLast, findLastIndex (ES2023)
const lastEven = nums.findLast(n => n % 2 === 0);
console.log(lastEven);  // 4

const lastEvenIndex = nums.findLastIndex(n => n % 2 === 0);
console.log(lastEvenIndex);  // 2
```

---

## Practice Exercises

### Exercise 1: Implement a simple Observable

```javascript
// Create an Observable class using ES6 features
class Observable {
    // Your code here
}

// Usage:
const obs = new Observable();
const unsubscribe = obs.subscribe(value => console.log(value));
obs.next("Hello");  // logs "Hello"
unsubscribe();
obs.next("World");  // nothing logged
```

**Solution:**
```javascript
class Observable {
    #subscribers = new Set();

    subscribe(callback) {
        this.#subscribers.add(callback);
        return () => this.#subscribers.delete(callback);
    }

    next(value) {
        this.#subscribers.forEach(callback => callback(value));
    }
}
```

### Exercise 2: Deep freeze object

```javascript
// Create a function that deeply freezes an object
function deepFreeze(obj) {
    // Your code here
}
```

**Solution:**
```javascript
function deepFreeze(obj) {
    Object.freeze(obj);

    Object.keys(obj).forEach(key => {
        const value = obj[key];
        if (value && typeof value === "object" && !Object.isFrozen(value)) {
            deepFreeze(value);
        }
    });

    return obj;
}
```

---

## Key Takeaways

1. **let/const** replace var with block scoping
2. **Arrow functions** are concise but don't bind `this`
3. **Template literals** enable string interpolation
4. **Destructuring** extracts values elegantly
5. **Spread/rest** operators work with arrays and objects
6. **Classes** provide clean OOP syntax
7. **Map/Set** are better for certain use cases than objects/arrays
8. **Optional chaining** (?.) safely accesses nested properties
9. **Nullish coalescing** (??) handles null/undefined specifically
10. **Private fields** (#) enable true encapsulation

---

**Next Lesson:** [Day 3-4 - Modules](./week-04-day-03-04-modules.md)
