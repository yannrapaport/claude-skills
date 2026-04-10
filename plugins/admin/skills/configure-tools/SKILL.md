---
name: admin:configure-tools
description: Add, modify, or remove MCP servers in a project's .mcp.json and secrets
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - AskUserQuestion
---

# /admin:configure-tools — Manage MCP tools for a project

## Step 1 — Select project

Find all projects with a `.mcp.json`:
```bash
find ~/projects -name '.mcp.json' -not -path '*/node_modules/*' 2>/dev/null
```

Use `AskUserQuestion` to let the user pick one. Infer `{slug}` from the project directory name or confirm with user.

## Step 2 — Show current config

Read the selected `.mcp.json` and `~/projects/admin/dotfiles/secrets/op.{slug}.env`.

Display:

| MCP Server | Runner | Env vars | Secret refs |
|------------|--------|----------|-------------|

## Step 3 — Choose action

Use `AskUserQuestion`:
1. **Add a tool**
2. **Modify a tool**
3. **Remove a tool**
4. **Done**

### Add a tool

Present known tools:

| Tool | Package | Runner | Env vars |
|------|---------|--------|----------|
| ClickUp | `@taazkareem/clickup-mcp-server` | npx | `CLICKUP_API_TOKEN`, `CLICKUP_TEAM_ID` |
| Hostinger | `hostinger-api-mcp` | npx | `HOSTINGER_API_TOKEN` |
| Shopify | `shopify-mcp` | npx | `SHOPIFY_CLIENT_ID`, `SHOPIFY_CLIENT_SECRET`, `MYSHOPIFY_DOMAIN` |
| Klaviyo | `klaviyo-mcp-server@latest` | uvx | `KLAVIYO_API_KEY` |
| Moodify | `https://moodify.theproductguy.cloud/mcp` | http | `MOODIFY_TOKEN` |

For secrets: ask for 1Password reference (`op://vault/item/field`).
For static values: use literal in `.mcp.json` directly.

### Modify a tool

List current servers, let user pick. Show current config, ask what to change.

### Remove a tool

List current servers, let user pick. Remove its env vars from `op.{slug}.env` if not shared.

## Step 4 — Apply changes

Show diff and confirm before writing. Then re-inject secrets:
```bash
op inject -i ~/projects/admin/dotfiles/secrets/op.{slug}.env | sed 's/^/export /' > ~/.secrets/env.{slug}
chmod 600 ~/.secrets/env.{slug}
```

Loop back to Step 3 until "Done".

## Step 5 — Commit

```bash
cd ~/projects/admin/dotfiles
git add secrets/op.{slug}.env
git commit -m "feat({slug}): update MCP tools"
git push
```
