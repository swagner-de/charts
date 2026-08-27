# dawarich

![Version: 0.2.0](https://img.shields.io/badge/Version-0.2.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.14.0](https://img.shields.io/badge/AppVersion-1.14.0-informational?style=flat-square)
Self-hosted location history tracker and Google Timeline alternative
**Homepage:** <https://dawarich.app/>

## Features
- Single pod running both the Rails/Puma web server (`:3000`) and the Sidekiq
  worker, sharing the storage/tmp volumes
- PostgreSQL/PostGIS database via the CloudNativePG operator (required)
- Packaged Redis subchart (or bring your own via `redis.external`)
- Env-based OpenID Connect and SMTP, injected via `envFromSecrets`

## Database
This chart **requires PostGIS**. The packaged plain `postgres` subchart is
intentionally **not supported** — use the CloudNativePG operator and provide a
PostGIS cluster via `rawResources`, e.g.:

```yaml
cnpg:
  enabled: true
  clusterName: dawarich-db
rawResources:
  db:
    enabled: true
    manifest:
      apiVersion: postgresql.cnpg.io/v1
      kind: Cluster
      spec:
        instances: 1
        imageName: ghcr.io/cloudnative-pg/postgis:17-3.5
        bootstrap:
          initdb:
            postInitApplicationSQL:
              - CREATE EXTENSION IF NOT EXISTS postgis;
        storage:
          size: 5Gi
```

The `postInitApplicationSQL` line is required: the CNPG application role is not a
superuser, so Dawarich cannot create the `postgis` extension itself.

## Redis
The packaged Redis subchart is enabled by default with authentication. To use an
existing Redis instead:

```yaml
redis:
  enabled: false
  external:
    host: my-redis.example.com
    secretKeyRef:
      name: my-redis
      key: redis-password
```

## Install

```bash
helm install dawarich oci://ghcr.io/swagner-de/charts/dawarich
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |
| oci://registry-1.docker.io/cloudpirates | redis | 0.34.23 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| applicationHosts | string | `"localhost,::1,127.0.0.1"` | Comma-separated hostnames Rails accepts (Rack host authorization). Include your external hostname here, e.g. "dawarich.example.com". |
| applicationProtocol | string | `"http"` | Protocol Dawarich generates absolute URLs with (http or https). Set to "https" when served behind a TLS-terminating gateway/ingress. |
| backgroundProcessingConcurrency | int | `3` | Number of Sidekiq job threads on the worker container. |
| cnpg | object | `{"clusterName":"dawarich-db","enabled":true,"prependReleaseName":false}` | CloudNativePG database configuration. This chart REQUIRES PostGIS; the packaged (plain) postgres subchart is intentionally NOT supported. Provide the cluster via rawResources using a PostGIS image and seed the extension via bootstrap.initdb.postInitApplicationSQL. |
| cnpg.clusterName | string | `"dawarich-db"` | CNPG cluster name (must match the rendered rawResources Cluster name) |
| cnpg.enabled | bool | `true` | Enable CNPG operator integration (required) |
| cnpg.prependReleaseName | bool | `false` | Prepend release name and chart name to clusterName (use when the CNPG cluster is deployed via rawResources and the release name differs from the chart name) |
| config | object | `{}` | Extra environment variables (plain, non-secret). Use for OIDC display/redirect settings, SMTP host settings, etc. See https://dawarich.app/docs/self-hosting/environment-variables/ |
| envFromSecrets | list | `[]` | Existing secrets injected as environment variables into both containers. Use this to provide SECRET_KEY_BASE, OIDC client credentials, and SMTP credentials from ExternalSecrets. Each entry: {name: <secret>, optional: <bool>}. |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration (disabled by default; prefer the Gateway API `route`). |
| ingress.main.enabled | bool | `false` | Enable ingress |
| networkPolicy | object | `{"enabled":true,"gatewayNamespace":"envoy-gateway-system"}` | NetworkPolicy configuration. When enabled, restricts the pod to: ingress from the gateway namespace (to :3000), and egress to DNS, the CNPG database, Redis, and outbound HTTP(S) (reverse geocoding, OIDC, email). |
| networkPolicy.enabled | bool | `true` | Create a NetworkPolicy for the app pod |
| networkPolicy.gatewayNamespace | string | `"envoy-gateway-system"` | Namespace of the Gateway/ingress controller allowed to reach the app |
| persistence | object | `{"storage":{"accessMode":"ReadWriteOnce","enabled":true,"existingClaim":"","size":"10Gi"}}` | Persistent storage configuration. |
| persistence.storage | object | `{"accessMode":"ReadWriteOnce","enabled":true,"existingClaim":"","size":"10Gi"}` | ActiveStorage data (imported files, exports, generated data), mounted at /var/app/storage and shared by the web and worker containers. |
| persistence.storage.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.storage.enabled | bool | `true` | Enable storage persistence |
| persistence.storage.existingClaim | string | `""` | Use an existing PVC instead of provisioning one |
| persistence.storage.size | string | `"10Gi"` | Storage volume size |
| prometheus | object | `{"enabled":false}` | Prometheus / Yabeda metrics exporter. |
| prometheus.enabled | bool | `false` | Enable the in-process Prometheus exporter |
| redis | object | `{"architecture":"standalone","auth":{"enabled":true,"existingSecret":"","existingSecretPasswordKey":"redis-password","password":"CHANGEME"},"enabled":true,"external":{"host":"","secretKeyRef":{}},"persistence":{"enabled":true,"size":"1Gi"}}` | Redis configuration. Uses the packaged Redis subchart by default; set `enabled: false` and fill in `external` to use an existing Redis. |
| redis.architecture | string | `"standalone"` | Redis architecture |
| redis.auth | object | `{"enabled":true,"existingSecret":"","existingSecretPasswordKey":"redis-password","password":"CHANGEME"}` | Redis authentication |
| redis.auth.enabled | bool | `true` | Enable Redis authentication |
| redis.auth.existingSecret | string | `""` | Use an existing Secret for the Redis password (instead of `password`) |
| redis.auth.existingSecretPasswordKey | string | `"redis-password"` | Key in the existing Secret holding the Redis password (also the key in the auto-generated Secret when the packaged Redis manages its own password) |
| redis.auth.password | string | `"CHANGEME"` | Redis password |
| redis.enabled | bool | `true` | Enable the packaged Redis subchart |
| redis.external | object | `{"host":"","secretKeyRef":{}}` | External Redis (used only when `redis.enabled: false`) |
| redis.external.host | string | `""` | External Redis host |
| redis.external.secretKeyRef | object | `{}` | Reference to a Secret holding the external Redis password |
| redis.persistence | object | `{"enabled":true,"size":"1Gi"}` | Redis persistence |
| redis.persistence.enabled | bool | `true` | Enable Redis persistence |
| redis.persistence.size | string | `"1Gi"` | Redis volume size |
| secrets | object | `{"app":{"enabled":true,"stringData":{"SECRET_KEY_BASE":"CHANGEME"}}}` | Application secrets rendered into a Secret and injected via envFrom. For production, disable this and supply an existing Secret via envFromSecrets (e.g. an ExternalSecret) carrying SECRET_KEY_BASE. |
| secrets.app.enabled | bool | `true` | Create the packaged application Secret |
| secrets.app.stringData.SECRET_KEY_BASE | string | `"CHANGEME"` | Rails secret key base (REQUIRED). Generate with `openssl rand -hex 64`. A stable value is required — a changing key invalidates all sessions. |
| storeGeodata | bool | `true` | Store reverse-geocoding results in the database. |
| timeZone | string | `"Europe/London"` | Time zone used for displaying timestamps. |
| webConcurrency | int | `1` | Number of preloaded Puma workers on the web container. One is plenty for a household instance; raise for busier deployments. |

## Authentication (OIDC)
Dawarich supports OpenID Connect via environment variables. Provide the client
credentials and issuer through an external Secret referenced in `envFromSecrets`
(`OIDC_CLIENT_ID`, `OIDC_CLIENT_SECRET`, `OIDC_ISSUER`) and set the non-secret
options in `config` (`OIDC_REDIRECT_URI`, `OIDC_PROVIDER_NAME`, …). The callback
URL is `<applicationProtocol>://<host>/users/auth/openid_connect/callback`.

## Email (SMTP)
SMTP is configured via environment variables. Set the host settings in `config`
(`SMTP_SERVER`, `SMTP_PORT`, `SMTP_DOMAIN`, `SMTP_STARTTLS`/`SMTP_SSL`,
`SMTP_AUTHENTICATION`) and supply `SMTP_USERNAME`/`SMTP_PASSWORD` via an external
Secret in `envFromSecrets`.

## Security
The web and worker containers run fully hardened: `runAsNonRoot: true` (UID/GID
1000), `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, all
capabilities dropped, seccomp `RuntimeDefault`. Writable paths (`/var/app/tmp`,
`/var/app/log`, `/var/app/public`) are `emptyDir`s; `/var/app/storage` is the
persistent volume.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
