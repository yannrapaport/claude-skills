---
name: admin:add-profile
description: Create a new project workspace — .mcp.json, direnv secrets, zshrc aliases
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - AskUserQuestion
---

# /admin:add-profile — Create a new project workspace

Wizard to set up a new Claude Code project: MCP tools, secrets, and shell aliases.

## Phase 1 — Project info

Ask the user:
1. **Project slug** (e.g., `rakam`) — used for filenames, aliases, secrets
2. **Project directory** (e.g., `~/projects/tpg/rakam`) — where `.mcp.json` and `.envrc` will live
3. **Workspace alias prefix** — confirm `cd{slug}` and `cc{slug}` are the right alias names

## Phase 2 — MCP tools

Ask which MCP tools this project needs. Present known options:

| Tool | Package | Runner | Env vars |
|------|---------|--------|----------|
| ClickUp | `@taazkareem/clickup-mcp-server` | npx | `CLICKUP_API_TOKEN`, `CLICKUP_TEAM_ID` |
| Hostinger | `hostinger-api-mcp` | npx | `HOSTINGER_API_TOKEN` |
| Shopify | `shopify-mcp` | npx | `SHOPIFY_CLIENT_ID`, `SHOPIFY_CLIENT_SECRET`, `MYSHOPIFY_DOMAIN` |
| Klaviyo | `klaviyo-mcp-server@latest` | uvx | `KLAVIYO_API_KEY` |
| Moodify | `https://moodify.theproductguy.cloud/mcp` | http | `MOODIFY_TOKEN` |

Use `AskUserQuestion` with multiSelect. Include "None" and "Other (custom)" options.

For each custom tool, ask: server name, package/URL, runner (npx/uvx/http), env vars needed.

## Phase 3 — Secrets

For each env var that is a secret (not a static value like a domain), ask for its 1Password reference:
- Format: `op://vault/item/field`
- Suggest: "Most secrets are in the `Env` vault, e.g. `op://Env/ClickUp API Rakam/credential`"

For static env vars (e.g. `MYSHOPIFY_DOMAIN`), use the literal value directly in `.mcp.json`.

## Phase 4 — Generate files

### 4.1 `.mcp.json`

Write to `{project_dir}/.mcp.json`:

```json
{
  "mcpServers": {
    "{tool}": {
      "command": "npx",
      "args": ["-y", "{package}"],
      "env": {
        "{ENV_VAR}": "${ENV_VAR}"
      }
    }
  }
}
```

For HTTP tools:
```json
"{tool}": {
  "type": "http",
  "url": "{url}",
  "headers": {
    "Authorization": "Bearer ${TOKEN_VAR}"
  }
}
```

### 4.2 `op.{slug}.env`

Write to `~/projects/admin/dotfiles/secrets/op.{slug}.env`:

```
ENV_VAR=op://Env/Item Name/field
```

Static env vars (non-secret) go directly into `.mcp.json`, not here.

### 4.3 `.envrc`

Write to `{project_dir}/.envrc`:

```
source_env ~/.secrets/env.{slug}
```

Then run `direnv allow {project_dir}`.

### 4.4 Shell aliases

Read `~/projects/admin/dotfiles/zshrc`. Find the `# Project navigation` and `# cd + claude` sections.

Add after existing entries:
```zsh
alias cd{slug}='cd {project_dir}'
```
and:
```zsh
alias cc{slug}='cd{slug} && claude'
```

Show proposed additions and confirm before writing.

## Phase 5 — Inject secrets

```bash
op inject -i ~/projects/admin/dotfiles/secrets/op.{slug}.env | sed 's/^/export /' > ~/.secrets/env.{slug}
chmod 600 ~/.secrets/env.{slug}
```

If `op` is not authenticated, tell the user to run `op signin` first.

## Phase 6 — Commit

```bash
cd ~/projects/admin/dotfiles
git add secrets/op.{slug}.env zshrc
git commit -m "feat: add {slug} project workspace"
git push
```

## Phase 7 — Summary

> **Workspace `{slug}` created:**
> - MCP tools: `{project_dir}/.mcp.json` ({tool list})
> - Secrets: `dotfiles/secrets/op.{slug}.env` → `~/.secrets/env.{slug}`
> - direnv: `.envrc` configured and allowed
> - Aliases: `cd{slug}` / `cc{slug}`
>
> **To activate:** `source ~/.zshrc` then `cc{slug}` to launch Claude in this workspace.
