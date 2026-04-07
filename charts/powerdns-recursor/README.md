# powerdns-recursor

A Helm chart for [PowerDNS Recursor](https://www.powerdns.com/recursor/), a high-performance recursive DNS resolver.

## Features

- DNSSEC validation enabled by default
- YAML-based configuration (recursor.yml)
- DNS served on port 5353 internally (mapped to 53 via Service, no `NET_BIND_SERVICE` needed)
- Configurable forwarding zones

## Install

```bash
helm install powerdns-recursor oci://ghcr.io/swagner-de/charts/powerdns-recursor --version 0.1.10
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `config.incoming.allow_from` | Networks allowed to query | `[10.0.0.0/8, 127.0.0.0/8]` |
| `config.dnssec.validation` | DNSSEC validation mode | `validate` |
| `config.recursor.forward_zones` | Forward zones configuration | `[]` |
| `config.logging.loglevel` | Log verbosity level | `4` |

### Forward Zones

Configure DNS forwarding for specific zones:

```yaml
config:
  recursor:
    forward_zones:
      - zone: "internal.example.com"
        forwarders:
          - 10.0.0.1
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID 1883
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
- DNS on port 5353 internally avoids need for privileged ports
