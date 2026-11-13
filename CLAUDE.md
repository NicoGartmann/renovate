# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) and AI coding assistants when working with code in this repository.

## What this repo is

Central self-hosted [Renovate](https://docs.renovatebot.com/) runner and modular shareable configuration preset repository for `NicoGartmann`'s projects (currently scanning itself and [`docker-config`](https://github.com/NicoGartmann/docker-config)).

## Architecture & Conventions

1. **Scheduled GitHub Action (`.github/workflows/renovate.yml`)**:
   - Runs every Monday at `05:00 UTC` (`06:00 CET` / `07:00 CEST`) and supports manual runs via `workflow_dispatch` (with optional `repositories` override, `dry_run` mode, and `log_level`).
   - Authenticates via a dedicated GitHub App (`actions/create-github-app-token@v2` using `RENOVATE_APP_ID` and `RENOVATE_APP_PRIVATE_KEY` repository secrets).
   - **Never set a `schedule` property inside `presets/*.json` or `renovate.json`**: Execution timing is controlled exclusively by the GitHub Actions cron trigger so manual `workflow_dispatch` runs can open PRs at any time.

2. **Two-Tier Preset Hierarchy (`presets/`)**:
   - `presets/base.json`: Global rules shared across all repositories (`config:recommended`, `:semanticCommits`, `:dependencyDashboard`, `automerge: false`, `prHourlyLimit: 0`, `prConcurrentLimit: 0`, `minimumReleaseAge: "3 days"`, `major-update` label).
   - Technology-specific presets (such as `presets/docker-compose.json`) extend `local>NicoGartmann/renovate//presets/base.json` and encapsulate ecosystem rules (e.g., `pinDigests: false`, custom regex versioning for `ghcr.io/servercontainers/samba`, warning labels/notes for database major upgrades).
   - Target repositories extend presets via `"github>NicoGartmann/renovate//presets/<preset>.json"`.

3. **Explicit Repository Opt-In (`config.js`)**:
   - `onboarding: false` and `requireConfig: 'required'` ensure Renovate only processes repositories listed in `config.js` that contain an explicit `renovate.json`.

4. **Language & Commit Style**:
   - Documentation (`README.md`) and preset descriptions/PR notes are written in German; `CLAUDE.md` is kept in English.
   - Use Conventional Commits with ticket scope when applicable (e.g., `feat(SUP-147): ...`, `docs(SUP-147): ...`).
