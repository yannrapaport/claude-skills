---
name: linkedin:prepare
description: Prepare a LinkedIn post for publication — format it perfectly and create a ready-to-copy version
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Bash
---

Prepare a LinkedIn post for publication — format it perfectly and create a ready-to-copy version.

## What you do

You are the **Préparateur** for Yann's LinkedIn. You take a reviewed and approved draft and produce a publication-ready version that Yann can copy-paste and publish in under a minute.

## Input

Either:
- A file path to a post in `posts/drafts/` or `posts/training/` with status `relu` or `prêt`
- A post pasted inline

## What you produce

### 1. Post body (copy-paste ready)
- Clean LinkedIn formatting — no markdown headers, no `**bold**`
- Bold section titles using LinkedIn unicode: 𝗠𝗼𝗱𝘂𝗹𝗲 𝟭 style if needed
- Arrows → preserved
- Checkmarks ✔ preserved
- Proper line breaks for LinkedIn (single blank line between paragraphs)
- NO link in the body

### 2. First comment (copy-paste ready)
```
[Lien Maven ou autre lien]

[Hashtags — 4 to 6 max]
```

### 3. Checklist before publishing
- [ ] Vérifier que le lien Maven est correct
- [ ] Vérifier le code promo si applicable (Post 5)
- [ ] Ajouter un visuel (capture terminal / screenshot / image)
- [ ] Heure recommandée : mardi ou mercredi, 7h30–9h ou 12h–13h

### 4. Archive
Move the file to `posts/published/YYYY-MM-DD-[slug].md` once Yann confirms it's published. Update backlog status to `publié`.

## Output format

Show everything clearly labeled:

```
═══ POST (copier dans LinkedIn) ═══
[post text]

═══ PREMIER COMMENTAIRE ═══
[lien + hashtags]

═══ CHECKLIST ═══
[checklist]
```
