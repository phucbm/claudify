# claudify

Bootstrap `.claude/` structure for any existing repo.

Run `/claudify:init` from the root of any repo and get a clean, well-structured `.claude/` folder — generated from your actual codebase, not generic templates. Follows official [Claude Code memory best practices](https://code.claude.com/docs/en/memory).

## Install

```shell
/plugin marketplace add phucbm/claudify
/plugin install claudify@phucbm
```

## Usage

From the root of any repo:

```shell
/claudify:init
```

## What gets generated

| File | Description |
|---|---|
| `.claude/CLAUDE.md` | Entry point with `@import` directives, exact commands, and codebase-specific rules (≤ 80 lines) |
| `.claude/docs/architecture.md` | Stack, structure, data flow, and key files (≤ 150 lines) |
| `.claude/docs/agents.md` | AI/LLM layer details — only generated if your repo uses AI (≤ 150 lines) |

If `.claude/` already exists with content, claudify shows you exactly what will change and asks for confirmation before writing anything.

## Tip

To keep your CLAUDE.md files healthy over time, use the official `claude-md-management` plugin:

```shell
/plugin install claude-md-management@claude-plugins-official
```
