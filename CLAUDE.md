# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Unofficial Helm 3 chart for deploying [Uptime Kuma](https://github.com/louislam/uptime-kuma) on Kubernetes and OpenShift. Also includes a custom non-root OCI container image build (OpenShift-compatible).

## Repository Layout

- `charts/uptime-kuma/` — The Helm chart (Chart.yaml, values.yaml, values.schema.json, templates/)
- `container/Containerfile` — Multi-arch OCI image build (node:20-bookworm-slim base, non-root UID 3310, includes apprise + cloudflared)
- `.github/workflows/` — CI: container builds (buildah, multi-platform), helm-release (chart-releaser), template-sync, GHCR cleanup
- `.github/renovate.json` — Automated dependency updates (container deps, CI deps, Chart.yaml appVersion)

## Chart Architecture

- **StatefulSet** (single replica) is the primary workload — not a Deployment
- **Persistence**: defaults to EmptyDir; enable PVC via `persistence.enabled`
- **Exposure**: supports either OpenShift `Route` OR Kubernetes `Ingress` (mutually exclusive)
- **ServiceMonitor**: optional Prometheus scraping with basic auth (API key from Uptime Kuma settings)
- **Extra Certificates**: optional CA cert injection via ConfigMap
- Container listens on port 3001; Service defaults to port 80
- Image tag defaults to `Chart.appVersion` if `image.tag` is empty
- Values are validated against `values.schema.json` (JSON Schema draft 2020-12)

### Key Template Helpers (`_helpers.tpl`)

- `uptime-kuma.containerImage` — constructs `registry/repository:tag` (defaults registry to docker.io)
- `uptime-kuma.persistentVolumeClaimName` — uses fullname or `persistence.claimNameOverwrite`
- `uptime-kuma.labels` — includes `commonLabels` from values
- `uptime-kuma.selectorLabels` — `app.kubernetes.io/name` + `app.kubernetes.io/instance`

## Common Commands

```bash
# Lint the chart
helm lint charts/uptime-kuma

# Lint with custom values
helm lint charts/uptime-kuma -f my-values.yaml

# Template render (dry-run)
helm template test-release charts/uptime-kuma

# Template with specific values
helm template test-release charts/uptime-kuma --set persistence.enabled=true

# Validate values against schema
helm template test-release charts/uptime-kuma --validate

# Install locally (requires cluster)
helm install test-release charts/uptime-kuma

# Run built-in connection test
helm test test-release
```

## Code Style

- YAML: 2-space indentation, UTF-8, LF line endings, trailing newline (see `.editorconfig`)
- Helm templates use standard `{{ }}` Go template syntax
- Documentation is in AsciiDoc (`.adoc`), not Markdown

## Versioning and Releases

- Chart version (`version` in Chart.yaml) follows SemVer
- App version (`appVersion`) tracks the upstream Uptime Kuma release
- Renovate automatically proposes version bumps for container base images and appVersion
- Helm releases are triggered manually via `workflow_dispatch` on the helm-release workflow
- Container builds trigger on pushes to `container/**` on main
- Git tags follow semantic versioning, e.g. `v1.5.0`
