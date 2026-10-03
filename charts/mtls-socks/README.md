# mtls-socks

![Version: 0.1.1](https://img.shields.io/badge/Version-0.1.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v1.11.3](https://img.shields.io/badge/AppVersion-v1.11.3-informational?style=flat-square)
SOCKS5 proxy exposed only over mutually-authenticated TLS via ghostunnel, with cert-manager-issued per-user client certificates
**Homepage:** <https://github.com/ghostunnel/ghostunnel>

## Architecture

A SOCKS5 proxy that is reachable **only** over mutually-authenticated TLS. Each
client runs a local `ghostunnel client` that presents its certificate, connects
to the exposed port, and gets a plaintext local SOCKS5 port.

```
┌────────────────┐   mTLS    ┌──────────────────────────────────────────┐
│  client host   │──────────▶│  Service (LoadBalancer/ClusterIP) :8443    │
│  ghostunnel    │  client   │                                            │
│  client mode   │  cert     │   ┌─────────────┐   loopback  ┌─────────┐  │
│  → local :1080 │           │   │ ghostunnel  │────────────▶│  socks  │  │
└────────────────┘           │   │ server mode │ 127.0.0.1   │ :1080   │  │
                             │   │  :8443 mTLS │             │ (loop)  │  │
                             │   └─────────────┘             └────┬────┘  │
                             └──────────────────────────────────── │ ─────┘
                                                                    ▼
                                                              public internet
                                                       (private ranges blocked)
```

- **ghostunnel** (server mode) terminates client mTLS, verifies the client
  certificate against the chart's CA, checks the CN against the `--allow-cn`
  allowlist, and forwards the raw TCP stream to the loopback SOCKS server.
- **socks** is a distroless SOCKS5 server bound to `127.0.0.1` inside the pod.
  It runs with no SOCKS-level authentication — mTLS is the sole access gate and
  the server is unreachable except via the in-pod ghostunnel.
- **cert-manager** provisions a fresh self-signed CA, a CA issuer, the server
  certificate, and one client certificate per enrolled user.
- **NetworkPolicies** default-deny; inbound is limited to the mTLS port and
  outbound to DNS + the public IPv4 and IPv6 internet.

## Requirements

- [cert-manager](https://cert-manager.io/) installed in the cluster.

## Install

```bash
helm install mtls-socks oci://ghcr.io/swagner-de/charts/mtls-socks
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Enrolling users

Each entry under `users` becomes a cert-manager `Certificate` (client auth) and
a ghostunnel `--allow-cn` entry:

```yaml
users:
  - name: alice
    commonName: alice
    duration: 2160h   # 90d
```

The client key material lands in a Secret named `<release>-user-<name>` (keys
`tls.crt`, `tls.key`, `ca.crt`) — see [Using the proxy](#using-the-proxy-client)
for how a client consumes it.

## Using the proxy (client)

Each client authenticates with its own certificate over an mTLS tunnel; there is
no username/password. The steps below have been verified end-to-end.

**1. Install ghostunnel locally** — the same tool runs in client mode:

```bash
brew install ghostunnel        # macOS; see ghostunnel.dev for other platforms
```

**2. Obtain your credential bundle** — three files: your client certificate,
its private key, and the CA that signed the server. Whoever enrolled you either
hands you these or, with cluster access, you export them from your user Secret:

```bash
kubectl -n <namespace> get secret <release>-user-<name> \
  -o jsonpath='{.data.tls\.crt}' | base64 -d > client.crt
kubectl -n <namespace> get secret <release>-user-<name> \
  -o jsonpath='{.data.tls\.key}' | base64 -d > client.key   # keep this private
kubectl -n <namespace> get secret <release>-user-<name> \
  -o jsonpath='{.data.ca\.crt}'  | base64 -d > ca.crt
```

**3. Start the local tunnel** — this presents your certificate to the server and
exposes a plaintext SOCKS5 port on loopback:

```bash
ghostunnel client \
  --listen 127.0.0.1:1080 \
  --target <proxy-host>:<proxy-port> \
  --cert client.crt --key client.key --cacert ca.crt \
  --override-server-name <server-name>
```

- `<proxy-host>:<proxy-port>` — the address the Service is exposed on (ask your
  operator). The default listen port is `8443`.
- `<server-name>` — must match one of the server certificate's `dnsNames`
  (`pki.server.dnsNames`); ghostunnel verifies the server against it.

**4. Point applications at the local SOCKS5 port** `127.0.0.1:1080`:

```bash
curl --socks5-hostname 127.0.0.1:1080 https://example.com
# or a browser / app configured with SOCKS5 host 127.0.0.1 port 1080
```

Using `--socks5-hostname` (not `--socks5`) sends DNS resolution through the
tunnel as well. Your certificate is valid for the `duration` set at enrollment;
re-export the Secret after renewal to refresh it. Access is revoked by removing
you from `users` — the server stops admitting your CN and your Secret is deleted.

## Security

- Both containers run distroless: `runAsNonRoot: true`, UID/GID 65532,
  `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, all
  capabilities dropped, seccomp `RuntimeDefault`.
- SOCKS server binds to loopback only; the sole entry point is ghostunnel mTLS.
- Egress NetworkPolicy is value-driven (`networkPolicy.egressRules`); the
  default permits IPv6 GUA and blocks private, CGNAT, and metadata IPv4 ranges.
  DNS is always permitted.

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| ghostunnel.listenPort | int | `8443` | Port the mTLS tunnel listens on (exposed by the Service) |
| global | object | `{"createDefaultServiceAccount":false}` | bjw-s common global options |
| networkPolicy.egressRules | list | `[{"to":[{"ipBlock":{"cidr":"0.0.0.0/0","except":["10.0.0.0/8","172.16.0.0/12","192.168.0.0/16","100.64.0.0/10","169.254.169.254/32"]}},{"ipBlock":{"cidr":"2000::/3"}}]}]` | Egress destination rules appended after the always-permitted DNS rule (port 53). Each entry is a Kubernetes NetworkPolicyEgressRule. The default allows the public IPv4 and IPv6 internet while blocking private, CGNAT, metadata, ULA, link-local, and loopback ranges. |
| networkPolicy.enabled | bool | `true` | Create NetworkPolicies (default-deny, allow inbound mTLS + configured egress) |
| pki | object | `{"caDuration":"87600h","caRenewBefore":"720h","server":{"dnsNames":["mtls-socks"],"duration":"8760h","ipAddresses":[],"renewBefore":"720h"}}` | cert-manager PKI. The chart creates a fresh self-signed CA and a CA issuer that signs the server certificate and every per-user client certificate. |
| pki.caDuration | string | `"87600h"` | CA certificate validity |
| pki.caRenewBefore | string | `"720h"` | Renew the CA this long before expiry |
| pki.server.dnsNames | list | `["mtls-socks"]` | DNS SANs for the server certificate (clients verify against these) |
| pki.server.duration | string | `"8760h"` | Server certificate validity |
| pki.server.ipAddresses | list | `[]` | IP SANs for the server certificate. Add the LoadBalancer IP so external clients can verify the server identity. |
| pki.server.renewBefore | string | `"720h"` | Renew the server certificate this long before expiry |
| replicas | int | `1` | Number of proxy replicas |
| service.main.type | string | `"ClusterIP"` | Service type. Set to LoadBalancer (with annotations) to expose the tunnel externally. |
| socks.image.repository | string | `"serjs/go-socks5-proxy"` | SOCKS5 server image (distroless, non-root, statically compiled) |
| socks.image.tag | string | `"v0.0.4"` | SOCKS5 server image tag |
| socks.port | int | `1080` | Loopback port the SOCKS5 server listens on inside the pod |
| users | list | `[]` | Enrolled users. Each entry produces one cert-manager client Certificate and one ghostunnel `--allow-cn` allowlist entry. The client certificate is stored in a Secret named `<release>-user-<name>` (keys tls.crt, tls.key, ca.crt). |

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
