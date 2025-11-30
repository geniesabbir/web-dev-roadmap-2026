# Day 1: CI/CD Concepts - Automating Your Development Workflow

## Introduction

Continuous Integration and Continuous Deployment (CI/CD) automate the process of building, testing, and deploying code. Instead of manually running tests and deploying, every code change automatically goes through a pipeline that ensures quality and delivers to production. Today, you'll learn the fundamental concepts that power modern software delivery.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand CI/CD principles and benefits
- Identify pipeline stages and components
- Design a CI/CD workflow for web applications
- Choose appropriate tools for your needs
- Plan deployment strategies

---

## What is CI/CD?

### Continuous Integration (CI)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Continuous Integration                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Developer A ──┐                                                │
│                │      ┌──────────┐     ┌──────────┐             │
│  Developer B ──┼────► │   Git    │ ──► │  Build   │             │
│                │      │  Merge   │     │  + Test  │             │
│  Developer C ──┘      └──────────┘     └──────────┘             │
│                                              │                   │
│                                              ▼                   │
│                                        ┌──────────┐             │
│                                        │ Feedback │             │
│                                        │  < 10min │             │
│                                        └──────────┘             │
│                                                                  │
│  Key Principles:                                                │
│  • Merge code frequently (at least daily)                       │
│  • Automated builds on every commit                             │
│  • Automated tests run automatically                            │
│  • Fast feedback (< 10 minutes)                                 │
│  • Fix broken builds immediately                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Continuous Delivery (CD)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Continuous Delivery                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CI Pipeline                                                     │
│      │                                                          │
│      ▼                                                          │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                │
│  │  Build   │ ──► │  Test    │ ──► │  Stage   │                │
│  │ Artifact │     │  Suite   │     │  Deploy  │                │
│  └──────────┘     └──────────┘     └──────────┘                │
│                                          │                      │
│                                          ▼                      │
│                                    ┌──────────┐                 │
│                                    │  Manual  │                 │
│                                    │ Approval │                 │
│                                    └──────────┘                 │
│                                          │                      │
│                                          ▼                      │
│                                    ┌──────────┐                 │
│                                    │Production│                 │
│                                    │  Deploy  │                 │
│                                    └──────────┘                 │
│                                                                  │
│  • Always deployable to production                              │
│  • Manual trigger for production deployment                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Continuous Deployment

```
┌─────────────────────────────────────────────────────────────────┐
│                   Continuous Deployment                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────────┐      │
│  │ Code │ ► │Build │ ► │ Test │ ► │Stage │ ► │Production│      │
│  │ Push │   │      │   │      │   │      │   │  Deploy  │      │
│  └──────┘   └──────┘   └──────┘   └──────┘   └──────────┘      │
│                                                                  │
│  • Every passing build deploys automatically                    │
│  • No manual intervention                                       │
│  • Requires robust test suite                                   │
│  • Feature flags for incomplete features                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Benefits of CI/CD

### For Developers

| Benefit | Description |
|---------|-------------|
| Fast feedback | Know within minutes if code breaks something |
| Reduced risk | Small, frequent changes are easier to debug |
| Less manual work | Automated testing and deployment |
| Confidence | Tests prove code works before deployment |

### For Teams

| Benefit | Description |
|---------|-------------|
| Collaboration | Everyone integrates code frequently |
| Visibility | Pipeline status shows project health |
| Consistency | Same process for every change |
| Documentation | Pipeline as code documents the process |

### For Business

| Benefit | Description |
|---------|-------------|
| Faster delivery | Ship features more frequently |
| Higher quality | Automated testing catches bugs early |
| Lower costs | Less time spent on manual processes |
| Reliability | Consistent, repeatable deployments |

---

## Pipeline Stages

### Typical Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                       CI/CD Pipeline                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. SOURCE          2. BUILD           3. TEST                  │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐              │
│  │ Git Push │  ──► │ Install  │  ──► │  Unit    │              │
│  │ Webhook  │      │  Build   │      │  Lint    │              │
│  └──────────┘      │ Compile  │      │  Type    │              │
│                    └──────────┘      └──────────┘              │
│                                            │                    │
│  6. MONITOR         5. DEPLOY         4. RELEASE               │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐              │
│  │ Metrics  │  ◄── │ Rolling  │  ◄── │ Artifact │              │
│  │ Alerts   │      │ Deploy   │      │ Version  │              │
│  │ Logs     │      │ Verify   │      │ Tag      │              │
│  └──────────┘      └──────────┘      └──────────┘              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Stage Details

#### 1. Source Stage
```yaml
# Triggers pipeline
- Push to branch
- Pull request opened
- Scheduled (cron)
- Manual trigger
```

#### 2. Build Stage
```yaml
# Compile and prepare
- Install dependencies
- Compile TypeScript
- Bundle assets
- Build Docker image
```

#### 3. Test Stage
```yaml
# Quality checks
- Lint code (ESLint)
- Type check (TypeScript)
- Unit tests (Jest)
- Integration tests
- E2E tests (Playwright)
- Security scan
```

#### 4. Release Stage
```yaml
# Package for deployment
- Create artifact
- Tag version
- Push to registry
- Generate changelog
```

#### 5. Deploy Stage
```yaml
# Ship to environment
- Deploy to staging
- Run smoke tests
- Deploy to production
- Verify deployment
```

#### 6. Monitor Stage
```yaml
# Post-deployment
- Check metrics
- Monitor errors
- Alert on issues
- Rollback if needed
```

---

## Pipeline Types

### Branch-Based Pipeline

```
main branch:
  └── Full pipeline → Production

feature/* branches:
  └── CI only → No deployment

develop branch:
  └── Full pipeline → Staging
```

```yaml
# Example: Different pipelines per branch
on:
  push:
    branches:
      - main      # Deploy to production
      - develop   # Deploy to staging
  pull_request:
    branches:
      - main      # CI only
```

### Environment Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    Environment Promotion                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐   │
│  │   Dev   │ ──► │   QA    │ ──► │ Staging │ ──► │  Prod   │   │
│  │         │     │         │     │         │     │         │   │
│  └─────────┘     └─────────┘     └─────────┘     └─────────┘   │
│       │               │               │               │         │
│    Auto           Auto/QA        Manual           Manual        │
│   Deploy          Approve        Approve          Approve       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Deployment Strategies

### Rolling Deployment

```
┌─────────────────────────────────────────────────────────────────┐
│                    Rolling Deployment                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1:  [v1] [v1] [v1] [v1]    All running v1                │
│                                                                  │
│  Step 2:  [v2] [v1] [v1] [v1]    Update one at a time          │
│                                                                  │
│  Step 3:  [v2] [v2] [v1] [v1]    Continue rolling              │
│                                                                  │
│  Step 4:  [v2] [v2] [v2] [v2]    All running v2                │
│                                                                  │
│  Pros: Zero downtime, gradual rollout                           │
│  Cons: Multiple versions running simultaneously                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Blue-Green Deployment

```
┌─────────────────────────────────────────────────────────────────┐
│                   Blue-Green Deployment                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    Load Balancer                                │
│                         │                                        │
│           ┌─────────────┴─────────────┐                         │
│           │                           │                         │
│           ▼                           ▼                         │
│     ┌──────────┐               ┌──────────┐                    │
│     │   Blue   │               │  Green   │                    │
│     │   (v1)   │               │   (v2)   │                    │
│     │  Active  │               │  Standby │                    │
│     └──────────┘               └──────────┘                    │
│                                                                  │
│  1. Deploy v2 to Green (inactive)                               │
│  2. Test Green environment                                      │
│  3. Switch traffic to Green                                     │
│  4. Blue becomes standby (quick rollback)                       │
│                                                                  │
│  Pros: Instant rollback, zero downtime                          │
│  Cons: Double infrastructure cost                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Canary Deployment

```
┌─────────────────────────────────────────────────────────────────┐
│                    Canary Deployment                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: 95% ────► [v1] [v1] [v1] [v1]                         │
│           5% ────► [v2]                   Canary                │
│                                                                  │
│  Step 2: 75% ────► [v1] [v1] [v1]                              │
│          25% ────► [v2]                   Expand                │
│                                                                  │
│  Step 3: 50% ────► [v1] [v1]                                   │
│          50% ────► [v2] [v2]              Continue              │
│                                                                  │
│  Step 4: 100% ───► [v2] [v2] [v2] [v2]    Complete             │
│                                                                  │
│  Pros: Limited blast radius, real user testing                  │
│  Cons: Complex routing, longer rollout                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## CI/CD Tools

### Popular Options

| Tool | Type | Best For |
|------|------|----------|
| GitHub Actions | Cloud | GitHub repositories |
| GitLab CI | Cloud/Self | GitLab repositories |
| Jenkins | Self-hosted | Enterprise, flexibility |
| CircleCI | Cloud | Fast builds, Docker |
| Travis CI | Cloud | Open source projects |
| Azure DevOps | Cloud | Microsoft ecosystem |

### Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                    Tool Comparison                               │
├─────────────────┬───────────────────────────────────────────────┤
│ GitHub Actions  │ ✓ Free for public repos                       │
│                 │ ✓ Tight GitHub integration                    │
│                 │ ✓ Large marketplace                           │
│                 │ ✓ Matrix builds                               │
├─────────────────┼───────────────────────────────────────────────┤
│ GitLab CI       │ ✓ Built into GitLab                          │
│                 │ ✓ Auto DevOps                                 │
│                 │ ✓ Container registry included                 │
│                 │ ✓ Self-hosted option                          │
├─────────────────┼───────────────────────────────────────────────┤
│ Jenkins         │ ✓ Highly customizable                        │
│                 │ ✓ Thousands of plugins                        │
│                 │ ✓ Self-hosted control                         │
│                 │ ✗ More setup/maintenance                      │
└─────────────────┴───────────────────────────────────────────────┘
```

---

## Designing Your Pipeline

### Web Application Pipeline

```yaml
# Typical full-stack application pipeline

stages:
  - lint:
      - ESLint
      - Prettier check
      - TypeScript check

  - test:
      - Unit tests (Jest)
      - Integration tests
      - E2E tests (Playwright)

  - build:
      - Build frontend
      - Build backend
      - Build Docker images

  - security:
      - Dependency audit
      - SAST scan
      - Container scan

  - deploy-staging:
      - Deploy to staging
      - Run smoke tests

  - deploy-production:
      - Manual approval
      - Deploy to production
      - Health check
      - Monitor
```

### Microservices Pipeline

```yaml
# Per-service pipeline

trigger:
  paths:
    - services/user-service/**

stages:
  - test:
      working-directory: services/user-service
      steps:
        - npm test

  - build:
      steps:
        - docker build -t user-service

  - deploy:
      steps:
        - Deploy only user-service
```

---

## Best Practices

### Pipeline Design

```yaml
# 1. Fail fast - run quick checks first
stages:
  - lint        # 30 seconds
  - type-check  # 1 minute
  - unit-test   # 2 minutes
  - build       # 3 minutes
  - e2e-test    # 10 minutes

# 2. Parallelize where possible
jobs:
  lint:
    runs-on: ubuntu-latest
  test:
    runs-on: ubuntu-latest
  # Both run simultaneously

# 3. Cache dependencies
- uses: actions/cache@v3
  with:
    path: node_modules
    key: npm-${{ hashFiles('package-lock.json') }}

# 4. Use artifacts for build outputs
- uses: actions/upload-artifact@v3
  with:
    name: build
    path: dist/
```

### Security

```yaml
# 1. Never commit secrets
# Use secret management
env:
  API_KEY: ${{ secrets.API_KEY }}

# 2. Pin action versions
uses: actions/checkout@v4  # Not @latest

# 3. Scan for vulnerabilities
- run: npm audit
- run: trivy image myapp

# 4. Limit permissions
permissions:
  contents: read
  packages: write
```

### Reliability

```yaml
# 1. Idempotent deployments
# Running twice should have same result

# 2. Health checks after deploy
- run: curl --fail https://myapp.com/health

# 3. Automatic rollback
- if: failure()
  run: ./rollback.sh

# 4. Notifications
- if: failure()
  uses: slack-notify-action
```

---

## Practice Exercises

### Exercise 1: Design a Pipeline

```markdown
Design a CI/CD pipeline for a Next.js application with:
- Frontend (React)
- API routes
- PostgreSQL database
- Deployed to Vercel

Consider:
- What stages are needed?
- What tests should run?
- How will you handle database migrations?
- What's your deployment strategy?
```

### Exercise 2: Compare Strategies

```markdown
Compare deployment strategies for:
1. A high-traffic e-commerce site
2. An internal admin dashboard
3. A mobile app backend

Which strategy would you choose for each and why?
```

### Exercise 3: Pipeline Optimization

```markdown
Given a pipeline that takes 30 minutes:
- Lint: 2 min
- Unit tests: 8 min
- Build: 5 min
- E2E tests: 15 min

How would you optimize it to run faster?
```

---

## Quick Reference

### Pipeline Stages

| Stage | Purpose | Duration |
|-------|---------|----------|
| Lint | Code quality | < 1 min |
| Test | Verify behavior | 2-10 min |
| Build | Create artifact | 2-5 min |
| Security | Vulnerability scan | 1-5 min |
| Deploy | Ship to environment | 2-10 min |

### Deployment Strategies

| Strategy | Downtime | Rollback | Complexity |
|----------|----------|----------|------------|
| Rolling | None | Medium | Low |
| Blue-Green | None | Instant | Medium |
| Canary | None | Easy | High |
| Recreate | Yes | Deploy old | Low |

---

## Key Takeaways

1. **CI catches bugs early** - Automated testing on every commit
2. **CD reduces deployment risk** - Small, frequent releases
3. **Fail fast** - Run quick checks first
4. **Parallelize** - Run independent jobs simultaneously
5. **Automate everything** - Consistent, repeatable process
6. **Monitor deployments** - Catch issues quickly

---

## What's Next?

Tomorrow, we'll dive into **GitHub Actions** - writing workflows, understanding syntax, and building your first automated pipeline.
