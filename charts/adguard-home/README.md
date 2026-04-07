# adguard-home

A Helm chart for [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome), a network-wide DNS ad and tracker blocker.

## Features

- Runs as non-root user (UID 1000) with read-only root filesystem
- Init container automatically configures DNS to listen on port 5353 (no `NET_BIND_SERVICE` needed)
- Service exposes DNS on port 53 (mapped to 5353 internally), DNS-over-TLS (853), and DNS-over-QUIC (853/UDP)
- Optional Prometheus exporter sidecar for metrics
- Optional ServiceMonitor for Prometheus Operator

## Install

```bash
helm install adguard-home oci://ghcr.io/swagner-de/charts/adguard-home --version 0.5.5
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `persistence.config.enabled` | Enable persistent storage for config | `true` |
| `persistence.config.size` | Config volume size | `4Gi` |
| `persistence.data.enabled` | Enable persistent storage for data | `true` |
| `persistence.data.size` | Data volume size | `4Gi` |
| `persistence.certs.enabled` | Mount TLS certificates from a Secret | `false` |
| `serviceMonitor.main.enabled` | Enable Prometheus ServiceMonitor | `false` |

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

This chart runs with restrictive security defaults:

- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

DNS is served on port 5353 internally (mapped to 53 via the Service) so no `NET_BIND_SERVICE` capability is required.
