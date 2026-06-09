# immich

![Version: 0.11.4](https://img.shields.io/badge/Version-0.11.4-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v2.7.5](https://img.shields.io/badge/AppVersion-v2.7.5-informational?style=flat-square)
Self-hosted photo and video backup solution with machine learning
**Homepage:** <https://immich.app/>

## Features
- Separate server and machine-learning controllers for independent scaling
- GPU acceleration support for machine learning (OpenVINO, CUDA, etc.)
- CloudNativePG (CNPG) for PostgreSQL with pgvecto.rs/VectorChord extension
- Redis for caching (packaged subchart or external)
- Config assembly from ConfigMaps and Secrets via init container
- Prometheus metrics endpoints and optional ServiceMonitor

## Install

```bash
helm install immich oci://ghcr.io/swagner-de/charts/immich
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |
| oci://registry-1.docker.io/cloudpirates | redis | 0.30.3 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cnpgClusterName | string | `"db"` | CNPG cluster name for database |
| config | object | `{}` | Immich configuration (merged into config file) |
| configFromSecret | object | `{}` | Additional configuration from Kubernetes Secrets |
| machineLearning | object | `{"acceleration":""}` | Machine learning acceleration configuration |
| machineLearning.acceleration | string | `""` | ML acceleration backend (e.g., openvino, cuda) |
| persistence | object | `{"library":{"accessMode":"ReadWriteOnce","size":"1Ti","type":"persistentVolumeClaim"},"ml-cache":{"accessMode":"ReadWriteOnce","size":"10Gi","type":"persistentVolumeClaim"}}` | Persistent storage configuration |
| persistence.library | object | `{"accessMode":"ReadWriteOnce","size":"1Ti","type":"persistentVolumeClaim"}` | Photo library volume |
| persistence.library.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.library.size | string | `"1Ti"` | Photo library volume size |
| persistence.library.type | string | `"persistentVolumeClaim"` | Volume type |
| persistence.ml-cache | object | `{"accessMode":"ReadWriteOnce","size":"10Gi","type":"persistentVolumeClaim"}` | Machine learning model cache volume |
| persistence.ml-cache.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.ml-cache.size | string | `"10Gi"` | ML cache volume size |
| persistence.ml-cache.type | string | `"persistentVolumeClaim"` | Volume type |
| rawResources | object | `{"db":{"enabled":true,"manifest":{"apiVersion":"postgresql.cnpg.io/v1","kind":"Cluster","spec":{"bootstrap":{"initdb":{"postInitApplicationSQL":["CREATE EXTENSION IF NOT EXISTS vchord CASCADE;","CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE;"],"postInitSQL":["CREATE EXTENSION IF NOT EXISTS vchord CASCADE;","CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE;"]}},"imageName":"ghcr.io/tensorchord/cloudnative-vectorchord:18-1.1.0","instances":1,"postgresql":{"shared_preload_libraries":["vchord.so"]},"storage":{"size":"5Gi"}}}}}` | Raw Kubernetes resources |
| rawResources.db | object | `{"enabled":true,"manifest":{"apiVersion":"postgresql.cnpg.io/v1","kind":"Cluster","spec":{"bootstrap":{"initdb":{"postInitApplicationSQL":["CREATE EXTENSION IF NOT EXISTS vchord CASCADE;","CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE;"],"postInitSQL":["CREATE EXTENSION IF NOT EXISTS vchord CASCADE;","CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE;"]}},"imageName":"ghcr.io/tensorchord/cloudnative-vectorchord:18-1.1.0","instances":1,"postgresql":{"shared_preload_libraries":["vchord.so"]},"storage":{"size":"5Gi"}}}}` | CNPG database cluster |
| rawResources.db.enabled | bool | `true` | Enable CNPG database cluster |
| rawResources.db.manifest | object | `{"apiVersion":"postgresql.cnpg.io/v1","kind":"Cluster","spec":{"bootstrap":{"initdb":{"postInitApplicationSQL":["CREATE EXTENSION IF NOT EXISTS vchord CASCADE;","CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE;"],"postInitSQL":["CREATE EXTENSION IF NOT EXISTS vchord CASCADE;","CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE;"]}},"imageName":"ghcr.io/tensorchord/cloudnative-vectorchord:18-1.1.0","instances":1,"postgresql":{"shared_preload_libraries":["vchord.so"]},"storage":{"size":"5Gi"}}}` | Resource manifest |
| redis | object | `{"architecture":"standalone","auth":{"enabled":true},"enabled":true,"persistence":{"enabled":true,"size":"1Gi"}}` | Redis configuration (subchart) |
| redis.architecture | string | `"standalone"` | Redis architecture |
| redis.auth | object | `{"enabled":true}` | Redis authentication configuration |
| redis.auth.enabled | bool | `true` | Enable Redis authentication |
| redis.enabled | bool | `true` | Enable packaged Redis |
| redis.persistence | object | `{"enabled":true,"size":"1Gi"}` | Redis persistence configuration |
| redis.persistence.enabled | bool | `true` | Enable Redis persistence |
| redis.persistence.size | string | `"1Gi"` | Redis volume size |
| route | object | `{"main":{"enabled":false}}` | Gateway API HTTPRoute configuration |
| route.main.enabled | bool | `false` | Enable Gateway API HTTPRoute |
| serviceMonitor | object | `{"main":{"enabled":false,"serviceName":"immich-server"}}` | Prometheus ServiceMonitor configuration |
| serviceMonitor.main.enabled | bool | `false` | Enable Prometheus ServiceMonitor |
| serviceMonitor.main.serviceName | string | `"immich-server"` | Service name for the ServiceMonitor |

## Security
- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true` (server), `false` (machine-learning, due to model downloads)
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
