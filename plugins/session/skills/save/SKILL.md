---
name: session:save
description: Save current conversation context to a scratchpad file for resumption after /clear
allowed-tools:
  - Write
  - Read
  - Bash
  - AskUserQuestion
---

# Save Context

Save the current conversation state so it can be restored after `/clear`.

## Instructions

1. **Gather state** by reviewing the conversation. Identify:
   - What the user was working on (task/goal)
   - Key decisions made
   - What's been completed so far
   - What the next step is
   - Any important data, file paths, IDs, or references discussed

2. **Ask the user** if there's anything else to capture that you might have missed.

3. **Write the file** to `_scratchpad/context.md` in the current working directory:

```markdown
# Session Context — {YYYY-MM-DD HH:MM}

## Task
{What the user was working on}

## Completed
{What's been done so far}

## Decisions
{Key decisions made during the session}

## Next Steps
{What to do next}

## Key References
{File paths, IDs, URLs, data, or anything needed to continue}
```

4. **Print this message** after saving:

```
Context saved to _scratchpad/context.md
After /clear, run: /session:resume
```
