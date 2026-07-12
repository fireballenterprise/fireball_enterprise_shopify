# Fireball Enterprise Shopify Dawn Theme

Shopify Dawn theme fork for [Fireball Enterprise](https://fireballenterprise.com). Customized theme with Python Invoke tooling for local linting, git workflow automation, and AI Copilot prompts.

[![tests](https://github.com/fireballenterprise/fireball_enterprise_shopify/actions/workflows/tests.yml/badge.svg?branch=main)](https://github.com/fireballenterprise/fireball_enterprise_shopify/actions/workflows/tests.yml?query=branch%3Amain)

## Prerequisites

- [Python](https://www.python.org/) `>=3.14`
- [uv](https://docs.astral.sh/uv/) (dependency/environment management)
- [Shopify CLI](https://shopify.dev/docs/storefronts/themes/tools/cli) (`npm install -g @shopify/cli`)

## Setup

```sh
./setup.sh
```

Creates a `.venv` with `uv`, installs dependencies, and installs the Shopify CLI. Update `properties.yml` with the local repo path before running tasks.

## Project Structure

```
pyproject.toml         # Dependencies (Python >=3.14), ruff/pylint config
invoke.yml              # Invoke config (auto_dash_names: false)
setup.sh                # Initial environment setup (uv venv + uv sync)
properties.yml          # Project configuration (repo path, remote, Shopify store)
assets/                 # Theme JS, CSS, images (synced to Shopify)
config/                 # Theme settings schema (synced to Shopify)
layout/                 # Theme layout files (synced to Shopify)
locales/                # Translation strings (synced to Shopify)
sections/               # Theme sections (synced to Shopify)
snippets/               # Theme snippets (synced to Shopify)
templates/              # Theme templates (synced to Shopify)
modules/
  claude/               # sync.py — syncs .claude/commands/ from .github/prompts/ source of truth
  common/               # cli.py, properties.py, utils.py — shared helpers
  dawn/                 # list.py, version.py, upgrade.py — upstream Dawn tag listing, current version, and upgrade-staging
  repo/                 # pull.py, push.py, log.py, squash.py, rebase.py — git workflow modules
  shopify/              # pull.py, upgrade.py, env.py — Shopify theme workflow modules
  version/              # bump.py — bump the root VERSION file for dev deploys and releases
  versioning/           # lib.py, workflows.py — dependency/action version checks
tasks/
  __init__.py           # Wires the invoke Collection (repo, shopify, ruff, tests, uv, version, ver, dawn, fix, test)
  repo.py               # repo.pull, repo.push, repo.log, repo.squash, repo.rebase
  shopify.py             # shopify.pull, shopify.upgrade, shopify.fix, shopify.env
  ruff.py               # ruff.fix, ruff.format
  tests.py              # tests.actionlint, tests.pylint, tests.rufflint, tests.theme_check, tests.yamllint
  uv.py                 # uv.upgrade
  version.py            # version.bump_build, version.bump_release
  versioning.py         # ver.libs, ver.workflows, ver.all
  dawn.py               # dawn.list, dawn.version, dawn.upgrade
  combos.py             # Top-level aliases: fix, test
.github/
  instructions/         # Copilot instructions per concern
  prompts/               # /push, /pull, /squash, /rebase, /test, /fix, /shopify, /pr, /pr-notes,
                         # /punch-it-chewy, /versioning, /dawn, /sync-setup, /setup, /docs prompts
  workflows/            # Thin callers into fireballenterprise/workflows@1 (logic lives there)
    tests.yml           # PR to development: Ruff + Pylint + theme-check + yamllint + actionlint
    deploy.yml          # Push to development: bump VERSION build + deploy to dev theme (or prd manually)
    release.yml         # Manual: finalize VERSION + promote development → main + deploy prd + GitHub Release
    dawn_sync.yml       # Manual: sync dawn_vanilla with upstream Shopify/dawn
```

## Branch Strategy

This repo is a fork of [Shopify/dawn](https://github.com/Shopify/dawn). All three branches are protected.

| Branch | Purpose |
|--------|---------|
| `dawn_vanilla` | Tracks upstream `Shopify/dawn` — merge upstream releases here first |
| `development` | Active development branch — open PRs targeting this branch |
| `main` | Production — merges from `development` only, triggers prod deploy |

```
upstream/main → dawn_vanilla → development → main (production)
```

## Invoke Tasks

```sh
uv run --no-sync invoke                  # List all available tasks
uv run --no-sync invoke test             # Ruff + Pylint + theme-check + yamllint + actionlint
uv run --no-sync invoke fix               # Ruff autocorrect + format + theme-check autocorrect
uv run --no-sync invoke repo.pull         # Pull from git remote (stash → pull --rebase → restore)
uv run --no-sync invoke repo.push         # Push to git remote (fix → test → commit → push)
uv run --no-sync invoke repo.log          # Save a session log to logs/
uv run --no-sync invoke repo.squash       # Anchored squash all commits to root + optional force push
uv run --no-sync invoke repo.rebase       # Rebase onto remote default branch (optionally squash first)
uv run --no-sync invoke shopify.pull      # Pull live theme from Shopify store → local repo
uv run --no-sync invoke shopify.upgrade   # Upgrade dawn_vanilla from upstream Shopify/dawn, merge into development
uv run --no-sync invoke shopify.env       # Print Shopify CLI env var exports (eval "$(uv run --no-sync invoke shopify.env)")
uv run --no-sync invoke ver.libs       # Check pyproject.toml deps against latest releases, prompt to update
uv run --no-sync invoke ver.workflows  # Check .github/workflows/ action refs against latest majors, prompt to update
uv run --no-sync invoke ver.all        # Run every version check (libs, workflows)
uv run --no-sync invoke dawn.list      # List upstream Shopify/dawn tags, latest highlighted
uv run --no-sync invoke dawn.version   # Print the Dawn theme version currently checked out on this branch
uv run --no-sync invoke dawn.upgrade   # Stage a Dawn upgrade onto a review branch for manual conflict resolution
uv run --no-sync invoke version.bump_build      # Advance VERSION for a dev deploy (new minor's first build, or next build)
uv run --no-sync invoke version.bump_release  # Finalize VERSION for release by dropping the build suffix
uv run --no-sync invoke uv.upgrade            # Install the dependency versions currently locked in pyproject.toml
```

## Shopify CLI

```sh
# Load store domain + theme token into this shell session (values sourced from properties.yml)
eval "$(uv run --no-sync invoke shopify.env)"

# Sync live Shopify theme edits → local repo
shopify theme pull

# Push local theme changes → Shopify store
shopify theme push
```

After pulling, commit to keep the repo in sync with GUI edits made in the Shopify theme editor.

## AI Prompts

| Prompt | Command | Description |
|--------|---------|-------------|
| `/push` | `uv run --no-sync invoke repo.push` | Fix, test, commit, and push to git |
| `/pull` | `uv run --no-sync invoke repo.pull` | Stash, pull latest, restore stash |
| `/test` | `uv run --no-sync invoke test` | Run all linters |
| `/fix` | `uv run --no-sync invoke fix` | Auto-correct Ruff + theme-check offenses |
| `/squash` | `uv run --no-sync invoke repo.squash` | Anchored squash all commits to root |
| `/rebase` | `uv run --no-sync invoke repo.rebase` | Rebase onto remote default branch |
| `/shopify` | `uv run --no-sync invoke shopify.pull` / `shopify.upgrade` | Pull live theme or upgrade Dawn from upstream |
| `/versioning` | `uv run --no-sync invoke ver.all` | Check pyproject.toml/workflow action versions |
| `/dawn` | `uv run --no-sync invoke dawn.list` / `dawn.upgrade` | List upstream Dawn tags, or stage + help resolve a Dawn upgrade |
| `/setup` | `./setup.sh` | Run initial project setup |
| `/pr` | `uv run --no-sync invoke repo.pr_create` | Create a GitHub Pull Request into `development` |
| `/pr-notes` | `uv run --no-sync invoke repo.pr_diff` / `repo.pr_notes_save` | Draft PR notes from the branch diff |
| `/punch-it-chewy` | `uv run --no-sync invoke repo.push` + `/pr-notes` steps | Push the branch, then draft notes and open a PR |
| `/sync-setup` | `uv run --no-sync invoke skeleton.locate_source` | Sync shared tooling in from the skeleton repo |
| `/docs` | — | Audit instructions, `.claude/commands/` sync, and READMEs for drift |

## Modules

| Module | Purpose |
|--------|---------|
| `modules/claude/` | Syncs `.claude/commands/` from `.github/prompts/` source of truth |
| `modules/common/` | CLI helpers, `properties.yml` config reader, output/utility helpers |
| `modules/dawn/` | List upstream Shopify/dawn tags, stage a Dawn upgrade for manual review |
| `modules/repo/` | Git workflow automation (pull, push, log, squash, rebase) |
| `modules/shopify/` | Shopify theme pull, Dawn upstream upgrade, and CLI env var workflow |
| `modules/version/` | Bump the root `VERSION` file for dev deploys and releases |
| `modules/versioning/` | Dependency lock and workflow action-ref checks |

See [modules/README.md](modules/README.md) for full details.

## CI

GitHub Actions runs Ruff, Pylint, theme-check, yamllint, and actionlint on PRs into `development` via the `tests.yml` thin caller. Pushes to `development` bump the VERSION build and deploy to the dev theme; the manual Release workflow promotes `development` → `main`, deploys production, and publishes a GitHub Release. All logic lives in [fireballenterprise/workflows](https://github.com/fireballenterprise/workflows), referenced at the floating major tag `@1`.


