# Day 2: Unit Testing - Testing Functions in Isolation

## Introduction

Unit tests verify that individual functions and methods work correctly in isolation. They're fast, focused, and form the foundation of your test suite. Today, you'll learn how to write effective unit tests for your backend services, utilities, and business logic.

## Learning Objectives

By the end of this lesson, you will be able to:
- Write focused unit tests for pure functions
- Test service layer methods
- Use Jest's assertion and mocking features
- Test async functions and error handling
- Apply unit testing best practices

---

## What Makes a Good Unit Test?

### The FIRST Principles

```
F - Fast      : Runs in milliseconds
I - Isolated  : No dependencies on other tests
R - Repeatable: Same result every time
S - Self-validating: Pass or fail, no manual checking
T - Timely    : Written alongside the code
```

### Anatomy of a Unit Test

```javascript
describe('Function/Module name', () => {
  // Setup shared test data
  beforeEach(() => {
    // Reset state before each test
  });

  describe('method name', () => {
    it('should do something when given valid input', () => {
      // Arrange - Set up test data
      const input = { ... };

      // Act - Call the function
      const result = myFunction(input);

      // Assert - Verify the result
      expect(result).toBe(expected);
    });

    it('should throw error when given invalid input', () => {
      // Test edge cases and errors
    });
  });
});
```

---

## Testing Pure Functions

Pure functions are easiest to test - same input always produces same output.

### Utility Functions

```javascript
// src/utils/validators.ts
export function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

export function isStrongPassword(password: string): {
  valid: boolean;
  errors: string[];
} {
  const errors: string[] = [];

  if (password.length < 8) {
    errors.push('Password must be at least 8 characters');
  }
  if (!/[A-Z]/.test(password)) {
    errors.push('Password must contain an uppercase letter');
  }
  if (!/[a-z]/.test(password)) {
    errors.push('Password must contain a lowercase letter');
  }
  if (!/[0-9]/.test(password)) {
    errors.push('Password must contain a number');
  }

  return { valid: errors.length === 0, errors };
}

export function slugify(text: string): string {
  return text
    .toLowerCase()
    .trim()
    .replace(/[^\w\s-]/g, '')
    .replace(/[\s_-]+/g, '-')
    .replace(/^-+|-+$/g, '');
}
```

### Tests

```javascript
// tests/unit/utils/validators.test.ts
import {
  isValidEmail,
  isStrongPassword,
  slugify
} from '../../../src/utils/validators';

describe('Validators', () => {
  describe('isValidEmail', () => {
    it('should return true for valid emails', () => {
      const validEmails = [
        'test@example.com',
        'user.name@domain.org',
        'user+tag@example.co.uk'
      ];

      validEmails.forEach(email => {
        expect(isValidEmail(email)).toBe(true);
      });
    });

    it('should return false for invalid emails', () => {
      const invalidEmails = [
        'invalid',
        '@nodomain.com',
        'no@domain',
        'spaces in@email.com',
        ''
      ];

      invalidEmails.forEach(email => {
        expect(isValidEmail(email)).toBe(false);
      });
    });
  });

  describe('isStrongPassword', () => {
    it('should accept strong passwords', () => {
      const result = isStrongPassword('SecurePass123');

      expect(result.valid).toBe(true);
      expect(result.errors).toHaveLength(0);
    });

    it('should reject short passwords', () => {
      const result = isStrongPassword('Short1');

      expect(result.valid).toBe(false);
      expect(result.errors).toContain(
        'Password must be at least 8 characters'
      );
    });

    it('should require uppercase letters', () => {
      const result = isStrongPassword('lowercase123');

      expect(result.valid).toBe(false);
      expect(result.errors).toContain(
        'Password must contain an uppercase letter'
      );
    });

    it('should require numbers', () => {
      const result = isStrongPassword('NoNumbers');

      expect(result.valid).toBe(false);
      expect(result.errors).toContain(
        'Password must contain a number'
      );
    });

    it('should return all validation errors', () => {
      const result = isStrongPassword('short');

      expect(result.valid).toBe(false);
      expect(result.errors.length).toBeGreaterThan(1);
    });
  });

  describe('slugify', () => {
    it('should convert text to lowercase', () => {
      expect(slugify('Hello World')).toBe('hello-world');
    });

    it('should replace spaces with hyphens', () => {
      expect(slugify('my blog post')).toBe('my-blog-post');
    });

    it('should remove special characters', () => {
      expect(slugify('Hello! World?')).toBe('hello-world');
    });

    it('should handle multiple spaces', () => {
      expect(slugify('too   many   spaces')).toBe('too-many-spaces');
    });

    it('should trim leading/trailing hyphens', () => {
      expect(slugify('  hello  ')).toBe('hello');
    });

    it('should handle empty string', () => {
      expect(slugify('')).toBe('');
    });
  });
});
```

---

## Testing Business Logic

### Price Calculator Service

```javascript
// src/services/priceCalculator.ts
interface CartItem {
  productId: string;
  price: number;
  quantity: number;
}

interface Discount {
  type: 'percentage' | 'fixed';
  value: number;
  minPurchase?: number;
}

export class PriceCalculator {
  calculateSubtotal(items: CartItem[]): number {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  applyDiscount(subtotal: number, discount: Discount): number {
    if (discount.minPurchase && subtotal < discount.minPurchase) {
      return subtotal;
    }

    if (discount.type === 'percentage') {
      return subtotal * (1 - discount.value / 100);
    }

    return Math.max(0, subtotal - discount.value);
  }

  calculateTax(amount: number, taxRate: number): number {
    return Math.round(amount * taxRate * 100) / 100;
  }

  calculateTotal(
    items: CartItem[],
    discount?: Discount,
    taxRate: number = 0.1
  ): {
    subtotal: number;
    discount: number;
    tax: number;
    total: number;
  } {
    const subtotal = this.calculateSubtotal(items);
    const afterDiscount = discount
      ? this.applyDiscount(subtotal, discount)
      : subtotal;
    const discountAmount = subtotal - afterDiscount;
    const tax = this.calculateTax(afterDiscount, taxRate);
    const total = afterDiscount + tax;

    return {
      subtotal,
      discount: discountAmount,
      tax,
      total: Math.round(total * 100) / 100
    };
  }
}
```

### Tests

```javascript
// tests/unit/services/priceCalculator.test.ts
import { PriceCalculator } from '../../../src/services/priceCalculator';

describe('PriceCalculator', () => {
  let calculator: PriceCalculator;

  beforeEach(() => {
    calculator = new PriceCalculator();
  });

  describe('calculateSubtotal', () => {
    it('should calculate subtotal for single item', () => {
      const items = [{ productId: '1', price: 10, quantity: 2 }];

      expect(calculator.calculateSubtotal(items)).toBe(20);
    });

    it('should calculate subtotal for multiple items', () => {
      const items = [
        { productId: '1', price: 10, quantity: 2 },
        { productId: '2', price: 15, quantity: 1 }
      ];

      expect(calculator.calculateSubtotal(items)).toBe(35);
    });

    it('should return 0 for empty cart', () => {
      expect(calculator.calculateSubtotal([])).toBe(0);
    });
  });

  describe('applyDiscount', () => {
    it('should apply percentage discount', () => {
      const discount = { type: 'percentage' as const, value: 10 };

      expect(calculator.applyDiscount(100, discount)).toBe(90);
    });

    it('should apply fixed discount', () => {
      const discount = { type: 'fixed' as const, value: 15 };

      expect(calculator.applyDiscount(100, discount)).toBe(85);
    });

    it('should not go below zero with fixed discount', () => {
      const discount = { type: 'fixed' as const, value: 50 };

      expect(calculator.applyDiscount(30, discount)).toBe(0);
    });

    it('should not apply discount below minimum purchase', () => {
      const discount = {
        type: 'percentage' as const,
        value: 10,
        minPurchase: 100
      };

      expect(calculator.applyDiscount(50, discount)).toBe(50);
    });

    it('should apply discount at minimum purchase', () => {
      const discount = {
        type: 'percentage' as const,
        value: 10,
        minPurchase: 100
      };

      expect(calculator.applyDiscount(100, discount)).toBe(90);
    });
  });

  describe('calculateTax', () => {
    it('should calculate tax correctly', () => {
      expect(calculator.calculateTax(100, 0.1)).toBe(10);
    });

    it('should round to 2 decimal places', () => {
      expect(calculator.calculateTax(33.33, 0.1)).toBe(3.33);
    });
  });

  describe('calculateTotal', () => {
    it('should calculate complete order total', () => {
      const items = [
        { productId: '1', price: 50, quantity: 2 }
      ];
      const discount = { type: 'percentage' as const, value: 10 };

      const result = calculator.calculateTotal(items, discount, 0.1);

      expect(result).toEqual({
        subtotal: 100,
        discount: 10,
        tax: 9,
        total: 99
      });
    });

    it('should calculate without discount', () => {
      const items = [{ productId: '1', price: 100, quantity: 1 }];

      const result = calculator.calculateTotal(items);

      expect(result).toEqual({
        subtotal: 100,
        discount: 0,
        tax: 10,
        total: 110
      });
    });
  });
});
```

---

## Testing Async Functions

### Async Service

```javascript
// src/services/userService.ts
import { prisma } from '../lib/prisma';
import bcrypt from 'bcrypt';

export class UserService {
  async createUser(data: {
    email: string;
    password: string;
    name: string;
  }) {
    const existing = await prisma.user.findUnique({
      where: { email: data.email }
    });

    if (existing) {
      throw new Error('Email already registered');
    }

    const hashedPassword = await bcrypt.hash(data.password, 10);

    const user = await prisma.user.create({
      data: {
        ...data,
        password: hashedPassword
      }
    });

    const { password, ...userWithoutPassword } = user;
    return userWithoutPassword;
  }

  async findByEmail(email: string) {
    return prisma.user.findUnique({ where: { email } });
  }

  async verifyPassword(email: string, password: string): Promise<boolean> {
    const user = await this.findByEmail(email);

    if (!user) {
      return false;
    }

    return bcrypt.compare(password, user.password);
  }
}
```

### Async Tests

```javascript
// tests/unit/services/userService.test.ts
import { UserService } from '../../../src/services/userService';
import { prisma } from '../../helpers/prisma';

describe('UserService', () => {
  let userService: UserService;

  beforeEach(() => {
    userService = new UserService();
  });

  describe('createUser', () => {
    it('should create a new user', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'Password123',
        name: 'Test User'
      };

      const user = await userService.createUser(userData);

      expect(user).toMatchObject({
        email: 'test@example.com',
        name: 'Test User'
      });
      expect(user).not.toHaveProperty('password');
    });

    it('should hash the password', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'Password123',
        name: 'Test User'
      };

      await userService.createUser(userData);

      const savedUser = await prisma.user.findUnique({
        where: { email: 'test@example.com' }
      });

      expect(savedUser?.password).not.toBe('Password123');
      expect(savedUser?.password).toMatch(/^\$2[ab]\$/);
    });

    it('should throw error for duplicate email', async () => {
      const userData = {
        email: 'duplicate@example.com',
        password: 'Password123',
        name: 'Test User'
      };

      await userService.createUser(userData);

      await expect(userService.createUser(userData))
        .rejects
        .toThrow('Email already registered');
    });
  });

  describe('verifyPassword', () => {
    beforeEach(async () => {
      await userService.createUser({
        email: 'verify@example.com',
        password: 'CorrectPassword123',
        name: 'Test'
      });
    });

    it('should return true for correct password', async () => {
      const result = await userService.verifyPassword(
        'verify@example.com',
        'CorrectPassword123'
      );

      expect(result).toBe(true);
    });

    it('should return false for incorrect password', async () => {
      const result = await userService.verifyPassword(
        'verify@example.com',
        'WrongPassword'
      );

      expect(result).toBe(false);
    });

    it('should return false for non-existent user', async () => {
      const result = await userService.verifyPassword(
        'nonexistent@example.com',
        'AnyPassword'
      );

      expect(result).toBe(false);
    });
  });
});
```

---

## Testing Error Handling

```javascript
// src/services/orderService.ts
export class OrderService {
  async createOrder(userId: string, items: OrderItem[]) {
    if (items.length === 0) {
      throw new ValidationError('Order must have at least one item');
    }

    for (const item of items) {
      const product = await prisma.product.findUnique({
        where: { id: item.productId }
      });

      if (!product) {
        throw new NotFoundError(`Product ${item.productId} not found`);
      }

      if (product.stock < item.quantity) {
        throw new InsufficientStockError(
          `Insufficient stock for ${product.name}`
        );
      }
    }

    // Create order...
  }
}

// Custom errors
export class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

export class NotFoundError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'NotFoundError';
  }
}

export class InsufficientStockError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'InsufficientStockError';
  }
}
```

### Error Tests

```javascript
// tests/unit/services/orderService.test.ts
import {
  OrderService,
  ValidationError,
  NotFoundError,
  InsufficientStockError
} from '../../../src/services/orderService';
import { createProduct } from '../../helpers/factories';

describe('OrderService', () => {
  let orderService: OrderService;

  beforeEach(() => {
    orderService = new OrderService();
  });

  describe('createOrder', () => {
    it('should throw ValidationError for empty items', async () => {
      await expect(orderService.createOrder('user-1', []))
        .rejects
        .toThrow(ValidationError);

      await expect(orderService.createOrder('user-1', []))
        .rejects
        .toThrow('Order must have at least one item');
    });

    it('should throw NotFoundError for invalid product', async () => {
      const items = [{ productId: 'non-existent', quantity: 1 }];

      await expect(orderService.createOrder('user-1', items))
        .rejects
        .toThrow(NotFoundError);
    });

    it('should throw InsufficientStockError when stock is low', async () => {
      const product = await createProduct({ stock: 5 });
      const items = [{ productId: product.id, quantity: 10 }];

      await expect(orderService.createOrder('user-1', items))
        .rejects
        .toThrow(InsufficientStockError);
    });

    it('should throw specific error type', async () => {
      try {
        await orderService.createOrder('user-1', []);
      } catch (error) {
        expect(error).toBeInstanceOf(ValidationError);
        expect(error.name).toBe('ValidationError');
      }
    });
  });
});
```

---

## Test Organization Patterns

### Describe Blocks for Organization

```javascript
describe('UserService', () => {
  // Shared setup
  let service: UserService;

  beforeEach(() => {
    service = new UserService();
  });

  // Group by method
  describe('createUser', () => {
    describe('with valid input', () => {
      it('should create user successfully', () => {});
      it('should hash password', () => {});
      it('should return user without password', () => {});
    });

    describe('with invalid input', () => {
      it('should reject invalid email', () => {});
      it('should reject weak password', () => {});
      it('should reject duplicate email', () => {});
    });
  });

  describe('updateUser', () => {
    describe('when user exists', () => {
      it('should update name', () => {});
      it('should update email', () => {});
    });

    describe('when user does not exist', () => {
      it('should throw NotFoundError', () => {});
    });
  });
});
```

### Test Data Tables

```javascript
describe('isValidEmail', () => {
  // Test multiple cases with table
  const validEmails = [
    'simple@example.com',
    'very.common@example.com',
    'disposable.style.email.with+symbol@example.com',
    'other.email-with-hyphen@example.com',
    'fully-qualified-domain@example.com',
    'user.name+tag+sorting@example.com'
  ];

  const invalidEmails = [
    ['plainaddress', 'missing @ symbol'],
    ['@no-local-part.com', 'missing local part'],
    ['no-domain@.com', 'missing domain'],
    ['spaces in@email.com', 'contains spaces'],
    ['', 'empty string']
  ];

  test.each(validEmails)('should accept %s', (email) => {
    expect(isValidEmail(email)).toBe(true);
  });

  test.each(invalidEmails)(
    'should reject %s because %s',
    (email, reason) => {
      expect(isValidEmail(email)).toBe(false);
    }
  );
});
```

---

## Best Practices

### 1. Test One Thing Per Test

```javascript
// BAD: Testing multiple things
it('should create and validate user', async () => {
  const user = await createUser(data);
  expect(user).toBeDefined();
  expect(user.email).toBe(data.email);
  expect(await verifyPassword(data.email, data.password)).toBe(true);
});

// GOOD: Separate tests
it('should create user with correct email', async () => {
  const user = await createUser(data);
  expect(user.email).toBe(data.email);
});

it('should hash password correctly', async () => {
  await createUser(data);
  expect(await verifyPassword(data.email, data.password)).toBe(true);
});
```

### 2. Use Descriptive Test Names

```javascript
// BAD
it('works', () => {});
it('test 1', () => {});

// GOOD
it('should return empty array when no products match filter', () => {});
it('should throw ValidationError when email is invalid', () => {});
```

### 3. Arrange-Act-Assert Pattern

```javascript
it('should calculate discount correctly', () => {
  // Arrange
  const cart = createCart([
    { price: 100, quantity: 2 },
    { price: 50, quantity: 1 }
  ]);
  const discount = { type: 'percentage', value: 10 };

  // Act
  const result = calculateTotal(cart, discount);

  // Assert
  expect(result.discount).toBe(25);
  expect(result.total).toBe(225);
});
```

---

## Practice Exercise

Write unit tests for a `ProductService`:

1. **Create ProductService** with methods:
   - `createProduct(data)`
   - `updateStock(productId, quantity)`
   - `searchProducts(query)`
   - `calculateDiscount(product, discountCode)`

2. **Write tests for**:
   - Creating products with valid/invalid data
   - Updating stock (increment/decrement)
   - Searching with various filters
   - Applying different discount types

3. **Include edge cases**:
   - Empty search results
   - Negative stock prevention
   - Invalid discount codes

---

## Key Takeaways

1. **Test behavior, not implementation** - Focus on what, not how
2. **Keep tests isolated** - No dependencies between tests
3. **Use descriptive names** - Tests as documentation
4. **Test edge cases** - Empty arrays, null values, boundaries
5. **One assertion focus** - Test one thing per test
6. **Fast feedback** - Unit tests should run quickly

---

## What's Next?

Tomorrow, we'll explore **Integration Testing** - testing how multiple components work together, including database interactions and API endpoints.
