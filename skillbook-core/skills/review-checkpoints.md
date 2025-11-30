---
name: review-checkpoints
description: Use to understand when and how to get feedback during the skillbook workflow. Defines phase-dependent review methods.
requires:
  tools: []
  skills: []
---

# Review Checkpoints

Different phases need different feedback methods. This skill defines when to use each.

## Checkpoint Table

| Phase | Method | How |
|-------|--------|-----|
| Brainstorm | Ask user | Use AskUserQuestion tool |
| Document (ADRs) | Ask user | Use AskUserQuestion tool |
| Model (architecture) | Zen MCP or ask user | Try mcp__zen__chat first, fall back to asking |
| Model (threats) | Ask user | Security decisions need human sign-off |
| UI mockup | Ask user | "Please review mockups/<name>.html" |
| Write test | Code review | superpowers:requesting-code-review |
| Component in library | Playwright | Automated, run tests |
| Develop (integration) | Code review | superpowers:requesting-code-review |
| Feature complete | Playwright + ask user | Tests then human review |
| Before merge | Code review | superpowers:requesting-code-review |

## Methods

### Ask User

Use the AskUserQuestion tool for decisions that need human judgment:
- Direction and scope
- Architectural decisions
- Security decisions
- Visual design approval
- Feature completion sign-off

### Zen MCP

Use `mcp__zen__chat` for quick external validation:
- Architecture review
- Code pattern review
- Best practice checks

Falls back to asking user if Zen MCP unavailable.

### superpowers:requesting-code-review

Use for technical reviews:
- Test design
- Implementation quality
- Pre-merge checks

Dispatches `superpowers:code-reviewer` subagent.

### Playwright

Use for automated UI verification:
- Component works in isolation
- Feature works end-to-end
- No human needed for this checkpoint

## When to Use Each

### Human judgment required
- "Is this the right direction?"
- "Should we use approach A or B?"
- "Is this security model acceptable?"
- "Does this UI look right?"

→ Ask user

### Technical validation
- "Is this test well-designed?"
- "Is this code clean?"
- "Ready to merge?"

→ superpowers:requesting-code-review

### Automated checks
- "Does this component render?"
- "Does this feature work?"

→ Playwright

### Quick second opinion
- "Is this architecture reasonable?"
- "Am I missing something?"

→ Zen MCP (or ask user)

## Success Criteria

- [ ] Correct method used for each phase
- [ ] Human reviews completed where required
- [ ] Automated checks passing
- [ ] No phases skipped
