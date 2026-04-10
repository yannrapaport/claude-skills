---
name: pm-copilot:weekly-report
description: Generate a weekly status report — auto-pull ClickUp accomplishments, ask user for blockers and plans, post to Slack and Notion
---

# /pm-copilot:weekly-report — Generate a weekly report

You are generating a weekly status report for the user. The report pulls accomplishments automatically and asks the user for blockers and plans.

## Input

Optional `--profile <name>` to override auto-detection. Otherwise, detect from the current working directory or ask.

## Step 1: Load the profile

Read profiles to determine which customer this report is for:
```bash
ls ~/projects/tpg/pm-copilot/profiles/*.yaml
```

The profile tells you which ClickUp space/lists to query.

## Step 2: Pull accomplishments from ClickUp

Using the ClickUp MCP tools, query for tasks completed or updated this week:

1. Get the current week's date range (Monday to today)
2. Query tasks updated this week across the profile's tracked lists
3. Filter for tasks that were completed (status changed to done/closed) or significantly updated
4. Group by list/project for readability

Present the auto-pulled accomplishments to the user:
> "Here's what I found from ClickUp this week. Want to add, remove, or edit anything?"

## Step 3: Gather blockers and next week

Ask the user:
1. **Blockers:** "Any blockers or risks to flag?"
2. **Next week:** "What are your priorities for next week?"
3. **Highlights:** "Anything else to call out? (decisions made, dependencies, escalations)"

## Step 4: Assemble the report

Build two versions:

### Slack summary (concise)
```
*Weekly Update — {Customer} — {date range}*

*Done:*
• {accomplishment 1}
• {accomplishment 2}

*Blockers:*
• {blocker 1}

*Next week:*
• {priority 1}
• {priority 2}
```

### Notion full page (detailed)
A structured page with sections:
- **Accomplishments** — with links to ClickUp tasks
- **Blockers & Risks** — with context and impact
- **Next Week** — priorities and expected outcomes
- **Decisions & Notes** — anything the user flagged

## Step 5: Review and send

Show both versions to the user for approval.

On approval:
1. **Slack:** Post the summary to the configured Slack channel (using Slack MCP if available, or tell the user to paste it)
2. **Notion:** Create the full page in the configured Notion database (using Notion MCP)

## Step 6: Confirm

Tell the user what was posted and where, with links.
