---
name: maestro:list-groups
description: List the wired WhatsApp channels available for /maestro:send, with their canonical aliases, full names, and bound agents. Read-only DB lookup on Nexus.
allowed-tools:
  - Bash
---

## What this skill does

Lists every wired NanoClaw messaging group on Nexus along with its canonical alias (the identifier you pass to `/maestro:send`).

Source: `~/projects/tpg/nanoclaw/scripts/send.ts --list` on Nexus, which joins `messaging_groups`, `messaging_group_agents` and `agent_groups` from the SQLite DB.

## Procedure

```bash
ssh nexus 'cd ~/projects/tpg/nanoclaw && pnpm tsx scripts/send.ts --list'
```

Show the resulting table to the user verbatim — it's the source of truth.

## Alias resolution rules (for context if asked)

- When a `groups/<channel>_<slug>/` folder owns a single messaging_group, the alias is `<slug>` (e.g. `alex-nat-yannus`).
- When several messaging_groups share the same folder (e.g. `whatsapp_tccv-test/` covers six TCCV contacts), each gets a slugified `<group-name>` alias (e.g. `kania-romain-tccv`, `poule-14`).
- The `/maestro:send` resolver also accepts substring matches and exact group-name matches; ambiguity errors out with the candidate list.

## Related

- `/maestro:send` — actually send via an alias.
- Doc: `~/ai-brain/tools/maestro-radar.md`.
