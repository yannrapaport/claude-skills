---
name: session:resume
description: Resume work from a previously saved session context
allowed-tools:
  - Read
  - Bash
---

# Resume Context

Restore conversation state from a previously saved scratchpad file.

## Instructions

1. **Read** `_scratchpad/context.md` in the current working directory.
   - If the file doesn't exist, tell the user: "No saved context found. Nothing to resume."
   - If it exists, continue.

2. **Present** a brief summary of the restored context:
   - What you were working on
   - Where you left off
   - What the next step is

3. **Delete** the file after loading: `rm _scratchpad/context.md`. Remove the `_scratchpad/` directory too if it's empty.

4. **Ask** the user: "Ready to continue from where you left off?"
