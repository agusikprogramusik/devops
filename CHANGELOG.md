# Changelog

All notable changes to the **devops** repository will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] - 2026-10-14

### Added
- **PostgreSQL 18 + pgvector Subsystem**: Added dedicated PostgreSQL 18 deployment, ClusterIP service, and persistent volume claim using official bleeding-edge `pgvector/pgvector:pg18` image.
- **Enterprise Vector Storage**: Switched `mcp-knowledge-server` default storage backend to `postgres` with native `vector(1024)` column types and HNSW cosine distance indexing (`m=16, ef_construction=64`).
- Bumped `mcp-knowledge-server` chart version to `0.3.0`.

---

## [0.3.4] - 2026-10-14

### Changed
- **TEI Attention Buffer Calibration**: Set `--max-batch-tokens 1024` and `--max-client-batch-size 8` for `BAAI/bge-m3` in Hugging Face TEI. This maintains the 6Gi RAM ceiling while reducing the quadratic attention score tensor allocation from ~1.07GB per layer down to ~67MB per layer, preventing ONNX runtime warmup OOM on CPU.
- Bumped `mcp-knowledge-server` chart version to `0.2.4`.

---

## [0.3.3] - 2026-10-14

### Changed
- **TEI Memory & Batch Optimization**: Increased TEI memory limit to `6Gi` (request `3Gi`) and configured `--max-batch-tokens 4096` / `--max-client-batch-size 16` to drastically reduce attention buffer overhead during `BAAI/bge-m3` model warm-up.
- Bumped `mcp-knowledge-server` chart version to `0.2.3`.

---

## [0.3.2] - 2026-10-14

### Changed
- **TEI Memory Allocation**: Increased memory limit to `5Gi` (request `2.5Gi`) for Hugging Face TEI pod to accommodate ONNX layer runtime buffers during `BAAI/bge-m3` model warm-up and prevent cgroup OOMKilled errors.
- Bumped `mcp-knowledge-server` chart version to `0.2.2`.

---

## [0.3.1] - 2026-10-14

### Fixed
- **TEI Startup Probe**: Added dedicated Kubernetes `startupProbe` with 350-second tolerance window to allow downloading large Hugging Face model weights (`BAAI/bge-m3`) without prematurely triggering liveness probe termination (Exit Code 137).
- Bumped `mcp-knowledge-server` chart version to `0.2.1`.

---

## [0.3.0] - 2026-10-14

### Added
- **Hugging Face Text Embeddings Inference (TEI) Service (Method B)**:
  - Added dedicated TEI Kubernetes Deployment and Service templates in `charts/mcp-knowledge-server` (`tei-deployment.yaml`, `tei-service.yaml`).
  - Added `tei` configuration block in `values.yaml` targeting `ghcr.io/huggingface/text-embeddings-inference:cpu-latest` with `BAAI/bge-m3` model.
  - Injected `EMBEDDING_API_URL: http://mcp-knowledge-server-tei:80` into MCP knowledge server container environment.
  - Bumped `mcp-knowledge-server` chart version and image tag to `0.2.0`.

---

## [0.2.3] - 2026-10-14

### Changed
- **MCP Knowledge Server Chart**: Bumped chart version to `0.1.4` and image tag to `0.1.4` (Python 3.14.8 runtime).

---

## [0.2.2] - 2026-10-14

### Changed
- **MCP Knowledge Server Chart**: Bumped chart and image tag to `0.1.3`.

---

## [0.2.1] - 2026-10-14

### Changed
- **MCP Knowledge Server Chart**: Bumped chart and image tag to `0.1.2`, configured `ghcr-secret` in `imagePullSecrets`, and set `optional: true` on PostgreSQL secret references for graceful offline fallback.

---

## [0.2.0] - 2026-10-14

### Changed
- **MCP Knowledge Server Helm Chart**: Bumped `mcp-knowledge-server` Helm chart version and `appVersion` to `0.1.1`, matching GHCR container image tag `0.1.1`.

---

## [0.1.0] - 2026-09-06

### Added
- **Common Library Helm Chart (`charts/common-library`)**: Reusable Go template abstractions for Deployment, Service, Ingress, ServiceAccount, and Probes.
- **MCP Knowledge Server Helm Chart (`charts/mcp-knowledge-server`)**: Initial packaging for Central Model Context Protocol (MCP) server.
- Documentation standards in `docs/architecture` and `docs/standards`.
