---
name: maestro:help
description: Show available Maestro commands and their current status (NanoClaw v2 era).
allowed-tools: []
---

## Maestro Commands

Stack actuelle : **NanoClaw v2** + persona **Herbert** (depuis 2026-04-28). Code : `~/projects/tpg/nanoclaw/`. Service : `systemctl --user status nanoclaw.service` sur Nexus.

| Command | État |
|---------|------|
| `/maestro:send <alias> <message>` | ✅ Opérationnel depuis 2026-05-05 (mode C1, via Herbert) |
| `/maestro:list-groups` | ✅ Opérationnel depuis 2026-05-05 |
| `/maestro:send-raw <alias> <message>` | ❌ Pas reconstruit — voir "Plan futur" dans `~/ai-brain/tools/maestro-radar.md` (C2) |
| `/maestro:status` | ⚠️ DEPRECATED — pointait vers le container clawdbot. À redéfinir sous NanoClaw. |
| `/maestro:deploy` | ⚠️ DEPRECATED — `/docker/claudius/` legacy, n'existe plus |
| `/maestro:contacts` | ⚠️ DEPRECATED — `clawdbot.json` plus lu |
| `/maestro:config` | ⚠️ DEPRECATED — `clawdbot.json` plus lu |

### Caveat C1 (`/maestro:send`)

Le message passé est un **prompt** que l'agent (Herbert / Coach Roger) reçoit comme inbound. L'agent répond avec sa persona — c'est cette réponse qui arrive dans WhatsApp, pas le texte brut. Pour copy-paste exact, attendre C2.

Pour les groupes avec `engage_pattern='[Hh]erbert\b'` (ex: `la-plushy-team`, `operation-summer-body`), inclure "Herbert" dans le prompt ou il sera silencieusement droppé.

### Diagnostic rapide

- Status service : `ssh nexus 'systemctl --user status nanoclaw.service'`
- Logs récents : `ssh nexus 'tail -30 ~/projects/tpg/nanoclaw/logs/nanoclaw.log'`
- Vérifier qu'un message est arrivé : grep `Message delivered` dans les logs ; pour un drop, grep `MESSAGE DROPPED`.

Doc complète : `~/ai-brain/tools/maestro-radar.md`
