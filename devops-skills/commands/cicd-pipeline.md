# CI/CD Pipeline Expert

**Description**: Build robust continuous integration and deployment pipelines

## CI/CD Principles

- **Continuous Integration**: Automatically test code changes
- **Continuous Deployment**: Automatically deploy to production
- **Continuous Delivery**: Keep code deployable, manual release

## GitHub Actions

### Basic Workflow
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

### Deploy Workflow
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/myapp:$IMAGE_TAG .
          docker push $ECR_REGISTRY/myapp:$IMAGE_TAG

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster production \
            --service myapp \
            --force-new-deployment
```

### Matrix Testing
```yaml
# Test multiple versions
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        node-version: [16, 18, 20]

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

## GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# Run tests
test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run lint
    - npm test
  coverage: '/Statements\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Build Docker image
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - main

# Deploy to production
deploy-production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - curl -X POST $DEPLOY_WEBHOOK_URL
  environment:
    name: production
    url: https://myapp.com
  only:
    - main
  when: manual  # Require manual approval
```

## Best Practices

### 1. **Fast Feedback**
```yaml
# Run fast checks first
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run lint  # Fast, fails early

  test:
    runs-on: ubuntu-latest
    needs: lint  # Only run if lint passes
    steps:
      - uses: actions/checkout@v3
      - run: npm test
```

### 2. **Caching**
```yaml
# Cache dependencies
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### 3. **Secrets Management**
```yaml
# Use secrets, never hardcode
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  API_KEY: ${{ secrets.API_KEY }}
```

### 4. **Environment-Specific Deploys**
```yaml
# Deploy to different environments
deploy-staging:
  environment:
    name: staging
    url: https://staging.myapp.com
  only:
    - develop

deploy-production:
  environment:
    name: production
    url: https://myapp.com
  only:
    - main
  when: manual  # Require approval
```

### 5. **Rollback Strategy**
```yaml
# Tag releases for easy rollback
- name: Create release tag
  run: |
    git tag v${{ github.run_number }}
    git push origin v${{ github.run_number }}

# Rollback command (manual)
# git checkout v123
# docker pull myapp:v123
```

## Pipeline Checklist

- [ ] Run on every push/PR
- [ ] Fast feedback (< 10 minutes)
- [ ] Test multiple environments
- [ ] Cache dependencies
- [ ] Secrets in vault/secrets manager
- [ ] Notifications on failure
- [ ] Manual approval for production
- [ ] Automated rollback capability
- [ ] Monitor deployment health

## When to Use This Skill

- Setting up CI/CD pipelines
- Automating testing and deployment
- Configuring GitHub Actions or GitLab CI
- Improving build times
- Implementing deployment strategies

---

**Remember**: Good CI/CD gives fast, reliable feedback on every change!
