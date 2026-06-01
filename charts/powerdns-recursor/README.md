# powerdns-recursor

![Version: 0.2.1](https://img.shields.io/badge/Version-0.2.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 5.3.7](https://img.shields.io/badge/AppVersion-5.3.7-informational?style=flat-square)
PowerDNS recursive DNS resolver with DNSSEC validation
**Homepage:** <https://www.powerdns.com/recursor/>

## Features
- DNSSEC validation enabled by default
- YAML-based configuration (recursor.yml)
- DNS served on port 5353 internally (mapped to 53 via Service, no `NET_BIND_SERVICE` needed)
- Configurable forwarding zones

## Install

```bash
helm install powerdns-recursor oci://ghcr.io/swagner-de/charts/powerdns-recursor
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| config | object | `{"dnssec":{"log_bogus":true,"validation":"validate"},"incoming":{"allow_from":["10.0.0.0/8","127.0.0.0/8"]},"logging":{"loglevel":4},"recursor":{"forward_zones":[{"forwarders":["10.0.0.1"],"zone":"somezone.com"}]}}` | PowerDNS Recursor configuration (recursor.yml format) |
| config.dnssec | object | `{"log_bogus":true,"validation":"validate"}` | DNSSEC settings |
| config.dnssec.log_bogus | bool | `true` | Log DNSSEC bogus results |
| config.dnssec.validation | string | `"validate"` | DNSSEC validation mode |
| config.incoming | object | `{"allow_from":["10.0.0.0/8","127.0.0.0/8"]}` | Incoming query settings |
| config.incoming.allow_from | list | `["10.0.0.0/8","127.0.0.0/8"]` | Networks allowed to query |
| config.logging | object | `{"loglevel":4}` | Logging settings |
| config.logging.loglevel | int | `4` | Log verbosity level |
| config.recursor | object | `{"forward_zones":[{"forwarders":["10.0.0.1"],"zone":"somezone.com"}]}` | Recursor settings |
| config.recursor.forward_zones | list | `[{"forwarders":["10.0.0.1"],"zone":"somezone.com"}]` | Forward zone configuration |

## Security
- `runAsNonRoot: true`, UID 1883
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
- DNS on port 5353 internally avoids need for privileged ports

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
