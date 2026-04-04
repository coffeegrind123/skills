# How /skillify Works

`/skillify` captures a session's repeatable process into a reusable SKILL.md file. Run it at the end of a task you want to automate.

## Usage

```
/skillify [optional description of the process]
```

## Flow

1. **Analyzes your session** — reads session memory + all user messages to understand the repeatable process
2. **Interviews you in 4 rounds** via interactive questions:
   - **Round 1:** Confirm skill name, description, goals, success criteria
   - **Round 2:** Confirm high-level steps, arguments, inline vs forked execution, save location
   - **Round 3:** Break down each step — success criteria, parallelism, human checkpoints, constraints
   - **Round 4:** Trigger phrases for auto-invocation, gotchas
3. **Writes SKILL.md** with frontmatter + structured steps
4. **Shows output** for review before saving

## Save Locations

- **This repo:** `.claude/skills/<name>/SKILL.md` — for project-specific workflows
- **Personal:** `~/.claude/skills/<name>/SKILL.md` — follows you across all repos

## SKILL.md Format

```markdown
---
name: skill-name
description: one-line description
allowed-tools:
  - Bash(gh:*)
  - Read
  - Write
when_to_use: "Use when the user wants to... Examples: 'trigger phrase', 'another phrase'"
argument-hint: "[PR number] [target branch]"
arguments:
  - pr_number
  - target_branch
context: fork  # omit for inline execution
---

# Skill Title

Description of what this skill does.

## Inputs

- `$pr_number`: The PR to cherry-pick
- `$target_branch`: The release branch to target

## Goal

Clearly stated goal with defined success artifacts.

## Steps

### 1. Step Name

What to do. Be specific and actionable. Include commands.

**Success criteria**: What proves this step is done.
**Artifacts**: Data later steps need (PR number, commit SHA, etc.)

### 2. Another Step

Instructions for this step.

**Success criteria**: Verification that it worked.
**Human checkpoint**: Pause and ask before proceeding (for irreversible actions).

### 3a. Parallel Step A

Steps with sub-numbers (3a, 3b) run concurrently.

### 3b. Parallel Step B

Runs at the same time as 3a.

### 4. Final Step [human]

Steps with [human] in the title require the user to act.

**Rules**: Hard constraints for this step.
```

## Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill name (used as `/name` command) |
| `description` | Yes | One-line description |
| `allowed-tools` | Yes | Minimum tool permissions (use patterns like `Bash(gh:*)` not `Bash`) |
| `when_to_use` | Yes | When Claude should auto-invoke. Start with "Use when..." + trigger phrases |
| `argument-hint` | No | Hint showing argument placeholders |
| `arguments` | No | List of argument names (use `$name` in body for substitution) |
| `context` | No | `fork` for self-contained tasks, omit for inline |

## Per-Step Annotations

| Annotation | Required | Description |
|------------|----------|-------------|
| **Success criteria** | Yes (every step) | What proves the step is done |
| **Execution** | No | `Direct` (default), `Task agent`, `Teammate`, or `[human]` |
| **Artifacts** | No | Data this step produces for later steps |
| **Human checkpoint** | No | Pause for user confirmation (irreversible actions) |
| **Rules** | No | Hard constraints — especially from user corrections during the reference session |

## Execution Modes

- **Direct** (default): Claude executes the step itself
- **Task agent**: Spawns a subagent for straightforward tasks
- **Teammate**: Agent with true parallelism and inter-agent communication
- **[human]**: User performs this step manually

## Tips

- Steps that can run concurrently use sub-numbers: 3a, 3b
- Keep simple skills simple — a 2-step skill doesn't need annotations on every step
- Pay attention to places where the user corrected you during the session — those become **Rules**
- `$ARGUMENTS` in the skill body gets replaced with whatever the user types after `/skill-name`
- `${CLAUDE_SKILL_DIR}` resolves to the skill's own directory (for bundled scripts)
