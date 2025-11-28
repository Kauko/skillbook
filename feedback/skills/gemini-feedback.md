---
name: gemini-feedback
description: Use when user asks for a second opinion, wants AI feedback on designs, or needs architectural review from another model.
requires:
  tools: []
  skills: []
  mcp: [zen]
  env: [GEMINI_API_KEY]
---

# Gemini Feedback

Get feedback from Gemini on designs, code, and decisions via Zen MCP.

## Prerequisites

```bash
[ -n "$GEMINI_API_KEY" ] || { echo "Set GEMINI_API_KEY environment variable"; exit 1; }
# Zen MCP server must be configured in Claude settings
```

## When to Use

- After completing design documents
- Before significant commits
- When making architectural decisions
- When user asks for second opinion

## Tool Selection

| Context | Zen Tool |
|---------|----------|
| Design/architecture | `thinkdeep` |
| Code review | `codereview` |
| Quick opinion | `chat` |
| High-stakes decision | `consensus` (multi-model) |

## Workflow

### 1. Gather Context

```
Use zen [tool] to review:

[Full design/code/decision]

Focus on:
- Potential issues or blind spots
- Missing considerations
- Alternative approaches
```

### 2. Iterate (1-3 rounds)

After initial feedback:
```
Follow up on [specific point from feedback]:
- How would you address [concern]?
- Compare tradeoffs of [option A] vs [option B]
```

### 3. Synthesize

After dialogue, create actionable summary:

```markdown
## Feedback Summary

### Confirmed strengths
- [Points validated by feedback]

### Issues to address
- [ ] [Specific issue with recommended fix]

### Parking lot (future consideration)
- [Deferred items]

### Decision rationale
[If decision was validated, note why]
```

## Example Prompts

**Design review:**
```
Use zen thinkdeep to analyze this API design:
[design content]
Focus on security, scalability, and developer experience.
```

**Code review:**
```
Use zen codereview to review these changes:
[git diff]
Check for edge cases and error handling.
```

**Architecture decision:**
```
Use zen consensus with gemini and claude to evaluate:
Option A: [description]
Option B: [description]
Constraints: [list]
```

## Output

After feedback session:
1. Update design/code based on findings
2. Add feedback summary to decision record
3. Note any deferred items for future work

## Success Criteria

- [ ] Zen MCP responds to prompts
- [ ] Feedback synthesized into actionable items
- [ ] Issues to address documented with recommended fixes
