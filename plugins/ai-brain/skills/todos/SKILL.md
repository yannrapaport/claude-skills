---
name: ai-brain:todos
description: Display current active todos
allowed-tools:
  - Read
---
<objective>
Display the current contents of active-todos.md in a readable format.
</objective>

<context>
**Vault root resolution:**
- If cwd matches `$HOME/.cache/ai-brain-worktrees/session-*` → vault root = cwd.
- Otherwise → vault root = `$HOME/ai-brain`.

Data file: `<vault>/todos/active-todos.md`
</context>

<process>
1. Read `<vault>/todos/active-todos.md` (resolved per <context>).
2. Display the full contents to the user.
</process>
