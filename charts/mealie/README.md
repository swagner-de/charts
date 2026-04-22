# mealie

![Version: 0.17.2](https://img.shields.io/badge/Version-0.17.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v3.16.0](https://img.shields.io/badge/AppVersion-v3.16.0-informational?style=flat-square)
Self-hosted recipe manager and meal planner
**Homepage:** <https://mealie.io/>

## Features
- PostgreSQL database (packaged subchart or CNPG operator)
- Optional OIDC authentication
- Optional SMTP email configuration
- Optional OpenAI integration for recipe parsing
- CNPG support with automatic database connection configuration

## Install

```bash
helm install mealie oci://ghcr.io/swagner-de/charts/mealie
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |
| oci://registry-1.docker.io/cloudpirates | postgres | 0.19.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cnpg | object | `{"clusterName":"mealie-db","enabled":false}` | CloudNativePG database configuration |
| cnpg.clusterName | string | `"mealie-db"` | CNPG cluster name |
| cnpg.enabled | bool | `false` | Enable CNPG operator |
| config | object | `{"ALLOW_SIGNUP":"false"}` | Mealie application configuration |
| config.ALLOW_SIGNUP | string | `"false"` | Allow new user signups |
| db | object | `{"POSTGRES_DB":"mealie","POSTGRES_PASSWORD":"CHANGEME","POSTGRES_USER":"mealie"}` | Database credentials |
| db.POSTGRES_DB | string | `"mealie"` | PostgreSQL database name |
| db.POSTGRES_PASSWORD | string | `"CHANGEME"` | PostgreSQL password |
| db.POSTGRES_USER | string | `"mealie"` | PostgreSQL username |
| envFromSecrets | string | `nil` | Additional secrets to mount as environment variables |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration |
| ingress.main.enabled | bool | `false` | Enable ingress |
| mail | object | `{"SMTP_HOST":"None"}` | SMTP mail configuration See https://docs.mealie.io/documentation/getting-started/installation/backend-config/#email |
| mail.SMTP_HOST | string | `"None"` | SMTP host |
| oidc | string | `nil` | OIDC authentication configuration See https://docs.mealie.io/documentation/getting-started/installation/backend-config/#oidc |
| openai | object | `{}` | OpenAI integration configuration |
| persistence | object | `{"data":{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}}` | Persistent storage configuration |
| persistence.data | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}` | Data volume |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.enabled | bool | `true` | Enable data persistence |
| persistence.data.size | string | `"10Gi"` | Data volume size |
| postgres | object | `{"auth":{"database":"mealie","password":"CHANGEME","username":"mealie"},"enabled":true,"persistence":{"accessMode":"ReadWriteOnce","enabled":true,"size":"2Gi"}}` | PostgreSQL database configuration (subchart) |
| postgres.auth | object | `{"database":"mealie","password":"CHANGEME","username":"mealie"}` | PostgreSQL authentication |
| postgres.auth.database | string | `"mealie"` | Database name |
| postgres.auth.password | string | `"CHANGEME"` | Database password |
| postgres.auth.username | string | `"mealie"` | Database username |
| postgres.enabled | bool | `true` | Enable packaged PostgreSQL |
| postgres.persistence | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"2Gi"}` | PostgreSQL persistence |
| postgres.persistence.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| postgres.persistence.enabled | bool | `true` | Enable PostgreSQL persistence |
| postgres.persistence.size | string | `"2Gi"` | PostgreSQL volume size |
| secrets | object | `{"mail":{"enabled":false,"stringData":null},"oidc":{"enabled":false,"stringData":null},"openai":{"enabled":false,"stringData":null}}` | Kubernetes secrets configuration |
| secrets.mail | object | `{"enabled":false,"stringData":null}` | SMTP mail secret |
| secrets.mail.enabled | bool | `false` | Enable mail secret creation |
| secrets.mail.stringData | string | `nil` | Mail secret data |
| secrets.oidc | object | `{"enabled":false,"stringData":null}` | OIDC authentication secret |
| secrets.oidc.enabled | bool | `false` | Enable OIDC secret creation |
| secrets.oidc.stringData | string | `nil` | OIDC secret data |
| secrets.openai | object | `{"enabled":false,"stringData":null}` | OpenAI integration secret |
| secrets.openai.enabled | bool | `false` | Enable OpenAI secret creation |
| secrets.openai.stringData | string | `nil` | OpenAI secret data |

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
