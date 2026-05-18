# powerdns-auth

![Version: 0.4.4](https://img.shields.io/badge/Version-0.4.4-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 5.0.4](https://img.shields.io/badge/AppVersion-5.0.4-informational?style=flat-square)
PowerDNS authoritative DNS server with LMDB backend and external-dns support
**Homepage:** <https://www.powerdns.com/auth/>

## Features
- LMDB backend for zone storage
- Init container for automatic zone loading from zone files
- Optional external-dns sidecar for Kubernetes-driven DNS record management
- DNS served on port 5353 internally (mapped to 53 via Service, no `NET_BIND_SERVICE` needed)
- API/web interface on port 8081

## Install

```bash
helm install powerdns-auth oci://ghcr.io/swagner-de/charts/powerdns-auth
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| config | string | `"launch=lmdb\nlmdb-filename=/data/pdns.lmdb\n\n# Logging\nlog-dns-queries=yes\nloglevel=4\n\nwebserver=yes\nwebserver-address=0.0.0.0\nwebserver-allow-from=10.0.0.0/8\n"` | PowerDNS configuration (pdns.conf format) |
| externalDNS | object | `{"apiKey":{"data":"change-me"},"config":{"pdns-server":"http://powerdns-auth:8081","provider":"pdns","sources":[]},"enabled":false}` | External-DNS sidecar configuration |
| externalDNS.apiKey | object | `{"data":"change-me"}` | API key for external-dns |
| externalDNS.apiKey.data | string | `"change-me"` | API key value |
| externalDNS.config | object | `{"pdns-server":"http://powerdns-auth:8081","provider":"pdns","sources":[]}` | External-DNS configuration |
| externalDNS.config.pdns-server | string | `"http://powerdns-auth:8081"` | PowerDNS server URL |
| externalDNS.config.provider | string | `"pdns"` | DNS provider |
| externalDNS.config.sources | list | `[]` | DNS record sources |
| externalDNS.enabled | bool | `false` | Enable external-dns sidecar |
| persistence | object | `{"data":{"size":"2Gi","type":"persistentVolumeClaim"}}` | Persistent storage configuration |
| persistence.data | object | `{"size":"2Gi","type":"persistentVolumeClaim"}` | Data volume for LMDB |
| persistence.data.size | string | `"2Gi"` | Data volume size |
| persistence.data.type | string | `"persistentVolumeClaim"` | Volume type |
| secretConfig | object | `{"data":"api=yes\napi-key=CHANGEME\n"}` | Sensitive PowerDNS configuration (api-key, etc.) |
| secretConfig.data | string | `"api=yes\napi-key=CHANGEME\n"` | Secret config data |
| zone | object | `{"name":"my-zone","zonefile":"$TTL 2d    ; default TTL for zone\n$ORIGIN myzone.com. ; base domain-name\n; Start of Authority RR defining the key characteristics of the zone (domain)\n@         IN      SOA   ns1.myzone.com. hostmaster.myzone.com. (\n                                2003080800 ; serial number\n                                12h        ; refresh\n                                15m        ; update retry\n                                3w         ; expiry\n                                2h         ; minimum\n                                )\n; name server RR for the domain\n          IN      NS      ns1.myzone.com.\nns1        IN      A      10.0.0.1\nns1        IN      AAAA   fdfd::\n"}` | DNS zone configuration |
| zone.name | string | `"my-zone"` | Zone name |
| zone.zonefile | string | `"$TTL 2d    ; default TTL for zone\n$ORIGIN myzone.com. ; base domain-name\n; Start of Authority RR defining the key characteristics of the zone (domain)\n@         IN      SOA   ns1.myzone.com. hostmaster.myzone.com. (\n                                2003080800 ; serial number\n                                12h        ; refresh\n                                15m        ; update retry\n                                3w         ; expiry\n                                2h         ; minimum\n                                )\n; name server RR for the domain\n          IN      NS      ns1.myzone.com.\nns1        IN      A      10.0.0.1\nns1        IN      AAAA   fdfd::\n"` | Zone file content |

## Security
- `runAsNonRoot: true`, UID/GID 1883
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
- DNS on port 5353 internally avoids need for privileged ports

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
