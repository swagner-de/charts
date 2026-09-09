# emby

![Version: 0.2.0](https://img.shields.io/badge/Version-0.2.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 4.10.0.40](https://img.shields.io/badge/AppVersion-4.10.0.40-informational?style=flat-square)
Emby media server for streaming movies, TV, music, and more
**Homepage:** <https://emby.media/>

## Features
- Persistent volumes for config, transcoding, and media library
- Shareable media library via `persistence.media.existingClaim`
- Hardware transcoding support via `/dev/dri` host path mount
- Read-only root filesystem with emptyDir for tmp
- Rootless upstream image ([home-operations/emby](https://github.com/home-operations/containers/tree/main/apps/emby)): no s6-overlay, runs as UID 65534

## Install

```bash
helm install emby oci://ghcr.io/swagner-de/charts/emby
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| persistence | object | `{"config":{"accessMode":"ReadWriteOnce","enabled":true,"size":"15Gi"},"media":{"accessMode":"ReadWriteOnce","enabled":true,"size":"100Gi"},"transcode":{"accessMode":"ReadWriteOnce","enabled":true,"size":"70Gi"}}` | Persistent storage configuration |
| persistence.config | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"15Gi"}` | Configuration volume (Emby programdata: settings, metadata, db) |
| persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.config.enabled | bool | `true` | Enable config persistence |
| persistence.config.size | string | `"15Gi"` | Config volume size |
| persistence.media | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"100Gi"}` | Media library volume. Set persistence.media.existingClaim to share an existing library PVC (e.g. a Jellyfin media claim), ideally mounted read-only via advancedMounts. |
| persistence.media.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.media.enabled | bool | `true` | Enable media persistence |
| persistence.media.size | string | `"100Gi"` | Media volume size (ignored when existingClaim is set) |
| persistence.transcode | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"70Gi"}` | Transcoding temporary volume. Point Emby at /transcode in Dashboard > Transcoding > "Transcoding temporary path". |
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
