# wyoming

![Version: 0.2.0](https://img.shields.io/badge/Version-0.2.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: whisper-3.8.1_piper-2.5.2](https://img.shields.io/badge/AppVersion-whisper--3.8.1_piper--2.5.2-informational?style=flat-square)
Wyoming speech services for Home Assistant Assist
**Homepage:** <https://www.home-assistant.io/voice_control/>

## Features
- Independently enabled Wyoming Whisper speech-to-text service
- Independently enabled Wyoming Piper text-to-speech service
- Separate ClusterIP services and persistent model volumes
- No external ingress by default

## Install

```bash
helm install wyoming oci://ghcr.io/swagner-de/charts/wyoming
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.3.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| piper.enabled | bool | `true` | Enable the Piper text-to-speech service |
| piper.extraArgs | list | `[]` | Additional arguments appended to the generated Piper arguments |
| piper.image.repository | string | `"docker.io/rhasspy/wyoming-piper"` | Piper image repository |
| piper.image.tag | string | `"2.5.2"` | Piper image tag renovate: datasource=docker depName=rhasspy/wyoming-piper |
| piper.persistence.accessMode | string | `"ReadWriteOnce"` | PVC access mode |
| piper.persistence.enabled | bool | `true` | Persist downloaded Piper voices |
| piper.persistence.mountPath | string | `"/data"` | Voice data mount path |
| piper.persistence.size | string | `"2Gi"` | PVC size |
| piper.persistence.storageClass | string | `""` | StorageClass; empty uses the cluster default |
| piper.port | int | `10200` | Wyoming TCP port |
| piper.resources.limits.memory | string | `"1Gi"` |  |
| piper.resources.requests.cpu | string | `"100m"` |  |
| piper.resources.requests.memory | string | `"256Mi"` |  |
| piper.voice | string | `"en_US-lessac-medium"` | Piper voice name |
| whisper.enabled | bool | `true` | Enable the Whisper speech-to-text service |
| whisper.extraArgs | list | `[]` | Additional arguments appended to the generated Whisper arguments |
| whisper.image.repository | string | `"docker.io/rhasspy/wyoming-whisper"` | Whisper image repository |
| whisper.image.tag | string | `"3.8.1"` | Whisper image tag renovate: datasource=docker depName=rhasspy/wyoming-whisper |
| whisper.language | string | `"en"` | Language code. Set to an empty string for automatic detection. |
| whisper.model | string | `"small-int8"` | Whisper model name |
| whisper.persistence.accessMode | string | `"ReadWriteOnce"` | PVC access mode |
| whisper.persistence.enabled | bool | `true` | Persist downloaded Whisper models |
| whisper.persistence.mountPath | string | `"/data"` | Model data mount path |
| whisper.persistence.size | string | `"10Gi"` | PVC size |
| whisper.persistence.storageClass | string | `""` | StorageClass; empty uses the cluster default |
| whisper.port | int | `10300` | Wyoming TCP port |
| whisper.resources.limits.memory | string | `"4Gi"` |  |
| whisper.resources.requests.cpu | string | `"500m"` |  |
| whisper.resources.requests.memory | string | `"1Gi"` |  |

## Security
- Runs as non-root UID/GID 65534
- Read-only root filesystems with writable model and temporary volumes
- Service account token automount disabled
- Privilege escalation disabled and all capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
