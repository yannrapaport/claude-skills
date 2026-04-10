---
name: todo-manager:add
description: Add a task to active todos (with cross-project routing)
argument-hint: "[task description] [#project-tag] [!importance]"
allowed-tools:
  - Read
  - Edit
  - Write
  - Glob
  - Grep
---
<objective>
Add a new task with auto-detected activity label and importance. Route to the correct file (hub or project TODO.md) based on #tag, keyword match, or fallback. Add pilotage entry in hub when routing to a project.
</objective>

<context>
Assistant instructions: `/Users/yannrapaport/ai-brain/assistants/todo-manager.md`
Hub file: `/Users/yannrapaport/ai-brain/todos/active-todos.md`
Routing config: `~/.claude/todo-routing.yaml`

Task to add: $ARGUMENTS
</context>

<process>

## Step 0 — Validate input

If $ARGUMENTS is empty or blank, respond with:
```
Usage: /todo-manager:add [task description] [#project-tag] [!importance]

Examples:
  /todo-manager:add Review sprint PRD #rakam !must
  /todo-manager:add Fix deploy pipeline
  /todo-manager:add LinkedIn post !may
```
Then stop.

## Step 1 — Load routing config

Try to Read `~/.claude/todo-routing.yaml` (expand `~` to `/Users/yannrapaport`).

- If the file exists, parse it. Extract `hub`, `defaults`, and `projects` list.
- If the file does NOT exist, skip routing entirely — treat everything as hub (active-todos.md). Continue to Step 2.

## Step 2 — Parse input

Read the Activity Labels and Task Format sections from `/Users/yannrapaport/ai-brain/assistants/todo-manager.md`.

Parse $ARGUMENTS to extract:
- **Task description**: full text with any `#tag` and `!importance` removed, trimmed
- **Explicit #tag**: if input contains `#something`, extract it (case-insensitive)
- **Importance**: `!must`, `!should`, or `!may` (default `!should`)
- **Activity label**: auto-detect from keywords in todo-manager.md (e.g., "rakam sprint" -> Rakam). If ambiguous, ask.

## Step 3 — Route to target file

**Skip this step if routing config was not loaded (no YAML file).**

### Routing priority (compact version):

1. **Explicit #tag** — Match `#tag` against project `name` fields (case-insensitive). If no match, list valid tags and stop.
2. **Keyword match** — Scan task text against each project's `keywords`. Highest score wins. If tie between multiple projects, go to step 4 (disambiguation).
3. **No match / zero keyword score** — Go to step 4 (disambiguation). Do NOT silently fall back to hub.

### Disambiguation (when routing is uncertain):

Present the user with a numbered list of all known projects from the config, plus the hub as fallback:

```
I'm not sure where to route this task. Where should it go?

  1. rakam        (~/projects/tpg/rakam/TODO.md)
  2. atlanticlog  (~/projects/tpg/atlanticlog/TODO.md)
  3. aid          (~/projects/tpg/training/TODO.md)
  4. lp           (~/projects/la-pelucherie/TODO.md)
  5. maestro      (~/projects/maestro/TODO.md)
  → hub          (active-todos.md) — default if none fits
```

Wait for the user's answer. Accept a number, a project name, or "hub". Then route accordingly.

Record the chosen target: project (name, path, pilotage setting) or hub.

## Step 4 — Detect target file format

Expand `~` in the target path.

- If target file does NOT exist: check parent directory exists (Glob). If yes, ask to create it. If no, error and stop.
- If target file exists: Read the first 30 lines and classify:
  - **Sectioned checklist**: has `## ` headings AND `- [ ]`/`* [ ]` items
  - **Flat checklist**: has checklist items but no `## ` headings
  - **Empty/minimal**: no checklist items

Also detect checkbox style (`- [ ]` vs `* [ ]`). Default to `- [ ]`.

## Step 5 — Build the task line

**For hub (active-todos.md):**
Use the AI Brain format: `- [ ] Task description (Activity) !importance`

**For project TODO.md:**
Use the project's detected format. Do NOT add `(Activity)` or `!importance` unless the project file already uses them. Match the existing style.

## Step 6 — Write the task

Read the full target file.

**Sectioned checklist:**
- Use the configured `section` for this target (from YAML). If empty, find the first section with unchecked items. If none, use the last section.
- Append after the last item in that section using Edit.

**Flat checklist:**
- Append after the last checklist item using Edit.

**Empty/new file:**
- Use Write to set initial content.

## Step 7 — Pilotage (hub cross-reference)

**Skip if:** task was routed to hub, or routing config not loaded, or pilotage is false for this project (check project-level override, then `defaults.pilotage`).

If pilotage applies:
1. Read the hub file (`hub.path`).
2. Build pilotage entry: `- [ ] [project_name]: [task_text] -> voir [project_path]`
3. Append to hub using the same section logic (use `hub.section` if configured).
4. If hub write fails, report partial success.

## Step 8 — Report result

**Hub route:**
```
Added to active-todos.md:
[task_line]
```

**Project route (no pilotage):**
```
Added to [project_name] ([path]):
[task_line]
```

**Project route (with pilotage):**
```
Added to [project_name] ([path]):
[task_line]

Hub entry added (active-todos.md):
[pilotage_line]
```

</process>

<constraints>
- NEVER use the Bash tool. All file operations must use Read, Edit, Write, Glob, or Grep.
- Use Write ONLY for creating new files. Use Edit for all modifications to existing files.
- ALWAYS Read a file before using Edit on it.
- When pilotage applies: write to the project file FIRST, then the hub.
- Expand `~` to `/Users/yannrapaport` before any file operation.
- If routing config is missing, behave exactly like the original add command: write to active-todos.md with AI Brain format.
- Write scope: only write to paths listed in `hub.path` or `projects[].path` from the loaded config. Do not write elsewhere.
</constraints>
</output>
