---
name: maestro:deploy
description: "[DEPRECATED] Push clawdbot.json + restart container (n'existe plus depuis 2026-04-28)"
allowed-tools:
  - Bash
---
<deprecated>
⚠️ **CETTE SKILL EST OBSOLÈTE depuis 2026-04-28.**

Le container `clawdbot` n'existe plus, `clawdbot.json` n'est plus lu, `/docker/claudius/` est legacy. NanoClaw se déploie autrement (push code vers `~/projects/tpg/nanoclaw/` + `systemctl --user restart nanoclaw.service`).

À reconstruire pour cibler NanoClaw.
</deprecated>

<objective>
Deploy the current Maestro configuration to Nexus and restart the container.
</objective>

<process>
1. Copy the config file to the Docker volume where Maestro actually reads it:
   ```bash
   scp ~/projects/maestro/infra/clawdbot.json nexus:/tmp/clawdbot-deploy.json
   ssh nexus "sudo cp /tmp/clawdbot-deploy.json /var/lib/docker/volumes/clawdbot-ownk_clawdbot_config/_data/clawdbot.json && rm /tmp/clawdbot-deploy.json"
   ```
   Also keep the copy at `/docker/claudius/` in sync:
   ```bash
   scp ~/projects/maestro/infra/clawdbot.json nexus:/docker/claudius/clawdbot.json
   ```
2. Restart the container:
   ```bash
   ssh nexus "cd /docker/claudius && docker compose restart"
   ```
3. Wait a few seconds, then check status:
   ```bash
   ssh nexus "docker ps --filter name=clawdbot --format 'table {{.Names}}\t{{.Status}}'"
   ```
4. Report success/failure.
</process>
</output>
