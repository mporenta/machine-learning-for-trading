---
name: create-agent-skills
description: Guides the agent through creating well-structured Claude Code Agent Skills by following a discovery, design, and generation workflow. Use when asked to create a skill, build a skill, make a new skill, or when the user wants to package a workflow or capability as a reusable skill. Requires reading relevant repo files to understand context before generating skill content.
allowed-tools: [Read, Write, Glob, Grep, Bash, Edit, WebSearch, WebFetch, SlashCommand, AskUserQuestion]
---

# Creating Claude Code Agent Skills

This skill guides you through creating well-structured, effective SKILL.md files for Claude Code. Follow the phases in order—do not skip discovery or design.

## Quick Start Checklist

1. [ ] Ask discovery questions (Phase 1)
2. [ ] Read context files provided by user
3. [ ] Propose skill design for approval (Phase 2)
4. [ ] Generate skill files (Phase 3)
5. [ ] Run validation tests (Phase 4)

---

## Phase 1: Discovery

**Do not write any skill content until you complete discovery.**

### Step 1.1: Ask Core Questions

Use `AskUserQuestion` to gather:

1. "What specific capability or workflow should this skill enable?"
2. "What triggers should cause the skill to activate? (keywords, file types, task types)"
3. "Will this skill need supporting files (scripts, templates, reference docs) or is it instructions-only?"

### Step 1.2: Gather Context Files

Ask: "What files should I read to understand the context for this skill? Provide paths to any relevant code, documentation, or examples."

Then use `Read` to examine each file. Look for:
- Existing patterns and conventions
- Domain-specific terminology
- Common workflows or decision points
- Error handling patterns
- Naming conventions

### Step 1.3: Follow-up Questions (As Needed)

Based on responses, ask relevant follow-ups:

| If the skill involves... | Ask about... |
|--------------------------|--------------|
| Code/scripts | Language, dependencies, execute vs. reference? |
| Multiple workflows | Decision points, when to choose path A vs B? |
| Domain-specific terms | Terminology, conventions, common mistakes? |
| Team sharing | Existing conventions, naming patterns, validation? |

### Step 1.4: Clarify Scope

Ask:
- "Is this a focused single-capability skill or multiple related tasks?"
- "What should be explicitly OUT of scope?"

---

## Phase 2: Skill Design

### Step 2.1: Propose Design

Present to the user for approval:

**1. Skill Name**
- Lowercase letters, numbers, hyphens only
- Max 64 characters
- Gerund form preferred (e.g., `processing-pdfs`, `managing-workflows`)
- Never include "anthropic" or "claude"

**2. Description**
- Max 1024 characters
- Third person ("Guides the agent..." not "Guide the agent...")
- Must include WHAT it does AND WHEN to use it
- No XML tags

**3. File Structure**
- Single SKILL.md for simple skills
- Multi-file with progressive disclosure for complex skills
- Keep SKILL.md body under 500 lines

**4. Allowed Tools** (if restrictions appropriate)
- Only specify if the skill should NOT have access to certain tools
- Example: `allowed-tools: [Read, Write, Glob]` excludes Bash

### Step 2.2: Get Confirmation

Do not proceed to generation until the user confirms the design.

---

## Phase 3: Generation

### Step 3.1: Create Skill Directory
```
.claude/skills/{skill-name}/
```

### Step 3.2: Write SKILL.md

Follow the template structure:
```markdown
---
name: {skill-name}
description: {description}
allowed-tools: [{tools}]  # Optional - omit if no restrictions
---

# {Skill Title}

Brief overview of what this skill does.

## Quick Start

Immediate, actionable steps to use this skill.

## Main Workflow

### Step 1: {First Step}
{Instructions}

### Step 2: {Second Step}
{Instructions}

## Examples

### Example: {Scenario Name}
{Concrete, copy-paste example}

## Reference

For detailed guidelines, see [BEST-PRACTICES.md](BEST-PRACTICES.md).
```

### Step 3.3: Write Supporting Files (If Needed)

For each supporting file:
- Keep references one level deep (SKILL.md → REFERENCE.md, never SKILL.md → A.md → B.md)
- Files over 100 lines should include a table of contents
- Use forward slashes for all paths

See [BEST-PRACTICES.md](BEST-PRACTICES.md) for detailed authoring guidelines.

### Step 3.4: Save Files

Use `Write` to save all files to `.claude/skills/{skill-name}/`

---

## Phase 4: Validation

After generating the skill, validate it works correctly.

See [VALIDATION.md](VALIDATION.md) for the complete testing procedure.

### Quick Validation Checklist

1. [ ] Test a prompt that SHOULD trigger the skill
2. [ ] Test a prompt that should NOT trigger the skill
3. [ ] Verify file structure is correct
4. [ ] Confirm YAML frontmatter passes validation

Report validation results to the user before considering the skill complete.

---

## Out of Scope

This skill does NOT handle:
- Uploading skills to Claude API (different process)
- Managing skills in claude.ai settings
- Creating skills that require Bash execution (consider if `allowed-tools` restriction is appropriate)
