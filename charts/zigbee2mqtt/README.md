# zigbee2mqtt

![Version: 0.8.1](https://img.shields.io/badge/Version-0.8.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2.10.1](https://img.shields.io/badge/AppVersion-2.10.1-informational?style=flat-square)
Zigbee-to-MQTT bridge for smart home devices
**Homepage:** <https://www.zigbee2mqtt.io/>

## Features
- USB Zigbee adapter passthrough via hostPath mount
- udev access for device detection
- Configurable MQTT connection via environment variables

## Install

```bash
helm install zigbee2mqtt oci://ghcr.io/swagner-de/charts/zigbee2mqtt
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| controllers | object | `{"main":{"containers":{"main":{"env":null,"resources":{"requests":{"cpu":"200m","memory":"200Mi"}}}}}}` | Controller configuration |
| controllers.main.containers.main.env | string | `nil` | Environment variables |
| controllers.main.containers.main.resources | object | `{"requests":{"cpu":"200m","memory":"200Mi"}}` | Resource requests and limits |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration |
| ingress.main.enabled | bool | `false` | Enable ingress |
| persistence | object | `{"data":{"enabled":true,"type":"emptyDir"}}` | Persistent storage configuration |
| persistence.data | object | `{"enabled":true,"type":"emptyDir"}` | Data volume |
| persistence.data.enabled | bool | `true` | Enable data persistence |
| persistence.data.type | string | `"emptyDir"` | Volume type |
| usbPath | string | `"/dev/ttyACM0"` | Host path to USB Zigbee adapter |

## Security
- `runAsNonRoot: true`, UID 65534 (nobody), GID 20 (dialout)
- `readOnlyRootFilesystem: true`
- `privileged: true` (required for USB device access)
- `SYS_ADMIN` capability added (required for USB device access)
- Seccomp profile: `RuntimeDefault`

**Note:** This chart requires privileged mode and `SYS_ADMIN` capability for USB Zigbee adapter access. The container needs direct access to the host USB device. If your cluster uses a device plugin (e.g., `intel-device-plugins`), you may be able to reduce these privileges.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
