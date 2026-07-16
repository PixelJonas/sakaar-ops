# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## What This Repo Is

A GitOps-managed OpenShift SNO cluster (Sakaar, a Hetzner-hosted
bare-metal libvirt VM — see the `infra-ops` repo for the host itself). All
cluster state is declared here and reconciled by ArgoCD using the
app-of-apps pattern, mirroring `home-ops`'s structure. Secrets are sourced
from Doppler (the `infra-ops` project's `prd` config — reused rather than
a dedicated project, since this cluster's secrets already live there) via
External Secrets Operator, and never committed to git.

## Architecture

**Deployment flow:** two one-time prerequisites applied from `infra-ops`
(OpenShift GitOps operator + External Secrets Operator subscriptions, the
`eso-token-cluster` bootstrap secret — see README.md) → `bootstrap/initial/`
installs/configures the ArgoCD instance → ArgoCD reads `bootstrap/base/` →
an ApplicationSet discovers and deploys everything under
`components-infra/`.

**Sync waves** control ordering via the `argocd.argoproj.io/sync-wave`
annotation: wave 1 = ESO namespace, wave 2 = ESO's Doppler ClusterSecretStore
+ cert-manager-operator, wave 3 = certificates (depends on cert-manager's
CRDs existing).

**Secrets flow:** `infra-ops` Doppler project → ESO `ClusterSecretStore`
(`doppler-cluster`) → `ExternalSecret` CRs per component → native
Kubernetes `Secret` objects. Never store secret values in this repo.

## Validation

```bash
./scripts/validate_manifests.sh
yamllint .
```

## Conventions

- **Kustomize-first:** prefer `kustomization.yaml` overlays over raw YAML
  duplication, matching `home-ops`.
- **Sync waves:** new infra components should set an appropriate
  `sync-wave` annotation reflecting real dependencies (CRDs, operators)
  on earlier waves.
- **YAML style:** enforced by `.yamllint` — no `document-start` (`---`),
  line length warnings ignored, truthy values (`yes`/`no`) are allowed.
- **Secret scanning:** never commit keys, tokens, or kubeconfigs — see
  `.gitignore`.
