# qbittorrent

A Helm chart for [qBittorrent](https://www.qbittorrent.org/), a free BitTorrent client with a web UI.

## Features

- VueTorrent web UI automatically downloaded via init container
- Optional [gluetun](https://github.com/qdm12/gluetun) VPN sidecar (disabled by default)
- Separate persistent volumes for config, VueTorrent UI, and downloads

## Install

```bash
helm install qbittorrent oci://ghcr.io/swagner-de/charts/qbittorrent --version 0.2.0
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `persistence.config.size` | Config volume size | `100Mi` |
| `persistence.vuetorrent.size` | VueTorrent UI volume size | `200Mi` |
| `persistence.downloads.size` | Downloads volume size | `300Gi` |
| `controllers.main.containers.gluetun.enabled` | Enable gluetun VPN sidecar | `false` |

### VPN Sidecar

Enable the gluetun VPN sidecar to route all traffic through a VPN:

```yaml
controllers:
  main:
    containers:
      gluetun:
        enabled: true
        env:
          VPN_SERVICE_PROVIDER: "your-provider"
          VPN_TYPE: "wireguard"
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsUser: 65534` (nobody), `runAsGroup: 65534`
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

**Note:** The gluetun sidecar runs as root (UID 0) with `NET_ADMIN` capability, which is required for VPN tunnel setup. It is disabled by default.
