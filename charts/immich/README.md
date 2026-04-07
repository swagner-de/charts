# immich

A Helm chart for [Immich](https://immich.app/), a self-hosted photo and video backup solution.

## Features

- Separate server and machine-learning controllers for independent scaling
- GPU acceleration support for machine learning (OpenVINO, CUDA, etc.)
- CloudNativePG (CNPG) for PostgreSQL with pgvecto.rs/VectorChord extension
- Redis for caching (packaged subchart or external)
- Config assembly from ConfigMaps and Secrets via init container
- Prometheus metrics endpoints and optional ServiceMonitor

## Install

```bash
helm install immich oci://ghcr.io/swagner-de/charts/immich --version 0.5.4
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `persistence.library.size` | Photo library volume size | `1Ti` |
| `persistence.ml-cache.size` | ML model cache volume size | `10Gi` |
| `redis.enabled` | Enable packaged Redis | `true` |
| `redis.auth.enabled` | Enable Redis auth | `true` |
| `cnpgClusterName` | CNPG cluster name for database | `db` |
| `machineLearning.acceleration` | ML acceleration backend (e.g., `openvino`) | `""` |
| `config` | Immich configuration (merged into config file) | `{}` |
| `configFromSecret` | Additional config from Kubernetes Secrets | `{}` |
| `route.main.enabled` | Enable Gateway API HTTPRoute | `false` |
| `serviceMonitor.main.enabled` | Enable Prometheus ServiceMonitor | `false` |

### Machine Learning Acceleration

```yaml
machineLearning:
  acceleration: openvino  # Uses immich-machine-learning:<version>-openvino image tag
```

### Configuration from Secrets

Merge sensitive configuration (e.g., OIDC) from external Secrets:

```yaml
configFromSecret:
  oauth:
    secretName: immich-oidc
    secretKey: oauth.yaml
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true` (server), `false` (machine-learning, due to model downloads)
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
