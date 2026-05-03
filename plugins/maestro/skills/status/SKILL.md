---
name: maestro:status
description: "[DEPRECATED] Check clawdbot container status (container n'existe plus, voir nanoclaw.service)"
allowed-tools:
  - Bash
---
<deprecated>
⚠️ **CETTE SKILL EST OBSOLÈTE depuis 2026-04-28.**

Il n'y a plus de container `clawdbot` à monitorer. La stack tourne maintenant sous systemd user service. Pour vérifier l'état :

```bash
ssh nexus "systemctl --user status nanoclaw.service && tail -20 ~/projects/tpg/nanoclaw/logs/nanoclaw.log"
```

À reconstruire pour cibler NanoClaw.
</deprecated>

<objective>
Check the status of the Maestro (clawdbot) container running on Nexus.
</objective>

<process>
1. Run `ssh nexus "docker ps --filter name=clawdbot --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"` to show container status.
2. Get the actual container name from step 1 (e.g. `claudius-clawdbot-1` in compose v2). Run `ssh nexus "docker logs <container-name> --tail 10 2>&1"` to show recent logs.
3. Present a summary: container running/stopped, uptime, port mapping.
</process>
