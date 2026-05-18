# mosquitto

![Version: 0.1.2](https://img.shields.io/badge/Version-0.1.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2.0.22](https://img.shields.io/badge/AppVersion-2.0.22-informational?style=flat-square)
Eclipse Mosquitto MQTT broker with TLS and password authentication
**Homepage:** <https://mosquitto.org/>

## Features
- Eclipse Mosquitto MQTT broker with TLS encryption
- Password authentication via init container
- Persistent data storage
- LoadBalancer service for direct MQTT access

## Install

```bash
helm install mosquitto oci://ghcr.io/swagner-de/charts/mosquitto
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| config | string | `"listener 1883\npersistence_location /mosquitto/data/\npassword_file /mosquitto/passwd/passwd\nallow_anonymous false\nlog_dest stdout\nlog_type all\n"` | Mosquitto configuration file content |
| existingUsersSecret | string | `""` | Existing secret containing a `users` key with YAML-formatted user list When set, the chart will not create its own users secret Expected format: YAML list with name/password entries, e.g.:   - name: myuser     password: mypassword |
| persistence | object | `{"data":{"accessMode":"ReadWriteOnce","enabled":false,"size":"2Gi"}}` | Persistent storage configuration |
| persistence.data | object | `{"accessMode":"ReadWriteOnce","enabled":false,"size":"2Gi"}` | Data volume for mosquitto persistence |
| service | object | `{"main":{"type":"ClusterIP"}}` | Service configuration |
| tlsSecretName | string | `""` | TLS certificate secret name (e.g. from cert-manager) Set to enable TLS volume mount at /tls |
| users | list | `[]` | Mosquitto user credentials for password authentication Ignored when existingUsersSecret is set |

## Security
- `runAsNonRoot: true`, UID/GID 1883 (mosquitto)
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
- TLS encryption on port 8883

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
