---
name: session:pop
description: Restore and delete the ephemeral stash saved by session:stash
allowed-tools:
  - Read
  - Bash
---

# session:pop

Restore the ephemeral stash saved by `session:stash`. The stash file is deleted after loading.

## Instructions

1. **Read** `_scratchpad/context.md` in the current working directory.
   - If the file doesn't exist:
     ```
     Aucun stash trouvé. Rien à restaurer.
     (Tu cherches un checkpoint persistant ? Utilise : session:log)
     ```
     Stop.

2. **Present** a brief summary of the restored context:
   - What you were working on
   - Where you left off
   - What the next step is

3. **Delete** the stash file after loading:
   ```bash
   rm _scratchpad/context.md
   rmdir _scratchpad 2>/dev/null || true
   ```

4. **Ask** the user: "Prêt à reprendre ?"
