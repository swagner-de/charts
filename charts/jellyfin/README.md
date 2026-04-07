# jellyfin

A Helm chart for [Jellyfin](https://jellyfin.org/), a free software media server.

## Features

- Persistent volumes for config, transcoding, and media library
- Hardware transcoding support via `/dev/dri` host path mount
- Read-only root filesystem with emptyDir for cache and tmp

## Install

```bash
helm install jellyfin oci://ghcr.io/swagner-de/charts/jellyfin --version 0.1.9
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `persistence.config.size` | Config volume size | `15Gi` |
| `persistence.transcode.size` | Transcoding volume size | `70Gi` |
| `persistence.media.size` | Media library volume size | `100Gi` |

### Hardware Transcoding

To enable GPU-based transcoding, mount `/dev/dri` and add the render group:

```yaml
persistence:
  dev-dri:
    enabled: true
    type: hostPath
    hostPath: /dev/dri
    advancedMounts:
      main:
        main:
        - path: /dev/dri

defaultPodOptions:
  securityContext:
    supplementalGroups: [992]  # render group GID
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID/GID 65534 (nobody)
- `readOnlyRootFilesystem: true`
- `privileged: true` (required for hardware transcoding access; can be disabled if not using GPU)
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

**Note:** The container currently runs as privileged to access GPU hardware for transcoding. If you do not need hardware transcoding, consider overriding this in your values.
