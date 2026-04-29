---
name: admin:refresh-secrets
description: Re-inject Bitwarden secrets into ~/.secrets/ — env files (for direnv) AND arbitrary files (OAuth tokens, service-account JSON keys, etc.) for all project workspaces
allowed-tools:
  - Bash
---

# /admin:refresh-secrets — Refresh secrets from Bitwarden

Re-injects all `bw.*.env` and `bw.*.files` templates from `~/projects/admin/dotfiles/secrets/` into `~/.secrets/` via the Bitwarden CLI and the `bw-inject` / `bw-files` helpers.

## Inputs and outputs

| Template | Output | Use case |
|----------|--------|----------|
| `bw.{slug}.env` | `~/.secrets/env.{slug}` (export-prefixed) | Env vars consumed by direnv + `.mcp.json` substitution |
| `bw.{slug}.files` | `~/.secrets/<custom path>` | OAuth tokens, service-account JSON keys, anything not env-shaped |

## Template syntax

**`bw.{slug}.env`** — one `KEY=value` per line:

```
SHOPIFY_CLIENT_ID=bw://Shopify - La Pelucherie/client_id
KLAVIYO_API_KEY=bw://Klaviyo API/password
SHOPIFY_STORE=la-pelucherie.myshopify.com
```

**`bw.{slug}.files`** — one `<dest>=<bw://Item/field>` per line:

```
~/.secrets/lp-seo-tracking-sa.json=bw://lp-seo-tracking-sa/notes
~/.secrets/lp-seo-oauth-token.json=bw://lp-seo-oauth-token/notes
```

Both formats:
- Resolve `password`, `username`, `notes`, `totp`, `uri` as standard fields; anything else as a custom field on the BW item.
- Skip blank lines and `#` comments.
- `~` in destination paths expands to `$HOME`.

## Steps

1. **Ensure Bitwarden vault is unlocked:**

```bash
bw status | jq -r .status
```

If status is not `unlocked`, prompt the user to run in their terminal:
```bash
! export BW_SESSION=$(bw unlock --raw)
```
(The `!` prefix runs the command in this Claude Code session so the BW_SESSION is available to subsequent shell calls.)

If still not unlocked after that, stop and report.

2. **Find all templates:**

```bash
ls ~/projects/admin/dotfiles/secrets/bw.*.env ~/projects/admin/dotfiles/secrets/bw.*.files 2>/dev/null
```

3. **For each `bw.{slug}.env`** (env file output):

```bash
bw-inject < ~/projects/admin/dotfiles/secrets/bw.{slug}.env \
  | sed 's/^/export /' > ~/.secrets/env.{slug}.new && \
  mv ~/.secrets/env.{slug}.new ~/.secrets/env.{slug} && \
  chmod 600 ~/.secrets/env.{slug}
```

(Atomic rename: if `bw-inject` fails mid-template, the existing `env.{slug}` is preserved.)

4. **For each `bw.{slug}.files`** (file output):

```bash
bw-files < ~/projects/admin/dotfiles/secrets/bw.{slug}.files
```

`bw-files` writes each resolved value directly to the destination path, atomic-replacing any existing file (chmod 600).

If `bw-inject` or `bw-files` fails for a slug: report the error, leave existing files untouched, continue with the next slug.

5. **Summary table:**

| Slug | env | files |
|------|-----|-------|
| perso | ✓ | — |
| rakam | ✓ | — |
| tpg | ✓ | — |
| lp | ✓ | ✓ (3 files) |

## Notes

- `~/.secrets/*` are never committed to git
- After refreshing, direnv reloads automatically when you `cd` into a project that sources `~/.secrets/env.{slug}` from its `.envrc`
- Env vars are also consumed at Claude Code launch via `.mcp.json` env var substitution
- Bitwarden master password is stored in Apple Passwords (iCloud Keychain)
- For binary files (rare), prefer Bitwarden attachments + a manual retrieval step
- Legacy `op.*.env` templates (1Password) are deprecated — migrate to `bw.*.env` over time
