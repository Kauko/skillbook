---
name: gemini-feedback
description: Use after completing designs, before significant commits, or when making architectural decisions to get Gemini's feedback via Zen MCP. Facilitates iterative dialogue and organizes findings into actionable items.
---

# Gemini Feedback

Get feedback from Gemini on designs, code, and decisions via Zen MCP.

## Prerequisites

- Zen MCP server installed and configured
- `GEMINI_API_KEY` set in environment

## When to Use

Suggest this skill to the user:
- After completing a design document
- Before committing significant code changes
- When making architectural decisions
- When they ask for a second opinion or feedback

## Workflow

### Phase 1: Gather Context

1. **Identify the subject** - What needs feedback?
   - Design document or architectural plan
   - Code changes (diff or files)
   - A decision between options

2. **Select the Zen tool** based on context:

| Context | Tool | Invocation |
|---------|------|------------|
| Design/architecture | `thinkdeep` | "Use zen thinkdeep to analyze..." |
| Code changes | `codereview` | "Use zen codereview to review..." |
| Quick opinion | `chat` | "Use zen chat with gemini to..." |
| High-stakes decision | `consensus` | "Use zen consensus with gemini and [other model] to..." |

3. **Craft the initial prompt** - Include:
   - Full context (the design, code, or decision)
   - Specific aspects to focus on
   - Any constraints or requirements

### Phase 2: Iterative Dialogue

Conduct 1-3 rounds of feedback:

**Round 1: Initial feedback**
```
Use zen [tool] to review this [design/code/decision]:

[Full context here]

Focus on:
- Potential issues or blind spots
- Missing considerations
- Alternative approaches
```

**Rounds 2-3: Follow-up**

After each response:
1. Identify points needing clarification
2. Challenge weak arguments or ask for specifics
3. Present counterarguments if you disagree

Example follow-ups:
- "You mentioned [concern]. Can you elaborate on the specific failure mode?"
- "How would you handle [edge case] with your suggested approach?"
- "What are the tradeoffs of [alternative] vs the current approach?"

**After 1-3 rounds, ask the user:**
> "We've had [N] rounds of feedback. Continue the dialogue or synthesize findings?"

### Phase 3: Organize & Act

Parse all feedback into these categories:

**Critical Concerns**
Issues that should be addressed before proceeding.

**Suggestions**
Improvements worth considering but not blocking.

**Agreements**
Validation of good decisions - what Gemini thinks is solid.

**Open Questions**
Things Gemini wasn't sure about or needs more context for.

Present the organized summary, then offer:
- "Want me to create todos for the critical concerns?"
- "Should I update the design to address [specific concern]?"
- "Any points you want to discuss further?"

## Example Sessions

### Design Review

**You:** "I've finished the authentication design. Get Gemini's feedback."

**Claude:** "I'll use zen thinkdeep for this architectural review."

```
Use zen thinkdeep to analyze this authentication system design:

[Design content]

Focus on:
- Security vulnerabilities
- Scalability concerns
- Missing edge cases
```

*[Gemini responds with initial feedback]*

**Claude:** "Gemini raised concerns about token refresh and rate limiting. Let me dig deeper."

```
Continue with zen thinkdeep:

You mentioned the token refresh strategy is unclear. Specifically:
1. What attack vectors does this open?
2. What's the recommended refresh window?
```

*[After 2-3 rounds]*

**Claude:** "Here's the organized feedback:

**Critical Concerns:**
- No token refresh mechanism - enables indefinite session hijacking
- Missing rate limiting on /auth endpoints

**Suggestions:**
- Consider refresh token rotation
- Add device fingerprinting for suspicious login detection

**Agreements:**
- JWT structure is solid
- Role-based access control approach is appropriate

Want me to create todos for the critical concerns?"

### Code Review Before Commit

**You:** "Review these auth changes with Gemini before I commit."

**Claude:** "I'll use zen codereview for the code changes."

```
Use zen codereview to review these changes:

[git diff output]

Focus on:
- Security issues
- Error handling
- Edge cases
```

### Quick Decision

**You:** "Should we use Redis or PostgreSQL for sessions?"

**Claude:** "This is a significant architectural decision. I'll use zen consensus to get multiple perspectives."

```
Use zen consensus with gemini-pro and claude to evaluate:

Session storage choice: Redis vs PostgreSQL

Context:
- Expected 10k concurrent users
- Need session data for analytics
- Team has PostgreSQL expertise

Evaluate: performance, operational complexity, data durability
```

## Tool Selection Guide

**Use `thinkdeep` when:**
- Reviewing designs or architecture
- Complex problems needing extended reasoning
- You want thorough analysis of tradeoffs

**Use `codereview` when:**
- Reviewing code changes
- Pre-commit validation
- Looking for bugs or security issues

**Use `chat` when:**
- Quick questions
- Simple validation
- Informal second opinion

**Use `consensus` when:**
- High-stakes decisions
- You want multiple perspectives
- Choosing between significant alternatives
