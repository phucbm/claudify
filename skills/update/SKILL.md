---
name: update
description: Keep CLAUDE.md files in sync with the current codebase using claude-md-management
allowed-tools: Read Glob Bash Write Edit
---

Refresh this repo's CLAUDE.md files to reflect the current codebase state.

## Step 1 — Check plugin availability

Look at the skills listed in `<system-reminder>`. Search for `claude-md-management` or `revise-claude-md`.

**If found:** proceed to Step 2.

**If not found:** output exactly:

> `claude-md-management` plugin is not installed. Run:
> ```
> /plugin install claude-md-management@claude-plugins-official
> ```
> Then re-run `/claudify:update`.

Stop here.

## Step 2 — Run revise-claude-md

Invoke the `claude-md-management:revise-claude-md` skill now.
