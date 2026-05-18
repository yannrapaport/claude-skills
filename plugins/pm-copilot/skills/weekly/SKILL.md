---
name: pm-copilot:weekly
description: Assist with the Rakam Product Weekly — prep the meeting (Notion draft + page), update the report after the meeting from Fireflies/Granola, and draft the Slack highlights. Rakam-specific. Use args `prep`, `update`, or `slack`.
---

# /pm-copilot:weekly — Rakam Product Weekly workflow

Assist Yann in producing the **Rakam Product Weekly**: a Notion page (Meetings DB) that drives Monday's 16:30 meeting, plus a `#internal-product` Slack highlights post after the meeting.

**Rakam-specific.** Do **not** pull from ClickUp — it is not kept up to date.

## Args

| Arg      | When            | What                                                        |
|----------|-----------------|-------------------------------------------------------------|
| `prep`   | Before Monday   | Build the draft, iterate with Yann, publish the Notion page |
| `update` | After Monday    | Refresh the Notion page from the meeting transcript         |
| `slack`  | After `update`  | Draft the `#internal-product` highlights message            |

If no arg is given, ask which phase.

## Rakam references

- **Notion Meetings DB**: `https://www.notion.so/b1772073609e4298b7c67718b8d73361`
  - Data source ID: `e5450e5a-b4e7-4a06-9dc7-aae2edd19790`
- **Product Meetings hub** (user-added topics for next meeting): `https://www.notion.so/Product-Meetings-2cb17b6f64ef80eaa0decbd7dd9089d5`
- **Slack channel `#internal-product`**: `C08J43VJDEE`
- **Journal**: `~/projects/tpg/rakam/journal/YYYY-MM-DD.md` (often partial)
- **Scratchpad draft**: `~/projects/tpg/rakam/_scratchpad/YYYY-MM-DD-product-weekly-draft.md`
- **Internal docs are in English** (conversation with Yann stays French)
- **Spelling**: Rechad, Fedi, Bachar, MonEcho, DIAM, Société.com, AlumnForce, Beetween, VAL

---

## Phase 1 — `prep`

Goal: build the draft + publish the Notion page before the Monday 16:30 meeting.

1. **Pull the previous weekly** from the Meetings DB. Its structure + topics carry over by default; only drop topics explicitly closed last week.
2. **Build the week's picture**:
   - Read `~/projects/tpg/rakam/journal/*.md` since the previous weekly (often sparse).
   - List Granola meetings in range via `mcp__claude_ai_Granola__list_meetings` (custom range from previous weekly date to today).
   - Fetch the **Product Meetings** hub for any user-added next-meeting topics.
   - Filter Granola to Rakam-only (drop CDiscount, AtlanticLog, personal calls).
3. **Align with Yann**: present the Rakam-relevant meetings + hub topics + carry-overs. Ask which to dig into, what's closed, and any context only he holds (travel, holidays, missed sessions). Don't pull transcripts on your own — ask first.
4. **Write the draft** to `_scratchpad/YYYY-MM-DD-product-weekly-draft.md` mirroring the previous report's structure:

   ```
   <!-- notion: TBD -->

   # 🏭 Product — Weekly Product YYYY-MM-DD

   ## 🧭 Org              (only if an org item this week)

   ## 🟧 Rakam Agents
   ### Support Agent — priority #1
   ### Société.com Copilot
   ### MCP template

   ## 🟨 Rakam Lab · 🟫 Tests Suite

   ## 🟩 RS 0.3.x · 🟦 Observability

   ## 🟪 Data Collection

   ## 🟥 Action Items
   - [ ] **Owner** — task *(carry-over if applicable)*
   ```

   Style: factual, no analysis/decisions sections, bold headlines, lean enough to leave room for the team to discuss. The report **conducts** the meeting, it doesn't replace it.

5. **Submit to Yann** for review. Iterate.
6. **Publish to Notion** with `mcp__claude_ai_Notion__notion-create-pages` under data source `e5450e5a-b4e7-4a06-9dc7-aae2edd19790`:
   - `icon`: `🏭`
   - Properties:
     - `Name`: `Weekly Product YYYY-MM-DD`
     - `Label`: `["Product"]`
     - `Type`: `Team weekly`
     - `date:Event time:start`: `YYYY-MM-DD`
     - `date:Event time:is_datetime`: `0`
   - `content`: the draft body (strip the `# 🏭 Product …` H1 and the `<!-- notion: -->` comment — they're not part of the Notion body).
7. **Backfill** the scratchpad's `<!-- notion: -->` comment with the new page URL.

---

## Phase 2 — `update`

Goal: refresh the published Notion page from the actual Monday meeting.

1. **Locate inputs**:
   - The current week's Notion page (`Weekly Product YYYY-MM-DD`).
   - The Fireflies transcript: `mcp__claude_ai_Fireflies__fireflies_get_transcripts` (today, mine), then `fireflies_get_transcript` for the full content. The weekly is typically Monday 16:30, ~60 min, organized by Yann.
2. **Cross-reference** each section against the transcript: confirm, add, close. Don't reshape the structure unless asked.
3. **Beware Fireflies hallucinations** in the summary's action items. Action items sometimes invent attributions that aren't in the transcript. **Grep the raw transcript** before adding a contested item.
4. **Patch Notion** via `notion-update-page` `update_content` (search/replace).
5. **Consolidate team-impacting action items** in the `🟥 Action Items` section. Skip pure Yann-internal follow-ups unless they block someone else.
6. **Report misses**: tell Yann what you found in Fireflies but did **not** add (hallucination, redundant, unclear) so he can decide.

---

## Phase 3 — `slack`

Goal: post **highlights only** to `#internal-product` (channel `C08J43VJDEE`). The Notion page is the full version — Slack does not duplicate it.

### Format

```
:factory: Weekly Product YYYY-MM-DD — highlights :point_down: (<NOTION_URL|full version>)

Org
{1–2 load-bearing public bullets — skip internal-only details like day-rate changes}

Support Agent — priority #1
{1–3 bullets on concrete deliveries / customer-facing milestones}

Société.com Copilot
{1 condensed bullet, @mentions where relevant}

Ping me with anything to adjust :wave:
```

### Rules

- **Sections kept**: Org, Support Agent, Société.com Copilot. Other Notion sections (MCP, Lab/Tests, RS 0.3.x, Data Collection, V1 deadlines list, action items) stay in Notion only.
- **Slack mrkdwn**: plain newlines as bullets (no `•`), no bold on section names, mentions as `@Name` (or `<@UID|Name>` if you have the user ID), links as `<URL|label>`.
- **Tone**: relaxed, conversational. Standard close: `Ping me with anything to adjust :wave:` (no trailing `---`).
- **Skip on Slack**: V1 deadlines per client, roadmap skeleton mentions, Open V1 scope, internal-only org details.

### Action

Use `mcp__claude_ai_Slack__slack_send_message_draft` against channel `C08J43VJDEE`. Yann reviews + sends from Slack.

---

## Memory anchor

The full method is also captured in user memory (`feedback_weekly_report_method.md`). This skill is the executable version.
