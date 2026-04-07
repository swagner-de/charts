# powerdns-auth

A Helm chart for [PowerDNS Authoritative Server](https://www.powerdns.com/auth/), a versatile authoritative DNS server.

## Features

- LMDB backend for zone storage
- Init container for automatic zone loading from zone files
- Optional external-dns sidecar for Kubernetes-driven DNS record management
- DNS served on port 5353 internally (mapped to 53 via Service, no `NET_BIND_SERVICE` needed)
- API/web interface on port 8081

## Install

```bash
helm install powerdns-auth oci://ghcr.io/swagner-de/charts/powerdns-auth --version 0.4.0
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `persistence.data.size` | LMDB data volume size | `2Gi` |
| `config` | PowerDNS config (pdns.conf format) | LMDB + webserver defaults |
| `secretConfig.data` | Sensitive PowerDNS config (api-key, etc.) | `api-key=CHANGEME` |
| `secretConfig.existingSecret` | Use existing Secret for sensitive config | — |
| `zone.name` | DNS zone name | `my-zone` |
| `zone.zonefile` | Zone file content | Example SOA record |
| `externalDNS.enabled` | Enable external-dns sidecar | `false` |
| `externalDNS.apiKey.data` | API key for external-dns | `change-me` |

### External-DNS Integration

Enable automatic DNS record management from Kubernetes resources:

```yaml
externalDNS:
  enabled: true
  apiKey:
    existingSecret:
      name: pdns-api
      key: API_KEY
  config:
    sources:
      - gateway-httproute
    provider: pdns
    pdns-server: http://powerdns-auth:8081
    domain-filter: example.com
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID/GID 1883
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
- DNS on port 5353 internally avoids need for privileged ports
