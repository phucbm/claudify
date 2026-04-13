---
name: init
description: Bootstrap a clean .claude/ folder structure for any existing repo following official Claude Code best practices
disable-model-invocation: true
allowed-tools: Read Glob Bash Write Edit
---

Bootstrap a well-structured `.claude/` folder for this repo. Every file you generate must reflect facts discovered from the actual codebase — no generic advice, no placeholder text.

## Step 1 — Read the repo

Collect context in parallel:

1. **File tree** — run `find . -maxdepth 2 -not -path './.git/*' -not -path './node_modules/*' -not -path './.claude-plugin/*' | sort`
2. **Manifest files** — read whichever exist: `package.json`, `composer.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`
3. **README** — read `README.md`, `README.rst`, or `README.txt` if present
4. **Existing .claude/** — if the directory exists, list its contents and read all `.md` files inside
5. **Entry files** — based on detected stack, read the main entry point(s): `src/index.ts`, `src/main.ts`, `main.py`, `app.py`, `main.go`, `index.php`, `app.rb`, `lib/index.js`, or equivalent

## Step 2 — Detect

From what you read, determine:

- **Language & framework** — e.g. "TypeScript + Next.js 14", "Python + FastAPI", "PHP + WordPress + ACF"
- **Build / test / lint / dev commands** — extract exact commands from `scripts` in manifests; do not guess
- **AI / LLM / agent features** — search imports and dependencies for: `openai`, `anthropic`, `langchain`, `llama`, `ollama`, `transformers`, `langfuse`, `llamaindex`, vector store clients (pinecone, weaviate, chroma), agent orchestration patterns. Set `has_ai = true` if found.
- **Existing .claude/ quality** — classify as: missing, minimal/empty, scattered (has content but unstructured), or well-structured

## Step 3 — Safety check (if .claude/ already exists with content)

If `.claude/` exists and contains non-trivial markdown files:

1. List all current files in `.claude/`
2. List every file you plan to create or overwrite
3. Show the user: **"Found existing `.claude/`. I'll create/overwrite: [list]. Proceed? (yes/no)"**
4. Wait for confirmation. If declined, stop and tell the user which files they can create manually.

## Step 4 — Generate files

### .claude/CLAUDE.md

Entry point only. Reference docs with `@` imports.

Required sections:
- **Commands** — only commands confirmed to exist in the manifest (skip any that don't exist)
- **Rules** — 3–8 concrete rules specific to this codebase that would prevent real mistakes; apply the test: "Would removing this cause Claude to make a mistake here? If not, cut it."
- **Imports** — `@.claude/docs/architecture.md` always; `@.claude/docs/agents.md` only if `has_ai = true`

Template (adapt content to actual repo):
```
# [Project name]

## Commands
- Build: `[exact command]`
- Dev: `[exact command]`
- Test: `[exact command]`
- Lint: `[exact command]`

## Rules
- [Concrete rule derived from this codebase]
- [Concrete rule derived from this codebase]
...

@.claude/docs/architecture.md
```

Hard limit: **80 lines**.

### .claude/docs/architecture.md

Include only what is true about this repo:
- **Stack** — language version, framework, key libraries with their roles
- **Structure** — top-level directories and what lives in each
- **Data flow** — how a typical request or main operation moves through the system
- **Key files** — the 5–10 files Claude is most likely to touch and what each does
- **Decisions** — architectural choices that affect how code should be written (e.g. "SSR not CSR", "uses edge runtime", "no ORM — raw SQL via pg")

Hard limit: **150 lines**.

### .claude/docs/agents.md (only if `has_ai = true`)

Include:
- How the AI/agent layer is structured
- Where prompts live and how they are managed
- Models used and configuration
- Tool-calling or retrieval patterns
- Any rate limits, cost controls, or guardrails

Skip entirely if `has_ai = false`.

Hard limit: **150 lines**.

## Step 5 — Write the files

Use the Write tool to create each file. Create the `docs/` subdirectory if needed.

After writing, print a concise summary:
- Files created
- Files skipped and why
- One suggested next step

---

> To keep your CLAUDE.md files healthy over time, install the official maintenance plugin:
> `/plugin install claude-md-management@claude-plugins-official`
