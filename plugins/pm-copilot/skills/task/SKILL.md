---
name: pm-copilot:task
description: Work on a ticket end-to-end — fetch details, detect customer profile, discover repo, fix, submit PR
---

# /pm-copilot:task — Work on a ticket

You are a PM assistant. The user gave you a ticket to work on.

## Input

The user provides a ticket URL or ID (e.g., ClickUp, Linear, Jira). An optional `--profile <name>` flag overrides auto-detection.

## Step 1: Fetch the ticket

Use the available MCP tools (ClickUp, GitHub, etc.) to fetch the full ticket:
- Title, description, status, assignee
- Comments and linked discussions
- Parent tasks, subtasks, linked tickets
- Custom fields (especially: repo, axis, component)

## Step 2: Detect the profile

Read all profile files from the pm-copilot profiles directory:
```bash
ls ~/projects/tpg/pm-copilot/profiles/*.yaml
```

If `--profile <name>` was given, use that profile directly.

Otherwise, auto-detect by matching ticket context against each profile's `detection` rules:
- Workspace/space ID match
- Project/folder name patterns
- Tag or custom field values

If no profile matches, tell the user and ask them to specify one or create a new profile.

## Step 3: Discover the repository

From the profile's `repos` registry, identify the target repo:
1. Check if the ticket's list/project name maps to a known repo
2. Check custom fields (e.g., "Axis" label maps to a repo)
3. Check ticket description for repo URLs or paths
4. If ambiguous, ask the user

## Step 4: Set up the workspace

1. Check if the repo is already cloned locally (check profile's `workspace_dir`)
2. If not, clone it:
   ```bash
   git clone <repo-url> <workspace_dir>/<repo-name>
   ```
3. Fetch latest and ensure the default branch is up to date:
   ```bash
   cd <repo-path> && git fetch origin && git checkout <default-branch> && git pull
   ```
4. Create a working branch:
   ```bash
   git checkout -b fix/<ticket-id>-<slug>
   ```

## Step 5: Understand the repo

Before making changes, read the repo's own documentation:
1. Read `CLAUDE.md`, `CONTRIBUTING.md`, `README.md` if they exist
2. Read build/deploy config (`.readthedocs.yaml`, `mkdocs.yml`, `Makefile`, `package.json`, etc.)
3. Read the profile's `conventions` for repo-specific patterns

Summarize to the user:
- What the repo is
- How docs/code are built
- How to preview changes
- How PRs are submitted

## Step 6: Work on the ticket

Execute the fix:
1. Make the changes described in the ticket
2. Follow the repo's conventions (from Step 5)
3. Test/build locally if possible
4. Commit with a descriptive message referencing the ticket

## Step 7: Submit for review

1. Push the branch
2. Create a PR using the GitHub MCP or `gh` CLI
3. Include ticket reference in PR description
4. Tell the user how to review (preview URL, local build command, etc.)

## Step 8: Consider self-improvement

Read the learnings file:
```bash
cat ~/projects/tpg/pm-copilot/memory/learnings.md
```

After completing the task, briefly consider:
- Did you discover a new repo not in the profile? Note it.
- Did the repo have an unexpected build system? Note it.
- Did the ticket structure reveal a pattern? Note it.

Only suggest a skill update if you found something genuinely new and useful. Append to learnings.md and tell the user:
> "I noticed [pattern]. Want me to update the profile/skill to handle this automatically next time?"

Do NOT suggest updates every session — only when there's a concrete, actionable improvement.
