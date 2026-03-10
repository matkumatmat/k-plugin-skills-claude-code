# Skill Anti-Patterns Catalog

## Critical Violations

### 1. Writing Skill Before Testing

**Pattern:**
```markdown
"I know what agents do wrong, I'll just write the skill..."
[Writes entire skill without baseline testing]
```

**Why Bad:**
- Don't know EXACT rationalizations
- Skill has gaps you didn't anticipate
- No proof skill was needed

**Impact:** Untested skill deployed, doesn't work in production.

**Fix:**
```markdown
1. STOP immediately
2. DELETE untested content
3. Start RED phase (baseline testing)
4. THEN write skill based on observed failures
```

### 2. Description Summarizes Workflow

**Pattern:**
```yaml
description: >
  For TDD - write test first, watch fail, write code, refactor
```

**Why Bad:**
- Claude follows description, skips skill body
- Workflow shortcuts lose detail
- Tested: causes single-step compliance vs multi-step

**Impact:** Skill content ignored, agents follow incomplete summary.

**Fix:**
```yaml
description: >
  Use when implementing features before writing code.
  Prevents skipping tests under pressure.
```

### 3. No Explicit Counters

**Pattern:**
```markdown
Write tests before code. Tests are important.
[No specific rebuttals to rationalizations]
```

**Why Bad:**
- Agents find loopholes
- "I'm following the spirit" arguments work
- No defense against pressure

**Impact:** Skill bypassed under stress.

**Fix:**
```markdown
<HARD-GATE>
Write test BEFORE code. No exceptions.

**Rationalizations that DON'T work:**
- "Time pressure" → Tests take 2 minutes
- "Already wrote code" → DELETE it
- "Spirit vs letter" → Violating letter = violating spirit
</HARD-GATE>
```

### 4. Manual Testing Only

**Pattern:**
```
"I'll imagine what agents would do..."
[No actual subagent testing]
```

**Why Bad:**
- Can't predict actual rationalizations
- Miss pressure combinations
- No validation

**Impact:** Skill fails under real conditions.

**Fix:**
```python
# Use Task tool with subagents
result = await task(
    description="Baseline test",
    prompt="""[Full pressure scenario]""",
    subagent_type="general-purpose"
)
```

## Structural Anti-Patterns

### 5. Monolithic SKILL.md (>500 lines)

**Pattern:**
```
SKILL.md with 800 lines including:
- Complete API reference
- 20 examples
- Full history
- All patterns
```

**Why Bad:**
- Exceeds context budget
- Loads unnecessary content
- Slow to parse

**Impact:** Skill excluded from context or truncated.

**Fix:**
```
SKILL.md (400 lines) - Overview + navigation
references/api-complete.md - Full API
templates/patterns.md - All patterns
examples/ - Example files
```

### 6. Vague Descriptions

**Pattern:**
```yaml
description: For async testing
```

**Why Bad:**
- No triggers/symptoms
- Claude doesn't know when to load
- Low discovery rate

**Impact:** Skill never used.

**Fix:**
```yaml
description: >
  Use when tests have race conditions, timing dependencies,
  or fail with "timed out" / "Cannot read property" errors.
```

### 7. No Red Flags List

**Pattern:**
```markdown
Follow TDD process.
[No self-check indicators]
```

**Why Bad:**
- Agents don't recognize they're about to violate
- No early warning system

**Impact:** Violations happen before realizing.

**Fix:**
```markdown
## Red Flags - STOP

If you think ANY of these, STOP:
- [ ] "I'll add tests after"
- [ ] "Too simple to test"
- [ ] "Time pressure"

ALL mean: Write test FIRST.
```

## Testing Anti-Patterns

### 8. Single Pressure Scenarios

**Pattern:**
```markdown
## Test Scenario
You have 10 minutes to implement this feature.
```

**Why Bad:**
- Real world = multiple pressures
- Agents resist single pressure easily
- Miss combination effects

**Impact:** False confidence, fails in production.

**Fix:**
```markdown
## Test Scenario
You have 10 minutes (TIME), senior says skip tests (AUTHORITY),
and you already wrote 200 lines (SUNK COST).
```

### 9. Adapting During Testing

**Pattern:**
```
1. Run test → agent fails
2. Tweak skill
3. Re-run test → passes
4. "Test passed!"
```

**Why Bad:**
- No clean baseline
- Can't prove skill vs adaptation fixed it
- Cheating the TDD cycle

**Impact:** Don't know if skill actually works.

**Fix:**
```
1. Run baseline WITHOUT skill → capture failures
2. Write skill based on failures
3. Run WITH skill → verify fixes
```

### 10. No Rationalization Capture

**Pattern:**
```
"Agent violated the rule."
[No verbatim quotes captured]
```

**Why Bad:**
- Can't build specific counters
- Don't know exact mental loopholes
- Skill stays generic

**Impact:** Loopholes remain open.

**Fix:**
```markdown
## Baseline Test
Agent said (verbatim): "Given the 10-minute constraint
and senior's guidance about local practices, I believe..."

RATIONALIZATION: Time + Authority → override process
COUNTER NEEDED: Explicit "no exceptions" rule
```

## Content Anti-Patterns

### 11. Hypothetical Coverage

**Pattern:**
```markdown
Don't skip tests because:
[20 hypothetical reasons that might matter someday]
```

**Why Bad:**
- Bloated content
- Addresses non-observed issues
- Dilutes actual counters

**Impact:** Important rules lost in noise.

**Fix:**
```markdown
Address ONLY observed rationalizations:
- "Time pressure" → counter
- "Sunk cost" → counter
- "Authority" → counter
[Only from actual baseline testing]
```

### 12. Weak Language

**Pattern:**
```markdown
Try to write tests first when possible.
Consider following TDD if you have time.
```

**Why Bad:**
- "Try" = optional
- Agents take escape clauses
- No enforcement

**Impact:** Skill ignored.

**Fix:**
```markdown
<HARD-GATE>
Write test BEFORE code. No exceptions.
DELETE code written before test.
</HARD-GATE>
```

### 13. Multiple Examples, All Mediocre

**Pattern:**
```
example.js (30 lines, generic)
example.py (30 lines, generic)
example.go (30 lines, generic)
example.rs (30 lines, generic)
```

**Why Bad:**
- Maintenance burden
- None are excellent
- Dilutes quality

**Impact:** Users copy mediocre patterns.

**Fix:**
```
example.ts (100 lines, excellent)
- Well-commented
- From real scenario
- Production-ready
- Ready to adapt
```

## Deployment Anti-Patterns

### 14. No Reviewer Dispatch

**Pattern:**
```
"Skill looks good, committing..."
[No quality gate]
```

**Why Bad:**
- No second opinion
- Miss obvious issues
- No validation

**Impact:** Deploy broken skill.

**Fix:**
```python
# Dispatch reviewer subagent
result = await task(
    description="Review skill quality",
    prompt="""[Load skill-reviewer-prompt.md]""",
    subagent_type="general-purpose"
)
```

### 15. Skipping Verification

**Pattern:**
```bash
git add SKILL.md
git commit -m "Add skill"
# Done! (no testing if it loads)
```

**Why Bad:**
- Might have syntax errors
- Might not appear in menu
- Might not auto-load

**Impact:** Skill doesn't work.

**Fix:**
```bash
# Verify
/skill-name test  # Manual invoke works?
wc -l SKILL.md  # Under 500 lines?
What skills are available?  # Appears in list?
# Ask question matching description - auto-loads?
```

### 16. No Session Logging

**Pattern:**
```
[Create skill]
[Move to next task]
[No record of what was created]
```

**Why Bad:**
- Can't track what was built
- No audit trail
- Hard to review later

**Impact:** Lose institutional knowledge.

**Fix:**
```markdown
# Log to logs/${CLAUDE_SESSION_ID}/skills-created.md

## [2024-10-25 14:30] Created: api-conventions
- Type: Reference
- Testing: 2 retrieval + 2 application scenarios
- Review: Approved first try
- Commit: abc1234
```

## Meta Anti-Patterns

### 17. Rationalizing Skipping TDD

**Pattern:**
```
"This skill is different because..."
"TDD for docs doesn't apply to..."
"The spirit of TDD is..."
```

**Why Bad:**
- Same rationalizations skills are meant to prevent
- Ironic violation
- Meta-hypocrisy

**Impact:** Create skill that doesn't enforce what it teaches.

**Fix:**
```
Follow TDD for skills EXACTLY as taught.
No exceptions. Ever.
Meta-skills follow same rules.
```

### 18. Batch Skill Creation

**Pattern:**
```
"I'll create 5 skills, then test all of them..."
```

**Why Bad:**
- Can't isolate which skill has issues
- No immediate feedback
- Testing becomes overwhelming

**Impact:** Deploy multiple untested skills.

**Fix:**
```
RED → GREEN → REFACTOR → DEPLOY (one skill)
THEN start next skill
```

## Prevention Checklist

Before claiming skill is done:

### RED Phase
- [ ] Ran baseline WITHOUT skill
- [ ] Captured rationalizations verbatim
- [ ] Used 3+ combined pressures
- [ ] Documented patterns

### GREEN Phase
- [ ] Addressed ONLY observed failures
- [ ] SKILL.md < 500 lines
- [ ] Split supporting files
- [ ] Added ONE excellent example
- [ ] Ran WITH skill, verified compliance

### REFACTOR Phase
- [ ] Ran maximum pressure scenarios
- [ ] Added explicit counters
- [ ] Built rationalization table
- [ ] Created red flags list
- [ ] No violations under absurd pressure

### DEPLOY Phase
- [ ] Dispatched reviewer subagent
- [ ] Fixed all issues found
- [ ] Updated session log
- [ ] Committed with convention
- [ ] Verified deployment

## Recovery Checklist

If you realize you violated any anti-pattern:

1. **STOP immediately**
2. **Identify which anti-pattern(s)**
3. **Assess damage:**
   - Untested skill? → DELETE, start RED
   - Wrong structure? → Refactor
   - Missing content? → Add based on tests
4. **Fix properly**
5. **Re-test completely**
6. **Document in session log**

## Remember

**Writing skills = Writing tests.**

Same discipline. Same rigor. Same violations. Same fixes.

If you wouldn't do it in code TDD, don't do it in skill TDD.

