# homeassistant

![Version: 1.15.0](https://img.shields.io/badge/Version-1.15.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2026.6.0](https://img.shields.io/badge/AppVersion-2026.6.0-informational?style=flat-square)
Home automation platform with optional LDAP, Matter bridge, and CNPG database support
**Homepage:** <https://www.home-assistant.io/>

## Features
- Optional LDAP authentication sidecar
- Optional Matter bridge (matterbridge) with Layer 2 network attachment for mDNS
- CloudNativePG (CNPG) database support with automatic connection configuration
- Optional Prometheus ServiceMonitor with bearer token auth

## Install

```bash
helm install homeassistant oci://ghcr.io/swagner-de/charts/homeassistant
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cnpg | object | `{"clusterName":"homeassistant-db","enabled":false,"prependReleaseName":false}` | CloudNativePG database configuration |
| cnpg.clusterName | string | `"homeassistant-db"` | CNPG cluster name |
| cnpg.enabled | bool | `false` | Enable CNPG operator for database |
| cnpg.prependReleaseName | bool | `false` | Prepend release name and chart name to clusterName (use when CNPG cluster is deployed via rawResources) |
| controllers | object | `{"main":{"containers":{"ldap":{"enabled":false},"main":{"env":[{"name":"TZ","value":"Europe/Berlin"}]}}}}` | Controller configuration |
| controllers.main.containers.ldap | object | `{"enabled":false}` | LDAP authentication sidecar |
| controllers.main.containers.ldap.enabled | bool | `false` | Enable LDAP sidecar |
| controllers.main.containers.main.env | list | `[{"name":"TZ","value":"Europe/Berlin"}]` | Environment variables |
| hamcp | object | `{"enabled":false,"secretPath":"/private"}` | HA MCP Server configuration |
| hamcp.enabled | bool | `false` | Enable HA MCP Server controller |
| hamcp.secretPath | string | `"/private"` | MCP secret path (used when secretName is not set) |
| ingress | object | `{"main":{"enabled":false}}` | Ingress configuration |
| ingress.main.enabled | bool | `false` | Enable ingress |
| ldapSecret | string | `"homeassistant-ldap"` | External secret or define the secrets below |
| matterbridge | object | `{"enabled":false,"mdns_interface":"net1"}` | Matterbridge configuration |
| matterbridge.enabled | bool | `false` | Enable Matter bridge controller |
| matterbridge.mdns_interface | string | `"net1"` | mDNS network interface |
| persistence | object | `{"config":{"accessMode":"ReadWriteOnce","size":"5Gi"}}` | Persistent storage configuration |
| persistence.config.accessMode | string | `"ReadWriteOnce"` | Storage access mode |
| persistence.config.size | string | `"5Gi"` | Volume size |
| secrets | object | `{"ldap":{"enabled":false,"stringData":{"HA_LDAP_BIND_DN":"cn=ldapservice,ou=users,dc=ldap,dc=goauthentik,dc=io","HA_LDAP_BIND_DN_PASSWORD":""}},"matterbridge":{"enabled":false},"prometheus":{"enabled":false,"stringData":{"token":""}}}` | Kubernetes secrets configuration |
| secrets.ldap | object | `{"enabled":false,"stringData":{"HA_LDAP_BIND_DN":"cn=ldapservice,ou=users,dc=ldap,dc=goauthentik,dc=io","HA_LDAP_BIND_DN_PASSWORD":""}}` | LDAP secret |
| secrets.ldap.enabled | bool | `false` | Enable LDAP secret creation |
| secrets.ldap.stringData | object | `{"HA_LDAP_BIND_DN":"cn=ldapservice,ou=users,dc=ldap,dc=goauthentik,dc=io","HA_LDAP_BIND_DN_PASSWORD":""}` | LDAP secret data |
| secrets.matterbridge | object | `{"enabled":false}` | Matterbridge secret |
| secrets.matterbridge.enabled | bool | `false` | Enable Matterbridge secret creation |
| secrets.prometheus | object | `{"enabled":false,"stringData":{"token":""}}` | Prometheus secret |
| secrets.prometheus.enabled | bool | `false` | Enable Prometheus secret creation |
| secrets.prometheus.stringData | object | `{"token":""}` | Prometheus secret data |

## Security
- `runAsNonRoot: true`, UID/GID 568
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
