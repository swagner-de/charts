# arr-stack

A Helm chart for the [*arr media automation stack](https://wiki.servarr.com/), deploying Sonarr, Radarr, Prowlarr, Bazarr, Flaresolverr, and Configarr as individual controllers.

## Features

- Each service runs as a separate controller and can be individually enabled/disabled
- Per-service persistence configuration
- Optional Configarr CronJob for automated quality profile management
- Shared or individual storage volumes per service

## Install

```bash
helm install arr-stack oci://ghcr.io/swagner-de/charts/arr-stack --version 0.2.8
```

## Components

| Component | Default Port | Description | Enabled by Default |
|-----------|-------------|-------------|-------------------|
| Sonarr | 8989 | TV series management | `true` |
| Radarr | 7878 | Movie management | `true` |
| Prowlarr | 9696 | Indexer management | `true` |
| Bazarr | 6767 | Subtitle management | `true` |
| Flaresolverr | 8191 | Cloudflare bypass proxy | `true` |
| Configarr | — | Quality profile sync (CronJob) | `false` |

## Configuration

Each component is configured under its own key in `values.yaml`:

```yaml
sonarr:
  enabled: true
  port: 8989
  image:
    repository: ghcr.io/home-operations/sonarr
    tag: '4.0.17.2953'
  persistence:
    config:
      type: persistentVolumeClaim
      mountPath: /config
      size: 100Mi
```

### Configarr

Configarr syncs quality profiles from TRaSH Guides. Enable it and provide configuration:

```yaml
configarr:
  enabled: true
  config: |
    trashGuideUrl: https://github.com/TRaSH-Guides/Guides
    recyclarrConfigUrl: https://github.com/recyclarr/config-templates
    sonarr:
      series:
        base_url: http://sonarr:8989
        api_key: !secret SONARR_API_KEY
  secrets:
    existingSecret: "my-configarr-secrets"
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

All containers run with restrictive security defaults:

- `runAsUser: 65534` (nobody), `runAsGroup: 65534`
- `readOnlyRootFilesystem: true` (except Flaresolverr)
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

Flaresolverr runs as UID 1000 and requires a writable root filesystem due to its Chromium dependency.
