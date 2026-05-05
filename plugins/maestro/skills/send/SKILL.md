---
name: maestro:send
description: Send a message via Herbert in a wired WhatsApp channel (NanoClaw admin transport). Pass <alias> and <message>; the message becomes a prompt that Herbert sees as inbound in the target channel and responds to. Not a raw send — Herbert reformulates with his persona.
allowed-tools:
  - Bash
  - AskUserQuestion
---

## What this skill does

Pushes a message into NanoClaw's CLI socket admin transport (`~/projects/tpg/nanoclaw/data/cli.sock` on Nexus) targeting a wired WhatsApp channel. The daemon synthesizes an InboundEvent for that channel, the wired agent (Herbert / Coach Roger / …) sees it as a regular inbound message and engages.

**Important**: this is *not* a raw delivery. The message you pass is a **prompt** the agent receives. The agent then writes its own reply with its persona — that reply is what arrives in the WhatsApp channel.

## Usage

```
/maestro:send <alias> <message...>
```

If args are missing, ask the user.

- `<alias>` — short identifier (e.g. `alex-nat-yannus`, `la-plushy-team`, `yann-dm`). Run `/maestro:list-groups` to discover.
- `<message...>` — the prompt (full sentence; agent will reformulate).

## Caveat per group

Each wired channel has an `engage_pattern` that controls when the agent responds. As of last check (verify in `~/ai-brain/tools/maestro-radar.md` if uncertain):

- `engage_pattern='.'` (always engage) — any prompt triggers the agent. Groups: `alex-nat-yannus`, `gabs-rapaport`, `pouffons-coussins`, `yann-dm`, all TCCV individual DMs, `poule-14`, `test-tccv`.
- `engage_pattern='[Hh]erbert\b'` — the prompt **must contain "Herbert"** or it gets dropped silently. Groups: `la-plushy-team`, `operation-summer-body`. For these, prefix the message with "Herbert,".

## Procedure

1. If `<alias>` is missing: ask the user (suggest running `/maestro:list-groups` first).
2. If `<message>` is missing: ask the user.
3. Execute via SSH on Nexus. For messages containing quotes or special chars, prefer `printf %q` quoting:
   ```bash
   ssh nexus "cd ~/projects/tpg/nanoclaw && pnpm tsx scripts/send.ts <alias> $(printf '%q' "<message>")"
   ```
   Simpler form for plain text:
   ```bash
   ssh nexus 'cd ~/projects/tpg/nanoclaw && pnpm tsx scripts/send.ts <alias> "<message>"'
   ```
4. Read the script's stderr for the resolution hint (`→ Herbert on whatsapp/...`) and confirm to user.
5. Tell the user to check WhatsApp — there is no synchronous confirmation of agent reply (the agent runs async in a container, ~10-30s).

## Failure modes

- `NanoClaw daemon not reachable` — socket is stale or service down. `ssh nexus 'systemctl --user restart nanoclaw.service'` (warn: interrupts active sessions ~5 sec).
- `Alias is ambiguous` — script lists candidates; pick a more specific alias and retry.
- `No group matches` — alias not wired; run `/maestro:list-groups`.
- Agent doesn't reply — check `engage_pattern` (see caveat) or tail `~/projects/tpg/nanoclaw/logs/nanoclaw.log` for `MESSAGE DROPPED` lines.

## Related

- `/maestro:list-groups` — list available aliases.
- Doc: `~/ai-brain/tools/maestro-radar.md`.
- Source: `~/projects/tpg/nanoclaw/scripts/send.ts` on Nexus.
