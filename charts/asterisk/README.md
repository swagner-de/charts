# asterisk

A Helm chart for [Asterisk](https://www.asterisk.org/), an open-source PBX with PJSIP, TLS signaling, and SRTP media encryption.

## Features

- PJSIP-based SIP stack with UDP, TCP, and TLS transports
- Inter-site peering via encrypted PJSIP trunks (TLS + SRTP)
- Upstream SIP trunk registration to VoIP providers
- Init container resolves `__PLACEHOLDER__` variables in config templates from a Kubernetes Secret
- Compatible with ExternalSecrets — supply an existing secret with the same key names
- RTP port range (default 10000–10099) exposed as individual service ports for LoadBalancer use
- Optional cert-manager Certificate for SIP-TLS via rawResources

## Install

```bash
helm install asterisk oci://ghcr.io/swagner-de/charts/asterisk --version 0.1.0
```

## Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `rtp.start` | First RTP port | `10000` |
| `rtp.end` | Last RTP port | `10099` |
| `persistence.astdb.enabled` | Persist `/var/lib/asterisk/astdb` (internal state DB) | `true` |
| `persistence.astdb.size` | AstDB volume size | `100Mi` |
| `persistence.tls.enabled` | Mount TLS certificate secret | `false` |
| `config.pjsip` | PJSIP config template with `__PLACEHOLDER__` variables | Upstream trunk + peering defaults |
| `config.extensions` | Dialplan config template with `__PLACEHOLDER__` variables | Two-site routing defaults |
| `config.modules` | Asterisk modules config | PJSIP-only defaults |
| `config.logger` | Asterisk logging config | Console + messages |
| `secretConfig.data` | Key-value pairs substituted into config templates | `CHANGEME` placeholders |
| `secretConfig.existingSecret` | Use existing Secret instead of inline data | — |

### Variable Substitution

Config templates (`config.pjsip`, `config.extensions`) use `__VARIABLE__` placeholders. At startup, an init container reads each key from the credentials secret (mounted as files at `/secrets/`) and replaces the matching `__KEY__` marker in the templates using shell parameter expansion.

For example, `password=__UPSTREAM_PASSWORD__` in `config.pjsip` becomes `password=s3cret` when the secret contains key `UPSTREAM_PASSWORD` with value `s3cret`.

This works identically with inline secrets and ExternalSecrets — the secret just needs matching key names:

```yaml
# Inline
secretConfig:
  data:
    UPSTREAM_HOST: sip.provider.de
    UPSTREAM_PASSWORD: s3cret

# Or with ExternalSecret
secretConfig:
  existingSecret:
    name: asterisk-credentials
```

Add custom placeholders by adding keys to `secretConfig.data` (or the ExternalSecret) and using `__KEY_NAME__` in the config templates.

### Extensions

The chart ships without concrete phone extensions. Add them in your values override by appending to `config.pjsip`:

```yaml
config:
  pjsip: |
    ... base config ...

    [101](local-phone)
    auth=101-auth
    aors=101
    callerid=__EXT101_CALLERID__

    [101-auth](local-phone-auth)
    username=101
    password=__EXT101_PASSWORD__

    [101](local-phone-aor)
```

Then add `EXT101_PASSWORD` to `secretConfig.data` or your ExternalSecret.

### LoadBalancer

SIP and RTP require a LoadBalancer with `externalTrafficPolicy: Local` for proper NAT handling:

```yaml
service:
  main:
    type: LoadBalancer
    externalTrafficPolicy: Local
    annotations:
      metallb.universe.tf/loadBalancerIPs: "10.10.10.10"
```

See [values.yaml](values.yaml) for the full list of configurable values.

## Security

- `runAsNonRoot: true`, UID 1000 / GID 999 (asterisk)
- `readOnlyRootFilesystem: false` (Asterisk writes runtime state)
- `allowPrivilegeEscalation: false`
- Capabilities: CHOWN, SETUID, SETGID (required by Asterisk); all others dropped
- Seccomp profile: `RuntimeDefault`
- Init container runs with `readOnlyRootFilesystem: true`
