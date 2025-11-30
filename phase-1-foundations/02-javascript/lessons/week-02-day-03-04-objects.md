# Week 2, Day 3-4: Objects

## What are Objects?

Objects are collections of key-value pairs. They're the most important data structure in JavaScript and are used to represent real-world entities.

```javascript
// Object represents a person
const person = {
    name: "John",
    age: 30,
    isEmployed: true,
    hobbies: ["reading", "coding"]
};

// Object represents a product
const product = {
    id: 1,
    name: "Laptop",
    price: 999.99,
    inStock: true
};
```

---

## Creating Objects

### Object Literal (Most Common)

```javascript
// Empty object
const empty = {};

// Object with properties
const user = {
    firstName: "John",
    lastName: "Doe",
    age: 30,
    email: "john@example.com"
};

// Property names can be strings (required if contains special chars/spaces)
const special = {
    "first-name": "John",
    "last name": "Doe",
    "123": "number key"
};

// Computed property names
const key = "dynamicKey";
const obj = {
    [key]: "value",
    [`${key}2`]: "value2"
};
console.log(obj);  // { dynamicKey: "value", dynamicKey2: "value2" }
```

### Object Constructor

```javascript
// Using Object constructor (rarely used)
const user = new Object();
user.name = "John";
user.age = 30;

// Object.create() - create with specific prototype
const personProto = {
    greet() {
        return `Hello, I'm ${this.name}`;
    }
};

const john = Object.create(personProto);
john.name = "John";
console.log(john.greet());  // "Hello, I'm John"

// Create object with no prototype
const pureObject = Object.create(null);
console.log(pureObject.toString);  // undefined (no inherited methods)
```

### Shorthand Properties (ES6)

```javascript
const name = "John";
const age = 30;

// Old way
const userOld = {
    name: name,
    age: age
};

// Shorthand (when property name equals variable name)
const user = { name, age };
console.log(user);  // { name: "John", age: 30 }

// Mixed
const email = "john@example.com";
const user2 = {
    name,
    age,
    email,
    city: "New York"  // Regular property
};
```

---

## Accessing Properties

### Dot Notation

```javascript
const user = {
    name: "John",
    age: 30,
    address: {
        city: "New York",
        zip: "10001"
    }
};

console.log(user.name);           // "John"
console.log(user.age);            // 30
console.log(user.address.city);   // "New York"
console.log(user.email);          // undefined (doesn't exist)
```

### Bracket Notation

```javascript
const user = {
    name: "John",
    "first-name": "John",
    age: 30
};

// Same as dot notation
console.log(user["name"]);        // "John"
console.log(user["age"]);         // 30

// Required for special characters
console.log(user["first-name"]);  // "John"
// user.first-name  // ERROR: interpreted as user.first minus name

// Dynamic property access
const prop = "name";
console.log(user[prop]);          // "John"

// Computed access
const index = 2;
const props = ["name", "age", "city"];
console.log(user[props[index - 1]]);  // 30 (accesses "age")
```

### Optional Chaining (?.)

```javascript
const user = {
    name: "John",
    address: {
        city: "New York"
    }
};

// Without optional chaining
// console.log(user.profile.avatar);  // ERROR!

// With optional chaining
console.log(user.profile?.avatar);   // undefined (no error)
console.log(user.address?.city);     // "New York"

// Chaining multiple levels
const company = {
    ceo: {
        name: "Jane",
        contact: { email: "jane@company.com" }
    }
};

console.log(company.ceo?.contact?.email);    // "jane@company.com"
console.log(company.cfo?.contact?.email);    // undefined
console.log(company.ceo?.contact?.phone);    // undefined

// With bracket notation
const key = "contact";
console.log(company.ceo?.[key]?.email);      // "jane@company.com"

// With method calls
const user2 = {
    getName() {
        return "John";
    }
};

console.log(user2.getName?.());    // "John"
console.log(user2.getAge?.());     // undefined
```

---

## Modifying Objects

### Adding Properties

```javascript
const user = { name: "John" };

// Add new properties
user.age = 30;
user["email"] = "john@example.com";
user.address = { city: "New York" };

console.log(user);
// { name: "John", age: 30, email: "john@example.com", address: { city: "New York" } }
```

### Updating Properties

```javascript
const user = {
    name: "John",
    age: 30
};

user.name = "Jane";
user.age = 25;

console.log(user);  // { name: "Jane", age: 25 }
```

### Deleting Properties

```javascript
const user = {
    name: "John",
    age: 30,
    email: "john@example.com"
};

delete user.email;
console.log(user);  // { name: "John", age: 30 }

// Check if property exists
console.log("email" in user);  // false
```

### Preventing Modifications

```javascript
// Freeze - no changes allowed
const frozen = Object.freeze({
    name: "John",
    age: 30
});

frozen.name = "Jane";      // Silently fails (or error in strict mode)
frozen.email = "test";     // Silently fails
delete frozen.age;         // Silently fails
console.log(frozen);       // { name: "John", age: 30 }

// Seal - can modify existing, but can't add/delete
const sealed = Object.seal({
    name: "John",
    age: 30
});

sealed.name = "Jane";      // Works!
sealed.email = "test";     // Silently fails
delete sealed.age;         // Silently fails
console.log(sealed);       // { name: "Jane", age: 30 }

// Check status
console.log(Object.isFrozen(frozen));  // true
console.log(Object.isSealed(sealed));  // true

// Note: Freeze/seal are shallow (nested objects not affected)
const nested = Object.freeze({
    data: { value: 1 }
});
nested.data.value = 2;     // Works! (nested object not frozen)
console.log(nested.data.value);  // 2
```

---

## Object Methods

### Methods in Objects

```javascript
const user = {
    firstName: "John",
    lastName: "Doe",

    // Method (function property)
    getFullName: function() {
        return `${this.firstName} ${this.lastName}`;
    },

    // Shorthand method syntax (ES6)
    greet() {
        return `Hello, I'm ${this.firstName}`;
    },

    // Arrow function (careful with 'this'!)
    // arrowMethod: () => {
    //     return this.firstName;  // 'this' is NOT the object!
    // }
};

console.log(user.getFullName());  // "John Doe"
console.log(user.greet());        // "Hello, I'm John"
```

### The 'this' Keyword

```javascript
const user = {
    name: "John",
    greet() {
        console.log(`Hello, I'm ${this.name}`);
    }
};

user.greet();  // "Hello, I'm John"

// 'this' refers to the object that calls the method
const greetFunc = user.greet;
greetFunc();  // "Hello, I'm undefined" (or error in strict mode)
// 'this' is now global object (or undefined in strict mode)

// Fix with bind
const boundGreet = user.greet.bind(user);
boundGreet();  // "Hello, I'm John"

// Arrow functions don't have their own 'this'
const user2 = {
    name: "Jane",
    greet: () => {
        console.log(`Hello, I'm ${this.name}`);  // 'this' is outer scope
    },
    delayedGreet() {
        // Arrow function here is useful - inherits 'this'
        setTimeout(() => {
            console.log(`Hello, I'm ${this.name}`);  // Works!
        }, 1000);
    }
};

user2.greet();  // "Hello, I'm undefined" (arrow doesn't bind 'this')
user2.delayedGreet();  // After 1s: "Hello, I'm Jane"
```

---

## Object Iteration

### for...in Loop

```javascript
const user = {
    name: "John",
    age: 30,
    city: "New York"
};

// Iterates over enumerable properties
for (let key in user) {
    console.log(`${key}: ${user[key]}`);
}
// name: John
// age: 30
// city: New York

// Also iterates inherited properties!
const child = Object.create(user);
child.hobby = "coding";

for (let key in child) {
    console.log(key);  // hobby, name, age, city (includes inherited)
}

// Filter to own properties only
for (let key in child) {
    if (child.hasOwnProperty(key)) {
        console.log(key);  // hobby (only own property)
    }
}
```

### Object.keys(), Object.values(), Object.entries()

```javascript
const user = {
    name: "John",
    age: 30,
    city: "New York"
};

// Get all keys
const keys = Object.keys(user);
console.log(keys);  // ["name", "age", "city"]

// Get all values
const values = Object.values(user);
console.log(values);  // ["John", 30, "New York"]

// Get key-value pairs as arrays
const entries = Object.entries(user);
console.log(entries);
// [["name", "John"], ["age", 30], ["city", "New York"]]

// Iterating with forEach
Object.keys(user).forEach(key => {
    console.log(`${key}: ${user[key]}`);
});

// Iterating with for...of
for (const [key, value] of Object.entries(user)) {
    console.log(`${key}: ${value}`);
}

// Creating object from entries
const entriesArr = [["a", 1], ["b", 2], ["c", 3]];
const obj = Object.fromEntries(entriesArr);
console.log(obj);  // { a: 1, b: 2, c: 3 }
```

---

## Object Methods (Static)

### Object.assign()

```javascript
// Merge objects (shallow copy)
const target = { a: 1, b: 2 };
const source = { b: 3, c: 4 };

const result = Object.assign(target, source);
console.log(result);  // { a: 1, b: 3, c: 4 }
console.log(target);  // { a: 1, b: 3, c: 4 } (target is modified!)

// Copy object (don't modify original)
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original);
console.log(copy);  // { a: 1, b: 2 }

// Merge multiple sources
const merged = Object.assign({},
    { a: 1 },
    { b: 2 },
    { c: 3 }
);
console.log(merged);  // { a: 1, b: 2, c: 3 }

// Later sources override earlier ones
const override = Object.assign({},
    { name: "John" },
    { name: "Jane" }
);
console.log(override);  // { name: "Jane" }
```

### Spread Operator (Modern Alternative)

```javascript
const user = { name: "John", age: 30 };

// Copy object
const copy = { ...user };

// Merge objects
const merged = { ...user, city: "New York" };
console.log(merged);  // { name: "John", age: 30, city: "New York" }

// Override properties
const updated = { ...user, age: 31 };
console.log(updated);  // { name: "John", age: 31 }

// Merge multiple objects
const obj1 = { a: 1 };
const obj2 = { b: 2 };
const obj3 = { c: 3 };
const combined = { ...obj1, ...obj2, ...obj3 };
console.log(combined);  // { a: 1, b: 2, c: 3 }

// Order matters for overriding
const overridden = {
    ...{ name: "John" },
    name: "Jane"
};
console.log(overridden);  // { name: "Jane" }
```

### Object.hasOwn() and hasOwnProperty()

```javascript
const user = { name: "John" };

// Check if property exists directly on object (not inherited)
console.log(Object.hasOwn(user, "name"));      // true
console.log(Object.hasOwn(user, "toString"));  // false (inherited)

// Older method (still works)
console.log(user.hasOwnProperty("name"));      // true

// Check if property exists at all (including inherited)
console.log("name" in user);       // true
console.log("toString" in user);   // true (inherited from Object)
```

### Checking Object Properties

```javascript
const user = {
    name: "John",
    age: null,
    email: undefined
};

// Check if property exists
console.log("name" in user);   // true
console.log("city" in user);   // false

// Check for undefined values
console.log(user.email === undefined);  // true
console.log(user.city === undefined);   // true (also true for missing!)

// Better: use 'in' or hasOwn
console.log("email" in user);  // true (exists, but undefined)
console.log("city" in user);   // false (doesn't exist)

// Check for null or undefined
console.log(user.age == null);    // true (== catches both)
console.log(user.email == null);  // true
```

---

## Destructuring Objects

### Basic Destructuring

```javascript
const user = {
    name: "John",
    age: 30,
    city: "New York"
};

// Extract properties into variables
const { name, age, city } = user;
console.log(name);  // "John"
console.log(age);   // 30
console.log(city);  // "New York"

// Order doesn't matter (unlike arrays)
const { city: c, name: n } = user;
console.log(n, c);  // "John", "New York"
```

### Renaming Variables

```javascript
const user = {
    name: "John",
    age: 30
};

// Rename during destructuring
const { name: userName, age: userAge } = user;
console.log(userName);  // "John"
console.log(userAge);   // 30

// Useful when property names conflict
const response = { data: { name: "John" } };
const { data: userData } = response;
console.log(userData);  // { name: "John" }
```

### Default Values

```javascript
const user = {
    name: "John"
};

// Provide defaults for missing properties
const { name, age = 25, city = "Unknown" } = user;
console.log(name);  // "John"
console.log(age);   // 25 (default)
console.log(city);  // "Unknown" (default)

// Combine with renaming
const { name: n, age: a = 30 } = user;
console.log(n, a);  // "John", 30
```

### Nested Destructuring

```javascript
const user = {
    name: "John",
    address: {
        city: "New York",
        country: "USA"
    },
    scores: [90, 85, 92]
};

// Destructure nested object
const { name, address: { city, country } } = user;
console.log(city);     // "New York"
console.log(country);  // "USA"

// Destructure with array
const { scores: [first, second] } = user;
console.log(first, second);  // 90, 85

// Deep nesting with defaults
const {
    name: n,
    address: {
        city: c = "Unknown",
        zip = "00000"
    } = {}
} = user;
console.log(n, c, zip);  // "John", "New York", "00000"
```

### Rest Pattern in Destructuring

```javascript
const user = {
    name: "John",
    age: 30,
    city: "New York",
    country: "USA"
};

// Extract some, collect rest
const { name, age, ...rest } = user;
console.log(name);  // "John"
console.log(age);   // 30
console.log(rest);  // { city: "New York", country: "USA" }

// Useful for removing properties
const { city, ...userWithoutCity } = user;
console.log(userWithoutCity);  // { name: "John", age: 30, country: "USA" }
```

### Destructuring in Function Parameters

```javascript
// Without destructuring
function greet(user) {
    console.log(`Hello, ${user.name}!`);
}

// With destructuring
function greet({ name, age }) {
    console.log(`Hello, ${name}! You are ${age}.`);
}

greet({ name: "John", age: 30 });  // "Hello, John! You are 30."

// With defaults
function greet({ name = "Guest", age = 0 } = {}) {
    console.log(`Hello, ${name}! You are ${age}.`);
}

greet();                    // "Hello, Guest! You are 0."
greet({ name: "John" });    // "Hello, John! You are 0."

// Complex destructuring in parameters
function processUser({
    name,
    address: { city } = {},
    settings: { theme = "light" } = {}
}) {
    console.log(`${name} from ${city}, using ${theme} theme`);
}

processUser({
    name: "John",
    address: { city: "NYC" }
});
// "John from NYC, using light theme"
```

---

## Object Comparison

### Reference vs Value

```javascript
// Objects are compared by reference, not value
const obj1 = { name: "John" };
const obj2 = { name: "John" };
const obj3 = obj1;

console.log(obj1 === obj2);  // false (different references)
console.log(obj1 === obj3);  // true (same reference)

// Modifying through one reference affects all
obj3.name = "Jane";
console.log(obj1.name);  // "Jane" (same object!)
```

### Deep Equality Check

```javascript
// Shallow comparison
function shallowEqual(obj1, obj2) {
    const keys1 = Object.keys(obj1);
    const keys2 = Object.keys(obj2);

    if (keys1.length !== keys2.length) return false;

    return keys1.every(key => obj1[key] === obj2[key]);
}

console.log(shallowEqual({ a: 1 }, { a: 1 }));  // true
console.log(shallowEqual({ a: { b: 1 } }, { a: { b: 1 } }));  // false (nested!)

// Deep comparison (simple version)
function deepEqual(obj1, obj2) {
    if (obj1 === obj2) return true;

    if (typeof obj1 !== "object" || typeof obj2 !== "object") return false;
    if (obj1 === null || obj2 === null) return false;

    const keys1 = Object.keys(obj1);
    const keys2 = Object.keys(obj2);

    if (keys1.length !== keys2.length) return false;

    return keys1.every(key => deepEqual(obj1[key], obj2[key]));
}

console.log(deepEqual({ a: { b: 1 } }, { a: { b: 1 } }));  // true

// Practical: Use JSON (simple cases only)
function jsonEqual(obj1, obj2) {
    return JSON.stringify(obj1) === JSON.stringify(obj2);
}
```

---

## Shallow vs Deep Copy

### Shallow Copy

```javascript
const original = {
    name: "John",
    address: { city: "NYC" }
};

// Shallow copies (nested objects still reference original)
const copy1 = { ...original };
const copy2 = Object.assign({}, original);

// Modifying nested object affects original!
copy1.address.city = "LA";
console.log(original.address.city);  // "LA" (!)

// Primitive values are copied
copy1.name = "Jane";
console.log(original.name);  // "John" (unaffected)
```

### Deep Copy

```javascript
// Method 1: JSON (limited - doesn't work with functions, undefined, etc.)
const original = {
    name: "John",
    address: { city: "NYC" }
};

const deepCopy = JSON.parse(JSON.stringify(original));
deepCopy.address.city = "LA";
console.log(original.address.city);  // "NYC" (unaffected!)

// Limitations of JSON method
const withFunction = {
    greet: () => "Hello",
    date: new Date(),
    undef: undefined
};

const jsonCopy = JSON.parse(JSON.stringify(withFunction));
console.log(jsonCopy);
// { date: "2024-01-01T00:00:00.000Z" }
// greet is lost! date is now string! undefined is lost!

// Method 2: structuredClone (modern, better)
const structuredCopy = structuredClone(original);
structuredCopy.address.city = "Chicago";
console.log(original.address.city);  // "NYC"

// structuredClone handles more types but still not functions
const withDate = { date: new Date() };
const cloned = structuredClone(withDate);
console.log(cloned.date instanceof Date);  // true!

// Method 3: Custom recursive function
function deepClone(obj) {
    if (obj === null || typeof obj !== "object") return obj;

    if (Array.isArray(obj)) {
        return obj.map(item => deepClone(item));
    }

    const cloned = {};
    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
            cloned[key] = deepClone(obj[key]);
        }
    }
    return cloned;
}
```

---

## Constructor Functions

Create objects with shared structure:

```javascript
// Constructor function (capitalize first letter by convention)
function Person(name, age) {
    this.name = name;
    this.age = age;

    this.greet = function() {
        return `Hello, I'm ${this.name}`;
    };
}

// Create instances with 'new'
const john = new Person("John", 30);
const jane = new Person("Jane", 25);

console.log(john.name);     // "John"
console.log(jane.name);     // "Jane"
console.log(john.greet());  // "Hello, I'm John"

// Check instance
console.log(john instanceof Person);  // true

// What 'new' does:
// 1. Creates a new empty object
// 2. Sets prototype
// 3. Binds 'this' to new object
// 4. Executes constructor
// 5. Returns the object
```

### Prototypes

```javascript
// Methods on prototype are shared (memory efficient)
function Person(name, age) {
    this.name = name;
    this.age = age;
}

// Add method to prototype
Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

Person.prototype.getAge = function() {
    return this.age;
};

const john = new Person("John", 30);
const jane = new Person("Jane", 25);

// Both share the same method
console.log(john.greet === jane.greet);  // true

// Check prototype
console.log(Object.getPrototypeOf(john) === Person.prototype);  // true
```

---

## ES6 Classes

Modern syntax for constructor functions:

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

    // Static method (called on class, not instance)
    static create(name, age) {
        return new Person(name, age);
    }
}

// Create instance
const john = new Person("John", 30);
console.log(john.greet());  // "Hello, I'm John"
console.log(john.info);     // "John, 30 years old"

john.nickname = "Johnny";
console.log(john.nickname); // "Johnny"

// Static method
const jane = Person.create("Jane", 25);
console.log(jane.name);  // "Jane"
```

### Class Inheritance

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

    // Override parent method
    speak() {
        return `${this.name} barks`;
    }

    // New method
    fetch() {
        return `${this.name} fetches the ball`;
    }
}

const dog = new Dog("Rex", "German Shepherd");
console.log(dog.name);    // "Rex"
console.log(dog.breed);   // "German Shepherd"
console.log(dog.speak()); // "Rex barks"
console.log(dog.fetch()); // "Rex fetches the ball"

console.log(dog instanceof Dog);     // true
console.log(dog instanceof Animal);  // true
```

### Private Fields (ES2022)

```javascript
class BankAccount {
    // Private field (starts with #)
    #balance = 0;

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

    getBalance() {
        return this.#balance;
    }
}

const account = new BankAccount(100);
account.deposit(50);
console.log(account.getBalance());  // 150

// Cannot access private field directly
// console.log(account.#balance);  // Syntax Error!
console.log(account.balance);       // undefined
```

---

## Practical Examples

### Example 1: User Management

```javascript
class User {
    #password;

    constructor({ name, email, password }) {
        this.name = name;
        this.email = email;
        this.#password = password;
        this.createdAt = new Date();
    }

    validatePassword(password) {
        return this.#password === password;
    }

    updatePassword(oldPassword, newPassword) {
        if (this.validatePassword(oldPassword)) {
            this.#password = newPassword;
            return true;
        }
        return false;
    }

    toJSON() {
        return {
            name: this.name,
            email: this.email,
            createdAt: this.createdAt
        };
    }

    static fromJSON(json) {
        const data = typeof json === "string" ? JSON.parse(json) : json;
        return new User({ ...data, password: "" });
    }
}

const user = new User({
    name: "John",
    email: "john@example.com",
    password: "secret123"
});

console.log(user.validatePassword("secret123"));  // true
console.log(user.toJSON());
// { name: "John", email: "john@example.com", createdAt: ... }
```

### Example 2: Shopping Cart

```javascript
class ShoppingCart {
    #items = [];

    addItem(product, quantity = 1) {
        const existing = this.#items.find(item => item.product.id === product.id);

        if (existing) {
            existing.quantity += quantity;
        } else {
            this.#items.push({ product, quantity });
        }
    }

    removeItem(productId) {
        this.#items = this.#items.filter(item => item.product.id !== productId);
    }

    updateQuantity(productId, quantity) {
        const item = this.#items.find(item => item.product.id === productId);
        if (item) {
            item.quantity = quantity;
            if (quantity <= 0) {
                this.removeItem(productId);
            }
        }
    }

    get items() {
        return [...this.#items];  // Return copy
    }

    get total() {
        return this.#items.reduce((sum, item) => {
            return sum + (item.product.price * item.quantity);
        }, 0);
    }

    get itemCount() {
        return this.#items.reduce((count, item) => count + item.quantity, 0);
    }

    clear() {
        this.#items = [];
    }
}

const cart = new ShoppingCart();

cart.addItem({ id: 1, name: "Shirt", price: 29.99 }, 2);
cart.addItem({ id: 2, name: "Pants", price: 49.99 });
cart.addItem({ id: 1, name: "Shirt", price: 29.99 });  // Adds to existing

console.log(cart.items);
console.log(`Total: $${cart.total.toFixed(2)}`);   // $139.96
console.log(`Items: ${cart.itemCount}`);           // 4
```

### Example 3: State Management

```javascript
function createStore(initialState = {}) {
    let state = { ...initialState };
    const listeners = [];

    return {
        getState() {
            return { ...state };
        },

        setState(newState) {
            state = { ...state, ...newState };
            listeners.forEach(listener => listener(state));
        },

        subscribe(listener) {
            listeners.push(listener);
            // Return unsubscribe function
            return () => {
                const index = listeners.indexOf(listener);
                if (index > -1) {
                    listeners.splice(index, 1);
                }
            };
        },

        reset() {
            state = { ...initialState };
            listeners.forEach(listener => listener(state));
        }
    };
}

// Usage
const store = createStore({ count: 0, user: null });

const unsubscribe = store.subscribe(state => {
    console.log("State changed:", state);
});

store.setState({ count: 1 });  // "State changed: { count: 1, user: null }"
store.setState({ user: { name: "John" } });  // "State changed: { count: 1, user: {...} }"

unsubscribe();  // Stop listening
store.setState({ count: 2 });  // No log (unsubscribed)
```

---

## Practice Exercises

### Exercise 1: Deep Merge

```javascript
// Create a function that deeply merges two objects

function deepMerge(target, source) {
    // Your code here
}

const obj1 = {
    a: 1,
    b: { c: 2, d: 3 },
    e: [1, 2]
};

const obj2 = {
    b: { c: 4, f: 5 },
    e: [3, 4],
    g: 6
};

console.log(deepMerge(obj1, obj2));
// { a: 1, b: { c: 4, d: 3, f: 5 }, e: [3, 4], g: 6 }
```

**Solution:**
```javascript
function deepMerge(target, source) {
    const result = { ...target };

    for (const key of Object.keys(source)) {
        if (source[key] instanceof Object && !Array.isArray(source[key])) {
            if (target[key] instanceof Object && !Array.isArray(target[key])) {
                result[key] = deepMerge(target[key], source[key]);
            } else {
                result[key] = { ...source[key] };
            }
        } else {
            result[key] = source[key];
        }
    }

    return result;
}
```

### Exercise 2: Object Difference

```javascript
// Find the difference between two objects

function objectDiff(obj1, obj2) {
    // Your code here
    // Return { added: {...}, removed: {...}, changed: {...} }
}

const before = { a: 1, b: 2, c: 3 };
const after = { a: 1, b: 5, d: 4 };

console.log(objectDiff(before, after));
// { added: { d: 4 }, removed: { c: 3 }, changed: { b: { from: 2, to: 5 } } }
```

**Solution:**
```javascript
function objectDiff(obj1, obj2) {
    const added = {};
    const removed = {};
    const changed = {};

    // Find removed and changed
    for (const key of Object.keys(obj1)) {
        if (!(key in obj2)) {
            removed[key] = obj1[key];
        } else if (obj1[key] !== obj2[key]) {
            changed[key] = { from: obj1[key], to: obj2[key] };
        }
    }

    // Find added
    for (const key of Object.keys(obj2)) {
        if (!(key in obj1)) {
            added[key] = obj2[key];
        }
    }

    return { added, removed, changed };
}
```

### Exercise 3: Object Path Access

```javascript
// Get/set values using dot-notation path strings

function getByPath(obj, path) {
    // Your code here
}

function setByPath(obj, path, value) {
    // Your code here
}

const data = {
    user: {
        name: "John",
        address: {
            city: "NYC"
        }
    }
};

console.log(getByPath(data, "user.name"));          // "John"
console.log(getByPath(data, "user.address.city"));  // "NYC"
console.log(getByPath(data, "user.age"));           // undefined

setByPath(data, "user.age", 30);
console.log(data.user.age);  // 30
```

**Solution:**
```javascript
function getByPath(obj, path) {
    return path.split(".").reduce((current, key) => {
        return current?.[key];
    }, obj);
}

function setByPath(obj, path, value) {
    const keys = path.split(".");
    const lastKey = keys.pop();

    const target = keys.reduce((current, key) => {
        if (!(key in current)) {
            current[key] = {};
        }
        return current[key];
    }, obj);

    target[lastKey] = value;
}
```

---

## Key Takeaways

1. **Objects store key-value pairs** and are passed by reference
2. **Dot notation** for simple keys, **bracket notation** for dynamic/special keys
3. **Optional chaining (?.)** safely accesses nested properties
4. **Destructuring** extracts properties into variables
5. **Spread operator** copies and merges objects (shallow)
6. **Object.keys/values/entries** iterate object properties
7. **Classes** are modern syntax for constructor functions
8. **Private fields (#)** hide implementation details
9. **Shallow copy** doesn't clone nested objects
10. **structuredClone** creates deep copies (most types)

---

## Self-Check Questions

1. What's the difference between dot and bracket notation?
2. How do you safely access nested properties?
3. What's the difference between shallow and deep copy?
4. What does Object.freeze() do?
5. How do you iterate over object properties?
6. What's the difference between 'in' and hasOwn()?
7. How does 'this' work in object methods?

---

**Next Lesson:** [Day 5 - DOM Basics](./week-02-day-05-dom-basics.md)
