# claudify

Bootstrap `.claude/` structure for any existing repo.

Run `/claudify:init` from the root of any repo and get a clean, well-structured `.claude/` folder — generated from your actual codebase, not generic templates. Follows official [Claude Code memory best practices](https://code.claude.com/docs/en/memory).

## Install

Add the phucbm marketplace (once, ever):
```shell
/plugin marketplace add phucbm/skills
```

Install plugin:
```shell
/plugin install claudify@phucbm
```

Update all phucbm plugins:
```shell
/plugin marketplace update phucbm
```

## Usage

Bootstrap `.claude/` for a repo:
```shell
/claudify:init
```

Refresh CLAUDE.md to match the current codebase:
```shell
/claudify:update
```

`/claudify:update` delegates to `claude-md-management@claude-plugins-official`. If that plugin is not installed, it will tell you how to install it.

## What gets generated

| File | Description |
|---|---|
| `.claude/CLAUDE.md` | Entry point with `@import` directives, exact commands, and codebase-specific rules (≤ 80 lines) |
| `.claude/docs/architecture.md` | Stack, structure, data flow, and key files (≤ 150 lines) |
| `.claude/docs/agents.md` | AI/LLM layer details — only generated if your repo uses AI (≤ 150 lines) |

If `.claude/` already exists with content, claudify shows you exactly what will change and asks for confirmation before writing anything.

## Auto-hook

`/claudify:init` writes a `PostToolUse` hook to `.claude/settings.json` that fires whenever Claude edits a `.md` file and reminds you to run `/claudify:update`.
