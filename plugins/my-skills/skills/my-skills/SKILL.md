---
name: my-skills
description: List my custom skills (from ~/.claude/skills/)
---

# My Custom Skills

List all installed custom skills from the plugin system.

> **Source of truth: `~/projects/admin/claude-skills/`**

## Steps

1. Read `~/projects/admin/claude-skills/.claude-plugin/marketplace.json`
   - This single file lists all installed plugins with their names and metadata

2. For each plugin in marketplace.json:
   - List its skills directory: `~/projects/admin/claude-skills/plugins/<plugin-name>/skills/`
   - For each skill subdirectory found, read its `SKILL.md`
   - Extract `name` and `description` from YAML frontmatter
   - Command name comes from the `name` field (e.g. `todo-manager:daily` → `/todo-manager:daily`)

## Output

Group by plugin name with `───` underlines. Format:

```
Namespace
───
  /namespace:command       Description
```

This is the single source of truth — do NOT read CLAUDE.md for the command list.
