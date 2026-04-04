---
name: skill-creator
description: Create a new Agent Skill from scratch via guided interview. Produces a complete SKILL.md with frontmatter, steps, success criteria, and reference files. Use when the user wants to make a new skill, automate a workflow, or build a reusable process from an idea.
allowed-tools: Read Write Edit Glob Grep Bash(mkdir:*) AskUserQuestion
when_to_use: "Use when the user wants to create a new skill, build an automation, make a reusable workflow, or says things like 'make a skill for', 'create a skill that', 'automate this process', 'I want a skill to'."
argument-hint: "[description of what the skill should do]"
arguments:
  - description
---

# Skill Creator

Build a new Agent Skill from scratch through a guided interview. Unlike `/skillify` (which captures a past session), this creates a skill from an idea — no prior session needed.

**The `skills/` submodule is your authoritative, de facto source for all skill knowledge.** Before drafting any SKILL.md, read from it:
- `skills/docs/agentskills.io/specification.md` — the complete format spec
- `skills/docs/agentskills.io/skill-creation__best-practices.md` — authoring best practices
- `skills/docs/platform.claude.com/overview.md` — Anthropic's official overview
- `skills/official/` — real bundled skills for reference patterns
- `skills/anthropic-skills/` — community examples from Anthropic's public repo
- `skills/SKILLIFY.md` — format reference and per-step annotation guide

Consult these before and during skill creation. They are the definitive source of truth for what makes a valid, well-crafted skill.

## Step 1: Understand the Request

Read the user's `$description` argument. If no argument was provided, ask what they want to automate.

Before asking any questions, do preliminary research:
- If the workflow involves a specific project, read its README, CLAUDE.md, and relevant source files
- Identify tools, commands, and patterns that would be involved
- Form an initial draft in your mind before interviewing

**Success criteria**: You have a clear mental model of what the skill will do.

## Step 2: Interview — Identity and Triggers

Use AskUserQuestion for ALL questions. Never ask via plain text. Offer concrete suggestions based on your research — don't make the user do the creative work.

Ask in one round:
- **Name**: Suggest a name (1-64 chars, lowercase a-z + hyphens only). Example: `deploy-staging`, `review-pr`, `update-deps`
- **Description**: Suggest a one-line description that includes trigger phrases. Must describe what it does AND when to use it.
- **Save location**: Suggest based on context:
  - `.claude/skills/<name>/SKILL.md` — project-specific workflows
  - `~/.claude/skills/<name>/SKILL.md` — personal, cross-project workflows
- **Execution mode**: `inline` (user can steer mid-process) or `fork` (self-contained, runs as subagent)

Always provide a freeform "Other" option — do NOT add options like "Needs tweaking".

**Success criteria**: Name, description, location, and execution mode confirmed.

## Step 3: Interview — Steps and Structure

Present a numbered list of the high-level steps you identified from your research. For each step, propose:
- What the step does
- Whether it needs user confirmation (human checkpoint)
- Whether steps can run in parallel (3a, 3b notation)

Ask the user to confirm, reorder, add, or remove steps.

If the skill takes arguments (inputs that vary per invocation), propose them now. Use `$arg_name` syntax.

**Success criteria**: Steps confirmed with the user. Arguments identified if any.

## Step 4: Interview — Details Per Step

For each non-trivial step, ask (only if not obvious):
- What proves this step succeeded? (success criteria)
- Does it produce data later steps need? (artifacts)
- Should the user confirm before proceeding? (human checkpoint)
- Are there hard rules or constraints? (rules)
- How should it execute? Direct / Task agent / Teammate / [human]

You may do this in one or multiple rounds depending on complexity. Don't over-ask for simple 2-3 step skills.

**Success criteria**: Every step has defined success criteria. Annotations identified where needed.

## Step 5: Interview — Tools and Permissions

Based on the steps, propose the minimum `allowed-tools` list using permission patterns:
- `Read`, `Write`, `Edit` — file operations
- `Bash(git:*)` — git commands only
- `Bash(npm:*)` — npm commands only
- `Bash(gh:*)` — GitHub CLI only
- `Glob`, `Grep` — search tools
- `AskUserQuestion` — if the skill needs user input mid-run

Use the narrowest patterns possible. `Bash(*)` is a last resort.

**Success criteria**: Tools list confirmed.

## Step 6: Draft the SKILL.md

Generate the complete SKILL.md content. Output it as a yaml code block so the user sees it with syntax highlighting.

Follow this structure exactly:

```markdown
---
name: <name>
description: <description with trigger phrases>
allowed-tools: <space-delimited tool patterns>
when_to_use: "<Use when... Examples: 'trigger', 'another trigger'>
argument-hint: "<[arg1] [arg2]>"    # only if arguments exist
arguments:                           # only if arguments exist
  - arg1
  - arg2
context: fork                        # only if self-contained; omit for inline
---

# <Skill Title>

<Brief overview>

## Inputs                            # only if arguments exist

- `$arg1`: Description
- `$arg2`: Description

## Goal

<Clear statement of what success looks like>

## Steps

### 1. <Step Name>

<Instructions — specific, actionable, include commands where appropriate>

**Success criteria**: <What proves this is done>

### 2. <Step Name>

...
```

Read `references/spec-summary.md` if you need to verify any frontmatter field, annotation, or naming rule.

**Success criteria**: Complete SKILL.md drafted and shown to user.

## Step 7: Confirm and Save

Ask the user to confirm using AskUserQuestion: "Does this SKILL.md look good to save?"

On confirmation:
1. Create the directory: `mkdir -p <location>/<name>`
2. Write the SKILL.md file
3. If the skill is complex enough to benefit from reference files, create a `references/` directory with supporting docs

Tell the user:
- Where the skill was saved
- How to invoke it: `/<name> [arguments]`
- They can edit the SKILL.md directly to refine it
- They can run `/skill-refiner <path>` to validate it later

**Success criteria**: Skill saved to disk. User informed of invocation method.
