---
name: ai-brain:wrap-up
description: End-of-session orchestrator — journals the session recap into the project's journal/ (cwd-aware), commits + pushes the current repo, syncs Nexus when run from Mac, and offers a checkpoint when there are loose ends.
argument-hint: "[optional-note]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - TaskList
---

<objective>
Close out a working session cleanly. Journal the session recap into the current project's `journal/` (when it has one), commit + push the current repo, propagate to the other machine, and surface whether a checkpoint is worth it. No hidden side-effects, no auto-checkpoint.
</objective>

<context>
The skill runs from the **current repo's working directory**. It does not iterate over multiple repos (use `git-sync-all` for that).

**Special case — worktree session :** if `pwd` is inside `~/.cache/*-worktrees/session-*`, the skill auto-detects an isolated session (created by `aib`, `ccwt`, or similar wrapper). In this case it commits the WIP in the worktree, then merges the `session-*` branch back into `main` in the main worktree, pushes, and cleans up the worktree + branch. See Step 0 and Step 3.5.

Path mapping Mac ↔ Nexus is **identical**: `~/projects/<X>` on one machine maps to `~/projects/<X>` on the other.

`$ARGUMENTS` (optional) is appended to the auto-drafted commit message as additional context. If absent, the message is derived from the diff alone.

Auto-commit is the default when there are uncommitted changes — but only when **all** safety conditions hold (see Step 3). When any condition fails, stop and report to the user instead of forcing through.
</context>

<process>

## Step 0 — Detect worktree session

1. Run `pwd`.
2. If `pwd` matches `$HOME/.cache/*-worktrees/session-*` (glob, not regex):
   - Set `IS_WORKTREE_SESSION=true`.
   - Capture `WORKTREE_PATH="$(pwd)"` and `SESSION_BRANCH="$(basename "$WORKTREE_PATH")"`.
3. Otherwise → `IS_WORKTREE_SESSION=false`, normal wrap-up flow only.

When `IS_WORKTREE_SESSION=true`:
- Step 1's `repo_path` will be the worktree (not the main worktree). This is expected.
- Step 3 commits the WIP **in the worktree** (on branch `session-XXX`).
- Step 3.5 (after Step 3) merges `session-XXX` back into `main` in the main worktree, pushes, and cleans up.
- Step 5 (checkpoint decision) still applies independently. Note: its signals (`journal/`, `scratchpad/in-progress.md`) are ai-brain-specific — for non-ai-brain worktrees, expect "not needed" silently.

## Step 1 — Detect machine + repo

1. Read `~/.claude/machine-context.md` to determine `machine` (`mac` or `nexus`).
2. Run `pwd` and `git rev-parse --show-toplevel 2>/dev/null` to find the repo root.
3. If not in a git repo → skip Steps 2–4 entirely. Jump to Step 5 (checkpoint decision) and Step 6 (summary).
4. Capture: `branch=$(git symbolic-ref --short HEAD 2>/dev/null)`, `upstream=$(git rev-parse --abbrev-ref @{u} 2>/dev/null)`, `repo_path` (absolute), `repo_name` (basename).

## Step 2 — Inspect repo state

Run in parallel:
- `git status --porcelain` (working tree + index)
- `git log @{u}..HEAD --oneline 2>/dev/null` (unpushed commits)
- `git rev-parse --git-dir`/state files: check for `MERGE_HEAD`, `REBASE_HEAD`, `CHERRY_PICK_HEAD`
- `git diff --stat` (size of changes)

Categorise:
- **Clean**: no porcelain output, no unpushed commits → nothing to do in Step 3, may still need Step 4 if upstream is ahead.
- **Modified-only**: only tracked files changed → safe candidate for auto-commit.
- **Untracked present**: new files exist → require user judgment (some may be secrets, build artifacts, scratch).
- **In-progress git operation**: `MERGE_HEAD`/`REBASE_HEAD`/etc. exists → STOP. Tell the user, do not commit.

## Step 2.5 — Journal the session recap (cwd-aware)

Runs **before** Step 3 so the journal entry rides the same commit + push. The journal is resolved from the **current repo** (cwd → repo root), reusing `/ai-brain:save`'s convention:

- Journal path = `<repo_root>/journal/<today>.md` **only if** a `journal/` folder already exists at the repo root.
- If there is no `journal/` folder → **skip this step silently**. Never create the folder. (This is what makes it cwd-aware: rakam/atlanticlog/aid/la-pelucherie have one; the ai-brain vault and opt-out repos don't, so they're skipped.)

**Gate — journal only when the session did real work:**
- Real work = uncommitted changes, unpushed commits, **or** the conversation produced substantive decisions / investigations / completed subtasks.
- Trivial or no-op session (nothing changed, nothing of substance discussed) → skip silently.

**Write the entry (auto-write, but show it):**
1. `today=$(date +%Y-%m-%d)`, `now=$(date +%H:%M)`.
2. If `journal/<today>.md` does **not** exist, create it with a header `# <today> — <Label>`. Derive `<Label>` by copying the `# <date> — <Label>` suffix from the most recent existing file in the same `journal/` (keeps the project's exact label casing, e.g. `Rakam`). If the folder is empty, fall back to the repo name.
3. Append a section at the end of the file:
   ```markdown

   ### Session <now>
   - <3–5 bullets: decisions taken, what moved forward, blockers — drawn from THIS conversation, in the same register as the existing hand-written sections>
   ```
   Bullets capture the session's **substance**, not the commit subject. Concrete over generic ("Posté le post-mortem delivery à Roman, discussion lundi" — not "worked on docs"). Respect the vault's content boundaries: only what actually happened this session.
4. **Display the drafted entry in the chat**, then write it — no confirmation prompt.

The file written here is staged automatically in Step 3 (even if brand-new) — it is skill-authored and always safe.

## Step 3 — Auto-commit + push (with safety checks)

Skip this step entirely if Step 2 saw a clean tree **and** Step 2.5 wrote no journal entry. If Step 2.5 wrote/created a journal file, treat the tree as non-clean and proceed (the journal change must be committed + pushed). Otherwise apply the gate:

**Block auto-commit (stop and report) if any of:**
- In-progress merge/rebase/cherry-pick (from Step 2).
- Detached HEAD (no `branch`).
- Diff > 500 lines (suspiciously large — ask the user before proceeding).
- Untracked files matching any of: `*.env`, `*.env.*` (except `.env.example`), `*credential*`, `*secret*`, `*.key`, `*.pem`, `id_rsa*`, `id_ed25519*`. **Never stage these.**
- An untracked file inside a path that already contains tracked files of the same kind (likely a forgotten add) — surface to user, don't auto-stage.
- Pre-commit hook fails on the first commit attempt → STOP, report, do **not** retry with `--no-verify`.

**When safe, proceed:**

1. Stage only **modified tracked files**: `git add -u`. Do not use `git add -A` or `git add .`. **Exception:** always stage the journal file written in Step 2.5 via `git add <journal-path>`, even if it is a brand-new untracked file — it is skill-authored and exempt from the untracked-file prompt below.
2. For each *other* untracked file that passed the gate (no secret pattern, not in a "forgotten add" cluster), ask the user once with the full list before staging — or skip them entirely if the user has not addressed them.
3. Draft a commit message from `git diff --cached`:
   - Subject: imperative, ≤72 chars, derived from the dominant change.
   - If `$ARGUMENTS` is non-empty, append it as a body paragraph.
   - Footer: `Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>`.
4. Commit via heredoc (preserve formatting).
5. If a pre-commit hook fails → fix-and-retry is **out of scope** for wrap-up. Report the failure and exit without push. Never `--amend` someone else's commit.
6. If `upstream` exists, `git push`. If the branch has no upstream, ask once whether to set it (`git push -u origin <branch>`) or skip push.
7. If push fails because the branch has diverged: `git pull --rebase` then retry push **only if** the rebase had no conflicts. On conflict, stop and report.

## Step 3.5 — Merge worktree session-* into main (if IS_WORKTREE_SESSION)

Skip this step entirely if `IS_WORKTREE_SESSION=false`.

When `IS_WORKTREE_SESSION=true`:

1. Resolve the main worktree path dynamically (works for any repo, not just ai-brain) and cd there:
   ```bash
   MAIN_WORKTREE="$(git worktree list --porcelain | awk '/^worktree / {print $2; exit}')"
   cd "$MAIN_WORKTREE"
   ```
   The first `worktree` entry in `git worktree list --porcelain` is always the main worktree.
2. `git pull --rebase --autostash origin main` — récupère les éventuels merges d'autres sessions (Mac/Nexus). `--autostash` met de côté les modifs/untracked non liés (digests email cron, claudius/, etc.) avant le rebase et les restitue après. Sans ça, le rebase refuse de tourner sur working tree dirty.
   - If rebase **conflict** → STOP. Report to user with the conflicting files. Do not auto-resolve. The user must resolve manually before re-running wrap-up.
   - If `--autostash` pop **conflict** after rebase → STOP, report. The session merge has not happened yet; the user resolves the pop conflict in main, then re-runs wrap-up.
3. `git merge --no-ff "$SESSION_BRANCH" -m "merge: $SESSION_BRANCH"`.
   - If merge **conflict** → STOP. Surface the conflicting files. The session branch (and the worktree) is preserved so the user can resolve and re-run.
4. `git push origin main`.
   - If push fails (divergence after the local pull-rebase) → STOP, report. Do not force.
5. `git worktree remove "$WORKTREE_PATH"`.
6. `git branch -d "$SESSION_BRANCH"` (safe delete — succeeds only if merged).
7. Capture for the final summary: `Worktree: $SESSION_BRANCH merged into main, removed, branch deleted.`

On any failure in steps 2-6, leave the state as-is and report exactly which step failed. The user can clean up manually via:
- `git worktree remove --force <path>` + `git branch -D <branch>`

## Step 4 — Sync (pull-based, no SSH)

Sync between Mac and Nexus is **pull-based only**: the push in Step 3 makes the work available on `origin`; the other machine pulls itself.

- For `dotfiles`: a SessionStart hook auto-pulls on the next session.
- For other repos: the user pulls manually at the start of the next session on the other machine, or relies on whatever hook that repo defines.

This skill **never SSHes** to trigger a pull elsewhere. It does not assume the other machine is reachable. If Step 3 succeeded, sync is considered complete from this skill's perspective.

In the Step 6 summary, mention which machine still needs a manual pull (e.g., "Nexus will need `git pull` on next session" or "Mac will need `git pull` on next session").

## Step 5 — Checkpoint decision (detect, don't auto-create)

Evaluate whether a checkpoint via `/ai-brain:save` would be useful. Score signals:

**Signals FOR a checkpoint:**
- TaskList shows tasks in `pending` or `in_progress` status (call TaskList).
- The repo has a `journal/` folder and the user mentioned plans for "tomorrow" / "next session" in this conversation.
- The conversation contained non-trivial decisions or investigations that aren't fully captured in the commit (e.g., "we decided X because Y", "I tried Z and it didn't work").
- A `scratchpad/in-progress.md` file exists in the repo with active items.
- The session spanned multiple distinct subtasks or hit blockers.

**Signals AGAINST:**
- Everything is committed, pushed, synced. No pending tasks. Session goal was achieved end-to-end.
- The conversation was a single linear edit with no surrounding context worth preserving.

**Decision:**
- If signals FOR clearly outweigh AGAINST → ask the user once: `"Checkpoint utile ? [Y/n] — signaux : <1-line summary of why>"`. If yes, instruct: "Lance `/ai-brain:save` pour créer le checkpoint."
- If unclear → ask the same question without a default, list the signals.
- If clean session → skip silently. Do not invoke `/ai-brain:save` automatically.

Never call `/ai-brain:save` from this skill directly. The user decides and runs it.

Note: the session recap is **already journaled** in Step 2.5. If the user then runs `/ai-brain:save`, its journal step only appends a short checkpoint-pointer line — complementary to the recap, not a duplicate.

## Step 6 — Final summary

Output a compact report:

```
✓ wrap-up complete
  Repo:    <repo_name> (<branch>)
  Machine: <mac|nexus>

  Journal: appended to journal/<today>.md  | skipped (no journal/ folder)  | skipped (no real work)
  Commit:  <hash> "<subject>"  | none (clean tree)
  Push:    pushed to origin/<branch>  | skipped (<reason>)
  Worktree: <SESSION_BRANCH> merged into main, removed, branch deleted  | (n/a — not a worktree session)
  Other machine: pull needed on <mac|nexus> at <path>  | up-to-date (nothing pushed)

  Checkpoint: proposed | not needed
```

Adapt fields to what actually happened. If a step was blocked (Step 3 safety gate, push conflict, Nexus unreachable), make the blocker visible in the summary.

</process>

<constraints>
- **Never `git add -A` or `git add .`** — always stage tracked-modified (`git add -u`) plus explicitly listed untracked files the user approved.
- **Never bypass hooks** (`--no-verify`, `--no-gpg-sign`). On hook failure, stop and report.
- **Never `--amend`** existing commits. New work = new commit.
- **Never force-push.** If a push is rejected non-fast-forward, do a `git pull --rebase` and retry once. On conflict, stop.
- **Never auto-create the checkpoint.** Step 5 only proposes; the user runs `/ai-brain:save`.
- **Journal only into an existing `journal/` folder.** Never create the folder — repos opt in by having it. No `journal/` → skip Step 2.5 silently. Auto-write the recap (no prompt) but always show it in the chat first.
- **One repo per invocation.** Do not iterate.
- **No SSH at all.** Sync is pull-based; `git push` on the current machine, the other machine pulls itself.
- **Secrets gate is non-negotiable.** If a candidate file matches the secret patterns, refuse to stage even if the user insists in the same turn — they should rename or move it first.
- **Worktree merge is non-destructive.** On any conflict or push failure in Step 3.5, leave the worktree and `session-*` branch intact so the user can resolve manually. Never `--force` the cleanup. Never bypass the merge.
</constraints>
