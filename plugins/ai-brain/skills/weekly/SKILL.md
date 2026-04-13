---
name: ai-brain:weekly
description: Run your weekly end ritual
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
Run the weekly ritual (Steps 1 through 6) from the todo-manager assistant instructions.
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
2. Execute the **Weekly Ritual** protocol (Steps 1 through 6) exactly as documented
3. Steps 1-3: Review week, celebrate, learn
4. Step 4: Scan monthly tasks for promotion candidates (CRITICAL — always check full backlog)
5. Step 5: Plan blocks — ask user for schedule in free text, then schedule blocks + floating blocks
6. Step 6: Update weekly-schedule.md and active-todos.md, transition Today's Focus to Monday, Notion sync
</process>
