# Dynamic Context Injection

## Syntax: !`command`

Execute shell commands BEFORE Claude sees skill content.

## How It Works

```yaml
---
name: git-analyzer
---

Current branch: !`git branch --show-current`
Status: !`git status --short`
```

**Execution order:**
1. Skill invoked
2. Commands execute (`git branch --show-current`, `git status --short`)
3. Output replaces `!`command`` placeholders
4. Claude receives fully-rendered content

## Use Cases

### Real-Time Repository State

```markdown
## Current Git State
- Branch: !`git branch --show-current`
- Uncommitted: !`git status --short | wc -l` files
- Last commit: !`git log -1 --format="%h - %s"`
- Ahead/behind: !`git status -sb | head -1`
```

### Environment Info

```markdown
## Runtime Context
- Timestamp: !`date +"%Y-%m-%d %H:%M:%S"`
- User: !`whoami`
- Directory: !`pwd`
- Node version: !`node --version 2>/dev/null || echo "Not installed"`
```

### File Stats

```markdown
## Codebase Stats
- Total skills: !`ls -1 ~/.claude/skills/ | wc -l`
- This skill size: !`wc -l ~/.claude/skills/my-skill/SKILL.md | awk '{print $1}'`
- Last modified: !`stat -f %Sm ~/.claude/skills/my-skill/SKILL.md`
```

### Dynamic Lists

```markdown
## Available Skills
!`ls -1 ~/.claude/skills/ | sed 's/^/- /'`

## Recent Commits
!`git log --oneline -5 | sed 's/^/- /'`
```

## Patterns

### Conditional Execution

```markdown
## Git Status (if available)
!`git status --short 2>/dev/null || echo "Not a git repository"`
```

### Format Output

```markdown
## Changed Files
!`git diff --name-only | head -10 | sed 's/^/- [ ] /'`

Renders as:
- [ ] src/api/routes.py
- [ ] tests/test_routes.py
```

### Combine Commands

```markdown
## Repository Summary
- Files changed: !`git diff --name-only | wc -l`
- Lines added: !`git diff --shortstat | grep -oE '[0-9]+ insertion' | awk '{print $1}'`
- Lines deleted: !`git diff --shortstat | grep -oE '[0-9]+ deletion' | awk '{print $1}'`
```

## Substitution Variables

Combine with session/argument substitution:

```markdown
## Session Context
- Session ID: ${CLAUDE_SESSION_ID}
- Invoked at: !`date +"%H:%M:%S"`
- Working on: $ARGUMENTS
- Branch: !`git branch --show-current`
```

## Security Considerations

### Safe Commands

✅ Read-only operations:
- `git status`, `git log`
- `ls`, `cat`, `wc`
- `date`, `whoami`

### Dangerous Commands

❌ Avoid destructive operations:
- `rm`, `mv` (modifies files)
- `git push`, `git reset --hard`
- Commands that change state

### Input Validation

If using `$ARGUMENTS` in commands:

```markdown
# ❌ UNSAFE - Command injection risk
!`cat $ARGUMENTS`

# ✅ SAFE - Validate first, or use fixed paths
!`ls -l ~/.claude/skills/ | grep "$ARGUMENTS" || echo "Not found"`
```

## Error Handling

### Fallback Values

```markdown
# Command fails → show fallback
!`git status --short 2>/dev/null || echo "No git repository"`
```

### Silent Errors

```markdown
# Suppress errors completely
!`command 2>/dev/null`
```

### Exit Code Check

```markdown
# Only show if successful
!`command && echo "Success" || echo "Failed"`
```

## Performance

### Fast Commands

Prefer commands that complete <100ms:
- `git status --short` ✅
- `ls -1` ✅
- `wc -l file` ✅

### Avoid Slow Commands

Commands >1s delay skill invocation:
- `find / -name` ❌ (slow)
- `git log --all --graph` ❌ (slow on large repos)
- External API calls ❌

### Cache When Possible

```bash
# Cache expensive operation
if [ ! -f /tmp/expensive-cache ]; then
    expensive-operation > /tmp/expensive-cache
fi
cat /tmp/expensive-cache
```

## Limitations

### Only in Skill Body

❌ **Doesn't work in frontmatter:**

```yaml
---
description: Current branch: !`git branch --show-current`
---
```

✅ **Use in body:**

```yaml
---
description: Analyzes current git repository state
---

Current branch: !`git branch --show-current`
```

### Executes Once

Commands run when skill loads, not continuously:

```markdown
Time: !`date`
# This timestamp won't update during skill execution
```

### Shell Context

Commands run in system shell, not Claude's context:
- No access to Claude variables (except `${CLAUDE_SESSION_ID}`)
- CWD = Claude Code working directory
- Environment = system environment

## Testing Injection

### Manual Test

```bash
# Preview what Claude will see
echo "Branch: $(git branch --show-current)"
echo "Status: $(git status --short)"
```

### Verify Substitution

```bash
# Extract and test command
grep '!\`' SKILL.md | sed 's/.*!\`\(.*\)`.*/\1/' | while read cmd; do
    echo "Testing: $cmd"
    eval "$cmd"
done
```

## Examples

### PR Summary Skill

```yaml
---
name: pr-summary
description: Summarize GitHub PR changes
---

## PR Context
- PR Number: !`gh pr view --json number -q .number`
- Title: !`gh pr view --json title -q .title`
- Author: !`gh pr view --json author -q .author.login`
- Status: !`gh pr view --json state -q .state`

## Changes
!`gh pr diff --name-only | sed 's/^/- /'`

## Comments
!`gh pr view --comments | head -20`

Summarize this PR focusing on intent and impact.
```

### Skill Status Checker

```yaml
---
name: skill-health
description: Check skill deployment status
---

## Installed Skills
Total: !`ls -1 ~/.claude/skills/ 2>/dev/null | wc -l`

## Skills by Size
!`find ~/.claude/skills -name SKILL.md -exec wc -l {} + | sort -rn | head -5 | awk '{print "- " $2 ": " $1 " lines"}'`

## Git Status
!`cd ~/.claude/skills && git status --short 2>/dev/null || echo "Not tracked"`

## Last Modified
!`find ~/.claude/skills -name SKILL.md -type f -exec stat -f "%Sm %N" -t "%Y-%m-%d %H:%M" {} + | sort -r | head -5 | sed 's/^/- /'`
```

## Best Practices

1. **Keep commands simple** - Complex logic in scripts, not injection
2. **Handle errors** - Use `|| echo "fallback"`
3. **Format output** - Use `sed`/`awk` for clean rendering
4. **Test independently** - Run commands manually first
5. **Document purpose** - Comment why data is injected
6. **Optimize speed** - Avoid slow operations
7. **Validate input** - If using `$ARGUMENTS`
8. **Provide fallbacks** - Handle missing tools/repos

## Troubleshooting

| Problem | Cause | Solution |
|---------|-------|----------|
| Command not found | Tool not installed | Add fallback: `cmd 2>/dev/null \|\| echo "N/A"` |
| Empty output | Wrong directory | Use absolute paths or `cd` first |
| Slow loading | Expensive command | Optimize or cache |
| Injection visible | Wrong syntax | Use !\`cmd\` not !{cmd} or $(cmd) |
| Security warning | Dangerous command | Use read-only operations |

