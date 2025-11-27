# Zen Feedback Plugin Design

## Overview

A Claude Code plugin that guides getting feedback from Gemini (via Zen MCP) on designs, code, and decisions. Selects the right Zen tool based on context, facilitates iterative dialogue, then organizes findings into actionable items.

## Plugin Structure

```
skillbook/
└── feedback/
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        └── gemini-feedback.md
```

## When to Trigger

The skill description prompts Claude to suggest it at key moments:

- After completing a design document
- Before committing significant code changes
- When making architectural decisions
- When explicitly asked for feedback/second opinion

Manual invocation only - no auto-run.

## Tool Selection

| Context | Zen Tool | When |
|---------|----------|------|
| Design/architecture review | `thinkdeep` | Complex reasoning needed |
| Code changes | `codereview` | Before commits, PR reviews |
| Quick opinion | `chat` | Simple questions, validation |
| High-stakes decision | `consensus` | Multiple model perspectives |

Skill assesses context and recommends tool; user can override.

## Workflow

### Phase 1: Gather Context

1. Identify what needs feedback (design doc, code, decision)
2. Select appropriate Zen tool based on context
3. Craft initial prompt with full context

### Phase 2: Iterative Dialogue (1-3 rounds)

1. Send to Gemini via Zen MCP
2. Review response, identify areas needing clarification
3. Ask follow-up questions or present counterarguments
4. After 1-3 rounds, ask: "Continue dialogue or synthesize?"

### Phase 3: Organize & Act

1. Parse all feedback into categories:
   - **Critical concerns** - Issues that should be addressed
   - **Suggestions** - Improvements worth considering
   - **Agreements** - Validation of good decisions
   - **Questions** - Things Gemini wasn't sure about
2. Present organized summary
3. Offer next steps: "Create todos for concerns?" / "Update the design?"

## Prerequisites

- Zen MCP server installed and configured
- `GEMINI_API_KEY` set in environment

## Example Invocations

**After completing a design:**
> "I've finished the auth system design. Get Gemini's feedback."

**Before a commit:**
> "Review these changes with Gemini before I commit."

**Architectural decision:**
> "Should we use Redis or PostgreSQL for session storage? Get consensus."
