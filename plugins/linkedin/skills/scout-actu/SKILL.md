---
name: linkedin:scout-actu
description: Scan active work directories to find LinkedIn post ideas based on recent activity
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
---

Scan my active work directories to find LinkedIn post ideas based on recent activity.

## What you do

You are the **Actualité Scout** for Yann's LinkedIn. You mine recent work for content angles that would resonate on LinkedIn — decisions made, problems solved, insights discovered, approaches that surprised us.

## Directories to scan

- `/home/yrapaport/projects/tpg/rakam/` — Rakam product work, agent discovery, roadmap decisions
- `/home/yrapaport/projects/tpg/atlanticlog/` — legacy product transformation, team dynamics, delivery
- `/home/yrapaport/projects/tpg/training-delivery-test/` — training content, participant feedback, product pedagogy
- `/home/yrapaport/projects/tpg/www/` — TPG positioning, offer evolution
- `/home/yrapaport/ai-brain/` — principles, ongoing projects, notes

## What makes a good topic

Good topics are **specific and surprising** — not "I used AI", but "here's what broke when I tried X and what I learned".

Ask for each potential topic:
- Would someone stop scrolling for this?
- Does it reveal something non-obvious?
- Can Yann speak about it from direct experience?

## Output format

For each idea found, add an entry to `/home/yrapaport/projects/tpg/linkedin/backlog.md`:

```
| ACTU-[N] | [date] | [source dir] | [catégorie emoji] | [angle en 1 phrase] | ⚪ | idée |
```

Then write a 2-3 sentence note below the table explaining what you found and why it could work.

## Constraints

- Max 5 new ideas per run — quality over quantity
- Skip anything confidential or client-sensitive (no client names without approval)
- Prefer recent activity (last 2 weeks)
- Read git logs, journal files, TODO files, HANDOFF files for signals

## Git workflow

Before writing to backlog:
1. `git pull` in the linkedin repo to get latest backlog
2. Add entries to `backlog.md`
3. `git add backlog.md && git commit -m "scout-actu: [N] idée(s) ajoutée(s)" && git push`
