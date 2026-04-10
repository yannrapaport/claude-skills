---
name: pm-copilot:help
description: Show available pm-copilot skills and check required MCP servers
---

# /pm-copilot:help — Show available skills

List all pm-copilot skills with descriptions.

## Instructions

Read the commands directory and display a summary:

```bash
ls ~/projects/tpg/pm-copilot/skills/*/SKILL.md
```

Then display:

---

## PM Assistant Commands

| Command | Description |
|---------|-------------|
| `/pm-copilot:task <ticket-url>` | Work on a ticket — fetch details, discover repo, clone, fix, submit PR |
| `/pm-copilot:weekly-report` | Generate weekly report — auto-pull ClickUp data, post to Slack & Notion |
| `/pm-copilot:help` | Show this help |

> **Note:** Profile management has moved to `/admin:add-profile`, `/admin:configure-tools`, and `/admin:list-profiles`.

## Options

- `--profile <name>` — Override auto-detection (works with `task` and `weekly-report`)

## Profiles

List installed profiles:
```bash
ls ~/projects/tpg/pm-copilot/profiles/*.yaml
```

Show each profile's `name` and `display_name`.

## MCP servers

Check that required MCP servers are configured:
- **ClickUp** — ticket access
- **GitHub** — PRs and repo operations
- **Notion** — page read/write for reports

Report which are present and which are missing from `~/.claude/settings.json`.

## Links

- **Repo:** https://github.com/yannrapaport/pm-copilot
- **Install:** `bash <(curl -s https://raw.githubusercontent.com/yannrapaport/pm-copilot/main/install.sh)`
- **Update:** `cd ~/projects/tpg/pm-copilot && git pull`

---
