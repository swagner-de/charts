{​{ template "chart.header" . }}
{​{ template "chart.badgesSection" . }}
{​{ template "chart.description" . }}
{​{ template "chart.homepageLine" . }}

[Tuwunel](https://tuwunel.chat/) is a Matrix homeserver written in Rust, the
successor to conduwuit. This chart runs a single hardened StatefulSet with a
PVC for the RocksDB database and media, configured entirely through a mounted
TOML file.

## Configuration

`config.server_name` is the Matrix server name (the suffix in
`@user:<server_name>`). It is written to the database on the **first boot and
cannot be changed afterwards** without wiping the database — set it before the
first install.

The base config is rendered from `values.yaml` into a ConfigMap and mounted at
`/config/tuwunel.toml`. Anything not exposed as a value can be added verbatim
under `[global]` via `config.extraConfig`.

## Routing and .well-known

tuwunel listens on `config.port` (default `8008`). Front it with a reverse
proxy or a Gateway API `HTTPRoute` and forward at least:

```
/_matrix/*
/_tuwunel/*
/.well-known/*
```

When `config.server_name` differs from the host tuwunel is reached on, enable
`wellKnown` so tuwunel serves `/.well-known/matrix/client`,
`/.well-known/matrix/server` and `/.well-known/openid-configuration`. The
reverse proxy must forward `/.well-known/*` to tuwunel (modern clients and the
built-in OIDC server need those endpoints served dynamically, not as static
files).

## Migrations

The first boot after an upgrade may run a one-time database migration, during
which the listener stays closed and `tuwunel --health-check` reports unhealthy.
The chart gives the pod a long **startup** probe
(`migrationProbe.failureThreshold` × 10s) so the liveness and readiness probes
stay suspended until it finishes, and raises `terminationGracePeriodSeconds` so
a migration is never killed part way through. Size both past your largest
expected migration.

## OIDC / SSO

tuwunel can delegate login to an external OpenID Connect provider via a
`[[global.identity_provider]]` block. Because that block carries a
`client_secret`, this chart does not template it from values. Instead, enable
`oidc` and provide a Secret whose key (`oidc.secretKey`, default `oidc.toml`)
holds the complete TOML fragment; it is mounted at `/config/oidc.toml` and
layered on top of the base config. Enabling the built-in OIDC server for
next-generation clients (Element X) also requires `wellKnown.client` to be set.

Example fragment to place in the Secret:

```toml
[[global.identity_provider]]
brand = "my-idp"
client_id = "..."
client_secret = "..."
issuer_url = "https://idp.example.com"
callback_url = "https://matrix.example.com/_matrix/client/unstable/login/sso/callback/my-idp"
scope = ["openid", "profile", "email"]
default = true
discovery = true
trusted = true
registration = true
```

See the [tuwunel configuration docs](https://tuwunel.chat/configuration.html)
for the full `[[global.identity_provider]]` option set.

## Install

```bash
helm install {​{ template "chart.name" . }} oci://ghcr.io/swagner-de/charts/{​{ template "chart.name" . }}
```

{​{ template "chart.requirementsSection" . }}

{​{ template "chart.valuesSection" . }}

## Security

The container runs fully hardened: `runAsNonRoot: true`,
`readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, all
capabilities dropped and seccomp `RuntimeDefault`. The database lives on a PVC
mounted at `/var/lib/tuwunel` and `/tmp` is a writable emptyDir.

{​{ template "chart.maintainersSection" . }}
