# zigbee2mqtt

A Helm chart for [Zigbee2MQTT](https://www.zigbee2mqtt.io/), a Zigbee-to-MQTT bridge for smart home devices.

## Features

- USB Zigbee adapter passthrough via hostPath mount
- udev access for device detection
- Configurable MQTT connection via environment variables

## Install

```bash
helm install zigbee2mqtt oci://ghcr.io/swagner-de/charts/zigbee2mqtt --version 0.7.2
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `usbPath` | Host path to USB Zigbee adapter | `/dev/ttyACM0` |
| `persistence.data.enabled` | Enable persistent data storage | `true` |
| `persistence.data.type` | Storage type for data | `emptyDir` |
| `controllers.main.containers.main.env` | Environment variables (MQTT settings) | `{}` |

### MQTT Connection

Configure the MQTT broker connection via environment variables:

```yaml
controllers:
  main:
    containers:
      main:
        env:
          ZIGBEE2MQTT_CONFIG_MQTT_BASE_TOPIC: zigbee2mqtt
          ZIGBEE2MQTT_CONFIG_MQTT_SERVER: mqtts://mosquitto.example.com
          ZIGBEE2MQTT_CONFIG_MQTT_USER: zigbee2mqtt
        envFrom:
          - secret: zigbee2mqtt-mqtt  # Secret containing ZIGBEE2MQTT_CONFIG_MQTT_PASSWORD
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID 65534 (nobody), GID 20 (dialout)
- `readOnlyRootFilesystem: true`
- `privileged: true` (required for USB device access)
- `SYS_ADMIN` capability added (required for USB device access)
- Seccomp profile: `RuntimeDefault`

**Note:** This chart requires privileged mode and `SYS_ADMIN` capability for USB Zigbee adapter access. The container needs direct access to the host USB device. If your cluster uses a device plugin (e.g., `intel-device-plugins`), you may be able to reduce these privileges.
