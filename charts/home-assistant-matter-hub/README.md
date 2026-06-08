# skel

A skeleton chart template for creating new charts in this repository.

## Usage

Copy this chart as a starting point for a new application:

```bash
cp -r charts/skel charts/my-new-chart
```

Then update:

1. **`Chart.yaml`** — Set chart name, description, version, appVersion, and home URL.
2. **`templates/hardcoded.yaml`** — Replace the alpine container with your application's image and configuration.
3. **`values.yaml`** — Define user-configurable defaults.
4. **`README.md`** — Document configuration options and security settings.

## What's Included

The skeleton provides a working chart with:

- Security-hardened defaults (non-root, read-only filesystem, all capabilities dropped)
- Example init container
- ConfigMap and Secret from values
- Persistence volume mount
- Service definition
- Probe placeholders (disabled by default)

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for the full guide on adding new charts.
