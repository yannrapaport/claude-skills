---
name: maestro:send-raw
description: Send raw WhatsApp message via SSH (no LLM processing)
allowed-tools:
  - Bash
  - AskUserQuestion
---
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
