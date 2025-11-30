# Day 4: Mocking - Isolating Components for Testing

## Introduction

Mocking allows you to replace real implementations with controlled test doubles. This is essential when testing code that depends on external services, databases, or complex dependencies. Today, you'll learn various mocking techniques to write fast, isolated, and reliable tests.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand different types of test doubles
- Mock functions and modules with Jest
- Mock database operations with Prisma
- Mock external API calls
- Use spies to verify function calls

---

## Types of Test Doubles

```
┌─────────────────────────────────────────────────────────────────┐
│                      Test Doubles                                │
├──────────┬──────────┬──────────┬──────────┬────────────────────┤
│  Dummy   │  Stub    │  Spy     │  Mock    │  Fake              │
├──────────┼──────────┼──────────┼──────────┼────────────────────┤
│ Passed   │ Returns  │ Records  │ Verifies │ Working            │
│ but not  │ canned   │ calls    │ behavior │ implementation     │
│ used     │ answers  │ made     │ and      │ (simplified)       │
│          │          │          │ returns  │                    │
└──────────┴──────────┴──────────┴──────────┴────────────────────┘
```

### Examples

```javascript
// Dummy - Just fills a parameter
const dummyLogger = { log: () => {} };

// Stub - Returns predefined data
const userStub = { id: '1', name: 'Test User' };

// Spy - Tracks calls
const spy = jest.fn();

// Mock - Full replacement with expectations
const mockUserService = {
  findById: jest.fn().mockResolvedValue(userStub)
};

// Fake - Working but simplified
class FakeEmailService {
  sentEmails = [];
  send(to, subject, body) {
    this.sentEmails.push({ to, subject, body });
    return Promise.resolve({ success: true });
  }
}
```

---

## Jest Mock Functions

### Creating Mocks

```javascript
// Basic mock function
const mockFn = jest.fn();

// Mock with return value
const mockGetUser = jest.fn().mockReturnValue({ id: '1', name: 'Test' });

// Mock with different return values
const mockRandom = jest.fn()
  .mockReturnValueOnce(0.1)
  .mockReturnValueOnce(0.5)
  .mockReturnValue(0.9);

// Mock async function
const mockFetch = jest.fn().mockResolvedValue({ data: [] });

// Mock that throws
const mockError = jest.fn().mockRejectedValue(new Error('Failed'));

// Mock implementation
const mockCalculate = jest.fn().mockImplementation((a, b) => a + b);
```

### Asserting on Mocks

```javascript
describe('Mock Assertions', () => {
  const mockFn = jest.fn();

  beforeEach(() => {
    mockFn.mockClear();
  });

  it('should track calls', () => {
    mockFn('arg1', 'arg2');
    mockFn('arg3');

    // Was called
    expect(mockFn).toHaveBeenCalled();

    // Called specific number of times
    expect(mockFn).toHaveBeenCalledTimes(2);

    // Called with specific arguments
    expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');

    // Last call arguments
    expect(mockFn).toHaveBeenLastCalledWith('arg3');

    // Access all calls
    expect(mockFn.mock.calls).toEqual([
      ['arg1', 'arg2'],
      ['arg3']
    ]);
  });

  it('should track return values', () => {
    mockFn.mockReturnValue('result');
    const result = mockFn();

    expect(mockFn.mock.results[0]).toEqual({
      type: 'return',
      value: 'result'
    });
  });
});
```

---

## Mocking Modules

### Auto Mocking

```javascript
// Jest automatically mocks the entire module
jest.mock('../src/services/emailService');

import { EmailService } from '../src/services/emailService';

describe('with auto mock', () => {
  it('all methods are mocks', () => {
    const service = new EmailService();

    // All methods are jest.fn()
    expect(jest.isMockFunction(service.send)).toBe(true);
  });
});
```

### Manual Module Mock

```javascript
// tests/__mocks__/emailService.ts
export const EmailService = jest.fn().mockImplementation(() => ({
  send: jest.fn().mockResolvedValue({ messageId: 'test-123' }),
  sendBulk: jest.fn().mockResolvedValue({ sent: 10, failed: 0 })
}));

// In test file
jest.mock('../src/services/emailService');

import { EmailService } from '../src/services/emailService';

describe('EmailService', () => {
  it('should mock send', async () => {
    const service = new EmailService();

    const result = await service.send('to@test.com', 'Subject', 'Body');

    expect(result.messageId).toBe('test-123');
    expect(service.send).toHaveBeenCalledWith('to@test.com', 'Subject', 'Body');
  });
});
```

### Partial Module Mock

```javascript
// Mock only specific exports
jest.mock('../src/utils/helpers', () => ({
  ...jest.requireActual('../src/utils/helpers'),
  formatDate: jest.fn().mockReturnValue('2024-01-01')
}));

import { formatDate, formatCurrency } from '../src/utils/helpers';

describe('Partial mock', () => {
  it('should mock formatDate', () => {
    expect(formatDate(new Date())).toBe('2024-01-01');
  });

  it('should use real formatCurrency', () => {
    expect(formatCurrency(10)).toBe('$10.00');
  });
});
```

---

## Mocking Prisma

### Prisma Mock Setup

```javascript
// tests/__mocks__/prisma.ts
import { PrismaClient } from '@prisma/client';
import { mockDeep, mockReset, DeepMockProxy } from 'jest-mock-extended';

export const prismaMock = mockDeep<PrismaClient>();

// Reset between tests
beforeEach(() => {
  mockReset(prismaMock);
});

export { prismaMock as prisma };
```

### Using Prisma Mock

```javascript
// tests/unit/services/userService.test.ts
import { prismaMock } from '../__mocks__/prisma';
import { UserService } from '../../src/services/userService';

// Tell Jest to use the mock
jest.mock('../../src/lib/prisma', () => ({
  prisma: prismaMock
}));

describe('UserService', () => {
  const userService = new UserService();

  describe('findById', () => {
    it('should return user when found', async () => {
      const mockUser = {
        id: '1',
        email: 'test@example.com',
        name: 'Test User',
        createdAt: new Date()
      };

      prismaMock.user.findUnique.mockResolvedValue(mockUser);

      const result = await userService.findById('1');

      expect(result).toEqual(mockUser);
      expect(prismaMock.user.findUnique).toHaveBeenCalledWith({
        where: { id: '1' }
      });
    });

    it('should return null when not found', async () => {
      prismaMock.user.findUnique.mockResolvedValue(null);

      const result = await userService.findById('999');

      expect(result).toBeNull();
    });
  });

  describe('createUser', () => {
    it('should create and return user', async () => {
      const inputData = {
        email: 'new@example.com',
        name: 'New User',
        password: 'hashedpassword'
      };

      const createdUser = {
        id: '1',
        ...inputData,
        createdAt: new Date()
      };

      prismaMock.user.create.mockResolvedValue(createdUser);

      const result = await userService.createUser(inputData);

      expect(result).toEqual(createdUser);
      expect(prismaMock.user.create).toHaveBeenCalledWith({
        data: inputData
      });
    });
  });

  describe('findMany with filters', () => {
    it('should apply filters correctly', async () => {
      const mockUsers = [
        { id: '1', name: 'Admin', role: 'ADMIN' },
        { id: '2', name: 'Admin 2', role: 'ADMIN' }
      ];

      prismaMock.user.findMany.mockResolvedValue(mockUsers);

      const result = await userService.findByRole('ADMIN');

      expect(result).toEqual(mockUsers);
      expect(prismaMock.user.findMany).toHaveBeenCalledWith({
        where: { role: 'ADMIN' }
      });
    });
  });
});
```

---

## Mocking External APIs

### Mocking Fetch/Axios

```javascript
// Mock global fetch
global.fetch = jest.fn();

describe('External API calls', () => {
  beforeEach(() => {
    (global.fetch as jest.Mock).mockReset();
  });

  it('should handle successful API response', async () => {
    const mockResponse = {
      ok: true,
      json: jest.fn().mockResolvedValue({ data: 'test' })
    };

    (global.fetch as jest.Mock).mockResolvedValue(mockResponse);

    const service = new ExternalApiService();
    const result = await service.fetchData();

    expect(result).toEqual({ data: 'test' });
    expect(global.fetch).toHaveBeenCalledWith(
      'https://api.example.com/data',
      expect.any(Object)
    );
  });

  it('should handle API errors', async () => {
    (global.fetch as jest.Mock).mockResolvedValue({
      ok: false,
      status: 500
    });

    const service = new ExternalApiService();

    await expect(service.fetchData()).rejects.toThrow('API Error');
  });
});
```

### Mocking Axios

```javascript
jest.mock('axios');
import axios from 'axios';

const mockedAxios = axios as jest.Mocked<typeof axios>;

describe('Service using Axios', () => {
  it('should fetch data', async () => {
    mockedAxios.get.mockResolvedValue({
      data: { users: [] }
    });

    const service = new ApiService();
    const result = await service.getUsers();

    expect(result).toEqual({ users: [] });
    expect(mockedAxios.get).toHaveBeenCalledWith('/api/users');
  });

  it('should handle errors', async () => {
    mockedAxios.get.mockRejectedValue(new Error('Network Error'));

    const service = new ApiService();

    await expect(service.getUsers()).rejects.toThrow('Network Error');
  });
});
```

---

## Spies

### Spying on Methods

```javascript
describe('Spies', () => {
  it('should spy on existing method', () => {
    const calculator = {
      add: (a: number, b: number) => a + b
    };

    const spy = jest.spyOn(calculator, 'add');

    const result = calculator.add(2, 3);

    expect(result).toBe(5);  // Original implementation runs
    expect(spy).toHaveBeenCalledWith(2, 3);
    expect(spy).toHaveBeenCalledTimes(1);

    spy.mockRestore();  // Restore original
  });

  it('should spy and mock implementation', () => {
    const logger = {
      log: (message: string) => console.log(message)
    };

    const spy = jest.spyOn(logger, 'log').mockImplementation(() => {});

    logger.log('test message');

    expect(spy).toHaveBeenCalledWith('test message');
    // Original console.log not called

    spy.mockRestore();
  });

  it('should spy on module method', () => {
    const dateSpy = jest.spyOn(Date, 'now').mockReturnValue(1000);

    expect(Date.now()).toBe(1000);

    dateSpy.mockRestore();
  });
});
```

---

## Mocking Time

```javascript
describe('Time-based tests', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('should handle setTimeout', () => {
    const callback = jest.fn();

    setTimeout(callback, 1000);

    expect(callback).not.toHaveBeenCalled();

    jest.advanceTimersByTime(1000);

    expect(callback).toHaveBeenCalledTimes(1);
  });

  it('should handle setInterval', () => {
    const callback = jest.fn();

    setInterval(callback, 100);

    jest.advanceTimersByTime(350);

    expect(callback).toHaveBeenCalledTimes(3);
  });

  it('should test debounced function', () => {
    const fn = jest.fn();
    const debouncedFn = debounce(fn, 500);

    debouncedFn();
    debouncedFn();
    debouncedFn();

    expect(fn).not.toHaveBeenCalled();

    jest.advanceTimersByTime(500);

    expect(fn).toHaveBeenCalledTimes(1);
  });

  it('should mock Date', () => {
    jest.setSystemTime(new Date('2024-01-15'));

    expect(new Date().toISOString()).toContain('2024-01-15');
  });
});
```

---

## Mocking Email/SMS Services

```javascript
// src/services/notificationService.ts
interface NotificationProvider {
  send(to: string, message: string): Promise<boolean>;
}

export class NotificationService {
  constructor(private provider: NotificationProvider) {}

  async notifyUser(userId: string, message: string) {
    const user = await this.getUser(userId);
    return this.provider.send(user.email, message);
  }
}

// tests/unit/services/notificationService.test.ts
describe('NotificationService', () => {
  it('should send notification via provider', async () => {
    const mockProvider = {
      send: jest.fn().mockResolvedValue(true)
    };

    const service = new NotificationService(mockProvider);

    // Mock getUser
    jest.spyOn(service as any, 'getUser').mockResolvedValue({
      id: '1',
      email: 'user@example.com'
    });

    const result = await service.notifyUser('1', 'Hello!');

    expect(result).toBe(true);
    expect(mockProvider.send).toHaveBeenCalledWith(
      'user@example.com',
      'Hello!'
    );
  });

  it('should handle provider failure', async () => {
    const mockProvider = {
      send: jest.fn().mockRejectedValue(new Error('Provider down'))
    };

    const service = new NotificationService(mockProvider);

    jest.spyOn(service as any, 'getUser').mockResolvedValue({
      id: '1',
      email: 'user@example.com'
    });

    await expect(service.notifyUser('1', 'Hello!'))
      .rejects
      .toThrow('Provider down');
  });
});
```

---

## Complete Mock Example

### Service with Multiple Dependencies

```javascript
// src/services/orderService.ts
export class OrderService {
  constructor(
    private db: Database,
    private paymentGateway: PaymentGateway,
    private emailService: EmailService,
    private inventoryService: InventoryService
  ) {}

  async createOrder(userId: string, items: OrderItem[]) {
    // 1. Check inventory
    for (const item of items) {
      const available = await this.inventoryService.checkStock(item.productId);
      if (available < item.quantity) {
        throw new Error(`Insufficient stock for ${item.productId}`);
      }
    }

    // 2. Calculate total
    const total = items.reduce((sum, i) => sum + i.price * i.quantity, 0);

    // 3. Create order in DB
    const order = await this.db.orders.create({
      userId,
      items,
      total,
      status: 'PENDING'
    });

    // 4. Process payment
    const payment = await this.paymentGateway.charge(userId, total);

    // 5. Update order status
    await this.db.orders.update(order.id, { status: 'PAID', paymentId: payment.id });

    // 6. Update inventory
    for (const item of items) {
      await this.inventoryService.decreaseStock(item.productId, item.quantity);
    }

    // 7. Send confirmation email
    await this.emailService.sendOrderConfirmation(userId, order);

    return order;
  }
}
```

### Tests with All Mocks

```javascript
// tests/unit/services/orderService.test.ts
describe('OrderService', () => {
  let orderService: OrderService;
  let mockDb: jest.Mocked<Database>;
  let mockPaymentGateway: jest.Mocked<PaymentGateway>;
  let mockEmailService: jest.Mocked<EmailService>;
  let mockInventoryService: jest.Mocked<InventoryService>;

  beforeEach(() => {
    mockDb = {
      orders: {
        create: jest.fn(),
        update: jest.fn()
      }
    } as any;

    mockPaymentGateway = {
      charge: jest.fn()
    } as any;

    mockEmailService = {
      sendOrderConfirmation: jest.fn()
    } as any;

    mockInventoryService = {
      checkStock: jest.fn(),
      decreaseStock: jest.fn()
    } as any;

    orderService = new OrderService(
      mockDb,
      mockPaymentGateway,
      mockEmailService,
      mockInventoryService
    );
  });

  describe('createOrder', () => {
    const userId = 'user-1';
    const items = [
      { productId: 'prod-1', quantity: 2, price: 50 }
    ];

    it('should create order successfully', async () => {
      // Setup mocks
      mockInventoryService.checkStock.mockResolvedValue(10);
      mockDb.orders.create.mockResolvedValue({
        id: 'order-1',
        userId,
        items,
        total: 100,
        status: 'PENDING'
      });
      mockPaymentGateway.charge.mockResolvedValue({ id: 'payment-1' });
      mockDb.orders.update.mockResolvedValue({});
      mockInventoryService.decreaseStock.mockResolvedValue(undefined);
      mockEmailService.sendOrderConfirmation.mockResolvedValue(undefined);

      // Execute
      const result = await orderService.createOrder(userId, items);

      // Assert
      expect(result.id).toBe('order-1');
      expect(mockInventoryService.checkStock).toHaveBeenCalledWith('prod-1');
      expect(mockPaymentGateway.charge).toHaveBeenCalledWith(userId, 100);
      expect(mockInventoryService.decreaseStock).toHaveBeenCalledWith('prod-1', 2);
      expect(mockEmailService.sendOrderConfirmation).toHaveBeenCalled();
    });

    it('should fail when inventory is insufficient', async () => {
      mockInventoryService.checkStock.mockResolvedValue(1);  // Only 1 available

      await expect(orderService.createOrder(userId, items))
        .rejects
        .toThrow('Insufficient stock');

      // Verify no payment was attempted
      expect(mockPaymentGateway.charge).not.toHaveBeenCalled();
    });

    it('should rollback on payment failure', async () => {
      mockInventoryService.checkStock.mockResolvedValue(10);
      mockDb.orders.create.mockResolvedValue({ id: 'order-1' });
      mockPaymentGateway.charge.mockRejectedValue(new Error('Payment failed'));

      await expect(orderService.createOrder(userId, items))
        .rejects
        .toThrow('Payment failed');

      // Verify inventory was not decreased
      expect(mockInventoryService.decreaseStock).not.toHaveBeenCalled();
    });
  });
});
```

---

## Practice Exercise

Create mocks for a `BookingService`:

1. **Dependencies to mock**:
   - Database (Prisma)
   - Calendar API (external)
   - Email service
   - Payment processor

2. **Methods to test**:
   - `createBooking(userId, serviceId, dateTime)`
   - `cancelBooking(bookingId)`
   - `rescheduleBooking(bookingId, newDateTime)`

3. **Scenarios to cover**:
   - Successful booking
   - Slot already taken
   - Payment failure
   - Cancellation refund

---

## Key Takeaways

1. **Mock external dependencies** - APIs, databases, services
2. **Use jest.fn()** for simple mocks
3. **jest.mock()** for module mocking
4. **jest.spyOn()** to track real implementations
5. **Reset mocks** between tests
6. **Don't over-mock** - Integration tests have value too

---

## What's Next?

Tomorrow, we'll learn about **Test Coverage** - measuring how much of your code is tested and using coverage to improve your test suite.
