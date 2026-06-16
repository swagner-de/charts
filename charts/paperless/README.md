# paperless

![Version: 0.50.3](https://img.shields.io/badge/Version-0.50.3-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2.20.15](https://img.shields.io/badge/AppVersion-2.20.15-informational?style=flat-square)
Document management system with OCR and full-text search
**Homepage:** <https://docs.paperless-ngx.com/>

## Features
- PostgreSQL database (packaged subchart or CNPG operator)
- Redis for task queue (packaged subchart or external)
- Gotenberg and Tika sidecars for document conversion
- Init container for directory setup
- Configurable OCR settings, consumer polling, and barcode scanning
- OIDC authentication support

## Install

```bash
helm install paperless oci://ghcr.io/swagner-de/charts/paperless
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |
| oci://registry-1.docker.io/cloudpirates | postgres | 0.19.5 |
| oci://registry-1.docker.io/cloudpirates | redis | 0.30.4 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cnpg | object | `{"clusterName":"paperless-db","enabled":false,"prependReleaseName":false}` | CloudNativePG database configuration |
| cnpg.clusterName | string | `"paperless-db"` | CNPG cluster name |
| cnpg.enabled | bool | `false` | Enable CNPG operator |
| cnpg.prependReleaseName | bool | `false` | Prepend release name and chart name to clusterName (use when CNPG cluster is deployed via rawResources) |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration |
| ingress.main.enabled | bool | `false` | Enable ingress |
| paperless | object | `{"admin":{"existingSecret":""},"config":null}` | Paperless-ngx application configuration |
| paperless.admin | object | `{"existingSecret":""}` | Admin credentials |
| paperless.admin.existingSecret | string | `""` | Existing secret with admin credentials |
| paperless.config | string | `nil` | Paperless config (keys converted to PAPERLESS_* env vars) |
| persistence | object | `{"consumption":{"accessMode":"ReadWriteOnce","size":"1Gi"},"data":{"accessMode":"ReadWriteOnce","size":"100Gi"}}` | Persistent storage configuration |
| persistence.consumption | object | `{"accessMode":"ReadWriteOnce","size":"1Gi"}` | Consumption directory volume |
| persistence.consumption.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.consumption.size | string | `"1Gi"` | Consumption directory volume size |
| persistence.data | object | `{"accessMode":"ReadWriteOnce","size":"100Gi"}` | Document storage volume |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.size | string | `"100Gi"` | Document storage volume size |
| postgres | object | `{"auth":{"database":"paperless","password":"CHANGEME","username":"paperless"},"enabled":true,"persistence":{"enabled":true,"size":"10Gi"}}` | PostgreSQL database configuration (subchart) |
| postgres.auth | object | `{"database":"paperless","password":"CHANGEME","username":"paperless"}` | PostgreSQL authentication |
| postgres.auth.database | string | `"paperless"` | Database name |
| postgres.auth.password | string | `"CHANGEME"` | Database password |
| postgres.auth.username | string | `"paperless"` | Database username |
| postgres.enabled | bool | `true` | Enable packaged PostgreSQL |
| postgres.persistence | object | `{"enabled":true,"size":"10Gi"}` | PostgreSQL persistence |
| postgres.persistence.enabled | bool | `true` | Enable PostgreSQL persistence |
| postgres.persistence.size | string | `"10Gi"` | PostgreSQL volume size |
| redis | object | `{"architecture":"standalone","auth":{"enabled":true,"password":"CHANGEME"},"enabled":true,"persistence":{"enabled":true,"size":"1Gi"}}` | Redis configuration (subchart) |
| redis.architecture | string | `"standalone"` | Redis architecture |
| redis.auth | object | `{"enabled":true,"password":"CHANGEME"}` | Redis authentication |
| redis.auth.enabled | bool | `true` | Enable Redis authentication |
| redis.auth.password | string | `"CHANGEME"` | Redis password |
| redis.enabled | bool | `true` | Enable packaged Redis |
| redis.persistence | object | `{"enabled":true,"size":"1Gi"}` | Redis persistence |
| redis.persistence.enabled | bool | `true` | Enable Redis persistence |
| redis.persistence.size | string | `"1Gi"` | Redis volume size |

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
