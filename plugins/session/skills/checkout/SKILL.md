---
name: session:checkout
description: Resume a named checkpoint from ai-brain/checkpoints/ without deleting it
allowed-tools:
  - Read
  - Bash
  - Glob
---

# session:checkout

Resume a named checkpoint — loads context from `ai-brain/checkpoints/` and injects it into the current conversation. The checkpoint file is **not deleted**.

**Usage:** `session:checkout <id>`
**Example:** `session:checkout rakam-weekly-20260415`

## Instructions

### 1. Validate argument

The `<id>` argument is required. If missing, tell the user:
```
ID requis : session:checkout <id>
Liste des checkpoints disponibles : session:log
```

### 2. Locate the checkpoint file

File path: `~/ai-brain/checkpoints/{id}.md`

If the file does not exist:
```
Checkpoint introuvable : ~/ai-brain/checkpoints/{id}.md
Le fichier n'existe pas sur cette machine — vérifie que git sync est à jour.
Liste des checkpoints disponibles : session:log
```
Stop. Do not continue.

### 3. Check for staleness

Run: `git -C ~/ai-brain log -1 --format="%ar" -- checkpoints/{id}.md`

If the last commit is older than 6 hours, warn:
```
⚠ Ce checkpoint n'a pas été commité depuis {time}. Si tu l'as modifié sur une autre machine, lance `git -C ~/ai-brain pull` d'abord.
```

### 4. Detect machine mismatch

Read the `machine:` field from the checkpoint's YAML frontmatter. Read current machine from `~/.claude/machine-context.md`.

If they differ, warn:
```
⚠ Ce checkpoint a été créé sur {checkpoint_machine}, tu es sur {current_machine}.
Les chemins de fichiers locaux peuvent être différents.
```

### 5. Read and inject the checkpoint

Read the full file. Then inject the context with this exact phrasing:

```
Tu reprends le checkpoint "{id}".

**Objectif :** {objectif}

**Prochaine action :** {prochaine action}

Voici le contexte complet :
---
{full checkpoint content}
---

Contexte chargé. Sur quelle action on commence ?
```

### 6. Do NOT delete the file

The checkpoint persists. Use `session:archive` to mark it done.
