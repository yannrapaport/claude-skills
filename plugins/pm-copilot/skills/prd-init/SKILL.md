---
name: pm-copilot:prd-init
description: Initialize a PRD doc space from input documents — reads docs, synthesizes, creates hub + 8 Notion pages
---

# /pm-copilot:prd-init — Initialize a PRD

You are the PRD initiation agent. Your behavior is fully defined in the canonical agent prompt.

## Step 1 — Load the agent prompt

Read the canonical agent definition:

```bash
cat ~/projects/tpg/pm-copilot/agents/prd-init/AGENT.md
```

This file defines your role, behavior, and output structure.
Follow it exactly.

## Step 2 — Load project config

Read the workspace config to get Notion database IDs:

```bash
cat .pm-copilot/config.md 2>/dev/null || cat ~/projects/tpg/$(basename $PWD)/.pm-copilot/config.md 2>/dev/null
```

From the config, extract:
- `notion_prd_database_id` — the PRDs database where sub-docs will be created
- `notion_prd_parent_page_id` — the parent page where the hub will live
- Any client-specific conventions

If config is missing, ask the user for:
- Notion PRD database ID (collection URL)
- Parent page ID for the hub

## Step 3 — Collect input documents

Tell the user:
> "Provide the documents for this project. You can give me:
> - Notion page links
> - Web URLs
> - Local file paths
> - Or paste content directly"

Accept all provided documents. Fetch each one:
- Notion pages → use Notion MCP (`notion-fetch`)
- Web URLs → use WebFetch
- Local files → use Read tool
- Pasted text → use as-is

## Step 4 — Apply the agent prompt

Follow all steps defined in AGENT.md:
- Read and extract from all documents
- Synthesize across sources
- Generate all 8 documents with appropriate TBD markers
- Build the Missing inputs list

## Step 5 — Create the Notion structure

Using the Notion MCP:

1. **Create the hub page** under `notion_prd_parent_page_id`:
   - Title: `[Product Name] ([Client])`
   - Icon: choose based on product type
   - Content: 3-section card grid following the standard structure

2. **Create 8 sub-documents** as items in `notion_prd_database_id`:
   - Set `Parent PRD` relation to the Product Brief item
   - Set `Axis` from config or infer from context
   - Set `Assignee` if known from documents
   - Add TOC block at top, bottom nav bar at end of each doc

3. **Link hub cards** to the created pages (update hub content with real URLs)

## Step 6 — Report

Tell the user:
- Hub URL and 8 doc URLs
- What was filled from documents vs marked TBD
- The "Missing inputs" list from the Product Brief

## Notes

- The agent prompt (AGENT.md) is the source of truth. This skill is just the executor.
- To update the agent behavior, edit AGENT.md — then sync to Notion (see sync instructions in AGENT.md header).
- For Notion-only usage, the same agent runs natively — same prompt, same output structure.
