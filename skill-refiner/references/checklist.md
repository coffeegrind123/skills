# Skill Validation Checklist — Detailed Reference

Full validation criteria for Agent Skills. Each item includes the rule, why it matters, and how to fix violations.

## Frontmatter Validation

### name (REQUIRED)
- **Rule**: 1-64 characters, lowercase `a-z`, numbers `0-9`, hyphens `-` only
- **Rule**: No leading hyphens (`-my-skill`), trailing hyphens (`my-skill-`), or consecutive hyphens (`my--skill`)
- **Rule**: Must exactly match the parent directory name
- **Why**: Agents discover skills by directory name; mismatch = skill won't load
- **Fix**: Rename directory to match `name`, or update `name` to match directory. Lowercase everything.

### description (REQUIRED)
- **Rule**: 1-1024 characters, non-empty
- **Rule**: Must describe WHAT the skill does AND WHEN to use it
- **Rule**: Should include trigger phrases (example user messages that should activate this skill)
- **Why**: This is the ONLY text loaded at startup for all skills (~100 tokens). It's how the agent decides whether to activate a skill. Vague descriptions = skill never triggers.
- **Fix**: Append "Use when [scenarios]. Examples: '[trigger]', '[trigger]'." to the description.

**Bad**: `"Helps with database migrations"`
**Good**: `"Run database migrations with rollback support, seed data, and schema validation. Use when the user mentions migrations, database schema changes, or says 'migrate', 'rollback', 'seed the db'."`

### allowed-tools
- **Rule**: Space-delimited list of tool permission patterns
- **Rule**: Use narrow patterns: `Bash(git:*)` not `Bash`, `Bash(npm test:*)` not `Bash(npm:*)`
- **Why**: Pre-approves tools so the skill runs without permission prompts. Overly broad = security risk.
- **Fix**: Analyze commands in each step. Map to narrowest pattern that covers them.

**Pattern syntax**:
- `Read` — file read tool
- `Write` — file write tool  
- `Edit` — file edit tool
- `Bash(git:*)` — any git command
- `Bash(npm test:*)` — npm test only
- `Bash(gh pr:*)` — GitHub PR commands
- `Bash(python:*)` — any python command
- `Bash(*)` — any bash command (last resort)

### when_to_use
- **Rule**: Should start with "Use when..."
- **Rule**: Include 3+ example trigger phrases
- **Why**: Separate from `description` — this field is specifically for auto-invocation matching
- **Fix**: Write "Use when the user wants to [action]. Examples: '[phrase1]', '[phrase2]', '[phrase3]'."

### arguments
- **Rule**: YAML list of argument names
- **Rule**: Every `$arg_name` in the body must have a matching entry in `arguments`
- **Rule**: Every entry in `arguments` should be referenced as `$name` somewhere in the body
- **Why**: Orphaned arguments confuse the agent; missing entries mean substitution fails
- **Fix**: Align the two lists. Remove unused arguments. Add missing `$` references.

### context
- **Rule**: Either `fork` or omit entirely
- **Rule**: Do NOT write `context: inline` — inline is the default
- **Why**: `fork` spawns a subagent with its own context. Inline runs in the current conversation.
- **Fix**: Remove `context: inline`. Only set `context: fork` for self-contained skills.

### argument-hint
- **Rule**: Shows placeholder text for arguments
- **Rule**: Use square brackets for optional, angle brackets for required
- **Why**: Displayed in autocomplete/help to guide the user
- **Example**: `"[PR number] [target branch]"` or `"<input file>"`

## Body Validation

### Token Budget
- **Rule**: Body should be < 500 lines AND < 5,000 tokens
- **Estimate**: tokens ~= lines x 4 (rough average for markdown)
- **Why**: Full body loads into context when skill triggers. Large skills crowd out conversation.
- **Fix**: Move detailed reference material to `references/` directory. Add conditional loading triggers: "Read `references/X.md` if you encounter Y."

### Step Structure
- **Rule**: Steps use `### N. Step Name` format
- **Rule**: Sequential: `1.`, `2.`, `3.` (no gaps)
- **Rule**: Parallel: `3a.`, `3b.` (run concurrently)
- **Rule**: Human: `### 4. Step Name [human]` (user performs this step)
- **Why**: Agents parse step structure for execution planning. Broken numbering = confused execution.
- **Fix**: Renumber steps. Add sub-letters for parallel steps.

### Success Criteria
- **Rule**: EVERY step must have a `**Success criteria**:` line
- **Why**: Without it, the agent doesn't know when a step is done. It may skip steps or loop forever.
- **Fix**: Add success criteria based on what the step produces or verifies. Be specific: "CI pipeline passes" not "things work."

### Annotations
These are optional per step but should be used when relevant:

- **Artifacts**: Data the step produces for later steps. Example: "The PR URL"
- **Human checkpoint**: When to pause for user confirmation. Use for irreversible actions.
- **Rules**: Hard constraints. Example: "Never force-push to main"
- **Execution**: How to run: Direct / Task agent / Teammate / [human]. Only specify if not Direct.

## File Structure Validation

### Directory Name
- **Rule**: Directory name must match the `name` frontmatter field exactly
- **Why**: Claude Code discovers skills by scanning directories

### Reference Files
- **Rule**: Every file referenced in the body must exist on disk
- **Rule**: References should have conditional loading triggers (not just "see references/")
- **Why**: Unconditional references waste tokens. Missing files cause errors.
- **Fix**: Add "Read `references/X.md` if/when [condition]." Remove references to nonexistent files.

### File Depth
- **Rule**: Keep supporting files one level deep: `references/file.md`, `scripts/run.sh`
- **Rule**: Avoid deep nesting: `references/deep/nested/file.md`
- **Why**: Simpler paths = fewer agent errors

### Scripts
- **Rule**: Scripts in `scripts/` should be self-contained
- **Rule**: Include shebangs (`#!/usr/bin/env python3`)
- **Rule**: Document dependencies if any
- **Why**: Agent runs scripts directly; missing deps = runtime failure

## Quality Validation

### Specificity
- **Rule**: Instructions should be actionable, not vague
- **Bad**: "Handle errors appropriately"
- **Good**: "If the API returns 429, wait 60 seconds and retry. If it returns 4xx, log the error body and skip."

### Context Spending
- **Rule**: Don't explain general knowledge (what git is, how HTTP works)
- **Rule**: DO explain project-specific conventions, edge cases, gotchas
- **Why**: General knowledge wastes tokens. Project-specific knowledge is what makes skills valuable.

### Gotchas Section
- **Rule**: Non-trivial skills should have a gotchas section
- **Content**: Environment facts, naming inconsistencies, soft deletes, timezone handling, etc.
- **Why**: Highest-value content — things the agent would get wrong without explicit instruction

### Validation Loops
- **Rule**: Complex skills should include a validation step (do work -> check -> fix -> repeat)
- **Why**: Agent self-correction is more reliable than hoping for perfect output
