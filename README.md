# Skills Collection

Agent skills for Claude Code.

## Structure

- `official/` — Built-in skills extracted from Claude Code source
- `anthropic-skills/` — Git submodule of [anthropics/skills](https://github.com/anthropics/skills)
- `docs/` — Auto-synced documentation (updated every 3 days via GitHub Actions)
  - `docs/agentskills.io/` — Agent Skills specification and guides
  - `docs/platform.claude.com/` — Anthropic platform skill docs

## Auto-sync

The `.github/workflows/sync-docs.yml` workflow runs every 3 days to pull latest docs from agentskills.io and platform.claude.com. Can also be triggered manually via workflow_dispatch.
