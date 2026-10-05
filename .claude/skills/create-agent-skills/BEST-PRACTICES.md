# Skill Authoring Best Practices

Reference this file when generating skill content. These guidelines ensure skills are discoverable, maintainable, and effective.

---

## YAML Frontmatter Requirements

### name
- **Format:** lowercase letters, numbers, hyphens only
- **Length:** max 64 characters
- **Style:** gerund form preferred (`processing-data` not `process-data`)
- **Forbidden:** "anthropic", "claude", XML tags

### description
- **Length:** max 1024 characters, non-empty
- **Format:** third person, no XML tags
- **Content:** MUST include both WHAT it does AND WHEN to use it

**Good example:**
```yaml
description: Manages GitHub workflows including branch creation, commits, and pull requests. Use when working with git operations, creating branches, pushing changes, or managing PRs.
```

**Bad example:**
```yaml
description: Helps with GitHub stuff.
```

### allowed-tools (Optional)
Only include if the skill should restrict tool access:
```yaml
allowed-tools: [Read, Write, Glob, AskUserQuestion]
```

---

## Content Guidelines

### Be Concise
Claude is smart—only add context Claude doesn't already have. Avoid:
- Explaining what git commands do (Claude knows)
- Basic programming concepts
- Information easily inferred from context

### Use Concrete Examples
Replace abstract descriptions with copy-paste examples:

**Abstract (avoid):**
> Create appropriate branch names following conventions.

**Concrete (preferred):**
> Create branch: `feature/add-user-auth-1234` where `1234` is the issue number.

### Consistent Terminology
Pick terms and stick with them:
- If you say "skill" don't later say "capability" or "module"
- If you say "workflow" don't later say "process" or "procedure"

### No Time-Sensitive Information
Avoid:
- Version numbers that will become outdated
- "Current" or "latest" references
- Dates without context

If unavoidable, put in a clearly marked "Legacy Patterns" section.

---

## File Structure Guidelines

### SKILL.md Size
- Keep body under 500 lines
- If longer, split into supporting files with progressive disclosure

### Supporting Files
- Reference only one level deep (SKILL.md → REFERENCE.md)
- Never chain references (SKILL.md → A.md → B.md)
- Files over 100 lines need a table of contents

### Path Format
- Always use forward slashes: `scripts/validate.py`
- Never backslashes: `scripts\validate.py` ❌

---

## Workflow Skills

### Decision Points
Make conditional logic explicit:
```markdown
### Choose Your Path

**If creating a new feature:**
→ Go to [New Feature Workflow](#new-feature-workflow)

**If fixing a bug:**
→ Go to [Bug Fix Workflow](#bug-fix-workflow)
```

### Checklists
Provide copy-paste checklists for multi-step processes:
```markdown
## Pre-Flight Checklist

- [ ] Branch is up to date with main
- [ ] All tests pass locally
- [ ] No uncommitted changes
```

### Validation Steps
Insert validation between critical operations:
```markdown
### Step 3: Verify Before Proceeding

Confirm the following before continuing:
1. File exists at expected path
2. Content matches expected format
3. No error messages in output

If any check fails, stop and report to user.
```

---

## Skills with Scripts

### Error Handling
Scripts should handle errors explicitly:
```python
# Good: Explicit error handling
if not file_path.exists():
    print(f"ERROR: File not found: {file_path}")
    sys.exit(1)

# Bad: Let Claude figure it out
file_path.read_text()  # May throw unclear exception
```

### Document Magic Numbers
Explain any non-obvious values:
```python
MAX_RETRIES = 3  # Based on typical network timeout patterns
CHUNK_SIZE = 8192  # Optimal for most filesystems
```

### Execution Intent
Be explicit about whether to run or reference:
```markdown
**Run** `scripts/validate.py` to check the configuration.

**See** `scripts/algorithm.py` for the sorting implementation details.
```

---

## Common Mistakes to Avoid

| Mistake | Why It's Bad | Fix |
|---------|--------------|-----|
| Vague description | Skill won't trigger correctly | Include specific keywords and use cases |
| Too broad scope | Skill becomes unwieldy | Split into focused skills |
| Missing examples | Users don't know how to use it | Add concrete, copy-paste examples |
| Chained file refs | Progressive disclosure breaks | Keep refs one level deep |
| Backslash paths | Cross-platform issues | Always use forward slashes |