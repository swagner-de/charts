# Helm Charts

A collection of Helm charts for self-hosted applications, built on the [bjw-s common library chart](https://bjw-s-labs.github.io/helm-charts/).

## Charts

<!-- charts-table-start -->
| Chart | Version | Description |
|-------|---------|-------------|
| [adguard-home](charts/adguard-home) | `0.8.3` | DNS-level ad and tracker blocking with optional Prometheus exporter |
| [adventurelog](charts/adventurelog) | `0.1.0` | Self-hosted travel companion to log trips, plan itineraries, and map your adventures |
| [akvorado](charts/akvorado) | `0.5.0` | NetFlow/IPFIX/sFlow collector with ClickHouse analytics |
| [arr-stack](charts/arr-stack) | `0.10.3` | Media automation stack with Sonarr, Radarr, Prowlarr, Bazarr, Flaresolverr, Configarr, and UmlautAdaptarr |
| [carconnectivity](charts/carconnectivity) | `0.3.0` | Car telemetry data retrieval with plugin support (VW, ABRP, WebUI) |
| [dawarich](charts/dawarich) | `0.1.7` | Self-hosted location history tracker and Google Timeline alternative |
| [flowcus](charts/flowcus) | `0.3.0` | Lightweight NetFlow/IPFIX collector with embedded web UI and columnar storage |
| [freeradius](charts/freeradius) | `0.2.0` | FreeRADIUS server with LDAP backend for WPA Enterprise and MAC authentication |
| [ghostfolio](charts/ghostfolio) | `0.33.2` | Open source wealth management software |
| [home-assistant-matter-hub](charts/home-assistant-matter-hub) | `0.2.1` | Matter bridge for Home Assistant with mDNS support via Layer 2 network attachment |
| [homeassistant](charts/homeassistant) | `1.23.2` | Home automation platform with optional LDAP, Matter bridge, and CNPG database support |
| [homepage](charts/homepage) | `0.8.0` | Customizable application dashboard for your homelab |
| [immich](charts/immich) | `0.18.6` | Self-hosted photo and video backup solution with machine learning |
| [invidious](charts/invidious) | `0.11.2` | Privacy-focused alternative YouTube frontend with companion service and PostgreSQL |
| [jellyfin](charts/jellyfin) | `0.3.0` | Free software media server for streaming movies, TV, music, and more |
| [linkwarden](charts/linkwarden) | `0.3.2` | Self-hosted collaborative bookmark manager to collect, organize and archive webpages |
| [mealie](charts/mealie) | `0.28.2` | Self-hosted recipe manager and meal planner |
| [mosquitto](charts/mosquitto) | `0.2.0` | Eclipse Mosquitto MQTT broker with TLS and password authentication |
| [music-assistant](charts/music-assistant) | `0.3.0` | Music Assistant - free, opensource Media player for your local music and online music providers |
| [netra](charts/netra) | `0.3.0` | ASN traffic analysis dashboard for NetFlow/IPFIX flow data |
| [paperless](charts/paperless) | `0.58.6` | Document management system with OCR and full-text search |
| [powerdns-auth](charts/powerdns-auth) | `0.7.0` | PowerDNS authoritative DNS server with LMDB backend and external-dns support |
| [powerdns-recursor](charts/powerdns-recursor) | `0.3.0` | PowerDNS recursive DNS resolver with DNSSEC validation |
| [qbittorrent](charts/qbittorrent) | `0.5.0` | BitTorrent client with web UI and optional VPN (gluetun) sidecar |
| [samba](charts/samba) | `0.3.0` | Multi-user Samba SMB file server with per-share encryption and Time Machine support |
| [searxng](charts/searxng) | `0.1.0` | Privacy-respecting metasearch engine |
| [skel](charts/skel) | `0.3.0` | Skeleton chart template for creating new charts |
| [unifi-os-server](charts/unifi-os-server) | `0.2.0` | UniFi OS Server - self-hosted UniFi Network with Organizations, IdP, and Site Magic SD-WAN support |
| [wanderer](charts/wanderer) | `0.5.0` | Self-hosted trail database and route planner for hiking, biking and other outdoor activities |
| [zigbee2mqtt](charts/zigbee2mqtt) | `0.13.0` | Zigbee-to-MQTT bridge for smart home devices |
<!-- charts-table-end -->

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

## Security Scanning

Container images used by all charts are scanned weekly for CVEs. Findings are uploaded to the [GitHub Security tab](../../security/code-scanning) with a combined risk score that weighs real-world exploitability (EPSS + CISA KEV) against raw CVSS impact.

See [docs/cve-risk-scoring.md](docs/cve-risk-scoring.md) for details on the scoring methodology.

## Architecture

All charts use the [bjw-s common library chart](https://bjw-s-labs.github.io/helm-charts/) (v4.x) as their base dependency. Each chart has a single `templates/hardcoded.yaml` that defines sensible defaults which are merged with user-provided values at render time.

Charts that require databases support both packaged [PostgreSQL](https://github.com/cloudpirates/helm-charts)/[Redis](https://github.com/cloudpirates/helm-charts) subcharts and [CloudNativePG](https://cloudnative-pg.io/) (CNPG) operator-managed clusters.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding new charts, testing, and submitting changes.

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.
