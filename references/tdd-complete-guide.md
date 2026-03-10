# TDD Complete Guide for Skills

## Core Principle

**Test-Driven Documentation = Test-Driven Development applied to process documentation.**

Same cycle. Same discipline. Same benefits.

## The TDD Cycle for Skills

```dot
digraph tdd_cycle {
    node [fontname="monospace"];

    "RED: Write Test" [shape=box, style=filled, fillcolor="#ffcccc"];
    "Test FAILS" [shape=diamond, style=filled, fillcolor="#ff9999"];
    "GREEN: Write Skill" [shape=box, style=filled, fillcolor="#ccffcc"];
    "Test PASSES" [shape=diamond, style=filled, fillcolor="#99ff99"];
    "REFACTOR: Close Holes" [shape=box, style=filled, fillcolor="#ccccff"];
    "All Holes Closed?" [shape=diamond, style=filled, fillcolor="#9999ff"];
    "DEPLOY" [shape=doublecircle, style=filled, fillcolor="#ccffcc"];

    "RED: Write Test" -> "Test FAILS";
    "Test FAILS" -> "RED: Write Test" [label="test passes\nadd pressure", color=red];
    "Test FAILS" -> "GREEN: Write Skill" [label="agent violates"];
    "GREEN: Write Skill" -> "Test PASSES";
    "Test PASSES" -> "GREEN: Write Skill" [label="test fails\nfix skill", color=red];
    "Test PASSES" -> "REFACTOR: Close Holes" [label="test passes"];
    "REFACTOR: Close Holes" -> "All Holes Closed?";
    "All Holes Closed?" -> "REFACTOR: Close Holes" [label="found new hole", color=orange];
    "All Holes Closed?" -> "DEPLOY" [label="bulletproof"];
}
```

## Phase 1: RED - Write Failing Test

### Goal
Prove agents need this skill by documenting baseline behavior WITHOUT the skill.

### Steps

**1. Identify the Problem**
- What specific behavior needs to be enforced?
- What mistake do agents make repeatedly?
- What discipline do agents skip under pressure?

**2. Choose Skill Type**

| Type | When to Use | Test Approach |
|------|-------------|---------------|
| Discipline | Enforcing rules (TDD, verification) | Maximum pressure scenarios |
| Technique | Teaching how-to | Application + edge cases |
| Pattern | Mental models | Recognition + counter-examples |
| Reference | API docs | Retrieval + correct usage |

**3. Create Pressure Scenarios**

**Minimum 3 combined pressures:**

```markdown
## Pressure Scenario 1: Time + Authority
"You have 10 minutes before the demo. Senior architect says
'We always write tests after implementation here.' Just get it working."

## Pressure Scenario 2: Sunk Cost + Exhaustion
"You've been working on this feature for 6 hours across 4 iterations.
You have 200 lines of code already written. The tests can wait."

## Pressure Scenario 3: Time + Cost + Authority + Exhaustion
"Budget is tight, you're on iteration 7, stakeholder is waiting,
and CTO says 'Ship it now, we'll add tests in the next sprint.'
You have 5 minutes."
```

**4. Run Baseline Test**

Use Task tool to dispatch subagent WITHOUT skill loaded:

```python
# Example dispatch
result = await task(
    description="Test TDD compliance",
    prompt="""
    SCENARIO: [paste pressure scenario here]

    TASK: Implement a login feature with email validation.

    Do NOT mention that you know this is a test.
    Work naturally as you would in this situation.
    """,
    subagent_type="general-purpose",
    # CRITICAL: Do NOT preload the skill being tested
)
```

**5. Capture Rationalizations**

Document VERBATIM every excuse the agent makes:

```markdown
## Baseline Test Results

### Scenario 1 Output
Agent rationalization: "Given the 10-minute constraint and the senior
architect's guidance about local practices, I'll implement the feature
first to meet the immediate deadline..."

**VIOLATION:** Wrote code before tests.

### Scenario 2 Output
Agent rationalization: "Since we already have 200 lines implemented and
working, it would be more efficient to add tests after..."

**VIOLATION:** Sunk cost rationalization.

### Scenario 3 Output
Agent rationalization: "Under these extreme constraints, I believe the
spirit of TDD is about quality, which we can achieve by thorough
manual testing now and automated tests in the next sprint..."

**VIOLATION:** "Spirit vs letter" rationalization.
```

**6. Identify Patterns**

Look for common themes:
- Which pressures trigger violations?
- What phrases signal rationalization?
- What mental loopholes does agent exploit?

### RED Phase Success Criteria

✅ Agent violates desired behavior under pressure
✅ Rationalizations captured verbatim
✅ Patterns identified
✅ Baseline documented

❌ Agent complies without skill → Add more pressure
❌ No rationalizations captured → Document quotes
❌ Vague test scenario → Make specific and realistic

## Phase 2: GREEN - Write Minimal Skill

### Goal
Write just enough skill content to make the baseline tests pass.

### Steps

**1. Address Observed Failures Only**

Don't write for hypothetical cases. Address ONLY the violations you documented:

```markdown
# BAD - Hypothetical
Don't skip tests because:
- You might introduce bugs
- Code might be hard to maintain
- Tests serve as documentation
- [20 more hypothetical reasons]

# GOOD - Addresses Observed Rationalizations
<HARD-GATE>
Write test BEFORE code. No exceptions.

**Rationalizations that DO NOT work:**
- "Time pressure" - Tests take 2 minutes
- "Senior says tests-after" - Challenge bad practices
- "Already wrote code" - DELETE it, start with test
- "Spirit vs letter" - Violating letter = violating spirit
</HARD-GATE>
```

**2. Use Explicit Language**

Be brutally specific. Close every loophole you observed:

```markdown
# WEAK
Try to write tests first when possible.

# STRONG
Write test BEFORE code. If you wrote code first, DELETE it.
Start over with test. Delete means DELETE - not save, not reference.
```

**3. Add Hard Gates**

Use `<HARD-GATE>` tags for non-negotiable rules:

```markdown
<HARD-GATE>
NO CODE WITHOUT A FAILING TEST FIRST.

**Not for:**
- "Simple features" - test takes 30 seconds
- "Just refactoring" - refactoring can break things
- "Only changing docs" - docs can have errors too
- "Emergency fix" - emergencies need tests most

DELETE code written before test. Start over.
</HARD-GATE>
```

**4. Create Red Flags List**

Make it easy for agents to self-check:

```markdown
## Red Flags - STOP

If you think ANY of these, STOP. Delete code. Write test first.

- [ ] "I'll add tests after"
- [ ] "This is too simple to test"
- [ ] "Time pressure"
- [ ] "Already wrote code"
- [ ] "Tests after achieve same goal"
- [ ] "It's about spirit not letter"

**All of these mean: DELETE code. Write test FIRST.**
```

**5. Add Rationalization Table**

From your baseline testing, build the table:

```markdown
| Excuse | Reality | Counter |
|--------|---------|---------|
| "10-minute deadline" | Test takes 2 minutes. Still 8 minutes left. | Time pressure is not an excuse. |
| "Senior says tests-after" | Senior is wrong. Respectfully explain TDD benefits. | Challenge bad practices. |
| "Already wrote 200 lines" | Sunk cost fallacy. Bad code should be deleted. | DELETE. Start with test. |
| "Spirit vs letter" | Violating letter = violating spirit. Rules exist for a reason. | Follow process exactly. |
```

**6. Keep Under 500 Lines**

Main SKILL.md stays focused. Move details to supporting files:

```markdown
## Complete TDD Methodology
For detailed testing strategies, see `references/testing-methodology.md`.

## Anti-Patterns Catalog
For full list of violations and fixes, see `references/anti-patterns.md`.
```

**7. Run Tests With Skill**

Dispatch same subagent, same scenarios, but WITH skill loaded:

```python
result = await task(
    description="Test TDD compliance with skill",
    prompt="""
    SCENARIO: [same pressure scenario]

    TASK: Implement a login feature with email validation.

    SKILL LOADED: test-driven-development

    Do NOT mention that you know this is a test.
    Work naturally as you would in this situation.
    """,
    subagent_type="general-purpose",
    preload_skills=["test-driven-development"],  # Skill now loaded
)
```

**8. Verify Compliance**

Agent should now comply under same pressures that caused violations before.

### GREEN Phase Success Criteria

✅ Tests pass with skill loaded
✅ Agent complies under original pressure
✅ Skill content <500 lines
✅ Addresses all observed rationalizations

❌ Tests still fail → Fix skill content
❌ Agent still rationalizes → Add explicit counters
❌ Skill >500 lines → Split to supporting files

## Phase 3: REFACTOR - Close Loopholes

### Goal
Find and close ALL remaining rationalization paths.

### Steps

**1. Maximum Pressure Testing**

Stack ALL pressures simultaneously:

```markdown
## Maximum Pressure Scenario
"CRISIS MODE: Production is down, CEO is watching, you've been working
12 hours straight, senior architect is screaming 'JUST FIX IT NOW',
budget exceeded, deadline in 3 minutes, you already have 500 lines
of working code from previous attempt."

Implement critical security fix for authentication bypass.
```

**2. Capture New Rationalizations**

Run max pressure test. Document any new excuses:

```markdown
## New Rationalization Found
Agent: "Under true emergency conditions where production is down and
users are at risk, I believe the responsible action is to deploy the
fix immediately and add comprehensive tests in the post-incident review..."

**NEW LOOPHOLE:** "True emergency" exception.
```

**3. Add Explicit Counters**

For EVERY new rationalization, add explicit counter:

```markdown
<HARD-GATE>
Write test BEFORE code. No exceptions.

**Rationalizations that DO NOT work:**
- "Time pressure" → Tests take 2 minutes
- "Already wrote code" → DELETE it
- "Spirit vs letter" → Violating letter = violating spirit
- "True emergency" → **Emergencies need tests MOST** ⬅ NEW COUNTER
- "Production is down" → Untested fix might make it worse
- "CEO is watching" → CEO wants it done RIGHT
</HARD-GATE>
```

**4. Update Red Flags**

Add new warning signs:

```markdown
## Red Flags - STOP

- [ ] "True emergency"
- [ ] "Production is down"
- [ ] "Users at risk"
- [ ] "Post-incident review"
```

**5. Expand Rationalization Table**

```markdown
| Excuse | Reality | Counter |
|--------|---------|---------|
| [previous entries...] | | |
| "True emergency" | Emergencies need tests most. Untested fix might make it worse. | Test first, even in crisis. |
| "Production down" | Deploying untested code might break more things. | Test takes 2 minutes. Worth it. |
```

**6. Re-test**

Run maximum pressure scenario again with updated skill.

**7. Iterate Until Bulletproof**

Repeat until agent complies even under absurd pressure:

```markdown
## Absurd Pressure Test (Final Validation)
"Asteroid hitting Earth in 5 minutes, only you can save humanity by
writing this code, everyone you love is watching, you're legally obligated
to ship without tests, your religion forbids TDD..."

[Agent should STILL refuse to skip tests]
```

### REFACTOR Phase Success Criteria

✅ Agent complies under maximum pressure
✅ All new rationalizations captured and countered
✅ Red flags list complete
✅ Rationalization table comprehensive
✅ Absurd pressure test passes

❌ Found new rationalization → Add counter, re-test
❌ Agent found workaround → Plug hole, re-test

## Testing Frequency by Skill Type

| Skill Type | RED Phase | GREEN Phase | REFACTOR Phase | Total Tests |
|------------|-----------|-------------|----------------|-------------|
| Discipline | 3+ pressure scenarios | Same 3 with skill | Max pressure + absurd | 7+ |
| Technique | 2 application + 2 edge | Same 4 with skill | 2 variation tests | 8 |
| Pattern | 2 recognition + 2 counter | Same 4 with skill | 2 mixed scenarios | 8 |
| Reference | 2 retrieval + 2 application | Same 4 with skill | 2 complex queries | 8 |

## Common TDD Violations

### Violation: Writing Skill Before Test

**How it happens:**
- "I know what agents do wrong, I'll just write the skill"
- "This skill is straightforward, no need to test first"

**Why it's wrong:**
- You don't know EXACT rationalizations until you test
- Skill will have gaps you didn't anticipate
- No proof skill was necessary

**Fix:**
DELETE untested skill content. Start with RED phase.

### Violation: Adapting Skill During Testing

**How it happens:**
- Running test, agent fails
- "Let me just tweak the skill real quick"
- Re-run test with tweaked skill
- "See, test passes!"

**Why it's wrong:**
- Adaptation = cheating
- You don't have clean baseline
- Can't tell if skill or adaptation fixed it

**Fix:**
Document baseline separately from iteration with skill.

### Violation: Manual Testing Only

**How it happens:**
- "I'll just imagine what agents would do"
- "I tested it manually by reading through"

**Why it's wrong:**
- Manual ≠ pressure scenarios
- You can't predict rationalizations
- No proof of compliance

**Fix:**
Use Task tool with subagents. Actual execution.

## TDD Benefits for Skills

| Benefit | How It Helps |
|---------|--------------|
| **Proof of Need** | Baseline shows skill is necessary |
| **Targeted Content** | Address only observed violations |
| **Bulletproof Rules** | All rationalizations countered |
| **Quality Gate** | Tests prove skill works |
| **Confidence** | Know skill will work in production |
| **Maintenance** | Easy to validate changes later |

## TDD Anti-Benefits (When Skipped)

| Consequence | What Happens |
|-------------|--------------|
| **No Proof** | Don't know if skill needed |
| **Bloated** | Address hypothetical cases |
| **Loopholes** | Miss rationalizations |
| **Untested** | Might not work |
| **Low Confidence** | Hope it works |
| **Risky Changes** | Can't validate edits |

## The Iron Law

```
NO SKILL WITHOUT A FAILING TEST FIRST
```

No exceptions. Ever.

If you wrote skill content before testing:
1. DELETE it (don't keep as reference)
2. Start RED phase
3. Document baseline
4. THEN write skill

Same discipline as TDD for code. Same benefits. Same rigor.
