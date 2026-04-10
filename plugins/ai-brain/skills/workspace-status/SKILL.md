---
name: ai-brain:workspace-status
description: Show workspace status (purpose, todos, journal, git)
---

# Workspace Status

Show a structured status report for the current workspace.

## Step 1 — Identify workspace

Check `pwd` and match against known workspace roots:

| Workspace | Path |
|-----------|------|
| Rakam | `~/projects/tpg/rakam` |
| AtlanticLog | `~/projects/tpg/atlanticlog` |
| La Pelucherie | `~/projects/la-pelucherie` |
| AI Brain | `~/ai-brain` or the Obsidian vault |
| ai-discipline | `~/projects/tpg/training` |

If not in a known workspace, check if the current directory has a `CLAUDE.md` and `TODO.md` — treat it as a generic workspace. If neither, say so and stop.

## Step 2 — Gather data

Collect ALL of the following in parallel:

### 2a. Purpose
Read `CLAUDE.md` in the workspace root. Extract the role/objective in one sentence.

### 2b. Todos
Read `TODO.md` in the workspace root. Count items by priority tag (`!must`, `!should`, `!could`). List all `!must` items. If no priority tags, list the first 5 unchecked items.

Also check for nested TODO files (e.g. `ots-modules/TODO.md`) — mention them with their item count if they exist.

### 2c. Last journal entry
Find the most recent file in `journal/` (by filename sort, descending). Show its date and first 5 non-empty lines as a preview.

### 2d. Git status
Run these for the workspace root:
- `git status --short` — uncommitted changes
- `git log -1 --format='%h %ar — %s'` — last commit
- `git rev-list --left-right --count HEAD...@{upstream} 2>/dev/null` — ahead/behind remote

### 2e. Repos health
If `repos/` directory exists, for each subdirectory that is a git repo:
- Current branch
- Dirty/clean (any uncommitted changes?)
- Last commit date (`git log -1 --format='%ar'`)

## Step 3 — Output

Present as a clean dashboard. Example:

```
## Rakam — Status

**Role:** Head of Product — structurer l'org produit
**Last active:** 2026-03-27 (3 days ago)

### Must-do
- [ ] PRDs + kickoff lundi 30/03
- [ ] Kickoff Societe.com Copilot (mer 01/04)

### Todos: 2 must / 7 should / 0 could
Also: `ots-modules/TODO.md` (4 items)

### Last journal (2026-03-27)
> Discovery session with Edmee on data collection...

### Git
Workspace: clean, synced with origin (last commit: abc1234 3d ago — "Add journal entry")
Repos:
  rakam-app      main    2 dirty files    3d ago
  rakam-docs     main    clean            5d ago
```

Rules:
- Keep it scannable — no prose, just structured data
- If a section has no data (e.g. no journal/ folder), skip it silently
- Dates should show both absolute and relative ("2026-03-27, 3d ago")
- Flag anything that looks stale (no commit in 7+ days, no journal in 7+ days) with a warning
