# ghostfolio

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 3.26.0](https://img.shields.io/badge/AppVersion-3.26.0-informational?style=flat-square)
Open source wealth management software
**Homepage:** <https://ghostfol.io/>

## Features
- CloudNativePG (CNPG) for PostgreSQL
- Redis for caching (packaged subchart or external)
- Gateway API HTTPRoute or Ingress support
- Health probes against `/api/v1/health`

## Install

```bash
helm install ghostfolio oci://ghcr.io/swagner-de/charts/ghostfolio
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |
| oci://registry-1.docker.io/cloudpirates | redis | 0.32.1 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| appSecrets | object | `{}` | Ghostfolio application secrets rendered into a Secret and mounted via envFrom. ACCESS_TOKEN_SALT and JWT_SECRET_KEY are required — either populate here or provide via `envFromSecrets`. |
| cnpg | object | `{"clusterName":"db","enabled":true,"prependReleaseName":true}` | CloudNativePG database configuration |
| cnpg.clusterName | string | `"db"` | CNPG cluster name |
| cnpg.enabled | bool | `true` | Enable CNPG-provided PostgreSQL |
| cnpg.prependReleaseName | bool | `true` | Prepend release name and chart name to clusterName (use when CNPG cluster is deployed via rawResources) |
| config | object | `{}` | Ghostfolio application configuration (converted to environment variables) See https://github.com/ghostfolio/ghostfolio/blob/main/.env.example |
| envFromSecrets | list | `[]` | Additional secrets to mount as environment variables |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration |
| ingress.main.enabled | bool | `false` | Enable ingress |
| rawResources | object | `{"db":{"enabled":true,"manifest":{"apiVersion":"postgresql.cnpg.io/v1","kind":"Cluster","spec":{"imageCatalogRef":{"apiGroup":"postgresql.cnpg.io","kind":"ClusterImageCatalog","major":16,"name":"postgresql"},"instances":1,"storage":{"size":"5Gi"}}}}}` | Raw Kubernetes resources |
| rawResources.db | object | `{"enabled":true,"manifest":{"apiVersion":"postgresql.cnpg.io/v1","kind":"Cluster","spec":{"imageCatalogRef":{"apiGroup":"postgresql.cnpg.io","kind":"ClusterImageCatalog","major":16,"name":"postgresql"},"instances":1,"storage":{"size":"5Gi"}}}}` | CNPG database cluster |
| rawResources.db.enabled | bool | `true` | Enable CNPG database cluster |
| rawResources.db.manifest | object | `{"apiVersion":"postgresql.cnpg.io/v1","kind":"Cluster","spec":{"imageCatalogRef":{"apiGroup":"postgresql.cnpg.io","kind":"ClusterImageCatalog","major":16,"name":"postgresql"},"instances":1,"storage":{"size":"5Gi"}}}` | Resource manifest |
| redis | object | `{"architecture":"standalone","auth":{"enabled":true},"enabled":true,"persistence":{"enabled":true,"size":"1Gi"}}` | Redis configuration (subchart) |
| redis.architecture | string | `"standalone"` | Redis architecture |
| redis.auth | object | `{"enabled":true}` | Redis authentication |
| redis.auth.enabled | bool | `true` | Enable Redis authentication |
| redis.enabled | bool | `true` | Enable packaged Redis |
| redis.persistence | object | `{"enabled":true,"size":"1Gi"}` | Redis persistence |
| redis.persistence.enabled | bool | `true` | Enable Redis persistence |
| redis.persistence.size | string | `"1Gi"` | Redis volume size |
| route | object | `{"main":{"enabled":false}}` | Gateway API HTTPRoute configuration |
| route.main.enabled | bool | `false` | Enable Gateway API HTTPRoute |

## Security
- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
