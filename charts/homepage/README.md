# homepage

A Helm chart for [Homepage](https://gethomepage.dev/), a modern, fully static, fast, and secure application dashboard.

## Features

- Kubernetes service discovery via ClusterRole RBAC (Gateway API, Ingress, Traefik IngressRoutes)
- Configuration via `values.yaml` — services, bookmarks, widgets, and settings
- Secret variable injection (`HOMEPAGE_VAR_*` / `HOMEPAGE_FILE_*`) for API keys
- Read-only root filesystem with emptyDir for runtime config

## Install

```bash
helm install homepage oci://ghcr.io/swagner-de/charts/homepage --version 0.5.2
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `config.kubernetes.mode` | Kubernetes discovery mode | `cluster` |
| `config.settings` | Homepage settings (language, title, etc.) | `{}` |
| `config.services` | Service definitions | `[]` |
| `config.bookmarks` | Bookmark definitions | `[]` |
| `config.widgets` | Widget definitions | `[]` |
| `secretReplacements` | Key-value pairs for `HOMEPAGE_VAR_*` secrets | `{}` |
| `additionalSecrets` | External secrets to mount as env vars | `[]` |

### Secret Variable Injection

Use `secretReplacements` to inject sensitive values:

```yaml
secretReplacements:
  HOMEPAGE_VAR_API_KEY: "your-api-key"
```

These replace `{{HOMEPAGE_VAR_API_KEY}}` placeholders in your config.

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsUser: 1000`, `runAsGroup: 1000`
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
- ServiceAccount with minimal ClusterRole for Kubernetes resource discovery
