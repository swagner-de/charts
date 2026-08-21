# wanderer

![Version: 0.4.0](https://img.shields.io/badge/Version-0.4.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v0.20.0](https://img.shields.io/badge/AppVersion-v0.20.0-informational?style=flat-square)
Self-hosted trail database and route planner for hiking, biking and other outdoor activities
**Homepage:** <https://wanderer.to/>

## Features
- All three wanderer services in one hardened pod: meilisearch (search),
  PocketBase backend (db) and the SvelteKit frontend (web), talking over localhost
- Separate PVCs for the search index, PocketBase data/plugins and uploaded media
- Secrets injected via envFrom (packaged Secret or an external Secret)

## Routing
wanderer needs **two** externally reachable hosts, because the frontend talks to
PocketBase **directly from the browser** (`PUBLIC_POCKETBASE_URL`):

- `origin` (e.g. `https://wanderer.example.com`) → `<release>-main:3000` (frontend)
- `pocketbaseUrl` (e.g. `https://wanderer-db.example.com`) → `<release>-pocketbase:8090`

Set both values to their external URLs and point an Ingress or a Gateway API
`HTTPRoute` at each service. `origin` must be identical on the frontend and
PocketBase, or PocketBase rejects the web UI's form submissions with
*"Cross-site POST form submissions are forbidden"*.

## Secrets
Two secrets are required (see `values.yaml`):

- `MEILI_MASTER_KEY` — meilisearch master key, min 16 bytes, shared by all services
- `POCKETBASE_ENCRYPTION_KEY` — PocketBase AES key, **exactly 32 characters**

Provide them via the packaged Secret (`secrets.app`) or an external Secret listed
under `envFromSecrets`.

## Install

```bash
helm install wanderer oci://ghcr.io/swagner-de/charts/wanderer
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| db | object | `{"config":{}}` | Extra environment variables for the PocketBase (db) container, e.g. SMTP (POCKETBASE_SMTP_*) or the plugin sync schedule. See https://wanderer.to/run/environment-configuration/#pocketbase |
| envFromSecrets | list | `[]` | Existing secrets injected as environment variables into all three containers. Use this to provide MEILI_MASTER_KEY / POCKETBASE_ENCRYPTION_KEY (and optional POCKETBASE_SMTP_* credentials) from an ExternalSecret. Each entry: {name: <secret>, optional: <bool>}. |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration (disabled by default; prefer the Gateway API `route`). wanderer needs TWO routes: the frontend (-> main:3000) and PocketBase (-> pocketbase:8090), on separate hostnames. |
| ingress.main.enabled | bool | `false` | Enable ingress |
| origin | string | `"http://localhost:3000"` | Public URL the wanderer web frontend is served on. Sets ORIGIN for the frontend and PocketBase (must match, or PocketBase rejects cross-site form submissions). Override with your external URL, e.g. https://wanderer.example.com |
| persistence | object | `{"db":{"accessMode":"ReadWriteOnce","enabled":true,"size":"5Gi"},"search":{"accessMode":"ReadWriteOnce","enabled":true,"size":"5Gi"},"uploads":{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}}` | Persistent storage configuration |
| persistence.db | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"5Gi"}` | PocketBase data + installed plugins, mounted at /pb_data and /data/plugins |
| persistence.search | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"5Gi"}` | Meilisearch index data, mounted at /meili_data |
| persistence.uploads | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}` | User-uploaded trail media, mounted at /app/uploads |
| plugins | object | `{"bundles":[],"enabled":false,"version":""}` | PocketBase provider plugins to install at startup. When enabled with a non-empty bundle list, an init container downloads each bundle from the wanderer GitHub release, verifies it against the release SHA256SUMS, and extracts it into /data/plugins (persisted on the db PVC). It re-downloads on every start, keeping plugins pinned to the release tag. See https://wanderer.to/run/installation/plugins |
| plugins.bundles | list | `[]` | Plugin bundle ids to install (release assets named wanderer-plugin-<id>.tar.gz), e.g. [strava, komoot, hammerhead] |
| plugins.enabled | bool | `false` | Enable the plugin-install init container |
| plugins.version | string | `""` | Release tag to pull bundles from. Empty uses the chart appVersion, so plugins stay in lockstep with the app image. |
| pocketbaseUrl | string | `"http://localhost:8090"` | Public URL of the PocketBase backend (sets PUBLIC_POCKETBASE_URL). The frontend talks to PocketBase directly from the BROWSER, so this must be externally reachable — expose the `pocketbase` service on its own host/route, e.g. https://wanderer-db.example.com |
| search | object | `{"config":{}}` | Extra environment variables for the meilisearch (search) container. See https://www.meilisearch.com/docs/learn/configuration/instance_options |
| secrets | object | `{"app":{"enabled":true,"stringData":{"MEILI_MASTER_KEY":"CHANGEME","POCKETBASE_ENCRYPTION_KEY":"CHANGEME-CHANGEME-CHANGEME-32chr"}}}` | Application secrets rendered into a Secret and injected via envFrom into all three containers. For production, disable this and supply an existing Secret via envFromSecrets (e.g. an ExternalSecret). |
| secrets.app.enabled | bool | `true` | Create the packaged application Secret |
| secrets.app.stringData.MEILI_MASTER_KEY | string | `"CHANGEME"` | Meilisearch master API key (REQUIRED, min 16 bytes). Shared by all three services. Generate with `openssl rand -base64 32`. |
| secrets.app.stringData.POCKETBASE_ENCRYPTION_KEY | string | `"CHANGEME-CHANGEME-CHANGEME-32chr"` | PocketBase AES encryption key (REQUIRED, EXACTLY 32 characters). Used to encrypt secrets at rest. Generate with `openssl rand -hex 16`. |
| web | object | `{"config":{}}` | Extra environment variables for the SvelteKit (web) container, e.g. the geocoding/routing backends or signup/privacy toggles. See https://wanderer.to/run/environment-configuration/#frontend |

## SMTP
Email (password resets, notifications) is sent by PocketBase. Configure it either
in the PocketBase admin UI or via `POCKETBASE_SMTP_*` variables under `db.config`
(the password should come from an external Secret). See
<https://wanderer.to/run/environment-configuration/#pocketbase>.

## OAuth / OIDC
wanderer supports OAuth2 providers, but they are configured **at runtime in the
PocketBase admin panel** (users collection → Options → OAuth2) — there are no
environment variables for the issuer/client id/secret, so this chart does not
expose them. The provider redirect URL is `<origin>/login/redirect`.

## Security
All three containers run fully hardened: `runAsNonRoot: true` (UID/GID 1000),
`readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, all capabilities
dropped, seccomp `RuntimeDefault`. Each gets a writable `/tmp` emptyDir.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
