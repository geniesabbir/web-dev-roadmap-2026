# Day 5: Test Coverage - Measuring and Improving Your Tests

## Introduction

Test coverage measures how much of your code is executed when tests run. While 100% coverage doesn't guarantee bug-free code, coverage reports help identify untested areas. Today, you'll learn how to generate, interpret, and use coverage reports to improve your test suite.

## Learning Objectives

By the end of this lesson, you will be able to:
- Generate test coverage reports
- Understand coverage metrics
- Identify gaps in test coverage
- Set and enforce coverage thresholds
- Improve coverage strategically

---

## Understanding Coverage Metrics

### Types of Coverage

```
┌────────────────────────────────────────────────────────────────┐
│                    Coverage Metrics                             │
├──────────────────┬─────────────────────────────────────────────┤
│ Statement        │ % of statements executed                     │
│ Branch           │ % of if/else branches taken                  │
│ Function         │ % of functions called                        │
│ Line             │ % of lines executed                          │
└──────────────────┴─────────────────────────────────────────────┘
```

### Example

```javascript
// src/utils/calculator.ts
export function calculate(a: number, b: number, operation: string) {
  if (operation === 'add') {        // Branch 1
    return a + b;                    // Statement 1
  } else if (operation === 'subtract') {  // Branch 2
    return a - b;                    // Statement 2
  } else if (operation === 'multiply') {  // Branch 3
    return a * b;                    // Statement 3
  } else {                           // Branch 4
    throw new Error('Unknown operation');  // Statement 4
  }
}

// Test with partial coverage
describe('calculate', () => {
  it('should add numbers', () => {
    expect(calculate(2, 3, 'add')).toBe(5);
  });
});

// Coverage report:
// Statements: 2/4 = 50%
// Branches: 1/4 = 25%
// Functions: 1/1 = 100%
// Lines: 2/4 = 50%
```

---

## Generating Coverage Reports

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',

  // Coverage configuration
  collectCoverage: false,  // Enable via CLI flag
  coverageDirectory: 'coverage',

  // What to collect coverage from
  collectCoverageFrom: [
    'src/**/*.{js,ts}',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts',
    '!src/index.ts',
    '!src/types/**',
    '!src/**/__mocks__/**'
  ],

  // Coverage thresholds
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    // Per-file thresholds
    './src/services/': {
      branches: 90,
      functions: 90
    }
  },

  // Coverage reporters
  coverageReporters: [
    'text',           // Console output
    'text-summary',   // Summary in console
    'html',           // HTML report
    'lcov',           // For CI tools
    'json'            // JSON data
  ]
};
```

### Running Coverage

```bash
# Generate coverage report
npm test -- --coverage

# Watch mode with coverage
npm test -- --coverage --watchAll

# Coverage for specific files
npm test -- --coverage --collectCoverageFrom='src/services/**/*.ts'

# Update snapshots with coverage
npm test -- --coverage --updateSnapshot
```

---

## Reading Coverage Reports

### Console Output

```
--------------------|---------|----------|---------|---------|-------------------
File                | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
--------------------|---------|----------|---------|---------|-------------------
All files           |   85.23 |    78.45 |   89.12 |   84.56 |
 services           |   92.34 |    88.23 |   95.00 |   91.45 |
  userService.ts    |   95.00 |    90.00 |  100.00 |   94.00 | 45,67-69
  orderService.ts   |   89.00 |    85.00 |   90.00 |   88.00 | 23-25,78,112
 controllers        |   78.56 |    70.12 |   82.00 |   77.89 |
  userController.ts |   80.00 |    72.00 |   85.00 |   79.00 | 34-40,89-95
  authController.ts |   77.00 |    68.00 |   79.00 |   76.00 | 56-62,101-110
 utils              |   88.00 |    82.00 |   92.00 |   87.00 |
  validators.ts     |   90.00 |    85.00 |   95.00 |   89.00 | 23
  formatters.ts     |   86.00 |    79.00 |   89.00 |   85.00 | 12-14,45
--------------------|---------|----------|---------|---------|-------------------
```

### HTML Report

```bash
# Open HTML coverage report
open coverage/lcov-report/index.html
```

The HTML report provides:
- File-by-file coverage breakdown
- Line-by-line highlighting
- Clickable navigation
- Branch coverage visualization

---

## Improving Coverage

### Finding Untested Code

```javascript
// coverage/lcov-report/userService.ts.html shows:

export class UserService {
  async createUser(data: CreateUserInput) {
    // ✓ Covered
    const existing = await this.prisma.user.findUnique({
      where: { email: data.email }
    });

    // ✗ Branch not covered - missing test for duplicate email
    if (existing) {
      throw new Error('Email already exists');
    }

    // ✓ Covered
    return this.prisma.user.create({ data });
  }

  // ✗ Function not covered - no tests for deleteUser
  async deleteUser(id: string) {
    return this.prisma.user.delete({ where: { id } });
  }
}
```

### Adding Missing Tests

```javascript
// Add test for uncovered branch
describe('createUser', () => {
  it('should throw error for duplicate email', async () => {
    // Create existing user
    await createUser({ email: 'existing@example.com' });

    // Try to create duplicate
    await expect(
      userService.createUser({
        email: 'existing@example.com',
        name: 'Test'
      })
    ).rejects.toThrow('Email already exists');
  });
});

// Add tests for uncovered function
describe('deleteUser', () => {
  it('should delete user', async () => {
    const user = await createUser();

    await userService.deleteUser(user.id);

    const deleted = await prisma.user.findUnique({
      where: { id: user.id }
    });
    expect(deleted).toBeNull();
  });

  it('should throw error for non-existent user', async () => {
    await expect(
      userService.deleteUser('non-existent-id')
    ).rejects.toThrow();
  });
});
```

---

## Coverage Thresholds

### Global Thresholds

```javascript
// jest.config.js
coverageThreshold: {
  global: {
    branches: 80,
    functions: 80,
    lines: 80,
    statements: 80
  }
}

// Test will fail if coverage drops below thresholds:
// Jest: "Coverage threshold for branches (80%) not met: 75%"
```

### Per-Directory Thresholds

```javascript
coverageThreshold: {
  global: {
    branches: 75,
    functions: 75,
    lines: 75,
    statements: 75
  },
  './src/services/': {
    branches: 90,
    functions: 90,
    lines: 90,
    statements: 90
  },
  './src/utils/': {
    branches: 95,
    functions: 95,
    lines: 95,
    statements: 95
  }
}
```

### Per-File Thresholds

```javascript
coverageThreshold: {
  './src/services/paymentService.ts': {
    branches: 100,
    functions: 100,
    lines: 100,
    statements: 100
  }
}
```

---

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm run test:ci
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true

      - name: Comment coverage on PR
        uses: marocchino/sticky-pull-request-comment@v2
        if: github.event_name == 'pull_request'
        with:
          path: coverage/coverage-summary.txt
```

### Coverage Badge

```markdown
<!-- README.md -->
[![codecov](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)](https://codecov.io/gh/username/repo)
```

---

## Coverage Best Practices

### What to Cover

```javascript
// HIGH PRIORITY - Business logic
export class OrderService {
  calculateTotal(items, discount) {
    // Critical - must test thoroughly
  }

  applyDiscount(total, code) {
    // Critical - money involved
  }
}

// MEDIUM PRIORITY - Data access
export class UserRepository {
  findById(id) { /* ... */ }
  create(data) { /* ... */ }
}

// LOWER PRIORITY - Simple getters/setters
export class User {
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

### What to Exclude

```javascript
// jest.config.js
collectCoverageFrom: [
  'src/**/*.ts',

  // Exclude
  '!src/index.ts',           // Entry point
  '!src/types/**',           // Type definitions
  '!src/migrations/**',      // Database migrations
  '!src/**/*.d.ts',          // Declaration files
  '!src/config/**',          // Configuration
  '!src/**/__mocks__/**',    // Test mocks
  '!src/scripts/**'          // One-time scripts
]
```

### Meaningful Coverage

```javascript
// BAD: Covers code but doesn't test behavior
it('should call createUser', async () => {
  await userService.createUser({ email: 'test@test.com' });
  // No assertions!
});

// GOOD: Tests actual behavior
it('should create user with hashed password', async () => {
  const result = await userService.createUser({
    email: 'test@test.com',
    password: 'plaintext'
  });

  expect(result.email).toBe('test@test.com');
  expect(result.password).not.toBe('plaintext');
  expect(result.password).toMatch(/^\$2[ab]\$/);
});
```

---

## Advanced Coverage Patterns

### Testing Error Paths

```javascript
// Ensure error handling is covered
describe('Error handling', () => {
  it('should handle database errors', async () => {
    prismaMock.user.findUnique.mockRejectedValue(
      new Error('Connection failed')
    );

    await expect(userService.findById('1'))
      .rejects
      .toThrow('Connection failed');
  });

  it('should handle validation errors', async () => {
    await expect(userService.createUser({ email: 'invalid' }))
      .rejects
      .toThrow('Invalid email');
  });
});
```

### Testing Edge Cases

```javascript
describe('Edge cases', () => {
  it('should handle empty array', () => {
    expect(calculateTotal([])).toBe(0);
  });

  it('should handle null input', () => {
    expect(() => processData(null)).toThrow('Input required');
  });

  it('should handle very large numbers', () => {
    expect(calculateTotal([{ price: Number.MAX_SAFE_INTEGER }]))
      .toBe(Number.MAX_SAFE_INTEGER);
  });

  it('should handle unicode strings', () => {
    expect(slugify('Héllo Wörld 你好')).toBe('hello-world');
  });
});
```

### Branch Coverage Examples

```javascript
// Function with multiple branches
function getDiscount(user, orderTotal) {
  if (!user) {
    return 0;
  }

  if (user.isPremium && orderTotal > 100) {
    return 0.2;
  } else if (user.isPremium) {
    return 0.1;
  } else if (orderTotal > 100) {
    return 0.05;
  }

  return 0;
}

// Tests for all branches
describe('getDiscount', () => {
  it('should return 0 for no user', () => {
    expect(getDiscount(null, 100)).toBe(0);
  });

  it('should return 20% for premium user with large order', () => {
    expect(getDiscount({ isPremium: true }, 150)).toBe(0.2);
  });

  it('should return 10% for premium user with small order', () => {
    expect(getDiscount({ isPremium: true }, 50)).toBe(0.1);
  });

  it('should return 5% for regular user with large order', () => {
    expect(getDiscount({ isPremium: false }, 150)).toBe(0.05);
  });

  it('should return 0 for regular user with small order', () => {
    expect(getDiscount({ isPremium: false }, 50)).toBe(0);
  });
});
```

---

## Coverage Reports in Practice

### Generating Summary Report

```javascript
// scripts/coverage-summary.js
const fs = require('fs');

const coverage = JSON.parse(
  fs.readFileSync('coverage/coverage-summary.json', 'utf8')
);

const total = coverage.total;

console.log('Coverage Summary');
console.log('================');
console.log(`Statements: ${total.statements.pct}%`);
console.log(`Branches: ${total.branches.pct}%`);
console.log(`Functions: ${total.functions.pct}%`);
console.log(`Lines: ${total.lines.pct}%`);

// Check thresholds
const threshold = 80;
const failed = Object.entries(total).some(
  ([key, value]) => key !== 'linesCovered' && value.pct < threshold
);

if (failed) {
  console.error(`\n❌ Coverage below ${threshold}% threshold`);
  process.exit(1);
} else {
  console.log(`\n✅ All coverage above ${threshold}%`);
}
```

### Tracking Coverage Over Time

```javascript
// Save coverage data after each run
const fs = require('fs');

const coverage = JSON.parse(
  fs.readFileSync('coverage/coverage-summary.json', 'utf8')
);

const history = JSON.parse(
  fs.readFileSync('coverage-history.json', 'utf8') || '[]'
);

history.push({
  date: new Date().toISOString(),
  statements: coverage.total.statements.pct,
  branches: coverage.total.branches.pct,
  functions: coverage.total.functions.pct,
  lines: coverage.total.lines.pct
});

fs.writeFileSync('coverage-history.json', JSON.stringify(history, null, 2));
```

---

## Practice Exercise

Improve test coverage for a project:

1. **Generate coverage report**
2. **Identify lowest coverage files**
3. **Add tests for**:
   - Untested functions
   - Uncovered branches
   - Error handling paths
4. **Aim for 80%+ coverage**
5. **Set up coverage thresholds** in CI

---

## Key Takeaways

1. **Coverage is a guide, not a goal** - 100% coverage ≠ bug-free
2. **Focus on critical code** - Business logic first
3. **Test behavior, not lines** - Meaningful assertions
4. **Use thresholds** - Prevent coverage regression
5. **Cover branches** - if/else, try/catch
6. **Exclude appropriately** - Types, configs, migrations

---

## Congratulations!

You've completed the Backend Testing section! You now know how to:
- Set up Jest testing environment
- Write unit tests for isolated functions
- Create integration tests for API endpoints
- Mock dependencies effectively
- Generate and improve test coverage

Next up is **Phase 4: DevOps** - where you'll learn Linux CLI, Docker, CI/CD, and cloud deployment!
