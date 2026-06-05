# arr-stack

![Version: 0.6.2](https://img.shields.io/badge/Version-0.6.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)
Media automation stack with Sonarr, Radarr, Prowlarr, Bazarr, Flaresolverr, and Configarr
**Homepage:** <https://wiki.servarr.com/>

## Features
- Each service runs as a separate controller and can be individually enabled/disabled
- Per-service persistence configuration
- Optional Configarr CronJob for automated quality profile management
- Shared or individual storage volumes per service

## Install

```bash
helm install arr-stack oci://ghcr.io/swagner-de/charts/arr-stack
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| bazarr | object | `{"enabled":true,"image":{"repository":"ghcr.io/home-operations/bazarr","tag":"1.5.6"},"persistence":{"config":{"accessMode":"ReadWriteOnce","mountPath":"/config","size":"100Mi","type":"persistentVolumeClaim"}},"port":6767}` | Bazarr subtitle management configuration |
| bazarr.enabled | bool | `true` | Enable Bazarr |
| bazarr.image | object | `{"repository":"ghcr.io/home-operations/bazarr","tag":"1.5.6"}` | Container image configuration |
| bazarr.image.repository | string | `"ghcr.io/home-operations/bazarr"` | Image repository |
| bazarr.image.tag | string | `"1.5.6"` | Image tag |
| bazarr.persistence | object | `{"config":{"accessMode":"ReadWriteOnce","mountPath":"/config","size":"100Mi","type":"persistentVolumeClaim"}}` | Persistence configuration |
| bazarr.persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| bazarr.persistence.config.mountPath | string | `"/config"` | Mount path inside the container |
| bazarr.persistence.config.size | string | `"100Mi"` | Volume size |
| bazarr.persistence.config.type | string | `"persistentVolumeClaim"` | Volume type |
| bazarr.port | int | `6767` | Service port |
| configarr | object | `{"config":"trashGuideUrl: https://github.com/TRaSH-Guides/Guides\nrecyclarrConfigUrl: https://github.com/recyclarr/config-templates\n\nsonarr:\n  series:\n    base_url: http://sonarr:8989\n    api_key: !secret SONARR_API_KEY\n\n    quality_definition:\n      type: series\n\n    include:\n      # WEB-1080p\n      - template: sonarr-quality-definition-series\n      - template: sonarr-v4-quality-profile-web-1080p\n      - template: sonarr-v4-custom-formats-web-1080p\n\n      # WEB-2160p\n      - template: sonarr-v4-quality-profile-web-2160p\n      - template: sonarr-v4-custom-formats-web-2160p\n\n    custom_formats: []\nradarr: {}\n","enabled":false,"persistence":{"cache":{"accessMode":"ReadWriteOnce","size":"100Mi","type":"emptyDir"}},"secrets":{"data":{},"existingSecret":""}}` | Configarr quality profile sync configuration |
| configarr.config | string | `"trashGuideUrl: https://github.com/TRaSH-Guides/Guides\nrecyclarrConfigUrl: https://github.com/recyclarr/config-templates\n\nsonarr:\n  series:\n    base_url: http://sonarr:8989\n    api_key: !secret SONARR_API_KEY\n\n    quality_definition:\n      type: series\n\n    include:\n      # WEB-1080p\n      - template: sonarr-quality-definition-series\n      - template: sonarr-v4-quality-profile-web-1080p\n      - template: sonarr-v4-custom-formats-web-1080p\n\n      # WEB-2160p\n      - template: sonarr-v4-quality-profile-web-2160p\n      - template: sonarr-v4-custom-formats-web-2160p\n\n    custom_formats: []\nradarr: {}\n"` | Configarr configuration (YAML format) |
| configarr.enabled | bool | `false` | Enable Configarr CronJob |
| configarr.persistence | object | `{"cache":{"accessMode":"ReadWriteOnce","size":"100Mi","type":"emptyDir"}}` | Persistence configuration |
| configarr.persistence.cache.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| configarr.persistence.cache.size | string | `"100Mi"` | Volume size |
| configarr.persistence.cache.type | string | `"emptyDir"` | Volume type |
| configarr.secrets | object | `{"data":{},"existingSecret":""}` | Configarr secrets configuration |
| configarr.secrets.data | object | `{}` | Secret data key-value pairs |
| configarr.secrets.existingSecret | string | `""` | Use an existing secret for Configarr |
| flaresolverr | object | `{"enabled":true,"image":{"repository":"ghcr.io/flaresolverr/flaresolverr","tag":"v3.5.0"},"port":8191}` | Flaresolverr Cloudflare bypass proxy configuration |
| flaresolverr.enabled | bool | `true` | Enable Flaresolverr |
| flaresolverr.image | object | `{"repository":"ghcr.io/flaresolverr/flaresolverr","tag":"v3.5.0"}` | Container image configuration |
| flaresolverr.image.repository | string | `"ghcr.io/flaresolverr/flaresolverr"` | Image repository |
| flaresolverr.image.tag | string | `"v3.5.0"` | Image tag |
| flaresolverr.port | int | `8191` | Service port |
| prowlarr | object | `{"enabled":true,"image":{"repository":"ghcr.io/home-operations/prowlarr","tag":"2.4.0.5391"},"persistence":{"config":{"accessMode":"ReadWriteOnce","mountPath":"/config","size":"100Mi","type":"persistentVolumeClaim"}},"port":9696}` | Prowlarr indexer management configuration |
| prowlarr.enabled | bool | `true` | Enable Prowlarr |
| prowlarr.image | object | `{"repository":"ghcr.io/home-operations/prowlarr","tag":"2.4.0.5391"}` | Container image configuration |
| prowlarr.image.repository | string | `"ghcr.io/home-operations/prowlarr"` | Image repository |
| prowlarr.image.tag | string | `"2.4.0.5391"` | Image tag |
| prowlarr.persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| prowlarr.persistence.config.mountPath | string | `"/config"` | Mount path inside the container |
| prowlarr.persistence.config.size | string | `"100Mi"` | Volume size |
| prowlarr.persistence.config.type | string | `"persistentVolumeClaim"` | Volume type |
| prowlarr.port | int | `9696` | Service port |
| radarr | object | `{"enabled":true,"image":{"repository":"ghcr.io/home-operations/radarr","tag":"6.2.1"},"persistence":{"config":{"accessMode":"ReadWriteOnce","mountPath":"/config","size":"100Mi","type":"persistentVolumeClaim"}},"port":7878}` | Radarr movie management configuration |
| radarr.enabled | bool | `true` | Enable Radarr |
| radarr.image | object | `{"repository":"ghcr.io/home-operations/radarr","tag":"6.2.1"}` | Container image configuration |
| radarr.image.repository | string | `"ghcr.io/home-operations/radarr"` | Image repository |
| radarr.image.tag | string | `"6.2.1"` | Image tag |
| radarr.persistence | object | `{"config":{"accessMode":"ReadWriteOnce","mountPath":"/config","size":"100Mi","type":"persistentVolumeClaim"}}` | Persistence configuration |
| radarr.persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| radarr.persistence.config.mountPath | string | `"/config"` | Mount path inside the container |
| radarr.persistence.config.size | string | `"100Mi"` | Volume size |
| radarr.persistence.config.type | string | `"persistentVolumeClaim"` | Volume type |
| radarr.port | int | `7878` | Service port |
| sonarr | object | `{"enabled":true,"image":{"repository":"ghcr.io/home-operations/sonarr","tag":"4.0.17.2969"},"persistence":{"config":{"accessMode":"ReadWriteOnce","mountPath":"/config","size":"100Mi","type":"persistentVolumeClaim"}},"port":8989}` | Sonarr TV series management configuration |
| sonarr.enabled | bool | `true` | Enable Sonarr |
| sonarr.image | object | `{"repository":"ghcr.io/home-operations/sonarr","tag":"4.0.17.2969"}` | Container image configuration |
| sonarr.image.repository | string | `"ghcr.io/home-operations/sonarr"` | Image repository |
| sonarr.image.tag | string | `"4.0.17.2969"` | Image tag |
| sonarr.persistence | object | `{"config":{"accessMode":"ReadWriteOnce","mountPath":"/config","size":"100Mi","type":"persistentVolumeClaim"}}` | Persistence configuration |
| sonarr.persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| sonarr.persistence.config.mountPath | string | `"/config"` | Mount path inside the container |
| sonarr.persistence.config.size | string | `"100Mi"` | Volume size |
| sonarr.persistence.config.type | string | `"persistentVolumeClaim"` | Volume type |
| sonarr.port | int | `8989` | Service port |

## Security
All containers run with restrictive security defaults:

- `runAsUser: 65534` (nobody), `runAsGroup: 65534`
- `readOnlyRootFilesystem: true` (except Flaresolverr)
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

Flaresolverr runs as UID 1000 and requires a writable root filesystem due to its Chromium dependency.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
