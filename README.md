# app-chart

A generic, production-ready Helm chart for deploying any containerized application to Kubernetes.

## Overview

This chart provides a flexible, reusable foundation for deploying containerized workloads. Instead of writing a new chart for every microservice, teams use this single chart with different `values.yaml` overrides to deploy Node.js APIs, Python services, static sites, Go binaries, and anything else that runs in a container.

## Features

- **Full deployment lifecycle** -- Deployment, Service, Ingress, ServiceAccount, ConfigMap, Secret
- **Autoscaling** -- HorizontalPodAutoscaler with CPU, memory, and custom metric targets
- **Disruption budgets** -- PodDisruptionBudget for safe cluster operations
- **Security hardened** -- Non-root containers, read-only filesystem, dropped capabilities by default
- **Health checks** -- Liveness, readiness, and startup probes with sensible defaults
- **Flexible networking** -- ClusterIP, NodePort, or LoadBalancer services with optional Ingress
- **Init and sidecar containers** -- First-class support for multi-container pod patterns
- **Environment management** -- Inline env vars, ConfigMap refs, Secret refs, and envFrom
- **Topology awareness** -- Node selectors, affinities, tolerations, and topology spread constraints
- **Rolling deployments** -- Configurable deployment strategy with rollback safety
- **Checksum annotations** -- Automatic pod restart on ConfigMap or Secret changes
- **Helm tests** -- Built-in connection test for validation

## Quick Start

### Install with defaults

```bash
helm install myapp ./app-chart
```

### Install with custom values

```bash
helm install myapp ./app-chart \
  --set image.repository=registry.example.com/acme/myapp \
  --set image.tag=1.2.3 \
  --set service.targetPort=3000
```

### Install from a values file

```bash
helm install myapp ./app-chart -f my-values.yaml
```

### Upgrade a release

```bash
helm upgrade myapp ./app-chart -f my-values.yaml
```

### Dry run to preview manifests

```bash
helm template myapp ./app-chart -f my-values.yaml
```

### Run tests

```bash
helm test myapp
```

## Parameters

### Global

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of pod replicas (ignored when HPA is enabled) | `1` |
| `nameOverride` | Override the chart name | `""` |
| `fullnameOverride` | Override the full release name | `""` |

### Image

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Container image repository | `nginx` |
| `image.tag` | Image tag (defaults to chart `appVersion`) | `""` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `imagePullSecrets` | Registry pull secrets | `[]` |

### Service Account

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.create` | Create a service account | `true` |
| `serviceAccount.annotations` | Service account annotations (e.g., IRSA role ARN) | `{}` |
| `serviceAccount.name` | Service account name override | `""` |
| `serviceAccount.automountServiceAccountToken` | Mount the SA token into pods | `true` |

### Service

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type: ClusterIP, NodePort, LoadBalancer | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port | `8080` |
| `service.nodePort` | Node port (only for NodePort type) | `""` |
| `service.annotations` | Service annotations | `{}` |

### Ingress

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable Ingress resource | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | List of host rules | `[]` |
| `ingress.tls` | TLS configuration | `[]` |

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `128Mi` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `256Mi` |

### Horizontal Pod Autoscaler

| Parameter | Description | Default |
|-----------|-------------|---------|
| `hpa.enabled` | Enable HPA | `false` |
| `hpa.minReplicas` | Minimum replicas | `2` |
| `hpa.maxReplicas` | Maximum replicas | `10` |
| `hpa.targetCPUUtilizationPercentage` | Target CPU utilization | `75` |
| `hpa.targetMemoryUtilizationPercentage` | Target memory utilization | `""` |
| `hpa.customMetrics` | Custom autoscaling metrics | `[]` |

### Pod Disruption Budget

| Parameter | Description | Default |
|-----------|-------------|---------|
| `pdb.enabled` | Enable PDB | `false` |
| `pdb.minAvailable` | Minimum available pods | `1` |
| `pdb.maxUnavailable` | Maximum unavailable pods | `""` |

### ConfigMap

| Parameter | Description | Default |
|-----------|-------------|---------|
| `configmap.enabled` | Create a ConfigMap from values | `false` |
| `configmap.data` | Key-value data for ConfigMap | `{}` |

### Secret

| Parameter | Description | Default |
|-----------|-------------|---------|
| `secret.enabled` | Create a Secret from values (base64 encoded) | `false` |
| `secret.data` | Key-value data for Secret | `{}` |

### Environment Variables

| Parameter | Description | Default |
|-----------|-------------|---------|
| `env` | List of env var definitions (name/value or name/valueFrom) | `[]` |
| `envFrom` | List of envFrom sources (configMapRef, secretRef) | `[]` |

### Probes

| Parameter | Description | Default |
|-----------|-------------|---------|
| `livenessProbe` | Liveness probe configuration | HTTP GET `/healthz` |
| `readinessProbe` | Readiness probe configuration | HTTP GET `/readyz` |
| `startupProbe` | Startup probe configuration | HTTP GET `/healthz` |

### Security

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podSecurityContext` | Pod-level security context | `runAsNonRoot: true, runAsUser: 1000` |
| `securityContext` | Container-level security context | `readOnlyRootFilesystem: true, drop ALL` |

### Scheduling

| Parameter | Description | Default |
|-----------|-------------|---------|
| `nodeSelector` | Node labels for pod assignment | `{}` |
| `tolerations` | Tolerations for pod scheduling | `[]` |
| `affinity` | Affinity rules | `{}` |
| `topologySpreadConstraints` | Topology spread constraints | `[]` |

### Volumes

| Parameter | Description | Default |
|-----------|-------------|---------|
| `volumes` | Additional volumes for the pod | `[]` |
| `volumeMounts` | Volume mounts for the main container | `[]` |

### Multi-Container Pods

| Parameter | Description | Default |
|-----------|-------------|---------|
| `initContainers` | Init containers to run before the main container | `[]` |
| `sidecarContainers` | Sidecar containers alongside the main container | `[]` |

### Deployment Behavior

| Parameter | Description | Default |
|-----------|-------------|---------|
| `strategy.type` | Deployment strategy | `RollingUpdate` |
| `strategy.rollingUpdate.maxSurge` | Max surge during rollout | `1` |
| `strategy.rollingUpdate.maxUnavailable` | Max unavailable during rollout | `0` |
| `terminationGracePeriodSeconds` | Grace period for pod shutdown | `30` |
| `podAnnotations` | Annotations added to all pods | `{}` |
| `podLabels` | Labels added to all pods | `{}` |

## Examples

The `examples/` directory contains ready-to-use value files for common scenarios:

### Node.js application

Deploys a Node.js API with HPA, ingress, TLS, environment variables from secrets, Prometheus scrape annotations, and topology spread constraints.

```bash
helm install node-api ./app-chart -f examples/nodejs-app.yaml
```

### Python FastAPI service

Deploys a FastAPI application with database migrations as an init container, configmap-driven settings, secret-based credentials, pod anti-affinity, and custom resource limits.

```bash
helm install fastapi ./app-chart -f examples/python-api.yaml
```

### Static website

Deploys an Nginx-based static site with multi-host ingress, TLS, minimal resources, and a pod disruption budget.

```bash
helm install website ./app-chart -f examples/static-site.yaml
```

## Common Patterns

### Using external secrets

Reference secrets managed outside this chart (e.g., by External Secrets Operator):

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: external-db-secret
        key: password
```

### Adding IAM roles for service accounts (IRSA)

```yaml
serviceAccount:
  create: true
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/my-role
```

### Running database migrations before startup

```yaml
initContainers:
  - name: migrate
    image: registry.example.com/acme/myapp:1.0.0
    command: ["python", "manage.py", "migrate"]
    envFrom:
      - secretRef:
          name: db-credentials
```

### Adding a sidecar proxy

```yaml
sidecarContainers:
  - name: cloud-sql-proxy
    image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.1.0
    args:
      - "--structured-logs"
      - "project:region:instance"
    securityContext:
      runAsNonRoot: true
```

### Mounting writable directories with read-only root filesystem

```yaml
securityContext:
  readOnlyRootFilesystem: true

volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir:
      sizeLimit: 100Mi

volumeMounts:
  - name: tmp
    mountPath: /tmp
  - name: cache
    mountPath: /app/.cache
```

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-change`
3. Make changes and test with `helm template` and `helm lint`
4. Verify with a dry run: `helm install test ./app-chart --dry-run`
5. Submit a pull request with a clear description of the change

### Development commands

```bash
# Lint the chart
helm lint ./app-chart

# Render templates locally
helm template test ./app-chart -f examples/nodejs-app.yaml

# Validate against a cluster
helm install test ./app-chart --dry-run --debug

# Run tests after install
helm test myapp
```
