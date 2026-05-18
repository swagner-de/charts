# netra

![Version: 0.1.1](https://img.shields.io/badge/Version-0.1.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.0.8](https://img.shields.io/badge/AppVersion-1.0.8-informational?style=flat-square)
ASN traffic analysis dashboard for NetFlow/IPFIX flow data
**Homepage:** <https://github.com/consi/netra>

## Features
- Analyzes NetFlow v5/v9 and IPFIX data to map traffic to autonomous systems (ASNs)
- Built-in real-time web dashboard with SSE updates
- Prometheus-compatible metrics endpoint at `/metrics`
- Automatic ASN database download and daily refresh from iptoasn.com
- Works with MikroTik Traffic Flow (NetFlow v5/v9)

## Install

```bash
helm install netra oci://ghcr.io/swagner-de/charts/netra
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| persistence | object | `{"data":{"accessMode":"ReadWriteOnce","enabled":true,"size":"1Gi"}}` | Persistent storage configuration |
| persistence.data | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"1Gi"}` | ASN database volume |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.enabled | bool | `true` | Enable persistent storage for ASN database |
| persistence.data.size | string | `"1Gi"` | Volume size |

## MikroTik Configuration

Configure Traffic Flow on your MikroTik router to send NetFlow to this collector:

```
/ip traffic-flow set enabled=yes interfaces=all
/ip traffic-flow target add dst-address=<netra-ip> port=2055 version=9
```

## Security
- `runAsNonRoot: true`, UID/GID 65534 (nobody)
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
