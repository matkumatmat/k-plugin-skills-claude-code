# CSO: Claude Search Optimization

## Critical Problem

Future Claude needs to FIND your skill before using it. If description doesn't match user's language or symptoms, skill never loads.

**CSO = SEO for Claude's skill discovery.**

## The Description Field

### Purpose

Description serves ONE purpose: **Help Claude decide "Should I load this skill right now?"**

### Critical Rule

**Description = WHEN to use, NOT WHAT the skill does.**

```yaml
# ❌ BAD - Describes workflow
description: >
  For TDD - write test first, watch it fail, write minimal code to pass,
  then refactor while keeping tests green.

# ✅ GOOD - Describes triggering conditions only
description: >
  Use when implementing any feature or bugfix, before writing implementation code.
  Prevents skipping tests under time pressure, sunk cost, or authority pressure.
```

### Why This Matters

Testing revealed: **When description summarizes workflow, Claude follows description instead of reading full skill.**

**Example:**
- Description: "code review between tasks"
- Skill body: [detailed flowchart showing TWO-stage review]
- Result: Claude did ONE review (followed description, skipped flowchart)

**Fix:**
- Description: "Use when executing implementation plans with independent tasks"
- Result: Claude read full skill, followed two-stage review process

**The trap:** Workflow summaries create shortcuts Claude takes. Skill body becomes unused documentation.

## Format Rules

### Start with "Use when"

This focuses description on triggering conditions:

```yaml
# ✅ GOOD examples
description: Use when implementing any feature or bugfix...
description: Use when tests have race conditions or pass/fail inconsistently...
description: Use when exploring unfamiliar codebase sections...
description: Use when creating skills, editing skills, or verifying they work...
```

### Third Person

Descriptions inject into system prompt. Write in third person:

```yaml
# ❌ BAD - First person
description: I can help you with async tests when they're flaky

# ✅ GOOD - Third person
description: Use when tests have timing dependencies or pass/fail inconsistently
```

### Technology-Agnostic (Unless Skill Is Specific)

Describe the PROBLEM, not the language-specific symptom:

```yaml
# ❌ BAD - Mentions tech but skill isn't tech-specific
description: Use when tests use setTimeout/sleep and are flaky

# ✅ GOOD - Describes problem
description: Use when tests have race conditions, timing dependencies,
  or pass/fail inconsistently

# ✅ ALSO GOOD - Tech-specific skill
description: Use when using React Router and handling authentication redirects
```

### Include Concrete Triggers

Use words users actually say:

```yaml
# ❌ BAD - Vague
description: For async testing

# ✅ GOOD - Concrete symptoms
description: Use when tests are flaky, hang indefinitely, have race conditions,
  or fail with "timed out" errors
```

## Keyword Optimization

### Error Messages

Include actual error text users might mention:

```
"Hook timed out"
"ENOTEMPTY"
"Cannot read property of undefined"
"Connection refused"
```

### Symptoms

Natural language users use:

```
"flaky tests"
"hanging process"
"zombie server"
"test pollution"
"race condition"
```

### Tools and Technologies

Actual command/library names:

```
"React Router"
"SQLAlchemy"
"FastAPI"
"pytest fixtures"
```

### Synonyms

Multiple ways to say same thing:

```
timeout / hang / freeze
cleanup / teardown / afterEach
setup / beforeEach / initialization
```

## Length Guidelines

### Target: <500 Characters

**Why:** Description loads into context for EVERY conversation. Every character counts.

**Technique - Be ruthlessly concise:**

```yaml
# ❌ VERBOSE (142 characters)
description: >
  Use this skill when you are working with tests that sometimes pass and
  sometimes fail, particularly when the failures seem related to timing

# ✅ CONCISE (78 characters)
description: >
  Use when tests have timing dependencies or pass/fail inconsistently
```

### Maximum: 1024 Characters

Hard limit from Claude Code. Includes all frontmatter fields combined.

### Check Length

```bash
# Count description characters
cat SKILL.md | grep -A 10 'description:' | wc -c

# Or check whole frontmatter
cat SKILL.md | sed -n '/^---$/,/^---$/p' | wc -c
```

## Description Templates by Skill Type

### Discipline Skills (Enforcement)

```yaml
description: >
  Use when [action that requires discipline], before [mistake point].
  Prevents [specific violations] under [specific pressures].
```

**Examples:**

```yaml
description: >
  Use when implementing features or bugfixes, before writing implementation code.
  Prevents skipping tests under time pressure or authority pressure.

description: >
  Use when completing tasks or claiming work is done, before committing or creating PRs.
  Prevents claiming success without running verification commands.
```

### Technique Skills (How-To)

```yaml
description: >
  Use when [problem symptom] or [error condition].
  Applies [technique name] to [fix what].
```

**Examples:**

```yaml
description: >
  Use when tests have race conditions, timing dependencies, or fail inconsistently.
  Applies condition-based waiting patterns.

description: >
  Use when debugging complex failures or unclear error messages.
  Applies root-cause tracing methodology.
```

### Pattern Skills (Mental Models)

```yaml
description: >
  Use when [design decision point] or [complexity symptom].
  Applies [pattern name] thinking model.
```

**Examples:**

```yaml
description: >
  Use when facing feature creep or unclear requirements.
  Applies YAGNI principle to scope decisions.

description: >
  Use when codebase feels tangled or hard to understand.
  Applies information hiding and loose coupling patterns.
```

### Reference Skills (Documentation)

```yaml
description: >
  API reference for [library/tool name].
  Use when working with [specific features].
```

**Examples:**

```yaml
description: >
  API reference for PptxGenJS PowerPoint generation.
  Use when creating, formatting, or exporting presentations.

description: >
  SQLAlchemy ORM patterns and query reference.
  Use when building database models or complex queries.
```

## Multi-Line Descriptions

Use `>` for natural line breaks that fold into one paragraph:

```yaml
description: >
  Use when implementing features or bugfixes, before writing implementation code.
  Prevents skipping tests under time pressure, sunk cost, or authority pressure.
  Enforces RED-GREEN-REFACTOR cycle strictly.
```

**Renders as:** "Use when implementing features or bugfixes, before writing implementation code. Prevents skipping tests under time pressure, sunk cost, or authority pressure. Enforces RED-GREEN-REFACTOR cycle strictly."

## What NOT to Include

### ❌ Workflow Steps

```yaml
# BAD - Describes HOW skill works
description: >
  Write test first, watch it fail, write code to pass, refactor
```

### ❌ Implementation Details

```yaml
# BAD - Too much detail
description: >
  Uses subagents to dispatch parallel tasks with code review between each,
  then aggregates results and merges branches
```

### ❌ Historical Context

```yaml
# BAD - Origin story
description: >
  Created after discovering agents skip tests under pressure in Session 2024-10-15
```

### ❌ Philosophical Reasoning

```yaml
# BAD - Why it matters
description: >
  TDD improves code quality, reduces bugs, serves as documentation,
  and makes refactoring safer
```

### ❌ Comparisons

```yaml
# BAD - Alternatives
description: >
  Better than manual testing or tests-after approach
```

## Testing Your Description

### Discovery Test

Ask question that SHOULD trigger your skill:

```bash
User: "My tests are flaky and sometimes fail with timeout errors"

Expected: Claude loads your async-testing skill
```

If skill doesn't load:
1. Check if description includes "flaky"
2. Check if description includes "timeout"
3. Add missing keywords

### Irrelevance Test

Ask question that should NOT trigger skill:

```bash
User: "How do I center a div in CSS?"

Expected: Claude does NOT load your async-testing skill
```

If skill loads incorrectly:
1. Description too generic
2. Add more specific triggers
3. Make context more narrow

### Length Test

```bash
wc -c SKILL.md  # Should be < 1024 for entire frontmatter
```

## Optimization Checklist

Before deploying skill, verify description:

- [ ] Starts with "Use when"
- [ ] Describes WHEN, not WHAT
- [ ] Third person voice
- [ ] Technology-agnostic (unless skill is tech-specific)
- [ ] Includes concrete symptoms/triggers
- [ ] Includes error messages if applicable
- [ ] Includes tool/library names if applicable
- [ ] Under 500 characters (ideally)
- [ ] Under 1024 characters (hard limit)
- [ ] No workflow summary
- [ ] No implementation details
- [ ] No philosophical reasoning
- [ ] Tested with discovery test
- [ ] Tested with irrelevance test

## Real-World Examples

### Example 1: TDD Skill

```yaml
# ❌ BEFORE (workflow summary)
description: >
  For test-driven development - write test first, watch it fail, write minimal
  code to pass, then refactor while keeping tests green. Helps maintain quality.

# ✅ AFTER (triggering conditions only)
description: >
  Use when implementing any feature or bugfix, before writing implementation code.
  Prevents skipping tests under time pressure, sunk cost, or authority pressure.
```

**Impact:** 50% description size. Claude now reads full skill instead of following summary.

### Example 2: Async Testing Skill

```yaml
# ❌ BEFORE (tech-specific symptoms)
description: >
  Use when tests use setTimeout, sleep, or async/await and have timing issues

# ✅ AFTER (problem-focused)
description: >
  Use when tests have race conditions, timing dependencies, or pass/fail inconsistently.
  Symptoms: timeouts, flakiness, "Cannot read property" errors after async operations.
```

**Impact:** Added symptom keywords. Discovery rate improved 3x.

### Example 3: Debugging Skill

```yaml
# ❌ BEFORE (vague)
description: For debugging complex issues

# ✅ AFTER (concrete)
description: >
  Use when error messages are unclear, root cause is non-obvious, or
  multiple potential causes exist. Applies systematic root-cause tracing.
```

**Impact:** Claude now loads skill for debugging sessions, not just any bug.

## Advanced: Dynamic Context in Description

**WARNING:** Description is static. Do NOT use `!`command`` in description.

```yaml
# ❌ WRONG - Dynamic injection doesn't work in frontmatter
description: >
  Current branch: !`git branch --show-current`

# ✅ RIGHT - Generic description, use dynamic injection in BODY
description: >
  Use when analyzing git repository state or planning next steps

# Then in body:
## Current Context
- Branch: !`git branch --show-current`
```

## CSO Success Metrics

| Metric | How to Measure | Target |
|--------|----------------|--------|
| **Discovery Rate** | User asks relevant question → skill loads | >90% |
| **False Positives** | Skill loads for irrelevant questions | <5% |
| **Description Length** | Character count | <500 chars |
| **Keyword Coverage** | Common symptoms included | 100% |
| **Load Time** | Skill in context or not | <100ms |

Track these in testing logs to optimize iteratively.
