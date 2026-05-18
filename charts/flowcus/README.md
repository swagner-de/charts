# flowcus

![Version: 0.1.2](https://img.shields.io/badge/Version-0.1.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 0.10.7](https://img.shields.io/badge/AppVersion-0.10.7-informational?style=flat-square)
Lightweight NetFlow/IPFIX collector with embedded web UI and columnar storage
**Homepage:** <https://github.com/consi/flowcus>

## Features
- Lightweight NetFlow v5/v9 and IPFIX collector with embedded columnar storage
- Built-in React web UI with flow query language (FQL)
- Single container deployment with zero external dependencies
- Persistent storage for flow data and settings
- Works with MikroTik Traffic Flow (NetFlow v5/v9)

## Install

```bash
helm install flowcus oci://ghcr.io/swagner-de/charts/flowcus
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| persistence | object | `{"data":{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}}` | Persistent storage configuration |
| persistence.data | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}` | Flow data and settings volume |
| persistence.data.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.data.enabled | bool | `true` | Enable persistent storage for flow data |
| persistence.data.size | string | `"10Gi"` | Volume size |

## MikroTik Configuration

Configure Traffic Flow on your MikroTik router to send NetFlow to this collector:

```
/ip traffic-flow set enabled=yes interfaces=all
/ip traffic-flow target add dst-address=<flowcus-ip> port=4739 version=9
```

## Security
- `runAsNonRoot: true`, UID/GID 65534 (nobody)
- `readOnlyRootFilesystem: false` (required for embedded database writes)
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
