# Day 5: Advanced Workflows - Reusable Patterns and Optimization

## Introduction

As your CI/CD pipelines grow, you'll need advanced patterns to keep them maintainable, efficient, and DRY (Don't Repeat Yourself). Today, you'll learn to create reusable workflows, composite actions, optimize pipeline performance, and implement advanced patterns used in production environments.

## Learning Objectives

By the end of this lesson, you will be able to:
- Create reusable workflows
- Build composite actions
- Optimize workflow performance
- Implement advanced triggering patterns
- Monitor and debug workflows

---

## Reusable Workflows

### Creating a Reusable Workflow

```yaml
# .github/workflows/reusable-build.yml

name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js version'
        required: false
        type: string
        default: '18'
      build-command:
        description: 'Build command'
        required: false
        type: string
        default: 'npm run build'
    secrets:
      npm-token:
        description: 'NPM token for private packages'
        required: false
    outputs:
      artifact-name:
        description: 'Name of the uploaded artifact'
        value: ${{ jobs.build.outputs.artifact-name }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-name: ${{ steps.upload.outputs.artifact-name }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: npm ci
        env:
          NODE_AUTH_TOKEN: ${{ secrets.npm-token }}

      - name: Build
        run: ${{ inputs.build-command }}

      - name: Upload artifact
        id: upload
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ github.sha }}
          path: dist/

      - name: Set output
        run: echo "artifact-name=build-${{ github.sha }}" >> $GITHUB_OUTPUT
```

### Calling a Reusable Workflow

```yaml
# .github/workflows/ci.yml

name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
      build-command: 'npm run build:prod'
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: ${{ needs.build.outputs.artifact-name }}

      - name: Deploy
        run: echo "Deploying..."
```

### Reusable Workflow from Another Repository

```yaml
jobs:
  build:
    uses: organization/shared-workflows/.github/workflows/node-build.yml@main
    with:
      node-version: '18'
    secrets: inherit  # Pass all secrets
```

---

## Composite Actions

### Creating a Composite Action

```yaml
# .github/actions/setup-project/action.yml

name: 'Setup Project'
description: 'Setup Node.js project with caching'

inputs:
  node-version:
    description: 'Node.js version'
    required: false
    default: '18'
  install-command:
    description: 'Install command'
    required: false
    default: 'npm ci'

outputs:
  cache-hit:
    description: 'Whether cache was hit'
    value: ${{ steps.cache.outputs.cache-hit }}

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}

    - name: Get npm cache directory
      id: npm-cache-dir
      shell: bash
      run: echo "dir=$(npm config get cache)" >> $GITHUB_OUTPUT

    - name: Cache dependencies
      id: cache
      uses: actions/cache@v3
      with:
        path: |
          ${{ steps.npm-cache-dir.outputs.dir }}
          node_modules
        key: ${{ runner.os }}-node-${{ inputs.node-version }}-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-${{ inputs.node-version }}-

    - name: Install dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      shell: bash
      run: ${{ inputs.install-command }}
```

### Using Composite Action

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup project
        uses: ./.github/actions/setup-project
        with:
          node-version: '20'

      - run: npm test
      - run: npm run build
```

### Complex Composite Action

```yaml
# .github/actions/deploy-vercel/action.yml

name: 'Deploy to Vercel'
description: 'Deploy application to Vercel'

inputs:
  vercel-token:
    description: 'Vercel token'
    required: true
  vercel-org-id:
    description: 'Vercel organization ID'
    required: true
  vercel-project-id:
    description: 'Vercel project ID'
    required: true
  production:
    description: 'Deploy to production'
    required: false
    default: 'false'

outputs:
  url:
    description: 'Deployment URL'
    value: ${{ steps.deploy.outputs.url }}

runs:
  using: 'composite'
  steps:
    - name: Install Vercel CLI
      shell: bash
      run: npm install -g vercel

    - name: Pull Vercel Environment
      shell: bash
      run: |
        vercel pull --yes --token=${{ inputs.vercel-token }}
      env:
        VERCEL_ORG_ID: ${{ inputs.vercel-org-id }}
        VERCEL_PROJECT_ID: ${{ inputs.vercel-project-id }}

    - name: Build
      shell: bash
      run: |
        if [ "${{ inputs.production }}" = "true" ]; then
          vercel build --prod --token=${{ inputs.vercel-token }}
        else
          vercel build --token=${{ inputs.vercel-token }}
        fi
      env:
        VERCEL_ORG_ID: ${{ inputs.vercel-org-id }}
        VERCEL_PROJECT_ID: ${{ inputs.vercel-project-id }}

    - name: Deploy
      id: deploy
      shell: bash
      run: |
        if [ "${{ inputs.production }}" = "true" ]; then
          url=$(vercel deploy --prebuilt --prod --token=${{ inputs.vercel-token }})
        else
          url=$(vercel deploy --prebuilt --token=${{ inputs.vercel-token }})
        fi
        echo "url=$url" >> $GITHUB_OUTPUT
      env:
        VERCEL_ORG_ID: ${{ inputs.vercel-org-id }}
        VERCEL_PROJECT_ID: ${{ inputs.vercel-project-id }}
```

---

## Advanced Matrix Strategies

### Dynamic Matrix

```yaml
jobs:
  prepare:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - uses: actions/checkout@v4
      - id: set-matrix
        run: |
          # Generate matrix from changed files
          changed=$(git diff --name-only HEAD~1 | grep "^packages/" | cut -d/ -f2 | sort -u | jq -R -s -c 'split("\n")[:-1]')
          echo "matrix={\"package\":$changed}" >> $GITHUB_OUTPUT

  build:
    needs: prepare
    if: needs.prepare.outputs.matrix != '{"package":[]}'
    strategy:
      matrix: ${{ fromJson(needs.prepare.outputs.matrix) }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build --workspace=packages/${{ matrix.package }}
```

### Matrix with Fail-Fast Control

```yaml
strategy:
  fail-fast: false  # Don't cancel other jobs if one fails
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [16, 18, 20]
    exclude:
      - os: windows-latest
        node: 16
    include:
      - os: ubuntu-latest
        node: 20
        coverage: true
```

### Matrix with Max Parallel

```yaml
strategy:
  max-parallel: 2  # Only 2 jobs at a time
  matrix:
    environment: [dev, staging, prod]
```

---

## Workflow Optimization

### Caching Strategies

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Cache npm
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      # Cache Next.js build
      - name: Cache Next.js
        uses: actions/cache@v3
        with:
          path: |
            ${{ github.workspace }}/.next/cache
          key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
          restore-keys: |
            ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-

      # Cache Playwright browsers
      - name: Cache Playwright
        uses: actions/cache@v3
        with:
          path: ~/.cache/ms-playwright
          key: ${{ runner.os }}-playwright-${{ hashFiles('**/package-lock.json') }}

      - run: npm ci
      - run: npm run build
```

### Concurrency Control

```yaml
# Cancel in-progress runs for same PR
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# Don't cancel production deployments
concurrency:
  group: production-deploy
  cancel-in-progress: false
```

### Conditional Job Execution

```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      backend: ${{ steps.filter.outputs.backend }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            frontend:
              - 'frontend/**'
            backend:
              - 'backend/**'

  frontend-tests:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running frontend tests"

  backend-tests:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running backend tests"
```

---

## Advanced Triggers

### Workflow Dispatch with Inputs

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: choice
        options:
          - development
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: true
        type: string
      skip-tests:
        description: 'Skip tests'
        required: false
        type: boolean
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Deploying ${{ inputs.version }} to ${{ inputs.environment }}"
          echo "Skip tests: ${{ inputs.skip-tests }}"
```

### Repository Dispatch

```yaml
# Triggered by external systems
on:
  repository_dispatch:
    types: [deploy, rollback]

jobs:
  handle-event:
    runs-on: ubuntu-latest
    steps:
      - name: Handle deploy
        if: github.event.action == 'deploy'
        run: |
          echo "Version: ${{ github.event.client_payload.version }}"
          echo "Environment: ${{ github.event.client_payload.environment }}"

      - name: Handle rollback
        if: github.event.action == 'rollback'
        run: |
          echo "Rolling back to: ${{ github.event.client_payload.version }}"
```

```bash
# Trigger via API
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"deploy","client_payload":{"version":"1.2.3","environment":"production"}}'
```

### Workflow Chaining

```yaml
# .github/workflows/ci.yml
name: CI
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

# .github/workflows/cd.yml
name: CD
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
    branches: [main]

jobs:
  deploy:
    if: github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

---

## Monitoring and Debugging

### Workflow Annotations

```yaml
steps:
  - name: Check code quality
    run: |
      # Create warning
      echo "::warning file=app.js,line=10::Consider refactoring this function"

      # Create error
      echo "::error file=app.js,line=20::Missing error handling"

      # Create notice
      echo "::notice::Build completed successfully"

      # Group output
      echo "::group::Test Results"
      npm test
      echo "::endgroup::"
```

### Debug Logging

```yaml
# Enable debug logging
# Set secret: ACTIONS_RUNNER_DEBUG = true
# Set secret: ACTIONS_STEP_DEBUG = true

steps:
  - name: Debug info
    run: |
      echo "GitHub context:"
      echo '${{ toJson(github) }}'

      echo "Runner context:"
      echo '${{ toJson(runner) }}'

      echo "Environment:"
      env | sort
```

### Slack Notifications

```yaml
jobs:
  notify:
    runs-on: ubuntu-latest
    if: always()
    needs: [build, test, deploy]
    steps:
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          fields: repo,message,commit,author,action,eventName,ref,workflow
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

### Job Summary

```yaml
steps:
  - name: Generate summary
    run: |
      echo "## Build Summary" >> $GITHUB_STEP_SUMMARY
      echo "" >> $GITHUB_STEP_SUMMARY
      echo "| Metric | Value |" >> $GITHUB_STEP_SUMMARY
      echo "|--------|-------|" >> $GITHUB_STEP_SUMMARY
      echo "| Tests | 150 passed |" >> $GITHUB_STEP_SUMMARY
      echo "| Coverage | 85% |" >> $GITHUB_STEP_SUMMARY
      echo "| Build Time | 2m 30s |" >> $GITHUB_STEP_SUMMARY
```

---

## Complete Advanced Pipeline

```yaml
# .github/workflows/complete-pipeline.yml

name: Complete Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      skip-tests:
        type: boolean
        default: false

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Detect changes
  changes:
    runs-on: ubuntu-latest
    outputs:
      src: ${{ steps.filter.outputs.src }}
      docs: ${{ steps.filter.outputs.docs }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            src:
              - 'src/**'
              - 'package.json'
            docs:
              - 'docs/**'

  # Quality checks
  quality:
    needs: changes
    if: needs.changes.outputs.src == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-project
      - run: npm run lint
      - run: npm run type-check

  # Tests (parallel)
  test:
    needs: [changes, quality]
    if: needs.changes.outputs.src == 'true' && !inputs.skip-tests
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3]
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-project
      - run: npm test -- --shard=${{ matrix.shard }}/3

  # Build
  build:
    needs: [quality, test]
    if: always() && needs.quality.result == 'success' && (needs.test.result == 'success' || needs.test.result == 'skipped')
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4

      - uses: ./.github/actions/setup-project

      - run: npm run build

      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha
            type=ref,event=branch

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Deploy staging
  staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: ./.github/actions/deploy-vercel
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}

  # Deploy production
  production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: ./.github/actions/deploy-vercel
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          production: 'true'

  # Summary
  summary:
    needs: [quality, test, build, staging, production]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Generate summary
        run: |
          echo "## Pipeline Summary" >> $GITHUB_STEP_SUMMARY
          echo "| Job | Status |" >> $GITHUB_STEP_SUMMARY
          echo "|-----|--------|" >> $GITHUB_STEP_SUMMARY
          echo "| Quality | ${{ needs.quality.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Tests | ${{ needs.test.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Build | ${{ needs.build.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Staging | ${{ needs.staging.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Production | ${{ needs.production.result }} |" >> $GITHUB_STEP_SUMMARY
```

---

## Practice Exercises

### Exercise 1: Reusable Workflow

```yaml
# Create a reusable workflow for:
# - Node.js setup
# - Dependency installation
# - Testing with coverage
# - Build artifact creation
```

### Exercise 2: Composite Action

```yaml
# Create a composite action that:
# - Sets up the project
# - Runs security audit
# - Checks for outdated packages
# - Reports findings
```

### Exercise 3: Monorepo Pipeline

```yaml
# Create a pipeline for a monorepo that:
# - Detects changed packages
# - Only builds/tests affected packages
# - Deploys only changed services
```

---

## Quick Reference

### Reusable Workflow Syntax

```yaml
on:
  workflow_call:
    inputs:
      name:
        type: string
    secrets:
      token:
        required: true
    outputs:
      result:
        value: ${{ jobs.job.outputs.result }}
```

### Composite Action Syntax

```yaml
runs:
  using: 'composite'
  steps:
    - run: echo "Hello"
      shell: bash
```

### Useful Contexts

| Context | Description |
|---------|-------------|
| `github` | Workflow run info |
| `env` | Environment variables |
| `job` | Current job info |
| `steps` | Step outputs |
| `runner` | Runner info |
| `secrets` | Secret values |
| `needs` | Dependent job outputs |

---

## Key Takeaways

1. **Reusable workflows** - DRY across repositories
2. **Composite actions** - Encapsulate complex steps
3. **Smart caching** - Dramatically speeds up builds
4. **Concurrency control** - Prevents resource waste
5. **Conditional execution** - Only run what's needed
6. **Monitor everything** - Summaries, notifications, logs

---

## Congratulations!

You've completed the CI/CD section! You now know how to:
- Understand CI/CD principles and pipelines
- Write GitHub Actions workflows
- Build comprehensive testing pipelines
- Deploy to various platforms
- Create reusable, optimized workflows

Next up is **Cloud Deployment** - deploying to Vercel, Railway, AWS, and monitoring your applications!
