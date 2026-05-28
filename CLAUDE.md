# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

`mieruka-test` is an empty sandbox for iterating on the end-to-end installation of the `pro-dev-skillset` plugin marketplace and the `mieruka` visual design companion. Nothing is built here — this repo exists purely to test and refine the install flow.

## Sibling repos

| Repo | Path | Role |
|------|------|------|
| `pro-dev-skillset` | `../pro-dev-skillset` | Private Claude Code plugin marketplace — source for all `pro-*` plugins |
| `mieruka` | `../mieruka/mieruka` | Visual design companion app — installs into any repo as `.mieruka/` |

## Install flow

### Step 1 — Bootstrap the pro-dev-skillset plugin stack

```bash
bash <(gh api repos/jodybrewster/pro-dev-skillset/contents/templates/bootstrap.sh --jq .content | base64 -d)
```

This writes `.claude/settings.json` and installs `pro-starter@pro-dev-skillset`, which cascades to the full skill stack: `pro-core`, `pro-quality`, `pro-nextjs`, `pro-design`, `pro-testing`, `pro-data`, `pro-spec`, and `pro-mieruka`.

Verify with: `claude plugin list`

If `pro-mieruka` doesn't appear or shows as "not found", refresh the marketplace cache first:

```bash
claude plugin marketplace update pro-dev-skillset
claude plugin install pro-mieruka@pro-dev-skillset --scope project
```

### Step 2 — Initialize mieruka with `/init-mieruka`

Once `pro-mieruka` is installed (included via `pro-starter`), the `/init-mieruka` command is available in this Claude Code session. Run it:

```
/init-mieruka
```

Behind the scenes, this resolves the mieruka binary (local build until it's published to npm) and runs `mieruka init`, which:

1. Checks Node 20+ and the `claude` CLI
2. Asks for the design-inspiration MCP — a local path or a git URL (it clones, installs, and builds)
3. Asks for a [Serper.dev](https://serper.dev) API key (free tier works; stored at `.mieruka/config.json`)
4. Runs `npx playwright install chromium`
5. Scaffolds `.mieruka/` (SQLite + config), writes a starter `DESIGN.md`, appends `.mieruka/` to `.gitignore`
6. Registers the MCP server: `claude mcp add --transport http --scope project mieruka http://127.0.0.1:7777/mcp`

For non-interactive use, pass flags directly:

```bash
node ~/Projects/mieruka/mieruka/bin/mieruka.mjs init --mcp-path /path/to/design-inspiration-mcp/dist/index.js --serper-key YOUR_KEY
```

> **Note:** mieruka is not yet published to npm. The `/init-mieruka` command resolves the local build automatically. Once published, `npx mieruka init` will work directly.

### Step 3 — Start mieruka

```bash
node ~/Projects/mieruka/mieruka/bin/mieruka.mjs dev
```

Opens at http://127.0.0.1:7777.

## Reset / wipe

When iterating on the install flow, wipe everything in this repo except this file and git history:

```bash
find . -mindepth 1 -maxdepth 1 ! -name 'CLAUDE.md' ! -name '.git' -exec rm -rf {} +
```

Run from the root of this repo. After wiping, re-run the install flow from Step 1.

## pro-mieruka plugin

`pro-mieruka` lives at `../pro-dev-skillset/plugins/pro-mieruka/`. It is a dependency of `pro-starter`, so installing `pro-starter` gives you `/init-mieruka` automatically.

It provides one command:

- **`/init-mieruka`** — installs mieruka into the current project

To modify `pro-mieruka`:

1. Edit files in `../pro-dev-skillset/plugins/pro-mieruka/`
2. Bump `version` in `plugin.json` and in `..pro-dev-skillset/.claude-plugin/marketplace.json`
3. Bump `pro-starter` version too (since it depends on pro-mieruka)
4. Validate: `claude plugin validate ../pro-dev-skillset/plugins/pro-mieruka --strict`
5. In this repo, update the installed copy: `claude plugin update`

See `../pro-dev-skillset/RELEASING.md` for the full version-bump law.

## Troubleshooting

- **`claude plugin list` doesn't show pro-mieruka** — the plugin cache may be stale. Run `claude plugin update` or re-run the bootstrap script.
- **`/init-mieruka` command not found** — open Claude Code fresh after installing (`claude plugin list` should show `pro-mieruka@pro-dev-skillset`).
- **`npx mieruka init` fails with "package not found"** — mieruka is not yet published to npm. `/init-mieruka` uses the local build at `../mieruka/mieruka/bin/mieruka.mjs` automatically.
- **Port 7777 in use** — run `npx mieruka dev -p 7778` and re-register the MCP: `claude mcp add --transport http --scope project mieruka http://127.0.0.1:7778/mcp`.
