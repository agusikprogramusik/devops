# DevOps — Common Helm Library Charts (`devops`)

This repository provides reusable, enterprise-grade Helm Library Charts (`type: library`) designed to eliminate boilerplate, enforce organizational security standards, and simplify Kubernetes resource declarations across all microservices.

---

## Architectural Concept

In traditional Helm setups, each microservice copies hundreds of lines of YAML boilerplate (`deployment.yaml`, `service.yaml`, `hpa.yaml`, etc.). 

With the **Helm Library Chart pattern**, the entire Kubernetes specification is encapsulated in reusable Go template definitions within `charts/common-library`. Downstream application charts (such as `order-service-chart` or `payment-service-chart`) simply declare a dependency on `common-library` and invoke templates with a single line:

```yaml
{{- include "common.deployment" . -}}
```

---

## Available Templates in `common-library`

| Template Name | Source File | Description |
| :--- | :--- | :--- |
| `common.deployment` | `templates/_deployment.tpl` | Generates a complete Kubernetes `Deployment` with pod security context, container security context, configurable health probes (`livenessProbe`, `readinessProbe`), environment variable injections, volume mounts, and affinity rules. |
| `common.service` | `templates/_service.tpl` | Creates a Kubernetes `Service` (ClusterIP, NodePort, or LoadBalancer) pointing to the pod container port. |
| `common.ingress` | `templates/_ingress.tpl` | Deploys a standard Kubernetes `Ingress` with support for `ingressClassName`, custom annotations (e.g. cert-manager), multi-host routing, and TLS termination. |
| `common.httproute` | `templates/_httproute.tpl` | Deploys a Gateway API `HTTPRoute` resource with parentRefs and rule matching. |
| `common.hpa` | `templates/_hpa.tpl` | Provisions a `HorizontalPodAutoscaler` (autoscaling/v2) with configurable CPU and memory target utilization percentages. |
| `common.serviceaccount` | `templates/_serviceaccount.tpl` | Generates a dedicated `ServiceAccount` with configurable annotations and `automountServiceAccountToken` toggles. |
| `common.names.*` | `templates/_helpers.tpl` | Standard Helm helper functions computing chart name, full name, standard labels (`app.kubernetes.io/name`, `instance`, `version`, `managed-by`), and selector labels. |

---

## Standardized Security Posture

The library chart enforces strict security defaults on all workloads:

- **Non-Root Execution**: `runAsNonRoot: true` and `runAsUser: 1001`.
- **Privilege Escalation Blocked**: `allowPrivilegeEscalation: false`.
- **Capability Dropping**: Drops all default Linux capabilities (`capabilities: { drop: ["ALL"] }`).
- **Health Verification**: Enforces both `livenessProbe` and `readinessProbe` to guarantee zero-downtime rolling updates.

---

## How to Consume in Application Charts

### 1. Declare Dependency in `Chart.yaml`
```yaml
apiVersion: v2
name: my-microservice
type: application
version: 0.1.0
dependencies:
  - name: common-library
    version: "1.0.0"
    repository: "oci://ghcr.io/agusikprogramusik/helm-charts"
```

### 2. Update Helm Dependencies
```bash
helm dependency update
```

### 3. Replace Manifest Files with One-Liners
In your application chart's `templates/` folder:
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

---

## CI/CD & Automated Publishing

Chart changes are validated and published automatically using GitHub Actions (`.github/workflows/release.yaml`):

```mermaid
flowchart LR
    Push["Push to main\n(charts/common-library/**)"]
    Lint["helm lint\ncharts/common-library"]
    Login["docker/login-action\n(Login to ghcr.io)"]
    Package["helm package\ncharts/common-library"]
    Publish["helm push <package>.tgz\noci://ghcr.io/<owner>/helm-charts"]

    Push --> Lint --> Login --> Package --> Publish
```

Whenever changes to `charts/common-library/**` are merged to `main`:
1. The chart is linted with `helm lint`.
2. The runner logs in to GitHub Container Registry (`ghcr.io`).
3. The chart package is created and pushed as an OCI artifact to `oci://ghcr.io/agusikprogramusik/helm-charts`.

---

## Application Chart: `mcp-knowledge-server`

Location: [`charts/mcp-knowledge-server`](charts/mcp-knowledge-server)

Deploys the central **Model Context Protocol (MCP)** server alongside a dedicated **Hugging Face Text Embeddings Inference (TEI)** microservice and PostgreSQL 18 with `pgvector` in namespace `ai-platform`.

```mermaid
flowchart LR
    subgraph K8s["Kubernetes (ai-platform namespace)"]
        subgraph Serving["MCP Knowledge Server"]
            MCP["mcp-knowledge-server (2 replicas)\nImage: ghcr.io/agusikprogramusik/mcp-knowledge-server:0.5.0\nMemory: 256Mi | Port: 8000\nEndpoints: /mcp (Streamable), /sse"]
        end
        subgraph Inference["Inference Subsystem (Method B)"]
            TEI["mcp-knowledge-server-tei (1 replica)\nImage: ghcr.io/huggingface/text-embeddings-inference:cpu-latest\nModel: BAAI/bge-small-en-v1.5 | Memory Limit: 2Gi\nMax Batch Tokens: 2048 | Max Client Batch: 16"]
        end
        subgraph Storage["Vector & Full-Text Store"]
            PG[("PostgreSQL 18 + pgvector\nHNSW (<=>) + GIN FTS (RRF)")]
        end
        MCP -->|"HTTP POST /embed (port 80)"| TEI
        MCP <-->|"Hybrid Search RRF"| PG
    end
```

### Key Values Configuration:
* `image.tag: "0.5.0"`
* `tei.enabled: true`
* `tei.modelId: "BAAI/bge-small-en-v1.5"`
* `tei.maxBatchTokens: 2048` (optimized for English-optimized model)
* `tei.maxClientBatchSize: 16`
* `tei.resources.limits.memory: 2Gi` (request `1Gi`)
* `postgres.enabled: true` (PostgreSQL 18 + pgvector 0.8.7)



## Re-indexing Documentation

### Repository Re-indexing Script

Located at [`mcp-knowledge-server/reindex_repos.py`](../mcp-knowledge-server/reindex_repos.py)

The script re-indexes all documentation files using the configured embedding model (`BAAI/bge-small-en-v1.5`):

```bash
# Re-index all repositories in workspace
cd mcp-knowledge-server
python reindex_repos.py

# Wipe store first and perform a clean re-index
python reindex_repos.py --clean

# Re-index specific services in custom workspace
python reindex_repos.py --workspace /path/to/workspace --services devops photoblog
```

**Features:**
- Automatically scans workspace for service `docs/` and `README.md` files
- Fully configurable via `.env` or environment variables (`EMBEDDING_API_URL`, `POSTGRES_*`, `WORKSPACE_ROOT`)
- Batched inference via Hugging Face Text Embeddings Inference (TEI)
- Automatic PostgreSQL pgvector dimension migration detection (384-dim)
- Supports clean store wipe (`--clean`) and hash bypass (`--force`)

