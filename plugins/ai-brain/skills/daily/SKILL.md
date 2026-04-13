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
Assistant instructions: `/Users/yannrapaport/ai-brain/assistants/todos.md`

Data files:
- Active todos: `/Users/yannrapaport/ai-brain/todos/active-todos.md`
- Weekly schedule: `/Users/yannrapaport/ai-brain/todos/weekly-schedule.md`
- Rituals: `/Users/yannrapaport/ai-brain/todos/rituals.md`
- Sync index: `/Users/yannrapaport/ai-brain/todos/notion-sync-index.json`
</context>

<process>
1. Read the assistant instructions from `/Users/yannrapaport/ai-brain/assistants/todos.md`
2. Execute the **Daily Ritual** protocol (Steps 0 through 7) exactly as documented
3. Step 0: Pre-flight health checks (Notion, Slack, sync index)
4. Steps 1-4: Review, celebrate, learn, handle incomplete tasks (ALWAYS ask user)
5. Step 4.5: Notion sync (if Notion available)
6. Steps 5-6: Plan tomorrow, transition Today's Focus
7. Step 7: Send to Slack (if Slack available)
</process>
