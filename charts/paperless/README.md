# paperless

A Helm chart for [Paperless-ngx](https://docs.paperless-ngx.com/), a document management system with OCR and full-text search.

## Features

- PostgreSQL database (packaged subchart or CNPG operator)
- Redis for task queue (packaged subchart or external)
- Gotenberg and Tika sidecars for document conversion
- Init container for directory setup
- Configurable OCR settings, consumer polling, and barcode scanning
- OIDC authentication support

## Install

```bash
helm install paperless oci://ghcr.io/swagner-de/charts/paperless --version 0.39.0
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `persistence.data.size` | Document storage volume size | `100Gi` |
| `persistence.consumption.size` | Consumption directory volume size | `1Gi` |
| `paperless.admin.existingSecret` | Existing secret with admin credentials | `""` |
| `paperless.config` | Paperless config (converted to `PAPERLESS_*` env vars) | `{}` |
| `redis.enabled` | Enable packaged Redis | `true` |
| `redis.auth.password` | Redis password | `CHANGEME` |
| `postgres.enabled` | Enable packaged PostgreSQL | `true` |
| `postgres.auth.password` | PostgreSQL password | `CHANGEME` |
| `cnpg.enabled` | Use CNPG operator instead | `false` |
| `ingress.main.enabled` | Enable ingress | `false` |

### Paperless Configuration

All keys under `paperless.config` are converted to uppercase environment variables prefixed with `PAPERLESS_`:

```yaml
paperless:
  config:
    ocr_language: "deu"
    time_zone: "Europe/Berlin"
    consumer_polling: "60"
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
