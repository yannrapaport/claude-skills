---
name: session:stash
description: Ephemeral save of current session context before /clear — use session:commit for persistent cross-machine checkpoints
allowed-tools:
  - Write
  - Read
  - Bash
---

# session:stash

Save the current conversation state ephemerally — to be restored immediately after `/clear` in the same session.

**Use this when:** you need to `/clear` context and resume right after.
**Use `session:commit` instead when:** you want to pause work and resume later, possibly on another machine.

## Instructions

1. **Gather state** by reviewing the conversation. Identify:
   - What the user was working on (task/goal)
   - Key decisions made
   - What's been completed so far
   - What the next step is
   - Any important data, file paths, IDs, or references discussed

2. **Write the file** to `_scratchpad/context.md` in the current working directory:

```markdown
# Session Stash — {YYYY-MM-DD HH:MM}

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

3. **Print this message** after saving:

```
Stash sauvegardé dans _scratchpad/context.md
Après /clear, lance : session:pop

(Pour un checkpoint persistant cross-machine : session:commit <nom>)
```
