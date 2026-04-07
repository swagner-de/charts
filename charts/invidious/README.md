# invidious

A Helm chart for [Invidious](https://github.com/iv-org/invidious), a privacy-focused alternative YouTube frontend.

## Features

- Invidious server with companion service for enhanced video playback
- PostgreSQL database (packaged subchart or CNPG operator)
- HMAC-signed companion key authentication
- CNPG support with automatic database connection configuration

## Install

```bash
helm install invidious oci://ghcr.io/swagner-de/charts/invidious --version 0.4.9
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `config.domain` | Public domain name | `mydomain.example` |
| `config.https_only` | Force HTTPS | `true` |
| `companionKey.value` | Shared secret for companion service | `CHANGEME` |
| `companionKey.existingSecret` | Use existing Secret for companion key | — |
| `hmacKey.value` | HMAC signing key | `CHANGEME` |
| `hmacKey.existingSecret` | Use existing Secret for HMAC key | — |
| `companion.publicUrl` | Public URL for the companion service | — |
| `postgres.enabled` | Enable packaged PostgreSQL | `true` |
| `postgres.auth.password` | PostgreSQL password | `CHANGEME` |
| `cnpg.enabled` | Use CNPG operator instead of packaged Postgres | `false` |
| `cnpg.clusterName` | CNPG cluster name | `invidious` |
| `ingress.main.enabled` | Enable ingress | `false` |

### Using Existing Secrets

For production, use existing Kubernetes Secrets instead of inline values:

```yaml
companionKey:
  existingSecret:
    name: invidious-companion-key
    key: key

hmacKey:
  existingSecret:
    name: invidious-hmac-key
    key: key
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
