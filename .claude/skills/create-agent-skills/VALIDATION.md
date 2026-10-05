# Skill Validation Procedure

After generating a skill, run these tests before considering it complete.

---

## Step 1: Structural Validation

### 1.1 Verify File Structure

Use `Glob` to confirm all expected files exist:
```
.claude/skills/{skill-name}/
├── SKILL.md           # Required
├── *.md               # Any supporting files
└── scripts/           # Optional
```

### 1.2 Validate YAML Frontmatter

Check SKILL.md frontmatter against requirements:

| Field | Requirement | Check |
|-------|-------------|-------|
| `name` | lowercase, hyphens, numbers only | No uppercase, underscores, spaces |
| `name` | max 64 chars | Count characters |
| `name` | no reserved words | No "anthropic" or "claude" |
| `description` | non-empty | Has content |
| `description` | max 1024 chars | Count characters |
| `description` | no XML tags | No `<` or `>` |
| `description` | includes WHAT and WHEN | Both purposes present |

---

## Step 2: Trigger Testing

### 2.1 Positive Trigger Test

Craft a prompt that SHOULD trigger the skill. The prompt should:
- Use keywords from the description
- Match a documented use case
- Be realistic (something a user would actually ask)

**Example for a git skill:**
> "Create a new feature branch for issue #1234"

Ask the user: "I'll test if this prompt triggers the skill correctly: '{prompt}'. Should I proceed?"

### 2.2 Negative Trigger Test

Craft a prompt that should NOT trigger the skill. The prompt should:
- Be in a related domain but outside scope
- Use similar but distinct keywords

**Example for a git skill:**
> "Explain how git rebase works" (knowledge question, not workflow)

### 2.3 Edge Case Test

If the skill has conditional workflows, test the decision points:
- Test each branch of conditional logic
- Test boundary conditions

---

## Step 3: Content Validation

### 3.1 Link Check

For each file reference in SKILL.md:
1. Verify the referenced file exists
2. Verify the file contains relevant content
3. Verify no chained references (A → B → C)

### 3.2 Example Validation

For each example in the skill:
1. Verify it's complete (not truncated)
2. Verify it's syntactically correct
3. Verify it matches current conventions

---

## Step 4: Report Results

Present validation results to the user:
```markdown
## Skill Validation Results: {skill-name}

### Structure
- [x] All files created
- [x] YAML frontmatter valid

### Trigger Tests
- [x] Positive test: "{prompt}" - Triggered correctly
- [x] Negative test: "{prompt}" - Did not trigger (correct)

### Content
- [x] All file references valid
- [x] Examples verified

**Status: PASSED** ✓
```

If any test fails, report the specific failure and offer to fix it.

---

## Quick Validation Checklist

Copy this checklist for each skill validation:
```markdown
- [ ] Files exist in `.claude/skills/{name}/`
- [ ] `name` is lowercase, hyphens, max 64 chars
- [ ] `description` is non-empty, max 1024 chars, has WHAT and WHEN
- [ ] Positive trigger test passed
- [ ] Negative trigger test passed
- [ ] All file references resolve
- [ ] Examples are complete and correct
```