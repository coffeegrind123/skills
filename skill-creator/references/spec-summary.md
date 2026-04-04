# Agent Skills Spec — Quick Reference

Consult this when drafting a SKILL.md to verify fields, constraints, and best practices.

## Frontmatter Fields

| Field | Required | Constraints |
|-------|----------|-------------|
| `name` | YES | 1-64 chars, lowercase `a-z` + numbers + hyphens. No leading/trailing/consecutive hyphens. Must match parent directory name. |
| `description` | YES | 1-1024 chars. Must describe what it does AND when to use it. Include trigger phrases. |
| `allowed-tools` | NO | Space-delimited permission patterns. Use `Bash(git:*)` not `Bash`. |
| `when_to_use` | NO | Start with "Use when..." Include example trigger phrases. |
| `argument-hint` | NO | Show placeholders: `"[PR number] [target branch]"` |
| `arguments` | NO | YAML list of parameter names. Use `$name` in body for substitution. |
| `context` | NO | `fork` for self-contained; omit for inline. |
| `license` | NO | License name or file reference. |
| `compatibility` | NO | Environment requirements. |
| `metadata` | NO | Arbitrary key-value pairs (author, version). |

## Name Rules

- Lowercase only: `a-z`, `0-9`, `-`
- No uppercase, spaces, underscores
- No leading or trailing hyphens
- No consecutive hyphens (`my--skill` is invalid)
- 1-64 characters
- Must match parent directory name exactly

## Per-Step Annotations

| Annotation | Required | Description |
|------------|----------|-------------|
| **Success criteria** | YES (every step) | What proves the step is complete |
| **Execution** | NO | `Direct` (default), `Task agent`, `Teammate`, `[human]` |
| **Artifacts** | NO | Data produced for later steps (IDs, URLs, paths) |
| **Human checkpoint** | NO | Pause for user confirmation (irreversible actions) |
| **Rules** | NO | Hard constraints that must not be violated |

## Step Notation

- Sequential: `1.`, `2.`, `3.`
- Parallel: `3a.`, `3b.` (run concurrently)
- Human: Add `[human]` to title: `### 4. Deploy [human]`

## Token Budget

- SKILL.md body should be < 500 lines, < 5,000 tokens
- Move detailed reference material to `references/` directory
- Tell agent WHEN to load each reference file (not just "see references/")

## Description Best Practices

Bad: `"Helps with deployments"`

Good: `"Deploy to staging and production with rollback support. Use when the user wants to deploy, push to staging, release to prod, or mentions deployment pipelines. Examples: 'deploy to staging', 'push this to prod', 'release v2.1'."`

## Allowed-Tools Patterns

Narrow patterns are better:
- `Bash(git:*)` — any git command
- `Bash(npm test:*)` — npm test only
- `Bash(gh pr:*)` — GitHub PR commands only
- `Read Write Edit` — file operations
- `Bash(*)` — last resort (allows any command)

## Progressive Disclosure

| Level | When Loaded | Budget |
|-------|------------|--------|
| Metadata (name + description) | Always (all skills) | ~100 tokens |
| SKILL.md body | When triggered | < 5,000 tokens |
| Reference files | On demand | Unlimited |
