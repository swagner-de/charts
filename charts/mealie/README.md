# mealie

A Helm chart for [Mealie](https://mealie.io/), a self-hosted recipe manager and meal planner.

## Features

- PostgreSQL database (packaged subchart or CNPG operator)
- Optional OIDC authentication
- Optional SMTP email configuration
- Optional OpenAI integration for recipe parsing
- CNPG support with automatic database connection configuration

## Install

```bash
helm install mealie oci://ghcr.io/swagner-de/charts/mealie --version 0.15.0
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `persistence.data.size` | Data volume size | `10Gi` |
| `config.ALLOW_SIGNUP` | Allow new user signups | `"false"` |
| `db.POSTGRES_PASSWORD` | PostgreSQL password | `CHANGEME` |
| `postgres.enabled` | Enable packaged PostgreSQL | `true` |
| `cnpg.enabled` | Use CNPG operator instead | `false` |
| `secrets.mail.enabled` | Create SMTP secret | `false` |
| `secrets.oidc.enabled` | Create OIDC secret | `false` |
| `secrets.openai.enabled` | Create OpenAI secret | `false` |
| `ingress.main.enabled` | Enable ingress | `false` |

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
