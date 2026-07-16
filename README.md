# sakaar-ops

GitOps-managed configuration for the Sakaar OpenShift SNO cluster
(`sakaar.janz.cloud`), reconciled by ArgoCD using the app-of-apps pattern —
mirrors [home-ops](https://github.com/PixelJonas/home-ops)'s structure and
conventions, scoped down to a single node / single cluster.

## Bootstrap prerequisites (one-time, done from infra-ops)

Two things must exist on the cluster **before** `./bootstrap.sh sakaar` is
run for the first time — ArgoCD can't deploy the operator that lets ArgoCD
run, and External Secrets Operator can't fetch its own credential from a
secret store it doesn't have access to yet:

1. The OpenShift GitOps operator and the External Secrets Operator OLM
   subscriptions, applied via `mise run sakaar:gitops-apply` from the
   [infra-ops](https://github.com/PixelJonas/infra-ops) repo (private —
   these manifests live there, not here, since they're one-time host/cluster
   bootstrap, not ongoing GitOps state).
2. The `eso-token-cluster` Secret (namespace `external-secrets-operator`,
   key `dopplerToken`) — also created by that same mise task, from the
   `SAKAAR_ESO_DOPPLER_TOKEN` Doppler secret. This secret is intentionally
   never git-tracked (see `components-infra/external-secrets-operator/base/kustomization.yaml`
   and `components-infra/external-secrets-store-doppler/base/`) — it's the
   credential that lets ESO reach every other secret, so it can't itself
   come from ESO.

Once both exist:

```bash
export KUBECONFIG=/path/to/sakaar/kubeconfig
oc project openshift-gitops
./bootstrap.sh sakaar
```

## Repository Structure

```
bootstrap/
  initial/          # One-time ArgoCD installation manifests
  base/             # App-of-apps ApplicationSet + cluster-config AppProject
  overlays/sakaar/  # This cluster's application list
components-infra/   # ESO, cert-manager-operator, certificates
components-apps/    # Reserved for future application workloads (currently empty)
scripts/            # validate_manifests.sh
```

## Validation

```bash
./scripts/validate_manifests.sh
yamllint .
```

CI runs both on every push/PR via `.github/workflows/validate-manifests.yaml`.
