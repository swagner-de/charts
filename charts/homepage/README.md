# homepage

![Version: 0.5.4](https://img.shields.io/badge/Version-0.5.4-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v1.12.3](https://img.shields.io/badge/AppVersion-v1.12.3-informational?style=flat-square)
Customizable application dashboard for your homelab
**Homepage:** <https://gethomepage.dev/>

## Features
- Kubernetes service discovery via ClusterRole RBAC (Gateway API, Ingress, Traefik IngressRoutes)
- Configuration via `values.yaml` — services, bookmarks, widgets, and settings
- Secret variable injection (`HOMEPAGE_VAR_*` / `HOMEPAGE_FILE_*`) for API keys
- Read-only root filesystem with emptyDir for runtime config

## Install

```bash
helm install homepage oci://ghcr.io/swagner-de/charts/homepage
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 4.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| additionalSecrets | list | `[]` | Additional external secrets to mount as environment variables A list of external secrets to be passed as environment variables to the container. |
| config | object | `{"bookmarks":[{"Section 1":[{"Scanner":[{"href":"scanner.local","icon":"mdi-scanner"}]}]}],"kubernetes":{"gateway":true,"mode":"cluster"},"services":[],"settings":{"language":"de","title":"Home Infra"},"widgets":[{"kubernetes":{"cluster":{"cpu":true,"label":"cluster","memory":true,"show":true,"showLabel":true},"nodes":{"cpu":true,"memory":true,"show":true,"showLabel":true}}},{"openmeteo":{"cache":5,"format":{"maximumFractionDigits":1},"latitude":49.488888,"longitude":8.469167,"timezone":"Europe/Berlin","units":"metric"}}]}` | Homepage application configuration |
| config.bookmarks | list | `[{"Section 1":[{"Scanner":[{"href":"scanner.local","icon":"mdi-scanner"}]}]}]` | Bookmark definitions |
| config.kubernetes | object | `{"gateway":true,"mode":"cluster"}` | Kubernetes service discovery configuration |
| config.kubernetes.gateway | bool | `true` | Enable Gateway API discovery |
| config.kubernetes.mode | string | `"cluster"` | Discovery mode (cluster or disabled) |
| config.services | list | `[]` | Service definitions |
| config.settings | object | `{"language":"de","title":"Home Infra"}` | General settings |
| config.settings.language | string | `"de"` | UI language |
| config.settings.title | string | `"Home Infra"` | Dashboard title |
| config.widgets | list | `[{"kubernetes":{"cluster":{"cpu":true,"label":"cluster","memory":true,"show":true,"showLabel":true},"nodes":{"cpu":true,"memory":true,"show":true,"showLabel":true}}},{"openmeteo":{"cache":5,"format":{"maximumFractionDigits":1},"latitude":49.488888,"longitude":8.469167,"timezone":"Europe/Berlin","units":"metric"}}]` | Widget definitions |
| controllers | object | `{"main":{"containers":{"main":{"env":{"HOMEPAGE_ALLOWED_HOSTS":"mydomain.com"},"resources":{"limits":{"cpu":"500m","memory":"500Mi"},"requests":{"cpu":"100m","memory":"100Mi"}}}}}}` | Controller configuration |
| controllers.main.containers.main.env | object | `{"HOMEPAGE_ALLOWED_HOSTS":"mydomain.com"}` | Environment variables |
| controllers.main.containers.main.env.HOMEPAGE_ALLOWED_HOSTS | string | `"mydomain.com"` | Allowed hosts for Homepage |
| controllers.main.containers.main.resources | object | `{"limits":{"cpu":"500m","memory":"500Mi"},"requests":{"cpu":"100m","memory":"100Mi"}}` | Resource requests and limits |
| secretReplacements | object | `{}` | Secret variable replacements (HOMEPAGE_VAR_* / HOMEPAGE_FILE_*) for API keys |

## Security
- `runAsUser: 1000`, `runAsGroup: 1000`
- `readOnlyRootFilesystem: true`
- `allowPrivilegeEscalation: false`
- All capabilities dropped
- Seccomp profile: `RuntimeDefault`
- ServiceAccount with minimal ClusterRole for Kubernetes resource discovery

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
