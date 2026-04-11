---
name: pm-copilot:prd-sync
description: Sync PRD content (agent prompt + PRD docs) between local and remote — push, pull, or status. Backend configured per project in .pm-copilot/config.md (notion | confluence | …)
---

# /pm-copilot:prd-sync — Sync PRD content

Synchronizes two things:
- **Prompt** — `agents/prd-init/AGENT.md` ↔ remote platform page
- **Docs** — local PRD markdown files ↔ remote PRD pages

Remote backend is determined by `.pm-copilot/config.md` in the current workspace.

## Input

`push | pull | status` with optional scope flags `--prompt`, `--docs`, and optional file/name argument.

If no subcommand given, run `status`.

---

## Step 1 — Load config

```bash
# Walk up from cwd until found
cat .pm-copilot/config.md 2>/dev/null
```

Extract from `prd_sync` section:
- `backend` — `notion` (default) or other
- `prompt_page_id` — remote page for the agent prompt
- `prd_parent_page_id` — where PRD hubs live remotely
- `prd_database_id` — PRD sub-docs collection (Notion)

If no project config found: fall back to the global defaults in
`~/projects/tpg/pm-copilot/agents/prd-init/AGENT.md` header
(field `notion_page` for prompt sync).

---

## Subcommand: `push [--prompt] [--docs] [file]`

Default scope: both prompt and docs.

### Push prompt

1. Read `~/projects/tpg/pm-copilot/agents/prd-init/AGENT.md`
2. Get remote URL from `notion_page:` in the YAML header
3. Fetch current remote content
4. Show diff summary — what changed
5. **Ask for confirmation** before overwriting
6. Update remote page (`notion-update-page` with `replace_content`)
7. Update `last_synced_to_notion: <today>` in AGENT.md header

### Push docs

For each local PRD file with a `<!-- prd-sync: <url> -->` comment or `prd_sync:` frontmatter:

1. Read local file
2. Identify the remote page (from inline marker, or search by title if missing)
3. Fetch current remote content → show diff → ask for confirmation
4. Update via `notion-update-page`
5. Update `last_synced:` date in local file header

If a specific `file` argument is given: push only that file.

**Push does not create** — creation is handled by `/pm-copilot:prd-init`.
If a local file has no remote link, warn and skip.

---

## Subcommand: `pull [--prompt] [--docs] [name]`

Default scope: both prompt and docs.

### Pull prompt

1. Get remote URL from `notion_page:` in AGENT.md header (or from config)
2. Fetch remote page content
3. Show diff against current AGENT.md
4. **Ask for confirmation** before overwriting
5. Overwrite AGENT.md content (preserve YAML header)
6. Update `last_synced_to_notion: <today>` in header

### Pull docs

If a `name` argument is given: search remote by name and pull that PRD.
Otherwise: pull all PRD docs linked in local files.

For each:
1. Fetch remote page content
2. Show diff against local file (if exists)
3. Ask for confirmation
4. Write to local file, preserving `<!-- prd-sync: <url> -->` marker

---

## Subcommand: `status`

Show sync state for both prompt and all linked PRD docs.

1. **Prompt status:**
   - Read `last_synced_to_notion:` from AGENT.md header
   - Get `git log -1 --format="%ci"` for AGENT.md
   - Fetch remote page last-edited time
   - Report: local newer / remote newer / in sync

2. **Docs status:**
   - Find all local files with `<!-- prd-sync: -->` markers
   - For each: compare local git date vs remote last-edited time

3. **Report table:**

```
| Item           | Local modified | Remote modified | Status         |
|----------------|---------------|-----------------|----------------|
| AGENT.md       | 2026-04-11    | 2026-04-11      | ✅ In sync     |
| prd-val.md     | 2026-04-10    | 2026-04-08      | ← Local newer  |
| prd-client2.md | —             | 2026-04-05      | → Remote only  |
```

4. Suggest: `push --prompt`, `pull --docs prd-client2`, etc.

---

## Backend routing

```
backend: notion      → Notion MCP (notion-fetch, notion-update-page)
backend: confluence  → [not yet implemented — add when needed]
```

If backend is unsupported: tell the user and stop.
