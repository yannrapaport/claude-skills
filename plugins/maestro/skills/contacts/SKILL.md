---
name: maestro:contacts
description: "[DEPRECATED] Manage clawdbot.json contacts (config n'est plus lu depuis 2026-04-28)"
allowed-tools:
  - Read
  - Edit
  - AskUserQuestion
---
<deprecated>
⚠️ **CETTE SKILL EST OBSOLÈTE depuis 2026-04-28.**

Le fichier `~/projects/maestro/infra/clawdbot.json` n'est plus lu (le container clawdbot ne tourne plus). Les contacts WhatsApp et les groupes sont maintenant configurés dans NanoClaw : `~/projects/tpg/nanoclaw/groups/whatsapp_*/container.json`.

À reconstruire pour cibler NanoClaw.
</deprecated>

<objective>
Manage WhatsApp contacts in the Maestro clawdbot configuration.
</objective>

<context>
Config file: `~/projects/maestro/infra/clawdbot.json`

Contact management involves three sections:
- `channels.whatsapp.allowFrom` — phone numbers allowed to DM
- `bindings` — maps contacts to agents (claudius-admin or claudius-lite)
- Agent assignment: Yann (+33627636796) → claudius-admin, others → claudius-lite
</context>

<process>
1. Read `~/projects/maestro/infra/clawdbot.json`.
2. Parse and display current contacts:
   - List each phone number from `allowFrom`
   - Show which agent each is bound to (from `bindings`)
   - Show group bindings separately
3. Ask the user what they want to do: **Add contact**, **Remove contact**, or **Done**.
4. If adding:
   - Ask for phone number (international format, e.g. +33612345678)
   - Ask for agent: claudius-admin or claudius-lite (default: claudius-lite)
   - Add to `allowFrom` array
   - Add a binding entry with the chosen agent, kind: "dm"
5. If removing:
   - Show numbered list of contacts
   - Ask which to remove
   - Remove from `allowFrom` and corresponding `bindings` entry
6. After changes, remind the user to run `/maestro:deploy` to push the config to Nexus.
</process>
</output>
