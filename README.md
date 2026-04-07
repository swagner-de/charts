# Helm Charts

A collection of Helm charts for self-hosted applications, built on the [bjw-s common library chart](https://bjw-s-labs.github.io/helm-charts/).

## Charts

| Chart | Description |
|-------|-------------|
| [adguard-home](charts/adguard-home) | DNS-level ad and tracker blocking with optional Prometheus exporter |
| [arr-stack](charts/arr-stack) | Media automation stack (Sonarr, Radarr, Prowlarr, Bazarr, Flaresolverr, Configarr) |
| [homeassistant](charts/homeassistant) | Home automation platform with optional LDAP and Matter bridge support |
| [homepage](charts/homepage) | Customizable application dashboard ([gethomepage.dev](https://gethomepage.dev)) |
| [immich](charts/immich) | Self-hosted photo and video backup solution |
| [invidious](charts/invidious) | Privacy-focused alternative YouTube frontend |
| [jellyfin](charts/jellyfin) | Free software media server |
| [mealie](charts/mealie) | Self-hosted recipe manager and meal planner |
| [paperless](charts/paperless) | Document management system with OCR |
| [powerdns-auth](charts/powerdns-auth) | PowerDNS authoritative DNS server |
| [powerdns-recursor](charts/powerdns-recursor) | PowerDNS recursive DNS resolver |
| [qbittorrent](charts/qbittorrent) | BitTorrent client with optional VPN (gluetun) sidecar |
| [skel](charts/skel) | Skeleton chart template for creating new charts |
| [zigbee2mqtt](charts/zigbee2mqtt) | Zigbee-to-MQTT bridge for smart home devices |

## Usage

Charts are published as OCI artifacts. To install a chart:

```bash
helm install <release-name> oci://ghcr.io/swagner-de/charts/<chart-name> --version <version>
```

For example:

```bash
helm install mealie oci://ghcr.io/swagner-de/charts/mealie --version 0.15.0
```

To customize the installation, create a `values.yaml` file and pass it during install:

```bash
helm install mealie oci://ghcr.io/swagner-de/charts/mealie --version 0.15.0 -f values.yaml
```

## Architecture

All charts use the [bjw-s common library chart](https://bjw-s-labs.github.io/helm-charts/) (v4.x) as their base dependency. Each chart has a single `templates/hardcoded.yaml` that defines sensible defaults which are merged with user-provided values at render time.

Charts that require databases support both packaged [PostgreSQL](https://github.com/cloudpirates/helm-charts)/[Redis](https://github.com/cloudpirates/helm-charts) subcharts and [CloudNativePG](https://cloudnative-pg.io/) (CNPG) operator-managed clusters.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding new charts, testing, and submitting changes.

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.
