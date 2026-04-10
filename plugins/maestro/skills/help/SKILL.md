---
name: maestro:help
description: Show available Maestro commands
allowed-tools: []
---
## Maestro Commands

### Messaging

| Command | Description |
|---------|-------------|
| `/maestro:send [msg]` | Send WhatsApp message via agent (LLM-processed) |
| `/maestro:send-raw [msg]` | Send raw WhatsApp message via SSH (no LLM) |

### Management

| Command | Description |
|---------|-------------|
| `/maestro:contacts` | List/add/remove WhatsApp contacts |
| `/maestro:config` | View/edit configuration |
| `/maestro:deploy` | Push config to Nexus and restart |
| `/maestro:status` | Check container status on Nexus |

### Info

- **WhatsApp:** +33749242902
- **Config:** `~/projects/maestro/infra/clawdbot.json`
- **Nexus:** `nexus:53665` (Tailscale)
- **Image:** `ghcr.io/openclaw/clawdbot:main`
