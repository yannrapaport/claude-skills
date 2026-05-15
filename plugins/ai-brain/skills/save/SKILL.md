---
name: ai-brain:save
description: Save a checkpoint of the current work session — writes a resume markdown, appends to the project journal (if any), updates TODOs, and commits to the vault for cross-machine sync
argument-hint: "[optional-slug]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
---

<objective>
Capture the current work session so it can be resumed later — on this machine or another (Mac ↔ Nexus). Single immutable markdown file, plus side-effects on journal and TODOs when relevant.
</objective>

<context>
**Vault root resolution (do this FIRST, mentally):**
- If current cwd matches `$HOME/.cache/ai-brain-worktrees/session-*`, the vault root is the cwd (you are inside an `aib` worktree session — write into the worktree so wrap-up can merge cleanly).
- Otherwise, the vault root is `$HOME/ai-brain`.

Notation: `<vault>` below refers to whichever root applies.

Checkpoint storage: `<vault>/checkpoints/<project>/YYYY-MM-DD-HHmm-<slug>.md`
The `checkpoints/` folder is git-tracked in the Obsidian vault, so a commit + push at the end of this skill makes the checkpoint available on Nexus (and any other machine with the vault).

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

Journal locations (per-project, append-only):
- `~/projects/tpg/rakam/journal/YYYY-MM-DD.md`
- `~/projects/tpg/atlanticlog/journal/YYYY-MM-DD.md`
- (other projects: only if a `journal/` folder exists at the project root)

If the project has no `journal/` folder, **skip the journal step silently** — do not auto-create it.

TODOs:
- Hub: `<vault>/todos/active-todos.md` (git-tracked dans le vault depuis V0 continuous-todo, synced via commit/push)
- Project: `<project-root>/TODO.md` if it exists

Argument: `$ARGUMENTS` (optional slug — if absent, derive a 2-4 word kebab-case slug from the session summary).
</context>

<process>

## Step 1 — Detect project + machine

1. Run `pwd` to get cwd.
2. Map to a project slug using the table in <context>. If no match, ask the user once: `"Quel projet ? (rakam / atlanticlog / lp / aid / maestro / ai-brain / autre — précise)"`.
3. Detect machine: `hostname` → if it contains `nexus` or matches `~/.claude/machine-context.md`, use `nexus`; otherwise `mac`.
4. Get the current ISO timestamp with timezone: `date +%Y-%m-%dT%H:%M%:z`.
5. Get the filename date+time: `date +%Y-%m-%d-%H%M`.

## Step 2 — Synthesize the session

From the **current conversation context** (not from files), build the checkpoint contents:

- **Résumé** (1-2 phrases) — pourquoi cette session, focus principal.
- **Done** — bullets concrets de ce qui a été fait (commits, décisions, livrables).
- **Next** — l'action concrète au prochain démarrage. Une phrase claire qui permet de reprendre sans relire toute la session.
- **Open / blockers** — questions ouvertes, attentes externes, points à trancher. Vide si rien.
- **Refs** — fichiers touchés, threads, liens, IDs externes. Inclure les chemins absolus.

Be concrete. Bullets > prose. No filler.

## Step 3 — Determine the slug

If `$ARGUMENTS` is non-empty, use it (lowercased, kebab-case).
Otherwise generate a 2-4 word kebab-case slug from the **Résumé** content (e.g. `copilot-tests-bachar`, `fix-imap-backend`, `roadmap-l2-draft`).

## Step 4 — Write the checkpoint file

Path: `<vault>/checkpoints/<project>/<filename-date>-<slug>.md`

Create the project subfolder if it doesn't exist (`mkdir -p`).

File contents:
```markdown
---
project: <slug>
created: <iso-timestamp>
machine: <mac|nexus>
---

## Résumé
<1-2 phrases>

## Done
- ...

## Next
- ...

## Open / blockers
- ...

## Refs
- ...
```

Use Write to create the file.

## Step 5 — Append to project journal (if applicable)

Determine the journal path:
- For `rakam` → `~/projects/tpg/rakam/journal/<today>.md`
- For `atlanticlog` → `~/projects/tpg/atlanticlog/journal/<today>.md`
- For others: check if `<project-root>/journal/` exists; if yes, use `<project-root>/journal/<today>.md`; else skip.

If the journal file for today exists: append a section at the end.
If it doesn't exist (but the `journal/` folder does): create today's file with a minimal header.

Section to append:
```markdown

### Session checkpoint <HH:MM>
<Résumé from step 2>

→ Checkpoint: `~/ai-brain/checkpoints/<project>/<filename>.md`
```

If the project has no `journal/` folder, skip silently.

## Step 6 — Update TODOs (only if there's clear movement)

Look at the **Done** and **Open / blockers** lists. Update TODOs only when an item maps unambiguously:

- An item in **Done** that matches an existing `- [ ]` line in `active-todos.md` or `<project-root>/TODO.md` → mark `- [x]` with ` ✓ <today>`.
- A new blocker or next-step worth tracking → propose adding it. Ask the user for confirmation before adding (one yes/no, list everything proposed in one message).

Be conservative. If in doubt, skip and mention it in the final summary instead.

## Step 7 — Commit + push the vault

Run from `<vault>` (which may be `$HOME/ai-brain` or the `aib` session worktree):

```bash
cd <vault>
git add checkpoints/
git commit -m "checkpoint: <project> — <slug>

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
git push
```

If `<vault>` is a worktree, the commit lands on the `session-*` branch and will be merged into `main` at the next `/ai-brain:wrap-up`. The push to origin may fail because the branch has no upstream — that's OK, the merge at wrap-up time will propagate it.

Do NOT add other paths (`.obsidian/workspace.json`, etc.) — only the checkpoint file.

If the working tree has unrelated staged changes, leave them alone (they'll show up in `git status` after — do not stage them).

If `$ARGUMENTS` contains the literal token `--no-push`, skip the push (still commit locally).

## Step 8 — Confirmation output

Show the user:
```
✓ Checkpoint saved
  Path:    ~/ai-brain/checkpoints/<project>/<filename>.md
  Project: <project>
  Machine: <mac|nexus>

  Résumé:  <Résumé from step 2>
  Next:    <first bullet of Next>

  Journal: <appended | skipped (no journal/ folder)>
  TODOs:   <X marked done, Y added | none touched>
  Vault:   <pushed | committed locally only>
```

</process>

<constraints>
- The checkpoint file is **immutable**. Never edit a previous checkpoint. To update state, create a new one.
- Frontmatter is exactly 3 fields: `project`, `created`, `machine`. No `id`, `updated`, `status`, or `tags`.
- Sections are exactly 5: Résumé, Done, Next, Open / blockers, Refs. Do not invent new sections.
- Synthesize from the **conversation**, not by reading random files. The agent already knows what happened.
- Do not commit anything outside `checkpoints/` in step 7.
- Do not auto-create `journal/` folders — projects opt in by creating the folder themselves.
- TODO updates must be conservative. When in doubt, surface to the user instead of editing silently.
</constraints>
