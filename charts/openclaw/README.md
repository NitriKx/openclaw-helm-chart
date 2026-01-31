# OpenClaw Helm Chart

A Helm chart for deploying the Moltbot application on Kubernetes.

## Installation

### Add Helm Repository

```bash
helm repo add openclaw https://yourusername.github.io/moltbot-helm-chart
helm repo update
```

### Install Chart

```bash
# Install with default values
helm install my-moltbot openclaw/openclaw

# Install with custom values
helm install my-moltbot openclaw/openclaw -f values.yaml

# Install in a specific namespace
helm install my-moltbot openclaw/openclaw -n moltbot-namespace --create-namespace
```

## Upgrading

```bash
helm upgrade my-moltbot openclaw/openclaw -f values.yaml
```

## Uninstalling

```bash
helm uninstall my-moltbot
```

## Configuration

### Global Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of pod replicas | `1` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full resource names | `""` |

### Image Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.registry` | Docker registry (empty for Docker Hub) | `""` |
| `image.repository` | Docker image repository | `moltbot/moltbot` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `image.tag` | Image tag (overrides appVersion) | `""` |
| `imagePullSecrets` | Image pull secrets | `[]` |

#### Example: Using a Custom Registry

```yaml
image:
  registry: "ghcr.io"
  repository: "myorg/moltbot"
  tag: "1.0.0"
```

Or for a private registry:

```yaml
image:
  registry: "registry.example.com:5000"
  repository: "myproject/moltbot"
  tag: "1.0.0"

imagePullSecrets:
  - name: registry-credentials
```

### Service Account

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.automount` | Automount SA token | `true` |
| `serviceAccount.annotations` | SA annotations | `{}` |
| `serviceAccount.name` | SA name (auto-generated if empty) | `""` |

### Security Context

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podSecurityContext.fsGroup` | File system group | `2000` |
| `securityContext.capabilities.drop` | Dropped capabilities | `["ALL"]` |
| `securityContext.readOnlyRootFilesystem` | Read-only root filesystem | `true` |
| `securityContext.runAsNonRoot` | Run as non-root | `true` |
| `securityContext.runAsUser` | User ID | `1000` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port | `8080` |
| `service.annotations` | Service annotations | `{}` |

### Ingress Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | Ingress hosts configuration | See values.yaml |
| `ingress.tls` | Ingress TLS configuration | `[]` |

#### Example Ingress Configuration

```yaml
ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: moltbot.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: moltbot-tls
      hosts:
        - moltbot.example.com
```

### Resource Limits

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU limit | Not set |
| `resources.limits.memory` | Memory limit | Not set |
| `resources.requests.cpu` | CPU request | Not set |
| `resources.requests.memory` | Memory request | Not set |

#### Example Resource Configuration

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

### Health Checks

| Parameter | Description | Default |
|-----------|-------------|---------|
| `livenessProbe` | Liveness probe configuration | HTTP check on `/healthz` |
| `readinessProbe` | Readiness probe configuration | HTTP check on `/ready` |

#### Example Custom Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 60
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 10
  periodSeconds: 5
```

### Autoscaling

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable HPA | `false` |
| `autoscaling.minReplicas` | Minimum replicas | `1` |
| `autoscaling.maxReplicas` | Maximum replicas | `100` |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU % | `80` |
| `autoscaling.targetMemoryUtilizationPercentage` | Target Memory % | Not set |

#### Example Autoscaling Configuration

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80
```

### Volumes

| Parameter | Description | Default |
|-----------|-------------|---------|
| `volumes` | Additional volumes | `[]` |
| `volumeMounts` | Additional volume mounts | `[]` |

#### Example Volume Configuration

```yaml
volumes:
  - name: config
    configMap:
      name: my-config
  - name: cache
    emptyDir: {}

volumeMounts:
  - name: config
    mountPath: /etc/config
    readOnly: true
  - name: cache
    mountPath: /tmp/cache
```

### Environment Variables

| Parameter | Description | Default |
|-----------|-------------|---------|
| `env` | Environment variables | `[]` |

#### Example Environment Variables

```yaml
env:
  - name: LOG_LEVEL
    value: "debug"
  - name: API_ENDPOINT
    value: "https://api.example.com"
  - name: SECRET_VALUE
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: api-key
```

### ConfigMap

| Parameter | Description | Default |
|-----------|-------------|---------|
| `configMap.enabled` | Create ConfigMap | `false` |
| `configMap.data` | ConfigMap data | `{}` |

#### Example ConfigMap Configuration

```yaml
configMap:
  enabled: true
  data:
    config.yaml: |
      server:
        port: 8080
        host: 0.0.0.0
    settings.json: |
      {
        "debug": true
      }
```

### Secret Management

The chart supports two methods for managing secrets:

#### Option 1: Using Existing Secret (Recommended for Production)

Reference a pre-existing Kubernetes secret:

```yaml
existingSecret: "my-existing-secret"
```

The existing secret should contain all required keys that your application needs. The keys will be automatically exposed as environment variables in the deployment.

Create your secret separately:

```bash
kubectl create secret generic my-existing-secret \
  --from-literal=api-key=your-api-key \
  --from-literal=db-password=your-password
```

#### Option 2: Creating Secret from Values

For development/testing, create secrets directly from values:

```yaml
secrets:
  api-key: "your-api-key"
  db-password: "your-password"
  token: "your-token"
```

**Warning**: This method stores secrets in your values file. Not recommended for production.

Secret keys will be automatically converted to environment variable names:
- `api-key` becomes `API_KEY`
- `db-password` becomes `DB_PASSWORD`

### Extra Manifests

Deploy additional Kubernetes resources alongside the chart:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `extraManifests` | Additional Kubernetes manifests | `[]` |

#### Example Extra Manifests

```yaml
extraManifests:
  # Additional ConfigMap
  - apiVersion: v1
    kind: ConfigMap
    metadata:
      name: custom-config
      namespace: {{ .Release.Namespace }}
    data:
      custom-key: "custom-value"

  # Additional Secret
  - apiVersion: v1
    kind: Secret
    metadata:
      name: custom-secret
    type: Opaque
    stringData:
      password: "changeme"

  # NetworkPolicy
  - apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: {{ include "openclaw.fullname" . }}-netpol
    spec:
      podSelector:
        matchLabels:
          {{- include "openclaw.selectorLabels" . | nindent 6 }}
      policyTypes:
        - Ingress
      ingress:
        - from:
            - podSelector:
                matchLabels:
                  app: frontend

  # PersistentVolumeClaim
  - apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: {{ include "openclaw.fullname" . }}-data
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
```

You can use template functions in extraManifests. Available template functions include:
- `{{ .Release.Name }}`
- `{{ .Release.Namespace }}`
- `{{ include "openclaw.fullname" . }}`
- `{{ include "openclaw.labels" . }}`

### Pod Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podAnnotations` | Pod annotations | `{}` |
| `podLabels` | Additional pod labels | `{}` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Pod tolerations | `[]` |
| `affinity` | Pod affinity rules | `{}` |

#### Example Pod Scheduling

```yaml
nodeSelector:
  disktype: ssd

tolerations:
  - key: "special"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values:
                  - openclaw
          topologyKey: kubernetes.io/hostname
```

## Examples

### Minimal Production Configuration

```yaml
replicaCount: 3

image:
  tag: "1.0.0"

existingSecret: "moltbot-secrets"

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

### Full Production Configuration with Ingress

```yaml
replicaCount: 3

image:
  repository: moltbot/moltbot
  tag: "1.0.0"
  pullPolicy: IfNotPresent

existingSecret: "moltbot-secrets"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
    - host: moltbot.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: moltbot-tls
      hosts:
        - moltbot.example.com

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values:
                  - openclaw
          topologyKey: kubernetes.io/hostname
```

### Development Configuration

```yaml
replicaCount: 1

image:
  tag: "latest"
  pullPolicy: Always

secrets:
  api-key: "dev-api-key"

service:
  type: NodePort

resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

## Troubleshooting

### Check Pod Status

```bash
kubectl get pods -l app.kubernetes.io/name=openclaw
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Check Service

```bash
kubectl get svc -l app.kubernetes.io/name=openclaw
kubectl describe svc <service-name>
```

### Check Ingress

```bash
kubectl get ingress
kubectl describe ingress <ingress-name>
```

### Test Chart Locally

```bash
# Lint the chart
helm lint .

# Dry run
helm install test-release . --dry-run --debug

# Template rendering
helm template test-release .
```

## Maintenance

### Update Chart Dependencies

```bash
helm dependency update
```

### Package Chart

```bash
helm package .
```

## Support

For issues and questions:
- GitHub Issues: https://github.com/yourusername/moltbot-helm-chart/issues
- Docker Hub: https://hub.docker.com/r/moltbot/moltbot
