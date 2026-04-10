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
All vault paths are relative to `/Users/yannrapaport/ai-brain/` (symlink to the Obsidian vault).

Commands directory: `/Users/yannrapaport/.claude/commands/`
</context>

<process>
Run ALL four domains in order. Do not halt on individual check failures — complete every domain regardless of findings. After all domains complete, produce the structured report, then append tasks for any WARN or FAIL findings.

---

## Domain 1: Assistant Instruction Lint

**Files to scan:**
- `system/assistants/todo-manager.md`

**Changelog exemption (CRITICAL):** Before applying any lint rules, identify the `## Changelog` section in each file. Skip lint rules LINT-01 through LINT-04 for any line within a changelog section.

**Lint rules (apply to prose body only, not changelog):**

| Rule | Pattern | Severity | Notes |
|------|---------|----------|-------|
| LINT-01 | `meta-assistant/` path prefix anywhere in text | WARN | Correct prefix is `system/`, `data/`, or `scripts/` |
| LINT-02 | `notion-sync.md` referenced as a live file | WARN | File deleted; only changelog references expected (exempted) |
| LINT-03 | `~/iCloud Drive/` paths | WARN | Storage migrated to ProtonDrive |
| LINT-04 | Tool names not matching `mcp__claude_ai_*` pattern (e.g. `notion_retrieve_database`, `slack_get_user`) | WARN | Should use real Claude.ai MCP tool names |
| LINT-05 | `[[wiki-links]]` pointing to files that do not exist in the vault | WARN | Check existence |
| LINT-06 | References to any file in `system/` that no longer exists on disk | FAIL | Hard broken reference |

For each match: record file name, line number, matched text, and rule ID.

**Extended scope (informational only — no tasks created):** Briefly scan these files for LINT-01 patterns:
- `system/context/goals.md`
- `system/activities/ai-brain/plan.md`
- `data/todos/rituals.md` — EXCEPTION: if LINT-01 found here, DO create tasks (operational data file).

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

---

## Domain 3: System Integrity

1. **Assistant files exist:** `system/assistants/todo-manager.md`. FAIL if missing.
2. **Data files exist:** `data/todos/active-todos.md`, `data/todos/weekly-schedule.md`, `data/todos/rituals.md`, `data/todos/notion-sync-index.json`. FAIL if any missing.
3. **Sync index freshness:** Read `last_sync` from `data/todos/notion-sync-index.json`. >24h → WARN.
4. **Command files exist:** Verify `/Users/yannrapaport/.claude/commands/todo-manager/` contains `daily.md`, `weekly.md`, `monthly.md`, `show.md`, `add.md`, `sync.md`. Verify `/Users/yannrapaport/.claude/commands/ai-brain/health-check.md` exists. WARN if any missing.
5. **MCP connectivity:** Attempt `mcp__claude_ai_Notion__notion-fetch` with `id: "5d941771-1135-4b0f-a0f9-e8580f46f786"`. WARN if unreachable.

---

## Domain 4: Data File Cross-References

**Check:** Read `data/todos/rituals.md`. Scan for `meta-assistant/` path prefix references (LINT-01). These are operational paths — stale paths mean rituals silently fail.

- If found → WARN, create tasks
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
3. **Deduplication:** Search `data/todos/active-todos.md` for existing `[health-check]` tasks with matching key phrase. Skip if found.
4. **Append location:** After the last `#ai-brain` line in `data/todos/active-todos.md`.
5. If zero findings: "None — all checks passed."
</process>
