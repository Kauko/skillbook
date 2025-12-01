---
name: using-skillbook
description: Use when starting any conversation in a skillbook project - establishes the plan → document → model → test → develop → verify workflow and lists all available skills
requires:
  tools: []
  skills: []
---

# Using Skillbook

This skill establishes the mandatory workflow for skillbook projects.

## MANDATORY WORKFLOW

Before implementing ANY feature, follow these phases in order:

1. **PLAN** - Brainstorm and write detailed plans
2. **DOCUMENT** - ADRs, arc42, requirements
3. **MODEL** - Architecture, data schemas, threats, UI mockups
4. **TEST** - Write failing test first
5. **DEVELOP** - REPL-driven, quality checks
6. **VERIFY** - Playwright tests, human review

**NEVER skip phases. NEVER jump to code.**

**Process Weight:** This workflow is intentionally heavy. If you think it's excessive for a task, **ask the user** how to proceed. Don't silently skip phases.

**Task Tracking:** Use `beads-tasks` skill in EVERY phase. Create beads tasks as you enter each phase, mark them done as you complete work. This creates a traceable record.

See `docs/workflow.md` for complete workflow documentation.

## Agents

When implementing features, dispatch the `skillbook:clojure-developer` agent. It enforces TDD + REPL workflow.

## Review Checkpoints

| Phase | Method |
|-------|--------|
| Brainstorm | Ask user |
| Document (ADRs) | Ask user |
| Model (architecture) | Zen MCP or ask user |
| Model (threats) | Ask user |
| UI mockup | Ask user |
| Write test | superpowers:requesting-code-review |
| Component in library | Playwright |
| Develop (integrate) | superpowers:requesting-code-review |
| Feature complete | Playwright + ask user |
| Before merge | superpowers:requesting-code-review |

## Skills by Phase

### Every feature (always)
- `superpowers:brainstorming` - before implementing anything
- `superpowers:writing-plans` - detailed implementation steps
- `superpowers:test-driven-development` - write test first
- `clojure-repl` - REPL for all Clojure development
- `quality-check` - before every commit
- `superpowers:verification-before-completion` - before claiming done

### Project initialization
- `project-init`, `init-obsidian-vault`
- `git-lint-setup`, `milestoner-releases`, `beads-tasks`
- `clj-kondo-linting`, `cljfmt-formatting`, `splint-linting`

### Architecture changes
- `adr-management` - document decision first
- `overarch-modeling` → `structurizr-diagrams` → `sync-architecture`
- `arc42-docs`

### Data structures
- `malli-schemas` → `property-testing` → `guardrails-contracts`

### Requirements
- `iso25010-quality`, `scenari-specs`

### Concurrent/distributed logic
- `tla-concepts` → `recife-modeling`

### Security
- `threagile-analysis` → `policy-as-code` → `conjtest-testing`
- `dependency-audit`

### UI work
- `ui-mockups` - HTML/CSS mockup, human approves
- `zero-components` - component library first
- `playwright-testing` - verify component and feature

### Infrastructure
- `optimizing-dockerfiles`

### Code style
- `clojure-style`

### Feedback
- `gemini-feedback`, `review-checkpoints`
- `superpowers:requesting-code-review`

### When things break
- `error-recovery`, `superpowers:systematic-debugging`

## Tool Requirements

All tools are blocking. If missing, stop and notify with install instructions.

See `docs/workflow.md` for complete tool list.

## Obsidian Linking

All vault content must use `[[links]]`. See `docs/workflow.md` for linking conventions.

## Success Criteria

- [ ] Workflow phases followed in order
- [ ] Review checkpoints completed
- [ ] All vault content interconnected with links
- [ ] Quality checks pass before completion
