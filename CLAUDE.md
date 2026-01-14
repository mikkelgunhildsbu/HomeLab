# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Kubernetes HomeLab GitOps repository. ArgoCD automatically syncs and deploys applications from this repo to the cluster.

## Architecture

**GitOps Flow:**
- `bootstrap/applicationset.yaml` defines an ArgoCD ApplicationSet that auto-discovers and deploys all apps in `apps/*`
- Each subdirectory in `apps/` becomes a separate ArgoCD Application with its own namespace (matching the directory name)
- Changes pushed to `main` branch are automatically synced to the cluster (auto-prune, self-heal enabled)

**Ingress Pattern:**
- Envoy Gateway (`apps/envoy/`) provides the central Gateway in the `envoy` namespace
- Applications define HTTPRoute resources pointing to the shared gateway
- External access via Cloudflare Tunnel (`apps/cloudflared/`)
- Internal services use Traefik ingress (e.g., pi-hole, argo)

**App Types:**
1. **Helm-based apps** (pi-hole, envoy): Have `Chart.yaml` with dependencies and `values.yaml` for configuration
2. **Raw manifest apps** (strava-stats, cloudflared, argo): Plain Kubernetes YAML files

## Adding a New Application

1. Create a new directory under `apps/` (directory name = namespace name)
2. Add either:
   - Helm chart: `Chart.yaml` + `values.yaml` + optional `templates/`
   - Raw manifests: Kubernetes YAML files directly in the directory
3. Push to main - ArgoCD will auto-discover and deploy

## Cross-Namespace References

When an HTTPRoute in one namespace needs to reference the Gateway in `envoy` namespace, add a ReferenceGrant (see `apps/envoy/templates/referencegrant.yaml` for example).

## Secrets

Secrets are gitignored (`apps/strava-stats/secret.yaml`). They must be created manually in the cluster or managed separately.
