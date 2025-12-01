# Skillbook Workflow Enforcement Design

## Problem Statement

Skillbook provides 34 skills across 15 plugins for design-first, assurance-driven development. However, Claude Code doesn't follow the intended workflow automatically:

1. **Skills are reactive** - Claude must choose to load them, but often skips to code directly
2. **Tool availability ignored** - Claude continues silently when required tools/MCPs are missing
3. **Workflow not enforced** - The plan → document → model → test → develop → verify sequence isn't followed
4. **REPL underutilized** - Claude doesn't use REPL-driven development even when available
5. **No UI verification** - Playwright MCP not used, mockups not created before implementation

## Solution Overview

Three complementary pieces:

1. **`/skillbook:new-project` slash command** - Explicit entry point that enforces workflow from the start
2. **`using-skillbook` skill** - Loaded at session start, provides comprehensive workflow reference
3. **CLAUDE.md template** - Short directive file created in each project, points to the skill

Plus supporting skills for gaps in current coverage, and two agents for workflow enforcement.

## Components

### 1. `/skillbook:new-project` Slash Command

Location: `skillbook-core/commands/new-project.md`

**Behavior:**

```
/skillbook:new-project
    │
    ▼
PRE-FLIGHT CHECK
    ├─ Check all required tools (see Tool Requirements below)
    ├─ Check required MCPs (playwright, deepwiki)
    ├─ Report missing pieces with install instructions
    └─ Ask: "Continue anyway?" or "Fix first?"
    │
    ▼
BRAINSTORMING (superpowers:brainstorming)
    ├─ "What are we building?"
    ├─ Clarify scope, constraints, quality attributes
    └─ Output: design document in docs/plans/
    │
    ▼
INITIALIZATION (project-init skill)
    ├─ Create vault/ structure
    ├─ Create CLAUDE.md from template
    ├─ Set up quality tools
    └─ Initialize git workflow
    │
    ▼
AUTONOMOUS WORKFLOW
    └─ Claude follows using-skillbook guidance
```

### 2. `using-skillbook` Skill

Location: `skillbook-core/skills/using-skillbook.md`

**Purpose:** Loaded at session start, enforces the workflow, provides complete skills reference.

**Content structure:**

```markdown
---
name: using-skillbook
description: Use when starting any conversation in a skillbook project - establishes the plan → document → model → test → develop → verify workflow
---

# Using Skillbook

## MANDATORY WORKFLOW

Before implementing ANY feature, follow these phases in order:

1. PLAN - Brainstorm and write detailed plans
2. DOCUMENT - ADRs, arc42, requirements
3. MODEL - Architecture, data schemas, threats, UI mockups
4. TEST - Write failing test first
5. DEVELOP - REPL-driven, quality checks
6. VERIFY - Playwright tests, human review

NEVER skip phases. NEVER jump to code.

## Review Checkpoints

[Phase-dependent feedback table - see below]

## Skills by Phase

[Complete categorized list - see below]

## Tool Requirements

[All tools with install commands]
```

### 3. CLAUDE.md Template

Created by `/skillbook:new-project` in each project.

```markdown
# {Project Name}

This is a skillbook project. Follow the plan → document → model → test → develop → verify workflow.

Load the `using-skillbook` skill for the complete reference.

## Phases (never skip)
1. Plan - brainstorm, write plans
2. Document - ADRs, arc42, requirements
3. Model - architecture, data schemas, threats, UI mockups
4. Test - write failing test first
5. Develop - REPL-driven, quality checks
6. Verify - Playwright tests, human review

## Tools
- REPL: clojure-repl skill
- UI testing: Playwright MCP
- Documentation lookup: deepwiki MCP
- Mockups: ui-mockups skill
```

## Review Checkpoints

Phase-dependent feedback methods:

| Phase | Feedback Method | Rationale |
|-------|-----------------|-----------|
| Brainstorm | Ask user | Direction needs human judgment |
| Document (ADRs) | Ask user | Architectural decisions need stakeholder input |
| Model (architecture) | Zen MCP or ask user | Can get quick external validation |
| Model (threats) | Ask user | Security decisions need human sign-off |
| UI mockup | Ask user | Human approves visual design before any code |
| Write test | superpowers:requesting-code-review | Technical review of test design |
| Component in library | Playwright | Automated verification that component works |
| Develop (integrate) | superpowers:requesting-code-review | Code review after integration |
| Feature complete | Playwright + ask user | Automated tests + human sees it working |
| Before merge | superpowers:requesting-code-review | Final technical review |

## Skills by Phase

### Every feature (always follow this loop)
- superpowers:brainstorming → before implementing anything
- superpowers:writing-plans → detailed implementation steps
- superpowers:test-driven-development → write test first, watch it fail
- clojure-repl → use REPL for all Clojure development
- quality-check → before every commit
- superpowers:verification-before-completion → before claiming done

### Project initialization (once per project)
- project-init → orchestrates setup
- init-obsidian-vault → creates vault/ structure
- git-lint-setup → commit conventions
- milestoner-releases → changelog setup
- beads-tasks → task tracking setup
- clj-kondo-linting → static analysis setup
- cljfmt-formatting → formatter setup
- splint-linting → idiom linter setup

### Architecture changes
- adr-management → document the decision first
- overarch-modeling → update architecture model
- structurizr-diagrams → generate C4 diagrams
- sync-architecture → regenerate all derived formats
- arc42-docs → update relevant sections

### Data structures
- malli-schemas → define schema first
- property-testing → generate test cases from schemas
- guardrails-contracts → runtime validation

### Requirements & quality attributes
- iso25010-quality → define quality requirements
- scenari-specs → Gherkin BDD specs
- slo-sli-observability → define SLOs/SLIs

### Concurrent or distributed logic
- tla-concepts → understand TLA+ concepts
- recife-modeling → formal specification, run TLC

### Security-sensitive changes
- threagile-analysis → threat modeling
- policy-as-code → generate OPA/Rego policies
- conjtest-testing → validate against policies
- dependency-audit → check dependencies

### UI work
- ui-mockups → HTML/CSS mockup, human approves before code
- zero-components → implement in component library first
- playwright-testing → verify component works, then verify feature

### Infrastructure
- terraform-iac → generate from architecture
- optimizing-dockerfiles → Docker optimization

### Code style (continuous)
- clojure-style → conventions reference

### Feedback & validation
- gemini-feedback → external model feedback
- review-checkpoints → phase-dependent feedback
- superpowers:requesting-code-review → technical code review

### When things break
- error-recovery → systematic failure handling
- superpowers:systematic-debugging → root cause analysis

## Agents

Two agents for workflow enforcement. Starting with 2 to validate the approach before adding more.

### 1. `skillbook:project-init`

Location: `skillbook-core/agents/project-init.md`

**Purpose:** Runs the full project initialization workflow in isolation.

**Invoked by:** `/skillbook:new-project` slash command

**Workflow:**
1. Pre-flight check (all tools, all MCPs)
2. Report missing with install instructions
3. Ask: continue or fix first?
4. Run superpowers:brainstorming
5. Run project-init skill (creates vault, CLAUDE.md, etc.)
6. Report completion, hand back to main conversation

**System prompt includes:**
- All tool requirements with install commands
- Brainstorming guidance
- Project structure expectations
- Success criteria

**Tools available:**
- All read/search tools (Glob, Grep, Read)
- All write tools (Write, Edit)
- Bash for tool checks
- AskUserQuestion for decisions

### 2. `skillbook:clojure-developer`

Location: `skillbook-core/agents/clojure-developer.md`

**Purpose:** Implements features following TDD + REPL workflow.

**Invoked by:**
- Claude Code (when user asks to implement something)
- Human directly (dispatching the agent)

**Workflow:**
1. Check REPL is available (`clj-nrepl-eval --discover-ports`)
2. If UI work: create mockup first, ask user approval
3. Write failing test (TDD)
4. Implement using REPL (evaluate incrementally)
5. If UI: implement component in library, Playwright verify
6. Integrate, Playwright verify feature
7. Run quality-check
8. Request code review (superpowers:requesting-code-review)
9. Report completion

**System prompt includes:**
- TDD rules (test first, watch it fail)
- REPL usage (always evaluate, never just write)
- UI workflow (mockup → approve → component → integrate)
- Quality gates (quality-check before done)
- Linking requirements for any vault content created

**Tools available:**
- All read/search tools
- All write tools
- Bash for REPL, quality tools, Playwright
- AskUserQuestion for mockup approval, feature review

**Key enforcement:**
```
NEVER write Clojure code without:
1. A failing test first
2. REPL evaluation of each function
3. quality-check passing before claiming done
```

## New Skills to Create

### 1. `using-skillbook`
- **Location:** `skillbook-core/skills/using-skillbook.md`
- **Purpose:** Session start skill, workflow enforcement
- **Contains:** Mandatory workflow, review checkpoints, skills-by-phase reference, tool requirements

### 2. `review-checkpoints`
- **Location:** `skillbook-core/skills/review-checkpoints.md`
- **Purpose:** Define when and how to get feedback
- **Contains:** Phase-dependent feedback table, how to use each method (ask user, Zen MCP, superpowers:requesting-code-review)

### 3. `ui-mockups`
- **Location:** `clojure-dev/skills/ui-mockups.md`
- **Purpose:** Create HTML/CSS mockups before implementation
- **Workflow:**
  1. Create mockup in `mockups/` directory
  2. Ask user to review
  3. Only proceed to implementation after approval
- **Requires:** None (just HTML/CSS files)

### 4. `playwright-testing`
- **Location:** `clojure-dev/skills/playwright-testing.md`
- **Purpose:** UI testing with Playwright MCP
- **Requires:** Playwright MCP installed
- **Contains:** How to write tests, verify components, verify features

### 5. `deepwiki-lookup`
- **Location:** `skillbook-core/skills/deepwiki-lookup.md`
- **Purpose:** Documentation lookup when information is missing
- **Requires:** deepwiki MCP installed
- **Contains:** When to use, how to query, integrating results

## Skill Updates

### `zero-components` (update)
- **Add:** Component library workflow
- **New content:**
  - Components must be created in library first
  - Playwright verifies component works before integration
  - Link to ui-mockups skill (mockup must be approved first)

## Slash Command to Create

### `/skillbook:new-project`
- **Location:** `skillbook-core/commands/new-project.md`
- **Behavior:** Pre-flight check → brainstorm → init → autonomous workflow

## Tool Requirements

All tools are **blocking** - Claude should stop and notify if any are missing.

### Clojure Development
| Tool | Install | Purpose |
|------|---------|---------|
| `bb` | `brew install borkdude/brew/babashka` | Scripting, running tools |
| `clj-nrepl-eval` | `bbin install ...` (see clojure-repl skill) | REPL evaluation |
| `clojure` | `brew install clojure/tools/clojure` | Clojure CLI |
| `clj-kondo` | `brew install borkdude/brew/clj-kondo` | Static analysis |
| `splint` | `brew install noahtheduke/tap/splint` | Idiom linting |
| `shadow-cljs` | `npm install -g shadow-cljs` | ClojureScript |

### Architecture & Modeling
| Tool | Install | Purpose |
|------|---------|---------|
| `overarch` | See overarch-modeling skill | Architecture model |
| `structurizr-cli` | `brew install structurizr-cli` | C4 diagrams |

### Security & Compliance
| Tool | Install | Purpose |
|------|---------|---------|
| `threagile` | See threagile-analysis skill | Threat modeling |
| `opa` | `brew install opa` | Policy validation |
| `conftest` | `brew install conftest` | Policy testing |

### Formal Verification
| Tool | Install | Purpose |
|------|---------|---------|
| `tlc` | See recife-modeling skill | TLA+ model checking |

### Infrastructure
| Tool | Install | Purpose |
|------|---------|---------|
| `terraform` | `brew install terraform` | IaC |
| `docker` | Docker Desktop | Containers |

### MCPs
| MCP | Purpose |
|-----|---------|
| Playwright MCP | UI testing |
| deepwiki MCP | Documentation lookup |

### Other
| Tool | Install | Purpose |
|------|---------|---------|
| `bd` (beads) | See beads-tasks skill | Task tracking |

## UI Workflow

Complete flow for UI work:

```
1. UI Mockup
   └─ Create HTML/CSS in mockups/
   └─ Ask user to review
   └─ Human approves visual design

2. Write Test
   └─ Write failing test for component
   └─ superpowers:requesting-code-review

3. Implement Component
   └─ zero-components skill
   └─ Create in component library (vault/components/)
   └─ Demo in vault/components/demos/

4. Verify Component
   └─ Playwright tests component in isolation
   └─ Automated, no human needed

5. Integrate
   └─ Add component to feature
   └─ superpowers:requesting-code-review

6. Verify Feature
   └─ Playwright tests full feature
   └─ Ask user to review completed feature

7. Merge
   └─ superpowers:requesting-code-review
```

## Implementation Order

1. `docs/workflow.md` - Canonical workflow reference (agents/skills reference this)
2. `using-skillbook` skill - Core workflow enforcement
3. `skillbook:project-init` agent - Project initialization
4. `skillbook:clojure-developer` agent - Development workflow enforcement
5. `/skillbook:new-project` command - Entry point (invokes project-init agent)
6. `review-checkpoints` skill - Feedback methods
7. `ui-mockups` skill - Mockup workflow
8. `playwright-testing` skill - UI verification
9. `deepwiki-lookup` skill - Documentation lookup
10. Update `zero-components` - Component library workflow
11. Update existing skills with linking requirements (9 skills)
12. CLAUDE.md template - Created by slash command

## Obsidian Linking Requirements

All vault content must be interconnected. Linking guidance is **embedded in each skill** that creates vault content, not a separate skill.

### Link Types

| From | To | Example |
|------|----|---------|
| ADR | Related ADRs | `Supersedes [[0001-use-clojure]]` |
| ADR | Arc42 sections | `See [[04-solution#authentication]]` |
| ADR | Beads task | `Task: [[beads/TASK-123]]` |
| Arc42 section | ADRs | `Decision: [[decisions/0005-use-malli]]` |
| Arc42 section | Threat model | `Threats: [[security/threats#data-flow]]` |
| Arc42 section | Components | `Uses [[components/button]]` |
| Component doc | Mockup | `Design: [[mockups/button]]` |
| Component doc | Related components | `See also [[components/icon]]` |
| Feature spec | Components used | `Components: [[components/button]], [[components/card]]` |
| Feature spec | Beads task | `Task: [[beads/FEAT-456]]` |
| Threat model | ADRs | `Mitigation: [[decisions/0008-encrypt-at-rest]]` |
| Any document | Glossary terms | `[[glossary#domain-term]]` on first use |

### Embedding in Skills

Each skill that creates vault content includes a "Linking Requirements" section:

**Example for `adr-management`:**
```markdown
## Linking Requirements
When creating an ADR:
- Link to superseded ADRs: `Supersedes [[0001-old-decision]]`
- Link to related arc42 sections: `See [[04-solution#relevant-heading]]`
- Link to beads task if applicable: `Task: [[beads/TASK-123]]`
- Link to glossary terms on first use: `[[glossary#term]]`
```

**Example for `zero-components`:**
```markdown
## Linking Requirements
When documenting a component:
- Link to approved mockup: `Design: [[mockups/component-name]]`
- Link to related components: `See also [[components/related]]`
- Link to features using this component: `Used in [[features/feature-name]]`
```

### Skills Requiring Link Guidance

These skills create vault content and need embedded linking requirements:

- `adr-management` - ADRs link to other ADRs, arc42, beads, glossary
- `arc42-docs` - Sections link to ADRs, threats, components, specs
- `init-obsidian-vault` - Creates glossary template with linking conventions
- `iso25010-quality` - Quality requirements link to arc42 section 10
- `scenari-specs` - Specs link to components, ADRs, beads tasks
- `overarch-modeling` - Model docs link to arc42, ADRs
- `threagile-analysis` - Threat docs link to ADRs (mitigations), arc42
- `zero-components` - Component docs link to mockups, related components
- `ui-mockups` - Mockups link to requirements, components (after implementation)
- `beads-tasks` - Tasks link to ADRs, specs, components they implement

## Workflow Documentation

The canonical workflow reference lives at `docs/workflow.md` in this repository.

This file:
- Explains the full plan → document → model → test → develop → verify cycle
- Documents all phases with their skills
- Provides examples of the workflow in action
- Is the source of truth that `using-skillbook` skill references

**Why this matters:** Without this documentation, every new conversation requires re-explaining the entire workflow. With it, Claude (and humans) can reference a single authoritative source.

### `docs/workflow.md` Structure

```markdown
# Skillbook Workflow

## Overview
Design-first, assurance-driven development.

## Phases

### 1. Plan
[Skills, outputs, review checkpoint]

### 2. Document
[Skills, outputs, review checkpoint]

### 3. Model
[Skills, outputs, review checkpoint]

### 4. Test
[Skills, outputs, review checkpoint]

### 5. Develop
[Skills, outputs, review checkpoint]

### 6. Verify
[Skills, outputs, review checkpoint]

## Review Checkpoints
[Full table with rationale]

## Tool Requirements
[Full list with install commands]

## Examples
[Concrete workflow examples]
```

## Success Criteria

- [ ] `/skillbook:new-project` creates a fully configured project
- [ ] Pre-flight check catches missing tools with install instructions
- [ ] Claude follows plan → document → model → test → develop → verify
- [ ] Claude uses REPL for all Clojure development
- [ ] UI mockups are created and approved before implementation
- [ ] Components are created in library first, verified by Playwright
- [ ] Review checkpoints happen at each phase
- [ ] Human reviews mockups and completed features
- [ ] Claude stops and notifies when tools are missing
- [ ] All vault content is interconnected with `[[links]]`
- [ ] `docs/workflow.md` exists and is comprehensive
- [ ] `skillbook:project-init` agent runs full initialization workflow
- [ ] `skillbook:clojure-developer` agent enforces TDD + REPL
- [ ] Claude dispatches clojure-developer agent when implementing features
