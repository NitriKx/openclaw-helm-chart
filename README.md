# Moltbot Helm Chart

[![Release Charts](https://github.com/yourusername/moltbot-helm-chart/actions/workflows/release-charts.yaml/badge.svg)](https://github.com/yourusername/moltbot-helm-chart/actions/workflows/release-charts.yaml)
[![Helm Chart Testing](https://github.com/yourusername/moltbot-helm-chart/actions/workflows/chart-testing.yaml/badge.svg)](https://github.com/yourusername/moltbot-helm-chart/actions/workflows/chart-testing.yaml)
[![Pre-commit Checks](https://github.com/yourusername/moltbot-helm-chart/actions/workflows/pre-commit.yaml/badge.svg)](https://github.com/yourusername/moltbot-helm-chart/actions/workflows/pre-commit.yaml)
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/openclaw)](https://artifacthub.io/packages/helm/openclaw/openclaw)

A production-ready Helm chart for deploying the [Moltbot](https://hub.docker.com/r/moltbot/moltbot) application on Kubernetes.

## Features

- **Flexible Secret Management**: Use existing secrets or create them from values
- **Extra Manifests Support**: Deploy additional Kubernetes resources directly from values
- **Production Ready**: Security contexts, resource limits, health checks, and autoscaling
- **Full Ingress Support**: Easily expose your application with configurable Ingress
- **Automated Testing**: Comprehensive CI/CD with chart-testing and pre-commit hooks
- **Automated Releases**: Uses release-please for semantic versioning and changelog generation

## Quick Start

### Prerequisites

- Kubernetes 1.19+
- Helm 3.8+

### Installation

Add the Helm repository:

```bash
helm repo add openclaw https://yourusername.github.io/moltbot-helm-chart
helm repo update
```

Install the chart:

```bash
helm install my-moltbot openclaw/openclaw
```

### Using Custom Values

Create a `values.yaml` file:

```yaml
image:
  tag: "1.0.0"

service:
  type: LoadBalancer

secrets:
  api-key: "your-api-key"
```

Install with custom values:

```bash
helm install my-moltbot openclaw/openclaw -f values.yaml
```

## Configuration

See [charts/openclaw/README.md](charts/openclaw/README.md) for detailed configuration options.

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Docker image repository | `moltbot/moltbot` |
| `image.tag` | Docker image tag | Chart appVersion |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `ingress.enabled` | Enable ingress | `false` |
| `existingSecret` | Use existing secret | `""` |
| `secrets` | Create secret from values | `{}` |
| `extraManifests` | Additional manifests to deploy | `[]` |

## Secret Management

### Option 1: Using Existing Secret (Recommended for Production)

```yaml
existingSecret: "my-existing-secret"
```

### Option 2: Creating Secret from Values

```yaml
secrets:
  api-key: "your-api-key"
  db-password: "your-password"
```

## Extra Manifests

Deploy additional Kubernetes resources:

```yaml
extraManifests:
  - apiVersion: v1
    kind: ConfigMap
    metadata:
      name: custom-config
    data:
      key: value
  - apiVersion: v1
    kind: Secret
    metadata:
      name: extra-secret
    type: Opaque
    stringData:
      password: changeme
```

## Development

See [CONTRIBUTING.md](CONTRIBUTING.md) for development guidelines.

### Testing Locally

```bash
# Lint the chart
helm lint charts/openclaw

# Test template rendering
helm template my-moltbot charts/openclaw

# Install locally
helm install my-moltbot charts/openclaw --dry-run --debug
```

### Running Tests

```bash
# Install pre-commit hooks
pre-commit install

# Run all checks
pre-commit run --all-files

# Run chart testing
ct lint --config ct.yaml
ct install --config ct.yaml
```

## CI/CD

This repository uses GitHub Actions for:

- **Pre-commit Checks**: Runs on every PR to ensure code quality
- **Chart Testing**: Lints and installs charts in a kind cluster
- **Chart Releaser**: Publishes charts to GitHub Pages
- **Release Please**: Automates semantic versioning and changelog generation

## Repository Structure

```
.
├── .github/
│   └── workflows/          # GitHub Actions workflows
├── charts/
│   └── openclaw/          # Helm chart
│       ├── templates/     # Kubernetes manifests
│       ├── ci/           # Test values for CI
│       ├── Chart.yaml    # Chart metadata
│       ├── values.yaml   # Default values
│       └── README.md     # Chart documentation
├── .pre-commit-config.yaml
├── ct.yaml               # chart-testing config
├── cr.yaml               # chart-releaser config
└── README.md
```

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## License

[Your License Here]

## Links

- [Moltbot Docker Hub](https://hub.docker.com/r/moltbot/moltbot)
- [Artifact Hub](https://artifacthub.io/packages/helm/openclaw/openclaw)
- [GitHub Repository](https://github.com/yourusername/moltbot-helm-chart)
