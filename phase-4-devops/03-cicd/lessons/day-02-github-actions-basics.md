# Day 2: GitHub Actions Basics - Building Your First Workflows

## Introduction

GitHub Actions is a powerful CI/CD platform built directly into GitHub. It allows you to automate builds, tests, and deployments right from your repository. Today, you'll learn the fundamentals of GitHub Actions, understand the workflow syntax, and create your first automated pipelines.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand GitHub Actions components
- Write workflow YAML files
- Use actions from the marketplace
- Configure triggers and conditions
- Manage secrets and environment variables

---

## GitHub Actions Components

### Core Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Actions Architecture                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Workflow (.github/workflows/*.yml)                             │
│  └── Event (trigger)                                            │
│      └── Job 1 (runs on runner)                                 │
│          └── Step 1 (action or script)                          │
│          └── Step 2                                             │
│          └── Step 3                                             │
│      └── Job 2 (can run in parallel)                            │
│          └── Step 1                                             │
│          └── Step 2                                             │
│                                                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐                │
│  │  Workflow  │  │    Job     │  │    Step    │                │
│  │            │  │            │  │            │                │
│  │ Container  │  │  Runs on   │  │  Action or │                │
│  │ for all    │  │  a runner  │  │  Command   │                │
│  │ automation │  │  machine   │  │            │                │
│  └────────────┘  └────────────┘  └────────────┘                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Terminology

| Component | Description |
|-----------|-------------|
| **Workflow** | Automated process defined in YAML |
| **Event** | Trigger that starts the workflow |
| **Job** | Set of steps running on same runner |
| **Step** | Individual task in a job |
| **Action** | Reusable unit of code |
| **Runner** | Server that executes workflows |

---

## Your First Workflow

### Basic Structure

```yaml
# .github/workflows/ci.yml

name: CI Pipeline

# When to run
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

# What to run
jobs:
  build:
    # Runner environment
    runs-on: ubuntu-latest

    # Job steps
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build
```

### File Location

```
your-repo/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy.yml
│       └── release.yml
├── src/
├── package.json
└── README.md
```

---

## Workflow Triggers (Events)

### Common Events

```yaml
# Push to branch
on:
  push:
    branches:
      - main
      - 'feature/*'
    paths:
      - 'src/**'
      - 'package.json'

# Pull request
on:
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

# Manual trigger
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

# Scheduled (cron)
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC

# Multiple events
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
```

### Event Filters

```yaml
# Only on specific files
on:
  push:
    paths:
      - '**.js'
      - '**.ts'
      - 'package*.json'
    paths-ignore:
      - '**.md'
      - 'docs/**'

# Only on tags
on:
  push:
    tags:
      - 'v*'

# Release events
on:
  release:
    types: [published]
```

---

## Jobs Configuration

### Single Job

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

### Multiple Jobs

```yaml
jobs:
  # Jobs run in parallel by default
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test

  build:
    runs-on: ubuntu-latest
    # Wait for other jobs
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - run: npm run build
```

### Job Dependencies

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  build:
    needs: test  # Runs after test
    runs-on: ubuntu-latest
    steps:
      - run: npm run build

  deploy:
    needs: build  # Runs after build
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

### Conditional Jobs

```yaml
jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy-staging.sh

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy-production.sh
```

---

## Steps Configuration

### Using Actions

```yaml
steps:
  # Official GitHub action
  - name: Checkout
    uses: actions/checkout@v4

  # Action with inputs
  - name: Setup Node
    uses: actions/setup-node@v4
    with:
      node-version: '18'
      cache: 'npm'

  # Third-party action
  - name: Deploy to Vercel
    uses: amondnet/vercel-action@v25
    with:
      vercel-token: ${{ secrets.VERCEL_TOKEN }}
```

### Running Commands

```yaml
steps:
  # Single command
  - run: npm install

  # Multiple commands
  - run: |
      npm install
      npm run build
      npm test

  # With name
  - name: Install and build
    run: |
      npm ci
      npm run build

  # With working directory
  - name: Build frontend
    working-directory: ./frontend
    run: npm run build

  # With environment variables
  - name: Run tests
    run: npm test
    env:
      NODE_ENV: test
      CI: true
```

### Conditional Steps

```yaml
steps:
  - name: Deploy
    if: github.ref == 'refs/heads/main'
    run: ./deploy.sh

  - name: Notify on failure
    if: failure()
    run: ./notify-slack.sh

  - name: Cleanup
    if: always()
    run: ./cleanup.sh

  - name: Skip on draft PR
    if: github.event.pull_request.draft == false
    run: npm test
```

---

## Environment Variables and Secrets

### Environment Variables

```yaml
# Workflow level
env:
  NODE_ENV: production
  CI: true

jobs:
  build:
    runs-on: ubuntu-latest
    # Job level
    env:
      DATABASE_URL: postgres://localhost/test

    steps:
      - name: Build
        # Step level
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: npm run build
```

### Using Secrets

```yaml
# Repository secrets (Settings > Secrets)
steps:
  - name: Deploy
    env:
      AWS_ACCESS_KEY: ${{ secrets.AWS_ACCESS_KEY }}
      AWS_SECRET_KEY: ${{ secrets.AWS_SECRET_KEY }}
    run: ./deploy.sh

  - name: Push to registry
    run: |
      echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
      docker push myapp:latest
```

### GitHub Context

```yaml
steps:
  - name: Show context
    run: |
      echo "Repository: ${{ github.repository }}"
      echo "Branch: ${{ github.ref_name }}"
      echo "SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      echo "Event: ${{ github.event_name }}"
      echo "Run ID: ${{ github.run_id }}"
```

---

## Runners

### GitHub-Hosted Runners

```yaml
jobs:
  ubuntu:
    runs-on: ubuntu-latest  # Ubuntu 22.04

  ubuntu-specific:
    runs-on: ubuntu-22.04

  macos:
    runs-on: macos-latest

  windows:
    runs-on: windows-latest
```

### Matrix Builds

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
        os: [ubuntu-latest, windows-latest]

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

### Matrix with Include/Exclude

```yaml
strategy:
  matrix:
    node: [16, 18, 20]
    os: [ubuntu-latest, windows-latest]
    exclude:
      - node: 16
        os: windows-latest
    include:
      - node: 20
        os: ubuntu-latest
        experimental: true
```

---

## Caching

### Cache Dependencies

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Cache node modules
    uses: actions/cache@v3
    with:
      path: node_modules
      key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-

  - name: Install dependencies
    run: npm ci
```

### Setup Action with Cache

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: actions/setup-node@v4
    with:
      node-version: '18'
      cache: 'npm'  # Built-in caching

  - run: npm ci
```

### Cache Multiple Paths

```yaml
- uses: actions/cache@v3
  with:
    path: |
      node_modules
      ~/.npm
      .next/cache
    key: ${{ runner.os }}-modules-${{ hashFiles('**/package-lock.json') }}
```

---

## Artifacts

### Upload Artifacts

```yaml
steps:
  - name: Build
    run: npm run build

  - name: Upload build
    uses: actions/upload-artifact@v4
    with:
      name: build-files
      path: dist/
      retention-days: 5
```

### Download Artifacts

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/
      - run: ./deploy.sh
```

---

## Complete Workflow Example

### Full CI/CD Pipeline

```yaml
# .github/workflows/ci-cd.yml

name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'

jobs:
  # Lint job
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npm run lint

  # Test job
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npm test -- --coverage

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  # Build job
  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/

  # Deploy staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/

      - name: Deploy
        run: echo "Deploying to staging..."
        env:
          DEPLOY_TOKEN: ${{ secrets.STAGING_DEPLOY_TOKEN }}

  # Deploy production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/

      - name: Deploy
        run: echo "Deploying to production..."
        env:
          DEPLOY_TOKEN: ${{ secrets.PROD_DEPLOY_TOKEN }}
```

---

## Practice Exercises

### Exercise 1: Basic Workflow

```yaml
# Create a workflow that:
# 1. Triggers on push to main
# 2. Sets up Node.js
# 3. Installs dependencies
# 4. Runs tests
```

### Exercise 2: Matrix Testing

```yaml
# Create a workflow that:
# 1. Tests on Node 16, 18, and 20
# 2. Tests on Ubuntu and Windows
# 3. Caches dependencies
```

### Exercise 3: Multi-Job Pipeline

```yaml
# Create a workflow with:
# 1. Lint job
# 2. Test job (parallel with lint)
# 3. Build job (after lint and test)
# 4. Deploy job (after build, only on main)
```

---

## Quick Reference

### Workflow Syntax

```yaml
name: Workflow Name
on: [push, pull_request]
env:
  GLOBAL_VAR: value
jobs:
  job-name:
    runs-on: ubuntu-latest
    env:
      JOB_VAR: value
    steps:
      - uses: action/name@v1
        with:
          input: value
      - run: command
        env:
          STEP_VAR: value
```

### Common Actions

| Action | Purpose |
|--------|---------|
| `actions/checkout@v4` | Clone repository |
| `actions/setup-node@v4` | Setup Node.js |
| `actions/cache@v3` | Cache files |
| `actions/upload-artifact@v4` | Upload files |
| `actions/download-artifact@v4` | Download files |

### Context Variables

| Variable | Description |
|----------|-------------|
| `github.repository` | owner/repo |
| `github.ref` | refs/heads/branch |
| `github.sha` | Commit SHA |
| `github.actor` | User who triggered |
| `secrets.NAME` | Secret value |

---

## Key Takeaways

1. **Workflows live in `.github/workflows/`**
2. **YAML defines automation** - triggers, jobs, steps
3. **Jobs run in parallel** by default
4. **Use `needs` for dependencies** between jobs
5. **Cache dependencies** for faster builds
6. **Use secrets** for sensitive data

---

## What's Next?

Tomorrow, we'll build a **Testing Pipeline** - setting up automated testing with coverage reports, parallel test execution, and test result reporting.
