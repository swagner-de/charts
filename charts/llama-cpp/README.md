# llama-cpp

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: b11561](https://img.shields.io/badge/AppVersion-b11561-informational?style=flat-square)
Run a llama.cpp server with a cached GGUF model
**Homepage:** <https://github.com/ggml-org/llama.cpp>

## Features

- Official llama.cpp server image with configurable CPU or accelerator variant
- Checksum-verified GGUF downloads into persistent or ephemeral model storage
- Router mode with a configurable cap on simultaneously resident models
- OpenAI-compatible HTTP API with health probes and optional Prometheus metrics
- No external ingress by default

## Install

```bash
helm install llama-cpp oci://ghcr.io/swagner-de/charts/llama-cpp \
  --set models[0].url=https://example.com/model.gguf \
  --set models[0].filename=model.gguf \
  --set models[0].sha256=<sha256>
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.1.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| image.repository | string | `"ghcr.io/ggml-org/llama.cpp"` | llama.cpp image repository |
| image.tag | string | `""` | Image tag; defaults to the chart appVersion |
| image.variant | string | `"server"` | Image variant. The default is the CPU-only server image. |
| modelDownloader.prune | bool | `true` | Remove cached GGUF files that are not present in models |
| modelDownloader.pullPolicy | string | `"IfNotPresent"` | Model downloader image pull policy |
| modelDownloader.repository | string | `"docker.io/curlimages/curl"` | Model downloader image repository |
| modelDownloader.tag | string | `"8.16.0"` | Model downloader image tag renovate: datasource=docker depName=curlimages/curl |
| models | list | `[{"filename":"model.gguf","sha256":"CHANGEME","url":"https://example.invalid/model.gguf"}]` | GGUF models downloaded into the router model directory |
| persistence.models.accessMode | string | `"ReadWriteOnce"` | PVC access mode |
| persistence.models.enabled | bool | `true` | Persist downloaded models across pod restarts |
| persistence.models.mountPath | string | `"/models"` | Model cache mount path |
| persistence.models.size | string | `"5Gi"` | PVC size |
| persistence.models.storageClass | string | `""` | StorageClass; empty uses the cluster default |
| resources.limits.memory | string | `"4Gi"` |  |
| resources.requests.cpu | string | `"500m"` |  |
| resources.requests.memory | string | `"512Mi"` |  |
| server.contextSize | int | `8192` | Model context size |
| server.extraArgs | list | `[]` | Extra llama-server arguments |
| server.gpuLayers | int | `0` | Number of model layers offloaded to a GPU; zero is CPU-only |
| server.metrics | bool | `true` | Enable the Prometheus metrics endpoint |
| server.parallel | int | `1` | Number of parallel request slots |
| server.port | int | `8080` | HTTP listen port |
| server.router.autoload | bool | `true` | Automatically load the model selected by an API request |
| server.router.maxModels | int | `1` | Maximum number of models kept resident simultaneously |
| server.threads | int | `4` | Generation thread count |
| server.threadsBatch | int | `4` | Prompt processing thread count |
| serviceMonitor.enabled | bool | `false` | Create a Prometheus Operator ServiceMonitor |
| serviceMonitor.interval | string | `"1m"` | Metrics scrape interval |
| serviceMonitor.scrapeTimeout | string | `"10s"` | Metrics scrape timeout |

## Security

- Runs as non-root UID/GID 65534
- Read-only root filesystems with writable model and temporary volumes
- Service account token automount disabled
- Privilege escalation disabled and all capabilities dropped
- Seccomp profile: `RuntimeDefault`

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
