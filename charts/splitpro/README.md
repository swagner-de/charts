# splitpro

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v2.1.5](https://img.shields.io/badge/AppVersion-v2.1.5-informational?style=flat-square)
SplitPro — self-hosted open source alternative to Splitwise for sharing expenses

## Features
- Single hardened pod: the SplitPro app (Next.js) plus a bundled PostgreSQL
  container, both fully non-root with read-only root filesystems
- PostgreSQL image ships the `pg_cron` extension, required by SplitPro's
  recurring-transactions and cache-cleanup database migration
- Receipt uploads and database data on separate PVCs
- Secrets (`NEXTAUTH_SECRET`, `DATABASE_URL`, OIDC/OAuth credentials) injected
  via `envFromSecrets`; the DB password via an external Secret (or, for
  CI/standalone, a chart-managed one)

## Why a bundled database
SplitPro's schema hard-depends on `pg_cron`: the recurrence migration runs
`CREATE EXTENSION pg_cron` and adds a foreign key onto `cron.job`, and the app
runs `prisma migrate deploy` on every boot — so a database without `pg_cron`
fails the migration and the app never starts. The generic PostgreSQL and
CloudNativePG operand images do **not** include `pg_cron`, so this chart bundles
the upstream-maintained `ossapps/postgres` image (which does) as a plain in-pod
database rather than delegating to an external operator.

The database container is started with:

```
postgres -c shared_preload_libraries=pg_cron \
         -c cron.database_name=<postgres.database> \
         -c cron.timezone=UTC
```

## Database password
The `postgres` container reads `POSTGRES_PASSWORD` from a Secret. In GitOps,
create that Secret externally (e.g. via an operator) and point
`postgres.passwordSecret.name` at it, and supply the app's `DATABASE_URL` from
the same source through `envFromSecrets`. For CI/standalone use, set
`postgres.createSecret: true` and `postgres.password` to have the chart emit the
Secret itself.

## Authentication
SplitPro uses NextAuth and requires at least one provider (it has no
username/password login). Provide the provider's credentials through
`envFromSecrets` using the variable names from SplitPro's own configuration
docs (it supports email, Google, and generic OIDC providers). Set `siteUrl` to
the external URL; the OAuth callback is `<siteUrl>/api/auth/callback/<provider>`.
Set `config.DISABLE_EMAIL_SIGNUP: "true"` and `config.OAUTH_AUTO_REDIRECT:
"true"` to make an external provider the sole login path.

## Routing
The app is the single entrypoint on port `3000`. Enable `route.main` (Gateway
API `HTTPRoute`) or an external Ingress and point it at `<release>-main:3000`.

## Install

```bash
helm install splitpro oci://ghcr.io/swagner-de/charts/splitpro
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| config.CACHE_RETENTION_INTERVAL | string | `"2 days"` |  |
| config.CLEAR_CACHE_CRON_RULE | string | `"0 2 * * 0"` |  |
| config.CURRENCY_RATE_PROVIDER | string | `"frankfurter"` |  |
| config.DEFAULT_HOMEPAGE | string | `"/balances"` |  |
| config.DISABLE_EMAIL_SIGNUP | string | `"true"` |  |
| config.OAUTH_AUTO_REDIRECT | string | `"true"` |  |
| config.UPLOAD_MAX_FILE_SIZE_MB | string | `"10"` |  |
| envFromSecrets | list | `[]` |  |
| ingress.main.enabled | bool | `false` |  |
| networkPolicy.enabled | bool | `false` |  |
| networkPolicy.gatewayNamespace | string | `"envoy-gateway-system"` |  |
| nextauthSecret | string | `"CHANGEME"` |  |
| persistence.pgdata.accessMode | string | `"ReadWriteOnce"` |  |
| persistence.pgdata.enabled | bool | `true` |  |
| persistence.pgdata.size | string | `"10Gi"` |  |
| persistence.uploads.accessMode | string | `"ReadWriteOnce"` |  |
| persistence.uploads.enabled | bool | `true` |  |
| persistence.uploads.size | string | `"5Gi"` |  |
| postgres.createSecret | bool | `false` |  |
| postgres.database | string | `"splitpro"` |  |
| postgres.image.repository | string | `"ossapps/postgres"` |  |
| postgres.image.tag | string | `"17.7-trixie"` |  |
| postgres.password | string | `"CHANGEME"` |  |
| postgres.passwordSecret.name | string | `""` |  |
| postgres.user | string | `"splitpro"` |  |
| route.main.enabled | bool | `false` |  |
| siteUrl | string | `"CHANGEME"` | Public URL of the instance, used by NextAuth for callbacks and absolute URLs.    MUST match the browser URL. Example: https://splitpro.example.com |

## Security
Both containers run `runAsNonRoot: true`, `readOnlyRootFilesystem: true`,
`allowPrivilegeEscalation: false`, all capabilities dropped, and seccomp
`RuntimeDefault`. The app runs as UID 1000 (the image's `node` user, which owns
`/app/uploads`); PostgreSQL runs as UID 999. Writable paths (`/tmp`, the uploads
and data PVCs, and `/var/run/postgresql`) are backed by emptyDir/PVC volumes.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
