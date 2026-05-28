# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

`mieruka-test` is a sandbox for two things:

1. **Install flow testing** — iterating on the end-to-end installation of the `pro-dev-skillset` plugin marketplace and mieruka.
2. **SPDD + Mieruka proof of concept** — building a small real app through the SPDD framework while using Mieruka as a client-facing governance and observability layer. This proves the consulting workflow: structured, auditable development with a live dashboard clients can view, approve, and interact with.

## The consulting scenario

You are a senior solutions architect at a consultancy. A client has hired you to build a web application. Rather than disappearing for weeks and returning with code, you want to give the client:

- **Visibility** into what is being built and why, before a line of code is written
- **Governance** — structured approval gates at key transitions (discover → define → build → ship)
- **Evidence** — auditable artifacts (stories, REASONS canvases, design decisions) that explain every major choice
- **Progress** — a live dashboard they can check at any time without needing to read code or talk to Claude

**SPDD** (Structured Prompt-Driven Development) produces the artifacts. **Mieruka** surfaces them to the client.

See: https://martinfowler.com/articles/structured-prompt-driven/

## The sample app

To prove this workflow end-to-end, we will build a **Daily Standup Tracker** — a small web app where a team can log daily standups (what I did, what I'm doing, blockers), view a team feed, and flag blockers for visibility. Small enough to build in a session; real enough to exercise the full SPDD workflow.

## SPDD workflow (in order)

All commands come from `pro-spdd@pro-dev-skillset`:

| Command | What it produces |
|---------|-----------------|
| `/spdd-story` | INVEST-compliant user stories from requirements |
| `/spdd-analysis` | Domain analysis, risks, strategic framing |
| `/spdd-reasons-canvas` | Seven-part executable blueprint (Requirements, Entities, Approach, Structure, Operations, Norms, Safeguards) |
| `/spdd-generate` | Code generated from the Canvas |
| `/spdd-api-test` | Tests tied to Canvas behavior |
| `/spdd-prompt-update` | Update prompts when intent changes |
| `/spdd-sync` | Sync prompts ↔ code when either drifts |
| `/spdd-code-review` | Review generated code against the Canvas |
| `/spdd-reverse` | Reverse-engineer existing code back into Canvas artifacts |

## Mieruka governance extension (what needs to be built)

Mieruka currently surfaces design artifacts (DESIGN.md). To serve as a client governance layer for SPDD it needs new sections:

- **Stories** — view and approve user stories produced by `/spdd-story`
- **REASONS Canvas** — structured view of the seven-part blueprint; client can read and comment
- **Approval gates** — explicit sign-off at Discover → Define → Build → Ship transitions
- **Decision register** — log of every significant choice with rationale and who/what approved it
- **Progress timeline** — live feed of SPDD steps completed, artifacts produced, blockers raised

These views should read from `.mieruka/` files or MCP tool calls that Claude Code writes to during SPDD workflow execution.

## Sibling repos

| Repo | Path | Role |
|------|------|------|
| `pro-dev-skillset` | `../pro-dev-skillset` | Private Claude Code plugin marketplace — source for all `pro-*` plugins |
| `mieruka` | `../mieruka/mieruka` | Visual design companion — installs into any repo as `.mieruka/` |

## Install flow

### Step 1 — Bootstrap the plugin stack

```bash
bash <(gh api repos/jodybrewster/pro-dev-skillset/contents/templates/bootstrap.sh --jq .content | base64 -d)
```

Installs `pro-starter@pro-dev-skillset` → cascades to `pro-core`, `pro-quality`, `pro-nextjs`, `pro-design`, `pro-testing`, `pro-data`, `pro-spec`, `pro-mieruka`.

Verify: `claude plugin list`

If `pro-mieruka` doesn't appear, refresh the cache:

```bash
claude plugin marketplace update pro-dev-skillset
claude plugin install pro-mieruka@pro-dev-skillset --scope project
```

### Step 2 — Install pro-spdd

`pro-spdd` is opt-in (not in `pro-starter`). Install it separately:

```bash
claude plugin marketplace update pro-dev-skillset
claude plugin install pro-spdd@pro-dev-skillset --scope project
```

Verify: `claude plugin list | grep pro-spdd`

### Step 3 — Initialize mieruka

```
/init-mieruka
```

Or non-interactively:

```bash
npx mieruka init --mcp-path /path/to/design-inspiration-mcp/dist/index.js --serper-key YOUR_KEY
```

Scaffolds `.mieruka/` (SQLite + config) and registers the MCP server at `http://127.0.0.1:7777/mcp`.

> **Note:** mieruka is not yet published to npm. Run `npm link` inside `../mieruka/mieruka` once so `npx mieruka` works as if it were published.

### Step 4 — Start mieruka

```
/start-mieruka
```

Or: `npx mieruka dev` → http://127.0.0.1:7777

### Step 5 — Begin SPDD workflow

With everything running, start the SPDD process for the sample app:

```
/spdd-story     # decompose requirements into stories
/spdd-analysis  # domain analysis and risk framing
/spdd-reasons-canvas  # produce the REASONS blueprint
/spdd-generate  # generate code from the Canvas
```

## pro-mieruka commands

| Command | What it does |
|---------|-------------|
| `/init-mieruka` | Runs `npx mieruka init` — scaffolds `.mieruka/`, registers MCP |
| `/start-mieruka` | Runs `npx mieruka dev` — starts UI at http://127.0.0.1:7777 |
| `/stop-mieruka` | Kills the mieruka dev server |

To modify `pro-mieruka`: edit `../pro-dev-skillset/plugins/pro-mieruka/`, bump versions in `plugin.json` and `marketplace.json`, validate, commit, tag, and reinstall. See `../pro-dev-skillset/RELEASING.md`.

## Reset / wipe

```bash
find . -mindepth 1 -maxdepth 1 ! -name 'CLAUDE.md' ! -name '.git' -exec rm -rf {} +
```

Wipes everything except this file and git history. Re-run from Step 1.

## Troubleshooting

- **`pro-mieruka` or `pro-spdd` not in plugin list** — run `claude plugin marketplace update pro-dev-skillset` then install explicitly.
- **Commands not found after install** — restart Claude Code; commands load on session start.
- **`npx mieruka` fails** — mieruka isn't published yet. Run `npm link` inside `../mieruka/mieruka`.
- **Port 7777 in use** — `npx mieruka dev -p 7778` and re-register: `claude mcp add --transport http --scope project mieruka http://127.0.0.1:7778/mcp`.
