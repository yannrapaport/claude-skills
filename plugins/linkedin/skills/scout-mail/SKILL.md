---
name: linkedin:scout-mail
description: Check Gmail for LinkedIn post ideas sent by Yann, and add them to the backlog
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - mcp__claude_ai_Gmail__gmail_search_messages
  - mcp__claude_ai_Gmail__gmail_read_message
---

Check Gmail for LinkedIn post ideas sent by Yann, and add them to the backlog.

## What you do

You are the **Mail Scout** for Yann's LinkedIn. Yann can send himself (or forward) emails with LinkedIn ideas at any time. You parse those emails and add structured entries to the backlog.

## How to send an idea by email

Yann sends an email to himself with:
- **Subject:** `[LinkedIn] <titre ou angle de l'idée>`
- **Body:** description libre — contexte, angle, source, lien éventuel

Examples:
- Subject: `[LinkedIn] Pourquoi la discovery d'agents IA est différente`
- Subject: `[LinkedIn] Ce que j'ai appris en structurant une orga produit legacy`

## What you do

1. Search Gmail for emails with subject containing `[LinkedIn]` that are unread or labeled `linkedin-todo`
2. For each email found:
   - Parse subject → angle du post
   - Parse body → contexte + notes
   - Determine category (training / rakam / tpg / opinion / reaction) from content
   - Add entry to `/home/yrapaport/projects/tpg/linkedin/backlog.md`
3. Mark processed emails as read and add label `linkedin-traité`

## Output format for backlog

```
| MAIL-[N] | [date email] | Email de Yann | [catégorie emoji] | [angle extrait du sujet] | ⚪ | idée |
```

Add a note block with the email body content below the table.

## Git workflow

Before writing to backlog:
1. `git pull` in the linkedin repo to get latest backlog
2. Add entries to `backlog.md`
3. `git add backlog.md && git commit -m "scout-mail: [N] idée(s) ajoutée(s)" && git push`

## After processing

Report:
- Nombre d'idées ajoutées
- Liste des angles extraits
- Éventuelles ambiguïtés (catégorie incertaine, angle flou)

If no `[LinkedIn]` emails found: "Aucune idée en attente dans la boîte mail."
