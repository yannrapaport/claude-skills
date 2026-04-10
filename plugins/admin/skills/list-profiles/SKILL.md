---
name: admin:list-profiles
description: Overview of all project workspaces — MCP tools, secrets, aliases, direnv
allowed-tools:
  - Read
  - Bash
---

# /admin:list-profiles — Overview of all project workspaces

## Step 1 — Gather data

```bash
find ~/projects -name '.mcp.json' -not -path '*/node_modules/*' 2>/dev/null
ls ~/projects/admin/dotfiles/secrets/op.*.env 2>/dev/null
find ~/projects -name '.envrc' -not -path '*/node_modules/*' 2>/dev/null
grep -E 'alias (cd|cc)[a-z]' ~/projects/admin/dotfiles/zshrc
```

## Step 2 — Display summary

For each project slug found across all sources:

| Slug | Project dir | MCP servers | Secrets file | .envrc | Aliases | Status |
|------|-------------|-------------|--------------|--------|---------|--------|

**Status logic:**
- **OK** — `.mcp.json` + `op.*.env` + `.envrc` + `cd`/`cc` aliases all present
- **WARN: no .envrc** — MCP configured but direnv not set up
- **WARN: no secrets** — `.mcp.json` has `${VAR}` but no `op.*.env` found
- **WARN: no alias** — no `cc{slug}` in zshrc
- **WARN: secrets not injected** — `op.*.env` exists but `~/.secrets/env.{slug}` missing or older than 7 days

## Step 3 — Details (optional)

If the user asks for a specific project, show:
- Full `.mcp.json`
- All env vars and 1Password refs from `op.{slug}.env`
- Whether `~/.secrets/env.{slug}` exists and last modified date
