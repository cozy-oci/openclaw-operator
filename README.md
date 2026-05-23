# OpenClaw Operator on OKE

GitOps manifests for deploying OpenClaw Operator and one OpenClaw instance on the existing OKE cluster.

## Layout

- `charts/openclaw-operator/`: vendored OpenClaw Operator Helm chart `0.30.0`.
- `charts/openclaw-operator/values-oke.yaml`: OKE/Prometheus-specific chart values.
- `apps/openclaw/`: OpenClaw namespace, OCI FSS PV/PVC, OpenClawInstance, and Tailscale-only ingress.
- `argocd/`: Argo CD Application registrations.
- `docs/auth-profiles.md`: post-deploy provider auth profile copy procedure.

## Deployment Order

1. Apply `argocd/openclaw-operator.yaml`.
2. Wait for the OpenClaw CRDs and operator deployment.
3. Apply `argocd/openclaw.yaml`.
4. Copy `auth-profiles.json` into the PVC when provider auth is ready.

The instance intentionally starts without AI provider secrets. Provider credentials are expected to be managed later through the PVC-backed `~/.openclaw/agents/main/agent/auth-profiles.json` file.

The Tailscale sidecar uses a cluster-local Secret named `openclaw-tailscale-auth` in the `openclaw` namespace. This Secret is intentionally not committed to Git.
