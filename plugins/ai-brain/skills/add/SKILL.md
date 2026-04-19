---
name: ai-brain:add
description: Append a task to active-todos.md (hub) with #tag and !importance
argument-hint: "[task description] [#tag] [!importance]"
allowed-tools:
  - Read
  - Edit
---
<objective>
Append a task to the AI Brain hub (`active-todos.md`). No routing, no disambiguation, no project file. Just a clean, deterministic append.
</objective>

<context>
Hub file: `$HOME/ai-brain/todos/active-todos.md`
Format reference: `$HOME/ai-brain/assistants/todos.md`

Task to add: $ARGUMENTS
</context>

<process>

## Step 0 — Validate

If $ARGUMENTS is empty or blank, respond with:
```
Usage: /ai-brain:add [task description] [#tag] [!importance]

Examples:
  /ai-brain:add Review sprint PRD #rakam !must
  /ai-brain:add Fix deploy pipeline #tpg
  /ai-brain:add LinkedIn post #tpg !may
```
Then stop.

## Step 1 — Parse $ARGUMENTS

Extract:
- **#tag** : first `#word` token, lowercase. If absent, ask once: `"Quel tag ? (#tpg / #rakam / #atlanticlog / #aid / #lp / #maestro / #ai-brain / #perso)"`. Wait for answer.
- **importance** : `!must` / `!should` / `!may`. Default `!should` if not specified.
- **description** : everything else, trimmed (no `#tag`, no `!importance`).

## Step 2 — Pick section

Map `!importance` to a section under `## This Week`:
- `!must`   → `### Must`
- `!should` → `### Should`
- `!may`    → `### Unscheduled`

(No "smart" routing. No Today's Focus. Predictable destination based on importance.)

## Step 3 — Append

1. Read `$HOME/ai-brain/todos/active-todos.md`.
2. Locate the chosen section.
3. Find the last `- [ ]` line in that section (before the next `### ` or `## `).
4. Use Edit to insert a new line right after it:

```
- [ ] {description} #{tag} `!{importance}`
```

(Note the backticks around `!importance` — match the existing hub format.)

If the chosen section is missing (hub structure changed), report and stop. Don't fall back silently.

## Step 4 — Confirm

```
Added to active-todos.md (This Week → {Must|Should|Unscheduled}):
- [ ] {description} #{tag} `!{importance}`
```

</process>

<constraints>
- NEVER use Bash. Read + Edit only.
- ALWAYS Read the hub before Edit.
- Use the exact format `- [ ] description #tag `!importance`` with backticks around the importance.
- Do NOT touch project TODO.md files. Hub-only.
- Do NOT add to Today's Focus. That section is curated by the daily ritual.
</constraints>
