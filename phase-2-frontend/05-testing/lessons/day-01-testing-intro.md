# Day 1: Introduction to Frontend Testing

## Introduction

Testing is essential for building reliable, maintainable applications. Understanding different types of tests and when to use them helps you write code with confidence and catch bugs before they reach production.

## Learning Objectives

By the end of this lesson, you will:
- Understand why testing matters
- Know the different types of tests
- Understand the testing pyramid
- Set up a testing environment
- Write your first tests

---

## Why Test Your Code?

### The Cost of Bugs

```
Bug found during:
- Development:     $1 to fix
- Testing:         $10 to fix
- Production:      $100+ to fix
```

### Benefits of Testing

1. **Confidence** - Deploy without fear
2. **Documentation** - Tests describe expected behavior
3. **Refactoring Safety** - Change code without breaking features
4. **Better Design** - Testable code is usually better code
5. **Faster Development** - Catch bugs early, iterate quickly

### Real-World Impact

```tsx
// Without tests:
// "I think this works... let me deploy and see"
// *3 AM bug report from production*

// With tests:
// "All 247 tests pass. Deploying with confidence."
// *Peaceful sleep*
```

---

## Types of Tests

### 1. Unit Tests

Test individual functions or components in isolation:

```tsx
// Function to test
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// Unit test
test('calculateTotal sums item prices correctly', () => {
  const items = [
    { id: 1, price: 10, quantity: 2 },
    { id: 2, price: 15, quantity: 1 },
  ];

  expect(calculateTotal(items)).toBe(35);
});
```

**Characteristics:**
- Fast (milliseconds)
- Test one thing at a time
- No external dependencies
- Easy to write and maintain

### 2. Integration Tests

Test how multiple units work together:

```tsx
// Integration test - testing API + database
test('creates user and retrieves it', async () => {
  // Create user through API
  const createResponse = await request(app)
    .post('/api/users')
    .send({ name: 'John', email: 'john@example.com' });

  expect(createResponse.status).toBe(201);

  // Retrieve user through API
  const getResponse = await request(app)
    .get(`/api/users/${createResponse.body.id}`);

  expect(getResponse.body.name).toBe('John');
});
```

**Characteristics:**
- Slower than unit tests
- Test interactions between components
- May involve databases, APIs, etc.
- Catch integration bugs

### 3. End-to-End (E2E) Tests

Test complete user flows:

```tsx
// E2E test with Playwright
test('user can complete checkout', async ({ page }) => {
  // Navigate to product
  await page.goto('/products/1');

  // Add to cart
  await page.click('button:text("Add to Cart")');

  // Go to checkout
  await page.click('a:text("Checkout")');

  // Fill payment form
  await page.fill('[name="cardNumber"]', '4242424242424242');
  await page.fill('[name="expiry"]', '12/25');
  await page.fill('[name="cvc"]', '123');

  // Submit order
  await page.click('button:text("Place Order")');

  // Verify success
  await expect(page.locator('h1')).toContainText('Order Confirmed');
});
```

**Characteristics:**
- Slowest tests
- Test real user scenarios
- Use actual browser
- Catch UI and flow bugs

### 4. Component Tests

Test React components in isolation:

```tsx
// Component test
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('increments count when button clicked', () => {
  render(<Counter initialCount={0} />);

  const button = screen.getByRole('button', { name: /increment/i });
  const count = screen.getByTestId('count');

  expect(count).toHaveTextContent('0');

  fireEvent.click(button);

  expect(count).toHaveTextContent('1');
});
```

---

## The Testing Pyramid

```
        /\
       /  \
      / E2E \        <- Few, slow, expensive
     /______\
    /        \
   / Integration \    <- Some, moderate speed
  /______________\
 /                \
/    Unit Tests    \  <- Many, fast, cheap
/____________________\
```

### Recommended Distribution

| Test Type    | Coverage | Speed   | Cost    |
|-------------|----------|---------|---------|
| Unit        | 70%      | Fast    | Low     |
| Integration | 20%      | Medium  | Medium  |
| E2E         | 10%      | Slow    | High    |

### The Testing Trophy (Modern Approach)

Kent C. Dodds' alternative for React apps:

```
         Static Analysis (TypeScript, ESLint)
                    /\
                   /  \
                  / E2E \
                 /______\
                /        \
               / Integration \    <- Focus here!
              /______________\
             /                \
            /      Unit        \
           /____________________\
```

**Key insight:** Integration tests give the most confidence per effort in React apps.

---

## Testing Tools Overview

### Test Runners

| Tool     | Description                          |
|----------|--------------------------------------|
| **Jest** | Most popular, all-in-one solution   |
| **Vitest** | Fast, Vite-native, Jest-compatible |
| **Mocha** | Flexible, requires additional setup |

### React Testing Libraries

| Tool                        | Purpose                    |
|-----------------------------|----------------------------|
| **React Testing Library**   | Component testing          |
| **Enzyme** (deprecated)     | Legacy component testing   |
| **Testing Playground**      | Query helper extension     |

### E2E Testing

| Tool         | Description                    |
|--------------|--------------------------------|
| **Playwright** | Modern, fast, cross-browser  |
| **Cypress**    | Popular, great DevX          |
| **Puppeteer**  | Chrome-focused automation    |

### Mocking

| Tool         | Purpose                        |
|--------------|--------------------------------|
| **MSW**      | API mocking (recommended)      |
| **Jest mocks** | Function/module mocking       |
| **Faker**    | Generate fake test data        |

---

## Setting Up Your Testing Environment

### For a React + Vite Project

```bash
# Install testing dependencies
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

### Configure Vitest

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    css: true,
  },
});
```

### Setup File

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});
```

### Add Test Scripts

```json
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:run": "vitest run"
  }
}
```

### For Next.js Projects

```bash
# Install dependencies
npm install -D jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

```javascript
// jest.config.js
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  dir: './',
});

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jest-environment-jsdom',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
};

module.exports = createJestConfig(customJestConfig);
```

```javascript
// jest.setup.js
import '@testing-library/jest-dom';
```

---

## Writing Your First Tests

### Testing a Utility Function

```typescript
// utils/formatters.ts
export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount);
}

export function formatDate(date: Date): string {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}

export function slugify(text: string): string {
  return text
    .toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/\s+/g, '-')
    .trim();
}
```

```typescript
// utils/formatters.test.ts
import { describe, it, expect } from 'vitest';
import { formatCurrency, formatDate, slugify } from './formatters';

describe('formatCurrency', () => {
  it('formats positive numbers correctly', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
  });

  it('formats zero correctly', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });

  it('formats negative numbers correctly', () => {
    expect(formatCurrency(-50)).toBe('-$50.00');
  });

  it('rounds to two decimal places', () => {
    expect(formatCurrency(10.999)).toBe('$11.00');
  });
});

describe('formatDate', () => {
  it('formats date correctly', () => {
    const date = new Date('2024-03-15');
    expect(formatDate(date)).toBe('March 15, 2024');
  });
});

describe('slugify', () => {
  it('converts to lowercase', () => {
    expect(slugify('Hello World')).toBe('hello-world');
  });

  it('replaces spaces with hyphens', () => {
    expect(slugify('my blog post')).toBe('my-blog-post');
  });

  it('removes special characters', () => {
    expect(slugify('Hello! World?')).toBe('hello-world');
  });

  it('handles multiple spaces', () => {
    expect(slugify('hello    world')).toBe('hello-world');
  });

  it('handles empty string', () => {
    expect(slugify('')).toBe('');
  });
});
```

### Testing a React Component

```tsx
// components/Button.tsx
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary' | 'danger';
  loading?: boolean;
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary',
  loading = false,
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      className={`btn btn-${variant}`}
      aria-busy={loading}
    >
      {loading ? (
        <>
          <span className="spinner" aria-hidden="true" />
          <span className="sr-only">Loading...</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}
```

```tsx
// components/Button.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('does not call onClick when disabled', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick} disabled>Click me</Button>);

    fireEvent.click(screen.getByRole('button'));

    expect(handleClick).not.toHaveBeenCalled();
  });

  it('applies variant class correctly', () => {
    render(<Button variant="danger">Delete</Button>);
    expect(screen.getByRole('button')).toHaveClass('btn-danger');
  });

  it('shows loading state', () => {
    render(<Button loading>Submit</Button>);

    expect(screen.getByRole('button')).toHaveAttribute('aria-busy', 'true');
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('is accessible with loading state', () => {
    render(<Button loading>Submit</Button>);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
});
```

---

## Test-Driven Development (TDD)

### The TDD Cycle

```
    ┌─────────────┐
    │   RED       │  Write a failing test
    │  (Write)    │
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐
    │   GREEN     │  Write minimal code to pass
    │   (Pass)    │
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐
    │  REFACTOR   │  Improve the code
    │  (Clean)    │
    └──────┬──────┘
           │
           └────────► Repeat
```

### TDD Example

```typescript
// Step 1: RED - Write failing test
test('validates email format', () => {
  expect(isValidEmail('test@example.com')).toBe(true);
  expect(isValidEmail('invalid-email')).toBe(false);
  expect(isValidEmail('')).toBe(false);
});

// Step 2: GREEN - Minimal implementation
function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// Step 3: REFACTOR - Improve if needed
function isValidEmail(email: string): boolean {
  if (!email) return false;
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}
```

---

## Common Testing Patterns

### Arrange-Act-Assert (AAA)

```typescript
test('user can add item to cart', () => {
  // Arrange - Set up test data
  const cart = createEmptyCart();
  const item = { id: 1, name: 'Product', price: 10 };

  // Act - Perform the action
  addToCart(cart, item);

  // Assert - Verify the result
  expect(cart.items).toHaveLength(1);
  expect(cart.items[0]).toEqual(item);
});
```

### Given-When-Then (BDD Style)

```typescript
describe('Shopping Cart', () => {
  describe('given an empty cart', () => {
    describe('when adding an item', () => {
      it('then the cart should contain that item', () => {
        const cart = createEmptyCart();
        const item = { id: 1, name: 'Product', price: 10 };

        addToCart(cart, item);

        expect(cart.items).toContainEqual(item);
      });
    });
  });
});
```

### Test Fixtures

```typescript
// test/fixtures/users.ts
export const mockUsers = {
  admin: {
    id: '1',
    name: 'Admin User',
    email: 'admin@example.com',
    role: 'admin',
  },
  regular: {
    id: '2',
    name: 'Regular User',
    email: 'user@example.com',
    role: 'user',
  },
};

// In tests
import { mockUsers } from './fixtures/users';

test('admin can delete users', () => {
  const result = canDeleteUser(mockUsers.admin);
  expect(result).toBe(true);
});
```

---

## Best Practices

### 1. Test Behavior, Not Implementation

```typescript
// ❌ Bad - Testing implementation
test('sets isLoading to true', () => {
  const { result } = renderHook(() => useData());
  expect(result.current.state.isLoading).toBe(true);
});

// ✅ Good - Testing behavior
test('shows loading indicator while fetching', () => {
  render(<DataDisplay />);
  expect(screen.getByRole('progressbar')).toBeInTheDocument();
});
```

### 2. Use Descriptive Test Names

```typescript
// ❌ Bad
test('button test', () => { ... });

// ✅ Good
test('disables submit button when form is invalid', () => { ... });
```

### 3. One Assertion Per Test (Generally)

```typescript
// ❌ Bad - Multiple unrelated assertions
test('form validation', () => {
  expect(validateEmail('test@test.com')).toBe(true);
  expect(validatePhone('123-456-7890')).toBe(true);
  expect(validateAge(25)).toBe(true);
});

// ✅ Good - Focused tests
test('validates correct email format', () => {
  expect(validateEmail('test@test.com')).toBe(true);
});

test('validates correct phone format', () => {
  expect(validatePhone('123-456-7890')).toBe(true);
});
```

### 4. Keep Tests Independent

```typescript
// ❌ Bad - Tests depend on each other
let counter = 0;

test('increments counter', () => {
  counter++;
  expect(counter).toBe(1);
});

test('counter is now 1', () => {
  expect(counter).toBe(1); // Depends on previous test!
});

// ✅ Good - Independent tests
test('increments counter', () => {
  let counter = 0;
  counter++;
  expect(counter).toBe(1);
});
```

### 5. Use Setup and Teardown Wisely

```typescript
describe('UserService', () => {
  let db: Database;

  beforeAll(async () => {
    // Expensive setup - do once
    db = await connectToTestDatabase();
  });

  afterAll(async () => {
    // Cleanup
    await db.disconnect();
  });

  beforeEach(async () => {
    // Reset state before each test
    await db.clear();
  });

  test('creates user', async () => {
    const user = await UserService.create(db, { name: 'Test' });
    expect(user.id).toBeDefined();
  });
});
```

---

## Exercise: Write Tests for a Todo Module

Create tests for this todo module:

```typescript
// todo.ts
export interface Todo {
  id: string;
  text: string;
  completed: boolean;
  createdAt: Date;
}

export function createTodo(text: string): Todo {
  return {
    id: crypto.randomUUID(),
    text,
    completed: false,
    createdAt: new Date(),
  };
}

export function toggleTodo(todo: Todo): Todo {
  return { ...todo, completed: !todo.completed };
}

export function filterTodos(
  todos: Todo[],
  filter: 'all' | 'active' | 'completed'
): Todo[] {
  switch (filter) {
    case 'active':
      return todos.filter(t => !t.completed);
    case 'completed':
      return todos.filter(t => t.completed);
    default:
      return todos;
  }
}

export function getCompletedCount(todos: Todo[]): number {
  return todos.filter(t => t.completed).length;
}
```

---

## Solution

```typescript
// todo.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { createTodo, toggleTodo, filterTodos, getCompletedCount, Todo } from './todo';

// Mock crypto.randomUUID for predictable IDs
vi.stubGlobal('crypto', {
  randomUUID: () => 'test-uuid-123',
});

describe('createTodo', () => {
  beforeEach(() => {
    vi.useFakeTimers();
    vi.setSystemTime(new Date('2024-01-01'));
  });

  it('creates a todo with the given text', () => {
    const todo = createTodo('Buy groceries');
    expect(todo.text).toBe('Buy groceries');
  });

  it('creates a todo with completed set to false', () => {
    const todo = createTodo('Buy groceries');
    expect(todo.completed).toBe(false);
  });

  it('creates a todo with a unique id', () => {
    const todo = createTodo('Buy groceries');
    expect(todo.id).toBe('test-uuid-123');
  });

  it('creates a todo with current timestamp', () => {
    const todo = createTodo('Buy groceries');
    expect(todo.createdAt).toEqual(new Date('2024-01-01'));
  });
});

describe('toggleTodo', () => {
  it('toggles completed from false to true', () => {
    const todo: Todo = {
      id: '1',
      text: 'Test',
      completed: false,
      createdAt: new Date(),
    };

    const toggled = toggleTodo(todo);

    expect(toggled.completed).toBe(true);
  });

  it('toggles completed from true to false', () => {
    const todo: Todo = {
      id: '1',
      text: 'Test',
      completed: true,
      createdAt: new Date(),
    };

    const toggled = toggleTodo(todo);

    expect(toggled.completed).toBe(false);
  });

  it('returns a new object (immutable)', () => {
    const todo: Todo = {
      id: '1',
      text: 'Test',
      completed: false,
      createdAt: new Date(),
    };

    const toggled = toggleTodo(todo);

    expect(toggled).not.toBe(todo);
  });

  it('preserves other properties', () => {
    const createdAt = new Date();
    const todo: Todo = {
      id: '1',
      text: 'Test',
      completed: false,
      createdAt,
    };

    const toggled = toggleTodo(todo);

    expect(toggled.id).toBe('1');
    expect(toggled.text).toBe('Test');
    expect(toggled.createdAt).toBe(createdAt);
  });
});

describe('filterTodos', () => {
  const todos: Todo[] = [
    { id: '1', text: 'Active 1', completed: false, createdAt: new Date() },
    { id: '2', text: 'Completed 1', completed: true, createdAt: new Date() },
    { id: '3', text: 'Active 2', completed: false, createdAt: new Date() },
  ];

  it('returns all todos when filter is "all"', () => {
    const result = filterTodos(todos, 'all');
    expect(result).toHaveLength(3);
  });

  it('returns only active todos when filter is "active"', () => {
    const result = filterTodos(todos, 'active');

    expect(result).toHaveLength(2);
    expect(result.every(t => !t.completed)).toBe(true);
  });

  it('returns only completed todos when filter is "completed"', () => {
    const result = filterTodos(todos, 'completed');

    expect(result).toHaveLength(1);
    expect(result[0].text).toBe('Completed 1');
  });

  it('returns empty array when no todos match filter', () => {
    const activeTodos: Todo[] = [
      { id: '1', text: 'Active', completed: false, createdAt: new Date() },
    ];

    const result = filterTodos(activeTodos, 'completed');

    expect(result).toHaveLength(0);
  });
});

describe('getCompletedCount', () => {
  it('returns 0 for empty array', () => {
    expect(getCompletedCount([])).toBe(0);
  });

  it('returns 0 when no todos are completed', () => {
    const todos: Todo[] = [
      { id: '1', text: 'Test', completed: false, createdAt: new Date() },
    ];

    expect(getCompletedCount(todos)).toBe(0);
  });

  it('counts completed todos correctly', () => {
    const todos: Todo[] = [
      { id: '1', text: 'Test 1', completed: true, createdAt: new Date() },
      { id: '2', text: 'Test 2', completed: false, createdAt: new Date() },
      { id: '3', text: 'Test 3', completed: true, createdAt: new Date() },
    ];

    expect(getCompletedCount(todos)).toBe(2);
  });
});
```

---

## Key Takeaways

1. **Tests provide confidence** - Deploy without fear of breaking things
2. **Different tests for different purposes** - Unit, integration, and E2E each have their place
3. **Focus on integration tests** - They give the best confidence per effort for React apps
4. **Test behavior, not implementation** - Tests should survive refactoring
5. **Keep tests simple and focused** - One concept per test
6. **Use good naming** - Tests are documentation
7. **Set up proper tooling** - Vitest/Jest + React Testing Library is the standard

---

## What's Next?

Tomorrow we'll dive deep into **Jest fundamentals** - mastering assertions, matchers, mocking, and everything you need to write comprehensive tests!
