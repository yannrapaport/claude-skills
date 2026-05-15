---
name: ai-brain:daily
description: Run your evening ritual
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - Task
  - mcp__claude_ai_Notion__*
  - mcp__claude_ai_Slack__*
---
<objective>
Run the daily ritual (Steps 0 through 7) from the todo-manager assistant instructions.
</objective>

<context>
**Vault root resolution (do this FIRST, mentally):**
- If current cwd matches `$HOME/.cache/ai-brain-worktrees/session-*`, the vault root is the cwd (`aib` worktree session).
- Otherwise, the vault root is `$HOME/ai-brain`.

`<vault>` below refers to whichever root applies.

Assistant instructions: `<vault>/assistants/todos.md`

Data files:
- Active todos: `<vault>/todos/active-todos.md`
- Weekly schedule: `<vault>/todos/weekly-schedule.md`
- Rituals: `<vault>/todos/rituals.md`
- Sync index: `<vault>/todos/notion-sync-index.json`
</context>

<process>
1. Read the assistant instructions from `<vault>/assistants/todos.md`
2. Execute the **Daily Ritual** protocol (Steps 0 through 7) exactly as documented
3. Step 0: Pre-flight health checks (Notion, Slack, sync index)
4. Steps 1-4: Review, celebrate, learn, handle incomplete tasks (ALWAYS ask user)
5. Step 4.5: Notion sync (if Notion available)
6. Steps 5-6: Plan tomorrow, transition Today's Focus
7. Step 7: Send to Slack (if Slack available)
</process>
