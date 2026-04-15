---
name: session:log
description: List all active checkpoints in ai-brain/checkpoints/
allowed-tools:
  - Read
  - Bash
  - Glob
---

# session:log

List all active checkpoints stored in `ai-brain/checkpoints/`.

**Usage:** `session:log` — lists all active checkpoints
**Usage:** `session:log <project>` — filters by project

## Instructions

### 1. Glob checkpoint files

```bash
ls ~/ai-brain/checkpoints/*.md 2>/dev/null
```

If no files found:
```
Aucun checkpoint trouvé dans ~/ai-brain/checkpoints/
Pour créer un checkpoint : session:commit <nom>
```
Stop.

### 2. Read each file's frontmatter

For each `.md` file, extract from the YAML frontmatter:
- `id`
- `project`
- `updated`
- `machine`
- `status`

And from the body, extract the **first line** of `## Prochaine action`.

### 3. Filter

- If a `<project>` argument was provided, only show checkpoints where `project` matches.
- Only show checkpoints where `status: active` (skip `archived` and `done`).

### 4. Display as table

```
Checkpoints actifs
──────────────────────────────────────────────────────────────────
ID                              Projet     Mis à jour   Machine   Prochaine action
rakam-weekly-20260415           rakam      2026-04-15   mac       Rédiger section "Risques"
email-hygiene-pipeline-20260413 tpg        2026-04-13   nexus     Tester le cron 07:15

Pour reprendre : session:checkout <id>
Pour archiver  : session:archive <id>
```

If no active checkpoints match the filter:
```
Aucun checkpoint actif{for project <project>}.
```
