# jellyfin

![Version: 0.1.12](https://img.shields.io/badge/Version-0.1.12-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 10.11.8](https://img.shields.io/badge/AppVersion-10.11.8-informational?style=flat-square)
Free software media server for streaming movies, TV, music, and more
**Homepage:** <https://jellyfin.org/>

## Features
- Persistent volumes for config, transcoding, and media library
- Hardware transcoding support via `/dev/dri` host path mount
- Read-only root filesystem with emptyDir for cache and tmp

## Install

```bash
helm install jellyfin oci://ghcr.io/swagner-de/charts/jellyfin
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| persistence | object | `{"config":{"accessMode":"ReadWriteOnce","enabled":true,"size":"15Gi"},"media":{"accessMode":"ReadWriteOnce","enabled":true,"size":"100Gi"},"transcode":{"accessMode":"ReadWriteOnce","enabled":true,"size":"70Gi"}}` | Persistent storage configuration |
| persistence.config | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"15Gi"}` | Configuration volume |
| persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.config.enabled | bool | `true` | Enable config persistence |
| persistence.config.size | string | `"15Gi"` | Config volume size |
| persistence.media | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"100Gi"}` | Media library volume |
| persistence.media.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.media.enabled | bool | `true` | Enable media persistence |
| persistence.media.size | string | `"100Gi"` | Media volume size |
| persistence.transcode | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"70Gi"}` | Transcoding volume |
| persistence.transcode.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.transcode.enabled | bool | `true` | Enable transcode persistence |
| persistence.transcode.size | string | `"70Gi"` | Transcode volume size |

## Security
- `runAsNonRoot: true`, UID/GID 65534 (nobody)
- `readOnlyRootFilesystem: true`
- `privileged: true` (required for hardware transcoding access; can be disabled if not using GPU)
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

**Note:** The container currently runs as privileged to access GPU hardware for transcoding. If you do not need hardware transcoding, consider overriding this in your values.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
