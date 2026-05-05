---
name: ci-cd-patterns
description: Generic CI/CD pipeline patterns. Covers build stages, testing strategies, deployment patterns, artifact management, and integration with GitHub Actions, GitLab CI, Jenkins. Use when setting up CI/CD, designing pipeline architecture, creating deployment workflows, or when the user mentions CI/CD, pipeline, GitHub Actions, GitLab CI, Jenkins, or deployment automation.
---

# CI/CD Patterns Skill

Design and implement robust CI/CD pipelines across platforms.

## When to Use
- Setting up CI/CD for a new project
- Designing pipeline architecture
- Migrating between CI platforms
- Adding new stages to existing pipeline
- When the user mentions CI/CD, pipelines, or deployment automation

## Pipeline Stages

### Typical CI/CD Flow
```
┌─────────────┐
│   Commit    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Lint     │ ← Fast feedback
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Test     │ ← Unit, integration
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Build    │ ← Compile, package
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Scan     │ ← Security, vulnerabilities
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Deploy Stg │ ← Staging environment
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Tests    │ ← E2E, smoke tests
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Deploy Prd │ ← Production (manual approval?)
└─────────────┘
```

## GitHub Actions

### Basic Workflow
```yaml
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
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
```

### Multi-Stage Pipeline
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint

  test:
    needs: lint
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
      - run: ./deploy.sh
```

## GitLab CI

### Basic `.gitlab-ci.yml`
```yaml
stages:
  - lint
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"

lint:
  stage: lint
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run lint

test:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm test
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

## Jenkins Pipeline

### Declarative Pipeline
```groovy
pipeline {
    agent any
    
    stages {
        stage('Lint') {
            steps {
                sh 'npm ci'
                sh 'npm run lint'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results.xml'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }
    
    post {
        failure {
            mail to: 'team@example.com',
                 subject: "Build failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                 body: "Check console output at ${env.BUILD_URL}"
        }
    }
}
```

## Testing Strategies

### Parallel Testing
```yaml
# GitHub Actions - split tests across multiple jobs
test:
  strategy:
    matrix:
      shard: [1, 2, 3, 4]
  steps:
    - run: npm test -- --shard=${{ matrix.shard }}/4
```

### E2E Tests
```yaml
e2e:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - run: npm ci
    - run: npm run build
    - run: npm run start &
    - run: npm run test:e2e  # Playwright, Cypress, etc.
```

## Deployment Patterns

### Blue-Green Deployment
```
Blue (Live)    Green (Idle)
    ↓              ↓
[Current]     [New Version]
                  ↓
              Switch traffic
                  ↓
[Idle]        [Live]
```

### Canary Deployment
```yaml
deploy-canary:
  runs-on: ubuntu-latest
  steps:
    - run: deploy --canary 10%  # 10% of traffic
    - run: sleep 300  # Monitor for 5 minutes
    - run: deploy --full  # Full rollout if healthy
```

### Rolling Deployment
```
Old versions: [v1, v1, v1, v1]
Update 1:    [v2, v1, v1, v1]
Update 2:    [v2, v2, v1, v1]
Update 3:    [v2, v2, v2, v1]
Update 4:    [v2, v2, v2, v2] ✓
```

## Artifact Management

### Versioning
```bash
# Semantic versioning
VERSION="1.2.3"

# Or git-based versioning
VERSION=$(git describe --tags --always)
```

### Storing Artifacts
```yaml
# GitHub Actions
- uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: dist/

# Download in another job
- uses: actions/download-artifact@v4
  with:
    name: my-artifact
```

## Security Scanning

```yaml
security-scan:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'myapp:latest'
        format: 'table'
        exit-code: '1'
        severity: 'CRITICAL,HIGH'
```

## Environment Variables and Secrets

### GitHub Actions
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      NODE_ENV: production
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: ./deploy.sh
```

## Verification

After setting up CI/CD:
1. Push a test commit and verify pipeline runs
2. Check all stages execute in correct order
3. Verify artifacts are stored correctly
4. Test deployment to staging
5. Confirm secrets are properly loaded
6. Check notifications work (Slack, email)
