# Reusable GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows for the Gwiazdeczkowe Animacje CI/CD pipeline.

## Available Workflows

### 1. Docker Build and Push (`docker-build-push.yml`)

Builds a Docker image and pushes it to the registry.

**Usage:**
```yaml
jobs:
  build:
    uses: pienkow/reusable-workflows/.github/workflows/docker-build-push.yml@main
    with:
      image_name: 'gwiazdeczkowe/api'
      version: '0.1.32'
      dockerfile_path: 'Dockerfile'  # optional, default: 'Dockerfile'
      context_path: '.'  # optional, default: '.'
      registry: '192.168.1.6:30500'  # optional, default: '192.168.1.6:30500'
    secrets:
      REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}  # optional
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}  # optional
```

### 2. Helm Deploy (`helm-deploy.yml`)

Deploys application to Kubernetes using Helm.

**Usage:**
```yaml
jobs:
  deploy:
    uses: pienkow/reusable-workflows/.github/workflows/helm-deploy.yml@main
    with:
      chart_path: 'helm'
      release_name: 'api'
      namespace: 'gwiazdeczkowe-animacje'  # optional, default: 'gwiazdeczkowe-animacje'
      version: '0.1.32'
      create_namespace: false  # optional, default: false
      timeout: '10m'  # optional, default: '10m'
    secrets:
      KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

### 3. Lint and Validate (`lint-and-validate.yml`)

Runs linting and validation checks for Django, React, or Helm projects.

**Usage for Django:**
```yaml
jobs:
  lint:
    uses: pienkow/reusable-workflows/.github/workflows/lint-and-validate.yml@main
    with:
      project_type: 'django'
      python_version: '3.14'  # optional, default: '3.14'
      working_directory: '.'  # optional, default: '.'
```

**Usage for React:**
```yaml
jobs:
  lint:
    uses: pienkow/reusable-workflows/.github/workflows/lint-and-validate.yml@main
    with:
      project_type: 'react'
      node_version: '20'  # optional, default: '20'
      working_directory: '.'  # optional, default: '.'
```

**Usage for Helm:**
```yaml
jobs:
  lint:
    uses: pienkow/reusable-workflows/.github/workflows/lint-and-validate.yml@main
    with:
      project_type: 'helm'
      working_directory: 'helm'  # optional, default: '.'
```

## Infrastructure Details

- **Docker Registry:** 192.168.1.6:30500 (NodePort)
- **Kubernetes:** k3s cluster at 192.168.1.6
- **Default Namespace:** gwiazdeczkowe-animacje
- **Helm:** v3+

## Required Secrets

Configure these secrets in your repository:

- `KUBECONFIG` - Base64 encoded Kubernetes config file
- `REGISTRY_USERNAME` - Docker registry username (if authentication required)
- `REGISTRY_PASSWORD` - Docker registry password (if authentication required)

## Example: Complete CI/CD Pipeline

```yaml
name: CI/CD Pipeline

on:
  pull_request:
    branches: [main]
  release:
    types: [published]

jobs:
  # Run linting on PRs
  lint:
    if: github.event_name == 'pull_request'
    uses: pienkow/reusable-workflows/.github/workflows/lint-and-validate.yml@main
    with:
      project_type: 'django'
      working_directory: '.'

  # Build, push, and deploy on release
  build:
    if: github.event_name == 'release'
    uses: pienkow/reusable-workflows/.github/workflows/docker-build-push.yml@main
    with:
      image_name: 'gwiazdeczkowe/api'
      version: ${{ github.event.release.tag_name }}

  deploy:
    needs: build
    if: github.event_name == 'release'
    uses: pienkow/reusable-workflows/.github/workflows/helm-deploy.yml@main
    with:
      chart_path: 'helm'
      release_name: 'api'
      version: ${{ github.event.release.tag_name }}
    secrets:
      KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

## License

MIT
