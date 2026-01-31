# Contributing to Moltbot Helm Chart

Thank you for your interest in contributing to the Moltbot Helm Chart! This document provides guidelines and instructions for contributing to this project.

## Table of Contents

- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Development Workflow](#development-workflow)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)
- [Coding Standards](#coding-standards)
- [Release Process](#release-process)

## Getting Started

### Prerequisites

Before you begin, ensure you have the following tools installed:

- [Git](https://git-scm.com/)
- [Helm](https://helm.sh/) (v3.8+)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Docker](https://www.docker.com/) or [kind](https://kind.sigs.k8s.io/) for local testing
- [Python](https://www.python.org/) (3.x)
- [pre-commit](https://pre-commit.com/)

### Fork and Clone

1. Fork the repository on GitHub
2. Clone your fork locally:

```bash
git clone https://github.com/yourusername/moltbot-helm-chart.git
cd moltbot-helm-chart
```

3. Add the upstream repository:

```bash
git remote add upstream https://github.com/originalowner/moltbot-helm-chart.git
```

## Development Setup

### Install Pre-commit Hooks

Pre-commit hooks ensure code quality and consistency:

```bash
# Install pre-commit
pip install pre-commit

# Install the git hooks
pre-commit install

# (Optional) Run against all files
pre-commit run --all-files
```

### Install Development Tools

```bash
# Install chart-testing
brew install chart-testing  # macOS
# or download from https://github.com/helm/chart-testing/releases

# Install yamllint
pip install yamllint

# Install helm-docs (optional, for generating documentation)
brew install norwoodj/tap/helm-docs  # macOS
```

## Development Workflow

### Creating a Branch

Create a feature branch for your changes:

```bash
git checkout -b feature/my-new-feature
# or
git checkout -b fix/bug-description
```

Branch naming conventions:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `chore/` - Maintenance tasks

### Making Changes

1. Make your changes to the Helm chart
2. Update documentation if needed
3. Add or update tests as appropriate
4. Run pre-commit checks:

```bash
pre-commit run --all-files
```

### Linting the Chart

```bash
# Lint the chart
helm lint charts/openclaw

# Lint with custom values
helm lint charts/openclaw -f charts/openclaw/ci/default-values.yaml
```

### Testing Template Rendering

```bash
# Render templates
helm template my-release charts/openclaw

# Render with custom values
helm template my-release charts/openclaw -f values.yaml

# Debug mode
helm template my-release charts/openclaw --debug
```

## Testing

### Local Testing

#### Using Helm

```bash
# Create a test release (dry-run)
helm install test-release charts/openclaw --dry-run --debug

# Install locally
helm install test-release charts/openclaw -n test --create-namespace

# Test the installation
helm test test-release -n test

# Cleanup
helm uninstall test-release -n test
```

#### Using chart-testing (ct)

```bash
# Lint all charts
ct lint --config ct.yaml

# Lint changed charts only
ct lint --config ct.yaml --all

# Install and test charts in a kind cluster
kind create cluster --name chart-testing
ct install --config ct.yaml
kind delete cluster --name chart-testing
```

### Testing with Different Values

Test with the provided CI values:

```bash
# Test default values
helm install test charts/openclaw -f charts/openclaw/ci/default-values.yaml --dry-run

# Test with ingress
helm install test charts/openclaw -f charts/openclaw/ci/with-ingress.yaml --dry-run

# Test with extra manifests
helm install test charts/openclaw -f charts/openclaw/ci/with-extra-manifests.yaml --dry-run
```

### Running in Kind

```bash
# Create a kind cluster
kind create cluster

# Install the chart
helm install test-release charts/openclaw

# Check the deployment
kubectl get all

# Test
helm test test-release

# Cleanup
helm uninstall test-release
kind delete cluster
```

## Submitting Changes

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `chore`: Maintenance tasks
- `test`: Adding or updating tests
- `refactor`: Code refactoring
- `ci`: CI/CD changes

Examples:
```
feat(chart): add support for PodDisruptionBudget

fix(deployment): correct health check path

docs(readme): update installation instructions

chore(deps): update helm dependencies
```

### Creating a Pull Request

1. Push your changes to your fork:

```bash
git push origin feature/my-new-feature
```

2. Go to the GitHub repository and create a Pull Request

3. Fill in the PR template with:
   - Description of changes
   - Related issues (if any)
   - Testing performed
   - Screenshots (if applicable)

4. Ensure all CI checks pass:
   - Pre-commit checks
   - Chart linting
   - Chart installation tests

### PR Review Process

1. At least one maintainer review is required
2. All CI checks must pass
3. Address any review comments
4. Once approved, a maintainer will merge your PR

## Coding Standards

### Helm Chart Best Practices

- Follow [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- Use descriptive names for templates and values
- Include comments for complex logic
- Keep templates modular and reusable
- Use helper templates in `_helpers.tpl`

### YAML Formatting

- Use 2 spaces for indentation
- Maximum line length: 120 characters
- Use lowercase for keys
- Quote string values when necessary

### Template Guidelines

```yaml
# Good: Descriptive conditions
{{- if .Values.ingress.enabled }}
...
{{- end }}

# Good: Proper indentation in templates
metadata:
  name: {{ include "openclaw.fullname" . }}
  labels:
    {{- include "openclaw.labels" . | nindent 4 }}

# Good: Using helper functions
serviceAccountName: {{ include "openclaw.serviceAccountName" . }}
```

### Values.yaml Guidelines

- Group related values together
- Include comments explaining each value
- Provide sensible defaults
- Use descriptive names
- Document default values

## Release Process

This project uses automated releases via release-please:

1. Merge PRs to `main` branch
2. Release-please will automatically create a release PR
3. When the release PR is merged:
   - Version is bumped automatically
   - CHANGELOG is generated
   - GitHub release is created
   - Chart is published to GitHub Pages

### Manual Version Updates

If you need to update the chart version manually:

1. Update `version` in `charts/openclaw/Chart.yaml`
2. Update `appVersion` if needed (or let Renovate handle it)
3. Update `CHANGELOG.md` with changes
4. Commit with message: `chore(release): bump version to X.Y.Z`

### Updating Dependencies

This project uses Renovate for automatic dependency updates:

- Docker images: Automatically updated via Renovate
- GitHub Actions: Automatically updated
- Pre-commit hooks: Review and approve Renovate PRs

To manually update dependencies:

```bash
# Update Helm dependencies (if any)
helm dependency update charts/openclaw

# Update pre-commit hooks
pre-commit autoupdate
```

## Documentation

### Updating Documentation

When making changes, update relevant documentation:

- `README.md` - Project overview and quick start
- `charts/openclaw/README.md` - Detailed chart documentation
- `CHANGELOG.md` - Release notes (handled by release-please)
- Code comments - Explain complex logic

### Generating Documentation

```bash
# If using helm-docs
cd charts/openclaw
helm-docs
```

## Getting Help

- Open an issue for bugs or feature requests
- Check existing issues and PRs
- Join community discussions

## Code of Conduct

Be respectful and considerate in all interactions. We aim to maintain a welcoming and inclusive community.

## License

By contributing, you agree that your contributions will be licensed under the same license as the project.

---

Thank you for contributing to the Moltbot Helm Chart!
