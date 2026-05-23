# Managing AI Provider Auth Profiles

This deployment intentionally does not store AI provider credentials in Kubernetes Secrets or Git.

OpenClaw reads provider auth from the PVC-backed path:

```text
/home/openclaw/.openclaw/agents/main/agent/auth-profiles.json
```

The runtime format must be the canonical `version` + `profiles` shape:

```json
{
  "version": 1,
  "profiles": {
    "openai-codex:your-email@example.com": {
      "type": "oauth",
      "provider": "openai-codex",
      "access": "...",
      "refresh": "...",
      "expires": 1778027426540,
      "email": "your-email@example.com",
      "accountId": "..."
    }
  }
}
```

Do not paste the legacy flat shape directly, for example `{ "openai-codex:...": { ... } }`; OpenClaw 2026.5.2 does not treat that as a runtime auth store.

To copy an existing profile from another OpenClaw instance:

```bash
kubectl -n openclaw exec -i openclaw-0 -c auth-profile-maintainer -- sh -c \
  'mkdir -p /home/openclaw/.openclaw/agents/main/agent && cat > /home/openclaw/.openclaw/agents/main/agent/auth-profiles.json'
```

Paste the full JSON, then press `Ctrl-D`.

Restart the OpenClaw pod after replacing the file so the gateway loads the new auth snapshot:

```bash
kubectl -n openclaw delete pod openclaw-0
```

Verify:

```bash
kubectl -n openclaw exec -it openclaw-0 -c openclaw -- openclaw models status
```

This deployment sets the default model to `openai-codex/gpt-5.5`, so a valid `openai-codex` OAuth profile should appear under `Providers w/ OAuth/tokens` with no `Missing auth` entry for `openai`.

Do not commit `auth-profiles.json` to this repository. It can contain plaintext API keys, OAuth access tokens, and refresh tokens.
