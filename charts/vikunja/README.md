# vikunja

![Version: 0.1.2](https://img.shields.io/badge/Version-0.1.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2.5.0](https://img.shields.io/badge/AppVersion-2.5.0-informational?style=flat-square)
Self-hosted to-do list and task management application
**Homepage:** <https://vikunja.io/>

## Features
- PostgreSQL database (packaged subchart or CNPG operator)
- Optional OIDC authentication (via a mounted config file + env vars)
- Optional SMTP email configuration
- Persistent file attachment storage
- CNPG support with automatic database connection configuration

## Install

```bash
helm install vikunja oci://ghcr.io/swagner-de/charts/vikunja
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |
| oci://registry-1.docker.io/cloudpirates | postgres | 0.20.4 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cnpg | object | `{"clusterName":"vikunja-db","enabled":false,"prependReleaseName":false}` | CloudNativePG database configuration |
| cnpg.clusterName | string | `"vikunja-db"` | CNPG cluster name |
| cnpg.enabled | bool | `false` | Enable CNPG operator |
| cnpg.prependReleaseName | bool | `false` | Prepend release name and chart name to clusterName (use when CNPG cluster is deployed via rawResources) |
| config | object | `{"VIKUNJA_FILES_BASEPATH":"/app/vikunja/files","VIKUNJA_SERVICE_TIMEZONE":"UTC"}` | Non-secret application configuration, rendered to a ConfigMap and loaded as environment variables. Use Vikunja's `VIKUNJA_*` variable names. See https://vikunja.io/docs/config-options/ |
| config.VIKUNJA_FILES_BASEPATH | string | `"/app/vikunja/files"` | Path where uploaded file attachments are stored (matches the data volume) |
| config.VIKUNJA_SERVICE_TIMEZONE | string | `"UTC"` | Default timezone (tz database format) |
| configFile | object | `{"content":"","enabled":false}` | Optional config file mounted at /etc/vikunja/config.yml. Required for OIDC: Vikunja only reads a provider's env vars if the provider key is declared in a config file. The client id/secret/authurl are then supplied via env vars (VIKUNJA_AUTH_OPENID_PROVIDERS_<KEY>_*) from a secret. See https://vikunja.io/docs/config-options/#auth |
| configFile.content | string | `""` | Config file contents (YAML) |
| configFile.enabled | bool | `false` | Enable mounting a config file |
| db | object | `{"VIKUNJA_DATABASE_DATABASE":"vikunja","VIKUNJA_DATABASE_PASSWORD":"CHANGEME","VIKUNJA_DATABASE_USER":"vikunja"}` | Database credentials. Mounted as environment variables and reused by the packaged postgres subchart. Ignored when `cnpg.enabled` is true. |
| db.VIKUNJA_DATABASE_DATABASE | string | `"vikunja"` | PostgreSQL database name |
| db.VIKUNJA_DATABASE_PASSWORD | string | `"CHANGEME"` | PostgreSQL password |
| db.VIKUNJA_DATABASE_USER | string | `"vikunja"` | PostgreSQL username |
| envFromSecrets | list | `[]` | Additional secrets to mount as environment variables (e.g. OIDC, JWT, mail) |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration |
| ingress.main.enabled | bool | `false` | Enable ingress |
| mail | object | `{}` | Non-secret SMTP configuration (VIKUNJA_MAILER_*). See https://vikunja.io/docs/config-options/#mailer |
| persistence | object | `{"data":{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}}` | Persistent storage configuration |
| persistence.data | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}` | File attachments volume (mounted at /app/vikunja/files) |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.enabled | bool | `true` | Enable data persistence |
| persistence.data.size | string | `"10Gi"` | Data volume size |
| postgres | object | `{"auth":{"database":"vikunja","password":"CHANGEME","username":"vikunja"},"enabled":true,"persistence":{"accessMode":"ReadWriteOnce","enabled":true,"size":"2Gi"}}` | PostgreSQL database configuration (packaged subchart) |
| postgres.auth | object | `{"database":"vikunja","password":"CHANGEME","username":"vikunja"}` | PostgreSQL authentication |
| postgres.auth.database | string | `"vikunja"` | Database name |
| postgres.auth.password | string | `"CHANGEME"` | Database password |
| postgres.auth.username | string | `"vikunja"` | Database username |
| postgres.enabled | bool | `true` | Enable packaged PostgreSQL |
| postgres.persistence | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"2Gi"}` | PostgreSQL persistence |
| postgres.persistence.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| postgres.persistence.enabled | bool | `true` | Enable PostgreSQL persistence |
| postgres.persistence.size | string | `"2Gi"` | PostgreSQL volume size |
| secrets | object | `{"jwt":{"enabled":true,"stringData":{"VIKUNJA_SERVICE_SECRET":"CHANGEME"}},"mail":{"enabled":false,"stringData":null}}` | Kubernetes secrets configuration |
| secrets.jwt | object | `{"enabled":true,"stringData":{"VIKUNJA_SERVICE_SECRET":"CHANGEME"}}` | JWT signing secret. Must be stable across restarts or sessions are invalidated. In production supply it via `envFromSecrets` instead. |
| secrets.jwt.enabled | bool | `true` | Enable JWT secret creation |
| secrets.jwt.stringData | object | `{"VIKUNJA_SERVICE_SECRET":"CHANGEME"}` | JWT secret data |
| secrets.mail | object | `{"enabled":false,"stringData":null}` | SMTP mail credentials secret |
| secrets.mail.enabled | bool | `false` | Enable mail secret creation |
| secrets.mail.stringData | string | `nil` | Mail secret data |

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
