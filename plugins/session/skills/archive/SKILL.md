---
name: session:archive
description: Mark a checkpoint as archived (done) in ai-brain/checkpoints/
allowed-tools:
  - Read
  - Write
  - Bash
---

# session:archive

Mark a checkpoint as `archived` — it disappears from `session:log` but is preserved in git history.

**Usage:** `session:archive <id>`
**Example:** `session:archive rakam-weekly-20260415`

## Instructions

### 1. Validate argument

If no `<id>` provided:
```
ID requis : session:archive <id>
Liste des checkpoints : session:log
```

### 2. Locate the file

File path: `~/ai-brain/checkpoints/{id}.md`

If not found:
```
Checkpoint introuvable : {id}
```
Stop.

### 3. Update the status field

Read the file. Replace `status: active` with `status: archived` in the YAML frontmatter. Also update `updated: {YYYY-MM-DD}` to today's date.

Write the file back.

### 4. Confirm

```
Checkpoint archivé : {id}
Le fichier reste dans ~/ai-brain/checkpoints/ mais n'apparaît plus dans session:log.
```
