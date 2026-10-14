# Changelog

All notable changes to the **devops** repository will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
