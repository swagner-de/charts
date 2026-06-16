# qbittorrent

![Version: 0.4.2](https://img.shields.io/badge/Version-0.4.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 5.2.2](https://img.shields.io/badge/AppVersion-5.2.2-informational?style=flat-square)
BitTorrent client with web UI and optional VPN (gluetun) sidecar
**Homepage:** <https://www.qbittorrent.org/>

## Features
- VueTorrent web UI automatically downloaded via init container
- Optional [gluetun](https://github.com/qdm12/gluetun) VPN sidecar (disabled by default)
- Separate persistent volumes for config, VueTorrent UI, and downloads

## Install

```bash
helm install qbittorrent oci://ghcr.io/swagner-de/charts/qbittorrent
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| controllers | object | `{"main":{"containers":{"gluetun":{"enabled":false}}}}` | Controller configuration |
| controllers.main.containers.gluetun | object | `{"enabled":false}` | Gluetun VPN sidecar |
| controllers.main.containers.gluetun.enabled | bool | `false` | Enable gluetun VPN sidecar |
| persistence | object | `{"config":{"accessMode":"ReadWriteOnce","enabled":true,"size":"100Mi"},"downloads":{"accessMode":"ReadWriteOnce","enabled":true,"size":"300Gi"},"vuetorrent":{"accessMode":"ReadWriteOnce","enabled":true,"size":"200Mi"}}` | Persistent storage configuration |
| persistence.config | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"100Mi"}` | Configuration volume |
| persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.config.enabled | bool | `true` | Enable config persistence |
| persistence.config.size | string | `"100Mi"` | Config volume size |
| persistence.downloads | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"300Gi"}` | Downloads volume |
| persistence.downloads.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.downloads.enabled | bool | `true` | Enable downloads persistence |
| persistence.downloads.size | string | `"300Gi"` | Downloads volume size |
| persistence.vuetorrent | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"200Mi"}` | VueTorrent web UI volume |
| persistence.vuetorrent.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.vuetorrent.enabled | bool | `true` | Enable VueTorrent persistence |
| persistence.vuetorrent.size | string | `"200Mi"` | VueTorrent volume size |

## Security
- `runAsUser: 65534` (nobody), `runAsGroup: 65534`
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

**Note:** The gluetun sidecar runs as root (UID 0) with `NET_ADMIN` capability, which is required for VPN tunnel setup. It is disabled by default.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
