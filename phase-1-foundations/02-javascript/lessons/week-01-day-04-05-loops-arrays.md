# Week 1, Day 4-5: Loops & Arrays

## Loops Overview

Loops repeat code until a condition is met. JavaScript has several loop types:

1. **for** - When you know how many iterations
2. **while** - When you don't know how many iterations
3. **do...while** - Like while, but runs at least once
4. **for...of** - Iterate over array values
5. **for...in** - Iterate over object keys

---

## The for Loop

The most common loop for counted iterations:

```javascript
// Syntax
for (initialization; condition; increment) {
    // code to repeat
}

// Count from 0 to 4
for (let i = 0; i < 5; i++) {
    console.log(i);  // 0, 1, 2, 3, 4
}

// Count from 1 to 5
for (let i = 1; i <= 5; i++) {
    console.log(i);  // 1, 2, 3, 4, 5
}

// Count down
for (let i = 5; i > 0; i--) {
    console.log(i);  // 5, 4, 3, 2, 1
}

// Skip numbers (increment by 2)
for (let i = 0; i <= 10; i += 2) {
    console.log(i);  // 0, 2, 4, 6, 8, 10
}
```

### How for Loop Works

```javascript
for (let i = 0; i < 3; i++) {
    console.log(i);
}

// Step by step:
// 1. let i = 0         (initialization - runs once)
// 2. i < 3 → true      (condition check)
// 3. console.log(0)    (execute body)
// 4. i++  → i = 1      (increment)
// 5. i < 3 → true      (condition check)
// 6. console.log(1)    (execute body)
// 7. i++  → i = 2      (increment)
// 8. i < 3 → true      (condition check)
// 9. console.log(2)    (execute body)
// 10. i++ → i = 3      (increment)
// 11. i < 3 → false    (condition check)
// 12. Loop ends
```

### Practical for Loop Examples

```javascript
// Sum numbers 1 to 100
let sum = 0;
for (let i = 1; i <= 100; i++) {
    sum += i;
}
console.log(sum);  // 5050

// Multiplication table
let num = 7;
for (let i = 1; i <= 10; i++) {
    console.log(`${num} x ${i} = ${num * i}`);
}

// Print pattern
for (let i = 1; i <= 5; i++) {
    console.log("*".repeat(i));
}
// *
// **
// ***
// ****
// *****

// FizzBuzz
for (let i = 1; i <= 15; i++) {
    if (i % 3 === 0 && i % 5 === 0) {
        console.log("FizzBuzz");
    } else if (i % 3 === 0) {
        console.log("Fizz");
    } else if (i % 5 === 0) {
        console.log("Buzz");
    } else {
        console.log(i);
    }
}
```

---

## The while Loop

Loops while a condition is true:

```javascript
// Syntax
while (condition) {
    // code to repeat
}

// Count to 5
let i = 0;
while (i < 5) {
    console.log(i);  // 0, 1, 2, 3, 4
    i++;
}

// Wait for condition
let password = "";
while (password !== "secret") {
    password = prompt("Enter password:");  // Browser only
}

// Process until empty
let items = ["a", "b", "c"];
while (items.length > 0) {
    console.log(items.pop());  // c, b, a
}
```

### When to Use while vs for

```javascript
// Use for: when you know the number of iterations
for (let i = 0; i < 10; i++) {
    console.log(i);
}

// Use while: when iterations depend on a condition
let balance = 1000;
let years = 0;
while (balance < 2000) {
    balance *= 1.05;  // 5% interest
    years++;
}
console.log(`It takes ${years} years to double`);  // 15 years

// Random number until we get 6
let dice;
let rolls = 0;
while (dice !== 6) {
    dice = Math.floor(Math.random() * 6) + 1;
    rolls++;
}
console.log(`Rolled ${rolls} times to get 6`);
```

### Avoiding Infinite Loops

```javascript
// DANGER: Infinite loop (never ends)
while (true) {
    console.log("Forever!");  // Never stops
}

// DANGER: Condition never becomes false
let x = 10;
while (x > 0) {
    console.log(x);
    // Forgot x-- ! Loop never ends
}

// SAFE: Always ensure the condition can become false
let x = 10;
while (x > 0) {
    console.log(x);
    x--;  // This makes x eventually reach 0
}
```

---

## The do...while Loop

Executes at least once, then checks condition:

```javascript
// Syntax
do {
    // code to repeat
} while (condition);

// Always runs at least once
let i = 10;
do {
    console.log(i);  // 10 (runs once even though condition is false)
    i++;
} while (i < 5);

// Compare with while
let j = 10;
while (j < 5) {
    console.log(j);  // Never runs
    j++;
}

// User input validation
let age;
do {
    age = prompt("Enter your age (1-120):");
    age = parseInt(age);
} while (isNaN(age) || age < 1 || age > 120);
console.log(`Your age: ${age}`);

// Menu system
let choice;
do {
    console.log("1. Option A");
    console.log("2. Option B");
    console.log("3. Exit");
    choice = parseInt(prompt("Enter choice:"));

    switch (choice) {
        case 1:
            console.log("You chose A");
            break;
        case 2:
            console.log("You chose B");
            break;
        case 3:
            console.log("Goodbye!");
            break;
        default:
            console.log("Invalid choice");
    }
} while (choice !== 3);
```

---

## break and continue

Control loop execution:

### break - Exit Loop Early

```javascript
// Stop when we find what we're looking for
for (let i = 0; i < 100; i++) {
    if (i === 5) {
        console.log("Found 5!");
        break;  // Exit loop immediately
    }
    console.log(i);  // 0, 1, 2, 3, 4
}
console.log("Loop ended");

// Search for item
let numbers = [1, 3, 5, 7, 9, 2, 4];
let found = false;
for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] === 7) {
        found = true;
        console.log(`Found 7 at index ${i}`);
        break;
    }
}

// While loop with break
let attempts = 0;
while (true) {  // Infinite loop
    attempts++;
    let guess = Math.random();
    if (guess > 0.95) {
        console.log(`Got lucky on attempt ${attempts}!`);
        break;  // Exit infinite loop
    }
}
```

### continue - Skip to Next Iteration

```javascript
// Skip even numbers
for (let i = 0; i < 10; i++) {
    if (i % 2 === 0) {
        continue;  // Skip rest of this iteration
    }
    console.log(i);  // 1, 3, 5, 7, 9
}

// Skip invalid data
let data = [10, -5, 20, null, 30, undefined, 40];
let sum = 0;
for (let i = 0; i < data.length; i++) {
    if (data[i] === null || data[i] === undefined || data[i] < 0) {
        continue;  // Skip invalid values
    }
    sum += data[i];
}
console.log(sum);  // 100

// Process only certain items
let users = [
    { name: "John", active: true },
    { name: "Jane", active: false },
    { name: "Bob", active: true }
];

for (let i = 0; i < users.length; i++) {
    if (!users[i].active) {
        continue;  // Skip inactive users
    }
    console.log(users[i].name);  // John, Bob
}
```

### Labeled Statements (for nested loops)

```javascript
// Break from outer loop
outer: for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        if (i === 1 && j === 1) {
            break outer;  // Break from outer loop
        }
        console.log(`i=${i}, j=${j}`);
    }
}
// i=0, j=0
// i=0, j=1
// i=0, j=2
// i=1, j=0
// (stops when i=1, j=1)

// Continue outer loop
outer: for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        if (j === 1) {
            continue outer;  // Skip to next i
        }
        console.log(`i=${i}, j=${j}`);
    }
}
// i=0, j=0
// i=1, j=0
// i=2, j=0
```

---

## Nested Loops

Loops inside loops:

```javascript
// Multiplication table
for (let i = 1; i <= 5; i++) {
    let row = "";
    for (let j = 1; j <= 5; j++) {
        row += (i * j).toString().padStart(4);
    }
    console.log(row);
}
//    1   2   3   4   5
//    2   4   6   8  10
//    3   6   9  12  15
//    4   8  12  16  20
//    5  10  15  20  25

// Pattern printing
for (let i = 1; i <= 5; i++) {
    let stars = "";
    for (let j = 1; j <= i; j++) {
        stars += "* ";
    }
    console.log(stars);
}
// *
// * *
// * * *
// * * * *
// * * * * *

// 2D grid/matrix
let grid = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

for (let row = 0; row < grid.length; row++) {
    for (let col = 0; col < grid[row].length; col++) {
        console.log(`grid[${row}][${col}] = ${grid[row][col]}`);
    }
}
```

---

## Arrays

Arrays store ordered lists of values:

### Creating Arrays

```javascript
// Array literal (most common)
let fruits = ["apple", "banana", "cherry"];

// Empty array
let empty = [];

// Array constructor (rarely used)
let numbers = new Array(1, 2, 3);
let sized = new Array(5);  // Creates array with 5 empty slots

// Array.from() - convert iterable to array
let letters = Array.from("hello");  // ["h", "e", "l", "l", "o"]
let range = Array.from({ length: 5 }, (_, i) => i + 1);  // [1, 2, 3, 4, 5]

// Array.of() - create array from arguments
let arr = Array.of(1, 2, 3);  // [1, 2, 3]
let single = Array.of(5);     // [5] (different from new Array(5))

// Mixed types (allowed but not recommended)
let mixed = [1, "hello", true, null, { name: "John" }, [1, 2]];
```

### Accessing Array Elements

```javascript
let colors = ["red", "green", "blue", "yellow"];

// By index (0-based)
console.log(colors[0]);   // "red"
console.log(colors[2]);   // "blue"

// Last element
console.log(colors[colors.length - 1]);  // "yellow"

// Using at() (supports negative indices)
console.log(colors.at(0));    // "red"
console.log(colors.at(-1));   // "yellow" (last)
console.log(colors.at(-2));   // "blue" (second to last)

// Non-existent index
console.log(colors[10]);  // undefined

// Modifying elements
colors[1] = "lime";
console.log(colors);  // ["red", "lime", "blue", "yellow"]

// Adding at specific index
colors[10] = "purple";
console.log(colors.length);  // 11 (creates holes)
console.log(colors);  // ["red", "lime", "blue", "yellow", empty × 6, "purple"]
```

### Array Properties

```javascript
let arr = [1, 2, 3, 4, 5];

// Length
console.log(arr.length);  // 5

// Modify length (truncates or extends)
arr.length = 3;
console.log(arr);  // [1, 2, 3]

arr.length = 5;
console.log(arr);  // [1, 2, 3, empty × 2]

// Check if array
console.log(Array.isArray(arr));       // true
console.log(Array.isArray("hello"));   // false
console.log(Array.isArray({ 0: "a" })); // false
```

---

## Array Methods - Adding/Removing Elements

### push() and pop() - End of Array

```javascript
let stack = [];

// push() - add to end, returns new length
stack.push("a");        // returns 1
stack.push("b", "c");   // returns 3
console.log(stack);     // ["a", "b", "c"]

// pop() - remove from end, returns removed element
let last = stack.pop(); // returns "c"
console.log(stack);     // ["a", "b"]
console.log(last);      // "c"

// Pop from empty array
let empty = [];
console.log(empty.pop());  // undefined
```

### unshift() and shift() - Beginning of Array

```javascript
let queue = ["b", "c"];

// unshift() - add to beginning, returns new length
queue.unshift("a");       // returns 3
console.log(queue);       // ["a", "b", "c"]

queue.unshift("x", "y");  // returns 5
console.log(queue);       // ["x", "y", "a", "b", "c"]

// shift() - remove from beginning, returns removed element
let first = queue.shift();  // returns "x"
console.log(queue);         // ["y", "a", "b", "c"]
```

### splice() - Add/Remove at Any Position

```javascript
// splice(start, deleteCount, ...items)

let arr = ["a", "b", "c", "d", "e"];

// Remove elements
let removed = arr.splice(2, 2);  // Start at index 2, remove 2 elements
console.log(arr);      // ["a", "b", "e"]
console.log(removed);  // ["c", "d"]

// Insert elements (deleteCount = 0)
arr = ["a", "b", "c", "d"];
arr.splice(2, 0, "x", "y");  // Insert at index 2
console.log(arr);  // ["a", "b", "x", "y", "c", "d"]

// Replace elements
arr = ["a", "b", "c", "d"];
arr.splice(1, 2, "x", "y", "z");  // Remove 2, insert 3
console.log(arr);  // ["a", "x", "y", "z", "d"]

// Negative index (from end)
arr = ["a", "b", "c", "d"];
arr.splice(-2, 1);  // Remove 1 element at index -2 (c)
console.log(arr);   // ["a", "b", "d"]
```

### slice() - Extract Portion (Non-Mutating)

```javascript
// slice(start, end) - end is exclusive

let arr = ["a", "b", "c", "d", "e"];

console.log(arr.slice(1, 4));   // ["b", "c", "d"]
console.log(arr.slice(2));      // ["c", "d", "e"] (to end)
console.log(arr.slice(-2));     // ["d", "e"] (last 2)
console.log(arr.slice(-3, -1)); // ["c", "d"]
console.log(arr.slice());       // ["a", "b", "c", "d", "e"] (copy)

// Original unchanged
console.log(arr);  // ["a", "b", "c", "d", "e"]
```

### concat() - Merge Arrays (Non-Mutating)

```javascript
let arr1 = [1, 2];
let arr2 = [3, 4];
let arr3 = [5, 6];

let combined = arr1.concat(arr2);
console.log(combined);  // [1, 2, 3, 4]

let all = arr1.concat(arr2, arr3);
console.log(all);  // [1, 2, 3, 4, 5, 6]

// Can also add individual values
let mixed = arr1.concat(3, 4, arr3);
console.log(mixed);  // [1, 2, 3, 4, 5, 6]

// Spread operator (modern alternative)
let spread = [...arr1, ...arr2, ...arr3];
console.log(spread);  // [1, 2, 3, 4, 5, 6]
```

---

## Array Methods - Finding Elements

### indexOf() and lastIndexOf()

```javascript
let arr = ["a", "b", "c", "b", "d"];

// indexOf() - first occurrence
console.log(arr.indexOf("b"));     // 1
console.log(arr.indexOf("x"));     // -1 (not found)
console.log(arr.indexOf("b", 2));  // 3 (search from index 2)

// lastIndexOf() - last occurrence
console.log(arr.lastIndexOf("b")); // 3

// Check if exists
if (arr.indexOf("c") !== -1) {
    console.log("Found c!");
}
```

### includes()

```javascript
let fruits = ["apple", "banana", "cherry"];

console.log(fruits.includes("banana"));  // true
console.log(fruits.includes("grape"));   // false

// Better than indexOf for existence check
if (fruits.includes("apple")) {
    console.log("We have apples!");
}

// With start index
console.log(fruits.includes("apple", 1));  // false (starts from index 1)

// Works with NaN (indexOf doesn't)
let nums = [1, NaN, 3];
console.log(nums.includes(NaN));   // true
console.log(nums.indexOf(NaN));    // -1 (!)
```

### find() and findIndex()

```javascript
let users = [
    { id: 1, name: "John", age: 25 },
    { id: 2, name: "Jane", age: 30 },
    { id: 3, name: "Bob", age: 25 }
];

// find() - returns first matching element
let user = users.find(u => u.age === 25);
console.log(user);  // { id: 1, name: "John", age: 25 }

let notFound = users.find(u => u.age === 40);
console.log(notFound);  // undefined

// findIndex() - returns index of first match
let index = users.findIndex(u => u.name === "Jane");
console.log(index);  // 1

// findLast() and findLastIndex() (ES2023)
let lastUser = users.findLast(u => u.age === 25);
console.log(lastUser);  // { id: 3, name: "Bob", age: 25 }

let lastIndex = users.findLastIndex(u => u.age === 25);
console.log(lastIndex);  // 2
```

---

## Looping Through Arrays

### for Loop

```javascript
let fruits = ["apple", "banana", "cherry"];

// Traditional for loop
for (let i = 0; i < fruits.length; i++) {
    console.log(`${i}: ${fruits[i]}`);
}
// 0: apple
// 1: banana
// 2: cherry

// Backward iteration
for (let i = fruits.length - 1; i >= 0; i--) {
    console.log(fruits[i]);
}
// cherry, banana, apple
```

### for...of Loop

```javascript
let colors = ["red", "green", "blue"];

// Iterates over values
for (let color of colors) {
    console.log(color);
}
// red, green, blue

// With index using entries()
for (let [index, color] of colors.entries()) {
    console.log(`${index}: ${color}`);
}
// 0: red
// 1: green
// 2: blue
```

### for...in Loop (Not Recommended for Arrays)

```javascript
let arr = ["a", "b", "c"];

// Iterates over indices (as strings!)
for (let index in arr) {
    console.log(index, arr[index]);
}
// "0" "a"
// "1" "b"
// "2" "c"

// Problem: also iterates over added properties
arr.custom = "test";
for (let index in arr) {
    console.log(index);  // "0", "1", "2", "custom"
}

// Use for...of or forEach instead for arrays
```

### forEach()

```javascript
let numbers = [1, 2, 3, 4, 5];

// Basic forEach
numbers.forEach(function(num) {
    console.log(num);
});

// Arrow function
numbers.forEach(num => console.log(num));

// With index and array
numbers.forEach((num, index, array) => {
    console.log(`Index ${index}: ${num}`);
});

// Note: Cannot break out of forEach
// Use for...of or regular for if you need break
```

---

## Array Transformation Methods

### map() - Transform Each Element

```javascript
let numbers = [1, 2, 3, 4, 5];

// Double each number
let doubled = numbers.map(num => num * 2);
console.log(doubled);  // [2, 4, 6, 8, 10]

// Extract property from objects
let users = [
    { name: "John", age: 25 },
    { name: "Jane", age: 30 }
];
let names = users.map(user => user.name);
console.log(names);  // ["John", "Jane"]

// Transform with index
let indexed = numbers.map((num, i) => `${i}: ${num}`);
console.log(indexed);  // ["0: 1", "1: 2", "2: 3", "3: 4", "4: 5"]

// Chain with other methods
let result = numbers
    .map(n => n * 2)
    .map(n => n + 1);
console.log(result);  // [3, 5, 7, 9, 11]
```

### filter() - Select Elements

```javascript
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Get even numbers
let evens = numbers.filter(num => num % 2 === 0);
console.log(evens);  // [2, 4, 6, 8, 10]

// Get numbers greater than 5
let big = numbers.filter(num => num > 5);
console.log(big);  // [6, 7, 8, 9, 10]

// Filter objects
let users = [
    { name: "John", age: 25, active: true },
    { name: "Jane", age: 30, active: false },
    { name: "Bob", age: 20, active: true }
];

let activeUsers = users.filter(user => user.active);
console.log(activeUsers);
// [{ name: "John", ... }, { name: "Bob", ... }]

let adults = users.filter(user => user.age >= 25);
console.log(adults);
// [{ name: "John", ... }, { name: "Jane", ... }]

// Chaining filter and map
let activeNames = users
    .filter(user => user.active)
    .map(user => user.name);
console.log(activeNames);  // ["John", "Bob"]
```

### reduce() - Aggregate to Single Value

```javascript
// reduce(callback, initialValue)
// callback(accumulator, currentValue, index, array)

let numbers = [1, 2, 3, 4, 5];

// Sum all numbers
let sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum);  // 15

// How it works:
// acc=0, num=1 → 0+1=1
// acc=1, num=2 → 1+2=3
// acc=3, num=3 → 3+3=6
// acc=6, num=4 → 6+4=10
// acc=10, num=5 → 10+5=15

// Product
let product = numbers.reduce((acc, num) => acc * num, 1);
console.log(product);  // 120

// Find max
let max = numbers.reduce((acc, num) => num > acc ? num : acc, numbers[0]);
console.log(max);  // 5

// Count occurrences
let fruits = ["apple", "banana", "apple", "cherry", "banana", "apple"];
let count = fruits.reduce((acc, fruit) => {
    acc[fruit] = (acc[fruit] || 0) + 1;
    return acc;
}, {});
console.log(count);  // { apple: 3, banana: 2, cherry: 1 }

// Flatten nested arrays
let nested = [[1, 2], [3, 4], [5, 6]];
let flat = nested.reduce((acc, arr) => acc.concat(arr), []);
console.log(flat);  // [1, 2, 3, 4, 5, 6]

// Group by property
let people = [
    { name: "John", city: "NYC" },
    { name: "Jane", city: "LA" },
    { name: "Bob", city: "NYC" }
];

let byCity = people.reduce((acc, person) => {
    let city = person.city;
    if (!acc[city]) {
        acc[city] = [];
    }
    acc[city].push(person);
    return acc;
}, {});
console.log(byCity);
// { NYC: [{ name: "John", ... }, { name: "Bob", ... }], LA: [{ name: "Jane", ... }] }
```

### flat() and flatMap()

```javascript
// flat() - flatten nested arrays
let nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat());     // [1, 2, 3, 4, [5, 6]] (depth 1)
console.log(nested.flat(2));    // [1, 2, 3, 4, 5, 6] (depth 2)
console.log(nested.flat(Infinity));  // Flatten completely

// flatMap() - map then flatten (depth 1)
let sentences = ["Hello World", "How are you"];
let words = sentences.flatMap(s => s.split(" "));
console.log(words);  // ["Hello", "World", "How", "are", "you"]

// Equivalent to:
let words2 = sentences.map(s => s.split(" ")).flat();
```

---

## Array Testing Methods

### every() and some()

```javascript
let numbers = [2, 4, 6, 8, 10];

// every() - ALL elements must pass
let allEven = numbers.every(n => n % 2 === 0);
console.log(allEven);  // true

let allPositive = numbers.every(n => n > 0);
console.log(allPositive);  // true

// some() - AT LEAST ONE element must pass
let hasEven = numbers.some(n => n % 2 === 0);
console.log(hasEven);  // true

let hasNegative = numbers.some(n => n < 0);
console.log(hasNegative);  // false

// Practical examples
let users = [
    { name: "John", active: true },
    { name: "Jane", active: false },
    { name: "Bob", active: true }
];

let allActive = users.every(u => u.active);
console.log(allActive);  // false

let someActive = users.some(u => u.active);
console.log(someActive);  // true

// Check if form is valid
let fields = [
    { value: "John", valid: true },
    { value: "", valid: false },
    { value: "john@email.com", valid: true }
];

let formValid = fields.every(f => f.valid);
console.log(formValid);  // false
```

---

## Array Sorting and Reversing

### sort()

```javascript
// Default sort (converts to strings!)
let nums = [10, 5, 40, 25, 1000];
nums.sort();
console.log(nums);  // [10, 1000, 25, 40, 5] (!)  String comparison!

// Numeric sort
nums.sort((a, b) => a - b);  // Ascending
console.log(nums);  // [5, 10, 25, 40, 1000]

nums.sort((a, b) => b - a);  // Descending
console.log(nums);  // [1000, 40, 25, 10, 5]

// String sort
let fruits = ["banana", "Apple", "cherry"];
fruits.sort();
console.log(fruits);  // ["Apple", "banana", "cherry"]

// Case-insensitive sort
fruits.sort((a, b) => a.toLowerCase().localeCompare(b.toLowerCase()));
console.log(fruits);  // ["Apple", "banana", "cherry"]

// Sort objects by property
let users = [
    { name: "John", age: 30 },
    { name: "Jane", age: 25 },
    { name: "Bob", age: 35 }
];

// By age
users.sort((a, b) => a.age - b.age);
console.log(users);
// [{ name: "Jane", age: 25 }, { name: "John", age: 30 }, { name: "Bob", age: 35 }]

// By name
users.sort((a, b) => a.name.localeCompare(b.name));
console.log(users);
// [{ name: "Bob", ... }, { name: "Jane", ... }, { name: "John", ... }]
```

### reverse()

```javascript
let arr = [1, 2, 3, 4, 5];
arr.reverse();
console.log(arr);  // [5, 4, 3, 2, 1]

// Reverse without mutating
let original = [1, 2, 3];
let reversed = [...original].reverse();
console.log(original);  // [1, 2, 3]
console.log(reversed);  // [3, 2, 1]

// Or use toReversed() (ES2023)
let rev = original.toReversed();
console.log(rev);  // [3, 2, 1]
```

### toSorted() and toReversed() (Non-Mutating - ES2023)

```javascript
let numbers = [3, 1, 4, 1, 5];

// toSorted() - returns new sorted array
let sorted = numbers.toSorted((a, b) => a - b);
console.log(sorted);   // [1, 1, 3, 4, 5]
console.log(numbers);  // [3, 1, 4, 1, 5] (unchanged)

// toReversed() - returns new reversed array
let reversed = numbers.toReversed();
console.log(reversed);  // [5, 1, 4, 1, 3]
console.log(numbers);   // [3, 1, 4, 1, 5] (unchanged)
```

---

## Array Conversion

### join()

```javascript
let words = ["Hello", "World", "!"];

console.log(words.join(" "));   // "Hello World !"
console.log(words.join("-"));   // "Hello-World-!"
console.log(words.join(""));    // "HelloWorld!"
console.log(words.join());      // "Hello,World,!" (default comma)

// Create CSV
let data = ["John", 25, "NYC"];
let csv = data.join(",");
console.log(csv);  // "John,25,NYC"

// Create path
let path = ["users", "john", "profile"];
console.log(path.join("/"));  // "users/john/profile"
```

### Array to String

```javascript
let arr = [1, 2, 3];

console.log(arr.toString());    // "1,2,3"
console.log(arr.join(","));     // "1,2,3" (same, but more flexible)
console.log(String(arr));       // "1,2,3"
console.log(`${arr}`);          // "1,2,3"
```

---

## Spread Operator with Arrays

```javascript
let arr1 = [1, 2, 3];
let arr2 = [4, 5, 6];

// Combine arrays
let combined = [...arr1, ...arr2];
console.log(combined);  // [1, 2, 3, 4, 5, 6]

// Copy array (shallow)
let copy = [...arr1];
copy.push(4);
console.log(arr1);  // [1, 2, 3] (unchanged)
console.log(copy);  // [1, 2, 3, 4]

// Insert in middle
let middle = [1, 2, ...arr2, 7, 8];
console.log(middle);  // [1, 2, 4, 5, 6, 7, 8]

// Pass array as function arguments
function sum(a, b, c) {
    return a + b + c;
}
console.log(sum(...arr1));  // 6

// Find min/max
let numbers = [5, 2, 9, 1, 7];
console.log(Math.max(...numbers));  // 9
console.log(Math.min(...numbers));  // 1

// Convert string to array
let letters = [..."hello"];
console.log(letters);  // ["h", "e", "l", "l", "o"]
```

---

## Destructuring Arrays

```javascript
let colors = ["red", "green", "blue", "yellow"];

// Basic destructuring
let [first, second] = colors;
console.log(first);   // "red"
console.log(second);  // "green"

// Skip elements
let [, , third] = colors;
console.log(third);  // "blue"

// Rest pattern
let [head, ...rest] = colors;
console.log(head);  // "red"
console.log(rest);  // ["green", "blue", "yellow"]

// Default values
let [a, b, c, d, e = "purple"] = colors;
console.log(e);  // "purple" (no 5th element, uses default)

// Swap variables
let x = 1;
let y = 2;
[x, y] = [y, x];
console.log(x, y);  // 2, 1

// Function returning array
function getCoordinates() {
    return [10, 20];
}
let [xCoord, yCoord] = getCoordinates();
console.log(xCoord, yCoord);  // 10, 20

// Nested destructuring
let nested = [1, [2, 3], 4];
let [one, [two, three], four] = nested;
console.log(two);  // 2
```

---

## Practical Examples

### Example 1: Todo List

```javascript
let todos = [];

function addTodo(text) {
    todos.push({
        id: Date.now(),
        text: text,
        completed: false
    });
}

function removeTodo(id) {
    let index = todos.findIndex(t => t.id === id);
    if (index !== -1) {
        todos.splice(index, 1);
    }
}

function toggleTodo(id) {
    let todo = todos.find(t => t.id === id);
    if (todo) {
        todo.completed = !todo.completed;
    }
}

function getCompletedTodos() {
    return todos.filter(t => t.completed);
}

function getPendingTodos() {
    return todos.filter(t => !t.completed);
}

// Usage
addTodo("Learn JavaScript");
addTodo("Build a project");
addTodo("Get a job");

console.log(todos);
toggleTodo(todos[0].id);
console.log(getCompletedTodos());
```

### Example 2: Shopping Cart

```javascript
let cart = [];

function addToCart(product, quantity = 1) {
    let existing = cart.find(item => item.id === product.id);
    if (existing) {
        existing.quantity += quantity;
    } else {
        cart.push({ ...product, quantity });
    }
}

function removeFromCart(productId) {
    cart = cart.filter(item => item.id !== productId);
}

function updateQuantity(productId, quantity) {
    let item = cart.find(item => item.id === productId);
    if (item) {
        item.quantity = quantity;
        if (item.quantity <= 0) {
            removeFromCart(productId);
        }
    }
}

function getTotal() {
    return cart.reduce((total, item) => {
        return total + (item.price * item.quantity);
    }, 0);
}

function getItemCount() {
    return cart.reduce((count, item) => count + item.quantity, 0);
}

// Usage
addToCart({ id: 1, name: "Shirt", price: 29.99 }, 2);
addToCart({ id: 2, name: "Pants", price: 49.99 });
addToCart({ id: 1, name: "Shirt", price: 29.99 });  // Adds to existing

console.log(cart);
console.log(`Total: $${getTotal().toFixed(2)}`);
console.log(`Items: ${getItemCount()}`);
```

### Example 3: Data Processing

```javascript
let students = [
    { name: "Alice", scores: [85, 90, 78] },
    { name: "Bob", scores: [92, 88, 95] },
    { name: "Charlie", scores: [70, 65, 75] },
    { name: "Diana", scores: [88, 92, 90] }
];

// Calculate average for each student
let withAverages = students.map(student => ({
    ...student,
    average: student.scores.reduce((a, b) => a + b, 0) / student.scores.length
}));

console.log(withAverages);

// Get top performers (average >= 85)
let topPerformers = withAverages
    .filter(s => s.average >= 85)
    .map(s => s.name);

console.log("Top performers:", topPerformers);  // ["Alice", "Bob", "Diana"]

// Sort by average (highest first)
let ranked = [...withAverages].sort((a, b) => b.average - a.average);
console.log("Rankings:", ranked.map(s => s.name));  // ["Bob", "Diana", "Alice", "Charlie"]

// Class average
let classAverage = withAverages.reduce((sum, s) => sum + s.average, 0) / students.length;
console.log(`Class average: ${classAverage.toFixed(1)}`);  // 84.2
```

---

## Practice Exercises

### Exercise 1: Array Operations

```javascript
// Given an array of numbers, write functions to:
// 1. Find all even numbers
// 2. Double all numbers
// 3. Sum all numbers
// 4. Find the largest number

let numbers = [3, 7, 2, 9, 4, 1, 8, 5, 6];

// Your code here
```

**Solution:**
```javascript
let numbers = [3, 7, 2, 9, 4, 1, 8, 5, 6];

// 1. Find all even numbers
let evens = numbers.filter(n => n % 2 === 0);
console.log(evens);  // [2, 4, 8, 6]

// 2. Double all numbers
let doubled = numbers.map(n => n * 2);
console.log(doubled);  // [6, 14, 4, 18, 8, 2, 16, 10, 12]

// 3. Sum all numbers
let sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum);  // 45

// 4. Find the largest number
let max = Math.max(...numbers);
// or
let max2 = numbers.reduce((a, b) => a > b ? a : b);
console.log(max);  // 9
```

### Exercise 2: Remove Duplicates

```javascript
// Remove duplicate values from an array

let arr = [1, 2, 2, 3, 4, 4, 4, 5];

// Your code here
```

**Solution:**
```javascript
let arr = [1, 2, 2, 3, 4, 4, 4, 5];

// Method 1: Set
let unique1 = [...new Set(arr)];
console.log(unique1);  // [1, 2, 3, 4, 5]

// Method 2: filter
let unique2 = arr.filter((item, index) => arr.indexOf(item) === index);
console.log(unique2);  // [1, 2, 3, 4, 5]

// Method 3: reduce
let unique3 = arr.reduce((acc, item) => {
    if (!acc.includes(item)) {
        acc.push(item);
    }
    return acc;
}, []);
console.log(unique3);  // [1, 2, 3, 4, 5]
```

### Exercise 3: Flatten and Sum

```javascript
// Flatten this nested array and sum all numbers

let nested = [[1, 2], [3, [4, 5]], [6]];

// Your code here
```

**Solution:**
```javascript
let nested = [[1, 2], [3, [4, 5]], [6]];

let sum = nested.flat(Infinity).reduce((acc, n) => acc + n, 0);
console.log(sum);  // 21
```

---

## Key Takeaways

1. **for loop** - Best when you know iteration count
2. **while loop** - Best when iteration depends on condition
3. **for...of** - Cleanest way to iterate arrays
4. **break** exits loop, **continue** skips iteration
5. **Arrays are zero-indexed** - First element is `arr[0]`
6. **push/pop** work at end, **unshift/shift** at beginning
7. **map** transforms, **filter** selects, **reduce** aggregates
8. **sort() converts to strings** - Use comparison function for numbers
9. **Spread operator** `...` is powerful for copying and combining
10. **Destructuring** makes extracting values cleaner

---

## Self-Check Questions

1. What's the difference between for...of and for...in?
2. When would you use while vs for?
3. What does splice() return?
4. How do you properly sort numbers?
5. What's the difference between map() and forEach()?
6. How does reduce() work?
7. How do you safely copy an array?

---

**Next Lesson:** [Week 2, Day 1-2 - Functions](./week-02-day-01-02-functions.md)
