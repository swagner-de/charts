# immich-public-proxy

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 3.4.0](https://img.shields.io/badge/AppVersion-3.4.0-informational?style=flat-square)
Public sharing proxy for Immich
**Homepage:** <https://github.com/alangrainger/immich-public-proxy>

## Features
- Proxies public Immich shares without exposing the Immich server directly
- Configurable Immich and public-facing URLs
- TCP liveness/startup probes and HTTP readiness check on `/share/healthcheck`
- Stateless deployment with a temporary writable `/tmp` volume

## Install

```bash
helm install immich-public-proxy oci://ghcr.io/swagner-de/charts/immich-public-proxy
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| controllers.main.containers.main.probes.readiness.enabled | bool | `true` | Enable the upstream-aware readiness probe |
| immichUrl | string | `"http://immich-server:2283"` | URL of the Immich server |
| publicBaseUrl | string | `"http://localhost:3000"` | Public URL where this proxy is available |

## Security
This chart runs with restrictive security defaults:

- `runAsNonRoot: true`, UID/GID 1000
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
