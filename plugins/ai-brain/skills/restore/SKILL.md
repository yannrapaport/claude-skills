---
name: ai-brain:restore
description: Restore a session from a checkpoint — pulls the vault, lists checkpoints for the current project, loads the selected one
argument-hint: "[optional checkpoint filename or path]"
allowed-tools:
  - Bash
---

<objective>
Resume a previously-saved work session by loading a checkpoint markdown into the current conversation context. Pulls the vault first so checkpoints created on another machine (e.g. Mac) are visible here (e.g. Nexus).
</objective>

<performance>
This skill is mechanical. Do all filesystem work in a **single Bash call** — pull, detect project, list/print in one shot. No `Read` tool needed: `cat` returns the content directly.

Tip for the user: `/fast` mode (Opus 4.6) or `/model claude-haiku-4-5` makes mechanical skills feel snappier.
</performance>

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

## Single-shot mode — argument provided

If `$ARGUMENTS` is non-empty, run **one** Bash command that:
1. Pulls the vault.
2. Resolves the path:
   - Starts with `/` or `~` → use directly.
   - Bare filename → `~/ai-brain/checkpoints/<project>/<arg>.md` (add `.md` if missing).
3. `cat`s the resolved file.

```bash
cd ~/ai-brain && git pull --ff-only 2>&1 | tail -1 && echo "---" && cat <resolved_path>
```

Then format and present (see <presentation>).

## Interactive mode — no argument

Run **one** Bash command that pulls + lists checkpoints with their one-line résumé:

```bash
cd ~/ai-brain && git pull --ff-only 2>&1 | tail -1 && echo "---"
PROJECT=<detected>
DIR="$HOME/ai-brain/checkpoints/$PROJECT"
[ -d "$DIR" ] || { echo "Aucun checkpoint pour $PROJECT."; exit 0; }
i=0
for f in $(ls -t "$DIR"/*.md 2>/dev/null | head -10); do
  i=$((i+1))
  name=$(basename "$f" .md)
  resume=$(awk '/^## Résumé/{flag=1; next} /^## /{flag=0} flag && NF{print; exit}' "$f")
  printf "%d. %s\n   %s\n" "$i" "$name" "$resume"
done
```

Present the list to the user. Ask: `"Lequel ? (numéro, ou Entrée pour le plus récent)"`.

Once the user answers, run **one more** Bash command:
```bash
cat ~/ai-brain/checkpoints/<project>/<chosen>.md
```

Then format and present.

</process>

<presentation>
Display the checkpoint contents verbatim, in this format:

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

If **Refs** lists files (paths starting with `~/` or `/`), don't read them automatically — just list them. The user opts in with `"lis les refs"`.
</presentation>

<constraints>
- Always `git pull` first. The whole point is cross-machine sync.
- **Hard cap: 1 Bash call when an argument is given, 2 Bash calls in interactive mode.** Don't add Read tool calls — `cat` is enough.
- Read-only on the checkpoint. Never modify or delete checkpoint files.
- Display the checkpoint contents verbatim — don't paraphrase or compress.
- Don't auto-load Refs. The user opts in.
- If the pull fails (conflict, divergence), report the error and stop.
- If multiple `<project>` slugs are plausible from `pwd`, ask once instead of guessing.
</constraints>
