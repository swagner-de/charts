# akvorado

![Version: 0.1.3](https://img.shields.io/badge/Version-0.1.3-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2.3.0](https://img.shields.io/badge/AppVersion-2.3.0-informational?style=flat-square)
NetFlow/IPFIX/sFlow collector with ClickHouse analytics
**Homepage:** <https://github.com/akvorado/akvorado>

## Architecture

The chart deploys the full Akvorado stack:

```
┌─────────────┐     ┌─────────┐     ┌─────────┐     ┌────────────┐
│  Exporters  │────▶│  Inlet  │────▶│  Kafka  │────▶│   Outlet   │
│ (NetFlow/   │     │         │     │         │     │ (enriches) │
│  sFlow/BMP) │     └─────────┘     └─────────┘     └─────┬──────┘
└─────────────┘                                            │
                                                           ▼
┌─────────────┐                                     ┌────────────┐
│   Console   │◀────────────────────────────────────│ ClickHouse │
│  (Web UI)   │                                     │            │
└─────────────┘                                     └────────────┘
        ▲
        │
┌───────┴─────┐
│Orchestrator │  (coordinates all components, serves config)
└─────────────┘
```

- **Inlet** — receives flows (NetFlow v5/v9, IPFIX, sFlow) via UDP
- **Outlet** — enriches flows with SNMP metadata, GeoIP, routing info, then writes to ClickHouse
- **Kafka** — message bus between inlet and outlet
- **ClickHouse** — columnar store for flow data with automatic aggregation
- **Console** — web UI for querying and visualizing flows
- **Orchestrator** — coordinates configuration across all components

## Install

```bash
helm install akvorado oci://ghcr.io/swagner-de/charts/akvorado
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| akvorado | object | `{"asns":{},"console":{"http":{"cache":{"type":"memory"}}},"inlet":{"flow":{"inputs":[{"decoder":"netflow","listen":":2055","receive-buffer":212992,"type":"udp","workers":4},{"decoder":"netflow","listen":":4739","receive-buffer":212992,"type":"udp","workers":4},{"decoder":"sflow","listen":":6343","receive-buffer":212992,"type":"udp","workers":4}]}},"networkSources":{},"networks":{},"outlet":{"core":{"default-sampling-rate":0,"exporter-classifiers":[],"interface-classifiers":[]},"metadata":{"providers":[{"type":"snmp"}]},"routing":{"provider":{"receive-buffer":212992,"type":"bmp"}}},"resolutions":[{"interval":0,"ttl":"360h"},{"interval":"1m","ttl":"168h"},{"interval":"5m","ttl":"2160h"},{"interval":"1h","ttl":"8640h"}]}` | Akvorado configuration file content (akvorado.yaml) This is passed directly to the orchestrator. Service references (kafka brokers, clickhouse servers) are auto-injected by the chart. Add your network classifiers, ASN overrides, SNMP credentials here. |
| akvorado.asns | object | `{}` | Static ASN name overrides (displayed in console UI) |
| akvorado.console | object | `{"http":{"cache":{"type":"memory"}}}` | Console configuration |
| akvorado.inlet | object | `{"flow":{"inputs":[{"decoder":"netflow","listen":":2055","receive-buffer":212992,"type":"udp","workers":4},{"decoder":"netflow","listen":":4739","receive-buffer":212992,"type":"udp","workers":4},{"decoder":"sflow","listen":":6343","receive-buffer":212992,"type":"udp","workers":4}]}}` | Inlet flow input configuration |
| akvorado.networkSources | object | `{}` | Remote network sources (Netbox, AWS, GCP, etc.) |
| akvorado.networks | object | `{}` | Network prefix classification |
| akvorado.outlet | object | `{"core":{"default-sampling-rate":0,"exporter-classifiers":[],"interface-classifiers":[]},"metadata":{"providers":[{"type":"snmp"}]},"routing":{"provider":{"receive-buffer":212992,"type":"bmp"}}}` | Outlet metadata/routing configuration |
| akvorado.outlet.core.default-sampling-rate | int | `0` | Default sampling rate when exporter doesn't advertise one (0 = reject flows without rate) |
| akvorado.resolutions | list | `[{"interval":0,"ttl":"360h"},{"interval":"1m","ttl":"168h"},{"interval":"5m","ttl":"2160h"},{"interval":"1h","ttl":"8640h"}]` | ClickHouse data retention |
| clickhouse | object | `{"auth":{"existingSecret":"","existingSecretKey":"password","password":"","username":"akvorado"},"maxMemoryUsage":0,"persistence":{"size":"50Gi"}}` | ClickHouse configuration |
| clickhouse.auth.existingSecret | string | `""` | Existing secret with ClickHouse password (key defined by existingSecretKey) |
| clickhouse.auth.existingSecretKey | string | `"password"` | Key in the existing secret |
| clickhouse.auth.password | string | `""` | Password (used if existingSecret is empty; chart will create a Secret) |
| clickhouse.auth.username | string | `"akvorado"` | ClickHouse username for akvorado |
| clickhouse.maxMemoryUsage | int | `0` | Maximum server memory usage (bytes, 0 = auto) |
| clickhouse.persistence.size | string | `"50Gi"` | Storage size for ClickHouse data |
| console | object | `{"replicas":1}` | Console configuration |
| console.replicas | int | `1` | Number of console replicas |
| existingSecret | string | `""` | Existing secret containing credentials (keys: snmpCommunity, ipInfoToken) When set, the chart reads SNMP community and GeoIP token from this secret. |
| geoip | object | `{"enabled":false,"existingSecret":"","maxmindEditionIds":"GeoLite2-ASN GeoLite2-Country","provider":"maxmind"}` | GeoIP database configuration |
| geoip.enabled | bool | `false` | Enable GeoIP enrichment (requires API credentials) |
| geoip.existingSecret | string | `""` | Existing secret containing API credentials For maxmind: keys GEOIPUPDATE_ACCOUNT_ID and GEOIPUPDATE_LICENSE_KEY For ipinfo: key IPINFO_TOKEN |
| geoip.maxmindEditionIds | string | `"GeoLite2-ASN GeoLite2-Country"` | MaxMind edition IDs to download |
| geoip.provider | string | `"maxmind"` | Provider: maxmind or ipinfo |
| grafana | object | `{"enabled":false}` | Grafana integration (requires Grafana sidecar for auto-discovery) |
| grafana.enabled | bool | `false` | Enable Grafana datasource and dashboard provisioning via sidecar |
| inlet | object | `{"replicas":1}` | Inlet configuration |
| inlet.replicas | int | `1` | Number of inlet replicas |
| kafka | object | `{"heapSize":"256m","persistence":{"size":"10Gi"},"retentionMs":86400000,"topicPartitions":8}` | Kafka configuration |
| kafka.heapSize | string | `"256m"` | JVM heap size for Kafka broker |
| kafka.persistence.size | string | `"10Gi"` | Storage size for Kafka log dirs |
| kafka.retentionMs | int | `86400000` | Retention in milliseconds (1 day) |
| kafka.topicPartitions | int | `8` | Number of topic partitions |
| metrics | object | `{"enabled":false}` | Prometheus metrics and ServiceMonitor |
| metrics.enabled | bool | `false` | Enable ServiceMonitor resources (requires Prometheus Operator CRDs) |
| networkPolicies | object | `{"enabled":true,"inletAllowCIDR":["0.0.0.0/0","::/0"]}` | NetworkPolicy configuration |
| networkPolicies.enabled | bool | `true` | Enable NetworkPolicies for all components |
| networkPolicies.inletAllowCIDR | list | `["0.0.0.0/0","::/0"]` | CIDR ranges allowed to send flows to inlet (UDP) |
| outlet | object | `{"replicas":1}` | Outlet configuration |
| outlet.replicas | int | `1` | Number of outlet replicas |
| snmp | object | `{"community":"public"}` | SNMP configuration for metadata enrichment |
| snmp.community | string | `"public"` | SNMP community string (used if existingSecret is empty) |

## Secrets

When `existingSecret` is set, the chart reads credentials from a Kubernetes Secret:

| Key | Used for |
|-----|----------|
| `snmpCommunity` | SNMP v2c community string for interface metadata |
| `ipInfoToken` | IPInfo API token (when `geoip.provider=ipinfo`) |

The secret is mounted into the orchestrator pod. A `render-config` init container substitutes the SNMP community into the configuration at startup.

## GeoIP

GeoIP enrichment runs as a sidecar container alongside the orchestrator, downloading and refreshing databases every 24h.

**IPInfo** (recommended for ASN + Country):
```yaml
geoip:
  enabled: true
  provider: ipinfo
existingSecret: my-akvorado-secret  # must contain key: ipInfoToken
```

**MaxMind** (GeoLite2):
```yaml
geoip:
  enabled: true
  provider: maxmind
  existingSecret: my-geoip-secret  # keys: GEOIPUPDATE_ACCOUNT_ID, GEOIPUPDATE_LICENSE_KEY
```

## Grafana Integration

When `grafana.enabled: true`, the chart creates ConfigMaps with sidecar labels for automatic discovery:

- **Datasource** (`grafana_datasource: "1"`) — configures the Akvorado datasource pointing to the console API
- **Dashboard** (`grafana_dashboard: "1"`) — provides a default dashboard with traffic-by-AS panels and a Sankey diagram

**Required Grafana plugins** (add to your Grafana deployment):
- `ovhcloud-akvorado-datasource`
- `netsage-sankey-panel`

## Network Classification

Configure interface and exporter classifiers to categorize traffic:

```yaml
akvorado:
  outlet:
    core:
      default-sampling-rate: 1  # set >0 if exporters don't advertise rate
      exporter-classifiers:
        - ClassifySite("mysite") && ClassifyRole("edge")
      interface-classifiers:
        - Interface.Name == "wan" && ClassifyConnectivity("transit") && ClassifyExternal()
        - Interface.Name startsWith "wg" && ClassifyConnectivity("vpn") && ClassifyInternal()
        - ClassifyInternal()
```

Assign ASNs to private networks:
```yaml
akvorado:
  asns:
    "64512": "My Network"
  networks:
    10.0.0.0/8:
      name: private
      role: internal
      asn: 64512
```

## MikroTik Configuration

Configure IPFIX traffic-flow on RouterOS 7:

```
/ip/traffic-flow set enabled=yes interfaces=all
/ip/traffic-flow/target add dst-address=<inlet-ip> port=2055 version=9
/snmp set enabled=yes
/snmp/community set 0 name=<community> addresses=<inlet-pod-cidr>
```

## Security

- `runAsNonRoot: true`, UID/GID 65534 (nobody)
- `readOnlyRootFilesystem: true` (except ClickHouse, Kafka, GeoIP sidecar)
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
- NetworkPolicies restrict all inter-component traffic to required ports only

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
