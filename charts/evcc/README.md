# evcc

Local EV charging and home energy management

## Configuration

The default installation starts the setup UI without a YAML configuration. Devices,
the sponsor token, administrator password, and charging sessions persist on the
`/data` PVC (SQLite, including WAL/SHM files). Use a local filesystem-backed PVC,
not NFS/SMB. A single replica and Recreate strategy avoid concurrent database writers.

For declarative configuration, enable `config.enabled` and provide `config.content`.
For credentials in a configuration file, use `config.existingSecret` with an
`evcc.yaml` key. File-defined settings are not editable in the web UI. Secrets can
also be provided through `env` secretKeyRefs or `envFromSecrets`.

The web/API service listens on TCP 7070. The optional OCPP service listens on TCP
8887. Both are ClusterIP by default. Configure `route.main` for Gateway API or
`ingress.main` for an Ingress controller.

## Security

evcc protects administration with its own password, but normal charging controls
are not authentication-gated. Protect the web/API service with a trusted reverse
proxy or VPN. The OCPP listener has no native TLS or credential validation:
terminate WSS at a reverse proxy and enforce charger authentication there. Do not
expose plain port 8887 to an untrusted network. The OCPP route must not redirect to
interactive SSO. Preserve WebSocket upgrades and the `ocpp1.6` subprotocol.

## Validation

```sh
helm dependency build charts/evcc
helm template test charts/evcc > rendered.yaml
trivy config --severity HIGH,CRITICAL --exit-code 1 rendered.yaml
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| config | object | `{"content":"","enabled":false,"existingSecret":""}` | Configure devices in the web UI (persisted in SQLite) or supply optional YAML. |
| config.content | string | `""` | Non-secret evcc.yaml contents. File-defined settings are not editable in the UI. |
| config.existingSecret | string | `""` | Use an existing Secret instead of a generated ConfigMap (key: evcc.yaml). |
| controllers.main.containers.main.resources.limits.memory | string | `"512Mi"` |  |
| controllers.main.containers.main.resources.requests.cpu | string | `"50m"` |  |
| controllers.main.containers.main.resources.requests.memory | string | `"128Mi"` |  |
| env | object | `{}` | Extra environment variables, including secretKeyRef entries. |
| envFromSecrets | list | `[]` | Secrets to consume as environment variables. Never store credentials in values. |
| extraArgs | list | `[]` | Extra command-line arguments. |
| ingress | object | `{"main":{"enabled":false}}` | Standard bjw-s ingress configuration (disabled by default). |
| ocpp.enabled | bool | `true` |  |
| persistence.data.accessMode | string | `"ReadWriteOnce"` |  |
| persistence.data.enabled | bool | `true` |  |
| persistence.data.size | string | `"2Gi"` |  |
| route | object | `{"main":{"enabled":false,"hostnames":[],"parentRefs":[]}}` | Standard bjw-s Gateway API HTTPRoute configuration (disabled by default). |
| service.main.type | string | `"ClusterIP"` |  |
| service.ocpp.type | string | `"ClusterIP"` |  |
