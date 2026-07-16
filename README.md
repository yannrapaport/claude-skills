# claude-skills

Public [Claude Code](https://claude.com/claude-code) skills by [Yann Rapaport](https://theproductguy.fr) — Fractional CPTO @ The Product Guy.

A small marketplace of skills I actually use, shared as working examples.

## Skills

### `yannizer` — a voice-izer that learns your voice

Rewrites AI-drafted text (emails, LinkedIn posts, docs) so it sounds like a real person instead of a language model. It strips the usual AI tells (FR + EN) and reinjects a real voice.

The point isn't the rewrite — it's the **learning loop**. Every time you correct a proposal, the correction becomes a new rule in the skill. It sticks a little closer to your tone on each pass. A fixed prompt plateaus; a loop that captures your corrections doesn't.

The published profile is calibrated on *my* voice, as an example. Fork it, rename it, empty the voice profile, and recalibrate it on your own corrections.

> The skill file is written in French — it's calibrated on a French speaker's corpus. The method itself is language-agnostic.

A quick before / after:

```
AI draft   → Je me permets de revenir vers vous concernant notre échange.
             N'hésitez pas à me contacter pour toute question. Bien cordialement,
yannized   → Je reviens vers vous sur notre échange.
             Une question — dis-moi. À bientôt,
```

### `mastermind` — multi-agent review

Launches parallel expert agents (PM, Architect, Dev, …) to analyze a plan, codebase, or document from multiple perspectives, then synthesizes their findings. Useful for a second opinion before committing to an important decision.

## Install

```
/plugin marketplace add yannrapaport/claude-skills
/plugin install yannizer@claude-skills
/plugin install mastermind@claude-skills
```

## License

MIT — see [LICENSE](LICENSE).
