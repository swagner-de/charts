# music-assistant

![Version: 0.1.1](https://img.shields.io/badge/Version-0.1.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2.8.9](https://img.shields.io/badge/AppVersion-2.8.9-informational?style=flat-square)
Music Assistant - free, opensource Media player for your local music and online music providers
**Homepage:** <https://www.music-assistant.io/>

## Features
- Web UI on port 8095
- Audio streaming on port 8097
- Layer 2 network attachment support for mDNS player discovery

## Install

```bash
helm install music-assistant oci://ghcr.io/swagner-de/charts/music-assistant
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| logLevel | string | `"info"` |  |
| persistence.data.accessMode | string | `"ReadWriteOnce"` |  |
| persistence.data.enabled | bool | `true` |  |
| persistence.data.size | string | `"10Gi"` |  |

## Security
- `runAsNonRoot: true`, UID/GID 65534
- `readOnlyRootFilesystem: false`
- `allowPrivilegeEscalation: false`
- All capabilities dropped except `NET_RAW` (required for mDNS)
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
