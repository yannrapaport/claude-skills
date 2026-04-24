---
name: admin:refresh-secrets
description: Re-inject Bitwarden secrets into ~/.secrets/env.* for all project workspaces
allowed-tools:
  - Bash
---

# /admin:refresh-secrets — Refresh secrets from Bitwarden

Re-injects all `bw.*.env` templates into `~/.secrets/env.*` via the Bitwarden CLI and the `bw-inject` helper.

## Steps

1. **Ensure Bitwarden vault is unlocked:**

```bash
bw status | jq -r .status
```

If status is not `unlocked`, prompt the user to run in their terminal:
```bash
bw unlock --raw > /tmp/bw-session
chmod 600 /tmp/bw-session
```

Then export the session for this skill run:
```bash
export BW_SESSION=$(cat /tmp/bw-session)
```

If still not unlocked after that, stop and report.

2. **Find all secret templates:**

```bash
ls ~/projects/admin/dotfiles/secrets/bw.*.env
```

3. **For each `bw.{slug}.env`:**

```bash
bw-inject < ~/projects/admin/dotfiles/secrets/bw.{slug}.env \
  | sed 's/^/export /' > ~/.secrets/env.{slug}.new && \
  mv ~/.secrets/env.{slug}.new ~/.secrets/env.{slug} && \
  chmod 600 ~/.secrets/env.{slug}
```

(Write to `.new` first, then atomic rename — protects against partial writes if bw-inject fails mid-template.)

If `bw-inject` fails: report the error, leave the existing `env.{slug}` untouched, continue with the next template.

4. **Summary table:**

| Slug | Status |
|------|--------|
| perso | ✓ refreshed |
| rakam | ✓ refreshed |
| tpg | ✓ refreshed |
| lp | ✓ refreshed |

## Notes

- `~/.secrets/env.*` are never committed to git
- After refreshing, direnv reloads automatically when you `cd` into the project
- Secrets are consumed at Claude Code launch via `.mcp.json` env var substitution
- Bitwarden master password is stored in Apple Passwords (iCloud Keychain)
- Template syntax: `KEY=bw://Item Name/field` where `field` is `password`, `username`, or any custom field name
