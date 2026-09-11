---
service_name: "devops"
title: "Helm Chart Consumption & Dependency Guidelines"
category: "architecture"
scope: "macro"
owner_team: "devops-platform"
tags:
  - "helm"
  - "standards"
  - "packaging"
  - "oci"
dependencies:
  - "ghcr.io"
last_reviewed: "2026-09-06"
---

# Helm Chart Consumption & Packaging Guidelines

## 1. How Microservices Consume `common-library`

Microservice charts must not maintain raw Kubernetes manifests. Instead, they declare a dependency on `common-library`:

### Step 1: Declare Dependency in `Chart.yaml`
```yaml
apiVersion: v2
name: my-microservice
version: 1.0.0
type: application
dependencies:
  - name: common-library
    version: "1.0.0"
    repository: "oci://ghcr.io/agusikprogramusik/helm-charts"
```

### Step 2: One-Line Template Manifests
In the microservice chart's `templates/` directory, files contain only single-line includes:

```yaml
# templates/deployment.yaml
{{- include "common.deployment" . -}}

# templates/service.yaml
{{- include "common.service" . -}}

# templates/ingress.yaml
{{- include "common.ingress" . -}}
```

## 2. OCI Registry Release Cycle
Library charts are versioned using SemVer and published to GitHub Packages (GHCR) as OCI artifacts:
1. `git push` to `main` with changes in `charts/common-library/**`.
2. GitHub Actions runs `helm lint charts/common-library`.
3. Packages chart into `.tgz`: `helm package charts/common-library`.
4. Pushes OCI artifact: `helm push common-library-1.0.0.tgz oci://ghcr.io/<owner>/helm-charts`.
