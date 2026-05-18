# adguard-home

![Version: 0.6.3](https://img.shields.io/badge/Version-0.6.3-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 0.107.74](https://img.shields.io/badge/AppVersion-0.107.74-informational?style=flat-square)
DNS-level ad and tracker blocking with optional Prometheus exporter
**Homepage:** <https://github.com/AdguardTeam/AdGuardHome>

## Features
- Runs as non-root user (UID 1000) with read-only root filesystem
- Init container automatically configures DNS to listen on port 5353 (no `NET_BIND_SERVICE` needed)
- Service exposes DNS on port 53 (mapped to 5353 internally), DNS-over-TLS (853), and DNS-over-QUIC (853/UDP)
- Optional Prometheus exporter sidecar for metrics
- Optional ServiceMonitor for Prometheus Operator

## Install

```bash
helm install adguard-home oci://ghcr.io/swagner-de/charts/adguard-home
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| persistence | object | `{"certs":{"enabled":false},"config":{"accessMode":"ReadWriteOnce","enabled":true,"size":"4Gi"},"data":{"accessMode":"ReadWriteOnce","enabled":true,"size":"4Gi"}}` | Persistent storage configuration |
| persistence.certs | object | `{"enabled":false}` | TLS certificate volume |
| persistence.certs.enabled | bool | `false` | Enable TLS certificate mount from a Secret |
| persistence.config | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"4Gi"}` | Configuration volume |
| persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.config.enabled | bool | `true` | Enable persistent storage for config |
| persistence.config.size | string | `"4Gi"` | Volume size |
| persistence.data | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"4Gi"}` | Data volume |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.enabled | bool | `true` | Enable persistent storage for data |
| persistence.data.size | string | `"4Gi"` | Volume size |
| serviceMonitor | object | `{"main":{"enabled":false}}` | Prometheus ServiceMonitor configuration |
| serviceMonitor.main.enabled | bool | `false` | Enable Prometheus ServiceMonitor |

## Security
This chart runs with restrictive security defaults:

- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

DNS is served on port 5353 internally (mapped to 53 via the Service) so no `NET_BIND_SERVICE` capability is required.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
