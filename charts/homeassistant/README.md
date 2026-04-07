# homeassistant

A Helm chart for [Home Assistant](https://www.home-assistant.io/), an open-source home automation platform.

## Features

- Optional LDAP authentication sidecar
- Optional Matter bridge (matterbridge) with Layer 2 network attachment for mDNS
- CloudNativePG (CNPG) database support with automatic connection configuration
- Optional Prometheus ServiceMonitor with bearer token auth

## Install

```bash
helm install homeassistant oci://ghcr.io/swagner-de/charts/homeassistant --version 1.8.2
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `persistence.config.size` | Config volume size | `5Gi` |
| `controllers.main.containers.ldap.enabled` | Enable LDAP sidecar | `false` |
| `secrets.ldap.enabled` | Create LDAP secret (set credentials in `stringData`) | `false` |
| `secrets.prometheus.enabled` | Create Prometheus token secret | `false` |
| `matterbridge.enabled` | Enable Matter bridge controller | `false` |
| `cnpg.enabled` | Use CNPG operator for database | `false` |
| `cnpg.clusterName` | CNPG cluster name | `homeassistant-db` |
| `ingress.main.enabled` | Enable ingress | `false` |

### LDAP Authentication

Enable LDAP and provide credentials:

```yaml
controllers:
  main:
    containers:
      ldap:
        enabled: true

secrets:
  ldap:
    enabled: true
    stringData:
      HA_LDAP_BIND_DN_PASSWORD: "your-password"
      HA_LDAP_BIND_DN: "cn=ldapservice,ou=users,dc=example,dc=com"
```

### Matter Bridge

For Matter/Thread support with mDNS, enable the matterbridge controller and configure a network attachment:

```yaml
matterbridge:
  enabled: true
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID/GID 568
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
