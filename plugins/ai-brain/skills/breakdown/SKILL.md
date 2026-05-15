---
name: ai-brain:breakdown
description: Break an objective into tasks (3-7)
argument-hint: "[objective]"
allowed-tools:
  - Read
  - Edit
  - Grep
  - AskUserQuestion
---
<objective>
Break an objective into 3-7 actionable tasks and add them to active-todos.md.
</objective>

<context>
**Vault root resolution:**
- If cwd matches `$HOME/.cache/ai-brain-worktrees/session-*` → vault root = cwd.
- Otherwise → vault root = `$HOME/ai-brain`.

`<vault>` below refers to whichever root applies.

Assistant instructions: `<vault>/assistants/todos.md`
Data file: `<vault>/todos/active-todos.md`

Objective: $ARGUMENTS
</context>

<process>
1. Read the Plan Protocol, Activity Labels, and Task Format sections from `<vault>/assistants/todos.md`
2. Read `<vault>/todos/active-todos.md` to understand current tasks and structure
3. Propose 3-7 tasks as a numbered list with activity, importance, and horizon. If the objective needs more than 7 tasks, suggest using GSD instead.
4. Wait for user to validate or adjust (one round max)
5. Add confirmed tasks to active-todos.md in their respective horizon sections
</process>
