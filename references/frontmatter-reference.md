# Frontmatter Reference

Complete guide to all YAML frontmatter fields for skill configuration.

## Format

```yaml
---
field1: value
field2: >
  Multi-line
  value
field3: [list, of, values]
---

# Skill content starts here
```

## Required Fields

### None

All fields are optional, but `description` is highly recommended.

## Core Fields

### `name`

**Purpose:** Skill identifier and slash command.

**Format:** Kebab-case, letters-numbers-hyphens only.

**Max length:** 64 characters.

**Default:** Directory name if omitted.

```yaml
---
name: my-skill-name
---
```

**Rules:**
- ✅ `api-conventions`
- ✅ `test-driven-development`
- ✅ `fastapi-ddd-backend`
- ❌ `api_conventions` (use hyphens not underscores)
- ❌ `API-Conventions` (lowercase only)
- ❌ `api conventions` (no spaces)
- ❌ `api(conventions)` (no special chars)

**Becomes:**
- Slash command: `/my-skill-name`
- Tool invocation: `Skill(my-skill-name)`
- File reference: `skills/my-skill-name/SKILL.md`

### `description`

**Purpose:** Help Claude decide when to load this skill.

**Format:** String or multi-line with `>`.

**Max length:** ~500 characters recommended, 1024 hard limit (entire frontmatter).

**Default:** First paragraph of markdown content.

```yaml
---
description: >
  Use when implementing features or bugfixes, before writing implementation code.
  Prevents skipping tests under time pressure or authority pressure.
---
```

**Critical rules:**
- Start with "Use when"
- Describe WHEN to use (NOT what it does)
- Include concrete symptoms/triggers
- NO workflow summary (see CSO guide)
- Third person voice

**See:** `references/cso-optimization.md` for complete optimization guide.

## Invocation Control

### `disable-model-invocation`

**Purpose:** Control whether Claude can auto-load this skill.

**Format:** Boolean (`true` or `false`).

**Default:** `false` (Claude CAN auto-load).

```yaml
---
disable-model-invocation: true
---
```

**When `true`:**
- Only user can invoke with `/skill-name`
- Description NOT in Claude's context
- Use for: workflows with side effects (deploy, commit, send-email)

**When `false` (default):**
- Both user AND Claude can invoke
- Description always in context
- Claude auto-loads when relevant
- Use for: reference material, guidelines, techniques

### `user-invocable`

**Purpose:** Control whether skill appears in user's `/` menu.

**Format:** Boolean (`true` or `false`).

**Default:** `true` (shows in menu).

```yaml
---
user-invocable: false
---
```

**When `false`:**
- Skill hidden from `/` menu
- Only Claude can invoke (auto-load)
- Use for: background knowledge, coding conventions

**When `true` (default):**
- Skill appears in `/` menu
- User can invoke manually
- Use for: actionable commands

## Invocation Matrix

| `disable-model-invocation` | `user-invocable` | User Invoke | Claude Invoke | Description in Context |
|----------------------------|------------------|-------------|---------------|------------------------|
| `false` (default) | `true` (default) | ✅ Yes | ✅ Yes | ✅ Yes |
| `true` | `true` (default) | ✅ Yes | ❌ No | ❌ No |
| `false` (default) | `false` | ❌ No | ✅ Yes | ✅ Yes |
| `true` | `false` | ❌ No | ❌ No | ❌ No (unusable) |

## Execution Context

### `context`

**Purpose:** Where/how to execute skill.

**Format:** String.

**Options:** `fork` or (empty).

**Default:** Empty (inline execution).

```yaml
---
context: fork
---
```

**When `fork`:**
- Runs in isolated subagent
- No access to conversation history
- Skill content = subagent's task prompt
- Use for: independent research, analysis, generation

**When empty (inline):**
- Runs in main conversation
- Has conversation history
- Skill content = additional instructions
- Use for: guidelines, references, assistive tasks

### `agent`

**Purpose:** Which subagent type to use when `context: fork`.

**Format:** String.

**Options:** `Explore`, `Plan`, `general-purpose`, or custom agent name.

**Default:** `general-purpose`.

**Ignored if:** `context` is not `fork`.

```yaml
---
context: fork
agent: Explore
---
```

**Built-in agents:**
- `Explore` - Read-only exploration, optimized for codebase discovery
- `Plan` - Planning and architecture, no execution
- `general-purpose` - Full tools, flexible

**Custom agents:** Reference agent defined in `.claude/agents/`.

## User Experience

### `argument-hint`

**Purpose:** Autocomplete hint showing expected arguments.

**Format:** String with placeholder notation.

**Default:** None.

```yaml
---
argument-hint: [issue-number] [format]
---
```

**Examples:**
- `[filename]`
- `[issue-number]`
- `[source] [target]`
- `[commit-message]`

**Shows in:** CLI autocomplete when typing `/skill-name`.

## Tool Control

### `allowed-tools`

**Purpose:** Tools Claude can use WITHOUT requesting permission.

**Format:** Comma-separated list or array.

**Default:** None (normal permission rules apply).

```yaml
---
allowed-tools: Read, Grep, Glob
---
```

**With patterns:**
```yaml
---
allowed-tools: Read, Write, Bash(git *), Bash(npm *)
---
```

**Rules:**
- Tool names case-sensitive
- Use exact tool names from Claude Code
- Patterns in parentheses: `Bash(git *)` allows all git commands
- Other tools still require permission per settings

**Common combinations:**

```yaml
# Read-only mode
allowed-tools: Read, Grep, Glob

# Code writing
allowed-tools: Read, Write, Edit

# Full development
allowed-tools: Read, Write, Edit, Bash(git *), Bash(npm *)

# Testing
allowed-tools: Read, Bash(pytest *), Bash(npm test)
```

## Model Override

### `model`

**Purpose:** Override default model for this skill.

**Format:** String.

**Options:** `opus`, `sonnet`, `haiku`.

**Default:** Inherits from session or parent.

```yaml
---
model: opus
---
```

**When to use:**
- `opus` - Complex reasoning, architecture, critical decisions
- `sonnet` - General development (default in most contexts)
- `haiku` - Simple, fast tasks (cost optimization)

**Cost implications:**
- Opus: Most expensive, most capable
- Sonnet: Balanced
- Haiku: Cheapest, fastest

## Lifecycle Hooks

### `hooks`

**Purpose:** Shell commands to run at specific lifecycle events.

**Format:** Nested YAML with event names and timing.

**Default:** None.

```yaml
---
hooks:
  user-prompt-submit:
    before: echo "Starting skill..."
    after: echo "Skill completed"
  tool-use:
    before: echo "About to use tool"
---
```

**Available events:**
- `user-prompt-submit` - User sends message
- `tool-use` - Before/after tool execution
- `skill-invoke` - This skill starts/ends

**Timing:**
- `before` - Runs before event
- `after` - Runs after event

**Common patterns:**

```yaml
# Logging
hooks:
  user-prompt-submit:
    before: echo "[$(date)] Skill invoked" >> logs/${CLAUDE_SESSION_ID}.log

# Validation
hooks:
  user-prompt-submit:
    before: ./scripts/validate-env.sh

# Cleanup
hooks:
  skill-invoke:
    after: rm -f /tmp/skill-cache-*
```

## Complete Example

```yaml
---
# Identity
name: comprehensive-skill-example
description: >
  Use when demonstrating all frontmatter fields together.
  Shows complete configuration with all options.

# Invocation
disable-model-invocation: false
user-invocable: true

# Execution
context: fork
agent: Explore

# UX
argument-hint: [file-path] [action]

# Tools
allowed-tools: Read, Grep, Glob, Bash(git *)

# Model
model: sonnet

# Lifecycle
hooks:
  user-prompt-submit:
    before: echo "Starting at $(date)"
    after: echo "Completed at $(date)"
---

# Skill Content

Your skill markdown starts here...
```

## Field Validation

### Length Limits

```bash
# Check total frontmatter size
cat SKILL.md | sed -n '/^---$/,/^---$/p' | wc -c
# Should be < 1024 characters

# Check description only
cat SKILL.md | grep -A 10 'description:' | wc -c
# Should be < 500 characters (recommended)
```

### Name Validation

```bash
# Valid skill name
echo "my-skill-123" | grep -E '^[a-z0-9-]+$'
# Should match

# Invalid (uppercase, spaces, special chars)
echo "My Skill (v2)" | grep -E '^[a-z0-9-]+$'
# Should NOT match
```

## Field Dependencies

### `agent` requires `context: fork`

```yaml
# ❌ WRONG - agent ignored
---
agent: Explore
---

# ✅ RIGHT - agent used
---
context: fork
agent: Explore
---
```

### Tool patterns require tool name

```yaml
# ❌ WRONG - Invalid syntax
---
allowed-tools: Bash(*)
---

# ✅ RIGHT - Specific patterns
---
allowed-tools: Bash(git *), Bash(npm *)
---
```

## Testing Frontmatter

### Verify Parsing

```bash
# Extract frontmatter
cat SKILL.md | sed -n '/^---$/,/^---$/p'

# Validate YAML syntax
cat SKILL.md | sed -n '/^---$/,/^---$/p' | yq '.'
```

### Test Invocation

```bash
# Test manual invoke
/skill-name arg1 arg2

# Test auto-invoke
# Ask question matching description
"How do I handle async tests?"
```

### Verify Context

```bash
# Check if description in context
/context
# Look for skill name

# Test with disable-model-invocation
# Claude should NOT auto-load
```

## Common Mistakes

### Mistake: Workflow in Description

```yaml
# ❌ WRONG
description: >
  Write test first, watch fail, write code, refactor

# ✅ RIGHT
description: >
  Use when implementing features, before writing code
```

### Mistake: Wrong Name Format

```yaml
# ❌ WRONG
name: My_Skill (v2)

# ✅ RIGHT
name: my-skill-v2
```

### Mistake: Agent Without Fork

```yaml
# ❌ WRONG - Agent ignored
---
agent: Explore
---

# ✅ RIGHT
---
context: fork
agent: Explore
---
```

### Mistake: Too Long

```yaml
# ❌ WRONG - Exceeds budget
description: >
  [300 word essay about what skill does, its history,
   philosophy, benefits, comparisons, examples...]

# ✅ RIGHT - Concise triggers
description: >
  Use when implementing features before writing code.
  Prevents skipping tests under pressure.
```

## Frontmatter Checklist

Before deploying skill, verify:

- [ ] `name` is kebab-case, <64 chars
- [ ] `description` starts with "Use when"
- [ ] `description` describes WHEN not WHAT
- [ ] `description` <500 chars
- [ ] Total frontmatter <1024 chars
- [ ] `disable-model-invocation` appropriate for use case
- [ ] `user-invocable` appropriate for use case
- [ ] `context: fork` only if skill needs isolation
- [ ] `agent` specified if using `context: fork`
- [ ] `allowed-tools` minimal set needed
- [ ] YAML syntax valid (no parse errors)
- [ ] Tested manual invocation works
- [ ] Tested auto-invocation (if applicable)

## Resources

- **CSO Optimization**: `references/cso-optimization.md`
- **Official Docs**: https://code.claude.com/docs/skills
- **YAML Validator**: `yq` command-line tool
- **Length Check**: `wc -c`
