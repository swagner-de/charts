# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

A collection of Helm charts for self-hosted applications, all built on the [bjw-s common library chart](https://bjw-s-labs.github.io/helm-charts/) (v5.x). Charts are published as OCI artifacts to `ghcr.io/swagner-de/charts/`.

## Commands

### Lint a chart

```bash
ct lint --config .github/ct.yaml
```

### Render templates locally

```bash
helm dependency build charts/<name>
helm template test charts/<name>
helm template test charts/<name> -f custom-values.yaml
```

### Install test on KinD

```bash
ct install --config .github/ct.yaml --charts charts/<name>
```

Charts that use CNPG require the operator pre-installed on the cluster (CI does this automatically).

### Regenerate docs (happens automatically on push to main)

```bash
helm-docs --chart-search-root charts --log-level warning
```

## Architecture

### The hardcoded.yaml pattern

Every chart has exactly one template file: `templates/hardcoded.yaml`. This is the central design pattern:

1. A named template `{{- define "hardcoded" -}}` declares the application's deployment config (image, probes, env, volumes, security context, services, etc.)
2. At the bottom of the file, user values are deep-copied and merged with hardcoded defaults:
   ```
   {{- $ctx := deepCopy . -}}
   {{- $_ := include "hardcoded" . | fromYaml | mergeOverwrite $ctx.Values -}}
   {{ include "bjw-s.common.loader.all" $ctx }}
   ```
3. `bjw-s.common.loader.all` renders all Kubernetes resources (Deployments, Services, Ingress/HTTPRoute, ConfigMaps, Secrets, PVCs, etc.)

The merge order means hardcoded values **override** user values. Place user-configurable options in `values.yaml`; put non-negotiable application plumbing in `hardcoded.yaml`.

### Chart file structure

```
charts/<name>/
├── Chart.yaml              # metadata, dependencies, renovate image comment
├── Chart.lock              # locked dependency versions (committed)
├── templates/
│   └── hardcoded.yaml      # the only template file
├── values.yaml             # user-overridable defaults
├── ci/
│   └── test-values.yaml    # values used by ct install (optional)
├── README.md               # auto-generated, do not edit manually
├── README.md.gotmpl        # helm-docs template
├── .helmignore
└── .trivyignore.yaml       # chart-specific security exceptions (optional)
```

### Database support pattern

Charts that need a database typically offer two mutually exclusive options:
- `postgres.enabled: true` — uses a packaged PostgreSQL subchart (`oci://registry-1.docker.io/cloudpirates/postgres`)
- `cnpg.enabled: true` — uses the CloudNativePG operator (cluster defined via `rawResources`)

A fail guard in hardcoded.yaml prevents both from being enabled simultaneously.

### Renovate integration

Image versions are tracked via a comment in `Chart.yaml`:
```yaml
# renovate: image=ghcr.io/org/image
appVersion: 'v1.2.3'
```

Additional images in `hardcoded.yaml` are tracked via:
```yaml
repository: ghcr.io/org/image
tag: v1.2.3
```

Renovate auto-bumps `appVersion` and the chart `version` (patch) on image updates.

## Creating a New Chart

1. `cp -r charts/skel charts/<name>`
2. Edit `Chart.yaml`: set name, description, version, appVersion, add `# renovate: image=...` comment
3. Edit `templates/hardcoded.yaml`: configure containers, probes, volumes, services
4. Edit `values.yaml`: expose user-configurable options
5. Add `README.md.gotmpl` for helm-docs generation
6. If the chart needs a database, add the postgres subchart dependency to `Chart.yaml` and the CNPG pattern
7. Add `ci/test-values.yaml` if integration testing requires specific values (e.g., CNPG enabled)

## Conventions

- **Security-by-default**: all containers must set `runAsNonRoot: true`, `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, `seccompProfile.type: RuntimeDefault`, and `capabilities.drop: [ALL]`. Only add capabilities when strictly required.
- **No real secrets in values.yaml**: use `"CHANGEME"` or `None` as placeholders.
- **README.md is auto-generated**: never edit it directly; edit `README.md.gotmpl` instead.
- **Chart.lock is committed**: run `helm dependency update` when changing dependencies.
- **Semantic versioning**: chart `version` for the chart itself, `appVersion` for the upstream application.
- **Commit style**: semantic commits — `fix(chartname):`, `feat(chartname):`, `chore(deps):`, `docs:`.

## CI Pipeline (GitHub Actions)

On **pull request**:
- `lint-test.yaml` — ct lint + ct install on KinD (with CNPG operator)
- `helm-diff.yaml` — posts rendered template diff as PR comment
- `security-scan.yaml` — Trivy config scan, fails on HIGH/CRITICAL unless ignored

On **push to main**:
- `helm-docs.yaml` — regenerates READMEs and root chart table
- `oci-release.yaml` — publishes charts with bumped versions to `ghcr.io/swagner-de/charts/`

Weekly:
- `image-cve-scan.yaml` — Grype CVE scan of all container images with EPSS/KEV risk scoring
