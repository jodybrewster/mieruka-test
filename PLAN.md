# Plan: SPDD + Mieruka Consulting Workflow POC

## Goal

Prove that SPDD (Structured Prompt-Driven Development) + Mieruka can serve as a full consulting delivery stack: structured, auditable development with a live client governance dashboard.

Reference: https://martinfowler.com/articles/structured-prompt-driven/

---

## The scenario

A senior solutions architect at a consultancy uses Claude Code to build a client-facing web app. Instead of disappearing and returning with code, they give the client:

- **Visibility** — what is being built and why, before code is written (SPDD artifacts)
- **Governance** — approval gates at key transitions (Discover → Define → Build → Ship)
- **Evidence** — auditable stories, REASONS canvases, design decisions
- **Progress** — a live Mieruka dashboard the client checks without reading code

**The sample app:** Daily Standup Tracker — team members log standups (did/doing/blockers), view a team feed, flag blockers. Small enough to build in a session; real enough to exercise the full SPDD workflow.

---

## Current state (what's done)

### Install flow — working ✓
- Bootstrap script installs `pro-starter@pro-dev-skillset` (pro-core, pro-quality, pro-design, pro-testing, pro-data, pro-spec, pro-nextjs, pro-mieruka)
- `pro-mieruka@0.1.4` provides `/init-mieruka`, `/start-mieruka`, `/stop-mieruka`
- `npx mieruka init` scaffolds `.mieruka/`, registers MCP server — no DESIGN.md created, no format prompt
- `npx mieruka dev` starts UI at http://127.0.0.1:7777
- `pro-spdd@0.1.0` (opt-in) provides all 9 SPDD commands

### Repos
| Repo | Path | Notes |
|------|------|-------|
| `mieruka-test` | here | sandbox; wipe with find command in CLAUDE.md |
| `pro-dev-skillset` | `../pro-dev-skillset` | marketplace; pro-mieruka at 0.1.4 |
| `mieruka` | `../mieruka/mieruka` | npm-linked locally (`npm link` already run) |

### Known rough edge
Bootstrap says "already installed" for pro-starter (cache) — workaround:
```bash
claude plugin marketplace update pro-dev-skillset
claude plugin install pro-mieruka@pro-dev-skillset --scope project
claude plugin install pro-spdd@pro-dev-skillset --scope project
```

---

## Phases ahead

### Phase 1 — Full install + SPDD dry run (next session, start here)

Start clean:
```bash
find . -mindepth 1 -maxdepth 1 ! -name 'CLAUDE.md' ! -name 'PLAN.md' ! -name '.git' -exec rm -rf {} +
```

Install:
```bash
bash <(gh api repos/jodybrewster/pro-dev-skillset/contents/templates/bootstrap.sh --jq .content | base64 -d)
claude plugin marketplace update pro-dev-skillset
claude plugin install pro-mieruka@pro-dev-skillset --scope project
claude plugin install pro-spdd@pro-dev-skillset --scope project
/init-mieruka
/start-mieruka
```

Run SPDD on the Daily Standup Tracker:
```
/spdd-story          → produce user stories
/spdd-analysis       → domain analysis, risks
/spdd-reasons-canvas → REASONS blueprint
/spdd-generate       → generate the app
/spdd-api-test       → tests tied to Canvas
```

Observe: what artifacts land where? What does Mieruka not yet show that it should?

---

### Phase 2 — Mieruka governance extensions (build work)

Add to Mieruka (`../mieruka/mieruka`):

| View | What it shows | Data source |
|------|--------------|-------------|
| **Stories** | User stories from `/spdd-story`; client can approve | `.mieruka/stories/` or MCP tool |
| **Canvas** | REASONS seven-part blueprint; read-only + comments | `.mieruka/canvas/` or MCP tool |
| **Approval gates** | Discover → Define → Build → Ship sign-off | `.mieruka/gates/` |
| **Decision register** | Every significant choice with rationale + approver | `.mieruka/decisions/` |
| **Progress feed** | Live timeline of SPDD steps, artifacts produced, blockers | activity_events table (already exists) |

MCP tools to add to Mieruka server:
- `mieruka.write_story` / `mieruka.list_stories` / `mieruka.approve_story`
- `mieruka.write_canvas` / `mieruka.read_canvas`
- `mieruka.request_approval` / `mieruka.record_approval`
- `mieruka.log_decision`

These let Claude Code (running SPDD commands) write structured artifacts into Mieruka during the workflow.

---

### Phase 3 — pro-spdd + Mieruka bridge skill

Add a skill or hook to `pro-spdd` (or a new `pro-mieruka-spdd` plugin) that:
- After each SPDD command, pushes the produced artifact to Mieruka via MCP
- E.g. after `/spdd-story` → calls `mieruka.write_story` for each story
- After `/spdd-reasons-canvas` → calls `mieruka.write_canvas`
- Records each step in the progress feed

This closes the loop: SPDD drives the work; Mieruka surfaces it to the client automatically.

---

## Key commands to remember

```bash
# Wipe and restart
find . -mindepth 1 -maxdepth 1 ! -name 'CLAUDE.md' ! -name 'PLAN.md' ! -name '.git' -exec rm -rf {} +

# Plugin installs
claude plugin marketplace update pro-dev-skillset
claude plugin install pro-mieruka@pro-dev-skillset --scope project
claude plugin install pro-spdd@pro-dev-skillset --scope project

# Mieruka
/init-mieruka
/start-mieruka
/stop-mieruka

# SPDD workflow (in order)
/spdd-story
/spdd-analysis
/spdd-reasons-canvas
/spdd-generate
/spdd-api-test
/spdd-sync
/spdd-code-review

# Update pro-mieruka after edits
cd ../pro-dev-skillset
claude plugin validate plugins/pro-mieruka --strict
git add plugins/pro-mieruka/ .claude-plugin/marketplace.json
git commit && git push
claude plugin tag plugins/pro-mieruka --push
# then in mieruka-test:
claude plugin uninstall pro-mieruka@pro-dev-skillset --scope project
claude plugin install pro-mieruka@pro-dev-skillset --scope project
```
