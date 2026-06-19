# home-assistant-matter-hub

![Version: 0.1.2](https://img.shields.io/badge/Version-0.1.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2.0.48](https://img.shields.io/badge/AppVersion-2.0.48-informational?style=flat-square)
Matter bridge for Home Assistant with mDNS support via Layer 2 network attachment
**Homepage:** <https://riddix.github.io/home-assistant-matter-hub/>

## Features
- Matter bridge exposing Home Assistant devices to Matter controllers
- Layer 2 network attachment support for mDNS
- Built-in network policy restricting egress to Home Assistant only (no internet)
- Web UI on port 8482

## Install

```bash
helm install home-assistant-matter-hub oci://ghcr.io/swagner-de/charts/home-assistant-matter-hub
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| homeAssistantReleaseName | string | `"homeassistant"` | Release name of the Home Assistant instance (used for network policy egress target) |
| logLevel | string | `"info"` | Log level (info, warn, error, debug) |
| persistence | object | `{"data":{"accessMode":"ReadWriteOnce","enabled":true,"size":"1Gi"}}` | Persistent storage configuration |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.enabled | bool | `true` | Enable data persistence |
| persistence.data.size | string | `"1Gi"` | Volume size |

## Security
- `runAsNonRoot: true`, UID/GID 65534
- `readOnlyRootFilesystem: false`
- `allowPrivilegeEscalation: false`
- All capabilities dropped except `NET_RAW` (required for mDNS)
- Seccomp profile: `RuntimeDefault`
- Network policy: egress only to Home Assistant pods, no internet access

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
