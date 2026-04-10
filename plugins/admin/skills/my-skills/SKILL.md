---
name: admin:my-skills
description: List all personal skills from the yann-skills plugin marketplace
---

# My Skills

List all personal skills already loaded in your context from the `yann-skills` plugins.

## Instructions

Do NOT read any files. All skills are already in your system prompt.

Output every skill whose name matches `<namespace>:<command>` where the namespace belongs to `yann-skills` plugins:
`todo-manager`, `admin`, `session`, `ai-brain`, `dotfiles`, `linkedin`, `maestro`, `nexus`, `pm-copilot`, `mastermind`, `my-skills`

## Output format

Group by namespace with `───` underlines:

```
namespace
───
  /namespace:command       Description
```

Sort namespaces alphabetically. Align descriptions.
