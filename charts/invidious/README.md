# invidious

![Version: 0.6.0](https://img.shields.io/badge/Version-0.6.0-informational?style=flat-square) ![AppVersion: 2.20260207.0](https://img.shields.io/badge/AppVersion-2.20260207.0-informational?style=flat-square)
Privacy-focused alternative YouTube frontend with companion service and PostgreSQL
**Homepage:** <https://github.com/iv-org/invidious>

## Features
- Invidious server with companion service for enhanced video playback
- PostgreSQL database (packaged subchart or CNPG operator)
- HMAC-signed companion key authentication
- CNPG support with automatic database connection configuration

## Install

```bash
helm install invidious oci://ghcr.io/swagner-de/charts/invidious
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts | common | 5.0.1 |
| oci://registry-1.docker.io/cloudpirates | postgres | 0.19.5 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cnpg | object | `{"clusterName":"invidious","enabled":false,"prependReleaseName":false}` | CloudNativePG database configuration |
| cnpg.clusterName | string | `"invidious"` | CNPG cluster name |
| cnpg.enabled | bool | `false` | Enable CNPG operator |
| cnpg.prependReleaseName | bool | `false` | Prepend release name and chart name to clusterName (use when CNPG cluster is deployed via rawResources) |
| companion | object | `{}` | Companion service configuration |
| companionKey | object | `{"value":"CHANGEMEplease16"}` | Companion service shared secret (exactly 16 alphanumeric characters) |
| companionKey.value | string | `"CHANGEMEplease16"` | Companion key value |
| config | object | `{"captcha_enabled":false,"domain":"mydomain.example","enable_user_notifications":true,"external_port":443,"hsts":true,"https_only":true,"log_level":"debug","login_enabled":true,"popular_enabled":true,"registration_enabled":true,"statistics_enabled":true}` | Invidious application configuration (rendered into the config file) |
| config.captcha_enabled | bool | `false` | Enable captcha |
| config.domain | string | `"mydomain.example"` | Public domain name |
| config.enable_user_notifications | bool | `true` | Enable user notifications |
| config.external_port | int | `443` | External port |
| config.hsts | bool | `true` | Enable HSTS |
| config.https_only | bool | `true` | Force HTTPS |
| config.log_level | string | `"debug"` | Log level |
| config.login_enabled | bool | `true` | Allow login |
| config.popular_enabled | bool | `true` | Enable popular feed |
| config.registration_enabled | bool | `true` | Allow registration |
| config.statistics_enabled | bool | `true` | Enable statistics |
| configSource | object | `{"pvc":{"accessMode":"ReadWriteOnce","size":"32Mi","storageClass":""},"type":"configMap"}` | How the invidious config is delivered to the pod |
| configSource.pvc | object | `{"accessMode":"ReadWriteOnce","size":"32Mi","storageClass":""}` | PVC settings (only used when type=pvc) |
| configSource.pvc.accessMode | string | `"ReadWriteOnce"` | Access mode |
| configSource.pvc.size | string | `"32Mi"` | Storage size for the config PVC |
| configSource.pvc.storageClass | string | `""` | StorageClass (empty = cluster default) |
| configSource.type | string | `"configMap"` | Source type: `configMap` renders an immutable ConfigMap from .Values.config. `pvc` provisions a PVC, seeds it from the rendered config on first start, and leaves further edits to the user (re-seed by deleting the PVC). |
| hmacKey | object | `{"value":"CHANGEME"}` | HMAC signing key configuration |
| hmacKey.value | string | `"CHANGEME"` | HMAC key value |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration |
| ingress.main.enabled | bool | `false` | Enable ingress |
| persistence | object | `{"cache":{"type":"emptyDir"}}` | Persistent storage configuration |
| persistence.cache | object | `{"type":"emptyDir"}` | Cache volume (emptyDir) |
| persistence.cache.type | string | `"emptyDir"` | Volume type |
| postgres | object | `{"auth":{"database":"invidious","password":"CHANGEME","username":"invidious"},"enabled":true,"persistence":{"accessMode":"ReadWriteOnce","enabled":true,"size":"2Gi"}}` | PostgreSQL database configuration (subchart) |
| postgres.auth | object | `{"database":"invidious","password":"CHANGEME","username":"invidious"}` | PostgreSQL authentication |
| postgres.auth.database | string | `"invidious"` | Database name |
| postgres.auth.password | string | `"CHANGEME"` | Database password |
| postgres.auth.username | string | `"invidious"` | Database username |
| postgres.enabled | bool | `true` | Enable packaged PostgreSQL |
| postgres.persistence | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"2Gi"}` | PostgreSQL persistence |
| postgres.persistence.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| postgres.persistence.enabled | bool | `true` | Enable PostgreSQL persistence |
| postgres.persistence.size | string | `"2Gi"` | PostgreSQL volume size |

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
