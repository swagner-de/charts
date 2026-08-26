# linkwarden

![Version: 0.3.2](https://img.shields.io/badge/Version-0.3.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v2.16.1](https://img.shields.io/badge/AppVersion-v2.16.1-informational?style=flat-square)
Self-hosted collaborative bookmark manager to collect, organize and archive webpages
**Homepage:** <https://linkwarden.app/>

## Features
- PostgreSQL database (packaged subchart or CNPG operator)
- Optional Meilisearch full-text search engine
- Secrets injected via envFrom (packaged Secret or an external Secret)

## Install

```bash
helm install linkwarden oci://ghcr.io/swagner-de/charts/linkwarden
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |
| oci://registry-1.docker.io/cloudpirates | postgres | 0.20.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cnpg | object | `{"clusterName":"linkwarden-db","enabled":false,"prependReleaseName":false}` | CloudNativePG database configuration |
| cnpg.clusterName | string | `"linkwarden-db"` | CNPG cluster name |
| cnpg.enabled | bool | `false` | Enable CNPG operator integration |
| cnpg.prependReleaseName | bool | `false` | Prepend release name and chart name to clusterName (use when CNPG cluster is deployed via rawResources) |
| config | object | `{"NEXTAUTH_URL":"http://localhost:3000/api/v1/auth","NEXT_PUBLIC_DISABLE_REGISTRATION":"true"}` | Linkwarden application configuration (plain, non-secret env vars) See https://docs.linkwarden.app/self-hosting/environment-variables |
| config.NEXTAUTH_URL | string | `"http://localhost:3000/api/v1/auth"` | Public base URL used for auth callbacks. MUST end with /api/v1/auth. Override with your external URL, e.g. https://links.example.com/api/v1/auth |
| config.NEXT_PUBLIC_DISABLE_REGISTRATION | string | `"true"` | Disable public registration by default |
| db | object | `{"POSTGRES_DB":"linkwarden","POSTGRES_PASSWORD":"CHANGEME","POSTGRES_USER":"linkwarden"}` | Database credentials (used only when postgres.enabled; the packaged subchart path) |
| db.POSTGRES_DB | string | `"linkwarden"` | PostgreSQL database name |
| db.POSTGRES_PASSWORD | string | `"CHANGEME"` | PostgreSQL password |
| db.POSTGRES_USER | string | `"linkwarden"` | PostgreSQL username |
| envFromSecrets | list | `[]` | Existing secrets to inject as environment variables into the app (and Meilisearch) containers. Use this to provide NEXTAUTH_SECRET / MEILI_MASTER_KEY from an ExternalSecret. |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration |
| ingress.main.enabled | bool | `false` | Enable ingress |
| meilisearch | object | `{"enabled":true}` | Meilisearch full-text search engine (optional but recommended) |
| meilisearch.enabled | bool | `true` | Enable the packaged Meilisearch controller |
| persistence | object | `{"data":{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"},"meili":{"accessMode":"ReadWriteOnce","enabled":true,"size":"2Gi"}}` | Persistent storage configuration |
| persistence.data | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}` | Linkwarden data volume (uploaded files, archives, screenshots) mounted at /data/data |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.enabled | bool | `true` | Enable data persistence |
| persistence.data.size | string | `"10Gi"` | Data volume size |
| persistence.meili | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"2Gi"}` | Meilisearch data volume mounted at /meili_data (only used when meilisearch.enabled) |
| persistence.meili.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.meili.enabled | bool | `true` | Enable Meilisearch data persistence |
| persistence.meili.size | string | `"2Gi"` | Meilisearch volume size |
| postgres | object | `{"auth":{"database":"linkwarden","password":"CHANGEME","username":"linkwarden"},"enabled":true,"persistence":{"accessMode":"ReadWriteOnce","enabled":true,"size":"5Gi"}}` | PostgreSQL database configuration (packaged subchart) |
| postgres.auth | object | `{"database":"linkwarden","password":"CHANGEME","username":"linkwarden"}` | PostgreSQL authentication |
| postgres.auth.database | string | `"linkwarden"` | Database name |
| postgres.auth.password | string | `"CHANGEME"` | Database password |
| postgres.auth.username | string | `"linkwarden"` | Database username |
| postgres.enabled | bool | `true` | Enable packaged PostgreSQL |
| postgres.persistence | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"5Gi"}` | PostgreSQL persistence |
| postgres.persistence.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| postgres.persistence.enabled | bool | `true` | Enable PostgreSQL persistence |
| postgres.persistence.size | string | `"5Gi"` | PostgreSQL volume size |
| secrets | object | `{"app":{"enabled":true,"stringData":{"MEILI_MASTER_KEY":"CHANGEME","NEXTAUTH_SECRET":"CHANGEME"}}}` | Application secrets rendered into a Secret and injected via envFrom. For production, disable this and supply an existing Secret via envFromSecrets (e.g. an ExternalSecret) carrying NEXTAUTH_SECRET and MEILI_MASTER_KEY. |
| secrets.app.enabled | bool | `true` | Create the packaged application Secret |
| secrets.app.stringData.MEILI_MASTER_KEY | string | `"CHANGEME"` | Meilisearch master key (REQUIRED when meilisearch.enabled). >= 16 bytes. |
| secrets.app.stringData.NEXTAUTH_SECRET | string | `"CHANGEME"` | NextAuth session secret (REQUIRED). Generate with `openssl rand -base64 32`. |

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
