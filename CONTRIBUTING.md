# Contributing

Contributions are welcome! This guide covers how to add new charts, test changes, and submit pull requests.

## Creating a New Chart

1. Copy the skeleton chart:
   ```bash
   cp -r charts/skel charts/my-new-chart
   ```

2. Update `charts/my-new-chart/Chart.yaml` with the chart name, description, version, and appVersion.

3. Edit `charts/my-new-chart/templates/hardcoded.yaml` with your application's deployment configuration.

4. Update `charts/my-new-chart/values.yaml` with user-configurable defaults.

5. Add a `charts/my-new-chart/README.md` documenting configuration options.

## Chart Architecture

All charts follow the same pattern:

- **`Chart.yaml`** - Chart metadata, version, and dependencies.
- **`values.yaml`** - User-overridable defaults. Do not put secrets or real passwords here.
- **`templates/hardcoded.yaml`** - Defines application defaults that merge with values.yaml, then delegates rendering to the bjw-s common library chart.

The `hardcoded.yaml` pattern works by:
1. Defining a named template `hardcoded` with YAML configuration
2. Deep-copying the Helm context
3. Merging hardcoded values over user values
4. Calling `bjw-s.common.loader.all` to render all Kubernetes resources

## Testing

### Prerequisites

- [Helm](https://helm.sh/) v3.x
- [chart-testing (ct)](https://github.com/helm/chart-testing) for linting
- [KinD](https://kind.sigs.k8s.io/) for integration tests

### Lint

```bash
ct lint --config .github/ct.yaml
```

### Template Rendering

Verify your chart renders correctly:

```bash
helm dependency build charts/my-new-chart
helm template test charts/my-new-chart -f my-values.yaml
```

### Install Test

```bash
ct install --config .github/ct.yaml --charts charts/my-new-chart
```

## Pull Request Process

1. Ensure your chart lints cleanly with `ct lint`.
2. If adding a new chart, include a `README.md`.
3. Do not include real credentials or secrets in `values.yaml`.
4. The CI pipeline will automatically:
   - Lint your chart
   - Show a template diff on the PR
   - Run install tests in a KinD cluster (if applicable)
5. On merge, changed charts are automatically published to `ghcr.io/swagner-de/charts/`.

## Dependency Updates

Image version updates are handled automatically by [Renovate](https://docs.renovatebot.com/). Renovate uses comments in `Chart.yaml` (e.g., `# renovate: image=docker.io/org/image`) to track upstream image tags and bumps chart versions accordingly.
