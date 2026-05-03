---
name: maestro:help
description: "[DEPRECATED] Show available Maestro commands (toutes obsolètes depuis 2026-04-28)"
allowed-tools: []
---
## Maestro Commands — ⚠️ TOUTES DEPRECATED

Les commandes `/maestro:*` ciblaient le container Docker `claudius-clawdbot-1` qui n'existe plus depuis 2026-04-28. La stack WhatsApp tourne maintenant sous **NanoClaw v2** + persona **Herbert** (`systemctl --user status nanoclaw.service`).

| Command | État |
|---------|------|
| `/maestro:send [msg]` | ⚠️ Cassée — gateway clawdbot supprimé |
| `/maestro:send-raw [msg]` | ⚠️ Cassée — container n'existe plus |
| `/maestro:contacts` | ⚠️ Cassée — `clawdbot.json` plus lu |
| `/maestro:config` | ⚠️ Cassée — `clawdbot.json` plus lu |
| `/maestro:deploy` | ⚠️ Cassée — `/docker/claudius/` legacy |
| `/maestro:status` | ⚠️ Cassée — pas de container à monitorer |

### En attendant la reconstruction des skills NanoClaw

- Status : `ssh nexus "systemctl --user status nanoclaw.service && tail -20 ~/projects/tpg/nanoclaw/logs/nanoclaw.log"`
- Envoi WhatsApp proactif depuis local : pas de mécanisme clean. Demander à Yann d'envoyer manuellement, ou ajouter une tâche taguée dans `~/ai-brain/todos/active-todos.md` (Radar la remontera)
- Configs groupes : `~/projects/tpg/nanoclaw/groups/whatsapp_*/container.json`

Doc : `~/ai-brain/tools/maestro-radar.md`
