# unifi-os-server

![Version: 0.2.0](https://img.shields.io/badge/Version-0.2.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v1.4.0](https://img.shields.io/badge/AppVersion-v1.4.0-informational?style=flat-square)
UniFi OS Server - self-hosted UniFi Network with Organizations, IdP, and Site Magic SD-WAN support
**Homepage:** <https://github.com/lemker/unifi-os-server>

## Important: Security Context

This chart runs in **privileged mode**. The standard security primitives used by other
charts in this repository do **not** apply here:

- ❌ `runAsNonRoot` — container runs systemd, requires root
- ❌ `readOnlyRootFilesystem` — systemd and services need writable paths everywhere
- ❌ `allowPrivilegeEscalation: false` — requires privilege escalation
- ❌ `capabilities.drop: [ALL]` — needs full capability set
- ❌ `seccompProfile: RuntimeDefault` — systemd syscalls would be blocked

UniFi OS Server runs every component as systemd services internally, which requires
host cgroup access and tmpfs mounts. This is an inherent requirement of the upstream
project and cannot be worked around.

**Recommendation:** Run this in an isolated namespace with appropriate NetworkPolicies.

## Install

```bash
helm install unifi-os-server oci://ghcr.io/swagner-de/charts/unifi-os-server
```

## Device Adoption

After deployment, set the inform URL on your UniFi devices:

```bash
ssh admin@<device-ip>
set-inform http://<UOS_SYSTEM_IP>:8080/inform
```

By default, `UOS_SYSTEM_IP` is set to the pod IP. Override it in your values if you
use a LoadBalancer or external DNS name for device communication.

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 11443 | TCP | GUI and API (HTTPS) |
| 8080 | TCP | Device inform / communication |
| 3478 | UDP | STUN (required for remote management) |
| 10003 | UDP | Device discovery |

Additional ports (8444, 9543, 6789, 5005, 28082, 5671, 8880-8882 TCP; 5514 UDP syslog)
may be needed depending on your feature usage.

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| persistence.data.accessMode | string | `"ReadWriteOnce"` |  |
| persistence.data.enabled | bool | `true` |  |
| persistence.data.size | string | `"20Gi"` |  |

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
