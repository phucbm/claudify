---
name: init
description: Bootstrap a clean .claude/ folder structure for any existing repo following official Claude Code best practices
disable-model-invocation: true
allowed-tools: Read Glob Bash Write Edit
---

Bootstrap a well-structured `.claude/` folder for this repo. **Never write any file until the user confirms the plan.**

---

## Phase 1 — Read everything

Collect context in parallel:

1. **Full file tree** — `find . -maxdepth 3 -not -path './.git/*' -not -path './node_modules/*' -not -path './vendor/*' -not -path './.claude-plugin/*' | sort`
2. **Manifest files** — read whichever exist: `package.json`, `composer.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`
3. **All markdown files in the repo** — find them: `find . -name "*.md" -not -path './.git/*' -not -path './node_modules/*' -not -path './vendor/*' | sort` — then read every one. This includes root-level READMEs, any existing `.claude/` files, scattered docs, CONTRIBUTING.md, ARCHITECTURE.md, etc.
4. **Entry files** — based on detected stack, read the main entry point(s): `src/index.ts`, `src/main.ts`, `main.py`, `app.py`, `main.go`, `index.php`, `app.rb`, `lib/index.js`, or equivalent

---

## Phase 2 — Detect

From what you read, determine:

- **Language & framework** — e.g. "TypeScript + Next.js 14", "Python + FastAPI", "PHP + WordPress"
- **Build / test / lint / dev commands** — exact commands from manifest scripts only; do not guess
- **AI / LLM / agent features** — scan imports and dependencies for: `openai`, `anthropic`, `langchain`, `llama`, `ollama`, `transformers`, `langfuse`, `llamaindex`, vector store clients (pinecone, weaviate, chroma). Set `has_ai = true` if found.
- **Existing markdown inventory** — for every `.md` file found, classify it:
  - `keep-as-is` — already in the right place and well-structured, no changes needed
  - `rewrite` — right location but content is bloated, outdated, or poorly structured (e.g. a 400-line CLAUDE.md)
  - `move` — useful content but in the wrong location (e.g. `ARCHITECTURE.md` at root → `.claude/docs/architecture.md`)
  - `merge` — content should be folded into a `.claude/` file, original deleted
  - `redundant` — generic, empty, or fully duplicated content that adds no value
  - `new` — a `.claude/` file that needs to be created from scratch

---

## Phase 3 — Propose the plan

Present a clear, numbered action plan. **Do not write any files yet.**

Format it exactly like this:

```
## claudify plan

### Detected
- Stack: [language + framework]
- Commands: build `x`, test `x`, lint `x`, dev `x` (omit any that don't exist)
- AI features: yes / no

### Actions
1. CREATE   .claude/CLAUDE.md              — entry point with commands and rules
2. CREATE   .claude/docs/architecture.md  — stack, structure, data flow
3. CREATE   .claude/docs/agents.md        — AI layer (only if has_ai = true)
4. REWRITE  .claude/CLAUDE.md             — currently 400 lines; will be reduced to ≤80 with @imports
5. MOVE     ARCHITECTURE.md → .claude/docs/architecture.md, original deleted
6. MERGE    docs/dev-notes.md → relevant facts folded into .claude/docs/architecture.md, original deleted
7. DELETE   .claude/random-notes.md       — redundant, no actionable content

### No changes needed
- README.md — not a Claude config file, left untouched
```

For every **REWRITE** action, immediately follow it with a fenced before/after diff so the user can see exactly what changes:

```
#### REWRITE .claude/CLAUDE.md

**Before** (summarised — 400 lines):
> Copy-pasted README, generic advice, outdated npm commands, no structure

**After** (proposed — ~35 lines):
[Show the full proposed file content here, verbatim]
```

After all actions and diffs, always end with:

```
Proceed? (yes / no — or tell me what to adjust)
```

Rules for the action list:
- Only list files that will actually change
- Include a one-line reason for every action
- Group by type: CREATE first, then REWRITE, then MOVE/MERGE, then DELETE
- Files outside `.claude/` that are not markdown config (source code, assets, etc.) are never touched
- Always end with the confirmation prompt and stop

---

## Phase 4 — Wait for confirmation

Do not proceed until the user explicitly says yes (or equivalent). If they say no or ask to adjust, revise the plan and re-present it before doing anything.

---

## Phase 5 — Execute

Execute exactly the approved plan, in this order: CREATE → REWRITE → MOVE/MERGE → DELETE.

**Content rules for generated and rewritten files:**
- No generic advice ("write clean code", "add comments", "follow best practices")
- No placeholder text — every line must reflect actual facts discovered from this repo
- Apply the test to every rule in CLAUDE.md: "Would removing this line cause Claude to make a mistake on this codebase? If not, cut it."
- Respect hard line limits: CLAUDE.md ≤ 80 lines, architecture.md ≤ 150 lines, agents.md ≤ 150 lines

**.claude/CLAUDE.md** — entry point only, uses `@` imports:
```
# [Project name]

## Commands
- Build: `[exact command]`
- Dev: `[exact command]`
- Test: `[exact command]`
- Lint: `[exact command]`

## Rules
- [Codebase-specific rule]
- [Codebase-specific rule]
...

@.claude/docs/architecture.md
[@.claude/docs/agents.md — only if has_ai = true]
```

**.claude/docs/architecture.md** — tech stack, directory structure, data flow, key files, architectural decisions that affect how code is written.

**.claude/docs/agents.md** (only if `has_ai = true`) — AI/agent layer structure, prompt locations, models used, tool-calling or retrieval patterns, guardrails.

## Hook setup

After writing all `.claude/` files, create or update `.claude/settings.json` with a `PostToolUse` hook that reminds the user to run `/claudify:update` whenever a `.md` file is modified.

**If `.claude/settings.json` does not exist**, create it:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "file=$(cat | jq -r '.tool_input.file_path // empty'); [[ \"$file\" == *.md ]] && echo \"Reminder: MD file changed — run /claudify:update to keep CLAUDE.md in sync\""
          }
        ]
      }
    ]
  }
}
```

**If `.claude/settings.json` already exists**, read it first, then merge the `hooks.PostToolUse` entry in — do not overwrite existing keys (e.g. `permissions`).

---

## Phase 6 — Summary

After all writes are done, print:
- What was created / rewritten / moved / merged / deleted
- That a PostToolUse hook was added to `.claude/settings.json` to remind you to run `/claudify:update` on MD changes
- One next step for the user

> Run `/claudify:update` any time you want to refresh CLAUDE.md to match the current codebase.
> (Requires `claude-md-management@claude-plugins-official` — install once with `/plugin install claude-md-management@claude-plugins-official`)
