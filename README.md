# OpenClaw on OKE

Helm chart values and Kubernetes manifests for deploying OpenClaw Operator and one OpenClaw instance on the existing OKE cluster.

## Layout

- `charts/openclaw-operator/`: vendored OpenClaw Operator Helm chart `0.34.4`.
- `charts/openclaw-operator/values-oke.yaml`: OKE/Prometheus-specific chart values.
- `apps/openclaw/`: OpenClaw namespace, OCI FSS PV/PVC, OpenClawInstance, metrics resources, and Tailscale sidecar configuration.
- `docs/auth-profiles.md`: post-deploy provider auth profile copy procedure.

## Deployment Order

1. Install or upgrade the operator with Helm:

   ```bash
   helm upgrade --install openclaw-operator charts/openclaw-operator \
     -n openclaw \
     --create-namespace \
     -f charts/openclaw-operator/values-oke.yaml
   ```

2. Wait for the OpenClaw CRDs and operator deployment.
3. Apply the OpenClaw instance manifests:

   ```bash
   kubectl apply -k apps/openclaw
   ```

4. Copy `auth-profiles.json` into the PVC when provider auth is ready.

The instance intentionally starts without AI provider secrets. Provider credentials are expected to be managed later through the PVC-backed `~/.openclaw/agents/main/agent/auth-profiles.json` file.

The Tailscale sidecar uses a cluster-local Secret named `openclaw-tailscale-auth` in the `openclaw` namespace. This Secret is intentionally not committed to Git.

The Argo CD Applications previously used for this deployment were removed so the OpenClaw operator can update the `OpenClawInstance` image tag without GitOps reverting it.
