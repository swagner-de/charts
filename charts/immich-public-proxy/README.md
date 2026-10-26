# immich-public-proxy

![Version: 0.5.0](https://img.shields.io/badge/Version-0.5.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 4.1.2](https://img.shields.io/badge/AppVersion-4.1.2-informational?style=flat-square)
Public sharing proxy for Immich
**Homepage:** <https://github.com/alangrainger/immich-public-proxy>

## Features
- Proxies public Immich shares without exposing the Immich server directly
- Configurable Immich and public-facing URLs
- Optional visitor-upload service that lets visitors send photos back into a share
- TCP liveness/startup probes and HTTP readiness checks
- Stateless deployments with temporary writable `/tmp` volumes

## Install

```bash
helm install immich-public-proxy oci://ghcr.io/swagner-de/charts/immich-public-proxy
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.3.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| controllers.main.containers.main.probes.readiness.enabled | bool | `true` | Enable the upstream-aware readiness probe |
| immichUrl | string | `"http://immich-server:2283"` | URL of the Immich server |
| publicBaseUrl | string | `"http://localhost:3000"` | Public URL where this proxy is available |
| upload | object | `{"enabled":false,"readiness":true}` | Upload service configuration |
| upload.enabled | bool | `false` | Deploy the visitor-upload container alongside the gallery. Lets visitors send photos back into a share without an API key. See https://docs.ipp.nz/visitor-uploads |
| upload.readiness | bool | `true` | Enable the upstream-aware readiness probe on the upload container. Disable this in environments without a reachable Immich server. |

## Upload Service

Set `upload.enabled: true` to deploy the visitor-upload container alongside the gallery.
See [IPP docs — Visitor Uploads](https://docs.ipp.nz/visitor-uploads) for full details.

The upload service:
- Streams files to Immich with the share's own key (no API key needed)
- Stores nothing and writes nothing to disk
- Exposes `/healthcheck` (returns `ok` or 503) so IPP auto-hides the upload button when Immich is unreachable

When enabled the chart creates a second Deployment and Service (`<release>-upload`) on port 3000.
Route `/upload/*` traffic to it through your ingress or Gateway API HTTPRoute.

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
