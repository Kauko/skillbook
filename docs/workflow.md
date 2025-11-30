# Skillbook Workflow

Design-first, assurance-driven development for Clojure projects.

## Overview

Every feature follows this sequence:

```
PLAN → DOCUMENT → MODEL → TEST → DEVELOP → VERIFY
```

Never skip phases. Never jump to code.

## Phases

### 1. Plan

**Purpose:** Understand what we're building before writing anything.

**Skills:**
- `superpowers:brainstorming` - Refine the idea through dialogue
- `superpowers:writing-plans` - Create detailed implementation steps

**Outputs:**
- Design document in `docs/plans/YYYY-MM-DD-<feature>-design.md`
- Implementation plan in `docs/plans/YYYY-MM-DD-<feature>-implementation.md`

**Review checkpoint:** Ask user - direction needs human judgment.

### 2. Document

**Purpose:** Record decisions and requirements before implementation.

**Skills:**
- `adr-management` - Architectural Decision Records
- `arc42-docs` - Architecture documentation sections
- `iso25010-quality` - Quality requirements

**Outputs:**
- ADRs in `vault/decisions/`
- Updated arc42 sections in `vault/arc42/`
- Quality requirements in `vault/requirements/`

**Review checkpoint:** Ask user - architectural decisions need stakeholder input.

**Linking:** ADRs link to related ADRs, arc42 sections, beads tasks, glossary terms.

### 3. Model

**Purpose:** Define structure before code.

**Skills:**
- `overarch-modeling` - Architecture model (C4)
- `structurizr-diagrams` - Generate diagrams
- `malli-schemas` - Data schemas
- `threagile-analysis` - Threat model (if security-relevant)
- `recife-modeling` - Formal specs (if concurrent/distributed)
- `ui-mockups` - HTML/CSS mockups (if UI work)

**Outputs:**
- Architecture model in `models/`
- C4 diagrams in `vault/architecture/diagrams/`
- Malli schemas in source code
- Threat model in `vault/security/`
- Mockups in `mockups/`

**Review checkpoints:**
- Architecture: Zen MCP or ask user
- Threats: Ask user - security decisions need human sign-off
- UI mockups: Ask user - human approves visual design before code

**Linking:** Models link to ADRs, arc42 sections. Components link to mockups.

### 4. Test

**Purpose:** Write failing tests before implementation.

**Skills:**
- `superpowers:test-driven-development` - TDD workflow
- `property-testing` - Generate tests from Malli schemas

**Outputs:**
- Failing test files

**Review checkpoint:** `superpowers:requesting-code-review` - technical review of test design.

**Rules:**
- Write the test first
- Run it to see it fail
- Only then implement

### 5. Develop

**Purpose:** Implement using REPL-driven development.

**Skills:**
- `clojure-repl` - REPL evaluation for everything
- `zero-components` - UI components in library first
- `clojure-style` - Style conventions
- `quality-check` - Linting and formatting

**Outputs:**
- Implementation code
- Components in `vault/components/`
- Passing tests

**Review checkpoints:**
- Component in library: Playwright (automated)
- After integration: `superpowers:requesting-code-review`

**Rules:**
- Always use REPL - evaluate each function as you write it
- Components go in library first, then integrate
- Run quality-check before claiming done

**Linking:** Component docs link to mockups, related components.

### 6. Verify

**Purpose:** Confirm everything works before completion.

**Skills:**
- `playwright-testing` - UI verification
- `superpowers:verification-before-completion` - Final checks

**Outputs:**
- Passing Playwright tests
- Human approval of completed feature

**Review checkpoints:**
- Playwright tests pass (automated)
- Ask user to review completed feature
- `superpowers:requesting-code-review` before merge

## Review Checkpoints Summary

| Phase | Method | Rationale |
|-------|--------|-----------|
| Plan (brainstorm) | Ask user | Direction needs human judgment |
| Document (ADRs) | Ask user | Architectural decisions need stakeholder input |
| Model (architecture) | Zen MCP or ask user | Quick external validation |
| Model (threats) | Ask user | Security decisions need human sign-off |
| Model (UI mockup) | Ask user | Human approves visual design |
| Test (design) | superpowers:requesting-code-review | Technical review |
| Develop (component) | Playwright | Automated verification |
| Develop (integration) | superpowers:requesting-code-review | Code review |
| Verify (feature) | Playwright + ask user | Tests + human sees it working |
| Verify (merge) | superpowers:requesting-code-review | Final review |

## Tool Requirements

All tools are blocking. Stop and notify if any are missing.

### Clojure Development
| Tool | Install Command |
|------|-----------------|
| `bb` | `brew install borkdude/brew/babashka` |
| `clj-nrepl-eval` | See clojure-repl skill |
| `clojure` | `brew install clojure/tools/clojure` |
| `clj-kondo` | `brew install borkdude/brew/clj-kondo` |
| `splint` | `brew install noahtheduke/tap/splint` |
| `shadow-cljs` | `npm install -g shadow-cljs` |

### Architecture & Security
| Tool | Install Command |
|------|-----------------|
| `overarch` | See overarch-modeling skill |
| `structurizr-cli` | `brew install structurizr-cli` |
| `threagile` | See threagile-analysis skill |
| `opa` | `brew install opa` |
| `conftest` | `brew install conftest` |

### Formal Verification
| Tool | Install Command |
|------|-----------------|
| `tlc` | See recife-modeling skill |

### MCPs
| MCP | Purpose |
|-----|---------|
| Playwright MCP | UI testing |
| deepwiki MCP | Documentation lookup |

## UI Workflow Detail

```
1. Create mockup (HTML/CSS in mockups/)
2. Ask user to review mockup
3. User approves visual design
4. Write failing test for component
5. Implement component (zero-components, in library)
6. Playwright verifies component
7. Integrate component into feature
8. Code review
9. Playwright verifies feature
10. Ask user to review completed feature
```

## Obsidian Linking Conventions

All vault content must be interconnected:

| From | To | Example |
|------|----|---------|
| ADR | Related ADRs | `Supersedes [[0001-use-clojure]]` |
| ADR | Arc42 sections | `See [[04-solution#auth]]` |
| ADR | Beads task | `Task: [[beads/TASK-123]]` |
| Arc42 | ADRs | `Decision: [[decisions/0005-use-malli]]` |
| Arc42 | Threats | `Threats: [[security/threats#data-flow]]` |
| Component | Mockup | `Design: [[mockups/button]]` |
| Component | Related | `See also [[components/icon]]` |
| Any | Glossary | `[[glossary#term]]` on first use |

## Skills Reference

See `using-skillbook` skill for complete skills-by-phase list.

## Agents

| Agent | Purpose | Invoked by |
|-------|---------|------------|
| `skillbook:project-init` | Initialize new project | `/skillbook:new-project` |
| `skillbook:clojure-developer` | Implement features with TDD+REPL | Claude or human |
