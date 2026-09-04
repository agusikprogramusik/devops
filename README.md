# DevOps — Common Helm Library Charts

This repository provides reusable, enterprise-grade Helm Library Charts (`type: library`) designed for cloud-native microservices.

## Available Charts

- **`common-library`** (`charts/common-library`): Standardized Kubernetes templates (`Deployment`, `Service`, `Ingress`, `HTTPRoute`, `HPA`, `ServiceAccount`) with hardened security contexts, health probes, and standard labeling conventions.

## How to use in application charts

1. In your application's `Chart.yaml`, add the dependency:

```yaml
dependencies:
  - name: common-library
    version: "1.0.0"
    repository: "oci://ghcr.io/agusikprogramusik/helm-charts"
```

2. Replace boilerplate templates in your chart with 1-line includes:

- `templates/deployment.yaml`:
  ```yaml
  {{- include "common.deployment" . -}}
  ```
- `templates/service.yaml`:
  ```yaml
  {{- include "common.service" . -}}
  ```
- `templates/serviceaccount.yaml`:
  ```yaml
  {{- include "common.serviceaccount" . -}}
  ```
- `templates/ingress.yaml`:
  ```yaml
  {{- include "common.ingress" . -}}
  ```
- `templates/hpa.yaml`:
  ```yaml
  {{- include "common.hpa" . -}}
  ```

## Publishing

Whenever changes are pushed to `main` under `charts/common-library/**`, the GitHub Actions workflow validates and publishes the chart package to GHCR (`oci://ghcr.io/agusikprogramusik/helm-charts`).
