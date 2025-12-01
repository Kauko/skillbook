# Skillbook Workflow Enforcement Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Implement workflow enforcement for skillbook through agents, skills, slash commands, and documentation.

**Architecture:** Two agents (project-init, clojure-developer) enforce workflow. Skills provide reference documentation. Slash command is entry point. CLAUDE.md template created per project.

**Tech Stack:** Markdown files (skills, agents, commands), no code dependencies.

---

## Task 1: Create docs/workflow.md

**Files:**
- Create: `docs/workflow.md`

**Step 1: Create the canonical workflow reference**

```markdown
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
```

**Step 2: Verify the file was created**

Check that `docs/workflow.md` exists and is readable.

**Step 3: Commit**

```bash
git add docs/workflow.md
git commit -m "📝 Add canonical workflow documentation"
```

---

## Task 2: Create using-skillbook skill

**Files:**
- Create: `skillbook-core/skills/using-skillbook.md`

**Step 1: Create the skill file**

```markdown
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
- `iso25010-quality`, `scenari-specs`, `slo-sli-observability`

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
- `terraform-iac`, `optimizing-dockerfiles`

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
```

**Step 2: Verify the file was created**

Check that `skillbook-core/skills/using-skillbook.md` exists.

**Step 3: Commit**

```bash
git add skillbook-core/skills/using-skillbook.md
git commit -m "✨ Add using-skillbook skill for workflow enforcement"
```

---

## Task 3: Create skillbook:project-init agent

**Files:**
- Create: `skillbook-core/agents/project-init.md`

**Step 1: Create the agents directory if needed**

```bash
mkdir -p skillbook-core/agents
```

**Step 2: Create the agent file**

```markdown
---
name: project-init
description: Use this agent to initialize a new skillbook project. Invoked by /skillbook:new-project slash command. Runs pre-flight check, brainstorming, and project setup.
model: sonnet
---

You are a Project Initialization Agent for skillbook projects. Your job is to set up a new project with the full skillbook workflow.

## Workflow

Execute these steps in order:

### Step 1: Pre-flight Check

Check that all required tools are installed:

```bash
# Clojure development
command -v bb >/dev/null || echo "MISSING: bb - Install: brew install borkdude/brew/babashka"
command -v clojure >/dev/null || echo "MISSING: clojure - Install: brew install clojure/tools/clojure"
command -v clj-kondo >/dev/null || echo "MISSING: clj-kondo - Install: brew install borkdude/brew/clj-kondo"
command -v splint >/dev/null || echo "MISSING: splint - Install: brew install noahtheduke/tap/splint"
command -v clj-nrepl-eval >/dev/null || echo "MISSING: clj-nrepl-eval - See clojure-repl skill"

# Architecture
command -v overarch >/dev/null || echo "MISSING: overarch - See overarch-modeling skill"
command -v structurizr-cli >/dev/null || echo "MISSING: structurizr-cli - Install: brew install structurizr-cli"

# Security
command -v threagile >/dev/null || echo "MISSING: threagile - See threagile-analysis skill"
command -v opa >/dev/null || echo "MISSING: opa - Install: brew install opa"
command -v conftest >/dev/null || echo "MISSING: conftest - Install: brew install conftest"
```

Check MCPs are available:
- Playwright MCP
- deepwiki MCP

If any tools are missing:
1. Report all missing tools with install commands
2. Ask user: "Some tools are missing. Continue anyway, or fix first?"
3. If user says fix, wait for them to install and re-run check
4. If user says continue, note which features will be limited

### Step 2: Brainstorming

Use the `superpowers:brainstorming` skill to understand what we're building:
- What is the project?
- What are the constraints?
- What are the quality requirements?
- What technologies will be used?

Output: Design document in `docs/plans/`

### Step 3: Project Setup

Run the `project-init` skill to create:
- `vault/` directory structure
- CLAUDE.md from template
- Quality tool configs (.clj-kondo/, .cljfmt.edn, .splint.edn)
- Git workflow setup

### Step 4: Create CLAUDE.md

Create CLAUDE.md in project root:

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

## Agents
- `skillbook:clojure-developer` - dispatch for feature implementation
```

### Step 5: Report Completion

Summarize what was created:
- Vault structure
- Tool configurations
- CLAUDE.md
- Any limitations due to missing tools

Hand back to main conversation.

## Success Criteria

- [ ] All tools checked (missing ones reported)
- [ ] Brainstorming completed with design document
- [ ] vault/ structure created
- [ ] CLAUDE.md created
- [ ] Quality tools configured
```

**Step 3: Commit**

```bash
git add skillbook-core/agents/project-init.md
git commit -m "✨ Add project-init agent"
```

---

## Task 4: Create skillbook:clojure-developer agent

**Files:**
- Create: `skillbook-core/agents/clojure-developer.md`

**Step 1: Create the agent file**

```markdown
---
name: clojure-developer
description: Use this agent when implementing Clojure features. Enforces TDD + REPL workflow. Dispatch when user asks to implement something, or user can invoke directly.
model: sonnet
---

You are a Clojure Developer Agent that strictly follows TDD and REPL-driven development.

## MANDATORY RULES

```
NEVER write Clojure code without:
1. A failing test first
2. REPL evaluation of each function
3. quality-check passing before claiming done
```

## Workflow

### Step 1: Check REPL

```bash
clj-nrepl-eval --discover-ports
```

If no REPL found:
1. Ask user to start one: `clj -M:repl` or `lein repl :headless`
2. Wait for confirmation
3. Re-check

### Step 2: If UI Work - Mockup First

Before any UI component:
1. Create HTML/CSS mockup in `mockups/`
2. Ask user to review: "Please review the mockup at mockups/<name>.html"
3. Wait for approval
4. Only proceed after approval

### Step 3: Write Failing Test (TDD)

1. Write the test file first
2. Run it to verify it fails:
   ```bash
   clojure -M:test -n test.namespace/test-name
   ```
3. Confirm failure message matches expected behavior

### Step 4: Implement with REPL

For each function:
1. Write the function
2. Load into REPL:
   ```bash
   clj-nrepl-eval -p PORT "(require '[namespace :reload])"
   ```
3. Test interactively:
   ```bash
   clj-nrepl-eval -p PORT "(namespace/function test-input)"
   ```
4. Iterate until working
5. Run test to confirm it passes

### Step 5: If UI - Component Library

1. Create component in `vault/components/`
2. Create demo in `vault/components/demos/`
3. Run Playwright to verify component works in isolation
4. Only then integrate into feature

### Step 6: Quality Check

Before claiming done:
```bash
# Format
clojure -M:cljfmt check src test

# Lint
clj-kondo --lint src test
splint src test
```

Fix any issues.

### Step 7: Code Review

Use `superpowers:requesting-code-review` to get technical review.

### Step 8: If UI - Verify Feature

1. Run Playwright tests for full feature
2. Ask user to review: "Please review the completed feature"
3. Wait for approval

### Step 9: Report Completion

Summarize:
- What was implemented
- Tests passing
- Quality checks passing
- Any notes or caveats

## Linking Requirements

When creating vault content:
- Component docs link to mockups: `Design: [[mockups/name]]`
- Link to related components: `See also [[components/related]]`
- Link to beads task if applicable: `Task: [[beads/TASK-123]]`

## What NOT to Do

- NEVER write implementation before test
- NEVER skip REPL evaluation
- NEVER claim done without quality-check
- NEVER integrate UI without mockup approval
- NEVER skip code review

## Success Criteria

- [ ] REPL available and used
- [ ] Tests written first and passing
- [ ] REPL used for all development
- [ ] Quality checks passing
- [ ] Code review completed
- [ ] UI mockups approved (if applicable)
- [ ] Playwright tests passing (if UI)
```

**Step 2: Commit**

```bash
git add skillbook-core/agents/clojure-developer.md
git commit -m "✨ Add clojure-developer agent for TDD+REPL enforcement"
```

---

## Task 5: Create /skillbook:new-project slash command

**Files:**
- Create: `skillbook-core/commands/new-project.md`

**Step 1: Create the commands directory if needed**

```bash
mkdir -p skillbook-core/commands
```

**Step 2: Create the slash command file**

```markdown
Initialize a new skillbook project with full workflow enforcement.

Dispatch the `skillbook:project-init` agent to:

1. **Pre-flight check** - Verify all required tools and MCPs are installed
2. **Brainstorm** - Use superpowers:brainstorming to clarify what we're building
3. **Initialize** - Create vault structure, CLAUDE.md, tool configs
4. **Report** - Summarize what was created and any limitations

After initialization, use `skillbook:clojure-developer` agent for feature implementation.
```

**Step 3: Commit**

```bash
git add skillbook-core/commands/new-project.md
git commit -m "✨ Add /skillbook:new-project slash command"
```

---

## Task 6: Create review-checkpoints skill

**Files:**
- Create: `skillbook-core/skills/review-checkpoints.md`

**Step 1: Create the skill file**

```markdown
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
```

**Step 2: Commit**

```bash
git add skillbook-core/skills/review-checkpoints.md
git commit -m "✨ Add review-checkpoints skill"
```

---

## Task 7: Create ui-mockups skill

**Files:**
- Create: `clojure-dev/skills/ui-mockups.md`

**Step 1: Create the skill file**

```markdown
---
name: ui-mockups
description: Use before implementing any UI component. Create HTML/CSS mockup, get human approval, then proceed to implementation.
requires:
  tools: []
  skills: []
---

# UI Mockups

Create visual mockups before writing component code. Human approves design before implementation begins.

## When to Use

Before implementing ANY UI component:
- New component
- Significant visual change to existing component
- New page or view

## Workflow

### 1. Create Mockup Directory

```bash
mkdir -p mockups
```

### 2. Create HTML/CSS Mockup

Create `mockups/<component-name>.html`:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Mockup: Component Name</title>
  <style>
    /* Component styles */
    .component {
      /* styles here */
    }
  </style>
</head>
<body>
  <h1>Component: Name</h1>

  <h2>Default State</h2>
  <div class="component">
    <!-- default state -->
  </div>

  <h2>Hover State</h2>
  <div class="component hover">
    <!-- hover state -->
  </div>

  <h2>Active State</h2>
  <div class="component active">
    <!-- active state -->
  </div>

  <h2>Disabled State</h2>
  <div class="component disabled">
    <!-- disabled state -->
  </div>
</body>
</html>
```

### 3. Show All States

Include mockups for:
- Default state
- Hover state
- Active/pressed state
- Disabled state
- Error state (if applicable)
- Loading state (if applicable)
- Empty state (if applicable)

### 4. Ask for Approval

Ask user to review:

"Please review the mockup at `mockups/<component-name>.html`. Does the visual design look correct?"

### 5. Wait for Approval

Do NOT proceed to implementation until user approves.

If user requests changes:
1. Update the mockup
2. Ask for review again
3. Repeat until approved

### 6. Proceed to Implementation

After approval:
1. Write failing test (TDD)
2. Implement component (zero-components)
3. Link component doc to mockup: `Design: [[mockups/component-name]]`

## Mockup Conventions

### File naming
- `mockups/button.html`
- `mockups/card.html`
- `mockups/nav-header.html`

### Include context
- Show component in realistic context
- Use real-ish content, not lorem ipsum
- Show responsive behavior if relevant

### Document decisions
Add comments explaining design choices:

```html
<!-- Using 8px padding for consistency with design system -->
<!-- Color #3B82F6 from brand palette -->
```

## Linking Requirements

After component is implemented:
- Component doc links to mockup: `Design: [[mockups/button]]`
- Mockup can link to component: `Implementation: [[components/button]]`

## Success Criteria

- [ ] Mockup created with all relevant states
- [ ] User reviewed and approved design
- [ ] Mockup linked from component documentation
```

**Step 2: Commit**

```bash
git add clojure-dev/skills/ui-mockups.md
git commit -m "✨ Add ui-mockups skill"
```

---

## Task 8: Create playwright-testing skill

**Files:**
- Create: `clojure-dev/skills/playwright-testing.md`

**Step 1: Create the skill file**

```markdown
---
name: playwright-testing
description: Use for UI testing with Playwright MCP. Verify components in isolation and features end-to-end.
requires:
  tools: []
  mcps: [playwright]
  skills: [zero-components]
skip_when:
  - No UI components in the feature
  - Playwright MCP not installed
---

# Playwright Testing

UI testing using Playwright MCP for component and feature verification.

## Prerequisites

Playwright MCP must be installed. If not available, notify user:

"Playwright MCP is required for UI testing but not installed. Please install it to enable automated UI verification."

## When to Use

### Component Verification
After implementing a component in the library (`vault/components/`), verify it works:
- Renders correctly
- States work (hover, active, disabled)
- Interactions work (click, input)

### Feature Verification
After integrating components into a feature:
- Full user flow works
- Components interact correctly
- No visual regressions

## Component Testing

### 1. Start the component demo server

Ensure the component demo is accessible (e.g., `vault/components/demos/button.html`).

### 2. Write Playwright test

Test each state and interaction:

```javascript
// Test default rendering
await page.goto('/demos/button.html');
await expect(page.locator('.button')).toBeVisible();

// Test hover state
await page.locator('.button').hover();
await expect(page.locator('.button')).toHaveClass(/hover/);

// Test click
await page.locator('.button').click();
await expect(page.locator('.button')).toHaveClass(/active/);

// Test disabled
await expect(page.locator('.button.disabled')).toBeDisabled();
```

### 3. Run and verify

Run tests via Playwright MCP. All should pass before proceeding.

## Feature Testing

### 1. Identify user flows

List the key user journeys:
- User can log in
- User can create item
- User can edit item
- User can delete item

### 2. Write end-to-end tests

```javascript
// Login flow
await page.goto('/login');
await page.fill('[name="email"]', 'test@example.com');
await page.fill('[name="password"]', 'password');
await page.click('button[type="submit"]');
await expect(page).toHaveURL('/dashboard');
```

### 3. Run full suite

All feature tests must pass before asking user for review.

## Checkpoints

| When | What to Test | Human Review? |
|------|--------------|---------------|
| Component done | Component in isolation | No - automated |
| Feature done | End-to-end flows | Yes - after tests pass |

## Troubleshooting

### MCP not available
Notify user and suggest installation.

### Tests failing
1. Check component renders correctly
2. Check selectors are correct
3. Check for timing issues (add waits)
4. Check demo server is running

### Visual differences
1. Update mockup if design changed
2. Get approval for visual changes
3. Update tests to match

## Success Criteria

- [ ] Playwright MCP available
- [ ] Component tests passing
- [ ] Feature tests passing
- [ ] User reviewed completed feature
```

**Step 2: Commit**

```bash
git add clojure-dev/skills/playwright-testing.md
git commit -m "✨ Add playwright-testing skill"
```

---

## Task 9: Create deepwiki-lookup skill

**Files:**
- Create: `skillbook-core/skills/deepwiki-lookup.md`

**Step 1: Create the skill file**

```markdown
---
name: deepwiki-lookup
description: Use when documentation is missing or unclear. Query deepwiki MCP for library docs, API references, and examples.
requires:
  tools: []
  mcps: [deepwiki]
  skills: []
skip_when:
  - Documentation is already known
  - deepwiki MCP not installed
---

# Deepwiki Lookup

Use deepwiki MCP to find documentation when information is missing.

## Prerequisites

Deepwiki MCP must be installed. If not available, fall back to:
1. Web search
2. Reading source code
3. Asking user

## When to Use

- Unknown library API
- Unclear function behavior
- Missing configuration options
- Need examples for a library
- Version-specific documentation

## How to Query

### Library documentation

Query for specific library:

```
Query deepwiki for: "malli schema validation examples"
Query deepwiki for: "reitit routing configuration"
Query deepwiki for: "shadow-cljs hot reload setup"
```

### API reference

Query for specific functions:

```
Query deepwiki for: "malli/validate function signature"
Query deepwiki for: "reitit.ring/router options"
```

### Configuration

Query for config options:

```
Query deepwiki for: "shadow-cljs.edn configuration options"
Query deepwiki for: "clj-kondo config.edn linters"
```

## Integrating Results

When you find useful documentation:

1. **Use it** - Apply the information to your task
2. **Note the source** - Mention where you found it
3. **Update local docs if helpful** - Add to project documentation

## Fallbacks

If deepwiki MCP unavailable:

1. **Web search** - Use WebSearch tool
2. **Source code** - Read the library source
3. **Ask user** - They may know or have docs

## Success Criteria

- [ ] Found needed documentation
- [ ] Applied information correctly
- [ ] Noted source for reference
```

**Step 2: Commit**

```bash
git add skillbook-core/skills/deepwiki-lookup.md
git commit -m "✨ Add deepwiki-lookup skill"
```

---

## Task 10: Update zero-components skill

**Files:**
- Modify: `clojure-dev/skills/zero-components.md`

**Step 1: Read current file**

Read `clojure-dev/skills/zero-components.md` to understand current content.

**Step 2: Add component library workflow and linking requirements**

Add these sections to the skill:

```markdown
## Component Library Workflow

Components MUST be created in the library before integration:

### 1. Mockup First

Before implementing, create and get approval for mockup:
- Use `ui-mockups` skill
- Create `mockups/<component>.html`
- Get user approval

### 2. Create in Library

Create component in `vault/components/`:
- Component code
- Documentation
- Demo in `vault/components/demos/`

### 3. Verify with Playwright

Run Playwright tests on the isolated component:
- All states render correctly
- Interactions work
- No console errors

### 4. Then Integrate

Only after Playwright verification:
- Import into feature
- Connect to application state
- Test in context

## Linking Requirements

When documenting a component:
- Link to approved mockup: `Design: [[mockups/component-name]]`
- Link to related components: `See also [[components/related]]`
- Link to features using this component: `Used in [[features/feature-name]]`

Example component doc header:

```markdown
# Button Component

Design: [[mockups/button]]
See also: [[components/icon]], [[components/spinner]]
Used in: [[features/login]], [[features/checkout]]
```
```

**Step 3: Commit**

```bash
git add clojure-dev/skills/zero-components.md
git commit -m "📝 Add component library workflow to zero-components"
```

---

## Task 11: Add linking requirements to existing skills

**Files:**
- Modify: `requirements-documentation/skills/adr-management.md`
- Modify: `requirements-documentation/skills/arc42-docs.md`
- Modify: `requirements-documentation/skills/init-obsidian-vault.md`
- Modify: `requirements-documentation/skills/iso25010-quality.md`
- Modify: `requirements-documentation/skills/scenari-specs.md`
- Modify: `architecture-modeling/skills/overarch-modeling.md`
- Modify: `security-compliance/skills/threagile-analysis.md`
- Modify: `project-management/skills/beads-tasks.md`

**Step 1: Add linking requirements to adr-management**

Add this section:

```markdown
## Linking Requirements

When creating an ADR:
- Link to superseded ADRs: `Supersedes [[0001-old-decision]]`
- Link to related ADRs: `Related: [[0003-related-decision]]`
- Link to arc42 sections: `See [[04-solution#relevant-heading]]`
- Link to beads task if applicable: `Task: [[beads/TASK-123]]`
- Link to glossary terms on first use: `[[glossary#term]]`
```

**Step 2: Add linking requirements to arc42-docs**

Add this section:

```markdown
## Linking Requirements

When updating arc42 sections:
- Link to ADRs: `Decision: [[decisions/0005-use-malli]]`
- Link to threat model: `Threats: [[security/threats#data-flow]]`
- Link to components: `Uses [[components/button]]`
- Link to specifications: `Spec: [[specifications/feature]]`
- Link to glossary terms: `[[glossary#term]]`
```

**Step 3: Add linking requirements to init-obsidian-vault**

Add this section:

```markdown
## Linking Conventions

The vault uses Obsidian `[[wiki-links]]` for interconnection:

| From | To | Example |
|------|----|---------|
| ADR | Related ADRs | `Supersedes [[0001-use-clojure]]` |
| ADR | Arc42 sections | `See [[04-solution#auth]]` |
| Arc42 | ADRs | `Decision: [[decisions/0005-use-malli]]` |
| Arc42 | Threats | `Threats: [[security/threats#data-flow]]` |
| Component | Mockup | `Design: [[mockups/button]]` |
| Any | Glossary | `[[glossary#term]]` on first use |

Create a glossary template at `vault/glossary.md` with instructions for adding terms.
```

**Step 4: Add linking requirements to iso25010-quality**

Add this section:

```markdown
## Linking Requirements

When defining quality requirements:
- Link to arc42 section 10: `See [[arc42/10-quality]]`
- Link to related ADRs: `Decision: [[decisions/0007-performance-target]]`
- Link to SLOs if defined: `SLO: [[requirements/slo-response-time]]`
```

**Step 5: Add linking requirements to scenari-specs**

Add this section:

```markdown
## Linking Requirements

When creating Gherkin specs:
- Link to components: `Components: [[components/button]], [[components/form]]`
- Link to ADRs: `Decision: [[decisions/0010-validation-approach]]`
- Link to beads task: `Task: [[beads/FEAT-456]]`
- Link to arc42 runtime section: `See [[arc42/06-runtime]]`
```

**Step 6: Add linking requirements to overarch-modeling**

Add this section:

```markdown
## Linking Requirements

When documenting the architecture model:
- Link to ADRs: `Decision: [[decisions/0002-microservices]]`
- Link to arc42 sections: `See [[arc42/05-building-blocks]]`
- Link to threat model: `Threats: [[security/threats]]`
```

**Step 7: Add linking requirements to threagile-analysis**

Add this section:

```markdown
## Linking Requirements

When documenting threats and mitigations:
- Link to ADRs for mitigations: `Mitigation: [[decisions/0008-encrypt-at-rest]]`
- Link to arc42 crosscutting: `See [[arc42/08-crosscutting#security]]`
- Link to policies: `Policy: [[policies/encryption.rego]]`
```

**Step 8: Add linking requirements to beads-tasks**

Add this section:

```markdown
## Linking Requirements

When creating tasks:
- Link to ADRs: `Decision: [[decisions/0005-feature-approach]]`
- Link to specs: `Spec: [[specifications/feature.feature]]`
- Link to components: `Component: [[components/button]]`
- Link to arc42 sections: `Docs: [[arc42/06-runtime#scenario]]`
```

**Step 9: Commit all changes**

```bash
git add requirements-documentation/skills/adr-management.md
git add requirements-documentation/skills/arc42-docs.md
git add requirements-documentation/skills/init-obsidian-vault.md
git add requirements-documentation/skills/iso25010-quality.md
git add requirements-documentation/skills/scenari-specs.md
git add architecture-modeling/skills/overarch-modeling.md
git add security-compliance/skills/threagile-analysis.md
git add project-management/skills/beads-tasks.md
git commit -m "📝 Add Obsidian linking requirements to vault-creating skills"
```

---

## Task 12: Update plugin manifest

**Files:**
- Modify: `skillbook-core/.claude-plugin/manifest.json`

**Step 1: Read current manifest**

Read `skillbook-core/.claude-plugin/manifest.json` to understand structure.

**Step 2: Add new skills and agents to manifest**

Ensure the manifest includes:
- `using-skillbook` skill
- `review-checkpoints` skill
- `deepwiki-lookup` skill
- `project-init` agent
- `clojure-developer` agent
- `new-project` command

**Step 3: Commit**

```bash
git add skillbook-core/.claude-plugin/manifest.json
git commit -m "📦 Update manifest with new skills, agents, and command"
```

---

## Task 13: Update clojure-dev plugin manifest

**Files:**
- Modify: `clojure-dev/.claude-plugin/manifest.json`

**Step 1: Read current manifest**

Read `clojure-dev/.claude-plugin/manifest.json`.

**Step 2: Add new skills**

Ensure manifest includes:
- `ui-mockups` skill
- `playwright-testing` skill

**Step 3: Commit**

```bash
git add clojure-dev/.claude-plugin/manifest.json
git commit -m "📦 Update clojure-dev manifest with new skills"
```

---

## Task 14: Final verification

**Step 1: Verify all files exist**

```bash
ls -la docs/workflow.md
ls -la skillbook-core/skills/using-skillbook.md
ls -la skillbook-core/skills/review-checkpoints.md
ls -la skillbook-core/skills/deepwiki-lookup.md
ls -la skillbook-core/agents/project-init.md
ls -la skillbook-core/agents/clojure-developer.md
ls -la skillbook-core/commands/new-project.md
ls -la clojure-dev/skills/ui-mockups.md
ls -la clojure-dev/skills/playwright-testing.md
```

**Step 2: Verify git status**

```bash
git status
git log --oneline -15
```

**Step 3: Create summary commit**

If all looks good:

```bash
git add -A
git commit -m "✨ Complete skillbook workflow enforcement implementation

- Add docs/workflow.md canonical reference
- Add using-skillbook skill for workflow enforcement
- Add project-init agent for initialization
- Add clojure-developer agent for TDD+REPL
- Add /skillbook:new-project slash command
- Add review-checkpoints skill
- Add ui-mockups skill
- Add playwright-testing skill
- Add deepwiki-lookup skill
- Update zero-components with library workflow
- Add linking requirements to 9 existing skills
- Update plugin manifests"
```

---

Plan complete and saved to `docs/plans/2025-11-30-skillbook-workflow-enforcement-implementation.md`.

**Two execution options:**

1. **Subagent-Driven (this session)** - I dispatch fresh subagent per task, review between tasks, fast iteration

2. **Parallel Session (separate)** - Open new session with executing-plans, batch execution with checkpoints

Which approach?
