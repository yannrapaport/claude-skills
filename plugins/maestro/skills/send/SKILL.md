---
name: maestro:send
description: "[DEPRECATED] Send WhatsApp via claudius-admin agent (gateway clawdbot n'existe plus depuis 2026-04-28)"
allowed-tools:
  - Bash
  - AskUserQuestion
---
<deprecated>
⚠️ **CETTE SKILL EST CASSÉE depuis 2026-04-28.**

L'endpoint `http://nexus:53665/v1/chat/completions` (gateway clawdbot) et l'agent `claudius-admin` n'existent plus. La stack WhatsApp tourne maintenant sous **NanoClaw v2** + persona **Herbert**.

**Ne pas exécuter cette skill** — voir `~/ai-brain/tools/maestro-radar.md` pour l'état courant.
</deprecated>

<objective>
Send a WhatsApp message through the Maestro chat completions API. The message is processed by the LLM agent, which decides how to deliver it.
</objective>

<context>
Endpoint: `http://nexus:53665/v1/chat/completions`
Auth: Bearer token from `CLAWDBOT_GATEWAY_TOKEN` environment variable.
Model/agent: `claudius-admin` (has WhatsApp send capability).
</context>

<process>
1. Check if `$CLAWDBOT_GATEWAY_TOKEN` is set. If not, fetch it:
   ```bash
   export CLAWDBOT_GATEWAY_TOKEN=$(ssh nexus "grep CLAWDBOT_GATEWAY_TOKEN /docker/claudius/.env | cut -d= -f2")
   ```
2. If no message was provided in the command arguments, ask the user for the message.
3. Ask the user for the recipient (phone number in international format, e.g. +33612345678).
4. Send via chat completions API. Use a heredoc to avoid shell escaping issues:
   ```bash
   curl -s -X POST http://nexus:53665/v1/chat/completions \
     -H "Authorization: Bearer $CLAWDBOT_GATEWAY_TOKEN" \
     -H "Content-Type: application/json" \
     -d "$(cat <<PAYLOAD
   {"model":"claudius-admin","messages":[{"role":"user","content":"Send a WhatsApp message to <phone> saying: <message>"}]}
   PAYLOAD
   )"
   ```
5. Parse the JSON response and report the agent's reply (which confirms delivery).
</process>
</output>
