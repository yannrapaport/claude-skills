---
name: maestro:config
description: View/edit Maestro configuration
allowed-tools:
  - Read
  - Edit
  - AskUserQuestion
---
<objective>
View and interactively edit the Maestro clawdbot configuration.
</objective>

<context>
Config file: `~/projects/maestro/infra/clawdbot.json`
</context>

<process>
1. Read `~/projects/maestro/infra/clawdbot.json`.
2. Present a summary:
   - **Model:** `agents.defaults.model.primary` and fallbacks
   - **Agents:** list with id, model, sandbox mode
   - **Channels:** WhatsApp policies (dmPolicy, groupPolicy, selfChatMode)
   - **Gateway:** mode, auth, endpoints
   - **Hooks:** enabled/disabled
   - **Tools:** web search config
3. Ask the user what they want to change (or "Done" to exit).
4. For each change:
   - Edit the specific field in the JSON
   - Show the updated value
5. After changes, remind the user to run `/maestro:deploy` to push the config to Nexus.
</process>
</output>
