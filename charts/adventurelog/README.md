# adventurelog

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v0.13.0](https://img.shields.io/badge/AppVersion-v0.13.0-informational?style=flat-square)
Self-hosted travel companion to log trips, plan itineraries, and map your adventures
**Homepage:** <https://adventurelog.app/>

## Features
- Split deployment: hardened SvelteKit frontend + Django backend in one pod
- PostgreSQL/PostGIS database via the CloudNativePG operator (required)
- Secrets injected via envFrom (packaged Secret or an external Secret)

## Database
This chart **requires PostGIS** (the backend uses GeoDjango). The packaged plain
`postgres` subchart is intentionally **not supported** — use the CloudNativePG
operator and provide a PostGIS cluster via `rawResources`, e.g.:

```yaml
cnpg:
  enabled: true
  clusterName: adventurelog-db
rawResources:
  db:
    enabled: true
    manifest:
      apiVersion: postgresql.cnpg.io/v1
      kind: Cluster
      spec:
        instances: 1
        imageName: ghcr.io/cloudnative-pg/postgis:16-3.5
        bootstrap:
          initdb:
            postInitApplicationSQL:
              - CREATE EXTENSION IF NOT EXISTS postgis;
        storage:
          size: 5Gi
```

The `postInitApplicationSQL` line is required: the CNPG application role is not a
superuser, so GeoDjango cannot create the `postgis` extension itself.

## Routing
The frontend is the public entrypoint. The reverse proxy in front (Ingress or a
Gateway API `HTTPRoute`) must route the backend paths to the `backend` service:

- `/media`, `/static`, `/protectedMedia`, `/admin`, `/accounts` → `<release>-backend:80`
- everything else → `<release>-main:3000`

## Install

```bash
helm install adventurelog oci://ghcr.io/swagner-de/charts/adventurelog
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| admin | object | `{"email":"admin@example.com","username":"admin"}` | Default Django admin/superuser account, created on first start. Username and email are non-secret; the password comes from the Secret below. |
| admin.email | string | `"admin@example.com"` | Admin email |
| admin.username | string | `"admin"` | Admin username |
| cnpg | object | `{"clusterName":"adventurelog-db","enabled":true,"prependReleaseName":false}` | CloudNativePG database configuration. This chart REQUIRES PostGIS (GeoDjango); the packaged (plain) postgres subchart is intentionally NOT supported. Provide the cluster via rawResources using a PostGIS image and seed the extension via bootstrap.initdb.postInitApplicationSQL. |
| cnpg.clusterName | string | `"adventurelog-db"` | CNPG cluster name (must match the rendered rawResources Cluster name) |
| cnpg.enabled | bool | `true` | Enable CNPG operator integration (required) |
| cnpg.prependReleaseName | bool | `false` | Prepend release name and chart name to clusterName (use when the CNPG cluster is deployed via rawResources and the release name differs from the chart name) |
| config | object | `{}` | Extra backend (Django) environment variables (plain, non-secret). See https://adventurelog.app/docs/configuration/environment_variables.html |
| envFromSecrets | list | `[]` | Existing secrets injected as environment variables into the backend container. Use this to provide SECRET_KEY / DJANGO_ADMIN_PASSWORD from an ExternalSecret. Each entry: {name: <secret>, optional: <bool>}. Set optional: true for a secret that may not exist yet (e.g. an ExternalSecret pending its upstream item). |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration (disabled by default; prefer the Gateway API `route`) |
| ingress.main.enabled | bool | `false` | Enable ingress |
| networkPolicy | object | `{"enabled":true,"gatewayNamespace":"envoy-gateway-system"}` | NetworkPolicy configuration. When enabled, restricts the pod to: ingress from the gateway namespace (to the frontend/backend ports), and egress to DNS, the CNPG database, and outbound HTTP(S) (first-run country data, geocoding, OIDC, email). |
| networkPolicy.enabled | bool | `true` | Create a NetworkPolicy for the app pod |
| networkPolicy.gatewayNamespace | string | `"envoy-gateway-system"` | Namespace of the Gateway/ingress controller allowed to reach the app |
| persistence | object | `{"media":{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}}` | Persistent storage configuration |
| persistence.media | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}` | Backend media volume (uploaded photos, GPX tracks, generated images) mounted at /code/media |
| persistence.media.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.media.enabled | bool | `true` | Enable media persistence |
| persistence.media.size | string | `"10Gi"` | Media volume size |
| secrets | object | `{"app":{"enabled":true,"stringData":{"DJANGO_ADMIN_PASSWORD":"CHANGEME","SECRET_KEY":"CHANGEME"}}}` | Application secrets rendered into a Secret and injected via envFrom into the backend. For production, disable this and supply an existing Secret via envFromSecrets (e.g. an ExternalSecret) carrying SECRET_KEY and DJANGO_ADMIN_PASSWORD. |
| secrets.app.enabled | bool | `true` | Create the packaged application Secret |
| secrets.app.stringData.DJANGO_ADMIN_PASSWORD | string | `"CHANGEME"` | Django admin/superuser password (REQUIRED). You need this to log into the Django admin panel (e.g. to configure OIDC providers). |
| secrets.app.stringData.SECRET_KEY | string | `"CHANGEME"` | Django secret key (REQUIRED). Generate with `openssl rand -base64 48`. If left unset the image generates an ephemeral key that changes on every restart, invalidating all sessions — always provide a stable value. |
| siteUrl | string | `"http://localhost:8015"` | Public base URL AdventureLog is served on. Used to derive the frontend ORIGIN, backend CSRF_TRUSTED_ORIGINS, and media/OAuth callback URLs. Override with your external URL, e.g. https://adventurelog.example.com |

## OIDC / SSO
AdventureLog supports OpenID Connect, but providers are configured **at runtime in
the Django admin panel** under *Social applications* — there are no environment
variables for the issuer/client id/secret, so this chart does not expose them. Set
`siteUrl` to your external URL and, after deploy, log into `<siteUrl>/admin` with
the admin account to add your provider. The callback URL is
`<siteUrl>/accounts/oidc/<provider-id>/login/callback/`.

## Security
- **Frontend**: `runAsNonRoot: true` (UID/GID 1000), `readOnlyRootFilesystem: true`,
  `allowPrivilegeEscalation: false`, all capabilities dropped, seccomp `RuntimeDefault`.
- **Backend**: runs as root with a writable root filesystem — its image bundles
  supervisord/nginx/cron/memcached which require it. Seccomp `RuntimeDefault` is kept.
  See `.trivyignore.yaml` for the documented exceptions.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
