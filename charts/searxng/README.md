# searxng

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2026.8.22-9fea41204](https://img.shields.io/badge/AppVersion-2026.8.22--9fea41204-informational?style=flat-square)
Privacy-respecting metasearch engine
**Homepage:** <https://searxng.org/>

## Features
- Privacy-respecting metasearch engine
- `settings.yml` rendered from a ConfigMap (JSON output + disabled limiter for API consumers)
- `secret_key` supplied at runtime via the `SEARXNG_SECRET` env var (packaged Secret or an ExternalSecret)
- Stateless by default (emptyDir cache); optional PVC

## Install

```bash
helm install searxng oci://ghcr.io/swagner-de/charts/searxng
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| envFromSecrets | list | `[]` | Existing secrets to inject as environment variables (e.g. an ExternalSecret carrying SEARXNG_SECRET). |
| persistence | object | `{"data":{"accessMode":"ReadWriteOnce","enabled":false,"size":"1Gi"}}` | Persistent storage. SearXNG only needs a writable data/cache dir; an emptyDir is sufficient for a stateless instance, so persistence is disabled by default. |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.enabled | bool | `false` | Enable a PersistentVolumeClaim for /var/cache/searxng (otherwise emptyDir) |
| persistence.data.size | string | `"1Gi"` | Data volume size |
| route | object | `{"main":{"enabled":false}}` | HTTPRoute (Gateway API) configuration. Disabled by default — SearXNG is typically consumed in-cluster over its Service. |
| secrets | object | `{"app":{"enabled":true,"stringData":{"SEARXNG_SECRET":"CHANGEME"}}}` | Packaged Secret injected via envFrom. Use it to set SEARXNG_SECRET (overrides server.secret_key). In production disable this and supply an ExternalSecret via envFromSecrets instead. |
| secrets.app.enabled | bool | `true` | Create the packaged application Secret |
| secrets.app.stringData.SEARXNG_SECRET | string | `"CHANGEME"` | SearXNG secret_key. Generate with `openssl rand -hex 32`. |
| settings | string | `"use_default_settings: true\nserver:\n  secret_key: \"changeme\"\n  limiter: false\n  image_proxy: true\nsearch:\n  safe_search: 0\n  autocomplete: \"\"\n  formats:\n    - html\n    - json\n"` | SearXNG settings.yml, rendered into a ConfigMap and mounted at /etc/searxng. The secret_key here is a placeholder; override it at runtime with the SEARXNG_SECRET env var (see secrets / envFromSecrets). `json` output and a disabled limiter are required for API consumers such as Open WebUI. |

## Security
- `runAsNonRoot: true`, UID/GID 977
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
