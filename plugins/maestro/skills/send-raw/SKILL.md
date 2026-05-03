---
name: maestro:send-raw
description: "[DEPRECATED] Send raw WhatsApp message via SSH (cibles container Docker claudius-clawdbot-1, n'existe plus depuis 2026-04-28)"
allowed-tools:
  - Bash
  - AskUserQuestion
---
<deprecated>
⚠️ **CETTE SKILL EST CASSÉE depuis 2026-04-28.**

Le container Docker `claudius-clawdbot-1` n'existe plus sur Nexus. La stack WhatsApp tourne maintenant sous **NanoClaw v2** (`systemctl --user status nanoclaw.service`, code à `~/projects/tpg/nanoclaw/`) avec le persona **Herbert**.

Aucun mécanisme CLI proactif local n'a encore été reconstruit pour pousser un message à un groupe via NanoClaw. **Ne pas exécuter cette skill** — au lieu de ça :
- Demander à Yann d'envoyer manuellement, ou
- Ajouter une tâche taguée dans `~/ai-brain/todos/active-todos.md` que Radar remontera

Doc : `~/ai-brain/tools/maestro-radar.md`
</deprecated>

<objective>
Send a raw WhatsApp message directly via the clawdbot CLI on Nexus, bypassing LLM processing.
</objective>

<process>
1. If no message was provided in the command arguments, ask the user for the message.
2. Ask the user for the recipient (phone number in international format, e.g. +33612345678).
3. Send the message via SSH. IMPORTANT: Use a heredoc or printf to avoid shell escaping issues (e.g. `!` becoming `\!`). Example:
   ```bash
   ssh nexus << 'REMOTEOF'
   docker exec claudius-clawdbot-1 npx clawdbot message send --channel whatsapp --target '<phone>' --message '<message>'
   REMOTEOF
   ```
4. Report the result (success/failure).
</process>
</output>
