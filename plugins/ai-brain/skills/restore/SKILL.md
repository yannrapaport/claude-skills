---
name: ai-brain:restore
description: Restore a session from a checkpoint — pulls the vault, lists checkpoints for the current project, loads the selected one
argument-hint: "[optional checkpoint filename or path]"
allowed-tools:
  - Read
  - Bash
  - Glob
---

<objective>
Resume a previously-saved work session by loading a checkpoint markdown into the current conversation context. Pulls the vault first so checkpoints created on another machine (e.g. Mac) are visible here (e.g. Nexus).
</objective>

<context>
Checkpoint storage: `$HOME/ai-brain/checkpoints/<project>/YYYY-MM-DD-HHmm-<slug>.md`

Project slug detection (from `pwd`):
| cwd starts with… | slug |
|------------------|------|
| `~/projects/tpg/rakam` | `rakam` |
| `~/projects/tpg/atlanticlog` | `atlanticlog` |
| `~/projects/la-pelucherie` | `lp` |
| `~/projects/tpg/training` | `aid` |
| `~/projects/maestro` | `maestro` |
| `~/projects/tpg` (other) | `tpg` |
| `~/ai-brain` or the Obsidian vault | `ai-brain` |
| anywhere else | ask the user |

Argument: `$ARGUMENTS` (optional — either a full path, a bare filename, or empty for interactive selection).
</context>

<process>

## Step 1 — Pull the vault

Sync any checkpoints created on another machine:
```bash
cd ~/ai-brain && git pull --ff-only 2>&1
```

If the pull fails (conflict, divergence), report the error and stop. Don't try to resolve automatically — the user can rerun once resolved.

## Step 2 — Detect project

Run `pwd` and map to a project slug using the table in <context>. If no match, ask the user once which project's checkpoints to list.

## Step 3 — Resolve checkpoint to load

**Case A — `$ARGUMENTS` is a full path** (starts with `/` or `~`):
Use it directly. Skip to step 4.

**Case B — `$ARGUMENTS` is a bare filename** (e.g. `2026-04-28-1430-copilot-tests`):
Resolve to `~/ai-brain/checkpoints/<project>/<arg>.md` (add `.md` if missing).

**Case C — `$ARGUMENTS` is empty:**
1. List checkpoints in `~/ai-brain/checkpoints/<project>/` sorted by filename **descending** (most recent first).
2. If none exist, report `Aucun checkpoint pour <project>.` and stop.
3. Show up to 10 most recent as a numbered list:
   ```
   Checkpoints pour <project> :
   1. 2026-04-28-1430-copilot-tests       (mac, today 14:30)
   2. 2026-04-26-1015-data-collection-prd (mac, sat 10:15)
   3. ...
   ```
   For each, read just the frontmatter + Résumé to display the one-line résumé inline.
4. Ask: `"Lequel ? (numéro, ou Entrée pour le plus récent)"`. Wait for answer.
5. Resolve to the chosen path.

## Step 4 — Read and present

Read the checkpoint file. Display:

```
► Checkpoint loaded: <filename>
  Project: <project>  |  Machine: <mac|nexus>  |  Created: <iso>

═══ Résumé ═══
<résumé content>

═══ Done ═══
<bullets>

═══ Next ═══
<bullets>

═══ Open / blockers ═══
<bullets, or "none">

═══ Refs ═══
<bullets>
```

Then a one-line invitation:
```
→ Reprends sur : <first bullet of Next>
```

## Step 5 — Optional context loading

If the **Refs** section lists files (paths starting with `~/` or `/`), don't read them automatically — just list them. Let the user say `"lis les refs"` if they want them loaded into context.

</process>

<constraints>
- Always `git pull` first. The whole point is cross-machine sync.
- Read-only on the checkpoint. Never modify or delete checkpoint files in this skill.
- Display the checkpoint contents verbatim — don't paraphrase or compress.
- Don't auto-load Refs. The user opts in.
- If multiple `<project>` slugs are plausible from `pwd`, ask once instead of guessing.
</constraints>
