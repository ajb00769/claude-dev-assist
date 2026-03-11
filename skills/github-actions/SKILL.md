---
name: github-actions
description: GitHub Actions CI/CD workflows. Use when user says /github-actions, needs to create/edit workflows, debug failed runs, manage secrets, or set up deployments. Works with .github/workflows/*.yml files.
---

# GitHub Actions

Create and manage CI/CD workflows with GitHub Actions.

## Workflow Location

All workflows go in `.github/workflows/*.yml`

## Quick Start

1. Create workflow file: `.github/workflows/<name>.yml`
2. Define trigger events
3. Define jobs and steps
4. Commit and push

## Workflow Structure

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup runtime
        # Use the appropriate setup action for your stack:
        # actions/setup-python@v5, actions/setup-node@v4,
        # actions/setup-go@v5, actions/setup-java@v4, etc.
        uses: actions/setup-node@v4
        with:
          node-version: '22'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
```

## Common Triggers

```yaml
on:
  # Push to specific branches
  push:
    branches: [main]
    paths:
      - 'src/**'
      - '!docs/**'

  # Pull requests
  pull_request:
    types: [opened, synchronize]

  # Manual trigger
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'

  # Scheduled
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC
```

## Secrets Management

```bash
# CLI commands
gh secret set SECRET_NAME --body "value"
gh secret list
gh secret delete SECRET_NAME

# Environment secrets
gh secret set SECRET_NAME --env production --body "value"
```

**In workflow:**
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}

steps:
  - name: Deploy
    env:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
    run: ./deploy.sh
```

## Environments

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - run: echo "Deploying to staging"

  deploy-prod:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    needs: deploy-staging
    steps:
      - run: echo "Deploying to production"
```

## Debugging Failed Runs

```bash
# View workflow runs
gh run list

# View specific run
gh run view <run-id>

# View logs
gh run view <run-id> --log

# Watch run in progress
gh run watch <run-id>

# Re-run failed jobs
gh run rerun <run-id> --failed
```

## Common Patterns

### Job Dependencies

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [...]

  build:
    needs: test
    runs-on: ubuntu-latest
    steps: [...]

  deploy:
    needs: [test, build]
    runs-on: ubuntu-latest
    steps: [...]
```

### Matrix Builds

```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: ['18', '20', '22']
        os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
```

### Conditional Steps

```yaml
steps:
  - name: Deploy to prod
    if: github.ref == 'refs/heads/main'
    run: ./deploy-prod.sh

  - name: Deploy to staging
    if: github.ref == 'refs/heads/develop'
    run: ./deploy-staging.sh
```

### Caching

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm  # Adjust path for your package manager
    key: ${{ runner.os }}-deps-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-deps-
```

## Templates

See [assets/](assets/) for example workflow templates (adapt to your stack):
- `python-ci.yml` - Python testing and linting (example CI pattern)
- `docker-build.yml` - Docker build and push to registry (stack-agnostic)
- `azure-deploy.yml` - Deploy to Azure App Service (example deploy pattern)

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Workflow not triggering | Check branch names, path filters |
| Secret not found | Verify secret name, check environment scope |
| Permission denied | Add `permissions:` block to workflow |
| Checkout fails | Check `fetch-depth`, token permissions |