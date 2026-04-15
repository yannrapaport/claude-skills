---
name: session:commit
description: Save a named, persistent checkpoint of the current session to ai-brain/checkpoints/
allowed-tools:
  - Write
  - Read
  - Bash
  - Glob
---

# session:commit

Save a named checkpoint of the current conversation to `ai-brain/checkpoints/` for later resumption — potentially on another machine or in a future conversation.

**Usage:** `session:commit <name>`
**Example:** `session:commit weekly` → `rakam-weekly-20260415.md`

## Instructions

### 1. Validate argument

The `<name>` argument is **required**. It is everything after `session:commit` in the user's message (e.g., `session:commit working on PRD` → name is `working on PRD`).

If no name is provided, stop and tell the user:
```
Nom requis : session:commit <nom>
Exemple : session:commit weekly
```

### 2. Determine project slug

Detect the current project from the working directory path. Use the last meaningful directory segment:
- `~/projects/rakam/` → `rakam`
- `~/projects/tpg/email-hygiene/` → `email-hygiene`
- `~/ai-brain/` → `ai-brain`
- If unclear, use `perso`

### 3. Build the file ID

- Slugify the name: lowercase, spaces → hyphens, remove special chars
- Format: `{project}-{slug}-{YYYYMMDD}`
- Example: `rakam-working-on-prd-20260415`
- File path: `~/ai-brain/checkpoints/{id}.md`

If a file with this ID already exists → **overwrite silently** (same project + same name + same day = update).

### 4. Detect machine

Read `~/.claude/machine-context.md` to get the current machine name (e.g., `mac`, `nexus`). Use the `machine:` field value.

### 5. Synthesize checkpoint content

Review the conversation and extract:
- **Objectif** : what the user is trying to accomplish (1-3 sentences)
- **État** : what has been done, what is pending, current blockers
- **Décisions** : key decisions made during the session (bullet list)
- **Prochaine action** : **one sentence, one action, zero ambiguity** — the exact next step to take when resuming
- **Références clés** : file paths, URLs, IDs, data needed to continue

### 6. Write the checkpoint file

```markdown
---
id: {id}
project: {project}
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
machine: {machine}
status: active
tags: []
---

## Objectif
{What the user is trying to accomplish}

## État
{What has been done / what is pending / blockers}

## Décisions
- {Decision 1}
- {Decision 2}

## Prochaine action
{One sentence. One action. Zero ambiguity.}

## Références clés
- {file path or URL or ID}
```

### 7. Ensure checkpoints directory exists

```bash
mkdir -p ~/ai-brain/checkpoints
```

### 8. Confirm to the user

```
Checkpoint sauvegardé : ~/ai-brain/checkpoints/{id}.md
Pour reprendre : session:checkout {id}
```
