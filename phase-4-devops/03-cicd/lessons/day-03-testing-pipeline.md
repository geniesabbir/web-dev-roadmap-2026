# Day 3: Testing Pipeline - Automated Quality Assurance

## Introduction

A robust testing pipeline is the foundation of CI/CD. It catches bugs before they reach production, ensures code quality, and gives teams confidence to deploy frequently. Today, you'll learn to build comprehensive testing pipelines that run unit tests, integration tests, E2E tests, and generate coverage reports.

## Learning Objectives

By the end of this lesson, you will be able to:
- Set up automated test execution in GitHub Actions
- Configure parallel test execution
- Generate and publish coverage reports
- Run different test types (unit, integration, E2E)
- Optimize test pipeline performance

---

## Testing Pipeline Overview

### Complete Testing Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    Testing Pipeline                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐        │
│  │  Lint   │   │  Type   │   │  Unit   │   │ Integr. │        │
│  │         │   │  Check  │   │  Tests  │   │  Tests  │        │
│  └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘        │
│       │             │             │             │               │
│       └─────────────┴─────────────┴─────────────┘               │
│                           │                                      │
│                           ▼                                      │
│                    ┌─────────────┐                              │
│                    │  E2E Tests  │                              │
│                    └──────┬──────┘                              │
│                           │                                      │
│                           ▼                                      │
│                    ┌─────────────┐                              │
│                    │  Coverage   │                              │
│                    │   Report    │                              │
│                    └─────────────┘                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Unit Testing Pipeline

### Basic Unit Test Workflow

```yaml
# .github/workflows/test.yml

name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: junit.xml
```

### Jest Configuration for CI

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',

  // CI-specific settings
  ci: true,

  // Coverage settings
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],

  // Reporter for CI
  reporters: [
    'default',
    ['jest-junit', {
      outputDirectory: '.',
      outputName: 'junit.xml',
    }]
  ],

  // Fail on low coverage
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

### With Coverage Reports

```yaml
jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci

      - name: Run tests with coverage
        run: npm test -- --coverage --coverageReporters=lcov

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true

      - name: Coverage comment on PR
        uses: marocchino/sticky-pull-request-comment@v2
        if: github.event_name == 'pull_request'
        with:
          recreate: true
          path: coverage/coverage-summary.md
```

---

## Integration Testing Pipeline

### Database Integration Tests

```yaml
jobs:
  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci

      - name: Run migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379
```

### API Integration Tests

```yaml
jobs:
  api-tests:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: --health-cmd pg_isready

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci

      - name: Setup database
        run: |
          npx prisma migrate deploy
          npx prisma db seed
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/postgres

      - name: Start server
        run: npm run start &
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/postgres
          PORT: 3000

      - name: Wait for server
        run: npx wait-on http://localhost:3000/health

      - name: Run API tests
        run: npm run test:api
```

---

## E2E Testing Pipeline

### Playwright Tests

```yaml
jobs:
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Build application
        run: npm run build

      - name: Run E2E tests
        run: npx playwright test
        env:
          BASE_URL: http://localhost:3000

      - name: Upload Playwright report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['junit', { outputFile: 'e2e-results.xml' }]
  ],

  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Cypress Tests

```yaml
jobs:
  cypress-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci

      - name: Cypress run
        uses: cypress-io/github-action@v6
        with:
          build: npm run build
          start: npm run start
          wait-on: 'http://localhost:3000'
          browser: chrome
          record: true
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}

      - name: Upload screenshots
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: cypress-screenshots
          path: cypress/screenshots
```

---

## Parallel Testing

### Parallel Test Jobs

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci

      - name: Run tests (shard ${{ matrix.shard }}/4)
        run: npm test -- --shard=${{ matrix.shard }}/4
```

### Parallel Playwright

```yaml
jobs:
  e2e:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shardIndex: [1, 2, 3, 4]
        shardTotal: [4]

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'

      - run: npm ci
      - run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}

      - name: Upload blob report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: blob-report-${{ matrix.shardIndex }}
          path: blob-report/

  merge-reports:
    needs: e2e
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4

      - name: Download blob reports
        uses: actions/download-artifact@v4
        with:
          path: all-blob-reports
          pattern: blob-report-*
          merge-multiple: true

      - name: Merge reports
        run: npx playwright merge-reports --reporter html ./all-blob-reports

      - name: Upload merged report
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

---

## Code Quality Checks

### Linting and Type Checking

```yaml
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci

      - name: ESLint
        run: npm run lint

      - name: Prettier check
        run: npm run format:check

      - name: TypeScript
        run: npm run type-check

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4

      - name: Audit dependencies
        run: npm audit --audit-level=high

      - name: Check for secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: main
          head: HEAD
```

### Complete Quality Pipeline

```yaml
name: Quality Checks

on:
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint -- --format @microsoft/eslint-formatter-sarif --output-file eslint.sarif
      - uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: eslint.sarif

  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run type-check

  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm test -- --coverage
      - uses: codecov/codecov-action@v3

  build:
    needs: [lint, type-check, unit-tests]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
```

---

## Test Result Reporting

### PR Comments

```yaml
- name: Test Report
  uses: dorny/test-reporter@v1
  if: success() || failure()
  with:
    name: Jest Tests
    path: junit.xml
    reporter: jest-junit

- name: Coverage Report
  uses: irongut/CodeCoverageSummary@v1.3.0
  with:
    filename: coverage/cobertura-coverage.xml
    badge: true
    format: markdown
    output: both

- name: Add Coverage PR Comment
  uses: marocchino/sticky-pull-request-comment@v2
  if: github.event_name == 'pull_request'
  with:
    recreate: true
    path: code-coverage-results.md
```

### Status Badges

```yaml
# Generate badges
- name: Generate coverage badge
  uses: jaywcjlove/coverage-badges-cli@main
  with:
    source: coverage/coverage-summary.json
    output: coverage/badge.svg

# Update in README
![Coverage](https://img.shields.io/codecov/c/github/owner/repo)
![Tests](https://github.com/owner/repo/workflows/Tests/badge.svg)
```

---

## Complete Testing Workflow

```yaml
# .github/workflows/test.yml

name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: test-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # Quality checks (fast)
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm run format:check

  # Unit tests with coverage
  unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm test -- --coverage --ci
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  # Integration tests
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: --health-cmd pg_isready
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/postgres

  # E2E tests (parallel)
  e2e:
    runs-on: ubuntu-latest
    needs: [quality, unit]
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npx playwright test --shard=${{ matrix.shard }}/2
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report-${{ matrix.shard }}
          path: playwright-report/

  # All tests passed
  tests-passed:
    needs: [quality, unit, integration, e2e]
    runs-on: ubuntu-latest
    steps:
      - run: echo "All tests passed!"
```

---

## Practice Exercises

### Exercise 1: Basic Test Pipeline

```yaml
# Create a workflow that:
# 1. Runs on PR to main
# 2. Installs dependencies
# 3. Runs lint, type-check, and tests
# 4. Uploads coverage to Codecov
```

### Exercise 2: Integration Tests

```yaml
# Create a workflow with:
# 1. PostgreSQL service container
# 2. Database migrations
# 3. Integration tests
```

### Exercise 3: E2E Pipeline

```yaml
# Create a workflow with:
# 1. Playwright tests in parallel (4 shards)
# 2. Report merging
# 3. Screenshot uploads on failure
```

---

## Quick Reference

### Common Services

```yaml
services:
  postgres:
    image: postgres:14
    env:
      POSTGRES_PASSWORD: test
    ports:
      - 5432:5432

  mysql:
    image: mysql:8
    env:
      MYSQL_ROOT_PASSWORD: test
    ports:
      - 3306:3306

  redis:
    image: redis:7
    ports:
      - 6379:6379

  mongodb:
    image: mongo:6
    ports:
      - 27017:27017
```

### Test Commands

| Test Type | Command |
|-----------|---------|
| Unit | `npm test` |
| Integration | `npm run test:integration` |
| E2E | `npx playwright test` |
| Coverage | `npm test -- --coverage` |
| Watch | `npm test -- --watch` |

---

## Key Takeaways

1. **Run fast checks first** - Lint before tests
2. **Use service containers** for databases
3. **Parallel execution** speeds up tests
4. **Upload artifacts** for debugging failures
5. **Coverage reports** ensure quality
6. **Fail fast** on quality issues

---

## What's Next?

Tomorrow, we'll build a **Deployment Pipeline** - automating deployments to staging and production environments with approval gates and rollback capabilities.
