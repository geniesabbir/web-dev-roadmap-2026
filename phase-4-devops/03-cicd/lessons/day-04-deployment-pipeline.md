# Day 4: Deployment Pipeline - Shipping Code to Production

## Introduction

A deployment pipeline automates the process of getting your code from repository to production. It should be reliable, fast, and safe—with the ability to roll back if something goes wrong. Today, you'll learn to build deployment pipelines for various platforms including Vercel, Railway, and AWS.

## Learning Objectives

By the end of this lesson, you will be able to:
- Deploy to multiple environments (staging, production)
- Implement approval gates and manual triggers
- Deploy to Vercel, Railway, and AWS
- Handle environment-specific configurations
- Implement rollback strategies

---

## Deployment Pipeline Architecture

### Multi-Environment Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    Deployment Pipeline                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  develop branch              main branch                        │
│       │                           │                              │
│       ▼                           ▼                              │
│  ┌─────────┐                ┌─────────┐                         │
│  │  Build  │                │  Build  │                         │
│  │  Test   │                │  Test   │                         │
│  └────┬────┘                └────┬────┘                         │
│       │                          │                               │
│       ▼                          ▼                               │
│  ┌─────────┐                ┌─────────┐     ┌─────────┐        │
│  │ Staging │                │ Staging │ ──► │  Prod   │        │
│  │  Auto   │                │  Auto   │     │ Manual  │        │
│  └─────────┘                └─────────┘     └─────────┘        │
│                                  │               │               │
│                                  └───────────────┘               │
│                                    Approval Gate                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## GitHub Environments

### Setting Up Environments

```yaml
# Repository Settings > Environments

# Create environments:
# 1. staging
# 2. production (with protection rules)

# Protection rules for production:
# - Required reviewers: 1
# - Wait timer: 5 minutes
# - Restrict to main branch
```

### Using Environments in Workflow

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com

    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh staging

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production
      url: https://example.com

    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh production
```

---

## Deploying to Vercel

### Automatic Deployment

```yaml
# Vercel automatically deploys on push
# Just connect your GitHub repo in Vercel dashboard

# For custom workflow control:
name: Deploy to Vercel

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### Preview Deployments

```yaml
name: Vercel Preview

on:
  pull_request:
    branches: [main]

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy Preview
        uses: amondnet/vercel-action@v25
        id: deploy
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          # No --prod flag = preview deployment

      - name: Comment PR
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚀 Preview deployed to: ${{ steps.deploy.outputs.preview-url }}'
            })
```

### Vercel with Environment Variables

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Vercel CLI
        run: npm install -g vercel

      - name: Pull Vercel Environment
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy
        run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}
```

---

## Deploying to Railway

### Railway Deployment

```yaml
name: Deploy to Railway

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Railway CLI
        run: npm install -g @railway/cli

      - name: Deploy
        run: railway up --service ${{ secrets.RAILWAY_SERVICE_ID }}
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
```

### Railway with Docker

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Railway Registry
        run: echo ${{ secrets.RAILWAY_TOKEN }} | docker login -u railway --password-stdin registry.railway.app

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: registry.railway.app/${{ secrets.RAILWAY_PROJECT_ID }}:latest

      - name: Deploy
        run: |
          curl -X POST \
            -H "Authorization: Bearer ${{ secrets.RAILWAY_TOKEN }}" \
            "https://backboard.railway.app/graphql/v2" \
            -d '{"query": "mutation { deploymentRedeploy(id: \"${{ secrets.RAILWAY_DEPLOYMENT_ID }}\") { id } }"}'
```

---

## Deploying to AWS

### AWS S3 + CloudFront (Static Sites)

```yaml
name: Deploy to AWS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to S3
        run: aws s3 sync ./dist s3://${{ secrets.S3_BUCKET }} --delete

      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

### AWS ECS (Docker)

```yaml
name: Deploy to ECS

on:
  push:
    branches: [main]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: myapp
  ECS_SERVICE: myapp-service
  ECS_CLUSTER: myapp-cluster
  ECS_TASK_DEFINITION: .aws/task-definition.json
  CONTAINER_NAME: myapp

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Update ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: ${{ env.ECS_TASK_DEFINITION }}
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}

      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

### AWS Lambda

```yaml
name: Deploy Lambda

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '18'

      - run: npm ci --production

      - name: Zip function
        run: zip -r function.zip . -x "*.git*"

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to Lambda
        run: |
          aws lambda update-function-code \
            --function-name myfunction \
            --zip-file fileb://function.zip
```

---

## Docker Deployment

### Build and Push to Registry

```yaml
name: Docker Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            username/myapp:latest
            username/myapp:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull username/myapp:latest
            docker stop myapp || true
            docker rm myapp || true
            docker run -d --name myapp -p 80:3000 username/myapp:latest
```

### GitHub Container Registry

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

---

## Staged Deployments

### Complete Pipeline with Stages

```yaml
name: Deploy Pipeline

on:
  push:
    branches: [main]
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

jobs:
  # Build and test
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - run: npm ci
      - run: npm test
      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/

  # Deploy to staging
  staging:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com

    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/

      - name: Deploy to staging
        run: |
          echo "Deploying to staging..."
          # Add deployment commands

      - name: Smoke tests
        run: |
          curl --fail https://staging.example.com/health

  # Deploy to production
  production:
    needs: staging
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com

    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/

      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # Add deployment commands

      - name: Health check
        run: |
          curl --fail https://example.com/health

      - name: Notify success
        if: success()
        run: |
          echo "Deployment successful!"

      - name: Notify failure
        if: failure()
        run: |
          echo "Deployment failed!"
```

---

## Rollback Strategies

### Manual Rollback Workflow

```yaml
name: Rollback

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to rollback to'
        required: true
      environment:
        description: 'Environment'
        required: true
        type: choice
        options:
          - staging
          - production

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}

    steps:
      - name: Rollback deployment
        run: |
          echo "Rolling back to version ${{ inputs.version }}"
          # Example: Vercel rollback
          # vercel rollback ${{ inputs.version }}

          # Example: Docker rollback
          # docker pull myapp:${{ inputs.version }}
          # docker stop myapp && docker rm myapp
          # docker run -d --name myapp myapp:${{ inputs.version }}
```

### Automatic Rollback on Failure

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        id: deploy
        run: ./deploy.sh

      - name: Health check
        id: health
        run: |
          for i in {1..5}; do
            if curl --fail https://example.com/health; then
              exit 0
            fi
            sleep 10
          done
          exit 1

      - name: Rollback on failure
        if: failure() && steps.deploy.outcome == 'success'
        run: |
          echo "Health check failed, rolling back..."
          ./rollback.sh
```

---

## Database Migrations

### Migrations in Deployment

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Deploy application
        run: ./deploy.sh

      # If deploy fails, migrations are already applied
      # Consider migration rollback strategy
```

### Safe Migration Strategy

```yaml
jobs:
  # Run migrations separately
  migrate:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - name: Run migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

  # Deploy after migrations succeed
  deploy:
    needs: migrate
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh
```

---

## Practice Exercises

### Exercise 1: Vercel Deployment

```yaml
# Create a workflow that:
# 1. Builds a Next.js app
# 2. Deploys preview on PR
# 3. Deploys to production on merge to main
# 4. Comments preview URL on PR
```

### Exercise 2: Docker Deployment

```yaml
# Create a workflow that:
# 1. Builds Docker image
# 2. Pushes to GitHub Container Registry
# 3. Deploys to server via SSH
# 4. Runs health check
```

### Exercise 3: Staged Deployment

```yaml
# Create a workflow with:
# 1. Build job
# 2. Deploy to staging (automatic)
# 3. Smoke tests on staging
# 4. Deploy to production (manual approval)
# 5. Health check and rollback on failure
```

---

## Quick Reference

### Deployment Actions

| Platform | Action |
|----------|--------|
| Vercel | `amondnet/vercel-action@v25` |
| AWS | `aws-actions/configure-aws-credentials@v4` |
| Docker | `docker/build-push-action@v5` |
| SSH | `appleboy/ssh-action@master` |

### Environment Variables

```yaml
# In workflow
env:
  NODE_ENV: production

# In job
jobs:
  deploy:
    env:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}

# From GitHub Environments
environment:
  name: production
# Secrets defined in environment settings
```

---

## Key Takeaways

1. **Use environments** for staged deployments
2. **Add approval gates** for production
3. **Health checks** verify successful deploys
4. **Automatic rollback** on failures
5. **Version your deployments** for easy rollback
6. **Separate concerns** - migrations before deploy

---

## What's Next?

Tomorrow, we'll explore **Advanced Workflows** - reusable workflows, composite actions, matrix strategies, and workflow optimization techniques.
