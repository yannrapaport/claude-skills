---
name: admin:refresh-secrets
description: Re-inject 1Password secrets into ~/.secrets/env.* for all project workspaces
allowed-tools:
  - Bash
---

# /admin:refresh-secrets — Refresh secrets from 1Password

Re-injects all `op.*.env` files into `~/.secrets/env.*` via the 1Password CLI.

## Steps

1. **Check `op` CLI:**
```bash
op whoami
```
If not authenticated, stop and tell the user to run `op signin`.

2. **Find all secret files:**
```bash
ls ~/projects/admin/dotfiles/secrets/op.*.env
```

3. **For each `op.{slug}.env`:**
```bash
op inject -i ~/projects/admin/dotfiles/secrets/op.{slug}.env \
  | sed 's/^/export /' > ~/.secrets/env.{slug}
chmod 600 ~/.secrets/env.{slug}
```

If `op inject` fails: report the error, do NOT overwrite the existing file, continue with others.

4. **Summary:**

| Slug | Status |
|------|--------|
| rakam | ✓ refreshed |
| lp | ✓ refreshed |

## Notes

- `~/.secrets/env.*` are never committed to git
- After refreshing, direnv reloads automatically when you `cd` into the project
- Secrets are consumed at Claude Code launch via `.mcp.json` env var substitution
