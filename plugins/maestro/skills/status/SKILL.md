---
name: maestro:status
description: Check Maestro container status on Nexus
allowed-tools:
  - Bash
---
<objective>
Check the status of the Maestro (clawdbot) container running on Nexus.
</objective>

<process>
1. Run `ssh nexus "docker ps --filter name=clawdbot --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"` to show container status.
2. Get the actual container name from step 1 (e.g. `claudius-clawdbot-1` in compose v2). Run `ssh nexus "docker logs <container-name> --tail 10 2>&1"` to show recent logs.
3. Present a summary: container running/stopped, uptime, port mapping.
</process>
