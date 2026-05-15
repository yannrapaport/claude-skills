---
name: ai-brain:health-check
description: Exhaustive audit of the AI Brain system — PASS/WARN/FAIL report with auto-created tasks
allowed-tools:
  - Read
  - Edit
  - Bash
  - Grep
  - Glob
  - mcp__claude_ai_Notion__*
---
<objective>
Run a full health check across the AI Brain system. Produce a structured PASS/WARN/FAIL report and append tasks for every finding to active-todos.md.
</objective>

<context>
**Vault root resolution:**
- If cwd matches `$HOME/.cache/ai-brain-worktrees/session-*` → vault root = cwd (`aib` session).
- Otherwise → vault root = `$HOME/ai-brain`.

`<vault>` below refers to whichever root applies.

All vault paths are relative to `<vault>/` (symlink to the Obsidian vault when on main). The vault is flat: top-level dirs are `assistants/`, `todos/`, `context/`, `knowledge/`, `projects/`, `scratchpad/`, `tools/`, `infra/`, `templates/`, `archive/`, `checkpoints/`, `claudius/`.

Commands directory: `$HOME/.claude/commands/`. Most former slash commands were migrated to plugin skills in `$HOME/.claude/plugins/cache/tpg-skills/`.
</context>

<process>
Run ALL four domains in order. Do not halt on individual check failures — complete every domain regardless of findings. After all domains complete, produce the structured report, then append tasks for any WARN or FAIL findings.

---

## Domain 1: Assistant Instruction Lint

**Files to scan:**
- `assistants/todos.md`

**Changelog exemption (CRITICAL):** Before applying any lint rules, identify the `## Changelog` section in each file. Skip lint rules LINT-01 through LINT-04 for any line within a changelog section.

**Archive exemption:** Skip all lint rules for files under `archive/` — those are historical.

**Lint rules (apply to prose body only, not changelog):**

| Rule | Pattern | Severity | Notes |
|------|---------|----------|-------|
| LINT-01 | `meta-assistant/`, `system/`, or `data/` path prefix anywhere in text | WARN | Vault was flattened — top-level dirs only (`assistants/`, `todos/`, `context/`, …) |
| LINT-02 | `notion-sync.md` referenced as a live file | WARN | File deleted; only changelog references expected (exempted) |
| LINT-03 | `~/iCloud Drive/` paths | WARN | Storage migrated to ProtonDrive |
| LINT-04 | Tool names not matching `mcp__claude_ai_*` pattern (e.g. `notion_retrieve_database`, `slack_get_user`) | WARN | Should use real Claude.ai MCP tool names |
| LINT-05 | `[[wiki-links]]` pointing to files that do not exist in the vault | WARN | Check existence |
| LINT-06 | References to any vault file path that no longer exists on disk (e.g. `assistants/todo-manager.md` when actual is `assistants/todos.md`) | FAIL | Hard broken reference |

For each match: record file name, line number, matched text, and rule ID.

**Extended scope (informational only — no tasks created):** Briefly scan these files for LINT-01 patterns:
- `context/goals.md`
- `context/core.md`
- `context/principles.md`

**Operational exception:** scan `todos/rituals.md` with full LINT-01 + LINT-06 enforcement. If any stale-prefix or broken-reference match → DO create tasks (operational file, stale paths mean rituals silently fail).

---

## Domain 2: Plugin Dependency Inventory

**Files to read:**
- `.obsidian/community-plugins.json`
- `.obsidian/core-plugins.json`
- Manifests in `.obsidian/plugins/*/manifest.json`

**For each community plugin, report:** Name, version, enabled/disabled, workflow dependency, impact if removed.

**Known workflow dependency map:**

| Plugin | Dependency | Impact if Removed |
|--------|-----------|-------------------|
| obsidian-git | Vault version history via UI | Lose UI git; Claude Code bash still works |
| obsidian-local-rest-api | None currently (Obsidian MCP removed) | No impact |
| obsidian-excalidraw-plugin | None | No impact |

**Check logic:**
- ENABLED + has dependency → PASS
- ENABLED + no dependency → WARN (potential bloat)
- DISABLED + has dependency → FAIL
- Unknown plugin (not in map) → WARN (ask user whether it's load-bearing)

---

## Domain 3: System Integrity

1. **Assistant files exist:** `assistants/todos.md`. FAIL if missing.
2. **Data files exist:** `todos/active-todos.md`, `todos/weekly-schedule.md`, `todos/rituals.md`. FAIL if any missing. `todos/notion-sync-index.json` → FAIL if missing, but see check #3.
3. **Notion sync status:**
   - Read `last_sync` from `todos/notion-sync-index.json`. If >24h → WARN ("stale sync").
   - Attempt `mcp__claude_ai_Notion__notion-fetch` with the `notion_database_id` from the index. If 404/unreachable → FAIL ("sync target lost — decide resume-or-abandon").
4. **Plugin skill availability:** verify the `ai-brain` plugin is discoverable — check `$HOME/.claude/plugins/cache/tpg-skills/ai-brain/*/skills/health-check/SKILL.md` exists. WARN if missing.

---

## Domain 4: Data File Cross-References

**Check:** Read `todos/rituals.md`. Scan for:
- LINT-01 stale prefixes (`meta-assistant/`, `system/`, `data/`)
- LINT-06 broken vault references (e.g. a link to `assistants/todo-manager.md` that no longer exists)

These are operational paths — stale paths mean rituals silently fail.

- LINT-01 match → WARN, create task
- LINT-06 match → FAIL, create task
- Report findings with line numbers

---

## Report Format

```markdown
# Health Check Report — {YYYY-MM-DD}

## Summary

| Domain | Status | Findings |
|--------|--------|----------|
| 1. Assistant Lint | PASS/WARN/FAIL | N issue(s) |
| 2. Plugin Inventory | PASS/WARN/FAIL | N issue(s) |
| 3. System Integrity | PASS/WARN/FAIL | N issue(s) |
| 4. Data File Cross-References | PASS/WARN/FAIL | N issue(s) |

**Overall:** PASS / WARN / FAIL

---

## Domain 1: Assistant Lint
[Findings per file, or "No issues found"]

## Domain 2: Plugin Inventory
[Table of plugins with status]

## Domain 3: System Integrity
[Check results]

## Domain 4: Data File Cross-References
[Findings or "No stale references found"]

---

## Tasks Created
{List each task appended, or "None — all checks passed"}
```

**Status logic:** FAIL if any domain FAIL; WARN if any WARN and none FAIL; PASS if all PASS.

---

## Task Creation Rules

1. Collect all WARN/FAIL findings.
2. For each finding:
   - WARN → `- [ ] [health-check] {description} #ai-brain !should`
   - FAIL → `- [ ] [health-check] {description} #ai-brain !must`
3. **Deduplication:** Search `todos/active-todos.md` for existing `[health-check]` tasks with matching key phrase. Skip if found.
4. **Append location:** After the last `#ai-brain` line in `todos/active-todos.md`.
5. If zero findings: "None — all checks passed."
</process>
