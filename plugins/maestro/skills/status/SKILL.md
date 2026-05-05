---
name: maestro:status
description: Show NanoClaw service health on Nexus — systemd unit state, recent logs, active sessions, socket reachability. Use when verifying that Herbert is alive before sending a message, or troubleshooting a drop.
allowed-tools:
  - Bash
---

## What this skill does

Reports the health of the NanoClaw stack on Nexus :
- systemd user service state (`nanoclaw.service`)
- main process uptime and memory
- recent log tail (last 20 lines)
- CLI socket reachability (the surface used by `/maestro:send`)
- active container sessions (= ongoing agent invocations)

## Procedure

Run all checks in a single SSH round-trip and show the consolidated output to the user :

```bash
ssh nexus 'echo "=== service ===" && systemctl --user status nanoclaw.service --no-pager 2>&1 | head -10 && echo && echo "=== socket ===" && ls -la ~/projects/tpg/nanoclaw/data/cli.sock 2>&1 && echo && echo "=== active containers ===" && docker ps --filter name=nanoclaw-v2 --format "table {{.Names}}\t{{.Status}}" 2>&1 && echo && echo "=== recent logs ===" && tail -20 ~/projects/tpg/nanoclaw/logs/nanoclaw.log 2>&1'
```

## Reading the output

- **Service `Active: active (running)`** = OK. `Active: failed` or `inactive` = needs `systemctl --user restart nanoclaw.service`.
- **Socket** : the file should exist (`srw-------`) with a recent mtime. If it's old or missing while service is running, restart (stale socket bug).
- **Active containers** : 0-N rows. Each row = one agent session currently running. Long-running ones may be hung (kill via `docker kill <name>` if needed).
- **Logs** : look for `Message delivered` (good), `MESSAGE DROPPED` (sender access denied), `Container exited code=137` (killed by absolute-ceiling timeout — expected for long sessions), or stack traces.

## Common follow-ups

- "Herbert ne répond pas" → check `MESSAGE DROPPED` in logs. If `accessReason="not_member"` on a strict group, the sender lacks the owner role (see `~/ai-brain/tools/maestro-radar.md`).
- "Le service est down" → `ssh nexus 'systemctl --user restart nanoclaw.service'` (≈5 sec interruption).
- "Le socket est stale" (LISTEN but `ECONNREFUSED`) → same restart fixes it.

## Related

- `/maestro:send`, `/maestro:list-groups`
- Doc : `~/ai-brain/tools/maestro-radar.md`
