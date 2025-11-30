# Day 2: Jest Fundamentals

## Introduction

Jest is the most popular JavaScript testing framework. It's fast, feature-rich, and requires minimal configuration. Whether you're using Jest directly or Vitest (which has Jest-compatible APIs), mastering these fundamentals is essential.

## Learning Objectives

By the end of this lesson, you will:
- Master Jest matchers and assertions
- Understand test organization with describe/it
- Mock functions, modules, and timers
- Handle async code in tests
- Use setup and teardown hooks

---

## Core Jest Concepts

### Test Structure

```typescript
// Basic test
test('adds 1 + 2 to equal 3', () => {
  expect(1 + 2).toBe(3);
});

// Using 'it' (alias for 'test')
it('adds 1 + 2 to equal 3', () => {
  expect(1 + 2).toBe(3);
});

// Grouping tests with describe
describe('Math operations', () => {
  describe('addition', () => {
    it('adds positive numbers', () => {
      expect(1 + 2).toBe(3);
    });

    it('adds negative numbers', () => {
      expect(-1 + -2).toBe(-3);
    });
  });

  describe('multiplication', () => {
    it('multiplies positive numbers', () => {
      expect(2 * 3).toBe(6);
    });
  });
});
```

### The `expect` API

```typescript
// Basic syntax
expect(value).matcher(expected);

// Examples
expect(result).toBe(5);           // Strict equality (===)
expect(result).toEqual({ a: 1 }); // Deep equality
expect(result).toBeTruthy();      // Boolean check
```

---

## Jest Matchers

### Equality Matchers

```typescript
describe('Equality Matchers', () => {
  // toBe - strict equality (===)
  test('toBe uses strict equality', () => {
    expect(2 + 2).toBe(4);
    expect('hello').toBe('hello');

    // Fails for objects/arrays (different references)
    // expect({ a: 1 }).toBe({ a: 1 }); // FAILS!
  });

  // toEqual - deep equality
  test('toEqual for objects and arrays', () => {
    const obj = { a: 1, b: { c: 2 } };
    expect(obj).toEqual({ a: 1, b: { c: 2 } });

    const arr = [1, 2, [3, 4]];
    expect(arr).toEqual([1, 2, [3, 4]]);
  });

  // toStrictEqual - deep equality + undefined properties
  test('toStrictEqual is stricter', () => {
    const obj = { a: 1, b: undefined };

    expect(obj).toEqual({ a: 1 });              // Passes
    // expect(obj).toStrictEqual({ a: 1 });     // Fails!
    expect(obj).toStrictEqual({ a: 1, b: undefined }); // Passes
  });
});
```

### Truthiness Matchers

```typescript
describe('Truthiness Matchers', () => {
  test('null', () => {
    const n = null;
    expect(n).toBeNull();
    expect(n).toBeDefined();
    expect(n).not.toBeUndefined();
    expect(n).not.toBeTruthy();
    expect(n).toBeFalsy();
  });

  test('zero', () => {
    const z = 0;
    expect(z).not.toBeNull();
    expect(z).toBeDefined();
    expect(z).not.toBeUndefined();
    expect(z).not.toBeTruthy();
    expect(z).toBeFalsy();
  });

  test('empty string', () => {
    const s = '';
    expect(s).toBeFalsy();
    expect(s).toBeDefined();
  });
});
```

### Number Matchers

```typescript
describe('Number Matchers', () => {
  test('comparison matchers', () => {
    const value = 10;

    expect(value).toBeGreaterThan(5);
    expect(value).toBeGreaterThanOrEqual(10);
    expect(value).toBeLessThan(15);
    expect(value).toBeLessThanOrEqual(10);
  });

  test('floating point equality', () => {
    const result = 0.1 + 0.2;

    // expect(result).toBe(0.3);      // FAILS due to floating point
    expect(result).toBeCloseTo(0.3);  // Passes!
    expect(result).toBeCloseTo(0.3, 5); // 5 decimal places
  });

  test('NaN and Infinity', () => {
    expect(NaN).toBeNaN();
    expect(1 / 0).toBe(Infinity);
  });
});
```

### String Matchers

```typescript
describe('String Matchers', () => {
  test('toMatch with regex', () => {
    expect('hello world').toMatch(/world/);
    expect('hello world').toMatch(/^hello/);
    expect('test@example.com').toMatch(/\S+@\S+\.\S+/);
  });

  test('toContain for substrings', () => {
    expect('hello world').toContain('world');
    expect('hello world').not.toContain('foo');
  });

  test('string length', () => {
    expect('hello').toHaveLength(5);
  });
});
```

### Array Matchers

```typescript
describe('Array Matchers', () => {
  const fruits = ['apple', 'banana', 'cherry'];

  test('toContain for arrays', () => {
    expect(fruits).toContain('banana');
    expect(fruits).not.toContain('grape');
  });

  test('toHaveLength', () => {
    expect(fruits).toHaveLength(3);
  });

  test('toContainEqual for objects', () => {
    const users = [
      { name: 'Alice', age: 30 },
      { name: 'Bob', age: 25 },
    ];

    expect(users).toContainEqual({ name: 'Bob', age: 25 });
  });

  test('array subset', () => {
    expect(fruits).toEqual(expect.arrayContaining(['apple', 'cherry']));
  });
});
```

### Object Matchers

```typescript
describe('Object Matchers', () => {
  const user = {
    name: 'John',
    email: 'john@example.com',
    age: 30,
    address: {
      city: 'New York',
      country: 'USA',
    },
  };

  test('toHaveProperty', () => {
    expect(user).toHaveProperty('name');
    expect(user).toHaveProperty('name', 'John');
    expect(user).toHaveProperty('address.city', 'New York');
  });

  test('toMatchObject for partial matching', () => {
    expect(user).toMatchObject({
      name: 'John',
      address: { city: 'New York' },
    });
  });

  test('object subset with expect.objectContaining', () => {
    expect(user).toEqual(
      expect.objectContaining({
        name: 'John',
        email: expect.any(String),
      })
    );
  });
});
```

### Exception Matchers

```typescript
describe('Exception Matchers', () => {
  function divide(a: number, b: number): number {
    if (b === 0) throw new Error('Division by zero');
    return a / b;
  }

  test('toThrow', () => {
    expect(() => divide(10, 0)).toThrow();
    expect(() => divide(10, 0)).toThrow('Division by zero');
    expect(() => divide(10, 0)).toThrow(/zero/);
    expect(() => divide(10, 0)).toThrow(Error);
  });

  test('toThrowError', () => {
    class CustomError extends Error {
      constructor(message: string) {
        super(message);
        this.name = 'CustomError';
      }
    }

    function throwCustom() {
      throw new CustomError('Something went wrong');
    }

    expect(throwCustom).toThrow(CustomError);
  });
});
```

### Using `.not`

```typescript
describe('Negating matchers with .not', () => {
  test('negation examples', () => {
    expect(5).not.toBe(10);
    expect('hello').not.toContain('x');
    expect([1, 2, 3]).not.toContain(4);
    expect({ a: 1 }).not.toHaveProperty('b');
    expect(() => {}).not.toThrow();
  });
});
```

---

## Mock Functions

### Creating Mock Functions

```typescript
describe('Mock Functions', () => {
  test('basic mock function', () => {
    const mockFn = jest.fn();

    mockFn('hello');
    mockFn('world');

    expect(mockFn).toHaveBeenCalled();
    expect(mockFn).toHaveBeenCalledTimes(2);
    expect(mockFn).toHaveBeenCalledWith('hello');
    expect(mockFn).toHaveBeenLastCalledWith('world');
  });

  test('mock with return value', () => {
    const mockFn = jest.fn().mockReturnValue(42);

    expect(mockFn()).toBe(42);
    expect(mockFn()).toBe(42);
  });

  test('mock with different return values', () => {
    const mockFn = jest.fn()
      .mockReturnValueOnce(1)
      .mockReturnValueOnce(2)
      .mockReturnValue(0);

    expect(mockFn()).toBe(1);
    expect(mockFn()).toBe(2);
    expect(mockFn()).toBe(0);
    expect(mockFn()).toBe(0);
  });

  test('mock implementation', () => {
    const mockFn = jest.fn((x: number) => x * 2);

    expect(mockFn(5)).toBe(10);
    expect(mockFn).toHaveBeenCalledWith(5);
  });
});
```

### Mock Function Properties

```typescript
describe('Mock Function Properties', () => {
  test('accessing call information', () => {
    const mockFn = jest.fn();

    mockFn('first', 'call');
    mockFn('second', 'call');

    // All calls
    expect(mockFn.mock.calls).toEqual([
      ['first', 'call'],
      ['second', 'call'],
    ]);

    // First call arguments
    expect(mockFn.mock.calls[0]).toEqual(['first', 'call']);

    // Number of calls
    expect(mockFn.mock.calls.length).toBe(2);
  });

  test('accessing return values', () => {
    const mockFn = jest.fn()
      .mockReturnValueOnce('first')
      .mockReturnValueOnce('second');

    mockFn();
    mockFn();

    expect(mockFn.mock.results).toEqual([
      { type: 'return', value: 'first' },
      { type: 'return', value: 'second' },
    ]);
  });

  test('resetting mocks', () => {
    const mockFn = jest.fn().mockReturnValue(42);

    mockFn();
    mockFn();

    // Clear call history but keep implementation
    mockFn.mockClear();
    expect(mockFn.mock.calls.length).toBe(0);
    expect(mockFn()).toBe(42);

    // Reset everything
    mockFn.mockReset();
    expect(mockFn()).toBeUndefined();
  });
});
```

### Mocking Modules

```typescript
// api.ts
export async function fetchUser(id: string) {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// userService.ts
import { fetchUser } from './api';

export async function getUserDisplayName(id: string) {
  const user = await fetchUser(id);
  return `${user.firstName} ${user.lastName}`;
}
```

```typescript
// userService.test.ts
import { getUserDisplayName } from './userService';
import { fetchUser } from './api';

// Mock the entire module
jest.mock('./api');

// TypeScript: Cast to mock
const mockedFetchUser = fetchUser as jest.MockedFunction<typeof fetchUser>;

describe('getUserDisplayName', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('returns formatted display name', async () => {
    mockedFetchUser.mockResolvedValue({
      id: '1',
      firstName: 'John',
      lastName: 'Doe',
    });

    const result = await getUserDisplayName('1');

    expect(result).toBe('John Doe');
    expect(mockedFetchUser).toHaveBeenCalledWith('1');
  });

  test('handles API error', async () => {
    mockedFetchUser.mockRejectedValue(new Error('Network error'));

    await expect(getUserDisplayName('1')).rejects.toThrow('Network error');
  });
});
```

### Partial Module Mocking

```typescript
// Only mock specific exports
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'), // Keep actual implementations
  formatDate: jest.fn().mockReturnValue('2024-01-01'), // Mock this one
}));
```

### Manual Mocks

```typescript
// __mocks__/axios.ts
export default {
  get: jest.fn(() => Promise.resolve({ data: {} })),
  post: jest.fn(() => Promise.resolve({ data: {} })),
  create: jest.fn(() => ({
    get: jest.fn(() => Promise.resolve({ data: {} })),
    post: jest.fn(() => Promise.resolve({ data: {} })),
  })),
};

// In test file
jest.mock('axios');
```

---

## Async Testing

### Testing Promises

```typescript
async function fetchData(): Promise<string> {
  return new Promise(resolve => {
    setTimeout(() => resolve('data'), 100);
  });
}

async function fetchError(): Promise<string> {
  return new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Network error')), 100);
  });
}

describe('Async Testing', () => {
  // Method 1: async/await
  test('async/await', async () => {
    const data = await fetchData();
    expect(data).toBe('data');
  });

  // Method 2: return promise
  test('returning promise', () => {
    return fetchData().then(data => {
      expect(data).toBe('data');
    });
  });

  // Method 3: resolves/rejects matchers
  test('using resolves', async () => {
    await expect(fetchData()).resolves.toBe('data');
  });

  test('using rejects', async () => {
    await expect(fetchError()).rejects.toThrow('Network error');
  });
});
```

### Testing Callbacks

```typescript
function fetchDataCallback(callback: (data: string) => void) {
  setTimeout(() => callback('data'), 100);
}

describe('Callback Testing', () => {
  test('done callback', (done) => {
    fetchDataCallback((data) => {
      try {
        expect(data).toBe('data');
        done();
      } catch (error) {
        done(error);
      }
    });
  });
});
```

### Mocking Async Functions

```typescript
describe('Mocking Async', () => {
  test('mockResolvedValue', async () => {
    const mockFetch = jest.fn().mockResolvedValue({ id: 1, name: 'John' });

    const result = await mockFetch();

    expect(result).toEqual({ id: 1, name: 'John' });
  });

  test('mockRejectedValue', async () => {
    const mockFetch = jest.fn().mockRejectedValue(new Error('Failed'));

    await expect(mockFetch()).rejects.toThrow('Failed');
  });

  test('mockImplementation with async', async () => {
    const mockFetch = jest.fn().mockImplementation(async (id: string) => {
      await new Promise(r => setTimeout(r, 10));
      return { id, name: 'User' };
    });

    const result = await mockFetch('123');

    expect(result).toEqual({ id: '123', name: 'User' });
  });
});
```

---

## Mocking Timers

```typescript
function delayedGreeting(callback: (msg: string) => void) {
  setTimeout(() => callback('Hello!'), 1000);
}

function repeatingTask(callback: () => void) {
  setInterval(callback, 500);
}

describe('Timer Mocking', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  test('setTimeout with fake timers', () => {
    const callback = jest.fn();

    delayedGreeting(callback);

    expect(callback).not.toHaveBeenCalled();

    jest.advanceTimersByTime(1000);

    expect(callback).toHaveBeenCalledWith('Hello!');
  });

  test('setInterval with fake timers', () => {
    const callback = jest.fn();

    repeatingTask(callback);

    jest.advanceTimersByTime(1500);

    expect(callback).toHaveBeenCalledTimes(3);
  });

  test('runAllTimers', () => {
    const callback = jest.fn();

    delayedGreeting(callback);

    jest.runAllTimers();

    expect(callback).toHaveBeenCalled();
  });

  test('runOnlyPendingTimers', () => {
    const callback = jest.fn();
    let count = 0;

    function recursive() {
      count++;
      if (count < 3) {
        setTimeout(recursive, 100);
      }
    }

    setTimeout(recursive, 100);

    jest.runOnlyPendingTimers();

    expect(count).toBe(1);
  });
});
```

---

## Setup and Teardown

```typescript
describe('Setup and Teardown', () => {
  let database: Database;
  let connection: Connection;

  // Run once before all tests in this describe block
  beforeAll(async () => {
    database = await Database.connect();
  });

  // Run once after all tests in this describe block
  afterAll(async () => {
    await database.disconnect();
  });

  // Run before each test
  beforeEach(async () => {
    connection = await database.getConnection();
    await connection.beginTransaction();
  });

  // Run after each test
  afterEach(async () => {
    await connection.rollback();
    connection.release();
  });

  test('creates a user', async () => {
    const user = await connection.query('INSERT INTO users...');
    expect(user).toBeDefined();
  });

  // Nested describe blocks have their own setup/teardown
  describe('with admin user', () => {
    let adminUser: User;

    beforeEach(async () => {
      adminUser = await createAdmin(connection);
    });

    test('admin can delete users', async () => {
      expect(await canDelete(adminUser)).toBe(true);
    });
  });
});
```

### Execution Order

```typescript
describe('outer', () => {
  beforeAll(() => console.log('1 - beforeAll outer'));
  afterAll(() => console.log('6 - afterAll outer'));
  beforeEach(() => console.log('2 - beforeEach outer'));
  afterEach(() => console.log('4 - afterEach outer'));

  test('test', () => console.log('3 - test'));

  describe('inner', () => {
    beforeAll(() => console.log('1.5 - beforeAll inner'));
    beforeEach(() => console.log('2.5 - beforeEach inner'));
    afterEach(() => console.log('3.5 - afterEach inner'));

    test('inner test', () => console.log('3 - inner test'));
  });
});

// Output:
// 1 - beforeAll outer
// 2 - beforeEach outer
// 3 - test
// 4 - afterEach outer
// 1.5 - beforeAll inner
// 2 - beforeEach outer
// 2.5 - beforeEach inner
// 3 - inner test
// 3.5 - afterEach inner
// 4 - afterEach outer
// 6 - afterAll outer
```

---

## Skipping and Focusing Tests

```typescript
describe('Test Control', () => {
  // Skip this test
  test.skip('this test is skipped', () => {
    expect(true).toBe(false);
  });

  // Only run this test (useful for debugging)
  test.only('only this test runs', () => {
    expect(true).toBe(true);
  });

  // Skip entire describe block
  describe.skip('skipped suite', () => {
    test('this will not run', () => {});
  });

  // Only run this describe block
  describe.only('focused suite', () => {
    test('this will run', () => {});
  });

  // Placeholder for future tests
  test.todo('implement this feature');
});
```

---

## Snapshot Testing

```typescript
describe('Snapshot Testing', () => {
  test('object snapshot', () => {
    const user = {
      name: 'John',
      email: 'john@example.com',
      createdAt: new Date('2024-01-01'),
    };

    expect(user).toMatchSnapshot();
  });

  test('inline snapshot', () => {
    const result = formatCurrency(1234.56);

    expect(result).toMatchInlineSnapshot(`"$1,234.56"`);
  });

  test('component snapshot', () => {
    const tree = renderer.create(<Button>Click me</Button>).toJSON();

    expect(tree).toMatchSnapshot();
  });
});

// Update snapshots: jest --updateSnapshot or jest -u
```

---

## Test Coverage

### Configuration

```javascript
// jest.config.js
module.exports = {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  coverageReporters: ['text', 'lcov', 'html'],
};
```

### Running Coverage

```bash
# Run with coverage
jest --coverage

# Watch mode with coverage
jest --watchAll --coverage
```

### Understanding Coverage Report

```
----------|---------|----------|---------|---------|
File      | % Stmts | % Branch | % Funcs | % Lines |
----------|---------|----------|---------|---------|
All files |   85.71 |    83.33 |      80 |   85.71 |
 utils.ts |   85.71 |    83.33 |      80 |   85.71 |
----------|---------|----------|---------|---------|
```

- **Statements**: % of statements executed
- **Branches**: % of if/else branches covered
- **Functions**: % of functions called
- **Lines**: % of lines executed

---

## Exercise: Test a Shopping Cart Module

Write comprehensive tests for this shopping cart:

```typescript
// cart.ts
export interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

export interface Cart {
  items: CartItem[];
  discount: number;
}

export function createCart(): Cart {
  return { items: [], discount: 0 };
}

export function addItem(cart: Cart, item: Omit<CartItem, 'quantity'>): Cart {
  const existingItem = cart.items.find(i => i.id === item.id);

  if (existingItem) {
    return {
      ...cart,
      items: cart.items.map(i =>
        i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
      ),
    };
  }

  return {
    ...cart,
    items: [...cart.items, { ...item, quantity: 1 }],
  };
}

export function removeItem(cart: Cart, itemId: string): Cart {
  return {
    ...cart,
    items: cart.items.filter(i => i.id !== itemId),
  };
}

export function updateQuantity(cart: Cart, itemId: string, quantity: number): Cart {
  if (quantity <= 0) {
    return removeItem(cart, itemId);
  }

  return {
    ...cart,
    items: cart.items.map(i =>
      i.id === itemId ? { ...i, quantity } : i
    ),
  };
}

export function applyDiscount(cart: Cart, percentage: number): Cart {
  if (percentage < 0 || percentage > 100) {
    throw new Error('Discount must be between 0 and 100');
  }

  return { ...cart, discount: percentage };
}

export function calculateTotal(cart: Cart): number {
  const subtotal = cart.items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );

  const discountAmount = subtotal * (cart.discount / 100);
  return Math.round((subtotal - discountAmount) * 100) / 100;
}

export function getItemCount(cart: Cart): number {
  return cart.items.reduce((sum, item) => sum + item.quantity, 0);
}
```

---

## Solution

```typescript
// cart.test.ts
import { describe, it, expect } from 'vitest';
import {
  createCart,
  addItem,
  removeItem,
  updateQuantity,
  applyDiscount,
  calculateTotal,
  getItemCount,
  Cart,
} from './cart';

describe('Shopping Cart', () => {
  const sampleItem = { id: '1', name: 'Product 1', price: 10 };
  const anotherItem = { id: '2', name: 'Product 2', price: 20 };

  describe('createCart', () => {
    it('creates an empty cart', () => {
      const cart = createCart();

      expect(cart.items).toEqual([]);
      expect(cart.discount).toBe(0);
    });
  });

  describe('addItem', () => {
    it('adds new item to empty cart', () => {
      const cart = createCart();
      const newCart = addItem(cart, sampleItem);

      expect(newCart.items).toHaveLength(1);
      expect(newCart.items[0]).toEqual({ ...sampleItem, quantity: 1 });
    });

    it('increments quantity for existing item', () => {
      let cart = createCart();
      cart = addItem(cart, sampleItem);
      cart = addItem(cart, sampleItem);

      expect(cart.items).toHaveLength(1);
      expect(cart.items[0].quantity).toBe(2);
    });

    it('adds different items separately', () => {
      let cart = createCart();
      cart = addItem(cart, sampleItem);
      cart = addItem(cart, anotherItem);

      expect(cart.items).toHaveLength(2);
    });

    it('returns new cart object (immutable)', () => {
      const cart = createCart();
      const newCart = addItem(cart, sampleItem);

      expect(newCart).not.toBe(cart);
    });
  });

  describe('removeItem', () => {
    it('removes item from cart', () => {
      let cart = createCart();
      cart = addItem(cart, sampleItem);
      cart = addItem(cart, anotherItem);
      cart = removeItem(cart, '1');

      expect(cart.items).toHaveLength(1);
      expect(cart.items[0].id).toBe('2');
    });

    it('handles removing non-existent item', () => {
      let cart = createCart();
      cart = addItem(cart, sampleItem);
      cart = removeItem(cart, 'non-existent');

      expect(cart.items).toHaveLength(1);
    });

    it('handles empty cart', () => {
      const cart = createCart();
      const newCart = removeItem(cart, '1');

      expect(newCart.items).toHaveLength(0);
    });
  });

  describe('updateQuantity', () => {
    it('updates item quantity', () => {
      let cart = createCart();
      cart = addItem(cart, sampleItem);
      cart = updateQuantity(cart, '1', 5);

      expect(cart.items[0].quantity).toBe(5);
    });

    it('removes item when quantity is 0', () => {
      let cart = createCart();
      cart = addItem(cart, sampleItem);
      cart = updateQuantity(cart, '1', 0);

      expect(cart.items).toHaveLength(0);
    });

    it('removes item when quantity is negative', () => {
      let cart = createCart();
      cart = addItem(cart, sampleItem);
      cart = updateQuantity(cart, '1', -1);

      expect(cart.items).toHaveLength(0);
    });
  });

  describe('applyDiscount', () => {
    it('applies valid discount', () => {
      let cart = createCart();
      cart = applyDiscount(cart, 10);

      expect(cart.discount).toBe(10);
    });

    it('throws for negative discount', () => {
      const cart = createCart();

      expect(() => applyDiscount(cart, -10)).toThrow(
        'Discount must be between 0 and 100'
      );
    });

    it('throws for discount over 100', () => {
      const cart = createCart();

      expect(() => applyDiscount(cart, 101)).toThrow(
        'Discount must be between 0 and 100'
      );
    });

    it('allows 0% discount', () => {
      const cart = applyDiscount(createCart(), 0);
      expect(cart.discount).toBe(0);
    });

    it('allows 100% discount', () => {
      const cart = applyDiscount(createCart(), 100);
      expect(cart.discount).toBe(100);
    });
  });

  describe('calculateTotal', () => {
    it('returns 0 for empty cart', () => {
      const cart = createCart();
      expect(calculateTotal(cart)).toBe(0);
    });

    it('calculates total without discount', () => {
      let cart = createCart();
      cart = addItem(cart, { id: '1', name: 'A', price: 10 });
      cart = addItem(cart, { id: '1', name: 'A', price: 10 }); // quantity: 2
      cart = addItem(cart, { id: '2', name: 'B', price: 15 });

      expect(calculateTotal(cart)).toBe(35); // 10*2 + 15
    });

    it('applies discount correctly', () => {
      let cart = createCart();
      cart = addItem(cart, { id: '1', name: 'A', price: 100 });
      cart = applyDiscount(cart, 10);

      expect(calculateTotal(cart)).toBe(90); // 100 - 10%
    });

    it('handles floating point correctly', () => {
      let cart = createCart();
      cart = addItem(cart, { id: '1', name: 'A', price: 10.33 });
      cart = addItem(cart, { id: '2', name: 'B', price: 20.67 });

      expect(calculateTotal(cart)).toBe(31);
    });

    it('returns 0 with 100% discount', () => {
      let cart = createCart();
      cart = addItem(cart, { id: '1', name: 'A', price: 100 });
      cart = applyDiscount(cart, 100);

      expect(calculateTotal(cart)).toBe(0);
    });
  });

  describe('getItemCount', () => {
    it('returns 0 for empty cart', () => {
      const cart = createCart();
      expect(getItemCount(cart)).toBe(0);
    });

    it('counts total items including quantities', () => {
      let cart = createCart();
      cart = addItem(cart, sampleItem);
      cart = addItem(cart, sampleItem);
      cart = addItem(cart, anotherItem);

      expect(getItemCount(cart)).toBe(3); // 2 + 1
    });
  });

  describe('integration scenarios', () => {
    it('handles full shopping workflow', () => {
      let cart = createCart();

      // Add items
      cart = addItem(cart, { id: '1', name: 'Shirt', price: 29.99 });
      cart = addItem(cart, { id: '2', name: 'Pants', price: 49.99 });
      cart = addItem(cart, { id: '1', name: 'Shirt', price: 29.99 }); // Second shirt

      expect(getItemCount(cart)).toBe(3);

      // Update quantity
      cart = updateQuantity(cart, '2', 2); // Two pants

      expect(getItemCount(cart)).toBe(4);

      // Apply discount
      cart = applyDiscount(cart, 20);

      // Calculate: (29.99*2 + 49.99*2) = 159.96
      // After 20% discount: 127.97
      expect(calculateTotal(cart)).toBeCloseTo(127.97, 2);

      // Remove item
      cart = removeItem(cart, '1');

      expect(getItemCount(cart)).toBe(2);
      // (49.99*2) * 0.8 = 79.98
      expect(calculateTotal(cart)).toBeCloseTo(79.99, 2);
    });
  });
});
```

---

## Key Takeaways

1. **Use the right matcher** - `toBe` for primitives, `toEqual` for objects/arrays
2. **Mock functions track calls** - Access via `.mock.calls` and `.mock.results`
3. **Mock modules for isolation** - Use `jest.mock()` to replace dependencies
4. **Handle async properly** - Use async/await or return promises
5. **Fake timers for time-based code** - Use `jest.useFakeTimers()`
6. **Setup/teardown for shared state** - `beforeEach`, `afterEach`, etc.
7. **Coverage is a guide, not a goal** - 100% coverage doesn't mean bug-free

---

## What's Next?

Tomorrow we'll learn **React Testing Library** - the standard way to test React components by focusing on user behavior rather than implementation details!
